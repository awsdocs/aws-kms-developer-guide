# AWS managed policy for power users<a name="aws-managed-policies"></a>

You can use the `AWSKeyManagementServicePowerUser` managed policy to give IAM principals in your account the permissions of a power user\. Power users can create KMS keys, use and manage the KMS keys they create, and view all KMS keys and IAM identities\. Principals who have the `AWSKeyManagementServicePowerUser` managed policy can also get permissions from other sources, including key policies, other IAM policies, and grants\. 

`AWSKeyManagementServicePowerUser` is an AWS managed IAM policy\. For more information about AWS managed policies, see [AWS managed policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies) in the *IAM User Guide*\.

**Note**  
Permissions in this policy that are specific to a KMS key, such as `kms:TagResource` and `kms:GetKeyRotationStatus`, are effective only when the key policy for that KMS key [explicitly allows the AWS account to use IAM policies](key-policy-default.md#key-policy-default-allow-root-enable-iam) to control access to the key\. To determine whether a permission is specific to a KMS key, see [AWS KMS permissions](kms-api-permissions-reference.md) and look for a value of **KMS key** in the **Resources** column\.   
This policy gives a power user permissions on any KMS key with a key policy that permits the operation\. For cross\-account permissions, such as `kms:DescribeKey` and `kms:ListGrants`, this might include KMS keys in untrusted AWS accounts\. For details, see [Best practices for IAM policies](iam-policies-best-practices.md) and [Allowing users in other accounts to use a KMS key](key-policy-modifying-external-accounts.md)\. To determine whether a permission is valid on KMS keys in other accounts, see [AWS KMS permissions](kms-api-permissions-reference.md) and look for a value of **Yes** in the **Cross\-account use** column\.   
To allow principals to view the AWS KMS console without errors, the principal needs the [tag:GetResources](https://docs.aws.amazon.com/resourcegroupstagging/latest/APIReference/API_GetResources.html) permission, which is not included in the `AWSKeyManagementServicePowerUser` policy\. You can allow this permission in a separate IAM policy\.

The [AWSKeyManagementServicePowerUser](https://console.aws.amazon.com/iam/home#policies/arn:aws:iam::aws:policy/AWSKeyManagementServicePowerUser) managed IAM policy includes the following permissions\.
+ Allows principals to create KMS keys\. Because this process includes setting the key policy, power users can give themselves and others permission to use and manage the KMS keys they create\.
+ Allows principals to create and delete [aliases](kms-alias.md) and [tags](tagging-keys.md) on all KMS keys\. Changing a tag or alias can allow or deny permission to use and manage the KMS key\. For details, see [ABAC for AWS KMS](abac.md)\.
+ Allows principals to get detailed information about all KMS keys, including their key ARN, cryptographic configuration, key policy, aliases, tags, and [rotation status](rotate-keys.md)\.
+ Allows principals to list IAM users, groups, and roles\.
+ This policy does not allow principals to use or manage KMS keys that they didn't create\. However, they can change aliases and tags on all KMS keys, which might allow or deny them permission to use or manage a KMS key\.

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