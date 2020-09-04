# How to choose your CMK configuration<a name="symm-asymm-choose"></a>

The type of CMK that you create depends largely on how your plan to use the CMK, your security requirements, and your authorization requirements\. When creating your CMK, remember that the cryptographic configuration of the CMK, including its key spec and key usage, are established when you create the CMK and cannot be changed\. For help with creating symmetric and asymmetric CMK, see [Creating keys](create-keys.md)\.

AWS KMS supports two CMK key types: **Symmetric** and **Asymmetric**\. Each key type is associated particular [key usage](#symm-asymm-choose-key-usage) and [key spec](#symm-asymm-choose-key-spec) options\.

Use the following guidance to determine which type of CMK you need based on your use case\.

**Encrypt and decrypt data**  
Use a [symmetric CMK](symm-asymm-concepts.md#symmetric-cmks) for most use cases that require encrypting and decrypting data\. The symmetric encryption algorithm that AWS KMS uses is fast, efficient, and assures the confidentiality and authenticity of data\. It supports authenticated encryption with additional authenticated data \(AAD\), defined as an [encryption context](concepts.md#encrypt_context)\. This type of CMK requires both the sender and recipient of encrypted data to have valid AWS credentials to call AWS KMS\.  
If your use case requires encryption outside of AWS by users who cannot call AWS KMS, [asymmetric CMKs](symm-asymm-concepts.md#asymmetric-cmks) are a good choice\. You can distribute the public key of the asymmetric CMK to allow these users to encrypt data\. And your applications that need to decrypt that data can use the private key of the asymmetric CMK within AWS KMS\.

**Sign messages and verify signatures**  
To sign messages and verify signatures, you must use an [asymmetric CMK](symm-asymm-concepts.md#asymmetric-cmks)\. You can use a CMK with a [key spec](#symm-asymm-choose-key-spec) that represents an RSA key pair or an elliptic curve \(ECC\) key pair\. The key spec you choose is determined by the signing algorithm that you want to use\. In some cases, the users who will verify signatures are outside of AWS and can’t call the [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) operation\. In that case, [choose a key spec](#symm-asymm-choose-key-spec) associated with a signing algorithm that these users can support in their local applications\. 

**Perform public key encryption**  
To perform public key encryption, you must use an [asymmetric CMK](symm-asymm-concepts.md#asymmetric-cmks) with an [RSA key spec](#key-spec-rsa-encryption)\. [Elliptic curve \(ECC\) key specs](#key-spec-ecc) cannot be used for public key encryption\. To encrypt data in AWS KMS with the public key of an RSA CMK, use the [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation\. You can also [download the public key](download-public-key.md) and share it with the parties that need to encrypt data outside of AWS KMS\.  
When you download the public key of an asymmetric CMK, you can use it outside of AWS KMS\. But it is no longer subject to the security controls that protect the CMK in AWS KMS\. For example, you cannot use KMS key policies or grants to control use of the public key\. Nor can you control whether the key is used only for encryption and decryption using the RSA encryption algorithms that AWS KMS supports\. For more details, see [Special Considerations for Downloading Public Keys](download-public-key.md#download-public-key-considerations)\.  
To decrypt data that was encrypted with the public key outside of AWS KMS, call the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. The `Decrypt` operation fails if the data was encrypted under a public key from a CMK with a [key usage](#symm-asymm-choose-key-usage) of `SIGN_VERIFY`\. It will also fail if it was encrypted by using an algorithm that KMS does not support for RSA CMKs\.  
To avoid these errors, anyone using a public key outside of AWS KMS must store the key configuration\. The AWS KMS console and the [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) response provide the information that you must include when you share the public key\.

**Use with integrated AWS services**  <a name="cmks-aws-service"></a>
To create a CMK for use with an [AWS service that is integrated with AWS KMS](service-integration.md), consult the documentation for the service\. All AWS services that encrypt data on your behalf require a [symmetric CMK](symm-asymm-concepts.md#symmetric-cmks)\.

In addition to these considerations, CMKs with different key specs have different prices and different request quotas\. For information about AWS KMS pricing, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/)\. For information about request quotas, see [Request quotas](requests-per-second.md)\.

## Selecting the key usage<a name="symm-asymm-choose-key-usage"></a>

The [key usage](concepts.md#key-usage) of a CMK determines whether the CMK is used for encryption and decryption \-or\- signing and verification\. You cannot choose both\. Using a CMK for more than one type of operations makes the product of both operations more vulnerable to attack\.

As shown in the following table, symmetric CMKs can be used only for encryption and decryption\. Elliptic curve \(ECC\) CMKs can be used only for signing and verification\. Key usage decisions are really made only for RSA CMKs\.


**Valid key usage for CMK types**  

| CMK type | Encrypt and decrypt | Sign and verify | 
| --- | --- | --- | 
| Symmetric CMKs | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-successful.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-failed.png) | 
| Asymmetric CMKs with RSA key pairs | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-successful.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-successful.png) | 
| Asymmetric CMKs with ECC key pairs | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-failed.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/icon-successful.png) | 

In the AWS KMS console, you first choose the key type \(symmetric or asymmetric\), and then, for asymmetric CMKs, the key usage\. If you select a symmetric key type, the key usage options do not appear, because symmetric CMKs only support encryption and decryption\. The key usage that you choose determines which [key specs](#symm-asymm-choose-key-spec) are displayed\. 

To choose a key usage in the AWS KMS console:
+ For CMKs with elliptic curve \(ECC\) key material, choose **Sign and verify**\.
+ For CMKs with RSA key material, choose **Encrypt and decrypt** or **Sign and verify**\.

To determine the key usage that principals in your account are permitted to use for CMKs, use the [kms:CustomerMasterKeyUsage](policy-conditions.md#conditions-kms-customer-master-key-usage) condition key\.

## Selecting the key spec<a name="symm-asymm-choose-key-spec"></a>

When you create an asymmetric CMK, you select its [key spec](concepts.md#key-spec)\. The *key spec*, which is a property of every customer master key \(CMK\), represents the cryptographic configuration of your CMK\. You choose the key spec when you create the CMK, and you cannot change it\. If you've selected the wrong key spec, [delete the CMK](deleting-keys.md), and create a new one\.

**Note**  
In AWS KMS API operations, the key spec for CMKs is known as the `CustomerMasterKeySpec`\. This distinguishes it from the key spec for data keys \(`KeySpec`\) and data key pairs \(`KeyPairSpec`\), and the key spec used when wrapping key material for import \(`WrappingKeySpec`\)\. Each key spec type has different values\.

The key spec determines whether the CMK is symmetric or asymmetric, the type of key material in the CMK, and the encryption algorithms or signing algorithms that AWS KMS supports for the CMK\. The key spec that you choose is typically determined by your use case and regulatory requirements\.

To determine the key specs that principals in your account are permitted to use for CMKs, use the [kms:CustomerMasterKeySpec](policy-conditions.md#conditions-kms-customer-master-key-spec) condition key\.

AWS KMS supports the following key specs for CMKs:
+ [Symmetric CMKs](#key-spec-symmetric-default) \(default; encryption and decryption\)
  + SYMMETRIC\_DEFAULT
+ [RSA key specs](#key-spec-rsa) \(encryption and decryption \-or\- signing and verification\)
  + RSA\_2048
  + RSA\_3072
  + RSA\_4096
+ [Elliptic curve key specs](#key-spec-ecc)
  + Asymmetric NIST\-recommended [elliptic curve key pairs](http://tools.ietf.org/html/rfc5753/) \(signing and verification\)
    + ECC\_NIST\_P256 \(secp256r1\)
    + ECC\_NIST\_P384 \(secp384r1\)
    + ECC\_NIST\_P521 \(secp521r1\)
  + Other asymmetric elliptic curve key pairs \(signing and verification\)
    + ECC\_SECG\_P256K1 \([secp256k1](https://en.bitcoin.it/wiki/Secp256k1)\), commonly used for cryptocurrency\.

**Topics**

The following topics provide technical information about the key specs\.
+ [SYMMETRIC\_DEFAULT Key Spec](#key-spec-symmetric-default)
+ [RSA Key Specs](#key-spec-rsa)
+ [Elliptic Curve Key Specs](#key-spec-ecc)

### SYMMETRIC\_DEFAULT key spec<a name="key-spec-symmetric-default"></a>

The default key spec, SYMMETRIC\_DEFAULT, is the key spec for symmetric CMKs\. When you select the **Symmetric** key type in the AWS KMS console, it selects the `SYMMETRIC_DEFAULT` key spec\. In the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation, if you don't specify a `CustomerMasterKeySpec` value, SYMMETRIC\_DEFAULT is selected\. If you don't have a reason to use a different key spec, SYMMETRIC\_DEFAULT is a good choice\.

The encryption algorithm for symmetric CMKs is also known as SYMMETRIC\_DEFAULT\. Currently, this represents a symmetric algorithm based on [Advanced Encryption Standard](https://csrc.nist.gov/csrc/media/publications/fips/197/final/documents/fips-197.pdf) \(AES\) in [Galois Counter Mode](http://csrc.nist.gov/publications/nistpubs/800-38D/SP-800-38D.pdf) \(GCM\) with 256\-bit keys, an industry standard for secure encryption\. The ciphertext that this algorithm generates supports additional authenticated data \(AAD\), such as an [encryption context](concepts.md#encrypt_context), and GCM provides an additional integrity check on the ciphertext\. For technical details, see the [AWS Key Management Service Cryptographic Details](https://d0.awsstatic.com/whitepapers/KMS-Cryptographic-Details.pdf) whitepaper\.

Data encrypted under AES\-256\-GCM is protected now and in the future\. Cryptographers consider this algorithm to be *quantum resistant*\. Theoretical future, large\-scale quantum computing attacks on ciphertexts created under 256\-bit AES\-GCM keys [reduce the effective security of the key to 128 bits](https://www.etsi.org/images/files/ETSIWhitePapers/QuantumSafeWhitepaper.pdf)\. But, this security level is sufficient to make brute force attacks on AWS KMS ciphertexts infeasible\.

You can use a symmetric CMK in AWS KMS to encrypt, decrypt, and re\-encrypt data, and generate data keys and data key pairs\. AWS services that are integrated with AWS KMS use symmetric CMKs to encrypt your data at rest\. You can [import your own key material](importing-keys.md) into a symmetric CMK and create symmetric CMKs in [custom key stores](custom-key-store-overview.md)\. For a table comparing the operations that you can perform on symmetric and asymmetric CMKs, see [Comparing Symmetric and Asymmetric CMKs](symm-asymm-compare.md)\.

### RSA key specs<a name="key-spec-rsa"></a>

When you use an RSA key spec, AWS KMS creates an asymmetric CMK with an RSA key pair\. The private key never leaves AWS KMS unencrypted\. You can use the public key within AWS KMS, or download the public key for use outside of AWS KMS\. 

**Warning**  
When you encrypt data outside of AWS KMS, be sure that you can decrypt your ciphertext\. If you use the public key from a CMK that has been deleted from AWS KMS, the public key from a CMK configured for signing and verification, or an encryption algorithm that is not supported by the CMK, the data is unrecoverable\.

In AWS KMS, you can use asymmetric CMKs with RSA key pairs for encryption and decryption, or signing and verification, but not both\. This property, known as [*key usage*](#symm-asymm-choose-key-usage), is determined separately from the key spec, but you should make that decision before you select a key spec\. 

AWS KMS supports the following RSA key specs for encryption and decryption or signing and verification:
+ RSA\_2048
+ RSA\_3072
+ RSA\_4096

RSA key specs differ by the length of the RSA key in bits\. The RSA key spec that you choose might be determined by your security standards or the requirements of your task\. In general, use the largest key that is practical and affordable for your task\. CMKs with different RSA key specs are priced differently and are subject to different request quotas\. For information about AWS KMS pricing, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/)\. For information about request quotas, see [Request quotas](requests-per-second.md)\.

#### RSA key specs for encryption and decryption<a name="key-spec-rsa-encryption"></a>

When an RSA asymmetric CMK is used for encryption and decryption, you encrypt with the public key and decrypt with the private key\. When you call the `Encrypt` operation in AWS KMS for an RSA CMK, AWS KMS uses the public key in the RSA key pair and the encryption algorithm you specify to encrypt your data\. To decrypt the ciphertext, call the `Decrypt` operation and specify the same CMK and encryption algorithm\. AWS KMS then uses the private key in the RSA key pair to decrypt your data\. 

You can also download the public key and use it to encrypt data outside of AWS KMS\. Be sure to use an encryption algorithm that AWS KMS supports for RSA CMKs\. To decrypt the ciphertext, call the `Decrypt` function with the same CMK and encryption algorithm\. 

AWS KMS supports two encryption algorithms for CMKs with RSA key specs\. These algorithms, which are defined in [PKCS \#1 v2\.2](https://tools.ietf.org/html/rfc8017), differ in the hash function they use internally\. In AWS KMS, the RSAES\_OAEP algorithms always use the same hash function for both hashing purposes and for the [mask generation function](https://tools.ietf.org/html/rfc8017#appendix-B.2) \(MGF1\)\. You are required to specify an encryption algorithm when you call the [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operations\. You can choose a different algorithm for each request\.


**Supported encryption algorithms for RSA key specs**  

| Encryption algorithm | Algorithm description | 
| --- | --- | 
| RSAES\_OAEP\_SHA\_1 | PKCS \#1 v2\.2, Section 7\.1\. RSA encryption with OAEP Padding using SHA\-1 for both the hash and in the MGF1 mask generation function along with an empty label\. | 
| RSAES\_OAEP\_SHA\_256 | PKCS \#1, Section 7\.1\. RSA encryption with OAEP Padding using SHA\-256 for both the hash and in the MGF1 mask generation function along with an empty label\. | 

You cannot configure a CMK to use a particular encryption algorithm\. However, you can use the [kms:EncryptionAlgorithm](policy-conditions.md#conditions-kms-encryption-algorithm) policy condition to specify the encryption algorithms that principals are allowed to use with the CMK\. 

To get the encryption algorithms for a CMK, [view the cryptographic configuration](viewing-keys-console.md#viewing-console-details) of the CMK in the AWS KMS console or use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. AWS KMS also provides the key spec and encryption algorithms when you download your public key, either in the AWS KMS console or by using the [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) operation\. 

You might choose an RSA key spec based on the length of the plaintext data that you can encrypt in each request\. The following table shows the maximum size, in bytes, of the plaintext that you can encrypt in a single call to the [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation\. The values differ with the key spec and encryption algorithm\. To compare, you can use a symmetric CMK to encrypt up to 4096 bytes at one time\. 

To compute the maximum plaintext length in bytes for these algorithms, use the following formula: \(*key\_size\_in\_bits* / 8\) \- \(2 \* *hash\_length\_in\_bits*/8\) \- 2\. For example, for RSA\_2048 with SHA\-256, the maximum plaintext size in bytes is \(2048/8\) \- \(2 \* 256/8\) \-2 = 190\.


**Maximum plaintext size \(in bytes\) in an Encrypt operation**  

|  | Encryption algorithm | Key spec | RSAES\_OAEP\_SHA\_1 | RSAES\_OAEP\_SHA\_256 | 
| --- | --- | --- | --- | --- | 
| RSA\_2048 | 214 | 190 | 
| RSA\_3072 | 342 | 318  | 
| RSA\_4096 | 470 | 446  | 

#### RSA key specs for signing and verification<a name="key-spec-rsa-sign"></a>

When an RSA asymmetric CMK is used for signing and verification, you generate the signature for a message with the private key and verify the signature with the public key\. 

When you call the `Sign` operation in AWS KMS for an asymmetric CMK, AWS KMS uses the private key in the RSA key pair, the message, and the signing algorithm you specify, to generate a signature\. To verify the signature, call the [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) operation\. Specify the signature, plus the same CMK, message, and signing algorithm\. AWS KMS then uses the public key in the RSA key pair to verify the signature\. You can also download the public key and use it to verify the signature outside of AWS KMS\. 

AWS KMS supports the following signing algorithms for CMKs with RSA key spec\. You are required to specify an signing algorithm when you call the [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html) and [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) operations\. You can choose a different algorithm for each request\.


**Supported signing algorithms for RSA key specs**  

| Signing algorithm | Algorithm description | 
| --- | --- | 
| RSASSA\_PKCS1\_V1\_5\_SHA\_256 | PKCS \#1 v2\.2, Section 8\.2, RSA signature with PKCS \#1v1\.5 Padding and SHA\-256 | 
| RSASSA\_PKCS1\_V1\_5\_SHA\_384 | PKCS \#1 v2\.2, Section 8\.2, RSA signature with PKCS \#1v1\.5 Padding and SHA\-384 | 
| RSASSA\_PKCS1\_V1\_5\_SHA\_512 | PKCS \#1 v2\.2, Section 8\.2, RSA signature with PKCS \#1v1\.5 Padding and SHA\-512 | 
| RSASSA\_PSS\_SHA\_256 | PKCS \#1 v2\.2, Section 8\.1, RSA signature with PSS padding using SHA\-256 for both the message digest and the MGF1 mask generation function along with a 256\-bit salt | 
| RSASSA\_PSS\_SHA\_384 | PKCS \#1 v2\.2, Section 8\.1, RSA signature with PSS padding using SHA\-384 for both the message digest and the MGF1 mask generation function along with a 384\-bit salt | 
| RSASSA\_PSS\_SHA\_512 | PKCS \#1 v2\.2, Section 8\.1, RSA signature with PSS padding using SHA\-512 for both the message digest and the MGF1 mask generation function along with a 512\-bit salt | 

You cannot configure a CMK to use particular signing algorithms\. However, you can use the [kms:SigningAlgorithm](policy-conditions.md#conditions-kms-signing-algorithm) policy condition to specify the signing algorithms that principals are allowed to use with the CMK\. 

To get the signing algorithms for a CMK, [view the cryptographic configuration](viewing-keys-console.md#viewing-console-details) of the CMK in the AWS KMS console or by using the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. AWS KMS also provides the key spec and signing algorithms when you download your public key, either in the AWS KMS console or by using the [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) operation\. 

### Elliptic curve key specs<a name="key-spec-ecc"></a>

****

When you use an elliptic curve \(ECC\) key spec, AWS KMS creates an asymmetric CMK with an ECC key pair for signing and verification\. The private key that generates signature never leaves AWS KMS unencrypted\. You can use the public key to [verify signatures](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) within AWS KMS, or [download the public key](importing-keys-get-public-key-and-token.md) for use outside of AWS KMS\. 

AWS KMS supports the following ECC key specs for asymmetric CMKs\.
+ Asymmetric NIST\-recommended elliptic curve key pairs \(signing and verification\)
  + ECC\_NIST\_P256 \(secp256r1\)
  + ECC\_NIST\_P384 \(secp384r1\)
  + ECC\_NIST\_P521 \(secp521r1\)
+ Other asymmetric elliptic curve key pairs \(signing and verification\)
  + ECC\_SECG\_P256K1 \([secp256k1](https://en.bitcoin.it/wiki/Secp256k1)\), commonly used for cryptocurrencies\.

The ECC key spec that you choose might be determined by your security standards or the requirements of your task\. In general, use the curve with the most points that is practical and affordable for your task\.

If you're creating an asymmetric CMK to use with cryptocurrencies, use the ECC\_SECG\_P256K1 key spec\. You can also use this key spec for other purposes, but it is required for Bitcoin, and other cryptocurrencies\.

CMKs with different ECC key specs are priced differently and are subject to different request quotas\. For information about AWS KMS pricing, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/)\. For information about request quotas, see [Request quotas](requests-per-second.md)\.

The following table shows the signing algorithms that AWS KMS supports for each of the ECC key specs\. You cannot configure a CMK to use particular signing algorithms\. However, you can use the [kms:SigningAlgorithm](policy-conditions.md#conditions-kms-signing-algorithm) policy condition to specify the signing algorithms that principals are allowed to use with the CMK\. 


**Supported signing algorithms for ECC key specs**  

| Key spec | Signing algorithm | Algorithm description | 
| --- | --- | --- | 
| ECC\_NIST\_P256  | ECDSA\_SHA\_256 | NIST FIPS 186\-4, Section 6\.4, ECDSA signature using the curve specified by the key and SHA\-256 for the message digest\. | 
| ECC\_NIST\_P384 | ECDSA\_SHA\_384 | NIST FIPS 186\-4, Section 6\.4, ECDSA signature using the curve specified by the key and SHA\-384 for the message digest\. | 
| ECC\_NIST\_P521 | ECDSA\_SHA\_512 | NIST FIPS 186\-4, Section 6\.4, ECDSA signature using the curve specified by the key and SHA\-512 for the message digest\. | 
| ECC\_SECG\_P256K1 | ECDSA\_SHA\_256 | NIST FIPS 186\-4, Section 6\.4, ECDSA signature using the curve specified by the key and SHA\-256 for the message digest\. | 