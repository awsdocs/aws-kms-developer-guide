# Creating Keys<a name="create-keys"></a>

You can create [customer master keys](concepts.md#master_keys) \(CMKs\) in the AWS Management Console or by using the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation\.

**Topics**
+ [Creating CMKs \(Console\)](#create-keys-console)
+ [Creating CMKs \(KMS API\)](#create-keys-api)

## Creating CMKs \(Console\)<a name="create-keys-console"></a>

You can use the AWS Management Console to create customer master keys \(CMKs\)\.

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
To allow principals in the external accounts to use the CMK, Administrators of the external account must create IAM policies that provide these permissions\. For more information, see [Allowing Users in Other Accounts to Use a CMK](key-policy-modifying-external-accounts.md)\.

1. Choose **Next**\.

1. Review the key policy document that was created from your choices\. You can edit it, too\. 

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