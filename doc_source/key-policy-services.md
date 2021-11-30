# Permissions for AWS services in key policies<a name="key-policy-services"></a>

Many AWS services use AWS KMS keys to protect the resources they manage\. When a service uses [AWS owned keys](concepts.md#aws-owned-cmk) or [AWS managed keys](concepts.md#aws-managed-cmk), the service establishes and maintains the key policies for these KMS keys\. 

However, when you use a [customer managed key](concepts.md#customer-cmk) with an AWS service, you set and maintain the key policy\. That key policy must allow the service the minimum permissions that it requires to protect the resource on your behalf\. We recommend that you follow the principle of least privilege: give the service only the permissions that it requires\. You can do this effectively by learning which permissions the service needs and using [AWS global condition keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) and [AWS KMS condition keys](policy-conditions.md) to refine the permissions\. 

To find the permissions that the service requires on a customer managed key, see the encryption documentation for the service\. For example, for the permissions that Amazon Elastic Block Store \(Amazon EBS\) requires, see *Permissions for IAM users* in the [Amazon EC2 User Guide for Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#ebs-encryption-permissions) and [Amazon EC2 User Guide for Windows Instances](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/EBSEncryption.html#ebs-encryption-permissions)\. For the permissions that Secrets Manager requires, see [Authorizing use of the KMS key](https://docs.aws.amazon.com/secretsmanager/latest/userguide/security-encryption.html#security-encryption-authz) in the *AWS Secrets Manager User Guide*\.

## Implementing least privileged permissions<a name="key-policy-least-privilege"></a>

When you give an AWS service permission to use a KMS key, ensure that the permission is valid only for the resources that the service must access on your behalf\. This least privilege strategy helps to prevent unauthorized use of a KMS key when requests are passed between AWS services\.

To implement a least privilege strategy, use we recommend using AWS KMS encryption context condition keys and the global source ARN or source account condition keys\.

### Using encryption context condition keys<a name="least-privilege-encryption-context"></a>

The most effective way to implement least privileged permissions when using AWS KMS resources is to include the [kms:EncryptionContext:*context\-key*](policy-conditions.md#conditions-kms-encryption-context) or [kms:EncryptionContextKeys](policy-conditions.md#conditions-kms-encryption-context-keys) condition keys in the policy that allows principals to call AWS KMS cryptographic operations\. These condition keys are particularly effective because they associate the permission with the [encryption context](concepts.md#encrypt_context) that is bound to the ciphertext when the resource is encrypted\. 

Use encryption context conditions keys only when the action in the policy statement is [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) or an AWS KMS symmetric cryptographic operation that takes an `EncryptionContext` parameter, such as the operations like [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) or [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)\. \(For a list of supported operations, see [kms:EncryptionContext:*context\-key*](policy-conditions.md#conditions-kms-encryption-context) or [kms:EncryptionContextKeys](policy-conditions.md#conditions-kms-encryption-context-keys)\.\) If you use these condition keys to allow other operations, such as [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html), permission will be denied\.

Set the value to the encryption context that the service uses when it encrypts the resource\. This information is typically available in the Security chapter of the service documentation\. For example, the [encryption context for AWS Proton](https://docs.aws.amazon.com/proton/latest/adminguide/data-protection.html#encryption-context) identifies the AWS Proton resource and its associated template\. The [AWS Secrets Manager encryption context](https://docs.aws.amazon.com/secretsmanager/latest/userguide/security-encryption.html#security-encryption-encryption-context) identifies the secret and its version\. The [encryption context for Amazon Location](https://docs.aws.amazon.com/location/latest/developerguide/encryption-at-rest.html#location-encryption-context) identifies the tracker or collection\. 

The following example key policy statement allows Amazon Location Service to create grants on behalf of authorized users\. This policy statement limits the permission by using the [kms:ViaService](policy-conditions.md#conditions-kms-via-service), [kms:CallerAccount](policy-conditions.md#conditions-kms-caller-account), and `kms:EncryptionContext:context-key` condition keys to tie the permission to a particular tracker resource\.

```
{
  "Sid": "Allow Amazon Location to create grants on behalf of authorized users",
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/LocationTeam"
  },
  "Action": "kms:CreateGrant",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:ViaService": "geo.us-west-2.amazonaws.com",
      "kms:CallerAccount": "111122223333",
      "kms:EncryptionContext:aws:geo:arn": "arn:aws:geo:us-west-2:111122223333:tracker/SAMPLE-Tracker"
    }
  }
}
```

### Using `aws:SourceArn` or `aws:SourceAccount` condition keys<a name="least-privilege-source-arn"></a>

When the principal in a key policy statement is an [AWS service principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services), we strongly recommend that you use the [https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourcearn](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourcearn) or [https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourceaccount](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourceaccount) global condition keys, in addition to the `kms:EncryptionContext:context-key` condition key\. The ARN and account values are included in the authorization context only when a request comes to AWS KMS from another AWS service\. This combination of conditions implements least privileged permissions and avoids a potential [confused deputy scenario](https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html)\. Service principals are not typically used as principals in a key policy, but some AWS services, such as AWS CloudTrail, require it\. 

To use the `aws:SourceArn` or `aws:SourceAccount` global condition keys, set the value to the Amazon Resource Name \(ARN\) or account of the resource that is being encrypted\. For example, in a key policy statement that gives AWS CloudTrail permission to encrypt a trail, set the value of `aws:SourceArn` to the ARN of the trail\. Whenever possible, use `aws:SourceArn`, which is more specific\. Set the value to the ARN or an ARN pattern with wildcard characters\. If you don't know the ARN of the resource, use `aws:SourceAccount` instead\. 

In the following example key policy, the principal who gets the permissions is the AWS CloudTrail service principal, `cloudtrail.amazonaws.com`\. To implement least privilege, this policy uses the `aws:SourceArn` and `kms:EncryptionContext:context-key` condition keys\. The policy statement allows CloudTrail to use the KMS key to [generate the data key](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) that it uses to encrypt a trail\. The `aws:SourceArn` and `kms:EncryptionContext:context-key` conditions are evaluated independently\. Any request to use the KMS key for the specified operation must satisfy both conditions\.

To restrict the service's permission to the `finance` trail in the example account \(111122223333\) and `us-west-2` Region, this policy statement sets the `aws:SourceArn` condition key to the ARN of a particular trail\. The condition statement uses the [ArnEquals](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_ARN) operator to ensure that every element in the ARN is evaluated independently when matching\. The example also uses the `kms:EncryptionContext:context-key` condition key to limit the permission to trails in a particular account and Region\. 

Before using this key policy, replace the example account ID, Region, and trail name with valid values from your account\. 

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Allow CloudTrail to encrypt logs",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Action": "kms:GenerateDataKey",
      "Resource": "*",
      "Condition": {
        "ArnEquals": {
          "aws:SourceArn": [
            "arn:aws:cloudtrail:us-west-2:111122223333:trail/finance"
          ]
        },
        "StringLike": {
          "kms:EncryptionContext:aws:cloudtrail:arn": [
            "arn:aws:cloudtrail:*:111122223333:trail/*"
          ]
        }
      }
    }
  ]
}
```