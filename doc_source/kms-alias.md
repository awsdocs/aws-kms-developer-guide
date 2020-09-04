# Using aliases<a name="kms-alias"></a>

An *alias* is a friendly name for a [customer master key](concepts.md#master_keys) \(CMK\)\. For example, an alias lets you refer to a CMK as `test-key` instead of `1234abcd-12ab-34cd-56ef-1234567890ab`\. 

You can use an alias to identify a CMK in the AWS KMS console, in the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation, and in [cryptographic operations](concepts.md#cryptographic-operations), such as [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) and [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)\. 

Aliases also make it easy to recognize an [AWS managed CMKs](concepts.md#aws-managed-cmk)\. Aliases for these CMKs always have the form: `aws/<service-name>`\. For example, the alias for the AWS managed CMK for Amazon DynamoDB is `aws/dynamodb`\. You can establish similar alias standards for your projects, such as prefacing your aliases with the name of a project or category\.

Much of the power of aliases come from your ability to change the CMK associated with an alias at any time\. Aliases can make your code easier to write and maintain\. For example, suppose you use an alias to refer to a particular CMK and you want to change the CMK\. In that case, just associate the alias with a different CMK\. You don't need to change your code\. 

Aliases also make it easier to reuse the same code in different AWS Regions\. Create aliases with the same name in multiple Regions and associate each alias with a CMK in its Region\. When the code runs in each Region, the alias refers to its associated CMK in that Region\. For an example, see [Using aliases in your applications](#alias-using)\.

The AWS KMS API provides full control of aliases in each account and Region\. The API includes operations to create an alias \([CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html)\), view alias names and alias ARNs \([ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html)\), change the CMK associated with an alias \([UpdateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html)\), and delete an alias \([DeleteAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteAlias.html)\)\. For examples of managing aliases multiple programming languages, see [Working with aliases](programming-aliases.md)\.

The following resources can help you learn more:
+ For information about CMK identifiers, including aliases, see [Key identifiers \(KeyId\)](concepts.md#key-id)\.
+ For help finding the aliases associated with a CMK, see [Finding the alias name and alias ARN](find-cmk-alias.md)
+ For information about resource quotas for aliases and rate quotas for API operations related to aliases, see [Quotas](limits.md)\.
+ For examples of creating and managing aliases in multiple programming languages, see [Working with aliases](programming-aliases.md)\.

**Topics**
+ [About aliases](#alias-about)
+ [Creating an alias](#alias-create)
+ [Viewing aliases](#alias-view)
+ [Using aliases in your applications](#alias-using)
+ [Updating aliases](#alias-update)
+ [Deleting an alias](#alias-delete)
+ [Controlling access to aliases](#alias-access)
+ [Finding aliases in AWS CloudTrail logs](#alias-ct)

## About aliases<a name="alias-about"></a>

Learn how aliases work in AWS KMS\.

**An alias is an independent AWS resource**  
An alias is not a property of a CMK\. The actions that you take on the alias don't affect its associated CMK\. You can create an alias for a CMK and then update the alias so it's associated with a different CMK\. You can even delete the alias without any effect on the associated CMK\.  
Each alias has two formats\.  
+ An *alias ARN* is an Amazon Resource Name \(ARN\) that uniquely identifies the alias\.

  ```
  # Alias ARN
  arn:aws:kms:us-west-2:111122223333:alias/ExampleAlias
  ```
+ An *alias name* that begins with `alias`\. The alias name is unique only to an account and Region\.

  ```
  # Alias name
  alias/ExampleAlias
  ```

**Each alias is associated with one CMK at a time**  
The alias and its CMK must be in the same account and Region\.   
You can associate an alias with any [customer managed CMK](concepts.md#customer-cmk) in the same AWS account and Region\. However, you do not have permission to associate an alias with an [AWS managed CMK](concepts.md#aws-managed-cmk)\.  
For example, this [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) output shows that the `test-key` alias is associated with exactly one target CMK, which is represented by the `TargetKeyId` property\.  

```
{
     "AliasName": "alias/test-key",
     "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/test-key",
     "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab"
}
```

**Multiple aliases can be associated with the same CMK**  
For example, you can associate the `test-key` and `project-key` aliases with the same CMK\. Only one alias appears in the AWS KMS console, but the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) response displays all aliases or all aliases for a given CMK\.  

```
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
```

**An alias must be unique in an account and Region**  
For example, you can have only one `test-key` alias in each Region\. Aliases are case\-sensitive, but aliases that differ only in their capitalization are very prone to error\. You cannot change an alias name\. However, you can delete the alias and create a new alias with the desired name\.

**You can create an alias with the same name in different Regions**  
For example, you can have a `finance-key` alias in US East \(N\. Virginia\) and a `finance-key` alias in Europe \(Frankfurt\)\. Each alias would be associated with a CMK in its Region\. If your code refers to an alias name like `alias/finance-key`, you can run it in multiple Regions\. In each Region, it uses a different CMK\. For details, see [Using aliases in your applications](#alias-using)\.

**You can change the CMK associated with an alias**  
You can use the [UpdateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html) operation to associate an alias with a different CMK\. For example, if the `finance-key` alias is associated with the `1234abcd-12ab-34cd-56ef-1234567890ab` CMK, you can update it so it is associated with the `0987dcba-09fe-87dc-65ba-ab0987654321` CMK\.  
However, the current and new CMK must be the same type \(both symmetric or both asymmetric\), and they must have the same [key usage](concepts.md#key-usage) \(ENCRYPT\_DECRYPT or SIGN\_VERIFY\)\. This restriction prevents errors in code that uses aliases\. If you must associate an alias with a different type of key, and you have mitigated the risks, you can delete and recreate the alias\.

**Some CMKs don't have aliases**  
When you create a CMK in the AWS KMS console, you must give it a new alias\. But an alias is not required when you use the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation to create a CMK\. Also, you can use the [UpdateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html) operation to change the CMK that's associated with an alias and the[DeleteAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteAlias.html) operation to delete an alias\. As a result, some CMKs might have several aliases, and some might have none\.

**AWS creates aliases in your account**  
AWS creates aliases in your account for [AWS managed CMKs](concepts.md#aws-managed-cmk)\. These aliases have names of the form `alias/aws/<service-name>`, such as `alias/aws/s3`\.   
Some AWS aliases have no CMK\. These predefined aliases are usually associated with an AWS managed CMK when you start using the service\. Aliases that AWS creates in your account, including predefined aliases, do not count against your [AWS KMS aliases quota](resource-limits.md#aliases-limit)\.

**Use aliases to identify CMKs**  
You can use an [alias name](concepts.md#key-id-alias-name) or [alias ARN](concepts.md#key-id-alias-ARN) to identify a CMK in AWS KMS [cryptographic operations](concepts.md#cryptographic-operations) and in the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. However, you cannot use an alias names or alias ARNs in API operations that manage CMKs, such as [DisableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) or [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html)\. For information about the valid [key identifiers](concepts.md#key-id) for each AWS KMS API operation, see the descriptions of the `KeyId` parameters in the AWS Key Management Service API Reference\.  
You cannot use an alias name or alias ARN to identify a CMK in the `Resource` element of an [IAM policy](iam-policies.md)\. This restriction prevents errors that might occur if you intended to control access to a particular CMK, but the alias is deleted or updated to a different CMK\.  

## Creating an alias<a name="alias-create"></a>

You can create aliases in the AWS KMS console or by using AWS KMS API operations\. 

The alias must be string of 1–256 characters\. It can contain only alphanumeric characters, forward slashes \(/\), underscores \(\_\), and dashes \(\-\)\. The alias name for a [customer managed CMK](concepts.md#customer-cmk) cannot begin with `alias/aws/`\. The `alias/aws/` prefix is reserved for [AWS managed CMKs](concepts.md#aws-managed-cmk)\.

You can create an alias for a new CMK or for an existing CMK\. You might add an alias so that a particular CMK is used in a project or application\. 

### Creating an alias \(console\)<a name="alias-create-console"></a>

You can create an alias in the AWS KMS console, but only as part of the process of [creating a CMK](create-keys.md), where a new alias is required for every new CMK\. 

You cannot use the AWS KMS console to create a new alias for an existing CMK or to update or delete aliases\. To update or delete aliases, including aliases that you create in the console, use the AWS KMS API operations\.

### Creating an alias \(AWS KMS API\)<a name="alias-create-api"></a>

To create an alias, use the [CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html) operation\. Unlike the process of creating CMKs in the console, the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation doesn't create an alias for a new CMK\. 

You can use the `CreateAlias` operation to create an alias for a new CMK with no alias\. You can also use the `CreateAlias` operation to add an alias to any existing CMK or to recreate an alias that was accidentally deleted\. 

In the AWS KMS API operations, the alias name must begin with `alias/` followed by a name, such as `alias/ExampleAlias`\. The alias must be unique in the account and Region\. To find the alias names that are already in use, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. The alias name is case sensitive\.

The `TargetKeyId` can be any [customer managed CMK](concepts.md#customer-cmk) in the same AWS Region\. To identify the CMK, use its [key ID](concepts.md#key-id-key-id) or [key ARN](concepts.md#key-id-key-ARN)\. You cannot use another alias\.

The following example creates the `example-key` alias and associates it with the specified CMK\. These examples use the AWS Command Line Interface \(AWS CLI\)\. For examples in multiple programming languages, see [Working with aliases](programming-aliases.md)\.

```
$ aws kms create-alias \
    --alias-name alias/example-key \
    --target-key-id 1234abcd-12ab-34cd-56ef-1234567890ab
```

`CreateAlias` doesn't return any output\. To see the new alias, use the `ListAliases` operation\. For details, see [Viewing aliases \(AWS KMS API\)](#alias-view-api)\.

## Viewing aliases<a name="alias-view"></a>

Aliases make it easy to recognize CMKs in the AWS KMS console\. But the console displays only one alias for each CMK\. To see all aliases for a CMK, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. The [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation, which returns detailed information about a CMK, doesn't include aliases\.

### Viewing aliases \(console\)<a name="alias-view-console"></a>

The **Customer managed keys** and **AWS managed keys** pages in the AWS KMS console display the alias associated with each CMK\. You can also search, sort, and filter CMKs based on their alias\.

The following image of the AWS KMS console shows the aliases on the **Customer managed keys** page of an example account\. As shown in the image, some CMKs don't have an alias\.

![\[Aliases in the Customer managed keys page of the AWS KMS console\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/alias-console-sm.png)

The details page for each CMK displays one of the aliases for the CMK\. To open the details page for a CMK, in the CMK table, choose its alias or key ID\.

This page displays one alias name but doesn't display the alias ARN\. For help with finding the alias ARN, see [Finding the alias name and alias ARN](find-cmk-alias.md)\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/alias-console-key-detail.png)

You can use the alias to recognize an AWS managed CMK, as shown in this example **AWS managed keys** page\. The aliases for AWS managed CMKs always have the format: `aws/<service-name>`\. For example, the alias for the AWS managed CMK for Amazon DynamoDB is `aws/dynamodb`\.

![\[Aliases in the AWS managed keys page of the AWS KMS console\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/alias-console-aws-managed-sm.png)

The AWS KMS console requires that you specify an alias when you create a CMK\. It also displays the alias name \(if any\) that is associated with the CMK\. But you cannot use the console to create additional aliases, change the CMK associated with an alias, or delete an alias\. Those tasks can be done only with the AWS KMS API\.

Also, the AWS KMS console displays only one alias for each CMK and it does not display the alias ARN\. To view all aliases for a particular CMK or to view the alias ARN for any alias in the account and Region, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\.

### Viewing aliases \(AWS KMS API\)<a name="alias-view-api"></a>

The [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation returns the alias name and alias ARN of aliases in the account and Region\. The output includes aliases for AWS managed CMKs and for customer managed CMKs\. The aliases for AWS managed CMKs have the format `aws/<service-name>`, such as `aws/dynamodb`\.

The response might also include aliases that have no `TargetKeyId` field\. These are predefined aliases that AWS has created but has not yet associated with a CMK\. Aliases that AWS creates in your account, including predefined aliases, do not count against your [AWS KMS aliases quota](resource-limits.md#aliases-limit)\.

```
$ aws kms list-aliases
{
    "Aliases": [
        {
            "AliasName": "alias/access-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/access-key",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321"
        },        
        {
            "AliasName": "alias/ECC-P521-Sign",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/ECC-P521-Sign",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab"
        },
        {
            "AliasName": "alias/ImportedKey",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/ImportedKey",
            "TargetKeyId": "1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d"
        },
        {
            "AliasName": "alias/finance-project",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/finance-project",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321"
        },
        {
            "AliasName": "alias/aws/dynamodb",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/aws/dynamodb",
            "TargetKeyId": "0987ab65-43cd-21ef-09ab-87654321cdef"
        },
        {
            "AliasName": "alias/aws/ebs",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/aws/ebs",
            "TargetKeyId": "abcd1234-09fe-ef90-09fe-ab0987654321"        
        }
    ]
}
```

To get all aliases that are associated with a particular CMK, use the optional `KeyId` parameter of the `ListAliases` operation\. The `KeyId` parameter takes the [key ID](concepts.md#key-id-key-id) or [key ARN](concepts.md#key-id-key-ARN) of the CMK\. 

This example gets all aliases associated with the `0987dcba-09fe-87dc-65ba-ab0987654321` CMK\.

```
$ aws kms list-aliases --key-id 0987dcba-09fe-87dc-65ba-ab0987654321
{
    "Aliases": [
        {
            "AliasName": "alias/access-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/access-key",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321"
        },
        {
            "AliasName": "alias/finance-project",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/finance-project",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321"
        }
    ]
}
```

The `KeyId` parameter doesn't take wildcard characters, but you can use the features of your programming language to filter the response\. 

For example, the following AWS CLI command gets only the aliases for AWS managed CMKs\.

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
        "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321"
    }
]
```

## Using aliases in your applications<a name="alias-using"></a>

You can use an alias to represent a CMK in your application code\. The `KeyId` parameter in the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) and [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) operations and in all [cryptographic operations](concepts.md#cryptographic-operations) accepts an alias name or alias ARN\.

For example, the following `GenerateDataKey` command uses an alias name \(`alias/finance`\) to identify a CMK\. The alias name is the value of the `KeyId` parameter\. 

```
$ aws kms generate-data-key --key-id alias/finance --key-spec AES_256
```

One of the most powerful uses of aliases is in applications that run in multiple AWS Regions\. For example, you might have a global application that uses an RSA [asymmetric CMK](symm-asymm-concepts.md#asymmetric-cmks) for signing and verification\. 
+ In US West \(Oregon\) \(us\-west\-2\), you want to use `arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab`\. 
+ In Europe \(Frankfurt\) \(eu\-central\-1\), you want to use `arn:aws:kms:eu-central-1:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321`
+ In Asia Pacific \(Singapore\) \(ap\-southeast\-1\), you want to use `arn:aws:kms:ap-southeast-1:111122223333:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d`\.

You could create a different version of your application in each Region or use a dictionary or switch statement to select the right CMK for each Region\. But it's much easier to create an alias with the same alias name in each Region\. Remember that the alias name is case\-sensitive\.

```
aws --region us-west-2 kms create-alias \
    --alias-name alias/new-app \
    --key-id arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab

aws --region eu-central-1 kms create-alias \
    --alias-name alias/new-app \
    --key-id arn:aws:kms:eu-central-1:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321

aws --region ap-southeast-1 kms create-alias \
    --alias-name alias/new-app \
    --key-id arn:aws:kms:ap-southeast-1:111122223333:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d
```

Then, use the alias in your code\. When your code runs in each Region, the alias will refer to its associated CMK in that Region\. For example, this code calls the [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html) operation with an alias name\.

```
aws kms sign --key-id alias/new-app \
    --message $message \
    --message-type RAW \
    --signing-algorithm RSASSA_PSS_SHA_384
```

However, there is a risk that the alias might be deleted or updated to be associated with a different CMK\. In that case, the application's attempts to verify signatures using the alias name will fail, and you might need to recreate or update the alias\.

To mitigate this risk, be cautious about giving principals permission to manage the aliases that you use in your application\. For details, see [Controlling access to aliases](#alias-access)\.

There are several other solutions for applications that encrypt data in multiple AWS Regions, including the [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/)\.

## Updating aliases<a name="alias-update"></a>

Because an alias is an independent resource, you can change the CMK associated with an alias\. For example, if the `test-key` alias is associated with one CMK, you can use the [UpdateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html) operation to associate it with a different CMK\. This is one of several ways to [manually rotate a CMK](rotate-keys.md) without changing its key material\. You might also update a CMK so that an application that was using one CMK for new resources is now using a different CMK\.

You cannot update an alias in the AWS KMS console\. Also, you cannot use `UpdateAlias` \(or any other operation\) to change an alias name\. To change an alias name, delete the current alias and then create a new alias for the CMK\.

When you update an alias, the current CMK and the new CMK must be the same type \(both symmetric or both asymmetric\)\. They must also have the same key usage \(`ENCRYPT_DECRYPT` or `SIGN_VERIFY`\)\. This restriction prevents cryptographic errors in code that uses aliases\. 

The following example begins by using the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation to show that the `test-key` alias is currently associated with CMK `1234abcd-12ab-34cd-56ef-1234567890ab`\. 

```
$ aws kms list-aliases --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
{
    "Aliases": [
        {
            "AliasName": "alias/test-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/test-key",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab"
        }
    ]
}
```

Next, it uses the `UpdateAlias` operation to change the CMK that is associated with the `test-key` alias to CMK `0987dcba-09fe-87dc-65ba-ab0987654321`\. You don't need to specify the currently associated CMK, only the new \("target"\) CMK\. The alias name is case sensitive\.

```
$ aws kms update-alias --alias-name 'alias/test-key' --target-key-id 0987dcba-09fe-87dc-65ba-ab0987654321
```

To verify that the alias is now associated with the target CMK, use the `ListAliases` operation again\. This AWS CLI command uses the `--query` parameter to get only the `test-key` alias\.

```
$ aws kms list-aliases --query 'Aliases[?AliasName==`alias/test-key`]'
[
    {
        "AliasName": "alias/test-key",
        "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/test-key",
        "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321"
    }
]
```

## Deleting an alias<a name="alias-delete"></a>

To delete an alias, use the [DeleteAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteAlias.html) operation\. There is no way to delete an alias in the AWS KMS console\. Before deleting an alias, make sure that it's not in use\. Although deleting an alias doesn't affect the associated CMK, it might create problems for any application that uses the alias\.

If you delete an alias by mistake, you can create a new alias with the same name and associate it with the same or a different CMK\.

For example, the following command deletes the `test-key` alias\. This command does not return any output\. The alias name is case\-sensitive\.

```
$ aws kms delete-alias --alias name alias/test-key
```

To verify that the alias is deleted, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. The following command use the `--query` parameter in the AWS CLI to get only the `test-key` alias\. The empty brackets in the response indicate that the `ListAliases` response didn't include a `test-key` alias\. To eliminate the brackets, use the `--output text` parameter and value\.

```
$ aws kms list-aliases --query 'Aliases[?AliasName==`alias/test-key`]'
[]
```

## Controlling access to aliases<a name="alias-access"></a>

When you create or change an alias, you affect the alias and its associated CMK\. Therefore, principals who manage aliases must have permission to call the alias operation on the alias and on all affected CMKs\. You can provide these permissions by using [key policies](key-policies.md), [IAM policies](iam-policies.md) and [grants](grants.md)\. 

For information about controlling access to all AWS KMS operations, see [AWS KMS API permissions reference](kms-api-permissions-reference.md)\.

Permissions to create and manage aliases work as follows\.

**kms:CreateAlias**  
To create an alias, the principal needs the following permissions for both the alias and for the associated CMK\.   
+ `kms:CreateAlias` for the alias\. Provide this permission in an IAM policy that is attached to the principal who is allowed to create the alias\.

  The following example policy statement specifies the alias in a `Resource` element\. But you can specify a Resource value of `"*"` to allow the principal to create any alias in the account and Region\. Permission to create an alias can also be included in a `kms:Create*` permission for all resources in an account and Region\.

  ```
  {
    "Sid": "IAM policy for an alias",
    "Effect": "Allow",
    "Action": [
      "kms:CreateAlias",
      "kms:UpdateAlias"
      "kms:DeleteAlias"
    ],
    "Resource": "arn:aws:kms:us-west-2:111122223333:alias/test-key"
  }
  ```
+ `kms:CreateAlias` for the CMK\. This permission must be provided in a key policy or in an IAM policy that is delegated from the key policy\.

  ```
  {
    "Sid": "Key policy for 1234abcd-12ab-34cd-56ef-1234567890ab",
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::111122223333:user/KMSAdminUser"},
    "Action": [
      "kms:CreateAlias",
      "kms:DescribeKey"
    ],
    "Resource": "*"
  }
  ```

**kms:ListAliases**  
To list aliases in the account and Region, the principal must have `kms:ListAliases` permission in an IAM policy\. This policy isn't related to any particular CMK or alias resource\.  
For example, the following IAM policy statement gives the principal permission to list all CMKs and aliases in the account and Region\.  

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
      "kms:ListKeys",
      "kms:ListAliases"
    ],
    "Resource": "*"
  }
}
```

**kms:UpdateAlias**  
To change the CMK that is associated with an alias, the principal needs three permission elements: one for the alias, one for the current CMK, and one for the new CMK\.  
For example, suppose you want to change the `test-key` alias from the CMK with key ID 1234abcd\-12ab\-34cd\-56ef\-1234567890ab to the CMK with key ID 0987dcba\-09fe\-87dc\-65ba\-ab0987654321\. In that case, include policy statements similar to the examples in this section\.  
+ `kms:UpdateAlias` for the alias\. You provide this permission in an IAM policy that is attached to the principal\. The following IAM policy specifies a particular alias\. But you list multiple alias ARNs or use a `Resource` value of `"*"` to apply the permission to all aliases in the account and Region\.

  ```
  {
    "Sid": "IAM policy for an alias",
    "Effect": "Allow",
    "Action": [
      "kms:UpdateAlias",
      "kms:ListAliases",
      "kms:ListKeys",
    ],
    "Resource": "arn:aws:kms:us-west-2:111122223333:alias/test-key"
  }
  ```
+ `kms:UpdateAlias` for the CMK that is currently associated with the alias\. This permission must be provided in a key policy or in an IAM policy that is delegated from the key policy\.

  ```
  {
    "Sid": "Key policy for 1234abcd-12ab-34cd-56ef-1234567890ab",
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::111122223333:user/KMSAdminUser"},
    "Action": [
      "kms:UpdateAlias",
      "kms:DescribeKey"
    ],
    "Resource": "*"
  }
  ```
+ `kms:UpdateAlias` for the CMK that the operation associates with the alias\. This permission must be provided in a key policy or in an IAM policy that is delegated from the key policy\.

  ```
  {
    "Sid": "Key policy for 0987dcba-09fe-87dc-65ba-ab0987654321",
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::111122223333:user/KMSAdminUser"},
    "Action": [
      "kms:UpdateAlias", 
      "kms:DescribeKey"
    ],
    "Resource": "*"
  }
  ```

**kms:DeleteAlias**  
To delete an alias, the principal needs permission for the alias and for the associated CMK\.   
As always, you should exercise caution when giving principals permission to delete a resource\. However, deleting an alias has no effect on the associated CMK\. Although it might cause a failure in an application that relies on the alias, if you mistakenly delete an alias, you can recreate it\.   
+ `kms:DeleteAlias` for the alias\. Provide this permission in an IAM policy attached to the principal who is allowed to delete the alias\.

  The following example policy statement specifies the alias in a `Resource` element\. But you can specify a Resource value of `"*"` to allow the principal to create any alias in the account and Region\.

  ```
  {
    "Sid": "IAM policy for an alias",
    "Effect": "Allow",
    "Action": [
      "kms:CreateAlias",
      "kms:UpdateAlias"
      "kms:DeleteAlias"
    ],
    "Resource": "arn:aws:kms:us-west-2:111122223333:alias/test-key"
  }
  ```
+ `kms:DeleteAlias` for the associated CMK\. This permission must be provided in a key policy or in an IAM policy that is delegated from the key policy\.

  ```
  {
    "Sid": "Key policy for 1234abcd-12ab-34cd-56ef-1234567890ab",
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::111122223333:user/KMSAdminUser"},
    "Action": [
      "kms:CreateAlias",
      "kms:UpdateAlias",
      "kms:DeleteAlias",
      "kms:DescribeKey"
    ],
    "Resource": "*"
  }
  ```

## Finding aliases in AWS CloudTrail logs<a name="alias-ct"></a>

When you use an alias to represent a customer master key \(CMK\) in an AWS KMS API operation, the alias and the key ARN of the CMK are recorded in the AWS CloudTrail log entry for the event\. The alias appears in the `requestParameters` field\. The key ARN appears in the `resources` field\. This is true even when an AWS service uses an AWS managed CMK in your account\. 

For example, the following [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) request uses the `project-key` alias to represent a CMK\.

```
$ aws kms generate-data-key --key-id alias/project-key --key-spec AES_256
```

When this request is recorded in the CloudTrail log, the log entry includes both the alias and the key ARN of the actual CMK that was used\. 

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "ABCDE",
        "arn": "arn:aws:iam::111122223333:role/ProjectDev",
        "accountId": "111122223333",
        "accessKeyId": "FFHIJ",
        "userName": "example-dev"
    },
    "eventTime": "2020-06-29T23:36:41Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "GenerateDataKey",
    "awsRegion": "us-west-2",
    "sourceIPAddress": "205.205.123.000",
    "userAgent": "aws-cli/1.18.89 Python/3.6.10 Linux/4.9.217-0.1.ac.205.84.332.metal1.x86_64 botocore/1.17.12",
    "requestParameters": {
        "keyId": "alias/project-key",
        "keySpec": "AES_256"
    },
    "responseElements": null,
    "requestID": "d93f57f5-d4c5-4bab-8139-5a1f7824a363",
    "eventID": "d63001e2-dbc6-4aae-90cb-e5370aca7125",
    "readOnly": true,
    "resources": [
        {
            "accountId": "111122223333",
            "type": "AWS::KMS::Key",
            "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
        }
    ],
    "eventType": "AwsApiCall",
    "recipientAccountId": "111122223333"
}
```

For details about logging AWS KMS operations in CloudTrail logs, see [Logging AWS KMS API calls with AWS CloudTrail](logging-using-cloudtrail.md)\.