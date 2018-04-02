# How Amazon Simple Storage Service \(Amazon S3\) Uses AWS KMS<a name="services-s3"></a>

This topic discusses how to protect data at rest within Amazon S3 data centers by using AWS KMS\. There are two ways to use AWS KMS with Amazon S3\. You can use server\-side encryption to protect your data with a customer master key or you can use a AWS KMS customer master key with the Amazon S3 encryption client to protect your data on the client side\. 

**Topics**
+ [Server\-Side Encryption: Using SSE\-KMS](#sse)
+ [Using the Amazon S3 Encryption Client](#sse-client)
+ [Encryption Context](#s3-encryption-context)

## Server\-Side Encryption: Using SSE\-KMS<a name="sse"></a>

You can protect data at rest in Amazon S3 by using three different modes of server\-side encryption: SSE\-S3, SSE\-C, or SSE\-KMS\. 
+ SSE\-S3 requires that Amazon S3 manage the data and master encryption keys\. For more information about SSE\-S3, see [Protecting Data Using Server\-Side Encryption with AWS\-Managed Encryption Keys\. ](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingServerSideEncryption.html) 
+ SSE\-C requires that you manage the encryption key\. For more information about SSE\-C, see [Protecting Data Using Server\-Side Encryption with Customer\-Provided Encryption Keys \(SSE\-C\)\. ](http://docs.aws.amazon.com/AmazonS3/latest/dev/ServerSideEncryptionCustomerKeys.html) 
+ SSE\-KMS requires that AWS manage the data key but you manage the master key in AWS KMS\. The remainder of this topic discusses how to protect data by using server\-side encryption with AWS KMS\-managed keys \(SSE\-KMS\)\. 

You can request encryption and the master key you want by using the Amazon S3 console or API\. In the console, check the appropriate box to perform encryption and select your key from the list\. For the Amazon S3 API, specify encryption and choose your key by setting the appropriate headers in a GET or PUT request\. For more information, see [ Protecting Data Using Server\-Side Encryption with AWS KMS\-Managed Keys \(SSE\-KMS\)\. ](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html) 

You can choose a specific customer\-managed master key or accept the AWS\-managed key for Amazon S3 under your account\. If you choose to encrypt your data, AWS KMS and Amazon S3 perform the following actions:
+ Amazon S3 requests a plaintext data key and a copy of the key encrypted by using the specified customer\-managed master key or the AWS\-managed master key\.
+ AWS KMS creates a data key, encrypts it by using the master key, and sends both the plaintext data key and the encrypted data key to Amazon S3\.
+ Amazon S3 encrypts the data using the data key and removes the plaintext key from memory as soon as possible after use\. 
+ Amazon S3 stores the encrypted data key as metadata with the encrypted data\. 

Amazon S3 and AWS KMS perform the following actions when you request that your data be decrypted\. 
+ Amazon S3 sends the encrypted data key to AWS KMS\.
+ AWS KMS decrypts the key by using the appropriate master key and sends the plaintext key back to Amazon S3\.
+ Amazon S3 decrypts the ciphertext and removes the plaintext data key from memory as soon as possible\. 

## Using the Amazon S3 Encryption Client<a name="sse-client"></a>

You can use the Amazon S3 encryption client in the AWS SDK from your own application to encrypt objects and upload them to Amazon S3\. This method allows you to encrypt your data locally to ensure its security as it passes to the Amazon S3 service\. The S3 service receives your encrypted data and does not play a role in encrypting or decrypting it\. 

The Amazon S3 encryption client encrypts the object by using envelope encryption\. The client calls AWS KMS as a part of the encryption call you make when you pass your data to the client\. AWS KMS verifies that you are authorized to use the customer master key and, if so, returns a new plaintext data key and the data key encrypted under the customer master key\. The encryption client encrypts the data by using the plaintext key and then deletes the key from memory\. The encrypted data key is sent to Amazon S3 to store alongside your encrypted data\. 

## Encryption Context<a name="s3-encryption-context"></a>

Each service that is integrated with AWS KMS specifies an encryption context when requesting data keys, encrypting, and decrypting\. The encryption context is additional authenticated information that AWS KMS uses to check for data integrity\. That is, when an encryption context is specified for an encryption operation, the service also specifies it for the decryption operation or decryption will not succeed\. If you are using SSE\-KMS or the Amazon S3 encryption client to perform encryption, Amazon S3 uses the bucket path as the encryption context\. In the `requestParameters` field of a CloudTrail log file, the encryption context will look similar to this\. 

```
"encryptionContext": {
    "aws:s3:arn": "arn:aws:s3:::bucket_name/file_name"
},
```

For more information, see [Encryption Context](encryption-context.md)\. 