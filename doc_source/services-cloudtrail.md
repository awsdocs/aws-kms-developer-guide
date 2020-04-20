# How AWS CloudTrail uses AWS KMS<a name="services-cloudtrail"></a>

You can use AWS CloudTrail to record AWS API calls and other activity for your AWS account and to save the recorded information to log files in an Amazon Simple Storage Service \(Amazon S3\) bucket that you choose\. By default, the log files delivered by CloudTrail to your S3 bucket are encrypted using server\-side encryption with Amazon S3–managed encryption keys \(SSE\-S3\)\. But you can choose instead to use server\-side encryption with AWS KMS–managed keys \(SSE\-KMS\)\. To learn how to encrypt your CloudTrail log files with AWS KMS, see [Encrypting CloudTrail Log Files with AWS KMS–Managed Keys \(SSE\-KMS\)](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/encrypting-cloudtrail-log-files-with-aws-kms.html) in the *AWS CloudTrail User Guide*\.

**Important**  
AWS CloudTrail and Amazon S3 support only [symmetric customer master keys](symm-asymm-concepts.md#symmetric-cmks) \(CMKs\)\. You cannot use an [asymmetric CMK](symm-asymm-concepts.md#asymmetric-cmks) to encrypt your CloudTrail Logs\. For help determining whether a CMK is symmetric or asymmetric, see [Identifying symmetric and asymmetric CMKs](find-symm-asymm.md)\.

**Topics**
+ [Understanding when your CMK is used](#cloudtrail-details)
+ [Understanding how often your CMK is used](#cloudtrail-requests)

## Understanding when your CMK is used<a name="cloudtrail-details"></a>

Encrypting CloudTrail log files with AWS KMS builds on the Amazon S3 feature called server\-side encryption with AWS KMS–managed keys \(SSE\-KMS\)\. To learn more about SSE\-KMS, see [How Amazon Simple Storage Service \(Amazon S3\) uses AWS KMS](services-s3.md) in this guide or [Protecting Data Using Server\-Side Encryption with AWS KMS–Managed Keys \(SSE\-KMS\)](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html) in the *Amazon Simple Storage Service Developer Guide*\.

When you configure AWS CloudTrail to use SSE\-KMS to encrypt your log files, CloudTrail and Amazon S3 use your KMS customer master key \(CMK\) when you perform certain actions with those services\. The following sections explain when and how those services can use your CMK, and provide additional information that you can use to validate this explanation\.

**Contents**
+ [You configure CloudTrail to encrypt log files with your customer master key \(CMK\)](#cloudtrail-details-update-configuration)
+ [CloudTrail puts a log file into your S3 bucket](#cloudtrail-details-put-log-file)
+ [You get an encrypted log file from your S3 bucket](#cloudtrail-details-get-log-file)

### You configure CloudTrail to encrypt log files with your customer master key \(CMK\)<a name="cloudtrail-details-update-configuration"></a>

When you [update your CloudTrail configuration to use your CMK](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/create-kms-key-policy-for-cloudtrail-update-trail.html), CloudTrail sends a [https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) request to AWS KMS to verify that the CMK exists and that CloudTrail has permission to use it for encryption\. CloudTrail does not use the resulting data key\.

The `GenerateDataKey` request includes the following information for the [encryption context](concepts.md#encrypt_context):
+ The [Amazon Resource Name \(ARN\)](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) of the CloudTrail trail
+ The ARN of the S3 bucket and path where the CloudTrail log files are delivered

The `GenerateDataKey` request results in an entry in your CloudTrail logs similar to the following example\. When you see a log entry like this one, you can determine that CloudTrail \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/1.png)\) called the AWS KMS \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/2.png)\) `GenerateDataKey` operation \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/3.png)\) for a specific trail \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/4.png)\)\. AWS KMS created the data key under a specific CMK \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/5.png)\)\.

**Note**  
You might need to scroll to the right to see some of the callouts in the following example log entry\.

