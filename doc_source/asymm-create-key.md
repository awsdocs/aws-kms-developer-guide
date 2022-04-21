# Creating asymmetric KMS keys<a name="asymm-create-key"></a>

You can create [asymmetric KMS keys](symmetric-asymmetric.md#asymmetric-cmks) in the AWS KMS console, by using the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) API, or by using an [AWS CloudFormation template](creating-resources-with-cloudformation.md)\. An asymmetric KMS key represents a public and private key pair that can be used for encryption or signing\. The private key remains within AWS KMS\. To download the public key for use outside of AWS KMS, see [Downloading public keys](download-public-key.md)\.

When creating a KMS key to encrypt data that you store or manage in an AWS service, use a symmetric encryption KMS key\. AWS services that integrate with AWS KMS do not support asymmetric KMS keys\. For help deciding whether to create a symmetric or asymmetric KMS key, see [Choosing a KMS key type](key-types.md#symm-asymm-choose)\.

For information about the permissions required to create KMS keys, see [Permissions for creating KMS keys](create-keys.md#create-key-permissions)\.

**Topics**
+ [Creating asymmetric KMS keys \(console\)](#create-asymmetric-keys-console)
+ [Creating asymmetric KMS keys \(AWS KMS API\)](#create-asymmetric-keys-api)

## Creating asymmetric KMS keys \(console\)<a name="create-asymmetric-keys-console"></a>

You can use the AWS Management Console to create asymmetric AWS KMS keys \(KMS keys\)\. Each asymmetric KMS key represents a public and private key pair\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Choose **Create key**\.

1. To create an asymmetric KMS key, in **Key type,** choose **Asymmetric**\.

   For information about how to create an symmetric encryption KMS key in the AWS KMS console, see [Creating symmetric encryption KMS keys \(console\)](create-keys.md#create-keys-console)\.

1. To create an asymmetric KMS key for public key encryption, in **Key usage**, choose **Encrypt and decrypt**\. Or, to create an asymmetric KMS key for signing messages and verifying signatures, in **Key usage**, choose **Sign and verify**\.

   For help choosing a key usage value, see [Selecting the key usage](key-types.md#symm-asymm-choose-key-usage)\.

1. Select a specification \(**Key spec**\) for your asymmetric KMS key\. 

   Often the key spec that you select is determined by regulatory, security, or business requirements\. It might also be influenced by the size of messages that you need to encrypt or sign\. In general, longer encryption keys are more resistant to brute\-force attacks\.

   For help choosing a key spec, see [Selecting the key spec](key-types.md#symm-asymm-choose-key-spec)\.

1. Choose **Next**\.

1. Type an [alias](kms-alias.md) for the KMS key\. The alias name cannot begin with **aws/**\. The **aws/** prefix is reserved by Amazon Web Services to represent AWS managed keys in your account\.

   An *alias* is a friendly name that you can use to identify the KMS key in the console and in some AWS KMS APIs\. We recommend that you choose an alias that indicates the type of data you plan to protect or the application you plan to use with the KMS key\. 

   Aliases are required when you create a KMS key in the AWS Management Console\. You cannot specify an alias when you use the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation, but you can use the console or the [CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html) operation to create an alias for an existing KMS key\. For details, see [Using aliases](kms-alias.md)\.

1. \(Optional\) Type a description for the KMS key\.

   Enter a description that explains the type of data you plan to protect or the application you plan to use with the KMS key\.

   You can add a description now or update it any time unless the [key state](key-state.md) is `Pending Deletion` or `Pending Replica Deletion`\. To add, change, or delete the description of an existing customer managed key, [edit the description](editing-keys.md) in the AWS Management Console or use the [UpdateKeyDescription](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateKeyDescription.html) operation\.

1. \(Optional\) Type a tag key and an optional tag value\. To add more than one tag to the KMS key, choose **Add tag**\.

   When you add tags to your AWS resources, AWS generates a cost allocation report with usage and costs aggregated by tags\. Tags can also be used to control access to a KMS key\. For information about tagging KMS keys, see [Tagging keys](tagging-keys.md) and [ABAC for AWS KMS](abac.md)\. 

1. Choose **Next**\.

1. Select the IAM users and roles that can administer the KMS key\.
**Note**  
This key policy gives the AWS account full control of this KMS key\. It allows account administrators to use IAM policies to give other principals permission to manage the KMS key\. For details, see [Default key policy](key-policy-default.md)\.

1. \(Optional\) To prevent the selected IAM users and roles from deleting this KMS key, in the **Key deletion** section at the bottom of the page, clear the **Allow key administrators to delete this key** check box\.

1. Choose **Next**\.

1. Select the IAM users and roles that can use the KMS key for [cryptographic operations](concepts.md#cryptographic-operations)\.
**Note**  
This key policy gives the AWS account full control of this KMS key\. It allows account administrators to use IAM policies to give other principals permission to use the KMS key in cryptographic operations\. For details, see [Default key policy](key-policy-default.md)\.

1. \(Optional\) You can allow other AWS accounts to use this KMS key for cryptographic operations\. To do so, in the **Other AWS accounts** section at the bottom of the page, choose **Add another AWS account** and enter the AWS account identification number of an external account\. To add multiple external accounts, repeat this step\.
**Note**  
To allow principals in the external accounts to use the KMS key, administrators of the external account must create IAM policies that provide these permissions\. For more information, see [Allowing users in other accounts to use a KMS key](key-policy-modifying-external-accounts.md)\.

1. Choose **Next**\.

1. Review the key settings that you chose\. You can still go back and change all settings\.

1. Choose **Finish** to create the KMS key\.

## Creating asymmetric KMS keys \(AWS KMS API\)<a name="create-asymmetric-keys-api"></a>

You can use the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation to create an asymmetric AWS KMS key\. These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

When you create an asymmetric KMS key, you must specify the `KeySpec` parameter, which determines the type of keys you create\. Also, you must specify a `KeyUsage` value of ENCRYPT\_DECRYPT or SIGN\_VERIFY\. You cannot change these properties after the KMS key is created\.

The `CreateKey` operation doesn't let you specify an alias, but you can use the [CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html) operation to create an alias for your new KMS key\.

The following example uses the `CreateKey` operation to create an asymmetric KMS key of 4096\-bit RSA keys designed for public key encryption\.

```
$ aws kms create-key --key-spec RSA_4096 --key-usage ENCRYPT_DECRYPT
{
    "KeyMetadata": {
        "KeyState": "Enabled",
        "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
        "KeyManager": "CUSTOMER",
        "Description": "",
        "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
        "CreationDate": 1569973196.214,
        "MultiRegion": false,
        "KeySpec": "RSA_4096",
        "CustomerMasterKeySpec": "RSA_4096",
        "KeyUsage": "ENCRYPT_DECRYPT",
        "EncryptionAlgorithms": [
            "RSAES_OAEP_SHA_1",
            "RSAES_OAEP_SHA_256"
        ],
        "AWSAccountId": "111122223333",
        "Origin": "AWS_KMS",
        "Enabled": true
    }
}
```

The following example command creates an asymmetric KMS key that represents a pair of ECDSA keys used for signing and verification\. You cannot create an elliptic curve key pair for encryption and decryption\.

```
$ aws kms create-key --key-spec ECC_NIST_P521 --key-usage SIGN_VERIFY
{
    "KeyMetadata": {
        "KeyState": "Enabled",
        "KeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
        "CreationDate": 1570824817.837,
        "Origin": "AWS_KMS",
        "SigningAlgorithms": [
            "ECDSA_SHA_512"
        ],
        "Arn": "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321",
        "AWSAccountId": "111122223333",
        "KeySpec": "ECC_NIST_P521",
        "CustomerMasterKeySpec": "ECC_NIST_P521",
        "KeyManager": "CUSTOMER",
        "Description": "",
        "Enabled": true,
        "MultiRegion": false,
        "KeyUsage": "SIGN_VERIFY"
    }
}
```