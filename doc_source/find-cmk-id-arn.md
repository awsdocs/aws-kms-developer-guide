# Finding the key ID and ARN<a name="find-cmk-id-arn"></a>

To identify an AWS KMS CMK, you can use its [key ID](concepts.md#key-id-key-id) or its Amazon Resource Name \([key ARN](concepts.md#key-id-key-ARN)\)\. In [cryptographic operations](concepts.md#cryptographic-operations), you can also use the [alias name](concepts.md#key-id-alias-name) or [alias ARN](concepts.md#key-id-alias-ARN)\.

For detailed information about the CMK identifiers that AWS KMS supports, see [Key identifiers \(KeyId\)](concepts.md#key-id)\.

## To find the key ID and ARN \(console\)<a name="find-cmk-arn"></a>

1. Open the AWS KMS console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. To view the keys in your account that you create and manage, in the navigation pane choose **Customer managed keys**\. To view the keys in your account that AWS creates and manages for you, in the navigation pane, choose **AWS managed keys**\.

1. To find the [key ID](concepts.md#key-id-key-id) for a CMK, see the row that begins with the CMK alias\. 

   The **Key ID** column appears in the tables by default\. If the Key ID column doesn't appear in your table, use the procedure described in [Customizing your CMK tables](viewing-keys-console.md#viewing-console-customize) to restore it\. You can also view the key ID of a CMK on its details page\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/find-key-id-new.png)

1. To find the Amazon Resource Name \(ARN\) of the CMK, choose the key ID or alias\. The [key ARN](concepts.md#key-id-key-ARN) appears in the **General Configuration** section\.   
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/find-key-arn-new-new.png)

## To find the key ID and key ARN \(AWS KMS API\)<a name="find-cmk-arn-api"></a>

To find the [key ID](concepts.md#key-id-key-id) and [key ARN](concepts.md#key-id-key-ARN) of a customer master key \(CMK\), use the [ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html) operation\. For examples in multiple programming languages, see [Getting key IDs and ARNs](programming-keys.md#listing-keys) and [Get key IDs and ARNs](viewing-keys-cli.md#viewing-keys-list-keys)\.

The `ListKeys` response includes the key ID and key ARN for every CMK in the account and Region\.

```
$ aws kms list-keys
{
    "Keys": [
        {
            "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "KeyArn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
        },
        {
            "KeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
            "KeyArn": "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321"
        }
    ]
}
```