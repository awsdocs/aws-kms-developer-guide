# Enabling and disabling keys<a name="enabling-keys"></a>

You can disable and re\-enable customer managed keys\. When you create a KMS key, it is enabled by default\. If you disable a KMS key, it cannot be used in any [cryptographic operation](concepts.md#cryptographic-operations) until you re\-enable it\.

Because it's temporary and easily undone, disabling a KMS key is a safe alternative to deleting a KMS key, an action that is destructive and irreversible\. If you are considering deleting a KMS key, disable it first and set a [CloudWatch alarm](deleting-keys-creating-cloudwatch-alarm.md) or similar mechanism to be certain that you'll never need to use the key to decrypt encrypted data\. 

When you disable a KMS key, it becomes unusable right away \(subject to eventual consistency\)\. However, resources encrypted with [data keys](concepts.md#data-keys) protected by the KMS key are not affected until the the KMS key is used again, such as to decrypt the data key\. This issue affects AWS services, many of which use data keys to protect your resources\. For details, see [How unusable KMS keys affect data keys](concepts.md#unusable-kms-keys)\.

You cannot enable or disable [AWS managed keys](concepts.md#aws-managed-cmk) or [AWS owned keys](concepts.md#aws-owned-cmk)\. AWS managed keys are permanently enabled for use by [services that use AWS KMS](service-integration.md)\. AWS owned keys are managed solely by the service that owns them\.

**Note**  
AWS KMS does not rotate the key material of customer managed keys while they are disabled\. For more information, see [How automatic key rotation works](rotate-keys.md#rotate-keys-how-it-works)\.

**Topics**
+ [Enabling and disabling KMS keys \(console\)](#enabling-keys-console)
+ [Enabling and disabling KMS keys \(AWS KMS API\)](#enabling-keys-api)

## Enabling and disabling KMS keys \(console\)<a name="enabling-keys-console"></a>

You can use the AWS KMS console to enable and disable [customer managed keys](concepts.md#customer-cmk)\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Choose the check box for the KMS keys that you want to enable or disable\.

1. To enable a KMS key, choose **Key actions**, **Enable**\. To disable a KMS key, choose **Key actions**, **Disable**\.

## Enabling and disabling KMS keys \(AWS KMS API\)<a name="enabling-keys-api"></a>

The [EnableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKey.html) operation enables a disabled AWS KMS key\. These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. The `key-id` parameter is required\.

This operation does not return any output\. To see the key status, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\.

```
$ aws kms enable-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
```

The [DisableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) operation disables an enabled KMS key\. The `key-id` parameter is required\.

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
        "MultiRegion": false,
        "Enabled": false,
        "KeyState": "Disabled",
        "KeyUsage": "ENCRYPT_DECRYPT",        
        "CreationDate": 1502910355.475,
        "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
        "AWSAccountId": "111122223333"
        "KeySpec": "SYMMETRIC_DEFAULT",
        "CustomerMasterKeySpec": "SYMMETRIC_DEFAULT",
        "EncryptionAlgorithms": [
            "SYMMETRIC_DEFAULT"
        ]
    }
}
```