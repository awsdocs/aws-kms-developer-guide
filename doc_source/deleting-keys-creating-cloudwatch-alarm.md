# Creating an alarm that detects use of a KMS key pending deletion<a name="deleting-keys-creating-cloudwatch-alarm"></a>

You can combine the features of AWS CloudTrail, Amazon CloudWatch Logs, and Amazon Simple Notification Service \(Amazon SNS\) to create an Amazon CloudWatch alarm that notifies you when someone in your account tries to use a KMS key that is pending deletion\. If you receive this notification, you might want to cancel deletion of the KMS key and reconsider your decision to delete it\.

The following procedures create an alarm that notifies you whenever the "`Key ARN is pending deletion`" error message is written to your CloudTrail log files\. This error message indicates that a person or application tried to use the KMS key in a [cryptographic operation](concepts.md#cryptographic-operations)\. Because the notification is linked to the error message, it is not triggered when you use API operations that are permitted on KMS keys that are pending deletion, such as `ListKeys`, `CancelKeyDeletion`, and `PutKeyPolicy`\. To see a list of the AWS KMS API operations that return this error message, see [Key states of AWS KMS keys](key-state.md)\.

The notification email that you receive does not list the KMS key or the cryptographic operation\. You can find that information in [your CloudTrail log](logging-using-cloudtrail.md)\. Instead, the email reports that the alarm state changed from **OK** to **Alarm**\. For more information about CloudWatch alarms and state changes, see [Using Amazon CloudWatch alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html) in the *Amazon CloudWatch User Guide*\.

**Warning**  
This Amazon CloudWatch alarm cannot detect use of the public key of an asymmetric KMS key outside of AWS KMS\. For details about the special risks of deleting asymmetric KMS keys used for public key cryptography, including creating ciphertexts that cannot be decrypted, see [Deleting asymmetric KMS keys](deleting-keys.md#deleting-asymmetric-cmks)\.

**Topics**
+ [Requirements for a CloudWatch alarm](#cloudwatch-alarm-prerequisites)
+ [Creating the CloudWatch alarm](#deleting-keys-cloudwatch-create-alarm)

## Requirements for a CloudWatch alarm<a name="cloudwatch-alarm-prerequisites"></a>

Before you create a CloudWatch alarm, you must create an AWS CloudTrail trail and configure CloudTrail to deliver CloudTrail log files to Amazon CloudWatch Logs\. You also need an Amazon SNS topic for the alarm notification\.
+ [Create a CloudTrail trail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail.html)\. 

  CloudTrail is automatically enabled on your AWS account when you create the account\. However, for an ongoing record of events in your account, including events for AWS KMS, create a trail\. 
+ [Configure CloudTrail to deliver your log files CloudWatch Logs](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/send-cloudtrail-events-to-cloudwatch-logs.html)\.

  Configure delivery of your CloudTrail log files to CloudWatch Logs\. This allows CloudWatch Logs to monitor the logs for AWS KMS API requests that attempt to use a KMS key that is pending deletion\.
+ [Create an Amazon SNS topic](https://docs.aws.amazon.com/sns/latest/dg/sns-create-topic.html)\.

  When your alarm triggers, it notifies you by sending an email message to an email address in an Amazon Simple Notification Service \(Amazon SNS\) topic\.

## Creating the CloudWatch alarm<a name="deleting-keys-cloudwatch-create-alarm"></a>

In this procedure, you create a CloudWatch log group metric filter that finds instances of the pending deletion exception\. Then, you create a CloudWatch alarm based on the log group metric\. For information about log group metric filters, see [Creating metrics from log events using filters](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/MonitoringLogData.html) in the Amazon CloudWatch Logs User Guide\.

1. Create a CloudWatch metric filter that parses CloudTrail logs\.

   Follow the instructions in [Create a metric filter for a log group](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CreateMetricFilterProcedure.html) using the following required values\. For other fields, accept the default values and provide names as requested\.    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/deleting-keys-creating-cloudwatch-alarm.html)

1. Create a CloudWatch alarm based on the metric filter that you created in Step 1\.

   Follow the instructions in [Creating a CloudWatch alarm based on a log group\-metric filter](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Create_alarm_log_group_metric_filter.html) using the following required values\. For other fields, accept the default values and provide names as requested\.    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/deleting-keys-creating-cloudwatch-alarm.html)

After you complete this procedure, you will receive a notification each time your new CloudWatch alarm enters the `ALARM` state\. If you receive a notification for this alarm, it might mean that a KMS key that is scheduled for deletion is still needed to encrypt or decrypt data\. In that case, [cancel deletion of the KMS key](deleting-keys-scheduling-key-deletion.md) and reconsider your decision to delete it\.