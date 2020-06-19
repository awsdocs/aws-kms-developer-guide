# Using symmetric and asymmetric keys<a name="symmetric-asymmetric"></a>

AWS KMS protects the [customer master keys](concepts.md#master_keys) \(CMKs\) that you use to protect your data and data keys\. Your secret keys are generated and used only in hardware security modules designed so that no one, including AWS employees, can access the plaintext key material\. 

You can create and manage the CMKs in your AWS account, including setting the [key policies](key-policies.md), [IAM policies](iam-policies.md), and [grants](grants.md) that control access to your CMKs, [enabling and disabling](enabling-keys.md) the CMKs, [creating tags](tagging-keys.md) and aliases, and [deleting the CMKs](deleting-keys.md)\. You can use your CMKs to protect your resources in AWS [services that are integrated with AWS KMS](service-integration.md)\. And, you can audit all operations that use or manage your CMKs in [AWS CloudTrail logs](logging-using-cloudtrail.md)\.

AWS KMS supports symmetric and asymmetric CMKs\.
+ [Symmetric CMK](symm-asymm-concepts.md#symmetric-cmks): Represents a single 256\-bit secret encryption key that never leaves AWS KMS unencrypted\. To use your symmetric CMK, you must call AWS KMS\.
+ [Asymmetric CMK](symm-asymm-concepts.md#asymmetric-cmks): Represents a mathematically related public key and private key pair that you can use for encryption and decryption or signing and verification, but not both\. The private key never leaves AWS KMS unencrypted\. You can use the public key within AWS KMS by calling the AWS KMS API operations, or download the public key and use it outside of AWS KMS\. 

**Note**  
Asymmetric CMKs and asymmetric data key pairs are supported in all AWS Regions that AWS KMS supports except for China \(Beijing\) and China \(Ningxia\)\.

AWS KMS also provides symmetric [data keys](concepts.md#data-keys) and asymmetric [data key pairs](concepts.md#data-key-pairs) that are designed to be used for client\-side cryptography outside of AWS KMS\. The symmetric data key and the private key in an asymmetric data key pair are protected by a symmetric CMK in AWS KMS\. 
+ Symmetric data key — A symmetric encryption key that you can use to encrypt data outside of AWS KMS\. This key is protected by a symmetric CMK in AWS KMS\. 
+ Asymmetric data key pair — An RSA or elliptic curve \(ECC\) key pair that consists of a public key and a private key\. You can use your data key pair outside of AWS KMS to encrypt and decrypt data, or sign messages and verify signatures\. The private key is protected by a symmetric CMK in AWS KMS\.

For information about how to create and use data keys and data key pairs, see [Data keys](concepts.md#data-keys) and [Data key pairs](concepts.md#data-key-pairs)\. To learn how to limit the types of data key pairs that principals in your account are permitted to generate, use the [kms:DataKeyPairSpec](policy-conditions.md#conditions-kms-data-key-spec) condition key\.

This topic explains how symmetric and asymmetric CMKs work, how they differ, and how to decide which type of CMK you need to protect your data\. It also explains how symmetric data keys and asymmetric data key pairs work and how to use them outside of AWS KMS\. 

**Learn more**
+ For a table that compares the AWS KMS API operations that apply to each type of CMK, see [Comparing symmetric and asymmetric CMKs](symm-asymm-compare.md)\.
+ To find out whether a CMK is symmetric or asymmetric, see [Identifying symmetric and asymmetric CMKs](find-symm-asymm.md)\. 
+ To examine the difference in the default key policy that the AWS KMS console sets for symmetric and asymmetric CMKs, see [Allows key users to use the CMK with AWS services](key-policies.md#key-policy-service-integration)\. 
+ To specify the key specs, key usage, encryption algorithms, and signing algorithms that principals in your account can use for CMKs, see [AWS KMS condition keys](policy-conditions.md#conditions-kms)\.
+ To learn about the request quotas that apply to different types of CMKs, see [Request quotas](requests-per-second.md)\.
+ To learn how to sign messages and verify signatures with asymmetric CMKs, see [Digital signing with the new asymmetric keys feature of AWS KMS](http://aws.amazon.com/blogs/security/digital-signing-asymmetric-keys-aws-kms/) in the *AWS Security Blog*\.

**Topics**
+ [About symmetric and asymmetric CMKs](symm-asymm-concepts.md)
+ [How to choose your CMK configuration](symm-asymm-choose.md)
+ [Viewing the cryptographic configuration of CMKs](symm-asymm-crypto-config.md)
+ [Comparing symmetric and asymmetric CMKs](symm-asymm-compare.md)