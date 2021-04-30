# Finding the alias name and alias ARN<a name="find-cmk-alias"></a>

An alias is a friendly name for an AWS KMS [customer master key](concepts.md#master_keys) \(CMK\)\. You can find the [alias name](concepts.md#key-id-alias-name) and [alias ARN](concepts.md#key-id-alias-ARN) in the AWS KMS console or AWS KMS API\.

For detailed information about the CMK identifiers that AWS KMS supports, see [Key identifiers \(KeyId\)](concepts.md#key-id)\. For help finding the key ID and key ARN, see [Finding the key ID and ARN](find-cmk-id-arn.md)\.

**Topics**
+ [To find the alias name and alias ARN \(console\)](#find-alias-console)
+ [To find the alias name and alias ARN \(AWS KMS API\)](#find-cmk-arn-api)

## To find the alias name and alias ARN \(console\)<a name="find-alias-console"></a>

The AWS KMS console displays the aliases associated with the CMK\. 

1. Open the AWS KMS console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. To view the keys in your account that you create and manage, in the navigation pane choose **Customer managed keys**\. To view the keys in your account that AWS creates and manages for you, in the navigation pane, choose **AWS managed keys**\.

1. The **Aliases** column displays the alias for each CMK\. If a CMK does not have an alias, a dash \(**\-**\) appears in the **Aliases** column\.

   If a CMK has multiple aliases, the **Aliases** column also has an alias summary, such as **\(\+*n* more\)**\. For example, the following CMK has two aliases, one of which is `master-key-test`\. 

   To find the alias name and alias ARN of all aliases for the CMK, use the **Aliases** tab\. 
   + To go directly to the **Aliases** tab, in the **Aliases** column, choose the alias summary \(**\+*n* more**\)\. An alias summary appears only if the CMK has more than one alias\.
   + Or, choose the alias or key ID of the CMK \(which opens the detail page for the CMK\) and then choose the **Aliases** tab\. The tabs are under the **General configuration** section\.   
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/find-alias-name-1-sm.png)

1. The **Aliases** tab displays the alias name and alias ARN of all aliases for a CMK\. You can also create and delete aliases for the CMK on this tab\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/alias-tab-1.png)

## To find the alias name and alias ARN \(AWS KMS API\)<a name="find-cmk-arn-api"></a>

To find the [alias name](concepts.md#key-id-alias-name) and [alias ARN](concepts.md#key-id-alias-ARN) of a customer master key \(CMK\), use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. For examples in multiple programming languages, see [Listing aliases](programming-aliases.md#list-aliases) and [Get alias names and ARNsGet tags](viewing-keys-cli.md#viewing-keys-list-aliases)\.

By default, the response includes the alias name and alias ARN for every alias in the account and Region\. To get only the aliases for a particular CMK, use the `KeyId` parameter\.

For example, the following command gets only the aliases for an example CMK with key ID `1234abcd-12ab-34cd-56ef-1234567890ab`\. 

```
$ aws kms list-aliases --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
{
    "Aliases": [
        {
            "AliasName": "alias/test-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/test-key",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "CreationDate": 1593622000.191,
            "LastUpdatedDate: 1593622000.191
        },
        {
            "AliasName": "alias/project-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/project-key",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab"
            "CreationDate": 1516435200.399,
            "LastUpdatedDate: 1516435200.399
        }
    ]
}
```