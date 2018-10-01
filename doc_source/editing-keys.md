# Editing Keys<a name="editing-keys"></a>

You can use the key detail page of the **Encryption keys** section of the AWS Management Console to edit some of the properties of the customer master keys \(CMKs\) that you manage\. You can change the description, add and remove administrators and users, manage tags, and enable and disable key rotation\. 

You can also use the operations in the [AWS Key Management Service \(AWS KMS\) API](https://docs.aws.amazon.com/kms/latest/APIReference/) to edit the CMKs that you manage\. You cannot changes the properties of AWS managed CMKs\.

**Topics**
+ [Editing CMKs \(console\)](#editing-keys-console)
+ [Editing CMKs \(API\)](#editing-keys-cli)

## Editing CMKs \(console\)<a name="editing-keys-console"></a>

To edit a customer\-managed CMK, start at the key details page for the CMK\.

**To view the key details page for a CMK \(console\)**

1. Sign in to the AWS Management Console and open the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Choose the alias of the CMK whose details you want to see\.
**Note**  
You cannot edit AWS managed CMKs, which are denoted by the orange AWS icon\.

On the key details page, you can view metadata about the CMK, and you can edit the CMK in the following ways:

**Change the description**  
In the **Summary** section, type a brief description of the CMK in the **Description** box\. When you are finished, choose **Save Changes**\.  

![\[Summary section of the console's key details page\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-summary.png)

**Add and remove key administrators, and allow or disallow key administrators to delete the CMK**  
Use the controls in the **Key Administrators** area in the **Key Policy** section of the page\.  

![\[Key administrators area in the console's key policy section\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-administrators.png)

**Add and remove key users, and allow and disallow external AWS accounts to use the CMK**  
Use the controls in the **Key Users** area in the **Key Policy** section of the page\.  

![\[Key users area in the console's key policy section\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-users.png)

**Add, edit, and remove tags**  
Use the controls in the **Tags** section of the page\.  

![\[Tags section of the console's key details page\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-tags.png)

**Enable or disable rotation**  
Use the controls in the **Key Rotation** section of the page to enable and disable [automatic rotation](rotate-keys.md) of the cryptographic material in a CMK\.  

![\[Key rotation section of the console's key details page\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-rotation.png)

## Editing CMKs \(API\)<a name="editing-keys-cli"></a>

You can use the [AWS Key Management Service \(AWS KMS\) API](https://docs.aws.amazon.com/kms/latest/APIReference/) to edit your customer\-managed CMKs\. These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. This section demonstrates several operations that return details about existing CMKs\.

You cannot edit the properties of AWS managed CMKs\.

**Topics**
+ [UpdateKeyDescription: Change the Description of a CMK](#editing-keys-edit-description)
+ [PutKeyPolicy: Change the Key Policy for a CMK](#editing-keys-edit-key-policy)
+ [Enable and Disable Key Rotation](#editing-keys-enable-key-rotation)

**Tip**  
For information about adding, deleting, and editing tags, see [Tagging Keys](tagging-keys.md)\.

### UpdateKeyDescription: Change the Description of a CMK<a name="editing-keys-edit-description"></a>

The [UpdateKeyDescription](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateKeyDescription.html) operation adds or changes the description of a CMK\. To see the description, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\.

For example, this call to the `UpdateKeyDescription` operation changes the description of the specified CMK\.

```
$ aws kms update-key-description --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
                                 --description "Example key"
```

To get the description of a key, use the `DescribeKey` operation, as shown in the following example\.

```
$ aws kms describe-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
        
{
    "KeyMetadata": {
    "Origin": "AWS_KMS",
    "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
    "Description": "Example key",
    "KeyManager": "CUSTOMER",
    "Enabled": true,
    "KeyUsage": "ENCRYPT_DECRYPT",
    "KeyState": "Enabled",
    "CreationDate": 1499988169.234,
    "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
    "AWSAccountId": "111122223333"
    }
}
```

### PutKeyPolicy: Change the Key Policy for a CMK<a name="editing-keys-edit-key-policy"></a>

The [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operation changes the key policy of the CMK to the policy that you specify\. The policy includes permissions for administrators, users, and roles\. For a detailed example, see [PutKeyPolicy Examples](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html#API_PutKeyPolicy_Examples)\.

### Enable and Disable Key Rotation<a name="editing-keys-enable-key-rotation"></a>

The [EnableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKeyRotation.html) operation enables [automatic rotation](rotate-keys.md) of the cryptographic material in a CMK\. The [DisableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKeyRotation.html) operation disables it\. The [GetKeyRotationStatus](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyRotationStatus.html) operation returns a Boolean value that tells you whether automatic key rotation is enabled \(**true**\) or disabled \(**false**\)\. 

For an example, see [Rotating Customer Master Keys](rotate-keys.md)\.