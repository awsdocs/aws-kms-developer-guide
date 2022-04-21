# Default key policy<a name="key-policy-default"></a>

When you create a KMS key, you can specify the key policy for the new KMS key\. If you don't provide one, AWS KMS creates one for you\. The default key policy that AWS KMS uses differs depending on whether you create the key in the AWS KMS console or you use the AWS KMS API\.

**Default key policy when you create a KMS key programmatically**  
When you create a KMS key programmatically with the [AWS KMS API](https://docs.aws.amazon.com/kms/latest/APIReference/) \(including by using the [AWS SDKs](https://aws.amazon.com/tools/#sdk), [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/) or [AWS Tools for PowerShell](https://docs.aws.amazon.com/powershell/latest/userguide/)\), and you don't specify a key policy, AWS KMS applies a very simple default key policy\. This default key policy has one policy statement that gives the AWS account that owns the KMS key permission to use IAM policies to allow access to all AWS KMS operations on the KMS key\. For more information about this policy statement, see [Allows access to the AWS account and enables IAM policies](#key-policy-default-allow-root-enable-iam)\.

**Default key policy when you create a KMS key with the AWS Management Console**  
When you [create a KMS key with the AWS Management Console](create-keys.md), the key policy begins with the policy statement that [allows access to the AWS account and enables IAM policies](#key-policy-default-allow-root-enable-iam)\. The console then adds a [key administrators statement](#key-policy-default-allow-administrators), a [key users statement](#key-policy-default-allow-users), and \(for most key types\) a statement that allows principals to use the KMS key with [other AWS services](#key-policy-service-integration)\. You can use the features of the AWS KMS console to specify the IAM users and roles and AWS accounts who are key administrators and those who are key users \(or both\)\.

**Permissions**
+ [Allows access to the AWS account and enables IAM policies](#key-policy-default-allow-root-enable-iam)
+ [Allows key administrators to administer the KMS key](#key-policy-default-allow-administrators)
+ [Allows key users to use the KMS key](#key-policy-default-allow-users)
  + [Allows key users to use a KMS key for cryptographic operations](#key-policy-users-crypto)
  + [Allows key users to use the KMS key with AWS services](#key-policy-service-integration)

## Allows access to the AWS account and enables IAM policies<a name="key-policy-default-allow-root-enable-iam"></a>

The following default key policy statement is critical\. 
+ It gives the AWS account that owns the KMS key full access to the KMS key\. 

  Unlike other AWS resource policies, a AWS KMS key policy does not automatically give permission to the account or any of its users\. To give permission to account administrators, the key policy must include an explicit statement that provides this permission, like this one\.
+ It allows the account to use IAM policies to allow access to the KMS key, in addition to the key policy\.

  Without this permission, IAM policies that allow access to the key are ineffective, although IAM policies that deny access to the key are still effective\. 
+ It reduces the risk of the key becoming unmanageable by giving access control permission to the account administrators, including the account root user, which cannot be deleted\. 

The following key policy statement is the entire default key policy for KMS keys created programmatically\. It's the first policy statement in the default key policy for KMS keys created in the AWS KMS console\.

```
{
  "Sid": "Enable IAM policies",
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:root"
   },
  "Action": "kms:*",
  "Resource": "*"
}
```

**Allows IAM policies to allow access to the KMS key\.**  <a name="allow-iam-policies"></a>
The key policy statement shown above gives the AWS account that owns the key permission to use IAM policies, as well as key policies, to allow all actions \(`kms:*`\) on the KMS key\.   
The principal in this key policy statement is the [account principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-accounts), which is represented by an ARN in this format: `arn:aws:iam::account-id:root`\. The account principal represents the AWS account and its administrators\.   
When the principal in a key policy statement is the account principal, the policy statement doesn't give any IAM users or roles permission to use the KMS key\. Instead, it allows the account to use IAM policies to *delegate* the permissions specified in the policy statement\. This default key policy statement allows the account to use IAM policies to delegate permission for all actions \(`kms:*`\) on the KMS key\.

**Reduces the risk of the KMS key becoming unmanageable\.**  
Unlike other AWS resource policies, a AWS KMS key policy does not automatically give permission to the account or any of its users\. To give permission to any principal, including the [account principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-accounts), you must use a key policy statement that provides the permission explicitly\. You are not required to give the account principal, or any principal, access to the KMS key\. However, giving access to the account principal helps you prevent the key from becoming unmanageable\.  
For example, suppose you create a key policy that gives only one user access to the KMS key\. If you then delete that user, the key becomes unmanageable and you must [contact AWS Support](https://console.aws.amazon.com/support/home#/case/create) to regain access to the KMS key\.   
The key policy statement shown above gives permission to control the key to the [account principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-accounts), which represents the AWS account and its administrators, including the [account root user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html)\. The account root user is the only principal that cannot be deleted unless you delete the AWS account\. IAM best practices discourage acting on behalf of the account root user, except in an emergency\. However, you might need to act as the account root user if you delete all other users and roles with access to the KMS key\. 

## Allows key administrators to administer the KMS key<a name="key-policy-default-allow-administrators"></a>

The default key policy created by the console allows you to choose IAM users and roles in the account and make them *key administrators*\. This statement is called the *key administrators statement*\. Key administrators have permissions to manage the KMS key, but do not have permissions to use the KMS key in [cryptographic operations](concepts.md#cryptographic-operations)\. You can add IAM users and roles to the list of key administrators when you create the KMS key in the default view or the policy view\. 

**Warning**  
Because key administrators have permission to change the key policy and create grants, they can give themselves and others AWS KMS permissions not specified in this policy\.  
Principals who have permission to manage tags and aliases can also control access to a KMS key\. For details, see [ABAC for AWS KMS](abac.md)\.

The following example shows the key administrators statement in the default view of the AWS KMS console\.

![\[Key administrators in the console's default key policy, default view\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-administrators-sm.png)

The following is an example key administrators statement in the policy view of the AWS KMS console\. This key administrators statement is for a single\-region symmetric encryption KMS key\.

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

The default key administrators statement for the most common KMS key, a single\-Region symmetric encryption KMS key, allows the following permissions\. For detailed information about each permission, see the [AWS KMS permissions](kms-api-permissions-reference.md)\.

When you use the AWS KMS console to create a KMS key, the console adds the users and roles you specify to the `Principal` element in the key administrators statement\.

Many of these permissions contain the wildcard character \(`*`\), which allows all permissions that begin with the specified verb\. As a result, when AWS KMS adds new API operations, key administrators are automatically allowed to use them\. You don't have to update your key policies to include the new operations\. If you prefer to limit your key administrators to a fixed set of API operations, you can [change your key policy](key-policy-modifying.md)\.

**`kms:Create*`**  
Allows [`kms:CreateAlias`](kms-alias.md) and [`kms:CreateGrant`](grants.md)\. \(The `kms:CreateKey` permission is valid only in an IAM policy\.\)

**`kms:Describe*`**  
Allows [`kms:DescribeKey`](viewing-keys.md)\. The `kms:DescribeKey` permission is required to view the key details page for a KMS key in the AWS Management Console\.

**`kms:Enable*`**  
Allows [`kms:EnableKey`](enabling-keys.md)\. For symmetric encryption KMS keys, it also allows [`kms:EnableKeyRotation`](rotate-keys.md)\.

**`kms:List*`**  
Allows [`kms:ListGrants`](grants.md), [https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeyPolicies.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeyPolicies.html), and [`kms:ListResourceTags`](tagging-keys.md)\. \(The `kms:ListAliases` and `kms:ListKeys` permissions, which are required to view KMS keys in the AWS Management Console, are valid only in IAM policies\.\)

**`kms:Put*`**  
Allows [https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html)\. This permission allows key administrators to change the key policy for this KMS key\.

**`kms:Update*`**  
Allows [`kms:UpdateAlias`](alias-manage.md#alias-update) and [`kms:UpdateKeyDescription`](editing-keys.md)\. For multi\-Region keys, it allows [`kms:UpdatePrimaryRegion`](multi-region-keys-manage.md#update-primary-console) on this KMS key\.

**`kms:Revoke*`**  
Allows [`kms:RevokeGrant`](grant-manage.md#grant-delete), which allows key administrators to [delete a grant](grant-manage.md#grant-delete) even if they are not a [retiring principal](grants.md#terms-retiring-principal) in the grant\. 

**`kms:Disable*`**  
Allows [`kms:DisableKey`](enabling-keys.md)\. For symmetric encryption KMS keys, it also allows [`kms:DisableKeyRotation`](rotate-keys.md)\.

**`kms:Get*`**  
Allows [`kms:GetKeyPolicy`](key-policy-viewing.md) and [`kms:GetKeyRotationStatus`](rotate-keys.md)\. For KMS keys with imported key material, it allows [https://docs.aws.amazon.com/kms/latest/APIReference/API_GetParametersForImport.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetParametersForImport.html)\. For asymmetric KMS keys, it allows [https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html)\. The `kms:GetKeyPolicy` permission is required to view the key policy of a KMS key in the AWS Management Console\.

**`kms:Delete*`**  
Allows [`kms:DeleteAlias`](kms-alias.md)\. For keys with imported key material, it allows [`kms:DeleteImportedKeyMaterial`](importing-keys.md)\. The `kms:Delete*` permission does not allow key administrators to delete the KMS key \(`ScheduleKeyDeletion`\)\.

**`kms:TagResource`**  
Allows [`kms:TagResource`](tagging-keys.md), which allows key administrators to add tags to the KMS key\. Because tags can also be used to control access to the KMS key, this permission can allow administrators to allow or deny access to the KMS key\. For details, see [ABAC for AWS KMS](abac.md)\.

**`kms:UntagResource`**  
Allows [`kms:UntagResource`](tagging-keys.md), which allows key administrators to delete tags from the KMS key\. Because tags can be used to control access to the key, this permission can allow administrators to allow or deny access to the KMS key\. For details, see [ABAC for AWS KMS](abac.md)\.

**`kms:ScheduleKeyDeletion`**  
Allows [https://docs.aws.amazon.com/kms/latest/APIReference/API_ScheduleKeyDeletion.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_ScheduleKeyDeletion.html), which allows key administrators to [delete this KMS key](deleting-keys.md)\. To delete this permission, clear the **Allow key administrators to delete this key** option\.

**`kms:CancelKeyDeletion`**  
Allows [https://docs.aws.amazon.com/kms/latest/APIReference/API_CancelKeyDeletion.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_CancelKeyDeletion.html), which allows key administrators to [cancel deletion of this KMS key](deleting-keys.md)\. To delete this permission, clear the **Allow key administrators to delete this key** option\.

 

AWS KMS adds the following permissions to the default key administrators statement when you create [special\-purpose keys](key-types.md)\.

**`kms:ImportKeyMaterial`**  
The [https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html) permission allows key administrators to import key material into the KMS key\. This permission is included in the key policy only when you [create a KMS key with no key material](importing-keys-create-cmk.md)\.

**`kms:ReplicateKey`**  
The [https://docs.aws.amazon.com/kms/latest/APIReference/API_ReplicateKey.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReplicateKey.html) permission allows key administrators to [create a replica of a multi\-Region primary key](multi-region-keys-replicate.md) in a different AWS Region\. This permission is included in the key policy only when you create a multi\-Region primary or replica key\.

**`kms:UpdatePrimaryRegion`**  
The [https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdatePrimaryRegion.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdatePrimaryRegion.html) permission allows key administrators to [change a multi\-Region replica key to a multi\-Region primary key](multi-region-keys-manage.md#multi-region-update)\. This permission is included in the key policy only when you create a multi\-Region primary or replica key\.

## Allows key users to use the KMS key<a name="key-policy-default-allow-users"></a>

The default key policy that the console creates for KMS keys allows you to choose IAM users and roles in the account, and external AWS accounts, and make them *key users*\. 

The console adds two policy statements to the key policy for key users\.
+ [Use the KMS key directly](#key-policy-users-crypto) — The first key policy statement gives key users permission to use the KMS key directly for all supported [cryptographic operations](concepts.md#cryptographic-operations) for that type of KMS key\.
+ [Use the KMS key with AWS services](#key-policy-service-integration) — The second policy statement gives key users permission to allow AWS services that are integrated with AWS KMS to use the KMS key on their behalf to protect resources, such as [Amazon Simple Storage Service buckets](services-s3.md) and [Amazon DynamoDB tables](services-dynamodb.md)\.

You can add IAM users, IAM roles, and other AWS accounts to the list of key users when you create the KMS key\. You can also edit the list with the console's default view for key policies, as shown in the following image\. The default view for key policies is on the key details page\. For more information about allowing users in other AWS accounts to use the KMS key, see [Allowing users in other accounts to use a KMS key](key-policy-modifying-external-accounts.md)\.

![\[Key users in the console's default key policy, default view\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-users-sm.png)

The default *key users statements* for a single\-Region symmetric allows the following permissions\. For detailed information about each permission, see the [AWS KMS permissions](kms-api-permissions-reference.md)\.

When you use the AWS KMS console to create a KMS key, the console adds the users and roles you specify to the `Principal` element in each key users statement\.

```
{
  "Sid": "Allow use of the key",
  "Effect": "Allow",
  "Principal": {"AWS": [
    "arn:aws:iam::111122223333:user/ExampleUser",
    "arn:aws:iam::111122223333:role/ExampleRole",
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
    "arn:aws:iam::111122223333:user/ExampleUser",
    "arn:aws:iam::111122223333:role/ExampleRole",
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

## Allows key users to use a KMS key for cryptographic operations<a name="key-policy-users-crypto"></a>

Key users have permission to use the KMS key directly in all [cryptographic operations](concepts.md#cryptographic-operations) supported on the KMS key\. They can also use the [DescribeKey ](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html)operation to get detailed information about the KMS key in the AWS KMS console or by using the AWS KMS API operations\.

By default, the AWS KMS console adds key users statements like those in the following examples to the default key policy\. Because they support different API operations, the actions in the policy statements for symmetric encryption KMS keys, HMAC KMS keys, asymmetric KMS keys for public key encryption, and asymmetric KMS keys for signing and verification are slightly different\.

**Symmetric encryption KMS keys**  
The console adds the following statement to the key policy for symmetric encryption KMS keys\.  

```
{
  "Sid": "Allow use of the key",
  "Effect": "Allow",  
  "Principal": {"AWS": "arn:aws:iam::111122223333:user/ExampleUser"},
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

**HMAC KMS keys**  
The console adds the following statement to the key policy for HMAC KMS keys\.  

```
{
  "Sid": "Allow use of the key",
  "Effect": "Allow",  
  "Principal": {"AWS": "arn:aws:iam::111122223333:user/ExampleUser"},
  "Action": [
    "kms:DescribeKey",
    "kms:GenerateMac",
    "kms:VerifyMac"
  ],
  "Resource": "*"
}
```

**Asymmetric KMS keys for public key encryption**  
The console adds the following statement to the key policy for asymmetric KMS keys with a key usage of **Encrypt and decrypt**\.  

```
{
  "Sid": "Allow use of the key",
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
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

**Asymmetric KMS keys for signing and verification**  
The console adds the following statement to the key policy for asymmetric KMS keys with a key usage of **Sign and verify**\.  

```
{
  "Sid": "Allow use of the key",
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::111122223333:user/ExampleUser"},
  "Action": [
    "kms:DescribeKey",
    "kms:GetPublicKey",
    "kms:Sign",
    "kms:Verify"
  ],
  "Resource": "*"
}
```

The actions in these statements give the key users the following permissions\.

[https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html)  
Allows key users to encrypt data with this KMS key\.

[https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)  
Allows key users to decrypt data with this KMS key\.

[https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html)  
Allows key users to get detailed information about this KMS key including its identifiers, creation date, and key state\. It also allows the key users to display details about the KMS key in the AWS KMS console\.

`kms:GenerateDataKey*`  
Allows key users to request a symmetric data key or an asymmetric data key pair for client\-side cryptographic operations\. The console uses the \* wildcard character to represent permission for the following API operations: [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html), and [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html)\. These permissions are valid only on the symmetric KMS keys that encrypt the data keys\.

[kms:GenerateMac](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateMac.html)  
Allows key users to use an HMAC KMS key to generate an HMAC tag\.

[kms:GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html)  
Allows key users to download the public key of the asymmetric KMS key\. Parties with whom you share this public key can encrypt data outside of AWS KMS\. However, those ciphertexts can be decrypted only by calling the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation in AWS KMS\.

[kms:ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)\*   
Allows key users to re\-encrypt data that was originally encrypted with this KMS key, or to use this KMS key to re\-encrypt previously encrypted data\. The [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) operation requires access to both source and destination KMS keys\. To accomplish this, you can allow the `kms:ReEncryptFrom` permission on the source KMS key and `kms:ReEncryptTo` permission on the destination KMS key\. However, for simplicity, the console allows `kms:ReEncrypt*` \(with the `*` wildcard character\) on both KMS keys\.

[kms:Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html)  
Allows key users to sign messages with this KMS key\.

[kms:Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html)  
Allows key users to verify signatures with this KMS key\.

[kms:VerifyMac](https://docs.aws.amazon.com/kms/latest/APIReference/API_VerifyMac.html)  
Allows key users to use an HMAC KMS key to verify an HMAC tag\.

## Allows key users to use the KMS key with AWS services<a name="key-policy-service-integration"></a>

The default key policy in the console also gives key users the grant permissions they need to protect their data in AWS services that use grants\. AWS services often use grants to get specific and limited permission to use a KMS key\.

This key policy statement allows the key user to create, view, and revoke grants on the KMS key, but only when the grant operation request comes from an [AWS service integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration)\. The [kms:GrantIsForAWSResource](policy-conditions.md#conditions-kms-grant-is-for-aws-resource) policy condition doesn't allow the user to call these grant operations directly\. When the key user allows it, an AWS service can create a grant on the user's behalf that allows the service to use the KMS key to protect the user's data\. 

Key users require these grant permissions to use their KMS key with integrated services, but these permissions are not sufficient\. Key users also need permission to use the integrated services\. For details about giving users access to an AWS service that integrates with AWS KMS, consult the documentation for the integrated service\.

```
{
  "Sid": "Allow attachment of persistent resources",
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::111122223333:user/ExampleUser"},
  "Action": [
    "kms:CreateGrant",
    "kms:ListGrants",
    "kms:RevokeGrant"
  ],
  "Resource": "*",
  "Condition": {"Bool": {"kms:GrantIsForAWSResource": true}}
}
```

For example, key users can use these permissions on the KMS key in the following ways\.
+ Use this KMS key with Amazon Elastic Block Store \(Amazon EBS\) and Amazon Elastic Compute Cloud \(Amazon EC2\) to attach an encrypted EBS volume to an EC2 instance\. The key user implicitly gives Amazon EC2 permission to use the KMS key to attach the encrypted volume to the instance\. For more information, see [How Amazon Elastic Block Store \(Amazon EBS\) uses AWS KMS](services-ebs.md)\.
+ Use this KMS key with Amazon Redshift to launch an encrypted cluster\. The key user implicitly gives Amazon Redshift permission to use the KMS key to launch the encrypted cluster and create encrypted snapshots\. For more information, see [How Amazon Redshift uses AWS KMS](services-redshift.md)\.
+ Use this KMS key with other [AWS services integrated with AWS KMS](service-integration.md) that use grants to create, manage, or use encrypted resources with those services\.

The default key policy allows key users to delegate their grant permission to *all* integrated services that use grants\. However, you can create a custom key policy that restricts the permission to specified AWS services\. For more information, see the [kms:ViaService](policy-conditions.md#conditions-kms-via-service) condition key\.