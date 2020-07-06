# How Amazon Simple Email Service \(Amazon SES\) uses AWS KMS<a name="services-ses"></a>

You can use Amazon Simple Email Service \(Amazon SES\) to receive email, and \(optionally\) to encrypt the received email messages before storing them in an Amazon Simple Storage Service \(Amazon S3\) bucket that you choose\. When you configure Amazon SES to encrypt email messages, you must choose the AWS KMS [customer master key](concepts.md#master_keys) \(CMK\) under which Amazon SES encrypts the messages\. You can choose the [AWS managed CMK](concepts.md#aws-managed-cmk) for Amazon SES \(its alias is **aws/ses**\), or you can choose a symmetric [customer managed CMK](concepts.md#customer-cmk) that you created in AWS KMS\.

**Important**  
Amazon SES supports only [symmetric CMKs](symm-asymm-concepts.md#symmetric-cmks)\. You cannot use an [asymmetric CMK](symm-asymm-concepts.md#asymmetric-cmks) to encrypt your Amazon SES email messages\. For help determining whether a CMK is symmetric or asymmetric, see [Identifying symmetric and asymmetric CMKs](find-symm-asymm.md)\.

For more information about receiving email using Amazon SES, go to [Receiving Email with Amazon SES](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/receiving-email.html) in the *Amazon Simple Email Service Developer Guide*\.

**Topics**
+ [Overview of Amazon SES encryption using AWS KMS](#services-ses-overview)
+ [Amazon SES encryption context](#services-ses-encryptioncontext)
+ [Giving Amazon SES permission to use your AWS KMS customer master key \(CMK\)](#services-ses-permissions)
+ [Getting and decrypting email messages](#services-ses-decrypt)

## Overview of Amazon SES encryption using AWS KMS<a name="services-ses-overview"></a>

When you configure Amazon SES to receive email and encrypt the email messages before saving them to your S3 bucket, the process works like this:

1. You [create a receipt rule](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/receiving-email-receipt-rules.html) for Amazon SES, specifying the S3 action, an S3 bucket for storage, and a KMS customer master key \(CMK\) for encryption\.

1. Amazon SES receives an email message that matches your receipt rule\.

1. <a name="SES-requests-data-key"></a>Amazon SES requests a unique data key encrypted with the KMS CMK that you specified in the applicable receipt rule\.

1. AWS KMS creates a new data key, encrypts it with the specified CMK, and then sends the encrypted and plaintext copies of the data key to Amazon SES\.

1. Amazon SES uses the plaintext data key to encrypt the email message and then removes the plaintext data key from memory as soon as possible after use\.

1. <a name="SES-puts-message-in-bucket"></a>Amazon SES puts the encrypted email message and the encrypted data key in the specified S3 bucket\. The encrypted data key is stored as metadata with the encrypted email message\.

To accomplish [Step 3](#SES-requests-data-key) through [Step 6](#SES-puts-message-in-bucket), Amazon SES uses the AWS–provided Amazon S3 encryption client\. Use the same client to retrieve your encrypted email messages from Amazon S3 and decrypt them\. For more information, see [Getting and decrypting email messages](#services-ses-decrypt)\.

## Amazon SES encryption context<a name="services-ses-encryptioncontext"></a>

When Amazon SES requests a data key to encrypt your received email messages \([Step 3](#SES-requests-data-key) in the [Overview of Amazon SES encryption using AWS KMS](#services-ses-overview)\), it includes an [encryption context](concepts.md#encrypt_context) in the request\. The encryption context provides [additional authenticated data](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#term-aad) \(AAD\) that AWS KMS uses to ensure data integrity\. The encryption context is also written to your AWS CloudTrail log files, which can help you understand why a given customer master key \(CMK\) was used\. Amazon SES uses the following encryption context:
+ The ID of the AWS account in which you've configured Amazon SES to receive email messages
+ The rule name of the Amazon SES receipt rule that invoked the S3 action on the email message
+ The Amazon SES message ID for the email message

The following example shows a JSON representation of the encryption context that Amazon SES uses:

```
{
  "aws:ses:source-account": "111122223333",
  "aws:ses:rule-name": "example-receipt-rule-name",
  "aws:ses:message-id": "d6iitobk75ur44p8kdnnp7g2n800"
}
```

## Giving Amazon SES permission to use your AWS KMS customer master key \(CMK\)<a name="services-ses-permissions"></a>

To encrypt your email messages, you can use the [AWS managed customer master key \(CMK\)](concepts.md#aws-managed-cmk) in your account for Amazon SES \(**aws/ses**\), or you can use a [customer managed CMK](concepts.md#customer-cmk) that you create\. Amazon SES already has permission to use the AWS managed CMK on your behalf\. However, if you specify a customer managed CMK when you [add the S3 action](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/receiving-email-action-s3.html) to your Amazon SES receipt rule, you must give Amazon SES permission to use the CMK to encrypt your email messages\. 

To give Amazon SES permission to use your customer managed CMK, add the following statement to that CMK's [key policy](key-policies.md):

```
{
  "Sid": "Allow SES to encrypt messages using this CMK",
  "Effect": "Allow",
  "Principal": {"Service": "ses.amazonaws.com"},
  "Action": [
    "kms:Encrypt",
    "kms:GenerateDataKey*"
  ],
  "Resource": "*",
  "Condition": {
    "Null": {
      "kms:EncryptionContext:aws:ses:rule-name": false,
      "kms:EncryptionContext:aws:ses:message-id": false
    },
    "StringEquals": {"kms:EncryptionContext:aws:ses:source-account": "ACCOUNT-ID-WITHOUT-HYPHENS"}
  }
}
```

Replace `ACCOUNT-ID-WITHOUT-HYPHENS` with the 12\-digit ID of the AWS account in which you've configured Amazon SES to receive email messages\. This policy statement allows Amazon SES to encrypt data with this CMK only under these conditions:
+ Amazon SES must specify `aws:ses:rule-name` and `aws:ses:message-id` in the `EncryptionContext` of their AWS KMS API requests\.
+ Amazon SES must specify `aws:ses:source-account` in the `EncryptionContext` of their AWS KMS API requests, and the value for `aws:ses:source-account` must match the AWS account ID specified in the key policy\.

For more information about the encryption context that Amazon SES uses when encrypting your email messages, see [Amazon SES encryption context](#services-ses-encryptioncontext)\. For general information about how AWS KMS uses the encryption context, see [encryption context](concepts.md#encrypt_context)\.

## Getting and decrypting email messages<a name="services-ses-decrypt"></a>

Amazon SES does not have permission to decrypt your encrypted email messages and cannot decrypt them for you\. You must write code to get your email messages from Amazon S3 and decrypt them\. To make this easier, use the Amazon S3 encryption client\. The following AWS SDKs include the Amazon S3 encryption client:
+ [AWS SDK for Java](https://aws.amazon.com/sdk-for-java/) – See [https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/index.html?com/amazonaws/services/s3/AmazonS3EncryptionClient.html](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/index.html?com/amazonaws/services/s3/AmazonS3EncryptionClient.html) in the *AWS SDK for Java API Reference*\.
+ [AWS SDK for Ruby](https://aws.amazon.com/sdk-for-ruby/) – See [https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/S3/Encryption/Client.html](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/S3/Encryption/Client.html) in the *AWS SDK for Ruby API Reference*\.
+ [AWS SDK for \.NET](https://aws.amazon.com/sdk-for-net/) – See [https://docs.aws.amazon.com/sdkfornet/v3/apidocs/index.html?page=S3/TS3EncryptionS3EncryptionClient.html&tocid=Amazon_S3_Encryption_AmazonS3EncryptionClient](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/index.html?page=S3/TS3EncryptionS3EncryptionClient.html&tocid=Amazon_S3_Encryption_AmazonS3EncryptionClient) in the *AWS SDK for \.NET API Reference*\.
+ [AWS SDK for Go](https://aws.amazon.com/sdk-for-go/) – See [https://docs.aws.amazon.com/sdk-for-go/api/service/s3/s3crypto/](https://docs.aws.amazon.com/sdk-for-go/api/service/s3/s3crypto/) in the *AWS SDK for Go API Reference*\.

The Amazon S3 encryption client simplifies the work of constructing the necessary requests to Amazon S3 to retrieve the encrypted email message and to AWS KMS to decrypt the message's encrypted data key, and of decrypting the email message\. For example, to successfully decrypt the encrypted data key you must pass the same encryption context that Amazon SES passed when requesting the data key from AWS KMS \([Step 3](#SES-requests-data-key) in the [Overview of Amazon SES encryption using AWS KMS](#services-ses-overview)\)\. The Amazon S3 encryption client handles this, and much of the other work, for you\.

For sample code that uses the Amazon S3 encryption client in the AWS SDK for Java to do client\-side decryption, see the following:
+ [Using a CMK stored in AWS KMS](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingClientSideEncryption.html#client-side-encryption-kms-managed-master-key-intro) in the *Amazon Simple Storage Service Developer Guide*\.
+ [Amazon S3 Encryption with AWS Key Management Service](https://aws.amazon.com/blogs/developer/amazon-s3-encryption-with-aws-key-management-service/) on the AWS Developer Blog\.