# AWS Key Management Service Concepts<a name="concepts"></a>

Learn the basic terms and concepts in AWS Key Management Service \(AWS KMS\) and how they work together to help protect your data\.


+ [Customer Master Keys](#master_keys)
+ [Data Keys](#data-keys)
+ [Envelope Encryption](#enveloping)
+ [Encryption Context](#encrypt_context)
+ [Key Policies](#key_permissions)
+ [Grants](#grant)
+ [Grant Tokens](#grant_token)
+ [Auditing CMK Usage](#auditing_key_use)
+ [Key Management Infrastructure](#key_management)

## Customer Master Keys<a name="master_keys"></a>

The primary resources in AWS KMS are *customer master keys* \(CMKs\)\. CMKs are either customer managed or AWS managed\. You can use either type of CMK to protect up to 4 kilobytes \(4096 bytes\) of data directly\. Typically you use CMKs to protect *data encryption keys* \(or *data keys*\) which are then used to encrypt or decrypt larger amounts of data outside of the service\. CMKs never leave AWS KMS unencrypted, but data keys can\. AWS KMS does not store, manage, or track your data keys\.

There is one *AWS managed CMK* for each service that is integrated with AWS KMS\. When you create an encrypted resource in these services, you can choose to protect that resource under the AWS managed CMK for that service\. This CMK is unique to your AWS account and the AWS region in which it is used, and it protects the data keys used by the AWS services to protect your data\. Many of these services allow you to specify a customer managed CMK instead of the AWS managed CMK, which is useful when you need more granular control over the CMK\. For example, when you use a customer managed CMK with these services you can request that AWS KMS rotate the CMK's key material every year\. You cannot do this with an AWS managed CMK\.

## Data Keys<a name="data-keys"></a>

You use data keys to encrypt large data objects within your own application outside AWS KMS\. When you make a [GenerateDataKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) API request, AWS KMS returns a plaintext copy of the data key and a ciphertext that contains the data key encrypted under the specified CMK\. You use the plaintext data key in your application to encrypt data, and you typically store the encrypted data key alongside your encrypted data\. Security best practices dictate that you should remove the plaintext data key from memory as soon as practical after use\. To decrypt data in your application, you pass the encrypted data key with a [Decrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) API request\. AWS KMS uses your CMK to decrypt the data key into plaintext, and then returns it to you\. You use the plaintext data key to decrypt your data and then remove the plaintext data key from memory as soon as practical after use\.

**Note**  
You can use AWS KMS to generate, encrypt, and decrypt data keys, but AWS KMS does not store, manage, or track your data keys\. You must do this inside your application\.

## Envelope Encryption<a name="enveloping"></a>

AWS KMS uses *envelope encryption* to protect data\. Envelope encryption is the practice of encrypting plaintext data with a unique data key, and then encrypting the data key with a *key encryption key* \(KEK\)\. You might choose to encrypt the KEK with another KEK, and so on, but eventually you must have a *master key*\. The master key is an unencrypted \(plaintext\) key with which you can decrypt one or more other keys\.

Envelope encryption offers several benefits:

+ **Protecting data keys**

  When you encrypt a data key, you don't have to worry about where to store the encrypted data key, because the security of that data key is inherently protected by encryption\. You can safely store the encrypted data key alongside the encrypted data\.

+ **Encrypting the same data under multiple master keys**

  Encryption operations can be time consuming, particularly when the data being encrypted are large objects\. Instead of reencrypting raw data multiple times with different keys, you can reencrypt only the data keys that protect the raw data\.

+ **Combining the strengths of multiple algorithms**

  In general, symmetric key algorithms are faster and produce smaller ciphertexts than public key algorithms, but public key algorithms provide inherent separation of roles and easier key management\. You might want to combine the strengths of each\.

The following image provides an overview of envelope encryption\. In this scenario, the data key is encrypted with a single KEK, which is the master key\. In AWS KMS, your CMK represents the master key\.

![\[Envelope encryption\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/envelope-encryption.png)![\[Envelope encryption\]](http://docs.aws.amazon.com/kms/latest/developerguide/)

## Encryption Context<a name="encrypt_context"></a>

All AWS KMS cryptographic operations accept an optional set of key–value pairs that can contain additional contextual information about the data\. This set of key–value pairs is called *encryption context\.* When encryption context is used with an encryption operation, the encryption context for the corresponding decryption operation must match for the decryption to succeed\. Encryption context is not secret\. Encryption context is logged, and you can use it for auditing and when controlling access to AWS KMS API operation\. For more information, see [Encryption Context](encryption-context.md)\.

## Key Policies<a name="key_permissions"></a>

When you create a CMK, you choose who can manage or use that CMK\. These permissions are contained in a document called the *key policy*\. You can use the key policy to add, remove, or modify permissions at any time for a customer\-managed CMK, but you cannot edit the key policy for an AWS\-managed CMK\. For more information, see [Authentication and Access Control for AWS KMS](control-access.md)\.

## Grants<a name="grant"></a>

A *grant* is another mechanism for providing permissions, an alternative to the key policy\. You can use grants to give long\-term access that allows AWS principals to use your customer\-managed CMKs\. For more information, see [Using Grants](grants.md)\.

## Grant Tokens<a name="grant_token"></a>

When you create a grant, the permissions specified in the grant might not take effect immediately due to [eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency)\. If you need to mitigate the potential delay, use the *grant token* that you receive in the response to your [CreateGrant](http://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) API request\. You can pass the grant token with some AWS KMS API requests to make the permissions in the grant take effect immediately\. The following AWS KMS API operations accept grant tokens:

+ [CreateGrant](http://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html)

+ [Decrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)

+ [DescribeKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html)

+ [Encrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html)

+ [GenerateDataKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)

+ [GenerateDataKeyWithoutPlaintext](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html)

+ [ReEncrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)

+ [RetireGrant](http://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html)

A grant token is not a secret\. The grant token contains information about who the grant is for and therefore who can use it to cause the grant's permissions to take effect more quickly\.

## Auditing CMK Usage<a name="auditing_key_use"></a>

You can use AWS CloudTrail to audit key usage\. CloudTrail creates log files that contain a history of AWS API calls and related events for your account\. These log files include all AWS KMS API requests made with the AWS Management Console, AWS SDKs, and command line tools, as well as those made through integrated AWS services\. You can use these log files to get information about when the CMK was used, the operation that was requested, the identity of the requester, the IP address that the request came from, and so on\. For more information, see [Logging AWS KMS API Calls](logging-using-cloudtrail.md) and the [AWS CloudTrail User Guide](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\.

## Key Management Infrastructure<a name="key_management"></a>

A common practice in cryptography is to encrypt and decrypt with a publicly available and peer\-reviewed algorithm such as AES \(Advanced Encryption Standard\) and a secret key\. One of the main problems with cryptography is that it's very hard to keep a key secret\. This is typically the job of a key management infrastructure \(KMI\)\. AWS KMS operates the KMI for you\. AWS KMS creates and securely stores your master keys, called CMKs\. For more information about how AWS KMS operates, see the [AWS Key Management Service Cryptographic Details](https://d0.awsstatic.com/whitepapers/KMS-Cryptographic-Details.pdf) whitepaper\.