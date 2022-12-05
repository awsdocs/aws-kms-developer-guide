# Grants in AWS KMS<a name="grants"></a>

A *grant* is a policy instrument that allows [AWS principals](https://docs.aws.amazon.com/IAM/latest/UserGuide/intro-structure.html#intro-structure-principal) to use KMS keys in cryptographic operations\. It also can let them view a KMS key \(`DescribeKey`\) and create and manage grants\. When authorizing access to a KMS key, grants are considered along with [key policies](key-policies.md) and [IAM policies](iam-policies.md)\. Grants are often used for temporary permissions because you can create one, use its permissions, and delete it without changing your key policies or IAM policies\.

Grants are commonly used by AWS services that integrate with AWS KMS to encrypt your data at rest\. The service creates a grant on behalf of a user in the account, uses its permissions, and retires the grant as soon as its task is complete\. For details about how AWS services, use grants, see [How AWS services use AWS KMS](service-integration.md) or the *Encryption at rest* topic in the service's user guide or developer guide\.

For code examples that demonstrate how to work with grants in several programming languages, see [Working with grants](programming-grants.md)\.

**Topics**
+ [About grants](#about-grants)
+ [Grant concepts](#grant-concepts)
+ [Best practices for AWS KMS grants](#grant-best-practices)
+ [Creating grants](create-grant-overview.md)
+ [Managing grants](grant-manage.md)

## About grants<a name="about-grants"></a>

Grants are a very flexible and useful access control mechanism\. When you create a grant for a KMS key, the grant allows the grantee principal to call the specified grant operations on the KMS key provided that all conditions specified in the grant are met\. 
+ Each grant allows access to exactly one KMS key\. You can create a grant for a KMS key in a different AWS account\.
+ A grant can allow access to a KMS key, but not deny access\.
+ Each grant has one [grantee principal](#terms-grantee-principal)\. The grantee principal can represent one or more identities in the same AWS account as the KMS key or in a different account\.
+ A grant can only allow [grant operations](#terms-grant-operations)\. The grant operations must be supported by the KMS key in the grant\. If you specify an unsupported operation, the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) request fails with a `ValidationError` exception\.
+ The grantee principal can use the permissions that the grant gives them without specifying the grant, just as they would if the permissions came from a key policy or IAM policy\. However, when you create, retire, or revoke a grant, there might be a brief delay, usually less than five minutes, until the operation achieves [eventual consistency](#terms-eventual-consistency)\. To use the permissions in a grant immediately, [use a grant token](grant-manage.md#using-grant-token)\.
+ An authorized principal can delete the grant \([retire](#terms-retire-grant) or [revoke](#terms-revoke-grant) it\)\. Deleting a grant eliminates all permissions that the grant allowed\. You do not have to figure out which policies to add or remove to undo the grant\. 
+ AWS KMS limits the number of grants on each KMS key\. For details, see [Grants per KMS key: 50,000](resource-limits.md#grants-per-key)\.

Be cautious when creating grants and when giving others permission to create grants\. Permission to create grants has security implications, much like allowing the [kms:PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) permission to set policies\.
+ Users with permission to create grants for a KMS key \(`kms:CreateGrant`\) can use a grant to allow users and roles, including AWS services, to use the KMS key\. The principals can be identities in your own AWS account or identities in a different account or organization\.
+ Grants can allow only a subset of AWS KMS operations\. You can use grants to allow principals to view the KMS key, use it in cryptographic operations, and create and retire grants\. For details, see [Grant operations](#terms-grant-operations)\. You can also use [grant constraints](create-grant-overview.md#grant-constraints) to limit the permissions in a grant for a symmetric encryption key\.
+ Principals can get permission to create grants from a key policy or IAM policy\. Principals who get `kms:CreateGrant` permission from a policy can create grants for any [grant operation](#terms-grant-operations) on the KMS key\. These principals are not required to have the permission that they are granting on the key\. When you allow `kms:CreateGrant` permission in a policy, you can use [policy conditions](grant-manage.md#grant-authorization) to limit this permission\.
+ Principals can also get permission to create grants from a grant\. These principal can only delegate the permissions that they were granted, even if they have other permissions from a policy\. For details, see [Granting CreateGrant permission](create-grant-overview.md#grant-creategrant)\.

For help with concepts related to grants, see [Grant terminology](#grant-concepts)\.

## Grant concepts<a name="grant-concepts"></a>

To use grants effectively, you'll need to understand the terms and concepts that AWS KMS uses\. 

**Grant constraint**  <a name="terms-grant-constraint"></a>
A condition that limits the permissions in the grant\. Currently, AWS KMS supports grant constraints based on the [encryption context](concepts.md#encrypt_context) in the request for a cryptographic operation\. For details, see [Using grant constraints](create-grant-overview.md#grant-constraints)\.

**Grant ID**  <a name="terms-grant-id"></a>
The unique identifier of a grant for a KMS key\. You can use a grant ID, along with a [key identifier](concepts.md#key-id), to identify a grant in a [RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html) or [RevokeGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) request\.

**Grant operations**  <a name="terms-grant-operations"></a>
The AWS KMS operations that you can allow in a grant\. If you specify other operations, the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) request fails with a `ValidationError` exception\. These are also the operations that accept a [grant token](#grant_token)\. For detailed information about these permissions, see the [AWS KMS permissions](kms-api-permissions-reference.md)\.  
These grant operations actually represent permission to use the operation\. Therefore, for the `ReEncrypt` operation, you can specify `ReEncryptFrom`, `ReEncryptTo`, or both `ReEncrypt*`\.  
The grant operations are:  
+ Cryptographic operations
  + [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)
  + [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html)
  + [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)
  + [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html)
  + [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html)
  + [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html)
  + [GenerateMac](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateMac.html)
  + [ReEncryptFrom](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)
  + [ReEncryptTo](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)
  + [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html)
  + [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html)
  + [VerifyMac](https://docs.aws.amazon.com/kms/latest/APIReference/API_VerifyMac.html)
+ Other operations
  + [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html)
  + [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html)
  + [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html)
  + [RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html)
The grant operations that you allow must be supported by the KMS key in the grant\. If you specify an unsupported operation, the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) request fails with a `ValidationError` exception\. For example, grants for symmetric encryption KMS keys cannot allow the [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html), [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html), [https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateMac.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateMac.html) or [https://docs.aws.amazon.com/kms/latest/APIReference/API_VerifyMac.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_VerifyMac.html) operations\. Grants for asymmetric KMS keys cannot allow any operations that generate data keys or data key pairs\.

**Grant token**  <a name="grant_token"></a>
When you create a grant, there might be a brief delay, usually less than five minutes, until the new grant is available throughout AWS KMS, that is, until it achieves [eventual consistency](#terms-eventual-consistency)\. If you try a use a grant before it achieves eventual consistency, you might get an access denied error\. A grant token lets you refer to the grant and use the grant permissions immediately\.   
A *grant token* is a unique, nonsecret, variable\-length, base64\-encoded string that represents a grant\. You can use the grant token to identify the grant in any [grant operation](#terms-grant-operations)\. However, because the token value is a hash digest, it doesn't reveal any details about the grant\.  
A grant token is designed to be used only until the grant achieves eventual consistency\. After that, the [grantee principal](#terms-grantee-principal) can use the permission in the grant without providing a grant token or any other evidence of the grant\. You can use a grant token at any time, but once the grant is eventually consistent, AWS KMS uses the grant to determine permissions, not the grant token\.  
For example, the following command calls the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation\. It uses a grant token to represent the grant that gives the caller \(the grantee principal\) permission to call `GenerateDataKey` on the specified KMS key\.  

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
The identities that get the permissions specified in the grant\. Each grant has one grantee principal, but the grantee principal can represent multiple identities\.   
The grantee principal can be any AWS principal, including an AWS account \(root\), an [IAM user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html), an [IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html), a [federated role or user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers.html), or an assumed role user\. The grantee principal can be in the same account as the KMS key or a different account\. However, the grantee principal cannot be a [service principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services), an [IAM group](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups.html), or an [AWS organization](https://docs.aws.amazon.com/organizations/latest/userguide/)\.

**Retire \(a grant\)**  <a name="terms-retire-grant"></a>
Terminates a grant\. You retire a grant when you finish using the permissions\.  
Revoking and retiring a grant both delete the grant\. But retiring is done by a principal specified in the grant\. Revoking is typically done by a key administrator\. For details, see [Retiring and revoking grants](grant-manage.md#grant-delete)\.

**Retiring principal**  <a name="terms-retiring-principal"></a>
A principal who can [retire a grant](#terms-retire-grant)\. You can specify a retiring principal in a grant, but it is not required\. The retiring principal can be any AWS principal, including AWS accounts, IAM users, IAM roles, federated users, and assumed role users\. The retiring principal can be in the same account as the KMS key or a different account\.  
In addition to retiring principal specified in the grant, a grant can be retired by the AWS account in which the grant was created\. If the grant allows the `RetireGrant` operation, the [grantee principal](#terms-grantee-principal) can retire the grant\. Also, the AWS account or an AWS account that is the retiring principal can delegate the permission to retire a grant to an IAM principal in the same AWS account\. For details, see [Retiring and revoking grants](grant-manage.md#grant-delete)\.

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

## Best practices for AWS KMS grants<a name="grant-best-practices"></a>

AWS KMS recommends the following best practices when creating, using, and managing grants\.
+ Limit the permissions in the grant to those that the grantee principal requires\. Use the principle of [least privileged access](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege)\.
+ Use a specific grantee principal, such as an IAM role, and give the grantee principal permission to use only the API operations that they require\. 
+ Use the encryption context [grant constraints](#terms-grant-constraint) to ensure that callers are using the KMS key for the intended purpose\. For details about how to use the encryption context in a request to secure your data, see [How to Protect the Integrity of Your Encrypted Data by Using AWS Key Management Service and EncryptionContext](http://aws.amazon.com/blogs/security/how-to-protect-the-integrity-of-your-encrypted-data-by-using-aws-key-management-service-and-encryptioncontext/) in the *AWS Security Blog*\.
**Tip**  
Use the [EncryptionContextEqual](create-grant-overview.md#grant-constraints) grant constraint whenever possible\. The [EncryptionContextSubset](create-grant-overview.md#grant-constraints) grant constraint is more difficult to use correctly\. If you need to use it, read the documentation carefully and test the grant constraint to make sure it works as intended\.
+ Delete duplicate grants\. Duplicate grants have the same key ARN, API actions, grantee principal, encryption context, and name\. If you retire or revoke the original grant but leave the duplicates, the leftover duplicate grants constitute unintended escalations of privilege\. To avoid duplicating grants when retrying a `CreateGrant` request, use the [`Name` parameter](create-grant-overview.md#grant-create)\. To detect duplicate grants, use the [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation\. If you accidentally create a duplicate grant, retire or revoke it as soon as possible\. 
**Note**  
Grants for [AWS managed keys](concepts.md#aws-managed-cmk) might look like duplicates but have different grantee principals\.  
The `GranteePrincipal` field in the `ListGrants` response usually contains the grantee principal of the grant\. However, when the grantee principal in the grant is an AWS service, the `GranteePrincipal` field contains the [service principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services), which might represent several different grantee principals\.
+ Remember that grants do not automatically expire\. [Retire or revoke the grant](grant-manage.md#grant-delete) as soon as the permission is no longer needed\. Grants that are not deleted might create a security risk for encrypted resources\.