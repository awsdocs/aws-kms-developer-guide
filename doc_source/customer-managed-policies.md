# Customer managed policy examples<a name="customer-managed-policies"></a>

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

## Allow a user to view CMKs in the AWS KMS console<a name="iam-policy-example-read-only-console"></a>

The following IAM policy allows users read\-only access to the AWS KMS console\. Users with these permissions can view all CMKs in their AWS account, but they cannot create or change any CMKs\. 

To view CMKs on the **AWS managed keys** and **Customer managed keys** pages, principals require [kms:ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html) and [kms:ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) permissions\. The remaining permissions, particularly [kms:DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html), are required to view optional CMK table columns and data on the CMK detail pages\. The [iam:ListUsers](https://docs.aws.amazon.com/IAM/latest/APIReference/API_ListUsers.html) and [iam:ListRoles](https://docs.aws.amazon.com/IAM/latest/APIReference/API_ListRoles.html) permissions are required to display the key policy in default view without error\. To view data on the **Custom key stores** page and details about CMKs in custom key stores, principals also need [kms:DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) permission\.

If you limit a user's console access to particular CMKs, the console displays an error for each CMK that is not visible\. 

This policy includes of two policy statements\. The `Resource` element in the first policy statement allows the specified permissions on all CMKs in all Regions of the example AWS account\. Console viewers don't need additional access because the AWS KMS console displays only CMKs in the principal's account\. This is true even if they have permission to view CMKs in other AWS accounts\. The remaining AWS KMS and IAM permissions require a `"Resource": "*"` element because they don't apply to any particular CMK\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadOnlyAccessForAllCMKsInAccount",
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
      "Sid": "ReadOnlyAccessForOperationsWithNoCMK",
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

## Allow a user to create CMKs<a name="iam-policy-example-create-key"></a>

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

This IAM policy does not include `kms:PutKeyPolicy` permission or any other permissions that can be set in a key policy\. It's a [best practice](iam-policies-best-practices.md) to set these permissions in the key policy where they apply exclusively to one CMK\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "IAMPermissionsForParticularCMKs",
      "Effect": "Allow",
      "Action": {
        "kms:TagResource"         
      },
      "Resource": "arn:aws:kms:*:111122223333:key/*"
    },
    {
      "Sid": "IAMPermissionsForParticularAliases",
      "Effect": "Allow",
      "Action": {
        "kms:CreateAlias"
      },
      "Resource": "arn:aws:kms:*:111122223333:alias/*"
    },
    {
      "Sid": "IAMPermissionsForAllCMKs",
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

## Allow a user to encrypt and decrypt with any CMK in a specific AWS account<a name="iam-policy-example-encrypt-decrypt-one-account"></a>

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

## Allow a user to encrypt and decrypt with any CMK in a specific AWS account and Region<a name="iam-policy-example-encrypt-decrypt-one-account-one-region"></a>

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

## Allow a user to encrypt and decrypt with specific CMKs<a name="iam-policy-example-encrypt-decrypt-specific-cmks"></a>

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

## Prevent a user from disabling or deleting any CMKs<a name="iam-policy-example-deny-disable-delete"></a>

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