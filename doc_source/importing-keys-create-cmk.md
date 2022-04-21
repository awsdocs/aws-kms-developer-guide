# Importing key material step 1: Create an AWS KMS key without key material<a name="importing-keys-create-cmk"></a>

By default, AWS KMS creates key material for you when you create an AWS KMS key\. To import your own key material instead, start by creating a KMS key with no key material\. Then import the key material\. To create a KMS key with no key material, you can use the AWS Management Console or the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation\.

To create a key with no key material, specify a key spec of `SYMMETRIC_DEFAULT` \(the default value\) and an origin of `EXTERNAL`\. The key spec and origin of a KMS key are immutable values\. Once you create it, you cannot convert a KMS key designed for imported key material into a KMS key with key material from AWS KMS or any other source\.

The [key state](key-state.md) of a KMS key with an `EXTERNAL` origin and no key material is `PendingImport`\. A KMS key can remain in `PendingImport` state indefinitely\. However, you cannot use a KMS key in `PendingImport` state in cryptographic operations\.When you import key material, the key state of the KMS key changes to *enabled*, and you can use it in cryptographic operations\.

**AWS KMS records an entry in your AWS CloudTrail log when you [create the KMS key](ct-createkey.md), [download the public key and import token](ct-getparametersforimport.md), and [import the key material](ct-importkeymaterial.md)\. AWS KMS also records an entry when you delete imported key material or when AWS KMS [deletes expired key material](ct-deleteexpiredkeymaterial.md)\.**
+ [Creating a KMS key with no key material \(console\)](#importing-keys-create-cmk-console)
+ [Creating a KMS key with no key material \(AWS KMS API\)](#importing-keys-create-cmk-api)

For information about creating multi\-Region keys with imported key material, see [Importing key material into multi\-Region keys](multi-region-keys-import.md)\.

**Topics**
+ [Creating a KMS key with no key material \(console\)](#importing-keys-create-cmk-console)
+ [Creating a KMS key with no key material \(AWS KMS API\)](#importing-keys-create-cmk-api)

## Creating a KMS key with no key material \(console\)<a name="importing-keys-create-cmk-console"></a>

You can use the AWS Management Console to create a symmetric encryption KMS key with no key material\. Before you do this, you can configure the console to show the **Origin** column in the list of KMS keys\. Imported keys have an **Origin** value of **External**\.

You need to create a KMS key for the imported key material only once\. To reimport the same key material into an existing KMS key, see [Step 2: Download the public key and import token](importing-keys-get-public-key-and-token.md)\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Choose **Create key**\.

1. Choose **Symmetric**\. 

   You cannot import key material into an [asymmetric KMS key](symmetric-asymmetric.md)\.

1. In **Key usage**, the **Encrypt and decrypt** option is selected for you\. Do not change it\. 

   You cannot import key material into an [HMAC KMS key](hmac.md)\.

1. Expand **Advanced options**\.

1. For **Key material origin**, choose **External**\. 

   You cannot import key material into a KMS key in a [custom key store](custom-key-store-overview.md)\.

   Then select the check box next to **I understand the security, availability, and durability implications of using an imported key** to indicate that you understand the implications of using imported key material\. To read about these implications, see [About imported key material](importing-keys.md#importing-keys-considerations)\.

1. By default, this procedure creates a KMS key in the selected Region\. 

   To create a multi\-Region primary key with no key material, in the **Regionality** section, choose **Multi\-Region key**\. For details, see [Importing key material into multi\-Region keys](multi-region-keys-import.md)\.

1. Choose **Next**\.

1. Type an alias and \(optionally\) a description for the KMS key\. 

   Choose **Next**\.

1. \(Optional\)\. On the **Add tags** page, add tags that identify or categorize your KMS key\. 

   Choose **Next**\.

1. In the **Key administrators** section, select the IAM users and roles who can manage the KMS key\. For more information, see [Allows key administrators to administer the KMS key](key-policy-default.md#key-policy-default-allow-administrators)\. 
**Note**  
IAM policies can give other IAM users and roles permission to manage the KMS key\.

1. \(Optional\) To prevent the selected IAM users and roles from deleting this KMS key, in the **Key deletion** section at the bottom of the page, clear the **Allow key administrators to delete this key** check box\.

   Choose **Next**\.

1. In the **This account** section, select the IAM users and roles in this AWS account who can use the KMS key in [cryptographic operations](concepts.md#cryptographic-operations)\. For more information, see [Allows key users to use the KMS key](key-policy-default.md#key-policy-default-allow-users)\.
**Note**  
IAM policies can give other IAM users and roles permission to use the KMS key\.

1. \(Optional\) You can allow other AWS accounts to use this KMS key for cryptographic operations\. To do so, in the **Other AWS accounts** section at the bottom of the page, choose **Add another AWS account** and enter the AWS account ID of an external account\. To add multiple external accounts, repeat this step\.
**Note**  
To allow principals in the external accounts to use the KMS key, Administrators of the external account must create IAM policies that provide these permissions\. For more information, see [Allowing users in other accounts to use a KMS key](key-policy-modifying-external-accounts.md)\.

   Choose **Next**\.

1. Review the key settings that you chose\. You can still go back and change all settings\.

1. When you're done, choose **Finish** to create the key\.

   If the operation succeeds, you have created a KMS key with no key material\. Its status is **Pending import\.** To continue the process now, see [Downloading the public key and import token \(console\)](importing-keys-get-public-key-and-token.md#importing-keys-get-public-key-and-token-console)\. To continue the process later, choose **Cancel**\.

Next: [Step 2: Download the public key and import token](importing-keys-get-public-key-and-token.md)\.

## Creating a KMS key with no key material \(AWS KMS API\)<a name="importing-keys-create-cmk-api"></a>

To use the [AWS KMS API](https://docs.aws.amazon.com/kms/latest/APIReference/) to create a symmetric encryption KMS key with no key material, send a [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) request with the `Origin` parameter set to `EXTERNAL`\. The following example shows how to do this with the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/)\.

```
$ aws kms create-key --origin EXTERNAL
```

When the command is successful, you see output similar to the following\. The AWS KMS key's `Origin` is `EXTERNAL` and its `KeyState` is `PendingImport`\.

```
{
    "KeyMetadata": {
        "Origin": "EXTERNAL",
        "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
        "Description": "",
        "Enabled": false,
        "MultiRegion": false,
        "KeyUsage": "ENCRYPT_DECRYPT",
        "KeyState": "PendingImport",
        "CreationDate": 1568289600.0,
        "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
        "AWSAccountId": "111122223333",
        "KeyManager": "CUSTOMER",
        "KeySpec": "SYMMETRIC_DEFAULT",
        "CustomerMasterKeySpec": "SYMMETRIC_DEFAULT",
        "EncryptionAlgorithms": [
            "SYMMETRIC_DEFAULT"
        ]
    }
}
```

Copy the `KeyId` value from your command output to use in later steps, and then proceed to [Step 2: Download the public key and import token](importing-keys-get-public-key-and-token.md)\.