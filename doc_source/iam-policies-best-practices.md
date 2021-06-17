# Best practices for IAM policies<a name="iam-policies-best-practices"></a>

Securing access to AWS KMS customer master keys \(CMKs\) is critical to the security of all of your AWS resources\. AWS KMS CMKs are used to protect many of the most sensitive resources in your AWS account\. Take the time to design the [key policies](key-policies.md), IAM policies, [grants](grants.md), and [VPC endpoint policies](kms-vpc-endpoint.md#vpce-policy) that control access to your CMKs\.

In IAM policy statements that control access to CMKs, use the [least privileged principle](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege)\. Give IAM principals only the permissions they need on only the CMKs they must use or manage\.

**Use key policies**  
Whenever possible, provide permissions in key policies that affect one CMK, rather than in an IAM policy that can apply to many CMKs, including those in other AWS accounts\. This is particularly important for sensitive permissions like [kms:PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) and [kms:ScheduleKeyDeletion](https://docs.aws.amazon.com/kms/latest/APIReference/API_ScheduleKeyDeletion.html) but also for cryptographic operations that determine how your data is protected\.

**Limit CreateKey permission**  
Give permission to create keys \([kms:CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html)\) only to principals who need it\. Principals who create a CMK also set its key policy, so they can give themselves and others permission to use and manage the CMKs they create\. When you allow this permission, consider limiting it by using [policy conditions](policy-conditions.md)\. For example, you can use the `kms:CustomerMasterKeySpec` condition to limit the permission to symmetric CMKs\.

**Specify CMKs in an IAM policy**  
As a best practice, specify the [key ARN](concepts.md#key-id-key-ARN) of each CMK to which the permission applies in the `Resource` element of the policy statement\. This practice restricts the permission to the CMKs that principal requires\. For example, this `Resource` element lists only the CMKs the principal needs to use\.  

```
"Resource": [
    "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
    "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321"
]
```
When specifying CMKs is impractical, use a `Resource` value that limits access to CMKs in a trusted AWS account and Region, such as `arn:aws:kms:region:account:key/*`\. Or limit access to CMKs in all Regions \(\*\) of a trusted AWS account, such as `arn:aws:kms:*:account:key/*`\.  
You cannot use a [key ID](concepts.md#key-id-key-id), [alias name](concepts.md#key-id-alias-name), or [alias ARN](concepts.md#key-id-alias-ARN) to represent a CMK in the `Resource` field of an IAM policy\. If you specify an alias ARN, the policy applies to the alias, not to the CMK\. For information about IAM policies for aliases, see [Controlling access to aliases](alias-access.md)

**Avoid "Resource": "\*" in an IAM policy**  <a name="avoid-resource-star"></a>
Use wildcard characters \(\*\) judiciously\. In a key policy, the wildcard character in the `Resource` element represents the CMK to which the key policy is attached\. But in an IAM policy, a wildcard character alone in the `Resource` element \(`"Resource": "*"`\) applies the permissions to all CMKs in all AWS accounts that the principal's account has permission to use\. This might include [CMKs in other AWS accounts](key-policy-modifying-external-accounts.md), as well as CMKs in the principal's account\.  
For example, to use a CMK in another AWS account, a principal needs permission from the key policy of the CMK in the external account, and from an IAM policy in their own account\. Suppose that an arbitrary account gave your AWS account [kms:Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) permission on their CMKs\. If so, an IAM policy in your account that gives a role `kms:Decrypt` permission on all CMKs \(`"Resource": "*"`\) would satisfy the IAM part of the requirement\. As a result, principals who can assume that role can now decrypt ciphertexts using the CMK in the untrusted account\. Entries for their operations appear in the CloudTrail logs of both accounts\.  
In particular, avoid using `"Resource": "*"` in a policy statement that allows the following API operations\. These operations can be called on CMKs in other AWS accounts\.  
+ [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html)
+ [GetKeyRotationStatus](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyRotationStatus.html)
+ [Cryptographic operations](concepts.md#cryptographic-operations) \([Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html), [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html), [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html), [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html), [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html), [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html)\)
+ [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html), [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html), [ListRetirableGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListRetirableGrants.html), [RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html), [RevokeGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html)

**When to use "Resource": "\*"**  <a name="require-resource-star"></a>
In an IAM policy, use a wildcard character in the `Resource` element only for permissions that require it\. Only the following permissions require the `"Resource": "*"` element\.  
+ [kms:CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html)
+ [kms:GenerateRandom](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateRandom.html)
+ [kms:ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html)
+ [kms:ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html)
+ Permissions for custom key stores, such as [kms:CreateCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateCustomKeyStore.html) and [kms:ConnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_ConnectCustomKeyStore.html)\.
Permissions for alias operations \([kms:CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html), [kms:UpdateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html), [kms:DeleteAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteAlias.html)\) must be attached to the alias and the CMK\. You can use `"Resource": "*"` in an IAM policy to represent the aliases and the CMKs, or specify the aliases and CMKs in the `Resource` element\. For examples, see [Controlling access to aliases](alias-access.md)\.

 

The examples in this topic provide more information and guidance for designing IAM policies for CMKs\. For general AWS KMS best practice guidance, see the [AWS Key Management Service Best Practices](https://d0.awsstatic.com/whitepapers/aws-kms-best-practices.pdf)\. For IAM best practices for all AWS resources, see [Security best practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) in the *IAM User Guide*\.