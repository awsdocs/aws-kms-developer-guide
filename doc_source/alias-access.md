# Controlling access to aliases<a name="alias-access"></a>

When you create or change an alias, you affect the alias and its associated CMK\. Therefore, principals who manage aliases must have permission to call the alias operation on the alias and on all affected CMKs\. You can provide these permissions by using [key policies](key-policies.md), [IAM policies](iam-policies.md) and [grants](grants.md)\. 

**Note**  
Be cautious when giving principals permission to manage tags and aliases\. Changing a tag or alias can allow or deny permission to the CMK\. For details, see [Using ABAC for AWS KMS](abac.md) and [Using aliases to control access to CMKs](alias-authorization.md)\.

For information about controlling access to all AWS KMS operations, see [AWS KMS API permissions reference](kms-api-permissions-reference.md)\.

Permissions to create and manage aliases work as follows\.

## kms:CreateAlias<a name="alias-access-create"></a>

To create an alias, the principal needs the following permissions for both the alias and for the associated CMK\. 
+ `kms:CreateAlias` for the alias\. Provide this permission in an IAM policy that is attached to the principal who is allowed to create the alias\.

  The following example policy statement specifies a particular alias in a `Resource` element\. But you can list multiple alias ARNs, specify an alias pattern, such as "test\*", or specify a `Resource` value of `"*"` to allow the principal to create any alias in the account and Region\. Permission to create an alias can also be included in a `kms:Create*` permission for all resources in an account and Region\.

  ```
  {
    "Sid": "IAMPolicyForAnAlias",
    "Effect": "Allow",
    "Action": [
      "kms:CreateAlias",
      "kms:UpdateAlias"
      "kms:DeleteAlias"
    ],
    "Resource": "arn:aws:kms:us-west-2:111122223333:alias/test-key"
  }
  ```
+ `kms:CreateAlias` for the CMK\. This permission must be provided in a key policy or in an IAM policy that is delegated from the key policy\. 

  ```
  {
    "Sid": "Key policy for 1234abcd-12ab-34cd-56ef-1234567890ab",
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::111122223333:user/KMSAdminUser"},
    "Action": [
      "kms:CreateAlias",
      "kms:DescribeKey"
    ],
    "Resource": "*"
  }
  ```

