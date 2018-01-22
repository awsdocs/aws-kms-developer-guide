# Using Key Policies in AWS KMS<a name="key-policies"></a>

Key policies are the primary way to control access to customer master keys \(CMKs\) in AWS KMS\. They are not the only way to control access, but you cannot control access without them\. For more information, see [Managing Access to AWS KMS CMKs](control-access-overview.md#managing-access)\.

**Topics**

+ [Overview of Key Policies](#key-policy-overview)

+ [Default Key Policy](#key-policy-default)

+ [Example Key Policy](#key-policy-example)

## Overview of Key Policies<a name="key-policy-overview"></a>

A key policy is a document that uses [JSON \(JavaScript Object Notation\)](http://json.org/) to specify permissions\. You can work with these JSON documents directly, or you can use the AWS Management Console to work with them using a graphical interface called the *default view*\. For more information about the console's default view for key policies, see [Default Key Policy](#key-policy-default) and [Modifying a Key Policy](key-policy-modifying.md)\.

Key policy documents share a common JSON syntax with other permissions policies in AWS, and have the following basic structure:

```
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "statement identifier",
    "Effect": "effect",
    "Principal": "principal",
    "Action": "action",
    "Resource": "resource",
    "Condition": {"condition operator": {"condition context key": "context key value"}}
  }]
}
```

A key policy document must have a `Version` element\. We recommend setting the version to `2012-10-17` \(the latest version\)\. In addition, a key policy document must have one or more statements, and each statement consists of up to six elements:

+ **Sid** – \(Optional\) The Sid is a statement identifier, an arbitrary string you can use to identify the statement\.

+ **Effect** – \(Required\) The effect specifies whether to allow or deny the permissions in the policy statement\. The Effect must be Allow or Deny\. If you don't explicitly allow access to a CMK, access is implicitly denied\. You can also explicitly deny access to a CMK\. You might do this to make sure that a user cannot access it, even when a different policy allows access\.

+ **Principal** – \(Required\) The principal is the identity that the permissions in the policy statement apply to\. You can specify AWS accounts \(root\), IAM users, IAM roles, and some AWS services as principals in a key policy\. IAM groups are not valid principals\.

+ **Action** – \(Required\) Actions specify the API operations to allow or deny\. For example, the `kms:Encrypt` action corresponds to the AWS KMS [Encrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) API operation\. You can list more than one action in a policy statement\. For more information, see [AWS KMS API Permissions Reference](kms-api-permissions-reference.md)\.

+ **Resource** – \(Required\) In a key policy, you use `"*"` for the resource, which means "this CMK\." A key policy applies only to the CMK it is attached to\.

+ **Condition** – \(Optional\) Conditions specify requirements that must be met for a key policy to take effect\. With conditions, AWS can evaluate the context of an API request to determine whether or not the policy statement applies\. For more information, see [Using Policy Conditions](policy-conditions.md)\.

For more information about AWS policy syntax, see [AWS IAM Policy Reference](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) in the *IAM User Guide*\.

## Default Key Policy<a name="key-policy-default"></a>

**Default key policy when you create a CMK programmatically**  
When you create a CMK programmatically—that is, with the [AWS KMS API](http://docs.aws.amazon.com/kms/latest/APIReference/) \(including through the [AWS SDKs](https://aws.amazon.com/tools/#sdk) and [command line tools](https://aws.amazon.com/tools/#cli)\)—you have the option of providing the key policy for the new CMK\. If you don't provide one, AWS KMS creates one for you\. This default key policy has one policy statement that gives the AWS account \(root user\) that owns the CMK full access to the CMK and enables IAM policies in the account to allow access to the CMK\. For more information about this policy statement, see [Allows Access to the AWS Account and Enables IAM Policies](#key-policy-default-allow-root-enable-iam)\.

**Default key policy when you create a CMK with the AWS Management Console**  
When you create a CMK with the AWS Management Console, you can choose the IAM users, IAM roles, and AWS accounts that are given access to the CMK\. The users, roles, and accounts that you choose are added to a default key policy that the console creates for you\. With the console, you can use the default view to view or modify this key policy, or you can work with the key policy document directly\. The default key policy created by the console allows the following permissions, each of which is explained in the corresponding section\.

Permissions

### Allows Access to the AWS Account and Enables IAM Policies<a name="key-policy-default-allow-root-enable-iam"></a>

The default key policy gives the AWS account \(root user\) that owns the CMK full access to the CMK, which accomplishes the following two things\.

**1\. Reduces the risk of the CMK becoming unmanageable\.**  
You cannot delete your AWS account's root user, so allowing access to this user reduces the risk of the CMK becoming unmanageable\. Consider this scenario:  

1. A CMK's key policy allows *only* one IAM user, Alice, to manage the CMK\. This key policy does not allow access to the root user\.

1. Someone deletes IAM user Alice\.
In this scenario, the CMK is now unmanageable, and you must [contact AWS Support](https://console.aws.amazon.com/support/home#/case/create) to regain access to the CMK\. The root user does not have access to the CMK, because the root user can access a CMK only when the key policy explicitly allows it\. This is different from most other resources in AWS, which implicitly allow access to the root user\.

**2\. Enables IAM policies to allow access to the CMK\.**  
IAM policies by themselves are not sufficient to allow access to a CMK\. However, you can use them in combination with a CMK's key policy if the key policy enables it\. Giving the AWS account full access to the CMK does this; it enables you to use IAM policies to give IAM users and roles in the account access to the CMK\. It does not by itself give any IAM users or roles access to the CMK, but it enables you to use IAM policies to do so\. For more information, see [Managing Access to AWS KMS CMKs](control-access-overview.md#managing-access)\.

The following example shows the policy statement that allows access to the AWS account and thereby enables IAM policies\.

```
{
  "Sid": "Enable IAM User Permissions",
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::111122223333:root"},
  "Action": "kms:*",
  "Resource": "*"
}
```

### Allows Key Administrators to Administer the CMK<a name="key-policy-default-allow-administrators"></a>

The default key policy created by the console allows you to choose IAM users and roles in the account and make them *key administrators*\. Key administrators have permissions to manage the CMK, but do not have permissions to use the CMK to encrypt and decrypt data\.

**Warning**  
Even though key administrators do not have permissions to use the CMK to encrypt and decrypt data, they do have permission to modify the key policy\. This means they can give themselves these permissions\.

You can add IAM users and roles to the list of key administrators when you create the CMK\. You can also edit the list with the console's default view for key policies, as shown in the following image\. The default view for key policies is available on the key details page for each CMK\.

![\[Key administrators in the console's default key policy, default view\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-administrators.png)

When you use the console's default view to modify the list of key administrators, the console modifies the `Principal` element in a particular statement in the key policy\. This statement is called the *key administrators statement*\. The following example shows the key administrators statement\.

```
{
  "Sid": "Allow access for Key Administrators",
  "Effect": "Allow",
  "Principal": {"AWS": [
    "arn:aws:iam::111122223333:user/KMSAdminUser",
    "arn:aws:iam::111122223333:role/KMSAdminRole"
  ]},
  "Action": [
    "kms:Create*",
    "kms:Describe*",
    "kms:Enable*",
    "kms:List*",
    "kms:Put*",
    "kms:Update*",
    "kms:Revoke*",
    "kms:Disable*",
    "kms:Get*",
    "kms:Delete*",
    "kms:TagResource",
    "kms:UntagResource",
    "kms:ScheduleKeyDeletion",
    "kms:CancelKeyDeletion"
  ],
  "Resource": "*"
}
```

The key administrators statement allows the following permissions:

+ **kms:Create\*** – Allows key administrators to create aliases and grants for this CMK\.

+ **kms:Describe\*** – Allows key administrators to retrieve information about this CMK including its identifiers, creation date, state, and more\. This permission is necessary to view the key details page in the AWS Management Console\.

+ **kms:Enable\*** – Allows key administrators to set this CMK's state to enabled and to specify annual rotation of the CMK's key material\.

+ **kms:List\*** – Allows key administrators to retrieve lists of the aliases, grants, key policies, and tags for this CMK\. This permission is necessary to view the list of CMKs in the AWS Management Console\.

+ **kms:Put\*** – Allows key administrators to modify the key policy for this CMK\.

+ **kms:Update\*** – Allows key administrators to change the target of an alias to this CMK, and to modify this CMK's description\.

+ **kms:Revoke\*** – Allows key administrators to revoke the permissions for this CMK that are allowed by a grant\.

+ **kms:Disable\*** – Allows key administrators to set this CMK's state to disabled and to disable annual rotation of this CMK's key material\.

+ **kms:Get\*** – Allows key administrators to retrieve the key policy for this CMK and to determine whether this CMK's key material is rotated annually\. If this CMK's origin is external, it also allows key administrators to download the public key and import token for this CMK\. For more information about CMK origin, see [Importing Key Material](importing-keys.md)\.

+ **kms:Delete\*** – Allows key administrators to delete an alias that points to this CMK and, if this CMK's origin is external, to delete the imported key material\. For more information about imported key material, see [Importing Key Material](importing-keys.md)\. Note that this permission does not allow key administrators to delete the CMK\.

+ **kms:ImportKeyMaterial** – Allows key administrators to import key material into the CMK\.
**Note**  
This permission is not shown in the preceding example policy statement\. This permission is applicable only to CMKs whose origin is external\. It is automatically included in the key administrators statement when you use the console to create a CMK with no key material\. For more information, see [Importing Key Material](importing-keys.md)\.

+ **kms:TagResource** – Allows key administrators to add and update tags for this CMK\.

+ **kms:UntagResource** – Allows key administrators to remove tags from this CMK\.

+ **kms:ScheduleKeyDeletion** – Allows key administrators to delete this CMK\.

+ **kms:CancelKeyDeletion** – Allows key administrators to cancel the pending deletion of this CMK\.

The final two permissions in the preceding list, `kms:ScheduleKeyDeletion` and `kms:CancelKeyDeletion`, are included by default when you create a CMK\. However, you can optionally remove them from the key policy when you create a CMK by clearing the box for **Allow key administrators to delete this key**\. In the same way, you can use the key details page to remove them from the default key policy for existing CMKs\. For more information, see [Editing Keys](editing-keys.md)\.

Many of these permissions contain the wildcard character \(`*`\)\. That means that if AWS KMS adds new API operations in the future, key administrators will automatically be allowed to perform all new API operations that begin with Create, Describe, Enable, List, Put, Update, Revoke, Disable, Get, or Delete\.

**Note**  
The key administrators statement described in the preceding section is in the latest version of the default key policy\. For information about previous versions of the default key policy, see [Keeping Key Policies Up to Date](key-policy-upgrading.md)\.

### Allows Key Users to Use the CMK<a name="key-policy-default-allow-users"></a>

The default key policy created by the console allows you to choose IAM users and roles in the account, and external AWS accounts, and make them *key users*\. Key users have permissions to use the CMK directly for encryption and decryption\. They also have permission to delegate a subset of their own permissions to some of the AWS services that are integrated with AWS KMS\. Key users can implicitly give these services permissions to use the CMK in specific and limited ways\. This implicit delegation is done using grants\. For example, key users can do the following things:

+ Use this CMK with Amazon Elastic Block Store \(Amazon EBS\) and Amazon Elastic Compute Cloud \(Amazon EC2\) to attach an encrypted EBS volume to an EC2 instance\. The key user implicitly gives Amazon EC2 permission to use the CMK to attach the encrypted volume to the instance\. For more information, see [How Amazon Elastic Block Store \(Amazon EBS\) Uses AWS KMS](services-ebs.md)\.

+ Use this CMK with Amazon Redshift to launch an encrypted cluster\. The key user implicitly gives Amazon Redshift permission to use the CMK to launch the encrypted cluster and create encrypted snapshots\. For more information, see [How Amazon Redshift Uses AWS KMS](services-redshift.md)\.

+ Use this CMK with other AWS services integrated with AWS KMS, specifically the services that use grants, to create, manage, or use encrypted resources with those services\.

The default key policy gives key users permissions to allow these integrated services to use the CMK, but users also need permission to use the integrated services\. For details about giving users access to an AWS service that integrates with AWS KMS, consult the documentation for the integrated service\.

The default key policy gives key users permissions to use a CMK with *all* of the integrated services that use grants, or none of them\. You cannot use the default key policy to allow key users to use a CMK with some of the integrated services but not others\. However, you can create a custom key policy to do this\. For more information, see the [kms:ViaService](policy-conditions.md#conditions-kms-via-service) condition key\.

You can add IAM users, IAM roles, and external AWS accounts to the list of key users when you create the CMK\. You can also edit the list with the console's default view for key policies, as shown in the following image\. The default view for key policies is on the key details page\.

![\[Key users in the console's default key policy, default view\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-users.png)

When you use the console's default view to modify the list of key users, the console modifies the `Principal` element in two statements in the key policy\. These statements are called the *key users statements*\. The following examples show the key users statements\.

```
{
  "Sid": "Allow use of the key",
  "Effect": "Allow",
  "Principal": {"AWS": [
    "arn:aws:iam::111122223333:user/KMSUser",
    "arn:aws:iam::111122223333:role/KMSRole",
    "arn:aws:iam::444455556666:root"
  ]},
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:ReEncrypt*",
    "kms:GenerateDataKey*",
    "kms:DescribeKey"
  ],
  "Resource": "*"
}
```

```
{
  "Sid": "Allow attachment of persistent resources",
  "Effect": "Allow",
  "Principal": {"AWS": [
    "arn:aws:iam::111122223333:user/KMSUser",
    "arn:aws:iam::111122223333:role/KMSRole",
    "arn:aws:iam::444455556666:root"
  ]},
  "Action": [
    "kms:CreateGrant",
    "kms:ListGrants",
    "kms:RevokeGrant"
  ],
  "Resource": "*",
  "Condition": {"Bool": {"kms:GrantIsForAWSResource": true}}
}
```

The first of these two statements allows key users to use the CMK directly, and includes the following permissions:

+ **kms:Encrypt** – Allows key users to successfully request that AWS KMS encrypt data with this CMK\.

+ **kms:Decrypt** – Allows key users to successfully request that AWS KMS decrypt data with this CMK\.

+ **kms:ReEncrypt\*** – Allows key users to successfully request that AWS KMS re\-encrypt data that was originally encrypted with this CMK, or to use this CMK to re\-encrypt previously encrypted data\. The [ReEncrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) API operation requires access to two CMKs, the original one for decryption and a different one for subsequent encryption\. To accomplish this, you can allow the `kms:ReEncrypt*` permission for both CMKs \(note the wildcard character "`*`" in the permission\)\. Or you can allow the `kms:ReEncryptFrom` permission on the CMK for decryption and the `kms:ReEncryptTo` permission on the CMK for encryption\.

+ **kms:GenerateDataKey\*** – Allows key users to successfully request data encryption keys \(data keys\) to use for client\-side encryption\. Key users can choose to receive two copies of the data key—one in plaintext form and one that is encrypted with this CMK—or to receive only the encrypted form of the data key\.

+ **kms:DescribeKey** – Allows key users to retrieve information about this CMK including its identifiers, creation date, state, and more\.

The second of these two statements allows key users to use grants to delegate a subset of their own permissions to some of the AWS services that are integrated with AWS KMS, specifically the services that use grants\. This policy statement uses a condition element to allow these permissions only when the key user is delegating permissions to an integrated AWS service\. For more information about using conditions in a key policy, see [Using Policy Conditions](policy-conditions.md)\.

## Example Key Policy<a name="key-policy-example"></a>

The following example shows a complete key policy\. This key policy combines the example policy statements from the preceding default key policy section into a single key policy that accomplishes the following:

+ Allows the AWS account \(root user\) 111122223333 full access to the CMK, and thus enables IAM policies in the account to allow access to the CMK\.

+ Allows IAM user KMSAdminUser and IAM role KMSAdminRole to administer the CMK\.

+ Allows IAM user KMSUser, IAM role KMSRole, and AWS account 444455556666 to use the CMK\.

```
{
  "Version": "2012-10-17",
  "Id": "key-consolepolicy-2",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::111122223333:root"},
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow access for Key Administrators",
      "Effect": "Allow",
      "Principal": {"AWS": [
        "arn:aws:iam::111122223333:user/KMSAdminUser",
        "arn:aws:iam::111122223333:role/KMSAdminRole"
      ]},
      "Action": [
        "kms:Create*",
        "kms:Describe*",
        "kms:Enable*",
        "kms:List*",
        "kms:Put*",
        "kms:Update*",
        "kms:Revoke*",
        "kms:Disable*",
        "kms:Get*",
        "kms:Delete*",
        "kms:TagResource",
        "kms:UntagResource",
        "kms:ScheduleKeyDeletion",
        "kms:CancelKeyDeletion"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Allow use of the key",
      "Effect": "Allow",
      "Principal": {"AWS": [
        "arn:aws:iam::111122223333:user/KMSUser",
        "arn:aws:iam::111122223333:role/KMSRole",
        "arn:aws:iam::444455556666:root"
      ]},
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Allow attachment of persistent resources",
      "Effect": "Allow",
      "Principal": {"AWS": [
        "arn:aws:iam::111122223333:user/KMSUser",
        "arn:aws:iam::111122223333:role/KMSRole",
        "arn:aws:iam::444455556666:root"
      ]},
      "Action": [
        "kms:CreateGrant",
        "kms:ListGrants",
        "kms:RevokeGrant"
      ],
      "Resource": "*",
      "Condition": {"Bool": {"kms:GrantIsForAWSResource": "true"}}
    }
  ]
}
```

The following image shows an example of what this key policy looks like when viewed with the console's default view for key policies\.

![\[Key policy in the console's default view for key policies\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-full.png)