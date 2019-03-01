# Using Grants<a name="grants"></a>

AWS KMS supports two resource\-based access control mechanisms: [key policies](key-policies.md) and *grants*\. With grants you can programmatically delegate the use of KMS customer master keys \(CMKs\) to other AWS principals\. You can use them to allow access, but not deny it\. Grants are typically used to provide temporary permissions or more granular permissions\.

You can also use key policies to allow other principals to access a CMK, but key policies work best for relatively static permission assignments\. Also, key policies use the standard permissions model for AWS policies in which users either have or do not have permission to perform an action with a resource\. For example, users with the `kms:PutKeyPolicy` permission for a CMK can completely replace the key policy for a CMK with a different key policy of their choice\. To enable more granular permissions management, use grants\.

For code examples that demonstrate how to work with grants, see [Working with Grants](programming-grants.md)\.

**Create a Grant**

To create a grant, call the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) API operation\. Specify a CMK, the grantee principal that the grant allows to use the CMK, and a list of allowed operations\. The `CreateGrant` operation returns a grant ID that you can use to identify the grant in subsequent operations\. To customize the grant, use optional `Constraints` parameters to define [grant constraints](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) 

Grants can be revoked \(canceled\) by any user who has the [kms:RevokeGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) permission on the CMK\. Grants can be retired by any of the following:
+ The AWS account \(root user\) in which the grant was created
+ The retiring principal in the grant, if any
+ The grantee principal, if the grant includes [kms:RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html) permission

**Grant Constraints**

[Grant constraints](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) set conditions on the permissions that the grantee principal can perform\. Grants have two supported constraints, both of which involve the [encryption context](concepts.md#encrypt_context) in a request for a cryptographic operation:
+ `EncryptionContextEquals` specifies that the grant applies only when the encryption context pairs in the request are an exact, case\-sensitive match for the encryption context pairs in the grant constraint\. The pairs can appear in any order, but the keys and values in each pair cannot vary\.
+ `EncryptionContextSubset` specifies that the grant applies only when the encryption context in the request includes the encryption context specified in the grant constraint\. The encryption context in the request must be an exact, case\-sensitive match of the encryption context in the constraint, but it can include additional encryption context pairs\. The pairs can appear in any order, but the keys and values in each included pair cannot vary\.

For example, consider a grant that allows [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operations\. It includes an `EncryptionContextSubset` constraint with the following values\.

```
{"Department":"Finance","Classification":"Public"}
```

In this example, any of the following encryption context values would satisfy the `EncryptionContextSubset` constraint\.
+ `{"Department":"Finance","Classification":"Public"}`
+ `{"Classification":"Public","Department":"Finance"}`
+ `{"Customer":"12345","Department":"Finance","Classification":"Public","Purpose":"Test"}`

However, the following encryption context values would not satisfy the constraint, either because they are incomplete or do not include an exact, case\-sensitive match of the specified pairs\.
+ `{"Department":"Finance"}`
+ `{"department":"finance","classification":"public"}`
+ `{"Classification":"Public","Customer":"12345"}`

**Authorizing CreateGrant in a Key Policy**

When you create a key policy to control access to the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) API operation, you can use one or more policy conditions to limit the permission\. AWS KMS supports all of the following grant\-related condition keys\. For detailed information about these condition keys, see [AWS KMS Condition Keys](policy-conditions.md#conditions-kms)\.
+ [kms:GrantConstraintType](policy-conditions.md#conditions-kms-grant-constraint-type)
+ [kms:GrantIsForAWSResource](policy-conditions.md#conditions-kms-grant-is-for-aws-resource)
+ [kms:GrantOperations](policy-conditions.md#conditions-kms-grant-operations)
+ [kms:GranteePrincipal](policy-conditions.md#conditions-kms-grantee-principal)
+ [kms:RetiringPrincipal](policy-conditions.md#conditions-kms-retiring-principal)

**Granting CreateGrant Permission**

When a grant includes permission to call the `CreateGrant` operation, the grant only allows the grantee principal to create grants that are equally restrictive or more restrictive\. 

For example, consider a grant that allows the grantee principal to call the `GenerateDataKey`, `Decrypt`, and `CreateGrant` operations\. The grantee principal can use this permission to create a grant that includes any subset of the operations specified in the parent grant, such as `GenerateDataKey` and `Decrypt`\. But it cannot include other operations, such as `ScheduleKeyDeletion` or `ReEncrypt`\.

Also, the [grant constraints](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) in child grants must be equally restrictive or more restrictive than those in the parent grant\. For example, the child grant can add pairs to an `EncryptionContextSubset` constraint in the parent grant, but it cannot remove them\. The child grant can change an `EncryptionContextSubset` constraint to an `EncryptionContextEquals` constraint, but not the reverse\.