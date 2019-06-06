# Examining the Key Policy<a name="determining-access-key-policy"></a>

You can examine the key policy in two ways:
+ If the CMK was created in the AWS Management Console, you can use the console's *default view* on the key details page to view the principals listed in the key policy\. If you can view the key policy in this way, it means the key policy [allows access with IAM policies](key-policies.md#key-policy-default-allow-root-enable-iam)\. Be sure to [examine IAM policies](determining-access-iam-policies.md) to determine the complete list of principals that can access the CMK\.
+ You can use the [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation in the AWS KMS API to retrieve a copy of the key policy document, and then examine the document\. You can also view the policy document in the AWS Management Console\.

**Contents**
+ [Examining the Key Policy \(Console\)](#determining-access-key-policy-console)
+ [Examining the Key Policy Document \(KMS API\)](#determining-access-key-policy-document)

## Examining the Key Policy \(Console\)<a name="determining-access-key-policy-console"></a>

Authorized users can view and change the policy document for a customer master key \(CMK\) in the **Key Policy** section of the AWS Management Console\.

**Note**  
AWS KMS recently introduced a new console that makes it easier for you to organize and manage your KMS resources\. We encourage you to try it at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\. Please share your feedback by choosing **Feedback** in either console or in the lower\-right corner of this page\.  
The original console will remain available for a brief period to give you time to familiarize yourself with the new one\. To use the original console, go to [https://console\.aws\.amazon\.com/iam/home\#encryptionKeys](https://console.aws.amazon.com/iam/home#encryptionKeys)\.

### To examine a key policy \(new console\)<a name="determine-access-kms-console"></a>

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. In the list of CMK, choose the alias or key ID of the CMK that you want to examine\.

1. Under **Key policy**, the **Key administrators** section displays the list of IAM users and roles that can manage the CMK\. The **Key users** section lists the users, roles, and AWS accounts that can use this CMK in cryptographic operations\.
**Important**  
The IAM users, roles, and AWS accounts listed here are the ones that have been explicitly granted access in the key policy\. If you use IAM policies to allow access to CMKs, other IAM users and roles might have access to this CMK, even if they are not listed here\. Take care to [examine all IAM policies](determining-access-iam-policies.md) in this account to determine whether they allow access to this CMK\.

1. \(Optional\) To view the key policy document, choose **Switch to policy view**\.

### To examine a key policy \(original console\)<a name="determine-access-iam-console"></a>

1. Sign in to the AWS Management Console and go to [https://console\.aws\.amazon\.com/iam/home\#encryptionKeys](https://console.aws.amazon.com/iam/home#encryptionKeys)\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. In the list of CMKs, choose the alias of the CMK that you want to examine\.

1. In the **Key Policy** section of the key details page, find the list of IAM users and roles in the **Key Administrators** section, and another list in the **Key Users** section\. The listed users, roles, and AWS accounts all have access to manage or use this CMK\.
**Important**  
The IAM users, roles, and AWS accounts listed here are the ones that have been explicitly granted access in the key policy\. If you use IAM policies to allow access to CMKs, other IAM users and roles might have access to this CMK, even if they are not listed here\. Take care to [examine all IAM policies](determining-access-iam-policies.md) in this account to determine if they allow access to this CMK\.

1. \(Optional\) To view the key policy document, choose **Switch to policy view**\.

## Examining the Key Policy Document \(KMS API\)<a name="determining-access-key-policy-document"></a>

You can view the key policy document in a couple of ways:
+ Use the key details page of the AWS Management Console \(see the preceding section for instructions\)\.
+ To get the key policy document, use the [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation in the AWS KMS API\.

Examine the key policy document and take note of all principals specified in each policy statement's `Principal` element\. The IAM users, IAM roles, and AWS accounts in the `Principal` elements are those that have access to this CMK\.

The following examples use the policy statements found in the [default key policy](key-policies.md#key-policy-default) to demonstrate how to do this\.

**Example Policy Statement 1**  

```
{
  "Sid": "Enable IAM User Permissions",
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::111122223333:root"},
  "Action": "kms:*",
  "Resource": "*"
}
```
In the preceding policy statement, `arn:aws:iam::111122223333:root` refers to the AWS account 111122223333\. By default, a policy statement like this one is present in the key policy document when you create a new CMK with the console\. It is also present when you create a new CMK programmatically but do not provide a key policy\.  
A key policy document with a statement that allows access to the AWS account \(root user\) enables [IAM policies in the account to allow access to the CMK](key-policies.md#key-policy-default-allow-root-enable-iam)\. This means that IAM users and roles in the account might have access to the CMK even if they are not explicitly listed as principals in the key policy document\. Take care to [examine all IAM policies](determining-access-iam-policies.md) in all AWS accounts listed as principals to determine whether they allow access to this CMK\.

**Example Policy Statement 2**  

```
{
  "Sid": "Allow access for Key Administrators",
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::111122223333:user/KMSKeyAdmin"},
  "Action": [
    "kms:Describe*",
    "kms:Put*",
    "kms:Create*",
    "kms:Update*",
    "kms:Enable*",
    "kms:Revoke*",
    "kms:List*",
    "kms:Disable*",
    "kms:Get*",
    "kms:Delete*",
    "kms:ScheduleKeyDeletion",
    "kms:CancelKeyDeletion"
  ],
  "Resource": "*"
}
```
In the preceding policy statement, `arn:aws:iam::111122223333:user/KMSKeyAdmin` refers to the IAM user named KMSKeyAdmin in AWS account 111122223333\. This user is allowed to perform the actions listed in the policy statement, which are the administrative actions for managing a CMK\.

**Example Policy Statement 3**  

```
{
  "Sid": "Allow use of the key",
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::111122223333:role/EncryptionApp"},
  "Action": [
    "kms:DescribeKey",
    "kms:GenerateDataKey*",
    "kms:Encrypt",
    "kms:ReEncrypt*",
    "kms:Decrypt"
  ],
  "Resource": "*"
}
```
In the preceding policy statement, `arn:aws:iam::111122223333:role/EncryptionApp` refers to the IAM role named EncryptionApp in AWS account 111122223333\. Principals that can assume this role are allowed to perform the actions listed in the policy statement, which are the cryptographic actions for encrypting and decrypting data with a CMK\.

**Example Policy Statement 4**  

```
{
  "Sid": "Allow attachment of persistent resources",
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::111122223333:role/EncryptionApp"},
  "Action": [
    "kms:ListGrants",
    "kms:CreateGrant",
    "kms:RevokeGrant"
  ],
  "Resource": "*",
  "Condition": {"Bool": {"kms:GrantIsForAWSResource": true}}
}
```
In the preceding policy statement, `arn:aws:iam::111122223333:role/EncryptionApp` refers to the IAM role named EncryptionApp in AWS account 111122223333\. Principals that can assume this role are allowed to perform the actions listed in the policy statement\. These actions, when combined with the actions allowed in **Example policy statement 3**, are those necessary to delegate use of the CMK to most [AWS services that integrate with AWS KMS](service-integration.md), specifically the services that use [grants](grants.md)\. The `Condition` element ensures that the delegation is allowed only when the delegate is an AWS service that integrates with AWS KMS and uses grants for authorization\.

To learn all the different ways you can specify a principal in a key policy document, see [Specifying a Principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html#Principal_specifying) in the *IAM User Guide*\.

To learn more about AWS KMS key policies, see [Using Key Policies in AWS KMS](key-policies.md)\.