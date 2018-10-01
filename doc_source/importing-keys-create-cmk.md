# Importing Key Material Step 1: Create an AWS KMS Customer Master Key \(CMK\) With No Key Material<a name="importing-keys-create-cmk"></a>

By default, AWS KMS creates key material for you when you create a customer master key \(CMK\)\. To instead import your own key material, start by creating a CMK with no key material\. You distinguish between these two types of CMKs by the CMK's *origin*\. When AWS KMS creates the key material for you, the CMK's origin is `AWS_KMS`\. When you create a CMK with no key material, the CMK's origin is `EXTERNAL`, which indicates that the key material was generated outside of AWS KMS\.

A CMK with no key material is in the *pending import* state and is not available for use\. To use it, you must import key material as explained later\. When you import key material, the CMK's key state changes to *enabled*\. For more information about key state, see [How Key State Affects Use of a Customer Master Key](key-state.md)\.

To create a CMK with no key material, you can use the AWS Management Console or the AWS KMS API\. You can use the API directly by making HTTP requests, or through one of the [AWS SDKs](https://aws.amazon.com/tools/#sdk) or [command line tools](https://aws.amazon.com/tools/#cli)\.

**Topics**
+ [Create a CMK with No Key Material \(AWS Management Console\)](#importing-keys-create-cmk-console)
+ [Create a CMK with No Key Material \(AWS KMS API\)](#importing-keys-create-cmk-api)

## Create a CMK with No Key Material \(AWS Management Console\)<a name="importing-keys-create-cmk-console"></a>

You can use the AWS Management Console to create a CMK with no key material\. Before you do this, you can configure the console to show additional columns in the list of KMS CMKs to more easily distinguish your CMKs with imported key material\.

**To show additional columns in the list of KMS CMKs**

1. Sign in to the AWS Management Console and open the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Choose the settings button \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-icon-settings.png)\) in the upper\-right corner of the page\.

1. Select the check boxes for **Expiration Date** and **Origin**, and then choose **Close**\.

**To create a CMK with no key material \(console\)**

You need to create a CMK for the imported key material only once\. To reimport the same key material into an existing CMK, see [Step 2: Download the Public Key and Import Token](importing-keys-get-public-key-and-token.md)\.

1. Sign in to the AWS Management Console and open the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Choose **Create key**\.

1. Type an alias and \(optionally\) a description for the CMK\.

1. Choose **Advanced Options**\.

1. For **Key Material Origin**, choose **External**\. Then select the check box next to **I understand the security, availability, and durability implications of using an imported key** to indicate that you understand the implications of using imported key material\. To read about these implications, choose the **security, availability, and durability implications** link\.

   Choose **Next Step**\.

1. Select which IAM users and roles can administer the CMK\. For more information, see [Allows Key Administrators to Administer the CMK](key-policies.md#key-policy-default-allow-administrators)\.
**Note**  
All IAM users and roles with IAM policies that specify the appropriate permissions can also administer the CMK\.

   Choose **Next Step**\.

1. Select which IAM users and roles can use the CMK to encrypt and decrypt data\. For more information, see [Allows Key Users to Use the CMK](key-policies.md#key-policy-default-allow-users)\.
**Note**  
All IAM users and roles with IAM policies that specify the appropriate permissions can also use the CMK\.

1. \(Optional\) At the bottom of the page, you can give permissions to other AWS accounts to use the CMK to encrypt and decrypt data\. Choose **Add an External Account** and then type the AWS account ID of the account to give permissions to\. Repeat as necessary to add more than one external account\.
**Note**  
Administrators of the external accounts must also allow access to the CMK by creating IAM policies for their users\. For more information, see [Allowing External AWS Accounts to Access a CMK](key-policy-modifying.md#key-policy-modifying-external-accounts)\.

   Choose **Next Step**\.

1. Choose **Finish** to create the CMK\.

   After you complete this step, the console displays the **Import key material** wizard\. To continue the process now, see [Download the Public Key and Import Token \(AWS Management Console\)](importing-keys-get-public-key-and-token.md#importing-keys-get-public-key-and-token-console)\.

   Otherwise, choose **Skip and do this later**\. To find your CMK in the list, you can show additional columns as described previously, or you can look in the **Status** column for **Pending Import**\. Your new CMK remains in the **Pending Import** state until you import key material as described in the following steps\.

Proceed to [Step 2: Download the Public Key and Import Token](importing-keys-get-public-key-and-token.md)\.

## Create a CMK with No Key Material \(AWS KMS API\)<a name="importing-keys-create-cmk-api"></a>

To use the [AWS KMS API](https://docs.aws.amazon.com/kms/latest/APIReference/) to create a CMK with no key material, send a [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) request with the `Origin` parameter set to `EXTERNAL`\. The following example shows how to do this with the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/)\.

```
$ aws kms create-key --origin EXTERNAL
```

When the command is successful, you see output similar to the following\. The CMK's `Origin` is `EXTERNAL` and its `KeyState` is `PendingImport`\.

```
{
    "KeyMetadata": {
        "Origin": "EXTERNAL",
        "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
        "Description": "",
        "Enabled": false,
        "KeyUsage": "ENCRYPT_DECRYPT",
        "KeyState": "PendingImport",
        "CreationDate": 1470811233.761,
        "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
        "AWSAccountId": "111122223333"
    }
}
```

Copy the CMK's key ID from your command output to use in later steps, and then proceed to [Step 2: Download the Public Key and Import Token](importing-keys-get-public-key-and-token.md)\.