# Using IAM policies with AWS KMS<a name="iam-policies"></a>

You can use IAM policies, along with [key policies](key-policies.md), to control access to your customer master keys \(CMKs\) in AWS KMS\. 

All CMKs must have a key policy\. IAM policies are optional\. To use an IAM policy to control access to a CMK, the key policy for the CMK must give the account permission to use IAM policies\. Specifically, the key policy must include the [policy statement that enables IAM policies](key-policies.md#key-policy-default-allow-root-enable-iam)\.

IAM policies can control access to any AWS KMS operation\. Unlike key policies, IAM policies can control access to multiple CMKs and provide permissions for the operations of several related AWS services\. But IAM policies are particularly useful for controlling access to operations, such as [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html), that can't be controlled by a key policy because they don't involve any particular CMK\. 

**Note**  
This section explains how to use IAM policies to control access to AWS KMS operations\. For more general information about IAM, see the [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/)\.

**Topics**
+ [Overview of IAM policies](#iam-policies-overview)
+ [Specifying CMKs in IAM policy statements](#cmks-in-iam-policies)
+ [Permissions required to use the AWS KMS console](#console-permissions)
+ [AWS managed \(predefined\) policies for AWS KMS](#aws-managed-policies)
+ [Customer managed policy examples](#customer-managed-policies)

## Overview of IAM policies<a name="iam-policies-overview"></a>

You can use IAM policies in the following ways:
+ **Attach a permissions policy to a user or a group** – You can attach a policy that allows an IAM user or group of users to call AWS KMS operations\.
+ **Attach a permissions policy to a role for federation or cross\-account permissions** – You can attach an IAM policy to an IAM role to enable identity federation, allow cross\-account permissions, or give permissions to applications running on EC2 instances\. For more information about the various use cases for IAM roles, see [IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) in the *IAM User Guide*\.

The following example shows an IAM policy with AWS KMS permissions\. This policy allows the IAM identities to which it is attached to get all CMKs and aliases\.

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

For a table showing all of the AWS KMS API actions and the resources that they apply to, see the [AWS KMS API permissions reference](kms-api-permissions-reference.md)\.

## Specifying CMKs in IAM policy statements<a name="cmks-in-iam-policies"></a>

Some IAM policies control access to AWS KMS operations, such as [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html), that don't involve a particular customer master key \(CMK\)\. However, you can specify one or more CMKs as the resource in an IAM policy statement\.

**Note**  
To use an IAM policy to control access to a CMK, the key policy for the CMK must give the account permission to use IAM policies\. Specifically, the key policy must include the [policy statement that enables IAM policies](key-policies.md#key-policy-default-allow-root-enable-iam)\.

To specify particular CMKs in an IAM policy, use the [key ARN](concepts.md#key-id-key-ARN) of the CMK, that is, the Amazon Resource Name \(ARN\) of the CMK\. You cannot use a [key id](concepts.md#key-id-key-id), [alias name](concepts.md#key-id-alias-name), or [alias ARN](concepts.md#key-id-alias-ARN) to identify a CMK in an IAM policy statement\.

For example, the following IAM policy statement allows the principal to call the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html), [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operations on the CMKs listed in the **Resource** element of the policy statement\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
      "kms:DescribeKey",
      "kms:GenerateDataKey",
      "kms:Decrypt"
    ],
    "Resource": [
      "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
      "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321"
    ]
  }
}
```

To specify multiple CMKs, use a wildcard character \(\*\)\. For example, the following policy statement allows the principal to call the specified operations on any CMK in the example account\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
      "kms:DescribeKey",
      "kms:GenerateDataKey",
      "kms:Decrypt"
    ],
    "Resource": [
      ""arn:aws:kms:*:111122223333:key/*""
    ]
  }
}
```

To determine whether an AWS KMS operation involves a particular CMK, look for the **CMK** value in the **Resources** column of the table in [AWS KMS API permissions: Actions and resources reference](kms-api-permissions-reference.md)\.

## Permissions required to use the AWS KMS console<a name="console-permissions"></a>

To work with the AWS KMS console, users must have a minimum set of permissions that allow them to work with the AWS KMS resources in their AWS account\. In addition to these AWS KMS permissions, users must also have permissions to list IAM users and roles\. If you create an IAM policy that is more restrictive than the minimum required permissions, the AWS KMS console won't function as intended for users with that IAM policy\.

For the minimum permissions required to allow a user read\-only access to the AWS KMS console, see [Allow a user read\-only access to all CMKs through the AWS KMS console](#iam-policy-example-read-only-console)\.

To allow users to work with the AWS KMS console to create and manage CMKs, attach the **AWSKeyManagementServicePowerUser** managed policy to the user, as described in the following section\.

You don't need to allow minimum console permissions for users that are working with the AWS KMS API through the [AWS SDKs](https://aws.amazon.com/tools/#sdk) or [command line tools](https://aws.amazon.com/tools/#cli), though you do need to grant these users permission to use the API\. For more information, see [AWS KMS API permissions reference](kms-api-permissions-reference.md)\.

## AWS managed \(predefined\) policies for AWS KMS<a name="aws-managed-policies"></a>

AWS addresses many common use cases by providing standalone IAM policies that are created and managed by AWS\. These are called *AWS managed policies*\. AWS managed policies provide the necessary permissions for common use cases so you don't have to investigate which permissions are needed\. For more information, see [AWS Managed Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies) in the *IAM User Guide*\.

AWS provides one AWS managed policy for AWS KMS called [AWSKeyManagementServicePowerUser](https://console.aws.amazon.com/iam/home#policies/arn:aws:iam::aws:policy/AWSKeyManagementServicePowerUser)\. This policy allows the following permissions:
+ Allows users to list all CMKs and aliases\.
+ Allows users to retrieve information about each CMK, including its identifiers, creation date, rotation status, key policy, and more\.
+ Allows users to create CMKs that they can administer or use\. When users create a CMK, they can set permissions in the CMK's [key policy](key-policies.md)\. This means users can create CMKs with any permissions they want, including allowing themselves to administer or use the CMK\. The **AWSKeyManagementServicePowerUser** policy does not allow users to administer or use any other CMKs, only the ones they create\.

## Customer managed policy examples<a name="customer-managed-policies"></a>

In this section, you can find example IAM policies that allow permissions for various AWS KMS actions\.

**Important**  
Some of the permissions in the following policies are allowed only when the CMK's key policy also allows them\. For more information, see [AWS KMS API permissions reference](kms-api-permissions-reference.md)\.

**Topics**
+ [Allow a user read\-only access to all CMKs through the AWS KMS console](#iam-policy-example-read-only-console)
+ [Allow a user to encrypt and decrypt with any CMK in a specific AWS account](#iam-policy-example-encrypt-decrypt-one-account)
+ [Allow a user to encrypt and decrypt with any CMK in a specific AWS account and Region](#iam-policy-example-encrypt-decrypt-one-account-one-region)
+ [Allow a user to encrypt and decrypt with specific CMKs](#iam-policy-example-encrypt-decrypt-specific-cmks)
+ [Prevent a user from disabling or deleting any CMKs](#iam-policy-example-deny-disable-delete)

### Allow a user read\-only access to all CMKs through the AWS KMS console<a name="iam-policy-example-read-only-console"></a>

The following policy allows users read\-only access to the AWS KMS console\. That is, users can use the console to view all CMKs, but they cannot make changes to any CMKs or create new ones\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
      "kms:DescribeKey",
      "kms:GetKeyPolicy",
      "kms:GetKeyRotationStatus",
      "kms:GetPublicKey",
      "kms:ListKeys",
      "kms:ListAliases",
      "kms:ListKeyPolicies",      
      "iam:ListUsers",
      "iam:ListRoles"
    ],
    "Resource": "*"
  }
}
```

### Allow a user to encrypt and decrypt with any CMK in a specific AWS account<a name="iam-policy-example-encrypt-decrypt-one-account"></a>

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

### Allow a user to encrypt and decrypt with any CMK in a specific AWS account and Region<a name="iam-policy-example-encrypt-decrypt-one-account-one-region"></a>

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

### Allow a user to encrypt and decrypt with specific CMKs<a name="iam-policy-example-encrypt-decrypt-specific-cmks"></a>

The following policy allows a user to encrypt and decrypt data with the two CMKs specified in the `Resource` element\. When [specifying a CMK in an IAM policy statement](#cmks-in-iam-policies), use the key ARN of the CMK\.

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

### Prevent a user from disabling or deleting any CMKs<a name="iam-policy-example-deny-disable-delete"></a>

The following policy prevents a user from disabling or deleting any CMKs, even when another IAM policy or a key policy allows these permissions\. A policy that explicitly denies permissions overrides all other policies, even those that explicitly allow the same permissions\. For more information, see [Determining Whether a Request is Allowed or Denied](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html#policy-eval-denyallow) in the *IAM User Guide*\.

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