```
{
  "eventVersion": "1.02",
  "userIdentity": {
    "type": "IAMUser",
    "principalId": "AIDACKCEVSQ6C2EXAMPLE",
    "arn": "arn:aws:iam::086441151436:user/AWSCloudTrail",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/1.png)
    "accountId": "086441151436",
    "accessKeyId": "AKIAI44QH8DHBEXAMPLE",
    "userName": "AWSCloudTrail",
    "sessionContext": {"attributes": {
      "mfaAuthenticated": "false",
      "creationDate": "2015-11-11T21:15:33Z"
    }},
    "invokedBy": "internal.amazonaws.com"
  },
  "eventTime": "2015-11-11T21:15:33Z",
  "eventSource": "kms.amazonaws.com",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/2.png)
  "eventName": "GenerateDataKey",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/3.png)
  "awsRegion": "us-west-2",
  "sourceIPAddress": "internal.amazonaws.com",
  "userAgent": "internal.amazonaws.com",
  "requestParameters": {
    "keyId": "arn:aws:kms:us-west-2:111122223333:alias/ExampleAliasForCloudTrailCMK",
    "encryptionContext": {
      "aws:cloudtrail:arn": "arn:aws:cloudtrail:us-west-2:111122223333:trail/Default",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/4.png)
      "aws:s3:arn": "arn:aws:s3:::example-bucket-for-CT-logs/AWSLogs/111122223333/"
    },
    "keySpec": "AES_256"
  },
  "responseElements": null,
  "requestID": "581f1f11-88b9-11e5-9c9c-595a1fb59ac0",
  "eventID": "3cdb2457-c035-4890-93b6-181832b9e766",
  "readOnly": true,
  "resources": [{
    "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/5.png)
    "accountId": "111122223333"
  }],
  "eventType": "AwsServiceEvent",
  "recipientAccountId": "111122223333"
}
```

### CloudTrail puts a log file into your S3 bucket<a name="cloudtrail-details-put-log-file"></a>

Each time CloudTrail puts a log file into your S3 bucket, Amazon S3 sends a [https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) request to AWS KMS on behalf of CloudTrail\. In response to this request, AWS KMS generates a unique data key and then sends Amazon S3 two copies of the data key, one in plaintext and one that is encrypted with the specified CMK\. Amazon S3 uses the plaintext data key to encrypt the CloudTrail log file and then removes the plaintext data key from memory as soon as possible after use\. Amazon S3 stores the encrypted data key as metadata with the encrypted CloudTrail log file\.

The `GenerateDataKey` request includes the following information for the [encryption context](concepts.md#encrypt_context):
+ The [Amazon Resource Name \(ARN\)](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) of the CloudTrail trail
+ The ARN of the S3 object \(the CloudTrail log file\)

Each `GenerateDataKey` request results in an entry in your CloudTrail logs similar to the following example\. When you see a log entry like this one, you can determine that CloudTrail \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/1.png)\) called the AWS KMS \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/2.png)\) `GenerateDataKey` operation \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/3.png)\) for a specific trail \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/4.png)\) to protect a specific log file \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/5.png)\)\. AWS KMS created the data key under the specified CMK \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/6.png)\), shown twice in the same log entry\.

**Note**  
You might need to scroll to the right to see some of the callouts in the following example log entry\.

```
{
  "eventVersion": "1.02",
  "userIdentity": {
    "type": "AssumedRole",
    "principalId": "AROACKCEVSQ6C2EXAMPLE:i-34755b85",
    "arn": "arn:aws:sts::086441151436:assumed-role/AWSCloudTrail/i-34755b85",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/1.png)
    "accountId": "086441151436",
    "accessKeyId": "AKIAI44QH8DHBEXAMPLE",
    "sessionContext": {
      "attributes": {
        "mfaAuthenticated": "false",
        "creationDate": "2015-11-11T20:45:25Z"
      },
      "sessionIssuer": {
        "type": "Role",
        "principalId": "AROACKCEVSQ6C2EXAMPLE",
        "arn": "arn:aws:iam::086441151436:role/AWSCloudTrail",
        "accountId": "086441151436",
        "userName": "AWSCloudTrail"
      }
    },
    "invokedBy": "internal.amazonaws.com"
  },
  "eventTime": "2015-11-11T21:15:58Z",
  "eventSource": "kms.amazonaws.com",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/2.png)
  "eventName": "GenerateDataKey",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/3.png)
  "awsRegion": "us-west-2",
  "sourceIPAddress": "internal.amazonaws.com",
  "userAgent": "internal.amazonaws.com",
  "requestParameters": {
    "encryptionContext": {
      "aws:cloudtrail:arn": "arn:aws:cloudtrail:us-west-2:111122223333:trail/Default",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/4.png)
      "aws:s3:arn": "arn:aws:s3:::example-bucket-for-CT-logs/AWSLogs/111122223333/CloudTrail/us-west-2/2015/11/11/111122223333_CloudTrail_us-west-2_20151111T2115Z_7JREEBimdK8d2nC9.json.gz"![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/5.png)
    },
    "keyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/6.png)
    "keySpec": "AES_256"
  },
  "responseElements": null,
  "requestID": "66f3f74a-88b9-11e5-b7fb-63d925c72ffe",
  "eventID": "7738554f-92ab-4e27-83e3-03354b1aa898",
  "readOnly": true,
  "resources": [{
    "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/6.png)
    "accountId": "111122223333"
  }],
  "eventType": "AwsServiceEvent",
  "recipientAccountId": "111122223333"
}
```