You can use condition keys to limit the CMKs to which you can assign an alias\. For example, you can use the [kms:CustomerMasterKeySpec](policy-conditions.md#conditions-kms-customer-master-key-spec) condition key to allow the principal to create aliases only on asymmetric CMKs\. For a full list of conditions keys that you can use to limit the `kms:CreateAlias` permission on CMK resources, see [AWS KMS permissions](kms-api-permissions-reference.md)\.

## kms:ListAliases<a name="alias-access-view"></a>

To list aliases in the account and Region, the principal must have `kms:ListAliases` permission in an IAM policy\. Because this policy isn't related to any particular CMK or alias resource, the value of the resource element in the policy [must be `"*"`](iam-policies-best-practices.md#require-resource-star)\. 

For example, the following IAM policy statement gives the principal permission to list all CMKs and aliases in the account and Region\.

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

## kms:UpdateAlias<a name="alias-access-update"></a>

To change the CMK that is associated with an alias, the principal needs three permission elements: one for the alias, one for the current CMK, and one for the new CMK\.

For example, suppose you want to change the `test-key` alias from the CMK with key ID 1234abcd\-12ab\-34cd\-56ef\-1234567890ab to the CMK with key ID 0987dcba\-09fe\-87dc\-65ba\-ab0987654321\. In that case, include policy statements similar to the examples in this section\.
+ `kms:UpdateAlias` for the alias\. You provide this permission in an IAM policy that is attached to the principal\. The following IAM policy specifies a particular alias\. But you can list multiple alias ARNs, specify an alias pattern, such as "test\*", or specify a `Resource` value of `"*"` to allow the principal to update any alias in the account and Region\.

  ```
  {
    "Sid": "IAMPolicyForAnAlias",
    "Effect": "Allow",
    "Action": [
      "kms:UpdateAlias",
      "kms:ListAliases",
      "kms:ListKeys",
    ],
    "Resource": "arn:aws:kms:us-west-2:111122223333:alias/test-key"
  }
  ```
+ `kms:UpdateAlias` for the CMK that is currently associated with the alias\. This permission must be provided in a key policy or in an IAM policy that is delegated from the key policy\.

  ```
  {
    "Sid": "Key policy for 1234abcd-12ab-34cd-56ef-1234567890ab",
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::111122223333:user/KMSAdminUser"},
    "Action": [
      "kms:UpdateAlias",
      "kms:DescribeKey"
    ],
    "Resource": "*"
  }
  ```
+ `kms:UpdateAlias` for the CMK that the operation associates with the alias\. This permission must be provided in a key policy or in an IAM policy that is delegated from the key policy\.

  ```
  {
    "Sid": "Key policy for 0987dcba-09fe-87dc-65ba-ab0987654321",
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::111122223333:user/KMSAdminUser"},
    "Action": [
      "kms:UpdateAlias", 
      "kms:DescribeKey"
    ],
    "Resource": "*"
  }
  ```

You can use condition keys to limit either or both of CMKs in an `UpdateAlias` operation\. For example, you can use a [kms:ResourceAliases](policy-conditions.md#conditions-kms-resource-aliases) condition key to allow the principal to update aliases only when the target CMK already has a particular alias\. For a full list of conditions keys that you can use to limit the `kms:UpdateAlias` permission on a CMK resource, see [AWS KMS permissions](kms-api-permissions-reference.md)\.

## kms:DeleteAlias<a name="alias-access-delete"></a>

To delete an alias, the principal needs permission for the alias and for the associated CMK\. 

As always, you should exercise caution when giving principals permission to delete a resource\. However, deleting an alias has no effect on the associated CMK\. Although it might cause a failure in an application that relies on the alias, if you mistakenly delete an alias, you can recreate it\. 
+ `kms:DeleteAlias` for the alias\. Provide this permission in an IAM policy attached to the principal who is allowed to delete the alias\.

  The following example policy statement specifies the alias in a `Resource` element\. But you can list multiple alias ARNs, specify an alias pattern, such as "test\*" or specify a `Resource` value of `"*"` to allow the principal to delete any alias in the account and Region\.

  ```
  {
    "Sid": "IAMPolicyForAnAlias",
    "Effect": "Allow",
    "Action": [
      "kms:CreateAlias",
      "kms:UpdateAlias"
      "kms:DeleteAlias"
    ],
    "Resource": "arn:aws:kms:us-west-2:111122223333:alias/test-key"
  }
  ```
+ `kms:DeleteAlias` for the associated CMK\. This permission must be provided in a key policy or in an IAM policy that is delegated from the key policy\.

  ```
  {
    "Sid": "Key policy for 1234abcd-12ab-34cd-56ef-1234567890ab",
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::111122223333:user/KMSAdminUser"},
    "Action": [
      "kms:CreateAlias",
      "kms:UpdateAlias",
      "kms:DeleteAlias",
      "kms:DescribeKey"
    ],
    "Resource": "*"
  }
  ```

## Limiting alias permissions<a name="alias-access-limiting"></a>

You can use condition keys to limit alias permissions when the resource is a CMK\. For example, the following IAM policy allows the alias operations on CMKs in an account and Region\. However, it uses the [kms:CustomerMasterKeyUsage](policy-conditions.md#conditions-kms-customer-master-key-usage) condition key to limit the permissions to CMKs that are used for encryption and decryption\. 

For a full list of conditions keys that you can use to limit alias permission on a CMK resource, see [AWS KMS permissions](kms-api-permissions-reference.md)\.

```
{
  "Sid": "IAMPolicyCMKPermissions",
  "Effect": "Allow",
  "Resource": "arn:aws:kms:us-west-2:111122223333:key/*",
  "Action": [
    "kms:CreateAlias",
    "kms:UpdateAlias"
    "kms:DeleteAlias"
  ],
  "Condition": {
    "StringEquals": {
      "kms:CustomerMasterKeyUsage": "ENCRYPT_DECRYPT"
    }
  }  
},
```

You can't use condition keys in a policy statement where the resource is an alias\. To limit the aliases that a principal can manage, use the value of the `Resource` element of the IAM policy statement that controls access to the alias\. For example, the following policy statements allows the principal to create, update, or delete any alias in the AWS account and Region unless the alias begins with `Restricted`\.

```
{
  "Sid": "IAMPolicyForAnAliasAllow",
  "Effect": "Allow",
  "Action": [
    "kms:CreateAlias",
    "kms:UpdateAlias"
    "kms:DeleteAlias"
  ],
  "Resource": "arn:aws:kms:us-west-2:111122223333:alias/*"
},
{
  "Sid": "IAMPolicyForAnAliasDeny",
  "Effect": "Deny",
  "Action": [
    "kms:CreateAlias",
    "kms:UpdateAlias"
    "kms:DeleteAlias"
  ],
  "Resource": "arn:aws:kms:us-west-2:111122223333:alias/Restricted*"
}
```