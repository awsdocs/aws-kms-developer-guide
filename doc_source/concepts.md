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

The primary resources in AWS KMS are *customer master keys* \(CMKs\)\. You can use a CMK to encrypt and decrypt up to 4 KB \(4096 bytes\) of data\. Typically, you use CMKs to generate, encrypt, and decrypt the [data keys](#data-keys) that you use outside of AWS KMS to encrypt your data\. This strategy is known as [envelope encryption](#enveloping)\.

CMKs are created in AWS KMS and never leave AWS KMS unencrypted\.  To use or manage your CMK, you access them through AWS KMS\. This strategy differs from [data keys](#data-keys)\. AWS KMS does not store, manage, or track your data keys\. You must use them outside of AWS KMS\.

There are three types of CMKs in AWS accounts: customer managed CMKs, AWS managed CMKs, and AWS owned CMKs\. 


| Type of CMK | Can view | Can manage | Used only for my AWS account | 
| --- | --- | --- | --- | 
| [Customer managed CMK](#customer-cmk) | Yes | Yes | Yes | 
| [AWS managed CMK](#aws-managed-cmk) | Yes | No | Yes | 
| [AWS owned CMK](#aws-owned-cmk) | No | No | No | 

[AWS services that integrate with AWS KMS](service-integration.md) differ in their support for CMKs\. Some services encrypt your data by default with an AWS owned CMK\. Some encrypt under AWS managed CMKs that they create in your account\. Other services allow you specify a customer managed CMK that you have created\. And others support all types of CMKs to allow you the ease of an AWS owned CMK, the visibility of an AWS managed CMK, or the control of a customer\-managed CMK\.

### Customer managed CMKs<a name="customer-cmk"></a>

*Customer managed CMKs* are CMKs in your AWS account that you create, own, and manage\. You have full control over these CMKs, including establishing and maintaining their [key policies, IAM policies, and grants](control-access.md), [enabling and disabling](enabling-keys.md) them, [rotating their cryptographic material](rotate-keys.md), [adding tags](tagging-keys.md), [creating aliases](programming-aliases.md) that refer to the CMK, and [scheduling the CMKs for deletion](deleting-keys.md)\. 

You can use your customer managed CMKs in cryptographic operations and audit their use in AWS CloudTrail logs\. In addition, many [AWS services that integrate with AWS KMS](service-integration.md) let you specify a customer managed CMK to protect the data that they store and manage for you\. 

Customer managed CMKs incur a monthly fee and a fee for use in excess of the free tier\. They are counted against the AWS KMS [limits](limits.md) for your account\. For details, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/) and [Limits](limits.md)\.

### AWS managed CMKs<a name="aws-managed-cmk"></a>

*AWS managed CMKs* are CMKs in your account that are created, managed, and used on your behalf by an AWS service that integrates with AWS KMS\. You can identify AWS managed CMKs by their aliases, which have the format `aws/service-name`, such as `aws/redshift`\. 

You can view the AWS managed CMKs in your account, view their key policies, and audit their use in AWS CloudTrail logs\. However, you cannot manage these CMKs or change their permissions\. And, you cannot use AWS managed CMKs in cryptographic operations directly; the service that creates them uses them on your behalf\. To view the key policy for an AWS managed CMK, use the [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation\. You cannot view the key policy in the AWS Management Console, or change it by any means\. 

You do not pay a monthly fee for AWS managed CMKs\. They can be subject to fees for use in excess of the free tier, but some AWS services cover these costs for you\. For details, see the encryption section of the service documentation\. AWS managed CMKs do not count against limits on the number of CMKs in each region of your account, but when they are used on behalf of a principal in your account, they count against request rate limits\. For details, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/) and [Limits](limits.md)\.

### AWS owned CMKs<a name="aws-owned-cmk"></a>

*AWS owned CMKs* are not in your AWS account\. They are part of a collection of CMKs that AWS owns and manages for use in multiple AWS accounts\. AWS services can use AWS owned CMKs to protect your data\. 

You cannot view, manage, or use AWS owned CMKs, or audit their use\. However, you do not need to do any work or change any programs to protect the keys that encrypt your data\. 

You are not charged a monthly fee or a usage fee for use of AWS owned CMKs and they do not count against AWS KMS limits for your account\.

## Data Keys<a name="data-keys"></a>

*Data keys* are encryption keys that you can use to encrypt data, including large amounts of data and other data encryption keys\. 

You can use AWS KMS [customer master keys](#master_keys) \(CMKs\) to generate, encrypt, and decrypt data keys\. However, AWS KMS does not store, manage, or track your data keys, or perform cryptographic operations with data keys\. You must use and manage data keys outside of AWS KMS\.

**Create a data key**

To create a data key, call the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation\. AWS KMS uses the CMK that you specify to generate a data key\. The operation returns a plaintext copy of the data key and a copy of the data key encrypted under the CMK, as shown in the following image\.

![\[Generate a data key\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/generate-data-key.png)

AWS KMS also supports the [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html) operation, which returns only an encrypted data key\. When you need to use the data key, ask AWS KMS to [decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) it\.

**Encrypt data with a data key**

AWS KMS cannot use a data key to encrypt data, but you can use the data key outside of KMS, such as by using OpenSSL or a cryptographic library like the [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/)\.

After using the plaintext data key to encrypt data, remove it from memory as soon as possible\. You can safely store the encrypted data key with the encrypted data so it is available to decrypt the data\.

![\[Encrypt user data outside of AWS KMS\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/encrypt-with-data-key.png)

**Decrypt data with a data key**

To decrypt your data, pass the encrypted data key to the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. AWS KMS uses your CMK to decrypt the data key and then it returns the plaintext data key\. Use the plaintext data key to decrypt your data and then remove the plaintext data key from memory as soon as possible\.

The following diagram shows how to use the Decrypt operation to decrypt an encrypted data key\.

![\[Decrypting a data key\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/decrypt.png)

## Envelope Encryption<a name="enveloping"></a>

When you encrypt your data, your data is protected, but you have to protect your encryption key\. One strategy is to encrypt it\. *Envelope encryption* is the practice of encrypting plaintext data with a data key, and then encrypting the data key under another key\.

You can even encrypt the data encryption key under another encryption key, and encrypt that encryption key another encryption key\. But, eventually, one key must remain in plaintext so you can decrypt the keys and your data\. This top\-level plaintext key encryption key is known as the *master key*\.

![\[Envelope encryption\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/key-hierarchy-master.png)

AWS KMS helps you to protect your master keys by storing and managing them securely\. Master keys stored in AWS KMS, known as [customer master keys](#master_keys) \(CMKs\), never leave the AWS KMS [FIPS validated hardware security modules](https://csrc.nist.gov/projects/cryptographic-module-validation-program/Certificate/3139) unencrypted\. To use an AWS KMS CMK, you must call AWS KMS\.

![\[Envelope encryption with multiple key encryption keys\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/key-hierarchy-cmk.png)

Envelope encryption offers several benefits:
+ **Protecting data keys**

  When you encrypt a data key, you don't have to worry about storing the encrypted data key, because the data key is inherently protected by encryption\. You can safely store the encrypted data key alongside the encrypted data\.
+ **Encrypting the same data under multiple master keys**

  Encryption operations can be time consuming, particularly when the data being encrypted are large objects\. Instead of re\-encrypting raw data multiple times with different keys, you can re\-encrypt only the data keys that protect the raw data\.
+ **Combining the strengths of multiple algorithms**

  In general, symmetric key algorithms are faster and produce smaller ciphertexts than public key algorithms, but public key algorithms provide inherent separation of roles and easier key management\. Envelope encryption lets you combine the strengths of each strategy\.

## Encryption Context<a name="encrypt_context"></a>

All AWS KMS cryptographic operations accept an *encryption context*, an optional set of key–value pairs that can contain additional contextual information about the data\. When you provide an encryption context to an AWS KMS encryption operation, you must supply the same encryption context to the corresponding decryption operation\. Otherwise, the request to decrypt fails\. 

The encryption context is not secret\. It appears in plaintext in [AWS CloudTrail Logs](logging-using-cloudtrail.md) so you can use it to identify and categorize your cryptographic operations in logs and audits\.

For example, [Amazon Simple Storage Service](services-s3.md#s3-encryption-context) \(Amazon S3\) uses an encryption context in which the key is `aws:s3:arn` and the value is the S3 bucket path to the file that is being encrypted\.

```
"encryptionContext": {
    "aws:s3:arn": "arn:aws:s3:::bucket_name/file_name"
},
```

You can also use the encryption context to refine or limit access to customer master keys \(CMKs\) in your account\. You can use the encryption context [as a constraint in grants](grants.md) and as a *[condition in policy statements](policy-conditions.md)*\. 

For detailed information about the encryption context, see [Encryption Context](encryption-context.md)\.

## Key Policies<a name="key_permissions"></a>

When you create a CMK, you determine who can use and manage that CMK\. These permissions are contained in a document called the *key policy*\. You can use the key policy to add, remove, or change permissions at any time for a customer\-managed CMK, but you cannot edit the key policy for an AWS\-managed CMK\. For more information, see [Authentication and Access Control for AWS KMS](control-access.md)\.

## Grants<a name="grant"></a>

A *grant* is another mechanism for providing permissions, an alternative to the key policy\. You can use grants to give long\-term access that allows AWS principals to use your customer\-managed CMKs\. For more information, see [Using Grants](grants.md)\.

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

You can use AWS CloudTrail to audit key usage\. CloudTrail creates log files that contain a history of AWS API calls and related events for your account\. These log files include all AWS KMS API requests made with the AWS Management Console, AWS SDKs, and command line tools, as well as those made through integrated AWS services\. You can use these log files to get information about when the CMK was used, the operation that was requested, the identity of the requester, the IP address that the request came from, and so on\. For more information, see [Logging AWS KMS API Calls with AWS CloudTrail](logging-using-cloudtrail.md) and the [AWS CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\.

## Key Management Infrastructure<a name="key_management"></a>

A common practice in cryptography is to encrypt and decrypt with a publicly available and peer\-reviewed algorithm such as AES \(Advanced Encryption Standard\) and a secret key\. One of the main problems with cryptography is that it's very hard to keep a key secret\. This is typically the job of a key management infrastructure \(KMI\)\. AWS KMS operates the KMI for you\. AWS KMS creates and securely stores your master keys, called CMKs\. For more information about how AWS KMS operates, see the [AWS Key Management Service Cryptographic Details](https://d0.awsstatic.com/whitepapers/KMS-Cryptographic-Details.pdf) whitepaper\.