# Rotating AWS KMS keys<a name="rotate-keys"></a>

Cryptographic best practices discourage extensive reuse of encryption keys\. To create new cryptographic material for your KMS keys, you can create new KMS keys, and then change your applications or aliases to use the new KMS keys\. Or, you can enable automatic key rotation for an existing KMS key\. 

When you enable *automatic key rotation* for a customer managed key, AWS KMS generates new cryptographic material for the KMS key every year\. AWS KMS also saves the KMS key's older cryptographic material in perpetuity so it can be used to decrypt data that the KMS key encrypted\. AWS KMS does not delete any rotated key material until you [delete the KMS key](deleting-keys.md)\.

Key rotation changes only the KMS key's *key material*, which is the cryptographic material that is used in encryption operations\. The KMS key is the same logical resource, regardless of whether or how many times its key material changes\. The properties of the KMS key do not change, as shown in the following image\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/key-rotation-auto.png)

Automatic key rotation has the following benefits:
+ The properties of the KMS key, including its [key ID](concepts.md#key-id-key-id), [key ARN](concepts.md#key-id-key-ARN), region, policies, and permissions, do not change when the key is rotated\.
+ You do not need to change applications or aliases that refer to the key ID or key ARN of the KMS key\.
+ After you enable key rotation, AWS KMS rotates the KMS key automatically every year\. You don't need to remember or schedule the update\.

However, automatic key rotation has no effect on the data that the KMS key protects\. It does not rotate the data keys that the KMS key generated or re\-encrypt any data protected by the KMS key, and it will not mitigate the effect of a compromised data key\.

You might decide to create a new KMS key and use it in place of the original KMS key\. This has the same effect as rotating the key material in an existing KMS key, so it's often thought of as [manually rotating the key](#rotate-keys-manually)\. Manual rotation is a good choice when you want to control the key rotation schedule\. It also provides a way to rotate KMS keys that are not eligible for automatic key rotation, including [asymmetric KMS keys](symmetric-asymmetric.md), KMS keys in [custom key stores](custom-key-store-overview.md), and KMS keys with [imported key material](#rotate-keys)\.

**Key rotation and pricing**  
Rotating customer managed keys might result in extra monthly charges\. For details, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/)\. For more detailed information about key material and rotation, see [AWS Key Management Service Cryptographic Details](https://docs.aws.amazon.com/kms/latest/cryptographic-details/)\.

**Topics**
+ [How automatic key rotation works](#rotate-keys-how-it-works)
+ [How to enable and disable automatic key rotation](#rotating-keys-enable-disable)
+ [Rotating keys manually](#rotate-keys-manually)

## How automatic key rotation works<a name="rotate-keys-how-it-works"></a>

Key rotation in AWS KMS is a cryptographic best practice that is designed to be transparent and easy to use\. AWS KMS supports optional automatic key rotation only for [customer managed keys](concepts.md#customer-cmk)\.
+ **Managing key material\.** AWS KMS retains all key material for a KMS key, even if key rotation is disabled\. Key material is deleted only when the KMS key is deleted\. When you use a KMS key to encrypt, AWS KMS uses the current key material\. When you use the KMS key to decrypt, AWS KMS uses the key material that was used to encrypt\. 
+ **Enable and disable key rotation\.** Automatic key rotation is disabled by default on customer managed keys\. When you enable \(or re\-enable\) automatic key rotation, AWS KMS automatically rotates the KMS key 365 days after the enable date and every 365 days thereafter\. 
+ **Disabled KMS keys\.** While a KMS key is disabled, AWS KMS does not rotate it\. However, the key rotation status does not change, and you cannot change it while the KMS key is disabled\. When the KMS key is re\-enabled, if the key material is more than 365 days old, AWS KMS rotates it immediately and every 365 days thereafter\. If the key material is less than 365 days old, AWS KMS resumes the original key rotation schedule\.
+ **KMS keys pending deletion\.** While a KMS key is pending deletion, AWS KMS does not rotate it\. The key rotation status is set to `false` and you cannot change it while deletion is pending\. If deletion is canceled, the previous key rotation status is restored\. If the key material is more than 365 days old, AWS KMS rotates it immediately and every 365 days thereafter\. If the key material is less than 365 days old, AWS KMS resumes the original key rotation schedule\.
+ **AWS managed keys\.** You cannot manage key rotation for [AWS managed keys](concepts.md#aws-managed-cmk)\. AWS KMS automatically rotates AWS managed keys every three years \(1095 days\)\.  
+ **AWS owned keys\.** You cannot manage key rotation for AWS owned keys\. The [key rotation](#rotate-keys) strategy for an AWS owned key is determined by the AWS service that creates and manages the key\. For details, see the *Encryption at Rest* topic in the user guide or developer guide for the service\.
+ **AWS services\.** You can enable automatic key rotation on the [customer managed keys](concepts.md#customer-cmk) that you use for server\-side encryption in AWS services\. The annual rotation is transparent and compatible with AWS services\.
+ **Multi\-Region keys\.** You can enable and disable automatic key rotation for [multi\-Region keys](multi-region-keys-overview.md)\. You set the property only on the primary key\. When AWS KMS synchronizes the keys, it copies the property setting from the primary key to its replica keys\. When the key material of the primary key is rotated, AWS KMS automatically copies that key material to all of its replica keys\. For details, see [Rotating multi\-Region keys](multi-region-keys-manage.md#multi-region-rotate)\.
+ **Monitoring key rotation\.** When AWS KMS automatically rotates the key material for an [AWS managed key](concepts.md#aws-managed-cmk) or [customer managed key](concepts.md#customer-cmk), it writes a `KMS CMK Rotation` event to [Amazon CloudWatch Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/) and a [RotateKey event](ct-rotatekey.md) to your AWS CloudTrail log\. You can use these records to verify that the KMS key was rotated\.
+ **Unsupported KMS key types\.** Automatic key rotation is *not* supported on the following types of KMS keys, but you can [rotate these KMS keys manually](#rotate-keys-manually)\.
  + [Asymmetric KMS keys](symmetric-asymmetric.md#asymmetric-cmks)
  + KMS keys in [custom key stores](custom-key-store-overview.md)
  + KMS keys that have [imported key material](importing-keys.md)
+ **Eventual consistency**\. Automatic key rotation is subject to the same eventual consistency effects as other AWS KMS management operations\. There might be a slight delay before the new key material is available throughout AWS KMS\. However, rotating key material does not cause any interruption or delay in cryptographic operations\. The current key material is used in cryptographic operations until the new key material is available throughout AWS KMS\. When key material for a multi\-Region key is automatically rotated, AWS KMS uses the current key material until the new key material is available in all Regions with a related multi\-Region key\.

## How to enable and disable automatic key rotation<a name="rotating-keys-enable-disable"></a>

You can use the AWS KMS console or the AWS KMS API to enable and disable automatic key rotation, and view the rotation status of any customer managed key

When you enable automatic key rotation, AWS KMS rotates the KMS key 365 days after the enable date and every 365 days thereafter\. 

**Topics**
+ [Enabling and disabling key rotation \(console\)](#rotate-keys-console)
+ [Enabling and disabling key rotation \(AWS KMS API\)](#rotate-keys-api)

### Enabling and disabling key rotation \(console\)<a name="rotate-keys-console"></a>

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\. \(You cannot enable or disable rotation of AWS managed keys\. They are automatically rotated every three years\.\)

1. Choose the alias or key ID of a KMS key\.

1. Choose the **Key rotation** tab\.

   The **Key rotation** tab only appears on the detail page of symmetric KMS keys with key material that AWS KMS generated \(the **Origin** is **AWS\_KMS**\)\. You cannot automatically rotate asymmetric KMS keys, KMS keys with [imported key material](importing-keys.md), or KMS keys in [custom key stores](custom-key-store-overview.md)\. However, you can [rotate them manually](#rotate-keys-manually)\.

1. Select or clear the **Automatically rotate this KMS key every year** check box\. 
**Note**  
If a KMS key is disabled or pending deletion, the **Automatically rotate this KMS key every year** check box is cleared, and you cannot change it\. The key rotation status is restored when you enable the KMS key or cancel deletion\. For details, see [How automatic key rotation works](#rotate-keys-how-it-works) and [Key states of AWS KMS keys](key-state.md)\.

1. Choose **Save**\.

### Enabling and disabling key rotation \(AWS KMS API\)<a name="rotate-keys-api"></a>

You can use the [AWS Key Management Service \(AWS KMS\) API](https://docs.aws.amazon.com/kms/latest/APIReference/) to enable and disable automatic key rotation, and view the current rotation status of any customer managed key These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

The [EnableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKeyRotation.html) operation enables automatic key rotation for the specified KMS key\. The [DisableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKeyRotation.html) operation disables it\. To identify the KMS key in these operations, use its [key ID](concepts.md#key-id-key-id) or [key ARN](concepts.md#key-id-key-ARN)\. By default, key rotation is disabled for customer managed keys\.

The following example enables key rotation on the specified symmetric KMS key and uses the [GetKeyRotationStatus](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyRotationStatus.html) operation to see the result\. Then, it disables key rotation and, again, uses **GetKeyRotationStatus** to see the change\.

```
$ aws kms enable-key-rotation --key-id 1234abcd-12ab-34cd-56ef-1234567890ab

$ aws kms get-key-rotation-status --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
{
    "KeyRotationEnabled": true
}

$ aws kms disable-key-rotation --key-id 1234abcd-12ab-34cd-56ef-1234567890ab

$ aws kms get-key-rotation-status --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
{
    "KeyRotationEnabled": false
}
```

## Rotating keys manually<a name="rotate-keys-manually"></a>

You might want to create a new KMS key and use it in place of a current KMS key instead of enabling automatic key rotation\. When the new KMS key has different cryptographic material than the current KMS key, using the new KMS key has the same effect as changing the key material in an existing KMS key\. The process of replacing one KMS key with another is known as *manual key rotation*\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/key-rotation-manual.png)

You might prefer to rotate keys manually so you can control the rotation frequency\. It's also a good solution for KMS keys that are not eligible for automatic key rotation, such as asymmetric KMS keys, KMS keys in [custom key stores](custom-key-store-overview.md) and KMS keys with [imported key material](importing-keys.md)\.

**Note**  
When you begin using the new KMS key, be sure to keep the original KMS key enabled so that AWS KMS can decrypt data that the original KMS key encrypted\.

Because the new KMS key is a different resource from the current KMS key, it has a different key ID and ARN\. When you change KMS keys, you need to update references to the KMS key ID or ARN in your applications\. Aliases, which associate a friendly name with a KMS key, make this process easier\. Use an alias to refer to a KMS key in your applications\. Then, when you want to change the KMS key that the application uses, change the target KMS key of the alias\. For details, see [Using aliases in your applications](alias-using.md)\.

To update the target KMS key of an alias, use [UpdateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html) operation in the AWS KMS API\. For example, this command updates the `TestKey` alias to point to a new KMS key\. Because the operation does not return any output, the example uses the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation to show that the alias is now associated with a different KMS key and the `LastUpdatedDate` field is updated\. 

```
$ aws kms list-aliases
{
    "Aliases": [
        {
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/TestKey",
            "AliasName": "alias/TestKey",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "CreationDate": 1521097200.123,
            "LastUpdatedDate": 1521097200.123
        },
    ]
}


$ aws kms update-alias --alias-name alias/TestKey --target-key-id 0987dcba-09fe-87dc-65ba-ab0987654321
            
$ aws kms list-aliases
{
    "Aliases": [
        {
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/TestKey",
            "AliasName": "alias/TestKey",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
            "CreationDate": 1521097200.123,
            "LastUpdatedDate": 1604958290.722
        },
    ]
}
```