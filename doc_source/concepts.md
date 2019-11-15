# AWS Key Management Service Concepts<a name="concepts"></a>

Learn the basic terms and concepts in AWS Key Management Service \(AWS KMS\) and how they work together to help protect your data\.

**Topics**
+ [Customer Master Keys \(CMKs\)](#master_keys)
+ [Data Keys](#data-keys)
+ [Envelope Encryption](#enveloping)
+ [Encryption Context](#encrypt_context)
+ [Key Policies](#key_permissions)
+ [Grants](#grant)
+ [Grant Tokens](#grant_token)
+ [Auditing CMK Usage](#auditing_key_use)
+ [Key Management Infrastructure](#key_management)

## Customer Master Keys \(CMKs\)<a name="master_keys"></a>

Customer master keys are the primary resources in AWS KMS\.

A *customer master key* \(CMK\) is a logical representation of a master key\. The CMK includes metadata, such as the key ID, creation date, description, and key state\. The CMK also contains the key material used to encrypt and decrypt data\.

You can use a CMK to encrypt and decrypt up to 4 KB \(4096 bytes\) of data\. Typically, you use CMKs to generate, encrypt, and decrypt the [data keys](#data-keys) that you use outside of AWS KMS to encrypt your data\. This strategy is known as [envelope encryption](#enveloping)\.

CMKs are created in AWS KMS and never leave AWS KMS unencrypted\.  To manage your CMK, you can use the AWS Management Console or the [AWS KMS API](https://docs.aws.amazon.com/kms/latest/APIReference/)\. To use a CMK in cryptographic operations, you must use the AWS KMS API\. This strategy differs from [data keys](#data-keys)\. AWS KMS does not store, manage, or track your data keys\. You must use them outside of AWS KMS\.

By default, AWS KMS creates the key material for a CMK\. You cannot extract, export, view, or manage this key material\. Also, you cannot delete this key material; you must [delete the CMK](deleting-keys.md)\. However, you can [import your own key material](importing-keys.md) into a CMK or create the key material for a CMK in the AWS CloudHSM cluster associated with an [AWS KMS custom key store](custom-key-store-overview.md)\.

For information about creating and managing CMKs, see [Getting Started](getting-started.md)\. For information about using CMKs, see the [AWS Key Management Service API Reference](https://docs.aws.amazon.com/kms/latest/APIReference/)\. 

There are three types of CMKs in AWS accounts: customer managed CMKs, AWS managed CMKs, and AWS owned CMKs\. 


| Type of CMK | Can View CMK Metadata | Can Manage CMK | Used Only for My AWS Account | 
| --- | --- | --- | --- | 
| [Customer managed CMK](#customer-cmk) | Yes | Yes | Yes | 
| [AWS managed CMK](#aws-managed-cmk) | Yes | No | Yes | 
| [AWS owned CMK](#aws-owned-cmk) | No | No | No | 

To distinguish customer managed CMKs from AWS managed CMKs, use the `KeyManager` field in the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) API operation response\. For customer managed CMKs, the `KeyManager` value is `Customer`\. For AWS managed CMKs, the `KeyManager` value is `AWS`\.

[AWS services that integrate with AWS KMS](service-integration.md) differ in their support for CMKs\. Some AWS services encrypt your data by default with an AWS owned CMK or an AWS managed CMK\. Other AWS services offer to encrypt your data under a customer managed CMK that you choose\. And other AWS services support all types of CMKs to allow you the ease of an AWS owned CMK, the visibility of an AWS managed CMK, or the control of a customer managed CMK\.

### Customer Managed CMKs<a name="customer-cmk"></a>

*Customer managed CMKs* are CMKs in your AWS account that you create, own, and manage\. You have full control over these CMKs, including establishing and maintaining their [key policies, IAM policies, and grants](control-access.md), [enabling and disabling](enabling-keys.md) them, [rotating their cryptographic material](rotate-keys.md), [adding tags](tagging-keys.md), [creating aliases](programming-aliases.md) that refer to the CMK, and [scheduling the CMKs for deletion](deleting-keys.md)\. 

Customer managed CMKs appear on the **Customer managed keys** page of the AWS Management Console for AWS KMS\. To definitively identify a customer managed CMK, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) API operation\. For customer managed CMKs, the value of the `KeyManager` field of the `DescribeKey` response is `CUSTOMER`\.

You can use your customer managed CMKs in cryptographic operations and audit their use in AWS CloudTrail logs\. In addition, many [AWS services that integrate with AWS KMS](service-integration.md) let you specify a customer managed CMK to protect the data that they store and manage for you\. 

Customer managed CMKs incur a monthly fee and a fee for use in excess of the free tier\. They are counted against the AWS KMS [limits](limits.md) for your account\. For details, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/) and [Limits](limits.md)\.

### AWS Managed CMKs<a name="aws-managed-cmk"></a>

*AWS managed CMKs* are CMKs in your account that are created, managed, and used on your behalf by an AWS service that is integrated with AWS KMS\. 

You can [view the AWS managed CMKs](viewing-keys.md) in your account, [view their key policies](key-policy-viewing.md), and [audit their use](logging-using-cloudtrail.md) in AWS CloudTrail logs\. However, you cannot manage these CMKs or change their key policies\. And, you cannot use AWS managed CMKs in cryptographic operations directly; the service that creates them uses them on your behalf\. 

AWS managed CMKs appear on the **AWS managed keys** page of the AWS Management Console for AWS KMS\. You can also identify most AWS managed CMKs by their aliases, which have the format `aws/service-name`, such as `aws/redshift`\. To definitively identify an AWS managed CMK, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) API operation\. For AWS managed CMKs, the value of the `KeyManager` field of the `DescribeKey` response is `AWS`\.

You do not pay a monthly fee for AWS managed CMKs\. They can be subject to fees for use in excess of the free tier, but some AWS services cover these costs for you\. For details, see the encryption section of the service documentation\. AWS managed CMKs do not count against limits on the number of CMKs in each Region of your account\. But when they are used on behalf of a principal in your account, these CMKs count against request rate limits\. For details, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/) and [Limits](limits.md)\.

### AWS Owned CMKs<a name="aws-owned-cmk"></a>

*AWS owned CMKs* are not in your AWS account\. They are part of a collection of CMKs that AWS owns and manages for use in multiple AWS accounts\. AWS services can use AWS owned CMKs to protect your data\. 

You cannot view, manage, or use AWS owned CMKs, or audit their use\. However, you do not need to do any work or change any programs to protect the keys that encrypt your data\. 

You are not charged a monthly fee or a usage fee for use of AWS owned CMKs and they do not count against AWS KMS limits for your account\.

## Data Keys<a name="data-keys"></a>

*Data keys* are encryption keys that you can use to encrypt data, including large amounts of data and other data encryption keys\. 

You can use AWS KMS [customer master keys](#master_keys) \(CMKs\) to generate, encrypt, and decrypt data keys\. However, AWS KMS does not store, manage, or track your data keys, or perform cryptographic operations with data keys\. You must use and manage data keys outside of AWS KMS\.

### Create a Data Key<a name="data-keys-create"></a>

To create a data key, call the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation\. AWS KMS uses the CMK that you specify to generate a data key\. The operation returns a plaintext copy of the data key and a copy of the data key encrypted under the CMK\. The following image shows this operation\.

![\[Generate a data key\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/generate-data-key.png)

AWS KMS also supports the [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html) operation, which returns only an encrypted data key\. When you need to use the data key, ask AWS KMS to [decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) it\.

### Encrypt Data with a Data Key<a name="data-keys-encrypt"></a>

AWS KMS cannot use a data key to encrypt data\. But you can use the data key outside of KMS, such as by using OpenSSL or a cryptographic library like the [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/)\.

After using the plaintext data key to encrypt data, remove it from memory as soon as possible\. You can safely store the encrypted data key with the encrypted data so it is available to decrypt the data\.

![\[Encrypt user data outside of AWS KMS\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/encrypt-with-data-key.png)

### Decrypt Data with a Data Key<a name="data-keys-decryptt"></a>

To decrypt your data, pass the encrypted data key to the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. AWS KMS uses your CMK to decrypt the data key and then it returns the plaintext data key\. Use the plaintext data key to decrypt your data and then remove the plaintext data key from memory as soon as possible\.

The following diagram shows how to use the `Decrypt` operation to decrypt an encrypted data key\.

![\[Decrypting a data key\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/decrypt.png)

## Envelope Encryption<a name="enveloping"></a>

When you encrypt your data, your data is protected, but you have to protect your encryption key\. One strategy is to encrypt it\. *Envelope encryption* is the practice of encrypting plaintext data with a data key, and then encrypting the data key under another key\.

You can even encrypt the data encryption key under another encryption key, and encrypt that encryption key under another encryption key\. But, eventually, one key must remain in plaintext so you can decrypt the keys and your data\. This top\-level plaintext key encryption key is known as the *master key*\.

![\[Envelope encryption\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/key-hierarchy-master.png)

AWS KMS helps you to protect your master keys by storing and managing them securely\. Master keys stored in AWS KMS, known as [customer master keys](#master_keys) \(CMKs\), never leave the AWS KMS [FIPS validated hardware security modules](https://csrc.nist.gov/projects/cryptographic-module-validation-program/Certificate/3139) unencrypted\. To use an AWS KMS CMK, you must call AWS KMS\.

![\[Envelope encryption with multiple key encryption keys\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/key-hierarchy-cmk.png)

Envelope encryption offers several benefits:
+ **Protecting data keys**

  When you encrypt a data key, you don't have to worry about storing the encrypted data key, because the data key is inherently protected by encryption\. You can safely store the encrypted data key alongside the encrypted data\.
+ **Encrypting the same data under multiple master keys**

  Encryption operations can be time consuming, particularly when the data being encrypted are large objects\. Instead of re\-encrypting raw data multiple times with different keys, you can re\-encrypt only the data keys that protect the raw data\.
+ **Combining the strengths of multiple algorithms**

  In general, symmetric key algorithms are faster and produce smaller ciphertexts than public key algorithms\. But public key algorithms provide inherent separation of roles and easier key management\. Envelope encryption lets you combine the strengths of each strategy\.

## Encryption Context<a name="encrypt_context"></a>

All AWS KMS cryptographic operations \([https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), [https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html), [https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), and [https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html)\) accept an *encryption context*, an optional set of key–value pairs that can contain additional contextual information about the data\. AWS KMS uses the encryption context as [additional authenticated data](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#term-aad) \(AAD\) to support [authenticated encryption](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#define-authenticated-encryption)\. 

When you include an encryption context in an encryption request, it is cryptographically bound to the ciphertext such that the same encryption context is required to decrypt \(or decrypt and re\-encrypt\) the data\. If the encryption context provided in the decryption request is not an exact, case\-sensitive match, the decrypt request fails\. Only the order of the key\-value pairs in the encryption context can vary\.

The encryption context is not secret\. It appears in plaintext in [AWS CloudTrail Logs](logging-using-cloudtrail.md) so you can use it to identify and categorize your cryptographic operations\.

An encryption context can consist of any keys and values\. However, because it is not secret and not encrypted, your encryption context should not include sensitive information\. We recommend that your encryption context describe the data being encrypted or decrypted\. For example, when you encrypt a file, you might use part of the file path as encryption context\.

The key and value in an encryption context pair must be simple literal strings\. They cannot be integers or objects, or any type that is not fully resolved\. If you use a different type, such as an integer or float, AWS KMS interprets it as a string\.

```
"encryptionContext": {
    "department": "10103.0"
}
```

The encryption context key and value can include special characters, such as underscores \(\_\), dashes \(\-\), slashes \(/, \\\) and colons \(:\)\.

For example, [Amazon Simple Storage Service](services-s3.md#s3-encryption-context) \(Amazon S3\) uses an encryption context in which the key is `aws:s3:arn`\. The value is the S3 bucket path to the file that is being encrypted\.

```
"encryptionContext": {
    "aws:s3:arn": "arn:aws:s3:::bucket_name/file_name"
}
```

You can also use the encryption context to refine or limit access to customer master keys \(CMKs\) in your account\. You can use the encryption context [as a constraint in grants](grants.md) and as a *[condition in policy statements](policy-conditions.md)*\. 

To learn how to use encryption context to protect the integrity of encrypted data, see the post [How to Protect the Integrity of Your Encrypted Data by Using AWS Key Management Service and EncryptionContext](https://aws.amazon.com/blogs/security/how-to-protect-the-integrity-of-your-encrypted-data-by-using-aws-key-management-service-and-encryptioncontext/) on the AWS Security Blog\.

More about encryption context\.

### Encryption Context in Policies<a name="encryption-context-authorization"></a>

The encryption context is used primarily to verify integrity and authenticity\. But you can also use the encryption context as a condition for authorizing use of a customer master keys \(CMK\) in key policies and IAM policies\. 

The [kms:EncryptionContext:](policy-conditions.md#conditions-kms-encryption-context) and [kms:EncryptionContextKeys](policy-conditions.md#conditions-kms-encryption-context) condition keys allow \(or deny\) a permission only when the request includes particular encryption context keys or key–value pairs\. 

For example, the following key policy statement allows the `RoleForExampleApp` role to use the CMK in `Decrypt` operations\. It uses the `kms:EncryptionContext:` condition key to allow this permission only when the encryption context in the request includes an `AppName:ExampleApp` encryption context pair\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/RoleForExampleApp"
  },
  "Action": "kms:Decrypt",
  "Resource": "*",
  "Condition": {
    "ForAnyValue:StringEquals": {
      "kms:EncryptionContext:AppName": "ExampleApp"
    }
  }
}
```

For more information about these encryption context condition keys, see [Using Policy Conditions with AWS KMS](policy-conditions.md)\.

### Encryption Context in Grants<a name="encryption-context-grants"></a>

When you [create a grant](grants.md), you can include [grant constraints](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) that allow access only when a request includes a particular encryption context or encryption context keys\. For details about the `EncryptionContextEquals` and `EncryptionContextSubset` grant constraints, see [Grant Constraints](grants.md#grant-constraints)\.

To specify an encryption context constraint in a grant, use the `Constraints` parameter in the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) API operation\. This example uses the AWS Command Line Interface, but you can use any AWS SDK\. The grant that this command creates gives the `exampleUser` permission to call the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) API operation\. But that permission is effective only when the encryption context in the `Decrypt` request includes a `"Department": "IT"` encryption context pair\.

```
$ aws kms create-grant \
    --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
    --grantee-principal arn:aws:iam::111122223333:user/exampleUser \
    --operations Decrypt \
    --retiring-principal arn:aws:iam::111122223333:role/adminRole \
    --constraints EncryptionContextSubset={Department=IT}
```

The resulting grant looks like the following one\. Notice that the permission granted to `exampleUser` is effective only when the `Decrypt` request includes the encryption context pair specified in the grant constraint\. To find the grants on a CMK, use the [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) API operation\.

```
$ aws kms list-grants --key-id 1234abcd-12ab-34cd-56ef-1234567890ab

{
    "Grants": [
        {
            "Name": "",
            "IssuingAccount": "arn:aws:iam::111122223333:root",
            "GrantId": "8c94d1f12f5e69f440bae30eaec9570bb1fb7358824f9ddfa1aa5a0dab1a59b2",
            "Operations": [
                "Decrypt"
            ],
            "GranteePrincipal": "arn:aws:iam::111122223333:user/exampleUser",
            "Constraints": {
                "EncryptionContextSubset": {
                    "Department": "IT"
                }
            },
            "CreationDate": 1568565290.0,
            "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "RetiringPrincipal": "arn:aws:iam::111122223333:role/adminRole"
        }
    ]
}
```

AWS services often use encryption context constraints in the grants that give them permission to use CMKs in your AWS account\. For example, Amazon DynamoDB uses a grant like the following one to get permission to use the [AWS managed CMK](#aws-managed-cmk) for DynamoDB in your account\. The `EncryptionContextSubset` grant constraint in this grant makes the permissions in the grant effective only when the encryption context in the request includes `"subscriberID": "111122223333"` and `"tableName": "Services"` pairs\. This grant constraint means that the grant allows DynamoDB to use the specified CMK only for a particular table in your AWS account\.

To get this output, run the [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) API operation on the AWS managed CMK for DynamoDB in your account\.

```
$ aws kms list-grants --key-id 0987dcba-09fe-87dc-65ba-ab0987654321

{
    "Grants": [
        {
            "Operations": [
                "Decrypt",
                "Encrypt",
                "GenerateDataKey",
                "ReEncryptFrom",
                "ReEncryptTo",
                "RetireGrant",
                "DescribeKey"
            ],
            "IssuingAccount": "arn:aws:iam::111122223333:root",
            "Constraints": {
                "EncryptionContextSubset": {
                    "aws:dynamodb:tableName": "Services",
                    "aws:dynamodb:subscriberId": "111122223333"
                }
            },
            "CreationDate": 1518567315.0,
            "KeyId": "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321",
            "GranteePrincipal": "dynamodb.us-west-2.amazonaws.com",
            "RetiringPrincipal": "dynamodb.us-west-2.amazonaws.com",
            "Name": "8276b9a6-6cf0-46f1-b2f0-7993a7f8c89a",
            "GrantId": "1667b97d27cf748cf05b487217dd4179526c949d14fb3903858e25193253fe59"
        }
    ]
}
```

### Logging Encryption Context<a name="encryption-context-auditing"></a>

AWS KMS uses AWS CloudTrail to log the encryption context so you can determine which CMKs and data have been accessed\. The log entry shows exactly which CMK was used to encrypt or decrypt specific data referenced by the encryption context in the log entry\.

**Important**  
Because the encryption context is logged, it must not contain sensitive information\.

### Storing Encryption Context<a name="encryption-context-storing"></a>

To simplify use of any encryption context when you call the [https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) \(or [https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)\) API, you can store the encryption context alongside the encrypted data\. We recommend that you store only enough of the encryption context to help you create the full encryption context when you need it for encryption or decryption\. 

For example, if the encryption context is the fully qualified path to a file, store only part of that path with the encrypted file contents\. Then, when you need the full encryption context, reconstruct it from the stored fragment\. If someone tampers with the file, such as renaming it or moving it to a different location, the encryption context value changes and the decryption request fails\.

## Key Policies<a name="key_permissions"></a>

When you create a CMK, you determine who can use and manage that CMK\. These permissions are contained in a document called the *key policy*\. You can use the key policy to add, remove, or change permissions at any time for a customer managed CMK\. But you cannot edit the key policy for an AWS managed CMK\. For more information, see [Authentication and Access Control for AWS KMS](control-access.md)\.

## Grants<a name="grant"></a>

A *grant* is another mechanism for providing permissions\. It's an alternative to key policies\. Because grants can be very specific, and are easy to create and revoke, they are often used to provide temporary permissions or more granular permissions\. For more information, see [Using Grants](grants.md)\.

## Grant Tokens<a name="grant_token"></a>

When you create a grant, the permissions specified in the grant might not take effect immediately due to [eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency)\. If you need to mitigate the potential delay, use the *grant token* that you receive in the response to your [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) API request\. You can pass the grant token with some AWS KMS API requests to make the permissions in the grant take effect immediately\. The following AWS KMS API operations accept grant tokens:
+ [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html)
+ [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)
+ [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html)
+ [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html)
+ [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)
+ [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html)
+ [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)
+ [RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html)

A grant token is not a secret\. The grant token contains information about who the grant is for and therefore who can use it to cause the grant's permissions to take effect more quickly\.

## Auditing CMK Usage<a name="auditing_key_use"></a>

You can use AWS CloudTrail to audit key usage\. CloudTrail creates log files that contain a history of AWS API calls and related events for your account\. These log files include all AWS KMS API requests made with the AWS Management Console, AWS SDKs, and command line tools\. The log files also include requests to AWS KMS that AWS services make on your behalf\. You can use these log files to find important information, including when the CMK was used, the operation that was requested, the identity of the requester, and the source IP address\. For more information, see [Logging AWS KMS API Calls with AWS CloudTrail](logging-using-cloudtrail.md) and the [AWS CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\.

## Key Management Infrastructure<a name="key_management"></a>

A common practice in cryptography is to encrypt and decrypt with a publicly available and peer\-reviewed algorithm such as AES \(Advanced Encryption Standard\) and a secret key\. One of the main problems with cryptography is that it's very hard to keep a key secret\. This is typically the job of a key management infrastructure \(KMI\)\. AWS KMS operates the KMI for you\. AWS KMS creates and securely stores your master keys, called CMKs\. For more information about how AWS KMS operates, see the [AWS Key Management Service Cryptographic Details](https://d0.awsstatic.com/whitepapers/KMS-Cryptographic-Details.pdf) whitepaper\.