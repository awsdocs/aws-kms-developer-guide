# Using IAM policies with AWS KMS<a name="iam-policies"></a>

You can use IAM policies, along with [key policies](key-policies.md), [grants](grants.md), and [VPC endpoint policies](kms-vpc-endpoint.md#vpce-policy), to control access to your customer master keys \(CMKs\) in AWS KMS\. 

**Note**  
To use an IAM policy to control access to a CMK, the key policy for the CMK must give the account permission to use IAM policies\. Specifically, the key policy must include the [policy statement that enables IAM policies](key-policies.md#key-policy-default-allow-root-enable-iam)\.  
This section explains how to use IAM policies to control access to AWS KMS operations\. For more general information about IAM, see the [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/)\.

All CMKs must have a key policy\. IAM policies are optional\. To use an IAM policy to control access to a CMK, the key policy for the CMK must give the account permission to use IAM policies\. Specifically, the key policy must include the [policy statement that enables IAM policies](key-policies.md#key-policy-default-allow-root-enable-iam)\.

IAM policies can control access to any AWS KMS operation\. Unlike key policies, IAM policies can control access to multiple CMKs and provide permissions for the operations of several related AWS services\. But IAM policies are particularly useful for controlling access to operations, such as [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html), that can't be controlled by a key policy because they don't involve any particular CMK\.

If you access AWS KMS through an Amazon Virtual Private Cloud \(Amazon VPC\) endpoint, you can also use a VPC endpoint policy to limit access to your AWS KMS resources when using the endpoint\. For example, when using the VPC endpoint, you might only allow the principals in your AWS account to access your CMKs\. For details, see [Controlling access to a VPC endpoint](kms-vpc-endpoint.md#vpce-policy) \.

For help writing and formatting a JSON policy document, see the [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) in the *IAM User Guide*\.

**Topics**
+ [Overview of IAM policies](#iam-policies-overview)
+ [Best practices for IAM policies](#iam-policies-best-practices)
+ [Specifying CMKs in IAM policy statements](#cmks-in-iam-policies)
+ [Permissions required to use the AWS KMS console](#console-permissions)
+ [AWS managed policy for power users](#aws-managed-policies)
+ [Customer managed policy examples](#customer-managed-policies)

## Overview of IAM policies<a name="iam-policies-overview"></a>

You can use IAM policies in the following ways:
+ **Attach a permissions policy to a user or a group** – You can attach a policy that allows an IAM user or group of users to call AWS KMS operations\.
+ **Attach a permissions policy to a role for federation or cross\-account permissions** – You can attach an IAM policy to an IAM role to enable identity federation, allow cross\-account permissions, or give permissions to applications running on EC2 instances\. For more information about the various use cases for IAM roles, see [IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) in the *IAM User Guide*\.

The following example shows an IAM policy with AWS KMS permissions\. This policy allows the IAM identities to which it is attached to list all CMKs and aliases\.

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

Like all IAM policies, this policy doesn't have a `Principal` element\. When you attach an IAM policy to an IAM user or IAM role, the user or *assumed role user* gets the permissions specified in the policy\.

For a table showing all of the AWS KMS API actions and the resources that they apply to, see the [AWS KMS API permissions reference](kms-api-permissions-reference.md)\.

## Best practices for IAM policies<a name="iam-policies-best-practices"></a>

Securing access to AWS KMS customer master keys \(CMKs\) is critical to the security of all of your AWS resources\. AWS KMS CMKs are used to protect many of the most sensitive resources in your AWS account\. Take the time to design the [key policies](key-policies.md), IAM policies, [grants](grants.md), and [VPC endpoint policies](kms-vpc-endpoint.md#vpce-policy) that control access to your CMKs\.

In IAM policy statements that control access to CMKs, use the [least privileged principle](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege)\. Give IAM principals only the permissions they need on only the CMKs they must use or manage\.

**Use key policies**  
Whenever possible, provide permissions in key policies that affect one CMK, rather than in an IAM policy that can apply to many CMKs, including those in other AWS accounts\. This is particularly important for sensitive permissions like [kms:PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) and [kms:ScheduleKeyDeletion](https://docs.aws.amazon.com/kms/latest/APIReference/API_ScheduleKeyDeletion.html) but also for cryptographic operations that determine how your data is protected\.

**Limit CreateKey permission**  
Give permission to create keys \([kms:CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html)\) only to principals who need it\. Principals who create a CMK also set its key policy, so they can give themselves and others permission to use and manage the CMKs they create\. When you allow this permission, consider limiting it by using [policy conditions](policy-conditions.md)\. For example, you can use the `kms:CustomerMasterKeySpec` condition to limit the permission to symmetric CMKs\.

**Specify CMKs in an IAM policy**  
As a best practice, specify the [key ARN](concepts.md#key-id-key-ARN) of each CMK to which the permission applies in the `Resource` element of the policy statement\. This practice restricts the permission to the CMKs that principal requires\. For example, this `Resource` element lists only the CMKs the principal needs to use\.  

```
"Resource": [
    "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
    "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321"
]
```
When specifying CMKs is impractical, use a `Resource` value that limits access to CMKs in a trusted AWS account and Region, such as `arn:aws:kms:region:account:key/*`\. Or limit access to CMKs in all Regions \(\*\) of a trusted AWS account, such as `arn:aws:kms:*:account:key/*`\.

**Avoid "Resource": "\*" in an IAM policy**  
Use wildcard characters \(\*\) judiciously\. In a key policy, the wildcard character in the `Resource` element represents the CMK to which the key policy is attached\. But in an IAM policy, a wildcard character alone in the `Resource` element \(`"Resource": "*"`\) applies the permissions to all CMKs in all AWS accounts that the principal's account has permission to use\. This might include [CMKs in other AWS accounts](key-policy-modifying-external-accounts.md), as well as CMKs in the principal's account\.  
For example, to use a CMK in another AWS account, a principal needs permission from the key policy of the CMK in the external account, and from an IAM policy in their own account\. Suppose that an arbitrary account gave your AWS account [kms:Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) permission on their CMKs\. If so, an IAM policy in your account that gives a role `kms:Decrypt` permission on all CMKs \(`"Resource": "*"`\) would satisfy the IAM part of the requirement\. As a result, principals who can assume that role can now decrypt ciphertexts using the CMK in the untrusted account\. Entries for their operations appear in the CloudTrail logs of both accounts\.  
In particular, avoid using `"Resource": "*"` in a policy statement that allows the following API operations\. These operations can be called on CMKs in other AWS accounts\.  
+ [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html)
+ [GetKeyRotationStatus](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyRotationStatus.html)
+ [Cryptographic operations](concepts.md#cryptographic-operations) \([Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html), [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html), [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html), [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html), [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html), [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html)\)
+ [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html), [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html), [ListRetirableGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListRetirableGrants.html), [RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html), [RevokeGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html)

**When to use "Resource": "\*"**  
In an IAM policy, use a wildcard character in the `Resource` element only for permissions that require it\. Only the following permissions require the `"Resource": "*"` element\.  
+ [kms:CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html)
+ [kms:GenerateRandom](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateRandom.html)
+ [kms:ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html)
+ [kms:ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html)
+ Permissions for custom key stores, such as [kms:CreateCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateCustomKeyStore.html) and [kms:ConnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_ConnectCustomKeyStore.html)\.
Permissions for alias operations \([kms:CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html), [kms:UpdateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html), [kms:DeleteAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteAlias.html)\) must be attached to the alias and the CMK\. You can use `"Resource": "*"` in an IAM policy to represent the aliases and the CMKs, or specify the aliases and CMKs in the `Resource` element\. For examples, see [Controlling access to aliases](kms-alias.md#alias-access)\.

 

The examples in this topic provide more information and guidance for designing IAM policies for CMKs\. For general AWS KMS best practice guidance, see the [AWS Key Management Service Best Practices](https://d0.awsstatic.com/whitepapers/aws-kms-best-practices.pdf) whitepaper\.

## Specifying CMKs in IAM policy statements<a name="cmks-in-iam-policies"></a>

You can use an IAM policy to allow a principal to use or manage CMKs\. CMKs are specified in the `Resource` element of the policy statement\. 

When writing your policy statements, it's a [best practice](#iam-policies-best-practices) to limit the CMKs to those that the principals need to use, rather than giving them access to all CMKs\. 
+ To specify particular CMKs in an IAM policy statement, use the [key ARN](concepts.md#key-id-key-ARN) of each CMK\. You cannot use a [key id](concepts.md#key-id-key-id), [alias name](concepts.md#key-id-alias-name), or [alias ARN](concepts.md#key-id-alias-ARN) to identify a CMK in an IAM policy statement\. 

  For example: "`Resource": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab`"
+ To specify multiple CMKs in the account and Region, use wildcard characters \(\*\) in the Region or resource ID positions of the key ARN\. 

  For example, to specify all CMKs in the US West \(Oregon\) Region of an account, use "`Resource": "arn:aws:kms:us-west-2:111122223333:key/*`"\. To specify all CMKs in all Regions of the account, use "`Resource": "arn:aws:kms:*:111122223333:key/*`"\.
+ To represent all CMKs, use a wildcard character alone \(`"*"`\)\. Use this format for operations that don't use any particular CMK, namely [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html), [GenerateRandom](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateRandom.html), [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html), and [ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html)\.

For example, the following IAM policy statement allows the principal to call the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html), [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operations only on the CMKs listed in the `Resource` element of the policy statement\. Specifying CMKs by key ARN, which is a best practice, ensures that the permissions are limited only to the specified CMKs\.

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

To apply the permission to all CMKs in a particular trusted AWS account, you can use wildcard characters \(\*\) in the Region and key ID positions\. For example, the following policy statement allows the principal to call the specified operations on all CMKs in two trusted example accounts\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
      "kms:DescribeKey",
      "kms:GenerateDataKey",
      "kms:GenerateDataKeyPair"
    ],
    "Resource": [
      "arn:aws:kms:*:111122223333:key/*",
      "arn:aws:kms:*:444455556666:key/*"
    ]
  }
}
```

You can also use a wildcard character \(`"*"`\) alone in the `Resource` element\. Because it allows access to all CMKs the account has permission to use, it's recommended primarily for operations that don't involve a particular CMK and for `Deny` statements\. You can also use it in policy statements that allow only less sensitive read\-only operations\. To determine whether an AWS KMS operation involves a particular CMK, look for the **CMK** value in the **Resources** column of the table in [AWS KMS API permissions: Actions and resources reference](kms-api-permissions-reference.md)\.

For example, the following policy statement uses a `Deny` effect to prohibit the principals from using the specified operations on any CMK\. It uses a wildcard character in the `Resource` element to represent all CMKs\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Deny",
    "Action": [
      "kms:CreateKey",
      "kms:PutKeyPolicy",
      "kms:CreateGrant",
      "kms:ScheduleKeyDeletion"
    ],
    "Resource": "*"
  }
}
```

The following policy statement uses a wildcard character alone to represent all CMKs\. But it allows only less sensitive read\-only operations and operations that don't apply to any particular CMK\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
      "kms:CreateKey",
      "kms:ListKeys",
      "kms:ListAliases",
      "kms:ListResourceTags"
    ],
    "Resource": "*"
  }
}
```

## Permissions required to use the AWS KMS console<a name="console-permissions"></a>

To work with the AWS KMS console, users must have a minimum set of permissions that allow them to work with the AWS KMS resources in their AWS account\. In addition to these AWS KMS permissions, users must also have permissions to list IAM users and roles\. If you create an IAM policy that is more restrictive than the minimum required permissions, the AWS KMS console won't function as intended for users with that IAM policy\.

For the minimum permissions required to allow a user read\-only access to the AWS KMS console, see [Allow a user to view CMKs in the AWS KMS console](#iam-policy-example-read-only-console)\.

To allow users to work with the AWS KMS console to create and manage CMKs, attach the **AWSKeyManagementServicePowerUser** managed policy to the user, as described in the following section\.

You don't need to allow minimum console permissions for users that are working with the AWS KMS API through the [AWS SDKs](https://aws.amazon.com/tools/#sdk) or [command line tools](https://aws.amazon.com/tools/#cli)\. However, you do need to grant these users permission to use the API\. For more information, see [AWS KMS API permissions reference](kms-api-permissions-reference.md)\.

## AWS managed policy for power users<a name="aws-managed-policies"></a>

You can use an [AWS managed policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies) to give IAM principals in your account the permissions of a power user\. Power users can create CMKs, use and manage the CMKs they create, and view all CMKs and IAM identities\.

**Note**  
This policy gives the power user [kms:DescribeKey](https://docs.aws.amazon.com/IAM/latest/APIReference/API_DescribeKey.html) permissions on any CMK with a key policy that permits the operation\. This might include CMKs in untrusted AWS accounts\. For details, see [Best practices for IAM policies](#iam-policies-best-practices)\.

The [AWSKeyManagementServicePowerUser](https://console.aws.amazon.com/iam/home#policies/arn:aws:iam::aws:policy/AWSKeyManagementServicePowerUser) managed policy includes the following permissions\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kms:CreateAlias",
                "kms:CreateKey",
                "kms:DeleteAlias",
                "kms:Describe*",
                "kms:GenerateRandom",
                "kms:Get*",
                "kms:List*",
                "kms:TagResource",
                "kms:UntagResource",
                "iam:ListGroups",
                "iam:ListRoles",
                "iam:ListUsers"
            ],
            "Resource": "*"
        }
    ]
}
```
+ Allows users to create CMKs\. Because this process includes setting the key policy, power users can give themselves and others permission to use and manage the CMKs they create\.
+ Allows users to create and delete [aliases](kms-alias.md) and [tags](tagging-keys.md) on all CMKs\.
+ Allows users to get detailed information about all CMKs, including their key ARN, cryptographic configuration, key policy, aliases, tags, and [rotation status](rotate-keys.md)\.
+ Allows users to list IAM users, groups, and roles\.
+ This policy does not give these users permission to use or manage CMKs that they didn't create, although they can manages aliases and tags on all CMKs\.

Users who have the `AWSKeyManagementServicePowerUser` managed policy can also get permissions from other sources, including key policies, other IAM policies, and grants\. 

## Customer managed policy examples<a name="customer-managed-policies"></a>

In this section, you can find example IAM policies that allow permissions for various AWS KMS actions\.

**Important**  
Some of the permissions in the following policies are allowed only when the CMK's key policy also allows them\. For more information, see [AWS KMS API permissions reference](kms-api-permissions-reference.md)\.

For help writing and formatting a JSON policy document, see the [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) in the *IAM User Guide*\.

**Topics**
+ [Allow a user to view CMKs in the AWS KMS console](#iam-policy-example-read-only-console)
+ [Allow a user to create CMKs](#iam-policy-example-create-key)
+ [Allow a user to encrypt and decrypt with any CMK in a specific AWS account](#iam-policy-example-encrypt-decrypt-one-account)
+ [Allow a user to encrypt and decrypt with any CMK in a specific AWS account and Region](#iam-policy-example-encrypt-decrypt-one-account-one-region)
+ [Allow a user to encrypt and decrypt with specific CMKs](#iam-policy-example-encrypt-decrypt-specific-cmks)
+ [Prevent a user from disabling or deleting any CMKs](#iam-policy-example-deny-disable-delete)

### Allow a user to view CMKs in the AWS KMS console<a name="iam-policy-example-read-only-console"></a>

The following IAM policy allows users read\-only access to the AWS KMS console\. Users with these permissions can view all CMKs in their AWS account, but they cannot create or change any CMKs\. 

To view CMKs on the **AWS managed keys** and **Customer managed keys** pages, principals require [kms:ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html) and [kms:ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) permissions\. The remaining permissions, particularly [kms:DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html), are required to view optional CMK table columns and data on the CMK detail pages\. The [iam:ListUsers](https://docs.aws.amazon.com/IAM/latest/APIReference/API_ListUsers.html) and [iam:ListRoles](https://docs.aws.amazon.com/IAM/latest/APIReference/API_ListRoles.html) permissions are required to display the key policy in default view without error\. To view data on the **Custom key stores** page and details about CMKs in custom key stores, principals also need [kms:DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) permission\.

If you limit a user's console access to particular CMKs, the console displays an error for each CMK that is not visible\. 

This policy includes of two policy statements\. The `Resource` element in the first policy statement allows the specified permissions on all CMKs in all Regions of the example AWS account\. Console viewers don't need additional access because the AWS KMS console displays only CMKs in the principal's account\. This is true even if they have permission to view CMKs in other AWS accounts\. The remaining AWS KMS and IAM permissions require a `"Resource": "*"` element because they don't apply to any particular CMK\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Allow read-only permission to all CMKs in the account",
      "Effect": "Allow",
      "Action": [
        "kms:GetPublicKey",        
        "kms:GetKeyRotationStatus",
        "kms:GetKeyPolicy",
        "kms:DescribeKey",
        "kms:ListKeyPolicies",
        "kms:ListResourceTags"
      ],
      "Resource": "arn:aws:kms:*:111122223333:key/*"
    },
    {
      "Sid": "Allow read-only access to operations with no CMK resource",
      "Effect": "Allow",
      "Action": [
        "kms:ListKeys",
        "kms:ListAliases",
        "iam:ListRoles",
        "iam:ListUsers"
      ],
      "Resource": "*"
    }
  ]
}
```

### Allow a user to create CMKs<a name="iam-policy-example-create-key"></a>

The following IAM policy allows a user to create CMKs\. The value of the `Resource` element is `*` because the `CreateKey` operation does not use any particular AWS KMS resources \(CMKs or aliases\)\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": "kms:CreateKey",
    "Resource": "*"
  }
}
```

Principals who create keys might need some related permissions\.
+ **kms:PutKeyPolicy** — Principals who have `kms:CreateKey` permission can set the initial key policy for the CMK\. However, the `CreateKey` caller must have [kms:PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) permission, which lets them change the CMK's key policy, or they must specify the `BypassPolicyLockoutSafetyCheck` parameter of `CreateKey`, which is not recommended\. The `CreateKey` caller can get `kms:PutKeyPolicy` permission for the CMK from an IAM policy or they can include this permission in the key policy of the CMK that they're creating\.
+ **kms:TagResource** — To add tags to the CMK during the `CreateKey` operation, the `CreateKey` caller must have [kms:TagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) permission in an IAM policy\. Including this permission in the key policy of the new CMK isn't sufficient\. However, if the `CreateKey` caller includes `kms:TagResource` in the initial key policy, they can add tags in a separate call after the CMK is created\.
+ **kms:CreateAlias** — Principals who create a CMK in the AWS KMS console must have [kms:CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html) permission on the CMK and on the alias\. \(The console makes two calls; one to `CreateKey` and one to `CreateAlias`\)\. You must provide the alias permission in an IAM policy\. You can provide the CMK permission in a key policy or IAM policy\. For details, see [Controlling access to aliases](kms-alias.md#alias-access)\.

In addition to `kms:CreateKey`, the following IAM policy provides `kms:TagResource` permission on all CMKs in the AWS account and `kms:CreateAlias` permission on all aliases that the account\. It also includes some useful read\-only permissions that can be provided only in an IAM policy\. 

This IAM policy does not include `kms:PutKeyPolicy` permission or any other permissions that can be set in a key policy\. It's a [best practice](#iam-policies-best-practices) to set these permissions in the key policy where they apply exclusively to one CMK\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "IAM permissions for particular CMKs",
      "Effect": "Allow",
      "Action": {
        "kms:TagResource"         
      },
      "Resource": "arn:aws:kms:*:111122223333:key/*"
    },
    {
      "Sid": "IAM permissions for particular aliases",
      "Effect": "Allow",
      "Action": {
        "kms:CreateAlias"
      },
      "Resource": "arn:aws:kms:*:111122223333:alias/*"
    },
    {
      "Sid": "IAM permission that must be set for all CMKs",
      "Effect": "Allow",
      "Action": [
        "kms:CreateKey",
        "kms:ListKeys",
        "kms:ListAliases"
      ],
      "Resource": "*"
    }
  ]
}
```

### Allow a user to encrypt and decrypt with any CMK in a specific AWS account<a name="iam-policy-example-encrypt-decrypt-one-account"></a>

The following IAM policy allows a user to encrypt and decrypt data with any CMK in AWS account 111122223333\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
      "kms:Encrypt",
      "kms:Decrypt"
    ],
    "Resource": "arn:aws:kms:*:111122223333:key/*"
  }
}
```

### Allow a user to encrypt and decrypt with any CMK in a specific AWS account and Region<a name="iam-policy-example-encrypt-decrypt-one-account-one-region"></a>

The following IAM policy allows a user to encrypt and decrypt data with any CMK in AWS account 111122223333 in the US West \(Oregon\) Region\.

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

The following IAM policy allows a user to encrypt and decrypt data with the two CMKs specified in the `Resource` element\. When specifying a CMK in an IAM policy statement, you must use the [key ARN](concepts.md#key-id-key-ARN) of the CMK\.

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

The following IAM policy prevents a user from disabling or deleting any CMKs, even when another IAM policy or a key policy allows these permissions\. A policy that explicitly denies permissions overrides all other policies, even those that explicitly allow the same permissions\. For more information, see [Troubleshooting key access](policy-evaluation.md)\.

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