# ABAC for AWS KMS<a name="abac"></a>

Attribute\-based access control \(ABAC\) is an authorization strategy that defines permissions based on attributes\. AWS KMS supports ABAC by allowing you to control access to your customer managed keys based on the tags and aliases associated with the KMS keys\. The tag and alias condition keys that enable ABAC in AWS KMS provide a powerful and flexible way to authorize principals to use KMS keys without editing policies or managing grants\. But you should use these feature with care so principals aren't inadvertently allowed or denied access\. 

If you use ABAC, be aware that permission to manage tags and aliases is now an access control permission\. Be sure that you know the existing tags and aliases on all KMS keys before you deploy a policy that depends on tags or aliases\. Take reasonable precautions when adding, deleting, and updating aliases, and when tagging and untagging keys\. Give permissions to manage tags and aliases only to principals who need them, and limit the tags and aliases they can manage\. 

**Notes**  
When using ABAC for AWS KMS, be cautious about giving principals permission to manage tags and aliases\. Changing a tag or alias might allow or deny permission to a KMS key\. Key administrators who don't have permission to change key policies or create grants can control access to KMS keys if they have permission to manage tags or aliases\.   
It might take up to five minutes for tag and alias changes to affect KMS key authorization\. Recent changes might be visible in API operations before they affect authorization\.  
To control access to a KMS key based on its alias, you must use a condition key\. You cannot use an alias to represent a KMS key in the `Resource` element of a policy statement\. When an alias appears in the `Resource` element, the policy statement applies to the alias, not to the associated KMS key\.

**Learn more**
+ For details about AWS KMS support for ABAC, including examples, see [Using aliases to control access to KMS keys](alias-authorization.md) and [Using tags to control access to KMS keys](tag-authorization.md)\.
+ For more general information about using tags to control access to AWS resources, see [What is ABAC for AWS?](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_attribute-based-access-control.html) and [Controlling Access to AWS Resources Using Resource Tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_tags.html) in the *IAM User Guide*\.

## ABAC condition keys for AWS KMS<a name="about-abac-kms"></a>

To authorize access to KMS keys based on their tags and aliases, use the following condition keys in a key policy or IAM policy\.


