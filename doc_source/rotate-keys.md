# Rotating AWS KMS keys<a name="rotate-keys"></a>

Cryptographic best practices discourage extensive reuse of encryption keys\. To create new cryptographic material for your [customer managed keys](concepts.md#customer-cmk), you can create new KMS keys, and then change your applications or aliases to use the new KMS keys\. Or, you can enable automatic key rotation for an existing KMS key\. 

When you enable *automatic key rotation* for a KMS key, AWS KMS generates new cryptographic material for the KMS key every year\. AWS KMS saves all previous versions of the cryptographic material in perpetuity so you can decrypt any data encrypted with that KMS key\. AWS KMS does not delete any rotated key material until you [delete the KMS key](deleting-keys.md)\. You can [track the rotation](#monitor-key-rotation) of key material for your KMS keys in Amazon CloudWatch and AWS CloudTrail\.

When you use a rotated KMS key to encrypt data, AWS KMS uses the current key material\. When you use the rotated KMS key to decrypt ciphertext, AWS KMS uses the version of the key material that was used to encrypt it\. You cannot request a particular version of the key material\. Because AWS KMS transparently decrypts with the appropriate key material, you can safely use a rotated KMS key in applications and AWS services without code changes\. 

However, automatic key rotation has no effect on the data that the KMS key protects\. It does not rotate the [data keys](concepts.md#data-keys) that the KMS key generated or re\-encrypt any data protected by the KMS key, and it will not mitigate the effect of a compromised data key\.

AWS KMS supports automatic key rotation only for [symmetric encryption KMS keys](concepts.md#symmetric-cmks) with key material that AWS KMS creates\. Automatic rotation is optional for [customer managed KMS keys](#rotate-customer-keys)\. AWS KMS always rotates the key material for [AWS managed KMS keys](#rotate-aws-managed-keys) every year\. Rotation of [AWS owned KMS keys](#rotate-aws-owned-keys) varies\.

**Note**  
The rotation interval for AWS managed keys changed in May 2022\. For details, see [AWS managed keys](#rotate-aws-managed-keys)\.

Key rotation changes only the *key material*, which is the cryptographic secret that is used in encryption operations\. The KMS key is the same logical resource, regardless of whether or how many times its key material changes\. The properties of the KMS key do not change, as shown in the following image\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/key-rotation-auto.png)

Automatic key rotation has the following benefits:
+ The properties of the KMS key, including its [key ID](concepts.md#key-id-key-id), [key ARN](concepts.md#key-id-key-ARN), region, policies, and permissions, do not change when the key is rotated\.
+ You do not need to change applications or aliases that refer to the key ID or key ARN of the KMS key\.
+ Rotating key material does not affect the use of the KMS key in any AWS service\. 
+ After you enable key rotation, AWS KMS rotates the KMS key automatically every year\. You don't need to remember or schedule the update\.

You might decide to create a new KMS key and use it in place of the original KMS key\. This has the same effect as rotating the key material in an existing KMS key, so it's often thought of as [manually rotating the key](#rotate-keys-manually)\. Manual rotation is a good choice when you want to control the key rotation schedule\. It also provides a way to rotate KMS keys that are not eligible for automatic key rotation, including [asymmetric KMS keys](symmetric-asymmetric.md), [HMAC KMS keys](hmac.md), KMS keys in [custom key stores](custom-key-store-overview.md), and KMS keys with [imported key material](#rotate-keys)\.

**Key rotation and pricing**  
AWS KMS charges a monthly fee for each version of key material maintained for your KMS key\. For details, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/)\.

**Key rotation and quotas**  
Each KMS key counts as one key when calculating key resource quotas, regardless of the number of rotated key material versions\. 

For detailed information about key material and rotation, see [AWS Key Management Service Cryptographic Details](https://docs.aws.amazon.com/kms/latest/cryptographic-details/)\.

**Topics**
+ [How automatic key rotation works](#rotate-keys-how-it-works)
+ [How to enable and disable automatic key rotation](#rotating-keys-enable-disable)
+ [Rotating keys manually](#rotate-keys-manually)

## How automatic key rotation works<a name="rotate-keys-how-it-works"></a>

Key rotation in AWS KMS is a cryptographic best practice that is designed to be transparent and easy to use\. AWS KMS supports optional automatic key rotation only for [customer managed keys](concepts.md#customer-cmk)\.

**Managing key material**  
AWS KMS retains all key material for a KMS key, even if key rotation is disabled\. AWS KMS deletes key material only when you delete the KMS key\.

**Using key material**  
When you use a rotated KMS key to encrypt data, AWS KMS uses the current key material\. When you use the rotated KMS key to decrypt ciphertext, AWS KMS uses the same version of the key material that was used to encrypt it\. You cannot request a particular version of the key material\.

**Key manager differences**  <a name="rotate-customer-keys"></a>
Automatic key rotation options vary by key manager\.    
**Customer managed keys**  
Automatic key rotation is disabled by default on [customer managed keys](concepts.md#customer-cmk) but authorized users can enable and disable it\. When you enable \(or re\-enable\) automatic key rotation, AWS KMS automatically rotates the KMS key one year \(approximately 365 days\) after the enable date and every year thereafter\.  
**AWS managed keys**  <a name="rotate-aws-managed-keys"></a>
AWS KMS automatically rotates AWS managed keys every year \(approximately 365 days\)\. You cannot enable or disable key rotation for [AWS managed keys](concepts.md#aws-managed-cmk)\.   
In May 2022, AWS KMS changed the rotation schedule for AWS managed keys from every three years \(approximately 1,095 days\) to every year \(approximately 365 days\)\.  
New AWS managed keys are automatically rotated one year after they are created, and approximately every year thereafter\.   
Existing AWS managed keys are automatically rotated one year after their most recent rotation, and every year thereafter\.  
**AWS owned keys**  <a name="rotate-aws-owned-keys"></a>
You cannot enable or disable key rotation for AWS owned keys\. The [key rotation](#rotate-keys) strategy for an AWS owned key is determined by the AWS service that creates and manages the key\. For details, see the *Encryption at Rest* topic in the user guide or developer guide for the service\.

**Supported KMS key types**  
Automatic key rotation is supported only on [symmetric encryption KMS keys](concepts.md#symmetric-cmks) with key material that AWS KMS generates \(Origin = AWS\_KMS\)\.  
Automatic key rotation is *not* supported on the following types of KMS keys, but you can [rotate these KMS keys manually](#rotate-keys-manually)\.  
+ [Asymmetric KMS keys](symmetric-asymmetric.md#asymmetric-cmks)
+ [HMAC KMS keys](hmac.md)
+ KMS keys in [custom key stores](custom-key-store-overview.md)
+ KMS keys with [imported key material](importing-keys.md)

**Multi\-Region keys**  
You can enable and disable automatic key rotation for [multi\-Region keys](multi-region-keys-overview.md)\. You set the property only on the primary key\. When AWS KMS synchronizes the keys, it copies the property setting from the primary key to its replica keys\. When the key material of the primary key is rotated, AWS KMS automatically copies that key material to all of its replica keys\. For details, see [Rotating multi\-Region keys](multi-region-keys-manage.md#multi-region-rotate)\.

**Disabled KMS keys**  
While a KMS key is disabled, AWS KMS does not rotate it\. However, the key rotation status does not change, and you cannot change it while the KMS key is disabled\. When the KMS key is re\-enabled, if the key material is more than one year old, AWS KMS rotates it immediately and every year thereafter\. If the key material is less than one year old, AWS KMS resumes the original key rotation schedule\.

**KMS keys pending deletion**  
While a KMS key is pending deletion, AWS KMS does not rotate it\. The key rotation status is set to `false` and you cannot change it while deletion is pending\. If deletion is canceled, the previous key rotation status is restored\. If the key material is more than one year old, AWS KMS rotates it immediately and every year thereafter\. If the key material is less than one year old, AWS KMS resumes the original key rotation schedule\.

**AWS services**  
You can enable automatic key rotation on the [customer managed keys](concepts.md#customer-cmk) that you use for server\-side encryption in AWS services\. The annual rotation is transparent and compatible with AWS services\.

**Monitoring key rotation**  <a name="monitor-key-rotation"></a>
When AWS KMS automatically rotates the key material for an [AWS managed key](concepts.md#aws-managed-cmk) or [customer managed key](concepts.md#customer-cmk), it writes a `KMS CMK Rotation` event to [Amazon CloudWatch Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/) and a [RotateKey event](ct-rotatekey.md) to your AWS CloudTrail log\. You can use these records to verify that the KMS key was rotated\.

**Eventual consistency**  
Automatic key rotation is subject to the same eventual consistency effects as other AWS KMS management operations\. There might be a slight delay before the new key material is available throughout AWS KMS\. However, rotating key material does not cause any interruption or delay in cryptographic operations\. The current key material is used in cryptographic operations until the new key material is available throughout AWS KMS\. When key material for a multi\-Region key is automatically rotated, AWS KMS uses the current key material until the new key material is available in all Regions with a related multi\-Region key\.

## How to enable and disable automatic key rotation<a name="rotating-keys-enable-disable"></a>

Authorized users can use the AWS KMS console and the AWS KMS API to enable and disable automatic key rotation and view the key rotation status\.

When you enable automatic key rotation, AWS KMS rotates the key material of the KMS key one year after the enable date and every year thereafter\. 

**Topics**
+ [Enabling and disabling key rotation \(console\)](#rotate-keys-console)
+ [Enabling and disabling key rotation \(AWS KMS API\)](#rotate-keys-api)

### Enabling and disabling key rotation \(console\)<a name="rotate-keys-console"></a>

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\. \(You cannot enable or disable rotation of AWS managed keys\. They are automatically rotated every year\.\)

1. Choose the alias or key ID of a KMS key\.

1. Choose the **Key rotation** tab\.

   The **Key rotation** tab appears only on the detail page of symmetric encryption KMS keys with key material that AWS KMS generated \(the **Origin** is **AWS\_KMS**\), including [multi\-Region](multi-region-keys-manage.md#multi-region-rotate) symmetric encryption KMS keys\.

   You cannot automatically rotate asymmetric KMS keys, HMAC KMS keys, KMS keys with [imported key material](importing-keys.md), or KMS keys in [custom key stores](custom-key-store-overview.md)\. However, you can [rotate them manually](#rotate-keys-manually)\.

1. Select or clear the **Automatically rotate this KMS key every year** check box\. 
**Note**  
If a KMS key is disabled or pending deletion, the **Automatically rotate this KMS key every year** check box is cleared, and you cannot change it\. The key rotation status is restored when you enable the KMS key or cancel deletion\. For details, see [How automatic key rotation works](#rotate-keys-how-it-works) and [Key states of AWS KMS keys](key-state.md)\.

1. Choose **Save**\.

### Enabling and disabling key rotation \(AWS KMS API\)<a name="rotate-keys-api"></a>

You can use the [AWS Key Management Service \(AWS KMS\) API](https://docs.aws.amazon.com/kms/latest/APIReference/) to enable and disable automatic key rotation, and view the current rotation status of any customer managed key These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

The [EnableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKeyRotation.html) operation enables automatic key rotation for the specified KMS key\. The [DisableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKeyRotation.html) operation disables it\. To identify the KMS key in these operations, use its [key ID](concepts.md#key-id-key-id) or [key ARN](concepts.md#key-id-key-ARN)\. By default, key rotation is disabled for customer managed keys\.

The following example enables key rotation on the specified symmetric encryption KMS key and uses the [GetKeyRotationStatus](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyRotationStatus.html) operation to see the result\. Then, it disables key rotation and, again, uses **GetKeyRotationStatus** to see the change\.

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

You might prefer to rotate keys manually so you can control the rotation frequency\. It's also a good solution for KMS keys that are not eligible for automatic key rotation, such as asymmetric KMS keys, HMAC KMS keys, KMS keys in [custom key stores](custom-key-store-overview.md), and KMS keys with [imported key material](importing-keys.md)\.

**Note**  
When you begin using the new KMS key, be sure to keep the original KMS key enabled so that AWS KMS can decrypt data that the original KMS key encrypted\.

When you rotate KMS keys manually, you also need to update references to the KMS key ID or key ARN in your applications\. [Aliases](kms-alias.md), which associate a friendly name with a KMS key, can make this process easier\. Use an alias to refer to a KMS key in your applications\. Then, when you want to change the KMS key that the application uses, instead of editing your application code, change the target KMS key of the alias\. For details, see [Using aliases in your applications](alias-using.md)\.

**Note**  
Aliases that point to the latest version of a manually rotated KMS key are a good solution for the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html), [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html), [GenerateMac](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateMac.html), and [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html) operations\. Aliases are not permitted in operations that manage KMS keys, such as [DisableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) or [ScheduleKeyDeletion](https://docs.aws.amazon.com/kms/latest/APIReference/API_ScheduleKeyDeletion.html)\.  
When calling the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation on manually rotated symmetric encryption KMS keys, omit the `KeyId` parameter from the command\. AWS KMS automatically uses the KMS key that encrypted the ciphertext\.  
The `KeyId` parameter is required when calling `Decrypt` or [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) with an asymmetric KMS key, or calling [VerifyMac](https://docs.aws.amazon.com/kms/latest/APIReference/API_VerifyMac.html) with an HMAC KMS key\. These requests fail when the value of the `KeyId` parameter is an alias that no longer points to the KMS key that performed the cryptographic operation, such as when a key is manually rotated\. To avoid this error, you must track and specify the correct KMS key for each operation\.

To change the target KMS key of an alias, use [UpdateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html) operation in the AWS KMS API\. For example, this command updates the `alias/TestKey` alias to point to a new KMS key\. Because the operation does not return any output, the example uses the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation to show that the alias is now associated with a different KMS key and the `LastUpdatedDate` field is updated\. The ListAliases commands use the [`query` parameter](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-filter.html#cli-usage-filter-client-side-specific-values) in the AWS CLI to get only the `alias/TestKey` alias\.

```
$ aws kms list-aliases --query 'Aliases[?AliasName==`alias/TestKey`]'
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
            
$ aws kms list-aliases --query 'Aliases[?AliasName==`alias/TestKey`]'
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