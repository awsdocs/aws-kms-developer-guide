# Using policy conditions with AWS KMS<a name="policy-conditions"></a>

You can specify conditions in the key policies and AWS Identity and Access Management policies \([IAM policies](iam-policies.md)\) that control access to AWS KMS resources\. The policy statement is effective only when the conditions are true\. For example, you might want a policy statement to take effect only after a specific date\. Or, you might want a policy statement to control access only when a specific value appears in an API request\.

To specify conditions, you use predefined *condition keys* in the `Condition` element of a policy statement with [IAM condition policy operators](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html)\. Some condition keys apply generally to AWS others are specific to AWS KMS\.

**Topics**
+ [AWS global condition keys](#conditions-aws)
+ [AWS KMS condition keys](#conditions-kms)
+ [AWS KMS condition keys for AWS Nitro Enclaves](#conditions-nitro-enclaves)

## AWS global condition keys<a name="conditions-aws"></a>

AWS defines [global condition keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#AvailableKeys), a set of policy conditions keys for all AWS services that use IAM for access control\. AWS KMS supports all global condition keys\. You can use them in AWS KMS key policies and IAM policies\.  However, AWS KMS key policies do not permit the usage of the aws:ResourceTag condition key\.

For example, you can use the [aws:PrincipalArn](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-principalarn) global condition key to allow access to an AWS KMS key \(KMS key\) only when the principal in the request is represented by the Amazon Resource Name \(ARN\) in the condition key value\. To support [attribute\-based access control](abac.md) \(ABAC\) in AWS KMS, you can use the [aws:ResourceTag/*tag\-key*](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-resourcetag) global condition key in an IAM policy to allow access to KMS keys with a particular tag\.

To help prevent an AWS service from being used as a confused deputy in a policy where the principal is an [AWS service principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services), you can use the [https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourcearn](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourcearn) or [https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourceaccount](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourceaccount) global condition keys\. For details, see [Using `aws:SourceArn` or `aws:SourceAccount` condition keys](key-policy-services.md#least-privilege-source-arn)\.

For information about AWS global condition keys, including the types of requests in which they are available, see [AWS Global Condition Context Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) in the *IAM User Guide*\. For examples of using global condition keys in IAM policies, see [Controlling Access to Requests](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_tags.html#access_tags_control-requests) and [Controlling Tag Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_tags.html#access_tags_control-tag-keys) in the *IAM User Guide*\.

The following topics provide special guidance for using condition keys based on IP addresses and VPC endpoints\.

**Topics**
+ [Using the IP address condition in policies with AWS KMS permissions](#conditions-aws-ip-address)
+ [Using VPC endpoint conditions in policies with AWS KMS permissions](#conditions-aws-vpce)

### Using the IP address condition in policies with AWS KMS permissions<a name="conditions-aws-ip-address"></a>

You can use AWS KMS to protect your data in an [integrated AWS service](service-integration.md)\. But use caution when specifying the [IP address condition operators](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_IPAddress) or the `aws:SourceIp` condition key in the same policy statement that allows or denies access to AWS KMS\. For example, the policy in [AWS: Denies Access to AWS Based on the Source IP](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_aws_deny-ip.html) restricts AWS actions to requests from the specified IP range\.

Consider this scenario:

1. You attach a policy like the one shown at [AWS: Denies Access to AWS Based on the Source IP](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_aws_deny-ip.html) to an IAM user\. You set the value of the `aws:SourceIp` condition key to the range of IP addresses for the user's company\. This IAM user has other policies attached that allow it to use Amazon EBS, Amazon EC2, and AWS KMS\.

1. The user attempts to attach an encrypted EBS volume to an EC2 instance\. This action fails with an authorization error even though the user has permission to use all the relevant services\.

Step 2 fails because the request to AWS KMS to decrypt the volume's encrypted data key comes from an IP address that is associated with the Amazon EC2 infrastructure\. To succeed, the request must come from the IP address of the originating user\. Because the policy in step 1 explicitly denies all requests from IP addresses other than those specified, Amazon EC2 is denied permission to decrypt the EBS volume's encrypted data key\.

Also, the `aws:sourceIP` condition key is not effective when the request comes from an [Amazon VPC endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html)\. To restrict requests to a VPC endpoint, including an [AWS KMS VPC endpoint](kms-vpc-endpoint.md), use the `aws:sourceVpce` or `aws:sourceVpc` condition keys\. For more information, see [VPC Endpoints \- Controlling the Use of Endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html#vpc-endpoints-iam-access) in the *Amazon VPC User Guide*\. 

### Using VPC endpoint conditions in policies with AWS KMS permissions<a name="conditions-aws-vpce"></a>

[AWS KMS supports Amazon Virtual Private Cloud \(Amazon VPC\) endpoints](kms-vpc-endpoint.md) that are powered by [AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Introduction.html#what-is-privatelink)\. You can use the following [global condition keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#AvailableKeys) in key policies and IAM policies to control access to AWS KMS resources when the request comes from a VPC or uses a VPC endpoint\. For details, see [Using a VPC endpoint in a policy statement](kms-vpc-endpoint.md#vpce-policy-condition)\.
+ `aws:SourceVpc` limits access to requests from the specified VPC\. 
+ `aws:SourceVpce` limits access to requests from the specified VPC endpoint\. 

If you use these condition keys to control access to KMS keys, you might inadvertently deny access to AWS services that use AWS KMS on your behalf\. 

Take care to avoid a situation like the [IP address condition keys](#conditions-aws-ip-address) example\. If you restrict requests for a KMS key to a VPC or VPC endpoint, calls to AWS KMS from an integrated service, such as Amazon S3 or Amazon EBS, might fail\. This can happen even if the source request ultimately originates in the VPC or from the VPC endpoint\. 

## AWS KMS condition keys<a name="conditions-kms"></a>

AWS KMS provides an additional set of predefined condition keys that you can use in key policies and IAM policies\. These condition keys are specific to AWS KMS\. For example, you can use the `kms:EncryptionContext:context-key` condition key to require a particular [encryption context](concepts.md#encrypt_context) when controlling access to a symmetric KMS key\.

**Conditions for an API operation request**

Many AWS KMS condition keys control access to a KMS key based on the value of a parameter in the request for an AWS KMS operation\. For example, you can use the [kms:KeySpec](#conditions-kms-key-spec) condition key in an IAM policy to allow use of the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation only when the value of the `KeySpec` parameter in the `CreateKey` request is `RSA_4096`\. 

This type of condition works even when the parameter doesn't appear in the request, such as when you use the parameter's default value\. For example you can use the [kms:KeySpec](#conditions-kms-key-spec) condition key to allow users to use the `CreateKey` operation only when the value of the `KeySpec` parameter is `SYMMETRIC_DEFAULT`, which is the default value\. This condition allows requests that have the `KeySpec` parameter with the `SYMMETRIC_DEFAULT` value and requests that have no `KeySpec` parameter\.

**Conditions for KMS keys used in API operations**

Some AWS KMS condition keys can control access to operations based on a property of the KMS key that is used in the operation\. For example, you can use the [kms:KeyOrigin](#conditions-kms-key-origin) condition to allow principals to call [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) on a KMS key only when the `Origin` of the KMS key is `AWS_KMS`\. To find out if a condition key can be used in this way, see the description of the condition key\.

The operation must be a *KMS key resource operation*, that is, an operation that is authorized for a particular KMS key\. To identify the KMS key resource operations, in the [Actions and Resources Table](kms-api-permissions-reference.md#kms-api-permissions-reference-table), look for a value of `KMS key` in the `Resources` column for the operation\. If you use this type of condition key with an operation that is not authorized for a particular KMS key resource, like [ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html), the permission is not effective because the condition can never be satisfied\. There is no KMS key resource involved in authorizing the `ListKeys` operation and no `KeySpec` property\. 

The following topics describe each AWS KMS condition key and include example policy statements that demonstrate policy syntax\.

**Using set operators with condition keys**

When a policy condition compares two set of values, such as the set of tags in a request and the set of tags in a policy, you need tell AWS how to compare the sets\. IAM defines two set operators, `ForAnyValue` and `ForAllValues`, for this purpose\. Use set operators only with *multi\-valued condition keys*, which require them\. Do not use set operators with *single\-valued condition keys*\. As always, test your policy statements thoroughly before using them in a production environment\.

Condition keys are single\-valued or multi\-valued\. To determine whether an AWS KMS condition key is single\-valued or multi\-valued, see the **Value type** column in the condition key description\. 
+ *Single\-valued * condition keys have at most one value in the authorization context \(the request or resource\)\. For example, because each API call can originate from only one AWS account, [kms:CallerAccount](#conditions-kms-caller-account) is a single valued condition key\. Do not use a set operator with a single\-valued condition key\. 
+ *Multi\-valued* condition keys have multiple values in the authorization context \(the request or resource\)\. For example, because each KMS key can have multiple aliases, [kms:ResourceAliases](#conditions-kms-resource-aliases) can have multiple values\. Multi\-valued condition keys require a set operator\. 

Note that the difference between single\-valued and multi\-valued condition keys depends on the number of values in the authorization context; not the number of values in the policy condition\.

**Warning**  
Using a set operator with a single\-valued condition key can create a policy statement that is overly permissive \(or overly restrictive\)\. Use set operators only with multi\-valued condition keys\.  
If you create or update a policy that includes a `ForAllValues` set operator with the kms:EncryptionContext:*context\-key* or `aws:RequestTag/tag-key` condition keys, AWS KMS returns the following error message:  
`OverlyPermissiveCondition: Using the ForAllValues set operator with a single-valued condition key matches requests without the specified [encryption context or tag] or with an unspecified [encryption context or tag]. To fix, remove ForAllValues.`

For detailed information about the `ForAnyValue` and `ForAllValues` set operators, see [Using multiple keys and values](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_multi-value-conditions.html#reference_policies_multi-key-or-value-conditions) in the *IAM User Guide*\. For information about the risk of using the `ForAllValues` set operator with a single\-valued condition, see [Security Warning – ForAllValues with single valued key](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-reference-policy-checks.html#access-analyzer-reference-policy-checks-security-warning-forallvalues-with-single-valued-key) in the *IAM User Guide*\.

**Topics**
+ [kms:BypassPolicyLockoutSafetyCheck](#conditions-kms-bypass-policy-lockout-safety-check)
+ [kms:CallerAccount](#conditions-kms-caller-account)
+ [kms:CustomerMasterKeySpec \(deprecated\)](#conditions-kms-key-spec-replaced)
+ [kms:CustomerMasterKeyUsage \(deprecated\)](#conditions-kms-key-usage-replaced)
+ [kms:DataKeyPairSpec](#conditions-kms-data-key-spec)
+ [kms:EncryptionAlgorithm](#conditions-kms-encryption-algorithm)
+ [kms:EncryptionContext:*context\-key*](#conditions-kms-encryption-context)
+ [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys)
+ [kms:ExpirationModel](#conditions-kms-expiration-model)
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)
+ [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)
+ [kms:GrantOperations](#conditions-kms-grant-operations)
+ [kms:GranteePrincipal](#conditions-kms-grantee-principal)
+ [kms:KeyOrigin](#conditions-kms-key-origin)
+ [kms:KeySpec](#conditions-kms-key-spec)
+ [kms:KeyUsage](#conditions-kms-key-usage)
+ [kms:MessageType](#conditions-kms-message-type)
+ [kms:MultiRegion](#conditions-kms-multiregion)
+ [kms:MultiRegionKeyType](#conditions-kms-multiregion-key-type)
+ [kms:PrimaryRegion](#conditions-kms-primary-region)
+ [kms:ReEncryptOnSameKey](#conditions-kms-reencrypt-on-same-key)
+ [kms:RequestAlias](#conditions-kms-request-alias)
+ [kms:ResourceAliases](#conditions-kms-resource-aliases)
+ [kms:ReplicaRegion](#conditions-kms-replica-region)
+ [kms:RetiringPrincipal](#conditions-kms-retiring-principal)
+ [kms:SigningAlgorithm](#conditions-kms-signing-algorithm)
+ [kms:ValidTo](#conditions-kms-valid-to)
+ [kms:ViaService](#conditions-kms-via-service)
+ [kms:WrappingAlgorithm](#conditions-kms-wrapping-algorithm)
+ [kms:WrappingKeySpec](#conditions-kms-wrapping-key-spec)

### kms:BypassPolicyLockoutSafetyCheck<a name="conditions-kms-bypass-policy-lockout-safety-check"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:BypassPolicyLockoutSafetyCheck`  |  Boolean  | Single\-valued |  `CreateKey` `PutKeyPolicy`  |  IAM policies only Key policies and IAM policies  | 

The `kms:BypassPolicyLockoutSafetyCheck` condition key controls access to the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) and [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operations based on the value of the `BypassPolicyLockoutSafetyCheck` parameter in the request\. 

The following example IAM policy statement prevents users from bypassing the policy lockout safety check by denying them permission to create KMS keys when the value of the `BypassPolicyLockoutSafetyCheck` parameter in the `CreateKey` request is `true.` 

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Deny",
    "Action": "kms:CreateKey",
    "Resource": "*",
    "Condition": {
      "Bool": {
        "kms:BypassPolicyLockoutSafetyCheck": true
      }
    }
  }
}
```

You can also use the `kms:BypassPolicyLockoutSafetyCheck` condition key in an IAM policy or key policy to control access to the `PutKeyPolicy` operation\. The following example policy statement from a key policy prevents users from bypassing the policy lockout safety check when changing the policy of a KMS key\. 

Instead of using an explicit `Deny`, this policy statement uses `Allow` with the [Null condition operator](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_Null) to allow access only when the request does not include the `BypassPolicyLockoutSafetyCheck` parameter\. When the parameter is not used, the default value is `false`\. This slightly weaker policy statement can be overridden in the rare case that a bypass is necessary\. 

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": "kms:PutKeyPolicy",
    "Resource": "*",
    "Condition": {
      "Null": {
        "kms:BypassPolicyLockoutSafetyCheck": true
      }
    }
  }
}
```

**See also**
+ [kms:KeySpec](#conditions-kms-key-spec)
+ [kms:KeyOrigin](#conditions-kms-key-origin)
+ [kms:KeyUsage](#conditions-kms-key-usage)

### kms:CallerAccount<a name="conditions-kms-caller-account"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:CallerAccount`  |  String  | Single\-valued |  KMS key resource operations Custom key store operations  |  Key policies and IAM policies  | 

You can use this condition key to allow or deny access to all identities \(IAM users and roles\) in an AWS account\. In key policies, you use the `Principal` element to specify the identities to which the policy statement applies\. The syntax for the `Principal` element does not provide a way to specify all identities in an AWS account\. But you can achieve this effect by combining this condition key with a `Principal` element that specifies all AWS identities\.

You can use it to control access to any *KMS key resource operation*, that is, any AWS KMS operation that uses a particular KMS key\. To identify the KMS key resource operations, in the [Actions and Resources Table](kms-api-permissions-reference.md#kms-api-permissions-reference-table), look for a value of `KMS key` in the `Resources` column for the operation\. It is also valid for operations that manage [custom key stores](custom-key-store-overview.md)\.

For example, the following key policy statement demonstrates how to use the `kms:CallerAccount` condition key\. This policy statement is in the key policy for the AWS managed key for Amazon EBS\. It combines a `Principal` element that specifies all AWS identities with the `kms:CallerAccount` condition key to effectively allow access to all identities in AWS account 111122223333\. It contains an additional AWS KMS condition key \(`kms:ViaService`\) to further limit the permissions by only allowing requests that come through Amazon EBS\. For more information, see [kms:ViaService](#conditions-kms-via-service)\.

```
{
  "Sid": "Allow access through EBS for all principals in the account that are authorized to use EBS",
  "Effect": "Allow",
  "Principal": {"AWS": "*"},
  "Condition": {
    "StringEquals": {
      "kms:CallerAccount": "111122223333",
      "kms:ViaService": "ec2.us-west-2.amazonaws.com"
    }
  },
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:ReEncrypt*",
    "kms:GenerateDataKey*",
    "kms:CreateGrant",
    "kms:DescribeKey"
  ],
  "Resource": "*"
}
```

### kms:CustomerMasterKeySpec \(deprecated\)<a name="conditions-kms-key-spec-replaced"></a>

The `kms:CustomerMasterKeySpec` condition key is deprecated\. Instead, use the [kms:KeySpec](#conditions-kms-key-spec) condition key\.

The `kms:CustomerMasterKeySpec` and `kms:KeySpec` condition keys work the same way\. Only the names differ\. We recommend that you use `kms:KeySpec`\. However, to avoid breaking changes, AWS KMS supports both condition keys\.

### kms:CustomerMasterKeyUsage \(deprecated\)<a name="conditions-kms-key-usage-replaced"></a>

The `kms:CustomerMasterKeyUsage` condition key is deprecated\. Instead, use the [kms:KeyUsage](#conditions-kms-key-usage) condition key\.

The `kms:CustomerMasterKeyUsage` and `kms:KeyUsage` condition keys work the same way\. Only the names differ\. We recommend that you use `kms:KeyUsage`\. However, to avoid breaking changes, AWS KMS supports both condition keys\.

### kms:DataKeyPairSpec<a name="conditions-kms-data-key-spec"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:DataKeyPairSpec`  |  String  | Single\-valued |  `GenerateDataKeyPair` `GenerateDataKeyPairWithoutPlaintext`  |  Key policies and IAM policies  | 

You can use this condition key to control access to the [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html) and [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html) operations based on the value of the `KeyPairSpec` parameter in the request\. For example, you can allow a user to generate only particular types of data key pairs\.

The following example key policy statement uses the `kms:DataKeyPairSpec` condition key to allow a user to use the KMS key to generate only RSA data key pairs\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": [
    "kms:GenerateDataKeyPair",
    "kms:GenerateDataKeyPairWithoutPlaintext"
  ],
  "Resource": "*",
  "Condition": {
    "StringLike": {
      "kms:DataKeyPairSpec": "RSA*"
    }
  }
}
```

**See also**
+ [kms:KeySpec](#conditions-kms-key-spec)
+ [kms:EncryptionAlgorithm](#conditions-kms-encryption-algorithm)
+ [kms:EncryptionContext:*context\-key*](#conditions-kms-encryption-context)
+ [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys)

### kms:EncryptionAlgorithm<a name="conditions-kms-encryption-algorithm"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:EncryptionAlgorithm`  |  String  | Single\-valued |  `Decrypt` `Encrypt` `GenerateDataKey` `GenerateDataKeyPair` `GenerateDataKeyPairWithoutPlaintext` `GenerateDataKeyWithoutPlaintext` `ReEncrypt`  |  Key policies and IAM policies  | 

You can use the `kms:EncryptionAlgorithm` condition key to control access to cryptographic operations based on the encryption algorithm that is used in the operation\. For the [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), and [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) operations, it controls access based on the value of the [EncryptionAlgorithm](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html#KMS-Decrypt-request-EncryptionAlgorithm) parameter in the request\. For operations that generate data keys and data key pairs, it controls access based on the encryption algorithm that is used to encrypt the data key\.

This condition key has no effect on operations performed outside of AWS KMS, such as encrypting with the public key in an asymmetric KMS key pair outside of AWS KMS\.

**EncryptionAlgorithm parameter in a request**

To allow users to use only a particular encryption algorithm with a KMS key, use a policy statement with a `Deny` effect and a `StringNotEquals` condition operator\. For example, the following example key policy statement prohibits principals who can assume the `ExampleRole` role from using this symmetric KMS key in the specified cryptographic operations unless the encryption algorithm in the request is `RSAES_OAEP_SHA_256`\. 

Unlike a policy statement that allows a user to use a particular encryption algorithm, a policy statement with a double\-negative like this one prevents other policies and grants for this KMS key from allowing this role to use other encryption algorithms\. The `Deny` in this key policy statement takes precedence over any key policy or IAM policy with an `Allow` effect, and it takes precedence over all grants for this KMS key and its principals\.

```
{
  "Sid": "Allow only one encryption algorithm with this asymmetric KMS key",
  "Effect": "Deny",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/ExampleRole"
  },
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:ReEncrypt*"
  ],
  "Resource": "*",
  "Condition": {
    "StringNotEquals": {
      "kms:EncryptionAlgorithm": "RSAES_OAEP_SHA_256"
    }
  }
}
```

**Encryption algorithm used for the operation**

You can also use the `kms:EncryptionAlgorithm` condition key to control access to operations based on the encryption algorithm used in the operation, even when the algorithm isn't specified in the request\. This allows you to require or forbid the `SYMMETRIC_DEFAULT` algorithm, which might not be specified in a request because it's the default value\.

This feature lets you use the `kms:EncryptionAlgorithm` condition key to control access to the operations that generate data keys and data key pairs\. These operations use only symmetric KMS keys and the `SYMMETRIC_DEFAULT` algorithm\.

For example, this IAM policy limits its principals to symmetric encryption\. It denies access to any KMS key in the example account for cryptographic operations unless the encryption algorithm specified in the request or used in the operation is SYMMETRIC\_DEFAULT\. Including `GenerateDataKey*` adds [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html), and [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html) to the permissions\. The condition has no effect on these operations because they always use a symmetric encryption algorithm\.

```
{
  "Sid": "AllowOnlySymmetricAlgorithm",
  "Effect": "Deny",
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:ReEncrypt*",
    "kms:GenerateDataKey*"
  ],
  "Resource": "arn:aws:kms:us-west-2:111122223333:key/*",
  "Condition": {
    "StringNotEquals": {
      "kms:EncryptionAlgorithm": "SYMMETRIC_DEFAULT"
    }
  }
}
```

**See also**
+ [kms:SigningAlgorithm](#conditions-kms-signing-algorithm)

### kms:EncryptionContext:*context\-key*<a name="conditions-kms-encryption-context"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:EncryptionContext:context-key`  |  String  | Single\-valued |  `CreateGrant` `Encrypt` `Decrypt` `GenerateDataKey` `GenerateDataKeyPair` `GenerateDataKeyPairWithoutPlaintext` `GenerateDataKeyWithoutPlaintext` `ReEncrypt`  |  Key policies and IAM policies  | 

You can use the `kms:EncryptionContext:context-key` condition key to control access to a [symmetric KMS key](concepts.md#symmetric-cmks) based on the [encryption context](concepts.md#encrypt_context) in a request for a [cryptographic operation](concepts.md#cryptographic-operations)\. Use this condition key to evaluate both the key and the value in the encryption context pair\. To evaluate only the encryption context keys or require an encryption context regardless of keys or values, use the [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys) condition key\.

**Note**  
This condition key is valid in key policy statements and IAM policy statements even though it does not appear in the IAM console or the IAM *Service Authorization Reference*\.

You cannot specify an encryption context in a cryptographic operation with an [asymmetric KMS key](symmetric-asymmetric.md#asymmetric-cmks)\. The standard asymmetric encryption algorithms that AWS KMS uses do not support an encryption context\.

To use the kms:EncryptionContext:*context\-key* condition key, replace the *context\-key* placeholder with the encryption context key\. Replace the *context\-value* placeholder with the encryption context value\.

```
"kms:EncryptionContext:context-key": "context-value"
```

For example, the following condition key specifies an encryption context in which the key is `AppName` and the value is `ExampleApp` \(`AppName = ExampleApp`\)\.

```
"kms:EncryptionContext:AppName": "ExampleApp"
```

This is a [single\-valued condition key](#set-operators)\. The key in the condition key specifies a particular encryption context key \(*context\-key*\)\. Although you can include multiple encryption context pairs in each API request, the encryption context pair with the specified *context\-key* can have only one value\. For example, the `kms:EncryptionContext:Department` condition key applies only to encryption context pairs with a `Department` key, and any given encryption context pair with the `Department` key can have only one value\.

Do not use a set operator with the `kms:EncryptionContext:context-key` condition key\. If you create a policy statement with an `Allow` action, the `kms:EncryptionContext:context-key` condition key, and the `ForAllValues` set operator, the condition allows requests with no encryption context and requests with encryption context pairs that are not specified in the policy condition\.

**Warning**  
Do not use a `ForAnyValue` or `ForAllValues` set operator with this single\-valued condition key\. These set operators can create a policy condition that does not require values you intend to require and allows values you intend to forbid\.  
If you create or update a policy that includes a `ForAllValues` set operator with the kms:EncryptionContext:*context\-key*, AWS KMS returns the following error message:  
`OverlyPermissiveCondition:EncryptionContext: Using the ForAllValues set operator with a single-valued condition key matches requests without the specified encryption context or with an unspecified encryption context. To fix, remove ForAllValues.`

To require a particular encryption context pair, use the `kms:EncryptionContext:context-key` condition key with the `StringEquals` operator \.

The following example key policy statement allows principals who can assume the role to use the KMS key in a `GenerateDataKey` request only when the encryption context in the request includes the `AppName:ExampleApp` pair\. Other encryption context pairs are permitted\.

The key name is not case sensitive\. The case sensitivity of the value is determined by the condition operator, such as `StringEquals`\. For details, see [Case sensitivity of the encryption context condition](#conditions-kms-encryption-context-case)\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/RoleForExampleApp"
  },
  "Action": "kms:GenerateDataKey",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:EncryptionContext:AppName": "ExampleApp"
    }
  }
}
```

To require an encryption context pair and forbid all other encryption context pairs, use both kms:EncryptionContext:*context\-key* and [`kms:EncryptionContextKeys`](#conditions-kms-encryption-context-keys) in the policy statement\. The following key policy statement uses the `kms:EncryptionContext:AppName` condition to require the `AppName=ExampleApp` encryption context pair in the request\. It also uses a `kms:EncryptionContextKeys` condition key with the `ForAllValues` set operator to allow only the `AppName` encryption context key\. 

The `ForAllValues` set operator limits encryption context keys in the request to `AppName`\. If the `kms:EncryptionContextKeys` condition with the `ForAllValues` set operator was used alone in a policy statement, this set operator would allow requests with no encryption context\. However, if the request had no encryption context, the `kms:EncryptionContext:AppName` condition would fail\. For details about the `ForAllValues` set operator, see [Using multiple keys and values](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_multi-value-conditions.html#reference_policies_multi-key-or-value-conditions) in the *IAM User Guide*\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::712816755609:user/alice"
  },
  "Action": "kms:GenerateDataKey",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:EncryptionContext:AppName": "ExampleApp"
    },
    "ForAllValues:StringEquals": {
      "kms:EncryptionContextKeys": [
        "AppName"
      ]
    }
  }
}
```

You can also use this condition key to deny access to a KMS key for a particular operation\. The following example key policy statement uses a `Deny` effect to forbid the principal from using the KMS key if the encryption context in the request includes a `Stage=Restricted` encryption context pair\. This condition allows a request with other encryption context pairs, including encryption context pairs with the `Stage` key and other values, such as `Stage=Test`\.

```
{
  "Effect": "Deny",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/RoleForExampleApp"
  },
  "Action": "kms:GenerateDataKey",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:EncryptionContext:Stage": "Restricted"
    }
  }
}
```

#### Using multiple encryption context pairs<a name="conditions-kms-encryption-context-many"></a>

You can require or forbid multiple encryption context pairs\. You can also require one of several encryption context pairs\. For details about the logic used to interpret these conditions, see [Creating a condition with multiple keys or values](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_multi-value-conditions.html) in the IAM User Guide\. 

**Note**  
Earlier versions of this topic displayed policy statements that used the `ForAnyValue` and `ForAllValues` set operators with the kms:EncryptionContext:*context\-key* condition key\. Using a set operator with a [single\-valued condition key](#set-operators) can result in policies that allow requests with no encryption context and unspecified encryption context pairs\.   
For example, a policy condition with the `Allow` effect, the `ForAllValues` set operator, and the `"kms:EncryptionContext:Department": "IT"` condition key does not limit the encryption context to the "Department=IT" pair\. It allows requests with no encryption context and requests with unspecified encryption context pairs, such as `Stage=Restricted`\.  
Please review your policies and eliminate the set operator from any condition with kms:EncryptionContext:*context\-key*\. Attempts to create or update a policy with this format fail with an `OverlyPermissiveCondition` exception\. To resolve the error, delete the set operator\.

To require multiple encryption context pairs, list the pairs in the same condition\. The following example key policy statement requires two encryption context pairs, `Department=IT` and `Project=Alpha`\. Because the conditions have different keys \(`kms:EncryptionContext:Department` and `kms:EncryptionContext:Project`\), they are implicitly connected by an AND operator\. Other encryption context pairs are permitted, but not required\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/RoleForExampleApp"
  },
  "Action": "kms:Decrypt",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:EncryptionContext:Department": "IT",
      "kms:EncryptionContext:Project": "Alpha"
    }
  }
}
```

To require one encryption context pair OR another pair, place each condition key in a separate policy statement\. The following example key policy requires `Department=IT` *or* `Project=Alpha` pairs, or both\. Other encryption context pairs are permitted, but not required\.

```
{
 "Effect": "Allow",
 "Principal": {
  "AWS": "arn:aws:iam::111122223333:role/RoleForExampleApp"
 },
 "Action": "kms:GenerateDataKey",
 "Resource": "*",
 "Condition": {
  "StringEquals": {
   "kms:EncryptionContext:Department": "IT"
  }
 }
},
{
 "Effect": "Allow",
 "Principal": {
  "AWS": "arn:aws:iam::111122223333:role/RoleForExampleApp"
 },
 "Action": "kms:GenerateDataKey",
 "Resource": "*",
 "Condition": {
  "StringEquals": {
   "kms:EncryptionContext:Project": "Alpha"
  }
 }
}
```

To require particular encryption pairs and exclude all other encryption context pairs, use both kms:EncryptionContext:*context\-key* and [`kms:EncryptionContextKeys`](#conditions-kms-encryption-context-keys) in the policy statement\. The following key policy statement uses the kms:EncryptionContext:*context\-key* condition to require an encryption context with both `Department=IT` *and* `Project=Alpha` pairs\. It uses a `kms:EncryptionContextKeys` condition key with the `ForAllValues` set operator to allow only the `Department` and `Project` encryption context keys\. 

The `ForAllValues` set operator limits encryption context keys in the request to `Department` and `Project`\. If it were used alone in a condition, this set operator would allow requests with no encryption context, but in this configuration, the kms:EncryptionContext:*context\-key* in this condition would fail\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::712816755609:user/alice"
  },
  "Action": "kms:GenerateDataKey",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:EncryptionContext:Department": "IT",
      "kms:EncryptionContext:Project": "Alpha"
    },
    "ForAllValues:StringEquals": {
      "kms:EncryptionContextKeys": [
        "Department",
        "Project"
      ]
    }
  }
}
```

You can also forbid multiple encryption context pairs\. The following example key policy statement uses a `Deny` effect to forbid the principal from using the KMS keys if the encryption context in the request includes a `Stage=Restricted` or `Stage=Production`\.pair\. 

Multiple values \(`Restricted` and `Production`\) for the same key \(`kms:EncryptionContext:Stage`\) are implicitly connected by a OR\. For details, see [Evaluation logic for conditions with multiple keys or values](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_multi-value-conditions.html#reference_policies_multiple-conditions-eval) in the *IAM User Guide*\.

```
{
  "Effect": "Deny",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/RoleForExampleApp"
  },
  "Action": "kms:GenerateDataKey",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:EncryptionContext:Stage": [
         "Restricted",
         "Production"
      ] 
    }
  }
}
```

#### Case sensitivity of the encryption context condition<a name="conditions-kms-encryption-context-case"></a>

The encryption context that is specified in a decryption operation must be an exact, case\-sensitive match for the encryption context that is specified in the encryption operation\. Only the order of pairs in an encryption context with multiple pair can vary\.

However, in policy conditions, the condition key is not case sensitive\. The case sensitivity of the condition value is determined by the [policy condition operator](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html) that you use, such as `StringEquals` or `StringEqualsIgnoreCase`\.

As such, the condition key, which consists of the `kms:EncryptionContext:` prefix and the *`context-key`* replacement, is not case sensitive\. A policy that uses this condition does not check the case of either element of the condition key\. The case sensitivity of the value, that is, the *`context-value`* replacement, is determined by the policy condition operator\.

For example, the following policy statement allows the operation when the encryption context includes an `Appname` key, regardless of its capitalization\. The `StringEquals` condition requires that `ExampleApp` be capitalized as it is specified\. 

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/RoleForExampleApp"
  },
  "Action": "kms:Decrypt",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:EncryptionContext:Appname": "ExampleApp"
    }
  }
}
```

To require a case\-sensitive encryption context key, use the [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys) policy condition with a case\-sensitive condition operator, such as `StringEquals`\. In this policy condition, because the encryption context key is the value in this policy condition, its case sensitivity is determined by the condition operator\. 

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/RoleForExampleApp"
  },
  "Action": "kms:GenerateDataKey",
  "Resource": "*",
  "Condition": {
    "ForAnyValue:StringEquals": {
      "kms:EncryptionContextKeys": "AppName"
    }
  }
}
```

To require a case\-sensitive evaluation of both the encryption context key and value, use the `kms:EncryptionContextKeys` and kms:EncryptionContext:*context\-key* policy conditions together in the same policy statement\. The case\-sensitive condition operator \(such as `StringEquals`\) always applies to the value of the condition\. The encryption context key \(such as `AppName`\) is the value of the `kms:EncryptionContextKeys` condition\. The encryption context value \(such as `ExampleApp`\) is the value of the kms:EncryptionContext:*context\-key* condition\.

For example, in the following example key policy statement, because the `StringEquals` operator is case sensitive, both the encryption context key and the encryption context value are case sensitive\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/RoleForExampleApp"
  },
  "Action": "kms:GenerateDataKey",
  "Resource": "*",
  "Condition": {
    "ForAnyValue:StringEquals": {
      "kms:EncryptionContextKeys": "AppName"
    },
    "StringEquals": {
      "kms:EncryptionContext:AppName": "ExampleApp"
    }
  }
}
```

#### Using variables in an encryption context condition<a name="conditions-kms-encryption-context-variables"></a>

The key and value in an encryption context pair must be simple literal strings\. They cannot be integers or objects, or any type that is not fully resolved\. If you use a different type, such as an integer or float, AWS KMS interprets it as a literal string\.

```
"encryptionContext": {
    "department": "10103.0"
}
```

However, the value of the `kms:EncryptionContext:context-key` condition key can be an [IAM policy variable](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html)\. These policy variables are resolved at runtime based on values in the request\. For example, `aws:CurrentTime `resolves to the time of the request and `aws:username` resolves to the friendly name of the caller\.

You can use these policy variables to create a policy statement with a condition that requires very specific information in an encryption context, such as the caller's user name\. Because it contains a variable, you can use the same policy statement for all users who can assume the role\. You don't have to write a separate policy statement for each user\.

Consider a situation where you want to all users who can assume a role to use the same KMS key to encrypt and decrypt their data\. However, you want to allow them to decrypt only the data that they encrypted\. Start by requiring that every request to AWS KMS include an encryption context where the key is `user` and the value is the caller's AWS user name, such as the following one\.

```
"encryptionContext": {
    "user": "bob"
}
```

Then, to enforce this requirement, you can use a policy statement like the one in the following example\. This policy statement gives the `TestTeam` role permission to encrypt and decrypt data with the KMS key\. However, the permission is valid only when the encryption context in the request includes a `"user": "<username>"` pair\. To represent the user name, the condition uses the [https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html#policy-vars-infotouse](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html#policy-vars-infotouse) policy variable\.

When the request is evaluated, the caller's user name replaces the variable in the condition\. As such, the condition requires an encryption context of `"user": "bob"` for "bob" and `"user": "alice"` for "alice\."

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/TestTeam"
  },
  "Action": [
    "kms:Decrypt",
    "kms:Encrypt"
  ],
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:EncryptionContext:user": "${aws:username}"
    }
  }
}
```

You can use an IAM policy variable only in the value of the `kms:EncryptionContext:context-key` condition key\. You cannot use a variable in the key\.

You can also use [provider\-specific context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_oidc_user-id.html) in variables\. These context keys uniquely identify users who logged into AWS by using web identity federation\. 

Like all variables, these variables can be used only in the `kms:EncryptionContext:context-key` policy condition, not in the actual encryption context\. And they can be used only in the value of the condition, not in the key\.

For example, the following key policy statement is similar to the previous one\. However, the condition requires an encryption context where the key is `sub` and the value uniquely identifies a user logged into an Amazon Cognito user pool\. For details about identifying users and roles in Amazon Cognito, see [IAM Roles](https://docs.aws.amazon.com/cognito/latest/developerguide/iam-roles.html) in the [Amazon Cognito Developer Guide](https://docs.aws.amazon.com/cognito/latest/developerguide/)\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/TestTeam"
  },
  "Action": [
    "kms:Decrypt",
    "kms:Encrypt"
  ],
  "Resource": "*",
  "Condition": {
    "StringEquals": {
       "kms:EncryptionContext:sub": "${cognito-identity.amazonaws.com:sub}"
    }
  }
}
```

**See also**
+ [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys)
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)

### kms:EncryptionContextKeys<a name="conditions-kms-encryption-context-keys"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:EncryptionContextKeys`  |  String \(list\)  | Multi\-valued |  `CreateGrant` `Decrypt` `Encrypt` `GenerateDataKey` `GenerateDataKeyPair` `GenerateDataKeyPairWithoutPlaintext` `GenerateDataKeyWithoutPlaintext` `ReEncrypt`  |  Key policies and IAM policies  | 

You can use the `kms:EncryptionContextKeys` condition key to control access to a [symmetric KMS key](concepts.md#symmetric-cmks) based on the [encryption context](concepts.md#encrypt_context) in a request for a cryptographic operation\. Use this condition key to evaluate only the key in each encryption context pair\. To evaluate both the key and the value in the encryption context, use the `kms:EncryptionContext:context-key` condition key\.

You cannot specify an encryption context in a cryptographic operation with an [asymmetric KMS key](symmetric-asymmetric.md#asymmetric-cmks)\. The standard asymmetric encryption algorithms that AWS KMS uses do not support an encryption context\.

This is a [multi\-valued condition key](#set-operators)\. You can specifiy multiple encryption context pairs in each API request\. `kms:EncryptionContextKeys` compares the encryption context keys in the request to the set of encryption context keys in the policy\. To determine how these sets are compared, you must provide a `ForAnyValue` or `ForAllValues` set operator in the policy condition\. For details about the set operators, see [Using multiple keys and values](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_multi-value-conditions.html#reference_policies_multi-key-or-value-conditions) in the IAM User Guide\.
+ `ForAnyValue`: At least one encryption context key in the request must match an encryption context key in the policy condition\. Other encryption context keys are permitted\. If the request has no encryption context, the condition is not satisfied\.
+ `ForAllValues`: Every encryption context key in the request must match an encryption context key in the policy condition\. This set operator limits the encryption context keys to those in the policy condition\. It doesn't require any encryption context keys, but it forbids unspecified encryption context keys\.

The following example key policy statement uses the `kms:EncryptionContextKeys` condition key with the `ForAnyValue` set operator\. This policy statement allows use of a KMS key for the specified operations, but only when at least one of the encryption context pairs in the request includes the `AppName` key, regardless of its value\. 

For example, this key policy statement allows a `GenerateDataKey` request with two encryption context pairs, `AppName=Helper` and `Project=Alpha`, because the first encryption context pair satisfies the condition\. A request with only `Project=Alpha` or with no encryption context would fail\.

Because the [StringEquals](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html) condition operation is case sensitive, this policy statement requires the spelling and case of the encryption context key\. But you can use a condition operator that ignores the case of the key, such as `StringEqualsIgnoreCase`\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/RoleForExampleApp"
  },
  "Action": [
    "kms:Encrypt",
    "kms:GenerateDataKey*"
  ],
  "Resource": "*",
  "Condition": {
    "ForAnyValue:StringEquals": {
      "kms:EncryptionContextKeys": "AppName"
    }
  }
}
```

You can also use the `kms:EncryptionContextKeys` condition key to require an encryption context \(any encryption context\) in cryptographic operations that use the KMS key;\. 

The following example key policy statement uses the `kms:EncryptionContextKeys` condition key with the [Null condition operator](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_Null) to allow access to a KMS key only when encryption context in the API request is not null\. This condition does not check the keys or values of the encryption context\. It only verifies that the encryption context exists\. 

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/RoleForExampleApp"
  },
  "Action": [
    "kms:Encrypt",
    "kms:GenerateDataKey*"
  ],
  "Resource": "*",
  "Condition": {
    "Null": {
      "kms:EncryptionContextKeys": false
    }
  }
}
```

**See also**
+ [kms:EncryptionContext:*context\-key*](#conditions-kms-encryption-context)
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)

### kms:ExpirationModel<a name="conditions-kms-expiration-model"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:ExpirationModel`  |  String  | Single\-valued |  `ImportKeyMaterial`  |  Key policies and IAM policies  | 

The `kms:ExpirationModel` condition key controls access to the [ImportKeyMaterial](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html) operation based on the value of the [ExpirationModel](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html#KMS-ImportKeyMaterial-request-ExpirationModel) parameter in the request\. 

`ExpirationModel` is an optional parameter that determines whether the imported key material expires\. Valid values are `KEY_MATERIAL_EXPIRES` and `KEY_MATERIAL_DOES_NOT_EXPIRE`\. `KEY_MATERIAL_EXPIRES` is the default value\. 

The expiration date and time is determined by the value of the [ValidTo](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html#KMS-ImportKeyMaterial-request-ValidTo) parameter\. The `ValidTo` parameter is required unless the value of the `ExpirationModel` parameter is `KEY_MATERIAL_DOES_NOT_EXPIRE`\. You can also use the [kms:ValidTo](#conditions-kms-valid-to) condition key to require a particular expiration date as a condition for access\.

The following example policy statement uses the `kms:ExpirationModel` condition key to allow a user to import key material into a KMS key only when the request includes the `ExpirationModel` parameter and its value is `KEY_MATERIAL_DOES_NOT_EXPIRE`\. 

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": "kms:ImportKeyMaterial",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:ExpirationModel": "KEY_MATERIAL_DOES_NOT_EXPIRE"
    }
  }
}
```

You can also use the `kms:ExpirationModel` condition key to allow a user to import key material only when the key material expires\. The following example key policy statement uses the `kms:ExpirationModel` condition key with the [Null condition operator](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_Null) to allow a user to import key material only when the request does not have an `ExpirationModel` parameter\. The default value for ExpirationModel is `KEY_MATERIAL_EXPIRES`\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": "kms:ImportKeyMaterial",
  "Resource": "*",
  "Condition": {
    "Null": {
      "kms:ExpirationModel": true
    }
  }
}
```

**See also**
+ [kms:ValidTo](#conditions-kms-valid-to)
+ [kms:WrappingAlgorithm](#conditions-kms-wrapping-algorithm)
+ [kms:WrappingKeySpec](#conditions-kms-wrapping-key-spec)

### kms:GrantConstraintType<a name="conditions-kms-grant-constraint-type"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:GrantConstraintType`  |  String  | Single\-valued |  `CreateGrant`  |  Key policies and IAM policies  | 

You can use this condition key to control access to the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation based on the type of [grant constraint](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) in the request\. 

When you create a grant, you can optionally specify a grant constraint to allow the operations that the grant permit only when a particular [encryption context](concepts.md#encrypt_context) is present\. The grant constraint can be one of two types: `EncryptionContextEquals` or `EncryptionContextSubset`\. You can use this condition key to check that the request contains one type or the other\.

The following example key policy statement uses the `kms:GrantConstraintType` condition key to allow a user to create grants only when the request includes an `EncryptionContextEquals` grant constraint\. The example shows a policy statement in a key policy\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": "kms:CreateGrant",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:GrantConstraintType": "EncryptionContextEquals"
    }
  }
}
```

**See also**
+ [kms:EncryptionContext:*context\-key*](#conditions-kms-encryption-context)
+ [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys)
+ [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)
+ [kms:GrantOperations](#conditions-kms-grant-operations)
+ [kms:GranteePrincipal](#conditions-kms-grantee-principal)
+ [kms:RetiringPrincipal](#conditions-kms-retiring-principal)

### kms:GrantIsForAWSResource<a name="conditions-kms-grant-is-for-aws-resource"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:GrantIsForAWSResource`  |  Boolean  | Single\-valued |  `CreateGrant` `ListGrants` `RevokeGrant`  |  Key policies and IAM policies  | 

Allows or denies permission for the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html), [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html), or [RevokeGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) operations only when an [AWS services integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration) calls the operation on the user's behalf\. This policy condition doesn't allow the user to call these grant operations directly\.

The following example key policy statement uses the `kms:GrantIsForAWSResource` condition key\. It allows AWS services that are integrated with AWS KMS, such as Amazon EBS, to create grants on this KMS key on behalf of the specified principal\. 

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": "kms:CreateGrant",
  "Resource": "*",
  "Condition": {
    "Bool": {
      "kms:GrantIsForAWSResource": true
    }
  }
}
```

**See also**
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)
+ [kms:GrantOperations](#conditions-kms-grant-operations)
+ [kms:GranteePrincipal](#conditions-kms-grantee-principal)
+ [kms:RetiringPrincipal](#conditions-kms-retiring-principal)

### kms:GrantOperations<a name="conditions-kms-grant-operations"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:GrantOperations`  |  String  | Multi\-valued |  `CreateGrant`  |  Key policies and IAM policies  | 

You can use this condition key to control access to the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation based on the [grant operations](grants.md#terms-grant-operations) in the request\. For example, you can allow a user to create grants that delegate permission to encrypt but not decrypt\. For more information about grants, see [Using grants](grants.md)\.

This is a [multi\-valued condition key](#set-operators)\. `kms:GrantOperations` compares the set of grant operations in the `CreateGrant` request to the set of grant operations in the policy\. To determine how these sets are compared, you must provide a `ForAnyValue` or `ForAllValues` set operator in the policy condition\. For details about the set operators, see [Using multiple keys and values](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_multi-value-conditions.html#reference_policies_multi-key-or-value-conditions) in the IAM User Guide\.
+ `ForAnyValue`: At least one grant operation in the request must match one of the grant operations in the policy condition\. Other grant operations are permitted\.
+ ForAllValues: Every grant operation in the request must match a grant operation in the policy condition\. This set operator limits the grant operations to those specified in the policy condition\. It doesn't require any grant operations, but it forbids unspecified grant operations\.

  ForAllValues also returns true when there are no grant operations in the request, but `CreateGrant` doesn't permit it\. If the `Operations` parameter is missing or has a null value, the `CreateGrant` request fails\.

The following example key policy statement uses the `kms:GrantOperations` condition key to allow a user to create grants only when the grant operations are `Encrypt`, `ReEncryptTo`, or both\. If the grant includes any other operations, the `CreateGrant` request fails\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": "kms:CreateGrant",
  "Resource": "*",
  "Condition": {
    "ForAllValues:StringEquals": {
      "kms:GrantOperations": [
        "Encrypt",
        "ReEncryptTo"
      ]
    }
  }
}
```

If you change the set operator in the policy condition to `ForAnyValue`, the policy statement would require that at least one of the grant operations in the grant is `Encrypt` or `ReEncryptTo`, but it would allow other grant operations, such as `Decrypt` or `ReEncryptFrom`\.

**See also**
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)
+ [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)
+ [kms:GranteePrincipal](#conditions-kms-grantee-principal)
+ [kms:RetiringPrincipal](#conditions-kms-retiring-principal)

### kms:GranteePrincipal<a name="conditions-kms-grantee-principal"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:GranteePrincipal`  |  String  | Single\-valued |  `CreateGrant`  |  IAM and key policies  | 

You can use this condition key to control access to the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation based on the value of the [GranteePrincipal](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html#KMS-CreateGrant-request-GranteePrincipal) parameter in the request\. For example, you can allow a user to create grants to use a KMS key only when the grantee principal in the `CreateGrant` request matches the principal specified in the condition statement\.

The following example key policy statement uses the `kms:GranteePrincipal` condition key to allow a user to create grants for a KMS key only when the grantee principal in the grant is the `LimitedAdminRole`\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": "kms:CreateGrant",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:GranteePrincipal": "arn:aws:iam::111122223333:role/LimitedAdminRole"
    }
  }
}
```

**See also**
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)
+ [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)
+ [kms:GrantOperations](#conditions-kms-grant-operations)
+ [kms:RetiringPrincipal](#conditions-kms-retiring-principal)

### kms:KeyOrigin<a name="conditions-kms-key-origin"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:KeyOrigin`  |  String  | Single\-valued |  `CreateKey` KMS key resource operations  |  IAM policies Key policies and IAM policies  | 

The `kms:KeyOrigin` condition key controls access to operations based on the value of the `Origin` property of the KMS key that is created by or used in the operation\. It works as a resource condition or a request condition\.

You can use this condition key to control access to the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation based on the value of the [Origin](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html#KMS-CreateKey-request-Origin) parameter in the request\. Valid values for `Origin` are `AWS_KMS`, `AWS_CLOUDHSM`, and `EXTERNAL`\. 

For example, you can allow a user to create a KMS key only when the key material is generated in AWS KMS \(`AWS_KMS`\), only when the key material is generated in an AWS CloudHSM cluster that is associated with a [custom key store](custom-key-store-overview.md) \(`AWS_CLOUDHSM`\), or only when the [key material is imported](importing-keys.md) from an external source \(`EXTERNAL`\)\. 

The following example key policy statement uses the `kms:KeyOrigin` condition key to allow a user to create a KMS key only when AWS KMS creates the key material\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": "kms:CreateKey",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:KeyOrigin": "AWS_KMS"
    }
  }
}
```

You can also use the `kms:KeyOrigin` condition key to control access to operations that use or manage a KMS key based on the `Origin` property of the KMS key used for the operation\. The operation must be a *KMS key resource operation*, that is, an operation that is authorized for a particular KMS key\. To identify the KMS key resource operations, in the [Actions and Resources Table](kms-api-permissions-reference.md#kms-api-permissions-reference-table), look for a value of `KMS key` in the `Resources` column for the operation\.

For example, the following IAM policy allows principals to perform the specified KMS key resource operations, but only with KMS keys in the account that were created in a custom key store\.

```
{
  "Effect": "Allow",  
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:GenerateDataKey",
    "kms:GenerateDataKeyWithoutPlaintext",
    "kms:GenerateDataKeyPair",
    "kms:GenerateDataKeyPairWithoutPlaintext",
    "kms:ReEncrypt*"
  ],
  "Resource": "arn:aws:kms:us-west-2:111122223333:key/*",
  "Condition": {
    "StringEquals": {
      "kms:KeyOrigin": "AWS_CLOUDHSM"
    }
  }
}
```

**See also**
+ [kms:BypassPolicyLockoutSafetyCheck](#conditions-kms-bypass-policy-lockout-safety-check)
+ [kms:KeySpec](#conditions-kms-key-spec)
+ [kms:KeyUsage](#conditions-kms-key-usage)

### kms:KeySpec<a name="conditions-kms-key-spec"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:KeySpec`  |  String  |  `CreateKey` KMS key resource operations |  IAM policies Key policies and IAM policies  | 

The `kms:KeySpec` condition key controls access to operations based on the value of the `KeySpec` property of the KMS key that is created by or used in the operation\. 

You can use this condition key in an IAM policy to control access to the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation based on the value of the [KeySpec](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html#KMS-CreateKey-request-KeySpec) parameter in a `CreateKey` request\. For example, you can use this condition to allow users to create only symmetric KMS keys or only KMS keys with RSA keys\.

The following example IAM policy statement uses the `kms:KeySpec` condition key to allow the principals to create a KMS key only when the `KeySpec` in the request is `RSA_4096`\. 

```
{
  "Effect": "Allow",
  "Action": "kms:CreateKey",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:KeySpec": "RSA_4096"
    }
  }
}
```

You can also use the `kms:KeySpec` condition key to control access to operations that use or manage a KMS key based on the `KeySpec` property of the KMS key used for the operation\. The operation must be a *KMS key resource operation*, that is, an operation that is authorized for a particular KMS key\. To identify the KMS key resource operations, in the [Actions and Resources Table](kms-api-permissions-reference.md#kms-api-permissions-reference-table), look for a value of `KMS key` in the `Resources` column for the operation\. 

For example, the following IAM policy allows principals to perform the specified KMS key resource operations, but only with symmetric KMS keys in the account\. 

```
{
  "Effect": "Allow",
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:ReEncrypt*",    
    "kms:DescribeKey"
  ],
  "Resource": {
      "arn:aws:kms:us-west-2:111122223333:key/*"
  },
  "Condition": {
    "StringEquals": {
      "kms:KeySpec": "SYMMETRIC_DEFAULT"
    }
  }
}
```

**See also**
+ [kms:BypassPolicyLockoutSafetyCheck](#conditions-kms-bypass-policy-lockout-safety-check)
+ [kms:CustomerMasterKeySpec \(deprecated\)](#conditions-kms-key-spec-replaced)
+ [kms:DataKeyPairSpec](#conditions-kms-data-key-spec)
+ [kms:KeyOrigin](#conditions-kms-key-origin)
+ [kms:KeyUsage](#conditions-kms-key-usage)

### kms:KeyUsage<a name="conditions-kms-key-usage"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:KeyUsage`  |  String  |  `CreateKey` KMS key resource operations  |  IAM policies Key policies and IAM policies  | 

The `kms:KeyUsage` condition key controls access to operations based on the value of the `KeyUsage` property of the KMS key that is created by or used in the operation\. 

You can use this condition key to control access to the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation based on the value of the [KeyUsage](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html#KMS-CreateKey-request-KeyUsage) parameter in the request\. Valid values for `KeyUsage` are `ENCRYPT_DECRYPT` and `SIGN_VERIFY`\. 

For example, you can allow a user to create a KMS key only when the `KeyUsage` is `ENCRYPT_DECRYPT` or deny a user permission when the `KeyUsage` is `SIGN_VERIFY`\. 

The following example IAM policy statement uses the `kms:KeyUsage` condition key to allow a user to create a KMS key only when the `KeyUsage` is `ENCRYPT_DECRYPT`\.

```
{
  "Effect": "Allow",  
  "Action": "kms:CreateKey",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:KeyUsage": "ENCRYPT_DECRYPT"
    }
  }
}
```

You can also use the `kms:KeyUsage` condition key to control access to operations that use or manage a KMS key based on the `KeyUsage` property of the KMS key in the operation\. The operation must be a *KMS key resource operation*, that is, an operation that is authorized for a particular KMS key\. To identify the KMS key resource operations, in the [Actions and Resources Table](kms-api-permissions-reference.md#kms-api-permissions-reference-table), look for a value of `KMS key` in the `Resources` column for the operation\.

For example, the following IAM policy allows principals to perform the specified KMS key resource operations, but only with KMS keys in the account that are used for signing and verification\.

```
{
  "Effect": "Allow",
  "Action": [
    "kms:CreateGrant",
    "kms:DescribeKey",
    "kms:GetPublicKey",
    "kms:ScheduleKeyDeletion"
  ],
  "Resource": {
      "arn:aws:kms:us-west-2:111122223333:key/*"
  },
  "Condition": {
    "StringEquals": {
      "kms:KeyUsage": "SIGN_VERIFY"
    }
  }
}
```

**See also**
+ [kms:BypassPolicyLockoutSafetyCheck](#conditions-kms-bypass-policy-lockout-safety-check)
+ [kms:CustomerMasterKeyUsage \(deprecated\)](#conditions-kms-key-usage-replaced)
+ [kms:KeyOrigin](#conditions-kms-key-origin)
+ [kms:KeySpec](#conditions-kms-key-spec)

### kms:MessageType<a name="conditions-kms-message-type"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:MessageType`  |  String  | Single\-valued |  `Sign` `Verify`  | Key policies and IAM policies | 

The `kms:MessageType` condition key controls access to the [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html) and [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) operations based on the value of the `MessageType` parameter in the request\. Valid values for `MessageType` are `RAW` and `DIGEST`\. 

For example, the following key policy statement uses the `kms:MessageType` condition key to allow a user to use an asymmetric KMS key to sign a message, but not a message digest\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": "kms:Sign",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:MessageType": "RAW"
    }
  }
}
```

**See also**
+ [kms:SigningAlgorithm](#conditions-kms-signing-algorithm)

### kms:MultiRegion<a name="conditions-kms-multiregion"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:MultiRegion`  |  Boolean  |  `CreateKey` KMS key resource operations  |  Key policies and IAM policies  | 

You can use this condition key to allow operations only on single\-Region keys or only on [multi\-Region keys](multi-region-keys-overview.md)\. The `kms:MultiRegion` condition key controls access to AWS KMS operations on KMS keys and to the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation based on the value of the `MultiRegion` property of the KMS key\. Valid values are `true` \(multi\-Region\), and `false` \(single\-Region\)\. All KMS keys have a `MultiRegion` property\.

For example, the following IAM policy statement uses the `kms:MultiRegion` condition key to allow principals to create only single\-Region keys\. 

```
{
  "Effect": "Allow",
  "Action": "kms:CreateKey",
  "Resource": {
      "*"
  },
  "Condition": {
    "Bool": "kms:MultiRegion": false
  }
}
```

### kms:MultiRegionKeyType<a name="conditions-kms-multiregion-key-type"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:MultiRegionKeyType`  |  String  |  `CreateKey` KMS key resource operations  |  Key policies and IAM policies  | 

You can use this condition key to allow operations only on [multi\-Region primary keys](multi-region-keys-overview.md#mrk-primary-key) or only on [multi\-Region replica keys](multi-region-keys-overview.md#mrk-replica-key)\. The `kms:MultiRegionKeyType` condition key controls access to AWS KMS operations on KMS keys and the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation based on the `MultiRegionKeyType` property of the KMS key\. The valid values are `PRIMARY` and `REPLICA`\. Only multi\-Region keys have a `MultiRegionKeyType` property\.

Typically, you use the `kms:MultiRegionKeyType` condition key in an IAM policy to control access to multiple KMS keys\. However, because a given multi\-Region key can change to primary or replica, you might want to use this condition in a key policy to allow an operation only when the particular multi\-Region key is a primary or replica key\.

For example, the following IAM policy statement uses the `kms:MultiRegionKeyType` condition key to allow principals to schedule and cancel key deletion only on multi\-Region replica keys in the specified AWS account\. 

```
{
  "Effect": "Allow",  
  "Action": [
    "kms:ScheduleKeyDeletion",
    "kms:CancelKeyDeletion"
  ],
  "Resource": {
      "arn:aws:kms:*:111122223333:key/*"
  },
  "Condition": {
    "StringEquals": "kms:MultiRegionKeyType": "REPLICA"
  }
}
```

To allow or deny access to all multi\-Region keys, you can use both values or a null value with `kms:MultiRegionKeyType`\. However, the [kms:MultiRegion](#conditions-kms-multiregion) condition key is recommended for that purpose\.

### kms:PrimaryRegion<a name="conditions-kms-primary-region"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:PrimaryRegion`  |  String \(list\)  |  `UpdatePrimaryRegion`  |  Key policies and IAM policies  | 

You can use this condition key to limit the destination Regions in an [UpdatePrimaryRegion](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdatePrimaryRegion.html) operation\. These are AWS Regions that can host your multi\-Region primary keys\. 

The `kms:PrimaryRegion` condition key controls access to the [UpdatePrimaryRegion](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdatePrimaryRegion.html) operation based on the value of the `PrimaryRegion` parameter\. The `PrimaryRegion` parameter specifies the AWS Region of the [multi\-Region replica key](multi-region-keys-overview.md#mrk-replica-key) that is being promoted to primary\. The value of the condition is one or more AWS Region names, such as `us-east-1` or `ap-southeast-2`, or Region name patterns, such as `eu-*`

For example, the following key policy statement uses the `kms:PrimaryRegion` condition key to allow principals to update the primary region of a multi\-Region key to one of the four specified Regions\.

```
{
  "Effect": "Allow",
  "Action": "kms:UpdatePrimaryRegion",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/Developer"
  },
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:PrimaryRegion": [ 
         "us-east-1",
         "us-west-2",
         "eu-west-3",
         "ap-southeast-2"
      ]
    }
  }
}
```

### kms:ReEncryptOnSameKey<a name="conditions-kms-reencrypt-on-same-key"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:ReEncryptOnSameKey`  |  Boolean  | Single\-valued |  `ReEncrypt`  |  Key policies and IAM policies  | 

You can use this condition key to control access to the [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) operation based on whether the request specifies a destination KMS key that is the same one used for the original encryption\. 

For example, the following key policy statement uses the `kms:ReEncryptOnSameKey` condition key to allow a user to reencrypt only when the destination KMS key is the same one used for the original encryption\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": "kms:ReEncrypt*",
  "Resource": "*",
  "Condition": {
    "Bool": {
      "kms:ReEncryptOnSameKey": true
    }
  }
}
```

### kms:RequestAlias<a name="conditions-kms-request-alias"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:RequestAlias`  |  String \(list\)  | Single\-valued |  [Cryptographic operations](concepts.md#cryptographic-operations) [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html)  |  Key policies and IAM policies  | 

You can use this condition key to allow an operation only when the request uses a particular alias to identify the KMS key\. The `kms:RequestAlias` condition key controls access to a KMS key used in a cryptographic operation, `GetPublicKey`, or `DescribeKey` based on the [alias](kms-alias.md) that identifies that KMS key in the request\. \(This policy condition has no effect on the [GenerateRandom](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateRandom.html) operation because the operation doesn't use a KMS key or alias\.\) 

This condition supports [attribute\-based access control](abac.md) \(ABAC\) in AWS KMS, which lets you control access to KMS keys based on the tags and aliases of a KMS key\. You can use tags and aliases to allow or deny access to a KMS key without changing policies or grants\. For details, see [ABAC for AWS KMS](abac.md)\.

To specify the alias in this policy condition, use an [alias name](concepts.md#key-id-alias-name), such as `alias/project-alpha`, or an alias name pattern, such as `alias/*test*`\. You cannot specify an [alias ARN](concepts.md#key-id-alias-ARN) in the value of this condition key\.

To satisfy this condition, the value of the `KeyId` parameter in the request must be a matching alias name or alias ARN\. If the request uses a different [key identifier](concepts.md#key-id), it does not satisfy the condition, even if identifies the same KMS key\.

For example, the following key policy statement allows the principal to call the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation on the KMS key\. However this is permitted only when the value of the `KeyId` parameter in the request is `alias/finance-key` or an alias ARN with that alias name, such as `arn:aws:kms:us-west-2:111122223333:alias/finance-key`\.

```
{
  "Sid": "Key policy using a request alias condition",
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/developer"
  },
  "Action": "kms:GenerateDataKey",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:RequestAlias": "alias/finance-key"
    }
  }
}
```

You cannot use this condition key to control access to alias operations, such as [CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html) or [DeleteAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteAlias.html)\. For information about controlling access to alias operations, see [Controlling access to aliases](alias-access.md)\.

### kms:ResourceAliases<a name="conditions-kms-resource-aliases"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:ResourceAliases`  |  String \(list\)  | Multi\-valued | KMS key resource operations |  IAM policies only  | 

Use this condition key to control access to a KMS key based on the [aliases](kms-alias.md) that are associated with the KMS key\. The operation must be a *KMS key resource operation*, that is, an operation that is authorized for a particular KMS key\. To identify the KMS key resource operations, in the [Actions and Resources Table](kms-api-permissions-reference.md#kms-api-permissions-reference-table), look for a value of `KMS key` in the `Resources` column for the operation\.

This condition supports attribute\-based access control \(ABAC\) in AWS KMS\. With ABAC, you can control access to KMS keys based on the tags that are assigned to a KMS key and the aliases that are associated with a KMS key\. You can use tags and aliases to allow or deny access to a KMS key without changing policies or grants\. For details, see [ABAC for AWS KMS](abac.md)\.

An alias must be unique in an AWS account and Region, but this condition lets you control access to multiple KMS keys in the same Region \(using the `StringLike` comparison operator\) or to multiple KMS keys in different AWS Regions of each account\.

**Note**  
The [kms:ResourceAliases](#conditions-kms-resource-aliases) condition is effective only when the KMS key conforms to the [aliases per KMS key](resource-limits.md#aliases-per-key) quota\. If a KMS key exceeds this quota, principals who are authorized to use the KMS key by the `kms:ResourceAliases` condition are denied access to the KMS key\.

To specify the alias in this policy condition, use an [alias name](concepts.md#key-id-alias-name), such as `alias/project-alpha`, or an alias name pattern, such as `alias/*test*`\. You cannot specify an [alias ARN](concepts.md#key-id-alias-ARN) in the value of this condition key\. To satisfy the condition, the KMS key used in the operation must have the specified alias\. It does not matter whether or how the KMS key is identified in the request for the operation\.

This is a multivalued condition key that compares the set of aliases associated with a KMS key to the set of aliases in the policy\. To determine how these sets are compared, you must provide a `ForAnyValue` or `ForAllValues` set operator in the policy condition\. For details about the set operators, see [Using multiple keys and values](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_multi-value-conditions.html#reference_policies_multi-key-or-value-conditions) in the IAM User Guide\.
+ ForAnyValue: At least one alias associated with the KMS key must match an alias in the policy condition\. Other aliases are permitted\. If the KMS key has no aliases, the condition is not satisfied\.
+ ForAllValues: Every alias associated with the KMS key must match an alias in the policy\. This set operator limits the aliases associated with the KMS key to those in the policy condition\. It doesn't require any aliases, but it forbids unspecified aliases\.

For example, the following IAM policy statement allows the principal to call the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation on any KMS key in the specified AWS account that is associated with the `finance-key` alias\. \(The key policies of the affected KMS keys must also allow the principal's account to use them for this operation\.\) To indicate that the condition is satisfied when one of the many aliases that might be associated with the KMS key is `alias/finance-key`, the condition uses the `ForAnyValue` set operator\. 

Because the `kms:ResourceAliases` condition is based on the resource, not the request, a call to `GenerateDataKey` succeeds for any KMS key associated with the `finance-key` alias, even if the request uses a [key ID](concepts.md#key-id-key-id) or [key ARN](concepts.md#key-id-key-ARN) to identify the KMS key\. 

```
{
  "Sid": "AliasBasedIAMPolicy",
  "Effect": "Allow",
  "Action": "kms:GenerateDataKey",
  "Resource": [
    "arn:aws:kms:*:111122223333:key/*",
    "arn:aws:kms:*:444455556666:key/*",
  ],
  "Condition": {
    "ForAnyValue:StringEquals": {
      "kms:ResourceAliases": "alias/finance-key"
    }
  }
}
```

The following example IAM policy statement allows the principal to enable and disable KMS keys but only when all aliases of the KMS keys include "`Test`\." This policy statement uses two conditions\. The condition with the `ForAllValues` set operator requires that all aliases associated with the KMS key include "Test"\. The condition with the `ForAnyValue` set operator requires that the KMS key have at least one alias with "Test\." Without the `ForAnyValue` condition, this policy statement would have allowed the principal to use KMS keys that had no aliases\.

```
{
  "Sid": "AliasBasedIAMPolicy",
  "Effect": "Allow",
  "Action": [
    "kms:EnableKey",
    "kms:DisableKey"
  ],
  "Resource": "arn:aws:kms:*:111122223333:key/*",
  "Condition": {
    "ForAllValues:StringLike": {
      "kms:ResourceAliases": [
        "alias/*Test*"
      ]
    },
    "ForAnyValue:StringLike": {
      "kms:ResourceAliases": [
        "alias/*Test*"
      ]
    }
  }
}
```

### kms:ReplicaRegion<a name="conditions-kms-replica-region"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:ReplicaRegion`  |  String \(list\)  |  `ReplicateKey`  |  Key policies and IAM policies  | 

You can use this condition key to limit the AWS Regions in which a principal can replicate a [multi\-Region key](multi-region-keys-overview.md)\. The `kms:ReplicaRegion` condition key controls access to the [ReplicateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation based on the value of the [ReplicaRegion](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html#KMS-CreateGrant-request-RetiringPrincipal) parameter in the request\. This parameter specifies the AWS Region for the new [replica key](multi-region-keys-overview.md#mrk-replica-key)\. 

The value of the condition is one or more AWS Region names, such as `us-east-1` or `ap-southeast-2`, or name patterns, such as `eu-*`\. For a list of the names of AWS Regions that AWS KMS supports, see [AWS Key Management Service endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/kms.html) in the AWS General Reference\.

For example, the following key policy statement uses the `kms:ReplicaRegion` condition key to allow principals to call the [ReplicateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReplicateKey.html) operation only when the value of the `ReplicaRegion` parameter is one of the specified Regions\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/Administrator"
  },
  "Action": "kms:ReplicateKey"
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:ReplicaRegion": { 
         "us-east-1",
         "eu-west-3",
         "ap-southeast-2"
      }
    }
  }
}
```

This condition key controls access only to the [ReplicateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReplicateKey.html) operation\. To control access to the [UpdatePrimaryRegion](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdatePrimaryRegion.html) operation, use the [kms:PrimaryRegion](#conditions-kms-primary-region) condition key\.

### kms:RetiringPrincipal<a name="conditions-kms-retiring-principal"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:RetiringPrincipal`  |  String \(list\)  | Single\-valued |  `CreateGrant`  |  Key policies and IAM policies  | 

You can use this condition key to control access to the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation based on the value of the [RetiringPrincipal](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html#KMS-CreateGrant-request-RetiringPrincipal) parameter in the request\. For example, you can allow a user to create grants to use a KMS key only when the `RetiringPrincipal` in the `CreateGrant` request matches the `RetiringPrincipal` in the condition statement\.

The following example key policy statement allows a user to create grants for the \.KMS key The `kms:RetiringPrincipal` condition key restricts the permission to `CreateGrant` requests where the retiring principal in the grant is either the `LimitedAdminRole` or the `OpsAdmin` user\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": "kms:CreateGrant",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:RetiringPrincipal": [ 
         "arn:aws:iam::111122223333:role/LimitedAdminRole",
         "arn:aws:iam::111122223333:user/OpsAdmin"
      ]
    }
  }
}
```

**See also**
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)
+ [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)
+ [kms:GrantOperations](#conditions-kms-grant-operations)
+ [kms:GranteePrincipal](#conditions-kms-grantee-principal)

### kms:SigningAlgorithm<a name="conditions-kms-signing-algorithm"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:SigningAlgorithm`  |  String  | Single\-valued |  `Sign`  `Verify`  |  Key policies and IAM policies  | 

You can use the `kms:SigningAlgorithm` condition key to control access to the [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html) and [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) operations based on the value of the [SigningAlgorithm](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html#KMS-Sign-request-SigningAlgorithm) parameter in the request\. This condition key has no effect on operations performed outside of AWS KMS, such as verifying signatures with the public key in an asymmetric KMS key pair outside of AWS KMS\.

The following example key policy allows users who can assume the `testers` role to use the KMS key to sign messages only when the signing algorithm used for the request is an RSASSA\_PSS algorithm, such as `RSASSA_PSS_SHA512`\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/testers"
  },
  "Action": "kms:Sign",
  "Resource": "*",
  "Condition": {
    "StringLike": {
      "kms:SigningAlgorithm": "RSASSA_PSS*"
    }
  }
}
```

**See also**
+ [kms:EncryptionAlgorithm](#conditions-kms-encryption-algorithm)
+ [kms:MessageType](#conditions-kms-message-type)

### kms:ValidTo<a name="conditions-kms-valid-to"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:ValidTo`  |  Timestamp  | Single\-valued |  `ImportKeyMaterial`  |  Key policies and IAM policies  | 

The `kms:ValidTo` condition key controls access to the [ImportKeyMaterial](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html) operation based on the value of the [ValidTo](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html#KMS-ImportKeyMaterial-request-ValidTo) parameter in the request, which determines when the imported key material expires\. The value is expressed in [Unix time](https://en.wikipedia.org/wiki/Unix_time)\.

By default, the `ValidTo` parameter is required in an `ImportKeyMaterial` request\. However, if the value of the [ExpirationModel](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html#KMS-ImportKeyMaterial-request-ExpirationModel) parameter is `KEY_MATERIAL_DOES_NOT_EXPIRE`, the `ValidTo` parameter is invalid\. You can also use the [kms:ExpirationModel](#conditions-kms-expiration-model) condition key to require the `ExpirationModel` parameter or a specific parameter value\.

The following example policy statement allows a user to import key material into a KMS key\. The `kms:ValidTo` condition key limits the permission to `ImportKeyMaterial` requests where the `ValidTo` value is less than or equal to `1546257599.0` \(December 31, 2018 11:59:59 PM\)\. 

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": "kms:ImportKeyMaterial",
  "Resource": "*",
  "Condition": {
    "NumericLessThanEquals": {
      "kms:ValidTo": "1546257599.0"
    }
  }
}
```

**See also**
+ [kms:ExpirationModel](#conditions-kms-expiration-model) 
+ [kms:WrappingAlgorithm](#conditions-kms-wrapping-algorithm)
+ [kms:WrappingKeySpec](#conditions-kms-wrapping-key-spec)

### kms:ViaService<a name="conditions-kms-via-service"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:ViaService`  |  String  | Single\-valued |  KMS key resource operations  |  Key policies and IAM policies  | 

The `kms:ViaService` condition key limits use of an AWS KMS [AWS KMS key](concepts.md#kms_keys) \(KMS key\) to requests from specified AWS services\. You can specify one or more services in each `kms:ViaService` condition key\. The operation must be a *KMS key resource operation*, that is, an operation that is authorized for a particular KMS key\. To identify the KMS key resource operations, in the [Actions and Resources Table](kms-api-permissions-reference.md#kms-api-permissions-reference-table), look for a value of `KMS key` in the `Resources` column for the operation\.

For example, the following key policy statement uses the `kms:ViaService` condition key to allow a [customer managed key](concepts.md#customer-cmk) to be used for the specified actions only when the request comes from Amazon EC2 or Amazon RDS in the US West \(Oregon\) region on behalf of `ExampleUser`\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:ReEncrypt*",
    "kms:GenerateDataKey*",
    "kms:CreateGrant",
    "kms:ListGrants",
    "kms:DescribeKey"
  ],
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:ViaService": [
        "ec2.us-west-2.amazonaws.com",
        "rds.us-west-2.amazonaws.com"
      ]
    }
  }
}
```

You can also use a `kms:ViaService` condition key to deny permission to use a KMS key when the request comes from particular services\. For example, the following policy statement from a key policy uses a `kms:ViaService` condition key to prevent a customer managed key from being used for `Encrypt` operations when the request comes from AWS Lambda on behalf of `ExampleUser`\.

```
{
  "Effect": "Deny",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": [
    "kms:Encrypt"    
  ],
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:ViaService": [
          "lambda.us-west-2.amazonaws.com"
      ]
    }
  }
}
```

**Important**  
When you use the `kms:ViaService` condition key, the service makes the request on behalf of a principal in the AWS account\. These principals must have the following permissions:  
Permission to use the KMS key\. The principal needs to grant these permissions to the integrated service so the service can use the customer managed key on behalf of the principal\. For more information, see [How AWS services use AWS KMS](service-integration.md)\.
Permission to use the integrated service\. For details about giving users access to an AWS service that integrates with AWS KMS, consult the documentation for the integrated service\.

All [AWS managed keys](concepts.md#aws-managed-cmk) use a `kms:ViaService` condition key in their key policy document\. This condition allows the KMS key to be used only for requests that come from the service that created the KMS key\. To see the key policy for an AWS managed key, use the [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation\. 

The `kms:ViaService` condition key is valid in IAM and key policy statements\. The services that you specify must be [integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration) and support the `kms:ViaService` condition key\.

#### Services that support the `kms:ViaService` condition key<a name="viaService_table"></a>

The following table lists AWS services that are integrated with AWS KMS and support the use of the `kms:ViaService` condition key in customer managed keys The services in this table might not be available in all regions\. Use the `.amazonaws.com` suffix of the AWS KMS ViaService name in all AWS partitions\.

**Note**  
You might need to scroll horizontally or vertically to see all of the data in this table\.


| Service name | AWS KMS ViaService name | 
| --- | --- | 
| AWS App Runner | apprunner\.AWS\_region\.amazonaws\.com | 
| Amazon AppFlow | appflow\.AWS\_region\.amazonaws\.com | 
| Amazon Application Migration Service | mgn\.AWS\_region\.amazonaws\.com | 
| Amazon Athena | athena\.AWS\_region\.amazonaws\.com | 
| AWS Audit Manager | auditmanager\.AWS\_region\.amazonaws\.com | 
| Amazon Aurora | rds\.AWS\_region\.amazonaws\.com | 
| AWS Backup | backup\.AWS\_region\.amazonaws\.com | 
| AWS Backup Gateway | backup\-gateway\.AWS\_region\.amazonaws\.com | 
| AWS CodeArtifact | codeartifact\.AWS\_region\.amazonaws\.com | 
| Amazon CodeGuru Reviewer | codeguru\-reviewer\.AWS\_region\.amazonaws\.com | 
| Amazon Comprehend | comprehend\.AWS\_region\.amazonaws\.com | 
| Amazon Connect | connect\.AWS\_region\.amazonaws\.com | 
| Amazon Connect Customer Profiles | profile\.AWS\_region\.amazonaws\.com | 
| Amazon Connect Wisdom | wisdom\.AWS\_region\.amazonaws\.com | 
| AWS Database Migration Service \(AWS DMS\) | dms\.AWS\_region\.amazonaws\.com | 
| AWS Directory Service | directoryservice\.AWS\_region\.amazonaws\.com | 
| Amazon DynamoDB | dynamodb\.AWS\_region\.amazonaws\.com | 
| Amazon EC2 Systems Manager \(SSM\) | ssm\.AWS\_region\.amazonaws\.com | 
| Amazon Elastic Block Store \(Amazon EBS\) | ec2\.AWS\_region\.amazonaws\.com \(EBS only\) | 
| Amazon Elastic Container Registry \(Amazon ECR\) | ecr\.AWS\_region\.amazonaws\.com | 
| Amazon Elastic File System \(Amazon EFS\) | elasticfilesystem\.AWS\_region\.amazonaws\.com | 
| Amazon Elastic Kubernetes Service \(Amazon EKS\) | eks\.AWS\_region\.amazonaws\.com | 
| Amazon ElastiCache |  Include both ViaService names in the condition key value: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/policy-conditions.html)  | 
| Amazon OpenSearch Service \(OpenSearch Service\) | es\.AWS\_region\.amazonaws\.com | 
| AWS Elemental MediaTailor | mediatailor\.AWS\_region\.amazonaws\.com | 
| Amazon FinSpace | finspace\.AWS\_region\.amazonaws\.com | 
| Amazon Forecast | forecast\.AWS\_region\.amazonaws\.com | 
| Amazon FSx | fsx\.AWS\_region\.amazonaws\.com | 
| AWS Glue | glue\.AWS\_region\.amazonaws\.com | 
| Amazon HealthLake | healthlake\.AWS\_region\.amazonaws\.com | 
| AWS IoT SiteWise | iotsitewise\.AWS\_region\.amazonaws\.com | 
| Amazon Kendra | kendra\.AWS\_region\.amazonaws\.com | 
| Amazon Keyspaces \(for Apache Cassandra\) | cassandra\.AWS\_region\.amazonaws\.com | 
| Amazon Kinesis | kinesis\.AWS\_region\.amazonaws\.com | 
| Amazon Kinesis Data Firehose | firehose\.AWS\_region\.amazonaws\.com | 
| Amazon Kinesis Video Streams | kinesisvideo\.AWS\_region\.amazonaws\.com | 
| AWS Lambda | lambda\.AWS\_region\.amazonaws\.com | 
| Amazon Lex | lex\.AWS\_region\.amazonaws\.com | 
| AWS License Manager | license\-manager\.AWS\_region\.amazonaws\.com | 
| Amazon Location Service | geo\.AWS\_region\.amazonaws\.com | 
| Amazon Lookout for Equipment | lookoutequipment\.AWS\_region\.amazonaws\.com | 
| Amazon Lookout for Metrics | lookoutmetrics\.AWS\_region\.amazonaws\.com | 
| Amazon Lookout for Vision | lookoutvision\.AWS\_region\.amazonaws\.com | 
| Amazon Managed Blockchain | managedblockchain\.AWS\_region\.amazonaws\.com | 
| Amazon Managed Streaming for Apache Kafka \(Amazon MSK\) | kafka\.AWS\_region\.amazonaws\.com | 
| Amazon Managed Workflows for Apache Airflow \(MWAA\) | airflow\.AWS\_region\.amazonaws\.com | 
| Amazon MemoryDB | memorydb\.AWS\_region\.amazonaws\.com | 
| Amazon Monitron | monitron\.AWS\_region\.amazonaws\.com | 
| Amazon MQ | mq\.AWS\_region\.amazonaws\.com | 
| Amazon Neptune | rds\.AWS\_region\.amazonaws\.com | 
| Amazon Nimble Studio | nimble\.AWS\_region\.amazonaws\.com | 
| AWS Proton | proton\.AWS\_region\.amazonaws\.com | 
| Amazon Quantum Ledger Database \(Amazon QLDB\) | qldb\.AWS\_region\.amazonaws\.com | 
| Amazon RDS Performance Insights | rds\.AWS\_region\.amazonaws\.com | 
| Amazon Redshift | redshift\.AWS\_region\.amazonaws\.com | 
| Amazon Redshift query editor V2 | sqlworkbench\.AWS\_region\.amazonaws\.com | 
| Amazon Rekognition | rekognition\.AWS\_region\.amazonaws\.com | 
| Amazon Relational Database Service \(Amazon RDS\) | rds\.AWS\_region\.amazonaws\.com | 
| AWS Secrets Manager | secretsmanager\.AWS\_region\.amazonaws\.com | 
| Amazon Simple Email Service \(Amazon SES\) | ses\.AWS\_region\.amazonaws\.com | 
| Amazon Simple Notification Service \(Amazon SNS\) | sns\.AWS\_region\.amazonaws\.com | 
| Amazon Simple Queue Service \(Amazon SQS\) | sqs\.AWS\_region\.amazonaws\.com | 
| Amazon Simple Storage Service \(Amazon S3\) | s3\.AWS\_region\.amazonaws\.com | 
| AWS Snowball | importexport\.AWS\_region\.amazonaws\.com | 
| AWS Storage Gateway | storagegateway\.AWS\_region\.amazonaws\.com | 
| AWS Systems Manager Incident Manager | ssm\-incidents\.AWS\_region\.amazonaws\.com | 
| AWS Systems Manager Incident Manager Contacts | ssm\-contacts\.AWS\_region\.amazonaws\.com | 
| Amazon Timestream | timestream\.AWS\_region\.amazonaws\.com | 
| Amazon WorkMail | workmail\.AWS\_region\.amazonaws\.com | 
| Amazon WorkSpaces | workspaces\.AWS\_region\.amazonaws\.com | 
| AWS X\-Ray | xray\.AWS\_region\.amazonaws\.com | 

### kms:WrappingAlgorithm<a name="conditions-kms-wrapping-algorithm"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:WrappingAlgorithm`  |  String  | Single\-valued |  `GetParametersForImport`  |  Key policies and IAM policies  | 

This condition key controls access to the [GetParametersForImport](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetParametersForImport.html) operation based on the value of the [WrappingAlgorithm](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetParametersForImport.html#KMS-GetParametersForImport-request-WrappingAlgorithm) parameter in the request\. You can use this condition to require principals to use a particular algorithm to encrypt key material during the import process\. Requests for the required public key and import token fail when they specify a different wrapping algorithm\.

The following example key policy statement uses the `kms:WrappingAlgorithm` condition key to give the example user permission to call the `GetParametersForImport` operation, but prevents them from using the `RSAES_OAEP_SHA_1` wrapping algorithm\. When the `WrappingAlgorithm` in the `GetParametersForImport` request is `RSAES_OAEP_SHA_1`, the operation fails\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": "kms:GetParametersForImport",
  "Resource": "*",
  "Condition": {
    "StringNotEquals": {
      "kms:WrappingAlgorithm": "RSAES_OAEP_SHA_1"
    }
  }
}
```

**See also**
+ [kms:ExpirationModel](#conditions-kms-expiration-model)
+ [kms:ValidTo](#conditions-kms-valid-to)
+ [kms:WrappingKeySpec](#conditions-kms-wrapping-key-spec)

### kms:WrappingKeySpec<a name="conditions-kms-wrapping-key-spec"></a>


| AWS KMS condition keys | Condition type | Value type | API operations | Policy type | 
| --- | --- | --- | --- | --- | 
|  `kms:WrappingKeySpec`  |  String  | Single\-valued |  `GetParametersForImport`  |  Key policies and IAM policies  | 

This condition key controls access to the [GetParametersForImport](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetParametersForImport.html) operation based on the value of the [WrappingKeySpec](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetParametersForImport.html#KMS-GetParametersForImport-request-WrappingKeySpec) parameter in the request\. You can use this condition to require principals to use a particular type of public key during the import process\. If the request specifies a different key type, it fails\.

Because the only valid value for the `WrappingKeySpec` parameter value is `RSA_2048`, preventing users from using this value effectively prevents them from using the `GetParametersForImport` operation\. 

The following example policy statement uses the `kms:WrappingAlgorithm` condition key to require that the `WrappingKeySpec` in the request is `RSA_2048`\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": "kms:GetParametersForImport",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:WrappingKeySpec": "RSA_2048"
    }
  }
}
```

**See also**
+ [kms:ExpirationModel](#conditions-kms-expiration-model)
+ [kms:ValidTo](#conditions-kms-valid-to)
+ [kms:WrappingAlgorithm](#conditions-kms-wrapping-algorithm)

## AWS KMS condition keys for AWS Nitro Enclaves<a name="conditions-nitro-enclaves"></a>

[AWS Nitro Enclaves](https://docs.aws.amazon.com/enclaves/latest/user/) is an Amazon EC2 capability that lets you create isolated compute environments called [enclaves](https://docs.aws.amazon.com/enclaves/latest/user/nitro-enclave-concepts.html#term-enclave) to protect and process highly sensitive data\. AWS KMS provides condition keys to support AWS Nitro Enclaves\. These conditions keys work only when a request for an AWS KMS operation originates in an enclave\. 

When you call the `kms-decrypt`, `kms-generate-data-key`, or `kms-generate-random` [AWS Nitro Enclaves SDK](https://github.com/aws/aws-nitro-enclaves-sdk-c) APIs from an enclave, these APIs call the corresponding AWS KMS operation with a parameter that includes a signed [attestation document](https://docs.aws.amazon.com/enclaves/latest/user/nitro-enclave-concepts.html#term-attestdoc) from the enclave\. The signed attestation document proves the enclave's identity to AWS KMS\. 

The following condition keys let you limit the permissions for these operations based on the contents of the signed attestation document\. Before allowing an operation, AWS KMS compares the attestation document from the enclave to the values in these AWS KMS condition keys\.

### kms:RecipientAttestation:ImageSha384<a name="conditions-kms-recipient-image-sha"></a>


| AWS KMS Condition Keys | Condition Type | Value type | API Operations | Policy Type | 
| --- | --- | --- | --- | --- | 
|  `kms:RecipientAttestation:ImageSha384`  |  String  | Single\-valued |  `Decrypt` `GenerateDataKey` `GenerateRandom`  |  Key policies and IAM policies  | 

The `kms:RecipientAttestation:ImageSha384` condition key allows `kms-decrypt`, `kms-generate-data-key`, and `kms-generate-random` requests from an enclave only when the image hash from the signed attestation document in the request matches the value in the condition key\. The `ImageSha384` value corresponds to PCR\[0\] in the attestation document\. This condition key is effective only when you call the AWS Nitro Enclaves SDK APIs from an enclave\.

**Note**  
This condition key is valid in key policy statements and IAM policy statements even though it does not appear in the IAM console or the IAM *Service Authorization Reference*\.

For example, the following key policy statement allows the `data-processing` role to use the KMS key for the `kms-decrypt` \([Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)\), `kms-generate-data-key` \([GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)\), and `kms-generate-random` \([GenerateRandom](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateRandom.html)\) operations\. The `kms:RecipientAttestation:ImageSha384` condition key allows the operations only when the image hash value \(PCR\[0\]\) of the attestation document in the request matches the image hash value in the condition\. 

If the request doesn't include any attestation document, permission is denied because this condition isn't satisfied\.

```
{
  "Sid" : "Enable enclave data processing",
  "Effect" : "Allow",
  "Principal" : {
    "AWS" : "arn:aws:iam::111122223333:role/data-processing"
  },
  "Action": [
    "kms:Decrypt",
    "kms:GenerateDataKey",
    "kms:GenerateRandom"
  ],
  "Resource" : "*",
  "Condition": {
    "StringEqualsIgnoreCase": {
      "kms:RecipientAttestation:ImageSha384": "9fedcba8abcdef7abcdef6abcdef5abcdef4abcdef3abcdef2abcdef1abcdef1abcdef0abcdef1abcdef2abcdef3abcdef4abcdef5abcdef6abcdef7abcdef99"
    }
  }
}
```

### kms:RecipientAttestation:PCR<PCR\_ID><a name="conditions-kms-recipient-pcrs"></a>


| AWS KMS Condition Keys | Condition Type | Value type | API Operations | Policy Type | 
| --- | --- | --- | --- | --- | 
|  `kms:RecipientAttestation:PCR`  |  String  | Single\-valued |  `Decrypt` `GenerateDataKey` `GenerateRandom`  |  Key policies and IAM policies  | 

The `kms:RecipientAttestation:PCR<PCR_ID>` condition key allows `kms-decrypt`, `kms-generate-data-key`, and `kms-generate-random` requests from an enclave only when the platform configuration registers \(PCRs\) from the signed attestation document in the request match the PCRs in the condition key\. This condition key is effective only when you call the AWS Nitro Enclaves SDK APIs from an enclave\.

**Note**  
This condition key is valid in key policy statements and IAM policy statements even though it does not appear in the IAM console or the IAM *Service Authorization Reference*\.

To specify a PCR value, use the following format\. Concatenate the PCR ID to the condition key name\. The PCR value must be a lower\-case hexadecimal string of up to 96 bytes\.

```
"kms:RecipientAttestation:PCRPCR_ID": "PCR_value"
```

For example, the following condition key specifies a particular value for PCR\[1\], which corresponds to the hash of the kernel used for the enclave and the bootstrap process\.

```
kms:RecipientAttestation:PCR1: "0x1abcdef2abcdef3abcdef4abcdef5abcdef6abcdef7abcdef8abcdef9abcdef8abcdef7abcdef6abcdef5abcdef4abcdef3abcdef2abcdef1abcdef0abcde"
```

The following example key policy statement allows the `data-processing` role to use the KMS key for the `kms-decrypt` \([Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)\) operation\.

The `kms:RecipientAttestation:PCR` condition key in this statement allows the operation only when the PCR1 value in the signed attestation document in the request matches `kms:RecipientAttestation:PCR1` value in the condition\. Use the `StringEqualsIgnoreCase` policy operator to require a case\-insensitive comparison of the PCR values\.

If the request doesn't include an attestation document, permission is denied because this condition isn't satisfied\.

```
{
  "Sid" : "Enable enclave data processing",
  "Effect" : "Allow",
  "Principal" : {
    "AWS" : "arn:aws:iam::111122223333:role/data-processing"
  },
  "Action": "kms:Decrypt",
  "Resource" : "*",
  "Condition": {
    "StringEqualsIgnoreCase": {
      "kms:RecipientAttestation:PCR1": "0x1de4f2dcf774f6e3b679f62e5f120065b2e408dcea327bd1c9dddaea6664e7af7935581474844767453082c6f1586116376cede396a30a39a611b9aad7966c87"
    }
  }
}
```
