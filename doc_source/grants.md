# Using grants<a name="grants"></a>

AWS KMS supports two resource\-based access control mechanisms: [key policies](key-policies.md) and *grants*\. With grants you can programmatically delegate the use of KMS customer master keys \(CMKs\) to other [AWS principals](https://docs.aws.amazon.com/IAM/latest/UserGuide/intro-structure.html#intro-structure-principal)\. You can use them to allow access, but not deny it\. Because grants can be very specific, and are easy to create and revoke, they are often used to provide temporary permissions or more granular permissions\.

You can also use key policies to allow other AWS principals to access a CMK, but key policies work best for relatively static permission assignments\. Also, key policies use the standard permissions model for AWS policies in which users either have or do not have permission to perform an action with a resource\. For example, users with the `kms:PutKeyPolicy` permission for a CMK can completely replace the key policy for a CMK with a different key policy of their choice\. To enable more granular permissions management, use grants\.

For code examples that demonstrate how to work with grants, see [Working with grants](programming-grants.md)\.

## Create a grant<a name="grant-create"></a>

To create a grant, call the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation\. Specify a CMK, the grantee principal that the grant allows to use the CMK, and a list of allowed operations\. The `CreateGrant` operation returns a grant ID that you can use to identify the grant in subsequent operations\. To customize the grant, use optional `Constraints` parameters to define [grant constraints](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html)\.

For example, the following `CreateGrant` command creates a grant that allows `exampleUser` to call the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation on the specified [symmetric CMK](symm-asymm-concepts.md#symmetric-cmks)\. The grant uses the `RetiringPrincipal` parameter to designate a principal that can retire the grant\. It also includes a grant constraint that allows the permission only when the [encryption context](concepts.md#encrypt_context) in the request includes `"Department": "IT"`\.

```
$ aws kms create-grant \
    --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
    --grantee-principal arn:aws:iam::111122223333:user/exampleUser \
    --operations Decrypt \
    --retiring-principal arn:aws:iam::111122223333:role/adminRole \
    --constraints EncryptionContextSubset={Department=IT}
```

To view the grant, use the [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation\. 

**Note**  
The `GranteePrincipal` field in the `ListGrants` response usually contains the grantee principal of the grant\. However, when the grantee principal in the grant is an AWS service, the `GranteePrincipal` field contains the [service principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services), which might represent several different grantee principals\.

```
$ aws kms list-grants --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
{
    "Grants": [
        {
            "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "CreationDate": 1572216195.0,
            "GrantId": "abcde1237f76e4ba7987489ac329fbfba6ad343d6f7075dbd1ef191f0120514",
            "Constraints": {
                "EncryptionContextSubset": {
                    "Department": "IT"
                }
            },
            "RetiringPrincipal": "arn:aws:iam::111122223333:role/adminRole",
            "Name": "",
            "IssuingAccount": "arn:aws:iam::111122223333:root",
            "GranteePrincipal": "arn:aws:iam::111122223333:user/exampleUser",
            "Operations": [
                "Decrypt"
            ]
        }
    ]
}
```

Grants can be revoked \(deleted\) by any user who has the [kms:RevokeGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) permission on the CMK\. 

Grants can be retired by any of the following principals:
+ The AWS account \(root user\) in which the grant was created
+ The retiring principal in the grant, if any
+ The grantee principal, if the grant allows the [RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html) operation\.

## Grants for symmetric and asymmetric CMKs<a name="grants-asymm"></a>

You can create a grant that controls access to a symmetric CMK or an asymmetric CMK\. However, you cannot create a grant that allows a principal to perform an operation that is not supported by the CMK\. If you try, AWS KMS returns a `ValidationError` exception\.

**Symmetric CMKs**  
Grants for symmetric CMKs cannot allow the [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html), [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html), or [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) operations\. \(There are limited exceptions to this rule for legacy operations, but you should not create a grant for an operation that AWS KMS does not support\.\)

**Asymmetric CMKs**  
Grants for asymmetric CMKs cannot allow operations that generate data keys or data key pairs\. They also cannot allow operations related to [automatic key rotation](rotate-keys.md), [imported key material](importing-keys.md), or CMKs in [custom key stores](custom-key-store-overview.md)\.  
Grants for CMKs with a key usage of `SIGN_VERIFY` cannot allow encryption operations\. Grants for CMKs with a key usage of `ENCRYPT_DECRYPT` cannot allow the `Sign` or `Verify` operations\.

## Grant constraints<a name="grant-constraints"></a>

[Grant constraints](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) set conditions on the permissions that the grantee principal can perform\. AWS KMS supports two supported constraints, both of which involve the [encryption context](concepts.md#encrypt_context) in a request for a cryptographic operation\. 

**Note**  
You cannot use encryption context grant constraints in a grant for an asymmetric CMK\. The asymmetric encryption algorithms that AWS KMS uses do not support an encryption context\. 
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

## Authorizing CreateGrant in a key policy<a name="grant-authorization"></a>

When you create a key policy to control access to the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation, you can use one or more policy conditions to limit the permission\. AWS KMS supports all of the following grant\-related condition keys\. For detailed information about these condition keys, see [AWS KMS condition keys](policy-conditions.md#conditions-kms)\.
+ [kms:GrantConstraintType](policy-conditions.md#conditions-kms-grant-constraint-type)
+ [kms:GrantIsForAWSResource](policy-conditions.md#conditions-kms-grant-is-for-aws-resource)
+ [kms:GrantOperations](policy-conditions.md#conditions-kms-grant-operations)
+ [kms:GranteePrincipal](policy-conditions.md#conditions-kms-grantee-principal)
+ [kms:RetiringPrincipal](policy-conditions.md#conditions-kms-retiring-principal)

## Granting CreateGrant permission<a name="grant-creategrant"></a>

When a grant includes permission to call the `CreateGrant` operation, the grant only allows the grantee principal to create grants that are equally restrictive or more restrictive\. 

For example, consider a grant that allows the grantee principal to call the `GenerateDataKey`, `Decrypt`, and `CreateGrant` operations\. The grantee principal can use this permission to create a grant that includes any subset of the operations specified in the parent grant, such as `GenerateDataKey` and `Decrypt`\. But it cannot include other operations, such as `ScheduleKeyDeletion` or `ReEncrypt`\.

Also, the [grant constraints](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) in child grants must be equally restrictive or more restrictive than those in the parent grant\. For example, the child grant can add pairs to an `EncryptionContextSubset` constraint in the parent grant, but it cannot remove them\. The child grant can change an `EncryptionContextSubset` constraint to an `EncryptionContextEquals` constraint, but not the reverse\.