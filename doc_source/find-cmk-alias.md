# Finding the alias name and alias ARN<a name="find-cmk-alias"></a>

To identify an AWS KMS [customer master key](concepts.md#master_keys) \(CMK\) in a [cryptographic operation](concepts.md#cryptographic-operations), you can use its [key ID](concepts.md#key-id-key-id), [key ARN](concepts.md#key-id-key-ARN), [alias name](concepts.md#key-id-alias-name), or [alias ARN](concepts.md#key-id-alias-ARN)\. In other AWS KMS operations, only the key ID or key ARN are valid\.

For detailed information about the CMK identifiers that AWS KMS supports, see [Key identifiers \(KeyId\)](concepts.md#key-id)\.

**Topics**
+ [To find the alias name \(console\)](#find-alias-console)
+ [To derive the alias ARN \(console\)](#derive-alias-arn)
+ [To find the alias name and alias ARN \(AWS KMS API\)](#find-cmk-arn-api)

## To find the alias name \(console\)<a name="find-alias-console"></a>

The AWS KMS console displays one [alias name](concepts.md#key-id-alias-name) associated with the CMK\. If the CMK was created in the console, the console displays the alias that was assigned to the CMK when it was created\. To find other aliases associated with the CMK, if any, you need to use the AWS KMS API\.

1. Open the AWS KMS console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. To view the keys in your account that you create and manage, in the navigation pane choose **Customer managed keys**\. To view the keys in your account that AWS creates and manages for you, in the navigation pane, choose **AWS managed keys**\.

1. To find the alias name for a CMK, see the **Alias** column in the row for each CMK\. If a CMK does not have an alias, a dash \(**\-**\) appears in the **Alias** column\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/find-alias-name-1-sm.png)

1. You can also find the alias on the details page for a CMK\. To open the details page, choose the key ID or alias\. The alias appears in the **General Configuration** section\.   
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/find-alias-name-2.png)

## To derive the alias ARN \(console\)<a name="derive-alias-arn"></a>

The [alias ARN](concepts.md#key-id-alias-ARN) is not displayed in the AWS KMS console\. The **ARN** field displays the [key ARN](concepts.md#key-id-key-ARN)\. However, you can derive the alias ARN for this alias by using the values in the **Alias** and **ARN** fields\.

To derive the alias ARN, begin with the key ARN\. 

```
# Key ARN format
arn:<partition>:kms:<region>:<account>:key/<key-id>

# Example key ARN
arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
```

Replace `key` with `alias`\. Replace the key ID with the alias name\.

```
# Alias ARN format
arn:<partition>:kms:<region>:<account>:alias/<alias-name>

# Example alias ARN
arn:aws:kms:us-west-2:111122223333:alias/master-key-test
```

## To find the alias name and alias ARN \(AWS KMS API\)<a name="find-cmk-arn-api"></a>

To find the [alias name](concepts.md#key-id-alias-name) and [alias ARN](concepts.md#key-id-alias-ARN) of a customer master key \(CMK\), use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. For examples in multiple programming languages, see [Listing aliases](programming-aliases.md#list-aliases) and [Get alias names and ARNs](viewing-keys-cli.md#viewing-keys-list-aliases)\.

By default, the response includes the alias name and alias ARN for every alias in the account and Region\. To get only the aliases for a particular CMK, use the `KeyId` parameter\.

For example, the following command gets only the aliases for an example CMK with key ID `1234abcd-12ab-34cd-56ef-1234567890ab`\. 

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
```