# Creating multi\-Region replica keys<a name="multi-region-keys-replicate"></a>

You can create a [multi\-Region replica key](multi-region-keys-overview.md#mrk-primary-key) in the AWS KMS console, by using the [ReplicateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReplicateKey.html) operation, or by using a [AWS CloudFormation template](creating-resources-with-cloudformation.md)\. You cannot use the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation to create a replica key\.

When this operation completes, the new replica key has a transient [key state](key-state.md) of `Creating`\. This key state changes to `Enabled` \(or [PendingImport](multi-region-keys-import.md)\) after a few seconds when the process of creating the new replica key is complete\. While the key state is `Creating`, you can manage key, but you cannot yet use it in cryptographic operations\. If you are creating and using the replica key programmatically, retry on `KMSInvalidStateException` or call [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) to check its `KeyState` value before using it\. 

If you mistakenly delete a replica key, you can use this procedure to recreate it\. If you replicate the same primary key in the same Region, the new replica key you create will have the same [shared properties](multi-region-keys-overview.md#mrk-sync-properties) as the original replica key\.

**Learn more**
+ To create a multi\-Region replica key with imported key material, see [Creating a replica key with imported key material](multi-region-keys-import.md#mrk-import-replicate)\.
+ To use a AWS CloudFormation template to create a replica key, see [AWS::KMS::ReplicaKey](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-kms-replicakey.html) in the *AWS CloudFormation User Guide*\.

**Topics**
+ [Replica Regions](#replica-region)
+ [Creating replica keys \(console\)](#replicate-console)
+ [Creating a replica key \(AWS KMS API\)](#replicate-api)

## Replica Regions<a name="replica-region"></a>

You typically choose to replicate a multi\-Region key into an AWS Region based on your business model and regulatory requirements\. For example, you might replicate a key into Regions where you keep your resources\. Or, to comply with a disaster recovery requirement, you might replicate a key into geographically distant Regions\. 

The following are the AWS KMS requirements for replica Regions\. If the Region that you choose doesn't comply with these requirements, attempts to replicate a key fail\.
+ **One related multi\-Region key per Region** — You can't create a replica key in the same Region as its primary key, or in the same Region as another replica of the primary key\.

  If you try to replicate a primary key in a Region that already has a replica of that primary key, the attempt fails\. If the current replica key in the Region is in the [`PendingDeletion` key state](key-state.md), you can [cancel the replica key deletion](deleting-keys.md#deleting-keys-scheduling-key-deletion) or wait until the replica key is deleted\.
+ **Multiple unrelated multi\-Region keys in the same Region** — You can have multiple unrelated multi\-Region keys in the same Region\. For example, you can have two multi\-Region primary keys in the `us-east-1` Region\. Each of the primary keys can have a replica key in `us-west-2` Region\.
+ **Regions in the same partition** — The replica key Region must be in the same [AWS partition](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) as the primary key Region\.
+ **Region must be enabled** — If a Region is [disabled by default](https://docs.aws.amazon.com/general/latest/gr/rande-manage.html#rande-manage-enable), you cannot create any resources in that Region until it is enabled for your AWS account\. 

## Creating replica keys \(console\)<a name="replicate-console"></a>

In the AWS KMS console, you can create one or many replicas of a multi\-Region primary key in the same operation\. 

This procedure is similar to creating a standard single\-Region KMS key in the console\. However, because a replica key is based on the primary key, you do not select values for [shared properties](multi-region-keys-overview.md#mrk-sync-properties), such as the key spec \(symmetric or asymmetric\), key usage, or key origin\. 

You do specify properties that are not shared, including an alias, tags, a description, and a key policy\. As a convenience, the console displays the current property values of the primary key, but you can change them\. Even if you keep the primary key values, AWS KMS does not keep these values synchronized\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Select the key ID or alias of a [multi\-Region primary key](multi-region-keys-overview.md#mrk-primary-key)\. This opens the key details page for the KMS key\.

   To identify a multi\-Region primary key, use the tool icon in the upper right corner to add the **Regionality** column to the table\.

1. Choose the **Regionality** tab\.

1. In the **Related multi\-Region keys** section, choose **Create new replica keys**\.

   The **Related multi\-Region keys** section displays the Region of the primary key and its replica keys\. You can use this display to help you choose the Region for your new replica key\.

1. Choose one or more AWS Regions\. This procedure creates a replica key in each of the Regions you select\. 

   The menu includes only Regions in the same AWS partition as the primary key\. Regions that already have a related multi\-Region key are displayed, but not selectable\.  You might not have permission to replicate a key into all of the Regions on the menu\.

   When you are finished choosing Regions, close the menu\. The Regions you chose are displayed\. To cancel replication into a Region, choose the **X** beside the Region name\.

1. Type an [alias](kms-alias.md) for the replica key\. 

   The console displays one of the current aliases of the primary key, but you can change it\. You can give your multi\-Region primary key and its replicas the same alias or different aliases\. Aliases are not a [shared property](multi-region-keys-overview.md#mrk-sync-properties) of multi\-Region keys\. AWS KMS does not synchronize the aliases of multi\-Region keys\.

   Adding, deleting, or updating an alias can allow or deny permission to the KMS key\. For details, see [ABAC for AWS KMS](abac.md) and [Using aliases to control access to KMS keys](alias-authorization.md)\.

1. \(Optional\) Type a description of the replica key\.

   The console displays the current description of the primary key, but you can change it\. Descriptions are not a shared property of multi\-Region keys\. You can give your multi\-Region primary key and its replicas the same description or different descriptions\. AWS KMS does not synchronize the key descriptions of multi\-Region keys\.

1. \(Optional\) Type a tag key and an optional tag value\. To assign more than one tag to the replica key, choose **Add tag**\.

   The console displays the tags currently attached to the primary key, but you can change them\. Tags are not a shared property of multi\-Region keys\. You can give your multi\-Region primary key and its replicas the same tags or different tags\. AWS KMS does not synchronize the tags of multi\-Region keys\. 

   Tagging or untagging a KMS key can allow or deny permission to the KMS key\. For details, see [ABAC for AWS KMS](abac.md) and [Using tags to control access to KMS keys](tag-authorization.md)\.

1. Select the IAM users and roles that can administer the replica key\.
**Note**  
IAM policies can give other IAM users and roles permission to manage the replica keys\.

   This step begins the process of creating a [key policy](key-policies.md) for the replica key\. The console displays the current key policy of the primary key, but you can change it\. Key policies are not a shared property of multi\-Region keys\. You can give your multi\-Region primary key and its replicas the same key policy or different key policies\. AWS KMS does not synchronize key policies\. You can change the key policy of any KMS key at any time\.

1. Complete the steps for creating the key policy, including selecting key administrators\. After you review the key policy, choose **Finish** to create the replica key\.

## Creating a replica key \(AWS KMS API\)<a name="replicate-api"></a>

To create a multi\-Region replica key, use the [ReplicateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReplicateKey.html) operation\. You cannot use the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation to create a replica key\. This operation creates one replica key at a time\. The Region that you specify must comply with the [Region requirements](#replica-region) for replica keys\.

When you use the `ReplicateKey` operation, you don't specify values for any [shared properties](multi-region-keys-overview.md#mrk-sync-properties) of multi\-Region keys\. Shared property values are copied from the primary key and kept synchronized\. However, you can specify values for properties that are not shared\. Otherwise, AWS KMS applies the standard default values for KMS keys, not the values of the primary key\.

**Note**  
If you don't specify values for the `Tags`, `Description`, or `KeyPolicy` parameters, AWS KMS creates the replica key with no tags, an empty string description, and the [default key policy](key-policy-default.md)\.

For example, the following command creates a multi\-Region replica key in the Asia Pacific \(Sydney\) Region \(ap\-southeast\-2\)\. This replica key is modeled on the primary key in the US East \(N\. Virginia\) Region \(us\-east\-1\), which is identified by the value of the `KeyId` parameter\. This example accepts default values for all other properties, including the key policy\.

The response describes the new replica key\. It includes fields for shared properties, such as the `KeyId`, `KeySpec`, `KeyUsage`, and key material origin \(`Origin`\)\. It also includes properties that are independent of the primary key, such as the `Description`, key policy \(`ReplicaKeyPolicy`\), and tags \(`ReplicaTags`\)\. 

The response also includes the key ARN and region of the primary key and all of its replica keys, including the one that was just created in the ap\-southeast\-2 Region\. In this example, the `ReplicaKey` element shows that this primary key was already replicated in the Europe \(Ireland\) Region \(eu\-west\-1\)\.

```
$ aws kms replicate-key \
    --key-id arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab \
    --replica-region ap-southeast-2
{
    "ReplicaKeyMetadata": {
        "MultiRegion": true,
        "MultiRegionConfiguration": {
            "MultiRegionKeyType": "REPLICA",
            "PrimaryKey": {
                "Arn": "arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
                "Region": "us-east-1"
            },
            "ReplicaKeys": [
                {
                    "Arn": "arn:aws:kms:ap-southeast-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
                    "Region": "ap-southeast-2"
                },
                {
                    "Arn": "arn:aws:kms:eu-west-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
                    "Region": "eu-west-1"
                }
            ]
        },
        "AWSAccountId": "111122223333",
        "Arn": "arn:aws:kms:ap-southeast-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
        "CreationDate": 1607472987.918,
        "Description": "",
        "Enabled": true,
        "KeyId": "mrk-1234abcd12ab34cd56ef1234567890ab",
        "KeyManager": "CUSTOMER",
        "KeySpec": "SYMMETRIC_DEFAULT",
        "KeyState": "Enabled",
        "KeyUsage": "ENCRYPT_DECRYPT",
        "Origin": "AWS_KMS",
        "CustomerMasterKeySpec": "SYMMETRIC_DEFAULT",
        "EncryptionAlgorithms": [
            "SYMMETRIC_DEFAULT"
        ]
    },
    "ReplicaKeyPolicy": "{\n  \"Version\" : \"2012-10-17\",\n  \"Id\" : \"key-default-1\",...,
    "ReplicaTags": []
}
```