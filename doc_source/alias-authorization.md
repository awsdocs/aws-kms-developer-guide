# Using aliases to control access to CMKs<a name="alias-authorization"></a>

You can control access to AWS KMS customer master keys \(CMKs\) based on the aliases that are associated with the CMK\. To do so, use the [kms:RequestAlias](policy-conditions.md#conditions-kms-request-alias) and [kms:ResourceAliases](policy-conditions.md#conditions-kms-resource-aliases) condition keys\. This feature is part of AWS KMS support for [attribute\-based access control](abac.md) \(ABAC\)\.

The `kms:RequestAlias` condition key allows or denies access to a CMK based on the alias in a request\. The `kms:ResourceAliases` condition key allows or denies access to a CMK based on the aliases associated with the CMK\. 

These features do not allow you to identify a CMK by using an alias in the `resource` element of a policy statement\. When an alias is the value of a `resource` element, the policy applies to the alias resource, not to any CMK that might be associated with it\.

**Note**  
It might take up to five minutes for tag and alias changes to affect CMK authorization\. Recent changes might be visible in API operations before they affect authorization\.

When using aliases to control access to CMKs, consider the following:
+ Use aliases to reinforce the best practice of [least privileged access](iam-policies-best-practices.md)\. Give IAM principals only the permissions that they need for only the CMKs that they must use or manage\. For example, use aliases to identify the CMKs used for a project\. Then give the project team permission to use only CMKs with the project aliases\. 
+ Be cautious about giving principals the `kms:CreateAlias`, `kms:UpdateAlias`, or `kms:DeleteAlias` permissions that let them add, edit, and delete aliases\. When you use aliases to control access to CMKs, changing an alias can give principals permission to use CMKs that they didn't otherwise have permission to use\. It can also deny access to CMKs that other principals require to do their jobs\. 
+ Review the principals in your AWS account that currently have permission to manage aliases and adjust the permissions, if necessary\. Key administrators who don't have permission to change key policies or create grants can control access to CMKs if they have permission to manage aliases\. 

  For example, the console [default key policy for key administrators](key-policies.md#key-policy-default-allow-administrators) includes `kms:CreateAlias`, `kms:DeleteAlias`, and `kms:UpdateAlias` permission\. IAM policies might give alias permissions for all CMKs in your AWS account\. For example, the [AWSKeyManagementServicePowerUser](https://console.aws.amazon.com/iam/home#policies/arn:aws:iam::aws:policy/AWSKeyManagementServicePowerUser) managed policy allows principals to create, delete, and list aliases for all CMKs but not update them\.
+ Before setting a policy that depends on an alias, review the aliases on the CMKs in your AWS account\. Make sure that your policy applies only to the aliases that you intend to include\. Use [CloudTrail logs](alias-ct.md) and [CloudWatch alarms](monitoring-cloudwatch.md) to alert you to alias changes that might affect access to your CMKs\. Also, the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) response includes the creation date and last updated date for each alias\.
+ The alias policy conditions use pattern matching; they aren't tied to a particular instance of an alias\. A policy that uses alias\-based condition keys affects all new and existing aliases that match the pattern\. If you delete and recreate an alias that matches a policy condition, the condition applies to the new alias, just as it did to the old one\. 

The `kms:RequestAlias` condition key relies on the alias specified explicitly in an operation request\. The `kms:ResourceAliases` condition key depends on the aliases that are associated with a CMK, even if they don't appear in the request\.

## kms:RequestAlias<a name="alias-auth-request-alias"></a>

Allow or deny access to a CMK based on the alias that identifies the CMK in a request\. You can use the [kms:RequestAlias](policy-conditions.md#conditions-kms-request-alias) condition key in a [key policy](key-policies.md) or IAM policy\. It applies to operations that use an alias to identify a CMK in a request, namely [cryptographic operations](concepts.md#cryptographic-operations), [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html), and [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html)\. It is not valid for alias operations, such as [CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html) or [DeleteAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteAlias.html)\.

In the condition key, specify an [alias name](concepts.md#key-id-alias-name) or alias name pattern\. You cannot specify an [alias ARN](concepts.md#key-id-alias-ARN)\.

For example, the following key policy statement allows principals to use the specified operations on the CMK\. The permission is effective only when the request uses an alias that includes `alpha` to identify the CMK\.

```
{
  "Sid": "Key policy using a request alias condition",
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/alpha-developer"
  },
  "Action": [
    "kms:Decrypt",
    "kms:GenerateDataKey*",
    "kms:DescribeKey"
  ],
  "Resource": "*",
  "Condition": {
    "StringLike": {
      "kms:RequestAlias": "alias/*alpha*"
    }
  }
}
```

The following example request from an authorized principal would fulfill the condition\. However, a request that used a [key ID](concepts.md#key-id-key-id), a [key ARN](concepts.md#key-id-key-ARN), or a different alias would not fulfill the condition, even if these values identified the same CMK\.

```
$ aws kms describe-key --key-id "arn:aws:kms:us-west-2:111122223333:alias/project-alpha"
```

## kms:ResourceAliases<a name="alias-auth-resource-aliases"></a>

Allow or deny access to a CMK based on the aliases associated with the CMK, even if the alias isn't used in a request\. The [kms:ResourceAliases](policy-conditions.md#conditions-kms-resource-aliases) condition key lets you specify an alias or alias pattern, such as `alias/test*`, so you can use it in an IAM policy to control access to several CMKs in the same Region\. It's valid for any AWS KMS operation that uses a CMK\. 

For example, the following IAM policy lets the principals manage automatic key rotation on the CMKs in two AWS accounts\. However, the permission applies only to CMKs with aliases that begin with `restricted`\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AliasBasedIAMPolicy",
      "Effect": "Allow",
      "Action": [
        "kms:EnableKeyRotation",
        "kms:DisableKeyRotation",
        "kms:GetKeyRotationStatus"
      ],
      "Resource": [
        "arn:aws:kms:*:111122223333:key/*",
        "arn:aws:kms:*:444455556666:key/*"
      ],
      "Condition": {
        "ForAnyValue:StringLike": {
          "kms:ResourceAliases": "alias/restricted*"
        }
      }
    }
  ]
}
```

The `kms:ResourceAliases` condition is a condition of the resource, not the request\. As such, a request that doesn't specify the alias can still satisfy the condition\. 

The following example request, which specifies a matching alias, satisfies the condition\. 

```
$ aws kms enable-key-rotation --key-id "alias/restricted-project"
```

However, the following example request also satisfies the condition, provided that the specified CMK has an alias that begins with `restricted`, even if that alias isn't used in the request\.

```
$ aws kms enable-key-rotation --key-id "1234abcd-12ab-34cd-56ef-1234567890ab"
```