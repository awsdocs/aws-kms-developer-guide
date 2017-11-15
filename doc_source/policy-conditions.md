# Using Policy Conditions with AWS KMS<a name="policy-conditions"></a>

When you use key policies and IAM policies to control access to your AWS KMS resources, you can use a policy statement's `Condition` element to specify the circumstances in which the statement takes effect\. For example, you might want a policy statement to take effect only after a specific date\. Or, you might want a policy statement to control access based on whether a specific value exists in the API request\.

To specify conditions, you use predefined *condition keys* in a policy statement's `Condition` element\. Some condition keys apply generally to AWS, and some are specific to AWS KMS\.


+ [AWS Condition Keys](#conditions-aws)
+ [AWS KMS Condition Keys](#conditions-kms)

## AWS Condition Keys<a name="conditions-aws"></a>

AWS provides a set of predefined condition keys, called *AWS\-wide condition keys*, for all AWS services that support IAM for access control\. For example, you can use the `aws:MultiFactorAuthPresent` condition key to require multi\-factor authentication \(MFA\)\. For more information and a list of the AWS\-wide condition keys, see [Available Keys for Conditions](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html#AvailableKeys) in the *IAM User Guide*\.

### Using the IP Address Condition in Policies with AWS KMS Permissions<a name="conditions-aws-ip-address"></a>

If you use AWS KMS to protect your data in an integrated service, use caution when specifying the [IP address condition operators](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html#Conditions_IPAddress) or the `aws:SourceIp` condition key in the same policy statement that allows or denies access to AWS KMS\. For example, a policy like the one shown at [Block Requests That Don't Come From an Approved IP Address or Range](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html#iam-policy-example-deny-source-ip-address) in the *IAM User Guide* will deny access to the AWS services that make requests to AWS KMS on your behalf\. Consider this scenario:

1. You attach a policy like the one shown at [Block Requests That Don't Come From an Approved IP Address or Range](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html#iam-policy-example-deny-source-ip-address) to an IAM user\. This IAM user has other policies attached that allow it to use Amazon EBS, Amazon EC2, and AWS KMS\.

1. The user attempts to attach an encrypted EBS volume to an EC2 instance\. This action fails with an authorization error despite the user being allowed to use all of the relevant services\.

The action at step 2 fails because when the user attempts to attach the encrypted EBS volume to an EC2 instance, Amazon EC2 sends a request to AWS KMS to decrypt the volume's encrypted data key\. This request comes from an IP address associated with the Amazon EC2 infrastructure, not the IP address of the originating user\. The policy you attached at step 1 contains an explicit deny statement that denies all requests that don't come from the approved IP addresses, which means that Amazon EC2 is denied the access it needs to decrypt the EBS volume's encrypted data key\.

## AWS KMS Condition Keys<a name="conditions-kms"></a>

AWS KMS provides an additional set of predefined condition keys that you can use in key policies and IAM policies\. These condition keys are specific to AWS KMS\. For example, you can use the `kms:EncryptionContext` condition key to require a particular encryption context when controlling access to a KMS customer master key \(CMK\)\.

The following topics describe each AWS KMS condition key and include example policy statements that demonstrate policy syntax\.


+ [kms:BypassPolicyLockoutSafetyCheck](#conditions-kms-bypass-policy-lockout-safety-check)
+ [kms:CallerAccount](#conditions-kms-caller-account)
+ [kms:EncryptionContext:](#conditions-kms-encryption-context)
+ [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys)
+ [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)
+ [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)
+ [kms:GrantOperations](#conditions-kms-grant-operations)
+ [kms:ReEncryptOnSameKey](#conditions-kms-reencrypt-on-same-key)
+ [kms:ViaService](#conditions-kms-via-service)

### kms:BypassPolicyLockoutSafetyCheck<a name="conditions-kms-bypass-policy-lockout-safety-check"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  [kms:BypassPolicyLockoutSafetyCheck](#conditions-kms-bypass-policy-lockout-safety-check)  |  Boolean  |  `CreateKey`  |  IAM policies only  | 

You can use this condition key to control access to the [CreateKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation based on the `BypassPolicyLockoutSafetyCheck` parameter in the request\. For example, you can prevent users from bypassing the policy lockout safety check by denying them permission to create CMKs when the request's `BypassPolicyLockoutSafetyCheck` parameter is true, as in the following example policy statement\. The following example shows a policy statement in an IAM policy\.

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

### kms:CallerAccount<a name="conditions-kms-caller-account"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  [kms:CallerAccount](#conditions-kms-caller-account)  |  String  |  The `kms:CallerAccount` condition key exists for all AWS KMS operations *except* for these: `CreateKey`, `GenerateRandom`, `ListAliases`, `ListKeys`, `ListRetirableGrants`, `RetireGrant`\.  |  Key policies only  | 

You can use this condition key to allow or deny access to all identities \(IAM users and roles\) in an AWS account\. In key policies, you use the `Principal` element to specify the identities to which the policy statement applies\. The syntax for the `Principal` element does not provide a way to specify all identities in an AWS account, but you can achieve this effect by combining this condition key with a `Principal` element that specifies all AWS identities\.

For example, the following policy statement demonstrates how to use the `kms:CallerAccount` condition key\. This policy statement is in the key policy for the AWS\-managed CMK for Amazon EBS\. It combines a `Principal` element that specifies all AWS identities with the `kms:CallerAccount` condition key to effectively allow access to all identities in AWS account 111122223333\. It contains an additional AWS KMS condition key \(`kms:ViaService`\) to further limit the permissions by only allowing requests that come through Amazon EBS\. For more information, see [kms:ViaService](#conditions-kms-via-service)\.

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
|  [kms:EncryptionContext:](#conditions-kms-encryption-context)  |  String  |  `Encrypt` `Decrypt` `GenerateDataKey` `GenerateDataKeyWithoutPlaintext` `ReEncrypt`  |  IAM and key policies  | 

You can use this condition key prefix to control access based on the encryption context in the AWS KMS API request\. Encryption context is a set of key–value pairs that you can include with AWS KMS API operations that perform encryption and decryption \([Encrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [Decrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), [GenerateDataKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), and [ReEncrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)\)\. Use this condition key prefix to check both sides of the encryption context; that is, both the key and the value\. To use this condition key prefix, pair it with the encryption context key to form a custom condition key, like this:

`kms:EncryptionContext:encryption_context_key`

The following example policy statement uses the `kms:EncryptionContext:` condition key prefix to allow access to use a CMK only when the encryption context contains the following key–value pairs:

+ `AppName` = `ExampleApp`

+ `FilePath` = `/var/opt/secrets/`

To do this, the `kms:EncryptionContext:` condition key prefix is paired with each encryption context key to form custom condition keys \(`kms:EncryptionContext:AppName` and `kms:EncryptionContext:FilePath`\)\.

The following example shows a policy statement in a key policy\.

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

### kms:EncryptionContextKeys<a name="conditions-kms-encryption-context-keys"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  [kms:EncryptionContextKeys](#conditions-kms-encryption-context-keys)  |  String  |  `Encrypt` `Decrypt` `GenerateDataKey` `GenerateDataKeyWithoutPlaintext` `ReEncrypt`  |  IAM and key policies  | 

You can use this condition key to control access based on the encryption context in the AWS KMS API request\. Encryption context is a set of key–value pairs that you can include with AWS KMS API operations that perform encryption and decryption \([Encrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [Decrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), [GenerateDataKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), and [ReEncrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)\)\. Use this condition key to check only the encryption context keys, not the values\.

The following example policy statement uses the `kms:EncryptionContextKeys` condition key with the [Null condition operator](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html#Conditions_Null) to allow access to use a CMK only when the request contains encryption context\. It does this by allowing access only when the `kms:EncryptionContextKeys` condition key exists \(is not null\) in the API request\. It does not check the keys or values of the encryption context, only that encryption context exists\. The following example shows a policy statement in a key policy\.

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

### kms:GrantConstraintType<a name="conditions-kms-grant-constraint-type"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  [kms:GrantConstraintType](#conditions-kms-grant-constraint-type)  |  String  |  `CreateGrant`  |  IAM and key policies  | 

You can use this condition key to control access to the [CreateGrant](http://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation based on the type of grant constraint in the request\. When you create a grant, you can optionally specify a grant constraint to allow the operations permitted by the grant only when a particular encryption context is present\. The grant constraint can be one of two types: `EncryptionContextEquals` or `EncryptionContextSubset`\. You can use this condition key to check that the request contains one type or the other\. For more information about grant constraints, see [GrantConstraints](http://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) in the *AWS Key Management Service API Reference*\.

The following example policy statement uses the `kms:GrantConstraintType` condition key to allow a user to create grants only when the request includes an `EncryptionContextEquals` grant constraint\. The following example shows a policy statement in a key policy\.

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

### kms:GrantIsForAWSResource<a name="conditions-kms-grant-is-for-aws-resource"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  [kms:GrantIsForAWSResource](#conditions-kms-grant-is-for-aws-resource)  |  Boolean  |  `CreateGrant`  |  IAM and key policies  | 

You can use this condition key to control access to the [CreateGrant](http://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation based on whether the grant is created in the context of an AWS service integrated with AWS KMS\. This condition key is set to `true` when one of the following integrated services is used to create the grant:

+ Amazon Elastic Block Store \(Amazon EBS\) – For more information, see [How Amazon Elastic Block Store \(Amazon EBS\) Uses AWS KMS](services-ebs.md)\.

+ Amazon Relational Database Service \(Amazon RDS\) – For more information, see [How Amazon Relational Database Service \(Amazon RDS\) Uses AWS KMS](services-rds.md)\.

+ Amazon Redshift – For more information, see [How Amazon Redshift Uses AWS KMS](services-redshift.md)\.

+ AWS Certificate Manager \(ACM\) – For more information, see [ACM Private Key Security](http://docs.aws.amazon.com/acm/latest/userguide/kms.html) in the *AWS Certificate Manager User Guide*\.

For example, the following policy statement uses the `kms:GrantIsForAWSResource` condition key to allow a user to create grants only through one of the integrated services in the preceding list\. It does not allow the user to create grants directly\. The following example shows a policy statement in a key policy\.

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

### kms:GrantOperations<a name="conditions-kms-grant-operations"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  [kms:GrantOperations](#conditions-kms-grant-operations)  |  String  |  `CreateGrant`  |  IAM and key policies  | 

You can use this condition key to control access to the [CreateGrant](http://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation based on the grant operations in the request\. For example, you can allow a user to create grants that delegate permission to encrypt but not decrypt\.

The following example policy statement uses the `kms:GrantOperations` condition key to allow a user to create grants that delegate permission to encrypt and to re\-encrypt when this CMK is the destination CMK\. The following example shows a policy statement in a key policy\.

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

### kms:ReEncryptOnSameKey<a name="conditions-kms-reencrypt-on-same-key"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  [kms:ReEncryptOnSameKey](#conditions-kms-reencrypt-on-same-key)  |  Boolean  |  `ReEncrypt`  |  IAM and key policies  | 

You can use this condition key to control access to the [ReEncrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) operation based on whether the request specifies a destination CMK that is the same one used for the original encryption\. For example, the following policy statement uses the `kms:ReEncryptOnSameKey` condition key to allow a user to re\-encrypt only when the destination CMK is the same one used for the original encryption\. The following example shows a policy statement in a key policy\.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": "ReEncrypt*",
  "Resource": "*",
  "Condition": {
    "Bool": {
      "kms:ReEncryptOnSameKey": true
    }
  }
}
```

### kms:ViaService<a name="conditions-kms-via-service"></a>


| AWS KMS Condition Keys | Condition Type | API Operations | Policy Type | 
| --- | --- | --- | --- | 
|  [kms:ViaService](#conditions-kms-via-service)  |  String  |  The `kms:ViaService` condition key exists for all AWS KMS operations *except* for these: `CreateKey`, `GenerateRandom`, `ListAliases`, `ListKeys`, `ListRetirableGrants`, `RetireGrant`\.  |  IAM and key policies  | 

You can use a `kms:ViaService` condition key to control access to a CMK when the CMK is used in the context of an AWS service integrated with AWS KMS\. For example, you can allow a user to use a CMK only through one particular integrated service\. To do this, use the `kms:ViaService` condition key with the KMS ViaService Name for the service\. You can specify one or more services in each `kms:ViaService` condition key\.

**Important**  
When you use the kms:ViaService condition key, verify that the principals have the following permissions:  
Permission to use the CMK\. The principal needs to grant these permissions to the integrated service so it can use the CMK on its behalf\. For more information, see [How AWS Services use AWS KMS](service-integration.md)\.
Permission to use the integrated service\. For details about giving users access to an AWS service that integrates with AWS KMS, consult the documentation for the integrated service\.

The following table shows the KMS ViaService Name for each service that supports the `kms:ViaService` condition key\. The services in this table might not be available in all regions\.


**Services that support the `kms:ViaService` condition key**  

| Service Name | KMS ViaService Name | 
| --- | --- | 
| AWS Certificate Manager \(ACM\) | acm\.AWS\_region\.amazonaws\.com | 
| AWS CodeCommit | codecommit\.AWS\_region\.amazonaws\.com | 
| Amazon Connect | connect\.AWS\_region\.amazonaws\.com | 
| AWS Database Migration Service \(AWS DMS\) | dms\.AWS\_region\.amazonaws\.com | 
| Amazon EC2 Systems Manager | ssm\.AWS\_region\.amazonaws\.com | 
| Amazon Elastic Block Store \(Amazon EBS\) | ec2\.AWS\_region\.amazonaws\.com \(EBS only\) | 
| Amazon Elastic File System | elasticfilesystem\.AWS\_region\.amazonaws\.com | 
| Amazon Kinesis | kinesis\.AWS\_region\.amazonaws\.com | 
| AWS Lambda | lambda\.AWS\_region\.amazonaws\.com | 
| Amazon Lex | lex\.AWS\_region\.amazonaws\.com | 
| Amazon Lightsail | lightsail\.AWS\_region\.amazonaws\.com | 
| Amazon Redshift | redshift\.AWS\_region\.amazonaws\.com | 
| Amazon Relational Database Service \(Amazon RDS\) | rds\.AWS\_region\.amazonaws\.com | 
| Amazon Simple Email Service \(Amazon SES\) | ses\.AWS\_region\.amazonaws\.com | 
| Amazon Simple Storage Service \(Amazon S3\) | s3\.AWS\_region\.amazonaws\.com | 
| AWS Snowball | importexport\.AWS\_region\.amazonaws\.com | 
| Amazon SQS | sqs\.AWS\_region\.amazonaws\.com | 
| Amazon WorkMail | workmail\.AWS\_region\.amazonaws\.com | 
| Amazon WorkSpaces | workspaces\.AWS\_region\.amazonaws\.com | 

The following example shows a policy statement from a key policy\. The policy statement uses the `kms:ViaService` condition key to allow access to a set of AWS KMS actions only when the CMK is used through Amazon EBS or Amazon RDS in the US West \(Oregon\) region\. 

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