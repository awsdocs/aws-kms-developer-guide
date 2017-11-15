# Rotating Customer Master Keys<a name="rotate-keys"></a>

Cryptographic best practices discourage extensive reuse of encryption keys\. To create new cryptographic material for your AWS Key Management Service \(AWS KMS\) customer master keys \(CMKs\), you can create new CMKs, and then change your applications or aliases to use the new CMKs\. Or, you can enable automatic key rotation for an existing CMK\. 

When you enable *automatic key rotation* for a customer managed CMK, AWS KMS generates new cryptographic material for the CMK every year\. AWS KMS also saves the CMK's older cryptographic material so it can be used to decrypt data that it encrypted\.

Key rotation changes only the CMK's *backing key*, which is the cryptographic material that is used in encryption operations\. The properties of the CMK do not change, as shown in the following image\. The CMK is the same logical resource, regardless of whether or how many times its backing key changes\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/key-rotation-auto.png)

Automatic key rotation has the following benefits:

+ The properties of the CMK, including its key ID, key ARN, region, policies, and permissions, do not change when the key is rotated\.

+ You do not need to change applications or aliases that refer to the CMK ID or ARN\.

+ Once you enable key rotation, AWS KMS rotates the CMK automatically every year\. You don't need to remember or schedule the update\.

However, you might decide to create a new CMK and use it in place of the original CMK\. This has the same effect as rotating the key material in an existing CMK, so it's often thought of as manually rotating the key\. Manual rotation is a good choice when you want to control the key rotation schedule\. It also provides a way to rotate CMKs with imported key material\.

**More Information About Key Rotation**  
Rotating customer managed CMKs might result in extra monthly charges\. For details, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/)\. For more detailed information about backing keys and rotation, see the [ KMS Cryptographic Details ](https://d0.awsstatic.com/whitepapers/KMS-Cryptographic-Details.pdf) whitepaper\.


+ [How Automatic Key Rotation Works](#rotate-keys-how-it-works)
+ [How to Enable and Disable Automatic Key Rotation](#rotating-keys-enable-disable)
+ [Rotating Keys Manually](#rotate-keys-manually)

## How Automatic Key Rotation Works<a name="rotate-keys-how-it-works"></a>

Key rotation in AWS KMS is a cryptographic best practice that is designed to be transparent and easy to use\. 

+ Automatic key rotation is disabled on customer managed CMKs by default\. When you enable key rotation for a CMK, AWS KMS automatically creates a new backing key for the CMK every year \(365 days\)\. You do not need to schedule or manage the rotation\. 

   

+ AWS KMS retains all backing keys for a CMK\. When you use a CMK to encrypt, AWS KMS automatically uses the current backing key to perform the encryption\. When you use the CMK to decrypt, AWS KMS automatically uses the version of the backing key that was used to encrypt\. 

   

+ You can enable and disable automatic key rotation at any time\. Even when automatic key rotation is disabled, AWS KMS retains all backing keys that are associated with the CMK\. That way AWS KMS can decrypt any ciphertext that was encrypted with the CMK\.

   

+ While a CMK is in a disabled state, AWS KMS does not rotate it, even if automatic key rotation is enabled on the CMK\. However, when the CMK returns to an enabled state, key rotation is immediately reactivated\. If the backing key for the newly reenabled CMK is more than a year old, AWS KMS immediately rotates it\.

   

+ Automatic key rotation is available on all customer managed CMKs with KMS\-generated key material\. It is not available for CMKs that have imported key material \(the value of the **Origin** field is **External**\), but you can rotate these CMKs manually\. 

   

+ You cannot manage key rotation for AWS managed CMKs\. AWS KMS automatically rotates AWS managed keys every three years\.

   

+ When AWS KMS rotates a CMK, it writes the **KMS CMK Rotation** event to [Amazon CloudWatch Events](http://docs.aws.amazon.com/AmazonCloudWatch/latest/events/)\. You can use this event to verify that the CMK was rotated\. 

## How to Enable and Disable Automatic Key Rotation<a name="rotating-keys-enable-disable"></a>

You can enable and disable automatic key rotation and view the current rotation status of any customer managed CMK in the AWS KMS console and the AWS KMS API\. Key rotation is available only on customer managed CMKs, not AWS managed CMKs\.

### Enabling and Disabling Key Rotation in the Console<a name="rotate-keys-console"></a>

To edit a customer managed CMK, start at the key details page for the CMK\.

1. Sign in to the AWS Management Console and open the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Choose the alias of the CMK whose details you want to see\.
**Note**  
You cannot edit AWS managed CMKs, which are denoted by the orange AWS icon\.

1. Use the controls in the **Key Rotation** section of the page\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-rotate-key.png)

### Enabling and Disabling Key Rotation with the API<a name="rotate-keys-api"></a>

You can use the [AWS Key Management Service \(AWS KMS\) API](http://docs.aws.amazon.com/kms/latest/APIReference/) to enable and disable automatic key rotation, and view the current rotation status of any customer managed CMK\. These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

The [EnableKeyRotation](http://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKeyRotation.html) operation enables automatic key rotation for the specified CMK\. The [DisableKeyRotation](http://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKeyRotation.html) operation disables it\. To identify the CMK, use its key ID, key ARN, alias name, or alias ARN\. By default, key rotation is disabled for customer managed CMKs\.

The following example enables key rotation on the specified CMK and uses the [GetKeyRotationStatus](http://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyRotationStatus.html) operation to see the result\. Then, it disables key rotation and, again, uses **GetKeyRotationStatus** to see the change\.

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

You might prefer to rotate keys manually so you can control the rotation frequency\. It's also a good solution for CMKs that are not eligible for automatic key rotation, such as CMKs with imported key material\.

**Note**  
When you begin using the new CMK, be sure to keep the original CMK enabled so that AWS KMS can decrypt data that the original CMK encrypted\. When decrypting data, KMS identifies the CMK that was used to encrypt the data, and it uses the same CMK to decrypt the data\. As long as you keep both the original and new CMKs enabled, AWS KMS can decrypt any data that was encrypted by either CMK\.

Because the new CMK is a different resource from the current CMK, it has a different key ID and ARN\. When you change CMKs, you need to update references to the CMK ID or ARN in your applications\. Aliases, which associate a friendly name with a CMK, make this process easier\. Use an alias to refer to a CMK in your applications\. Then, when you want to change the CMK that the application uses, change the target CMK of the alias\.

To update the target CMK of an alias, use [UpdateAlias](http://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html) operation in the AWS KMS API\. For example, this command updates the TestCMK alias to point to a new CMK\. Because the operation does not return any output, the example uses the [ListAliases](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation to show that the alias is now associated with a different CMK\.

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

$ aws kms update-alias --alias-name TestCMK --target-key-id 0987dcba-09fe-87dc-65ba-ab0987654321
            
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