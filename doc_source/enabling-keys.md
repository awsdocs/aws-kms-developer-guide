# Enabling and Disabling Keys<a name="enabling-keys"></a>

You can disable and reenable the AWS Key Management Service \(AWS KMS\) customer master keys \(CMKs\) that you manage\. You cannot enable or disable AWS managed CMKs\.

When you create a CMK, it is enabled by default\. If you disable a CMK, it cannot be used to encrypt or decrypt data until you re\-enable it\. AWS managed CMKs are permanently enabled for use by [services that use AWS KMS](service-integration.md)\. You cannot disable them\.

You can also delete CMKs\. For more information, see [Deleting Customer Master Keys](deleting-keys.md)\.

**Note**  
AWS KMS does not rotate the backing keys of customer managed CMKs while they are disabled\. For more information, see [How Automatic Key Rotation Works](rotate-keys.md#rotate-keys-how-it-works)\.

**Topics**
+ [Enabling and Disabling CMKs \(Console\)](#enabling-keys-console)
+ [Enabling and Disabling CMKs \(KMS API\)](#enabling-keys-api)

## Enabling and Disabling CMKs \(Console\)<a name="enabling-keys-console"></a>

You can enable and disable customer managed CMKs from the IAM section of the AWS Management Console\.

**Note**  
AWS KMS recently introduced a new console that makes it easier for you to organize and manage your KMS resources\. It is available in all AWS Regions that AWS KMS supports except for AWS GovCloud \(US\)\. We encourage you to try the new AWS KMS console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.  
The original console will remain available for a brief period to give you time to familiarize yourself with the new one\. To use the original console, choose **Encryption Keys** in the IAM console or go to [https://console\.aws\.amazon\.com/iam/home?\#/encryptionKeys](https://console.aws.amazon.com/iam/home?#/encryptionKeys)\. Please share your feedback by choosing **Feedback** in either console or in the lower\-right corner of this page\.

### To enable or disable a CMK \(new console\)<a name="editing-keys-kms-console"></a>

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Select the check box for the CMKs that you want to enable or disable\.

1. To enable a CMK, choose **Key actions**, **Enable**\. To disable a CMK, choose **Key actions**, **Disable**\.

### To enable a CMK \(original console\)<a name="editing-keys-iam-console"></a>

**To enable a CMK \(console\)**

1. Sign in to the AWS Management Console and go to [https://console\.aws\.amazon\.com/iam/home?\#/encryptionKeys](https://console.aws.amazon.com/iam/home?#/encryptionKeys)\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Select the check box next to the alias of the CMKs that you want to enable or disable\.
**Note**  
You cannot disable AWS managed CMKs, which are denoted by the orange AWS icon\.

1. To enable a CMK, choose **Key actions**, **Enable**\. To disable a CMK, choose **Key actions**, **Disable**\.

## Enabling and Disabling CMKs \(KMS API\)<a name="enabling-keys-api"></a>

The [EnableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKey.html) operation enables a disabled AWS KMS customer master key \(CMK\)\. These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. The `key-id` parameter is required\.

This operation does not return any output\. To see the key status, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\.

```
$ aws kms enable-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
```

The [DisableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) operation disables an enabled CMK\. The `key-id` parameter is required\.

```
$ aws kms disable-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
```

This operation does not return any output\. To see the key status, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation, and see the Enabled field\.

```
$ aws kms describe-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
{
    "KeyMetadata": {
        "Origin": "AWS_KMS",
        "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
        "Description": "",
        "KeyManager": "CUSTOMER",
        "Enabled": false,
        "KeyUsage": "ENCRYPT_DECRYPT",
        "KeyState": "Disabled",
        "CreationDate": 1502910355.475,
        "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
        "AWSAccountId": "111122223333"
    }
}
```