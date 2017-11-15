# Encrypting and Decrypting Data Keys<a name="programming-encryption"></a>

This topic shows how to use the [Encrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [Decrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), and [ReEncrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) operations in the AWS KMS API\. 

These operations are designed to encrypt and decrypt data keys\. They use an AWS KMS customer master key \(CMK\) in the encryption operations and they cannot accept more than 4 KB \(4096 bytes\) of data\. Although you might use them to encrypt small amounts of data, such as a password or RSA key, they are not designed to encrypt application data\.

To encrypt application data, use the server\-side encryption features of an AWS service, or a client\-side encryption library, such as the [AWS Encryption SDK](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/) or the [Amazon S3 encryption client](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingClientSideEncryption.html)\. 


+ [Encrypting a Data Key](#encryption)
+ [Decrypting a Data Key](#decryption)
+ [Re\-Encrypting a Data Key Under a Different Customer Master Key](#reencryption)

## Encrypting a Data Key<a name="encryption"></a>

The [Encrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation is designed to encrypt data keys, but it is not frequently used\. The [GenerateDataKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [GenerateDataKeyWithoutPlaintext](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html) operations return encrypted data keys\. You might use this method when you are moving encrypted data to a new region and want to encrypt its data key with a CMK in the new region\. 

For details about the Java implementation of the Encrypt operation, see the [encrypt method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#encrypt-com.amazonaws.services.kms.model.EncryptRequest-) in the *AWS SDK for Java API Reference*\.

```
// Encrypt a data key
//
// Replace the fictitious keyID value with a valid key ID, key ARN, or alias of an AWS CMK.

String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
ByteBuffer plaintext = ByteBuffer.wrap(new byte[]{1,2,3,4,5,6,7,8,9,0});

EncryptRequest req = new EncryptRequest().withKeyId(keyId).withPlaintext(plaintext);
ByteBuffer ciphertext = kms.encrypt(req).getCiphertextBlob();
```

## Decrypting a Data Key<a name="decryption"></a>

To decrypt a data key, use the [Decrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. For details about the Java implementation, see the [decrypt method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#decrypt-com.amazonaws.services.kms.model.DecryptRequest-) in the *AWS SDK for Java API Reference*\.

The `ciphertextBlob` must be a byte buffer that was returned by the [GenerateDataKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), or [Encrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operations\.

```
// Decrypt a data key
//

ByteBuffer ciphertextBlob = Place your ciphertext here;

DecryptRequest req = new DecryptRequest().withCiphertextBlob(ciphertextBlob);
ByteBuffer plainText = kms.decrypt(req).getPlaintext();
```

## Re\-Encrypting a Data Key Under a Different Customer Master Key<a name="reencryption"></a>

To decrypt an encrypted data key, and then immediately re\-encrypt the data key under a different customer master key \(CMK\), use the [ReEncrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) operation\. The operations are performed entirely on the server side within AWS KMS, so they never expose your plaintext outside of AWS KMS\.

The `ciphertextBlob` must be a byte buffer that was returned by the [GenerateDataKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), or [Encrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operations\.

For details about the Java implementation, see the [reEncrypt method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#reEncrypt-com.amazonaws.services.kms.model.ReEncryptRequest-) in the *AWS SDK for Java API Reference*\.

```
// Re-encrypt a data key

ByteBuffer sourceCiphertextBlob = Place your ciphertext here;

// Replace the fictitious keyID value with a valid key ID, key ARN, or alias of an AWS CMK.
String destinationKeyId = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321";

ReEncryptRequest req = new ReEncryptRequest();
req.setCiphertextBlob(sourceCiphertextBlob);
req.setDestinationKeyId(destinationKeyId);
ByteBuffer destinationCipherTextBlob = kms.reEncrypt(req).getCiphertextBlob();
```