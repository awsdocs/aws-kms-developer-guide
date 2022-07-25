# Managing grants<a name="grant-manage"></a>

Principals with the required permissions can view, use and delete \(retire or revoke\) grants\. To refine permissions for creating and managing grants, AWS KMS supports several policy conditions that you can use in key policies and IAM policies\.

**Topics**
+ [Controlling access to grants](#grant-authorization)
+ [Viewing grants](#grant-view)
+ [Using a grant token](#using-grant-token)
+ [Retiring and revoking grants](#grant-delete)

## Controlling access to grants<a name="grant-authorization"></a>

You can control access to the operations that create and manage grants in key policies, IAM policies, and in grants\. Principals who get `CreateGrant` permission from a grant have [more limited grant permissions](create-grant-overview.md#grant-creategrant)\. 


| API operation | Key policy or IAM policy | Grant | 
| --- | --- | --- | 
| CreateGrant | ✓ | ✓ | 
| ListGrants | ✓ | \- | 
| ListRetirableGrants | ✓ | \- | 
| Retire Grants | \(Limited\. See [Retiring and revoking grants](#grant-delete)\) | ✓ | 
| RevokeGrant | ✓ | \- | 

When you use a key policy or IAM policy to control access to operations that create and manage grants, you can use one or more of the following policy conditions to limit the permission\. AWS KMS supports all of the following grant\-related condition keys\. For detailed information and examples, see [AWS KMS condition keys](policy-conditions.md#conditions-kms)\.

[kms:GrantConstraintType](policy-conditions.md#conditions-kms-grant-constraint-type)  
Allows principals to create a grant only when the grant includes the specified [grant constraint](create-grant-overview.md#grant-constraints)\.

[kms:GrantIsForAWSResource](policy-conditions.md#conditions-kms-grant-is-for-aws-resource)  
Allows principals to call `CreateGrant`, `ListGrants`, or `RevokeGrant` only when [an AWS service that is integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration) sends the request on the principal's behalf\.

[kms:GrantOperations](policy-conditions.md#conditions-kms-grant-operations)  
Allows principals to create a grant, but limits the grant to the specified operations\.

[kms:GranteePrincipal](policy-conditions.md#conditions-kms-grantee-principal)  
Allows principals to create a grant only for the specified [grantee principal](grants.md#terms-grantee-principal)\.

[kms:RetiringPrincipal](policy-conditions.md#conditions-kms-retiring-principal)  
Allows principals to create a grant only when the grant specifies a particular [retiring principal](grants.md#terms-retiring-principal)\.

## Viewing grants<a name="grant-view"></a>

To view the grant, use the [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation\. You must specify the KMS key to which the grants apply\. You can also filter the grant list by grant ID or grantee principal\. For more examples, see [Viewing a grant](programming-grants.md#list-grants)\.

To view all grants in the AWS account and Region with a particular [retiring principal](grants.md#terms-retiring-principal), use [ListRetirableGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListRetirableGrants.html)\. The responses include details about each grant\.

**Note**  
The `GranteePrincipal` field in the `ListGrants` response usually contains the grantee principal of the grant\. However, when the grantee principal in the grant is an AWS service, the `GranteePrincipal` field contains the [service principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services), which might represent several different grantee principals\.

For example, the following command lists all of the grants for a KMS key\.

```
$  aws kms list-grants --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
{
    "Grants": [
        {
            "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "CreationDate": 1572216195.0,
            "GrantId": "abcde1237f76e4ba7987489ac329fbfba6ad343d6f7075dbd1ef191f0120514a",
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

## Using a grant token<a name="using-grant-token"></a>

When you create a grant, the grant might not be effective immediately\. There's likely to be a brief interval, less than five minutes, until the grant achieves [eventual consistency](grants.md#terms-eventual-consistency), that is, before the new grant is available throughout AWS KMS\. Once the grant has achieved eventual consistency, the grantee principal can use the permissions in the grant without specifying the grant token or any evidence of the grant\. However, if grant that is so new that it is not yet known to all of AWS KMS, the request might fail with an `AccessDeniedException` error\.

To use the permissions in a new grant immediately, use the [grant token](grants.md#grant_token) for the grant\. Save the grant token that the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation returns\. Then submit the grant token in the request for the AWS KMS operation\. You can submit a grant token to any AWS KMS [grant operation](grants.md#terms-grant-operations) and you can submit multiple grant tokens in the same request\.



The following example uses the `CreateGrant` operation to create a grant that allows the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operations\. It saves the grant token that `CreateGrant` returns in the `token` variable\. Then, in a call to the `GenerateDataKey` operation, it uses the grant token in the `token` variable\.

```
# Create a grant; save the grant token 
$ token=$(aws kms create-grant \
    --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
    --grantee-principal arn:aws:iam::111122223333:user/appUser \
    --retiring-principal arn:aws:iam::111122223333:user/acctAdmin \
    --operations GenerateDataKey Decrypt \
    --query GrantToken \
    --output text)

# Use the grant token in a request
$ aws kms generate-data-key \
    --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
    –-key-spec AES_256 \
    --grant-tokens $token
```

Principals with permission can also use a grant token to retire a new grant even before the grant achieves eventual consistency\. \(The `RevokeGrant` operation doesn't accept a grant token\.\) For details, see [Retiring and revoking grants](#grant-delete)\.

```
# Retire the grant
$ aws kms retire-grant --grant-token $token
```

## Retiring and revoking grants<a name="grant-delete"></a>

To delete a grant, retire or revoke it\.

The [RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html) and [RevokeGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) operations are very similar to each other\. Both operations delete a grant, which eliminates the permissions the grant allows\. The primary difference between these operations is how they are authorized\.

RevokeGrant  
Like most AWS KMS operations, access to the `RevokeGrant` operation is controlled through [key policies](key-policies.md) and [IAM policies](iam-policies.md)\. The [RevokeGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) API can be called by any principal with `kms:RevokeGrant` permission\. This permission is included in the standard permissions given to key administrators\. Typically, administrators revoke a grant to deny permissions the grant allows\.

RetireGrant  
The grant determines who can retire it\. This design allows you to control the lifecycle of a grant without changing key policies or IAM policies\. Typically, you retire a grant when you are done using its permissions\.  
A grant can be retired by an optional [retiring principal](grants.md#terms-retiring-principal) specified in the grant\. The [grantee principal](grants.md#terms-grantee-principal) can also retire the grant, but only if they are also a retiring principal or the grant includes the `RetireGrant` operation\. As a backup, the AWS account in which the grant was created can retire the grant\.  
There is a `kms:RetireGrant` permission that can be used in IAM policies, but it has limited utility\. Principals specified in the grant can retire a grant without the `kms:RetireGrant` permission\. The `kms:RetireGrant` permission alone does not allow principals to retire a grant\. The `kms:RetireGrant` permission is not effective in a key policy\.  
+ To deny permission to retire a grant, you can use a `Deny` action with the `kms:RetireGrant` permission\.
+ The AWS account that owns the KMS key can delegate the `kms:RetireGrant` permission to an IAM user in the account\. 
+ If the retiring principal is a different AWS account, administrators in the other account can use `kms:RetireGrant` to delegate permission to retire the grant to an IAM user in that account\.

When you create, retire, or revoke a grant, there might be a brief delay, usually less than five minutes, until the operation achieves [eventual consistency](grants.md#terms-eventual-consistency)\. If you need to delete a new grant immediately, before it is available throughout AWS KMS, [use a grant token](#using-grant-token) to retire the grant\. You cannot use a grant token to revoke a grant\.