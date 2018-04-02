# Logging AWS KMS API Calls Using AWS CloudTrail<a name="logging-using-cloudtrail"></a>

AWS KMS is integrated with CloudTrail, a service that captures API calls made by or on behalf of AWS KMS in your AWS account and delivers the log files to an Amazon S3 bucket that you specify\. CloudTrail captures API calls from the AWS KMS console or from the AWS KMS API\. Using the information collected by CloudTrail, you can determine what request was made, the source IP address from which the request was made, who made the request, when it was made, and so on\. To learn more about CloudTrail, including how to configure and enable it, see the [AWS CloudTrail User Guide](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\.

When you enable CloudTrail logging in your AWS account, API calls made to AWS KMS actions are tracked in log files\. AWS KMS records are written together with other AWS service records in a log file\. CloudTrail determines when to create and write to a new log file based on a time period and file size\.

CloudTrail logs all of the AWS KMS actions\. For example, calls to the `CreateKey`, `Encrypt`, and `Decrypt` actions generate entries in the CloudTrail log files\.

Every log entry contains information about who generated the request\. The user identity information in the log helps you determine whether the request was made with root or IAM user credentials, with temporary security credentials for a role or federated user, or by another AWS service\. For more information, see [CloudTrail userIdentity Element](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-user-identity.html) in the CloudTrail Event Reference chapter in the *AWS CloudTrail User Guide*\.

You can store your log files in your bucket for as long as you want, but you can also define Amazon S3 lifecycle rules to archive or delete log files automatically\. By default, your log files are encrypted by using Amazon S3 server\-side encryption \(SSE\) with a key managed by the Amazon S3 service\.

You can choose to have CloudTrail publish Amazon SNS notifications when new log files are delivered if you want to take quick action upon log file delivery\. For more information, see [Configuring Amazon SNS Notifications for CloudTrail](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/configure-sns-notifications-for-cloudtrail.html) in the *AWS CloudTrail User Guide*\.

You can also aggregate AWS KMS log files from multiple AWS regions and multiple AWS accounts into a single Amazon S3 bucket\. For more information, see [Receiving CloudTrail Log Files from Multiple Regions](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/receive-cloudtrail-log-files-from-multiple-regions.html) in the *AWS CloudTrail User Guide*\.

CloudTrail log files can contain one or more log entries where each entry is made up of multiple JSON\-formatted events\. A log entry represents a single request from any source and includes information about the requested action, any parameters, the date and time of the action, and so on\. The log entries are not guaranteed to be in any particular order\. That is, they are not an ordered stack trace of the public API calls\. For more information about the fields that make up a log entry, see the [CloudTrail Event Reference](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference.html) in the *AWS CloudTrail User Guide*\.

For examples of what these CloudTrail log entries look like, see the following topics\.

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