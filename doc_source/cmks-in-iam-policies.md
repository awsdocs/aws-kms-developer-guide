# Specifying CMKs in IAM policy statements<a name="cmks-in-iam-policies"></a>

You can use an IAM policy to allow a principal to use or manage CMKs\. CMKs are specified in the `Resource` element of the policy statement\. 

When writing your policy statements, it's a [best practice](iam-policies-best-practices.md) to limit the CMKs to those that the principals need to use, rather than giving them access to all CMKs\. 
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

You can also use a wildcard character \(`"*"`\) alone in the `Resource` element\. Because it allows access to all CMKs the account has permission to use, it's recommended primarily for operations that don't involve a particular CMK and for `Deny` statements\. You can also use it in policy statements that allow only less sensitive read\-only operations\. To determine whether an AWS KMS operation involves a particular CMK, look for the **CMK** value in the **Resources** column of the table in [AWS KMS permissions](kms-api-permissions-reference.md)\.

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