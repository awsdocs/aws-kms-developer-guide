# Using grants<a name="grants"></a>

A *grant* is a policy instrument that allows [AWS principals](https://docs.aws.amazon.com/IAM/latest/UserGuide/intro-structure.html#intro-structure-principal) to use AWS KMS [customer master keys](concepts.md#master_keys) \(CMKs\) in cryptographic operations\. It also can let them view a CMK \(`DescribeKey`\) and create and manage grants\. When authorizing access to a CMK, grants are considered along with [key policies](key-policies.md) and [IAM policies](iam-policies.md)\. Grants are often used for temporary permissions because you can create one, use its permissions, and delete it without changing your key policies or IAM policies\.

Along with [key policies](key-policies.md), which are required, and [IAM policies](iam-policies.md), which are optional, grants provide a powerful and flexible component of your access control strategy\. Typically, you use key policies to establish long\-term, static permissions to the CMK\. You use IAM policies to control access to operations that don't involve a particular CMK, such as [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html), and to describe permissions that apply to multiple CMKs or include permissions for the resources of multiple AWS services\. Then, to provide temporary or limited access to a CMK, use a grant\.

Grants are commonly used by AWS services that integrate with AWS KMS to encrypt your data at rest\. The service creates a grant on behalf of a user in the account, uses its permissions, and retires the grant as soon as its task is complete\. For details about how AWS services, use grants, see [How AWS services use AWS KMS](service-integration.md) or the *Encryption at rest* topic in the service's user guide or developer guide\.

For code examples that demonstrate how to work with grants in several programming languages, see [Working with grants](programming-grants.md)\.

**Topics**
+ [About grants](#about-grants)
+ [Grants for symmetric and asymmetric CMKs](#grants-asymm)
+ [Grant terminology](#grant-concepts)
+ [Creating grants](create-grant-overview.md)
+ [Managing grants](grant-manage.md)

## About grants<a name="about-grants"></a>

Grants are a very flexible and useful access control mechanism\. When you attach a grant to a [customer master key](concepts.md#master_keys) \(CMK\), the *grant* allows a principal to call particular operations on a CMK when the conditions specified in the grant are met\. 
+ Each grant controls access to just one CMK\. The CMK can be in the same or a different AWS account\.
+ You can use a grant to allow access, but not to deny it\. Grants can only allow access to [grant operations](#terms-grant-operations)\.
+ Principals who get permissions from a grant can use those permissions without specifying the grant, just as they would if the permissions came from a key policy or IAM policy\. However, when you create, retire, or revoke a grant, there might be a brief delay, usually less than five minutes, until the operation achieves [eventual consistency](#terms-eventual-consistency)\. To use the permissions in a grant immediately, [use a grant token](grant-manage.md#using-grant-token)\.
+ You can use grants to allow principals in a different AWS account to use a CMK\. 
+ If a principal has the required permissions, they can delete the grant \([retire](#terms-retire-grant) or [revoke](#terms-revoke-grant) it\)\. These actions eliminate all permissions that the grant allowed\. You do not have to figure out which policies to add or adjust to undo the grant\. 
+ When you create, retire, or revoke a grant, there might be a brief interval, usually less than 5 minutes, until the change is available throughout AWS KMS\. For details, see [Eventual consistency for grants](#terms-eventual-consistency)\.

Be cautious when creating grants and when giving others permission to create grants\. Permission to create grants has security implications, much like allowing the [kms:PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) permission to set policies\.
+ Users with permission to create grants for a CMK \(`kms:CreateGrant`\) can use a grant to allow users and roles, including AWS services, to use the CMK\. The principals can be identities in your own AWS account or identities in a different account or organization\.
+ Grants can allow only a subset of AWS KMS operations\. You can use grants to allow principals to view the CMK, use it in cryptographic operations, and create and retire grants\. For details, see [Grant operations](#terms-grant-operations)\. You can also use [grant constraints](create-grant-overview.md#grant-constraints) to limit the permissions in a grant\.
+ Principals can get permission to create grants from a key policy or IAM policy\. These principals can create grants for any [grant operation](#terms-grant-operations) on the CMK, even if they don't have the permission\. When you allow `kms:CreateGrant` permission in a policy, you can use [policy conditions](grant-manage.md#grant-authorization) to limit this permission\.
+ Principals can also get permission to create grants from a grant\. These principal can only delegate the permissions that they were granted, even if they have other permissions from a policy\. For details, see [Granting CreateGrant permission](create-grant-overview.md#grant-creategrant)\.

For help with concepts related to grants, see [Grant terminology](#grant-concepts)\.

## Grants for symmetric and asymmetric CMKs<a name="grants-asymm"></a>

You can create a grant that controls access to a symmetric CMK or an asymmetric CMK\. However, you cannot create a grant that allows a principal to perform an operation that is not supported by the CMK\. If you try, AWS KMS returns a `ValidationError` exception\.

**Symmetric CMKs**  
Grants for symmetric CMKs cannot allow the [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html), [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html), or [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) operations\. \(There are limited exceptions to this rule for legacy operations, but you should not create a grant for an operation that AWS KMS does not support\.\)

**Asymmetric CMKs**  
Grants for asymmetric CMKs cannot allow operations that generate data keys or data key pairs\. They also cannot allow operations related to [automatic key rotation](rotate-keys.md), [imported key material](importing-keys.md), or CMKs in [custom key stores](custom-key-store-overview.md)\.  
Grants for CMKs with a key usage of `SIGN_VERIFY` cannot allow encryption operations\. Grants for CMKs with a key usage of `ENCRYPT_DECRYPT` cannot allow the `Sign` or `Verify` operations\.

## Grant terminology<a name="grant-concepts"></a>

To use grants effectively, you'll need to understand the terms and concepts that AWS KMS uses\. 

**Grant constraint**  <a name="terms-grant-constraint"></a>
A condition that limits the permissions in the grant\. Currently, AWS KMS supports grant constraints based on the [encryption context](concepts.md#encrypt_context) in the request for a cryptographic operation\. For details, see [Using grant constraints](create-grant-overview.md#grant-constraints)\.

**Grant ID**  <a name="terms-grant-id"></a>
The unique identifier of a grant for a CMK\. You can use a grant, along with a [key identifier](concepts.md#key-id), to identify a grant in a [RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html) or [RevokeGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) request\.

**Grant operations**  <a name="terms-grant-operations"></a>
The AWS KMS operations that you can allow in a grant\. These are also the operations that accept a [grant token](#grant_token)\. For detailed information about these permissions, see the [AWS KMS permissions](kms-api-permissions-reference.md)\.  
These operations actually represent permission to use the operation\. Therefore, for the `ReEncrypt` operation, you can specify `ReEncryptFrom`, `ReEncryptTo`, or both `ReEncrypt*`\.  
The grant operations are:  
+ Cryptographic operations
  + [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)
  + [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html)
  + [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)
  + [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html)
  + [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html)
  + [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html)
  + [ReEncryptFrom](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)
  + [ReEncryptTo](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)
  + [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html)
  + [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html)
+ Other operations
  + [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html)
  + [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html)
  + [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html)
  + [RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html)
You cannot create a grant for an operation that is not supported by the CMK\. If you try, AWS KMS returns a `ValidationError` exception\.  
+ Grants for [symmetric CMKs](symm-asymm-concepts.md#symmetric-cmks) cannot allow the [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html), [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html), or [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) operations\. \(There are limited exceptions to this rule for legacy operations, but you should not create a grant for an operation that AWS KMS does not support\.\)
+ Grants for [asymmetric CMKs](symm-asymm-concepts.md#asymmetric-cmks) cannot allow the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html), or [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html) operations\.
+ Grants for CMKs with a [key usage](concepts.md#key-usage) of `ENCRYPT_DECRYPT` cannot allow the [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html) or [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) operations\.
+ Grants for CMKs with a key usage of `SIGN_VERIFY` cannot allow the [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), or [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) operations\.

**Grant token**  <a name="grant_token"></a>
When you create a grant, there might be a brief delay, usually less than five minutes, until the new grant is available throughout AWS KMS, that is, until it achieves [eventual consistency](#terms-eventual-consistency)\. If you try a use a grant before it achieves eventual consistency, you might get an access denied error\. A grant token lets you refer to the grant and use the grant permissions immediately\.   
A *grant token* is unique, non\-secret, variable\-length, base64\-encoded string that represents a grant\. You can use the grant token to identify the grant in any [grant operation](#terms-grant-operations)\. However, because the token value is a hash digest, it doesn't reveal any details about the grant\.  
A grant token is designed to be used only until the grant achieves eventual consistency\. After that, the [grantee principal](#terms-grantee-principal) can use the permission in the grant without providing a grant token or any other evidence of the grant\. You can use a grant token at any time, but once the grant is eventually consistent, AWS KMS uses the grant to determine permissions, not the grant token\.  
For example, the following command calls the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation\. It uses a grant token to represent the grant that gives the caller \(the grantee principal\) permission to call `GenerateDataKey` on the specified CMK\.  

```
$ aws kms generate-data-key \
        --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
        --key-spec AES_256 \
        --grant-token $token
```
You can also use a grant token to identify a grant in operations that manage grants\. For example, the [retiring principal](#terms-retiring-principal) can use a grant token in a call to the [RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html) operation\.   

```
$ aws kms retire-grant \
        --grant-token $token
```
`CreateGrant` is the only operation that returns a grant token\. You cannot get a grant token from any other AWS KMS operation or from the [CloudTrail log event](ct-creategrant.md) for the CreateGrant operation\. The [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) and [ListRetirableGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListRetirableGrants.html) operations return the [grant ID](#terms-grant-id), but not a grant token\.  
For details, see [Using a grant token](grant-manage.md#using-grant-token)\.

**Grantee principal**  <a name="terms-grantee-principal"></a>
The identity that gets the permissions specified in the grant\. A grant must have at least one grantee principal\. The grantee principal can be any AWS principal, including an AWS account \(root\), an [IAM user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html), an [IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html), a [federated role or user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers.html), or an assumed role user\. The grantee principal can be in the same account as the CMK or a different account\. However, the grantee principal cannot be a [service principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services), an [IAM group](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups.html), or an [AWS organization](https://docs.aws.amazon.com/organizations/latest/userguide/)\.

**Retire \(a grant\)**  <a name="terms-retire-grant"></a>
Terminates a grant\. You retire a grant when you are done using its permissions\.  
Revoking and retiring a grant both delete the grant\. But retiring is done by a principal specified in the grant\. Revoking is typically done by a key administrator\. For details, see [Retiring and revoking grants](grant-manage.md#grant-delete)\.

**Retiring principal**  <a name="terms-retiring-principal"></a>
A principal who can [retire a grant](#terms-retire-grant)\. You can specify a retiring principal in a grant, but it is not required\. The retiring principal can be any AWS principal, including AWS accounts \(root\), IAM users, IAM roles, federated users, and assumed role users\. The retiring principal can be in the same account as the CMK or a different account\.  
In addition to retiring principal specified in the grant, a grant can be retired by the AWS account \(root user\) in which the grant was created\. If the grant allows the `RetireGrant` operation, the [grantee principal](#terms-grantee-principal) can retire the grant\. Also, the AWS account \(root user\) or an AWS account that is the retiring principal can delegate the permission to retire a grant to an IAM principal in the same AWS account\. For details, see [Retiring and revoking grants](grant-manage.md#grant-delete)\.

**Revoke \(a grant\)**  <a name="terms-revoke-grant"></a>
Terminates a grant\. You revoke a grant to actively deny the permissions that the grant allows\.   
Revoking and retiring a grant both delete the grant\. But retiring is done by a principal specified in the grant\. Revoking is typically done by a key administrator\. For details, see [Retiring and revoking grants](grant-manage.md#grant-delete)\.

**Eventual consistency \(for grants\)**  <a name="terms-eventual-consistency"></a>
When you create, retire, or revoke a grant, there might be a brief delay, usually less than five minutes, before the change is available throughout AWS KMS\. When this interval is complete, we consider the operation to have achieved [eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency)\.   
You might become aware of this brief delay if you get unexpected errors\. For example, If you try to manage a new grant or use the permissions in a new grant before the grant is known throughout AWS KMS, you might get an access denied error\. If you retire or revoke a grant, the grantee principal might still be able to use its permissions for a brief period until the grant is fully deleted\. The typical strategy is to retry the request, and some AWS SDKs include automatic backoff and retry logic\.  
AWS KMS has features to mitigate this brief delay\.   
+ To use the permissions in a new grant immediately, use a [grant token](grant-manage.md#using-grant-token)\. You can use a grant token to refer to a grant in any [grant operation](#terms-grant-operations)\. For instructions, see [Using a grant token](grant-manage.md#using-grant-token)\. 
+ The [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation has a `Name` parameter that prevents retry operations from creating duplicate grants\.
Grant tokens supersede the validity of the grant until all endpoints in the service have been updated with the new grant state\. In most cases, eventual consistency will be achieved within five minutes\.