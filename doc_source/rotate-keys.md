# Rotating Customer Master Keys<a name="rotate-keys"></a>

Cryptographic best practices discourage extensive reuse of encryption keys\. To create new cryptographic material for your AWS Key Management Service \(AWS KMS\) customer master keys \(CMKs\), you can create new CMKs, and then change your applications or aliases to use the new CMKs\. Or, you can enable automatic key rotation for an existing CMK\. 

When you enable *automatic key rotation* for a customer managed CMK, AWS KMS generates new cryptographic material for the CMK every year\. AWS KMS also saves the CMK's older cryptographic material so it can be used to decrypt data that it encrypted\.

Key rotation changes only the CMK's *backing key*, which is the cryptographic material that is used in encryption operations\. The CMK is the same logical resource, regardless of whether or how many times its backing key changes\. The properties of the CMK do not change, as shown in the following image\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/key-rotation-auto.png)

Automatic key rotation has the following benefits:
+ The properties of the CMK, including its key ID, key ARN, region, policies, and permissions, do not change when the key is rotated\.
+ You do not need to change applications or aliases that refer to the CMK ID or ARN\.
+ After you enable key rotation, AWS KMS rotates the CMK automatically every year\. You don't need to remember or schedule the update\.

However, automatic key rotation has no effect on the data that the CMK protects\. It does not rotate the data keys that the CMK generated or re\-encrypt any data protected by the CMK, and it will not mitigate the effect of a compromised data key\.

You might decide to create a new CMK and use it in place of the original CMK\. This has the same effect as rotating the key material in an existing CMK, so it's often thought of as [manually rotating the key](#rotate-keys-manually)\. Manual rotation is a good choice when you want to control the key rotation schedule\. It also provides a way to rotate CMKs with imported key material\.

