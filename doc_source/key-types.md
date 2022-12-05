# Special\-purpose keys<a name="key-types"></a>

AWS Key Management Service \(AWS KMS\) supports several different types of keys for different uses\.

When you create an AWS KMS key, by default, you get a symmetric encryption KMS key\. In AWS KMS, a *symmetric encryption KMS key* represents a 256\-bit AES\-GCM key that is used for encryption and decryption, except in China Regions, where it represents a symmetric a 128\-bit symmetric key that uses SM4 encryption\. Symmetric key material never leaves AWS KMS unencrypted\. Unless your task explicitly requires asymmetric encryption or HMAC keys, symmetric encryption KMS keys, which never leave AWS KMS unencrypted, are a good choice\. Also, [AWS services that are integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration) use only symmetric encryption KMS keys to encrypt your data\. These services do not support encryption with asymmetric KMS keys\. 

You can use a symmetric encryption KMS key in AWS KMS to encrypt, decrypt, and re\-encrypt data, generate data keys and data key pairs, and generate random byte strings\. You can [import your own key material](importing-keys.md) into a symmetric encryption KMS key and create symmetric encryption KMS keys in [custom key stores](custom-key-store-overview.md)\. For a table comparing the operations that you can perform on symmetric and asymmetric KMS keys, see [Key type reference](symm-asymm-compare.md)\.

