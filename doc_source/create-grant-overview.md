# Creating grants<a name="create-grant-overview"></a>

Before creating a grant, learn about the options for customizing your grant\. You can use *grant constraints* to limit the permissions in the grant\. Also, learn about granting `CreateGrant` permission\. Principals who get permission to create grants from a grant are limited in the grants that they can create\.

**Topics**
+ [Creating a grant](#grant-create)
+ [Using grant constraints](#grant-constraints)
+ [Granting CreateGrant permission](#grant-creategrant)

## Creating a grant<a name="grant-create"></a>

To create a grant, call the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation\. Specify a KMS key, a [grantee principal](grants.md#terms-grantee-principal), and a list of allowed [grant operations](grants.md#terms-grant-operations)\. You can also designate an optional [retiring principal](grants.md#terms-retiring-principal)\. To customize the grant, use optional `Constraints` parameters to define [grant constraints](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html)\.

When you create, retire, or revoke a grant, there might be a brief delay, usually less than five minutes, until the operation achieves [eventual consistency](grants.md#terms-eventual-consistency)\.

For example, the following `CreateGrant` command creates a grant that allows `exampleUser` to call the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation on the specified [symmetric KMS key](concepts.md#symmetric-cmks)\. The grant uses the `RetiringPrincipal` parameter to designate a principal that can retire the grant\. It also includes a grant constraint that allows the permission only when the [encryption context](concepts.md#encrypt_context) in the request includes `"Department": "IT"`\.

```
$  aws kms create-grant \
    --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
    --grantee-principal arn:aws:iam::111122223333:user/exampleUser \
    --operations Decrypt \
    --retiring-principal arn:aws:iam::111122223333:role/adminRole \
    --constraints EncryptionContextSubset={Department=IT}
```

If your code retries the `CreateGrant` operation, or uses an [AWS SDK that automatically retries requests](https://docs.aws.amazon.com/general/latest/gr/api-retries.html), use the optional [Name](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html#KMS-CreateGrant-request-Name) parameter to prevent the creation of duplicate grants\. If AWS KMS gets a `CreateGrant` request for a grant with the same properties as an existing grant, including the name, it recognizes the request as a retry, and does not create a new grant\. You cannot use the `Name` value to identify the grant in any AWS KMS operations\.

```
$ aws kms create-grant \
    --name IT-1234abcd-exampleUser-decrypt \
    --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
    --grantee-principal arn:aws:iam::111122223333:user/exampleUser \
    --operations Decrypt \
    --retiring-principal arn:aws:iam::111122223333:role/adminRole \
    --constraints EncryptionContextSubset={Department=IT}
```

For code examples that demonstrate how to work with grants in several programming languages, see [Working with grants](programming-grants.md)\.

## Using grant constraints<a name="grant-constraints"></a>

[Grant constraints](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) set conditions on the permissions that the grant gives to the grantee principal\. Grant constraints take the place of [condition keys](policy-conditions.md) in a [key policy](key-policies.md) or [IAM policy](iam-policies.md)\. Each grant constraint value can include up to 8 encryption context pairs\. The encryption context value in each grant constraint cannot exceed 384 characters\.

AWS KMS supports two grant constraints, `EncryptionContextEquals` and `EncryptionContextSubset`, both of which establish requirements for the [encryption context](concepts.md#encrypt_context) in a request for a cryptographic operation\. 

The encryption context grant constraints are designed to be used with [grant operations](grants.md#terms-grant-operations) that have an encryption context parameter\. 
+ You cannot use an encryption context constraint in a grant for an asymmetric KMS key\. Cryptographic operations with asymmetric KMS keys don't support an encryption context\.
+ The encryption context constraint is ignored for `DescribeKey` and `RetireGrant` operations\. `DescribeKey` and `RetireGrant` don't have an encryption context parameter, but you can include these operations in a grant that has an encryption context constraint\.
+ You can use an encryption context constraint in a grant for the `CreateGrant` operation\. The encryption context constraint requires that any grants created with the `CreateGrant` permission have an equally strict or stricter encryption context constraint\.

AWS KMS supports the following encryption context grant constraints\.

**EncryptionContextEquals**  
Use `EncryptionContextEquals` to specify the exact encryption context for permitted requests\.   
`EncryptionContextEquals` requires that the encryption context pairs in the request are an exact, case\-sensitive match for the encryption context pairs in the grant constraint\. The pairs can appear in any order, but the keys and values in each pair cannot vary\.   
For example, if the `EncryptionContextEquals` grant constraint requires the `"Department": "IT"` encryption context pair, the grant allows requests of the specified type only when the encryption context in the request is exactly `"Department": "IT"`\.

**EncryptionContextSubset**  
Use `EncryptionContextSubset` to require that requests include particular encryption context pairs\.  
`EncryptionContextSubset` requires that the request include all encryption context pairs in the grant constraint \(an exact, case\-sensitive match\), but the request can also have additional encryption context pairs\. The pairs can appear in any order, but the keys and values in each pair cannot vary\.   
For example, if the `EncryptionContextSubset` grant constraint requires the `Department=IT` encryption context pair, the grant allows requests of the specified type when the encryption context in the request is `"Department": "IT"`, or includes `"Department": "IT"` along with other encryption context pairs, such as `"Department": "IT","Purpose": "Test"`\.

To specify an encryption context constraint in a grant for a symmetric KMS key, use the `Constraints` parameter in the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation\. The grant that this command creates gives the `exampleUser` permission to call the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. But that permission is effective only when the encryption context in the `Decrypt` request is a `"Department": "IT"` encryption context pair\.

```
$ aws kms create-grant \
    --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
    --grantee-principal arn:aws:iam::111122223333:user/exampleUser \
    --operations Decrypt \
    --retiring-principal arn:aws:iam::111122223333:role/adminRole \
    --constraints EncryptionContextEquals={Department=IT}
```

The resulting grant looks like the following one\. Notice that the permission granted to `exampleUser` is effective only when the `Decrypt` request uses the same encryption context pair specified in the grant constraint\. To find the grants on a KMS key, use the [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation\.

```
$ aws kms list-grants --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
{
    "Grants": [
        {
            "Name": "",
            "IssuingAccount": "arn:aws:iam::111122223333:root",
            "GrantId": "8c94d1f12f5e69f440bae30eaec9570bb1fb7358824f9ddfa1aa5a0dab1a59b2",
            "Operations": [
                "Decrypt"
            ],
            "GranteePrincipal": "arn:aws:iam::111122223333:user/exampleUser",
            "Constraints": {
                "EncryptionContextEquals": {
                    "Department": "IT"
                }
            },
            "CreationDate": 1568565290.0,
            "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "RetiringPrincipal": "arn:aws:iam::111122223333:role/adminRole"
        }
    ]
}
```

To satisfy the `EncryptionContextEquals` grant constraint, the encryption context in the request for the `Decrypt` operation must be a `"Department": "IT"` pair\. A request like the following from the grantee principal would satisfy the `EncryptionContextEquals` grant constraint\.

```
$ aws kms decrypt \
    --key-id arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab\
    --ciphertext-blob fileb://encrypted_msg \
    --encryption-context Department=IT
```

When the grant constraint is `EncryptionContextSubset`, the encryption context pairs in the request must include the encryption context pairs in the grant constraint, but the request can also include other encryption context pairs\. The following grant constraint requires that one of encryption context pairs in the request is `"Deparment": "IT"`\.

```
"Constraints": {
   "EncryptionContextSubset": {
       "Department": "IT"
   }
}
```

The following request from the grantee principal would satisfy both of the `EncryptionContextEqual` and `EncryptionContextSubset` grant constraints in this example\.

```
$ aws kms decrypt \
    --key-id arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab \
    --ciphertext-blob fileb://encrypted_msg \
    --encryption-context Department=IT
```

However, a request like the following from the grantee principal would satisfy the `EncryptionContextSubset` grant constraint, but it would fail the `EncryptionContextEquals` grant constraint\.

```
$ aws kms decrypt \
    --key-id arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab \
    --ciphertext-blob fileb://encrypted_msg \
    --encryption-context Department=IT,Purpose=Test
```

AWS services often use encryption context constraints in the grants that give them permission to use KMS keys in your AWS account\. For example, Amazon DynamoDB uses a grant like the following one to get permission to use the [AWS managed key](concepts.md#aws-managed-cmk) for DynamoDB in your account\. The `EncryptionContextSubset` grant constraint in this grant makes the permissions in the grant effective only when the encryption context in the request includes `"subscriberID": "111122223333"` and `"tableName": "Services"` pairs\. This grant constraint means that the grant allows DynamoDB to use the specified KMS key only for a particular table in your AWS account\.

To get this output, run the [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation on the AWS managed key for DynamoDB in your account\.

```
$ aws kms list-grants --key-id 0987dcba-09fe-87dc-65ba-ab0987654321

{
    "Grants": [
        {
            "Operations": [
                "Decrypt",
                "Encrypt",
                "GenerateDataKey",
                "ReEncryptFrom",
                "ReEncryptTo",
                "RetireGrant",
                "DescribeKey"
            ],
            "IssuingAccount": "arn:aws:iam::111122223333:root",
            "Constraints": {
                "EncryptionContextSubset": {
                    "aws:dynamodb:tableName": "Services",
                    "aws:dynamodb:subscriberId": "111122223333"
                }
            },
            "CreationDate": 1518567315.0,
            "KeyId": "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321",
            "GranteePrincipal": "dynamodb.us-west-2.amazonaws.com",
            "RetiringPrincipal": "dynamodb.us-west-2.amazonaws.com",
            "Name": "8276b9a6-6cf0-46f1-b2f0-7993a7f8c89a",
            "GrantId": "1667b97d27cf748cf05b487217dd4179526c949d14fb3903858e25193253fe59"
        }
    ]
}
```

## Granting CreateGrant permission<a name="grant-creategrant"></a>

A grant can include permission to call the `CreateGrant` operation\. But when a [grantee principal](grants.md#terms-grantee-principal) gets permission to call `CreateGrant` from a grant, rather than from a policy, that permission is limited\. 
+ The grantee principal can only create grants that allow some or all of the operations in the parent grant\.
+ The [grant constraints](#grant-constraints) in the grants they create must be at least as strict as those in the parent grant\.

These limitations don't apply to principals who get `CreateGrant` permission from a policy, although their permissions can be limited by [policy conditions](grant-manage.md#grant-authorization)\.

For example, consider a grant that allows the grantee principal to call the `GenerateDataKey`, `Decrypt`, and `CreateGrant` operations\. We call a grant that allow `CreateGrant` permission a *parent grant*\.

```
# The original grant in a ListGrants response.
{
    "Grants": [
        {
            "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "CreationDate": 1572216195.0,
            "GrantId": "abcde1237f76e4ba7987489ac329fbfba6ad343d6f7075dbd1ef191f0120514a",
            "Operations": [
                "GenerateDataKey",
                "Decrypt",
                "CreateGrant
            ]
            "RetiringPrincipal": "arn:aws:iam::111122223333:role/adminRole",
            "Name": "",
            "IssuingAccount": "arn:aws:iam::111122223333:root",
            "GranteePrincipal": "arn:aws:iam::111122223333:user/exampleUser",
            "Constraints": {
                "EncryptionContextSubset": {
                    "Department": "IT"
                }
            },
        }
    ]
}
```

The grantee principal, exampleUser, can use this permission to create a grant that includes any subset of the operations specified in the original grant, such as `CreateGrant` and `Decrypt`\. The *child grant* cannot include other operations, such as `ScheduleKeyDeletion` or `ReEncrypt`\.

Also, the [grant constraints](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) in child grants must be as restrictive or more restrictive than those in the parent grant\. For example, the child grant can add pairs to an `EncryptionContextSubset` constraint in the parent grant, but it cannot remove them\. The child grant can change an `EncryptionContextSubset` constraint to an `EncryptionContextEquals` constraint, but not the reverse\.

For example, the grantee principal can use the `CreateGrant` permission that it got from the parent grant to create the following child grant\. The operations in the child grant are a subset of the operations in the parent grant and the grant constraints are more restrictive\.

```
# The child grant in a ListGrants response.
{
    "Grants": [
        {
            "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "CreationDate": 1572249600.0,
            "GrantId": "fedcba9999c1e2e9876abcde6e9d6c9b6a1987650000abcee009abcdef40183f",
            "Operations": [
                "CreateGrant"
                "Decrypt"
            ]
            "RetiringPrincipal": "arn:aws:iam::111122223333:user/exampleUser",
            "Name": "",
            "IssuingAccount": "arn:aws:iam::111122223333:root",
            "GranteePrincipal": "arn:aws:iam::111122223333:user/anotherUser",
            "Constraints": {
                "EncryptionContextEquals": {
                    "Department": "IT"
                }
            },
        }
    ]
}
```

The grantee principal in the child grant, `anotherUser`, can use their `CreateGrant` permission to create grants\. However, the grants that `anotherUser` creates must include the operations in its parent grant or a subset, and the grant constraints must be the same or stricter\. 