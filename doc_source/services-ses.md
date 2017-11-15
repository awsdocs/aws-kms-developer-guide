# How Amazon Simple Email Service \(Amazon SES\) Uses AWS KMS<a name="services-ses"></a>

You can use Amazon Simple Email Service \(Amazon SES\) to receive email, and \(optionally\) to encrypt the received email messages before storing them in an Amazon Simple Storage Service \(Amazon S3\) bucket that you choose\. When you configure Amazon SES to encrypt email messages, you must choose the KMS customer master key \(CMK\) under which Amazon SES encrypts the messages\. You can choose the default CMK in your account for Amazon SES with the alias **aws/ses**, or you can choose a custom CMK that you created separately in AWS KMS\.

For more information about receiving email using Amazon SES, go to [Receiving Email with Amazon SES](http://docs.aws.amazon.com/ses/latest/DeveloperGuide/receiving-email.html) in the *Amazon Simple Email Service Developer Guide*\.


+ [Overview of Amazon SES Encryption Using AWS KMS](#services-ses-overview)
+ [Amazon SES Encryption Context](#services-ses-encryptioncontext)
+ [Giving Amazon SES Permission to Use Your AWS KMS Customer Master Key \(CMK\)](#services-ses-permissions)
+ [Retrieving and Decrypting Email Messages](#services-ses-decrypt)

## Overview of Amazon SES Encryption Using AWS KMS<a name="services-ses-overview"></a>

When you configure Amazon SES to receive email and encrypt the email messages before saving them to your S3 bucket, the process works like this:

1. You [create a receipt rule](http://docs.aws.amazon.com/ses/latest/DeveloperGuide/receiving-email-receipt-rules.html) for Amazon SES, specifying the S3 action, an S3 bucket for storage, and a KMS customer master key \(CMK\) for encryption\.

1. Amazon SES receives an email message that matches your receipt rule\.

1. Amazon SES requests a unique data key encrypted with the KMS CMK that you specified in the applicable receipt rule\.

1. AWS KMS creates a new data key, encrypts it with the specified CMK, and then sends the encrypted and plaintext copies of the data key to Amazon SES\.

1. Amazon SES uses the plaintext data key to encrypt the email message and then removes the plaintext data key from memory as soon as possible after use\.

1. Amazon SES puts the encrypted email message and the encrypted data key in the specified S3 bucket\. The encrypted data key is stored as metadata with the encrypted email message\.

To accomplish [[ERROR] BAD/MISSING LINK TEXT](#SES-requests-data-key) through [[ERROR] BAD/MISSING LINK TEXT](#SES-puts-message-in-bucket), Amazon SES uses the AWS–provided Amazon S3 encryption client\. Use the same client to retrieve your encrypted email messages from Amazon S3 and decrypt them\. For more information, see [Retrieving and Decrypting Email Messages](#services-ses-decrypt)\.

## Amazon SES Encryption Context<a name="services-ses-encryptioncontext"></a>

When Amazon SES requests a data key to encrypt your received email messages \([[ERROR] BAD/MISSING LINK TEXT](#SES-requests-data-key) in the [Overview of Amazon SES Encryption Using AWS KMS](#services-ses-overview)\), it includes encryption context in the request\. The encryption context provides additional authenticated information that AWS KMS uses to ensure data integrity\. The encryption context is also written to your AWS CloudTrail log files, which can help you understand why a given customer master key \(CMK\) was used\. Amazon SES uses the following for the encryption context:

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

For more information about encryption context, go to [Encryption Context](encryption-context.md)\.

## Giving Amazon SES Permission to Use Your AWS KMS Customer Master Key \(CMK\)<a name="services-ses-permissions"></a>

You can use the default customer master key \(CMK\) in your account for Amazon SES with the alias **aws/ses**, or you can use a custom CMK you create\. If you use the default CMK for Amazon SES, you don't need to perform any steps to give Amazon SES permission to use it\. However, to specify a custom CMK when you [add the S3 action](http://docs.aws.amazon.com/ses/latest/DeveloperGuide/receiving-email-action-s3.html) to your Amazon SES receipt rule, you must ensure that Amazon SES has permission to use the CMK to encrypt your email messages\. To give Amazon SES permission to use your custom CMK, add the following statement to your CMK's key policy:

```
{
  "Sid": "Allow SES to encrypt messages using this master key",
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

For more information about the encryption context that Amazon SES uses when encrypting your email messages, go to [Amazon SES Encryption Context](#services-ses-encryptioncontext)\. For general information about encryption context, go to [Encryption Context](encryption-context.md)\.

## Retrieving and Decrypting Email Messages<a name="services-ses-decrypt"></a>

Amazon SES does not have permission to decrypt your encrypted email messages and cannot decrypt them for you\. You must write code to retrieve your email messages from Amazon S3 and decrypt them\. To make this easier, use the Amazon S3 encryption client\. The following AWS SDKs include the Amazon S3 encryption client:

+ [AWS SDK for Java](https://aws.amazon.com/sdk-for-java/) – See [http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/index.html?com/amazonaws/services/s3/AmazonS3EncryptionClient.html](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/index.html?com/amazonaws/services/s3/AmazonS3EncryptionClient.html) in the *AWS SDK for Java API Reference*\.

+ [AWS SDK for Ruby](https://aws.amazon.com/sdk-for-ruby/) – See [http://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/S3/Encryption/Client.html](http://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/S3/Encryption/Client.html) in the *AWS SDK for Ruby API Reference*\.

+ [AWS SDK for \.NET](https://aws.amazon.com/sdk-for-net/) – See [http://docs.aws.amazon.com/sdkfornet/v3/apidocs/index.html?page=S3/TS3EncryptionS3EncryptionClient.html&tocid=Amazon_S3_Encryption_AmazonS3EncryptionClient](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/index.html?page=S3/TS3EncryptionS3EncryptionClient.html&tocid=Amazon_S3_Encryption_AmazonS3EncryptionClient) in the *AWS SDK for \.NET API Reference*\.

+ [AWS SDK for Go](https://aws.amazon.com/sdk-for-go/) – See [http://docs.aws.amazon.com/sdk-for-go/api/service/s3/s3crypto/](http://docs.aws.amazon.com/sdk-for-go/api/service/s3/s3crypto/) in the *AWS SDK for Go API Reference*\.

The Amazon S3 encryption client simplifies the work of constructing the necessary requests to Amazon S3 to retrieve the encrypted email message and to AWS KMS to decrypt the message's encrypted data key, and of decrypting the email message\. For example, to successfully decrypt the encrypted data key you must pass the same encryption context that Amazon SES passed when requesting the data key from AWS KMS \([[ERROR] BAD/MISSING LINK TEXT](#SES-requests-data-key) in the [Overview of Amazon SES Encryption Using AWS KMS](#services-ses-overview)\)\. The Amazon S3 encryption client handles this, and much of the other work, for you\.

For sample code that uses the Amazon S3 encryption client in the AWS SDK for Java to do client\-side decryption, see the following:

+ [Example: Client\-Side Encryption \(Option 1: Using an AWS KMS–Managed Customer Master Key \(AWS SDK for Java\)\)](http://docs.aws.amazon.com/AmazonS3/latest/dev/client-side-using-kms-java.html) in the *Amazon Simple Storage Service Developer Guide*\.

+ [Amazon S3 Encryption with AWS Key Management Service](https://aws.amazon.com/blogs/developer/amazon-s3-encryption-with-aws-key-management-service/) on the AWS Developer Blog\.