AWS KMS also supports the following special\-purpose KMS key types:
+ [Asymmetric RSA keys](symmetric-asymmetric.md#asymmetric-cmks) for public key cryptography
+ [Asymmetric RSA and ECC keys](symmetric-asymmetric.md#asymmetric-cmks) for signing and verification
+ [Asymmetric SM2 keys](asymmetric-key-specs.md#key-spec-sm) \(China Regions only\) for public key cryptography or signing and verification
+ [HMAC keys](hmac.md) to generate and verify hash\-based message authentication codes
+ [Multi\-Region keys](multi-region-keys-overview.md) \(symmetric and asymmetric\) that work like copies of the same key in different AWS Regions
+ [Keys with imported key material](importing-keys.md) that you provide
+ [Keys in a custom key store](custom-key-store-overview.md) that is backed by a AWS CloudHSM cluster or an external key manager outside of AWS\.

## Choosing a KMS key type<a name="symm-asymm-choose"></a>

AWS KMS supports several types of KMS keys: symmetric encryption keys, symmetric HMAC keys, asymmetric encryption keys, and asymmetric signing keys\. 

KMS keys differ because they contain different cryptographic key material\.
+ [Symmetric encryption KMS key](concepts.md#symmetric-cmks): Represents a single 256\-bit AES\-GCM encryption key, except in China Regions, where it represents a 128\-bit SM4 encryption key\. Symmetric key material never leaves AWS KMS unencrypted\. To use your symmetric encryption KMS key, you must call AWS KMS\. 

  Symmetric encryption keys, which are the default KMS keys, are ideal for most uses\. If you need a KMS key to protect your data in an AWS service, use a symmetric encryption key unless you are instructed to use another type of key\.
+ [Asymmetric KMS key](symmetric-asymmetric.md#asymmetric-cmks): Represents a mathematically related public key and private key pair that you can use for encryption and decryption or signing and verification, but not both\. The private key never leaves AWS KMS unencrypted\. You can use the public key within AWS KMS by calling the AWS KMS API operations, or download the public key and use it outside of AWS KMS\. 
+ [HMAC KMS key](hmac.md) \(symmetric\): Represents a symmetric key of varying length that is used to generate and verify hash\-based message authentication codes\. The key material in an HMAC KMS key never leaves AWS KMS unencrypted\. To use your HMAC KMS key, you must call AWS KMS\.

The type of KMS key that you create depends largely on how you plan to use the KMS key, your security requirements, and your authorization requirements\. When creating your KMS key, remember that the cryptographic configuration of the KMS key, including its key spec and key usage, are established when you create the KMS key and cannot be changed\. 

Use the following guidance to determine which type of KMS key you need based on your use case\.

**Encrypt and decrypt data**  
Use a [symmetric KMS key](concepts.md#symmetric-cmks) for most use cases that require encrypting and decrypting data\. The symmetric encryption algorithm that AWS KMS uses is fast, efficient, and assures the confidentiality and authenticity of data\. It supports authenticated encryption with additional authenticated data \(AAD\), defined as an [encryption context](concepts.md#encrypt_context)\. This type of KMS key requires both the sender and recipient of encrypted data to have valid AWS credentials to call AWS KMS\.  
If your use case requires encryption outside of AWS by users who cannot call AWS KMS, [asymmetric KMS keys](symmetric-asymmetric.md#asymmetric-cmks) are a good choice\. You can distribute the public key of the asymmetric KMS key to allow these users to encrypt data\. And your applications that need to decrypt that data can use the private key of the asymmetric KMS key within AWS KMS\.

**Sign messages and verify signatures**  
To sign messages and verify signatures, you must use an [asymmetric KMS key](symmetric-asymmetric.md#asymmetric-cmks)\. You can use a KMS key with a [key spec](#symm-asymm-choose-key-spec) that represents an RSA key pair, an elliptic curve \(ECC\) key pair, or an SM2 key pair \(China Regions only\)\. The key spec you choose is determined by the signing algorithm that you want to use\. In some cases, the users who will verify signatures are outside of AWS and can’t call the [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) operation\. In that case, [choose a key spec](#symm-asymm-choose-key-spec) associated with a signing algorithm that these users can support in their local applications\. 

**Perform public key encryption**  
To perform public key encryption, you must use an [asymmetric KMS key](symmetric-asymmetric.md#asymmetric-cmks) with an [RSA key spec](asymmetric-key-specs.md#key-spec-rsa-encryption) or an [SM2 key spec](asymmetric-key-specs.md#key-spec-sm) \(China Regions only\)\. To encrypt data in AWS KMS with the public key of a KMS key pair, use the [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation\. You can also [download the public key](download-public-key.md) and share it with the parties that need to encrypt data outside of AWS KMS\.  
When you download the public key of an asymmetric KMS key, you can use it outside of AWS KMS\. But it is no longer subject to the security controls that protect the KMS key in AWS KMS\. For example, you cannot use AWS KMS key policies or grants to control use of the public key\. Nor can you control whether the key is used only for encryption and decryption using the encryption algorithms that AWS KMS supports\. For more details, see [Special Considerations for Downloading Public Keys](download-public-key.md#download-public-key-considerations)\.  
To decrypt data that was encrypted with the public key outside of AWS KMS, call the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. The `Decrypt` operation fails if the data was encrypted under a public key from a KMS key with a [key usage](#symm-asymm-choose-key-usage) of `SIGN_VERIFY`\. It will also fail if it was encrypted by using an algorithm that AWS KMS does not support for the key spec you selected\. For more information on key specs and supported algorithms, see [Asymmetric key specs](https://docs.aws.amazon.com/kms/latest/developerguide/asymmetric-key-specs.html)\.  
To avoid these errors, anyone using a public key outside of AWS KMS must store the key configuration\. The AWS KMS console and the [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) response provide the information that you must include when you share the public key\.

**Generate and verify HMAC codes**  
To generate and verify hash\-based message authentication codes, use an HMAC KMS key\. When you create an HMAC key in AWS KMS, AWS KMS creates and protects your key material and ensures that you use the correct MAC algorithms for your key\. HMAC codes can also be used as pseudo\-random numbers, and in certain scenarios for symmetric signing and tokenizing\.  
HMAC KMS keys are symmetric keys\. When creating an HMAC KMS key in the AWS KMS console, choose the `Symmetric` key type\.

**Use with AWS services**  <a name="cmks-aws-service"></a>
To create a KMS key for use with an [AWS service that is integrated with AWS KMS](service-integration.md), consult the documentation for the service\. AWS services that encrypt your data require a [symmetric encryption KMS key\.](concepts.md#symmetric-cmks)\.

In addition to these considerations, KMS keys with different key specs have different prices and different request quotas\. For information about AWS KMS pricing, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/)\. For information about request quotas, see [Request quotas](requests-per-second.md)\.

## Selecting the key usage<a name="symm-asymm-choose-key-usage"></a>

The [key usage](concepts.md#key-usage) of a KMS key determines whether the KMS key is used for encryption and decryption, or signing and verifying signatures, or generating and verifying HMAC tags\. Each KMS key has only one key usage\. Using a KMS key for more than one type of operation makes the product of all operations more vulnerable to attack\.

As shown in the following table, symmetric encryption KMS keys can be used only for encryption and decryption\. HMAC KMS keys can be used only for generating and verifying HMAC codes\. Elliptic curve \(ECC\) KMS keys can be used only for signing and verification\. You need to make a key usage decision only for RSA KMS keys\.


**Valid key usage for KMS key types**  

| KMS key type | Encrypt and decryptENCRYPT\_DECRYPT | Sign and verifySIGN\_VERIFY | Generate and verify MACGENERATE\_VERIFY\_MAC | 
| --- | --- | --- | --- | 
| Symmetric encryption KMS keys | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-successful.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-failed.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-failed.png) | 
| HMAC KMS keys \(symmetric\) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-failed.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-failed.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-successful.png) | 
| Asymmetric KMS keys with RSA key pairs | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-successful.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-successful.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-failed.png) | 
| Asymmetric KMS keys with ECC key pairs | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-failed.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-successful.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-failed.png) | 
| Asymmetric KMS keys with SM2 key pairs \(China Regions only\) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-successful.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-successful.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-failed.png) | 

In the AWS KMS console, you first choose the key type \(symmetric or asymmetric\) and then the key usage\. The key type you choose determines which key usage options are displayed\. The key usage you choose determines which [key specs](#symm-asymm-choose-key-spec), if any, are displayed\. 

To choose a key usage in the AWS KMS console:
+ For symmetric encryption KMS keys \(default\), choose **Encrypt and decrypt**\.
+ For HMAC KMS keys, choose **Generate and verify MAC**\.
+ For asymmetric KMS keys with elliptic curve \(ECC\) key material, choose **Sign and verify**\.
+ For asymmetric KMS keys with RSA key material, choose **Encrypt and decrypt** or **Sign and verify**\.
+ For asymmetric KMS keys with SM2 key material, choose **Encrypt and decrypt** or **Sign and verify**\. The SM2 key spec is available only in China Regions\.

To allow principals to create KMS keys only for a particular key usage, use the [kms:KeyUsage](conditions-kms.md#conditions-kms-key-usage) condition key\. You can also use the `kms:KeyUsage` condition key to allow principals to call API operations for a KMS key based on its key usage\. For example, you can allow permission to disable a KMS key only if its key usage is SIGN\_VERIFY\. 

## Selecting the key spec<a name="symm-asymm-choose-key-spec"></a>

When you create an asymmetric KMS key or an HMAC KMS key, you select its [key spec](concepts.md#key-spec)\. The *key spec*, which is a property of every AWS KMS key, represents the cryptographic configuration of your KMS key\. You choose the key spec when you create the KMS key, and you cannot change it\. If you've selected the wrong key spec, [delete the KMS key](deleting-keys.md), and create a new one\.

**Note**  
The key spec for a KMS key was known as a "customer master key spec\." The `CustomerMasterKeySpec` parameter of the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation is deprecated\. Instead, use the `KeySpec` parameter\. The response of the `CreateKey` and [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operations includes a `KeySpec` and `CustomerMasterKeySpec` member with the same value\.

The key spec determines whether the KMS key is symmetric or asymmetric, the type of key material in the KMS key, and the encryption algorithms, signing algorithms, or message authentication code \(MAC\) algorithms that AWS KMS supports for the KMS key\. The key spec that you choose is typically determined by your use case and regulatory requirements\.

To determine the key specs that principals in your account are permitted to use for KMS keys, use the [kms:KeySpec](conditions-kms.md#conditions-kms-key-spec) condition key\.

AWS KMS supports the following key specs for KMS keys:

[Symmetric encryption key spec](asymmetric-key-specs.md#key-spec-symmetric-default) \(default\)  
+ SYMMETRIC\_DEFAULT

[HMAC key specs](hmac.md#hmac-key-specs)  
+ HMAC\_224
+ HMAC\_256
+ HMAC\_384
+ HMAC\_512

[RSA key specs](asymmetric-key-specs.md#key-spec-rsa) \(encryption and decryption \-or\- signing and verification\)  
+ RSA\_2048
+ RSA\_3072
+ RSA\_4096

[Elliptic curve key specs](asymmetric-key-specs.md#key-spec-ecc)  
+ Asymmetric NIST\-recommended [elliptic curve key pairs](https://datatracker.ietf.org/doc/html/rfc5753/) \(signing and verification\)
  + ECC\_NIST\_P256 \(secp256r1\)
  + ECC\_NIST\_P384 \(secp384r1\)
  + ECC\_NIST\_P521 \(secp521r1\)
+ Other asymmetric elliptic curve key pairs \(signing and verification\)
  + ECC\_SECG\_P256K1 \([secp256k1](https://en.bitcoin.it/wiki/Secp256k1)\), commonly used for cryptocurrency\.

[SM2 key spec](asymmetric-key-specs.md#key-spec-sm) \(encryption and decryption \-or\- signing and verification\)  
+ SM2 \(China Regions only\)