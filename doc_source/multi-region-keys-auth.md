# Controlling access to multi\-Region keys<a name="multi-region-keys-auth"></a>

You can use multi\-Region keys in compliance, disaster recovery, and backup scenarios that would be more complex with single\-Region keys\. However, because the security properties of multi\-Region keys are significantly different from those of single\-Region keys, we recommend using caution when authorizing the creation, management, and use of multi\-Region keys\.

**Note**  
Existing IAM policy statements with wildcard characters in the `Resource` field now apply to both single\-Region and multi\-Region keys\. To restrict them to single\-Region KMS keys or multi\-Region keys, use the [kms:MultiRegion](conditions-kms.md#conditions-kms-multiregion) condition key\.

Use your authorization tools to prevent creation and use of multi\-Region keys in any scenario where a single\-Region will suffice\. Allow principals to replicate a multi\-Region key only into AWS Regions that require them\. Give permission for multi\-Region keys only to principals who need them and only for tasks that require them\.

You can use key policies, IAM policies, and grants to allow IAM principals to manage and use multi\-Region keys in your AWS account\. Each multi\-Region key is an independent resource with a unique key ARN and key policy\. You need to establish and maintain a key policy for each key and make sure that new and existing IAM policies implement your authorization strategy\. 

**Topics**
+ [Authorization basics for multi\-Region keys](#multi-region-auth-about)
+ [Authorizing multi\-Region key administrators and users](#multi-region-auth-users)
+ [Authorizing AWS KMS to synchronize multi\-Region keys](#multi-region-auth-slr)

## Authorization basics for multi\-Region keys<a name="multi-region-auth-about"></a>

When designing key policies and IAM policies for multi\-Region keys, consider the following principles\.
+ **Key policy** — Each multi\-Region key is an independent KMS key resource with its own [key policy](key-policies.md)\. You can apply the same or a different key policy to each key in the set of related multi\-Region keys\. Key policies are *not* [shared properties](multi-region-keys-overview.md#mrk-sync-properties) of multi\-Region keys\. AWS KMS does not copy or synchronize key policies among related multi\-Region keys\. 

  When you create a replica key in the AWS KMS console, the console displays the current key policy of the primary key as a convenience\. You can use this key policy, edit it, or delete and replace it\. But even if you accept the primary key policy unchanged, AWS KMS doesn't synchronize the policies\. For example, if you change the key policy of the primary key, the key policy of the replica key remains the same\.
+ **Default key policy** — When you create multi\-Region keys by using the [CreateKey](https://docs.aws.amazon.com/IAM/latest/APIReference/API_CreateKey.html) and `ReplicateKey` operations, the [default key policy](key-policy-default.md) is applied unless you specify a key policy in the request\. This is the same default key policy that is applied to single\-Region keys\.
+ **IAM policies** — As with all KMS keys, you can use IAM policies to control access to multi\-Region keys only when the [key policy allows it](key-policy-default.md#key-policy-default-allow-root-enable-iam)\. [IAM policies](iam-policies.md) apply to all AWS Regions by default\. However, you can use condition keys, such as [aws:RequestedRegion](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-requestedregion), to limit permissions to a particular Region\. 

  To create primary and replica keys, principals must have `kms:CreateKey` permission in an IAM policy that applies to the Region where the key is created\. 
+ **Grants** — AWS KMS [grants](grants.md) are Regional\. Each grant allows permissions to one KMS key\. You can use grants to allow permissions to a multi\-Region primary key or replica key\. But you cannot use a single grant to allow permissions to multiple KMS keys, even if they are related multi\-Region keys\.
+ **Key ARN** — Each multi\-Region key has a [unique key ARN](multi-region-keys-overview.md#mrk-how-it-works)\. The key ARNs of related multi\-Region keys have the same partition, account, and key ID, but different Regions\.

  To apply an IAM policy statement to a particular multi\-Region key, use its key ARN or a key ARN pattern that includes the Region\. To apply an IAM policy statement to all related multi\-Region keys, use a wildcard character \(\*\) in the Region element of the ARN, as shown in the following example\.

  ```
  {
    "Effect": "Allow",  
    "Action": [
      "kms:Describe*",
      "kms:List*"
    ],
    "Resource": {
        "arn:aws:kms:*::111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab"
    }
  }
  ```

  To apply a policy statement to all multi\-Region keys in your AWS account, you can use the [kms:MultiRegion](conditions-kms.md#conditions-kms-multiregion) policy condition or a key ID pattern that includes the distinctive `mrk-` prefix\.
+ **Service\-linked role** — Principals who create multi\-Region primary keys must have [iam:CreateServiceLinkedRole](https://docs.aws.amazon.com/IAM/latest/APIReference/API_CreateServiceLinkedRole.html) permission\.

  To synchronize the shared properties of related multi\-Region keys, AWS KMS assumes an IAM [service\-linked role](#multi-region-auth-slr)\. AWS KMS creates the service\-linked role in the AWS account whenever you create a multi\-Region primary key\. \(If the role exists, AWS KMS recreates it, which has no harmful effect\.\) The role is valid in all Regions\. To allow AWS KMS to create \(or recreate\) the service\-linked role, principals who create multi\-Region primary keys must have [iam:CreateServiceLinkedRole](https://docs.aws.amazon.com/IAM/latest/APIReference/API_CreateServiceLinkedRole.html) permission\.

## Authorizing multi\-Region key administrators and users<a name="multi-region-auth-users"></a>

Principals who create and manage multi\-Region keys need the following permissions in the primary and replica Regions:
+ `kms:CreateKey`
+ `kms:ReplicateKey`
+ `kms:UpdatePrimaryRegion`
+ `iam:CreateServiceLinkedRole`

### Creating a primary key<a name="mrk-auth-create-primary"></a>

To [create a multi\-Region primary key](create-primary-keys.md), the principal needs [kms:CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) and [iam:CreateServiceLinkedRole](https://docs.aws.amazon.com/IAM/latest/APIReference/API_CreateServiceLinkedRole.html) permissions in an IAM policy that is effective in the primary key's Region\. Principals who have these permissions can create single\-Region and multi\-Region keys unless you restrict their permissions\. 

The `iam:CreateServiceLinkedRole` permission allows AWS KMS to create the [**AWSServiceRoleForKeyManagementServiceMultiRegionKeys** role](#multi-region-auth-slr) to synchronize the [shared properties](multi-region-keys-overview.md#mrk-sync-properties) of related multi\-Region keys\.

For example, this IAM policy allows a principal to create any type of KMS key\.

```
{
  "Version": "2012-10-17",
  "Statement":{
      "Action": [
        "kms:CreateKey",
        "iam:CreateServiceLinkedRole"
      ],
      "Effect":"Allow",
      "Resource":"*"
  }
}
```

To allow or deny permission to create multi\-Region primary keys, use the [kms:MultiRegion](conditions-kms.md#conditions-kms-multiregion) condition key\. Valid values are `true` \(multi\-Region key\) or `false` \(single\-Region key\)\. For example, the following IAM policy statement uses a `Deny` action with the `kms:MultiRegion` condition key to prevent principals from creating multi\-Region keys\. 

```
{
  "Version": "2012-10-17",
  "Statement":{
      "Action":"kms:CreateKey",
      "Effect":"Deny",
      "Resource":"*",
      "Condition": {
          "Bool": "kms:MultiRegion": true
      }
  }
}
```

### Replicating keys<a name="mrk-auth-replicate"></a>

To [create a multi\-Region replica key](#mrk-auth-replicate), the principal needs the following permissions:
+  [kms:ReplicateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReplicateKey.html) permission in the key policy of the primary key\.
+ [kms:CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) permission in an IAM policy that is effective in the replica key Region\.

Use caution when allowing these permissions\. They allow principals to create KMS keys and the key policies that authorize their use\. The `kms:ReplicateKey` permission also authorizes the transfer of key material across Region boundaries within AWS KMS\.

To restrict the AWS Regions in which a multi\-Region key can be replicated, use the [kms:ReplicaRegion](conditions-kms.md#conditions-kms-replica-region) condition key\. It limits only the `kms:ReplicateKey` permission\. Otherwise, it has no effect\. For example, the following key policy allows the principal to replicate this primary key, but only in the specified Regions\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/Administrator"
  },
  "Action": "kms:ReplicateKey",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:ReplicaRegion": [
         "us-east-1",
         "eu-west-3",
         "ap-southeast-2"
      ]
    }
  }
}
```

### Updating the primary Region<a name="mrk-auth-update"></a>

Authorized principals can convert a replica key to a primary key, which changes the former primary key into a replica\. This action is known as [updating the primary Region](multi-region-keys-manage.md#multi-region-update)\. To update the primary Region, the principal needs [kms:UpdatePrimaryRegion](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdatePrimaryRegion.html) permission in both Regions\. You can provide these permissions in a key policy or IAM policy\.
+ `kms:UpdatePrimaryRegion` on the primary key\. This permission must be effective in the primary key Region\.
+ `kms:UpdatePrimaryRegion` on the replica key\. This permission must be effective in the replica key Region\.

For example, the following key policy gives users who can assume the Administrator role permission to update the primary Region of the KMS key\. This KMS key can be the primary key or a replica key in this operation\.

```
{
  "Effect": "Allow",
  "Resource": "*",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/Administrator"
  },
  "Action": "kms:UpdatePrimaryRegion"
}
```

To restrict the AWS Regions that can host a primary key, use the [kms:PrimaryRegion](conditions-kms.md#conditions-kms-primary-region) condition key\. For example, the following IAM policy statement allows the principals to update the primary Region of the multi\-Region keys in the AWS account, but only when the new primary Region is one of the specified Regions\.

```
{
  "Effect": "Allow",  
  "Action": "kms:UpdatePrimaryRegion",
  "Resource": {
      "arn:aws:kms:*:111122223333:key/*"
  },
  "Condition": {
    "StringEquals": {
      "kms:PrimaryRegion": [ 
         "us-west-2",
         "sa-east-1",
         "ap-southeast-1"
      ]
    }
  }
}
```

### Using and managing multi\-Region keys<a name="mrk-auth-using"></a>

By default, principals who have permission to use and manage KMS keys in an AWS account and Region also have permission to use and manage multi\-Region keys\. However, you can use the [kms:MultiRegion](conditions-kms.md#conditions-kms-multiregion) condition key to allow only single\-Region keys or only multi\-Region keys\. Or use the [kms:MultiRegionKeyType](conditions-kms.md#conditions-kms-multiregion-key-type) condition key to allow only multi\-Region primary keys or only replica keys\. Both condition keys controls access to the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation and to any operation that uses an existing KMS key, such as [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) or [EnableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKey.html)\.

The following example IAM policy statement uses the `kms:MultiRegion` condition key to prevent the principals from using or managing any multi\-Region key\.

```
{
  "Effect": "Deny",  
  "Action": "kms:*",
  "Resource": "*",
  "Condition": {
    "Bool": "kms:MultiRegion": true
    }
}
```

This example IAM policy statement uses the `kms:MultiRegionKeyType` condition to allow principals to schedule and cancel key deletion, but only on multi\-Region replica keys\.

```
{
  "Effect": "Allow",  
  "Action": [
    "kms:ScheduleKeyDeletion",
    "kms:CancelKeyDeletion"
  ],
  "Resource": {
      "arn:aws:kms:us-west-2:111122223333:key/*"
  },
  "Condition": {
    "StringEquals": "kms:MultiRegionKeyType": "REPLICA"
  }
}
```

## Authorizing AWS KMS to synchronize multi\-Region keys<a name="multi-region-auth-slr"></a>

To support [multi\-Region keys](#multi-region-keys-auth), AWS KMS uses an IAM service linked role\. This role gives AWS KMS the permissions it needs to synchronize [shared properties](multi-region-keys-overview.md#mrk-sync-properties)\. You can view the [SynchronizeMultiRegionKey](ct-synchronize-multi-region-key.md) CloudTrail event that records AWS KMS synchronizing shared properties in your AWS CloudTrail logs\.

### About the service\-linked role for multi\-Region keys<a name="about-key-store-slr"></a>

A [service\-linked role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) is an IAM role that gives one AWS service permission to call other AWS services on your behalf\. It's designed to make it easier for you to use the features of multiple integrated AWS services without having to create and maintain complex IAM policies\.

For multi\-Region keys, AWS KMS creates the **AWSServiceRoleForKeyManagementServiceMultiRegionKeys** service\-linked role with the **AWSKeyManagementServiceMultiRegionKeysServiceRolePolicy** policy\. This policy gives the role the `kms:SynchronizeMultiRegionKey` permission, which allows it to synchronize the shared properties of multi\-Region keys\.

Because the **AWSServiceRoleForKeyManagementServiceMultiRegionKeys** service\-linked role trusts only `mrk.kms.amazonaws.com`, only AWS KMS can assume this service\-linked role\. This role is limited to the operations that AWS KMS needs to synchronize multi\-Region shared properties\. It does not give AWS KMS any additional permissions\. For example, AWS KMS does not have permission to create, replicate, or delete any KMS keys\.

For more information about how AWS services use service\-linked roles, see [Using Service\-Linked Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) in the IAM User Guide\.

### Create the service\-linked role<a name="create-mrk-slr"></a>

AWS KMS automatically creates the **AWSServiceRoleForKeyManagementServiceMultiRegionKeys** service\-linked role in your AWS account when you create a multi\-Region key, if the role does not already exist\. You cannot create or re\-create this service\-linked role directly\. 

### Edit the service\-linked role description<a name="edit-mrk-slr"></a>

You cannot edit the role name or the policy statements in the **AWSServiceRoleForKeyManagementServiceMultiRegionKeys** service\-linked role, but you can edit the role description\. For instructions, see [Editing a Service\-Linked Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#edit-service-linked-role) in the *IAM User Guide*\.

### Delete the service\-linked role<a name="delete-mrk-slr"></a>

AWS KMS does not delete the **AWSServiceRoleForKeyManagementServiceMultiRegionKeys** service\-linked role from your AWS account and you cannot delete it\. However, AWS KMS does not assume the **AWSServiceRoleForKeyManagementServiceMultiRegionKeys** role or use any of its permissions unless you have multi\-Region keys in your AWS account and Region\. 