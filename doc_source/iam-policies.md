# Using IAM Policies with AWS KMS<a name="iam-policies"></a>

You can use IAM policies in combination with [key policies](key-policies.md) to control access to your customer master keys \(CMKs\) in AWS KMS\.

**Note**  
This section discusses using IAM in the context of AWS KMS\. It doesn't provide detailed information about the IAM service\. For complete IAM documentation, see the [IAM User Guide](http://docs.aws.amazon.com/IAM/latest/UserGuide/)\.

Policies attached to IAM identities \(that is, users, groups, and roles\) are called *identity\-based policies* \(or *IAM policies*\), and policies attached to resources outside of IAM are called *resource\-based policies*\. In AWS KMS, you must attach resource\-based policies to your CMKs\. These are called *key policies*\. All KMS CMKs have a key policy, and you must use it to control access to a CMK\. IAM policies by themselves are not sufficient to allow access to a CMK, though you can use them in combination with a CMK's key policy\. To do so, ensure that CMK's key policy includes the [policy statement that enables IAM policies](key-policies.md#key-policy-default-allow-root-enable-iam)\.

**Topics**
+ [Overview of IAM Policies](#iam-policies-overview)
+ [Permissions Required to Use the AWS KMS Console](#console-permissions)
+ [AWS Managed \(Predefined\) Policies for AWS KMS](#aws-managed-policies)
+ [Customer Managed Policy Examples](#customer-managed-policies)

## Overview of IAM Policies<a name="iam-policies-overview"></a>

You can use IAM policies in the following ways:
+ **Attach a permissions policy to a user or a group** – You can attach a policy that allows an IAM user or group of users to, for example, create new CMKs\.
+ **Attach a permissions policy to a role for federation or cross\-account permissions** – You can attach an IAM policy to an IAM role to enable identity federation, allow cross\-account permissions, or give permissions to applications running on EC2 instances\. For more information about the various use cases for IAM roles, see [IAM Roles](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) in the *IAM User Guide*\.

The following example shows an IAM policy with AWS KMS permissions\. This policy allows the IAM identities to which it is attached to retrieve a list of all CMKs and aliases\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
      "kms:ListKeys",
      "kms:ListAliases"
    ],
    "Resource": "*"
  }
}
```

This policy doesn't specify the `Principal` element because in IAM policies you don't specify the principal who gets the permissions\. When you attach this policy to an IAM user, that user is the implicit principal\. When you attach this policy to an IAM role, the *assumed role user* gets the permissions\.

For a table showing all of the AWS KMS API actions and the resources that they apply to, see the [AWS KMS API Permissions Reference](kms-api-permissions-reference.md)\.

## Permissions Required to Use the AWS KMS Console<a name="console-permissions"></a>

To work with the AWS KMS console, users must have a minimum set of permissions that allow them to work with the AWS KMS resources in their AWS account\. In addition to these AWS KMS permissions, users must also have permissions to list IAM users and roles\. If you create an IAM policy that is more restrictive than the minimum required permissions, the AWS KMS console won't function as intended for users with that IAM policy\.

For the minimum permissions required to allow a user read\-only access to the AWS KMS console, see [Allow a User Read\-Only Access to All CMKs through the AWS KMS Console](#iam-policy-example-read-only-console)\.

To allow users to work with the AWS KMS console to create and manage CMKs, attach the **AWSKeyManagementServicePowerUser** managed policy to the user, as described in the following section\.

You don't need to allow minimum console permissions for users that are working with the AWS KMS API through the [AWS SDKs](https://aws.amazon.com/tools/#sdk) or [command line tools](https://aws.amazon.com/tools/#cli), though you do need to grant these users permission to use the API\. For more information, see [AWS KMS API Permissions Reference](kms-api-permissions-reference.md)\.

## AWS Managed \(Predefined\) Policies for AWS KMS<a name="aws-managed-policies"></a>

AWS addresses many common use cases by providing standalone IAM policies that are created and managed by AWS\. These are called *AWS managed policies*\. AWS managed policies provide the necessary permissions for common use cases so you don't have to investigate which permissions are needed\. For more information, see [AWS Managed Policies](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies) in the *IAM User Guide*\.

AWS provides one AWS managed policy for AWS KMS called [AWSKeyManagementServicePowerUser](https://console.aws.amazon.com/iam/home#policies/arn:aws:iam::aws:policy/AWSKeyManagementServicePowerUser)\. This policy allows the following permissions:
+ Allows users to list all CMKs and aliases\.
+ Allows users to retrieve information about each CMK, including its identifiers, creation date, rotation status, key policy, and more\.
+ Allows users to create CMKs that they can administer or use\. When users create a CMK, they can set permissions in the CMK's [key policy](key-policies.md)\. This means users can create CMKs with any permissions they want, including allowing themselves to administer or use the CMK\. The **AWSKeyManagementServicePowerUser** policy does not allow users to administer or use any other CMKs, only the ones they create\.

## Customer Managed Policy Examples<a name="customer-managed-policies"></a>

In this section, you can find example IAM policies that allow permissions for various AWS KMS actions\.

**Important**  
Some of the permissions in the following policies are allowed only when the CMK's key policy also allows them\. For more information, see [AWS KMS API Permissions Reference](kms-api-permissions-reference.md)\.

**Topics**
+ [Allow a User Read\-Only Access to All CMKs through the AWS KMS Console](#iam-policy-example-read-only-console)
+ [Allow a User to Encrypt and Decrypt with Any CMK in a Specific AWS Account](#iam-policy-example-encrypt-decrypt-one-account)
+ [Allow a User to Encrypt and Decrypt with Any CMK in a Specific AWS Account and Region](#iam-policy-example-encrypt-decrypt-one-account-one-region)
+ [Allow a User to Encrypt and Decrypt with Specific CMKs](#iam-policy-example-encrypt-decrypt-specific-cmks)
+ [Prevent a User from Disabling or Deleting Any CMKs](#iam-policy-example-deny-disable-delete)

### Allow a User Read\-Only Access to All CMKs through the AWS KMS Console<a name="iam-policy-example-read-only-console"></a>

The following policy allows users read\-only access to the AWS KMS console\. That is, users can use the console to view all CMKs, but they cannot make changes to any CMKs or create new ones\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
      "kms:ListKeys",
      "kms:ListAliases",
      "kms:DescribeKey",
      "kms:ListKeyPolicies",
      "kms:GetKeyPolicy",
      "kms:GetKeyRotationStatus",
      "iam:ListUsers",
      "iam:ListRoles"
    ],
    "Resource": "*"
  }
}
```

### Allow a User to Encrypt and Decrypt with Any CMK in a Specific AWS Account<a name="iam-policy-example-encrypt-decrypt-one-account"></a>

The following policy allows a user to successfully request that AWS KMS encrypt and decrypt data with any CMK in AWS account 111122223333\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
      "kms:Encrypt",
      "kms:Decrypt"
    ],
    "Resource": [
      "arn:aws:kms:*:111122223333:key/*"
    ]
  }
}
```

### Allow a User to Encrypt and Decrypt with Any CMK in a Specific AWS Account and Region<a name="iam-policy-example-encrypt-decrypt-one-account-one-region"></a>

The following policy allows a user to successfully request that AWS KMS encrypt and decrypt data with any CMK in AWS account 111122223333 in the US West \(Oregon\) region\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
      "kms:Encrypt",
      "kms:Decrypt"
    ],
    "Resource": [
      "arn:aws:kms:us-west-2:111122223333:key/*"
    ]
  }
}
```

### Allow a User to Encrypt and Decrypt with Specific CMKs<a name="iam-policy-example-encrypt-decrypt-specific-cmks"></a>

The following policy allows a user to successfully request that AWS KMS encrypt and decrypt data with the two CMKs specified in the policy's `Resource` element\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
      "kms:Encrypt",
      "kms:Decrypt"
    ],
    "Resource": [
      "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
      "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321"
    ]
  }
}
```

### Prevent a User from Disabling or Deleting Any CMKs<a name="iam-policy-example-deny-disable-delete"></a>

The following policy prevents a user from disabling or deleting any CMKs, even when another IAM policy or a key policy allows these permissions\. A policy that explicitly denies permissions overrides all other policies, even those that explicitly allow the same permissions\. For more information, see [Determining Whether a Request is Allowed or Denied](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html#policy-eval-denyallow) in the *IAM User Guide*\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Deny",
    "Action": [
      "kms:DisableKey",
      "kms:ScheduleKeyDeletion"
    ],
    "Resource": "*"
  }
}
```