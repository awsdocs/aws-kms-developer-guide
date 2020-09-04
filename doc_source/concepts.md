# AWS Key Management Service concepts<a name="concepts"></a>

Learn the basic terms and concepts in AWS Key Management Service \(AWS KMS\) and how they work together to help protect your data\.

**Topics**
+ [Customer master keys \(CMKs\)](#master_keys)
+ [Data keys](#data-keys)
+ [Data key pairs](#data-key-pairs)
+ [Aliases](#alias-concept)
+ [Cryptographic operations](#cryptographic-operations)
+ [Key identifiers \(KeyId\)](#key-id)
+ [Key material origin](#key-origin)
+ [Key spec](#key-spec)
+ [Key usage](#key-usage)
+ [Envelope encryption](#enveloping)
+ [Encryption context](#encrypt_context)
+ [Key policies](#key_permissions)
+ [Grants](#grant)
+ [Grant tokens](#grant_token)
+ [Auditing CMK usage](#auditing_key_use)
+ [Key management infrastructure](#key_management)

## Customer master keys \(CMKs\)<a name="master_keys"></a>

Customer master keys are the primary resources in AWS KMS\.

A *customer master key* \(CMK\) is a logical representation of a master key\. The CMK includes metadata, such as the key ID, creation date, description, and key state\. The CMK also contains the key material used to encrypt and decrypt data\.

AWS KMS supports symmetric and asymmetric CMKs\. A *symmetric CMK* represents a 256\-bit key that is used for encryption and decryption\. An *asymmetric CMK* represents an RSA key pair that is used for encryption and decryption or signing and verification \(but not both\), or an elliptic curve \(ECC\) key pair that is used for signing and verification\. For detailed information about symmetric and asymmetric CMKs, see [Using symmetric and asymmetric keys](symmetric-asymmetric.md)\.

CMKs are created in AWS KMS\. Symmetric CMKs and the private keys of asymmetric CMKs never leave AWS KMS unencrypted\.  To manage your CMK, you can use the AWS Management Console or the [AWS KMS API](https://docs.aws.amazon.com/kms/latest/APIReference/)\. To use a CMK in [cryptographic operations](#cryptographic-operations), you must use the AWS KMS API\. This strategy differs from [data keys](#data-keys)\. AWS KMS does not store, manage, or track your data keys\. You must use them outside of AWS KMS\.

By default, AWS KMS creates the key material for a CMK\. You cannot extract, export, view, or manage this key material\. Also, you cannot delete this key material; you must [delete the CMK](deleting-keys.md)\. However, you can [import your own key material](importing-keys.md) into a CMK or create the key material for a CMK in the AWS CloudHSM cluster associated with an [AWS KMS custom key store](custom-key-store-overview.md)\.

For information about creating and managing CMKs, see [Getting started](getting-started.md)\. For information about using CMKs, see the [AWS Key Management Service API Reference](https://docs.aws.amazon.com/kms/latest/APIReference/)\.

AWS KMS supports three types of CMKs: customer managed CMKs, AWS managed CMKs, and AWS owned CMKs\. 


| Type of CMK | Can view CMK metadata | Can manage CMK | Used only for my AWS account | [Automatic rotation](rotate-keys.md) | 
| --- | --- | --- | --- | --- | 
| [Customer managed CMK](#customer-cmk) | Yes | Yes | Yes | Optional\. Every 365 days \(1 year\)\. | 
| [AWS managed CMK](#aws-managed-cmk) | Yes | No | Yes | Required\. Every 1095 days \(3 years\)\. | 
| [AWS owned CMK](#aws-owned-cmk) | No | No | No | Varies | 

To distinguish customer managed CMKs from AWS managed CMKs, use the `KeyManager` field in the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation response\. For customer managed CMKs, the `KeyManager` value is `Customer`\. For AWS managed CMKs, the `KeyManager` value is `AWS`\.

[AWS services that integrate with AWS KMS](service-integration.md) differ in their support for CMKs\. Some AWS services encrypt your data by default with an AWS owned CMK or an AWS managed CMK\. Other AWS services offer to encrypt your data under a customer managed CMK that you choose\. And other AWS services support all types of CMKs to allow you the ease of an AWS owned CMK, the visibility of an AWS managed CMK, or the control of a customer managed CMK\. For detailed information about the encryption options that an AWS service offers, see the *Encryption at Rest* topic in the user guide or the developer guide for the service\. 

### Customer managed CMKs<a name="customer-cmk"></a>

*Customer managed CMKs* are CMKs in your AWS account that you create, own, and manage\. You have full control over these CMKs, including establishing and maintaining their [key policies, IAM policies, and grants](control-access.md), [enabling and disabling](enabling-keys.md) them, [rotating their cryptographic material](rotate-keys.md), [adding tags](tagging-keys.md), [creating aliases](programming-aliases.md) that refer to the CMK, and [scheduling the CMKs for deletion](deleting-keys.md)\. 

Customer managed CMKs appear on the **Customer managed keys** page of the AWS Management Console for AWS KMS\. To definitively identify a customer managed CMK, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. For customer managed CMKs, the value of the `KeyManager` field of the `DescribeKey` response is `CUSTOMER`\.

You can use your customer managed CMKs in cryptographic operations and audit their use in AWS CloudTrail logs\. In addition, many [AWS services that integrate with AWS KMS](service-integration.md) let you specify a customer managed CMK to protect the data that they store and manage for you\. 

Customer managed CMKs incur a monthly fee and a fee for use in excess of the free tier\. They are counted against the AWS KMS [quotas](limits.md) for your account\. For details, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/) and [Quotas](limits.md)\.

### AWS managed CMKs<a name="aws-managed-cmk"></a>

*AWS managed CMKs* are CMKs in your account that are created, managed, and used on your behalf by an [AWS service that is integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration)\. Some AWS services support only an AWS managed CMK\. Others use an AWS owned CMK or offer you a choice of CMKs\.

You can [view the AWS managed CMKs](viewing-keys.md) in your account, [view their key policies](key-policy-viewing.md), and [audit their use](logging-using-cloudtrail.md) in AWS CloudTrail logs\. However, you cannot manage these CMKs, rotate them, or change their key policies\. And, you cannot use AWS managed CMKs in cryptographic operations directly; the service that creates them uses them on your behalf\. 

AWS managed CMKs appear on the **AWS managed keys** page of the AWS Management Console for AWS KMS\. You can also identify most AWS managed CMKs by their aliases, which have the format `aws/service-name`, such as `aws/redshift`\. To definitively identify an AWS managed CMK, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. For AWS managed CMKs, the value of the `KeyManager` field of the `DescribeKey` response is `AWS`\.

You do not pay a monthly fee for AWS managed CMKs\. They can be subject to fees for use in excess of the free tier, but some AWS services cover these costs for you\. For details, see the *Encryption at Rest* topic in the user guide or developer guide for the service\. AWS managed CMKs do not count against resource quotas on the number of CMKs in each Region of your account\. But when they are used on behalf of a principal in your account, these CMKs count against request quotas\. For details, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/) and [Quotas](limits.md)\.

### AWS owned CMKs<a name="aws-owned-cmk"></a>

*AWS owned CMKs* are a collection of CMKs that an AWS service owns and manages for use in multiple AWS accounts\. Although AWS owned CMKs are not in your AWS account, an AWS service can use its AWS owned CMKs to protect the resources in your account\.

You do not need to create or manage the AWS owned CMKs\. However, you cannot view, use, track, or audit them\. You are not charged a monthly fee or usage fee for AWS owned CMKs and they do not count against the [AWS KMS quotas](limits.md) for your account\.

The [key rotation](rotate-keys.md) strategy for an AWS owned CMK is determined by the AWS service that creates and manages the CMK\. For information about the types of CMKs that an AWS service supports, including AWS owned CMKs, see the *Encryption at Rest* topic in the user guide or developer guide for the service\.

## Data keys<a name="data-keys"></a>

*Data keys* are encryption keys that you can use to encrypt data, including large amounts of data and other data encryption keys\. 

You can use AWS KMS [customer master keys](#master_keys) \(CMKs\) to generate, encrypt, and decrypt data keys\. However, AWS KMS does not store, manage, or track your data keys, or perform cryptographic operations with data keys\. You must use and manage data keys outside of AWS KMS\.

### Create a data key<a name="data-keys-create"></a>

To create a data key, call the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation\. AWS KMS uses the CMK that you specify to generate a data key\. The operation returns a plaintext copy of the data key and a copy of the data key encrypted under the CMK\. The following image shows this operation\.

![\[Generate a data key\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/generate-data-key.png)

AWS KMS also supports the [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html) operation, which returns only an encrypted data key\. When you need to use the data key, ask AWS KMS to [decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) it\.

### Encrypt data with a data key<a name="data-keys-encrypt"></a>

AWS KMS cannot use a data key to encrypt data\. But you can use the data key outside of KMS, such as by using OpenSSL or a cryptographic library like the [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/)\.

After using the plaintext data key to encrypt data, remove it from memory as soon as possible\. You can safely store the encrypted data key with the encrypted data so it is available to decrypt the data\.

![\[Encrypt user data outside of AWS KMS\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/encrypt-with-data-key.png)

### Decrypt data with a data key<a name="data-keys-decryptt"></a>

To decrypt your data, pass the encrypted data key to the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. AWS KMS uses your CMK to decrypt the data key and then it returns the plaintext data key\. Use the plaintext data key to decrypt your data and then remove the plaintext data key from memory as soon as possible\.

The following diagram shows how to use the `Decrypt` operation to decrypt an encrypted data key\.

![\[Decrypting a data key\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/decrypt.png)

## Data key pairs<a name="data-key-pairs"></a>

*Data key pairs* are asymmetric data keys that consist of a mathematically\-related public key and private key\. They are designed to be used for client\-side encryption and decryption or signing and verification outside of AWS KMS\. 

Unlike the data key pairs that tools like OpenSSL generate, AWS KMS protects the private key in each data key pair under a symmetric CMK in AWS KMS that you specify\. However, AWS KMS does not store, manage, or track your data key pairs, or perform cryptographic operations with data key pairs\. You must use and manage data key pairs outside of AWS KMS\.

AWS KMS supports the following types of data key pairs:
+ RSA key pairs: RSA\_2048, RSA\_3072, and RSA\_4096
+ Elliptic curve key pairs, ECC\_NIST\_P256, ECC\_NIST\_P384, ECC\_NIST\_P521, and ECC\_SECG\_P256K1

The type of data key pair that you select usually depends on your use case or regulatory requirements\. Most certificates require RSA keys\. Elliptic curve keys are often used for digital signatures\. ECC\_SECG\_P256K1 keys are commonly used for cryptocurrencies\.

### Create a data key pair<a name="data-keys-pairs-create"></a>

To create a data key pair, call the [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html) or [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html) operations\. Specify the symmetric CMK you want to use to encrypt the private key\.

`GenerateDataKeyPair` returns a plaintext public key, a plaintext private key, and an encrypted private key\. Use this operation when you need a plaintext private key immediately, such as to generate a digital signature\.

`GenerateDataKeyPairWithoutPlaintext` returns a plaintext public key and an encrypted private key, but not a plaintext private key\. Use this operation when you don't need a plaintext private key immediately, such as when you're encrypting with a public key\. Later, when you need a plaintext private key to decrypt the data, you can call the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\.

The following image shows the `GenerateDataKeyPair` operation\. The `GenerateDataKeyWithoutPlaintext` operation omits the plaintext private key\.

![\[Generate a data key pair\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/generate-data-key-pair.png)

### Encrypt data with a data key pair<a name="data-key-pairs-encrypt"></a>

When you encrypt with a data key pair, you use the public key of the pair to encrypt the data and the private key of the same pair to decrypt the data\. Typically, data key pairs are used when many parties need to encrypt data that only the party that holds the private key can decrypt\.

The parties with the public key use that key to encrypt data, as shown in the following diagram\.

![\[Encrypt user data with the public key of a data key pair outside of AWS KMS\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/encrypt-with-data-key-pair.png)

### Decrypt data with a data key pair<a name="data-key-pairs-decrypt"></a>

To decrypt your data, use the private key in the data key pair\. For the operation to succeed, the public and private keys must be from the same data key pair, and you must use the same encryption algorithm\.

To decrypt the encrypted private key, pass it to the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. Use the plaintext private key to decrypt the data\. Then remove the plaintext private key from memory as soon as possible\.

The following diagram shows how to use the private key in a data key pair to decrypt ciphertext\.

![\[Decrypt the data with the private key in a data key pair outside of AWS KMS.\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/decrypt-with-data-key-pair.png)

### Sign messages with a data key pair<a name="data-key-pairs-sign"></a>

To generate a cryptographic signature for a message, use the private key in the data key pair\. Anyone with the public key can use it to verify that the message was signed with your private key and that it has not changed since it was signed\.

If your private key is encrypted, pass the encrypted private key to the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. AWS KMS uses your CMK to decrypt the data key and then it returns the plaintext private key\. Use the plaintext private key to generate the signature\. Then remove the plaintext private key from memory as soon as possible\.

To sign a message, create a message digest using a cryptographic hash function, such as the [dgst](https://www.openssl.org/docs/man1.0.2/man1/openssl-dgst.html) command in OpenSSL\. Then, pass your plaintext private key to the signing algorithm\. The result is a signature that represents the contents of the message\. 

The following diagram shows how to use the private key in a data key pair to sign a message\.

![\[Generate a cryptographic signature with the private key in a data key pair outside of AWS KMS.\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/sign-with-data-key-pair.png)

### Verify a signature with a data key pair<a name="data-key-pairs-verify"></a>

Anyone who has the public key in your data key pair can use it to verify the signature that you generated with your private key\. Verification confirms that an authorized user signed the message with the specified private key and signing algorithm, and the message hasn't changed since it was signed\. 

To be successful, the party verifying the signature must generate the same type of digest, use the same algorithm, and use the public key that corresponds to the private key used to sign the message\.

The following diagram shows how to use the public key in a data key pair to verify a message signature\.

![\[Verify a cryptographic signature with the public key in a data key pair outside of AWS KMS.\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/verify-with-data-key-pair.png)

## Aliases<a name="alias-concept"></a>

An *alias* is a friendly name for a CMK\. For example, you can refer to a CMK as *test\-key* instead of 1234abcd\-12ab\-34cd\-56ef\-1234567890ab\. 

Aliases make it easier to identify a CMK in the AWS Management Console\. You can also use an alias to identify a CMK in some AWS KMS operations, including [cryptographic operations](#cryptographic-operations)\. In applications, you can use a single alias to refer to different CMKs in each AWS Region\.

In AWS KMS, aliases are independent resources, not properties of a CMK\. As such, you can add, change, and delete an alias without affecting the associated CMK\.

**Learn more:**
+ For detailed information about aliases, see [Using aliases](kms-alias.md)\. 
+ For information about the formats of key identifiers, including aliases, see [Key identifiers \(KeyId\)](#key-id)\.
+ For help finding the aliases associated with a CMK, see [Finding the alias name and alias ARN](find-cmk-alias.md)
+ For examples of creating and managing aliases in multiple programming languages, see [Working with aliases](programming-aliases.md)\.

## Cryptographic operations<a name="cryptographic-operations"></a>

In AWS KMS, *cryptographic operations* are API operations that use CMKs to protect data\. Because CMKs remain within AWS KMS, you must call AWS KMS to use a CMK in a cryptographic operation\. 

To perform cryptographic operations with CMKs, use the AWS SDKs, AWS Command Line Interface \(AWS CLI\), or the AWS Tools for PowerShell\. You cannot perform cryptographic operations in the AWS KMS console\. For examples of calling the cryptographic operations in several programming languages, see [Programming the AWS KMS API](programming-top.md)\.

The following table lists the AWS KMS cryptographic operations\. It also shows the key type and [key usage](#key-usage) requirements for CMKs used in the operation\.


| Operation | CMK key type | CMK key usage | 
| --- | --- | --- | 
| [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) | Any | ENCRYPT\_DECRYPT | 
| [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) | Any | ENCRYPT\_DECRYPT | 
| [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) | Symmetric  | ENCRYPT\_DECRYPT | 
| [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html) | Symmetric \[1\] | ENCRYPT\_DECRYPT | 
| [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html) | Symmetric \[1\] | ENCRYPT\_DECRYPT | 
| [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html) | Symmetric | ENCRYPT\_DECRYPT | 
| [GenerateRandom](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateRandom.html) | N/A\. This operation doesn't use a CMK\. | N/A | 
| [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) | Any | ENCRYPT\_DECRYPT | 
| [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html) | Asymmetric | SIGN\_VERIFY | 
| [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) | Asymmetric | SIGN\_VERIFY | 

\[1\] `GenerateDataKeyPair` and `GenerateDataKeyPairWithoutPlaintext` generate an asymmetric data key pair that is protected by a symmetric CMK\.

For information about the permissions for cryptographic operations, see the [AWS KMS API permissions: Actions and resources reference](kms-api-permissions-reference.md)\. 

To make AWS KMS responsive and performant for all users, AWS KMS establishes quotas on number of cryptographic operations that can be called in each second\. For details, see [Shared quotas for cryptographic operations](requests-per-second.md#rps-shared-limit)\. 

## Key identifiers \(KeyId\)<a name="key-id"></a>

Key identifiers act as names for your AWS KMS customer master keys \(CMKs\)\. They help you to recognize your CMKs in the console\. You use them to indicate which CMKs you want to use in AWS KMS API operations, IAM policies, and grants\.

AWS KMS defines several key identifiers\. When you create a CMK, AWS KMS generates a key ARN and key ID, which are properties of the CMK\. When you create an alias, AWS KMS generates an alias ARN based on the alias name that you define\. You can view the key and alias identifiers in the AWS Management Console and in the AWS KMS API\. 

In the AWS KMS console, you can view and filter CMKs by their key ARN, key ID, or alias name, and sort by key ID and alias name\. For help finding the key identifiers in the console, see [Finding the key ID and ARN](find-cmk-id-arn.md)\.

In the AWS KMS API, the parameters that you use to identify a CMK are named `KeyId` or a variation, such as `TargetKeyId` or `DestinationKeyId`\. However, the values of those parameters are not limited to key IDs\. Some can take any valid key identifier\. For information about the values for each parameter, see the parameter description in the AWS Key Management Service API Reference\.

**Note**  
When using the AWS KMS API, be careful about the key identifier that you use\. Different APIs require different key identifiers\. In general, use the most complete key identifier that is practical for your task\.

AWS KMS supports the following key identifiers\.

**Key ARN**  <a name="key-id-key-ARN"></a>
The key ARN is the Amazon Resource Name \(ARN\) of a CMK\. It is a unique, fully qualified identifier for the CMK\. A key ARN includes the AWS account, Region, and the key ID\. For help finding the key ARN of a CMK, see [Finding the key ID and ARN](find-cmk-id-arn.md)\.  
The format of a key ARN is as follows:  

```
arn:<partition>:kms:<region>:<account-id>:key/<key-id>
```
The following is an example key ARN\.  

```
arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
```

**Key ID**  <a name="key-id-key-id"></a>
The key ID uniquely identifies a CMK within an account and Region\. For help finding the key ID of a CMK, see [Finding the key ID and ARN](find-cmk-id-arn.md)\.  
The following is an example key ID\.  

```
1234abcd-12ab-34cd-56ef-1234567890ab
```

**Alias ARN**  <a name="key-id-alias-ARN"></a>
The alias ARN is the Amazon Resource Name \(ARN\) of an AWS KMS alias\. It is a unique, fully qualified identifier for the alias, and for the CMK it represents\. An alias ARN includes the AWS account, Region, and the alias name\.   
At any given time, an alias ARN identifies one particular CMK\. However, because you can change the CMK associated with the alias, the alias ARN can identify different CMKs at different times\. For help finding the alias ARN of a CMK, see [ Finding the alias name and alias ARN  To identify an AWS KMS [customer master key](concepts.md#master_keys) \(CMK\) in a [cryptographic operation](concepts.md#cryptographic-operations), you can use its [key ID](concepts.md#key-id-key-id), [key ARN](concepts.md#key-id-key-ARN), [alias name](concepts.md#key-id-alias-name), or [alias ARN](concepts.md#key-id-alias-ARN)\. In other AWS KMS operations, only the key ID or key ARN are valid\. For detailed information about the CMK identifiers that AWS KMS supports, see [Key identifiers \(KeyId\)](concepts.md#key-id)\.   To find the alias name \(console\)  The AWS KMS console displays one [alias name](concepts.md#key-id-alias-name) associated with the CMK\. If the CMK was created in the console, the console displays the alias that was assigned to the CMK when it was created\. To find other aliases associated with the CMK, if any, you need to use the AWS KMS API\.   Open the AWS KMS console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.   To change the AWS Region, use the Region selector in the upper\-right corner of the page\.   To view the keys in your account that you create and manage, in the navigation pane choose **Customer managed keys**\. To view the keys in your account that AWS creates and manages for you, in the navigation pane, choose **AWS managed keys**\.   To find the alias name for a CMK, see the **Alias** column in the row for each CMK\. If a CMK does not have an alias, a dash \(**\-**\) appears in the **Alias** column\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/find-alias-name-1-sm.png)   You can also find the alias on the details page for a CMK\. To open the details page, choose the key ID or alias\. The alias appears in the **General Configuration** section\.   

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/find-alias-name-2.png)     To derive the alias ARN \(console\)  The [alias ARN](concepts.md#key-id-alias-ARN) is not displayed in the AWS KMS console\. The **ARN** field displays the [key ARN](concepts.md#key-id-key-ARN)\. However, you can derive the alias ARN for this alias by using the values in the **Alias** and **ARN** fields\. To derive the alias ARN, begin with the key ARN\.  

```
# Key ARN format
arn:<partition>:kms:<region>:<account>:key/<key-id>

# Example key ARN
arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
``` Replace `key` with `alias`\. Replace the key ID with the alias name\. 

```
# Alias ARN format
arn:<partition>:kms:<region>:<account>:alias/<alias-name>

# Example alias ARN
arn:aws:kms:us-west-2:111122223333:alias/master-key-test
```   To find the alias name and alias ARN \(AWS KMS API\)  To find the [alias name](concepts.md#key-id-alias-name) and [alias ARN](concepts.md#key-id-alias-ARN) of a customer master key \(CMK\), use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. For examples in multiple programming languages, see [Listing aliases](programming-aliases.md#list-aliases) and [Get alias names and ARNs](viewing-keys-cli.md#viewing-keys-list-aliases)\. By default, the response includes the alias name and alias ARN for every alias in the account and Region\. To get only the aliases for a particular CMK, use the `KeyId` parameter\. For example, the following command gets only the aliases for an example CMK with key ID `1234abcd-12ab-34cd-56ef-1234567890ab`\.  

```
$ aws kms list-aliases --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
{
    "Aliases": [
        {
            "AliasName": "alias/test-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/test-key",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab"
        },
        {
            "AliasName": "alias/project-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/project-key",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab"
        }
    ]
}
```  ](find-cmk-alias.md)\.  
The format of an alias ARN is as follows:  

```
arn:<partition>:kms:<region>:<account-id>:alias/<alias-name>
```
The following is the alias ARN for a fictitious `ExampleAlias`\.  

```
arn:aws:kms:us-west-2:111122223333:alias/ExampleAlias
```

**Alias name**  <a name="key-id-alias-name"></a>
The alias name uniquely identifies an associated CMK within an account and Region\. In the AWS KMS API, alias names always begin with `alias`\. For help finding the alias name of a CMK, see [ Finding the alias name and alias ARN  To identify an AWS KMS [customer master key](concepts.md#master_keys) \(CMK\) in a [cryptographic operation](concepts.md#cryptographic-operations), you can use its [key ID](concepts.md#key-id-key-id), [key ARN](concepts.md#key-id-key-ARN), [alias name](concepts.md#key-id-alias-name), or [alias ARN](concepts.md#key-id-alias-ARN)\. In other AWS KMS operations, only the key ID or key ARN are valid\. For detailed information about the CMK identifiers that AWS KMS supports, see [Key identifiers \(KeyId\)](concepts.md#key-id)\.   To find the alias name \(console\)  The AWS KMS console displays one [alias name](concepts.md#key-id-alias-name) associated with the CMK\. If the CMK was created in the console, the console displays the alias that was assigned to the CMK when it was created\. To find other aliases associated with the CMK, if any, you need to use the AWS KMS API\.   Open the AWS KMS console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.   To change the AWS Region, use the Region selector in the upper\-right corner of the page\.   To view the keys in your account that you create and manage, in the navigation pane choose **Customer managed keys**\. To view the keys in your account that AWS creates and manages for you, in the navigation pane, choose **AWS managed keys**\.   To find the alias name for a CMK, see the **Alias** column in the row for each CMK\. If a CMK does not have an alias, a dash \(**\-**\) appears in the **Alias** column\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/find-alias-name-1-sm.png)   You can also find the alias on the details page for a CMK\. To open the details page, choose the key ID or alias\. The alias appears in the **General Configuration** section\.   

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/find-alias-name-2.png)     To derive the alias ARN \(console\)  The [alias ARN](concepts.md#key-id-alias-ARN) is not displayed in the AWS KMS console\. The **ARN** field displays the [key ARN](concepts.md#key-id-key-ARN)\. However, you can derive the alias ARN for this alias by using the values in the **Alias** and **ARN** fields\. To derive the alias ARN, begin with the key ARN\.  

```
# Key ARN format
arn:<partition>:kms:<region>:<account>:key/<key-id>

# Example key ARN
arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
``` Replace `key` with `alias`\. Replace the key ID with the alias name\. 

```
# Alias ARN format
arn:<partition>:kms:<region>:<account>:alias/<alias-name>

# Example alias ARN
arn:aws:kms:us-west-2:111122223333:alias/master-key-test
```   To find the alias name and alias ARN \(AWS KMS API\)  To find the [alias name](concepts.md#key-id-alias-name) and [alias ARN](concepts.md#key-id-alias-ARN) of a customer master key \(CMK\), use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. For examples in multiple programming languages, see [Listing aliases](programming-aliases.md#list-aliases) and [Get alias names and ARNs](viewing-keys-cli.md#viewing-keys-list-aliases)\. By default, the response includes the alias name and alias ARN for every alias in the account and Region\. To get only the aliases for a particular CMK, use the `KeyId` parameter\. For example, the following command gets only the aliases for an example CMK with key ID `1234abcd-12ab-34cd-56ef-1234567890ab`\.  

```
$ aws kms list-aliases --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
{
    "Aliases": [
        {
            "AliasName": "alias/test-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/test-key",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab"
        },
        {
            "AliasName": "alias/project-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/project-key",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab"
        }
    ]
}
```  ](find-cmk-alias.md)\.  
The format of an alias name is as follows:  

```
alias/<alias-name>
```
For example:  

```
alias/ExampleAlias
```
The `aws/` prefix for an alias name is reserved for [AWS managed CMKs](#aws-managed-cmk)\. You cannot create an alias with this prefix\. For example, the alias name of the AWS managed CMK for Amazon Simple Storage Service \(Amazon S3\) is the following\.  

```
alias/aws/s3
```

## Key material origin<a name="key-origin"></a>

*Key material origin* is a CMK property that identifies the source of the key material in the CMK\. You choose the key material origin when you create the CMK, and you cannot change it\. To find the key material origin of a CMK, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation, or see the **Origin** value in the **Cryptographic configuration** section of the detail page for a CMK in the AWS KMS console\. For help, see [Viewing Keys](viewing-keys.md)\. 

CMKs can have one of the following key material origin values\.

**KMS \(default\)**  
API value: `AWS_KMS`  
AWS KMS creates and manages the key material for the CMK in its own key store\. This is the default and the recommended value for most CMKs\.  
For help creating keys with key material from AWS KMS, see [Creating keys](create-keys.md)\.

**External**  
API value: `EXTERNAL`  
The CMK has [imported key material](importing-keys.md)\. When you create a CMK with an `External` key material origin, the CMK has no key material\. Later, you can import key material into the CMK\. When you use imported key material, you need to secure and manage that key material outside of AWS KMS, including replacing the key material if it expires\. For details, see [About imported key material](importing-keys.md#importing-keys-considerations)\.  
For help creating a CMK for imported key material, see [Step 1: Create a CMK with no key material](importing-keys-create-cmk.md)\.

**Custom key store \(CloudHSM\)**  
API value: `AWS_CLOUDHSM`  
AWS KMS created the key material for the CMK in your [custom key store](custom-key-store-overview.md)\.  
For help creating a CMK in a custom key store, see [Creating CMKs in a custom key store](create-cmk-keystore.md)

## Key spec<a name="key-spec"></a>

*Key spec* is a CMK property that represents cryptographic configuration of the CMK\. The key spec determines whether the CMK is symmetric or asymmetric, the type of key material in the CMK, and the encryption algorithms or signing algorithms you can use with the CMK\. 

Typically, the key spec that you choose for your CMK is based on your use case and regulatory requirements\. You choose the key spec when you [create the CMK](create-keys.md), and you cannot change it\. If you've chosen the wrong key spec, [delete the CMK](deleting-keys.md), and create a new one\. 

For a list of key specs and help with choosing a key spec, see [Selecting the key spec](symm-asymm-choose.md#symm-asymm-choose-key-spec)\. To find the key spec of a CMK, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation, or see the **Cryptographic configuration** section of the detail page for a CMK in the AWS KMS console\. For help, see [Viewing Keys](viewing-keys.md)\. 

**Note**  
In AWS KMS API operations, the key spec for CMKs is known as the `CustomerMasterKeySpec`\. This distinguishes it from the key spec for data keys \(`KeySpec`\) and data key pairs \(`KeyPairSpec`\), and the key spec used when wrapping key material for import \(`WrappingKeySpec`\)\. Each key spec type has different values\.

To limit the key specs that principals can use when creating CMKs, use the [kms:CustomerMasterKeySpec](policy-conditions.md#conditions-kms-customer-master-key-spec) condition key\. You can also use the `kms:CustomerMasterKeySpec` condition key to allow principals to call AWS KMS operations for a CMK based on its key spec\. For example, you can deny permission to schedule deletion of CMK with an `RSA_4096` key spec\. 

## Key usage<a name="key-usage"></a>

*Key usage* is a CMK property that determines whether a CMK is used for encryption and decryption \-or\- signing and verification\. You cannot choose both\. Using a CMK for more than one type of operations makes the product of both operations more vulnerable to attack\.

The key usage for symmetric CMKs is always encryption and decryption\. The key usage for elliptic curve \(ECC\) CMKs is always signing and verification\. You only need to choose a key usage for RSA CMKs\. You choose the key usage when you [create the CMK](create-keys.md), and you cannot change it\. If you've chosen the wrong key usage, [delete the CMK](deleting-keys.md), and create a new one\. 

For choosing the key usage, see [Selecting the key usage](symm-asymm-choose.md#symm-asymm-choose-key-usage)\. To find the key usage of a CMK, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation, or see the **Cryptographic configuration** section of the detail page for a CMK in the AWS KMS console\. For help, see [Viewing Keys](viewing-keys.md)\. 

To allow principals to create CMKs only for signing and verification or only for encryption and decryption, use the [kms:CustomerMasterKeyUsage](policy-conditions.md#conditions-kms-customer-master-key-usage) condition key\. You can also use the `kms:CustomerMasterKeyUsage` condition key to allow principals to call API operations for a CMK based on its key usage\. For example, you can allow permission to disable a CMK only if its key usage is SIGN\_VERIFY\. 

## Envelope encryption<a name="enveloping"></a>

When you encrypt your data, your data is protected, but you have to protect your encryption key\. One strategy is to encrypt it\. *Envelope encryption* is the practice of encrypting plaintext data with a data key, and then encrypting the data key under another key\.

You can even encrypt the data encryption key under another encryption key, and encrypt that encryption key under another encryption key\. But, eventually, one key must remain in plaintext so you can decrypt the keys and your data\. This top\-level plaintext key encryption key is known as the *master key*\.

![\[Envelope encryption\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/key-hierarchy-master.png)

AWS KMS helps you to protect your master keys by storing and managing them securely\. Master keys stored in AWS KMS, known as [customer master keys](#master_keys) \(CMKs\), never leave the AWS KMS [FIPS validated hardware security modules](https://csrc.nist.gov/projects/cryptographic-module-validation-program/Certificate/3139) unencrypted\. To use an AWS KMS CMK, you must call AWS KMS\.

![\[Envelope encryption with multiple key encryption keys\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/key-hierarchy-cmk.png)

Envelope encryption offers several benefits:
+ **Protecting data keys**

  When you encrypt a data key, you don't have to worry about storing the encrypted data key, because the data key is inherently protected by encryption\. You can safely store the encrypted data key alongside the encrypted data\.
+ **Encrypting the same data under multiple master keys**

  Encryption operations can be time consuming, particularly when the data being encrypted are large objects\. Instead of re\-encrypting raw data multiple times with different keys, you can re\-encrypt only the data keys that protect the raw data\.
+ **Combining the strengths of multiple algorithms**

  In general, symmetric key algorithms are faster and produce smaller ciphertexts than public key algorithms\. But public key algorithms provide inherent separation of roles and easier key management\. Envelope encryption lets you combine the strengths of each strategy\.

## Encryption context<a name="encrypt_context"></a>

All AWS KMS [cryptographic operations](#cryptographic-operations) with symmetric CMKs accept an *encryption context*, an optional set of key–value pairs that can contain additional contextual information about the data\. AWS KMS uses the encryption context as [additional authenticated data](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#term-aad) \(AAD\) to support [authenticated encryption](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#define-authenticated-encryption)\. 

You cannot specify an encryption context in a cryptographic operation with an [asymmetric CMK](symm-asymm-concepts.md#asymmetric-cmks)\. The standard asymmetric encryption algorithms that AWS KMS uses do not support an encryption context\.

When you include an encryption context in an encryption request, it is cryptographically bound to the ciphertext such that the same encryption context is required to decrypt \(or decrypt and re\-encrypt\) the data\. If the encryption context provided in the decryption request is not an exact, case\-sensitive match, the decrypt request fails\. Only the order of the key\-value pairs in the encryption context can vary\.

The encryption context is not secret\. It appears in plaintext in [AWS CloudTrail Logs](logging-using-cloudtrail.md) so you can use it to identify and categorize your cryptographic operations\.

An encryption context can consist of any keys and values\. However, because it is not secret and not encrypted, your encryption context should not include sensitive information\. We recommend that your encryption context describe the data being encrypted or decrypted\. For example, when you encrypt a file, you might use part of the file path as encryption context\.

The key and value in an encryption context pair must be simple literal strings\. They cannot be integers or objects, or any type that is not fully resolved\. If you use a different type, such as an integer or float, AWS KMS interprets it as a string\.

```
"encryptionContext": {
    "department": "10103.0"
}
```

The encryption context key and value can include special characters, such as underscores \(\_\), dashes \(\-\), slashes \(/, \\\) and colons \(:\)\.

For example, [Amazon Simple Storage Service](services-s3.md#s3-encryption-context) \(Amazon S3\) uses an encryption context in which the key is `aws:s3:arn`\. The value is the S3 bucket path to the file that is being encrypted\.

```
"encryptionContext": {
    "aws:s3:arn": "arn:aws:s3:::bucket_name/file_name"
}
```

You can also use the encryption context to refine or limit access to customer master keys \(CMKs\) in your account\. You can use the encryption context [as a constraint in grants](grants.md) and as a *[condition in policy statements](policy-conditions.md)*\. 

To learn how to use encryption context to protect the integrity of encrypted data, see the post [How to Protect the Integrity of Your Encrypted Data by Using AWS Key Management Service and EncryptionContext](https://aws.amazon.com/blogs/security/how-to-protect-the-integrity-of-your-encrypted-data-by-using-aws-key-management-service-and-encryptioncontext/) on the AWS Security Blog\.

More about encryption context\.

### Encryption context in policies<a name="encryption-context-authorization"></a>

The encryption context is used primarily to verify integrity and authenticity\. But you can also use the encryption context to control access to symmetric customer master keys \(CMKs\) in key policies and IAM policies\. 

The [kms:EncryptionContext:](policy-conditions.md#conditions-kms-encryption-context) and [kms:EncryptionContextKeys](policy-conditions.md#conditions-kms-encryption-context) condition keys allow \(or deny\) a permission only when the request includes particular encryption context keys or key–value pairs\. 

For example, the following key policy statement allows the `RoleForExampleApp` role to use the CMK in `Decrypt` operations\. It uses the `kms:EncryptionContext:` condition key to allow this permission only when the encryption context in the request includes an `AppName:ExampleApp` encryption context pair\.

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
      "kms:EncryptionContext:AppName": "ExampleApp"
    }
  }
}
```

For more information about these encryption context condition keys, see [Using policy conditions with AWS KMS](policy-conditions.md)\.

### Encryption context in grants<a name="encryption-context-grants"></a>

When you [create a grant](grants.md), you can include [grant constraints](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) that allow access only when a request includes a particular encryption context or encryption context keys\. For details about the `EncryptionContextEquals` and `EncryptionContextSubset` grant constraints, see [Grant constraints](grants.md#grant-constraints)\.

To specify an encryption context constraint in a grant for a symmetric CMK, use the `Constraints` parameter in the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation\. This example uses the AWS Command Line Interface, but you can use any AWS SDK\. The grant that this command creates gives the `exampleUser` permission to call the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. But that permission is effective only when the encryption context in the `Decrypt` request includes a `"Department": "IT"` encryption context pair\.

```
$ aws kms create-grant \
    --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
    --grantee-principal arn:aws:iam::111122223333:user/exampleUser \
    --operations Decrypt \
    --retiring-principal arn:aws:iam::111122223333:role/adminRole \
    --constraints EncryptionContextSubset={Department=IT}
```

The resulting grant looks like the following one\. Notice that the permission granted to `exampleUser` is effective only when the `Decrypt` request includes the encryption context pair specified in the grant constraint\. To find the grants on a CMK, use the [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation\.

```
$ aws kms list-grants --key-id 1234abcd-12ab-34cd-56ef-1234567890ab

{
    "Grants": [
        {
            "Name": "",
            "IssuingAccount": "arn:aws:iam::111122223333:root",
            "GrantId": "8c94d1f12f5e69f440bae30eaec9570bb1fb7358824f9ddfa1aa5a0dab1a59b2",
            "Operations": [
                "Decrypt"
            ],
            "GranteePrincipal": "arn:aws:iam::111122223333:user/exampleUser",
            "Constraints": {
                "EncryptionContextSubset": {
                    "Department": "IT"
                }
            },
            "CreationDate": 1568565290.0,
            "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "RetiringPrincipal": "arn:aws:iam::111122223333:role/adminRole"
        }
    ]
}
```

AWS services often use encryption context constraints in the grants that give them permission to use CMKs in your AWS account\. For example, Amazon DynamoDB uses a grant like the following one to get permission to use the [AWS managed CMK](#aws-managed-cmk) for DynamoDB in your account\. The `EncryptionContextSubset` grant constraint in this grant makes the permissions in the grant effective only when the encryption context in the request includes `"subscriberID": "111122223333"` and `"tableName": "Services"` pairs\. This grant constraint means that the grant allows DynamoDB to use the specified CMK only for a particular table in your AWS account\.

To get this output, run the [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation on the AWS managed CMK for DynamoDB in your account\.

```
$ aws kms list-grants --key-id 0987dcba-09fe-87dc-65ba-ab0987654321

{
    "Grants": [
        {
            "Operations": [
                "Decrypt",
                "Encrypt",
                "GenerateDataKey",
                "ReEncryptFrom",
                "ReEncryptTo",
                "RetireGrant",
                "DescribeKey"
            ],
            "IssuingAccount": "arn:aws:iam::111122223333:root",
            "Constraints": {
                "EncryptionContextSubset": {
                    "aws:dynamodb:tableName": "Services",
                    "aws:dynamodb:subscriberId": "111122223333"
                }
            },
            "CreationDate": 1518567315.0,
            "KeyId": "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321",
            "GranteePrincipal": "dynamodb.us-west-2.amazonaws.com",
            "RetiringPrincipal": "dynamodb.us-west-2.amazonaws.com",
            "Name": "8276b9a6-6cf0-46f1-b2f0-7993a7f8c89a",
            "GrantId": "1667b97d27cf748cf05b487217dd4179526c949d14fb3903858e25193253fe59"
        }
    ]
}
```

### Logging encryption context<a name="encryption-context-auditing"></a>

AWS KMS uses AWS CloudTrail to log the encryption context so you can determine which CMKs and data have been accessed\. The log entry shows exactly which CMK was used to encrypt or decrypt specific data referenced by the encryption context in the log entry\.

**Important**  
Because the encryption context is logged, it must not contain sensitive information\.

### Storing encryption context<a name="encryption-context-storing"></a>

To simplify use of any encryption context when you call the [https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) or [https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) operations, you can store the encryption context alongside the encrypted data\. We recommend that you store only enough of the encryption context to help you create the full encryption context when you need it for encryption or decryption\. 

For example, if the encryption context is the fully qualified path to a file, store only part of that path with the encrypted file contents\. Then, when you need the full encryption context, reconstruct it from the stored fragment\. If someone tampers with the file, such as renaming it or moving it to a different location, the encryption context value changes and the decryption request fails\.

## Key policies<a name="key_permissions"></a>

When you create a CMK, you determine who can use and manage that CMK\. These permissions are contained in a document called the *key policy*\. You can use the key policy to add, remove, or change permissions at any time for a customer managed CMK\. But you cannot edit the key policy for an AWS managed CMK\. For more information, see [Authentication and access control for AWS KMS](control-access.md)\.

## Grants<a name="grant"></a>

A *grant* is another mechanism for providing permissions\. It's an alternative to key policies\. Because grants can be very specific, and are easy to create and revoke, they are often used to provide temporary permissions or more granular permissions\.

## Grant tokens<a name="grant_token"></a>

When you create a grant, the permissions specified in the grant might not take effect immediately due to [eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency)\. If you need to mitigate the potential delay, use the *grant token* that you receive in the response to your [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) request\. You can pass the grant token with some AWS KMS API requests to make the permissions in the grant take effect immediately\. The following AWS KMS API operations accept grant tokens:
+ [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html)
+ [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)
+ [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html)
+ [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html)
+ [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)
+ [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html)
+ [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)
+ [RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html)

A grant token is not a secret\. The grant token contains information about who the grant is for and therefore who can use it to cause the grant's permissions to take effect more quickly\.

## Auditing CMK usage<a name="auditing_key_use"></a>

You can use AWS CloudTrail to audit key usage\. CloudTrail creates log files that contain a history of AWS API calls and related events for your account\. These log files include all AWS KMS API requests made with the AWS Management Console, AWS SDKs, and command line tools\. The log files also include requests to AWS KMS that AWS services make on your behalf\. You can use these log files to find important information, including when the CMK was used, the operation that was requested, the identity of the requester, and the source IP address\. For more information, see [Logging with AWS CloudTrail](logging-using-cloudtrail.md) and the [AWS CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\.

## Key management infrastructure<a name="key_management"></a>

A common practice in cryptography is to encrypt and decrypt with a publicly available and peer\-reviewed algorithm such as AES \(Advanced Encryption Standard\) and a secret key\. One of the main problems with cryptography is that it's very hard to keep a key secret\. This is typically the job of a key management infrastructure \(KMI\)\. AWS KMS operates the KMI for you\. AWS KMS creates and securely stores your master keys, called [customer master keys](#master_keys)\. For more information about how AWS KMS operates, see the [AWS Key Management Service Cryptographic Details](https://d0.awsstatic.com/whitepapers/KMS-Cryptographic-Details.pdf) whitepaper\.