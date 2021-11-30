# Special\-purpose keys<a name="key-types"></a>

AWS Key Management Service \(AWS KMS\) supports several different types of keys for different uses\.

When you create an AWS KMS key in KMS, by default, you get a symmetric KMS key\. In AWS KMS, a *symmetric KMS key* represents a 256\-bit encryption key that never leaves AWS KMS unencrypted\. Symmetric keys are used in symmetric encryption, where the same key is used for encryption and decryption\. Unless your task explicitly requires asymmetric encryption, symmetric KMS keys, which never leave AWS KMS unencrypted, are a good choice\. Also, [AWS services that are integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration) use symmetric KMS keys to encrypt your data\. These services do not support encryption with asymmetric KMS keys\. 

You can use a symmetric KMS key in AWS KMS to encrypt, decrypt, and re\-encrypt data, generate data keys and data key pairs, and generate random byte strings\. You can [import your own key material](importing-keys.md) into a symmetric KMS key and create symmetric KMS keys in [custom key stores](custom-key-store-overview.md)\. For a table comparing the operations that you can perform on symmetric and asymmetric KMS keys, see [Comparing Symmetric and Asymmetric KMS keys](symm-asymm-compare.md)\.
+ [Asymmetric RSA keys](symmetric-asymmetric.md#asymmetric-cmks) for public key cryptography
+ [Asymmetric RSA and ECC keys](symmetric-asymmetric.md#asymmetric-cmks) for signing and verification
+ [Multi\-Region keys](multi-region-keys-overview.md) \(symmetric and asymmetric\) that work like copies of the same key in different AWS Regions
+ [Keys with imported key material](importing-keys.md) that you provide
+ [Keys in a custom key store](custom-key-store-overview.md) that is backed by a AWS CloudHSM cluster



**Topics**
+ [Asymmetric keys in AWS KMS](symmetric-asymmetric.md)
+ [Using multi\-Region keys in AWS KMS](multi-region-keys-overview.md)
+ [Importing key material in AWS KMS keys](importing-keys.md)
+ [Custom key stores](custom-key-store-overview.md)
+ [Key type reference](symm-asymm-compare.md)