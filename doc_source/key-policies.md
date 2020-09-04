# Using key policies in AWS KMS<a name="key-policies"></a>

Key policies are the primary way to control access to customer master keys \(CMKs\) in AWS KMS\. They are not the only way to control access, but you cannot control access without them\. For more information, see [Managing access to AWS KMS CMKs](control-access-overview.md#managing-access)\.

**Topics**
+ [Overview of key policies](#key-policy-overview)
+ [Default key policy](#key-policy-default)
+ [Example key policy](#key-policy-example)

For help writing and formatting a JSON policy document, see the [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) in the *IAM User Guide*\.

## Overview of key policies<a name="key-policy-overview"></a>

A key policy is a document that uses [JSON \(JavaScript Object Notation\)](http://json.org/) to specify permissions\. You can work with these JSON documents directly, or you can use the AWS Management Console to work with them using a graphical interface called the *default view*\. 

For more information about the console's default view for key policies, see [Default key policy](#key-policy-default) and [Changing a key policy](key-policy-modifying.md)\. For help writing and formatting a JSON policy document, see the [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) in the *IAM User Guide*\.

A key policy document cannot exceed 32 KB \(32,768 bytes\)\. Key policy documents use the same JSON syntax as other policy documents in AWS and have the following basic structure:

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
+ **Principal** – \(Required\) The [principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#Principal_specifying) is the identity that gets the permissions specified in the policy statement\. You can specify AWS accounts \(root\), IAM users, IAM roles, and some AWS services as principals in a key policy\. IAM groups are not valid principals\.
**Note**  
Do not set the Principal to an asterisk \(\*\) in any key policy statement that allows permissions\. An asterisk gives every identity in every AWS account permission to use the CMK, unless another policy statement explicitly denies it\. Users in other AWS accounts just need corresponding IAM permissions in their own accounts to use the CMK\.
+ **Action** – \(Required\) Actions specify the API operations to allow or deny\. For example, the `kms:Encrypt` action corresponds to the AWS KMS [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation\. You can list more than one action in a policy statement\. For more information, see [AWS KMS API permissions reference](kms-api-permissions-reference.md)\.
+ **Resource** – \(Required\) In a key policy, the value of the Resource element is `"*"`, which means "this CMK\." The asterisk \(`"*"`\) identifies the CMK to which the key policy is attached\.
+ **Condition** – \(Optional\) Conditions specify requirements that must be met for a key policy to take effect\. With conditions, AWS can evaluate the context of an API request to determine whether or not the policy statement applies\. For more information, see [Using policy conditions](policy-conditions.md)\.

For more information about AWS policy syntax, see [AWS IAM Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) in the *IAM User Guide*\.

## Default key policy<a name="key-policy-default"></a>

**Default key policy when you create a CMK programmatically**  
When you create a CMK programmatically—that is, with the [AWS KMS API](https://docs.aws.amazon.com/kms/latest/APIReference/) \(including through the [AWS SDKs](https://aws.amazon.com/tools/#sdk) and [command line tools](https://aws.amazon.com/tools/#cli)\)—you have the option of providing the key policy for the new CMK\. If you don't provide one, AWS KMS creates one for you\. This default key policy has one policy statement that gives the AWS account \(root user\) that owns the CMK full access to the CMK and enables IAM policies in the account to allow access to the CMK\. For more information about this policy statement, see [Allows access to the AWS account and enables IAM policies](#key-policy-default-allow-root-enable-iam)\.

**Default key policy when you create a CMK with the AWS Management Console**  
When you [create a CMK with the AWS Management Console](create-keys.md), you can choose the IAM users, IAM roles, and AWS accounts that are given access to the CMK\. The users, roles, and accounts that you choose are added to a default key policy that the console creates for you\. With the console, you can use the default view to view or modify this key policy, or you can work with the key policy document directly\. The default key policy created by the console allows the following permissions, each of which is explained in the corresponding section\.

**Permissions**
+ [Allows access to the AWS account and enables IAM policies](#key-policy-default-allow-root-enable-iam)
+ [Allows key administrators to administer the CMK](#key-policy-default-allow-administrators)
+ [Allows key users to use the CMK](#key-policy-default-allow-users)
  + [Allows key users to use a CMK for cryptographic operations](#key-policy-users-crypto)
  + [Allows key users to use the CMK with AWS services](#key-policy-service-integration)

### Allows access to the AWS account and enables IAM policies<a name="key-policy-default-allow-root-enable-iam"></a>

The default key policy gives the AWS account \(root user\) that owns the CMK full access to the CMK, which accomplishes the following two things\.

**1\. Reduces the risk of the CMK becoming unmanageable\.**  
You cannot delete your AWS account's root user, so allowing access to this user reduces the risk of the CMK becoming unmanageable\. Consider this scenario:  

1. A CMK's key policy allows *only* one IAM user, Alice, to manage the CMK\. This key policy does not allow access to the root user\.

1. Someone deletes IAM user Alice\.
In this scenario, the CMK is now unmanageable, and you must [contact AWS Support](https://console.aws.amazon.com/support/home#/case/create) to regain access to the CMK\. The root user does not have access to the CMK, because the root user can access a CMK only when the key policy explicitly allows it\. This is different from most other resources in AWS, which implicitly allow access to the root user\.

**2\. Enables IAM policies to allow access to the CMK\.**  
IAM policies by themselves are not sufficient to allow access to a CMK\. However, you can use them in combination with a CMK's key policy if the key policy enables it\. Giving the AWS account full access to the CMK does this; it enables you to use IAM policies to give IAM users and roles in the account access to the CMK\. It does not by itself give any IAM users or roles access to the CMK, but it enables you to use IAM policies to do so\. For more information, see [Managing access to AWS KMS CMKs](control-access-overview.md#managing-access)\.

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

### Allows key administrators to administer the CMK<a name="key-policy-default-allow-administrators"></a>

The default key policy created by the console allows you to choose IAM users and roles in the account and make them *key administrators*\. Key administrators have permissions to manage the CMK, but do not have permissions to use the CMK in [cryptographic operations](concepts.md#cryptographic-operations)\.

**Warning**  
Even though key administrators do not have permissions to use the CMK to encrypt and decrypt data, they do have permission to change the key policy\. This means they can give themselves any AWS KMS permission\.

You can add IAM users and roles to the list of key administrators when you create the CMK\. You can also edit the list with the console's default view for key policies, as shown in the following image\. The default view for key policies is available on the key details page for each CMK\.

![\[Key administrators in the console's default key policy, default view\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-administrators-sm.png)

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
+ **kms:Create\*** – Allows key administrators to create aliases and [grants](grants.md) for this CMK\.
+ **kms:Describe\*** – Allows key administrators to get information about this CMK including its identifiers, creation date, state, and more\. This permission is necessary to view the key details page in the AWS Management Console\.
+ **kms:Enable\*** – Allows key administrators to set this CMK's state to enabled\. For symmetric CMKs, it allows key administrators to specify [annual rotation of the CMK's key material](rotate-keys.md)\.
+ **kms:List\*** – Allows key administrators to get lists of the aliases, grants, key policies, and tags for this CMK\. This permission is necessary to view the list of CMKs in the AWS Management Console\.
+ **kms:Put\*** – Allows key administrators to change the key policy for this CMK\.
+ **kms:Update\*** – Allows key administrators to change the target of an alias to this CMK, and to change this CMK's description\.
+ **kms:Revoke\*** – Allows key administrators to revoke the permissions for this CMK that are allowed by a [grant](grants.md)\.
+ **kms:Disable\*** – Allows key administrators to set this CMK's key state to disabled\. For symmetric CMKs, it allows key administrators to disable [annual rotation of this CMK's key material](rotate-keys.md)\.
+ **kms:Get\*** – Allows key administrators to get the key policy for this CMK and to determine whether this CMK's key material is rotated annually\. For [symmetric CMKs](symm-asymm-concepts.md#symmetric-cmks) with [imported key material](importing-keys.md), it also allows key administrators to download the import token and public key that they need to import key material into the CMK\. For [asymmetric CMKs](symm-asymm-concepts.md#asymmetric-cmks), it allows key administrators to [download the public key](download-public-key.md) of the CMK\.
+ **kms:Delete\*** – Allows key administrators to delete an alias that is associated with this CMK\. For symmetric CMKs with [imported key material](importing-keys.md), it lets the key administrator, delete the imported key material\. Note that this permission does not allow key administrators to [delete the CMK](deleting-keys.md)\.
+ **kms:ImportKeyMaterial** – Allows key administrators to import key material into the CMK\. This permission is included in the key policy only when you [create a CMK with no key material](importing-keys-create-cmk.md)\.
**Note**  
This permission is not shown in the preceding example policy statement\.
+ **kms:TagResource** – Allows key administrators to add and update tags for this CMK\.
+ **kms:UntagResource** – Allows key administrators to remove tags from this CMK\.
+ **kms:ScheduleKeyDeletion** – Allows key administrators to [delete this CMK](deleting-keys.md)\.
+ **kms:CancelKeyDeletion** – Allows key administrators to cancel the pending deletion of this CMK\.

The final two permissions in the preceding list, `kms:ScheduleKeyDeletion` and `kms:CancelKeyDeletion`, are included by default when you [create a CMK](create-keys.md)\. However, you can optionally remove them from the key policy when you create a CMK by clearing the box for **Allow key administrators to delete this key**\. In the same way, you can use the key details page to remove them from the default key policy for existing CMKs\. For more information, see [Editing keys](editing-keys.md)\.

Many of these permissions contain the wildcard character \(`*`\)\. That means that if AWS KMS adds new API operations in the future, key administrators will automatically be allowed to perform all new API operations that begin with Create, Describe, Enable, List, Put, Update, Revoke, Disable, Get, or Delete\.

**Note**  
The key administrators statement described in the preceding section is in the latest version of the default key policy\. For information about previous versions of the default key policy, see [Keeping key policies up to date](key-policy-upgrading.md)\.

### Allows key users to use the CMK<a name="key-policy-default-allow-users"></a>

The default key policy that the console creates for symmetric CMKs allows you to choose IAM users and roles in the account, and external AWS accounts, and make them *key users*\. 

The console adds two policy statements to the key policy for key users\.
+ [Use the CMK directly](#key-policy-users-crypto) — The first key policy statement gives key users permission to use the CMK directly for all supported [cryptographic operations](concepts.md#cryptographic-operations) for that type of CMK\.
+ [Use the CMK with AWS services](#key-policy-service-integration) — The second policy statement gives key users permission to allow AWS services that are integrated with AWS KMS to use the CMK on their behalf to protect resources, such as [Amazon Simple Storage Service buckets](services-s3.md) and [Amazon DynamoDB tables](services-dynamodb.md)\.

You can add IAM users, IAM roles, and other AWS accounts to the list of key users when you create the CMK\. You can also edit the list with the console's default view for key policies, as shown in the following image\. The default view for key policies is on the key details page\. For more information about allowing users in other AWS accounts to use the CMK, see [Allowing users in other accounts to use a CMK](key-policy-modifying-external-accounts.md)\.

![\[Key users in the console's default key policy, default view\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-users-sm.png)

When you use the console's default view to change the list of key users, the console changes the `Principal` element in two statements in the key policy\. These statements are called the *key users statements*\. The following examples show the key users statements for symmetric CMKs\.

```
{
  "Sid": "Allow use of the key",
  "Effect": "Allow",
  "Principal": {"AWS": [
    "arn:aws:iam::111122223333:user/CMKUser",
    "arn:aws:iam::111122223333:role/CMKRole",
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
    "arn:aws:iam::111122223333:user/CMKUser",
    "arn:aws:iam::111122223333:role/CMKRole",
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

### Allows key users to use a CMK for cryptographic operations<a name="key-policy-users-crypto"></a>

Key users have permission to use the CMK directly in all [cryptographic operations](concepts.md#cryptographic-operations) supported on the CMK\. They can also use the [DescribeKey ](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html)operation to get detailed information about the CMK in the AWS KMS console or by using the AWS KMS API operations\.

By default, the AWS KMS console adds key users statements like those in the following examples to the default key policy\. Because they support different API operations, the actions in the policy statements for symmetric CMKs, asymmetric CMKs for public key encryption, and asymmetric CMKs for signing and verification are slightly different\.

**Symmetric CMKs**  
The console adds the following statement to the key policy for symmetric CMKs\.  

```
{
  "Sid": "Allow use of the key",
  "Effect": "Allow",  
  "Principal": {"AWS": "arn:aws:iam::111122223333:user/CMKUser"},
  "Action": [
    "kms:Decrypt",
    "kms:DescribeKey",
    "kms:Encrypt",
    "kms:GenerateDataKey*",
    "kms:ReEncrypt*"
  ],
  "Resource": "*"
}
```

**Asymmetric CMKs for public key encryption**  
The console adds the following statement to the key policy for asymmetric CMKs with a key usage of **Encrypt and decrypt**\.  

```
{
    "Sid": "Allow use of the key",
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::111122223333:user/CMKUser"},
    "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:DescribeKey",
        "kms:GetPublicKey"
    ],
    "Resource": "*"
}
```

**Asymmetric CMKs for signing and verification**  
The console adds the following statement to the key policy for asymmetric CMKs with a key usage of **Sign and verify**\.  

```
{
    "Sid": "Allow use of the key",
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::111122223333:user/CMKUser"},
    "Action": [
        "kms:DescribeKey",
        "kms:GetPublicKey",
        "kms:Sign",
        "kms:Verify"
    ],
    "Resource": "*"
}
```

The actions in these statements give the key users some of the following permissions\.
+ [kms:Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) – Allows key users to encrypt data with this CMK\.
+ [kms:Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) – Allows key users to decrypt data with this CMK\.
+ [kms:DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) – Allows key users to get detailed information about this CMK including its identifiers, creation date, and key state\. It also allows the key users to display details about the CMK in the AWS KMS console\.
+ **kms:GenerateDataKey\*** – Allows key users to request a symmetric data key or an asymmetric data key pair for client\-side cryptographic operations\. The console uses the \* wildcard character to represent permission for the following API operations: [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html), and [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html)\.
+ [kms:GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) – Allows key users to download the public key of the asymmetric CMK\. Parties with whom you share this public key can encrypt data outside of AWS KMS\. However, those ciphertexts can be decrypted only by calling the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation in AWS KMS\.
+ [kms:ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)\* – Allows key users to re\-encrypt data that was originally encrypted with this CMK, or to use this CMK to re\-encrypt previously encrypted data\. The [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) operation requires access to both source and destination CMKs\. To accomplish this, you can allow the `kms:ReEncryptFrom` permission on the source CMK and `kms:ReEncryptTo` permission on the destination CMK\. However, for simplicity, the console allows `kms:ReEncrypt*` \(with the `*` wildcard character\) on both CMKs\.
+ [kms:Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html) – Allows key users to sign messages with this CMK\.
+ [kms:Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) – Allows key users to verify signatures with this CMK\.

### Allows key users to use the CMK with AWS services<a name="key-policy-service-integration"></a>

The default key policy in the console also gives key users permission to allow [AWS services that are integrated with AWS KMS](service-integration.md) to use the CMK, particularly services that use grants\. 

Key users can implicitly give these services permissions to use the CMK in specific and limited ways\. This implicit delegation is done using [grants](grants.md)\. These grants allow the integrated AWS service to use the CMK to protect resources in the account\.

```
{
  "Sid": "Allow attachment of persistent resources",
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::111122223333:user/CMKUser"},
  "Action": [
    "kms:CreateGrant",
    "kms:ListGrants",
    "kms:RevokeGrant"
  ],
  "Resource": "*",
  "Condition": {"Bool": {"kms:GrantIsForAWSResource": true}}
}
```

For example, key users can use these permissions on the CMK in the following ways\.
+ Use this CMK with Amazon Elastic Block Store \(Amazon EBS\) and Amazon Elastic Compute Cloud \(Amazon EC2\) to attach an encrypted EBS volume to an EC2 instance\. The key user implicitly gives Amazon EC2 permission to use the CMK to attach the encrypted volume to the instance\. For more information, see [How Amazon Elastic Block Store \(Amazon EBS\) uses AWS KMS](services-ebs.md)\.
+ Use this CMK with Amazon Redshift to launch an encrypted cluster\. The key user implicitly gives Amazon Redshift permission to use the CMK to launch the encrypted cluster and create encrypted snapshots\. For more information, see [How Amazon Redshift uses AWS KMS](services-redshift.md)\.
+ Use this CMK with other [AWS services integrated with AWS KMS](service-integration.md), specifically the services that use grants, to create, manage, or use encrypted resources with those services\.

The [kms:GrantIsForAWSResource](policy-conditions.md#conditions-kms-grant-is-for-aws-resource) condition key allows key users to create and manage grants, but only when the grantee is an AWS service that uses grants\. The permission allows key users to use *all* of the integrated services that use grants\. However, you can create a custom key policy that allows particular AWS services to use the CMK on the key user's behalf\. For more information, see the [kms:ViaService](policy-conditions.md#conditions-kms-via-service) condition key\.

Key users need these grant permissions to use their CMK with integrated services, but these permissions are not sufficient\. Key users also need permission to use the integrated services\. For details about giving users access to an AWS service that integrates with AWS KMS, consult the documentation for the integrated service\.

## Example key policy<a name="key-policy-example"></a>

The following example shows a complete key policy for a symmetric CMK\. This key policy combines the example policy statements from the preceding [default key policy](#key-policy-default) section into a single key policy that accomplishes the following:
+ Allows the AWS account \(root user\) 111122223333 full access to the CMK, and thus enables IAM policies in the account to allow access to the CMK\.
+ Allows IAM user KMSAdminUser and IAM role KMSAdminRole to administer the CMK\.
+ Allows IAM user CMKUser, IAM role CMKRole, and AWS account 444455556666 to use the CMK\.

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
        "arn:aws:iam::111122223333:user/CMKUser",
        "arn:aws:iam::111122223333:role/CMKRole",
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
        "arn:aws:iam::111122223333:user/CMKUser",
        "arn:aws:iam::111122223333:role/CMKRole",
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

![\[Key policy in the console's default view for key policies\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-full-sm.png)