### You get an encrypted log file from your S3 bucket<a name="cloudtrail-details-get-log-file"></a>

Each time you get an encrypted CloudTrail log file from your S3 bucket, Amazon S3 sends a [https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) request to AWS KMS on your behalf to decrypt the log file's encrypted data key\. In response to this request, AWS KMS uses your CMK to decrypt the data key and then sends the plaintext data key to Amazon S3\. Amazon S3 uses the plaintext data key to decrypt the CloudTrail log file and then removes the plaintext data key from memory as soon as possible after use\.

The `Decrypt` request includes the following information for the [encryption context](concepts.md#encrypt_context):
+ The [Amazon Resource Name \(ARN\)](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) of the CloudTrail trail
+ The ARN of the S3 object \(the CloudTrail log file\)

Each `Decrypt` request results in an entry in your CloudTrail logs similar to the following example\. When you see a log entry like this one, you can determine that an IAM user in your AWS account \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/1.png)\) called the AWS KMS \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/2.png)\) `Decrypt` operation \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/3.png)\) for a specific trail \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/4.png)\) and a specific log file \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/5.png)\)\. AWS KMS decrypted the data key under a specific CMK \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/6.png)\)\.

**Note**  
You might need to scroll to the right to see some of the callouts in the following example log entry\.

```
{
  "eventVersion": "1.02",
  "userIdentity": {
    "type": "IAMUser",
    "principalId": "AIDACKCEVSQ6C2EXAMPLE",
    "arn": "arn:aws:iam::111122223333:user/cloudtrail-admin",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/1.png)
    "accountId": "111122223333",
    "accessKeyId": "AKIAIOSFODNN7EXAMPLE",
    "userName": "cloudtrail-admin",
    "sessionContext": {"attributes": {
      "mfaAuthenticated": "false",
      "creationDate": "2015-11-11T20:48:04Z"
    }},
    "invokedBy": "signin.amazonaws.com"
  },
  "eventTime": "2015-11-11T21:20:52Z",
  "eventSource": "kms.amazonaws.com",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/2.png)
  "eventName": "Decrypt",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/3.png)
  "awsRegion": "us-west-2",
  "sourceIPAddress": "internal.amazonaws.com",
  "userAgent": "internal.amazonaws.com",
  "requestParameters": {
    "encryptionContext": {
      "aws:cloudtrail:arn": "arn:aws:cloudtrail:us-west-2:111122223333:trail/Default",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/4.png)
      "aws:s3:arn": "arn:aws:s3:::example-bucket-for-CT-logs/AWSLogs/111122223333/CloudTrail/us-west-2/2015/11/11/111122223333_CloudTrail_us-west-2_20151111T2115Z_7JREEBimdK8d2nC9.json.gz"![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/5.png)
    }
  },
  "responseElements": null,
  "requestID": "16a0590a-88ba-11e5-b406-436f15c3ac01",
  "eventID": "9525bee7-5145-42b0-bed5-ab7196a16daa",
  "readOnly": true,
  "resources": [{
    "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/callouts/6.png)
    "accountId": "111122223333"
  }],
  "eventType": "AwsApiCall",
  "recipientAccountId": "111122223333"
}
```

## Understanding how often your CMK is used<a name="cloudtrail-requests"></a>

To predict costs and better understand your AWS bill, you might want to know how often CloudTrail uses your CMK\. AWS KMS charges for all API requests to the service that exceed the free tier\. For the exact charges, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/)\.

When you encrypt CloudTrail log files with AWS KMS–Managed Keys \(SSE\-KMS\), each time [CloudTrail puts a log file into your S3 bucket](#cloudtrail-details-put-log-file) it results in an AWS KMS API request\. Typically, CloudTrail puts a log file into your S3 bucket once every five minutes, which results in approximately 288 AWS KMS API requests per day, per region, and per AWS account\. For example:
+ If you enable this feature in two regions in a single AWS account, you can expect approximately 576 AWS KMS API requests per day \(2 x 288\)\.
+ If you enable this feature in two regions in each of three AWS accounts, you can expect approximately 1,728 AWS KMS API requests per day \(6 x 288\)\.

These numbers represent only the AWS KMS calls that result from `PUT` requests\. They do not count the [decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) calls to AWS KMS that result from `GET` requests when you get an encrypted log file from your S3 bucket\.