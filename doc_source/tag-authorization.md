# Using tags to control access to CMKs<a name="tag-authorization"></a>

You can control access to AWS KMS customer master keys \(CMKs\) based on the tags on the CMK\. For example, you can write an IAM policy that allows principals to enable and disable only the CMKs with a particular tag\. Or, you can use an IAM policy to prevent principals from using CMKs in cryptographic operations unless the CMK has a particular tag\. 

This feature is part of AWS KMS support for [attribute\-based access control](abac.md) \(ABAC\)\. For information about using tags to control access to AWS resources, see [What is ABAC for AWS?](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_attribute-based-access-control.html) and [Controlling Access to AWS Resources Using Resource Tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_tags.html) in the *IAM User Guide*\. For help resolving access issues related to ABAC, see [Troubleshooting ABAC for AWS KMS](abac.md#troubleshooting-tags-aliases)\.

**Note**  
It might take up to five minutes for tag and alias changes to affect CMK authorization\. Recent changes might be visible in API operations before they affect authorization\.

AWS KMS supports the use of the [aws:ResourceTag](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-resourcetag) [global condition context key](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html), which lets you control access to CMKs based on the tags on the CMK\. Because multiple CMKs can have the same tag, this feature lets you apply the permission to a select set of CMKs\. You can also easily change the CMKs in the set by changing their tags\. 

In AWS KMS, the `aws:ResourceTag` condition key is supported only in IAM policies\. It's designed to help you control access to multiple CMKs with the same tags\. It isn't supported in key policies, which apply only to one CMK\. Also, resource conditions apply only to operations that use an existing resource\. As such, you cannot use `aws:ResourceTag` to control access to operations like [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html), [ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html), or [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html)\.

Controlling access with tags provides a simple, scalable, and flexible way to manage permissions\. However, if not properly designed and managed, it can allow or deny access to your CMKs inadvertently\. If you are using tags to control access, consider the following practices\.
+ Use tags to reinforce the best practice of [least privileged access](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege)\. Give IAM principals only the permissions they need on only the CMKs they must use or manage\. For example, use tags to label the CMKs used for a project\. Then give the project team permission to use only CMKs with the project tag\.
+ Be cautious about giving principals the `kms:TagResource` and `kms:UntagResource` permissions that let them add, edit, and delete tags\. When you use tags to control access to CMKs, changing a tag can give principals permission to use CMKs that they didn't otherwise have permission to use\. It can also deny access to CMKs that other principals require to do their jobs\. Key administrators who don't have permission to change key policies or create grants can control access to CMKs if they have permission to manage tags\. 

  Whenever possible, use a policy condition, such as `aws:RequestTag` or `aws:TagKeys` to [limit a principal's tagging permissions](tag-permissions.md#tag-permissions-conditions) to particular tags or tag patterns on particular CMKs\.
+ Review the principals in your AWS account that currently have tagging and untagging permissions and adjust them, if necessary\. For example, the console [default key policy for key administrators](key-policies.md#key-policy-default-allow-administrators) includes `kms:TagResource` and `kms:UntagResource` permission on that CMK\. IAM policies might give tag and untag permissions on all CMKs\. For example, the [AWSKeyManagementServicePowerUser](aws-managed-policies.md) managed policy allows principals to tag, untag, and list tags on all CMKs\.
+ Before setting a policy that depends on a tag, review the tags on the CMKs in your AWS account\. Make sure that your policy applies only to the tags you intend to include\. Use [CloudTrail logs](logging-using-cloudtrail.md) and [CloudWatch alarms](monitoring-overview.md) to alert you to tag changes that might affect access to your CMKs\.
+ The tag\-based policy conditions use pattern matching; they aren't tied to a particular instance of a tag\. A policy that uses tag\-based condition keys affects all new and existing tags that match the pattern\. If you delete and recreate a tag that matches a policy condition, the condition applies to the new tag, just as it did to the old one\.

For example, the following IAM policy allows the principals to call the [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operations only on CMKs in your account that are the Asia Pacific \(Singapore\) Region and have a `"Project"="Alpha"` tag\. You might attach this policy to roles in the example Alpha project\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "IAMPolicyWithResourceTag",
      "Effect": "Allow",
      "Action": [
          "kms:GenerateDataKeyWithoutPlaintext",
          "kms:Decrypt"
      ]
      "Resource": "arn:aws:kms:ap-southeast-1:111122223333:key/*",
      "Condition": {
          "StringEquals": {"aws:ResourceTag/Project": "Alpha"}
      }
    }
  ]
}
```

The following example IAM policy allows the principals to use any CMK in the account for certain cryptographic operations\. But it prohibits the principals from using any AWS KMS operations on CMKs with a `"Type"="Reserved"` tag or no `"Type"` tag\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "IAMAllowCryptographicOperations",
      "Effect": "Allow",
      "Action": [
          "kms:Encrypt",
          "kms:GenerateDataKey*",
          "kms:Decrypt",
          "kms:ReEncrypt*"
      ],
      "Resource": "arn:aws:kms:*:111122223333:key/*"
    },
    {
      "Sid": "IAMDenyOnTag",
      "Effect": "Deny",
      "Action": "kms:*",
      "Resource": "arn:aws:kms:*:111122223333:key/*",
      "Condition": {
          "StringEquals": {"aws:ResourceTag/Type": "Reserved"}
      }
    },
    {
      "Sid": "IAMDenyNoTag",
      "Effect": "Deny",
      "Action": "kms:*",
      "Resource": "arn:aws:kms:*:111122223333:key/*",
      "Condition":{"Null":{"aws:ResourceTag/Type":"true"}
    }
  ]
}
```