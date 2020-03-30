# Creating an Amazon CloudWatch alarm to detect usage of a customer master key that is pending deletion<a name="deleting-keys-creating-cloudwatch-alarm"></a>

You can combine the features of AWS CloudTrail, Amazon CloudWatch Logs, and Amazon Simple Notification Service \(Amazon SNS\) that notify you when someone in your account tries to use a CMK that is pending deletion in a cryptographic operation\. If you receive this notification, you might want to cancel deletion of the CMK and reconsider your decision to delete it\.

The following procedures explain how to receive a notification whenever an AWS KMS API request that results in the "`Key ARN is pending deletion`" error message is written to your CloudTrail log files\. This error message indicates that a person or application tried to use the CMK in a cryptographic operation \(`Encrypt`, `Decrypt`, `GenerateDataKey`, `GenerateDataKeyWithoutPlaintext`, and `ReEncrypt`\)\. Because the notification is linked to the error message, it is not triggered when you use API operations that are permitted on CMKs that are pending deletion, such as `ListKeys`, `CancelKeyDeletion`, and `PutKeyPolicy`\. To see a list of the AWS KMS API operations that return this error message, see [Key state: Effect on your CMK](key-state.md)\.

The notification email that you receive does not list the CMK or the cryptographic operation\. You can find that information in [your CloudTrail log](logging-using-cloudtrail.md)\. Instead, the email reports that the alarm state changed from **OK** to **Alarm**\. For more information about CloudWatch Alarms and state changes, see [Creating Amazon CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html) in the *Amazon CloudWatch User Guide*\.

**Warning**  
This Amazon CloudWatch alarm cannot detect use of the public key of an asymmetric CMK outside of AWS KMS\. For details about the special risks of deleting asymmetric CMKs used for public key cryptography, including creating ciphertexts that cannot be decrypted, see [Deleting asymmetric CMKs](deleting-keys.md#deleting-asymmetric-cmks)\.

**Topics**
+ [Requirements for a CloudWatch alarm](#cloudwatch-alarm-prerequisites)
+ [Create the CloudWatch alarm](#deleting-keys-cloudwatch-create-alarm)

## Requirements for a CloudWatch alarm<a name="cloudwatch-alarm-prerequisites"></a>

Before you create a CloudWatch alarm, you must create an AWS CloudTrail trail and configure CloudTrail to deliver CloudTrail log files to Amazon CloudWatch Logs\.

1. [Create a CloudTrail trail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail.html)\. 

   CloudTrail is automatically enabled on your AWS account when you create the account\. However, for an ongoing record of events in your account, including events for AWS KMS, create a trail\. 

1. [Configure CloudTrail to deliver your log files CloudWatch Logs](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/send-cloudtrail-events-to-cloudwatch-logs.html)\.

   Configure delivery of your CloudTrail log files to CloudWatch Logs\. This allows CloudWatch Logs to monitor the logs for AWS KMS API requests that attempt to use a CMK that is pending deletion\.

## Create the CloudWatch alarm<a name="deleting-keys-cloudwatch-create-alarm"></a>

To receive a notification when AWS KMS API requests attempt to use a CMK that is pending deletion in a cryptographic operation, create a CloudWatch alarm and configure notifications\.

**To create a CloudWatch alarm that monitors attempted usage of a KMS CMK that is pending deletion**

1. Sign in to the AWS Management Console and open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. Use the Region selector on the upper right to choose the AWS Region you want to monitor\.

1. In the left navigation pane, choose **Logs**\.

1. In the list of **Log Groups**, choose the option button next to your log group\. Then choose **Create Metric Filter**\.

1. For **Filter Pattern**, type or paste the following:

   ```
   { $.eventSource = kms* && $.errorMessage = "* is pending deletion."}
   ```

   Choose **Assign Metric**\.

1. On the **Create Metric Filter and Assign a Metric** page, do the following:

   1. For **Metric Namespace**, type **CloudTrailLogMetrics**\.

   1. For **Metric Name**, type **KMSKeyPendingDeletionErrorCount**\.

   1. Choose **Show advanced metric settings** and for **Metric Value**, type **1**, if this is not the current value\.

   1. Choose **Create Filter**\.

1. In the filter box, choose **Create Alarm**\.

1. In the **Create Alarm** window, do the following:

   1. In the **Alarm Threshold** section, for **Name**, type **KMSKeyPendingDeletionErrorAlarm**\. You can also add an optional description\.

   1. Following **Whenever**, for **is**, choose **>=** and then type **1**\.

   1. For **1 out of *n* datapoints**, if necessary, type **1**\.

   1. In the **Additional settings** section, for **Treat missing data as**, choose **good \(not breaching threshold\)**\.

   1. In the **Actions** section, for **Send notification to**, do one of the following:
      + To use a new Amazon SNS topic, choose **New list**, and then type a new topic name, such as **KMSAlert**\. For **Email list**, type at least one email address\. You can type more than one email address by separating them with commas\.
      + To use an existing Amazon SNS topic, choose the name of the topic to use\.

   1. Choose **Create Alarm**\.   
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/cloudwatch-console-create-alarm.png)

1. If you chose to send notifications to an email address, open the email message you receive from no\-reply@sns\.amazonaws\.com with a subject "AWS Notification \- Subscription Confirmation\." Confirm your email address by choosing the **Confirm subscription** link in the email message\.
**Note**  
You will not receive email notifications until after you have confirmed your email address\.

After you complete this procedure, you will receive a notification each time this CloudWatch alarm enters the `ALARM` state\. If you receive a notification for this alarm, it might mean that someone or something still needs to use this CMK\. In that case, you should [cancel deletion of the CMK](deleting-keys.md#deleting-keys-scheduling-key-deletion) to give yourself more time to determine whether you really want to delete it\.