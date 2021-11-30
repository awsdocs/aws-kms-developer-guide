# How Amazon Simple Storage Service \(Amazon S3\) uses AWS KMS<a name="services-s3"></a>

This topic discusses how to protect data at rest within Amazon S3 data centers by using AWS KMS\. You can use *client\-side encryption* where you encrypt your data under an AWS KMS key before you send it to Amazon S3\. Or, you can use *server\-side encryption* where Amazon S3 encrypts your data at rest under an KMS key\. 

**Topics**
+ [Server\-Side Encryption: Using SSE\-KMS](#sse)
+ [Using the Amazon S3 encryption client](#sse-client)
+ [Encryption context](#s3-encryption-context)

## Server\-Side Encryption: Using SSE\-KMS<a name="sse"></a>

You can protect data at rest in Amazon S3 by using three different modes of server\-side encryption: SSE\-S3, SSE\-C, or SSE\-KMS\. 
+ SSE\-S3 requires that Amazon S3 manage the data and the encryption keys\. For more information about SSE\-S3, see [Protecting data using server\-side encryption with Amazon S3\-managed encryption keys \(SSE\-S3\)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingServerSideEncryption.html)\.
+ SSE\-C requires that you manage the encryption key\. For more information about SSE\-C, see [Protecting Data Using Server\-Side Encryption with Customer\-Provided Encryption Keys \(SSE\-C\)\. ](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ServerSideEncryptionCustomerKeys.html) 
+ SSE\-KMS requires that AWS manage the data key but you manage the [AWS KMS keys](concepts.md#kms_keys) in AWS KMS\. For more information, see [Protecting data using server\-side encryption with KMS keys \(SSE\-KMS\)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html)\.

The remainder of this topic discusses how to protect data by using server\-side encryption with AWS KMS\-managed keys \(SSE\-KMS\)\. 

You can request encryption and select a KMS key by using the Amazon S3 console or API\. In the console, check the appropriate box to perform encryption and select your KMS key from the list\. For the Amazon S3 API, specify encryption and choose your KMS key by setting the appropriate headers in a GET or PUT request\. For more information, see [Protecting data using server\-side encryption with KMS keys \(SSE\-KMS\)\.](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html) 

**Important**  
Amazon S3 supports only [symmetric KMS keys](concepts.md#symmetric-cmks)\. You cannot use an [asymmetric KMS key](symmetric-asymmetric.md#asymmetric-cmks) to encrypt your data in Amazon S3\. For help determining whether a KMS key is symmetric or asymmetric, see [Identifying symmetric and asymmetric KMS keys](find-symm-asymm.md)\.

You can choose a [customer managed key](concepts.md#customer-cmk) or the [AWS managed key](concepts.md#aws-managed-cmk) for Amazon S3 in your account\. If you choose to encrypt your data using the standard features, AWS KMS and Amazon S3 perform the following actions:
+ Amazon S3 requests a plaintext [data key](concepts.md#data-keys) and a copy of the key encrypted under the specified KMS key\.
+ AWS KMS generates a data key, encrypts it under the KMS key, and sends both the plaintext data key and the encrypted data key to Amazon S3\.
+ Amazon S3 encrypts the data using the data key and removes the plaintext key from memory as soon as possible after use\. 
+ Amazon S3 stores the encrypted data key as metadata with the encrypted data\. 

Amazon S3 and AWS KMS perform the following actions when you request that your data be decrypted\. 
+ Amazon S3 sends the encrypted data key to AWS KMS\.
+ AWS KMS decrypts the key by using the same KMS key and returns the plaintext data key to Amazon S3\.
+ Amazon S3 decrypts the ciphertext and removes the plaintext data key from memory as soon as possible\. 

If you use the optional [S3 Bucket Keys](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-key.html) feature, the following procedure is used\. The S3 Bucket Keys feature is designed to reduce calls to AWS KMS when objects in an encrypted bucket are accessed\. 
+ Amazon S3 requests a data key from AWS KMS using the KMS key for the bucket\. AWS KMS generates a data key and returns a plaintext and encrypted copy of the data key\.
+ Amazon S3 uses this data key as a *bucket key*\. Amazon S3 creates unique data keys outside of AWS KMS for objects in the bucket and encrypts those data keys under the bucket key\. Amazon S3 uses each bucket key for a time\-limited period\.

For more information about using S3 Bucket Keys, see [Reducing the cost of SSE\-KMS with Amazon S3 Bucket Keys](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-key.html) in the Amazon Simple Storage Service User Guide\.

## Using the Amazon S3 encryption client<a name="sse-client"></a>

You can use the [Amazon S3 Encryption Client](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingClientSideEncryption.html) in the AWS SDK in your own application to encrypt objects and upload them to Amazon S3\. This method allows you to encrypt your data locally to ensure its security as it passes to the Amazon S3 service\. The Amazon S3 service receives your encrypted data; it does not play a role in encrypting or decrypting it\. 

The Amazon S3 Encryption Client encrypts the object by using envelope encryption\. The client calls AWS KMS as a part of the encryption call you make when you pass your data to the client\. AWS KMS verifies that you are authorized to use the [AWS KMS key](concepts.md#kms_keys) that you specify and, if so, returns a new plaintext data key and the data key encrypted under the KMS key\. The Amazon S3 Encryption Client encrypts the data by using the plaintext key and then deletes the key from memory\. The encrypted data key is sent to Amazon S3 to store alongside your encrypted data\. 

## Encryption context<a name="s3-encryption-context"></a>

Each service that is integrated with AWS KMS specifies an [encryption context](concepts.md#encrypt_context) when requesting data keys, encrypting, and decrypting\. The encryption context is [additional authenticated data](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#term-aad) \(AAD\) that AWS KMS uses to check for data integrity\.

When an encryption context is specified for an encryption operation, Amazon S3 specifies the same encryption context for the decryption operation\. Otherwise, the decryption fails\.

For Amazon S3, the encryption context key is always `aws:s3:arn`\.

When you use SSE\-KMS or the Amazon S3 encryption client for encryption, the encryption context value is the bucket path\. In the `requestParameters` field of a CloudTrail log file, the encryption context will look similar to the following one\. 

```
"encryptionContext": {
    "aws:s3:arn": "arn:aws:s3:::bucket_name/file_name"
}
```

When you use SSE\-KMS with the optional S3 Bucket Keys feature, the encryption context value is the ARN of the bucket\.

```
"encryptionContext": {
    "aws:s3:arn": "arn:aws:s3:::bucket_name"
}
```