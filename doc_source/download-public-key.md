# Downloading public keys<a name="download-public-key"></a>

You can view, copy, and download the public key from an asymmetric KMS key pair by using the AWS Management Console or the AWS KMS API\. You must have `kms:GetPublicKey` permission on the asymmetric KMS key\.

Each asymmetric KMS key pair consists of a private key that never leaves AWS KMS unencrypted and a public key that you can download and share\. 

You might share a public key to let others encrypt data outside of AWS KMS that you can decrypt only with your private key\. Or, to allow others to verify a digital signature outside of AWS KMS that you have generated with your private key\.

When you use the public key in your asymmetric KMS key within AWS KMS, you benefit from the authentication, authorization, and logging that are part of every AWS KMS operation\. You also reduce of risk of encrypting data that cannot be decrypted\. These features are not effective outside of AWS KMS\. For details, see [Special considerations for downloading public keys](#download-public-key-considerations)\.

**Tip**  
Looking for data keys or SSH keys? This topic explains how to manage asymmetric keys in AWS Key Management Service, where the private key is not exportable\. For exportable data key pairs where the private key is protected by a symmetric encryption KMS key, see [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html)\. For help with downloading the public key associated with an Amazon EC2 instance, see *Retrieving the public key* in the [Amazon EC2 User Guide for Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/describe-keys.html#retrieving-the-public-key) and [Amazon EC2 User Guide for Windows Instances](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/describe-keys.html#retrieving-the-public-key)\.

**Topics**
+ [Special considerations for downloading public keys](#download-public-key-considerations)
+ [Downloading a public key \(console\)](#download-public-key-console)
+ [Downloading a public key \(AWS KMS API\)](#download-public-key-api)

## Special considerations for downloading public keys<a name="download-public-key-considerations"></a>

To protect your KMS keys, AWS KMS provides access controls, authenticated encryption, and detailed logs of every operation\. AWS KMS also allows you to prevent the use of KMS keys, temporarily or permanently\. Finally, AWS KMS operations are designed to minimize of risk of encrypting data that cannot be decrypted\. These features are not available when you use downloaded public keys outside of AWS KMS\. 

**Authorization**  
[Key policies](key-policies.md) and [IAM policies](iam-policies.md) that control access to the KMS key within AWS KMS have no effect on operations performed outside of AWS\. Any user who can get the public key can use it outside of AWS KMS even if they don't have permission to encrypt data or verify signatures with the KMS key\.

**Key usage restrictions**  
Key usage restrictions are not effective outside of AWS KMS\. If you call the [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation with a KMS key that has a `KeyUsage` of `SIGN_VERIFY`, the AWS KMS operation fails\. But if you encrypt data outside of AWS KMS with a public key from a KMS key with a `KeyUsage` of `SIGN_VERIFY`, the data cannot be decrypted\.

**Algorithm restrictions**  
Restrictions on the encryption and signing algorithms that AWS KMS supports are not effective outside of AWS KMS\. If you encrypt data with the public key from a KMS key outside of AWS KMS, and use an encryption algorithm that AWS KMS does not support, the data cannot be decrypted\. 

**Disabling and deleting KMS keys**  
Actions that you can take to prevent the use of KMS key in a cryptographic operation within AWS KMS do not prevent anyone from using the public key outside of AWS KMS\. For example, disabling a KMS key, scheduling deletion of a KMS key, deleting a KMS key, or deleting the key material from a KMS key have no effect on a public key outside of AWS KMS\. If you delete an asymmetric KMS key or delete or lose its key material, data that you encrypt with a public key outside of AWS KMS is unrecoverable\.

**Logging**  
AWS CloudTrail logs that record every AWS KMS operation, including the request, response, date, time, and authorized user, do not record the use of the public key outside of AWS KMS\.

**Offline verification with SM2 key pairs \(China Regions only\)**  
To verify a signature outside of AWS KMS with an SM2 public key, you must specify the distinguishing ID\. By default, AWS KMS uses `1234567812345678` as the distinguishing ID\. For more information, see [Offline verification with SM2 key pairs \(China Regions only\)\.](asymmetric-key-specs.md#key-spec-sm-offline-verification)

## Downloading a public key \(console\)<a name="download-public-key-console"></a>

You can use the AWS Management Console to view, copy, and download the public key from an asymmetric KMS key in your AWS account\. To download the public key from an asymmetric KMS key in different AWS account, use the AWS KMS API\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Choose the alias or key ID of an asymmetric KMS key\.

1. Choose the **Cryptographic configuration** tab\. Record the values of the **Key spec**, **Key usage**, and **Encryption algorithms** or **Signing Algorithms** fields\. You'll need to use these values to use the public key outside of AWS KMS\. Be sure to share this information when you share the public key\.

1. Choose the **Public key** tab\.

1. To copy the public key to your clipboard, choose **Copy**\. To download the public key to a file, choose **Download**\.

## Downloading a public key \(AWS KMS API\)<a name="download-public-key-api"></a>

The [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) operation returns the public key in an asymmetric KMS key\. It also returns critical information that you need to use the public key correctly outside of AWS KMS, including the key usage and encryption algorithms\. Be sure to save these values and share them whenever you share the public key\.

The examples in this section use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

To specify a KMS key, use its [key ID](concepts.md#key-id-key-id), [key ARN](concepts.md#key-id-key-ARN), [alias name](concepts.md#key-id-alias-name), or [alias ARN](concepts.md#key-id-alias-ARN)\. When using an alias name, prefix it with **alias/**\. To specify a KMS key in a different AWS account, you must use its key ARN or alias ARN\.

Before running this command, replace the example alias name with a valid identifier for the KMS key\. To run this command, you must have `kms:GetPublicKey` permissions on the KMS key\.

```
$ aws kms get-public-key --key-id alias/example_RSA_3072

{
    "KeySpec": "RSA_3072",
    "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
    "KeyUsage": "ENCRYPT_DECRYPT",
    "EncryptionAlgorithms": [
        "RSAES_OAEP_SHA_1",
        "RSAES_OAEP_SHA_256"
    ],
    "PublicKey": "MIIBojANBgkqhkiG..."
}
```