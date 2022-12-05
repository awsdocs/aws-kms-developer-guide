# Managing aliases<a name="alias-manage"></a>

Authorized users can create, view, and delete aliases\. You can also update an alias, that is, associate an existing alias with a different KMS key\.

**Topics**
+ [Creating an alias](#alias-create)
+ [Viewing aliases](#alias-view)
+ [Updating aliases](#alias-update)
+ [Deleting an alias](#alias-delete)

## Creating an alias<a name="alias-create"></a>

You can create aliases in the AWS KMS console or by using AWS KMS API operations\. 

The alias must be string of 1–256 characters\. It can contain only alphanumeric characters, forward slashes \(/\), underscores \(\_\), and dashes \(\-\)\. The alias name for a [customer managed key](concepts.md#customer-cmk) cannot begin with `alias/aws/`\. The `alias/aws/` prefix is reserved for [AWS managed key](concepts.md#aws-managed-cmk)\.

You can create an alias for a new KMS key or for an existing KMS key\. You might add an alias so that a particular KMS key is used in a project or application\. 

### Create an alias \(console\)<a name="alias-create-console"></a>

When you [create a KMS key](create-keys.md) in the AWS KMS console, you must create an alias for the new KMS key\. To create an alias for an existing KMS key, use the **Aliases** tab on the detail page for the KMS key\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\. You cannot manage aliases for AWS managed keys or AWS owned keys\.

1. In the table, choose the key ID or alias of the KMS key\. Then, on the KMS key detail page, choose the **Aliases** tab\.

   If a KMS key has multiple aliases, the **Aliases** column in the table displays one alias and an alias summary, such as **\(\+*n* more\)**\. Choosing the alias summary takes you directly to the **Aliases** tab on the KMS key detail page\.

1. On the **Aliases** tab, choose **Create alias**\. Enter an alias name and choose **Create alias**\.
**Note**  
In the console, you're not required to specify the `alias/` prefix\. The console adds it for you\. If you enter `alias/ExampleAlias`, the actual alias name will be `alias/alias/ExampleAlias`\.

### Create an alias \(AWS KMS API\)<a name="alias-create-api"></a>

To create an alias, use the [CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html) operation\. Unlike the process of creating KMS keys in the console, the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation doesn't create an alias for a new KMS key\.

You can use the `CreateAlias` operation to create an alias for a new KMS key with no alias\. You can also use the `CreateAlias` operation to add an alias to any existing KMS key or to recreate an alias that was accidentally deleted\. 

In the AWS KMS API operations, the alias name must begin with `alias/` followed by a name, such as `alias/ExampleAlias`\. The alias must be unique in the account and Region\. To find the alias names that are already in use, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. The alias name is case sensitive\.

The `TargetKeyId` can be any [customer managed key](concepts.md#customer-cmk) in the same AWS Region\. To identify the KMS key, use its [key ID](concepts.md#key-id-key-id) or [key ARN](concepts.md#key-id-key-ARN)\. You cannot use another alias\.

The following example creates the `example-key` alias and associates it with the specified KMS key\. These examples use the AWS Command Line Interface \(AWS CLI\)\. For examples in multiple programming languages, see [Working with aliases](programming-aliases.md)\.

```
$ aws kms create-alias \
    --alias-name alias/example-key \
    --target-key-id 1234abcd-12ab-34cd-56ef-1234567890ab
```

`CreateAlias` doesn't return any output\. To see the new alias, use the `ListAliases` operation\. For details, see [Viewing aliases \(AWS KMS API\)](#alias-view-api)\.

## Viewing aliases<a name="alias-view"></a>

Aliases make it easy to recognize KMS keys in the AWS KMS console\. You can view the aliases for a KMS key in the AWS KMS console or by using the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. The [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation, which returns the properties of a KMS key, doesn't include aliases\.

### Viewing aliases \(console\)<a name="alias-view-console"></a>

The **Customer managed keys** and **AWS managed keys** pages in the AWS KMS console display the alias associated with each KMS key\. You can also [search, sort, and filter](viewing-keys-console.md#viewing-console-filter) KMS keys based on their aliases\.

The following image of the AWS KMS console shows the aliases on the **Customer managed keys** page of an example account\. As shown in the image, some KMS keys don't have an alias\. 

When a KMS key has multiple aliases, the **Aliases** column displays one alias and an *alias summary* **\(\+*n* more\)**\. The alias summary shows how many additional aliases are associated with the KMS key and links to the display of all aliases for the KMS key on the **Aliases** tab\.

![\[Aliases in the Customer managed keys page of the AWS KMS console\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/alias-console-sm.png)

The **Aliases** tab on the details page for each KMS key displays the alias name and alias ARN of all aliases for the KMS key in the AWS account and Region\. You can also use the **Aliases** tab to [create aliases](#alias-create) and [delete aliases](#alias-delete)\.

To find the alias name and alias ARN of all aliases for the KMS key, use the **Aliases** tab\.
+ To go directly to the **Aliases** tab, in the **Aliases** column, choose the alias summary \(**\+*n* more**\)\. An alias summary appears only if the KMS key has more than one alias\.
+ Or, choose the alias or key ID of the KMS key \(which opens the detail page for the KMS key\) and then choose the **Aliases** tab\. The tabs are under the **General configuration** section\. 

The following image shows the **Aliases** tab for an example KMS key\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/alias-tab-2.png)

You can use the alias to recognize an AWS managed key, as shown in this example **AWS managed keys** page\. The aliases for AWS managed keys always have the format: `aws/<service-name>`\. For example, the alias for the AWS managed key for Amazon DynamoDB is `aws/dynamodb`\.

![\[Aliases in the AWS managed keys page of the AWS KMS console\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/alias-console-aws-managed-sm.png)

### Viewing aliases \(AWS KMS API\)<a name="alias-view-api"></a>

The [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation returns the alias name and alias ARN of aliases in the account and Region\. The output includes aliases for AWS managed keys and for customer managed keys\. The aliases for AWS managed keys have the format `aws/<service-name>`, such as `aws/dynamodb`\.

The response might also include aliases that have no `TargetKeyId` field\. These are predefined aliases that AWS has created but has not yet associated with a KMS key\.

```
$ aws kms list-aliases
{
    "Aliases": [
        {
            "AliasName": "alias/access-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/access-key",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
            "CreationDate": 1516435200.399,
            "LastUpdatedDate": 1516435200.399
        },        
        {
            "AliasName": "alias/ECC-P521-Sign",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/ECC-P521-Sign",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "CreationDate": 1693622000.704,
            "LastUpdatedDate": 1693622000.704
        },
        {
            "AliasName": "alias/ImportedKey",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/ImportedKey",
            "TargetKeyId": "1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d",
            "CreationDate": 1493622000.704,
            "LastUpdatedDate": 1521097200.235
        },
        {
            "AliasName": "alias/finance-project",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/finance-project",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
            "CreationDate": 1604958290.014,
            "LastUpdatedDate": 1604958290.014
        },
        {
            "AliasName": "alias/aws/dynamodb",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/aws/dynamodb",
            "TargetKeyId": "0987ab65-43cd-21ef-09ab-87654321cdef",
            "CreationDate": 1521097200.454,
            "LastUpdatedDate": 1521097200.454
        },
        {
            "AliasName": "alias/aws/ebs",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/aws/ebs",
            "TargetKeyId": "abcd1234-09fe-ef90-09fe-ab0987654321",
            "CreationDate": 1466518990.200,
            "LastUpdatedDate": 1466518990.200
        }
    ]
}
```

To get all aliases that are associated with a particular KMS key, use the optional `KeyId` parameter of the `ListAliases` operation\. The `KeyId` parameter takes the [key ID](concepts.md#key-id-key-id) or [key ARN](concepts.md#key-id-key-ARN) of the KMS key\. 

This example gets all aliases associated with the `0987dcba-09fe-87dc-65ba-ab0987654321` KMS key\.

```
$ aws kms list-aliases --key-id 0987dcba-09fe-87dc-65ba-ab0987654321
{
    "Aliases": [
        {
            "AliasName": "alias/access-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/access-key",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
            "CreationDate": "2018-01-20T15:23:10.194000-07:00",
            "LastUpdatedDate": "2018-01-20T15:23:10.194000-07:00"
        },
        {
            "AliasName": "alias/finance-project",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/finance-project",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
            "CreationDate": 1604958290.014,
            "LastUpdatedDate": 1604958290.014
        }
    ]
}
```

The `KeyId` parameter doesn't take wildcard characters, but you can use the features of your programming language to filter the response\. 

For example, the following AWS CLI command gets only the aliases for AWS managed keys\.

```
$ aws kms list-aliases --query 'Aliases[?starts_with(AliasName, `alias/aws/`)]'
```

The following command gets only the `access-key` alias\. The alias name is case\-sensitive\.

```
$ aws kms list-aliases --query 'Aliases[?AliasName==`alias/access-key`]'
[
    {
        "AliasName": "alias/access-key",
        "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/access-key",
        "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
        "CreationDate": "2018-01-20T15:23:10.194000-07:00",
        "LastUpdatedDate": "2018-01-20T15:23:10.194000-07:00"
    }
]
```

## Updating aliases<a name="alias-update"></a>

Because an alias is an independent resource, you can change the KMS key associated with an alias\. For example, if the `test-key` alias is associated with one KMS key, you can use the [UpdateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html) operation to associate it with a different KMS key\. This is one of several ways to [manually rotate a KMS key](rotate-keys.md) without changing its key material\. You might also update a KMS key so that an application that was using one KMS key for new resources is now using a different KMS key\.

You cannot update an alias in the AWS KMS console\. Also, you cannot use `UpdateAlias` \(or any other operation\) to change an alias name\. To change an alias name, delete the current alias and then create a new alias for the KMS key\.

When you update an alias, the current KMS key and the new KMS key must be the same type \(both symmetric or asymmetric or HMAC\)\. They must also have the same key usage \(`ENCRYPT_DECRYPT` or `SIGN_VERIFY` or GENERATE\_VERIFY\_MAC\)\. This restriction prevents cryptographic errors in code that uses aliases\. 

The following example begins by using the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation to show that the `test-key` alias is currently associated with KMS key `1234abcd-12ab-34cd-56ef-1234567890ab`\. 

```
$ aws kms list-aliases --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
{
    "Aliases": [
        {
            "AliasName": "alias/test-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/test-key",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "CreationDate": 1593622000.191,
            "LastUpdatedDate": 1593622000.191
        }
    ]
}
```

Next, it uses the `UpdateAlias` operation to change the KMS key that is associated with the `test-key` alias to KMS key `0987dcba-09fe-87dc-65ba-ab0987654321`\. You don't need to specify the currently associated KMS key, only the new \("target"\) KMS key\. The alias name is case sensitive\.

```
$ aws kms update-alias --alias-name 'alias/test-key' --target-key-id 0987dcba-09fe-87dc-65ba-ab0987654321
```

To verify that the alias is now associated with the target KMS key, use the `ListAliases` operation again\. This AWS CLI command uses the `--query` parameter to get only the `test-key` alias\. The `TargetKeyId` and `LastUpdatedDate` fields are updated\.

```
$ aws kms list-aliases --query 'Aliases[?AliasName==`alias/test-key`]'
[
    {
        "AliasName": "alias/test-key",
        "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/test-key",
        "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
        "CreationDate": 1593622000.191,
        "LastUpdatedDate": 1604958290.154
    }
]
```

## Deleting an alias<a name="alias-delete"></a>

You can delete an alias in the AWS KMS console or by using the [DeleteAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteAlias.html) operation\. Before deleting an alias, make sure that it's not in use\. Although deleting an alias doesn't affect the associated KMS key, it might create problems for any application that uses the alias\. If you delete an alias by mistake, you can create a new alias with the same name and associate it with the same or a different KMS key\.

If you delete a KMS key, all aliases associated with that KMS key are deleted\.

### Delete aliases \(console\)<a name="alias-delete-console"></a>

To delete an alias in the AWS KMS console, use the **Aliases** tab on the detail page for the KMS key\. You can delete multiple aliases for a KMS key at one time\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\. You cannot manage aliases for AWS managed keys or AWS owned keys\.

1. In the table, choose the key ID or alias of the KMS key\. Then, on the KMS key detail page, choose the **Aliases** tab\.

   If a KMS key has multiple aliases, the **Aliases** column in the table displays one alias and an alias summary, such as **\(\+*n* more\)**\. Choosing the alias summary takes you directly to the **Aliases** tab on the KMS key detail page\.

1. On the **Aliases** tab, select the check box next to the aliases that you want to delete\. Then choose **Delete**\.

### Delete an alias \(AWS KMS API\)<a name="alias-delete-api"></a>

To delete an alias, use the [DeleteAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteAlias.html) operation\. This operation deletes one alias at a time\. The alias name is case\-sensitive and it must be preceded by the `alias/` prefix\.

For example, the following command deletes the `test-key` alias\. This command does not return any output\. 

```
$ aws kms delete-alias --alias-name alias/test-key
```

To verify that the alias is deleted, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. The following command uses the `--query` parameter in the AWS CLI to get only the `test-key` alias\. The empty brackets in the response indicate that the `ListAliases` response didn't include a `test-key` alias\. To eliminate the brackets, use the `--output text` parameter and value\.

```
$ aws kms list-aliases --query 'Aliases[?AliasName==`alias/test-key`]'
[]
```