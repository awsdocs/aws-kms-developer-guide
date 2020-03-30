# Viewing a custom key store<a name="view-keystore"></a>

You can view the custom key stores in each account and region by using the AWS Management Console or the AWS KMS API\. 

For help with viewing the CMKs in your custom key store, see [Viewing CMKs in a custom key store](view-cmk-keystore.md)\.

**Topics**
+ [View a custom key store \(console\)](#view-keystore-console)
+ [View a custom key store \(API\)](#view-keystore-api)

## View a custom key store \(console\)<a name="view-keystore-console"></a>

When you view the custom key stores in the AWS Management Console, you can see the following:
+ The custom key store name
+ The ID of associated AWS CloudHSM cluster
+ The number of HSMs in the cluster
+ The current connection status

A connection status of **Disconnected** indicates that the custom key store is new and has never been connected, or it was intentionally [disconnected from its AWS CloudHSM cluster](disconnect-keystore.md)\. However, if your attempts to use a CMK in a connected custom key store fail, that might indicate a problem with the custom key store or its AWS CloudHSM cluster\. For help, see [How to fix a failing CMK](fix-keystore.md#fix-cmk-failed)\.

To view the custom key stores in a given account and Region, use the following procedure\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Custom key stores**\.

To customize the display, click the gear icon that appears below the **Create key store** button\.

## View a custom key store \(API\)<a name="view-keystore-api"></a>

To view your custom key stores, use the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation\. By default, this operation returns all custom key stores in the account and Region\. But you can use either the `CustomKeyStoreId` or `CustomKeyStoreName` parameter \(but not both\) to limit the output to a particular custom key store\. The output consists of the custom key store ID and name, the ID of the associated AWS CloudHSM cluster, and the connection state\. If the connection state indicates an error, the output also includes an error code that describes the reason for the error\.

The examples in this section use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

For example, the following command returns all custom key stores in the account and Region\. You can use the `Limit` and `Marker` parameters to page through the custom key stores in the output\.

```
$ aws kms describe-custom-key-stores
```

The following example command uses the `CustomKeyStoreName` parameter to get only the custom key store with the `ExampleKeyStore` friendly name\. You can use either the `CustomKeyStoreName` or `CustomKeyStoreId` parameter \(but not both\) in each command\.

The following example output represents a custom key store that is connected to its AWS CloudHSM cluster\. The `ConnectionState` element corresponds to the `Status` field in the console\.

```
$ aws kms describe-custom-key-stores --custom-key-store-name ExampleKeyStore
{
   "CustomKeyStores": [ 
      { 
         "CloudHsmClusterId": "cluster-1a23b4cdefg",
         "ConnectionState": "CONNECTED",
         "CreationDate": "1.499288695918E9",
         "CustomKeyStoreId": "cks-1234567890abcdef0",
         "CustomKeyStoreName": "ExampleKeyStore",
         "TrustAnchorCertificate": "<certificate appears here>"
      }
   ]
}
```

A `ConnectionState` of `Disconnected` indicates that a custom key store has never been connected or it was intentionally [disconnected from its AWS CloudHSM cluster](disconnect-keystore.md)\. However, if attempts to use a CMK in a connected custom key store fail, that might indicate a problem with the custom key store or its AWS CloudHSM cluster\. For help, see [How to fix a failing CMK](fix-keystore.md#fix-cmk-failed)\. 

If the `ConnectionState` of the custom key store is `FAILED`, the `DescribeCustomKeyStores` response includes a `ConnectionErrorCode` element that explains the reason for the error\.

For example, in the following output, the `INVALID_CREDENTIALS` value indicates that the custom key store connection failed because the [`kmsuser` password is invalid](fix-keystore.md#fix-keystore-password)\. For help with this and other connection error failures, see [Troubleshooting a custom key store](fix-keystore.md)\.

```
$ aws kms describe-custom-key-stores --custom-key-store-id cks-1234567890abcdef0
{
   "CustomKeyStores": [ 
      { 
         "CloudHsmClusterId": "cluster-1a23b4cdefg",
         "ConnectionErrorCode": "INVALID_CREDENTIALS"
         "ConnectionState": "FAILED",
         "CustomKeyStoreId": "cks-1234567890abcdef0",
         "CustomKeyStoreName": "ExampleKeyStore",
         "CreationDate": "1.499288695918E9",
         "TrustAnchorCertificate": "<certificate appears here>"
      }
   ]
}
```