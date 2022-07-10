# Asymmetric keys in AWS KMS<a name="symmetric-asymmetric"></a>

AWS KMS supports asymmetric KMS keys that represent a mathematically related RSA or elliptic curve \(ECC\) public and private key pair\. These key pairs are generated in AWS KMS hardware security modules certified under the [FIPS 140\-2 Cryptographic Module Validation Program](https://csrc.nist.gov/projects/cryptographic-module-validation-program/Certificate/3139), except in the China \(Beijing\) and China \(Ningxia\) Regions\. The private key never leaves the AWS KMS HSMs unencrypted\. You can download the public key for distribution and use outside of AWS\. You can create asymmetric KMS keys for encryption and decryption, or signing and verification, but not both\.

You can create and manage the asymmetric KMS keys in your AWS account, including setting the [key policies](key-policies.md), [IAM policies](iam-policies.md), and [grants](grants.md) that control access to the keys, [enabling and disabling](enabling-keys.md) the KMS keys, [creating tags](tagging-keys.md) and [aliases](alias-manage.md#alias-create), and [deleting the KMS keys](deleting-keys.md)\. You can audit all operations that use or manage your asymmetric KMS keys within AWS in [AWS CloudTrail logs](logging-using-cloudtrail.md)\.

AWS KMS also provides asymmetric [data key pairs](concepts.md#data-key-pairs) that are designed to be used for client\-side cryptography outside of AWS KMS\. The symmetric data key and the private key in an asymmetric data key pair are protected by a [symmetric encryption KMS key](concepts.md#symmetric-cmks) in AWS KMS\. 

This topic explains how asymmetric KMS keys work, how they differ from other KMS keys and, and how to decide which type of KMS key you need to protect your data\. It also explains how asymmetric data key pairs work and how to use them outside of AWS KMS\.

**Regions**

Asymmetric KMS keys and asymmetric data key pairs are supported in all AWS Regions that AWS KMS supports\.

**Learn more**
+ To create asymmetric KMS keys, see [Creating asymmetric KMS keys](asymm-create-key.md)\. To create symmetric encryption KMS keys, see [Creating keys](create-keys.md)\. 
+ To create multi\-Region asymmetric KMS keys, see [Creating multi\-Region keys](multi-region-keys-create.md)\.
+ To find out whether a KMS key is symmetric or asymmetric, see [Identifying asymmetric KMS keys](find-symm-asymm.md)\. 
+ For a table that compares the AWS KMS API operations that apply to each type of KMS key, see [Key type reference](symm-asymm-compare.md)\.
+ To control access to the key specs, key usage, encryption algorithms, and signing algorithms that principals in your account can use for KMS keys and data keys, see [AWS KMS condition keys](policy-conditions.md#conditions-kms)\.
+ To learn about the request quotas that apply to different types of KMS keys, see [Request quotas](requests-per-second.md)\.
+ To learn how to sign messages and verify signatures with asymmetric KMS keys, see [Digital signing with the new asymmetric keys feature of AWS KMS](http://aws.amazon.com/blogs/security/digital-signing-asymmetric-keys-aws-kms/) in the *AWS Security Blog*\.

**Topics**
+ [Asymmetric KMS keys](#asymmetric-cmks)
+ [Creating asymmetric KMS keys](asymm-create-key.md)
+ [Downloading public keys](download-public-key.md)
+ [Identifying asymmetric KMS keys](find-symm-asymm.md)
+ [Asymmetric key specs](asymmetric-key-specs.md)

## Asymmetric KMS keys<a name="asymmetric-cmks"></a>

You can create an asymmetric KMS key in AWS KMS\. An *asymmetric KMS key* represents a mathematically related public key and private key pair\. You can give the public key to anyone, even if they're not trusted, but the private key must be kept secret\. 

In an asymmetric KMS key, the private key is created in AWS KMS and never leaves AWS KMS unencrypted\. To use the private key, you must call AWS KMS\. You can use the public key within AWS KMS by calling the AWS KMS API operations\. Or, you can [download the public key](download-public-key.md) and use it outside of AWS KMS\.

If your use case requires encryption outside of AWS by users who cannot call AWS KMS, asymmetric KMS keys are a good choice\. However, if you are creating a KMS key to encrypt the data that you store or manage in an AWS service, use a symmetric encryption KMS key\. [AWS services that are integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration) use only symmetric encryption KMS keys to encrypt your data\. These services do not support encryption with asymmetric KMS keys\.

AWS KMS supports two types of asymmetric KMS keys\. 
+ **RSA KMS keys**: A KMS key with an RSA key pair for encryption and decryption or signing and verification \(but not both\)\. AWS KMS supports several key lengths for different security requirements\.
+ **Elliptic Curve \(ECC\) KMS keys**: A KMS key with an elliptic curve key pair for signing and verification\. AWS KMS supports several commonly\-used curves\.

For help choosing your asymmetric key configuration, see [Choosing a KMS key type](key-types.md#symm-asymm-choose)\. For technical details about the encryption and signing algorithms that AWS KMS supports for RSA KMS keys, see [RSA Key Specs](asymmetric-key-specs.md#key-spec-rsa)\. For technical details about the signing algorithms that AWS KMS supports for ECC KMS keys, see [Elliptic Curve Key Specs](asymmetric-key-specs.md#key-spec-ecc)\.

For a table comparing the operations that you can perform on symmetric and asymmetric KMS keys, see [Comparing Symmetric and Asymmetric KMS keys](symm-asymm-compare.md)\. For help determining whether a KMS key is symmetric or asymmetric, see [Identifying asymmetric KMS keys](find-symm-asymm.md)\.

**Regions**

Asymmetric KMS keys and asymmetric data key pairs are supported in all AWS Regions that AWS KMS supports\.