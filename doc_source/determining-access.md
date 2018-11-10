# Determining Access to an AWS KMS Customer Master Key<a name="determining-access"></a>

To determine the full extent of who or what currently has access to a customer master key \(CMK\) in AWS KMS, you must examine the CMK's key policy, all [grants](grants.md) that apply to the CMK, and potentially all AWS Identity and Access Management \(IAM\) policies\. You might do this to determine the scope of potential usage of a CMK, or to help you meet compliance or auditing requirements\. The following topics can help you generate a complete list of the AWS principals \(identities\) that currently have access to a CMK\.

**Topics**
+ [Understanding Policy Evaluation](#policy-evaluation)
+ [Examining the Key Policy](#determining-access-key-policy)
+ [Examining IAM Policies](#determining-access-iam-policies)
+ [Examining Grants](#determining-access-grants)

## Understanding Policy Evaluation<a name="policy-evaluation"></a>

When authorizing access to a CMK, AWS KMS evaluates the following:
+ The key policy that is attached to the CMK
+ All grants that apply to the CMK
+ All IAM policies that are attached to the IAM user or role making the request

In many cases, AWS KMS must evaluate the CMK's key policy and IAM policies together to determine whether access to the CMK is allowed or denied\. To do this, AWS KMS uses a process similar to the one described at [Determining Whether a Request is Allowed or Denied](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html#policy-eval-denyallow) in the *IAM User Guide*\. Remember, though, that IAM policies by themselves are not sufficient to allow access to a KMS CMK\. The CMK's key policy must also allow access\.

For example, assume that you have two CMKs and three users, all in the same AWS account\. The CMKs and users have the following policies:
+ CMK1's key policy [allows access to the AWS account \(root user\) and thereby enables IAM policies to allow access to CMK1](key-policies.md#key-policy-default-allow-root-enable-iam)\.
+ CMK2's key policy allows access to Alice and Charlie\.
+ Alice has no IAM policy\.
+ Bob's IAM policy allows all AWS KMS actions for all CMKs\.
+ Charlie's IAM policy denies all AWS KMS actions for all CMKs\.

Alice cannot access CMK1 because CMK1's key policy does not explicitly allow her access, and she has no IAM policy that allows access\. Alice can access CMK2 because the CMK's key policy explicitly allows her access\.

Bob can access CMK1 because CMK1's key policy enables IAM policies to allow access, and Bob has an IAM policy that allows access\. Bob cannot access CMK2 because the key policy for CMK2 does not allow access to the account, so Bob's IAM policy does not by itself allow access to CMK2\.

Charlie cannot access CMK1 or CMK2 because all AWS KMS actions are denied in his IAM policy\. The explicit deny in Charlie's IAM policy overrides the explicit allow in CMK2's key policy\.

## Examining the Key Policy<a name="determining-access-key-policy"></a>

You can examine the key policy in two ways:
+ If the CMK was created in the AWS Management Console, you can use the console's *default view* on the key details page to view the principals listed in the key policy\. If you can view the key policy in this way, it means the key policy [allows access with IAM policies](key-policies.md#key-policy-default-allow-root-enable-iam)\. Be sure to [examine IAM policies](#determining-access-iam-policies) to determine the complete list of principals that can access the CMK\.
+ You can use the [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation in the AWS KMS API to retrieve a copy of the key policy document, and then examine the document\. You can also view the policy document in the AWS Management Console\.

**Contents**
+ [Examining the Key Policy \(Console\)](#determining-access-key-policy-console)
+ [Examining the Key Policy Document \(KMS API\)](#determining-access-key-policy-document)

### Examining the Key Policy \(Console\)<a name="determining-access-key-policy-console"></a>

Authorized users can view and change the policy document for a customer master key \(CMK\) in the **Key Policy** section of the AWS Management Console\.

**Note**  
AWS KMS recently introduced a new console that makes it easier for you to organize and manage your KMS resources\. It is available in all AWS Regions that AWS KMS supports except for AWS GovCloud \(US\)\. We encourage you to try the new AWS KMS console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.  
The original console will remain available for a brief period to give you time to familiarize yourself with the new one\. To use the original console, choose **Encryption Keys** in the IAM console or go to [https://console\.aws\.amazon\.com/iam/home?\#/encryptionKeys](https://console.aws.amazon.com/iam/home?#/encryptionKeys)\. Please share your feedback by choosing **Feedback** in either console or in the lower\-right corner of this page\.

#### To examine a key policy \(new console\)<a name="determine-access-kms-console"></a>

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. In the list of CMK, choose the alias or key ID of the CMK that you want to examine\.

1. Under **Key policy**, the **Key administrators** section displays the list of IAM users and roles that can manage the CMK\. The **Key users** section lists the users, roles, and AWS accounts that can use this CMK in cryptographic operations\.
**Important**  
The IAM users, roles, and AWS accounts listed here are the ones that have been explicitly granted access in the key policy\. If you use IAM policies to allow access to CMKs, other IAM users and roles might have access to this CMK, even if they are not listed here\. Take care to [examine all IAM policies](#determining-access-iam-policies) in this account to determine whether they allow access to this CMK\.

1. \(Optional\) To view the key policy document, choose **Switch to policy view**\.

#### To examine a key policy \(original console\)<a name="determine-access-iam-console"></a>

1. Sign in to the AWS Management Console and go to [https://console\.aws\.amazon\.com/iam/home?\#/encryptionKeys](https://console.aws.amazon.com/iam/home?#/encryptionKeys)\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. In the list of CMKs, choose the alias of the CMK that you want to examine\.

1. In the **Key Policy** section of the key details page, find the list of IAM users and roles in the **Key Administrators** section, and another list in the **Key Users** section\. The listed users, roles, and AWS accounts all have access to manage or use this CMK\.
**Important**  
The IAM users, roles, and AWS accounts listed here are the ones that have been explicitly granted access in the key policy\. If you use IAM policies to allow access to CMKs, other IAM users and roles might have access to this CMK, even if they are not listed here\. Take care to [examine all IAM policies](#determining-access-iam-policies) in this account to determine if they allow access to this CMK\.

1. \(Optional\) To view the key policy document, choose **Switch to policy view**\.

### Examining the Key Policy Document \(KMS API\)<a name="determining-access-key-policy-document"></a>

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
A key policy document with a statement that allows access to the AWS account \(root user\) enables [IAM policies in the account to allow access to the CMK](key-policies.md#key-policy-default-allow-root-enable-iam)\. This means that IAM users and roles in the account might have access to the CMK even if they are not explicitly listed as principals in the key policy document\. Take care to [examine all IAM policies](#determining-access-iam-policies) in all AWS accounts listed as principals to determine whether they allow access to this CMK\.

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

## Examining IAM Policies<a name="determining-access-iam-policies"></a>

In addition to the key policy and grants, you can also use IAM policies in combination with a CMK's key policy to allow access to a CMK\. For more information about how IAM policies and key policies work together, see [Understanding Policy Evaluation](#policy-evaluation)\.

To determine which principals currently have access to a CMK through IAM policies, you can use the browser\-based [IAM Policy Simulator](https://policysim.aws.amazon.com/) tool, or you can make requests to the IAM API\.

**Contents**
+ [Examining IAM Policies with the IAM Policy Simulator](#determining-access-iam-policy-simulator)
+ [Examining IAM Policies with the IAM API](#determining-access-iam-api)

### Examining IAM Policies with the IAM Policy Simulator<a name="determining-access-iam-policy-simulator"></a>

The IAM Policy Simulator can help you learn which principals have access to a KMS CMK through an IAM policy\.

**To use the IAM Policy Simulator to determine access to a KMS CMK**

1. Sign in to the AWS Management Console and then open the IAM Policy Simulator at [https://policysim.aws.amazon.com/](https://policysim.aws.amazon.com/)\.

1. In the **Users, Groups, and Roles** pane, choose the user, group, or role whose policies you want to simulate\.

1. \(Optional\) Clear the check box next to any policies that you want to omit from the simulation\. To simulate all policies, leave all policies selected\.

1. In the **Policy Simulator** pane, do the following:

   1. For **Select service**, choose **Key Management Service**\.

   1. To simulate specific AWS KMS actions, for **Select actions**, choose the actions to simulate\. To simulate all AWS KMS actions, choose **Select All**\.

1. \(Optional\) The Policy Simulator simulates access to all KMS CMKs by default\. To simulate access to a specific KMS CMK, choose **Simulation Settings**and then type the Amazon Resource Name \(ARN\) of the KMS CMK to simulate\.

1. Choose **Run Simulation**\.

You can view the results of the simulation in the **Results** section\. Repeat steps 2 through 6 for every IAM user, group, and role in the AWS account\.

### Examining IAM Policies with the IAM API<a name="determining-access-iam-api"></a>

You can use the IAM API to examine IAM policies programmatically\. The following steps provide a general overview of how to do this:

1. For each AWS account listed as a principal in the CMK's key policy \(that is, each *root account* listed in this format: `"Principal": {"AWS": "arn:aws:iam::111122223333:root"}`\), use the [ListUsers](https://docs.aws.amazon.com/IAM/latest/APIReference/API_ListUsers.html) and [ListRoles](https://docs.aws.amazon.com/IAM/latest/APIReference/API_ListRoles.html) operations in the IAM API to retrieve a list of every IAM user and role in the account\.

1. For each IAM user and role in the list, use the [SimulatePrincipalPolicy](https://docs.aws.amazon.com/IAM/latest/APIReference/API_SimulatePrincipalPolicy.html) operation in the IAM API, passing in the following parameters:
   + For `PolicySourceArn`, specify the Amazon Resource Name \(ARN\) of a user or role from your list\. You can specify only one `PolicySourceArn` for each `SimulatePrincipalPolicy` API request, so you must call this API multiple times, once for each IAM user and role in your list\.
   + For the `ActionNames` list, specify every AWS KMS API action to simulate\. To simulate all AWS KMS API actions, use `kms:*`\. To test individual AWS KMS API actions, precede each API action with "`kms:`", for example "`kms:ListKeys`"\. For a complete list of all AWS KMS API actions, see [Actions](https://docs.aws.amazon.com/kms/latest/APIReference/API_Operations.html) in the *AWS Key Management Service API Reference*\.
   + \(Optional\) To determine whether the IAM users or roles have access to specific KMS CMKs, use the `ResourceArns` parameter to specify a list of the Amazon Resource Names \(ARNs\) of the CMKs\. To determine whether the IAM users or roles have access to any CMK, do not use the `ResourceArns` parameter\.

IAM responds to each `SimulatePrincipalPolicy` API request with an evaluation decision: `allowed`, `explicitDeny`, or `implicitDeny`\. For each response that contains an evaluation decision of `allowed`, the response includes the name of the specific AWS KMS API operation that is allowed\. It also includes the ARN of the CMK that was used in the evaluation, if any\.

## Examining Grants<a name="determining-access-grants"></a>

Grants are advanced mechanisms for specifying permissions that you or an AWS service integrated with AWS KMS can use to specify how and when a CMK can be used\. Grants are attached to a CMK, and each grant contains the principal who receives permission to use the CMK and a list of operations that are allowed\. Grants are an alternative to the key policy, and are useful for specific use cases\. For more information, see [Using Grants](grants.md)\.

To retrieve a list of grants attached to a CMK, use the AWS KMS [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) API \(or [list\-grants](https://docs.aws.amazon.com/cli/latest/reference/kms/list-grants.html) AWS CLI command\)\. You can examine the grants for a CMK to determine who or what currently has access to use the CMK via those grants\. For example, the following is a JSON representation of a grant that was obtained from the [list\-grants](https://docs.aws.amazon.com/cli/latest/reference/kms/list-grants.html) command in the AWS CLI\.

```
{"Grants": [{
  "Operations": ["Decrypt"],
  "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
  "Name": "0d8aa621-43ef-4657-b29c-3752c41dc132",
  "RetiringPrincipal": "arn:aws:iam::123456789012:root",
  "GranteePrincipal": "arn:aws:sts::111122223333:assumed-role/aws:ec2-infrastructure/i-5d476fab",
  "GrantId": "dc716f53c93acacf291b1540de3e5a232b76256c83b2ecb22cdefa26576a2d3e",
  "IssuingAccount": "arn:aws:iam::111122223333:root",
  "CreationDate": 1.444151834E9,
  "Constraints": {"EncryptionContextSubset": {"aws:ebs:id": "vol-5cccfb4e"}}
}]}
```

To find out who or what has access to use the CMK, look for the `"GranteePrincipal"` element\. In the preceding example, the grantee principal is an assumed role user that is associated with the EC2 instance i\-5d476fab\. The EC2 infrastructure uses this role to attach the encrypted EBS volume vol\-5cccfb4e to the instance\. In this case, the EC2 infrastructure role has permission to use the CMK because you previously created an encrypted EBS volume that is protected by this CMK\. You then attached the volume to an EC2 instance\.

The following is another example of a JSON representation of a grant that was obtained from the [list\-grants](https://docs.aws.amazon.com/cli/latest/reference/kms/list-grants.html) command in the AWS CLI\. In the following example, the grantee principal is another AWS account\.

```
{"Grants": [{
  "Operations": ["Encrypt"],
  "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
  "Name": "",
  "GranteePrincipal": "arn:aws:iam::444455556666:root",
  "GrantId": "f271e8328717f8bde5d03f4981f06a6b3fc18bcae2da12ac38bd9186e7925d11",
  "IssuingAccount": "arn:aws:iam::111122223333:root",
  "CreationDate": 1.444151269E9
}]}
```