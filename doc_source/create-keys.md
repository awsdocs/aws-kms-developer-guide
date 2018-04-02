# Creating Keys<a name="create-keys"></a>

You can use the IAM section of the AWS Management Console to create a customer master key \(CMK\)\. You can also use the [CreateKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation in the [AWS Key Management Service \(AWS KMS\) API](http://docs.aws.amazon.com/kms/latest/APIReference/)\.

**Topics**
+ [Creating CMKs \(Console\)](#create-keys-console)
+ [Creating CMKs \(API\)](#create-keys-api)

## Creating CMKs \(Console\)<a name="create-keys-console"></a>

You can use the AWS Management Console to create customer master keys \(CMKs\)\.

**To create a new CMK \(console\)**

1. Sign in to the AWS Management Console and open the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Choose **Create key**\.

1. Type an alias for the CMK\. An alias cannot begin with **aws**\. Aliases that begin with **aws** are reserved by Amazon Web Services to represent AWS\-managed CMKs in your account\.

   An alias is a display name that you can use to identify the CMK\. We recommend that you choose an alias that indicates the type of data you plan to protect or the application you plan to use with the CMK\.

1. \(Optional\) Type a description for the CMK\.

   We recommend that you choose a description that explains the type of data you plan to protect or the application you plan to use with the CMK\.

1. Choose **Next Step**\.

1. \(Optional\) Type a tag key and an optional tag value\. To add more than one tag to the CMK, choose **Add tag**\.

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
To refer to your new CMK programmatically and in command line interface operations, you need a key ID or key ARN\.  
The key ID is displayed in the [**Encryption keys** section](https://console.aws.amazon.com/iam/home#encryptionKeys) of the IAM console\. To find the key ARN, in the **Encryption keys** section, choose the region, and then choose the CMK alias\.   
You can also find the CMK ID and ARN by using the [ListKeys operation](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html) in the AWS KMS API\. For details, see [Finding the Key ID and ARN](viewing-keys.md#find-cmk-id-arn)\. 

## Creating CMKs \(API\)<a name="create-keys-api"></a>

The [CreateKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation creates a new AWS KMS customer master key \(CMK\)\. These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

This operation has no required parameters\. However, if you are creating a key with no key material, the `Origin` parameter is required\. You might also want to use the `Policy` parameter to specify a key policy\. You can change the key policy \([PutKeyPolicy](http://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html)\) and add optional elements, such as a [description](http://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) and [tags](http://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) at any time\.

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

If you do not specify a key policy for your new CMK, the [default key policy](key-policies.md#key-policy-default) that `CreateKey` applies is different from the default key policy that the console applies when you use it to create a new CMK\. 

For example, this call to the [GetKeyPolicy](http://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation returns the key policy that `CreateKey` applies\. It gives the AWS account root user access to the CMK and allows it to create AWS Identity and Access Management \(IAM\) policies for the CMK\. For detailed information about IAM policies and key policies for CMKs, see [Authentication and Access Control for AWS KMS](control-access.md)

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