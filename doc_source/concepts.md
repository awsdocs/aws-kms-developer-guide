# AWS KMS concepts<a name="concepts"></a>

Learn the basic terms and concepts used in AWS Key Management Service \(AWS KMS\) and how they work together to help protect your data\.

**Topics**
+ [AWS KMS keys](#kms_keys)
+ [Customer keys and AWS keys](#key-mgmt)
+ [Symmetric KMS keys](#symmetric-cmks)
+ [Asymmetric KMS keys](#asymmetric-keys-concept)
+ [Data keys](#data-keys)
+ [Data key pairs](#data-key-pairs)
+ [Aliases](#alias-concept)
+ [Custom key stores](#keystore-concept)
+ [Cryptographic operations](#cryptographic-operations)
+ [Key identifiers \(KeyId\)](#key-id)
+ [Key material](#key-material)
+ [Key material origin](#key-origin)
+ [Key spec](#key-spec)
+ [Key usage](#key-usage)
+ [Envelope encryption](#enveloping)
+ [Encryption context](#encrypt_context)
+ [Key policy](#key_permissions)
+ [Grant](#grant)
+ [Auditing KMS key usage](#auditing_key_use)
+ [Key management infrastructure](#key_management)

## AWS KMS keys<a name="kms_keys"></a>

AWS KMS keys \(KMS keys\) are the primary resource in AWS KMS\. You can use a KMS key to encrypt, decrypt, and re\-encrypt data\. It can also generate data keys that you can use outside of AWS KMS\. Typically, you'll use symmetric KMS keys, but you can create and use asymmetric KMS keys for encryption or signing\.

**Note**  
AWS KMS is replacing the term *customer master key \(CMK\)* with *AWS KMS key* and *KMS key*\. The concept has not changed\. To prevent breaking changes, AWS KMS is keeping some variations of this term\.

An *AWS KMS key* is a logical representation of an encryption key\. In addition to the key material used to encrypt and decrypt data, a KMS key includes metadata, such as the key ID, creation date, description, and key state\. 

The basic KMS key, a [*symmetric KMS key*](#symmetric-cmks), represents a cryptographic key used for encryption and decryption\. You can also create an [asymmetric KMS key](symmetric-asymmetric.md) \(RSA or elliptic curve \(ECC\)\) for encryption and decryption or signing and verification \(but not both\)\.

You create KMS keys in AWS KMS\. Symmetric KMS keys and the private keys of asymmetric KMS key never leave AWS KMS unencrypted\.  To use or manage your KMS keys, you must use AWS KMS\. For information about creating and managing KMS keys, see [Managing keys](getting-started.md)\. For information about using KMS keys, see the [AWS Key Management Service API Reference](https://docs.aws.amazon.com/kms/latest/APIReference/)\.

By default, AWS KMS creates the key material for a KMS key\. You cannot extract, export, view, or manage this key material\. Also, you cannot delete this key material; you must [delete the KMS key](deleting-keys.md)\. However, you can [import your own key material](importing-keys.md) into a KMS key or create the key material for a KMS key in the AWS CloudHSM cluster associated with an [AWS KMS custom key store](custom-key-store-overview.md)\.

AWS KMS also supports [multi\-Region keys](multi-region-keys-overview.md), which let you encrypt data in one AWS Region and decrypt it in a different AWS Region\. 

For information about creating and managing KMS keys, see [Managing keys](getting-started.md)\. For information about using KMS keys, see the [AWS Key Management Service API Reference](https://docs.aws.amazon.com/kms/latest/APIReference/)\.

## Customer keys and AWS keys<a name="key-mgmt"></a>

The KMS keys that you create are [customer managed keys](#customer-cmk)\. AWS services that use KMS keys to encrypt your service resources often create keys for you\. KMS keys that AWS services create in your AWS account are [AWS managed keys](#aws-managed-cmk)\. KMS keys that AWS services create in a service account are [AWS owned keys](#aws-owned-cmk)\. 


| Type of KMS key | Can view KMS key metadata | Can manage KMS key | Used only for my AWS account | [Automatic rotation](rotate-keys.md) | 
| --- | --- | --- | --- | --- | 
| Customer managed key | Yes | Yes | Yes | Optional\. Every 365 days \(1 year\)\. | 
| AWS managed key | Yes | No | Yes | Required\. Every 1095 days \(3 years\)\. | 
| AWS owned key | No | No | No | Varies | 

[AWS services that integrate with AWS KMS](service-integration.md) differ in their support for KMS keys\. Some AWS services encrypt your data by default with an AWS owned key or an AWS managed key\. Some AWS services support customer managed keys\. Other AWS services support all types of KMS keys to allow you the ease of an AWS owned key, the visibility of an AWS managed key, or the control of a customer managed key\. For detailed information about the encryption options that an AWS service offers, see the *Encryption at Rest* topic in the user guide or the developer guide for the service\.

### Customer managed keys<a name="customer-cmk"></a>

The KMS keys that you create are *customer managed keys*\. Customer managed keys are KMS keys in your AWS account that you create, own, and manage\. You have full control over these KMS keys, including establishing and maintaining their [key policies, IAM policies, and grants](control-access.md), [enabling and disabling](enabling-keys.md) them, [rotating their cryptographic material](rotate-keys.md), [adding tags](tagging-keys.md), [creating aliases](programming-aliases.md) that refer to the KMS keys, and [scheduling the KMS keys for deletion](deleting-keys.md)\. 

Customer managed keys appear on the **Customer managed keys** page of the AWS Management Console for AWS KMS\. To definitively identify a customer managed key, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. For customer managed keys, the value of the `KeyManager` field of the `DescribeKey` response is `CUSTOMER`\.

You can use your customer managed key in cryptographic operations and audit usage in AWS CloudTrail logs\. In addition, many [AWS services that integrate with AWS KMS](service-integration.md) let you specify a customer managed key to protect the data stored and managed for you\. 

Customer managed keys incur a monthly fee and a fee for use in excess of the free tier\. They are counted against the AWS KMS [quotas](limits.md) for your account\. For details, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/) and [Quotas](limits.md)\.

### AWS managed keys<a name="aws-managed-cmk"></a>

*AWS managed keys* are KMS keys in your account created, managed, and used on your behalf by an [AWS service integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration)\. Some AWS services support only an AWS managed key\. Others use an AWS owned key or offer you a choice of KMS keys\.

You can [view the AWS managed keys](viewing-keys.md) in your account, [view their key policies](key-policy-viewing.md), and [audit their use](logging-using-cloudtrail.md) in AWS CloudTrail logs\. However, you cannot manage these KMS keys, rotate them, or change their key policies\. And, you cannot use AWS managed keys in cryptographic operations directly; the service that creates them uses them on your behalf\. 

AWS managed keys appear on the **AWS managed keys** page of the AWS Management Console for AWS KMS\. You can also identify AWS managed keys by their aliases, which have the format `aws/service-name`, such as `aws/redshift`\. To definitively identify an AWS managed keys, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. For AWS managed keys, the value of the `KeyManager` field of the `DescribeKey` response is `AWS`\.

All AWS managed keys are automatically rotated every three years\. You cannot change this rotation schedule\.

You do not pay a monthly fee for AWS managed keys\. They can be subject to fees for use in excess of the free tier, but some AWS services cover these costs for you\. For details, see the *Encryption at Rest* topic in the user guide or developer guide for the service\. For details, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/)\.

AWS managed keys do not count against resource quotas on the number of KMS keys in each Region of your account\. But when used on behalf of a principal in your account, the KMS keys count against request quotas\. For details, see [Quotas](limits.md)\.

### AWS owned keys<a name="aws-owned-cmk"></a>

*AWS owned keys* are a collection of KMS keys that an AWS service owns and manages for use in multiple AWS accounts\. Although AWS owned keys are not in your AWS account, an AWS service can use the associated AWS owned keys to protect the resources in your account\.

You do not need to create or manage the AWS owned keys\. However, you cannot view, use, track, or audit them\. AWS does not charge you a monthly fee or usage fee for AWS owned keys and they do not count against the [AWS KMS quotas](limits.md) for your account\.

The rotation of AWS owned keys varies across services\. For information about the rotation of a particular AWS owned key, see the *Encryption at Rest* topic in the user guide or developer guide for the service\.

## Symmetric KMS keys<a name="symmetric-cmks"></a>

When you create an AWS KMS key, by default, you get a symmetric KMS key\. 

In AWS KMS, a *symmetric KMS key* represents a 256\-bit encryption key that never leaves AWS KMS unencrypted\. To use a symmetric KMS key, you must call AWS KMS\. Symmetric keys are used in symmetric encryption, where the same key is used for encryption and decryption\. Unless your task explicitly requires asymmetric encryption, symmetric KMS keys, which never leave AWS KMS unencrypted, are a good choice\.

[AWS services that are integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration) use symmetric KMS keys to encrypt your data\. These services do not support encryption with asymmetric KMS keys\. For help determining whether a KMS key is symmetric or asymmetric, see [Identifying symmetric and asymmetric KMS keys](find-symm-asymm.md)\.

Technically, the key spec for a symmetric key is SYMMETRIC\_DEFAULT, the key usage is ENCRYPT\_DECRYPT, and the encryption algorithm is SYMMETRIC\_DEFAULT\. For details, see [SYMMETRIC\_DEFAULT key spec](symm-asymm-choose.md#key-spec-symmetric-default)\.

You can use a symmetric KMS key in AWS KMS to encrypt, decrypt, and re\-encrypt data, and generate data keys and data key pairs\. You can create [multi\-Region](multi-region-keys-overview.md) symmetric KMS keys, [import your own key material](importing-keys.md) into a symmetric KMS key, and create symmetric KMS keys in [custom key stores](custom-key-store-overview.md)\. For a table comparing the operations that you can perform on KMS keys of different types, see [Key type reference](symm-asymm-compare.md)\.

## Asymmetric KMS keys<a name="asymmetric-keys-concept"></a>

You can create asymmetric KMS keys in AWS KMS\. An *asymmetric KMS key* represents a mathematically related public key and private key pair\. The private key never leaves AWS KMS unencrypted\. To use the private key, you must call AWS KMS\. You can use the public key within AWS KMS by calling the AWS KMS API operations, or you can [download the public key](download-public-key.md) and use it outside of AWS KMS\. You can also create [multi\-Region](multi-region-keys-overview.md) asymmetric KMS keys\.

You can create asymmetric KMS keys that represent RSA key pairs for public key encryption or signing and verification, or elliptic curve key pairs for signing and verification\. 

For more information about creating and using asymmetric KMS keys, see [Asymmetric keys in AWS KMS](symmetric-asymmetric.md)\.

## Data keys<a name="data-keys"></a>

*Data keys* are encryption keys you can use to encrypt data, including large amounts of data and other data encryption keys\. Unlike [KMS keys](#kms_keys), which can't be downloaded, data keys are returned to you for use outside of AWS KMS\. 

When AWS KMS generates data keys, it returns a plaintext data key for immediate use \(optional\) and an encrypted copy of the data key that you can safely store with the data\. When you are ready to decrypt the data, you first ask AWS KMS to decrypt the encrypted data key\. 

AWS KMS generates, encrypts, and decrypts data keys\. However, AWS KMS does not store, manage, or track your data keys, or perform cryptographic operations with data keys\. You must use and manage data keys outside of AWS KMS\. For help using the data keys securely, see the [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/)\.

### Create a data key<a name="data-keys-create"></a>

To create a data key, call the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation\. AWS KMS generates the data key\. Then it encrypts a copy of the data key under a symmetric KMS key that you specify\. The operation returns a plaintext copy of the data key and the copy of the data key encrypted under the KMS key\. The following image shows this operation\.

![\[Generate a data key\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/generate-data-key.png)

AWS KMS also supports the [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html) operation, which returns only an encrypted data key\. When you need to use the data key, ask AWS KMS to [decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) it\.

### Encrypt data with a data key<a name="data-keys-encrypt"></a>

AWS KMS cannot use a data key to encrypt data\. But you can use the data key outside of AWS KMS, such as by using OpenSSL or a cryptographic library like the [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/)\.

After using the plaintext data key to encrypt data, remove it from memory as soon as possible\. You can safely store the encrypted data key with the encrypted data so it is available to decrypt the data\.

![\[Encrypt user data outside of AWS KMS\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/encrypt-with-data-key.png)

### Decrypt data with a data key<a name="data-keys-decrypt"></a>

To decrypt your data, pass the encrypted data key to the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. AWS KMS uses your KMS key to decrypt the data key and then returns the plaintext data key\. Use the plaintext data key to decrypt your data and then remove the plaintext data key from memory as soon as possible\.

The following diagram shows how to use the `Decrypt` operation to decrypt an encrypted data key\.

![\[Decrypting a data key\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/decrypt.png)

## Data key pairs<a name="data-key-pairs"></a>

*Data key pairs* are asymmetric data keys consisting of a mathematically\-related public key and private key\. They are designed for use in client\-side encryption and decryption or signing and verification outside of AWS KMS\. 

Unlike the data key pairs that tools like OpenSSL generate, AWS KMS protects the private key in each data key pair under a symmetric KMS key in AWS KMS that you specify\. However, AWS KMS does not store, manage, or track your data key pairs, or perform cryptographic operations with data key pairs\. You must use and manage data key pairs outside of AWS KMS\.

AWS KMS supports the following types of data key pairs:
+ RSA key pairs: RSA\_2048, RSA\_3072, and RSA\_4096
+ Elliptic curve key pairs, ECC\_NIST\_P256, ECC\_NIST\_P384, ECC\_NIST\_P521, and ECC\_SECG\_P256K1

The type of data key pair that you select usually depends on your use case or regulatory requirements\. Most certificates require RSA keys\. Elliptic curve keys are often used for digital signatures\. ECC\_SECG\_P256K1 keys are commonly used for cryptocurrencies\. AWS KMS recommends that you use ECC key pairs for signing, and use RSA key pairs for either encryption or signing, but not both\. However, AWS KMS cannot enforce any restrictions on the use of data key pairs outside of AWS KMS\.

### Create a data key pair<a name="data-keys-pairs-create"></a>

To create a data key pair, call the [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html) or [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html) operations\. Specify the symmetric KMS key you want to use to encrypt the private key\.

`GenerateDataKeyPair` returns a plaintext public key, a plaintext private key, and an encrypted private key\. Use this operation when you need a plaintext private key immediately, such as to generate a digital signature\.

`GenerateDataKeyPairWithoutPlaintext` returns a plaintext public key and an encrypted private key, but not a plaintext private key\. Use this operation when you don't need a plaintext private key immediately, such as when you're encrypting with a public key\. Later, when you need a plaintext private key to decrypt the data, you can call the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\.

The following image shows the `GenerateDataKeyPair` operation\. The `GenerateDataKeyWithoutPlaintext` operation omits the plaintext private key\.

![\[Generate a data key pair\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/generate-data-key-pair.png)

### Encrypt data with a data key pair<a name="data-key-pairs-encrypt"></a>

When you encrypt with a data key pair, you use the public key of the pair to encrypt the data and the private key of the same pair to decrypt the data\. Typically, you use data key pairs when many parties need to encrypt data that only the party with the private key can decrypt\.

The parties with the public key use that key to encrypt data, as shown in the following diagram\.

![\[Encrypt user data with the public key of a data key pair outside of AWS KMS\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/encrypt-with-data-key-pair.png)

### Decrypt data with a data key pair<a name="data-key-pairs-decrypt"></a>

To decrypt your data, use the private key in the data key pair\. For the operation to succeed, the public and private keys must be from the same data key pair, and you must use the same encryption algorithm\.

To decrypt the encrypted private key, pass it to the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. Use the plaintext private key to decrypt the data\. Then remove the plaintext private key from memory as soon as possible\.

The following diagram shows how to use the private key in a data key pair to decrypt ciphertext\.

![\[Decrypt the data with the private key in a data key pair outside of AWS KMS.\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/decrypt-with-data-key-pair.png)

### Sign messages with a data key pair<a name="data-key-pairs-sign"></a>

To generate a cryptographic signature for a message, use the private key in the data key pair\. Anyone with the public key can use it to verify that the message was signed with your private key and that it has not changed since it was signed\.

If you encrypt your private key, pass the encrypted private key to the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. AWS KMS uses your KMS key to decrypt the data key and then it returns the plaintext private key\. Use the plaintext private key to generate the signature\. Then remove the plaintext private key from memory as soon as possible\.

To sign a message, create a message digest using a cryptographic hash function, such as the [https://www.openssl.org/docs/man1.0.2/man1/openssl-dgst.html](https://www.openssl.org/docs/man1.0.2/man1/openssl-dgst.html) command in OpenSSL\. Then, pass your plaintext private key to the signing algorithm\. The result is a signature that represents the contents of the message\. \(You might be able to sign shorter messages without first creating a digest\. The maximum message size varies with the signing tool you use\.\)

The following diagram shows how to use the private key in a data key pair to sign a message\.

![\[Generate a cryptographic signature with the private key in a data key pair outside of AWS KMS.\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/sign-with-data-key-pair.png)

### Verify a signature with a data key pair<a name="data-key-pairs-verify"></a>

Anyone who has the public key in your data key pair can use it to verify the signature that you generated with your private key\. Verification confirms that an authorized user signed the message with the specified private key and signing algorithm, and the message hasn't changed since it was signed\. 

To be successful, the party verifying the signature must generate the same type of digest, use the same algorithm, and use the public key that corresponds to the private key used to sign the message\.

The following diagram shows how to use the public key in a data key pair to verify a message signature\.

![\[Verify a cryptographic signature with the public key in a data key pair outside of AWS KMS.\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/verify-with-data-key-pair.png)

## Aliases<a name="alias-concept"></a>

Use an *alias* as a friendly name for a KMS key\. For example, you can refer to a KMS key as *test\-key* instead of 1234abcd\-12ab\-34cd\-56ef\-1234567890ab\. 

Aliases make it easier to identify a KMS key in the AWS Management Console\. You can use an alias to identify a KMS key in some AWS KMS operations, including [cryptographic operations](#cryptographic-operations)\. In applications, you can use a single alias to refer to different KMS keys in each AWS Region\.

You can also allow and deny access to KMS keys based on their aliases without editing policies or managing grants\. This feature is part of AWS KMS support for attribute\-based access control \(ABAC\)\. For details, see [ABAC for AWS KMS](abac.md)\.

In AWS KMS, aliases are independent resources, not properties of a KMS key\. As such, you can add, change, and delete an alias without affecting the associated KMS key\.

**Learn more:**
+ For detailed information about aliases, see [Using aliases](kms-alias.md)\. 
+ For information about the formats of key identifiers, including aliases, see [Key identifiers \(KeyId\)](#key-id)\.
+ For help finding the aliases associated with a KMS key, see [Finding the alias name and alias ARN](find-cmk-alias.md)
+ For examples of creating and managing aliases in multiple programming languages, see [Working with aliases](programming-aliases.md)\.

## Custom key stores<a name="keystore-concept"></a>

A *custom key store* is an AWS KMS resource associated with FIPS 140\-2 Level 3 hardware security modules \(HSMs\) in a AWS CloudHSM cluster that you own and manage\. 

When you create a AWS KMS key \(KMS key\) in your custom key store, AWS KMS generates a 256\-bit, persistent, non\-exportable Advanced Encryption Standard \(AES\) symmetric key in the associated AWS CloudHSM cluster\. This key material never leaves the HSMs unencrypted\. When you use a KMS key in a custom key store, the cryptographic operations are performed in the HSMs in the cluster\.

For more information, see [Custom key stores](custom-key-store-overview.md)\.

## Cryptographic operations<a name="cryptographic-operations"></a>

In AWS KMS, *cryptographic operations* are API operations that use KMS keys to protect data\. Because KMS keys remain within AWS KMS, you must call AWS KMS to use a KMS key in a cryptographic operation\. 

To perform cryptographic operations with KMS keys, use the AWS SDKs, AWS Command Line Interface \(AWS CLI\), or the AWS Tools for PowerShell\. You cannot perform cryptographic operations in the AWS KMS console\. For examples of calling the cryptographic operations in several programming languages, see [Programming the AWS KMS API](programming-top.md)\.

The following table lists the AWS KMS cryptographic operations\. It also shows the key type and [key usage](#key-usage) requirements for KMS keys used in the operation\.


| Operation | Key type | Key usage | 
| --- | --- | --- | 
| [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) | Any | ENCRYPT\_DECRYPT | 
| [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) | Any | ENCRYPT\_DECRYPT | 
| [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) | Symmetric  | ENCRYPT\_DECRYPT | 
| [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html) | Symmetric \[1\] | ENCRYPT\_DECRYPT | 
| [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html) | Symmetric \[1\] | ENCRYPT\_DECRYPT | 
| [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html) | Symmetric | ENCRYPT\_DECRYPT | 
| [GenerateRandom](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateRandom.html) | N/A\. This operation doesn't use a KMS key\. | N/A | 
| [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) | Any | ENCRYPT\_DECRYPT | 
| [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html) | Asymmetric | SIGN\_VERIFY | 
| [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) | Asymmetric | SIGN\_VERIFY | 

\[1\] `GenerateDataKeyPair` and `GenerateDataKeyPairWithoutPlaintext` generate an asymmetric data key pair that is protected by a symmetric KMS key\.

For information about the permissions for cryptographic operations, see the [AWS KMS permissions](kms-api-permissions-reference.md)\. 

To make AWS KMS responsive and highly functional for all users, AWS KMS establishes quotas on number of cryptographic operations called in each second\. For details, see [Shared quotas for cryptographic operations](requests-per-second.md#rps-shared-limit)\. 

## Key identifiers \(KeyId\)<a name="key-id"></a>

Key identifiers act as names for your KMS keys\. They help you to recognize your KMS keys in the console\. You use them to indicate which KMS keys you want to use in AWS KMS API operations, key policies, IAM policies, and grants\.

AWS KMS defines several key identifiers\. When you create a KMS key, AWS KMS generates a key ARN and key ID, which are properties of the KMS key\. When you create an alias, AWS KMS generates an alias ARN based on the alias name that you define\. You can view the key and alias identifiers in the AWS Management Console and in the AWS KMS API\. 

In the AWS KMS console, you can view and filter KMS keys by their key ARN, key ID, or alias name, and sort by key ID and alias name\. For help finding the key identifiers in the console, see [Finding the key ID and key ARN](find-cmk-id-arn.md)\.

In the AWS KMS API, the parameters you use to identify a KMS key are named `KeyId` or a variation, such as `TargetKeyId` or `DestinationKeyId`\. However, the values of those parameters are not limited to key IDs\. Some can take any valid key identifier\. For information about the values for each parameter, see the parameter description in the AWS Key Management Service API Reference\.

**Note**  
When using the AWS KMS API, be careful about the key identifier that you use\. Different APIs require different key identifiers\. In general, use the most complete and practical key identifier for your task\.

AWS KMS supports the following key identifiers\.

**Key ARN**  <a name="key-id-key-ARN"></a>
The key ARN is the Amazon Resource Name \(ARN\) of a KMS key\. It is a unique, fully qualified identifier for the KMS key\. A key ARN includes the AWS account, Region, and the key ID\. For help finding the key ARN of a KMS key, see [Finding the key ID and key ARN](find-cmk-id-arn.md)\.  
The format of a key ARN is as follows:  

```
arn:<partition>:kms:<region>:<account-id>:key/<key-id>
```
The following is an example key ARN for a single\-Region KMS key\.  

```
arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
```
The *key\-id* element of the key ARNs of [multi\-Region keys](multi-region-keys-overview.md) begin with the `mrk-` prefix\. The following is an example key ARN for a multi\-Region key\.  

```
arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab
```

**Key ID**  <a name="key-id-key-id"></a>
The key ID uniquely identifies a KMS key within an account and Region\. For help finding the key ID of a KMS key, see [Finding the key ID and key ARN](find-cmk-id-arn.md)\.  
The following is an example key ID for a single\-Region KMS key\.  

```
1234abcd-12ab-34cd-56ef-1234567890ab
```
The key IDs of [multi\-Region keys](multi-region-keys-overview.md) begin with the `mrk-` prefix\. The following is an example key ID for a multi\-Region key\.  

```
mrk-1234abcd12ab34cd56ef1234567890ab
```

**Alias ARN**  <a name="key-id-alias-ARN"></a>
The alias ARN is the Amazon Resource Name \(ARN\) of an AWS KMS alias\. It is a unique, fully qualified identifier for the alias, and for the KMS key it represents\. An alias ARN includes the AWS account, Region, and the alias name\.  
At any given time, an alias ARN identifies one particular KMS key\. However, because you can change the KMS key associated with the alias, the alias ARN can identify different KMS keys at different times\. For help finding the alias ARN of a KMS key, see [Finding the alias name and alias ARN](find-cmk-alias.md)\.  
The format of an alias ARN is as follows:  

```
arn:<partition>:kms:<region>:<account-id>:alias/<alias-name>
```
The following is the alias ARN for a fictitious `ExampleAlias`\.  

```
arn:aws:kms:us-west-2:111122223333:alias/ExampleAlias
```

**Alias name**  <a name="key-id-alias-name"></a>
The alias name is a string of up to 256 characters\. It uniquely identifies an associated KMS key within an account and Region\. In the AWS KMS API, alias names always begin with `alias/`\. For help finding the alias name of a KMS key, see [Finding the alias name and alias ARN](find-cmk-alias.md)\.  
The format of an alias name is as follows:  

```
alias/<alias-name>
```
For example:  

```
alias/ExampleAlias
```
The `aws/` prefix for an alias name is reserved for [AWS managed keys](#aws-managed-cmk)\. You cannot create an alias with this prefix\. For example, the alias name of the AWS managed key for Amazon Simple Storage Service \(Amazon S3\) is the following\.  

```
alias/aws/s3
```

## Key material<a name="key-material"></a>

*Key material* is the secret string of bits used in a cryptographic algorithm\. Key material must be kept secret to protect the cryptographic operations that use it\.

Each KMS key includes key material along with metadata, such as its [key ID](#key-id) and [key policy](key-policies.md)\. The [key material origin](#key-origin) can vary\. You can use key material that AWS KMS generates, key material that is generated in the AWS CloudHSM cluster of a [custom key store](custom-key-store-overview.md), or [import your own key material](importing-keys.md)\. If you use AWS KMS key material, you can enable [automatic rotation](rotate-keys.md) of your key material\.

By default, each KMS key has unique key material\. However, you can create a set of [multi\-Region keys](multi-region-keys-overview.md) with the same key material\.

## Key material origin<a name="key-origin"></a>

*Key material origin* is a KMS key property that identifies the source of the key material in the KMS key\. You choose the key material origin when you create the KMS key, and you cannot change it\. To find the key material origin of a KMS key, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation, or see the **Origin** value on the **Cryptographic configuration** tab of the detail page for a KMS key in the AWS KMS console\. For help, see [Viewing Keys](viewing-keys.md)\. 

KMS keys can have one of the following key material origin values\.

**KMS \(default\)**  
API value: `AWS_KMS`  
AWS KMS creates and manages the key material for the KMS key in its own key store\. This is the default and the recommended value for most KMS keys\.  
For help creating keys with key material from AWS KMS, see [Creating keys](create-keys.md)\.

**External**  
API value: `EXTERNAL`  
The KMS key has [imported key material](importing-keys.md)\. When you create a KMS key with an `External` key material origin, the KMS key has no key material\. Later, you can import key material into the KMS key\. When you use imported key material, you need to secure and manage that key material outside of AWS KMS, including replacing the key material if it expires\. For details, see [About imported key material](importing-keys.md#importing-keys-considerations)\.  
For help creating a KMS key for imported key material, see [Step 1: Create a KMS key with no key material](importing-keys-create-cmk.md)\.

**Custom key store \(CloudHSM\)**  
API value: `AWS_CLOUDHSM`  
AWS KMS created the key material for the KMS key in your [custom key store](custom-key-store-overview.md)\.  
For help creating a KMS key in a custom key store, see [Creating KMS keys in a custom key store](create-cmk-keystore.md)

## Key spec<a name="key-spec"></a>

*Key spec* is a property that represents the cryptographic configuration of a key\. The meaning of the key spec differs with the key type\.
+ [AWS KMS keys](#kms_keys) — The *key spec* determines whether the KMS key is symmetric or asymmetric\. It also determines the type of its key material, and the encryption algorithms or signing algorithms it supports\. You choose the key spec when you [create the KMS key](create-keys.md), and you cannot change it\.
**Note**  
The `KeySpec` for an KMS key was known as a `CustomerMasterKeySpec`\. The `CustomerMasterKeySpec` parameter of the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation is deprecated\. Instead, use the `KeySpec` parameter, which works the same way\. To prevent breaking changes, the response of the `CreateKey` and [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operations now includes both `KeySpec` and `CustomerMasterKeySpec` members with the same values\.

  For a list of key specs and help with choosing a key spec, see [Selecting the key spec](symm-asymm-choose.md#symm-asymm-choose-key-spec)\. To find the key spec of a KMS key, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation, or see the **Cryptographic configuration** tab on the detail page for a KMS key in the AWS KMS console\. For help, see [Viewing Keys](viewing-keys.md)\.

  To limit the key specs that principals can use when creating KMS keys, use the [kms:KeySpec](policy-conditions.md#conditions-kms-key-spec) condition key\. You can also use the `kms:KeySpec` condition key to allow principals to call AWS KMS operations for a KMS key based on its key spec\. For example, you can deny permission to schedule deletion of a KMS key with an `RSA_4096` key spec\. 
+ [Data keys](#data-keys) \([GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)\) — The *key spec* determines the length of an AES data key\. 
+ [Data keys pairs](#data-key-pairs) \([GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html)\) — The *key pair spec* determines the type of key material in the data key pair\.

## Key usage<a name="key-usage"></a>

*Key usage* is a property that determines whether a KMS key is used for encryption and decryption \-or\- signing and verification\. A key cannot support both\. Using a KMS key for more than one type of operations makes the product of both operations more vulnerable to attack\.

The key usage for symmetric KMS keys is always encryption and decryption\. The key usage for elliptic curve \(ECC\) KMS keys is always signing and verification\. You only need to choose a key usage for RSA KMS keys\. You choose the key usage when you [create the KMS key](create-keys.md), and you cannot change it\. If you've chosen the wrong key usage, [delete the KMS key](deleting-keys.md), and create a new one\. 

For choosing the key usage, see [Selecting the key usage](symm-asymm-choose.md#symm-asymm-choose-key-usage)\. To find the key usage of a KMS key, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation, or choose the **Cryptographic configuration** tab on the detail page for a KMS key in the AWS KMS console\. For help, see [Viewing Keys](viewing-keys.md)\. 

To allow principals to create KMS keys only for signing and verification or only for encryption and decryption, use the [kms:KeyUsage](policy-conditions.md#conditions-kms-key-usage) condition key\. You can also use the `kms:KeyUsage` condition key to allow principals to call API operations for a KMS key based on its key usage\. For example, you can allow permission to disable a KMS key only if its key usage is SIGN\_VERIFY\. 

## Envelope encryption<a name="enveloping"></a>

When you encrypt your data, your data is protected, but you have to protect your encryption key\. One strategy is to encrypt it\. *Envelope encryption* is the practice of encrypting plaintext data with a data key, and then encrypting the data key under another key\.

You can even encrypt the data encryption key under another encryption key, and encrypt that encryption key under another encryption key\. But, eventually, one key must remain in plaintext so you can decrypt the keys and your data\. This top\-level plaintext key encryption key is known as the *root key*\.

![\[Envelope encryption\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/key-hierarchy-root.png)

AWS KMS helps you to protect your encryption keys by storing and managing them securely\. Root keys stored in AWS KMS, known as [AWS KMS keys](#kms_keys), never leave the AWS KMS [FIPS validated hardware security modules](https://csrc.nist.gov/projects/cryptographic-module-validation-program/Certificate/3139) unencrypted\. To use a KMS key, you must call AWS KMS\.

![\[Envelope encryption with multiple key encryption keys\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/key-hierarchy-kms-key.png)

Envelope encryption offers several benefits:
+ **Protecting data keys**

  When you encrypt a data key, you don't have to worry about storing the encrypted data key, because the data key is inherently protected by encryption\. You can safely store the encrypted data key alongside the encrypted data\.
+ **Encrypting the same data under multiple keys**

  Encryption operations can be time consuming, particularly when the data being encrypted are large objects\. Instead of re\-encrypting raw data multiple times with different keys, you can re\-encrypt only the data keys that protect the raw data\.
+ **Combining the strengths of multiple algorithms**

  In general, symmetric key algorithms are faster and produce smaller ciphertexts than public key algorithms\. But public key algorithms provide inherent separation of roles and easier key management\. Envelope encryption lets you combine the strengths of each strategy\.

## Encryption context<a name="encrypt_context"></a>

All AWS KMS [cryptographic operations](#cryptographic-operations) with symmetric KMS keys accept an *encryption context*, an optional set of non\-secret key–value pairs that can contain additional contextual information about the data\. AWS KMS uses the encryption context as [additional authenticated data](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#term-aad) \(AAD\) to support [authenticated encryption](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#define-authenticated-encryption)\. 

You cannot specify an encryption context in a cryptographic operation with an [asymmetric KMS key](symmetric-asymmetric.md#asymmetric-cmks)\. The standard asymmetric encryption algorithms that AWS KMS uses do not support an encryption context\.

When you include an encryption context in an encryption request, it is cryptographically bound to the ciphertext such that the same encryption context is required to decrypt \(or decrypt and re\-encrypt\) the data\. If the encryption context provided in the decryption request is not an exact, case\-sensitive match, the decrypt request fails\. Only the order of the key\-value pairs in the encryption context can vary\.

The encryption context is not secret\. It appears in plaintext in [AWS CloudTrail Logs](logging-using-cloudtrail.md) so you can use it to identify and categorize your cryptographic operations\.

An encryption context can consist of up to 8151 characters, including the `=` or `:` separator\. It can use any keys and values\. However, because it is not secret and not encrypted, your encryption context should not include sensitive information\. We recommend that your encryption context describe the data being encrypted or decrypted\. For example, when you encrypt a file, you might use part of the file path as encryption context\.

The key and value in an encryption context pair must be simple literal strings\. They cannot be integers or objects, or any type that is not fully resolved\. If you use a different type, such as an integer or float, AWS KMS interprets it as a string\.

```
"encryptionContext": {
    "department": "10103.0"
}
```

The encryption context key and value can include special characters, such as underscores \(\_\), dashes \(\-\), slashes \(/, \\\) and colons \(:\)\.

For example, when encrypting volumes and snapshots created with the [Amazon Elastic Block Store](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html) \(Amazon EBS\) [CreateSnapshot](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateSnapshot.html) operation, Amazon EBS uses the volume ID as encryption context value\.

```
"encryptionContext": {
  "aws:ebs:id": "vol-abcde12345abc1234"
}
```

You can also use the encryption context to refine or limit access to AWS KMS keys in your account\. You can use the encryption context [as a constraint in grants](grants.md) and as a *[condition in policy statements](policy-conditions.md)*\. 

To learn how to use encryption context to protect the integrity of encrypted data, see the post [How to Protect the Integrity of Your Encrypted Data by Using AWS Key Management Service and EncryptionContext](https://aws.amazon.com/blogs/security/how-to-protect-the-integrity-of-your-encrypted-data-by-using-aws-key-management-service-and-encryptioncontext/) on the AWS Security Blog\.

More about encryption context\.

### Encryption context in policies<a name="encryption-context-authorization"></a>

The encryption context is used primarily to verify integrity and authenticity\. But you can also use the encryption context to control access to symmetric AWS KMS keys in key policies and IAM policies\. 

The [kms:EncryptionContext:](policy-conditions.md#conditions-kms-encryption-context) and [kms:EncryptionContextKeys](policy-conditions.md#conditions-kms-encryption-context) condition keys allow \(or deny\) a permission only when the request includes particular encryption context keys or key–value pairs\. 

For example, the following key policy statement allows the `RoleForExampleApp` role to use the KMS key in `Decrypt` operations\. It uses the `kms:EncryptionContext:context-key` condition key to allow this permission only when the encryption context in the request includes an `AppName:ExampleApp` encryption context pair\.

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
      "kms:EncryptionContext:AppName": "ExampleApp"
    }
  }
}
```

For more information about these encryption context condition keys, see [Using policy conditions with AWS KMS](policy-conditions.md)\.

### Encryption context in grants<a name="encryption-context-grants"></a>

When you [create a grant](grants.md), you can include [grant constraints](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) that establish conditions for the grant permissions\. AWS KMS supports two grant constraints, `EncryptionContextEquals` and `EncryptionContextSubset`, both of which involve the [encryption context](#encrypt_context) in a request for a cryptographic operation\. When you use these grant constraints, the permissions in the grant are effective only when the encryption context in the request for the cryptographic operation satisfies the requirements of the grant constraints\. 

For example, you can add an `EncryptionContextEquals` grant constraint to a grant that allows the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation\. With this constraint, the grant allows the operation only when the encryption context in the request is a case\-sensitive match for the encryption context in the grant constraint\.

```
$ aws kms create-grant \
    --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
    --grantee-principal arn:aws:iam::111122223333:user/exampleUser \
    --retiring-principal arn:aws:iam::111122223333:role/adminRole \
    --operations GenerateDataKey \
    --constraints EncryptionContextEquals={Purpose=Test}
```

A request like the following from the grantee principal would satisfy the `EncryptionContextEquals` constraint\.

```
$ aws kms generate-data-key \
    --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
    --key-spec AES_256 \
    --encryption-context Purpose=Test
```

For details about the grant constraints, see [Using grant constraints](create-grant-overview.md#grant-constraints)\. For detailed information about grants, see [Grants in AWS KMS](grants.md)\.

### Logging encryption context<a name="encryption-context-auditing"></a>

AWS KMS uses AWS CloudTrail to log the encryption context so you can determine which KMS keys and data have been accessed\. The log entry shows exactly which KMS keys was used to encrypt or decrypt specific data referenced by the encryption context in the log entry\.

**Important**  
Because the encryption context is logged, it must not contain sensitive information\.

### Storing encryption context<a name="encryption-context-storing"></a>

To simplify use of any encryption context when you call the [https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) or [https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) operations, you can store the encryption context alongside the encrypted data\. We recommend that you store only enough of the encryption context to help you create the full encryption context when you need it for encryption or decryption\. 

For example, if the encryption context is the fully qualified path to a file, store only part of that path with the encrypted file contents\. Then, when you need the full encryption context, reconstruct it from the stored fragment\. If someone tampers with the file, such as renaming it or moving it to a different location, the encryption context value changes and the decryption request fails\.

## Key policy<a name="key_permissions"></a>

When you create a KMS keys, you determine who can use and manage that KMS keys\. These permissions are contained in a document called the *key policy*\. You can use the key policy to add, remove, or change permissions at any time for a customer managed keys\. But you cannot edit the key policy for an AWS managed keys\. For more information, see [Key policies in AWS KMS](key-policies.md)\.

## Grant<a name="grant"></a>

A *grant* is a policy instrument that allows AWS principals to use AWS KMS keys in [cryptographic operations](#cryptographic-operations)\. It also can let them view a KMS keys \([DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html)\) and create and manage grants\. When authorizing access to a KMS key, grants are considered along with [key policies](key-policies.md) and [IAM policies](iam-policies.md)\. Grants are often used for temporary permissions because you can create one, use its permissions, and delete it without changing your key policies or IAM policies\. Because grants can be very specific, and are easy to create and revoke, they are often used to provide temporary permissions or more granular permissions\. 

For detailed information about grants, including grant terminology, see [Grants in AWS KMS](grants.md)\.

## Auditing KMS key usage<a name="auditing_key_use"></a>

You can use AWS CloudTrail to audit key usage\. CloudTrail creates log files that contain a history of AWS API calls and related events for your account\. These log files include all AWS KMS API requests made with the AWS Management Console, AWS SDKs, and command line tools\. The log files also include requests to AWS KMS that AWS services make on your behalf\. You can use these log files to find important information, including when the KMS keys was used, the operation that was requested, the identity of the requester, and the source IP address\. For more information, see [Logging with AWS CloudTrail](logging-using-cloudtrail.md) and the [AWS CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\.

## Key management infrastructure<a name="key_management"></a>

A common practice in cryptography is to encrypt and decrypt with a publicly available and peer\-reviewed algorithm such as AES \(Advanced Encryption Standard\) and a secret key\. One of the main problems with cryptography is that it's very hard to keep a key secret\. This is typically the job of a key management infrastructure \(KMI\)\. AWS KMS operates the key infrastructure for you\. AWS KMS creates and securely stores your root keys, called [AWS KMS keys](#kms_keys)\. For more information about how AWS KMS operates, see [AWS Key Management Service Cryptographic Details](https://docs.aws.amazon.com/kms/latest/cryptographic-details/)\.