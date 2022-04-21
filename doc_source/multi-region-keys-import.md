# Importing key material into multi\-Region keys<a name="multi-region-keys-import"></a>

You can import your own key material into a multi\-Region symmetric encryption KMS key\. The multi\-Region keys you create with your own key material are interoperable\. You can encrypt data in one Region and decrypt it in any other Region with a related multi\-Region key\. 

However, you must manage the key material\. 
+ AWS KMS does not copy or synchronize the key material from a primary key with imported key material to its replica keys\. You must import the same key material into related primary and replica keys\.
+ You set the expiration model and expiration dates for each key independently when you import the key material\. You can configure the same or a different expiration model and expiration dates for related multi\-Region keys\. If the key material approaches its expiration date, you must reimport the key material into the affected multi\-Region key\.

  The key states of related multi\-Region keys are independent of each other\. For example, if the key material in the primary key expires, its replica keys are unaffected\. 

The same [Region requirements for replica keys](multi-region-keys-replicate.md#replica-region) apply to multi\-Region keys with imported key material\. If you import the same key material into single\-Region keys or unrelated multi\-Region keys, these KMS keys are [not interoperable\.](#mrk-import-why) 

Multi\-Region keys with imported key material must be [symmetric KMS keys](concepts.md#symmetric-cmks) with a [key material origin](concepts.md#key-origin) of EXTERNAL\. AWS KMS does not support imported key material in [asymmetric KMS keys](symmetric-asymmetric.md#asymmetric-cmks) or KMS keys in [custom key stores](custom-key-store-overview.md)\. Also, you cannot enable [automatic key rotation](rotate-keys.md) of any KMS key with imported key material\.

Aside from their multi\-Region features, multi\-Region keys with imported key material are the same as other KMS keys with imported key material\. For detailed information about creating and configuring single\-Region keys with imported key material, see [About imported key material](importing-keys.md#importing-keys-considerations)\.

**Topics**
+ [Why aren't all KMS keys with imported key material interoperable?](#mrk-import-why)
+ [Creating a primary key with imported key material](#mrk-import-create-primary)
+ [Creating a replica key with imported key material](#mrk-import-replicate)

## Why aren't all KMS keys with imported key material interoperable?<a name="mrk-import-why"></a>

Single\-region KMS keys with imported key material are not interoperable, even when they have the same key material\. When AWS KMS uses a KMS key to encrypt data, it cryptographically binds some of the key metadata to the ciphertext\. This secures the ciphertext so that only the KMS key that encrypted data can decrypt that data\.

Multi\-Region keys are designed to be interoperable\. In addition to having the same key material, they have the same key ID and other metadata\. Thus, the ciphertexts they generate can be decrypted by any related multi\-Region key\. As a result, the trust properties of multi\-Region keys are different than those of single\-Region keys\. But for some customers, the benefit of decrypting in multiple Regions outweighs the security value of a ciphertext reliant on a single KMS key in a single AWS Region\.

## Creating a primary key with imported key material<a name="mrk-import-create-primary"></a>

To create a primary key with imported key material, you start by creating a symmetric encryption KMS key as a primary key with no key material\. Then, you import your key material into the primary key\.

The procedure for creating a multi\-Region primary key with no key material is almost the same as the procedure for [creating a single\-Region symmetric encryption key with no key material](importing-keys-create-cmk.md)\. The only difference is that you specify both a multi\-Region key and external key material\.

The permissions for creating a multi\-Region primary key with imported key material are the same as those required to [create a multi\-Region primary key](multi-region-keys-create.md) with AWS KMS key material, including the [kms:CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) and [iam:CreateServiceLinkedRole](https://docs.aws.amazon.com/IAM/latest/APIReference/API_CreateServiceLinkedRole.html) permissions in an IAM policy\. You can use the [kms:MultiRegionKeyType](policy-conditions.md#conditions-kms-multiregion-key-type) and [kms:KeyOrigin](policy-conditions.md#conditions-kms-key-origin) condition keys to allow or deny permission to create multi\-Region primary keys with imported key material\.

When creating a primary key in the AWS KMS console, use the settings in the **Advanced options** section\. Set **Key material origin** to **External**\. Set **Multi\-Region replication** to **Allow this key to be replicated into other Regions**\. You cannot change these properties after the KMS key is created\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/mrk-import-createkey-console.png)

When using API operations, use the `Origin` and `MultiRegion` parameters to create a multi\-Region key with an external key material origin\.

```
$ aws kms create-key --origin EXTERNAL --multi-region true
```

The result is a multi\-Region primary key with no key material and a key state of `PendingImport`\.

To enable this KMS key, you must download a public key and import token, use the public key to encrypt your key material, and then import your key material\. For instructions, see [Importing key material in AWS KMS keys](importing-keys.md)\.

## Creating a replica key with imported key material<a name="mrk-import-replicate"></a>

You can create a multi\-Region replica key in the AWS KMS console or by using the AWS KMS API operations\. To replicate a multi\-Region primary key with imported key material, you use the same procedure that you use to [create a replica key](multi-region-keys-replicate.md) with AWS KMS key material\. However, the result is different\. Instead of returning a replica key with the same key material as the primary key, the replicate process returns a replica key with no key material and a key state of `PendingImport`\. To enable the replica key, you must import the same key material into the replica key that you imported into its primary key\.

Although it doesn't replicate the key material, AWS KMS creates the replica key with the same [key ID](concepts.md#key-id-key-id), [key spec](concepts.md#key-spec), [key usage](concepts.md#key-usage), and [key material origin](concepts.md#key-origin) as the primary key\. It also ensures that the key material that you import into the replica key is identical to the key material that you imported into the primary key\.

To create a replica key with imported key material:

1. Create a [multi\-Region primary key](#mrk-import-create-primary) with imported key material\. 

1. Do one of the following\.

   In the AWS KMS console, choose a multi\-Region primary key with imported key material\. Then, on its **Regionality** tab, choose **Create new replica keys**\. For instructions, see [Creating replica keys \(console\)](multi-region-keys-replicate.md#replicate-console)\. 

   Or use the [ReplicateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReplicateKey.html) operation\. For the `KeyId` parameter, enter the key ID or key ARN of a multi\-Region primary key with imported key material\. For instructions, see [Creating a replica key \(AWS KMS API\)](multi-region-keys-replicate.md#replicate-api)\.

1. For each new replica key, follow the steps to [download a public key and import token](importing-keys-get-public-key-and-token.md)\. Use the public key to encrypt the primary key's key material, and then import the primary key's key material in the replica key\. You need a different public key and import token for each replica key\. 

   If the key material that you try to import into the replica key isn't the same the key material as its primary key, the operation fails\. AWS KMS doesn't require that the expiration model and expiration dates be coordinated, but you might establish business rules for your multi\-Region keys\. For instructions, see [Importing key material in AWS KMS keys](importing-keys.md)\.

### Permissions to replicate keys with imported key materials<a name="mrk-import-replica-permissions"></a>

To create a replica key with imported key material, you must have the following permissions\. 

In the primary key Region:
+ [kms:ReplicateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReplicateKey.html) on the primary key \(in the primary key's Region\)\. Include this permission in the primary key's key policy or in an IAM policy\.

In the replica key Region:
+ [kms:CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) in an IAM policy\.
+ [kms:GetParametersForImport](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetParametersForImport.html)\. You can include this permission in the key policy of the replica key or in an IAM policy\.
+ [kms:ImportKeyMaterial](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html)\. You can include this permission in the key policy of the replica key or in an IAM policy\.
+ [kms:TagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) is required to assign tags when replicating\. Include this permission in an IAM policy in the replica Region\.
+ [kms:CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html) is required to replicate a key in the AWS KMS console\. For details, see [Controlling access to aliases](alias-access.md)\.