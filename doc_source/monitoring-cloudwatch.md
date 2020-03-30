# Monitoring with Amazon CloudWatch<a name="monitoring-cloudwatch"></a>

You can monitor your customer master keys \(CMKs\) using Amazon CloudWatch, which collects and processes raw data from AWS KMS into readable, near real\-time metrics\. These data are recorded for a period of two weeks so that you can access historical information and gain a better understanding of the usage of your CMKs and their changes over time\. For more information about Amazon CloudWatch, see the [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)\.

**Topics**
+ [AWS KMS metrics and dimensions](#kms-metrics-dimensions)
+ [Creating CloudWatch alarms to monitor AWS KMS metrics](#creating-alarms)
+ [AWS KMS events](#kms-events)

## AWS KMS metrics and dimensions<a name="kms-metrics-dimensions"></a>

When you [import key material into a CMK](importing-keys.md) and set it to expire, AWS KMS sends metrics and dimensions to CloudWatch\. You can view the AWS KMS metrics using the AWS Management Console and the Amazon CloudWatch API\.

### AWS KMS Metrics<a name="kms-metrics"></a>

The `AWS/KMS` namespace includes the following metrics\.

**SecondsUntilKeyMaterialExpiration**  
This metric tracks the number of seconds remaining until imported key material expires\. This metric is valid only for CMKs whose origin is `EXTERNAL` and whose key material is or was set to expire\. The most useful statistic for this metric is `Minimum`, which tells you the smallest amount of time remaining for all data points in the specified statistic period\. The only valid unit for this metric is `Seconds`\.  
Use this metric to track the amount of time that remains until your imported key material expires\. When that amount of time falls below a threshold that you define, you might want to take action such as reimporting the key material with a new expiration date\. You can create a CloudWatch alarm to notify you when that happens\. For more information, see [Creating CloudWatch alarms to monitor AWS KMS metrics](#creating-alarms)\.

### Dimensions for AWS KMS Metrics<a name="kms-dimensions"></a>

AWS KMS metrics use the `AWS/KMS` namespace and have only one valid dimension: `KeyId`\. You can use this dimension to view metric data for a specific CMK or set of CMKs\.

### How do I view AWS KMS metrics?<a name="how-to-view-kms-metrics"></a>

You can view the AWS KMS metrics using the AWS Management Console and the Amazon CloudWatch API\.

**To view metrics using the CloudWatch console**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. If necessary, change the region\. From the navigation bar, select the region where your AWS resources reside\.

1. In the navigation pane, choose **Metrics**\.

1. In the content pane, choose the **All metrics** tab\. Then, below **AWS Namespaces**, choose **KMS**\.

1. Choose **Per\-Key Metrics** to view the individual metrics and dimensions\.

**To view metrics using the Amazon CloudWatch API**  
To view AWS KMS metrics using the CloudWatch API, send a [ListMetrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_ListMetrics.html) request with `Namespace` set to `AWS/KMS`\. The following example shows how to do this with the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/)\.

```
$ aws cloudwatch list-metrics --namespace AWS/KMS
```

## Creating CloudWatch alarms to monitor AWS KMS metrics<a name="creating-alarms"></a>

You can create a CloudWatch alarm that sends an Amazon SNS message when the value of the metric changes and causes the alarm to change state\. An alarm watches a single metric over a time period you specify, and performs one or more actions based on the value of the metric relative to a given threshold over a number of time periods\. The action is a notification sent to an Amazon SNS topic or Auto Scaling policy\. Alarms invoke actions for sustained state changes only\. CloudWatch alarms do not invoke actions simply because they are in a particular state; the state must have changed and been maintained for a specified number of periods\.

**Topics**
+ [Monitor the expiration of imported key material](#key-material-expiration-alarm)
+ [Monitor usage of CMKs that are pending deletion](#cmk-pending-deletion-alarm)

### Create a CloudWatch alarm to monitor the expiration of imported key material<a name="key-material-expiration-alarm"></a>

When you [import key material into a CMK](importing-keys.md), you can optionally specify a time at which the key material expires\. When the key material expires, AWS KMS deletes the key material and the CMK becomes unusable\. To use the CMK again, you must reimport key material\. You can create a CloudWatch alarm to notify you when the amount of time that remains until your imported key material expires falls below a threshold that you define \(for example, 10 days\)\. If you receive a notification from such an alarm, you might want to take action such as reimporting the key material with a new expiration date\.

**To create an alarm to monitor the expiration of imported key material \(AWS Management Console\)**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. If necessary, change the region\. From the navigation bar, select the region where your AWS resources reside\.

1. In the navigation pane, choose **Alarms**\. Then choose **Create Alarm**\.

1. Choose **Browse Metrics** and then choose **KMS**\.

1. Select the check box next to the key ID of the CMK to monitor\.

1. In the lower pane, use the menus to change the statistic to **Minimum** and the time period to **1 Minute**\. Then choose **Next**\.

1. In the **Create Alarm** window, do the following:

   1. For **Name**, type a name such as **KeyMaterialExpiresSoon**\.

   1. Following **Whenever:**, for **is:**, choose **<=** and then type the number of seconds for your threshold value\. For example, to be notified when the time that remains until your imported key material expires is 10 days or less, type **864000**\.

   1. For **for consecutive period\(s\)**, if necessary, type **1**\.

   1. For **Send notification to:**, do one of the following:
      + To use a new Amazon SNS topic, choose **New list** and then type a new topic name\. For **Email list:**, type at least one email address\. You can type more than one email address by separating them with commas\.
      + To use an existing Amazon SNS topic, choose the name of the topic to use\.

   1. Choose **Create Alarm**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/cloudwatch-console-create-alarm-key-material-expiration.png)

1. If you chose to send notifications to an email address, open the email message you receive from no\-reply@sns\.amazonaws\.com with subject "AWS Notification \- Subscription Confirmation\." Confirm your email address by choosing the **Confirm subscription** link in the email message\.
**Important**  
You will not receive email notifications until after you have confirmed your email address\.

### Create a CloudWatch alarm to monitor usage of CMKs that are pending deletion<a name="cmk-pending-deletion-alarm"></a>

When you [schedule key deletion](deleting-keys.md) for a CMK, AWS KMS enforces a waiting period before deleting the CMK\. You can use the waiting period to ensure that you don't need the CMK now or in the future\. You can also configure a CloudWatch alarm to warn you if a person or application attempts to use the CMK during the waiting period\. If you receive a notification from such an alarm, you might want to cancel deletion of the CMK\.

For more information, see [Creating an Amazon CloudWatch alarm to detect usage of a customer master key that is pending deletion](deleting-keys-creating-cloudwatch-alarm.md)\.

## AWS KMS events<a name="kms-events"></a>

AWS KMS integrates with Amazon CloudWatch Events to notify you of certain events that affect your CMKs\. Each event is represented in [JSON \(JavaScript Object Notation\)](http://json.org) and contains the event name, the date and time when the event occurred, the CMK affected, and more\. You can use CloudWatch Events to collect these events and set up rules that route them to one or more *targets* such as AWS Lambda functions, Amazon SNS topics, Amazon SQS queues, streams in Amazon Kinesis Data Streams, or built\-in targets\.

For more information about using CloudWatch Events with other kinds of events, including those emitted by AWS CloudTrail when it records a read/write API request, see the [Amazon CloudWatch Events User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/)\.

The following topics describe the CloudWatch Events that AWS KMS creates\.

**Topics**
+ [KMS CMK rotation](#kms-events-rotation)
+ [KMS imported key material expiration](#kms-events-expiration)
+ [KMS CMK deletion](#kms-events-deletion)

### KMS CMK rotation<a name="kms-events-rotation"></a>

When you enable [automatic key rotation](rotate-keys.md) for a [customer managed CMK](concepts.md#customer-cmk), AWS KMS creates new key material for the CMK each year\. The key material for [AWS managed CMKs](concepts.md#aws-managed-cmk) is automatically rotated every three years\. 

Whenever AWS KMS rotates key material, it sends a KMS CMK Rotation event to CloudWatch Events\. The following is an example of this event\.

```
{
  "version": "0",
  "id": "6a7e8feb-b491-4cf7-a9f1-bf3703467718",
  "detail-type": "KMS CMK Rotation",
  "source": "aws.kms",
  "account": "111122223333",
  "time": "2016-08-25T21:05:33Z",
  "region": "us-west-2",
  "resources": [
    "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
  ],
  "detail": {
    "key-id": "1234abcd-12ab-34cd-56ef-1234567890ab"
  }
}
```

### KMS imported key material expiration<a name="kms-events-expiration"></a>

When you [import key material into a CMK](importing-keys.md), you can optionally specify a time at which the key material expires\. When the key material expires, AWS KMS deletes the key material and sends a corresponding event to CloudWatch Events\. The following is an example of this event\.

```
{
  "version": "0",
  "id": "9da9af57-9253-4406-87cb-7cc400e43465",
  "detail-type": "KMS Imported Key Material Expiration",
  "source": "aws.kms",
  "account": "111122223333",
  "time": "2016-08-22T20:12:19Z",
  "region": "us-west-2",
  "resources": [
    "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
  ],
  "detail": {
    "key-id": "1234abcd-12ab-34cd-56ef-1234567890ab"
  }
}
```

### KMS CMK deletion<a name="kms-events-deletion"></a>

When you [schedule key deletion](deleting-keys.md) for a CMK, AWS KMS enforces a waiting period before deleting the CMK\. After the waiting period ends, AWS KMS deletes the CMK and sends a corresponding event to CloudWatch Events\. The following is an example of this event\.

```
{
  "version": "0",
  "id": "e9ce3425-7d22-412a-a699-e7a5fc3fbc9a",
  "detail-type": "KMS CMK Deletion",
  "source": "aws.kms",
  "account": "111122223333",
  "time": "2016-08-19T03:23:45Z",
  "region": "us-west-2",
  "resources": [
    "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
  ],
  "detail": {
    "key-id": "1234abcd-12ab-34cd-56ef-1234567890ab"
  }
}
```