**More Information About Key Rotation**  
Rotating customer managed CMKs might result in extra monthly charges\. For details, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/)\. For more detailed information about backing keys and rotation, see the [ KMS Cryptographic Details ](https://d0.awsstatic.com/whitepapers/KMS-Cryptographic-Details.pdf) whitepaper\.

**Topics**
+ [How Automatic Key Rotation Works](#rotate-keys-how-it-works)
+ [How to Enable and Disable Automatic Key Rotation](#rotating-keys-enable-disable)
+ [Rotating Keys Manually](#rotate-keys-manually)

## How Automatic Key Rotation Works<a name="rotate-keys-how-it-works"></a>

Key rotation in AWS KMS is a cryptographic best practice that is designed to be transparent and easy to use\. 
+ **Backing key management\.** AWS KMS retains all backing keys for a CMK, even if key rotation is disabled\. The backing keys are deleted only when the CMK is deleted\. When you use a CMK to encrypt, AWS KMS uses the current backing key\. When you use the CMK to decrypt, AWS KMS uses the backing key that was used to encrypt\. 

   
+ **Enable and disable key rotation\. **Automatic key rotation is disabled by default on customer managed CMKs\. When you enable \(or re\-enable\) key rotation, AWS KMS automatically rotates the CMK 365 days after the enable date and every 365 days thereafter\. 

   
+ **Disabled CMKs\.** While a CMK is disabled, AWS KMS does not rotate it\. However, the key rotation status does not change, and you cannot change it while the CMK is disabled\. When the CMK is re\-enabled, if the backing key is more than 365 days old, AWS KMS rotates it immediately and every 365 days thereafter\. If the backing key is less than 365 days old, AWS KMS resumes the original key rotation schedule\.

   
+ **CMKs pending deletion\.** While a CMK is pending deletion, AWS KMS does not rotate it\. The key rotation status is set to `false` and you cannot change it while deletion is pending\. If deletion is canceled, the previous key rotation status is restored\. If the backing key is more than 365 days old, AWS KMS rotates it immediately and every 365 days thereafter\. If the backing key is less than 365 days old, AWS KMS resumes the original key rotation schedule\.

   
+ **CMKs in custom key stores\.** Automatic key rotation is not available for CMKs in [custom key stores](custom-key-store-overview.md) \(the value of the **Origin** field is **AWS\_CloudHSM**\), but you can [rotate these CMKs manually](#rotate-keys-manually)\. 

   
+ **Imported CMKs\.** Automatic key rotation is not available for CMKs that have [imported key material](importing-keys.md) \(the value of the **Origin** field is **External**\), but you can [rotate these CMKs manually](#rotate-keys-manually)\. 

   
+ **AWS managed CMKs\.** You cannot manage key rotation for AWS managed CMKs\. AWS KMS automatically rotates AWS managed keys every three years \(1095 days\)\.

   
+ **Logging key rotation\.** When AWS KMS rotates a CMK, it writes the **KMS CMK Rotation** event to [Amazon CloudWatch Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/)\. You can use this event to verify that the CMK was rotated\. 

## How to Enable and Disable Automatic Key Rotation<a name="rotating-keys-enable-disable"></a>

You can use the AWS KMS console or the AWS KMS API to enable and disable automatic key rotation, and view the rotation status of any customer managed CMK\.

When you enable automatic key rotation, AWS KMS rotates the CMK 365 days after the enable date and every 365 days thereafter\.

**Topics**
+ [Enabling and Disabling Key Rotation \(Console\)](#rotate-keys-console)
+ [Enabling and Disabling Key Rotation \(KMS API\)](#rotate-keys-api)

### Enabling and Disabling Key Rotation \(Console\)<a name="rotate-keys-console"></a>

**Note**  
AWS KMS recently introduced a new console that makes it easier for you to organize and manage your KMS resources\. We encourage you to try it at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\. Please share your feedback by choosing **Feedback** in either console or in the lower\-right corner of this page\.  
The original console will remain available for a brief period to give you time to familiarize yourself with the new one\. To use the original console, go to [https://console\.aws\.amazon\.com/iam/home\#encryptionKeys](https://console.aws.amazon.com/iam/home#encryptionKeys)\.

#### Enable and Disable Key Rotation \(new console\)<a name="rotate-keys-kms-console"></a>

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\. \(You cannot enable or disable rotation of AWS managed keys\. They are automatically rotated every three years\.\)

1. Choose the alias or key ID of a CMK\.

1. Under **General configuration**, choose the **Key rotation** tab\.

   If the CMK was created without key material \(its **Origin** is **EXTERNAL**\), there is no **Key rotation** tab\. You cannot automatically rotate these CMK, but you can [rotate them manually](#rotate-keys-manually)\.

1. Select or clear the **Automatically rotate this CMK every year** check box\. 
**Note**  
If a CMK is disabled or pending deletion, the **Automatically rotate this CMK every year** check box is cleared, and you cannot change it\. The key rotation status is restored when you enable the CMK or cancel deletion\. For details, see [How Automatic Key Rotation Works](#rotate-keys-how-it-works) and [How Key State Affects Use of a Customer Master Key](key-state.md)\.

1. Choose **Save**\.

#### Enable and Disable Key Rotation \(original console\)<a name="rotate-keys-iam-console"></a>

1. Sign in to the AWS Management Console and go to [https://console\.aws\.amazon\.com/iam/home\#encryptionKeys](https://console.aws.amazon.com/iam/home#encryptionKeys)\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Choose the alias of the CMK whose details you want to see\.
**Note**  
You cannot edit AWS managed CMKs, which are identified by the orange AWS icon\.

1. Use the controls in the **Key Rotation** section of the page\.
**Note**  
If a CMK is disabled or pending deletion, the **Key Rotation** check box is cleared, and you cannot change it\. This reminds you that AWS KMS does not rotate CMKs while they are disabled or pending deletion\. The key rotation status is restored when you re\-enable the CMK or cancel deletion\. For details, see [How Automatic Key Rotation Works](#rotate-keys-how-it-works) and [How Key State Affects Use of a Customer Master Key](key-state.md)\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-rotate-key.png)

### Enabling and Disabling Key Rotation \(KMS API\)<a name="rotate-keys-api"></a>

You can use the [AWS Key Management Service \(AWS KMS\) API](https://docs.aws.amazon.com/kms/latest/APIReference/) to enable and disable automatic key rotation, and view the current rotation status of any customer managed CMK\. These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

The [EnableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKeyRotation.html) operation enables automatic key rotation for the specified CMK\. The [DisableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKeyRotation.html) operation disables it\. To identify the CMK, use its key ID, key ARN, alias name, or alias ARN\. By default, key rotation is disabled for customer managed CMKs\.

The following example enables key rotation on the specified CMK and uses the [GetKeyRotationStatus](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyRotationStatus.html) operation to see the result\. Then, it disables key rotation and, again, uses **GetKeyRotationStatus** to see the change\.

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

## Rotating Keys Manually<a name="rotate-keys-manually"></a>

You might want to create a new CMK and use it in place of a current CMK instead of enabling automatic key rotation\. When the new CMK has different cryptographic material than the current CMK, using the new CMK has the same effect as changing the backing key in an existing CMK\. The process of replacing one CMK with another is known as *manual key rotation*\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/key-rotation-manual.png)

You might prefer to rotate keys manually so you can control the rotation frequency\. It's also a good solution for CMKs that are not eligible for automatic key rotation, such as CMKs in [custom key stores](custom-key-store-overview.md) or CMKs with [imported key material](importing-keys.md)\.

**Note**  
When you begin using the new CMK, be sure to keep the original CMK enabled so that AWS KMS can decrypt data that the original CMK encrypted\. When decrypting data, KMS identifies the CMK that was used to encrypt the data, and it uses the same CMK to decrypt the data\. As long as you keep both the original and new CMKs enabled, AWS KMS can decrypt any data that was encrypted by either CMK\.

Because the new CMK is a different resource from the current CMK, it has a different key ID and ARN\. When you change CMKs, you need to update references to the CMK ID or ARN in your applications\. Aliases, which associate a friendly name with a CMK, make this process easier\. Use an alias to refer to a CMK in your applications\. Then, when you want to change the CMK that the application uses, change the target CMK of the alias\.

To update the target CMK of an alias, use [UpdateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html) operation in the AWS KMS API\. For example, this command updates the `TestCMK` alias to point to a new CMK\. Because the operation does not return any output, the example uses the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation to show that the alias is now associated with a different CMK\.

```
$ aws kms list-aliases
{
    "Aliases": [
        {
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/TestCMK",
            "AliasName": "alias/TestCMK",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab"
        },
    ]
}            

$ aws kms update-alias --alias-name alias/TestCMK --target-key-id 0987dcba-09fe-87dc-65ba-ab0987654321
            
$ aws kms list-aliases
{
    "Aliases": [
        {
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/TestCMK",
            "AliasName": "alias/TestCMK",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321"
        },
    ]
}
```