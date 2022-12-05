# Deleting multi\-Region keys<a name="multi-region-keys-delete"></a>

When you are no longer using a multi\-Region primary key or replica key, you can schedule its deletion\. 

Although deleting KMS keys should always be done with caution, deleting a replica of a multi\-Region key is less risky, provided that the primary key still exists in AWS KMS\. If you delete a replica key from its Region, but discover ciphertext that was encrypted under the deleted key, you can decrypt that ciphertext with any related multi\-Region key\. You can also recreate the replica key by replicating the primary key again into the replica key Region\. 

However, deleting a primary key and all of its replica key is a very dangerous operation — equivalent to deleting a single\-Region key\. 

**Warning**  
Deleting a KMS key is destructive and potentially dangerous\. You should proceed only when you are sure that you don't need to use the KMS key anymore and won't need to use it in the future\. If you are not sure, you should [disable the KMS key](enabling-keys.md) instead of deleting it\.

To delete a primary key, you must first delete all of its replica keys\. If you must delete a primary key from a particular Region without deleting its replica keys, change the primary key to a replica key by [updating the primary Region](multi-region-keys-manage.md#multi-region-update)\.

Before you schedule the deletion of any KMS key, review the cautions in the [Deleting AWS KMS keys](deleting-keys.md) topic, and the topics that explain how to [determine past use of a KMS key](deleting-keys-determining-usage.md) and how to [set a CloudWatch alarm](deleting-keys-creating-cloudwatch-alarm.md) that alerts you to use of the KMS key during the waiting period\. Before deleting the primary key of an asymmetric multi\-Region key, review the [Deleting asymmetric keys](deleting-keys.md#deleting-asymmetric-cmks) topic\. 

**Topics**
+ [Permissions for deleting multi\-Region keys](#multi-region-delete-permissions)
+ [How to delete a replica key](#replica-delete)
+ [How to delete a primary key](#primary-delete)

## Permissions for deleting multi\-Region keys<a name="multi-region-delete-permissions"></a>

To schedule the deletion of a multi\-Region key, you need only the following permission\.
+ [kms:ScheduleKeyDeletion](https://docs.aws.amazon.com/kms/latest/APIReference/API_ScheduleKeyDeletion.html) — to schedule the deletion of the multi\-Region key and set its waiting period\.

We also strongly recommend that you have the following related permissions\.
+ [kms:CancelKeyDeletion](https://docs.aws.amazon.com/kms/latest/APIReference/API_CancelKeyDeletion.html) — to cancel the scheduled deletion of the multi\-Region key\.
+ [kms:DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) — to view the key state of the multi\-Region key and the list of related multi\-Region keys\.
+ [kms:DisableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKey.html) — to give you the option to disable a multi\-Region key instead of deleting it\.
+ [kms:EnableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKey.html) — to restore the functionality of a multi\-Region key after canceling its deletion\.

You might also include permission to replicate the primary key and change the primary key\.
+ [kms:ReplicateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReplicateKey.html)
+ [kms:UpdateReplicaRegion](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateReplicaRegion.html)

You can include these permissions in an IAM policy, but it's a best practice to put them in a key policy where they apply only to the KMS key that you need to manage\.

## How to delete a replica key<a name="replica-delete"></a>

You can use the AWS KMS console or the AWS KMS API to delete a replica key\. You can delete a replica key at any time\. It doesn't depend on the key state of any other KMS key\.

If you mistakenly delete a replica key, you can recreate it by replicating the same primary key in the same Region\. The new replica key you create will have the same [shared properties](multi-region-keys-overview.md#mrk-sync-properties) as the original replica key\.

The procedure for deleting a multi\-Region replica key is the same as deleting a single\-Region key\. 

![\[Deleting a multi-Region replica key\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/multi-region-keys-delete-replica.png)

1. Schedule deletion of the replica key\. Select a waiting period of 7\-30 days\. The default waiting period is 30 days\.

1. During the waiting period, the [key state](key-state.md) of the replica key changes to `Pending deletion` \(`PendingDeletion`\) and you cannot use it in cryptographic operations\. 

1. You can cancel the scheduled deletion of the replica key at any point in the waiting period\. The key state changes to `Disabled`, but you can [re\-enable](enabling-keys.md) the KMS key\.

1. When the waiting period expires, AWS KMS deletes the replica key\.

You can view a record of your actions in your AWS CloudTrail log\. AWS KMS records the operations that [schedule deletion of the KMS key](ct-schedule-key-deletion.md) and the action that [deletes the KMS key](ct-delete-key.md)\.

### Deleting a replica key \(console\)<a name="replica-delete-console"></a>

To schedule the deletion of a multi\-Region replica key, use the [same procedure](deleting-keys-scheduling-key-deletion.md#deleting-keys-scheduling-key-deletion-console) you use to schedule the deletion of a single\-Region key\.

Because related replica keys are in different AWS Regions, you cannot schedule the deletion of more than one replica key at a time\. To delete all related replica keys, use a pattern like the following one\.

**To schedule deletion of all related replica keys**

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Use the Region selector in the upper\-right corner to choose the Region of the multi\-Region primary key\.

1. Choose its alias or key ID of the primary key\.

1. Choose the **Regionality** tab\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-regionality-tab-sm.png)

1. In the **Related multi\-Region keys** section, choose the key ARN of a replica key\. 

   This action opens the key details page of the replica key in a new browser tab\. The console is set to the replica key Region\.

1. From the **Key actions** menu, choose **Schedule key deletion**\. 

   This action starts the process of scheduling deletion of the key\. Complete the schedule key deletion process\. For details, see [Scheduling and canceling key deletion \(console\)](deleting-keys-scheduling-key-deletion.md#deleting-keys-scheduling-key-deletion-console)\.

1. Return to the browser tab that displays the **Regionality** tab of the primary key\. \(You might need to refresh the page to see the updated status of the replica keys\.\) Choose the key ARN of another replica key and repeat the process of scheduling deletion of the replica key\.

### Deleting a replica key \(AWS KMS API\)<a name="replica-delete-api"></a>

To schedule the deletion of a multi\-Region replica key, use the [ScheduleKeyDeletion](https://docs.aws.amazon.com/kms/latest/APIReference/API_ScheduleKeyDeletion.html) operation\. To specify the KMS key, use its [key ID](concepts.md#key-id-key-id) or [key ARN](concepts.md#key-id-key-ARN)\. When working with multi\-Region keys, you can reduce the incidence of errors by using the key ARN with its explicit Region value\.

For example, this command deletes a replica key from the us\-west\-2 \(US West \(Oregon\)\) Region\. Because the command doesn't specify a waiting period, the waiting period is set to the default of 30 days\. 

```
$ aws kms schedule-key-deletion \
    --region us-west-2 \
    --key-id arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab
```

When the command succeeds, it returns the key ARN \(`KeyId`\), the waiting period \(`PendingWindowInDays`\), the deletion date \(`DeletionDate`\), and the current key state \(`KeyState`\), which is expected to be `PendingDeletion`\. 

When deleting a multi\-Region replica key, be sure to verify that the key ID and Region values in the key ARN are the ones that you expect\.

```
{
    "KeyId": "arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
    "DeletionDate": 1599523200.0,
    "KeyState": "PendingDeletion",
    "PendingWindowInDays": 30
}
```

To delete all replicas of a multi\-Region primary key programmatically, create a list of the Regions that contain replica keys\. Then, for each Region in the list, call the `ScheduleKeyDeletion` operation, as shown above\.

Unlike a single\-Region key that is permanently deleted, you can restore a replica key by [replicating the primary key](multi-region-keys-replicate.md) into the Region where the deleted replica key was located\. 

To check the status of the replica key and view the primary key and replica keys of a multi\-Region key, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\.

## How to delete a primary key<a name="primary-delete"></a>

You can schedule the deletion of a multi\-Region primary key at any time\. However, AWS KMS will not delete a multi\-Region primary key that has replica keys, even if they are scheduled for deletion\. 

To delete a primary key, you must schedule the deletion all of its replica keys, and then wait for the replica keys to be deleted\. The required waiting period for deleting a primary key begins when the last of its replica keys is deleted\. If you must delete a primary key from a particular Region without deleting its replica keys, change the primary key to a replica key by [updating the primary Region](multi-region-keys-manage.md#multi-region-update)\.

If a primary key has no replica keys, the process is identical to [deleting a replica key](#replica-delete-console) or [deleting any regional KMS key](deleting-keys.md)\.

While a primary key is scheduled for deletion, you cannot use it in cryptographic operations and you cannot replicate it\. However, unless they are also scheduled for deletion, its replica keys are unaffected\. 

You can use the AWS KMS console or the AWS KMS API to schedule the deletion of primary and replica keys\. You can schedule deletion of the primary key before, after, or at the same time that you schedule deletion of the replica keys\. The process might look something like the following one\.

1. Schedule the deletion of the primary key\. Select a waiting period of 7\-30 days\. The default waiting period is 30 days\. However, the waiting period for the primary key does not begin until all replica keys are deleted\. 

   If any replica keys still exist, the [key state](key-state.md) of the primary key changes to `Pending replica deletion` \(`PendingReplicaDeletion`\)\. Otherwise, it changes to `Pending deletion` \(`PendingDeletion`\)\. In either case, you cannot use the primary key in cryptographic operations and you cannot replicate it\. 

   Scheduling the deletion of a primary key doesn't affect the replica keys\. Their key state remains enabled and you can use them in cryptographic operations\. If the replica keys are not deleted, the `Pending replica deletion` state of the primary key can persist indefinitely\.

   ```
   KMS key:                    Key state:
   Primary (us-east-1)           Pending replica deletion (waiting period 30 days -- not started)
   Replica (us-west-2)           Enabled
   Replica (eu-west-1)           Enabled
   Replica (ap-southeast-2)      Enabled
   ```  
![\[Scheduling deletion of a multi-Region primary key\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/multi-region-keys-delete-primary-1.png)

1. Schedule deletion of each replica key\. Select a waiting period of 7\-30 days\. The default waiting period is 30 days\. You can delete multiple replica keys at the same time\. Their waiting periods run concurrently\. During the waiting period, the [key state](key-state.md) of the replica keys changes to `Pending deletion` \(`PendingDeletion`\) and you cannot use these KMS keys in cryptographic operations\. 

   For example, if you have a three replica keys, you can schedule deletion of all three at the same time\. They can have the same or different waiting periods\. Notice that the waiting period on the primary key has not yet begun\. Its key state is `PendingReplicaDeletion` because it has existing replica keys\.

   ```
   KMS key:                    Key state:
   Primary key (us-east-1)       Pending replica deletion (waiting period 30 days -- not started)
   Replica (us-west-2)           Pending deletion (7 days)
   Replica (eu-west-1)           Pending deletion (7 days)
   Replica (ap-southeast-2)      Pending deletion (30 days)
   ```

1. You can cancel the scheduled deletion of the primary key or any replica key until it is deleted\. The key state changes to `Disabled`, but you can [re\-enable](enabling-keys.md) the KMS key\.

1. When the waiting period of the last replica key expires, AWS KMS deletes the last replica key\. The key state of the primary key changes from `Pending replica deletion` \(`PendingReplicaDeletion`\) to `Pending deletion` \(`PendingDeletion`\) and the 7\-30 day waiting period for the primary key begins\.

   ```
   KMS key:                    Key state:
   Primary key (us-east-1)       Pending deletion (waiting period 30 days)
   ```  
![\[Deleting all replica keys of a multi-Region key\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/multi-region-keys-delete-primary-2.png)

1. When its waiting period expires, AWS KMS deletes the primary key\.

**The minimum time to delete a primary key with replicas is 14 days\.**

If you schedule key deletion of the primary key and all replica keys with a waiting period of 7 days, the replica keys are deleted after 7 days\. The primary key is deleted on the 14th day\.
+ Day 1: Schedule the deletion of the primary and replica keys with the minimum waiting period of 7 days\. The 7\-day deletion waiting periods for the replica keys start\. The deletion waiting period for the primary key does not yet start\.
+ Day 7: The deletion waiting periods for the replica keys end\. AWS KMS deletes all replica keys\. When the last replica key is deleted, the 7\-day deletion waiting period for the primary key starts\.
+ Day 14: The deletion waiting period for the primary key ends\. AWS KMS deletes the primary key\.

You can view a record of your actions in your AWS CloudTrail log\. AWS KMS records the operations that [schedule deletion of each KMS key](ct-schedule-key-deletion.md) and the action that [deletes the KMS key](ct-delete-key.md)\.

### Deleting a primary key \(console\)<a name="primary-delete-console"></a>

To delete a multi\-Region primary key, use the following procedure\.

**To schedule key deletion**

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Select the check box next to the primary key that you want to delete\. You can also select one or more KMS keys, including the replicas of this primary key\.

1. Choose **Key actions**, **Schedule key deletion**\.

1. Read and consider the warning, and the information about canceling the deletion during the waiting period\. If you decide to cancel the deletion, choose **Cancel**\.

1. For **Waiting period \(in days\)**, enter a number of days between 7 and 30\. If you selected multiple KMS keys, the waiting period that you choose applies to all selected KMS keys\. The waiting period for replica keys runs concurrently, but the waiting period for the primary key does not begin until AWS KMS deletes the last of the replica keys\.

1. Select the check box next to **Confirm that you want to delete this key in *<number of days>* days**\.

1. Choose **Schedule deletion**\.

To check the deletion status of your KMS keys, on the [detail page](viewing-keys-console.md#viewing-console-details) for the primary key, see the **General configuration** section\. The key state appears in the **Status** field\. When the key state of the primary key changes to `Pending deletion` the **Scheduled deletion date** is displayed\.

You can also check the key state \(**Status**\) of all primary and replica keys on the **Regionality** tab of the detail page for any multi\-Region key\. For details, see [Viewing multi\-Region keys](multi-region-keys-view.md)\.

### Deleting a primary key \(AWS KMS API\)<a name="primary-delete-api"></a>

To delete a multi\-Region replica key, use the [ScheduleKeyDeletion](https://docs.aws.amazon.com/kms/latest/APIReference/API_ScheduleKeyDeletion.html) operation\. To specify the KMS key, use its [key ID](concepts.md#key-id-key-id) or [key ARN](concepts.md#key-id-key-ARN)\. When working with multi\-Region keys, you can reduce the incidence of errors by using the key ARN with its explicit Region value\.

For example, this command deletes a primary key from the us\-east\-1 \(US East \(N\. Virginia\)\) Region\. Because the command doesn't specify a waiting period, the waiting period is set to the default of 30 days\. 

```
$ aws kms schedule-key-deletion \
    --key-id arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab
```

When the command succeeds, it returns the key ARN, the resulting key state, and the waiting period \(`PendingWindowInDays`\)\. 

If the primary key has no replicas, the key state of the primary key is `PendingDeletion` and the output includes the `DeletionDate` field\. If any replica keys remain, the key state of the primary key is `PendingReplicaDeletion` and `DeletionDate` is omitted because it is uncertain\. Even if the replica keys are also scheduled for deletion, you might cancel the scheduled deletion\.

When deleting a multi\-Region primary key, be sure to verify that the key ID and Region values in the key ARN are the ones that you expect\.

```
{
    "KeyId": "arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
    "KeyState": "PendingReplicaDeletion",
    "PendingWindowInDays": 30
}
```

To check the deletion status of your KMS keys, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation on the primary key or any remaining replica keys\. The waiting period clock for the primary key does not start until the last replica is deleted and the key state changes to `PendingDeletion`\. 

To calculate the expected deletion date of the primary key, loop through the replica key ARNs in the response, run `DescribeKey` on each one, get the latest `DeletionDate` value, and then add the `PendingDeletionWindowInDays` value for the primary key\. The waiting periods for the replica keys run concurrently\.

In the following example, the KMS key is a multi\-Region primary key with existing replica keys\. Because the key state is `PendingReplicaDeletion`, the response includes the waiting period \(`PendingWindowInDays`\), but not the `DeletionDate`\. The actual deletion date of the primary key depends on when the replica keys are deleted\.

```
$ aws kms describe-key \
    --key-id arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab

{
    "KeyMetadata": {
        "AWSAccountId": "111122223333",
        "KeyId": "mrk-1234abcd12ab34cd56ef1234567890ab",
        "Arn": "arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
        "CreationDate": 1597902361.481,
        "Enabled": false,
        "Description": "",
        "KeySpec": "SYMMETRIC_DEFAULT",        
        "KeyState": "PendingReplicaDeletion",
        "KeyUsage": "ENCRYPT_DECRYPT",
        "Origin": "AWS_KMS",
        "KeyManager": "CUSTOMER",
        "CustomerMasterKeySpec": "SYMMETRIC_DEFAULT",
        "EncryptionAlgorithms": [
            "SYMMETRIC_DEFAULT"
        ],
        "MultiRegion": true,
        "MultiRegionConfiguration": {
            "MultiRegionKeyType": "PRIMARY",
            "PrimaryKey": {
                "Arn": "arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
                "Region": "us-east-1"
            },
            "ReplicaKeys": [
                {
                    "Arn": "arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
                    "Region": "us-west-2"
                },
{
                    "Arn": "arn:aws:kms:eu-west-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
                    "Region": "eu-west-1"
                },
                {
                    "Arn": "arn:aws:kms:ap-southeast-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
                    "Region": "ap-southeast-2"
                }
            ]
        },
        "PendingDeletionWindowInDays": 30
    }
}
```

When all replicas are deleted, the `DescribeKey` output shows the remaining primary key with a key state of `PendingDeletion`\. While the key state is `PendingDeletion`, the `DeletionDate` field appears instead of the `PendingWindowInDays` field\.

```
$ aws kms describe-key \
    --key-id arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab

{
    "KeyMetadata": {
        "AWSAccountId": "111122223333",
        "KeyId": "mrk-1234abcd12ab34cd56ef1234567890ab",
        "Arn": "",
        "CreationDate": 1597902361.481,
        "Enabled": false,
        "Description": "",
        "KeySpec": "SYMMETRIC_DEFAULT",
        "KeyState": "PendingDeletion",
        "KeyUsage": "ENCRYPT_DECRYPT",
        "DeletionDate": 1597968000.0,
        "Origin": "AWS_KMS",
        "KeyManager": "CUSTOMER",
        "CustomerMasterKeySpec": "SYMMETRIC_DEFAULT",
        "EncryptionAlgorithms": [
            "SYMMETRIC_DEFAULT"
        ],
        "MultiRegion": true,
        "MultiRegionConfiguration": {
            "MultiRegionKeyType": "PRIMARY",
            "PrimaryKey": {
                "Arn": "arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
                "Region": "us-east-1"
            },
            "ReplicaKeys": []
        }
    }
}
```