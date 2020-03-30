# Viewing a key policy<a name="key-policy-viewing"></a>

You can view the key policy for an AWS KMS [customer managed CMK](concepts.md#customer-cmk) or an [AWS managed CMK](concepts.md#aws-managed-cmk) in your account by using the AWS Management Console or the [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation in the AWS KMS API\. You cannot use these techniques to view the key policy of a CMK in a different AWS account\. 

To learn more about AWS KMS key policies, see [Using key policies in AWS KMS](key-policies.md)\. To learn how to determine which users and roles have access to a CMK, see [Determining access to an AWS KMS customer master key](determining-access.md)\.

**Topics**
+ [Viewing a key policy \(console\)](#key-policy-viewing-console)
+ [Viewing a key policy \(AWS KMS API\)](#key-policy-viewing-api)

## Viewing a key policy \(console\)<a name="key-policy-viewing-console"></a>

Authorized users can view the key policy for an [AWS managed CMK](concepts.md#aws-managed-cmk) or a [customer managed CMK](concepts.md#customer-cmk) on the **Key policy** tab of the AWS Management Console\. 

To view the key policy for a CMK in the AWS Management Console, you must have [kms:ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html), [kms:DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html), and [kms:GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) permissions\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. To view the keys in your account that AWS creates and manages for you, in the navigation pane, choose **AWS managed keys**\. To view the keys in your account that you create and manage, in the navigation pane choose **Customer managed keys**\.

1. In the list of CMKs, choose the alias or key ID of the CMK that you want to examine\.

1. Choose the **Key policy** tab\.

   In the **Key policy** section, you might see the key policy document\. This is *policy view*\. In the key policy statements, you can see the principals who have been given access to the CMK by the key policy, and you can see the actions they can perform\.

   The following example shows the policy view for the [default key policy](key-policies.md#key-policy-default)\.   
![\[View of the default key policy in policy view in the AWS KMS console\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-view.png)

   Or, if you created the CMK in the AWS Management Console, you will see the *default view* with sections for **Key administrators**, **Key deletion**, and **Key Users**\. To see the key policy document, choose **Switch to policy view**\.

   The following example shows the default view for the [default key policy](key-policies.md#key-policy-default)\.   
![\[View of the default key policy in default view in the AWS KMS console\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-full-vsm.png)

## Viewing a key policy \(AWS KMS API\)<a name="key-policy-viewing-api"></a>

To get the key policy for an [AWS managed CMK](concepts.md#aws-managed-cmk) or a [customer managed CMK](concepts.md#customer-cmk) in your AWS account, use the [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation in the AWS KMS API\. You cannot use this operation to view a key policy in a different account\.

The following example uses the [get\-key\-policy](https://docs.aws.amazon.com/cli/latest/reference/kms/get-key-policy.html) command in the AWS Command Line Interface \(AWS CLI\), but you can use any AWS SDK to make this request\. 

Note that the `PolicyName` parameter is required even though `default` is its only valid value\. Also, this command requests the output in text, rather than JSON, to make it easier to view\.

Before running this command, replace the example key ID with a valid one from your account\.

```
$ aws kms get-key-policy --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --policy-name default --output text
```

The response should be similar to the following one, which returns the [default key policy](key-policies.md#key-policy-default)\.

```
{
  "Version" : "2012-10-17",
  "Id" : "key-consolepolicy-3",
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