# Using Policy Conditions with AWS KMS<a name="policy-conditions"></a>

You can specify conditions in the [key policies](key-policies.md) and [IAM policies](iam-policies.md) that control access to AWS KMS resources\. The policy statement is effective only when the conditions are true\. For example, you might want a policy statement to take effect only after a specific date\. Or, you might want a policy statement to control access only when a specific value appears in an API request\.

To specify conditions, you use predefined *condition keys* in the `Condition` element of a policy statement with [IAM condition policy operators](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html)\. Some condition keys apply generally to AWS; others are specific to AWS KMS\.

**Topics**
+ [AWS Global Condition Keys](#conditions-aws)
+ [AWS KMS Condition Keys](#conditions-kms)

## AWS Global Condition Keys<a name="conditions-aws"></a>

AWS provides [global condition keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#AvailableKeys), a set of predefined condition keys for all AWS services that use IAM for access control\. For example, you can use the `aws:PrincipalType` condition key to allow access only when the principal in the request is the type you specify\.

AWS KMS supports all global condition keys, including the `aws:TagKeys` and `aws:RequestTag` condition keys that control access based on the resource tag in the request\. This condition key is supported by some, but not all, AWS services\.

**Topics**
+ [Using the IP Address Condition in Policies with AWS KMS Permissions](#conditions-aws-ip-address)
+ [Using VPC Endpoint Conditions in Policies with AWS KMS Permissions](#conditions-aws-vpce)

### Using the IP Address Condition in Policies with AWS KMS Permissions<a name="conditions-aws-ip-address"></a>

You can use AWS KMS to protect your data in an [integrated AWS service](service-integration.md)\. But use caution when specifying the [IP address condition operators](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_IPAddress) or the `aws:SourceIp` condition key in the same policy statement that allows or denies access to AWS KMS\. For example, the policy in [AWS: Denies Access to AWS Based on the Source IP](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_aws_deny-ip.html) restricts AWS actions to requests from the specified IP range\.

Consider this scenario:

1. You attach a policy like the one shown at [AWS: Denies Access to AWS Based on the Source IP](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_aws_deny-ip.html) to an IAM user\. You set the value of the `aws:SourceIp` condition key to the range of IP addresses for the user's company\. This IAM user has other policies attached that allow it to use Amazon EBS, Amazon EC2, and AWS KMS\.

1. The user attempts to attach an encrypted EBS volume to an EC2 instance\. This action fails with an authorization error even though the user has permission to use all the relevant services\.

Step 2 fails because the request to AWS KMS to decrypt the volume's encrypted data key comes from an IP address that is associated with the Amazon EC2 infrastructure\. To succeed, the request must come from the IP address of the originating user\. Because the policy in step 1 explicitly denies all requests from IP addresses other than those specified, Amazon EC2 is denied permission to decrypt the EBS volume's encrypted data key\.

Also, the `aws:sourceIP` condition key is not effective when the request comes from an [Amazon VPC endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html)\. To restrict requests to a VPC endpoint, including an [AWS KMS VPC endpoint](kms-vpc-endpoint.md), use the `aws:sourceVpce` or `aws:sourceVpc` condition keys\. For more information, see [VPC Endpoints \- Controlling the Use of Endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html#vpc-endpoints-iam-access) in the *Amazon VPC User Guide*\. 

### Using VPC Endpoint Conditions in Policies with AWS KMS Permissions<a name="conditions-aws-vpce"></a>

[AWS KMS supports Amazon Virtual Private Cloud \(Amazon VPC\) endpoints](kms-vpc-endpoint.md) that are powered by [AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Introduction.html#what-is-privatelink)\. You can use the following [global condition keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#AvailableKeys) in IAM policies to allow or deny access to a particular VPC or VPC endpoint\. You can also use these global condition keys in [AWS KMS key policies](kms-vpc-endpoint.md#vpce-policy) to restrict access to AWS KMS CMKs to requests from the VPC or VPC endpoint\. 
+ `aws:SourceVpc` limits access to requests from the specified VPC\. 
+ `aws:SourceVpce` limits access to requests from the specified VPC endpoint\. 

If you use these condition keys in a key policy statement that allows or denies access to AWS KMS CMKs, you might inadvertently deny access to services that use AWS KMS on your behalf\. 

Take care to avoid a situation like the [IP address condition keys](#conditions-aws-ip-address) example\. If you restrict requests for a CMK to a VPC or VPC endpoint, calls to AWS KMS from an integrated service, such as Amazon S3 or Amazon EBS, might fail\. This can happen even if the source request ultimately originates in the VPC or from the VPC endpoint\. 

## AWS KMS Condition Keys<a name="conditions-kms"></a>

AWS KMS provides an additional set of predefined condition keys that you can use in key policies and IAM policies\. These condition keys are specific to AWS KMS\. For example, you can use the `kms:EncryptionContext` condition key to require a particular [encryption context](encryption-context.md) when controlling access to a KMS customer master key \(CMK\)\.

The following topics describe each AWS KMS condition key and include example policy statements that demonstrate policy syntax\.

**Topics**
+ [kms:BypassPolicyLockoutSafetyCheck](#conditions-kms-bypass-policy-lockout-safety-check)
+ [kms:CallerAccount](#conditions-kms-caller-account)
+ [kms:EncryptionContext:](#conditions-kms-encryption-context)
+ [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys)
+ [kms:ExpirationModel](#conditions-kms-expiration-model)
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)
+ [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)
+ [kms:GrantOperations](#conditions-kms-grant-operations)
+ [kms:GranteePrincipal](#conditions-kms-grantee-principal)
+ [kms:KeyOrigin](#conditions-kms-key-origin)
+ [kms:ReEncryptOnSameKey](#conditions-kms-reencrypt-on-same-key)
+ [kms:RetiringPrincipal](#conditions-kms-retiring-principal)
+ [kms:ValidTo](#conditions-kms-valid-to)
+ [kms:ViaService](#conditions-kms-via-service)
+ [kms:WrappingAlgorithm](#conditions-kms-wrapping-algorithm)
+ [kms:WrappingKeySpec](#conditions-kms-wrapping-key-spec)

### kms:BypassPolicyLockoutSafetyCheck<a name="conditions-kms-bypass-policy-lockout-safety-check"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:BypassPolicyLockoutSafetyCheck`  |  Boolean  |  `CreateKey` `PutKeyPolicy`  |  CreateKey: IAM policies only PutKeyPolicy: IAM and key policies  | 

The `kms:BypassPolicyLockoutSafetyCheck` condition key controls access to the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) and [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operations based on the value of the `BypassPolicyLockoutSafetyCheck` parameter in the request\. 

The following example IAM policy statement prevents users from bypassing the policy lockout safety check by denying them permission to create CMKs when the value of the `BypassPolicyLockoutSafetyCheck` parameter in the `CreateKey` request is `true.` 

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

You can also use the `kms:BypassPolicyLockoutSafetyCheck` condition key in an IAM policy or key policy to control access to the `PutKeyPolicy` operation\. The following example policy statement from a key policy prevents users from bypassing the policy lockout safety check when changing the policy of a CMK\. 

Instead of using an explicit `Deny`, this policy statement uses `Allow` with the [Null condition operator](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_Null) to allow access only when the request does not include the `BypassPolicyLockoutSafetyCheck` parameter\. When the parameter is not used, the default value is `false`\. This slightly weaker policy statement can be overriden in the rare case that a bypass is necessary\. 

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

**See Also**
+ [kms:KeyOrigin](#conditions-kms-key-origin)

### kms:CallerAccount<a name="conditions-kms-caller-account"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:CallerAccount`  |  String  |  The `kms:CallerAccount` condition key exists for all AWS KMS operations *except* for these: `CreateKey`, `GenerateRandom`, `ListAliases`, `ListKeys`, `ListRetirableGrants`, `RetireGrant`\.  |  Key policies only  | 

You can use this condition key to allow or deny access to all identities \(IAM users and roles\) in an AWS account\. In key policies, you use the `Principal` element to specify the identities to which the policy statement applies\. The syntax for the `Principal` element does not provide a way to specify all identities in an AWS account\. But you can achieve this effect by combining this condition key with a `Principal` element that specifies all AWS identities\.

For example, the following policy statement demonstrates how to use the `kms:CallerAccount` condition key\. This policy statement is in the key policy for the AWS managed CMK for Amazon EBS\. It combines a `Principal` element that specifies all AWS identities with the `kms:CallerAccount` condition key to effectively allow access to all identities in AWS account 111122223333\. It contains an additional AWS KMS condition key \(`kms:ViaService`\) to further limit the permissions by only allowing requests that come through Amazon EBS\. For more information, see [kms:ViaService](#conditions-kms-via-service)\.

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

### kms:EncryptionContext:<a name="conditions-kms-encryption-context"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:EncryptionContext:`  |  String  |  `CreateGrant` `Encrypt` `Decrypt` `GenerateDataKey` `GenerateDataKeyWithoutPlaintext` `ReEncrypt`  |  IAM and key policies  | 

You can use the `kms:EncryptionContext:` condition key prefix to control access to a CMK based on the encryption context in a request for a cryptographic operation\. Use this condition key prefix to evaluate both the key and the value in the encryption context pair\. To evaluate only the encryption context keys, use the [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys) condition key\.

An [encryption context](encryption-context.md) is a set of nonsecret key–value pairs that you can include in a request for any AWS KMS cryptographic operation \([Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), and [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)\) and the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation\. When you specify an encryption context in an encryption operation, you must specify the same encryption context in the decryption operation\. Otherwise, the decryption request fails\.

To use the `kms:EncryptionContext:` condition key prefix, replace the `encryption_context_key` placeholder with the encryption context key\. Replace the `encryption_context_value` placeholder with the encryption context value\.

`"kms:EncryptionContext:encryption_context_key": "encryption_context_value"`

For example, the following condition key specifies an encryption context in which the key is `AppName` and the value is `ExampleApp`\.

```
"kms:EncryptionContext:AppName": "ExampleApp"
```

The following example key policy statement uses this condition key\. This policy allows the principal to use the CMK in a `GenerateDataKey` request only when the encryption context in the request is `"AppName": "ExampleApp"`\.

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

To require more than one encryption context pair, you can include multiple instances of the `kms:EncryptionContext:` condition\. For example, the following example policy statement requires both of the following encryption context pairs\. The order in which the pairs are specified does not matter\.
+ `"AppName": "ExampleApp"`
+ `"FilePath": "/var/opt/secrets/"`

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
      "kms:EncryptionContext:AppName": "ExampleApp",
      "kms:EncryptionContext:FilePath": "/var/opt/secrets/"
    }
  }
}
```

The encryption context that is specified in a decryption operation must be an exact, case\-sensitive match for the encryption context that is specified in the encryption operation\. Only the order of pairs in an encryption context with multiple pair can vary\.

However, in policy conditions, the condition key is not case sensitive\. The case sensitivity of the condition value is determined by the [policy condition operator](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html) that you use, such as `StringEquals` or `StringEqualsIgnoreCase`\.

As such, the condition key, which consists of the `kms:EncryptionContext:` prefix and the *`encryption_context_key`* replacement, is not case sensitive\. A policy that uses this condition does not check the case of either element of the condition key\. The case sensitivity of the value, that is, the *`encryption_context_value`* replacement, is determined by the policy condition operator\.

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

To require a case\-sensitive encryption context key, use the [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys) policy condition with a case\-sensitive condition operator, such as `StringEquals`\. In this policy condition, because the encryption context key is the policy condition value, its case sensitivity is determined by the condition operator\. 

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
      "kms:EncryptionContextKey": "AppName"
    }
  }
}
```

To require a case\-sensitive evaluation of both the encryption context key and value, use the `kms:EncryptionContextKey` and `kms:EncryptionContext:` policy conditions together in the same policy statement\. For example, in the following example policy statement, because the `StringEquals` operator is case sensitive, both the encryption context key and the encryption context value are case sensitive\.

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
      "kms:EncryptionContextKey": "AppName",
      "kms:EncryptionContext:AppName": "ExampleApp"
    }
  }
}
```

**See Also**
+ [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys)
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)

### kms:EncryptionContextKeys<a name="conditions-kms-encryption-context-keys"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:EncryptionContextKeys`  |  String \(list\)  |  `CreateGrant``Encrypt` `Decrypt` `GenerateDataKey` `GenerateDataKeyWithoutPlaintext` `ReEncrypt`  |  IAM and key policies  | 

You can use the `kms:EncryptionContextKeys` condition key to control access to a CMK based on the encryption context in a request for a cryptographic operation\. Use this condition key prefix to evaluate only the key in each encryption context pair\. To evaluate both the key and the value,, use the [kms:EncryptionContext:](#conditions-kms-encryption-context) condition key prefix\.

You can use this condition key to control access based on the [encryption context](encryption-context.md) in the AWS KMS API request\. Encryption context is a set of key–value pairs that you can include in AWS KMS cryptographic operations \([Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), and [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)\) and the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation\.

The following example policy statement uses the `kms:EncryptionContextKeys` condition key to allow use of a CMK for the specified operations only when the encryption context in the request includes the `AppName` key, regardless of its value\. 

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
    "StringEquals": {
      "kms:EncryptionContextKeys": "AppName"
    }
  }
}
```

Because the [StringEquals](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html) condition operation is case sensitive, the previous policy statement requires the spelling and case of the encryption context key\. But you can use a condition operator that ignores the case of the key, such as `StringEqualsIgnoreCase`\.

You can specify multiple encryption context keys in each condition\. For example, the following policy statement allows the specified operations only when the encryption context in the request includes both the `AppName` and `FilePath` keys, regardless of their values\. The order of keys in the encryption context does not matter\.

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
    "StringEquals": {
      "kms:EncryptionContextKeys": [
        "AppName",
        "FilePath"
      ]
    }
  }
}
```

You can also use the `kms:EncryptionContextKeys` condition key to require an encryption context in cryptographic operations that use the CMK\. 

The following example key policy statement uses the `kms:EncryptionContextKeys` condition key with the [Null condition operator](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_Null) to allow access to CMK only when the `kms:EncryptionContextKeys` condition key exists \(is not null\) in the API request\. It does not check the keys or values of the encryption context, only that the encryption context exists\. 

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

**See Also**
+ [kms:EncryptionContext:](#conditions-kms-encryption-context)
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)

### kms:ExpirationModel<a name="conditions-kms-expiration-model"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:ExpirationModel`  |  String  |  `ImportKeyMaterial`  |  IAM and key policies  | 

The `kms:ExpirationModel` condition key controls access to the [ImportKeyMaterial](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html) operation based on the value of the [ExpirationModel](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html#KMS-ImportKeyMaterial-request-ExpirationModel) parameter in the request\. 

`ExpirationModel` is an optional parameter that determines whether the imported key material expires\. Valid values are `KEY_MATERIAL_EXPIRES` and `KEY_MATERIAL_DOES_NOT_EXPIRE`\. `KEY_MATERIAL_EXPIRES` is the default value\. 

The expiration date and time is determined by the value of the [ValidTo](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html#KMS-ImportKeyMaterial-request-ValidTo) parameter\. The `ValidTo` parameter is required unless the value of the `ExpirationModel` parameter is `KEY_MATERIAL_DOES_NOT_EXPIRE`\. You can also use the [kms:ValidTo](#conditions-kms-valid-to) condition key to require a particular expiration date as a condition for access\.

The following example policy statement uses the `kms:ExpirationModel` condition key to allow a user to import key material into a CMK only when the request includes the `ExpirationModel` parameter and its value is `KEY_MATERIAL_DOES_NOT_EXPIRE`\. 

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

You can also use the `kms:ExpirationModel` condition key to allow a user to import key material only when the key material expires, without [specifying an expiration date](#conditions-kms-valid-to) in the condition\. The following example policy statement uses the `kms:ExpirationModel` condition key with the [Null condition operator](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_Null) to allow a user to import key material only when the request does not have an `ExpirationModel` parameter\. 

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

**See Also**
+ [kms:ValidTo](#conditions-kms-valid-to)
+ [kms:WrappingAlgorithm](#conditions-kms-wrapping-algorithm)
+ [kms:WrappingKeySpec](#conditions-kms-wrapping-key-spec)

### kms:GrantConstraintType<a name="conditions-kms-grant-constraint-type"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:GrantConstraintType`  |  String  |  `CreateGrant`  |  IAM and key policies  | 

You can use this condition key to control access to the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation based on the type of [grant constraint](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) in the request\. 

When you create a grant, you can optionally specify a grant constraint to allow the operations that the grant permit only when a particular encryption context is present\. The grant constraint can be one of two types: `EncryptionContextEquals` or `EncryptionContextSubset`\. You can use this condition key to check that the request contains one type or the other\.

The following example policy statement uses the `kms:GrantConstraintType` condition key to allow a user to create grants only when the request includes an `EncryptionContextEquals` grant constraint\. The example shows a policy statement in a key policy\.

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

**See Also**
+ [kms:EncryptionContext:](#conditions-kms-encryption-context)
+ [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys)
+ [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)
+ [kms:GrantOperations](#conditions-kms-grant-operations)
+ [kms:GranteePrincipal](#conditions-kms-grantee-principal)
+ [kms:RetiringPrincipal](#conditions-kms-retiring-principal)

### kms:GrantIsForAWSResource<a name="conditions-kms-grant-is-for-aws-resource"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:GrantIsForAWSResource`  |  Boolean  |  `CreateGrant`  |  IAM and key policies  | 

You can use this condition key to control access to the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation based on whether the grant is created in the context of an [AWS service integrated with AWS KMS](service-integration.md)\. This condition key is set to `true` when one of the following integrated services is used to create the grant:
+ Amazon Elastic Block Store \(Amazon EBS\) – For more information, see [How Amazon Elastic Block Store \(Amazon EBS\) Uses AWS KMS](services-ebs.md)\.
+ Amazon Relational Database Service \(Amazon RDS\) – For more information, see [How Amazon Relational Database Service \(Amazon RDS\) Uses AWS KMS](services-rds.md)\.
+ Amazon Redshift – For more information, see [How Amazon Redshift Uses AWS KMS](services-redshift.md)\.
+ AWS Certificate Manager \(ACM\) – For more information, see [ACM Private Key Security](https://docs.aws.amazon.com/acm/latest/userguide/kms.html) in the *AWS Certificate Manager User Guide*\.

For example, the following policy statement uses the `kms:GrantIsForAWSResource` condition key to allow a user to create grants only through one of the integrated services in the preceding list\. It does not allow the user to create grants directly\. The example shows a policy statement in a key policy\.

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

**See Also**
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)
+ [kms:GrantOperations](#conditions-kms-grant-operations)
+ [kms:GranteePrincipal](#conditions-kms-grantee-principal)
+ [kms:RetiringPrincipal](#conditions-kms-retiring-principal)

### kms:GrantOperations<a name="conditions-kms-grant-operations"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:GrantOperations`  |  String  |  `CreateGrant`  |  IAM and key policies  | 

You can use this condition key to control access to the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation based on the grant operations in the request\. For example, you can allow a user to create grants that delegate permission to encrypt but not decrypt\.

The following example policy statement uses the `kms:GrantOperations` condition key to allow a user to create grants that delegate permission to encrypt and reencrypt when this CMK is the destination CMK\. The example shows a policy statement in a key policy\.

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

**See Also**
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)
+ [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)
+ [kms:GranteePrincipal](#conditions-kms-grantee-principal)
+ [kms:RetiringPrincipal](#conditions-kms-retiring-principal)

### kms:GranteePrincipal<a name="conditions-kms-grantee-principal"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:GranteePrincipal`  |  String  |  `CreateGrant`  |  IAM and key policies  | 

You can use this condition key to control access to the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation based on the value of the [GranteePrincipal](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html#KMS-CreateGrant-request-GranteePrincipal) parameter in the request\. For example, you can allow a user to create grants to use a CMK only when the grantee principal in the `CreateGrant` request matches the principal specified in the condition statement\.

The following example policy statement uses the `kms:GranteePrincipal` condition key to allow a user to create grants for a CMK only when the grantee principal in the grant is the `LimitedAdminRole`\.

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

**See Also**
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)
+ [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)
+ [kms:GrantOperations](#conditions-kms-grant-operations)
+ [kms:RetiringPrincipal](#conditions-kms-retiring-principal)

### kms:KeyOrigin<a name="conditions-kms-key-origin"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:KeyOrigin`  |  String  |  `CreateKey`  |  IAM policies  | 

You can use this condition key to control access to the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation based on the value of the [Origin](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html#KMS-CreateKey-request-Origin) parameter in the request\. Valid values for `Origin` are `AWS_KMS`, `AWS_CLOUDHSM`, and `EXTERNAL`\. 

For example, you can allow a user to create a CMK only when the key material is generated in KMS \(`AWS_KMS`\), only when the key material is generated in an AWS CloudHSM cluster that is associated with a [custom key store](custom-key-store-overview.md) \(`AWS_CLOUDHSM`\), or only when the [key material is imported](importing-keys.md) from an external source \(`EXTERNAL`\)\. 

The following example policy statement uses the `kms:KeyOrigin` condition key to allow a user to create a CMK only when the key origin is EXTERNAL, that is, the key material is imported\.

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
      "kms:KeyOrigin": "EXTERNAL"
    }
  }
}
```

**See Also**
+ [kms:BypassPolicyLockoutSafetyCheck](#conditions-kms-bypass-policy-lockout-safety-check)

### kms:ReEncryptOnSameKey<a name="conditions-kms-reencrypt-on-same-key"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:ReEncryptOnSameKey`  |  Boolean  |  `ReEncrypt`  |  IAM and key policies  | 

You can use this condition key to control access to the [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) operation based on whether the request specifies a destination CMK that is the same one used for the original encryption\. For example, the following policy statement uses the `kms:ReEncryptOnSameKey` condition key to allow a user to reencrypt only when the destination CMK is the same one used for the original encryption\. The example shows a policy statement in a key policy\.

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

### kms:RetiringPrincipal<a name="conditions-kms-retiring-principal"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:RetiringPrincipal`  |  String \(list\)  |  `CreateGrant`  |  IAM and key policies  | 

You can use this condition key to control access to the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation based on the value of the [RetiringPrincipal](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html#KMS-CreateGrant-request-RetiringPrincipal) parameter in the request\. For example, you can allow a user to create grants to use a CMK only when the `RetiringPrincipal` in the `CreateGrant` request matches the `RetiringPrincipal` in the condition statement\.

The following example policy statement allows a user to create grants for the CMK\. The `kms:RetiringPrincipal` condition key restricts the permission to `CreateGrant` requests where the retiring principal in the grant is either the `LimitedAdminRole` or the `OpsAdmin` user\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": "kms:CreateGrant",
  "Resource": "*",
  "Condition": {
    "ForAnyValue:StringEquals": {
      "kms:RetiringPrincipal": [ 
         "arn:aws:iam::111122223333:role/LimitedAdminRole",
         "arn:aws:iam::111122223333:user/OpsAdmin"
      ]
    }
  }
}
```

**See Also**
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)
+ [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)
+ [kms:GrantOperations](#conditions-kms-grant-operations)
+ [kms:GranteePrincipal](#conditions-kms-grantee-principal)

### kms:ValidTo<a name="conditions-kms-valid-to"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:ValidTo`  |  Timestamp  |  `ImportKeyMaterial`  |  IAM and key policies  | 

The `kms:ValidTo` condition key controls access to the [ImportKeyMaterial](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html) operation based on the value of the [ValidTo](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html#KMS-ImportKeyMaterial-request-ValidTo) parameter in the request, which determines when the imported key material expires\. The value is expressed in [Unix time](https://en.wikipedia.org/wiki/Unix_time)\.

By default, the `ValidTo` parameter is required in an `ImportKeyMaterial` request\. However, if the value of the [ExpirationModel](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html#KMS-ImportKeyMaterial-request-ExpirationModel) parameter is `KEY_MATERIAL_DOES_NOT_EXPIRE`, the `ValidTo` parameter is invalid\. You can also use the [kms:ExpirationModel](#conditions-kms-expiration-model) condition key to require the `ExpirationModel` parameter or a specific parameter value\.

The following example policy statement allows a user to import key material into a CMK\. The `kms:ValidTo` condition key limits the permission to `ImportKeyMaterial` requests where the `ValidTo` value is less than or equal to `1546257599.0` \(December 31, 2018 11:59:59 PM\)\. 

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

**See Also**
+ [kms:ExpirationModel](#conditions-kms-expiration-model) 
+ [kms:WrappingAlgorithm](#conditions-kms-wrapping-algorithm)
+ [kms:WrappingKeySpec](#conditions-kms-wrapping-key-spec)

### kms:ViaService<a name="conditions-kms-via-service"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:ViaService`  |  String  |  The `kms:ViaService` condition key is valid for all AWS KMS operations *except*: `CreateKey`, `GenerateRandom`, `ListAliases`, `ListKeys`, `ListRetirableGrants`, `RetireGrant`\.  |  IAM and key policies  | 

The `kms:ViaService` condition key limits use of a [customer managed CMK](concepts.md#master_keys) to requests from particular AWS services\. \(AWS managed CMKs in your account, such as aws/s3, are always restricted to the AWS service that created them\.\)

For example, you can use `kms:ViaService` to allow a user to use a customer managed CMK only for requests that Amazon S3 makes on their behalf\. Or you can use it to deny the user permission to a CMK when a request on their behalf comes from AWS Lambda\. 

The `kms:ViaService` condition key is valid in IAM and key policy statements\. The services that you specify must be integrated with AWS KMS, support customer managed CMKs, and support the `kms:ViaService` condition key\. You can specify one or more services in each `kms:ViaService` condition key\. 

**Important**  
When you use the `kms:ViaService` condition key, verify that the principals have the following permissions:  
Permission to use the CMK\. The principal needs to grant these permissions to the integrated service so the service can use the customer managed CMK on behalf of the principal\. For more information, see [How AWS Services use AWS KMS](service-integration.md)\.
Permission to use the integrated service\. For details about giving users access to an AWS service that integrates with AWS KMS, consult the documentation for the integrated service\.

The following table shows the KMS `ViaService` name for each service that supports customer managed CMKs and the `kms:ViaService` condition key\. The services in this table might not be available in all regions\.


**Services that support the `kms:ViaService` condition key**  

| Service Name | KMS ViaService Name | 
| --- | --- | 
| Amazon Connect | connect\.AWS\_region\.amazonaws\.com | 
| AWS Database Migration Service \(AWS DMS\) | dms\.AWS\_region\.amazonaws\.com | 
| Amazon EC2 Systems Manager | ssm\.AWS\_region\.amazonaws\.com | 
| Amazon Elastic Block Store \(Amazon EBS\) | ec2\.AWS\_region\.amazonaws\.com \(EBS only\) | 
| Amazon Elastic File System | elasticfilesystem\.AWS\_region\.amazonaws\.com | 
| Amazon Elasticsearch Service | es\.AWS\_region\.amazonaws\.com | 
| Amazon Kinesis | kinesis\.AWS\_region\.amazonaws\.com | 
| Amazon Kinesis Video Streams | kinesisvideo\.AWS\_region\.amazonaws\.com | 
| AWS Lambda | lambda\.AWS\_region\.amazonaws\.com | 
| Amazon Lex | lex\.AWS\_region\.amazonaws\.com | 
| Amazon Redshift | redshift\.AWS\_region\.amazonaws\.com | 
| Amazon Relational Database Service \(Amazon RDS\) | rds\.AWS\_region\.amazonaws\.com | 
| AWS Secrets Manager \(Secrets Manager\) | secretsmanager\.AWS\_region\.amazonaws\.com | 
| Amazon Simple Email Service \(Amazon SES\) | ses\.AWS\_region\.amazonaws\.com | 
| Amazon Simple Storage Service \(Amazon S3\) | s3\.AWS\_region\.amazonaws\.com | 
| AWS Snowball | importexport\.AWS\_region\.amazonaws\.com | 
| Amazon SQS | sqs\.AWS\_region\.amazonaws\.com | 
| Amazon WorkMail | workmail\.AWS\_region\.amazonaws\.com | 
| Amazon WorkSpaces | workspaces\.AWS\_region\.amazonaws\.com | 

The following example shows a policy statement from a key policy for an AWS managed CMK\. The policy statement uses the `kms:ViaService` condition key to allow the CMK to be used for the specified actions only when the request comes from Amazon EBS or Amazon RDS in the US West \(Oregon\) region\. 

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

### kms:WrappingAlgorithm<a name="conditions-kms-wrapping-algorithm"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:WrappingAlgorithm`  |  String  |  `GetParametersForImport`  |  IAM and key policies  | 

This condition key controls access to the [GetParametersForImport](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation based on the value of the [WrappingAlgorithm](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetParametersForImport.html#KMS-GetParametersForImport-request-WrappingAlgorithm) parameter in the request\. You can use this condition to require principals to use a particular algorithm to encrypt key material during the import process Requests for the required public key and import token fail when they specify a different wrapping algorithm\.

The following example policy statement uses the `kms:WrappingAlgorithm` condition key to fail if the `WrappingAlgorithm` in the request is `RSAES_OAEP_SHA_1`\. The operation succeeds, and returns a public key and import token, for any other `WrappingAlgorithm` value\. 

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

**See Also**
+ [kms:ExpirationModel](#conditions-kms-expiration-model)
+ [kms:ValidTo](#conditions-kms-valid-to)
+ [kms:WrappingKeySpec](#conditions-kms-wrapping-key-spec)

### kms:WrappingKeySpec<a name="conditions-kms-wrapping-key-spec"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  `kms:WrappingKeySpec`  |  String  |  `GetParametersForImport`  |  IAM and key policies  | 

This condition key controls access to the [GetParametersForImport](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation based on the value of the [WrappingKeySpec](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetParametersForImport.html#KMS-GetParametersForImport-request-WrappingKeySpec) parameter in the request\. You can use this condition to require principals to use a particular type of public key during the import process\. If the request specifies a different key type, it fails\.

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
    "StringNotEquals": {
      "kms:WrappingKeySpec": "RSA_2048"
    }
  }
}
```

**See Also**
+ [kms:ExpirationModel](#conditions-kms-expiration-model)
+ [kms:ValidTo](#conditions-kms-valid-to)
+ [kms:WrappingAlgorithm](#conditions-kms-wrapping-algorithm)