| ABAC condition key | Description | Policy type | AWS KMS operations | 
| --- | --- | --- | --- | 
| [aws:ResourceTag](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-resourcetag) | Tag \(key and value\) on the KMS key matches the tag \(key and value\) or tag pattern in the policy | IAM policy only | KMS key resource operations 2 | 
| [aws:RequestTag/*tag\-key*](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-requesttag) | Tag \(key and value\) in the request matches the tag \(key and value\) or tag pattern in the policy | Key policy and IAM policies1 | [TagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html), [UntagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_UntagResource.html) | 
| [aws:TagKeys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-tagkeys) | Tag keys in the request match the tag keys in the policy | Key policy and IAM policies1 | [TagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html), [UntagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_UntagResource.html) | 
| [kms:ResourceAliases](policy-conditions.md#conditions-kms-resource-aliases) | Aliases associated with the KMS key match the aliases or alias patterns in the policy | IAM policy only | KMS key resource operations 2 | 
| [kms:RequestAlias](policy-conditions.md#conditions-kms-request-alias) | Alias that represents the KMS key in the request matches the alias or alias patterns in the policy\. | Key policy and IAM policies1 | [Cryptographic operations](concepts.md#cryptographic-operations), [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html), [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) | 

1Any condition key that can be used in a key policy can also be used in an IAM policy, but only if [the key policy allows it](key-policies.md#key-policy-default-allow-root-enable-iam)\.

2A *KMS key resource operation* is an operation authorized for a particular KMS key\. To identify the KMS key resource operations, in the [AWS KMS permissions table](kms-api-permissions-reference.md#kms-api-permissions-reference-table), look for a value of KMS key in the `Resources` column for the operation\. 

For example, you can use these condition keys to create the following policies\.
+ An IAM policy with `aws:ResourceAliases` that allows permission to use KMS keys with a particular alias or alias pattern\. This is a bit different from policies that rely on tags: Although you can use alias patterns in a policy, each alias must be unique in an AWS account and Region\. This allows you to apply a policy to a select set of KMS keys without listing the key ARNs of the KMS keys in the policy statement\. To add or remove KMS keys from the set, change the alias of the KMS key\.
+ A key policy with `aws:RequestAlias` that allows principals to use a KMS key in a `Encrypt` operation, but only when the `Encrypt` request uses that alias to identify the KMS key\.
+ An IAM policy with `aws:ResourceTag/tag-key` that denies permission to use KMS keys with a particular tag key and tag value\. This lets you apply a policy to a select set of KMS keys without listing the key ARNs of the KMS keys in the policy statement\. To add or remove KMS keys from the set, tag or untag the KMS key\.
+ An IAM policy with `aws:RequestTag/tag-key` that allows principals to delete only `"Purpose"="Test"` KMS key tags\. 
+ An IAM policy with `aws:TagKeys` that denies permission to tag or untag a KMS key with a `Restricted` tag key\.

ABAC makes access management flexible and scalable\. For example, you can use the `aws:ResourceTag/tag-key` condition key to create an IAM policy that allows principals to use a KMS key for specified operations only when the KMS key has a `Purpose=Test` tag\. The policy applies to all KMS keys in all Regions of the AWS account\.

When attached to a user or role, the following IAM policy allows principals to use all existing KMS keys with a `Purpose=Test` tag for the specified operations\. To provide this access to new or existing KMS keys, you don't need to change the policy\. Just attach the `Purpose=Test` tag to the KMS keys\. Similarly, to remove this access from KMS keys with a `Purpose=Test` tag, edit or delete the tag\. 

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AliasBasedIAMPolicy",
      "Effect": "Allow",
      "Action": [
        "kms:Decrypt",
        "kms:Encrypt",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
      ],
      "Resource": "arn:aws:kms:*:111122223333:key/*",
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/Purpose": "Test"
        }
      }
    }
  ]
}
```

However, if you use this feature, be careful when managing tags and aliases\. Adding, changing, or deleting a tag or alias can inadvertently allow or deny access to a KMS key\. Key administrators who don't have permission to change key policies or create grants can control access to KMS keys if they have permission to manage tags and aliases\. To mitigate this risk, consider [limiting permissions to manage tags](tag-permissions.md#tag-permissions-conditions) and [aliases](alias-access.md#alias-access-limiting)\. For example, you might want to allow only select principals to manage `Purpose=Test` tags\. For details, see [Using aliases to control access to KMS keys](alias-authorization.md) and [Using tags to control access to KMS keys](tag-authorization.md)\.

## Tags or aliases?<a name="abac-tag-or-alias"></a>

AWS KMS supports ABAC with tags and aliases\. Both options provide a flexible, scalable access control strategy, but they're slightly different from each other\. 

You might decide to use tags or use aliases based on your particular AWS use patterns\. For example, if you have already given tagging permissions to most administrators, it might be easier to control an authorization strategy based on aliases\. Or, if you are close to the quota for [aliases per KMS key](resource-limits.md#aliases-per-key), you might prefer an authorization strategy based on tags\. 

The following benefits are of general interest\.

** Benefits of tag\-based access control**
+ Same authorization mechanism for different types of AWS resources\. 

  You can use the same tag or tag key to control access to multiple resource types, such as an Amazon Relational Database Service \(Amazon RDS\) cluster, an Amazon Elastic Block Store \(Amazon EBS\) volume, and an KMS key\. This feature enables several different authorization models that are more flexible than traditional role\-based access control\.
+ Authorize access to a group of KMS keys\.

  You can use tags to manage access to a group of KMS keys in the same AWS account and Region\. Assign the same tag or tag key to the KMS keys that you choose\. Then create a simple, easy\-to\-maintain policy statement that is based on the tag or tag key\. To add or remove a KMS key from your authorization group, add or remove the tag; you don't need to edit the policy\.

**Benefits of alias\-based access control**
+ Authorize access to cryptographic operations based on aliases\.

  Most request\-based policy conditions for attributes, including [aws:RequestTag/*tag\-key*](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-requesttag), affect only operations that add, edit, or delete the attribute\. But the [kms:RequestAlias](policy-conditions.md#conditions-kms-request-alias) condition key controls access to cryptographic operations based on the alias used to identify the KMS key in the request\. For example, you can give a principal permission to use a KMS key in a `Encrypt` operation but only when the value of the `KeyId` parameter is `alias/restricted-key-1`\. To satisfy this condition requires all of the following:
  + The KMS key must be associated with that alias\.
  + The request must use the alias to identify the KMS key\.
  + The principal must have permission to use the KMS key subject to the `kms:RequestAlias` condition\. 

  This is particularly useful if your applications commonly use alias names or alias ARNs to refer to KMS keys\.
+ Provide very limited permissions\.

  An alias must be unique in an AWS account and Region\. As a result, giving principals access to a KMS key based on an alias can be much more restrictive than giving them access based on a tag\. Unlike aliases, tags can be assigned to multiple KMS keys in the same account and Region\. If you choose, you can use an alias pattern, such as `alias/test*`, to give principals access to a group of KMS keys in the same account and Region\. However, allowing or denying access to a particular alias allows very strict control on KMS keys\.

## Troubleshooting ABAC for AWS KMS<a name="troubleshooting-tags-aliases"></a>

Controlling access to KMS keys based on their tags and aliases is convenient and powerful\. However, it's prone to a few predictable errors that you'll want to prevent\.

### Access changed due to tag change<a name="access-denied-tag"></a>

If a tag is deleted or its value is changed, principals who have access to a KMS key based only on that tag will be denied access to the KMS key\. This can also happen when a tag that is included in a deny policy statement is added to a KMS key\. Adding a policy\-related tag to a KMS key can allow access to principals who should be denied access to a KMS key\.

For example, suppose that a principal has access to a KMS key based on the `Project=Alpha` tag, such as the permission provided by the following example IAM policy statement\. 

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
      ],
      "Resource": "arn:aws:kms:ap-southeast-1:111122223333:key/*",
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/Project": "Alpha"
        }
      }
    }
  ]
}
```

If the tag is deleted from that KMS key or the tag value is changed, the principal no longer has permission to use the KMS key for the specified operations\. This might become evident when the principal tries to read or write data in an AWS service that uses a customer managed key To trace the tag change, review your CloudTrail logs for [TagResource](ct-tagresource.md) or [UntagResource entries](ct-untagresource.md)\.

To restore access without updating the policy, change the tags on the KMS key\. This action has minimal impact other than a brief period while it is taking effect throughout AWS KMS\. To prevent an error like this one, give tagging and untagging permissions only to principals who need it and [limit their tagging permissions](tag-permissions.md#tag-permissions-conditions) to tags they need to manage\. Before changing a tag, search policies to detect access that depends on the tag, and get KMS keys in all Regions that have the tag\. You might consider creating an Amazon CloudWatch alarm when particular tags are changed\.

### Access change due to alias change<a name="access-denied-alias"></a>

If an alias is deleted or associated with a different KMS key, principals who have access to the KMS key based only on that alias will be denied access to the KMS key\. This can also happen when an alias that is associated with a KMS key is included in a deny policy statement\. Adding a policy\-related alias to a KMS key can also allow access to principals who should be denied access to a KMS key\.

For example, the following IAM policy statement uses the [kms:ResourceAliases](policy-conditions.md#conditions-kms-resource-aliases) condition key to allow access to KMS keys in different Regions of the account with any of the specified aliases\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AliasBasedIAMPolicy",
      "Effect": "Allow",
      "Action": [
        "kms:List*",
        "kms:Describe*",
        "kms:Decrypt"
      ],
      "Resource": "arn:aws:kms:*:111122223333:key/*",
      "Condition": {
        "ForAnyValue:StringEquals": {
          "kms:ResourceAliases": [
            "alias/ProjectAlpha",
            "alias/ProjectAlpha_Test",
            "alias/ProjectAlpha_Dev"
          ]
        }
      }
    }
  ]
}
```

To trace the alias change, review your CloudTrail logs for [CreateAlias](ct-createalias.md), [UpdateAlias](ct-updatealias.md), and [DeleteAlias](ct-deletealias.md) entries\.

To restore access without updating the policy, change the alias associated with the KMS key\. Because each alias can be associated with only one KMS key in an account and Region, managing aliases is a bit more difficult than managing tags\. Restoring access to some principals on one KMS key can deny the same or other principals access to a different KMS key\. 

To prevent this error, give alias management permissions only to principals who need it and [limit their alias\-management permissions](alias-access.md#alias-access-limiting) to aliases they need to manage\. Before updating or deleting an alias, search policies to detect access that depends on the alias, and find KMS keys in all Regions that are associated with the alias\.

### Access denied due to alias quota<a name="access-denied-alias-quota"></a>

Users who are authorized to use a KMS key by an [kms:ResourceAliases](policy-conditions.md#conditions-kms-resource-aliases) condition will get an `AccessDenied` exception if the KMS key exceeds the default [aliases per KMS key](resource-limits.md#aliases-per-key) quota for that account and Region\. 

To restore access, delete aliases that are associated with the KMS key so it complies with the quota\. Or use an alternate mechanism to give users access to the KMS key\. 

### Delayed authorization change<a name="tag-alias-auth-delay"></a>

Changes that you make to tags and aliases might take up to five minutes to affect the authorization of KMS keys\. As a result, a tag or alias change might be reflected in the responses from API operations before they affect authorization\. This delay is likely to be longer than the brief eventual consistency delay that affects most AWS KMS operations\. 

For example, you might have an IAM policy that allows certain principals to use any KMS key with a `"Purpose"="Test"` tag\. Then you add the `"Purpose"="Test"` tag to a KMS key\. Although the [TagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) operation completes and [ListResourceTags](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListResourceTags.html) response confirms that the tag is assigned to the KMS key, the principals might not have access to the KMS key for up to five minutes\.

To prevent errors, build this expected delay into your code\. 

### Failed requests due to alias updates<a name="failed-requests"></a>

When you update an alias, you associate an existing alias with a different KMS key\. 

[Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) and [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) requests that specify the [alias name](concepts.md#key-id-alias-name) or [alias ARN](concepts.md#key-id-alias-ARN) might fail because the alias is now associated with a KMS key that didn't encrypt the ciphertext\. This situation typically returns an `IncorrectKeyException` or `NotFoundException`\. Or if the request has no `KeyId` or `DestinationKeyId` parameter, the operation might fail with `AccessDenied` exception because the caller no longer has access to the KMS key that encrypted the ciphertext\. 

You can trace the change by looking at CloudTrail logs for [CreateAlias](ct-createalias.md), [UpdateAlias](ct-updatealias.md), and [DeleteAlias](ct-deletealias.md) log entries\. You can also use the value of the `LastUpdatedDate` field in the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) response to detect a change\. 

For example, the following [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) example response shows that the `ProjectAlpha_Test` alias in the `kms:ResourceAliases` condition was updated\. As a result, the principals who have access based on the alias lose access to the previously associated KMS key\. Instead, they have access to the newly associated KMS key\. 

```
$ aws kms list-aliases --query 'Aliases[?starts_with(AliasName, `alias/ProjectAlpha`)]'

{
    "Aliases": [
        {
            "AliasName": "alias/ProjectAlpha_Test",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/ProjectAlpha_Test",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
            "CreationDate": 1566518783.394,
            "LastUpdatedDate": 1605308931.903
        },
        {
            "AliasName": "alias/ProjectAlpha_Restricted",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/ProjectAlpha_Restricted",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "CreationDate": 1553410800.010,
            "LastUpdatedDate": 1553410800.010
        }
    ]
}
```

The remedy for this change isn't simple\. You can update the alias again to associate it with the original KMS key\. However, before you act, you need to consider the effect of that change on the currently associated KMS key\. If principals used the latter KMS key in cryptographic operations, they might need continued access to it\. In this case, you might want to update the policy to ensure that principals have permission to use both of the KMS keys\. 

You can prevent an error like this one: Before updating an alias, search policies to detect access that depends on the alias\. Then get KMS keys in all Regions that are associated with the alias\. Give alias management permissions only to principals who need it and [limit their alias\-management permissions](alias-access.md#alias-access-limiting) to aliases they need to manage\.