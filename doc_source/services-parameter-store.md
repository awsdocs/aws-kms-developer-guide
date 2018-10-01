# How AWS Systems Manager Parameter Store Uses AWS KMS<a name="services-parameter-store"></a>

With AWS Systems Manager Parameter Store, you can create [Secure String parameters](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-about.html#sysman-paramstore-securestring), which are parameters that have a plaintext parameter name and an encrypted parameter value\. Parameter Store uses AWS KMS to encrypt and decrypt the parameter values of Secure String parameters

With [Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-paramstore.html) you can create, store, and manage data as parameters with values\. You can create a parameter in Parameter Store and use it in multiple applications and services subject to policies and permissions that you design\. When you need to change a parameter value, you change one instance, rather than managing an error\-prone change to numerous sources\. Parameter Store supports a hierarchical structure for parameter names, so you can qualify a parameter for specific uses\. 

To manage sensitive data, you can create Secure String parameters\. Parameter Store uses AWS KMS customer master keys \(CMKs\) to encrypt the parameter values of Secure String parameters when you create or change them\. It also uses CMKs to decrypt the parameter values when you access them\. You can use the default CMK that Parameter Store creates for your account or specify your own customer managed CMK\.

**Topics**
+ [Encrypting and Decrypting Secure String Parameters](#parameter-store-encrypt)
+ [Setting Permissions to Encrypt and Decrypt Parameter Values](#parameter-store-policies)
+ [Parameter Store Encryption Context](#parameter-store-encryption-context)
+ [Troubleshooting CMK Issues in Parameter Store](#parameter-store-cmk-fail)

## Encrypting and Decrypting Secure String Parameters<a name="parameter-store-encrypt"></a>

Parameter Store does not perform any cryptographic operations\. Instead, it relies on AWS KMS to encrypt and decrypt Secure String parameter values\. When you create or change a Secure String parameter value, Parameter Store calls the AWS KMS [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) API operation\. This operation uses an AWS KMS CMK directly to encrypt the parameter value instead of using the CMK to generate a [data key](concepts.md#data-keys)\. 

You can select the CMK that Parameter Store uses to encrypt the parameter value\. If you do not specify a CMK, Parameter Store uses the default `aws/ssm` CMK that Systems Manager automatically creates in your account\.

To view the default `aws/ssm` CMK for your account, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation in the AWS KMS API\. The following example uses the `describe-key` command in the AWS Command Line Interface \(AWS CLI\) with the `aws/ssm` alias name\.

```
aws kms describe-key --key-id alias/aws/ssm
```

To create a Secure String parameter, use the [PutParameter](https://docs.aws.amazon.com/systems-manager/latest/APIReference/API_PutParameter.html) operation in the Systems Manager API\. Include a `Type` parameter with a value of `SecureString`\. To specify an AWS KMS CMK, use the `KeyId` parameter\. The default is the default `aws/ssm` CMK for your account\. 

Parameter Store then calls the AWS KMS `Encrypt` API with the CMK and the plaintext parameter value\. AWS KMS returns the encrypted parameter value, which Parameter Store stores with the parameter name\.

The following example uses the Systems Manager [put\-parameter](https://docs.aws.amazon.com/cli/latest/reference/ssm/put-parameter.html) command and its `--type` parameter in the AWS CLI to create a Secure String parameter\. Because the command omits the optional `--key-id` parameter, Parameter Store uses the default `aws/ssm` CMK to encrypt the parameter value\.

```
aws ssm put-parameter --name MyParameter --value "secret_value" --type SecureString
```

The following similar example uses the `--key-id` parameter to specify a customer managed CMK\. The example uses a CMK ID to identify the CMK, but you can use any valid CMK identifier\.

```
aws ssm put-parameter --name param1 --value "secret" --type SecureString --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
```

When you get a Secure String parameter from Parameter Store, its value is encrypted\. To get a parameter, use the [GetParameter ](https://docs.aws.amazon.com/systems-manager/latest/APIReference/API_GetParameter.html) operation in the Systems Manager API\.

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

To decrypt the parameter value before returning it, set the `WithDecryption` parameter of `GetParameter` to `true`\. When you use `WithDecryption`, Parameter Store calls the AWS KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) API operation on your behalf to decrypt the parameter value\. As a result, the `GetParameter` request returns the parameter with a plaintext parameter value, as shown in the following example\.

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

The following workflow shows how Parameter Store uses an AWS KMS CMK\.

**Encrypt in Put Parameter:**

1. When you create a Secure String parameter, Parameter Store sends an `Encrypt` request to AWS KMS that includes the plaintext parameter value and the CMK that you chose\. During transmission to AWS KMS, the plaintext value in the Secure String parameter is protected by Transport Layer Security \(TLS\)\.

1. AWS KMS encrypts the parameter value with the specified CMK and returns the ciphertext to Parameter Store, which stores the parameter name and its encrypted value\.

**Decrypt in Get Parameter**

1. When you include the `WithDecryption` parameter in a request to get a Secure String parameter, Parameter Store sends a `Decrypt` request to AWS KMS with the encrypted Secure String parameter value\.

1. AWS KMS uses the same CMK to decrypt the encrypted value\. It returns the plaintext \(decrypted\) parameter value to Parameter Store\. During transmission, the plaintext data is protected by TLS\. 

1. Parameter Store returns the plaintext parameter value to you in the `GetParameter` response\.

## Setting Permissions to Encrypt and Decrypt Parameter Values<a name="parameter-store-policies"></a>

You can use IAM policies to allow or deny permission for a user to call the Systems Manager `PutParameter` and `GetParameter` operations\.

Also, if you are using customer managed CMKs, you can use IAM policies and key policies to allow or deny permission to use the CMKs in calls to the AWS KMS `Encrypt` and `Decrypt` operations\. However, you cannot establish access control policies for the default `aws/ssm` CMK\. For detailed information about controlling access to customer managed AWS KMS CMKs, see [Authentication and Access Control for AWS KMS](control-access.md)\.

The following example shows an IAM policy that allows the user to call the Systems Manager `GetParameter` operation on all parameters in the `/ReadableParameters` path\. The policy also allows the user to call the AWS KMS `Decrypt` operation on the specified customer managed CMK\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssm:GetParameter*"
            ],
            "Resource": "arn:aws:ssm:us-west-2:111122223333:parameter/ReadableParameters/*"
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

## Parameter Store Encryption Context<a name="parameter-store-encryption-context"></a>

An *encryption context* is a set of key–value pairs that contain arbitrary nonsecret data\. When you include an encryption context in a request to encrypt data, AWS KMS cryptographically binds the encryption context to the encrypted data\. To decrypt the data, you must pass in the same encryption context\. 

You can also use the encryption context to identify a cryptographic operation in audit records and logs\. The encryption context appears in plaintext in logs, such as [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) logs\. 

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

You can decrypt an encrypted Secure String parameter value by calling the AWS KMS `Decrypt` operation with the correct encryption context and the encrypted parameter value that the Systems Manager `GetParameter` operation returns\. However, we encourage you to use the `GetParameter` operation with the `WithDecryption` parameter to decrypt Parameter Store parameter values\. 

You can also include the encryption context in an IAM policy\. For example, you can permit a user to decrypt only one particular parameter value or set of parameter values\.

The following example IAM policy statement allows the user to the get value of the `MyParameter` parameter and to decrypt its value using the specified CMK\. However the permissions apply only when the encryption context matches specified string\. These permissions do not apply to any other parameter or CMK, and the call to `GetParameter` fails if the encryption context does not match the string\.

Before using a policy statement like this one, replace the example AWS account, region, and parameter name with valid values\.

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

## Troubleshooting CMK Issues in Parameter Store<a name="parameter-store-cmk-fail"></a>

To perform any operation on a Secure String parameter, Parameter Store must be able to use the AWS KMS CMK that you specify for your intended operation\. Most of the Parameter Store failures related to CMKs are caused by the following problems:
+ The credentials that an application is using do not have permission to perform the specified action on the CMK\. 

  To fix this error, run the application with different credentials or revise the IAM or key policy that is preventing the operation\. For help with AWS KMS IAM and key policies, see [Authentication and Access Control for AWS KMS](control-access.md)\.
+ The CMK is not found\. 

  This typically happens when you use an incorrect identifier for the CMK\. [Find the correct identifiers](viewing-keys.md#find-cmk-id-arn) for the CMK and try the command again\. 
+ The CMK is not enabled\. When this occurs, Parameter Store returns an InvalidKeyId exception with a detailed error message from AWS KMS\.

  To find the status of a CMK, use the [Status column](viewing-keys.md#viewing-keys-console) of the **Encryption keys** page of the [IAM console](https://console.aws.amazon.com/iam/home?#home) or the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation in the AWS KMS API\. If the CMK state is disabled, [enable it](enabling-keys.md)\. If it is pending import, you must complete the [import procedure](importing-keys.md)\. If the CMK is pending deletion, you must use a different CMK or [cancel the key deletion](deleting-keys.md#deleting-keys-scheduling-key-deletion)\. 