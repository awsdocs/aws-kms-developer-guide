# Deleting AWS KMS keys<a name="deleting-keys"></a>

Deleting an AWS KMS key is destructive and potentially dangerous\. It deletes the key material and all metadata associated with the KMS key and is irreversible\. After a KMS key is deleted, you can no longer decrypt the data that was encrypted under that KMS key, which means that data becomes unrecoverable\. You should delete a KMS key only when you are sure that you don't need to use it anymore\. If you are not sure, consider [disabling the KMS key](enabling-keys.md) instead of deleting it\. You can re\-enable a disabled KMS key if you need to use it again later, but you cannot recover a deleted KMS key\.

You can only schedule the deletion of a customer managed key\. You cannot delete AWS managed keys or AWS owned keys\.

Before deleting a KMS key, you might want to know how many ciphertexts were encrypted under that KMS key\. AWS KMS does not store this information and does not store any of the ciphertexts\. To get this information, you must determine past usage of a KMS key\. For help, go to [Determining past usage of a KMS key](deleting-keys-determining-usage.md)\.

AWS KMS never deletes your KMS keys unless you explicitly schedule them for deletion and the mandatory waiting period expires\.

However, you might choose to delete a KMS key for one or more of the following reasons:
+ To complete the key lifecycle for KMS keys that you no longer need
+ To avoid the management overhead and [costs](https://aws.amazon.com/kms/pricing/) associated with maintaining unused KMS keys
+ To reduce the number of KMS keys that count against your [KMS key resource quota](resource-limits.md#kms-keys-limit)

**Note**  
If you [close or delete your AWS account](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/close-account.html), your KMS keys become inaccessible and you are no longer billed for them\. You do not need to schedule deletion of your KMS keys separate from closing the account\.

AWS KMS records an entry in your AWS CloudTrail log when you [schedule deletion](ct-schedule-key-deletion.md) of the KMS key and when the [KMS key is actually deleted](ct-delete-key.md)\. 

For information about deleting multi\-Region primary and replica keys, see [Deleting multi\-Region keys](multi-region-keys-delete.md)\.

**Topics**
+ [About the waiting period](#deleting-keys-how-it-works)
+ [Deleting asymmetric KMS keys](#deleting-asymmetric-cmks)
+ [Deleting multi\-Region keys](#deleting-mrks)
+ [Scheduling and canceling key deletion](#deleting-keys-scheduling-key-deletion)
+ [Adding permission to schedule and cancel key deletion](#deleting-keys-adding-permission)
+ [Creating an Amazon CloudWatch alarm to detect usage of an AWS KMS key pending deletion](deleting-keys-creating-cloudwatch-alarm.md)
+ [Determining past usage of a KMS key](deleting-keys-determining-usage.md)

## About the waiting period<a name="deleting-keys-how-it-works"></a>

Because it is destructive and potentially dangerous to delete a KMS key, AWS KMS requires you to set a waiting period of 7 – 30 days\. The default waiting period is 30 days\.

However, the actual waiting period might be up to 24 hours longer than the one you scheduled\. To get the actual date and time when the KMS key will be deleted, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. Or in the AWS KMS console, on [detail page](viewing-keys-console.md#viewing-details-navigate) for the KMS key, in the **General configuration** section, see the **Scheduled deletion date**\. Be sure to note the time zone\.

During the waiting period, the KMS key status and key state is **Pending deletion**\.
+ A KMS key pending deletion cannot be used in any [cryptographic operations](concepts.md#cryptographic-operations)\. 
+ AWS KMS does not [rotate the key material](rotate-keys.md#rotate-keys-how-it-works) of KMS keys that are pending deletion\.

After the waiting period ends, AWS KMS deletes the KMS key, its aliases, and all related AWS KMS metadata\.

Use the waiting period to ensure that you don't need the KMS key now or in the future\. You can [configure an Amazon CloudWatch alarm](deleting-keys-creating-cloudwatch-alarm.md) to warn you if a person or application attempts to use the KMS key during the waiting period\. To recover the KMS key, you can cancel key deletion before the waiting period ends\. After the waiting period ends you cannot cancel key deletion, and AWS KMS deletes the KMS key\.

## Deleting asymmetric KMS keys<a name="deleting-asymmetric-cmks"></a>

Users [who are authorized](#deleting-keys-adding-permission) can delete symmetric or asymmetric KMS keys\. The procedure to schedule the deletion of these KMS keys is the same for both types of keys\. However, because the [public key of an asymmetric KMS key can be downloaded](download-public-key.md) and used outside of AWS KMS, the operation poses significant additional risks, especially for asymmetric KMS keys used for encryption \(the key usage is `ENCRYPT_DECRYPT`\)\.
+ When you schedule the deletion of a KMS key, the key state of KMS key changes to **Pending deletion**, and the KMS key cannot be used in [cryptographic operations](concepts.md#cryptographic-operations)\. However, scheduling deletion has no effect on public keys outside of AWS KMS\. Users who have the public key can continue to use them to encrypt messages\. They do not receive any notification that the key state is changed\. Unless the deletion is canceled, ciphertext created with the public key cannot be decrypted\.
+ Alarms, logs, and other strategies that detect attempted use of KMS key that is pending deletion cannot detect use of the public key outside of AWS KMS\.
+ When the KMS key is deleted, all AWS KMS actions involving that KMS key fail\. However, users who have the public key can continue to use them to encrypt messages\. These ciphertexts cannot be decrypted\.

If you must delete an asymmetric KMS key with a key usage of `ENCRYPT_DECRYPT`, use your CloudTrail Log entries to determine whether the public key has been downloaded and shared\. If it has, verify that the public key is not being used outside of AWS KMS\. Then, consider [disabling the KMS key](enabling-keys.md) instead of deleting it\.

## Deleting multi\-Region keys<a name="deleting-mrks"></a>

Users [who are authorized](#deleting-keys-adding-permission) can schedule the deletion of multi\-Region primary and replica keys\. However, AWS KMS will not delete a multi\-Region primary key that has replica keys\. Also, as long as its primary key exists, you can recreate a deleted multi\-Region replica key\. For details, see [Deleting multi\-Region keys](multi-region-keys-delete.md)\.

## Scheduling and canceling key deletion<a name="deleting-keys-scheduling-key-deletion"></a>

The following procedures describe how to schedule key deletion and cancel key deletion of single\-Region AWS KMS keys \(KMS keys\) in AWS KMS using the AWS Management Console, the AWS CLI, and the AWS SDK for Java\.

For information about scheduling the deletion of multi\-Region keys, see [Deleting multi\-Region keys](multi-region-keys-delete.md)\.

**Warning**  
Deleting a KMS key is destructive and potentially dangerous\. You should proceed only when you are sure that you don't need to use the KMS key anymore and won't need to use it in the future\. If you are not sure, you should [disable the KMS key](enabling-keys.md) instead of deleting it\.

Before you can delete a KMS key, you must have permission to do so\. If you rely on the key policy alone to specify AWS KMS permissions, you might need to add additional permissions before you can delete the KMS key\. For information about adding these permissions, go to [Adding permission to schedule and cancel key deletion](#deleting-keys-adding-permission)\.

AWS KMS records an entry in your AWS CloudTrail log when you [schedule deletion](ct-schedule-key-deletion.md) of the KMS key and when the [KMS key is actually deleted](ct-delete-key.md)\.

**Topics**
+ [Using the AWS Management Console](#deleting-keys-scheduling-key-deletion-console)
+ [Using the AWS CLI](#deleting-keys-scheduling-key-deletion-cli)
+ [Using the AWS SDK for Java](#deleting-keys-scheduling-key-deletion-java)

### Scheduling and canceling key deletion \(console\)<a name="deleting-keys-scheduling-key-deletion-console"></a>

In the AWS Management Console, you can schedule and cancel the deletion of multiple KMS keys at one time\.

**To schedule key deletion**

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

   You cannot schedule the deletion of [AWS managed keys](concepts.md#aws-managed-cmk) or [AWS owned keys](concepts.md#aws-owned-cmk)\.

1. Select the check box next to the KMS key that you want to delete\.

1. Choose **Key actions**, **Schedule key deletion**\.

1. Read and consider the warning, and the information about canceling the deletion during the waiting period\. If you decide to cancel the deletion, at the bottom of the page, choose **Cancel**\.

1. For **Waiting period \(in days\)**, enter a number of days between 7 and 30\. 

1. Review the KMS keys that you are deleting\.

1. Select the check box next to **Confirm you want to schedule this key for deletion in *<number of days>* days\.**\.

1. Choose **Schedule deletion**\.

The KMS key status changes to **Pending deletion**\.

**To cancel key deletion**

1. Open the AWS KMS console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Select the check box next to the KMS key that you want to recover\.

1. Choose **Key actions**, **Cancel key deletion**\.

The KMS key status changes from **Pending deletion** to **Disabled**\. To use the KMS key, you must [enable it](enabling-keys.md)\.

### Scheduling and canceling key deletion \(AWS CLI\)<a name="deleting-keys-scheduling-key-deletion-cli"></a>

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

### Scheduling and canceling key deletion \(AWS SDK for Java\)<a name="deleting-keys-scheduling-key-deletion-java"></a>

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

## Adding permission to schedule and cancel key deletion<a name="deleting-keys-adding-permission"></a>

If you use IAM policies to allow AWS KMS permissions, all IAM users and roles that have AWS administrator access \(`"Action": "*"`\) or AWS KMS full access \(`"Action": "kms:*"`\) are already allowed to schedule and cancel key the deletion of KMS keys\. If you rely on the key policy alone to allow AWS KMS permissions, you might need to add additional permissions to allow your IAM users and roles to delete KMS keys\. You can add those permissions in the AWS KMS console or by using the AWS KMS API\.

### Adding permission to schedule and cancel key deletion \(console\)<a name="deleting-keys-adding-permission-console"></a>

You can use the AWS Management Console to add permissions for scheduling and canceling key deletion\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Choose the alias or key ID of the KMS key whose permissions you want to change\.

1. Choose the **Key policy** tab\. Under **Key deletion**, select **Allow key administrators to delete this key** and then choose **Save changes**\.
**Note**  
If you do not see the **Allow key administrators to delete this key** option, this usually means that you have changed this key policy using the AWS KMS API\. In this case, you must update the key policy document manually\. Add the `kms:ScheduleKeyDeletion` and `kms:CancelKeyDeletion` permissions to the key administrators statement \(`"Sid": "Allow access for Key Administrators"`\) in the key policy, and then choose **Save changes**\. 

### Adding permission to schedule and cancel key deletion \(AWS CLI\)<a name="deleting-keys-adding-permission-cli"></a>

You can use the AWS Command Line Interface to add permissions for scheduling and canceling key deletion\.

**To add permission to schedule and cancel key deletion**

1. Use the [https://docs.aws.amazon.com/cli/latest/reference/kms/get-key-policy.html](https://docs.aws.amazon.com/cli/latest/reference/kms/get-key-policy.html) command to retrieve the existing key policy, and then save the policy document to a file\.

1. Open the policy document in your preferred text editor, add the `kms:ScheduleKeyDeletion` and `kms:CancelKeyDeletion` permissions to the policy statement that gives permissions to the key administrators \(for example, the policy statement with `"Sid": "Allow access for Key Administrators"`\)\. Then save the file\. The following example shows a policy statement with these two permissions:

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

1. Use the [https://docs.aws.amazon.com/cli/latest/reference/kms/put-key-policy.html](https://docs.aws.amazon.com/cli/latest/reference/kms/put-key-policy.html) command to apply the key policy to the KMS key\.