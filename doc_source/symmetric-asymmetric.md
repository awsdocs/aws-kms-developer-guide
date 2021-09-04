# Using symmetric and asymmetric keys<a name="symmetric-asymmetric"></a>

AWS KMS protects the [AWS KMS keys](concepts.md#kms_keys) that you use to protect your data and data keys\. Symmetric KMS keys and the private keys in asymmetric KMS keys are generated and used only in hardware security modules so that no one, including AWS employees, can access the plaintext key material\.

You can create and manage the KMS keys in your AWS account, including setting the [key policies](key-policies.md), [IAM policies](iam-policies.md), and [grants](grants.md) that control access to your KMS keys, [enabling and disabling](enabling-keys.md) the KMS keys, [creating tags](tagging-keys.md) and [aliases](alias-manage.md#alias-create), and [deleting the KMS keys](deleting-keys.md)\. You can use your KMS keys to protect your resources in AWS [services that are integrated with AWS KMS](service-integration.md)\. And, you can audit all operations that use or manage your KMS keys in [AWS CloudTrail logs](logging-using-cloudtrail.md)\.

AWS KMS supports symmetric and asymmetric KMS keys\.
+ [Symmetric KMS key](symm-asymm-concepts.md#symmetric-cmks): Represents a single 256\-bit secret encryption key that never leaves AWS KMS unencrypted\. To use your symmetric KMS key, you must call AWS KMS\.
+ [Asymmetric KMS key](symm-asymm-concepts.md#asymmetric-cmks): Represents a mathematically related public key and private key pair that you can use for encryption and decryption or signing and verification, but not both\. The private key never leaves AWS KMS unencrypted\. You can use the public key within AWS KMS by calling the AWS KMS API operations, or download the public key and use it outside of AWS KMS\. 

AWS KMS also supports symmetric [data keys](concepts.md#data-keys) and asymmetric [data key pairs](concepts.md#data-key-pairs) designed for use with other AWS services, and for client\-side signing and cryptography outside of AWS KMS\. The symmetric data key and the private key in an asymmetric data key pair are protected by a symmetric KMS key\.
+ Symmetric data key — A symmetric encryption key that you can use to encrypt data outside of AWS KMS\. This key is protected by a symmetric KMS key in AWS KMS\. 
+ Asymmetric data key pair — An RSA or elliptic curve \(ECC\) key pair that consists of a public key and a private key\. The private key is protected by a symmetric KMS key in AWS KMS\. You can use your data key pair outside of AWS KMS to encrypt and decrypt data, or sign messages and verify signatures\. 

  AWS KMS recommends that your use ECC key pairs for signing, and use RSA key pairs for either encryption or signing, but not both\. However, AWS KMS cannot enforce any restrictions on the use of data key pairs outside of AWS KMS\.

This topic explains how symmetric and asymmetric KMS keys work, how they differ, and how to decide which type of KMS key you need to protect your data\. It also explains how symmetric data keys and asymmetric data key pairs work and how to use them outside of AWS KMS\.

**Regions**

Asymmetric KMS keys and asymmetric data key pairs are supported in all AWS Regions that AWS KMS supports\.

**Learn more**
+ To create symmetric and asymmetric KMS keys, see [Creating keys](create-keys.md)\.
+ To find out whether a KMS key is symmetric or asymmetric, see [Identifying symmetric and asymmetric KMS keys](find-symm-asymm.md)\. 
+ For a table that compares the AWS KMS API operations that apply to each type of KMS key, see [Comparing symmetric and asymmetric KMS keys](symm-asymm-compare.md)\.
+ To examine the difference in the default key policy that the AWS KMS console sets for symmetric and asymmetric KMS keys, see [Allows key users to use the KMS key with AWS services](key-policies.md#key-policy-service-integration)\. 
+ To control access to the key specs, key usage, encryption algorithms, and signing algorithms that principals in your account can use for KMS keys and data keys, see [AWS KMS condition keys](policy-conditions.md#conditions-kms)\.
+ To learn about the request quotas that apply to different types of KMS keys, see [Request quotas](requests-per-second.md)\.
+ To learn how to sign messages and verify signatures with asymmetric KMS keys, see [Digital signing with the new asymmetric keys feature of AWS KMS](http://aws.amazon.com/blogs/security/digital-signing-asymmetric-keys-aws-kms/) in the *AWS Security Blog*\.

**Topics**
+ [About symmetric and asymmetric KMS keys](symm-asymm-concepts.md)
+ [Choosing your KMS key configuration](symm-asymm-choose.md)
+ [Viewing the cryptographic configuration of KMS keys](symm-asymm-crypto-config.md)
+ [Comparing symmetric and asymmetric KMS keys](symm-asymm-compare.md)