# Determining past usage of a KMS key<a name="deleting-keys-determining-usage"></a>

Before deleting a KMS key, you might want to know how many ciphertexts were encrypted under that key\. AWS KMS does not store this information, and does not store any of the ciphertexts\. Knowing how a KMS key was used in the past might help you decide whether or not you will need it in the future\. This topic suggest several strategies that can help you determine the past usage of a KMS key\.

**Warning**  
These strategies for determining past and actual usage are effective only for AWS users and AWS KMS operations\. They cannot detect use of the public key of an asymmetric KMS key outside of AWS KMS\. For details about the special risks of deleting asymmetric KMS keys used for public key cryptography, including creating ciphertexts that cannot be decrypted, see [Deleting asymmetric KMS keys](deleting-keys.md#deleting-asymmetric-cmks)\.

**Topics**
+ [Examining KMS key permissions to determine the scope of potential usage](#deleting-keys-usage-key-permissions)
+ [Examining AWS CloudTrail logs to determine actual usage](#deleting-keys-usage-cloudtrail)

## Examining KMS key permissions to determine the scope of potential usage<a name="deleting-keys-usage-key-permissions"></a>

Determining who or what currently has access to a KMS key might help you determine how widely the KMS key was used and whether it is still needed\. To learn how to determine who or what currently has access to a KMS key, go to [Determining access to AWS KMS keys](determining-access.md)\.

## Examining AWS CloudTrail logs to determine actual usage<a name="deleting-keys-usage-cloudtrail"></a>

You might be able to use a KMS key usage history to help you determine whether you have ciphertexts encrypted under a particular KMS key\. 

All AWS KMS API activity is recorded in AWS CloudTrail log files\. If you have [created a CloudTrail trail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail.html) in the region where your KMS keyis located, you can examine your CloudTrail log files to view a history of all AWS KMS API activity for a particular KMS key\. If you don't have a trail, you can still view recent events in your [CloudTrail event history](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events.html)\. For details about how AWS KMS uses CloudTrail, see [Logging AWS KMS API calls with AWS CloudTrail](logging-using-cloudtrail.md)\.

The following examples show CloudTrail log entries that are generated when a KMS key is used to protect an object stored in Amazon Simple Storage Service \(Amazon S3\)\. In this example, the object is uploaded to Amazon S3 using [Protecting data using server\-side encryption with KMS keys \(SSE\-KMS\)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html)\. When you upload an object to Amazon S3 with SSE\-KMS, you specify the KMS key to use for protecting the object\. Amazon S3 uses the AWS KMS [https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation to request a unique data key for the object, and this request event is logged in CloudTrail with an entry similar to the following:

```
{
  "eventVersion": "1.02",
  "userIdentity": {
    "type": "AssumedRole",
    "principalId": "AROACKCEVSQ6C2EXAMPLE:example-user",
    "arn": "arn:aws:sts::111122223333:assumed-role/Admins/example-user",
    "accountId": "111122223333",
    "accessKeyId": "AKIAIOSFODNN7EXAMPLE",
    "sessionContext": {
      "attributes": {
        "mfaAuthenticated": "false",
        "creationDate": "2015-09-10T23:12:48Z"
      },
      "sessionIssuer": {
        "type": "Role",
        "principalId": "AROACKCEVSQ6C2EXAMPLE",
        "arn": "arn:aws:iam::111122223333:role/Admins",
        "accountId": "111122223333",
        "userName": "Admins"
      }
    },
    "invokedBy": "internal.amazonaws.com"
  },
  "eventTime": "2015-09-10T23:58:18Z",
  "eventSource": "kms.amazonaws.com",
  "eventName": "GenerateDataKey",
  "awsRegion": "us-west-2",
  "sourceIPAddress": "internal.amazonaws.com",
  "userAgent": "internal.amazonaws.com",
  "requestParameters": {
    "encryptionContext": {"aws:s3:arn": "arn:aws:s3:::example_bucket/example_object"},
    "keySpec": "AES_256",
    "keyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
  },
  "responseElements": null,
  "requestID": "cea04450-5817-11e5-85aa-97ce46071236",
  "eventID": "80721262-21a5-49b9-8b63-28740e7ce9c9",
  "readOnly": true,
  "resources": [{
    "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
    "accountId": "111122223333"
  }],
  "eventType": "AwsApiCall",
  "recipientAccountId": "111122223333"
}
```

When you later download this object from Amazon S3, Amazon S3 sends a `Decrypt` request to AWS KMS to decrypt the object's data key using the specified KMS key\. When you do this, your CloudTrail log files include an entry similar to the following:

```
{
  "eventVersion": "1.02",
  "userIdentity": {
    "type": "AssumedRole",
    "principalId": "AROACKCEVSQ6C2EXAMPLE:example-user",
    "arn": "arn:aws:sts::111122223333:assumed-role/Admins/example-user",
    "accountId": "111122223333",
    "accessKeyId": "AKIAIOSFODNN7EXAMPLE",
    "sessionContext": {
      "attributes": {
        "mfaAuthenticated": "false",
        "creationDate": "2015-09-10T23:12:48Z"
      },
      "sessionIssuer": {
        "type": "Role",
        "principalId": "AROACKCEVSQ6C2EXAMPLE",
        "arn": "arn:aws:iam::111122223333:role/Admins",
        "accountId": "111122223333",
        "userName": "Admins"
      }
    },
    "invokedBy": "internal.amazonaws.com"
  },
  "eventTime": "2015-09-10T23:58:39Z",
  "eventSource": "kms.amazonaws.com",
  "eventName": "Decrypt",
  "awsRegion": "us-west-2",
  "sourceIPAddress": "internal.amazonaws.com",
  "userAgent": "internal.amazonaws.com",
  "requestParameters": {
    "encryptionContext": {"aws:s3:arn": "arn:aws:s3:::example_bucket/example_object"}},
  "responseElements": null,
  "requestID": "db750745-5817-11e5-93a6-5b87e27d91a0",
  "eventID": "ae551b19-8a09-4cfc-a249-205ddba330e3",
  "readOnly": true,
  "resources": [{
    "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
    "accountId": "111122223333"
  }],
  "eventType": "AwsApiCall",
  "recipientAccountId": "111122223333"
}
```

All AWS KMS API activity is logged by CloudTrail\. By evaluating these log entries, you might be able to determine the past usage of a particular KMS key, and this might help you determine whether or not you want to delete it\.

To see more examples of how AWS KMS API activity appears in your CloudTrail log files, go to [Logging AWS KMS API calls with AWS CloudTrail](logging-using-cloudtrail.md)\. For more information about CloudTrail go to the [AWS CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\.