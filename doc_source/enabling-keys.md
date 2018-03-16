# Enabling and Disabling Keys<a name="enabling-keys"></a>

You can disable and reenable the AWS Key Management Service \(AWS KMS\) customer master keys \(CMKs\) that you manage\. You cannot change the status of AWS managed CMKs\.

When you create a CMK, it is enabled by default\. If you disable a CMK, it cannot be used to encrypt or decrypt data until you re\-enable it\. AWS managed CMKs are permanently enabled for use by [services that use AWS KMS](service-integration.md)\. You cannot disable them\.

You can also delete CMKs\. For more information, see [Deleting Customer Master Keys](deleting-keys.md)\.

**Note**  
AWS KMS does not rotate the backing keys of customer\-managed CMKs while they are disabled\. For more information, see [How Automatic Key Rotation Works](rotate-keys.md#rotate-keys-how-it-works)\.


+ [Enabling and Disabling CMKs \(Console\)](#enabling-keys-console)
+ [Enabling and Disabling CMKs \(API\)](#enabling-keys-api)

## Enabling and Disabling CMKs \(Console\)<a name="enabling-keys-console"></a>

You can enable and disable customer managed CMKs from the IAM section of the AWS Management Console\.

**To enable a CMK \(console\)**

1. Sign in to the AWS Management Console and open the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Select the check box next to the alias of the CMKs that you want to enable or disable\.
**Note**  
You cannot disable AWS managed CMKs, which are denoted by the orange AWS icon\.

1. To enable a CMK, choose **Key actions**, **Enable**\. To disable a CMK, choose **Key actions**, **Disable**\.

## Enabling and Disabling CMKs \(API\)<a name="enabling-keys-api"></a>

The [EnableKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKey.html) operation enables a disabled AWS KMS customer master key \(CMK\)\. These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. The `key-id` parameter is required\.

This operation does not return any output\. To see the key status, use the [DescribeKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\.

```
$ aws kms enable-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
```

The [DisableKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) operation disables an enabled CMK\. The `key-id` parameter is required\.

```
$ aws kms disable-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
```

This operation does not return any output\. To see the key status, use the [DescribeKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation, and see the Enabled field\.

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