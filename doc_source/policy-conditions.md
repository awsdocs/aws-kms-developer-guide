# Using policy conditions with AWS KMS<a name="policy-conditions"></a>

You can specify conditions in the key policies and AWS Identity and Access Management policies \([IAM policies](iam-policies.md)\) that control access to AWS KMS resources\. The policy statement is effective only when the conditions are true\. For example, you might want a policy statement to take effect only after a specific date\. Or, you might want a policy statement to control access only when a specific value appears in an API request\.

To specify conditions, you use predefined *condition keys* in the `Condition` element of a policy statement with [IAM condition policy operators](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html)\. Some condition keys apply generally to AWS; others are specific to AWS KMS\.

**Topics**
+ [AWS global condition keys](#conditions-aws)
+ [AWS KMS condition keys](#conditions-kms)

## AWS global condition keys<a name="conditions-aws"></a>

AWS defines [global condition keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#AvailableKeys), a set of policy conditions keys for all AWS services that use IAM for access control\. You can use global condition keys in AWS KMS key policies and IAM policies\. For example, you can use the [aws:PrincipalArn](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-principalarn) condition key to allow access to a customer master key \(CMK\) only when the principal in the request is represented by the Amazon Resource Name \(ARN\) in the condition key value\.

AWS KMS supports all AWS global condition keys except for the following ones:
+ [aws:ResourceTag](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-resourcetag)
+ [aws:SourceAccount](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourceaccount)
+ [aws:SourceArn](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourcearn)

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

If you use these condition keys in a key policy statement that allows or denies access to AWS KMS CMKs, you might inadvertently deny access to AWS services that use AWS KMS on your behalf\. 

Take care to avoid a situation like the [IP address condition keys](#conditions-aws-ip-address) example\. If you restrict requests for a CMK to a VPC or VPC endpoint, calls to AWS KMS from an integrated service, such as Amazon S3 or Amazon EBS, might fail\. This can happen even if the source request ultimately originates in the VPC or from the VPC endpoint\. 

## AWS KMS condition keys<a name="conditions-kms"></a>

AWS KMS provides an additional set of predefined condition keys that you can use in key policies and IAM policies\. These condition keys are specific to AWS KMS\. For example, you can use the `kms:EncryptionContext` condition key to require a particular [encryption context](concepts.md#encrypt_context) when controlling access to an AWS KMS symmetric customer master key \(CMK\)\.

**Conditions for an API operation request**

Many of the AWS KMS condition keys control access to a CMK based on the value of a parameter in the request for an AWS KMS operation\. For example, you can use the [kms:CustomerMasterKeySpec](#conditions-kms-customer-master-key-spec) condition key in an IAM policy to allow use of the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation only when the value of the `CustomerMasterKeySpec` parameter in the `CreateKey` request is `RSA_4096`\. 

This type of condition works even when the parameter doesn't appear in the request, such as when you use the parameter's default value\. For example you can use the [kms:CustomerMasterKeySpec](#conditions-kms-customer-master-key-spec) condition key to allow users to use the `CreateKey` operation only when the value of the `CustomerMasterKeySpec` parameter is `SYMMETRIC_DEFAULT`, which is the default value\. This condition allows requests that have the `CustomerMasterKeySpec` parameter with the `SYMMETRIC_DEFAULT` value and requests that have no `CustomerMasterKeySpec` parameter\.

**Conditions for CMKs used in API operations**

Some of the AWS KMS condition keys control access to operations based on a property of the CMK that is used in the operation\. For example, you can use the [kms:KeyOrigin](#conditions-kms-key-origin) condition to allow principals to call [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) on a CMK only when the `Origin` of the CMK is `AWS_KMS`\. To find out if a condition key can be used in this way, see the description of the condition key\.

The operation must be a *CMK resource operation*, that is, an operation that is authorized for a particular CMK\. To identify the CMK resource operations, in the [Actions and Resources Table](kms-api-permissions-reference.md#kms-api-permissions-reference-table), look for a value of `CMK` in the `Resources` column for the operation\. If you use this type of condition key with an operation that is not authorized for a particular CMK resource, like [ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html), the permission is not effective because the condition can never be satisfied\. There is no CMK resource involved in authorizing the `ListKeys` operation and no `CustomerMasterKeySpec` property\. 

The following topics describe each AWS KMS condition key and include example policy statements that demonstrate policy syntax\.

**Topics**
+ [kms:BypassPolicyLockoutSafetyCheck](#conditions-kms-bypass-policy-lockout-safety-check)
+ [kms:CallerAccount](#conditions-kms-caller-account)
+ [kms:CustomerMasterKeySpec](#conditions-kms-customer-master-key-spec)
+ [kms:CustomerMasterKeyUsage](#conditions-kms-customer-master-key-usage)
+ [kms:DataKeyPairSpec](#conditions-kms-data-key-spec)
+ [kms:EncryptionAlgorithm](#conditions-kms-encryption-algorithm)
+ [kms:EncryptionContext:](#conditions-kms-encryption-context)
+ [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys)
+ [kms:ExpirationModel](#conditions-kms-expiration-model)
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)
+ [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)
+ [kms:GrantOperations](#conditions-kms-grant-operations)
+ [kms:GranteePrincipal](#conditions-kms-grantee-principal)
+ [kms:KeyOrigin](#conditions-kms-key-origin)
+ [kms:MessageType](#conditions-kms-message-type)
+ [kms:ReEncryptOnSameKey](#conditions-kms-reencrypt-on-same-key)
+ [kms:RetiringPrincipal](#conditions-kms-retiring-principal)
+ [kms:SigningAlgorithm](#conditions-kms-signing-algorithm)
+ [kms:ValidTo](#conditions-kms-valid-to)
+ [kms:ViaService](#conditions-kms-via-service)
+ [kms:WrappingAlgorithm](#conditions-kms-wrapping-algorithm)
+ [kms:WrappingKeySpec](#conditions-kms-wrapping-key-spec)

### kms:BypassPolicyLockoutSafetyCheck<a name="conditions-kms-bypass-policy-lockout-safety-check"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:BypassPolicyLockoutSafetyCheck`  |  Boolean  |  `CreateKey` `PutKeyPolicy`  |  IAM policies only Key policies and IAM policies  | 

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

**See also**
+ [kms:CustomerMasterKeySpec](#conditions-kms-customer-master-key-spec)
+ [kms:KeyOrigin](#conditions-kms-key-origin)
+ [kms:CustomerMasterKeyUsage](#conditions-kms-customer-master-key-usage)

### kms:CallerAccount<a name="conditions-kms-caller-account"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:CallerAccount`  |  String  |  CMK resource operations  |  Key policies only  | 

You can use this condition key to allow or deny access to all identities \(IAM users and roles\) in an AWS account\. In key policies, you use the `Principal` element to specify the identities to which the policy statement applies\. The syntax for the `Principal` element does not provide a way to specify all identities in an AWS account\. But you can achieve this effect by combining this condition key with a `Principal` element that specifies all AWS identities\.

Because this condition is valid only in key policies, you can use it to control access to any *CMK resource operation*, that is, any AWS KMS operation that uses a particular CMK\. To identify the CMK resource operations, in the [Actions and Resources Table](kms-api-permissions-reference.md#kms-api-permissions-reference-table), look for a value of `CMK` in the `Resources` column for the operation\.

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

### kms:CustomerMasterKeySpec<a name="conditions-kms-customer-master-key-spec"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:CustomerMasterKeySpec`  |  String  |  `CreateKey` CMK resource operations |  IAM policies Key policies and IAM policies  | 

The `kms:CustomerMasterKeySpec` condition key controls access to operations based on the value of the `CustomerMasterKeySpec` property of the CMK that is created by or used in the operation\. 

You can use this condition key in an IAM policy to control access to the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation based on the value of the [CustomerMasterKeySpec](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html#KMS-CreateKey-request-KeySpec) parameter in a `CreateKey` request\. For example, you can use this condition to allow users to create only symmetric CMKs or only CMKs with RSA keys\.

The following example IAM policy statement uses the `kms:CustomerMasterKeySpec` condition key to allow the principals to create a CMK only when the `CustomerMasterKeySpec` in the request is `RSA_4096`\. 

```
{
  "Effect": "Allow",
  "Action": "kms:CreateKey",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:CustomerMasterKeySpec": "RSA_4096"
    }
  }
}
```

You can also use the `kms:CustomerMasterKeySpec` condition key to control access to operations that use or manage a CMK based on the `CustomerMasterKeySpec` property of the CMK used for the operation\. The operation must be a *CMK resource operation*, that is, an operation that is authorized for a particular CMK\. To identify the CMK resource operations, in the [Actions and Resources Table](kms-api-permissions-reference.md#kms-api-permissions-reference-table), look for a value of `CMK` in the `Resources` column for the operation\. 

For example, the following IAM policy allows principals to perform the specified CMK resource operations, but only with the symmetric CMKs in the account\. 

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
      "kms:CustomerMasterKeySpec": "SYMMETRIC_DEFAULT"
    }
  }
}
```

**See also**
+ [kms:BypassPolicyLockoutSafetyCheck](#conditions-kms-bypass-policy-lockout-safety-check)
+ [kms:CustomerMasterKeyUsage](#conditions-kms-customer-master-key-usage)
+ [kms:DataKeyPairSpec](#conditions-kms-data-key-spec)
+ [kms:KeyOrigin](#conditions-kms-key-origin)

### kms:CustomerMasterKeyUsage<a name="conditions-kms-customer-master-key-usage"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:CustomerMasterKeyUsage`  |  String  |  `CreateKey` CMK resource operations  |  IAM policies Key policies and IAM policies  | 

The `kms:CustomerMasterKeyUsage` condition key controls access to operations based on the value of the `KeyUsage` property of the CMK that is created by or used in the operation\. 

You can use this condition key to control access to the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation based on the value of the [KeyUsage](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html#KMS-CreateKey-request-KeyUsage) parameter in the request\. Valid values for `KeyUsage` are `ENCRYPT_DECRYPT` and `SIGN_VERIFY`\. 

For example, you can allow a user to create a CMK only when the `KeyUsage` is `ENCRYPT_DECRYPT` or deny a user permission when the `KeyUsage` is `SIGN_VERIFY`\. 

The following example IAM policy statement uses the `kms:CustomerMasterKeyUsage` condition key to allow a user to create a CMK only when the `KeyUsage` is `ENCRYPT_DECRYPT`\.

```
{
  "Effect": "Allow",  
  "Action": "kms:CreateKey",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:CustomerMasterKeyUsage": "ENCRYPT_DECRYPT"
    }
  }
}
```

You can also use the `kms:CustomerMasterKeyUsage` condition key to control access to operations that use or manage a CMK based on the `KeyUsage` property of the CMK used for the operation\. The operation must be a *CMK resource operation*, that is, an operation that is authorized for a particular CMK\. To identify the CMK resource operations, in the [Actions and Resources Table](kms-api-permissions-reference.md#kms-api-permissions-reference-table), look for a value of `CMK` in the `Resources` column for the operation\.

For example, the following IAM policy allows principals to perform the specified CMK resource operations, but only with CMKs in the account that are used for signing and verification\.

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
      "kms:CustomerMasterKeyUsage": "SIGN_VERIFY"
    }
  }
}
```

**See also**
+ [kms:BypassPolicyLockoutSafetyCheck](#conditions-kms-bypass-policy-lockout-safety-check)
+ [kms:CustomerMasterKeySpec](#conditions-kms-customer-master-key-spec)
+ [kms:KeyOrigin](#conditions-kms-key-origin)

### kms:DataKeyPairSpec<a name="conditions-kms-data-key-spec"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:DataKeySpec`  |  String  |  `GenerateDataKeyPair` `GenerateDataKeyPairWithoutPlaintext` |  Key policies and IAM policies  | 

You can use this condition key to control access to the [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html) and [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html) operations based on the value of the `KeyPairSpec` parameter in the request\. For example, you can allow a user to generate only particular types of data key pairs\.

The following example key policy statement uses the `kms:DataKeyPairSpec` condition key to allow a user to use the CMK to generate only RSA data key pairs\.

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
+ [kms:CustomerMasterKeySpec](#conditions-kms-customer-master-key-spec)
+ [kms:EncryptionAlgorithm](#conditions-kms-encryption-algorithm)
+ [kms:EncryptionContext:](#conditions-kms-encryption-context)
+ [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys)

### kms:EncryptionAlgorithm<a name="conditions-kms-encryption-algorithm"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:EncryptionAlgorithm`  |  String  |  `Decrypt` `Encrypt` `GenerateDataKey` `GenerateDataKeyPair` `GenerateDataKeyPairWithoutPlaintext` `GenerateDataKeyWithoutPlaintext` `ReEncrypt` |  Key policies and IAM policies  | 

You can use the `kms:EncryptionAlgorithm` condition key to control access to cryptographic operations based on the encryption algorithm that is used in the operation\. For the [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), and [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) operations, it controls access based on the value of the [EncryptionAlgorithm](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html#KMS-Decrypt-request-EncryptionAlgorithm) parameter in the request\. For operations that generate data keys and data key pairs, it controls access based on the encryption algorithm that is used to encrypt the data key\.

This condition key has no effect on operations performed outside of AWS KMS, such as encrypting with the public key in an asymmetric CMK pair outside of AWS KMS\.

**EncryptionAlgorithm parameter in a request**

To allow users to use only a particular encryption algorithm with a CMK, use a policy statement with a `Deny` effect and a `StringNotEquals` condition operator\. For example, the following example key policy statement prohibits principals who can assume the `ExampleRole` role from using this symmetric CMK in the specified cryptographic operations unless the encryption algorithm in the request is `RSAES_OAEP_SHA_256`\. 

Unlike a policy statement that allows a user to use a particular encryption algorithm, a policy statement with a double\-negative like this one prevents other policies and grants for this CMK from allowing this role to use other encryption algorithms\. The `Deny` in this policy statement takes precedence over any key policy or IAM policy with an `Allow` effect, and it takes precedence over all grants for this CMK and its principals\.

```
{
  "Sid": "Allow only one encryption algorithm with this asymmetric CMK",
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

You can also use the `kms:EncryptionAlgorithm` condition key to control access to the operations that generate data keys and data key pairs\. These operations use only symmetric CMKs and the `SYMMETRIC_DEFAULT` algorithm\. 

For example, this IAM policy limits its principals to symmetric encryption\. It denies access to any CMK in the example account for cryptographic operations unless the encryption algorithm specified in the request or used in the operation is SYMMETRIC\_DEFAULT\. The addition of [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html), and [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html) have no immediate practical effect because you can't use an asymmetric CMK or asymmetric encryption algorithm to encrypt a data key or encrypt the private key in a data key pair\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Deny",
    "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:ReEncrypt*",
    "kms:GenerateDataKey*",
    "kms:GenerateDataKeyPair*"
  ],
    "Resource": {
      "arn:aws:kms:us-west-2:111122223333:key/*"
    },
  "Condition": {
    "StringNotEquals": {
      "kms:EncryptionAlgorithm": "SYMMETRIC_DEFAULT"
    }
  }
}
```

**See also**
+ [kms:SigningAlgorithm](#conditions-kms-signing-algorithm)

### kms:EncryptionContext:<a name="conditions-kms-encryption-context"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:EncryptionContext:`  |  String  |  `CreateGrant` `Encrypt` `Decrypt` `GenerateDataKey` `GenerateDataKeyPair` `GenerateDataKeyPairWithoutPlaintext` `GenerateDataKeyWithoutPlaintext` `ReEncrypt`  |  Key policies and IAM policies  | 

You can use the `kms:EncryptionContext:` condition key prefix to control access to a [symmetric CMK](symm-asymm-concepts.md#symmetric-cmks) based on the encryption context in a request for a cryptographic operation\. Use this condition key prefix to evaluate both the key and the value in the encryption context pair\. To evaluate only the encryption context keys, use the [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys) condition key\.

An [encryption context](concepts.md#encrypt_context) is a set of nonsecret key–value pairs that you can include in a request for any AWS KMS cryptographic operation that uses a symmetric CMK \([Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), and [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)\), and the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation\. When you specify an encryption context in an encryption operation, you must specify the same encryption context in the decryption operation\. Otherwise, the decryption request fails\.

You cannot specify an encryption context in a cryptographic operation with an [asymmetric CMK](symm-asymm-concepts.md#asymmetric-cmks)\. The standard asymmetric encryption algorithms that AWS KMS uses do not support an encryption context\.

To use the `kms:EncryptionContext:` condition key prefix, replace the `encryption_context_key` placeholder with the encryption context key\. Replace the `encryption_context_value` placeholder with the encryption context value\.

```
"kms:EncryptionContext:encryption_context_key": "encryption_context_value"
```

For example, the following condition key specifies an encryption context in which the key is `AppName` and the value is `ExampleApp`\.

```
"kms:EncryptionContext:AppName": "ExampleApp"
```

The following example key policy statement uses this condition key\. Because there can be multiple encryption context pairs in a request, the [condition operator](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_multi-value-conditions.html) must include `ForAnyValue` or `ForAllValues`\. 

This policy allows the principal to use the CMK in a `GenerateDataKey` request only when at least one of the encryption context pairs in the request is `"AppName": "ExampleApp"`\.

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
      "kms:EncryptionContext:AppName": "ExampleApp"
    }
  }
}
```

#### Requiring multiple encryption context pairs<a name="conditions-kms-encryption-context-many"></a>

To require more than one encryption context pair, you can include multiple instances of the `kms:EncryptionContext:` condition\. For example, the following example policy statement uses the `ForAllValues` operator to require both of the following encryption context pairs \(and no others\)\. The order in which the pairs are specified does not matter\.
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
    "ForAllValues:StringEquals": { 
      "kms:EncryptionContext:AppName": "ExampleApp",
      "kms:EncryptionContext:FilePath": "/var/opt/secrets/"
    }
  }
}
```

#### Case sensitivity of the encryption context condition<a name="conditions-kms-encryption-context-case"></a>

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
    "ForAnyValue:StringEquals": {
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
    "ForAnyValue:StringEquals": {
      "kms:EncryptionContextKey": "AppName"
    }
  }
}
```

To require a case\-sensitive evaluation of both the encryption context key and value, use the `kms:EncryptionContextKeys` and `kms:EncryptionContext:` policy conditions together in the same policy statement\. For example, in the following example policy statement, because the `StringEquals` operator is case sensitive, both the encryption context key and the encryption context value are case sensitive\.

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
      "kms:EncryptionContextKeys": "AppName",
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

However, the value in the `kms:EncryptionContext:` condition key pair can be an [IAM policy variable](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html)\. These policy variables are resolved at runtime based on values in the request\. For example, `aws:CurrentTime `resolves to the time of the request and `aws:username` resolves to the friendly name of the caller\.

You can use these policy variables to create a policy statement with a condition that requires very specific information in an encryption context, such as the caller's user name\. Because it contains a variable, you can use the same policy statement for all users who can assume the role\. You don't have to write a separate policy statement for each user\.

Consider a situation where you want to all users who can assume a role to use the same CMK to encrypt and decrypt their data\. However, you want to allow them to decrypt only the data that they encrypted\. Start by requiring that every request to AWS KMS include an encryption context where the key is `user` and the value is the caller's AWS user name, such as the following one\.

```
"encryptionContext": {
    "user": "bob"
}
```

Then, to enforce this requirement, you can use a policy statement like the one in the following example\. This policy statement gives the `TestTeam` role permission to encrypt and decrypt data with the CMK\. However, the permission is valid only when the encryption context in the request includes a `"user": "<username>"` pair\. To represent the user name, the condition uses the [https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html#policy-vars-infotouse](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html#policy-vars-infotouse) policy variable\.

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
  ]
  "Resource": "*",
  "Condition": {
    "ForAnyValue:StringEquals": {
       "kms:EncryptionContext:user": "${aws:username}"
    }
  }
}
```

You can use an IAM policy variable only in the value of the `kms:EncryptionContext:` condition key pair\. You cannot use a variable in the key\.

You can also use [provider\-specific context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_oidc_user-id.html) in variables\. These context keys uniquely identify users who logged into AWS by using web identity federation\. 

Like all variables, these variables can be used only in the `kms:EncryptionContext:` policy condition, not in the actual encryption context\. And they can be used only in the value of the condition, not in the key\.

For example, the following key policy statement is similar to the previous one\. However, the condition requires an encryption context where the key is `sub` and the value uniquely identifies a user logged into a Amazon Cognito user pool\. For details about identifying users and roles in Amazon Cognito, see [IAM Roles](https://docs.aws.amazon.com/cognito/latest/developerguide/iam-roles.html) in the [Amazon Cognito Developer Guide](https://docs.aws.amazon.com/cognito/latest/developerguide/)\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/TestTeam"
  },
  "Action": [
    "kms:Decrypt",
    "kms:Encrypt"
  ]
  "Resource": "*",
  "Condition": {
    "ForAnyValue:StringEquals": {
       "kms:EncryptionContext:sub": "${cognito-identity.amazonaws.com:sub}"
    }
  }
}
```

**See also**
+ [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys)
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)

### kms:EncryptionContextKeys<a name="conditions-kms-encryption-context-keys"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:EncryptionContextKeys`  |  String \(list\)  |  `CreateGrant` `Decrypt` `Encrypt` `GenerateDataKey` `GenerateDataKeyPair` `GenerateDataKeyPairWithoutPlaintext` `GenerateDataKeyWithoutPlaintext` `ReEncrypt`  |  Key policies and IAM policies  | 

You can use the `kms:EncryptionContextKeys` condition key to control access to a [symmetric CMK](symm-asymm-concepts.md#symmetric-cmks) based on the encryption context in a request for a cryptographic operation\. Use this condition key prefix to evaluate only the key in each encryption context pair\. To evaluate both the key and the value, use the [kms:EncryptionContext:](#conditions-kms-encryption-context) condition key prefix\.

You cannot specify an encryption context in a cryptographic operation with an [asymmetric CMK](symm-asymm-concepts.md#asymmetric-cmks)\. The standard asymmetric encryption algorithms that AWS KMS uses do not support an encryption context\. 

You can use this condition key to control access based on the [encryption context](concepts.md#encrypt_context) in the AWS KMS API request\. Encryption context is a set of key–value pairs that you can include in AWS KMS cryptographic operations with symmetric CMKs \([Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), and [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)\) and the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation\. Because there can be multiple encryption context pairs in a request, the [condition operator](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_multi-value-conditions.html) must include `ForAnyValue` or `ForAllValues`\. 

The following example policy statement uses the `kms:EncryptionContextKeys` condition key to allow use of a CMK for the specified operations only when at least one of the encryption context pairs in the request includes the `AppName` key, regardless of its value\. 

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

Because the [StringEquals](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html) condition operation is case sensitive, the previous policy statement requires the spelling and case of the encryption context key\. But you can use a condition operator that ignores the case of the key, such as `StringEqualsIgnoreCase`\.

You can specify multiple encryption context keys in each condition\. For example, the following policy statement uses the `ForAllValues` and `StringEquals` condition operators to allow the specified operations only when the encryption context in the request includes both the `AppName` and `FilePath` keys \(and no others\), regardless of their values\. The order of keys in the encryption context does not matter\.

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
    "ForAllValues:StringEquals": {
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

**See also**
+ [kms:EncryptionContext:](#conditions-kms-encryption-context)
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)

### kms:ExpirationModel<a name="conditions-kms-expiration-model"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:ExpirationModel`  |  String  |  `ImportKeyMaterial`  |  Key policies and IAM policies  | 

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

**See also**
+ [kms:ValidTo](#conditions-kms-valid-to)
+ [kms:WrappingAlgorithm](#conditions-kms-wrapping-algorithm)
+ [kms:WrappingKeySpec](#conditions-kms-wrapping-key-spec)

### kms:GrantConstraintType<a name="conditions-kms-grant-constraint-type"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:GrantConstraintType`  |  String  |  `CreateGrant`  |  Key policies and IAM policies  | 

You can use this condition key to control access to the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation based on the type of [grant constraint](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) in the request\. 

When you create a grant, you can optionally specify a grant constraint to allow the operations that the grant permit only when a particular [encryption context](concepts.md#encrypt_context) is present\. The grant constraint can be one of two types: `EncryptionContextEquals` or `EncryptionContextSubset`\. You can use this condition key to check that the request contains one type or the other\.

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

**See also**
+ [kms:EncryptionContext:](#conditions-kms-encryption-context)
+ [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys)
+ [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)
+ [kms:GrantOperations](#conditions-kms-grant-operations)
+ [kms:GranteePrincipal](#conditions-kms-grantee-principal)
+ [kms:RetiringPrincipal](#conditions-kms-retiring-principal)

### kms:GrantIsForAWSResource<a name="conditions-kms-grant-is-for-aws-resource"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:GrantIsForAWSResource`  |  Boolean  |  `CreateGrant` `ListGrants` `RevokeGrant` |  Key policies and IAM policies  | 

Allows or denies permission for the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html), [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html), or [RevokeGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) operations only when an [AWS services integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration) calls the operation on the user's behalf\. This policy condition doesn't allow the user to call these grant operations directly\.

The following example key policy statement uses the `kms:GrantIsForAWSResource` condition key\. It allows AWS services that are integrated with AWS KMS, such as Amazon EBS, to create grants on this CMK on behalf of the specified user\. 

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


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:GrantOperations`  |  String  |  `CreateGrant`  |  Key policies and IAM policies  | 

You can use this condition key to control access to the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation based on the grant operations in the request\. For example, you can allow a user to create grants that delegate permission to encrypt but not decrypt\.

The following example policy statement uses the `kms:GrantOperations` condition key to allow a user to create grants that delegate permission to encrypt and re\-encrypt when this CMK is the destination CMK\. The example shows a policy statement in a key policy\.

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

**See also**
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)
+ [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)
+ [kms:GranteePrincipal](#conditions-kms-grantee-principal)
+ [kms:RetiringPrincipal](#conditions-kms-retiring-principal)

### kms:GranteePrincipal<a name="conditions-kms-grantee-principal"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
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

**See also**
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)
+ [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)
+ [kms:GrantOperations](#conditions-kms-grant-operations)
+ [kms:RetiringPrincipal](#conditions-kms-retiring-principal)

### kms:KeyOrigin<a name="conditions-kms-key-origin"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:KeyOrigin`  |  String  |  `CreateKey` CMK resource operations  |  IAM policies Key policies and IAM policies  | 

The `kms:KeyOrigin` condition key controls access to operations based on the value of the `Origin` property of the CMK that is created by or used in the operation\. It works as a resource condition or a request condition\.

You can use this condition key to control access to the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation based on the value of the [Origin](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html#KMS-CreateKey-request-Origin) parameter in the request\. Valid values for `Origin` are `AWS_KMS`, `AWS_CLOUDHSM`, and `EXTERNAL`\. 

For example, you can allow a user to create a CMK only when the key material is generated in KMS \(`AWS_KMS`\), only when the key material is generated in an AWS CloudHSM cluster that is associated with a [custom key store](custom-key-store-overview.md) \(`AWS_CLOUDHSM`\), or only when the [key material is imported](importing-keys.md) from an external source \(`EXTERNAL`\)\. 

The following example policy statement uses the `kms:KeyOrigin` condition key to allow a user to create a CMK only when AWS KMS creates the key material\.

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

You can also use the `kms:KeyOrigin` condition key to control access to operations that use or manage a CMK based on the `Origin` property of the CMK used for the operation\. The operation must be a *CMK resource operation*, that is, an operation that is authorized for a particular CMK\. To identify the CMK resource operations, in the [Actions and Resources Table](kms-api-permissions-reference.md#kms-api-permissions-reference-table), look for a value of `CMK` in the `Resources` column for the operation\.

For example, the following IAM policy allows principals to perform the specified CMK resource operations, but only with CMKs in the account that were created in a custom key store\.

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
  "Resource": {
      "arn:aws:kms:us-west-2:111122223333:key/*"
  },
  "Condition": {
    "StringEquals": {
      "kms:KeyOrigin": "AWS_CLOUDHSM"
    }
  }
}
```

**See also**
+ [kms:BypassPolicyLockoutSafetyCheck](#conditions-kms-bypass-policy-lockout-safety-check)
+ [kms:CustomerMasterKeySpec](#conditions-kms-customer-master-key-spec)
+ [kms:CustomerMasterKeyUsage](#conditions-kms-customer-master-key-usage)

### kms:MessageType<a name="conditions-kms-message-type"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:MessageType`  |  String  |  `Sign` `Verify`  | Key policies and IAM policies | 

The `kms:MessageType` condition key controls access to the [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html) and [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) operations based on the value of the `MessageType` parameter in the request\. Valid values for `MessageType` are `RAW` and `DIGEST`\. 

For example, the following key policy statement uses the `kms:MessageType` condition key to allow a user to use an asymmetric CMK to sign a message, but not a message digest\.

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

### kms:ReEncryptOnSameKey<a name="conditions-kms-reencrypt-on-same-key"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:ReEncryptOnSameKey`  |  Boolean  |  `ReEncrypt`  |  Key policies and IAM policies  | 

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


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:RetiringPrincipal`  |  String \(list\)  |  `CreateGrant`  |  Key policies and IAM policies  | 

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


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:SigningAlgorithm`  |  String  |  `Sign`  `Verify` |  Key policies and IAM policies  | 

You can use the `kms:SigningAlgorithm` condition key to control access to the [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html) and [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) operations based on the value of the [SigningAlgorithm](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html#KMS-Sign-request-SigningAlgorithm) parameter in the request\. This condition key has no effect on operations performed outside of AWS KMS, such as verifying signatures with the public key in an asymmetric CMK pair outside of AWS KMS\.

The following example key policy allows users who can assume the `testers` role to use the CMK to sign messages only when the signing algorithm used for the request is an RSASSA\_PSS algorithm, such as `RSASSA_PSS_SHA512`\.

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


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:ValidTo`  |  Timestamp  |  `ImportKeyMaterial`  |  Key policies and IAM policies  | 

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

**See also**
+ [kms:ExpirationModel](#conditions-kms-expiration-model) 
+ [kms:WrappingAlgorithm](#conditions-kms-wrapping-algorithm)
+ [kms:WrappingKeySpec](#conditions-kms-wrapping-key-spec)

### kms:ViaService<a name="conditions-kms-via-service"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:ViaService`  |  String  |  CMK resource operations  |  Key policies and IAM policies  | 

The `kms:ViaService` condition key limits use of an AWS KMS [customer master key](concepts.md#master_keys) \(CMK\) to requests from specified AWS services\. You can specify one or more services in each `kms:ViaService` condition key\. The operation must be a *CMK resource operation*, that is, an operation that is authorized for a particular CMK\. To identify the CMK resource operations, in the [Actions and Resources Table](kms-api-permissions-reference.md#kms-api-permissions-reference-table), look for a value of `CMK` in the `Resources` column for the operation\.

For example, the following statement from a key policy uses the `kms:ViaService` condition key to allow a [customer managed CMK](concepts.md#customer-cmk) to be used for the specified actions only when the request comes from Amazon EC2 or Amazon RDS in the US West \(Oregon\) region on behalf of `ExampleUser`\.

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

You can also use a `kms:ViaService` condition key to deny permission to use a CMK when the request comes from particular services\. For example, the following policy statement from a key policy uses a `kms:ViaService` condition key to prevent a customer managed CMK from being used for `Encrypt` operations when the request comes from AWS Lambda on behalf of `ExampleUser`\.

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
Permission to use the CMK\. The principal needs to grant these permissions to the integrated service so the service can use the customer managed CMK on behalf of the principal\. For more information, see [How AWS services use AWS KMS](service-integration.md)\.
Permission to use the integrated service\. For details about giving users access to an AWS service that integrates with AWS KMS, consult the documentation for the integrated service\.

All [AWS managed CMKs](concepts.md#aws-managed-cmk) use a `kms:ViaService` condition key in their key policy document\. This condition allows the CMK to be used only for requests that come from the service that created the CMK\. To see the key policy for an AWS managed CMK, use the [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation\. 

The `kms:ViaService` condition key is valid in IAM and key policy statements\. The services that you specify must be [integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration) and support the `kms:ViaService` condition key\.

The following table lists AWS services that are integrated with AWS KMS, support customer managed CMKs, and support the use of the `kms:ViaService` condition key in customer managed CMKs\. The services in this table might not be available in all regions\. Use the `.amazonaws.com` suffix of the AWS KMS ViaService name in all AWS partitions\.


**Services that support the `kms:ViaService` condition key in customer managed CMKs**  

| Service name | AWS KMS ViaService name | 
| --- | --- | 
| AWS Backup | backup\.AWS\_region\.amazonaws\.com | 
| Amazon Connect | connect\.AWS\_region\.amazonaws\.com | 
| AWS Database Migration Service \(AWS DMS\) | dms\.AWS\_region\.amazonaws\.com | 
| AWS Directory Service | directoryservice\.AWS\_region\.amazonaws\.com | 
| Amazon DynamoDB | dynamodb\.AWS\_region\.amazonaws\.com | 
| Amazon EC2 Systems Manager | ssm\.AWS\_region\.amazonaws\.com | 
| Amazon Elastic Block Store \(Amazon EBS\) | ec2\.AWS\_region\.amazonaws\.com \(EBS only\) | 
| Amazon Elastic File System | elasticfilesystem\.AWS\_region\.amazonaws\.com | 
| Amazon Elasticsearch Service | es\.AWS\_region\.amazonaws\.com | 
| Amazon FSx | fsx\.AWS\_region\.amazonaws\.com | 
| AWS Glue | glue\.AWS\_region\.amazonaws\.com | 
| Amazon Kinesis | kinesis\.AWS\_region\.amazonaws\.com | 
| Amazon Kinesis Video Streams | kinesisvideo\.AWS\_region\.amazonaws\.com | 
| AWS Lambda | lambda\.AWS\_region\.amazonaws\.com | 
| Amazon Lex | lex\.AWS\_region\.amazonaws\.com | 
| Amazon Managed Streaming for Apache Kafka | kafka\.AWS\_region\.amazonaws\.com | 
| Amazon Neptune | rds\.AWS\_region\.amazonaws\.com | 
| Amazon Redshift | redshift\.AWS\_region\.amazonaws\.com | 
| Amazon Relational Database Service \(Amazon RDS\) | rds\.AWS\_region\.amazonaws\.com | 
| Amazon RDS Performance Insights | rds\.AWS\_region\.amazonaws\.com | 
| AWS Secrets Manager \(Secrets Manager\) | secretsmanager\.AWS\_region\.amazonaws\.com | 
| Amazon Simple Email Service \(Amazon SES\) | ses\.AWS\_region\.amazonaws\.com | 
| Amazon Simple Notification Service \(Amazon SNS\) | sns\.AWS\_region\.amazonaws\.com | 
| Amazon Simple Storage Service \(Amazon S3\) | s3\.AWS\_region\.amazonaws\.com | 
| AWS Snowball | importexport\.AWS\_region\.amazonaws\.com | 
| Amazon SQS | sqs\.AWS\_region\.amazonaws\.com | 
| Amazon WorkMail | workmail\.AWS\_region\.amazonaws\.com | 
| Amazon WorkSpaces | workspaces\.AWS\_region\.amazonaws\.com | 
| AWS X\-Ray | xray\.AWS\_region\.amazonaws\.com | 

### kms:WrappingAlgorithm<a name="conditions-kms-wrapping-algorithm"></a>


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:WrappingAlgorithm`  |  String  |  `GetParametersForImport`  |  Key policies and IAM policies  | 

This condition key controls access to the [GetParametersForImport](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetParametersForImport.html) operation based on the value of the [WrappingAlgorithm](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetParametersForImport.html#KMS-GetParametersForImport-request-WrappingAlgorithm) parameter in the request\. You can use this condition to require principals to use a particular algorithm to encrypt key material during the import process\. Requests for the required public key and import token fail when they specify a different wrapping algorithm\.

The following example policy statement uses the `kms:WrappingAlgorithm` condition key to give the example user permission to call the `GetParametersForImport` operation, but prevents them from using the `RSAES_OAEP_SHA_1` wrapping algorithm\. When the `WrappingAlgorithm` in the `GetParametersForImport` request is `RSAES_OAEP_SHA_1`, the operation fails\.

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


| AWS KMS condition keys | Condition type | API operations | Policy type | 
| --- | --- | --- | --- | 
|  `kms:WrappingKeySpec`  |  String  |  `GetParametersForImport`  |  Key policies and IAM policies  | 

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