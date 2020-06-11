# Monitoring customer master keys<a name="monitoring-overview"></a>

Monitoring is an important part of understanding the availability, state, and usage of your customer master keys \(CMKs\) in AWS KMS and maintaining the reliability, availability, and performance of your AWS solutions\. Collecting monitoring data from all the parts of your AWS solution will help you debug a multipoint failure if one occurs\. Before you start monitoring your CMKs, however, create a monitoring plan that includes answers to the following questions:
+ What are your monitoring goals?
+ What resources will you monitor?
+ How often will you monitor these resources?
+ What [monitoring tools](#monitoring-tools) will you use?
+ Who will perform the monitoring tasks?
+ Who should be notified when something happens?

The next step is to monitor your CMKs over time to establish a baseline for normal AWS KMS usage and expectations in your environment\. As you monitor your CMKs, store historical monitoring data so that you can compare it with current data, identify normal patterns and anomalies, and devise methods to address issues\.

For example, you can monitor AWS KMS API activity and events that affect your CMKs\. When data falls above or below your established norms, you might need to investigate or take corrective action\.

To establish a baseline for normal patterns, monitor the following items:
+ AWS KMS API activity for *data plane* operations\. These are [cryptographic operations](concepts.md#cryptographic-operations) that use a CMK, such as [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html), and [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)\.
+ AWS KMS API activity for *control plane* operations that are important to you\. These operations manage a CMK, and you might want to monitor those that change a CMK's availability \(such as [ScheduleKeyDeletion](https://docs.aws.amazon.com/kms/latest/APIReference/API_ScheduleKeyDeletion.html), [CancelKeyDeletion](https://docs.aws.amazon.com/kms/latest/APIReference/API_CancelKeyDeletion.html), [DisableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html), [EnableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKey.html), [ImportKeyMaterial](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html), and [DeleteImportedKeyMaterial](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteImportedKeyMaterial.html)\) or change a CMK's access control \(such as [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) and [RevokeGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html)\)\.
+ Other AWS KMS metrics \(such as the amount of time remaining until your [imported key material](importing-keys.md) expires\) and events \(such as the expiration of imported key material or the deletion or key rotation of a CMK\)\.

## Monitoring tools<a name="monitoring-tools"></a>

AWS provides various tools that you can use to monitor your CMKs\. You can configure some of these tools to do the monitoring for you, while some of the tools require manual intervention\. We recommend that you automate monitoring tasks as much as possible\.

### Automated monitoring tools<a name="monitoring-tools-automated"></a>

You can use the following automated monitoring tools to watch your CMKs and report when something has changed\.
+ **AWS CloudTrail Log Monitoring** – Share log files between accounts, monitor CloudTrail log files in real time by sending them to CloudWatch Logs, write log processing applications with the [CloudTrail Processing Library](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/use-the-cloudtrail-processing-library.html), and validate that your log files have not changed after delivery by CloudTrail\. For more information, see [Working with CloudTrail Log Files](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-working-with-log-files.html) in the *AWS CloudTrail User Guide*\.
+ **Amazon CloudWatch Alarms** – Watch a single metric over a time period that you specify, and perform one or more actions based on the value of the metric relative to a given threshold over a number of time periods\. The action is a notification sent to an Amazon Simple Notification Service \(Amazon SNS\) topic or Amazon EC2 Auto Scaling policy\. CloudWatch alarms do not invoke actions simply because they are in a particular state; the state must have changed and been maintained for a specified number of periods\. For more information, see [Monitoring with Amazon CloudWatch](monitoring-cloudwatch.md)\.
+ **Amazon CloudWatch Events** – Match events and route them to one or more target functions or streams to capture state information and, if necessary, make changes or take corrective action\. For more information, see [AWS KMS events](monitoring-cloudwatch.md#kms-events) and the [Amazon CloudWatch Events User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/)\.
+ **Amazon CloudWatch Logs** – Monitor, store, and access your log files from AWS CloudTrail or other sources\. For more information, see the [Amazon CloudWatch Logs User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/)\.

### Manual monitoring tools<a name="monitoring-tools-manual"></a>

Another important part of monitoring CMKs involves manually monitoring those items that the CloudWatch alarms and events don't cover\. The AWS KMS, CloudWatch, AWS Trusted Advisor, and other AWS dashboards provide an at\-a\-glance view of the state of your AWS environment\.

You can [customize](viewing-keys-console.md) the **AWS Managed Keys** and **Customer Managed Keys** pages of the [AWS KMS console](https://console.aws.amazon.com/kms) to display the following information about each CMK: 
+ Key ID
+ Status
+ Creation date
+ Expiration date \(for CMKs with [imported key material](importing-keys.md)\)
+ Origin
+ Custom key store ID \(for CMKs in [custom key stores](custom-key-store-overview.md)\)

The [CloudWatch console dashboard](https://console.aws.amazon.com/cloudwatch/home) shows the following:
+ Current alarms and status
+ Graphs of alarms and resources
+ Service health status

In addition, you can use CloudWatch to do the following:
+ Create [customized dashboards](https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/CloudWatch_Dashboards.html) to monitor the services you care about
+ Graph metric data to troubleshoot issues and discover trends
+ Search and browse all your AWS resource metrics
+ Create and edit alarms to be notified of problems

AWS Trusted Advisor can help you monitor your AWS resources to improve performance, reliability, security, and cost effectiveness\. Four Trusted Advisor checks are available to all users; more than 50 checks are available to users with a Business or Enterprise support plan\. For more information, see [AWS Trusted Advisor](https://aws.amazon.com/premiumsupport/trustedadvisor/)\.