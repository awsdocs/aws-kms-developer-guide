# Enabling and disabling keys<a name="enabling-keys"></a>

You can disable and reenable the [customer master keys](concepts.md#master_keys) \(CMKs\) that you manage\. You cannot enable or disable AWS managed CMKs\.

When you create a CMK, it is enabled by default\. If you disable a CMK, it cannot be used to encrypt or decrypt data until you re\-enable it\. AWS managed CMKs are permanently enabled for use by [services that use AWS KMS](service-integration.md)\. You cannot disable them\.

You can also delete CMKs\. For more information, see [Deleting customer master keys](deleting-keys.md)\.

**Note**  
AWS KMS does not rotate the backing keys of customer managed CMKs while they are disabled\. For more information, see [How automatic key rotation works](rotate-keys.md#rotate-keys-how-it-works)\.

**Topics**
+ [Enabling and disabling CMKs \(console\)](#enabling-keys-console)
+ [Enabling and disabling CMKs \(AWS KMS API\)](#enabling-keys-api)

## Enabling and disabling CMKs \(console\)<a name="enabling-keys-console"></a>

You can use the AWS KMS console to enable and disable [customer managed CMKs](concepts.md#customer-cmk)\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Select the check box for the CMKs that you want to enable or disable\.

1. To enable a CMK, choose **Key actions**, **Enable**\. To disable a CMK, choose **Key actions**, **Disable**\.

## Enabling and disabling CMKs \(AWS KMS API\)<a name="enabling-keys-api"></a>

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
        "KeyState": "Disabled",
        "KeyUsage": "ENCRYPT_DECRYPT",        
        "CreationDate": 1502910355.475,
        "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
        "AWSAccountId": "111122223333"
        "CustomerMasterKeySpec": "SYMMETRIC_DEFAULT",
        "EncryptionAlgorithms": [
            "SYMMETRIC_DEFAULT"
        ]
    }
}
```