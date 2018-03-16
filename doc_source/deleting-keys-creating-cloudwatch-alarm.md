# Creating an Amazon CloudWatch Alarm to Detect Usage of a Customer Master Key that is Pending Deletion<a name="deleting-keys-creating-cloudwatch-alarm"></a>

You can use a combination of AWS CloudTrail, Amazon CloudWatch Logs, and Amazon Simple Notification Service \(Amazon SNS\) to create an alarm that notifies you of AWS KMS API requests that attempt to use a customer master key \(CMK\) that is pending deletion\. If you receive a notification from such an alarm, you might want to cancel deletion of the CMK to give yourself more time to determine whether you want to delete it\.

The following procedures explain how to receive a notification whenever an AWS KMS API request that results in the "`Key ARN is pending deletion`" error message is written to your CloudTrail log files\. Note that not all AWS KMS API requests that use a CMK that is pending deletion result in this error message\. For example, you can successfully use the `ListKeys`, `CancelKeyDeletion`, and `PutKeyPolicy` APIs \(and others\) with CMK that are pending deletion\. The intent of this CloudWatch alarm is to detect usage that might indicate that a person or application still expects to use the CMK for cryptographic operations \(`Encrypt`, `Decrypt`, `GenerateDataKey`, and others\)\. To see a list of which AWS KMS APIs do and don't result in this error message, go to [How Key State Affects Use of a Customer Master Key](key-state.md)\.

Before you begin these procedures, you must have already turned on CloudTrail in the AWS account and region where you intend to monitor AWS KMS API requests\. For instructions, go to [Creating a Trail for the First Time](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-a-trail-using-the-console-first-time.html) in the *AWS CloudTrail User Guide*\.


+ [Part 1: Configure CloudTrail Log File Delivery to CloudWatch Logs](#deleting-keys-cloudwatch-configure-cloudtrail)
+ [Part 2: Create the CloudWatch Alarm](#deleting-keys-cloudwatch-create-alarm)

## Part 1: Configure CloudTrail Log File Delivery to CloudWatch Logs<a name="deleting-keys-cloudwatch-configure-cloudtrail"></a>

You must configure delivery of your CloudTrail log files to CloudWatch Logs so that CloudWatch Logs can monitor them for AWS KMS API requests that attempt to use a customer master key \(CMK\) that is pending deletion\.

**To configure CloudTrail log file delivery to CloudWatch Logs**

1. Sign in to the AWS Management Console and open the CloudTrail console at [https://console\.aws\.amazon\.com/cloudtrail/](https://console.aws.amazon.com/cloudtrail/)\.

1. On the top navigation bar, choose the AWS region in which you intend to monitor activity\.

1. If necessary, in the left navigation pane, choose **Configuration**\.

1. In the content pane, in the **CloudWatch Logs \(Optional\)** section, choose **Configure**\.

1. If necessary, for **New or existing log group**, type a name for the log group, such as **CloudTrail/DefaultLogGroup**, and then choose **Continue**\.

1. On the **AWS CloudTrail will deliver CloudTrail events associated with API activity in your account to your CloudWatch Logs log group** page, choose **Allow**\.

## Part 2: Create the CloudWatch Alarm<a name="deleting-keys-cloudwatch-create-alarm"></a>

To receive a notification when AWS KMS API requests attempt to use a customer master key \(CMK\) that is pending deletion, you must create a CloudWatch alarm and configure notification\.

**To create a CloudWatch alarm that monitors attempted usage of a KMS CMK that is pending deletion**

1. Sign in to the AWS Management Console and open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. On the top navigation bar, choose the AWS region in which you intend to monitor activity\.

1. In the left navigation pane, choose **Logs**\.

1. In the list of **Log Groups**, select the check box next to the log group that you created previously, such as **CloudTrail/DefaultLogGroup**\. Then choose **Create Metric Filter**\.

1. For **Filter Pattern**, type or paste the following:

   **\{ $\.eventSource = kms\* && $\.errorMessage = "\* is pending deletion\." \}**

   Choose **Assign Metric**\.

1. On the **Create Metric Filter and Assign a Metric** page, do the following:

   1. For **Metric Namespace**, type **CloudTrailLogMetrics**\.

   1. For **Metric Name**, type **KMSKeyPendingDeletionErrorCount**\.

   1. Choose **Show advanced metric settings**, and then if necessary for **Metric Value**, type **1**\.

   1. Choose **Create Filter**\.

1. In the filter box, choose **Create Alarm**\.

1. In the **Create Alarm** window, do the following:

   1. For **Name**, type **KMSKeyPendingDeletionErrorAlarm**\.

   1. Following **Whenever:**, for **is:**, choose **>=** and then type **1**\.

   1. For **for consecutive period\(s\)**, if necessary, type **1**\.

   1. Next to **Send notification to:**, do one of the following:

      + To use a new Amazon SNS topic, choose **New list**, and then type a new topic name\. For **Email list:**, type at least one email address\. You can type more than one email address by separating them with commas\.

      + To use an existing Amazon SNS topic, choose the name of the topic to use\.

   1. Choose **Create Alarm**\.   
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/cloudwatch-console-create-alarm.png)

1. If you chose to send notifications to an email address, open the email message you receive from no\-reply@sns\.amazonaws\.com with a subject "AWS Notification \- Subscription Confirmation\." Confirm your email address by choosing the **Confirm subscription** link in the email message\.
**Note**  
You will not receive email notifications until after you have confirmed your email address\.

After you complete this procedure, you will receive a notification each time this CloudWatch alarm enters the `ALARM` state\. If you receive a notification for this alarm, it might mean that someone or something still needs to use this CMK\. In that case, you should [cancel deletion of the CMK](deleting-keys.md#deleting-keys-scheduling-key-deletion) to give yourself more time to determine whether you really want to delete it\.