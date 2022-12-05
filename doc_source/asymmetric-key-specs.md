# Asymmetric key specs<a name="asymmetric-key-specs"></a>

The following topics provide technical information about the key specs that AWS KMS supports for asymmetric KMS keys\. Information about the SYMMETRIC\_DEFAULT key spec for symmetric encryption keys is included for comparison\.

**Topics**
+ [RSA key specs](#key-spec-rsa)
+ [Elliptic curve key specs](#key-spec-ecc)
+ [SM2 key spec \(China Regions only\)](#key-spec-sm)
+ [SYMMETRIC\_DEFAULT key spec](#key-spec-symmetric-default)

## RSA key specs<a name="key-spec-rsa"></a>

When you use an RSA key spec, AWS KMS creates an asymmetric KMS key with an RSA key pair\. The private key never leaves AWS KMS unencrypted\. You can use the public key within AWS KMS, or download the public key for use outside of AWS KMS\. 

**Warning**  
When you encrypt data outside of AWS KMS, be sure that you can decrypt your ciphertext\. If you use the public key from a KMS key that has been deleted from AWS KMS, the public key from a KMS key configured for signing and verification, or an encryption algorithm that is not supported by the KMS key, the data is unrecoverable\.

In AWS KMS, you can use asymmetric KMS keys with RSA key pairs for encryption and decryption, or signing and verification, but not both\. This property, known as [*key usage*](key-types.md#symm-asymm-choose-key-usage), is determined separately from the key spec, but you should make that decision before you select a key spec\. 

AWS KMS supports the following RSA key specs for encryption and decryption or signing and verification:
+ RSA\_2048
+ RSA\_3072
+ RSA\_4096

RSA key specs differ by the length of the RSA key in bits\. The RSA key spec that you choose might be determined by your security standards or the requirements of your task\. In general, use the largest key that is practical and affordable for your task\. KMS keys with different RSA key specs are priced differently and are subject to different request quotas\. For information about AWS KMS pricing, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/)\. For information about request quotas, see [Request quotas](requests-per-second.md)\.

### RSA key specs for encryption and decryption<a name="key-spec-rsa-encryption"></a>

When an RSA asymmetric KMS key is used for encryption and decryption, you encrypt with the public key and decrypt with the private key\. When you call the `Encrypt` operation in AWS KMS for an RSA KMS key, AWS KMS uses the public key in the RSA key pair and the encryption algorithm you specify to encrypt your data\. To decrypt the ciphertext, call the `Decrypt` operation and specify the same KMS key and encryption algorithm\. AWS KMS then uses the private key in the RSA key pair to decrypt your data\. 

You can also download the public key and use it to encrypt data outside of AWS KMS\. Be sure to use an encryption algorithm that AWS KMS supports for RSA KMS keys\. To decrypt the ciphertext, call the `Decrypt` function with the same KMS key and encryption algorithm\. 

AWS KMS supports two encryption algorithms for KMS keys with RSA key specs\. These algorithms, which are defined in [PKCS \#1 v2\.2](https://tools.ietf.org/html/rfc8017), differ in the hash function they use internally\. In AWS KMS, the RSAES\_OAEP algorithms always use the same hash function for both hashing purposes and for the [mask generation function](https://tools.ietf.org/html/rfc8017#appendix-B.2) \(MGF1\)\. You are required to specify an encryption algorithm when you call the [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operations\. You can choose a different algorithm for each request\.


**Supported encryption algorithms for RSA key specs**  

| Encryption algorithm | Algorithm description | 
| --- | --- | 
| RSAES\_OAEP\_SHA\_1 | PKCS \#1 v2\.2, Section 7\.1\. RSA encryption with OAEP Padding using SHA\-1 for both the hash and in the MGF1 mask generation function along with an empty label\. | 
| RSAES\_OAEP\_SHA\_256 | PKCS \#1, Section 7\.1\. RSA encryption with OAEP Padding using SHA\-256 for both the hash and in the MGF1 mask generation function along with an empty label\. | 

You cannot configure a KMS key to use a particular encryption algorithm\. However, you can use the [kms:EncryptionAlgorithm](conditions-kms.md#conditions-kms-encryption-algorithm) policy condition to specify the encryption algorithms that principals are allowed to use with the KMS key\. 

To get the encryption algorithms for a KMS key, [view the cryptographic configuration](viewing-keys-console.md#viewing-console-details) of the KMS key in the AWS KMS console or use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. AWS KMS also provides the key spec and encryption algorithms when you download your public key, either in the AWS KMS console or by using the [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) operation\. 

You might choose an RSA key spec based on the length of the plaintext data that you can encrypt in each request\. The following table shows the maximum size, in bytes, of the plaintext that you can encrypt in a single call to the [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation\. The values differ with the key spec and encryption algorithm\. To compare, you can use a symmetric encryption KMS key to encrypt up to 4096 bytes at one time\. 

To compute the maximum plaintext length in bytes for these algorithms, use the following formula: \(*key\_size\_in\_bits* / 8\) \- \(2 \* *hash\_length\_in\_bits*/8\) \- 2\. For example, for RSA\_2048 with SHA\-256, the maximum plaintext size in bytes is \(2048/8\) \- \(2 \* 256/8\) \-2 = 190\.


**Maximum plaintext size \(in bytes\) in an Encrypt operation**  

|  | Encryption algorithm | Key spec | RSAES\_OAEP\_SHA\_1 | RSAES\_OAEP\_SHA\_256 | 
| --- | --- | --- | --- | --- | 
| RSA\_2048 | 214 | 190 | 
| RSA\_3072 | 342 | 318  | 
| RSA\_4096 | 470 | 446  | 

### RSA key specs for signing and verification<a name="key-spec-rsa-sign"></a>

When an RSA asymmetric KMS key is used for signing and verification, you generate the signature for a message with the private key and verify the signature with the public key\. 

When you call the `Sign` operation in AWS KMS for an asymmetric KMS key, AWS KMS uses the private key in the RSA key pair, the message, and the signing algorithm you specify, to generate a signature\. To verify the signature, call the [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) operation\. Specify the signature, plus the same KMS key, message, and signing algorithm\. AWS KMS then uses the public key in the RSA key pair to verify the signature\. You can also download the public key and use it to verify the signature outside of AWS KMS\. 

AWS KMS supports the following signing algorithms for KMS keys with RSA key spec\. You are required to specify a signing algorithm when you call the [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html) and [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) operations\. You can choose a different algorithm for each request\.


**Supported signing algorithms for RSA key specs**  

| Signing algorithm | Algorithm description | 
| --- | --- | 
| RSASSA\_PKCS1\_V1\_5\_SHA\_256 | PKCS \#1 v2\.2, Section 8\.2, RSA signature with PKCS \#1v1\.5 Padding and SHA\-256 | 
| RSASSA\_PKCS1\_V1\_5\_SHA\_384 | PKCS \#1 v2\.2, Section 8\.2, RSA signature with PKCS \#1v1\.5 Padding and SHA\-384 | 
| RSASSA\_PKCS1\_V1\_5\_SHA\_512 | PKCS \#1 v2\.2, Section 8\.2, RSA signature with PKCS \#1v1\.5 Padding and SHA\-512 | 
| RSASSA\_PSS\_SHA\_256 | PKCS \#1 v2\.2, Section 8\.1, RSA signature with PSS padding using SHA\-256 for both the message digest and the MGF1 mask generation function along with a 256\-bit salt | 
| RSASSA\_PSS\_SHA\_384 | PKCS \#1 v2\.2, Section 8\.1, RSA signature with PSS padding using SHA\-384 for both the message digest and the MGF1 mask generation function along with a 384\-bit salt | 
| RSASSA\_PSS\_SHA\_512 | PKCS \#1 v2\.2, Section 8\.1, RSA signature with PSS padding using SHA\-512 for both the message digest and the MGF1 mask generation function along with a 512\-bit salt | 

You cannot configure a KMS key to use particular signing algorithms\. However, you can use the [kms:SigningAlgorithm](conditions-kms.md#conditions-kms-signing-algorithm) policy condition to specify the signing algorithms that principals are allowed to use with the KMS key\. 

To get the signing algorithms for a KMS key, [view the cryptographic configuration](viewing-keys-console.md#viewing-console-details) of the KMS key in the AWS KMS console or by using the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. AWS KMS also provides the key spec and signing algorithms when you download your public key, either in the AWS KMS console or by using the [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) operation\. 

## Elliptic curve key specs<a name="key-spec-ecc"></a>

****

When you use an elliptic curve \(ECC\) key spec, AWS KMS creates an asymmetric KMS key with an ECC key pair for signing and verification\. The private key that generates signature never leaves AWS KMS unencrypted\. You can use the public key to [verify signatures](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) within AWS KMS, or [download the public key](importing-keys-get-public-key-and-token.md) for use outside of AWS KMS\. 

AWS KMS supports the following ECC key specs for asymmetric KMS keys\.
+ Asymmetric NIST\-recommended elliptic curve key pairs \(signing and verification\)
  + ECC\_NIST\_P256 \(secp256r1\)
  + ECC\_NIST\_P384 \(secp384r1\)
  + ECC\_NIST\_P521 \(secp521r1\)
+ Other asymmetric elliptic curve key pairs \(signing and verification\)
  + ECC\_SECG\_P256K1 \([secp256k1](https://en.bitcoin.it/wiki/Secp256k1)\), commonly used for cryptocurrencies\.

The ECC key spec that you choose might be determined by your security standards or the requirements of your task\. In general, use the curve with the most points that is practical and affordable for your task\.

If you're creating an asymmetric KMS key to use with cryptocurrencies, use the ECC\_SECG\_P256K1 key spec\. You can also use this key spec for other purposes, but it is required for Bitcoin, and other cryptocurrencies\.

KMS keys with different ECC key specs are priced differently and are subject to different request quotas\. For information about AWS KMS pricing, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/)\. For information about request quotas, see [Request quotas](requests-per-second.md)\.

The following table shows the signing algorithms that AWS KMS supports for each of the ECC key specs\. You cannot configure a KMS key to use particular signing algorithms\. However, you can use the [kms:SigningAlgorithm](conditions-kms.md#conditions-kms-signing-algorithm) policy condition to specify the signing algorithms that principals are allowed to use with the KMS key\. 


**Supported signing algorithms for ECC key specs**  

| Key spec | Signing algorithm | Algorithm description | 
| --- | --- | --- | 
| ECC\_NIST\_P256  | ECDSA\_SHA\_256 | NIST FIPS 186\-4, Section 6\.4, ECDSA signature using the curve specified by the key and SHA\-256 for the message digest\. | 
| ECC\_NIST\_P384 | ECDSA\_SHA\_384 | NIST FIPS 186\-4, Section 6\.4, ECDSA signature using the curve specified by the key and SHA\-384 for the message digest\. | 
| ECC\_NIST\_P521 | ECDSA\_SHA\_512 | NIST FIPS 186\-4, Section 6\.4, ECDSA signature using the curve specified by the key and SHA\-512 for the message digest\. | 
| ECC\_SECG\_P256K1 | ECDSA\_SHA\_256 | NIST FIPS 186\-4, Section 6\.4, ECDSA signature using the curve specified by the key and SHA\-256 for the message digest\. | 

## SM2 key spec \(China Regions only\)<a name="key-spec-sm"></a>

The SM2 key spec is an elliptic curve key spec defined within the GM/T series of specifications published by [China's Office of State Commercial Cryptography Administration \(OSCCA\)](https://www.oscca.gov.cn/)\. The SM2 key spec is available only in China Regions\. When you use the SM2 key spec, AWS KMS creates an asymmetric KMS key with an SM2 key pair\. You can use your SM2 key pair within AWS KMS, or download the public key for use outside of AWS KMS\.

Unlike the ECC key spec, you can use an SM2 KMS key for signing and verification, *or* encryption and decryption\. You must specify the [key usage](key-types.md#symm-asymm-choose-key-usage) when you create the KMS key, and you cannot change it after the key is created\.

AWS KMS supports the following SM2 encryption and signing algorithms:
+   
**SM2PKE** encryption algorithm  
SM2PKE is an elliptic curve based encryption algorithm defined by OSCCA in GM/T 0003\.4\-2012\.
+   
**SM2DSA** signing algorithm  
SM2DSA is an elliptic curve based signing algorithm defined by OSCCA in GM/T 0003\.2\-2012\. SM2DSA requires a distinguishing ID that is hashed with the SM3 hashing algorithm and then combined with the message, or message digest, that you passed to AWS KMS\. This concatenated value is then hashed and signed by AWS KMS\.

### Offline operations with SM2 \(China Regions only\)<a name="key-spec-sm-offline"></a>

You can [download the public key](download-public-key.md) of your SM2 key pair for use in offline operations, that is, operations outside of AWS KMS\. However, when using your SM2 public key offline, you may need to manually perform extra conversions and calculations\. SM2DSA operations may require you to provide a distinguishing ID or calculate a message digest\. SM2PKE encrypt operations may require you to convert the raw ciphertext output to a format AWS KMS can accept\.

To help you with these operations, the `SM2OfflineOperationHelper` class for Java has methods that perform the tasks for you\. You can use this helper class as a model for other cryptographic providers\.

**Important**  
The `SM2OfflineOperationHelper` reference code is designed to be compatible with [Bouncy Castle](https://www.bouncycastle.org/java.html) version 1\.68\. For help with other versions, contact [bouncycastle\.org](https://www.bouncycastle.org)\.

#### Offline verification with SM2 key pairs \(China Regions only\)<a name="key-spec-sm-offline-verification"></a>

To verify a signature outside of AWS KMS with an SM2 public key, you must specify the distinguishing ID\. When you pass a raw message, [https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html#KMS-Sign-request-MessageType](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html#KMS-Sign-request-MessageType), to the [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html) API, AWS KMS uses the default distinguishing ID, `1234567812345678`, defined by OSCCA in GM/T 0009\-2012\. You cannot specify your own distinguishing ID within AWS KMS\.

However, if you are generating a message digest outside of AWS, you can specify your own distinguishing ID, then pass the message digest, [https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html#API_Sign_RequestSyntax](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html#API_Sign_RequestSyntax), to AWS KMS to sign\. To do this, change the `DEFAULT_DISTINGUISHING_ID` value in the `SM2OfflineOperationHelper` class\. The distinguishing ID you specify can be any string up to 8,192 characters long\. After AWS KMS signs the message digest, you need either the message digest or the message and the distinguishing ID used to compute the digest to verify it offline\.

#### `SM2OfflineOperationHelper` class<a name="key-spec-sm-offline-helper"></a>

Within AWS KMS, the raw ciphertext conversions and SM2DSA message digest calculations occur automatically\. Not all cryptographic providers implement SM2 in the same way\. Some libraries, like [OpenSSL](https://openssl.org/) versions 1\.1\.1 and later, perform these actions automatically\. AWS KMS confirmed this behavior in testing with OpenSSL version 3\.0\. Use the following `SM2OfflineOperationHelper` class with libraries, like [Bouncy Castle](https://www.bouncycastle.org/java.html), that require you to perform these conversions and calculations manually\.

The `SM2OfflineOperationHelper` class provides methods for the following offline operations:
+   
**Message digest calculation**  
To generate a message digest offline that you can use for offline verification, or that you can pass to AWS KMS to sign, use the `calculateSM2Digest` method\. The `calculateSM2Digest` method generates a message digest with the SM3 hashing algorithm\. The [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) API returns your public key in binary format\. You must parse the binary key into a Java PublicKey\. Provide the parsed public key with the message\. The method automatically combines your message with the default distinguishing ID, `1234567812345678`, but you can set your own distinguishing ID by changing the `DEFAULT_DISTINGUISHING_ID` value\.
+   
**Verify**  
To verify a signature offline, use the `offlineSM2DSAVerify` method\. The `offlineSM2DSAVerify` method uses the message digest calculated from the specified distinguishing ID, and original message you provide to verify the digital signature\. The [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) API returns your public key in binary format\. You must parse the binary key into a Java PublicKey\. Provide the parsed public key with the original message and the signature you want to verify\. For more details, see [ Offline verification with SM2 key pairs](#key-spec-sm-offline-verification)\.
+   
**Encrypt**  
To encrypt plaintext offline, use the `offlineSM2PKEEncrypt` method\. This method ensures the ciphertext is in a format AWS KMS can decrypt\. The `offlineSM2PKEEncrypt` method encrypts the plaintext, and then converts the raw ciphertext produced by SM2PKE to the ASN\.1 format\. The [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) API returns your public key in binary format\. You must parse the binary key into a Java PublicKey\. Provide the parsed public key with the plaintext that you want to encrypt\.  
If you're unsure whether you need to perform the conversion, use the following OpenSSL operation to test the format of your ciphertext\. If the operation fails, you need to convert the ciphertext to the ASN\.1 format\.  

  ```
  openssl asn1parse -inform DER -in ciphertext.der
  ```

By default, the `SM2OfflineOperationHelper` class uses the default distinguishing ID, `1234567812345678`, when generating message digests for SM2DSA operations\.

```
package com.amazon.kms.utils;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
import java.io.IOException;
import java.math.BigInteger;
import java.nio.ByteBuffer;
import java.nio.charset.StandardCharsets;
import java.security.InvalidKeyException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.security.NoSuchProviderException;
import java.security.PrivateKey;
import java.security.PublicKey;

import org.bouncycastle.crypto.CryptoException;
import org.bouncycastle.jce.interfaces.ECPublicKey;

import java.util.Arrays;

import org.bouncycastle.asn1.ASN1EncodableVector;
import org.bouncycastle.asn1.ASN1Integer;
import org.bouncycastle.asn1.DEROctetString;
import org.bouncycastle.asn1.DERSequence;
import org.bouncycastle.asn1.gm.GMNamedCurves;
import org.bouncycastle.asn1.x9.X9ECParameters;
import org.bouncycastle.crypto.CipherParameters;
import org.bouncycastle.crypto.params.ParametersWithID;
import org.bouncycastle.crypto.params.ParametersWithRandom;
import org.bouncycastle.crypto.signers.SM2Signer;
import org.bouncycastle.jcajce.provider.asymmetric.util.ECUtil;

public class SM2OfflineOperationHelper {
    // You can change the DEFAULT_DISTINGUISHING_ID value to set your own distinguishing ID,
    // the DEFAULT_DISTINGUISHING_ID can be any string up to 8,192 characters long.
    private static final byte[] DEFAULT_DISTINGUISHING_ID = "1234567812345678".getBytes(StandardCharsets.UTF_8);
    private static final X9ECParameters SM2_X9EC_PARAMETERS = GMNamedCurves.getByName("sm2p256v1");

    // ***calculateSM2Digest***
    // Calculate message digest
    public static byte[] calculateSM2Digest(final PublicKey publicKey, final byte[] message) throws
            NoSuchProviderException, NoSuchAlgorithmException {
        final ECPublicKey ecPublicKey = (ECPublicKey) publicKey;

        // Generate SM3 hash of default distinguishing ID, 1234567812345678
        final int entlenA = DEFAULT_DISTINGUISHING_ID.length * 8;
        final byte [] entla = new byte[] { (byte) (entlenA & 0xFF00), (byte) (entlenA & 0x00FF) };
        final byte [] a = SM2_X9EC_PARAMETERS.getCurve().getA().getEncoded();
        final byte [] b = SM2_X9EC_PARAMETERS.getCurve().getB().getEncoded();
        final byte [] xg = SM2_X9EC_PARAMETERS.getG().getXCoord().getEncoded();
        final byte [] yg = SM2_X9EC_PARAMETERS.getG().getYCoord().getEncoded();
        final byte[] xa = ecPublicKey.getQ().getXCoord().getEncoded();
        final byte[] ya = ecPublicKey.getQ().getYCoord().getEncoded();
        final byte[] za = MessageDigest.getInstance("SM3", "BC")
                .digest(ByteBuffer.allocate(entla.length + DEFAULT_DISTINGUISHING_ID.length + a.length + b.length + xg.length + yg.length +
                        xa.length + ya.length).put(entla).put(DEFAULT_DISTINGUISHING_ID).put(a).put(b).put(xg).put(yg).put(xa).put(ya)
                        .array());

        // Combine hashed distinguishing ID with original message to generate final digest
        return MessageDigest.getInstance("SM3", "BC")
                .digest(ByteBuffer.allocate(za.length + message.length).put(za).put(message)
                        .array());
    }

    // ***offlineSM2DSAVerify***
    // Verify digital signature with SM2 public key
    public static boolean offlineSM2DSAVerify(final PublicKey publicKey, final byte [] message,
            final byte [] signature) throws InvalidKeyException {
        final SM2Signer signer = new SM2Signer();
        CipherParameters cipherParameters = ECUtil.generatePublicKeyParameter(publicKey);
        cipherParameters = new ParametersWithID(cipherParameters, DEFAULT_DISTINGUISHING_ID);
        signer.init(false, cipherParameters);
        signer.update(message, 0, message.length);
        return signer.verifySignature(signature);
    }

    // ***offlineSM2PKEEncrypt***
    // Encrypt data with SM2 public key
    public static byte[] offlineSM2PKEEncrypt(final PublicKey publicKey, final byte [] plaintext) throws
            NoSuchPaddingException, NoSuchAlgorithmException, NoSuchProviderException, InvalidKeyException,
            BadPaddingException, IllegalBlockSizeException, IOException {
        final Cipher sm2Cipher = Cipher.getInstance("SM2", "BC");
        sm2Cipher.init(Cipher.ENCRYPT_MODE, publicKey);

        // By default, Bouncy Castle returns raw ciphertext in the c1c2c3 format
        final byte [] cipherText = sm2Cipher.doFinal(plaintext);

        // Convert the raw ciphertext to the ASN.1 format before passing it to AWS KMS
        final ASN1EncodableVector asn1EncodableVector = new ASN1EncodableVector();
        final int coordinateLength = (SM2_X9EC_PARAMETERS.getCurve().getFieldSize() + 7) / 8 * 2 + 1;
        final int sm3HashLength = 32;
        final int xCoordinateInCipherText = 33;
        final int yCoordinateInCipherText = 65;
        byte[] coords = new byte[coordinateLength];
        byte[] sm3Hash = new byte[sm3HashLength];
        byte[] remainingCipherText = new byte[cipherText.length - coordinateLength - sm3HashLength];

        // Split components out of the ciphertext
        System.arraycopy(cipherText, 0, coords, 0, coordinateLength);
        System.arraycopy(cipherText, cipherText.length - sm3HashLength, sm3Hash, 0, sm3HashLength);
        System.arraycopy(cipherText, coordinateLength, remainingCipherText, 0,cipherText.length - coordinateLength - sm3HashLength);

        // Build standard SM2PKE ASN.1 ciphertext vector
        asn1EncodableVector.add(new ASN1Integer(new BigInteger(1, Arrays.copyOfRange(coords, 1, xCoordinateInCipherText))));
        asn1EncodableVector.add(new ASN1Integer(new BigInteger(1, Arrays.copyOfRange(coords, xCoordinateInCipherText, yCoordinateInCipherText))));
        asn1EncodableVector.add(new DEROctetString(sm3Hash));
        asn1EncodableVector.add(new DEROctetString(remainingCipherText));

        return new DERSequence(asn1EncodableVector).getEncoded("DER");
    }
}
```

## SYMMETRIC\_DEFAULT key spec<a name="key-spec-symmetric-default"></a>

The default key spec, SYMMETRIC\_DEFAULT, is the key spec for symmetric encryption KMS keys\. When you select the **Symmetric** key type and the **Encrypt and decrypt** key usage in the AWS KMS console, it selects the `SYMMETRIC_DEFAULT` key spec\. In the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation, if you don't specify a `KeySpec` value, SYMMETRIC\_DEFAULT is selected\. If you don't have a reason to use a different key spec, SYMMETRIC\_DEFAULT is a good choice\.

SYMMETRIC\_DEFAULT currently represents AES\-256\-GCM, a symmetric algorithm based on [Advanced Encryption Standard](https://csrc.nist.gov/csrc/media/publications/fips/197/final/documents/fips-197.pdf) \(AES\) in [Galois Counter Mode](http://csrc.nist.gov/publications/nistpubs/800-38D/SP-800-38D.pdf) \(GCM\) with 256\-bit keys, an industry standard for secure encryption\. The ciphertext that this algorithm generates supports additional authenticated data \(AAD\), such as an [encryption context](concepts.md#encrypt_context), and GCM provides an additional integrity check on the ciphertext\. For technical details, see [AWS Key Management Service Cryptographic Details](https://docs.aws.amazon.com/kms/latest/cryptographic-details/)\.

Data encrypted under AES\-256\-GCM is protected now and in the future\. Cryptographers consider this algorithm to be *quantum resistant*\. Theoretical future, large\-scale quantum computing attacks on ciphertexts created under 256\-bit AES\-GCM keys [reduce the effective security of the key to 128 bits](https://www.etsi.org/images/files/ETSIWhitePapers/QuantumSafeWhitepaper.pdf)\. But, this security level is sufficient to make brute force attacks on AWS KMS ciphertexts infeasible\.

The only exception in China Regions, where SYMMETRIC\_DEFAULT represents a 128\-bit symmetric key that uses SM4 encryption\. You can only create a 128\-bit SM4 key within China Regions\. You cannot create a 256\-bit AES\-GCM KMS key in China Regions\.

You can use a symmetric encryption KMS key in AWS KMS to encrypt, decrypt, and re\-encrypt data, and generate data keys and data key pairs\. AWS services that are integrated with AWS KMS use symmetric encryption KMS keys to encrypt your data at rest\. You can [import your own key material](importing-keys.md) into a symmetric encryption KMS key and create symmetric encryption KMS keys in [custom key stores](custom-key-store-overview.md)\. For a table comparing the operations that you can perform on symmetric and asymmetric KMS keys, see [Comparing Symmetric and Asymmetric KMS keys](symm-asymm-compare.md)\.

For technical details about AWS KMS and symmetric encryption keys, see [AWS Key Management Service Cryptographic Details](https://docs.aws.amazon.com/kms/latest/cryptographic-details/)\.