# Viewing CMKs in a custom key store<a name="view-cmk-keystore"></a>

To view the customer master keys \(CMKs\) in a custom key store, use the same techniques that you would use to view any AWS KMS [customer managed CMKs](concepts.md#master_keys)\. To learn the basics, see [Viewing keys](viewing-keys.md)\. To identify the keys in your AWS CloudHSM cluster that serve as key material for your CMK, see [Finding CMKs and key material](find-key-material.md)\.

In the AWS Management Console, the CMKs in your custom key store are displayed along with all other customer managed CMKs your AWS account and Region\. 

However, the following values are specific to CMKs in a custom key store\.
+ The name and ID of the custom key store that stores the CMK\.
+ The cluster ID of the associated AWS CloudHSM cluster that contains their key material\.
+ An `Origin` value of `CloudHSM` in the AWS Management Console or `AWS_CLOUDHSM` in API responses\.
+ The [key state](key-state.md) value can be `Unavailable`\. For help resolving the status, see [How to fix unavailable CMKs](fix-keystore.md#fix-unavailable-cmks)\.

**To view the CMKs in a custom key store \(Console\)**

1. Open the AWS KMS console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. In the upper\-right corner, choose the gear icon, choose **Custom key store ID** and **Origin**, then choose **Confirm**\.

1. To identify CMKs in any custom key store, look for CMKs with an **Origin** value of **AWS\_CLOUDHSM**\. To identify CMKs in a particular custom key store, view the values in the **Custom key store ID** column\. 

1. Choose the alias or key ID of a CMK in a custom key store\. 

   This page displays detailed information about the CMK, including its Amazon Resource Name \(ARN\), key policy, and tags\.

1. Expand **Cryptographic configuration**\.

   This section includes information about the CMK's custom key store and cluster\.

**To view the CMKs in a custom key store \(API\)**

You use the same AWS KMS API operations to view the CMKs in a custom key store that you would use for any CMK, including [ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html), [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html), and [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html)\. For example, the following `describe-key` operation in the AWS CLI shows the special fields for a CMK in a custom key store\. Before running a command like this one, replace the example CMK ID with a valid value\.

```
$ aws kms describe-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab

{
   "KeyMetadata": { 
      "AWSAccountId": "111122223333",
      "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
      "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
      "CreationDate": 1537582718.431,
      "Enabled": true,
      "KeyManager": "CUSTOMER",
      "KeyState": "Enabled",
      "KeyUsage": "ENCRYPT_DECRYPT",
      "Origin": "AWS_CLOUDHSM",
      "CloudHsmClusterId": "cluster-1a23b4cdefg",
      "CustomKeyStoreId": "cks-1234567890abcdef0",
      "Description": "CMK in custom key store"
      "CustomerMasterKeySpec": "SYMMETRIC_DEFAULT",
      "EncryptionAlgorithms": [
         "SYMMETRIC_DEFAULT"
      ]
   }
}
```

For help finding the CMKs in a custom key store or identifying the keys in your AWS CloudHSM cluster that serve as key material for your CMK, see [Finding CMKs and key material](find-key-material.md)\.