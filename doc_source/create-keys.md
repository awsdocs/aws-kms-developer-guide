# Creating Keys<a name="create-keys"></a>

You can create customer master key \(CMK\) in the AWS Management Console or by using the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation\.

**Topics**
+ [Creating CMKs \(Console\)](#create-keys-console)
+ [Creating CMKs \(KMS API\)](#create-keys-api)

## Creating CMKs \(Console\)<a name="create-keys-console"></a>

You can use the AWS Management Console to create customer master keys \(CMKs\)\.

**Note**  
AWS KMS recently introduced a new console that makes it easier for you to organize and manage your KMS resources\. We encourage you to try it at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.  
The original console will remain available for a brief period to give you time to familiarize yourself with the new one\. To use the original console, choose **Encryption Keys** in the IAM console or go to [https://console\.aws\.amazon\.com/iam/home\#encryptionKeys](https://console.aws.amazon.com/iam/home#encryptionKeys)\. Please share your feedback by choosing **Feedback** in either console or in the lower\-right corner of this page\.

### To create a new CMK \(new console\)<a name="create-key-kms-console"></a>

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Choose **Create key**\.

1. Type an alias for the CMK\. The alias name cannot begin with **aws/**\. The **aws/** prefix is reserved by Amazon Web Services to represent AWS managed CMKs in your account\.

   An alias is a display name that you can use to identify the CMK\. We recommend that you choose an alias that indicates the type of data you plan to protect or the application you plan to use with the CMK\.

   Aliases are required when you create a CMK in the AWS Management Console\. They are optional when you use the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation\.

1. \(Optional\) Type a description for the CMK\.

   We recommend that you choose a description that explains the type of data you plan to protect or the application you plan to use with the CMK\.

1. Choose **Next**\.

1. \(Optional\) Type a tag key and an optional tag value\. To add more than one tag to the CMK, choose **Add tag**\.

   When you add tags to your AWS resources, AWS generates a cost allocation report with usage and costs aggregated by tags\. For information about tagging CMKs, see [Tagging Keys](tagging-keys.md)\.

1. Choose **Next**\.

1. Select the IAM users and roles that can administer the CMK\.
**Note**  
IAM policies can give other IAM users and roles permission to manage the CMK\.

1. \(Optional\) To prevent the selected IAM users and roles from deleting this CMK, in the **Key deletion** section at the bottom of the page, clear the **Allow key administrators to delete this key** check box\.

1. Choose **Next**\.

1. Select the IAM users and roles that can use the CMK for cryptographic operations\.
**Note**  
The AWS account \(root user\) has full permissions by default\. As a result, any IAM policies can also give users and roles permission use the CMK for cryptographic operations\.

1. \(Optional\) You can allow other AWS accounts to use this CMK for cryptographic operations\. To do so, in the **Other AWS accounts** section at the bottom of the page, choose **Add another AWS account** and enter the AWS account identification number of an external account\. To add multiple external accounts, repeat this step\.
**Note**  
To allow principals in the external accounts to use the CMK, Administrators of the external account must create IAM policies that provide these permissions\. For more information, see [Allowing External AWS Accounts to Access a CMK](key-policy-modifying.md#key-policy-modifying-external-accounts)\.

1. Choose **Next**\.

1. Review the key policy document that was created from your choices\. You can edit it, too\. 

1. Choose **Finish** to create the CMK\.

**Tip**  
To use your new CMK programmatically and in command line interface operations, you need a key ID or key ARN\. For detailed instructions, see [Finding the Key ID and ARN](viewing-keys.md#find-cmk-id-arn)

### To create a new CMK \(original console\)<a name="create-key-iam-console"></a>

1. Sign in to the AWS Management Console and go to [https://console\.aws\.amazon\.com/iam/home\#encryptionKeys](https://console.aws.amazon.com/iam/home#encryptionKeys)\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Choose **Create key**\.

1. Type an alias for the CMK\. The alias name cannot begin with `aws`\. The `aws` prefix is reserved by Amazon Web Services to identify [AWS managed CMKs](concepts.md#master_keys) in your account\.

   An alias is a display name that you can use to identify the CMK\. We recommend that you choose an alias that indicates the type of data you plan to protect or the application you plan to use with the CMK\. 

   Aliases are required when you create a CMK in the AWS Management Console\. They are optional when you use the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation\.

1. \(Optional\) Type a description for the CMK\.

   We recommend that you choose a description that explains the type of data you plan to protect or the application you plan to use with the CMK\.

1. Choose **Next Step**\.

1. \(Optional\) Type a [tag key](tagging-keys.md) and an optional tag value\. To add more than one tag to the CMK, choose **Add tag**\.

1. Choose **Next Step**\.

1. Select which IAM users and roles can administer the CMK\.
**Note**  
The AWS account \(root user\) has full permissions by default\. As a result, any IAM users and roles whose attached policies allow the appropriate permissions can also administer the CMK\.

1. \(Optional\) To prevent the IAM users and roles that you chose in the previous step from deleting this CMK, clear the box at the bottom of the page for **Allow key administrators to delete this key\.**

1. Choose **Next Step**\.

1. Select which IAM users and roles can use the CMK to encrypt and decrypt data with the AWS KMS API\.
**Note**  
The AWS account \(root user\) has full permissions by default\. As a result, any IAM users and roles whose attached policies allow the appropriate permissions can also use the CMK\.

1. \(Optional\) You can use the controls at the bottom of the page to specify other AWS accounts that can use this CMK to encrypt and decrypt data\. To do so, choose **Add an External Account** and then type the intended AWS account ID\. Repeat as necessary to add more than one external account\.
**Note**  
Administrators of the external accounts must also allow access to the CMK by creating IAM policies for their users\. For more information, see [Allowing External AWS Accounts to Access a CMK](key-policy-modifying.md#key-policy-modifying-external-accounts)\.

1. Choose **Next Step**\.

1. Choose **Finish** to create the CMK\.

**Tip**  
To use your new CMK programmatically and in command line interface operations, you need a key ID or key ARN\. For detailed instructions, see [Finding the Key ID and ARN](viewing-keys.md#find-cmk-id-arn)

## Creating CMKs \(KMS API\)<a name="create-keys-api"></a>

The [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation creates a new AWS KMS customer master key \(CMK\)\. These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

This operation has no required parameters\. However, if you are creating a key with no key material, the `Origin` parameter is required\. You might also want to use the `Policy` parameter to specify a key policy\. You can change the key policy \([PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html)\) and add optional elements, such as a [description](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) and [tags](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) at any time\.

The following is an example of a call to the `CreateKey` operation with no parameters\.

```
$ aws kms create-key
{
    "KeyMetadata": {
        "Origin": "AWS_KMS",
        "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
        "Description": "",
        "KeyManager": "CUSTOMER",
        "Enabled": true,
        "KeyUsage": "ENCRYPT_DECRYPT",
        "KeyState": "Enabled",
        "CreationDate": 1502910355.475,
        "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
        "AWSAccountId": "111122223333"
    }
}
```

If you do not specify a key policy for your new CMK, the [default key policy](key-policies.md#key-policy-default) that `CreateKey` applies differs from the default key policy that the console applies when you use it to create a new CMK\. 

For example, this call to the [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation returns the key policy that `CreateKey` applies\. It gives the AWS account root user access to the CMK and allows it to create AWS Identity and Access Management \(IAM\) policies for the CMK\. For detailed information about IAM policies and key policies for CMKs, see [Authentication and Access Control for AWS KMS](control-access.md)

```
$ aws kms get-key-policy --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --policy-name default --output text
{
  "Version" : "2012-10-17",
  "Id" : "key-default-1",
  "Statement" : [ {
    "Sid" : "Enable IAM User Permissions",
    "Effect" : "Allow",
    "Principal" : {
      "AWS" : "arn:aws:iam::111122223333:root"
    },
    "Action" : "kms:*",
    "Resource" : "*"
  } ]
}
```