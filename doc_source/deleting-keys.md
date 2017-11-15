# Deleting Customer Master Keys<a name="deleting-keys"></a>

Deleting a customer master key \(CMK\) in AWS Key Management Service \(AWS KMS\) is destructive and potentially dangerous\. It deletes the key material and all metadata associated with the CMK, and is irreversible\. After a CMK is deleted you can no longer decrypt the data that was encrypted under that CMK, which means that data becomes unrecoverable\. You should delete a CMK only when you are sure that you don't need to use it anymore\. If you are not sure, consider disabling the CMK instead of deleting it\. You can re\-enable a disabled CMK if you need to use it again later, but you cannot recover a deleted CMK\.

Before deleting a CMK, you might want to know how many ciphertexts were encrypted under that CMK\. AWS KMS does not store this information, and does not store any of the ciphertexts\. To get this information, you must determine on your own the past usage of a CMK\. For some guidance that might help you do this, go to [Determining Past Usage of a Customer Master Key](deleting-keys-determining-usage.md)\.

You might choose to delete a CMK for one or more of the following reasons:

+ To complete the key lifecycle for CMKs that you no longer need

+ To avoid the management overhead and [costs](https://aws.amazon.com/kms/pricing/) associated with maintaining unused CMKs

+ To reduce the number of CMKs that count against your limit


+ [How Deleting Customer Master Keys Works](#deleting-keys-how-it-works)
+ [Scheduling and Canceling Key Deletion](#deleting-keys-scheduling-key-deletion)
+ [Adding Permission to Schedule and Cancel Key Deletion](#deleting-keys-adding-permission)
+ [Creating an Amazon CloudWatch Alarm to Detect Usage of a Customer Master Key that is Pending Deletion](deleting-keys-creating-cloudwatch-alarm.md)
+ [Determining Past Usage of a Customer Master Key](deleting-keys-determining-usage.md)

## How Deleting Customer Master Keys Works<a name="deleting-keys-how-it-works"></a>

Because it is destructive and potentially dangerous to delete a customer master key \(CMK\), AWS KMS enforces a waiting period\. To delete a CMK in AWS KMS you *schedule key deletion*\. You can set the waiting period from a minimum of 7 days up to a maximum of 30 days\. The default waiting period is 30 days\. During the waiting period, the CMK is *pending deletion* which means it can't be used\. After the waiting period ends, AWS KMS deletes the CMK and all AWS KMS data associated with it, including all aliases that point to it\.

When you schedule key deletion, AWS KMS reports the date and time when the waiting period ends\. This date and time is at least the specified number of days from when you scheduled key deletion, but it can be up to 24 hours longer\. For example, when you schedule key deletion and specify a waiting period of 7 days, the end of the waiting period occurs no earlier than 7 days and no more than 8 days from the time of your request\. You can confirm the exact date and time when the waiting period ends in the AWS Management Console, AWS CLI, or AWS KMS API\.

Use the waiting period to ensure that you don't need the CMK now or in the future\. You can configure an Amazon CloudWatch alarm to warn you if a person or application attempts to use the CMK during the waiting period\. To recover the CMK, you can cancel key deletion before the waiting period ends\. After the waiting period ends you cannot cancel key deletion, and AWS KMS deletes the CMK\.

### How Deleting Customer Master Keys Affects AWS Services Integrated With AWS KMS<a name="deleting-keys-how-it-affects-aws-services-integrated-with-kms"></a>

Several AWS services integrate with AWS KMS to protect your data\. Some of these services, such as Amazon EBS and Amazon Redshift, continually modify data in your AWS account while they are in use\. These services protect your data using *envelope encryption*, which means the customer master key \(CMK\) in AWS KMS encrypts a data key, and the data key encrypts your data\. These data keys persist in memory as long as the data they are protecting is actively in use\. For more information about how envelope encryption works, go to [How Envelope Encryption Works with Supported AWS Services](workflow.md)\.

When you schedule a CMK for deletion it becomes unusable\. However, data keys that are actively in use are unaffected\. This means that scheduling a CMK for deletion does not immediately affect resources and data that are actively in use\.

For example, consider this scenario:

1. You create an encrypted EBS volume, at which time Amazon EBS requests a unique data key encrypted with the CMK that you specified when creating the volume\.

1. AWS KMS creates a new data key, encrypts it with the specified CMK, and then sends the encrypted data key to Amazon EBS to store with the volume\.

1. You attach the EBS volume to an EC2 instance, at which time Amazon EC2 calls the AWS KMS `Decrypt` API to decrypt the EBS volume's encrypted data key\. AWS KMS sends the decrypted \(plaintext\) data key to Amazon EC2\.

1. Amazon EC2 uses the plaintext data key in hypervisor memory to encrypt disk I/O to the EBS volume\. The data key persists in memory as long as the EBS volume is attached to the EC2 instance\.

1. You schedule the CMK for deletion\. This has no immediate effect on the EC2 instance or the EBS volume, because Amazon EC2 is using the plaintext data key—not the CMK—to encrypt the EBS volume\.

1. The key deletion waiting period ends, and AWS KMS deletes the CMK\. This has no immediate effect on the EC2 instance or the EBS volume, because Amazon EC2 is using the plaintext data key—not the CMK—to encrypt the EBS volume\.

However, the next time the encrypted EBS volume is attached to an EC2 instance, the attachment fails because at that time Amazon EC2 calls the AWS KMS `Decrypt` API \(step 3 in the preceding scenario\) but the CMK needed for decryption is unusable \(it is pending deletion or deleted\)\.

## Scheduling and Canceling Key Deletion<a name="deleting-keys-scheduling-key-deletion"></a>

The following procedures describe how to schedule key deletion and cancel key deletion in AWS Key Management Service \(AWS KMS\) using the AWS Management Console, the AWS CLI, and the AWS SDK for Java\.

**Warning**  
Deleting a customer master key \(CMK\) in AWS KMS is destructive and potentially dangerous\. You should proceed only when you are sure that you don't need to use the CMK anymore and won't need to use it in the future\. If you are not sure, you should disable the CMK instead of deleting it\.

Before you can delete a CMK, you must have permission to do so\. If you rely on the key policy alone to specify AWS KMS permissions, you might need to add additional permissions before you can delete the CMK\. For information about adding these permissions, go to [Adding Permission to Schedule and Cancel Key Deletion](#deleting-keys-adding-permission)\.

Ways to schedule and cancel key deletion

### Scheduling and Canceling Key Deletion \(AWS Management Console\)<a name="deleting-keys-scheduling-key-deletion-console"></a>

You can schedule and cancel key deletion in the AWS Management Console\.

**To schedule key deletion**

1. Sign in to the AWS Management Console and open the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Select the check box next to the CMK that you want to delete\.

1. Choose **Key Actions**, **Schedule key deletion**\.

1. For **Waiting period \(in days\)**, type a number of days between 7 and 30\. Choose **Schedule deletion**\.

The CMK status changes to **Pending Deletion**\.

**To cancel key deletion**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Select the check box next to the CMK that you want to recover\.

1. Choose **Key Actions**, **Cancel key deletion**\.

The CMK status changes from **Pending Deletion** to **Disabled**\. To use the CMK, you must enable it\.

### Scheduling and Canceling Key Deletion \(AWS CLI\)<a name="deleting-keys-scheduling-key-deletion-cli"></a>

Use the [http://docs.aws.amazon.com/cli/latest/reference/kms/schedule-key-deletion.html](http://docs.aws.amazon.com/cli/latest/reference/kms/schedule-key-deletion.html) command to schedule key deletion from the AWS CLI as shown in the following example\.

```
$ aws kms schedule-key-deletion --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --pending-window-in-days 10
```

When used successfully, the AWS CLI returns output like the output shown in the following example:

```
{
    "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
    "DeletionDate": 1442102400.0
}
```

Use the [http://docs.aws.amazon.com/cli/latest/reference/kms/cancel-key-deletion.html](http://docs.aws.amazon.com/cli/latest/reference/kms/cancel-key-deletion.html) command to cancel key deletion from the AWS CLI as shown in the following example\.

```
$ aws kms cancel-key-deletion --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
```

When used successfully, the AWS CLI returns output like the output shown in the following example:

```
{
    "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
}
```

The status of the CMK changes from **Pending Deletion** to **Disabled**\. To use the CMK, you must enable it\.

### Scheduling and Canceling Key Deletion \(AWS SDK for Java\)<a name="deleting-keys-scheduling-key-deletion-java"></a>

The following example demonstrates how to schedule a CMK for deletion with the AWS SDK for Java\. This example requires that you previously instantiated an `AWSKMSClient` as `kms`\.

```
String KeyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

int PendingWindowInDays = 10;

ScheduleKeyDeletionRequest scheduleKeyDeletionRequest =
new ScheduleKeyDeletionRequest().withKeyId(KeyId).withPendingWindowInDays(PendingWindowInDays);
kms.scheduleKeyDeletion(scheduleKeyDeletionRequest);
```

The following example demonstrates how to cancel key deletion with the AWS SDK for Java\. This example requires that you previously instantiated an `AWSKMSClient` as `kms`\.

```
String KeyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

CancelKeyDeletionRequest cancelKeyDeletionRequest =
new CancelKeyDeletionRequest().withKeyId(KeyId);
kms.cancelKeyDeletion(cancelKeyDeletionRequest);
```

The status of the CMK changes from **Pending Deletion** to **Disabled**\. To use the CMK, you must enable it\.

## Adding Permission to Schedule and Cancel Key Deletion<a name="deleting-keys-adding-permission"></a>

If you use IAM policies to allow AWS KMS permissions, all IAM users and roles that have AWS administrator access \(`"Action": "*"`\) or AWS KMS full access \(`"Action": "kms:*"`\) are already allowed to schedule and cancel key deletion for AWS KMS CMKs\. If you rely on the key policy alone to allow AWS KMS permissions, you might need to add additional permissions to allow your IAM users and roles to delete CMKs\. To add those permissions, see the following steps\. 

The following procedures describe how to add permissions to a key policy using the AWS Management Console or the AWS CLI\.

Ways to add permission to schedule and cancel key deletion

### Adding Permission to Schedule and Cancel Key Deletion \(AWS Management Console\)<a name="deleting-keys-adding-permission-console"></a>

You can use the AWS Management Console to add permissions for scheduling and canceling key deletion\.

**To add permission to schedule and cancel key deletion**

1. Sign in to the AWS Management Console and open the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Choose the alias of the CMK whose permissions you want to change\.

1. In the **Key Policy** section, under **Key Deletion**, select **Allow key administrators to delete this key** and then choose **Save Changes**\.
**Note**  
If you do not see the **Allow key administrators to delete this key** option, this likely means that you have previously modified this key policy using the AWS KMS API\. In this case you must update the key policy document manually\. Add the `kms:ScheduleKeyDeletion` and `kms:CancelKeyDeletion` permissions to the key administrators statement \(`"Sid": "Allow access for Key Administrators"`\) in the key policy, and then choose **Save Changes**\.   

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/kms-console-add-permissions-schedule-cancel-key-deletion.png)![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/)

### Adding Permission to Schedule and Cancel Key Deletion \(AWS CLI\)<a name="deleting-keys-adding-permission-cli"></a>

You can use the AWS Command Line Interface to add permissions for scheduling and canceling key deletion\.

**To add permission to schedule and cancel key deletion**

1. Use the [http://docs.aws.amazon.com/cli/latest/reference/kms/get-key-policy.html](http://docs.aws.amazon.com/cli/latest/reference/kms/get-key-policy.html) command to retrieve the existing key policy, and then save the policy document to a file\.

1. Open the policy document in your preferred text editor, add the `kms:ScheduleKeyDeletion` and `kms:CancelKeyDeletion` permissions to the policy statement that gives permissions to the key administrators \(for example, the policy statement with `"Sid": "Allow access for Key Administrators"`\), and save the file\. The following example shows a policy statement with these two permissions:

   ```
   {
     "Sid": "Allow access for Key Administrators",
     "Effect": "Allow",
     "Principal": {"AWS": "arn:aws:iam::111122223333:user/KMSKeyAdmin"},
     "Action": [
       "kms:Create*",
       "kms:Describe*",
       "kms:Enable*",
       "kms:List*",
       "kms:Put*",
       "kms:Update*",
       "kms:Revoke*",
       "kms:Disable*",
       "kms:Get*",
       "kms:Delete*",
       "kms:ScheduleKeyDeletion",
       "kms:CancelKeyDeletion"
     ],
     "Resource": "*"
   }
   ```

1. Use the [http://docs.aws.amazon.com/cli/latest/reference/kms/put-key-policy.html](http://docs.aws.amazon.com/cli/latest/reference/kms/put-key-policy.html) command to apply the key policy to the CMK\.