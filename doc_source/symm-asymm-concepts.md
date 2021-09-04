# About symmetric and asymmetric KMS keys<a name="symm-asymm-concepts"></a>

In AWS KMS, you can create symmetric and asymmetric KMS keys\. 

## Symmetric AWS KMS keys<a name="symmetric-cmks"></a>

When you create a AWS KMS key in KMS, by default, you get a symmetric KMS key\. 

In AWS KMS, a *symmetric KMS key* represents a 256\-bit encryption key that never leaves AWS KMS unencrypted\. To use a symmetric KMS key, you must call AWS KMS\. Symmetric keys are used in symmetric encryption, where the same key is used for encryption and decryption\.

Unless your task explicitly requires asymmetric encryption, symmetric KMS keys, which never leave AWS KMS unencrypted, are a good choice\. For information about the cryptographic configuration or *key spec* for symmetric KMS keys, see [SYMMETRIC\_DEFAULT Key Spec](symm-asymm-choose.md#key-spec-symmetric-default)\. For help creating a symmetric KMS key, see [Creating symmetric KMS key](create-keys.md#create-symmetric-cmk)\.

[AWS services that are integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration) use symmetric KMS keys to encrypt your data\. These services do not support encryption with asymmetric KMS keys\. For help determining whether a KMS key is symmetric or asymmetric, see [Identifying symmetric and asymmetric KMS keys](find-symm-asymm.md)\.

You can use a symmetric KMS key in AWS KMS to encrypt, decrypt, and re\-encrypt data, generate data keys and data key pairs, and generate random byte strings\. You can [import your own key material](importing-keys.md) into a symmetric KMS key and create symmetric KMS keys in [custom key stores](custom-key-store-overview.md)\. For a table comparing the operations that you can perform on symmetric and asymmetric KMS keys, see [Comparing Symmetric and Asymmetric KMS keys](symm-asymm-compare.md)\.

## Asymmetric AWS KMS keys<a name="asymmetric-cmks"></a>

You can create an asymmetric KMS key in AWS KMS\. An *asymmetric KMS key* represents a mathematically related public key and private key pair\. You can give the public key to anyone, even if they're not trusted, but the private key must be kept secret\. 

In an asymmetric KMS key, the private key is created in AWS KMS and never leaves AWS KMS unencrypted\. To use the private key, you must call AWS KMS\. You can use the public key within AWS KMS by calling the AWS KMS API operations\. Or, you can [download the public key](download-public-key.md) and use it outside of AWS KMS\.

If your use case requires encryption outside of AWS by users who cannot call AWS KMS, asymmetric KMS keys are a good choice\. However, if you are creating a KMS key to encrypt the data that you store or manage in an AWS service, use a symmetric KMS key\. [AWS services that are integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration) use symmetric KMS keys to encrypt your data\. These services do not support encryption with asymmetric KMS keys\.

AWS KMS supports two types of asymmetric KMS keys\. 
+ **RSA KMS keys**: A KMS key with an RSA key pair for encryption and decryption or signing and verification \(but not both\)\. AWS KMS supports several key lengths for different security requirements\.
+ **Elliptic Curve \(ECC\) KMS keys**: A KMS key with an elliptic curve key pair for signing and verification\. AWS KMS supports several commonly\-used curves\.

For technical details about the encryption and signing algorithms that AWS KMS supports for RSA KMS keys, see [RSA Key Specs](symm-asymm-choose.md#key-spec-rsa)\. For technical details about the signing algorithms that AWS KMS supports for ECC KMS keys, see [Elliptic Curve Key Specs](symm-asymm-choose.md#key-spec-ecc)\.

For a table comparing the operations that you can perform on symmetric and asymmetric KMS keys, see [Comparing Symmetric and Asymmetric KMS keys](symm-asymm-compare.md)\. For help determining whether a KMS key is symmetric or asymmetric, see [Identifying symmetric and asymmetric KMS keys](find-symm-asymm.md)\.

**Regions**

Asymmetric KMS keys and asymmetric data key pairs are supported in all AWS Regions that AWS KMS supports\.