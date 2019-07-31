# Logging AWS KMS API Calls with AWS CloudTrail<a name="logging-using-cloudtrail"></a>

AWS KMS is integrated with AWS CloudTrail, a service that provides a record of actions performed by a user, role, or an AWS service in AWS KMS\. CloudTrail captures all API calls for AWS KMS as events, including calls from the AWS KMS console and from code calls to the AWS KMS APIs\. If you create a trail, you can enable continuous delivery of CloudTrail events to an Amazon S3 bucket, including events for AWS KMS\. If you don't configure a trail, you can still view the most recent events in the CloudTrail console in **Event history**\. Using the information collected by CloudTrail, you can determine the request that was made to AWS KMS, the IP address from which the request was made, who made the request, when it was made, and additional details\.

To learn more about CloudTrail, see the [AWS CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\. To learn about other ways to monitor the use of your CMKs, see [Monitoring Customer Master Keys](monitoring-overview.md)\.

## AWS KMS Information in CloudTrail<a name="kms-info-in-cloudtrail"></a>

CloudTrail is enabled on your AWS account when you create the account\. When activity occurs in AWS KMS, that activity is recorded in a CloudTrail event along with other AWS service events in **Event history**\. You can view, search, and download recent events in your AWS account\. For more information, see [Viewing Events with CloudTrail Event History](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events.html)\. 

For an ongoing record of events in your AWS account, including events for AWS KMS, create a trail\. A trail enables CloudTrail to deliver log files to an Amazon S3 bucket\. By default, when you create a trail in the console, the trail applies to all regions\. The trail logs events from all regions in the AWS partition and delivers the log files to the Amazon S3 bucket that you specify\. Additionally, you can configure other AWS services to further analyze and act upon the event data collected in CloudTrail logs\. For more information, see:
+ [Overview for Creating a Trail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail.html)
+ [CloudTrail Supported Services and Integrations](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-aws-service-specific-topics.html#cloudtrail-aws-service-specific-topics-integrations)
+ [Configuring Amazon SNS Notifications for CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/getting_notifications_top_level.html)
+ [Receiving CloudTrail Log Files from Multiple Regions](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/receive-cloudtrail-log-files-from-multiple-regions.html) and [Receiving CloudTrail Log Files from Multiple Accounts](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-receive-logs-from-multiple-accounts.html)

CloudTrail logs all AWS KMS operations, including read\-only operations, such as `ListAliases` and `GetKeyPolicy`, operations that manage CMKs, such as `CreateKey` and `PutKeyPolicy`, and cryptographic operations, such as `GenerateDataKey`, `Encrypt`, and `Decrypt`\. Every operation generates an entry in the CloudTrail log files\.

Every event or log entry contains information about who generated the request\. The identity information helps you determine the following: 
+ Whether the request was made with root or IAM user credentials\.
+ Whether the request was made with temporary security credentials for a role or federated user\.
+ Whether the request was made by another AWS service\.

For more information, see the [CloudTrail userIdentity Element](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-user-identity.html)\.

## Understanding AWS KMS Log File Entries<a name="understanding-kms-entries"></a>

A *trail* is a configuration that enables delivery of events as log files to an Amazon S3 bucket that you specify\. CloudTrail log files contain one or more log entries\. An *event* represents a single request from any source and includes information about the requested action, the date and time of the action, request parameters, and so on\. CloudTrail log files are not an ordered stack trace of the public API calls, so they do not appear in any specific order\. 

For examples CloudTrail log entries for each API request, see the following topics\.

**Topics**
+ [CreateAlias](ct-createalias.md)
+ [CreateGrant](ct-creategrant.md)
+ [CreateKey](ct-createkey.md)
+ [Decrypt](ct-decrypt.md)
+ [DeleteAlias](ct-deletealias.md)
+ [DescribeKey](ct-describekey.md)
+ [DisableKey](ct-disablekey.md)
+ [EnableKey](ct-enablekey.md)
+ [Encrypt](ct-encrypt.md)
+ [GenerateDataKey](ct-generatedatakey.md)
+ [GenerateDataKeyWithoutPlaintext](ct-generatedatakeyplaintext.md)
+ [GenerateRandom](ct-generaterandom.md)
+ [GetKeyPolicy](ct-getkeypolicy.md)
+ [ListAliases](ct-listaliases.md)
+ [ListGrants](ct-listgrants.md)
+ [ReEncrypt](ct-reencrypt.md)
+ [Amazon EC2 Example One](ct-ec2one.md)
+ [Amazon EC2 Example Two](ct-ec2two.md)