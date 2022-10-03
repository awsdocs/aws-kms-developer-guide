# Monitoring with Amazon CloudWatch<a name="monitoring-cloudwatch"></a>

You can monitor your AWS KMS keys using [Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/), an AWS service that collects and processes raw data from AWS KMS into readable, near real\-time metrics\. These data are recorded for a period of two weeks so that you can access historical information and gain a better understanding of the usage of your KMS keys and their changes over time\.

You can use Amazon CloudWatch to alert you to important events, such as the following ones\.
+ The imported key material in a KMS key is nearing its expiration date\.
+ A KMS key that is pending deletion is still being used\. 
+ The key material in a KMS key was automatically rotated\.
+ A KMS key was deleted\.

You can also create an [Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/) alarm that alerts you when your request rate reaches a certain percentage of a quota value\. For details, see [Manage your AWS KMS API request rates using Service Quotas and Amazon CloudWatch](http://aws.amazon.com/blogs/security/manage-your-aws-kms-api-request-rates-using-service-quotas-and-amazon-cloudwatch/) in the *AWS Security Blog*\.

**Topics**
+ [AWS KMS metrics and dimensions](#kms-metrics-dimensions)
+ [Creating CloudWatch alarms to monitor KMS keys](#creating-alarms)

## AWS KMS metrics and dimensions<a name="kms-metrics-dimensions"></a>

When you [import key material into a KMS key](importing-keys.md) and set it to expire, AWS KMS sends metrics and dimensions to CloudWatch\. You can view the AWS KMS metrics using the AWS Management Console and the Amazon CloudWatch API\.

### AWS KMS Metrics<a name="kms-metrics"></a>

The `AWS/KMS` namespace includes the following metrics\.

**SecondsUntilKeyMaterialExpiration**  <a name="key-material-expiration-metric"></a>
This metric tracks the number of seconds remaining until the imported key material in a KMS key expires\. This metric is valid only for KMS keys with an origin of `EXTERNAL` and an expiration date\.  
Use this metric to track the time that remains until your imported key material expires\. When that time falls below a threshold that you define, you might want to take action such as reimporting the key material with a new expiration date\. You can [create a CloudWatch alarm](#key-material-expiration-alarm) to notify you when the expiration date is approaching\.   
The `SecondsUntilKeyMaterialExpiration` metric is specific to a KMS key\. You cannot use this metric to monitor multiple KMS keys or KMS keys that you might create in the future\.  
The most useful statistic for this metric is `Minimum`, which tells you the smallest amount of time remaining for all data points in the specified statistical period\. The only valid unit for this metric is `Seconds`\.

### Dimensions for AWS KMS Metrics<a name="kms-dimensions"></a>

AWS KMS metrics use the `AWS/KMS` namespace and have only one valid dimension: `KeyId`\. You can use this dimension to view metric data for a specific KMS key or set of KMS keys\.

### How do I view AWS KMS metrics?<a name="how-to-view-kms-metrics"></a>

You can view the AWS KMS metrics using the AWS Management Console and the Amazon CloudWatch API\.

**To view metrics using the CloudWatch console**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. If necessary, change the region\. From the navigation bar, select the region where your AWS resources reside\.

1. In the navigation pane, choose **Metrics**, **All metrics**\.

1. On the **Browse** tab, search for `KMS`, and them choose **KMS**\.

1. Choose **Per\-Key Metrics** to view the metrics and dimensions for each affected KMS key\.

   Only affected KMS keys appear in the display\. For example, for the `SecondsUntilKeyMaterialExpiration` metric, the list includes only KMS keys with imported key material that expires\.

1. For a graph of the metric value, choose the metric name, then choose `Add to graph`\. To convert the line graph to a value, choose **Line**, then choose **Number**\. 

**To view metrics using the Amazon CloudWatch API**  
To view AWS KMS metrics using the CloudWatch API, send a [ListMetrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_ListMetrics.html) request with `Namespace` set to `AWS/KMS`\. The following example shows how to do this with the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/)\.

```
$  aws cloudwatch list-metrics --namespace AWS/KMS

{
    "Metrics": [
        {
            "Namespace": "AWS/KMS",
            "MetricName": "SecondsUntilKeyMaterialExpiration",
            "Dimensions": [
                {
                    "Name": "KeyId",
                    "Value": "1234abcd-12ab-34cd-56ef-1234567890ab"
                }
            ]
        }
    ]
}
```

## Creating CloudWatch alarms to monitor KMS keys<a name="creating-alarms"></a>

You can create an Amazon CloudWatch alarm based on an AWS KMS metric\. The alarm sends an email message when a metric value exceeds a threshold specified in the alarm configuration\. The alarm can send the email message to an [Amazon Simple Notification Service \(Amazon SNS\) topic](https://docs.aws.amazon.com/sns/latest/dg/sns-create-topic.html) or an [Amazon EC2 Auto Scaling policy](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scale-based-on-demand.html#as-how-scaling-policies-work)\. For detailed information about CloudWatch alarms, see [Using Amazon CloudWatch alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html) in the Amazon CloudWatch User Guide

### Create an alarm for expiring imported key material<a name="key-material-expiration-alarm"></a>

You can use the [`SecondsUntilKeyMaterialExpiration`](#kms-metrics) metric to create a CloudWatch alarm that notifies you when the imported key material in a KMS key is about to expire\.

When you [import key material into a KMS key](importing-keys.md), you can optionally specify a date and time when the key material expires\. When the key material expires, AWS KMS deletes the key material and the KMS key becomes unusable\. To use the KMS key again, you must [reimport the key material](importing-keys.md#reimport-key-material)\. 

For instructions, see [Creating a CloudWatch alarm for expiration of imported key material](importing-keys.md#imported-key-material-expiration-alarm)\.

### Create an alarm for use of KMS keys that are pending deletion<a name="cmk-pending-deletion-alarm"></a>

When you [schedule deletion](deleting-keys.md) of a KMS key, AWS KMS enforces a waiting period before deleting the KMS key\. You can use the waiting period to ensure that you don't need the KMS key now or in the future\. You can also configure a CloudWatch alarm to warn you if a person or application attempts to use the KMS key in a [cryptographic operation](concepts.md#cryptographic-operations) during the waiting period\. If you receive a notification from such an alarm, you might want to cancel deletion of the KMS key\.

For instructions, see [Creating an alarm that detects use of a KMS key pending deletion](deleting-keys-creating-cloudwatch-alarm.md)\.