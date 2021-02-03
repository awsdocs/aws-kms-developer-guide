# Controlling access to tags<a name="tag-permissions"></a>

To add, view, and delete tags, either in the AWS KMS console or by using the API, principals need tagging permissions\. You can provide these permissions in [key policies](key-policies.md)\. You can also provide them in IAM policies \(including [VPC endpoint policies](kms-vpc-endpoint.md#vpce-policy)\), but only if [the key policy allows it](key-policies.md#allow-iam-policies)\. The [AWSKeyManagementServicePowerUser](aws-managed-policies.md) managed policy allows principals to tag, untag, and list tags on all CMKs the account can access\. 

You can also limit these permissions by using AWS global condition keys for tags\. In AWS KMS, these conditions can control access to tagging operations, such as [TagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) and [UntagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_UntagResource.html)\.

**Note**  
Be cautious when giving principals permission to manage tags and aliases\. Changing a tag or alias can allow or deny permission to the CMK\. For details, see [Using ABAC for AWS KMS](abac.md) and [Using tags to control access to CMKs](tag-authorization.md)\.

For example policies and more information, see [Controlling Access Based on Tag Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_tags.html#access_tags_control-tag-keys) in the *IAM User Guide*\.

Permissions to create and manage tags work as follows\.

**kms:TagResource**  
Allows principals to add or edit tags\. To add tags while creating a CMK, the principal must have permission in an IAM policy that isn't restricted to particular CMKs\.

**kms:ListResourceTags**  
Allows principals to view tags on CMKs\.

**kms:UntagResource**  
Allows principals to delete tags from CMKs\.

## Tag permissions in policies<a name="tag-permission-examples"></a>

You can provide tagging permissions in a key policy or IAM policy\. For example, the following example key policy gives select users tagging permission on the CMK\. It gives all users who can assume the example Administrator or Developer roles permission to view tags\.

```
{
  "Version": "2012-10-17",
  "Id": "key-policy-example",
  "Statement": [
    { 
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::111122223333:root"},
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow all tagging permissions",
      "Effect": "Allow",
      "Principal": {"AWS": [
        "arn:aws:iam::111122223333:user/LeadAdmin",
        "arn:aws:iam::111122223333:user/SupportLead"
      ]},
      "Action": [
          "kms:TagResource",
          "kms:ListResourceTags",
          "kms:UntagResource"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Allow roles to view tags",
      "Effect": "Allow",
      "Principal": {"AWS": [
        "arn:aws:iam::111122223333:role/Administrator"},
        "arn:aws:iam::111122223333:role/Developer"}
      ]},
      "Action": "kms:ListResourceTags",
      "Resource": "*"
    }
  ]
}
```

To give principals tagging permission on multiple CMKs, you can use an IAM policy\. For this policy to be effective, the key policy for each CMK must allow the account to use IAM policies to control access to the CMK\.

For example, the following IAM policy allows the principals to create CMKs\. It also allows them to create and manage tags on all CMKs in the specified account\. This combination allows the principals to use the [Tags](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html#KMS-CreateKey-request-Tags) parameter of the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation to add tags to a CMK while they are creating it\. 

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "IAMPolicyCreateKeys",
      "Effect": "Allow",
      "Action": "kms:CreateKey",
      "Resource": "*"
    },
    {
      "Sid": "IAMPolicyTags",
      "Effect": "Allow",
      "Action": [
        "kms:TagResource",
        "kms:UntagResource",
        "kms:ListResourceTags"
      ],
      "Resource": "arn:aws:kms:*:111122223333:key/*"
    }    
  ]
}
```

## Limiting tag permissions<a name="tag-permissions-conditions"></a>

You can limit tagging permissions by using [policy conditions](policy-conditions.md)\. The following policy conditions can be applied to the `kms:TagResource` and `kms:UntagResource` permissions\. For example, you can use the `aws:RequestTag` condition to allow a principal to add only particular tags, or prevent a principal from adding tags with particular tag keys\. Or, you can use the `kms:KeyOrigin` condition to prevent principals from tagging or untagging CMKs with [imported key material](importing-keys.md)\. 
+ [aws:RequestTag](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-requesttag)
+ [aws:ResourceTag](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-resourcetag) \(IAM policies only\)
+ [aws:TagKeys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-tag-keys)
+ [kms:CallerAccount](policy-conditions.md#conditions-kms-caller-account)
+ [kms:CustomerMasterKeySpec](policy-conditions.md#conditions-kms-customer-master-key-spec)
+ [kms:CustomerMasterKeyUsage](policy-conditions.md#conditions-kms-customer-master-key-usage)
+ [kms:KeyOrigin](policy-conditions.md#conditions-kms-key-origin)
+ [kms:ViaService](policy-conditions.md#conditions-kms-via-service)

As a best practice when you use tags to control access to CMKs, use the `aws:RequestTag` or `aws:TagKeys` condition key to determine which tags \(or tag keys\) are allowed\. 

For example, the following IAM policy is similar to the previous one\. However, this policy allows the principals to add and delete only tags with a `Project` tag key\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "IAMPolicyCreateKey",
      "Effect": "Allow",
      "Action": "kms:CreateKey",
      "Resource": "*"
    },
    {
      "Sid": "IAMPolicyViewAllTags",
      "Effect": "Allow",
      "Action": "kms:ListResourceTags",
      "Resource": "arn:aws:kms:*:111122223333:key/*",
    },
    {
      "Sid": "IAMPolicyManageTags",
      "Effect": "Allow",
      "Action": [
        "kms:TagResource",
        "kms:UntagResource"
      ],
      "Resource": "arn:aws:kms:*:111122223333:key/*",
      "Condition": {
          "ForAllValues:StringEquals": {"aws:TagKeys": "Project"}
      }
    }
  ]
}
```