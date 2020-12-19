# AWS managed policy for power users<a name="aws-managed-policies"></a>

You can use an [AWS managed policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies) to give IAM principals in your account the permissions of a power user\. Power users can create CMKs, use and manage the CMKs they create, and view all CMKs and IAM identities\.

**Note**  
This policy gives the power user [kms:DescribeKey](https://docs.aws.amazon.com/IAM/latest/APIReference/API_DescribeKey.html) permissions on any CMK with a key policy that permits the operation\. This might include CMKs in untrusted AWS accounts\. For details, see [Best practices for IAM policies](iam-policies-best-practices.md)\.  
This policy gives the power user permission to tag and untag resources and create and delete aliases\. Changing a tag or alias can allow or deny permission to use and manage the CMK\. For details, see [Using ABAC for AWS KMS](abac.md)\.

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