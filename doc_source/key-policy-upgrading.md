# Keeping Key Policies Up to Date<a name="key-policy-upgrading"></a>

When you [use the AWS Management Console to create a customer master key \(CMK\)](create-keys.md), you can choose the IAM users, IAM roles, and AWS accounts that you want to have access to the CMK\. These users, roles, and accounts are added to a [default key policy](key-policies.md#key-policy-default) that controls access to the CMK\. Occasionally, the default key policy for new CMKs is updated\. Typically, these updates correspond to new AWS KMS features\.

When you create a new CMK, the latest version of the default key policy is added to the CMK\. However, existing CMKs continue to use their existing key policy—that is, new versions of the default key policy are *not* automatically applied to existing CMKs\. Instead, the console alerts you that a newer version is available and prompts you to upgrade it\.

**Note**  
The console alerts you only when you are using the default key policy that was applied when you created the CMK\. If you manually modified the key policy document using the console's *policy view* or the [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) API operation, the console does not alert you when new permissions are available\.

For information about the permissions that are added to a key policy when you upgrade it, see [Changes to the Default Key Policy](#key-policy-changes)\. Upgrading to the latest version of the key policy should not cause problems because it only adds permissions; it doesn't remove any\. We recommend keeping your key policies up to date unless you have a specific reason not to\.

## Determining whether a newer version of the default key policy is available<a name="newer-version"></a>

You can use the AWS Management Console to learn whether a newer version of the default key policy is available\.

**Note**  
AWS KMS recently introduced a new console that makes it easier for you to organize and manage your KMS resources\. It is available in all [AWS Regions that AWS KMS supports](url-kms-regions) except for AWS GovCloud \(US\-East\) and AWS GovCloud \(US\-West\)\. We encourage you to try the new AWS KMS console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.  
The original console will remain available for a brief period to give you time to familiarize yourself with the new one\. To use the original console, choose **Encryption Keys** in the IAM console or go to [https://console\.aws\.amazon\.com/iam/home?\#/encryptionKeys](https://console.aws.amazon.com/iam/home?#/encryptionKeys)\. Please share your feedback by choosing **Feedback** in either console or in the lower\-right corner of this page\.

### To find a newer version of the default key policy \(new console\)<a name="new-version-kms-console"></a>

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Choose the alias or key ID of a CMK that uses the default key policy\.

1. Scroll down to the **Key policy** section of the page\.

   When a newer version of the default key policy is available, the console displays the following alert\.

   **A newer version of the default key policy is available\. Preview and upgrade to the new key policy\.**

### To find a newer version of the default key policy \(original console\)<a name="new-version-iam-console"></a>

1. Sign in to the AWS Management Console and go to [https://console\.aws\.amazon\.com/iam/home?\#/encryptionKeys](https://console.aws.amazon.com/iam/home?#/encryptionKeys)\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Choose the alias of the CMK whose key policy you want to see\. When a newer version of the default key policy is available, the console displays the following alert\.  
![\[Key policy upgrade alert on the console's key details page\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-upgrade.png)

## Upgrading to the latest version of the default key policy<a name="update-default-policy"></a>

When a new default key policy is available, the following alert is displayed in the Key Policy section of the console page\.

**A newer version of the default key policy is available\. Preview and upgrade to the new key policy\.**

**To upgrade to the latest version of the default key policy**

1. If you see an alert announcing a newer version of the default key policy, choose **Preview and upgrade to the new key policy**\.

1. Review the key policy document for the latest version of the default key policy\. For more information about the difference between the latest version and previous versions, see [Changes to the Default Key Policy](#key-policy-changes)\. After reviewing the key policy, choose **Upgrade key policy**\.

## Changes to the Default Key Policy<a name="key-policy-changes"></a>

In the [current version of the default key policy](key-policies.md#key-policy-default), the *key administrators statement* contains more permissions than those in previous versions\. These additional permissions correspond to new AWS KMS features\.

CMKs that use an earlier version of the default key policy might be missing the following permissions\. When you upgrade to the latest version of the default key policy, they're added to the key administrators statement\.

**kms:TagResource and kms:UntagResource**  
These permissions allow key administrators to add, update, and remove tags from the CMK\. They were added to the default key policy when AWS KMS released the [tagging feature](tagging-keys.md)\.

**kms:ScheduleKeyDeletion and kms:CancelKeyDeletion**  
These permissions allow key administrators to schedule and cancel deletion for the CMK\. They were added to the default key policy when AWS KMS released the [CMK deletion feature](deleting-keys.md)\.  
The `kms:ScheduleKeyDeletion` and `kms:CancelKeyDeletion` permissions are included by default when you [create a CMK](create-keys.md) and when you upgrade to the latest version of the default key policy\. However, you can optionally remove them from the default key policy when you create a CMK by clearing the box for **Allow key administrators to delete this key**\. In the same way, you can use the key details page to remove them from the default key policy for existing CMKs\. That includes CMKs whose key policy you upgraded to the latest version\.