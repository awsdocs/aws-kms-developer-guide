# Scheduling and canceling key deletion<a name="deleting-keys-scheduling-key-deletion"></a>

The following procedures describe how to schedule key deletion and cancel key deletion of single\-Region AWS KMS keys \(KMS keys\) in AWS KMS using the AWS Management Console, the AWS CLI, and the AWS SDK for Java\. 

For information about scheduling the deletion of multi\-Region keys, see [Deleting multi\-Region keys](multi-region-keys-delete.md)\.

**Warning**  
Deleting a KMS key is destructive and potentially dangerous\. You should proceed only when you are sure that you don't need to use the KMS key anymore and won't need to use it in the future\. If you are not sure, you should [disable the KMS key](enabling-keys.md) instead of deleting it\.

Before you can delete a KMS key, you must have permission to do so\. For information about giving these permissions to key administrators, see [Controlling access to key deletion](deleting-keys-adding-permission.md)\.

AWS KMS records an entry in your AWS CloudTrail log when you [schedule deletion](ct-schedule-key-deletion.md) of the KMS key and when the [KMS key is actually deleted](ct-delete-key.md)\.

## Scheduling and canceling key deletion \(console\)<a name="deleting-keys-scheduling-key-deletion-console"></a>

In the AWS Management Console, you can schedule and cancel the deletion of multiple KMS keys at one time\.

**To schedule key deletion**

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

   You cannot schedule the deletion of [AWS managed keys](concepts.md#aws-managed-cmk) or [AWS owned keys](concepts.md#aws-owned-cmk)\.

1. Choose the check box next to the KMS key that you want to delete\.

1. Choose **Key actions**, **Schedule key deletion**\.

1. Read and consider the warning, and the information about canceling the deletion during the waiting period\. If you decide to cancel the deletion, at the bottom of the page, choose **Cancel**\.

1. For **Waiting period \(in days\)**, enter a number of days between 7 and 30\. 

1. Review the KMS keys that you are deleting\.

1. Choose the check box next to **Confirm you want to schedule this key for deletion in *<number of days>* days\.**\.

1. Choose **Schedule deletion**\.

The KMS key status changes to **Pending deletion**\.

**To cancel key deletion**

1. Open the AWS KMS console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Choose the check box next to the KMS key that you want to recover\.

1. Choose **Key actions**, **Cancel key deletion**\.

The KMS key status changes from **Pending deletion** to **Disabled**\. To use the KMS key, you must [enable it](enabling-keys.md)\.

## Scheduling and canceling key deletion \(AWS CLI\)<a name="deleting-keys-scheduling-key-deletion-cli"></a>

Use the [https://docs.aws.amazon.com/cli/latest/reference/kms/schedule-key-deletion.html](https://docs.aws.amazon.com/cli/latest/reference/kms/schedule-key-deletion.html) command to schedule key deletion of a [customer managed key](concepts.md#customer-cmk), as shown in the following example\.

You cannot schedule the deletion of an AWS managed key or AWS owned key\.

```
$ aws kms schedule-key-deletion --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --pending-window-in-days 10
```

When used successfully, the AWS CLI returns output like the output shown in the following example:

```
{
    "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
    "DeletionDate": 1598304792.0,
    "KeyState": "PendingDeletion",
    "PendingWindowInDays": 10
}
```

Use the [https://docs.aws.amazon.com/cli/latest/reference/kms/cancel-key-deletion.html](https://docs.aws.amazon.com/cli/latest/reference/kms/cancel-key-deletion.html) command to cancel key deletion from the AWS CLI as shown in the following example\.

```
$ aws kms cancel-key-deletion --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
```

When used successfully, the AWS CLI returns output like the output shown in the following example:

```
{
    "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
}
```

The status of the KMS key changes from **Pending Deletion** to **Disabled**\. To use the KMS key, you must [enable it](enabling-keys.md)\.

## Scheduling and canceling key deletion \(AWS SDK for Java\)<a name="deleting-keys-scheduling-key-deletion-java"></a>

The following example demonstrates how to schedule the deletion of a customer managed key with the AWS SDK for Java\. This example requires that you previously instantiated an `AWSKMSClient` as `kms`\.

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

The status of the KMS key changes from **Pending Deletion** to **Disabled**\. To use the KMS key, you must [enable it](enabling-keys.md)\.