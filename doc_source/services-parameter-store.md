# How AWS Systems Manager Parameter Store uses AWS KMS<a name="services-parameter-store"></a>

With AWS Systems Manager Parameter Store, you can create [secure string parameters](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-about.html#sysman-paramstore-securestring), which are parameters that have a plaintext parameter name and an encrypted parameter value\. Parameter Store uses AWS KMS to encrypt and decrypt the parameter values of secure string parameters\.

With [Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-paramstore.html) you can create, store, and manage data as parameters with values\. You can create a parameter in Parameter Store and use it in multiple applications and services subject to policies and permissions that you design\. When you need to change a parameter value, you change one instance, rather than managing error\-prone changes to numerous sources\. Parameter Store supports a hierarchical structure for parameter names, so you can qualify a parameter for specific uses\. 

To manage sensitive data, you can create secure string parameters\. Parameter Store uses AWS KMS customer master keys \(CMKs\) to encrypt the parameter values of secure string parameters when you create or change them\. It also uses CMKs to decrypt the parameter values when you access them\. You can use the [AWS managed CMK](concepts.md#aws-managed-cmk) that Parameter Store creates for your account or specify your own [customer managed CMK](concepts.md#customer-cmk)\. 

**Important**  
Parameter Store supports only [symmetric CMKs](symm-asymm-concepts.md#symmetric-cmks)\. You cannot use an [asymmetric CMK](symm-asymm-concepts.md#asymmetric-cmks) to encrypt your parameters\. For help determining whether a CMK is symmetric or asymmetric, see [Identifying symmetric and asymmetric CMKs](find-symm-asymm.md)\.

Parameter Store supports two tiers of secure string parameters: *standard* and *advanced*\. Standard parameters, which cannot exceed 4096 bytes, are encrypted and decrypted directly under the CMK that you specify\. To encrypt and decrypt advanced secure string parameters, Parameter Store uses envelope encryption with the [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/)\. You can convert a standard secure string parameter to an advanced parameter, but you cannot convert an advanced parameter to a standard one\. For more information about the difference between standard and advanced secure string parameters, see [About Systems Manager Advanced Parameters](https://docs.aws.amazon.com/systems-manager/latest/userguide/parameter-store-advanced-parameters.html) in the AWS Systems Manager User Guide\.

**Topics**
+ [Protecting standard secure string parameters](#parameter-store-encrypt)
+ [Protecting advanced secure string parameters](#parameter-store-advanced-encrypt)
+ [Setting permissions to encrypt and decrypt parameter values](#parameter-store-policies)
+ [Parameter Store encryption context](#parameter-store-encryption-context)
+ [Troubleshooting CMK issues in Parameter Store](#parameter-store-cmk-fail)

## Protecting standard secure string parameters<a name="parameter-store-encrypt"></a>

Parameter Store does not perform any cryptographic operations\. Instead, it relies on AWS KMS to encrypt and decrypt secure string parameter values\. When you create or change a standard secure string parameter value, Parameter Store calls the AWS KMS [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation\. This operation uses a symmetric AWS KMS CMK directly to encrypt the parameter value instead of using the CMK to generate a [data key](concepts.md#data-keys)\. 

You can select the CMK that Parameter Store uses to encrypt the parameter value\. If you do not specify a CMK, Parameter Store uses the AWS managed CMK that Systems Manager automatically creates in your account\. This CMK has the `aws/ssm` alias\.

To view the default `aws/ssm` CMK for your account, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation in the AWS KMS API\. The following example uses the `describe-key` command in the AWS Command Line Interface \(AWS CLI\) with the `aws/ssm` alias name\.

```
aws kms describe-key --key-id alias/aws/ssm
```

To create a standard secure string parameter, use the [PutParameter](https://docs.aws.amazon.com/systems-manager/latest/APIReference/API_PutParameter.html) operation in the Systems Manager API\. Omit the `Tier` parameter or specify a value of `Standard`, which is the default\. Include a `Type` parameter with a value of `SecureString`\. To specify an AWS KMS CMK, use the `KeyId` parameter\. The default is the AWS managed CMK for your account, `aws/ssm`\. 

Parameter Store then calls the AWS KMS `Encrypt` operation with the CMK and the plaintext parameter value\. AWS KMS returns the encrypted parameter value, which Parameter Store stores with the parameter name\.

The following example uses the Systems Manager [put\-parameter](https://docs.aws.amazon.com/cli/latest/reference/ssm/put-parameter.html) command and its `--type` parameter in the AWS CLI to create a secure string parameter\. Because the command omits the optional `--tier` and `--key-id` parameters, Parameter Store creates a standard secure string parameter and encrypts it under the AWS managed CMK\.

```
aws ssm put-parameter --name MyParameter --value "secret_value" --type SecureString
```

The following similar example uses the `--key-id` parameter to specify a [customer managed CMK](concepts.md#customer-cmk)\. The example uses a CMK ID to identify the CMK, but you can use any valid CMK identifier\. Because the command omits the `Tier` parameter \(`--tier`\), Parameter Store creates a standard secure string parameter, not an advanced one\.

```
aws ssm put-parameter --name param1 --value "secret" --type SecureString --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
```

When you get a secure string parameter from Parameter Store, its value is encrypted\. To get a parameter, use the [GetParameter ](https://docs.aws.amazon.com/systems-manager/latest/APIReference/API_GetParameter.html) operation in the Systems Manager API\.

The following example uses the Systems Manager [get\-parameter](https://docs.aws.amazon.com/cli/latest/reference/ssm/get-parameter.html) command in the AWS CLI to get the `MyParameter` parameter from Parameter Store without decrypting its value\.

```
$  aws ssm get-parameter --name MyParameter

{
    "Parameter": {
        "Type": "SecureString", 
        "Name": "MyParameter", 
        "Value": "AQECAHgnOkMROh5LaLXkA4j0+vYi6tmM17Lg/9E464VRo68cvwAAAG8wbQYJKoZIhvcNAQcGoGAwXgIBADBZBgkqhkiG9w0BBwEwHgYJYZZIAWUDBAEuMBEEDImYOw44gna0Jm00hAIBEIAsjgr7mum1EnnXzE3xM8bGle0oKYcfVCHtBkfjIeZGTgL6Hg0fSDnpMHdcSXY="
    }
}
```

To decrypt the parameter value before returning it, set the `WithDecryption` parameter of `GetParameter` to `true`\. When you use `WithDecryption`, Parameter Store calls the AWS KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation on your behalf to decrypt the parameter value\. As a result, the `GetParameter` request returns the parameter with a plaintext parameter value, as shown in the following example\.

```
$  aws ssm get-parameter --name MyParameter --with-decryption

{
    "Parameter": {
        "Type": "SecureString", 
        "Name": "MyParameter", 
        "Value": "secret_value"
    }
}
```

The following workflow shows how Parameter Store uses an AWS KMS CMK to encrypt and decrypt a standard secure string parameter\.

### Encrypt a standard parameter<a name="param-store-standard-encrypt"></a>

1. When you use `PutParameter` to create a secure string parameter, Parameter Store sends an `Encrypt` request to AWS KMS\. That request includes the plaintext parameter value, the CMK that you chose, and the [Parameter Store encryption context](#parameter-store-encryption-context)\. During transmission to AWS KMS, the plaintext value in the secure string parameter is protected by Transport Layer Security \(TLS\)\.

1. AWS KMS encrypts the parameter value with the specified CMK and encryption context\. It returns the ciphertext to Parameter Store, which stores the parameter name and its encrypted value\.  
![\[Encrypting a standard secure string parameter value\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/service-pstore-standard.png)

### Decrypt a standard parameters<a name="param-store-standard-decrypt"></a>

1. When you include the `WithDecryption` parameter in a `GetParameter` request, Parameter Store sends a `Decrypt` request to AWS KMS with the encrypted secure string parameter value and the [Parameter Store encryption context](#parameter-store-encryption-context)\.

1. AWS KMS uses the same CMK and the supplied encryption context to decrypt the encrypted value\. It returns the plaintext \(decrypted\) parameter value to Parameter Store\. During transmission, the plaintext data is protected by TLS\.

1. Parameter Store returns the plaintext parameter value to you in the `GetParameter` response\.

## Protecting advanced secure string parameters<a name="parameter-store-advanced-encrypt"></a>

When you use `PutParameter` to create an advanced secure string parameter, Parameter Store uses [envelope encryption](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/how-it-works.html#envelope-encryption) with the AWS Encryption SDK and a symmetric AWS KMS customer master key \(CMK\) to protect the parameter value\. Each advanced parameter value is encrypted under a unique data key, and the data key is encrypted under an AWS KMS CMK\. You can use the [AWS managed CMK](concepts.md#aws-managed-cmk) for the account \(`aws/ssm`\) or any customer managed CMK\.

The [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/) is an open\-source, client\-side library that helps you to encrypt and decrypt data using industry standards and best practices\. It's supported on multiple platforms and in multiple programming languages, including a command\-line interface\. You can view the source code and contribute to its development in GitHub\. 

For each secure string parameter value, Parameter Store calls the AWS Encryption SDK to encrypt the parameter value using a unique data key that AWS KMS generates \([GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)\)\. The AWS Encryption SDK returns to Parameter Store an [encrypted message](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/concepts.html#message) that includes the encrypted parameter value and an encrypted copy of the unique data key\. Parameter Store stores the entire encrypted message in the secure string parameter value\. Then, when you get an advanced secure string parameter value, Parameter Store uses the AWS Encryption SDK to decrypt the parameter value\. This requires a call to AWS KMS to decrypt the encrypted data key\.

To create an advanced secure string parameter, use the [PutParameter](https://docs.aws.amazon.com/systems-manager/latest/APIReference/API_PutParameter.html) operation in the Systems Manager API\. Set the value of `Tier` parameter to `Advanced`\. Include a `Type` parameter with a value of `SecureString`\. To specify an AWS KMS CMK, use the `KeyId` parameter\. The default is the AWS managed CMK for your account, `aws/ssm`\. 

```
aws ssm put-parameter --name MyParameter --value "secret_value" --type SecureString --tier Advanced
```

The following similar example uses the `--key-id` parameter to specify a [customer managed CMK](concepts.md#customer-cmk)\. The example uses the Amazon Resource Name \(ARN\) of the CMK, but you can use any valid CMK identifier\. 

```
aws ssm put-parameter --name MyParameter --value "secret_value" --type SecureString --tier Advanced --key-id arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
```

When you get a secure string parameter from Parameter Store, its value is the encrypted message that the AWS Encryption SDK returned\. To get a parameter, use the [GetParameter ](https://docs.aws.amazon.com/systems-manager/latest/APIReference/API_GetParameter.html) operation in the Systems Manager API\.

The following example uses the Systems Manager `GetParameter` operation to get the `MyParameter` parameter from Parameter Store without decrypting its value\.

```
$  aws ssm get-parameter --name MyParameter

{
    "Parameter": {
        "Type": "SecureString", 
        "Name": "MyParameter", 
        "Value": "AQECAHgnOkMROh5LaLXkA4j0+vYi6tmM17Lg/9E464VRo68cvwAAAG8wbQYJKoZIhvcNAQcGoGAwXgIBADBZBgkqhkiG9w0BBwEwHgYJYZZIAWUDBAEuMBEEDImYOw44gna0Jm00hAIBEIAsjgr7mum1EnnXzE3xM8bGle0oKYcfVCHtBkfjIeZGTgL6Hg0fSDnpMHdcSXY="
    }
}
```

To decrypt the parameter value before returning it, set the `WithDecryption` parameter of `GetParameter` to `true`\. When you use `WithDecryption`, Parameter Store calls the AWS KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation on your behalf to decrypt the parameter value\. As a result, the `GetParameter` request returns the parameter with a plaintext parameter value, as shown in the following example\.

```
$  aws ssm get-parameter --name MyParameter --with-decryption

{
    "Parameter": {
        "Type": "SecureString", 
        "Name": "MyParameter", 
        "Value": "secret_value"
    }
}
```

You cannot convert an advanced secure string parameter to a standard one, but you can convert a standard secure string to an advanced one\. To convert a standard secure string parameter to an advanced secure string, use the `PutParameter` operation with the `Overwrite` parameter\. The `Type` must be `SecureString` and the `Tier` value must be `Advanced`\. The `KeyId` parameter, which identifies a customer managed CMK, is optional\. If you omit it, Parameter Store uses the AWS managed CMK for the account\. You can specify any CMK that the principal has permission to use, even if you used a different CMK to encrypt the standard parameter\.

When you use the `Overwrite` parameter, Parameter Store uses the AWS Encryption SDK to encrypt the parameter value\. Then it stores the newly encrypted message in Parameter Store\.

```
$ aws ssm put-parameter --name myStdParameter --value "secret_value"  --type SecureString --tier Advanced --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --overwrite
```

The following workflow shows how Parameter Store uses an AWS KMS CMK to encrypt and decrypt an advanced secure string parameter\.

### Encrypt an advanced parameter<a name="param-store-advanced-encrypt"></a>

1. When you use `PutParameter` to create an advanced secure string parameter, Parameter Store uses the AWS Encryption SDK and AWS KMS to encrypt the parameter value\. Parameter Store calls the AWS Encryption SDK with the parameter value, the AWS KMS CMK that you specified, and the [Parameter Store encryption context](#parameter-store-encryption-context)\.

1. The AWS Encryption SDK sends a [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) request to AWS KMS with the identifier of the CMK that you specified and the Parameter Store encryption context\. AWS KMS returns two copies of the unique data key: one in plaintext and one encrypted under the CMK\. \(The encryption context is used when encrypting the data key\.\)

1. The AWS Encryption SDK uses the plaintext data key to encrypt the parameter value\. It returns an [encrypted message](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/concepts.html#message) that includes the encrypted parameter value, the encrypted data key, and other data, including the Parameter Store encryption context\.

1. Parameter Store stores the encrypted message as the parameter value\.  
![\[Encrypting an advanced secure string parameter value\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/service-pstore-advanced.png)

### Decrypt an advanced parameter<a name="param-store-advanced-decrypt"></a>

1. You can include the `WithDecryption` parameter in a `GetParameter` request to get an advanced secure string parameter\. When you do, Parameter Store passes the [encrypted message](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/concepts.html#message) from the parameter value to a decryption method of the AWS Encryption SDK\.

1. The AWS Encryption SDK calls the KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. It passes in the encrypted data key and the Parameter Store encryption context from the encrypted message\.

1. AWS KMS uses the CMK and the Parameter Store encryption context to decrypt the encrypted data key\. Then it returns the plaintext \(decrypted\) data key to the AWS Encryption SDK\.

1. The AWS Encryption SDK uses the plaintext data key to decrypt the parameter value\. It returns the plaintext parameter value to Parameter Store\. 

1. Parameter Store verifies the encryption context and returns the plaintext parameter value to you in the `GetParameter` response\.

## Setting permissions to encrypt and decrypt parameter values<a name="parameter-store-policies"></a>

To encrypt a standard secure string parameter value, the user needs `kms:Encrypt` permission\. To encrypt an advanced secure string parameter value, the user needs `kms:GenerateDataKey` permission\. To decrypt either type of secure string parameter value, the user needs `kms:Decrypt` permission\. 

You can use IAM policies to allow or deny permission for a user to call the Systems Manager `PutParameter` and `GetParameter` operations\.

If you are using customer managed CMKs to encrypt your secure string parameter values, you can use IAM policies and key policies to manage encrypt and decrypt permissions\. However, you cannot establish access control policies for the default `aws/ssm` CMK\. For detailed information about controlling access to customer managed CMKs, see [Authentication and access control for AWS KMS](control-access.md)\.

The following example shows an IAM policy designed for standard secure string parameters\. It allows the user to call the Systems Manager `PutParameter` operation on all parameters in the `FinancialParameters` path\. The policy also allows the user to call the AWS KMS `Encrypt` operation on an example customer managed CMK\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssm:PutParameter"
            ],
            "Resource": "arn:aws:ssm:us-west-2:111122223333:parameter/FinancialParameters/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "kms:Encrypt"
            ],
            "Resource": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
        }
    ]
}
```

The next example shows an IAM policy that is designed for advanced secure string parameters\. It allows the user to call the Systems Manager `PutParameter` operation on all parameters in the `ReservedParameters` path\. The policy also allows the user to call the AWS KMS `GenerateDataKey` operation on an example customer managed CMK\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssm:PutParameter"
            ],
            "Resource": "arn:aws:ssm:us-west-2:111122223333:parameter/ReservedParameters/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "kms:GenerateDataKey"
            ],
            "Resource": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
        }
    ]
}
```

The final example also shows an IAM policy that can be used for standard or advanced secure string parameters\. It allows the user to call the Systems Manager `GetParameter` operations \(and related operations\) on all parameters in the `ITParameters` path\. The policy also allows the user to call the AWS KMS `Decrypt` operation on an example customer managed CMK\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssm:GetParameter*"
            ],
            "Resource": "arn:aws:ssm:us-west-2:111122223333:parameter/ITParameters/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "kms:Decrypt"
            ],
            "Resource": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
        }
    ]
}
```

## Parameter Store encryption context<a name="parameter-store-encryption-context"></a>

An *encryption context* is a set of key–value pairs that contain arbitrary nonsecret data\. When you include an encryption context in a request to encrypt data, AWS KMS cryptographically binds the encryption context to the encrypted data\. To decrypt the data, you must pass in the same encryption context\. 

You can also use the encryption context to identify a cryptographic operation in audit records and logs\. The encryption context appears in plaintext in logs, such as [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) logs\. 

The AWS Encryption SDK also takes an encryption context, although it handles it differently\. Parameter Store supplies the encryption context to the encryption method\. The AWS Encryption SDK cryptographically binds the encryption context to the encrypted data\. It also includes the encryption context in plain text in the header of the encrypted message that it returns\. However, unlike AWS KMS, the AWS Encryption SDK decryption methods do not take an encryption context as input\. Instead, when it decrypts data, the AWS Encryption SDK gets the encryption context from the encrypted message\. Parameter Store verifies that the encryption context includes the value that it expects before returning the plaintext parameter value to you\. 

Parameter Store uses the following encryption context in its cryptographic operations:
+ Key: `PARAMETER_ARN`
+ Value: The Amazon Resource Name \(ARN\) of the parameter that is being encrypted\. 

The format of the encryption context is as follows:

```
"PARAMETER_ARN":"arn:aws:ssm:<REGION_NAME>:<ACCOUNT_ID>:parameter/<parameter-name>"
```

For example, Parameter Store includes this encryption context in calls to encrypt and decrypt the `MyParameter` parameter in an example AWS account and region\.

```
"PARAMETER_ARN":"arn:aws:ssm:us-west-2:111122223333:parameter/MyParameter"
```

If the parameter is in a Parameter Store hierarchical path, the path and name are included in the encryption context\. For example, this encryption context is used when encrypting and decrypting the `MyParameter` parameter in the `/ReadableParameters` path in an example AWS account and region\.

```
"PARAMETER_ARN":"arn:aws:ssm:us-west-2:111122223333:parameter/ReadableParameters/MyParameter"
```

You can decrypt an encrypted secure string parameter value by calling the AWS KMS `Decrypt` operation with the correct encryption context and the encrypted parameter value that the Systems Manager `GetParameter` operation returns\. However, we encourage you to decrypt Parameter Store parameter values by using the `GetParameter` operation with the `WithDecryption` parameter\. 

You can also include the encryption context in an IAM policy\. For example, you can permit a user to decrypt only one particular parameter value or set of parameter values\.

The following example IAM policy statement allows the user to the get value of the `MyParameter` parameter and to decrypt its value using the specified CMK\. However the permissions apply only when the encryption context matches specified string\. These permissions do not apply to any other parameter or CMK, and the call to `GetParameter` fails if the encryption context does not match the string\.

Before using a policy statement like this one, replace the example ARNs with valid values\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssm:GetParameter*"
            ],
            "Resource": "arn:aws:ssm:us-west-2:111122223333:parameter/MyParameter",
        },
        {
            "Effect": "Allow",
            "Action": [
                "kms:Decrypt"
            ],
            "Resource": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "Condition": {
                "StringEquals": {
                    "kms:EncryptionContext:PARAMETER_ARN":"arn:aws:ssm:us-west-2:111122223333:parameter/MyParameter"
                }
            }
        }
    ]
}
```

## Troubleshooting CMK issues in Parameter Store<a name="parameter-store-cmk-fail"></a>

To perform any operation on a secure string parameter, Parameter Store must be able to use the AWS KMS CMK that you specify for your intended operation\. Most of the Parameter Store failures related to CMKs are caused by the following problems:
+ The credentials that an application is using do not have permission to perform the specified action on the CMK\. 

  To fix this error, run the application with different credentials or revise the IAM or key policy that is preventing the operation\. For help with AWS KMS IAM and key policies, see [Authentication and access control for AWS KMS](control-access.md)\.
+ The CMK is not found\. 

  This typically happens when you use an incorrect identifier for the CMK\. [Find the correct identifiers](find-cmk-id-arn.md) for the CMK and try the command again\. 
+ The CMK is not enabled\. When this occurs, Parameter Store returns an InvalidKeyId exception with a detailed error message from AWS KMS\. If the CMK state is `Disabled`, [enable it](enabling-keys.md)\. If it is `Pending Import`, complete the [import procedure](importing-keys.md)\. If the key state is `Pending Deletion`, [cancel the key deletion](deleting-keys.md#deleting-keys-scheduling-key-deletion) or use a different CMK\. 

  To find the [key state](key-state.md) of a CMK in the AWS KMS console, on the **Customer managed keys** or **AWS managed keys** page, see the [Status column](viewing-keys-console.md)\. To use the AWS KMS API to find the status of a CMK, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. 