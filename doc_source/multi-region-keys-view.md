# Viewing multi\-Region keys<a name="multi-region-keys-view"></a>

You can view single\-Region and multi\-Region keys in the AWS KMS console and by using the AWS KMS API operations\. 

**Topics**
+ [Viewing multi\-Region keys in the console](#mrk-view-console)
+ [Viewing multi\-Region keys in the API](#mrk-view-api)

## Viewing multi\-Region keys in the console<a name="mrk-view-console"></a>

In the AWS KMS console, you can view KMS keys in the selected Region\. However, if you have a multi\-Region key, you can see its related multi\-Region keys in other AWS Regions\.

The [**Customer managed keys** table](viewing-keys-console.md#viewing-console-navigate) in the AWS KMS console displays only KMS keys in the selected Region\. You can view multi\-Region primary and replica keys in the selected Region\. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

The AWS managed keys table does not have the regionality features because AWS managed keys are always single\-Region keys\. 
+ To make it easy to identify your multi\-Region keys, add the **Regionality** column to your key table\. For help, see [Customizing your KMS key tables](viewing-keys-console.md#viewing-console-customize)\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/mrk-view-regionality-column.png)
+ To display only single\-Region keys or only multi\-Region keys in your key table, filter your keys by the **Regionality** property of each key\. For help, see [Sorting and filtering your KMS keys](viewing-keys-console.md#viewing-console-filter)\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/mrk-view-regionality-filter.png)
+ You can also sort and filter your **Customer managed keys** table for the distinctive **mrk\-** key ID prefix\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/mrk-view-keyid.png)
+ For details about a multi\-Region primary key or replica key, [go to the detail page](viewing-keys-console.md#viewing-details-navigate) for the key, and choose the **Regionality** tab\.

  The **Regionality** tab for a primary key includes Change primary Region and Create new replica keys buttons\. \(The Regionality tab for a replica key has neither button\.\) The **Related multi\-Region keys** section lists all multi\-Region keys related to the current one\. If the current key is a replica key, this list includes the primary key\.

  If you choose a related multi\-Region key from the **Related multi\-Region keys** table, the AWS KMS console changes to the Region of the selected key and it opens the detail page for the key\. For example, if you choose the replica key in the `sa-east-1` Region from the example **Related multi\-Region keys** section below, the AWS KMS console changes to the `sa-east-1` Region to display the detail page for that replica key\. You might do this to view the alias or key policy for the replica key\. To change the Region again, use the Region selector at the top right corner of the page\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-regionality-tab.png)

## Viewing multi\-Region keys in the API<a name="mrk-view-api"></a>

To view multi\-Region keys in the AWS KMS API, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. It displays the specified key and all of its related multi\-Region keys\.

Like the AWS KMS console, AWS KMS API operations are Regional\. For example, when you call the [ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html) or [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operations, they return only the resources in the current or specified Region\. But when you call the `DescribeKey` operation on a multi\-Region key, the response includes all related multi\-Region keys in other AWS Regions\.

For example, the following `DescribeKey` request gets details about an example multi\-Region replica key in the Asia Pacific \(Tokyo\) \(`ap-northeast-1`\) Region\.

```
$ aws kms describe-key \
        --key-id arn:aws:kms:ap-northeast-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab \
        --region ap-northeast-1
```

Most of the `KeyMetadata` in the response describes the replica key in the Asia Pacific \(Tokyo\) Region that's the subject of the request\. However, the `MultiRegionConfiguration` element describes the primary key in the US West \(Oregon\) \(us\-west\-2\) Region and its replica keys in other AWS Regions, including the replica in the Asia Pacific \(Tokyo\) Region\. `DescribeKey` returns the same `MultiRegionConfiguration` value for all related multi\-Region keys\.

```
{
    "KeyMetadata": {
        "MultiRegion": true,        
        "AWSAccountId": "111122223333",
        "Arn": "arn:aws:kms:ap-northeast-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
        "CreationDate": 1586329200.918,
        "Description": "",
        "Enabled": true,
        "KeyId": "mrk-1234abcd12ab34cd56ef1234567890ab",
        "KeyManager": "CUSTOMER",
        "KeyState": "Enabled",
        "KeyUsage": "ENCRYPT_DECRYPT",
        "Origin": "AWS_KMS",
        "CustomerMasterKeySpec": "SYMMETRIC_DEFAULT",
        "EncryptionAlgorithms": [
            "SYMMETRIC_DEFAULT"
        ],
        "MultiRegionConfiguration": {
            "MultiRegionKeyType": "PRIMARY",
            "PrimaryKey": {
                "Arn": "arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
                "Region": "us-west-2"
            },
            "ReplicaKeys": [
                {
                    "Arn": "arn:aws:kms:eu-west-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
                    "Region": "eu-west-1"
                },
                {
                    "Arn": "arn:aws:kms:ap-northeast-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
                    "Region": "ap-northeast-1"
                },
                {
                    "Arn": "arn:aws:kms:sa-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab",
                    "Region": "sa-east-1"
                }
            ]
        }
    }
}
```