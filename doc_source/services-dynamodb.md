# How Amazon DynamoDB Uses AWS KMS<a name="services-dynamodb"></a>

[Amazon DynamoDB](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html) is a fully managed, scalable NoSQL database service\. DynamoDB integrates with AWS Key Management Service \(AWS KMS\) to support an optional [encryption at rest](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/EncryptionAtRest.html) server\-side encryption feature\. 

With *encryption at rest*, DynamoDB transparently encrypts all customer data in an encrypted table, including its primary key and local and global [secondary indexes](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html#HowItWorks.CoreComponents.SecondaryIndexes), whenever the table is persisted to disk\. \(If your table has a sort key, some of the sort keys that mark range boundaries are stored in plaintext in the table metadata\.\) When you access an encrypted table, DynamoDB decrypts the table data transparently\. You do not need to change your applications to use or manage encrypted tables\.

You enable encryption at rest when you create a table\. Each table is encrypted independently, so you can enable encryption on selected tables, while leaving others unencrypted\. 

Encryption at rest uses AWS KMS customer master keys in your AWS to protect your DynamoDB tables on disk\. This integration also enables you to audit access to your DynamoDB tables by examining the DynamoDB API calls to AWS KMS in audit logs and trails\.

**Topics**
+ [Using CMKs and Data Keys](#dynamodb-encrypt)
+ [Authorizing Use of the Service Default Key](#dynamodb-authz)
+ [DynamoDB Encryption Context](#dynamodb-encryption-context)
+ [Monitoring DynamoDB Interaction with AWS KMS](#dynamodb-cmk-fail)

## Using CMKs and Data Keys<a name="dynamodb-encrypt"></a>

The DynamoDB encryption at rest feature uses an AWS KMS CMK and a hierarchy of data keys to protect your table data\. 

**Service Default Key**  
When you create an encrypted table, DynamoDB creates a unique AWS managed [customer master key](concepts.md#master_keys) \(CMK\) in each region of your AWS account, if one does not already exist\. This CMK, `aws/dynamodb`, is known as the *service default key*\. Like all CMKs, the service default key never leaves AWS KMS unencrypted\. The encryption at rest feature does not support the use of customer managed CMKs\.  
The service default key in each region protects the table keys for tables in that region\.

**Table Keys**  
DynamoDB uses the service default key in AWS KMS to generate and encrypt a unique [data key](concepts.md#data-keys) for each table, known as the *table key*\. The table key persists for the lifetime of the encrypted table\.   
The table key is used as a master key\. DynamoDB generates data encryption keys and uses them to encrypt table data\. Then, it uses the table key to protect the data encryption keys\. DynamoDB generates a unique data encryption key for each underlying structure in a table, but multiple table items might be protected by the same data encryption key\.   

![\[Encrypting a DynamoDB Table with Encryption at Rest\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/service-ddb-encrypt.png)
When you access an encrypted table, DynamoDB sends a request to AWS KMS to use the service default key in AWS KMS to decrypt the table key\. Then, it uses the plaintext table key to decrypt the data encryption keys, and uses the plaintext data encryption keys to decrypt table data\.  
DynamoDB generates, uses, and stores the table key and data encryption keys outside of AWS KMS\. It protects all keys with [Advanced Encryption Standard](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) \(AES\) encryption and 256\-bit encryption keys\. It removes plaintext keys as soon as possible after using them\. Then, it stores the encrypted keys with the encrypted data so they are available to decrypt the table data on demand\.

**Table Key Caching**  
To avoid calling AWS KMS for every DynamoDB operation, DynamoDB caches the plaintext table keys for each principal in memory\. If DynamoDB gets a request for the cached table key after five minutes of inactivity, it sends a new request to AWS KMS to decrypt the table key\. This call will capture any changes to the authorization context of the service default key\.  
DynamoDB caches the table key in memory for up to 12 hours\. This enables DynamoDB to maintain your access to your table data in the extraordinary circumstance that AWS KMS becomes unresponsive for an extended period of time\.

## Authorizing Use of the Service Default Key<a name="dynamodb-authz"></a>

The authorization context on the DynamoDB [service default key](#dynamodb-encrypt) includes its key policy and grants that delegate the permissions to use it\. 

### Service Default Key Policy<a name="dynamodb-policies"></a>

When DynamoDB uses the service default key in cryptographic operations, it does so on behalf of the user who is accessing the [DynamoDB resource](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/access-control-overview.html)\. The key policy on the service default key gives users in the account permission to use the service default key for specified operations\. But permission is granted only when DynamoDB makes the request on the user's behalf\. The key policy does not allow any user to use the service default key directly\.

This key policy, like the policies of all [AWS managed keys](concepts.md#master_keys), is established by the service\. You cannot change it, but you can view it at any time\. To get the key policy for the DynamoDB service default key in your account, use the [GetKeyPolicy](http://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation\. 

The policy statements in the key policy have the following effect:
+ Allow users in the account to use the service default key for cryptographic operations when the request comes from DynamoDB on their behalf\. The policy also allows users to [create grants](#dynamodb-grants) for the CMK\.
+ Allows the AWS account root user to view the CMK properties and to [revoke the grant](http://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) that allows DynamoDB to use the service default key\. DynamoDB uses [grants](#dynamodb-grants) for ongoing maintenance operations\.
+ Allows DynamoDB to perform read\-only operations to get the service default key in your account\.

```
{
  "Version" : "2012-10-17",
  "Id" : "auto-dynamodb-1",
  "Statement" : [ {
    "Sid" : "Allow access through Amazon DynamoDB for all principals in the account that are authorized to use Amazon DynamoDB",
    "Effect" : "Allow",
    "Principal" : {
      "AWS" : "*"
    },
    "Action" : [ "kms:Encrypt", "kms:Decrypt", "kms:ReEncrypt*", "kms:GenerateDataKey*", "kms:CreateGrant", "kms:DescribeKey" ],
    "Resource" : "*",
    "Condition" : {
      "StringEquals" : {
        "kms:CallerAccount" : "111122223333",
        "kms:ViaService" : "dynamodb.us-west-2.amazonaws.com"
      }
    }
  }, {
    "Sid" : "Allow direct access to key metadata to the account",
    "Effect" : "Allow",
    "Principal" : {
      "AWS" : "arn:aws:iam::111122223333:root"
    },
    "Action" : [ "kms:Describe*", "kms:Get*", "kms:List*", "kms:RevokeGrant" ],
    "Resource" : "*"
  }, {
    "Sid" : "Allow DynamoDB Service with service principal name dynamodb.amazonaws.com to describe the key directly",
    "Effect" : "Allow",
    "Principal" : {
      "Service" : "dynamodb.amazonaws.com"
    },
    "Action" : [ "kms:Describe*", "kms:Get*", "kms:List*" ],
    "Resource" : "*"
  } ]
}
```

### Using Grants to Authorize DynamoDB<a name="dynamodb-grants"></a>

In addition to key policies, DynamoDB uses grants to set permissions on the DynamoDB [service default key](#dynamodb-encrypt)\. To view the grants on the DynamoDB service default key in your account, use the [ListGrants](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation\.

DynamoDB uses the grant permissions when it performs background system maintenance and continuous data protection tasks\. It also uses grants to generate [table keys](#dynamodb-encrypt)\.

Each grant is specific to a table\. If the account includes multiple encrypted tables, there is a grant of each type for each table\. The grant is constrained by the [DynamoDB encryption context](#dynamodb-encryption-context), which includes the table name and the AWS account ID, and it includes permission to the [retire the grant](http://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html) if it is no longer needed\. 

To create the grants, DynamoDB calls `CreateGrant` on behalf of the user who created the encrypted table\. Permission to create the grant comes from the [key policy](#dynamodb-policies), which allows account users to call [CreateGrant](http://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) on the service default key only when DynamoDB makes the request on an authorized user's behalf\. 

The key policy also allows the account root to [revoke the grant](http://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) on the service default key\. However, if you revoke the grant on an active encrypted table, DynamoDB will not be able to protect and maintain the table\.

## DynamoDB Encryption Context<a name="dynamodb-encryption-context"></a>

An [encryption context](concepts.md#encrypt_context) is a set of key–value pairs that contain arbitrary nonsecret data\. When you include an encryption context in a request to encrypt data, AWS KMS cryptographically binds the encryption context to the encrypted data\. To decrypt the data, you must pass in the same encryption context\. 

DynamoDB uses the same encryption context in all KMS cryptographic operations\. You can use the encryption context to identify a cryptographic operation in audit records and logs\. It also appears in plaintext in logs, such as [AWS CloudTrail](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) and [Amazon CloudWatch Logs](http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)\. 

The encryption context can also be used as a condition for authorization in policies and grants\. DynamoDB uses it to constrain the [grants](#dynamodb-grants) that allow access to the service default key\.

In its requests to AWS KMS, DynamoDB uses an encryption context with two key–value pairs\.

```
"encryptionContextSubset": {
    "aws:dynamodb:tableName": "Books"
    "aws:dynamodb:subscriberId": "111122223333"
}
```
+ **Table** – The first key–value pair identifies the table that DynamoDB is encrypting\. The key is `aws:dynamodb:tableName`\. The value is the name of the table\.

  ```
  "aws:dynamodb:tableName": "<table-name>"
  ```

  For example:

  ```
  "aws:dynamodb:tableName": "Books"
  ```
+ **Account** – The second key–value pair identifies the AWS account\. The key is `aws:dynamodb:subscriberId`\. The value is the account ID\.

  ```
  "aws:dynamodb:subscriberId": "<account-id>"
  ```

  For example:

  ```
  "aws:dynamodb:subscriberId": "111122223333"
  ```

## Monitoring DynamoDB Interaction with AWS KMS<a name="dynamodb-cmk-fail"></a>

You can use AWS CloudTrail and Amazon CloudWatch Logs to track the requests that DynamoDB sends to AWS KMS on your behalf\.

The `GenerateDataKey`, `Decrypt`, and `CreateGrant` requests are discussed in this section\. In addition, DynamoDB uses a [DescribeKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation to determine whether a DynamoDB service default key exists in the account and region\. It also uses a [RetireGrant](http://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html) operation to remove a grant when you delete a table\. 

**GenerateDataKey**  
When you enable encryption at rest on a table, DynamoDB creates a unique table key\. It sends a *[GenerateDataKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)* request to AWS KMS that specifies the `aws/dynamodb` service default key in your account\.   
The event that records the `GenerateDataKey` operation is similar to the following example event\. The user is DynamoDB\. The parameters include the Amazon Resource Name \(ARN\) of the service default key, a key specifier that requires a 256\-bit key, and the [encryption context](#dynamodb-encryption-context) that identifies the table and the AWS account\.  

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "AWSService", 
        "invokedBy": "dynamodb.amazonaws.com" 
    },
    "eventTime": "2018-02-14T00:15:17Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "GenerateDataKey",
    "awsRegion": "us-west-2",
    "sourceIPAddress": "dynamodb.amazonaws.com",
    "userAgent": "dynamodb.amazonaws.com",
    "requestParameters": {
        "encryptionContext": {
            "aws:dynamodb:tableName": "Services",
            "aws:dynamodb:subscriberId": "111122223333"
        }, 
        "keySpec": "AES_256", 
        "keyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
    }, 
    "responseElements": null,
    "requestID": "229386c1-111c-11e8-9e21-c11ed5a52190",
    "eventID": "e3c436e9-ebca-494e-9457-8123a1f5e979",
    "readOnly": true,
    "resources": [
        {
            "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "accountId": "111122223333",
            "type": "AWS::KMS::Key" 
        } 
    ],
    "eventType": "AwsApiCall",
    "recipientAccountId": "111122223333",
    "sharedEventID": "bf915fa6-6ceb-4659-8912-e36b69846aad"
}
```

**Decrypt**  
When you access an encrypted DynamoDB table, DynamoDB needs to decrypt the table key so that it can decrypt the keys below it in the hierarchy\. It then decrypts the data in the table\. To decrypt the table key\. DynamoDB sends a [Decrypt](http://docs.aws.amazon.com/kms/latest/APIReference//API_Decrypt.html) request to AWS KMS that specifies the `aws/dynamodb` service default key in your account\.  
The event that records the `Decrypt` operation is similar to the following example event\. The user is the principal in your AWS account who is accessing the table\. The parameters include the encrypted table key \(as a ciphertext blob\) and the [encryption context](#dynamodb-encryption-context) that identifies the table and the AWS account\. AWS KMS derives the ID of the service default key from the ciphertext\.   

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "AssumedRole",
        "principalId": "AROAIGDTESTANDEXAMPLE:user01",
        "arn": "arn:aws:sts::111122223333:assumed-role/Admin/user01",
        "accountId": "111122223333",
        "accessKeyId": "AKIAIOSFODNN7EXAMPLE",
        "sessionContext": {
            "attributes": {
                "mfaAuthenticated": "false", 
                "creationDate": "2018-02-14T16:42:15Z"
            },
            "sessionIssuer": {
                "type": "Role",
                "principalId": "AROAIGDT3HGFQZX4RY6RU",
                "arn": "arn:aws:iam::111122223333:role/Admin",
                "accountId": "111122223333",
                "userName": "Admin" 
            }
        },
        "invokedBy": "dynamodb.amazonaws.com"
    },
    "eventTime": "2018-02-14T16:42:39Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "Decrypt",
    "awsRegion": "us-west-2",
    "sourceIPAddress": "dynamodb.amazonaws.com",
    "userAgent": "dynamodb.amazonaws.com",
    "requestParameters": 
    {
        "encryptionContext":
        {
            "aws:dynamodb:tableName": "Books",
            "aws:dynamodb:subscriberId": "111122223333" 
        }
    }, 
    "responseElements": null, 
    "requestID": "11cab293-11a6-11e8-8386-13160d3e5db5",
    "eventID": "b7d16574-e887-4b5b-a064-bf92f8ec9ad3", 
    "readOnly": true, 
    "resources": [ 
        {
            "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "accountId": "111122223333", 
            "type": "AWS::KMS::Key" 
        }
    ],
    "eventType": "AwsApiCall", 
    "recipientAccountId": "111122223333"
}
```

**CreateGrant**  
DynamoDB uses [grants](#dynamodb-grants) to allow the service to perform continuous data protection and maintenance and durability tasks\.   
The grants that DynamoDB creates are specific to a table\. The principal in the [CreateGrant](http://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) request is the user who created the table\.   
The event that records the `CreateGrant` operation is similar to the following example event\. The parameters include the Amazon Resource Name \(ARN\) of the service default key, the grantee principal and retiring principal \( the DynamoDB service\), and the operations that the grant covers\. It also includes a constraint that requires all encryption operation use the specified [encryption context](#dynamodb-encryption-context)\.  

```
{ 
    "eventVersion": "1.05", 
    "userIdentity": 
    { 
        "type": "AssumedRole", 
        "principalId": "AROAIGDTESTANDEXAMPLE:user01", 
        "arn": "arn:aws:sts::111122223333:assumed-role/Admin/user01", 
        "accountId": "111122223333", 
        "accessKeyId": "AKIAIOSFODNN7EXAMPLE", 
        "sessionContext": { 
            "attributes": { 
                "mfaAuthenticated": "false", 
                "creationDate": "2018-02-14T00:12:02Z" 
            }, 
            "sessionIssuer": { 
                "type": "Role", 
                "principalId": "AROAIGDTESTANDEXAMPLE", 
                "arn": "arn:aws:iam::111122223333:role/Admin", 
                "accountId": "111122223333", 
                "userName": "Admin" 
            }
        }, 
        "invokedBy": "dynamodb.amazonaws.com" 
    }, 
    "eventTime": "2018-02-14T00:15:15Z", 
    "eventSource": "kms.amazonaws.com", 
    "eventName": "CreateGrant", 
    "awsRegion": "us-west-2", 
    "sourceIPAddress": "dynamodb.amazonaws.com", 
    "userAgent": "dynamodb.amazonaws.com", 
    "requestParameters": { 
        "keyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab", 
        "retiringPrincipal": "dynamodb.us-west-2.amazonaws.com", 
        "constraints": { 
            "encryptionContextSubset": {
                "aws:dynamodb:tableName": "Books",
                "aws:dynamodb:subscriberId": "111122223333" 
            } 
        }, 
        "granteePrincipal": "dynamodb.us-west-2.amazonaws.com", 
        "operations": [ 
            "DescribeKey", 
            "GenerateDataKey", 
            "Decrypt", 
            "Encrypt", 
            "ReEncryptFrom", 
            "ReEncryptTo", 
            "RetireGrant" 
        ] 
    }, 
    "responseElements": { 
        "grantId": "5c5cd4a3d68e65e77795f5ccc2516dff057308172b0cd107c85b5215c6e48bde" 
    }, 
    "requestID": "2192b82a-111c-11e8-a528-f398979205d8", 
    "eventID": "a03d65c3-9fee-4111-9816-8bf96b73df01", 
    "readOnly": false, 
    "resources": [ 
        { 
            "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "accountId": "111122223333", 
            "type": "AWS::KMS::Key" 
        } 
    ], 
    "eventType": "AwsApiCall",
    "recipientAccountId": "111122223333"
}
```