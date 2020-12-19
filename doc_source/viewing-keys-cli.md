# Viewing CMKs with the API<a name="viewing-keys-cli"></a>

You can use the [AWS Key Management Service \(AWS KMS\) API](https://docs.aws.amazon.com/kms/latest/APIReference/) to view your CMKs\. This section demonstrates several operations that return details about existing CMKs\. The examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

**Topics**
+ [ListKeys: Get the ID and ARN of all CMKs](#viewing-keys-list-keys)
+ [DescribeKey: Get detailed information about a CMK](#viewing-keys-describe-key)
+ [GetKeyPolicy: Get the key policy attached to a CMK](#viewing-keys-get-key-policy)
+ [ListAliases: Get alias names and ARNs for CMKs](#viewing-keys-list-aliases)
+ [ListResourceTags: Get the tags on CMKs](#viewing-keys-list-resource-tags)

## ListKeys: Get the ID and ARN of all CMKs<a name="viewing-keys-list-keys"></a>

The [ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html) operation returns the ID and Amazon Resource Name \(ARN\) of all CMKs in the account and Region\. 

For example, this call to the `ListKeys` operation returns the ID and ARN of each CMK in this fictitious account\. For examples in multiple programming languages, see [Getting key IDs and key ARNs of CMKs](programming-keys.md#listing-keys)\.

```
$ aws kms list-keys
        
{
  "Keys": [
    {
      "KeyArn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
      "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab"
    },
    {
      "KeyArn": "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321",
      "KeyId": "0987dcba-09fe-87dc-65ba-ab0987654321"
    },
    {
      "KeyArn": "arn:aws:kms:us-east-2:111122223333:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d",
      "KeyId": "1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d"
    }
}
```

## DescribeKey: Get detailed information about a CMK<a name="viewing-keys-describe-key"></a>

The [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation returns details about the specified CMK\. To identify the CMK, use its [key ID](concepts.md#key-id-key-id), [key ARN](concepts.md#key-id-key-ARN), [alias name](concepts.md#key-id-alias-name), or [alias ARN](concepts.md#key-id-alias-ARN)\. 

For example, this call to `DescribeKey` returns information about a symmetric CMK\. The fields in the response vary with the [customer master key spec](concepts.md#key-spec), [key state](key-state.md), and the [key material origin](concepts.md#key-origin)\. For examples in multiple programming languages, see [Viewing a customer master key](programming-keys.md#describing-keys)\.

```
$ aws kms describe-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab

{
    "KeyMetadata": {
        "Origin": "AWS_KMS",
        "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
        "Description": "",
        "KeyManager": "CUSTOMER",
        "Enabled": true,
        "CustomerMasterKeySpec": "SYMMETRIC_DEFAULT",
        "KeyUsage": "ENCRYPT_DECRYPT",
        "KeyState": "Enabled",
        "CreationDate": 1499988169.234,
        "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
        "AWSAccountId": "111122223333"
        "EncryptionAlgorithms": [
            "SYMMETRIC_DEFAULT"
        ]
    }
}
```

This example calls `DescribeKey` operation on an asymmetric CMK used for signing and verification\. The response includes the signing algorithms that AWS KMS supports for this CMK\.

```
$ aws kms describe-key --key-id 0987dcba-09fe-87dc-65ba-ab0987654321

{
    "KeyMetadata": {        
        "KeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
        "Origin": "AWS_KMS",
        "Arn": "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321",
        "KeyState": "Enabled",
        "KeyUsage": "SIGN_VERIFY",
        "CreationDate": 1569973196.214,
        "Description": "",
        "CustomerMasterKeySpec": "ECC_NIST_P521",
        "AWSAccountId": "111122223333",
        "Enabled": true,
        "KeyManager": "CUSTOMER"
        "SigningAlgorithms": [
            "ECDSA_SHA_512"
        ]
    }
}
```

You can use the `DescribeKey` operation on a predefined AWS alias, that is, an AWS alias with no key ID\. When you do, AWS KMS associates the alias with an [AWS managed CMK](concepts.md#master_keys) and returns its `KeyId` and `Arn` in the response\.

## GetKeyPolicy: Get the key policy attached to a CMK<a name="viewing-keys-get-key-policy"></a>

The [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation gets the key policy that is attached to the CMK\. To identify the CMK, use its key ID or key ARN\. You must also specify the policy name, which is always `default`\. \(If your output is difficult to read, add the `--output text` option to your command\.\)

For examples in multiple programming languages, see [Getting a key policy](programming-key-policies.md#get-policy)\.

```
$ aws kms get-key-policy --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --policy-name default

{
  "Version" : "2012-10-17",
  "Id" : "key-default-1",
  "Statement" : [ {
    "Sid" : "Enable IAM User Permissions",
    "Effect" : "Allow",
    "Principal" : {
      "AWS" : "arn:aws:iam::111122223333:root"
    },
    "Action" : "kms:*",
    "Resource" : "*"
  } ]
}
```

## ListAliases: Get alias names and ARNs for CMKs<a name="viewing-keys-list-aliases"></a>

The [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation returns aliases in the account and region\. The `TargetKeyId` in the response displays the key ID of the CMK that the alias refers to, if any\.

By default, the ListAliases command returns all aliases in the account and region\. This includes [aliases that you created](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html) and associated with your [customer managed CMKs](concepts.md#master_keys), and aliases that AWS created and associated with [AWS managed CMKs](concepts.md#master_keys) in your account\. You can recognize AWS aliases because their names have the format `aws/<service-name>`, such as `aws/dynamodb`\.

The response might also include aliases that have no `TargetKeyId` field, such as the `aws/redshift` alias in this example\. These are predefined aliases that AWS has created but has not yet associated with a CMK\.

For examples in multiple programming languages, see [Listing aliases](programming-aliases.md#list-aliases)\.

```
$ aws kms list-aliases

{
    "Aliases": [
        {
            "AliasName": "alias/access-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/access-key",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
            "CreationDate": 1516435200.399,
            "LastUpdatedDate: 1516435200.399
        },        
        {
            "AliasName": "alias/ECC-P521-Sign",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/ECC-P521-Sign",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "CreationDate": 1693622000.704,
            "LastUpdatedDate: 1693622000.704
        },
        {
            "AliasName": "alias/ImportedKey",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/ImportedKey",
            "TargetKeyId": "1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d",
            "CreationDate": 1493622000.704,
            "LastUpdatedDate: 1521097200.235
        },
        {
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/financeKey",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
            "AliasName": "alias/financeKey",
            "CreationDate": 1604958290.014,
            "LastUpdatedDate: 1604958290.014
        },
        {
            "AliasName": "alias/aws/dynamodb",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/aws/dynamodb",
            "TargetKeyId": "0987ab65-43cd-21ef-09ab-87654321cdef",
            "CreationDate": 1521097200.454,
            "LastUpdatedDate: 1521097200.454
        },
        {
            "AliasName": "alias/aws/ebs",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/aws/ebs",
            "TargetKeyId": "abcd1234-09fe-ef90-09fe-ab0987654321",
            "CreationDate": 1466518990.200,
            "LastUpdatedDate: 1466518990.200
        }
    ]
}
```

To get the aliases that refer to a particular CMK, use the `KeyId` parameter\. The parameter value can be the [key ID](concepts.md#key-id-key-id) or [key ARN](concepts.md#key-id-key-ARN)\. You cannot specify an [alias name](concepts.md#key-id-alias-name) or [alias ARN](concepts.md#key-id-alias-ARN)\.

The command in the following example gets the aliases that refer to a [customer managed CMK](concepts.md#customer-cmk)\. But you can use a command like this one to find the aliases that refer to [AWS managed CMKs](concepts.md#aws-managed-cmk), too\.

```
$ aws kms list-aliases --key-id arn:aws:kms:us-west-2:111122223333:key:0987dcba-09fe-87dc-65ba-ab0987654321

{
    "Aliases": [
        {
            "AliasName": "alias/access-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/access-key",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
            "CreationDate": 1516435200.399,
            "LastUpdatedDate: 1516435200.399
        },
        {
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/financeKey",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
            "AliasName": "alias/financeKey",
            "CreationDate": 1604958290.014,
            "LastUpdatedDate: 1604958290.014
        },
    ]
}
```

To get only the aliases for AWS managed CMKs, use the features of your programming language to filter the response\. 

```
$ aws kms list-aliases --query 'Aliases[?starts_with(AliasName, `alias/aws/`)]'
```

## ListResourceTags: Get the tags on CMKs<a name="viewing-keys-list-resource-tags"></a>

The [ListResourceTags](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation returns the tags on the specified CMK\. The API returns tags for one CMK, but you can run the command in a loop to get tags for all CMKs in the account and Region, or for a set of CMKs you select\. This API returns one page at a time, so if you have numerous tags on numerous CMKs, you might have to use the paginator in your programming language to get all of the tags you want\. 

The ListResourceTags operation returns tags for all AWS KMS CMKs, but [AWS managed CMKs](concepts.md#aws-managed-cmk) are not tagged\. 

To find the tags for a CMK, use the `ListResourceTags` operation\. The `KeyId` parameter is required\. It accepts a [key ID](concepts.md#key-id-key-id) or [key ARN](concepts.md#key-id-key-ARN)\. Before running this example, replace the example key ARN with a valid one\.

```
$  aws kms list-resource-tags --key-id arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
      {
    "Tags": [
        {
            "TagKey": "Department",
            "TagValue": "IT"
        },
        {
            "TagKey": "Purpose",
            "TagValue": "Test"
        }
    ],
    "Truncated": false
}
```

You might want to use the `ListResourceTags` operation to get all CMKs in the account and Region with a particular tag, tag key, or tag value\. To do this, use the filtering features of your programming language\.

For example, the following Bash script uses the [ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html) and `ListResourceTags` operations to get all CMKs in the account and Region with a `Project` tag key\. Both of these operations get only the first page of results\. If you have numerous CMKs or numerous tags, use the pagination features of your language to get the entire result from each operation\. Before running this example, replace the example key IDs with valid ones\.

```
TARGET_TAG_KEY='Project'

for key in $(aws kms list-keys --query 'Keys[*].KeyId' --output text); do
  key_tags=$(aws kms list-resource-tags --key-id "$key" --query "Tags[?TagKey==\`$TARGET_TAG_KEY\`]")
  if [ "$key_tags" != "[]" ]; then
    echo "Key: $key"
    echo "$key_tags"
  fi
done
```

The output is formatted like the following example output\.

```
Key: 0987dcba-09fe-87dc-65ba-ab0987654321
[
    {
        "TagKey": "Project",
        "TagValue": "Gamma"
    }
]
Key: 1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d
[
    {
        "TagKey": "Project",
        "TagValue": "Alpha"
    }
]
Key: 0987ab65-43cd-21ef-09ab-87654321cdef
[
    {
        "TagKey": "Project",
        "TagValue": "Alpha"
    }
]
```