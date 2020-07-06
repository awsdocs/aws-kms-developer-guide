# How Amazon Simple Storage Service \(Amazon S3\) uses AWS KMS<a name="services-s3"></a>

This topic discusses how to protect data at rest within Amazon S3 data centers by using AWS KMS\. You can use *client\-side encryption* where you encrypt your data under an AWS KMS customer master key \(CMK\) before you send it to Amazon S3\. Or, you can use *server\-side encryption* where Amazon S3 encrypts your data at rest under an AWS KMS CMK\. 

**Topics**
+ [Server\-Side Encryption: Using SSE\-KMS](#sse)
+ [Using the Amazon S3 encryption client](#sse-client)
+ [Encryption context](#s3-encryption-context)

## Server\-Side Encryption: Using SSE\-KMS<a name="sse"></a>

You can protect data at rest in Amazon S3 by using three different modes of server\-side encryption: SSE\-S3, SSE\-C, or SSE\-KMS\. 
+ SSE\-S3 requires that Amazon S3 manage the data and the encryption keys\. For more information about SSE\-S3, see [Protecting Data Using Server\-Side Encryption with Amazon S3\-Managed Encryption Keys \(SSE\-S3\)](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingServerSideEncryption.html)\.
+ SSE\-C requires that you manage the encryption key\. For more information about SSE\-C, see [Protecting Data Using Server\-Side Encryption with Customer\-Provided Encryption Keys \(SSE\-C\)\. ](https://docs.aws.amazon.com/AmazonS3/latest/dev/ServerSideEncryptionCustomerKeys.html) 
+ SSE\-KMS requires that AWS manage the data key but you manage the [customer master key](concepts.md#master_keys) \(CMK\) in AWS KMS\. 

The remainder of this topic discusses how to protect data by using server\-side encryption with AWS KMS\-managed keys \(SSE\-KMS\)\. 

You can request encryption and select a CMK by using the Amazon S3 console or API\. In the console, check the appropriate box to perform encryption and select your CMK from the list\. For the Amazon S3 API, specify encryption and choose your CMK by setting the appropriate headers in a GET or PUT request\. For more information, see [ Protecting Data Using Server\-Side Encryption with AWS KMS\-Managed Keys \(SSE\-KMS\)\. ](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html) 

**Important**  
Amazon S3 supports only [symmetric CMKs](symm-asymm-concepts.md#symmetric-cmks)\. You cannot use an [asymmetric CMK](symm-asymm-concepts.md#asymmetric-cmks) to encrypt your data in Amazon S3\. For help determining whether a CMK is symmetric or asymmetric, see [Identifying symmetric and asymmetric CMKs](find-symm-asymm.md)\.

You can choose a [customer managed CMK](concepts.md#customer-cmk) or the [AWS managed CMK](concepts.md#aws-managed-cmk) for Amazon S3 in your account\. If you choose to encrypt your data, AWS KMS and Amazon S3 perform the following actions:
+ Amazon S3 requests a plaintext [data key](concepts.md#data-keys) and a copy of the key encrypted under the specified CMK\.
+ AWS KMS generates a data key, encrypts it under the CMK, and sends both the plaintext data key and the encrypted data key to Amazon S3\.
+ Amazon S3 encrypts the data using the data key and removes the plaintext key from memory as soon as possible after use\. 
+ Amazon S3 stores the encrypted data key as metadata with the encrypted data\. 

Amazon S3 and AWS KMS perform the following actions when you request that your data be decrypted\. 
+ Amazon S3 sends the encrypted data key to AWS KMS\.
+ AWS KMS decrypts the key by using the same CMK and returns the plaintext data key to Amazon S3\.
+ Amazon S3 decrypts the ciphertext and removes the plaintext data key from memory as soon as possible\. 

## Using the Amazon S3 encryption client<a name="sse-client"></a>

You can use the [Amazon S3 Encryption Client](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingClientSideEncryption.html) in the AWS SDK in your own application to encrypt objects and upload them to Amazon S3\. This method allows you to encrypt your data locally to ensure its security as it passes to the Amazon S3 service\. The Amazon S3 service receives your encrypted data; it does not play a role in encrypting or decrypting it\. 

The Amazon S3 Encryption Client encrypts the object by using envelope encryption\. The client calls AWS KMS as a part of the encryption call you make when you pass your data to the client\. AWS KMS verifies that you are authorized to use the [customer master key](concepts.md#master_keys) \(CMK\) that you specify and, if so, returns a new plaintext data key and the data key encrypted under the CMK\. The Amazon S3 Encryption Client encrypts the data by using the plaintext key and then deletes the key from memory\. The encrypted data key is sent to Amazon S3 to store alongside your encrypted data\. 

## Encryption context<a name="s3-encryption-context"></a>

Each service that is integrated with AWS KMS specifies an [encryption context](concepts.md#encrypt_context) when requesting data keys, encrypting, and decrypting\. The encryption context is [additional authenticated data](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#term-aad) \(AAD\) that AWS KMS uses to check for data integrity\.

When an encryption context is specified for an encryption operation, Amazon S3 specifies the same encryption context for the decryption operation\. Otherwise, the decryption fails\.

If you use SSE\-KMS or the Amazon S3 encryption client for encryption, Amazon S3 uses the bucket path as the encryption context\. In the `requestParameters` field of a CloudTrail log file, the encryption context will look similar to the following one\. 

```
"encryptionContext": {
    "aws:s3:arn": "arn:aws:s3:::bucket_name/file_name"
},
```