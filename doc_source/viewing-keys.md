# Viewing Keys<a name="viewing-keys"></a>

You can use [AWS Management Console](https://console.aws.amazon.com/kms) or the [AWS Key Management Service \(AWS KMS\) API](https://docs.aws.amazon.com/kms/latest/APIReference/) to view customer master keys \(CMKs\), including CMKs that you manage and CMKs that are managed by AWS\.

**Topics**
+ [Viewing CMKs \(Console\)](#viewing-keys-console)
+ [Viewing CMKs \(KMS API\)](#viewing-keys-cli)
+ [Finding the Key ID and ARN](#find-cmk-id-arn)

## Viewing CMKs \(Console\)<a name="viewing-keys-console"></a>

You can see a list of your customer managed keys in the AWS Management Console\.

**Note**  
AWS KMS recently introduced a new console that makes it easier for you to organize and manage your KMS resources\. We encourage you to try it at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.  
The original console will remain available for a brief period to give you time to familiarize yourself with the new one\. To use the original console, choose **Encryption Keys** in the IAM console or go to [https://console\.aws\.amazon\.com/iam/home\#encryptionKeys](https://console.aws.amazon.com/iam/home#encryptionKeys)\. Please share your feedback by choosing **Feedback** in either console or in the lower\-right corner of this page\.

### To view your CMKs \(new console\)<a name="viewing-keys-kms-console"></a>

**To navigate to the CMK display**

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. To view the keys in your account that you create and manage, in the navigation pane choose **Customer managed keys**\. To view the keys in your account that AWS creates and manages for you, in the navigation pane, choose **AWS managed keys**\.
**Tip**  
To view [AWS managed CMKs](concepts.md#aws-managed-cmk) that are missing an alias, use the **Customer managed keys** page\.

**To find your CMKs**
+ On the **AWS managed keys** or **Customer managed keys** page, in the filter box, enter all or part of the alias name or key ID of a CMK\. The filter searches all [AWS managed CMKs](concepts.md#aws-managed-cmk) or all [customer managed CMKs](concepts.md#customer-cmk), even if you have too many to display on the current page\. It displays only the CMKs with alias names or key IDs that include the filtered phrase\.

  For example, when you type `exam` in the filter on the **Customer managed keys** page, only customer managed CMKs with `exam` in their alias or key ID fields are displayed, as shown in the following image\.  
![\[\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-filter.png)

**To customize the CMK display**

The display shows all the CMKs of each type in your AWS account and region\. By default, the page displays the alias, key ID, status, and creation date for each CMK, but you can customize it to show the information that you need\.

1. On the **AWS managed keys** or **Customer managed keys** page, choose the settings icon \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-icon-settings-new.png)\) in the upper\-right corner of the page\.

1. On the **Preferences** page, choose your preferred settings, and then choose **Confirm**\.

**To display more information about your CMK**
+ On the **AWS managed keys** or **Customer managed keys** page, choose the alias or key ID of the CMK\. 

The details include the CMK ID, Amazon Resource Name \(ARN\), alias, description, key policy, tags, and key rotation settings of a CMK\.

The Alias section lists only one alias\. To find all aliases associated with the CMK, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\.

### To view your CMKs \(original console\)<a name="viewing-keys-iam-console"></a>

**To navigate to the CMK display**

1. Sign in to the AWS Management Console and go to [https://console\.aws\.amazon\.com/iam/home\#encryptionKeys](https://console.aws.amazon.com/iam/home#encryptionKeys)\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

The **Encryption Keys** page lists the AWS managed and customer managed CMKs in your AWS account and region\. By default, the page displays the alias, key ID, status, and creation date for each CMK, but you can customize it to meet your needs\.

**To customize the CMK display \(optional\)**

1. Choose the settings button \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-icon-settings.png)\) in the upper\-right corner of the page\.

1. On the **Preferences** page, select your preferred options, and then choose **Close**\.

**To show detailed information about the CMK**

The details include the Amazon Resource Name \(ARN\), description, key policy, tags, and key rotation settings of the CMK\.
+ On the **Encryption keys** page, choose the alias or key ID of the CMK\.

**To find CMKs**

You can use the **Filter** box to find CMKs based on their aliases\.
+ In the **Filter** box, type all or part of the alias name of a CMK\. Only the CMKs with alias names that match the filter appear\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/filter-alias.png)

## Viewing CMKs \(KMS API\)<a name="viewing-keys-cli"></a>

You can use the [AWS Key Management Service \(AWS KMS\) API](https://docs.aws.amazon.com/kms/latest/APIReference/) to view your CMKs\. This section demonstrates several operations that return details about existing CMKs\. The examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

**Topics**
+ [ListKeys: Get the ID and ARN of All CMKs](#viewing-keys-list-keys)
+ [DescribeKey: Get Detailed Information About a CMK](#viewing-keys-describe-key)
+ [GetKeyPolicy: Get the Key Policy Attached to a CMK](#viewing-keys-get-key-policy)
+ [ListAliases: View CMKs by Alias Name](#viewing-keys-list-aliases)

### ListKeys: Get the ID and ARN of All CMKs<a name="viewing-keys-list-keys"></a>

The [ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html) operation returns the ID and Amazon Resource Name \(ARN\) of all CMKs in the account and region\. To see the aliases and key IDs of your CMKs that have aliases, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. 

For example, this call to the `ListKeys` operation returns the ID and ARN of each CMK in this fictitious account\.

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

### DescribeKey: Get Detailed Information About a CMK<a name="viewing-keys-describe-key"></a>

The [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation returns details about the specified CMK\. To identify the CMK, use its key ID, key ARN, alias name, or alias ARN\. 

For example, this call to `DescribeKey` returns information about an existing CMK\. The fields in the response vary with the key state and the key origin\.

```
$ aws kms describe-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab

{
    "KeyMetadata": {
        "Origin": "AWS_KMS",
        "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
        "Description": "",
        "KeyManager": "CUSTOMER",
        "Enabled": true,
        "KeyUsage": "ENCRYPT_DECRYPT",
        "KeyState": "Enabled",
        "CreationDate": 1499988169.234,
        "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
        "AWSAccountId": "111122223333"
    }
}
```

You can use the `DescribeKey` operation on a predefined AWS alias, that is, an AWS alias with no key ID\. When you do, AWS KMS associates the alias with an [AWS managed CMK](concepts.md#master_keys) and returns its `KeyId` and `Arn` in the response\.

### GetKeyPolicy: Get the Key Policy Attached to a CMK<a name="viewing-keys-get-key-policy"></a>

The [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation gets the key policy that is attached to the CMK\. To identify the CMK, use its key ID or key ARN\. You must also specify the policy name, which is always `default`\. \(If your output is difficult to read, add the `--output text` option to your command\.\)

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

### ListAliases: View CMKs by Alias Name<a name="viewing-keys-list-aliases"></a>

The [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation returns aliases in the account and region\. The `TargetKeyId` in the response displays the key ID of the CMK that the alias refers to, if any\.

By default, the ListAliases command returns all aliases in the account and region\. This includes [aliases that you created](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html) and associated with your [customer managed CMKs](concepts.md#master_keys), and aliases that AWS created and associated with [AWS managed CMKs](concepts.md#master_keys) in your account\. You can recognize AWS aliases because their names have the format `aws/<service-name>`, such as `aws/dynamodb`\.

The response might also include aliases that have no `TargetKeyId` field, such as the `aws/redshift` alias in this example\. These are predefined aliases that AWS has created but has not yet associated with a CMK\.

```
$ aws kms list-aliases

{
    "Aliases": [
        {
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/ImportedKey",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
            "AliasName": "alias/ExampleKey"
        },
        {
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/test-key",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "AliasName": "alias/test-key"
        },
        {
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/financeKey",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "AliasName": "alias/financeKey"
        },
        {
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/aws/dynamodb",
            "TargetKeyId": "1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d",
            "AliasName": "alias/aws/dynamodb"
        },
        {
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/aws/redshift",
            "AliasName": "alias/aws/redshift"
        },
        {
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/aws/s3",
            "TargetKeyId": "0987ab65-43cd-21ef-09ab-87654321cdef",
            "AliasName": "alias/aws/s3"
        }
    ]
}
```

To get the aliases that refer to a particular CMK, use the `KeyId` parameter\. The parameter value can be the Amazon Resource Name \(ARN\) of the CMK or the CMK ID\. You cannot specify an alias or alias ARN\.

The command in the following example gets the aliases that refer to a customer managed CMK\. But you can use a command like this one to find the aliases that refer to AWS managed CMKs, too\.

```
$ aws kms list-aliases --key-id arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab

{
    "Aliases": [
        {
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/test-key",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "AliasName": "alias/test-key"
        },
        {
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/financeKey",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "AliasName": "alias/financeKey"
        },
    ]
}
```

## Finding the Key ID and ARN<a name="find-cmk-id-arn"></a>

To identify your AWS KMS CMKs in programs, scripts, and command line interface \(CLI\) commands, you use the ID of the CMK or its Amazon Resource Name \(ARN\)\. Cryptographic operations also let you use the CMK alias\.

**Note**  
AWS KMS recently introduced a new console that makes it easier for you to organize and manage your KMS resources\. We encourage you to try it at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.  
The original console will remain available for a brief period to give you time to familiarize yourself with the new one\. To use the original console, choose **Encryption Keys** in the IAM console or go to [https://console\.aws\.amazon\.com/iam/home\#encryptionKeys](https://console.aws.amazon.com/iam/home#encryptionKeys)\. Please share your feedback by choosing **Feedback** in either console or in the lower\-right corner of this page\.

### To find the CMK ID and ARN \(new console\)<a name="find-cmk-arn"></a>

1. Open the AWS KMS console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. To view the keys in your account that you create and manage, in the navigation pane choose **Customer managed keys**\. To view the keys in your account that AWS creates and manages for you, in the navigation pane, choose **AWS managed keys**\.

1. To find the key ID, see the row that begins with the CMK alias\. Each row displays the key ID and alias, along with the status and creation date of each CMK\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/find-key-id-new.png)

1. To find the Amazon Resource Name \(ARN\) of the CMK, choose the key ID or alias\. This opens a page of details that includes the ARN\.   
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/find-key-arn-new.png)

### To find the CMK ID and ARN \(original console\)<a name="find-cmk-arn-iam-console"></a>

**To navigate to the CMK display**

1. Go to the original AWS KMS console at https://console\.aws\.amazon\.com/iam/home\#encryptionKeys\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. To find key ID, look on the row for the CMK alias\. Each row displays the key ID and alias, along with the status and creation date of each CMK\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/find-key-id.png)

1. To find the CMK ARN \(key ARN\), choose the alias name\. This opens a page of details that includes the key ARN\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/find-key-arn.png)

### To find the CMK ID and ARN \(KMS API\)<a name="find-cmk-arn-api"></a>

**Use the ListKeys API operation**
+ To find the CMK ID and ARN, use the [ListKeys](#viewing-keys-list-keys) operation\.