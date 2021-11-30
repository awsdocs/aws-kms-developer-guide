# Managing multi\-Region keys<a name="multi-region-keys-manage"></a>

For most actions, you manage multi\-Region keys in the same way that you use and manage single\-Region keys\. You can enable and disable the keys, set and update aliases, key policies, grants, and tags\. However, management of multi\-Region keys differs in the following ways\.
+ You can [update the primary Region](#multi-region-update)\. This changes one of the replica keys to a primary key and the current primary key to a replica\.
+ You manage [automatic key rotation](#multi-region-rotate) only on the primary key\.
+ You can get the [public key](#multi-region-public-key) for an asymmetric multi\-Region key from any of the related primary or replica keys\.

The multi\-Region property that you set when you create KMS key is immutable\. You cannot convert a single\-Region key to multi\-Region key or a convert a multi\-Region key to a single\-Region key\. 

## Updating the primary Region<a name="multi-region-update"></a>

Every set of related multi\-Region keys must have a primary key\. But you can change the primary key\. This action, known as *updating the primary Region*, converts the current primary key to a replica key and converts one of the related replica keys to the primary key\. You might do this if you need to delete the current primary key while maintaining the replica keys, or to locate the primary key in the same Region as your key administrators\.

You can select any related replica key to be the new primary key\. Both the primary key and the replica key must be in the `Enabled` [key state](key-state.md) when the operation starts\. 

Even after this operation completes, the process of updating the primary Region might still be in progress for a few more seconds\. During this time, the old and new primary keys have a transient key state of [Updating](#update-primary-keystate)\. While the key state is `Updating`, you can use the keys in cryptographic operations, but you cannot replicate the new primary key or perform certain management operations, such as enabling or disabling these keys\. Operations such as [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) might display both the old and new primary keys as replicas\. The `Enabled` key state is restored when the update is complete\. 

Suppose you have a primary key in US East \(N\. Virginia\) \(us\-east\-1\) and a replica key in Europe \(Ireland\) \(eu\-west\-1\)\. You can use the update feature to change the primary key in US East \(N\. Virginia\) \(us\-east\-1\) to a replica key and change the replica key in Europe \(Ireland\) \(eu\-west\-1\) to the primary key\. 

![\[Updating the primary key\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/multi-region-keys-update-sm.png)

When the update process completes, the multi\-Region key in the Europe \(Ireland\) \(eu\-west\-1\) Region is a multi\-Region primary key and the key in the US East \(N\. Virginia\) \(us\-east\-1\) Region is its replica key\. If there are other related replica keys, they become replicas of the new primary key\. The next time that AWS KMS synchronizes the shared properties of the multi\-Region keys, it will get the [shared properties](multi-region-keys-overview.md#mrk-sync-properties) from the new primary key and copy them to its replica keys, including the former primary key\. 

The update operation has no effect on the [key ARN](concepts.md#key-id-key-ARN) of any multi\-Region key\. It also has no effect on shared properties, such as the key material, or on independent properties, such as the key policy\. However, you might want to [update the key policy](key-policy-modifying.md) of the new primary key\. For example, you might want to add [kms:ReplicateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReplicateKey.html) permission for trusted principals to the new primary key and remove it from the new replica key\.

### The `Updating` key state<a name="update-primary-keystate"></a>

The process of updating a primary Region takes a bit longer than the brief eventual consistency delay that affects most AWS KMS operations\. The process might still be in progress after the `UpdatePrimaryRegion` operation returns or you've completed the update procedure in the console\. Operations such as [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) might display both the old and new primary keys as replicas until the process completes\.

During the process of updating the primary Region, the old primary key and new primary key are in the `Updating` key state\. When the update process completes successfully, both keys return to the `Enabled` key state\. While in the `Updating` state, some management operations, such as enabling and disabling the keys, are not available\. However, you can continue to use both keys in cryptographic operations without interruption\. For information about the effect of the `Updating` key state, see [Key states of AWS KMS keys](key-state.md)\.

### Updating a primary Region \(console\)<a name="update-primary-console"></a>

You can update the primary key in the AWS KMS console\. Start on the key details page for the current primary key\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Select the key ID or alias of the [multi\-Region primary key](multi-region-keys-overview.md#mrk-primary-key)\. This opens the key details page for the primary key\.

   To identify a multi\-Region primary key, use the tool icon in the upper right corner to add the **Regionality** column to the table\.

1. Choose the **Regionality** tab\.

1. In the **Primary key** section, choose **Change primary Region**\.

1. Choose the Region of the new primary key\. You can choose only one Region from the menu\. 

   The **Change primary Regions** menu includes only Regions that have a related multi\-Region key\. You might not have [permission to update the primary Region](multi-region-keys-auth.md#mrk-auth-update) in all of the Regions on the menu\.

1. Choose **Change primary Region**\.

### Updating a primary Region \(AWS KMS API\)<a name="update-primary-api"></a>

To change the primary key in a set of related multi\-Region keys, use the [UpdatePrimaryRegion](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdatePrimaryRegion.html) operation\.

Use the `KeyId` parameter to identify the current primary key\. Use the `PrimaryRegion` parameter to indicate the AWS Region of the new primary key\. If the primary key doesn't already have a replica in the new primary Region, the operation fails\.

The following example changes the primary key from the multi\-Region key in the `us-west-2` Region to its replica in the `eu-west-1` Region\. The `KeyId` parameter identifies the current primary key in the `us-west-2` Region\. The `PrimaryRegion` parameter specifies the AWS Region of the new primary key, `eu-west-1`\.

```
$ aws kms update-primary-region \
      --key-id arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab \
      --primary-region eu-west-1
```

When successful, this operation doesn't return any output; just the HTTP status code\. To see the effect, call the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation on either of the multi\-Region keys\. You might want to wait until the key state returns to `Enabled`\. While the key state is [Updating](#update-primary-keystate), the values for the key might still be in flux\.

For example, the following `DescribeKey` call gets the details about the multi\-Region key in the `eu-west-1` Region\. The output shows that the multi\-Region key in the `eu-west-1` Region is now the primary key\. The related multi\-Region key \(same key ID\) in the `us-west-2` Region is now a replica key\.

```
$ aws kms describe-key \
      --key-id arn:aws:kms:eu-west-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab \

{
    "KeyMetadata": {
        "AWSAccountId": "111122223333",
        "KeyId": "mrk-1234abcd12ab34cd56ef1234567890ab",
        "Arn": "arn:aws:kms:eu-west-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
        "CreationDate": 1609193147.831,
        "Enabled": true,
        "Description": "multi-region-key",
        "KeySpec": "SYMMETRIC_DEFAULT",
        "KeyState": "Enabled",
        "KeyUsage": "ENCRYPT_DECRYPT",
        "Origin": "AWS_KMS",
        "KeyManager": "CUSTOMER",
        "CustomerMasterKeySpec": "SYMMETRIC_DEFAULT",
        "EncryptionAlgorithms": [
            "SYMMETRIC_DEFAULT"
        ],
        "MultiRegion": true,
        "MultiRegionConfiguration": { 
           "MultiRegionKeyType": "PRIMARY",
           "PrimaryKey": { 
              "Arn": "arn:aws:kms:eu-west-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
              "Region": "eu-west-1"
           },
           "ReplicaKeys": [ 
              { 
                 "Arn": "arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
                 "Region": "us-west-2"
              }
           ]
        }
    }
}
```

## Rotating multi\-Region keys<a name="multi-region-rotate"></a>

You can enable and disable [automatic rotation of the key material](rotate-keys.md) in multi\-Region keys\. Automatic key rotation is a [shared property](multi-region-keys-overview.md#mrk-sync-properties) of multi\-Region keys\.

You enable and disable automatic key rotation only on the primary key\. 
+ When AWS KMS synchronizes the multi\-Region keys, it copies the key rotation property setting from the primary key to all of its related replica keys\. 
+ When AWS KMS rotates the key material, it creates new key material for the primary key and then copies the new key material across Region boundaries to all related replica keys\. The key material never leaves AWS KMS unencrypted\. This step is carefully controlled to ensure that key material is fully synchronized before any key is used in a cryptographic operation\.
+ AWS KMS does not encrypt any data with the new key material until that key material is available in the primary key and every one of its replica keys\.
+ When you replicate a primary key that has been rotated, the new replica key has the current key material and all previous versions of the key material for its related multi\-Region keys\.

This pattern ensures that related multi\-Region keys are fully interoperable\. Any multi\-Region key can decrypt any ciphertext encrypted by a related multi\-Region key, even if the ciphertext was encrypted before the key was created\.

Automatic key rotation is not supported on asymmetric KMS keys or KMS keys with imported key material\. For information about automatic key rotation and instructions for enabling and disabling it, see [Rotating AWS KMS keys](rotate-keys.md)\.

## Downloading public keys<a name="multi-region-public-key"></a>

When you create a multi\-Region [asymmetric KMS key](symmetric-asymmetric.md#asymmetric-cmks), AWS KMS creates an RSA or elliptic curve \(ECC\) key pair for the primary key\. Then it copies that key pair to every replica of the primary key\. As a result, you can download the public key from the primary key or any of its replica keys\. You will always get the same key material\.

For information about downloading and using public keys outside of AWS KMS, see [Special considerations for downloading public keys](download-public-key.md#download-public-key-considerations)\. For instructions, see [Downloading public keys](download-public-key.md)\.