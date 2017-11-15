# Viewing Keys<a name="viewing-keys"></a>

You can use the [**Encryption keys** section of the AWS Management Console](https://console.aws.amazon.com/iam/home#encryptionKeys) to view customer master keys \(CMKs\), including CMKs that you manage and CMKs that are managed by AWS\. You can also use the operations in the [AWS Key Management Service \(AWS KMS\) API](http://docs.aws.amazon.com/kms/latest/APIReference/), such as [ListKeys](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html) and [DescribeKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html), to view CMKs\.


+ [Viewing CMKs \(Console\)](#viewing-keys-console)
+ [Viewing CMKs \(API\)](#viewing-keys-cli)
+ [Finding the Key ID and ARN](#find-cmk-id-arn)

## Viewing CMKs \(Console\)<a name="viewing-keys-console"></a>

You can see a list of your customer managed keys in the AWS Management Console\.

**To view your CMKs \(console\)**

1. Sign in to the AWS Management Console and open the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

The console shows all the CMKs in your AWS account in the chosen region, including customer\-managed and AWS managed CMKs\. The page displays the alias, key ID, status, and creation date for each CMK\.

**To show additional columns in the list of CMKs**

1. Choose the settings button \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-icon-settings.png)\) in the upper\-right corner of the page\.

1. Select the check boxes for the additional columns to show, and then choose **Close**\.

**To show detailed information about the CMK**

The details include the Amazon Resource Name \(ARN\), description, key policy, tags, and key rotation settings of the CMK\.

+ Choose the alias of the CMK\.   
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/choose-alias.png)

  If the CMK does not have an alias, choose the empty cell in the **Alias** column, as shown in the following image\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/no-alias.png)

**To find CMKs**

You can use the **Filter** box to find CMKs based on their aliases\.

+ In the **Filter** box, type all or part of the alias name of a CMK\. Only the CMKs with alias names that match the filter appear\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/filter-alias.png)

## Viewing CMKs \(API\)<a name="viewing-keys-cli"></a>

You can use the [AWS Key Management Service \(AWS KMS\) API](http://docs.aws.amazon.com/kms/latest/APIReference/) to view your CMKs\. This section demonstrates several operations that return details about existing CMKs\. The examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 


+ [ListKeys: Get the ID and ARN of All CMKs](#viewing-keys-list-keys)
+ [DescribeKey: Get Detailed Information About a CMK](#viewing-keys-describe-key)
+ [GetKeyPolicy: Get the Key Policy Attached to a CMK](#viewing-keys-get-key-policy)

### ListKeys: Get the ID and ARN of All CMKs<a name="viewing-keys-list-keys"></a>

The [ListKeys](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html) operation returns the ID and Amazon Resource Name \(ARN\) of all CMKs in the account and region\. To see the aliases and key IDs of your CMKs that have aliases, use the [ListAliases](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. 

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

The [DescribeKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation returns details about the specified CMK\. To identify the CMK, use its key ID, key ARN, alias name, or alias ARN\. 

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

### GetKeyPolicy: Get the Key Policy Attached to a CMK<a name="viewing-keys-get-key-policy"></a>

The [GetKeyPolicy](http://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation gets the key policy that is attached to the CMK\. To identify the CMK, use its key ID or key ARN\. You must also specify the policy name, which is always `default`\. \(If your output is difficult to read, add the `--output text` option to your command\.\)

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

## Finding the Key ID and ARN<a name="find-cmk-id-arn"></a>

To identify your AWS KMS CMKs in programs, scripts, and command line interface \(CLI\) commands, you use the ID of the CMK or its Amazon Resource Name \(ARN\)\. Some API operations also let you use the CMK alias\.

**To find the CMK ID and ARN \(console\)**

1. Open the **Encryption Keys** section of the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/home\#encryptionKeys](https://console.aws.amazon.com/iam/home#encryptionKeys)\.

1. For **Region**, choose the appropriate AWS region\. Do not use the region selector in the navigation bar \(top right corner\)\.

   The page displays the key ID and alias, along with the status and creation date of each CMK\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/find-key-id.png)

1. To find the CMK ARN \(key ARN\), choose the alias name\. This opens a page of details that includes the key ARN\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/find-key-arn.png)

**To find the CMK ID and ARN \(API\)**

+ To find the CMK ID and ARN, use the ListKeys operation \(see above\)\.