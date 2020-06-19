# Editing keys<a name="editing-keys"></a>

You can use the AWS KMS API and the key detail page of the AWS Management Console to edit some of the properties of your customer managed [customer master keys](concepts.md#master_keys) \(CMKs\)\. You can change the description, add and remove administrators and users, manage tags, and enable and disable key rotation\. 

You cannot change the properties of [AWS managed CMKs](concepts.md#master_keys)\.

**Topics**
+ [Editing CMKs \(console\)](#editing-keys-console)
+ [Editing CMKs \(AWS KMS API\)](#editing-keys-cli)

## Editing CMKs \(console\)<a name="editing-keys-console"></a>

Users who have the required permissions can change the properties of a customer managed CMK, including its description, tags, policies and grants, and rotation status in the AWS Management Console\.

You can [view](viewing-keys.md), but not edit, the properties of AWS managed CMKs\. To view the key policy for an AWS managed CMK, use the [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation\.

**Navigate to the CMK details page**  

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\. \(You cannot edit the properties of AWS managed keys\.\)

1. Choose the alias or key ID of the CMK that you want to edit\. Now, use the controls on the key details page to view and change the properties of the CMK\.

**Change the CMK description**  
You can add, change, or delete the description of your CMK unless its [key state](key-state.md) is `Pending Deletion`\. The description is optional\.  

1. In the upper\-right corner, choose **Edit**\.

1. For **Description**, type a brief description of the CMK\. You can also delete an existing description\.

   Enter a description that explains the type of data you plan to protect or the application you plan to use with the CMK\. The *Default master key that protects my \.\.\. when no other key is defined* description format is reserved for [AWS managed CMKs](concepts.md#aws-managed-cmk)\.

1. To save your changes, choose **Save**\.

**Change CMK administrators and users**  
You can change the key policy for your CMK\. Key policies define the IAM users, groups, and roles that can manage the CMK and use it for [cryptographic operations](concepts.md#cryptographic-operations)\.   
The AWS account \(root user\) has full permissions by default\. As a result, any IAM users and roles whose attached policies allow the appropriate permissions can also administer the CMK\. For detailed information about setting key policies and IAM policies, see [Authentication and access control for AWS KMS](control-access.md)\.  

1. On the details page for CMK, choose the **Key policy** tab\.

   If the key policy for the CMK is a default policy, the **Key policy** tab displays the default view with **Key administrators**, **Key deletion**, **Key users**, and **Other AWS accounts** sections\. Otherwise, the tab displays the key policy document\.

   To edit the key policy document directly, choose **Switch to policy view** \(if applicable\), choose **Edit**, edit the document, then choose **Save**\.

   The remaining steps in this procedure explain how to edit the key policy using the default view\.

1. To change the users and roles who can manage the CMK, use the **Key administrators** section\.
   + To add a key administrator, choose **Add**, choose or type a user or role, then choose **Add**\.
   + To remove a key administrator, check the box for the user or role, then choose **Remove**\.

1. To prevent the key administrators from scheduling deletion of the CMK, in the **Key deletion** section, clear the **Allow key administrators to delete this key** check box\.

1. To change the users and roles who can use the CMK in [cryptographic operations](concepts.md#cryptographic-operations), use the **Key users** section\.
   + To add a key user, choose **Add**, choose a user or role, then choose **Add**\.
   + To remove a key user, check the box for the user or role, then choose **Remove**\.

1. To change the other AWS accounts that can use the CMK in cryptographic operations, in the **Other AWS accounts** section, choose **Add other AWS accounts**\.
**Note**  
Adding an external account does not allow users and roles in the account to use the CMK\. To allow users an roles in an external account to use the CMK, an administrator of the external account must add IAM policies that provide these permissions\. For more information, see [Allowing users in other accounts to use a CMK](key-policy-modifying-external-accounts.md)\.
   + To add accounts, choose **Add another AWS account**, type the account number\. 
   + To remove accounts, on the row with the account number, choose **Remove**\. 

   When you are done, choose **Save changes**, then click the **X** to close the window\.

**Add, edit, and delete tags**  <a name="edit-tags"></a>
You can create and change the tags on your customer managed CMKs\. For detailed information about using tags in AWS KMS, see [Tagging keys](tagging-keys.md)\.  
+ On the details page for CMK, choose the **Tags** tab\.
  + To create your first tag, choose **Create tag**, type a tag name \(required\) and tag value \(optional\), and then choose **Save**\.

    If you leave the tag value blank, the actual tag value is a null or empty string\.
  + To add a tag, choose **Edit**, choose **Add tag**, type a tag name and tag value, and then choose **Save**\.
  + To change the name or value of a tag, choose **Edit**, make your changes, and then choose **Save**\.
  + To delete a tag, choose **Edit**\. On the tag row, choose **Remove**, and then choose **Save**\.

**Enable or disable rotation**  
You can enable and disable [automatic rotation](rotate-keys.md) of the cryptographic material in a symmetric [customer managed CMK](concepts.md#master_keys)\. This feature is not supported for asymmetric CMKs or for CMKs with imported key material\.  
[AWS managed CMKs](concepts.md#master_keys) are automatically rotated every three years\. You cannot enable or disable this feature\.  

1. On the details page for CMK, choose the **Key rotation** tab\.

1. To enable automatic key rotation, check the **Automatically rotate this CMK every year** check box\. To disable automatic key rotation, clear the check box\.

1. To save your changes, choose **Save**\.

## Editing CMKs \(AWS KMS API\)<a name="editing-keys-cli"></a>

You can use the [AWS Key Management Service \(AWS KMS\) API](https://docs.aws.amazon.com/kms/latest/APIReference/) to edit the properties of your [customer managed CMKs](concepts.md#master_keys)\. These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. This section demonstrates several operations that return details about existing CMKs\.

You cannot edit the properties of [AWS managed CMKs](concepts.md#master_keys) or [AWS owned CMKs](concepts.md#aws-owned-cmk)\.

### UpdateKeyDescription: Change the description of a CMK<a name="editing-keys-edit-description"></a>

The [UpdateKeyDescription](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateKeyDescription.html) operation replaces the description of a customer managed CMK with the one that you specify\. You can use it to add, change, or delete the description of a CMK\. To see the description, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\.

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
      "CustomerMasterKeySpec": "SYMMETRIC_DEFAULT",
      "EncryptionAlgorithms": [
        "SYMMETRIC_DEFAULT"
      ]
    }
}
```

### PutKeyPolicy: Change the key policy for a CMK<a name="editing-keys-edit-key-policy"></a>

The [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operation changes the key policy of a customer managed CMK to the policy that you specify\. The policy includes permissions for administrators, users, and roles\. For a detailed example, see [PutKeyPolicy Examples](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html#API_PutKeyPolicy_Examples)\.

### Enable and disable key rotation<a name="editing-keys-enable-key-rotation"></a>

The [EnableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKeyRotation.html) operation enables [automatic rotation](rotate-keys.md) of the cryptographic material in a symmetric customer managed CMK\. The [DisableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKeyRotation.html) operation disables it\. The [GetKeyRotationStatus](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyRotationStatus.html) operation returns a Boolean value that tells you whether automatic key rotation is enabled \(**true**\) or disabled \(**false**\)\. 

For an example, see [Rotating customer master keys](rotate-keys.md)\.

### Add, edit, and delete tags<a name="editing-keys-tag-key"></a>

To add or edit the tag on a customer managed CMK, use the [TagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) operation\. To add a tag, specify a new tag key and a tag value\. To edit a tag, specify an existing tag key and a new tag value\. 

To view the tags on any CMK, use the [ListResourceTags](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListResourceTags.html) operation\.

To delete a tag from a customer managed CMK, use the [UntagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_UntagResource.html) operation\.

For more information about using tags in AWS KMS, see [Tagging keys](tagging-keys.md)\.