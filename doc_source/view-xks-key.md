# Viewing KMS keys in an external key store<a name="view-xks-key"></a>

To view the KMS keys in an external key store, use the AWS KMS console or the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. You can use the same techniques that you would use to view any AWS KMS [customer managed keys](concepts.md#kms_keys)\. To learn the basics, see [Viewing keys](viewing-keys.md)\.

In the AWS KMS console, the KMS keys in your external key store are displayed on the **Customer managed keys** page, along with all other customer managed keys in your AWS account and Region\. To identify KMS keys in an external key store, filter by the distinctive origin value, **External key store**, and the custom key store ID\.

For more information, see [Viewing an external key store](view-xks-keystore.md), [Monitoring an external key store](xks-monitoring.md), and [Logging AWS KMS API calls with AWS CloudTrail](logging-using-cloudtrail.md)\.

**Topics**
+ [Properties of KMS keys in an external key store](#view-xks-key-properties)
+ [Viewing KMS keys in an external key store \(console\)](#view-xks-keys-console)
+ [Viewing KMS keys in an external key store \(AWS KMS API\)](#view-xks-keys-api)

## Properties of KMS keys in an external key store<a name="view-xks-key-properties"></a>

Like all KMS keys, the KMS keys in an external key store, have a [key ARN](concepts.md#key-id), [key spec](concepts.md#key-spec), and [key usage](concepts.md#key-usage) values, but they also have properties and property values specific to KMS keys in an external key store\. For example, the **Origin** value for all KMS keys in external key stores is **External key store**\.

For a KMS key in an external key store, the **Cryptographic configuration** tab in the AWS KMS console include two additional sections, **Custom key store** and **External key**\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/xks-view-key-80.png)

### Custom key store properties<a name="view-xks-custom-keystores"></a>

The following values appear in the **Custom key store** section of the **Cryptographic configuration** tab and in the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) response\. These properties apply to all custom key stores, including AWS CloudHSM key stores and external key stores\.

**Custom key store ID**  
A unique ID that AWS KMS assigns to the custom key store\.

**Custom key store name**  
A friendly name that you assign to the custom key store when you create it\. You can change this value at any time\.

**Custom key store type**  
The type of custom key store\. Valid values are AWS CloudHSM \(`AWS_CLOUDHSM`\) or External key store \(`EXTERNAL_KEY_STORE`\)\. You cannot change the type after you create the custom key store\.

**Creation date**  
The date that the custom key store was created\. This date is displayed in local time for the AWS Region\. 

**Connection state**  
Indicates whether the custom key store is connected to its backing key store\. The connection state is `DISCONNECTED` only if the custom key store has never been connected to its backing key store, or it has been intentionally disconnected\. For details, see [Connection state](xks-connect-disconnect.md#xks-connection-state)\.

### External key properties<a name="view-xks-external-key"></a>

External key properties appear in the **External key** section of the **Cryptographic configuration** tab and in the `XksKeyConfiguration` element of the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) response\. 

The **External key** section appears in the AWS KMS console only for KMS keys in external key stores\. It provides information about the external key associated with the KMS key\. The [*external key*](keystore-external.md#concept-external-key) is a cryptographic key outside of AWS that serves as the key material for the KMS key in the external key store\. When you encrypt or decrypt with the KMS key, the operation is performed by your [external key manager](keystore-external.md#concept-ekm) using the specified external key\.

The following values appear in the **External key** section\.

**External key ID**  
The identifier for the external key in its external key manager\. This is the value that the external key store proxy uses to identify the external key\. You specify the ID of the external key when you create the KMS key and you cannot change it\. If the external key ID value that you used to create the KMS key changes or becomes invalid, you must [schedule the KMS key for deletion](delete-xks-key.md) and [create a new KMS key](create-xks-keys.md) with the correct external key ID value\.

## Viewing KMS keys in an external key store \(console\)<a name="view-xks-keys-console"></a>

**To view the KMS keys in an external key store \(Console\)**

1. Open the AWS KMS console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. To identify the KMS keys in your external key store, add the **Origin** and **Custom key store ID** fields to your key table\. KMS keys in any external key store have an **Origin** value of **External key store**\.

   In the upper\-right corner, choose the gear icon, choose **Origin** and **Custom key store ID**, then choose **Confirm**\.

1. Choose the alias or key ID of a KMS key in an external key store\. 

1. To view the properties specific to KMS keys in an external key store, choose the **Cryptographic configuration** tab\. Special values for KMS keys in an external key store appear in the **Custom key store** and **External key** sections\. 

## Viewing KMS keys in an external key store \(AWS KMS API\)<a name="view-xks-keys-api"></a>

**To view the KMS keys in an external key store \(API\)**

You use the same AWS KMS API operations to view the KMS keys in an external key store that you would use for any KMS key, including [ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html), [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html), and [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html)\. For example, the following `describe-key` operation in the AWS CLI shows the special fields for a KMS key in an external key store\. Before running a command like this one, replace the example KMS key ID with a valid value\.

```
$ aws kms describe-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
{
  "KeyMetadata": {
    "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
    "AWSAccountId": "111122223333",
    "CreationDate": "2022-12-02T07:48:55-07:00",
    "CustomerMasterKeySpec": "SYMMETRIC_DEFAULT",
    "CustomKeyStoreId": "cks-1234567890abcdef0",
    "Description": "",
    "Enabled": true,
    "EncryptionAlgorithms": [
      "SYMMETRIC_DEFAULT"
    ],
    "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
    "KeyManager": "CUSTOMER",
    "KeySpec": "SYMMETRIC_DEFAULT",
    "KeyState": "Enabled",
    "KeyUsage": "ENCRYPT_DECRYPT",
    "MultiRegion": false,
    "Origin": "EXTERNAL_KEY_STORE",
    "XksKeyConfiguration": {      
      "Id": "bb8562717f809024"           
    }
  }
}
```