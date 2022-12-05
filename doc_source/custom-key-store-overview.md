# Custom key stores<a name="custom-key-store-overview"></a>

A *key store* is a secure location for storing cryptographic keys\. The default key store in AWS KMS also supports methods for generating and managing the keys that its stores\. By default, the cryptographic key material for the AWS KMS keys that you create in AWS KMS is generated in and protected by hardware security modules \(HSMs\) that are [FIPS 140\-2 validated cryptographic modules](https://csrc.nist.gov/projects/cryptographic-module-validation-program/Certificate/3139)\. Key material for your KMS keys never leave the HSMs unencrypted\.

However, if you require even more control of the HSMs, you can create a custom key store\. 

A *custom key store* is a logical key store within AWS KMS that is backed by a key manager outside of AWS KMS that you own and manage\. Custom key stores combine the convenient and comprehensive key management interface of AWS KMS with the ability to own and control the key material and cryptographic operations\. When you use a KMS key in a custom key store, the cryptographic operations are performed by your key manager using your cryptographic keys\. As a result, you assume more responsibility for the availability and durability of cryptographic keys, and for the operation of the HSMs\. 

AWS KMS supports two types of custom key stores\.
+ An [AWS CloudHSM key store](keystore-cloudhsm.md) is an AWS KMS custom key store backed by an AWS CloudHSM cluster\. When you create a KMS key in your AWS CloudHSM key store, AWS KMS generates a 256\-bit, persistent, non\-exportable Advanced Encryption Standard \(AES\) symmetric key in the associated AWS CloudHSM cluster\. This key material never leaves your AWS CloudHSM clusters unencrypted\. When you use a KMS key in AWS CloudHSM key store, the cryptographic operations are performed in the HSMs in the cluster\. AWS CloudHSM clusters are backed by hardware security modules \(HSMs\) certified at [FIPS 140\-2 Level 3](https://docs.aws.amazon.com/cloudhsm/latest/userguide/compliance.html)\.
+ An [external key store](keystore-external.md) is an AWS KMS custom key store backed by an external key manager outside of AWS that you own and control\. When you use a KMS key in your external key store, all encryption and decryption operations are performed by your external key manager using your cryptographic keys\. External key stores are designed to support a variety of external key managers from different vendors\.

  AWS KMS never directly views, accesses, or interacts with your external key manager or cryptographic keys\. When you encrypt or decrypt with a KMS key in an external key store, the operation is performed by your external key manager using your external keys\. You retain full control over your cryptographic keys, including the ability to refuse or halt a cryptographic operation without interacting with AWS\. However, due to distance and extra processing, KMS keys in an external key store might have poorer latency and performance, and might have different availability characteristics than KMS keys with key material in AWS KMS\.

These two types of custom key stores are quite different from the standard AWS KMS key store and from each other\. Their security models, locus of responsibility, performance, price, and the use cases are also very different\. Before choosing a custom key store, read the related documentation and confirm that the extra configuration and maintenance responsibility is a wise trade\-off for the extra control\. However, if the rules and regulations under which you operate require direct control of key material, a custom key store might be a good choice for you\.

**Unsupported features**

AWS KMS does not support the following features in custom key stores\.
+ [Asymmetric KMS keys](symmetric-asymmetric.md)
+ [Asymmetric data key pairs](concepts.md#data-key-pairs)
+ [HMAC KMS keys](hmac.md)
+ [KMS keys with imported key material](importing-keys.md)
+ [Automatic key rotation](rotate-keys.md)
+ [Multi\-Region keys](multi-region-keys-overview.md)

**Topics**
+ [AWS CloudHSM key stores](keystore-cloudhsm.md)
+ [External key stores](keystore-external.md)