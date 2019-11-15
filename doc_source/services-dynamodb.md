# How Amazon DynamoDB Uses AWS KMS<a name="services-dynamodb"></a>

[Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html) is a fully managed, scalable NoSQL database service\. DynamoDB integrates with AWS Key Management Service \(AWS KMS\) to support the [encryption at rest](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/EncryptionAtRest.html) server\-side encryption feature\.

With *encryption at rest*, DynamoDB transparently encrypts all customer data in a DynamoDB table, including its primary key and local and global [secondary indexes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html#HowItWorks.CoreComponents.SecondaryIndexes), whenever the table is persisted to disk\. \(If your table has a sort key, some of the sort keys that mark range boundaries are stored in plaintext in the table metadata\.\) When you access your table, DynamoDB decrypts the table data transparently\. You do not need to change your applications to use or manage encrypted tables\. 

Encryption at rest also protects [DynamoDB streams](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html), [global tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html), and [backups](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/BackupRestore.html) whenever these objects are saved to durable media\. Statements about tables in this topic apply to these objects, too\.

All DynamoDB tables are encrypted\. There is no option to enable or disable encryption for new or existing tables\. By default, all tables are encrypted under an [AWS owned customer master key](concepts.md#aws-owned-cmk) \(CMK\) in the DynamoDB service account\. However, you can select an option to encrypt some or all of your tables under an [AWS managed CMK](concepts.md#aws-managed-cmk) for DynamoDB in your account\. Encryption at rest does not support [customer managed CMKs](concepts.md#customer-cmk)\.

**Note**  
Before November 2018, encryption at rest was an optional feature that supported only the AWS managed CMK for DynamoDB\. If you enabled encryption at rest on any of your DynamoDB tables, they will continue to be encrypted under the AWS managed CMK unless you use the AWS Management Console or [UpdateTable](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_UpdateTable.html) operation to switch to an AWS owned CMK\.

**Client\-Side Encryption for DynamoDB**

In addition to encryption at rest, which is a *server\-side encryption* feature, AWS provides the [Amazon DynamoDB Encryption Client](https://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/)\. This *client\-side encryption* library enables you to protect your table data before submitting it to DynamoDB\. With server\-side encryption, your data is encrypted in transit over an HTTPS connection, decrypted at the DynamoDB endpoint, and then re\-encrypted before being stored in DynamoDB\. Client\-side encryption provides end\-to\-end protection for your data from its source to storage in DynamoDB\.

You can use the DynamoDB Encryption Client along with encryption at rest\. To help you decide if this strategy is right your DynamoDB data, see [Client\-Side or Server\-Side Encryption?](https://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/client-server-side.html) in the *Amazon DynamoDB Encryption Client Developer Guide*\.

**Topics**
+ [Using CMKs and Data Keys](#dynamodb-encrypt)
+ [Authorizing Use of the AWS Managed CMK](#dynamodb-authz)
+ [DynamoDB Encryption Context](#dynamodb-encryption-context)
+ [Monitoring DynamoDB Interaction with AWS KMS](#dynamodb-cmk-trail)

## Using CMKs and Data Keys<a name="dynamodb-encrypt"></a>

The DynamoDB encryption at rest feature uses an AWS KMS customer master key \(CMK\) and a hierarchy of data keys to protect your table data\. DynamoDB uses the same key hierarchy to protect DynamoDB streams, global tables, and backups when they are written to durable media\.

**Customer Master Key \(CMK\)**  
Encryption at rest protects your DynamoDB tables under an AWS KMS customer master key \(CMK\)\. By default, it uses an [AWS owned CMK](concepts.md#aws-owned-cmk), but DynamoDB supports an option to encrypt some or all of your tables under an [AWS managed CMK](concepts.md#aws-managed-cmk) for DynamoDB \(`aws/dynamodb`\) in your AWS account\. You can select the CMK for a table when you create or update the table, and you can make a different choice for each table\. The encryption at rest feature does not support the use of [customer managed CMKs](concepts.md#customer-cmk)\.  
Use the AWS managed CMK if you need any of the following features:  
+ You can [view the CMK](viewing-keys.md) and [view its key policy](key-policy-viewing.md)\. \(You cannot change the key policy\.\)
+ You can audit the encryption and decryption of your DynamoDB table by examining the DynamoDB API calls to AWS KMS in [AWS CloudTrail logs](#dynamodb-cmk-trail)\.
However, the AWS owned CMK is free of charge\. The AWS managed CMK [incurs a charge](https://aws.amazon.com/kms/pricing/) for each API call\.  
You can change the CMK for a table at any time, either in the DynamoDB console, or by using the [UpdateTable](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_UpdateTable.html) operation\. When you change the CMK, DynamoDB generates a new table key\. Then, it uses the new table key to re\-encrypt the data encryption keys\.  
The process of using a CMK to create a table key is the same no matter which CMK you choose\.

**Table Keys**  
DynamoDB uses the CMK for the table to generate and encrypt a unique [data key](concepts.md#data-keys) for the table, known as the *table key*\. The table key persists for the lifetime of the encrypted table\.   
The table key is used as a key encryption key\. DynamoDB uses this table key to protect data encryption keys that are used to encrypt the table data\. DynamoDB generates a unique data encryption key for each underlying structure in a table, but multiple table items might be protected by the same data encryption key\.  

![\[Encrypting a DynamoDB Table with Encryption at Rest\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/service-ddb-encrypt.png)
When you first access an encrypted table, DynamoDB sends a request to AWS KMS to use the CMK to decrypt the table key\. Then, it uses the plaintext table key to decrypt the data encryption keys, and uses the plaintext data encryption keys to decrypt table data\.  
DynamoDB generates, uses, and stores the table key and data encryption keys outside of AWS KMS\. It protects all keys with [Advanced Encryption Standard](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) \(AES\) encryption and 256\-bit encryption keys\. Then, it stores the encrypted keys with the encrypted data so they are available to decrypt the table data on demand\.

**Table Key Caching**  
To avoid calling AWS KMS for every DynamoDB operation, DynamoDB caches the plaintext table keys for each connection in memory\. If DynamoDB gets a request for the cached table key after five minutes of inactivity, it sends a new request to AWS KMS to decrypt the table key\. This call will capture any changes made to the access policies of the CMK in AWS KMS or AWS Identity and Access Management \(IAM\) since the last request to decrypt the table key\.

## Authorizing Use of the AWS Managed CMK<a name="dynamodb-authz"></a>

If you use an [AWS managed CMK](concepts.md#aws-managed-cmk) in your account to protect your DynamoDB table, the policies on that CMK must give DynamoDB permission to use it on your behalf\. The authorization context on the AWS managed CMK for DynamoDB includes its key policy and grants that delegate the permissions to use it\. 

Because the AWS managed CMK is in your account, you can view its policies and grants\. But, because it is managed by AWS, you cannot change the policies\.

DynamoDB does not need additional authorization to use the default [AWS owned CMK](concepts.md#master_keys) to protect the DynamoDB tables in your AWS account\.

**Topics**
+ [CMK Key Policy](#dynamodb-policies)
+ [Using Grants to Authorize DynamoDB](#dynamodb-grants)

### CMK Key Policy<a name="dynamodb-policies"></a>

When DynamoDB uses the [AWS managed CMK](concepts.md#aws-managed-cmk) for DynamoDB \(`aws/dynamodb`\) in cryptographic operations, it does so on behalf of the user who is accessing the [DynamoDB resource](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/access-control-overview.html)\. The key policy on the AWS managed CMK gives all users in the account permission to use the AWS managed CMK for specified operations\. But permission is granted only when DynamoDB makes the request on the user's behalf\. The [ViaService condition](policy-conditions.md#conditions-kms-via-service) in the key policy does not allow any user to use the AWS managed CMK unless the request originates with the DynamoDB service\.

This key policy, like the policies of all AWS managed keys, is established by AWS\. You cannot change it, but you can view it at any time\. For details, see [Viewing a Key Policy](key-policy-viewing.md)\.

The policy statements in the key policy have the following effect:
+ Allow users in the account to use the AWS managed CMK for DynamoDB in cryptographic operations when the request comes from DynamoDB on their behalf\. The policy also allows users to [create grants](#dynamodb-grants) for the CMK\.
+ Allows the AWS account root user to view the properties of the AWS managed CMK for DynamoDB and to [revoke the grant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) that allows DynamoDB to use the CMK\. DynamoDB uses [grants](#dynamodb-grants) for ongoing maintenance operations\.
+ Allows DynamoDB to perform read\-only operations to find the AWS managed CMK for DynamoDB in your account\.

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

In addition to key policies, DynamoDB uses grants to set permissions on the [AWS managed CMK](concepts.md#aws-managed-cmk) for DynamoDB \(aws/dynamodb\)\. To view the grants on the `aws/dynamodb` CMK in your account, use the [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation\. DynamoDB does not need grants, or any additional permissions, to use the [AWS owned CMK](concepts.md#aws-owned-cmk) to protect your table\.

DynamoDB uses the grant permissions when it performs background system maintenance and continuous data protection tasks\. It also uses grants to generate [table keys](#dynamodb-encrypt)\.

Each grant is specific to a table\. If the account includes multiple tables encrypted under the same AWS managed CMK, there is a grant of each type for each table\. The grant is constrained by the [DynamoDB encryption context](#dynamodb-encryption-context), which includes the table name and the AWS account ID, and it includes permission to the [retire the grant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html) if it is no longer needed\. 

To create the grants, DynamoDB calls [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) on behalf of the user who created the encrypted table\. Permission to create the grant comes from the [key policy](#dynamodb-policies), which allows account users to call `CreateGrant` on the AWS managed CMK only when DynamoDB makes the request on an authorized user's behalf\. 

The key policy also allows the account to [revoke the grant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) on the AWS managed CMK\. However, if you revoke the grant on an active encrypted table, DynamoDB will not be able to protect and maintain the table\.

## DynamoDB Encryption Context<a name="dynamodb-encryption-context"></a>

An [encryption context](concepts.md#encrypt_context) is a set of key–value pairs that contain arbitrary nonsecret data\. When you include an encryption context in a request to encrypt data, AWS KMS cryptographically binds the encryption context to the encrypted data\. To decrypt the data, you must pass in the same encryption context\. 

DynamoDB uses the same encryption context in all AWS KMS cryptographic operations\. If you use an [AWS managed CMK](concepts.md#aws-managed-cmk) to protect your DynamoDB table, you can use the encryption context to identify use of the CMK in audit records and logs\. It also appears in plaintext in logs, such as [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) and [Amazon CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)\. 

The encryption context can also be used as a condition for authorization in policies and grants\. DynamoDB uses the encryption context to constrain the [grants](#dynamodb-grants) that allow access to the AWS managed CMK in your account and region\.

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

## Monitoring DynamoDB Interaction with AWS KMS<a name="dynamodb-cmk-trail"></a>

If you use an [AWS managed CMK](concepts.md#aws-managed-cmk) to protect your DynamoDB tables, you can use AWS CloudTrail logs to track the requests that DynamoDB sends to AWS KMS on your behalf\.

The `GenerateDataKey`, `Decrypt`, and `CreateGrant` requests are discussed in this section\. In addition, if you use an AWS managed CMK for encryption at rest, DynamoDB uses a [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation to determine whether an `aws/dynamodb` CMK exists in the account and region\. It also uses a [RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html) operation to remove a grant when you delete a table\. 

**GenerateDataKey**  
When you enable encryption at rest on a table, DynamoDB creates a unique table key\. It sends a *[GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)* request to AWS KMS that specifies the CMK for the table\.   
The event that records the `GenerateDataKey` operation is similar to the following example event\. The user is the DynamoDB service account\. The parameters include the Amazon Resource Name \(ARN\) of the AWS managed CMK, a key specifier that requires a 256\-bit key, and the [encryption context](#dynamodb-encryption-context) that identifies the table and the AWS account\.  

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
When you access an encrypted DynamoDB table, DynamoDB needs to decrypt the table key so that it can decrypt the keys below it in the hierarchy\. It then decrypts the data in the table\. To decrypt the table key\. DynamoDB sends a [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) request to AWS KMS that specifies the CMK for the table\.  
The event that records the `Decrypt` operation is similar to the following example event\. The user is the principal in your AWS account who is accessing the table\. The parameters include the encrypted table key \(as a ciphertext blob\) and the [encryption context](#dynamodb-encryption-context) that identifies the table and the AWS account\. AWS KMS derives the ID of the CMK from the ciphertext\.   

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
When you use an [AWS managed CMK](concepts.md#aws-managed-cmk) to protect your DynamoDB table, DynamoDB uses [grants](#dynamodb-grants) to allow the service to perform continuous data protection and maintenance and durability tasks\. These grants are not required on [AWS owned CMKs](concepts.md#aws-owned-cmk)\.  
The grants that DynamoDB creates are specific to a table\. The principal in the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) request is the user who created the table\.   
The event that records the `CreateGrant` operation is similar to the following example event\. The parameters include the Amazon Resource Name \(ARN\) of the CMK for the table, the grantee principal and retiring principal \( the DynamoDB service\), and the operations that the grant covers\. It also includes a constraint that requires all encryption operation use the specified [encryption context](#dynamodb-encryption-context)\.  

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