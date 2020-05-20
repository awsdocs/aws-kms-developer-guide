# About symmetric and asymmetric CMKs<a name="symm-asymm-concepts"></a>

In AWS KMS, you can create symmetric and asymmetric CMKs\. 

## Symmetric customer master keys<a name="symmetric-cmks"></a>

When you create a customer master key \(CMK\) in KMS, by default, you get a symmetric CMK\. 

In AWS KMS, a *symmetric CMK* represents a 256\-bit encryption key that never leaves AWS KMS unencrypted\. To use a symmetric CMK, you must call AWS KMS\. Symmetric keys are used in symmetric encryption, where the same key is used for encryption and decryption\.

Unless your task explicitly requires asymmetric encryption, symmetric CMKs, which never leave AWS KMS unencrypted, are a good choice\. For information about the cryptographic configuration or *key spec* for symmetric CMKs, see [SYMMETRIC\_DEFAULT Key Spec](symm-asymm-choose.md#key-spec-symmetric-default)\. For help creating a symmetric CMK, see [Creating symmetric CMKs](create-keys.md#create-symmetric-cmk)\.

[AWS services that are integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration) use symmetric CMKs to encrypt your data\. These services do not support encryption with asymmetric CMKs\. For help determining whether a CMK is symmetric or asymmetric, see [Identifying symmetric and asymmetric CMKs](find-symm-asymm.md)\.

You can use a symmetric CMK in AWS KMS to encrypt, decrypt, and re\-encrypt data, generate data keys and data key pairs, and generate random byte strings\. You can [import your own key material](importing-keys.md) into a symmetric CMK and create symmetric CMKs in [custom key stores](custom-key-store-overview.md)\. For a table comparing the operations that you can perform on symmetric and asymmetric CMKs, see [Comparing Symmetric and Asymmetric CMKs](symm-asymm-compare.md)\.

## Asymmetric customer master keys<a name="asymmetric-cmks"></a>

**Note**  
Asymmetric CMKs and asymmetric data key pairs are supported in all AWS Regions that AWS KMS supports except for China \(Beijing\) and China \(Ningxia\)\.

You can create an asymmetric CMK in AWS KMS\. An *asymmetric CMK* represents a mathematically related public key and private key pair\. You can give the public key to anyone, even if they're not trusted, but the private key must be kept secret\. 

In an asymmetric CMK, the private key is created in AWS KMS and never leaves AWS KMS unencrypted\. To use the private key, you must call AWS KMS\. You can use the public key within AWS KMS by calling the AWS KMS API operations\. Or, you can [download the public key](download-public-key.md) and use it outside of AWS KMS\.

If your use case requires encryption outside of AWS by users who cannot call AWS KMS, asymmetric CMKs are a good choice\. However, if you are creating a CMK to encrypt the data that you store or manage in an AWS service, use a symmetric CMK\. [AWS services that are integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration) use symmetric CMKs to encrypt your data\. These services do not support encryption with asymmetric CMKs\.

AWS KMS supports two types of asymmetric CMKs\. 
+ **RSA CMKs**: A CMK with an RSA key pair for encryption and decryption or signing and verification \(but not both\)\. KMS supports several key lengths for different security requirements\.
+ **Elliptic Curve \(ECC\) CMKs**: A CMK with an elliptic curve key pair for signing and verification\. KMS supports several commonly\-used curves\.

For technical details about the encryption and signing algorithms that AWS KMS supports for RSA CMKs, see [RSA Key Specs](symm-asymm-choose.md#key-spec-rsa)\. For technical details about the signing algorithms that AWS KMS supports for ECC CMKs, see [Elliptic Curve Key Specs](symm-asymm-choose.md#key-spec-ecc)\.

For a table comparing the operations that you can perform on symmetric and asymmetric CMKs, see [Comparing Symmetric and Asymmetric CMKs](symm-asymm-compare.md)\. For help determining whether a CMK is symmetric or asymmetric, see [Identifying symmetric and asymmetric CMKs](find-symm-asymm.md)\.