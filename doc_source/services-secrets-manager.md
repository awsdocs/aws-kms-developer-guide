# How AWS Secrets Manager uses AWS KMS<a name="services-secrets-manager"></a>

[AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/Introduction.html) is an AWS service that encrypts and stores your secrets, and transparently decrypts and returns them to you in plaintext\. It's designed especially to store application secrets, such as login credentials, that change periodically and should not be hard\-coded or stored in plaintext in the application\. In place of hard\-coded credentials or table lookups, your application calls Secrets Manager\.

Secrets Manager also supports features that periodically rotate the secrets associated with commonly used databases\. It always encrypts newly rotated secrets before they are stored\.

Secrets Manager integrates with AWS Key Management Service \(AWS KMS\) to encrypt every version of every secret with a unique [data key](concepts.md#data-keys) that is protected by an AWS KMS [customer master key](concepts.md#master_keys) \(CMK\)\. This integration protects your secrets under encryption keys that never leave AWS KMS unencrypted\. It also enables you to set custom permissions on the CMK and audit the operations that generate, encrypt, and decrypt the data keys that protect your secrets\.

**Topics**
+ [Protecting the secret value](#asm-secret)
+ [Encrypting and decrypting secrets](#asm-encrypt)
+ [Using your AWS KMS CMK](#asm-using-cmk)
+ [Authorizing use of the CMK](#asm-authz)
+ [Secrets Manager encryption context](#asm-encryption-context)
+ [Monitoring Secrets Manager interaction with AWS KMS](#asm-logs)

## Protecting the secret value<a name="asm-secret"></a>

To protect a secret, Secrets Manager encrypts the *secret value* in a secret\.

In Secrets Manager, a [secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/terms-concepts.html#term_secret) consists of a *secret value*, also known as *protected secret text* or *encrypted secret data*, and related metadata and version information\. The secret value can be any string or binary data of up to 65,536 bytes, but it is typically a collection of name\-value pairs that comprise the login information for a server or database\.

Secrets Manager always encrypts the entire secret value before it stores the secret\. It decrypts the secret value transparently whenever you get or change the secret value\. There is no option to enable or disable encryption\. To encrypt and decrypt the secret value, Secrets Manager uses AWS KMS\.

## Encrypting and decrypting secrets<a name="asm-encrypt"></a>

To protect secrets, Secrets Manager uses [envelope encryption](concepts.md#enveloping) with AWS KMS [customer master keys](concepts.md#master_keys) \(CMKs\) and [data keys](concepts.md#data-keys)\.

Secrets Manager uses a unique data key to protect each secret value\. Whenever the secret value in a secret changes, Secrets Manager generates a new data key to protect it\. The data key is encrypted under an AWS KMS CMK and stored in the metadata of the secret, as shown in the following image\. To decrypt the secret, Secrets Manager must first decrypt the encrypted data key using the CMK in AWS KMS\. 

![\[Encrypting the secret value in a version of a secret\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/secrets-manager.png)

### An AWS KMS CMK for each secret<a name="cmk-per-secret"></a>

Each secret is associated with an AWS managed or customer managed [customer master key](concepts.md#master_keys) \(CMK\)\. Customer managed CMKs allow authorized users to [control access](control-access.md) to the CMK through policies and grants, manage [automatic rotation](rotate-keys.md), and use [imported key material](importing-keys.md)\.

**Important**  
Secrets Manager supports only [symmetric CMKs](symm-asymm-concepts.md#symmetric-cmks)\. You cannot use an [asymmetric CMK](symm-asymm-concepts.md#asymmetric-cmks) to encrypt your secrets\. For help determining whether a CMK is symmetric or asymmetric, see [Identifying symmetric and asymmetric CMKs](find-symm-asymm.md)\.

When you create a new secret, you can specify any symmetric customer managed CMK in the account and region, or the AWS managed CMK for Secrets Manager, `aws/secretsmanager`\. If you do not specify a CMK, or you select the console default value, **DefaultEncryptionKey**, Secrets Manager creates the `aws/secretsmanager` CMK, if it does not exist, and associates it with the secret\. You can use the same CMK or different CMKs for each secret in your account\.

You can change the CMK for a secret at any time, either in the Secrets Manager console, or by using the [UpdateSecret](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_UpdateSecret.html) operation\. When you change the CMK, Secrets Manager does not re\-encrypt the existing secret value under the new CMK\. However, the next time that the secret value changes, Secrets Manager encrypts it under the new CMK\.

To find the CMK that is associated with a secret, use the [ListSecrets](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_ListSecrets.html) or [DescribeSecret](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_DescribeSecret.html) operations\. When the secret is associated with the AWS managed CMK for Secrets Manager \(`aws/secretsmanager`\), these operations do not return a CMK identifier\.

Secrets Manager does not use the CMK to encrypt the secret value directly\. Instead, it uses the CMK to generate and encrypt a unique data key, and it uses the data key to encrypt the secret value\.

### A unique data key for each secret value<a name="dek-per-value"></a>

Every time that you create or change the secret value in a secret, Secrets Manager uses the CMK that is associated with the secret to generate and encrypt a unique 256\-bit Advanced Encryption Standard \(AES\) symmetric [data key](concepts.md#data-keys)\. Secrets Manager uses the plaintext data key to encrypt the secret value outside of AWS KMS, and then removes it from memory\. It stores the encrypted copy of the data key in the metadata of the secret\.

The secret value is ultimately protected by the CMK, which never leaves AWS KMS unencrypted\. Before Secrets Manager can decrypt the secret, it must ask AWS KMS to decrypt the encrypted data key\. 

### Encrypting a secret value<a name="asm-encrypt"></a>

To encrypt the secret value in a secret, Secrets Manager uses the following process\.

1. Secrets Manager calls the AWS KMS [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation with the ID of the CMK for the secret and a request for a 256\-bit AES symmetric key\. AWS KMS returns a plaintext data key and a copy of that data key encrypted under the CMK\.

1. Secrets Manager uses the plaintext data key and the Advanced Encryption Standard \(AES\) algorithm to encrypt the secret value outside of AWS KMS\. It removes the plaintext key from memory as soon as possible after using it\.

1. Secrets Manager stores the encrypted data key in the metadata of the secret so it is available to decrypt the secret value\. However, none of the Secrets Manager APIs return the encrypted secret or the encrypted data key\.

### Decrypting a secret value<a name="asm-decrypt"></a>

To decrypt an encrypted secret value, Secrets Manager must first decrypt the encrypted data key\. Because the data key is encrypted under the CMK for the secret in AWS KMS, Secrets Manager must make a request to AWS KMS\.

To decrypt an encrypted secret value:

1.  Secrets Manager calls the AWS KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation and passes in the encrypted data key\.

1. AWS KMS uses the CMK for the secret to decrypt the data key\. It returns the plaintext data key\.

1. Secrets Manager uses the plaintext data key to decrypt the secret value\. Then it removes the data key from memory as soon as possible\.

## Using your AWS KMS CMK<a name="asm-using-cmk"></a>

Secrets Manager uses the [customer master key](concepts.md#master_keys) \(CMK\) that is associated with a secret to generate a data key for each secret value\. It also uses the CMK to decrypt that data key when it needs to decrypt the encrypted secret value\. You can track the requests and responses in AWS CloudTrail events, [Amazon CloudWatch Logs](#asm-logs), and audit trails\.

The following Secrets Manager operations trigger a request to use your AWS KMS CMK\.

**GenerateDataKey**  
Secrets Manager calls the AWS KMS [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation in response to the following Secrets Manager operations\.  
+ [CreateSecret](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_CreateSecret.html) – If the new secret includes a secret value, Secrets Manager requests a new data key to encrypt it\. 
+ [PutSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_PutSecretValue.html)– Secrets Manager requests a new data key to encrypt the specified secret value\.
+ [UpdateSecret](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_UpdateSecret.html) – If the update changes the secret value, Secrets Manager requests a new data key to encrypt the new secret value\.
The [RotateSecret](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_RotateSecret.html) operation does not call `GenerateDataKey`, because it does not change the secret value\. However, if the Lambda function that `RotateSecret` invokes changes the secret value, its call to the `PutSecretValue` operation triggers a `GenerateDataKey` request\.

**Decrypt**  
To decrypt an encrypted secret value, Secrets Manager calls the AWS KMS [Decrypt](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_Decrypt.html) operation to decrypt the encrypted data key in the secret\. Then, it uses the plaintext data key to decrypt the encrypted secret value\.  
Secrets Manager calls the [Decrypt](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_Decrypt.html) operation in response to the following Secrets Manager operations\.   
+ [GetSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_GetSecretValue.html) – Secrets Manager decrypts the secret value before returning it to the caller\.
+ [PutSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_PutSecretValue.html) and [UpdateSecret](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_UpdateSecret.html) – Most `PutSecretValue` and `UpdateSecret` requests do not trigger a `Decrypt` operation\. However, when a `PutSecretValue` or `UpdateSecret` request attempts to change the secret value in an existing version of a secret, Secrets Manager decrypts the existing secret value and compares it to the secret value in the request to confirm that they are the same\. This action ensures the that Secrets Manager operations are idempotent\.

**Validating access to the CMK**  
When you establish or change the CMK that is associated with secret, Secrets Manager calls the `GenerateDataKey` and `Decrypt` operations with the specified CMK\. These calls confirm that the caller has permission to use the CMK for these operation\. Secrets Manager discards the results of these operations; it does not use them in any cryptographic operation\.  
You can identify these validation calls because the value of the `SecretVersionId` key [encryption context](#asm-encryption-context) in these requests is `RequestToValidateKeyAccess`\.  
In the past, Secrets Manager validation calls did not include an encryption context\. You might find calls with no encryption context in older AWS CloudTrail logs\.

## Authorizing use of the CMK<a name="asm-authz"></a>

When Secrets Manager uses a [customer master key](concepts.md#master_keys) \(CMK\) in cryptographic operations, it acts on behalf of the user who is creating or changing the secret value in the secret\.

To use the AWS KMS customer master key \(CMK\) for a secret on your behalf, the user must have the following permissions\. You can specify these required permissions in an IAM policy or key policy\.
+ kms:GenerateDataKey
+ kms:Decrypt

To allow the CMK to be used only for requests that originate in Secrets Manager, you can use the [kms:ViaService condition key](policy-conditions.md#conditions-kms-via-service) with the `secretsmanager.<region>.amazonaws.com` value\.

You can also use the keys or values in the [encryption context](#asm-encryption-context) as a condition for using the CMK for cryptographic operations\. For example, you can use a [string condition operator](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_String) in an IAM or key policy document, or use a [grant constraint](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) in a grant\.

### Key policy of the AWS managed CMK<a name="asm-policies"></a>

The key policy for the AWS managed CMK for Secrets Manager gives users permission to use the CMK for specified operations only when Secrets Manager makes the request on the user's behalf\. The key policy does not allow any user to use the CMK directly\.

This key policy, like the policies of all [AWS managed keys](concepts.md#master_keys), is established by the service\. You cannot change the key policy, but you can view it at any time\. For details, see [Viewing a key policy](key-policy-viewing.md)\.

The policy statements in the key policy have the following effect:
+ Allow users in the account to use the CMK for cryptographic operations only when the request comes from Secrets Manager on their behalf\. The `kms:ViaService` condition key enforces this restriction\.
+ Allows the AWS account to create IAM policies that allow users to view CMK properties and revoke grants\.
+ Although Secrets Manager does not use grants to gain access to the CMK, the policy also allows Secrets Manager to [create grants](grants.md) for the CMK on the user's behalf and allows the account to [revoke any grant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) that allows Secrets Manager to use the CMK\. These are standard elements of policy document for an AWS managed CMK\.

The following is a key policy for an example AWS managed CMK for Secrets Manager\.

```
{
  "Version" : "2012-10-17",
  "Id" : "auto-secretsmanager-1",
  "Statement" : [ {
    "Sid" : "Allow access through AWS Secrets Manager for all principals in the account that are authorized to use AWS S
ecrets Manager",
    "Effect" : "Allow",
    "Principal" : {
      "AWS" : "*"
    },
    "Action" : [ "kms:Encrypt", "kms:Decrypt", "kms:ReEncrypt*", "kms:GenerateDataKey*", "kms:CreateGrant", "kms:Describ
eKey" ],
    "Resource" : "*",
    "Condition" : {
      "StringEquals" : {
        "kms:ViaService" : "secretsmanager.us-west-2.amazonaws.com",
        "kms:CallerAccount" : "111122223333"
      }
    }
  },{
    "Sid" : "Allow direct access to key metadata to the account",
    "Effect" : "Allow",
    "Principal" : {
      "AWS" : "arn:aws:iam::111122223333:root"
    },
    "Action" : [ "kms:Describe*", "kms:Get*", "kms:List*", "kms:RevokeGrant" ],
    "Resource" : "*"
  } ]
}
```

## Secrets Manager encryption context<a name="asm-encryption-context"></a>

An [encryption context](concepts.md#encrypt_context) is a set of key–value pairs that contain arbitrary nonsecret data\. When you include an encryption context in a request to encrypt data, AWS KMS cryptographically binds the encryption context to the encrypted data\. To decrypt the data, you must pass in the same encryption context\. 

In its [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) requests to AWS KMS, Secrets Manager uses an encryption context with two name–value pairs that identify the secret and its version, as shown in the following example\. The names do not vary, but combined encryption context values will be different for each secret value\.

```
"encryptionContext": {
    "SecretARN": "arn:aws:secretsmanager:us-west-2:111122223333:secret:test-secret-a1b2c3",
    "SecretVersionId": "EXAMPLE1-90ab-cdef-fedc-ba987SECRET1"
}
```

You can use the encryption context to identify these cryptographic operation in audit records and logs, such as [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) and Amazon CloudWatch Logs, and as a condition for authorization in policies and grants\.

The Secrets Manager encryption context consists of two name\-value pairs\.
+ **SecretARN** – The first name–value pair identifies the secret\. The key is `SecretARN`\. The value is the Amazon Resource Name \(ARN\) of the secret\.

  ```
  "SecretARN": "ARN of an Secrets Manager secret"
  ```

  For example, if the ARN of the secret is `arn:aws:secretsmanager:us-west-2:111122223333:secret:test-secret-a1b2c3`, the encryption context would include the following pair\.

  ```
  "SecretARN": "arn:aws:secretsmanager:us-west-2:111122223333:secret:test-secret-a1b2c3"
  ```
+ **SecretVersionId** – The second name–value pair identifies the version of the secret\. The key is `SecretVersionId`\. The value is the version ID\.

  ```
  "SecretVersionId": "<version-id>"
  ```

  For example, if the version ID of the secret is `EXAMPLE1-90ab-cdef-fedc-ba987SECRET1`, the encryption context would include the following pair\.

  ```
  "SecretVersionId": "EXAMPLE1-90ab-cdef-fedc-ba987SECRET1"
  ```

When you establish or change the CMK for a secret, Secrets Manager sends [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) requests to AWS KMS to validate that the caller has permission to use the CMK for these operations\. It discards the responses; it does not use them on the secret value\.

In these validation requests, the value of the `SecretARN` is the actual ARN of the secret, but the `SecretVersionId` value is `RequestToValidateKeyAccess`, as shown in the following example encryption context\. This special value helps you to identify validation requests in logs and audit trails\.

```
"encryptionContext": {
    "SecretARN": "arn:aws:secretsmanager:us-west-2:111122223333:secret:test-secret-a1b2c3",
    "SecretVersionId": "RequestToValidateKeyAccess"
}
```

**Note**  
In the past, Secrets Manager validation requests did not include an encryption context\. You might find calls with no encryption context in older AWS CloudTrail logs\.

## Monitoring Secrets Manager interaction with AWS KMS<a name="asm-logs"></a>

You can use AWS CloudTrail and Amazon CloudWatch Logs to track the requests that Secrets Manager sends to AWS KMS on your behalf\.

**GenerateDataKey**  
When you [create or change](#asm-using-cmk) the secret value in a secret, Secrets Manager sends a *[GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)* request to AWS KMS that specifies the CMK for the secret\.   
The event that records the `GenerateDataKey` operation is similar to the following example event\. The request is invoked by `secretsmanager.amazonaws.com`\. The parameters include the Amazon Resource Name \(ARN\) of the CMK for the secret, a key specifier that requires a 256\-bit key, and the [encryption context](#asm-encryption-context) that identifies the secret and version\.  

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "AROAIGDTESTANDEXAMPLE:user01",
        "arn": "arn:aws:sts::111122223333:assumed-role/Admin/user01",
        "accountId": "111122223333",
        "accessKeyId": "AKIAIOSFODNN7EXAMPLE",
        "sessionContext": {
            "attributes": {
                "mfaAuthenticated": "false",
                "creationDate": "2018-05-31T23:23:41Z"
            }
        },
        "invokedBy": "secretsmanager.amazonaws.com"
    },
    "eventTime": "2018-05-31T23:23:41Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "GenerateDataKey",
    "awsRegion": "us-west-2",
    "sourceIPAddress": "secretsmanager.amazonaws.com",
    "userAgent": "secretsmanager.amazonaws.com",
    "requestParameters": {
        "keyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
        "keySpec": "AES_256",
        "encryptionContext": {
            "SecretARN": "arn:aws:secretsmanager:us-west-2:111122223333:secret:test-secret-a1b2c3",
            "SecretVersionId": "EXAMPLE1-90ab-cdef-fedc-ba987SECRET1"
        }
    },
    "responseElements": null,
    "requestID": "a7d4dd6f-6529-11e8-9881-67744a270888",
    "eventID": "af7476b6-62d7-42c2-bc02-5ce86c21ed36",
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

**Decrypt**  
Whenever you [get or change](#asm-using-cmk) the secret value of a secret, Secrets Manager sends a [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) request to AWS KMS to decrypt the encrypted data key\.  
The event that records the `Decrypt` operation is similar to the following example event\. The user is the principal in your AWS account who is accessing the table\. The parameters include the encrypted table key \(as a ciphertext blob\) and the [encryption context](#asm-encryption-context) that identifies the table and the AWS account\. AWS KMS derives the ID of the CMK from the ciphertext\.   

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "AROAIGDTESTANDEXAMPLE:user01",
        "arn": "arn:aws:sts::111122223333:assumed-role/Admin/user01",
        "accountId": "111122223333",
        "accessKeyId": "AKIAIOSFODNN7EXAMPLE",
        "sessionContext": {
            "attributes": {
                "mfaAuthenticated": "false",
                "creationDate": "2018-05-31T23:36:09Z"
            }
        },
        "invokedBy": "secretsmanager.amazonaws.com"
    },
    "eventTime": "2018-05-31T23:36:09Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "Decrypt",
    "awsRegion": "us-west-2",
    "sourceIPAddress": "secretsmanager.amazonaws.com",
    "userAgent": "secretsmanager.amazonaws.com",
    "requestParameters": {
        "encryptionContext": {
            "SecretARN": "arn:aws:secretsmanager:us-west-2:111122223333:secret:test-secret-a1b2c3",
            "SecretVersionId": "EXAMPLE1-90ab-cdef-fedc-ba987SECRET1"
        }
    },
    "responseElements": null,
    "requestID": "658c6a08-652b-11e8-a6d4-ffee2046048a",
    "eventID": "f333ec5c-7fc1-46b1-b985-cbda13719611",
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