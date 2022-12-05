# Connecting and disconnecting an AWS CloudHSM key store<a name="disconnect-keystore"></a>

New AWS CloudHSM key stores are not connected\. Before you can create and use AWS KMS keys in your AWS CloudHSM key store, you need to connect it to its associated AWS CloudHSM cluster\. You can connect and disconnect your AWS CloudHSM key store at any time, and [view its connection state](view-keystore.md#view-keystore-console)\. 

You are not required to connect your AWS CloudHSM key store\. You can leave an AWS CloudHSM key store in a disconnected state indefinitely and connect it only when you need to use it\. However, you might want to test the connection periodically to verify that the settings are correct and it can be connected\.

**Note**  
AWS CloudHSM key stores have a `DISCONNECTED` connection state only when the key store has never been connected or you explicitly disconnect it\. If your AWS CloudHSM key store connection state is `CONNECTED` but you are having trouble using it, make sure that its associated AWS CloudHSM cluster is active and contains at least one active HSMs\. For help with connection failures, see [Troubleshooting a custom key store](fix-keystore.md)\.

**Topics**
+ [Connecting an AWS CloudHSM key store](#about-hsm-connect)
+ [Disconnecting an AWS CloudHSM key store](#about-hsm-disconnect)
+ [Connect an AWS CloudHSM key store \(console\)](#connect-keystore-console)
+ [Connect a custom key store \(API\)](#connect-keystore-api)
+ [Disconnect an AWS CloudHSM key store \(console\)](#disconnect-keystore-console)
+ [Disconnect an AWS CloudHSM key store \(API\)](#disconnect-keystore-api)

## Connecting an AWS CloudHSM key store<a name="about-hsm-connect"></a>

When you connect an AWS CloudHSM key store, AWS KMS finds the associated AWS CloudHSM cluster, connects to it, logs into the AWS CloudHSM client as the [`kmsuser` crypto user](hsm-key-store-concepts.md#concept-kmsuser) \(CU\), and then rotates the `kmsuser` password\. AWS KMS remains logged into the AWS CloudHSM client as long as the AWS CloudHSM key store is connected\.

To establish the connection, AWS KMS creates a [security group](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) named `kms-<custom key store ID>` in the virtual private cloud \(VPC\) of the cluster\. The security group has a single rule that allows inbound traffic from the cluster security group\. AWS KMS also creates an [elastic network interface](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_ElasticNetworkInterfaces.html) \(ENI\) in each Availability Zone of the private subnet for the cluster\. AWS KMS adds the ENIs to the `kms-<cluster ID>` security group and the security group for the cluster\. The description of each ENI is `KMS managed ENI for cluster <cluster-ID>`\.

The connection process can take an extended amount of time to complete; up to 20 minutes\. 

Before you connect the AWS CloudHSM key store, verify that it meets the requirements\.
+ Its associated AWS CloudHSM cluster must contain at least one active HSM\. To find the number of HSMs in the cluster, view the cluster in the AWS CloudHSM console or use the [DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html) operation\. If necessary, you can [add an HSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/add-remove-hsm.html)\.
+ The cluster must have a [`kmsuser` crypto user](create-keystore.md#kmsuser-concept) \(CU\) account, but that CU cannot be logged into the cluster when you connect the AWS CloudHSM key store\. For help with logging out, see [How to log out and reconnect](fix-keystore.md#login-kmsuser-2)\.
+ The connection state of the AWS CloudHSM key store cannot be `DISCONNECTING` or `FAILED`\. To view the connection state, use the AWS KMS console or the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) response\. If the connection state is `FAILED`, disconnect the custom key store, fix the problem, and then connect it\.

For help with connection failures, see [How to fix a connection failure](fix-keystore.md#fix-keystore-failed)\.

When your AWS CloudHSM key store is connected, you can [create KMS keys in it](create-cmk-keystore.md) and use existing KMS keys in [cryptographic operations](use-cmk-keystore.md)\.

## Disconnecting an AWS CloudHSM key store<a name="about-hsm-disconnect"></a>

When you disconnect an AWS CloudHSM key store, AWS KMS logs out of the AWS CloudHSM client, disconnects from the associated AWS CloudHSM cluster, and removes the network infrastructure that it created to support the connection\.

While an AWS CloudHSM key store is disconnected, you can manage the AWS CloudHSM key store and its KMS keys, but you cannot create or use KMS keys in the AWS CloudHSM key store\. The connection state of the key store is `DISCONNECTED` and the [key state](key-state.md) of KMS keys in the custom key store is `Unavailable`, unless they are `PendingDeletion`\.  You can reconnect the AWS CloudHSM key store at any time\.

When you disconnect a custom key store, the KMS keys in the key store become unusable right away \(subject to eventual consistency\)\. However, resources encrypted with [data keys](concepts.md#data-keys) protected by the KMS key are not affected until the KMS key is used again, such as to decrypt the data key\. This issue affects AWS services, many of which use data keys to protect your resources\. For details, see [How unusable KMS keys affect data keys](concepts.md#unusable-kms-keys)\.

**Note**  
While a custom key store is disconnected, all attempts to create KMS keys in the custom key store or to use existing KMS keys in cryptographic operations will fail\. This action can prevent users from storing and accessing sensitive data\.

To better estimate the effect of disconnecting your custom key store, [identify the KMS keys](find-key-material.md#find-cmk-in-keystore) in the custom key store and [determine their past use](deleting-keys-determining-usage.md)\.

You might disconnect an AWS CloudHSM key store for reasons such as the following:
+ **To rotate of the `kmsuser` password\.** AWS KMS changes the `kmsuser` password each time that it connects to the AWS CloudHSM cluster\. To force a password rotation, just disconnect and reconnect\.
+ **To audit the key material** for the KMS keys in the AWS CloudHSM cluster\. When you disconnect the custom key store, AWS KMS logs out of the [`kmsuser` crypto user](hsm-key-store-concepts.md#concept-kmsuser) account in the AWS CloudHSM client\. This allows you to log into the cluster as the `kmsuser` CU and audit and manage the key material for the KMS key\.
+ **To immediately disable all KMS keys** in the AWS CloudHSM key store\. You can [disable and re\-enable KMS keys](enabling-keys.md) in an AWS CloudHSM key store by using the AWS Management Console or the [DisableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) operation\. These operations complete quickly, but they act on one KMS key at a time\. Disconnecting the AWS CloudHSM key store immediately changes the key state of all KMS keys in the AWS CloudHSM key store to `Unavailable`, which prevents them from being used in any cryptographic operation\.
+ **To repair a failed connection attempt**\. If an attempt to connect an AWS CloudHSM key store fails \(the connection state of the custom key store is `FAILED`\), you must disconnect the AWS CloudHSM key store before you try to connect it again\.

## Connect an AWS CloudHSM key store \(console\)<a name="connect-keystore-console"></a>

To connect an AWS CloudHSM key store in the AWS Management Console, begin by selecting the AWS CloudHSM key store from the **Custom key stores** page\. The connection process can take up to 20 minutes to complete\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Custom key stores**, **AWS CloudHSM key stores**\.

1. Choose the row of the AWS CloudHSM key store you want to connect\. 

   If the connection state of the AWS CloudHSM key store is **FAILED**, you must [disconnect the custom key store](#disconnect-keystore-console) before you connect it\.

1. From the **Key store actions** menu, choose **Connect**\.

AWS KMS begins the process of connecting your custom key store\. It finds the associated AWS CloudHSM cluster, builds the required network infrastructure, connects to it, logs into the AWS CloudHSM cluster as the `kmsuser` CU, and rotates the `kmsuser` password\. When the operation completes, the connection state changes to **CONNECTED**\. 

If the operation fails, an error message appears that describes the reason for the failure\. Before you try to connect again, [view the connection state](view-keystore.md) of your AWS CloudHSM key store\. If it is **FAILED**, you must [disconnect the custom key store](#disconnect-keystore-console) before you connect it again\. If you need help, see [Troubleshooting a custom key store](fix-keystore.md)\.

**Next:** [Creating KMS keys in an AWS CloudHSM key store](create-cmk-keystore.md)\.

## Connect a custom key store \(API\)<a name="connect-keystore-api"></a>

To connect a disconnected AWS CloudHSM key store, use the [ConnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_ConnectCustomKeyStore.html) operation\. The associated AWS CloudHSM cluster must contain at least one active HSM and the connection state cannot be `FAILED`\.

The connection process takes an extended amount of time to complete; up to 20 minutes\. Unless it fails quickly, the operation returns an HTTP 200 response and a JSON object with no properties\. However, this initial response does not indicate that the connection was successful\. To determine the connection state of the custom key store, see the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) response\.

The examples in this section use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

To identify the AWS CloudHSM key store, use its custom key store ID\. You can find the ID on the **Custom key stores** page in the console or by using the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation with no parameters\. Before running this example, replace the example ID with a valid one\.

```
$ aws kms connect-custom-key-store --custom-key-store-id cks-1234567890abcdef0
```

To verify that the AWS CloudHSM key store is connected, use the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation\. By default, this operation returns all custom keys stores in your account and Region\. But you can use either the `CustomKeyStoreId` or `CustomKeyStoreName` parameter \(but not both\) to limit the response to particular custom key stores\. The `ConnectionState` value of `CONNECTED` indicates that the custom key store is connected to its AWS CloudHSM cluster\.

**Note**  
The `CustomKeyStoreType` field was added to the `DescribeCustomKeyStores` response to distinguish AWS CloudHSM key stores from external key stores\.

```
$ aws kms describe-custom-key-stores --custom-key-store-id cks-1234567890abcdef0
{
   "CustomKeyStores": [
      "CustomKeyStoreId": "cks-1234567890abcdef0",
      "CustomKeyStoreName": "ExampleCloudHSMKeyStore",
      "CloudHsmClusterId": "cluster-1a23b4cdefg",
      "CustomKeyStoreType": "AWS_CLOUDHSM",
      "TrustAnchorCertificate": "<certificate string appears here>",
      "CreationDate": "1.499288695918E9",
      "ConnectionState": "CONNECTED"
   ],
}
```

If the `ConnectionState` value is failed, the `ConnectionErrorCode` element indicates the reason for the failure\. In this case, AWS KMS could not find an AWS CloudHSM cluster in your account with the cluster ID `cluster-1a23b4cdefg`\. If you deleted the cluster, you can [restore it from a backup](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-cluster-from-backup.html) of the original cluster and then [edit the cluster ID](update-keystore.md) for the custom key store\. For help responding to a connection error code, see [How to fix a connection failure](fix-keystore.md#fix-keystore-failed)\.

```
$ aws kms describe-custom-key-stores --custom-key-store-id cks-1234567890abcdef0
{
   "CustomKeyStores": [
      "CustomKeyStoreId": "cks-1234567890abcdef0",
      "CustomKeyStoreName": "ExampleKeyStore",
      "CloudHsmClusterId": "cluster-1a23b4cdefg",
      "CustomKeyStoreType": "AWS_CLOUDHSM",
      "TrustAnchorCertificate": "<certificate string appears here>",
      "CreationDate": "1.499288695918E9",
      "ConnectionState": "FAILED"
      "ConnectionErrorCode": "CLUSTER_NOT_FOUND"
   ],
}
```

**Next:** [Creating KMS keys in an AWS CloudHSM key store](create-cmk-keystore.md)\.

## Disconnect an AWS CloudHSM key store \(console\)<a name="disconnect-keystore-console"></a>

To disconnect a connected AWS CloudHSM key store in the AWS Management Console, begin by choosing the AWS CloudHSM key store from the **Custom Key Stores** page\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Custom key stores**, **AWS CloudHSM key stores**\.

1. Choose the row of the external key store you want to disconnect\. 

1. From the **Key store actions** menu, choose **Disconnect**\.

When the operation completes, the connection state changes from **DISCONNECTING** to **DISCONNECTED**\. If the operation fails, an error message appears that describes the problem and provides help on how to fix it\. If you need more help, see [Troubleshooting a custom key store](fix-keystore.md)\.

## Disconnect an AWS CloudHSM key store \(API\)<a name="disconnect-keystore-api"></a>

To disconnect a connected AWS CloudHSM key store, use the [DisconnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisconnectCustomKeyStore.html) operation\. If the operation is successful, AWS KMS returns an HTTP 200 response and a JSON object with no properties\.

The examples in this section use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

This example disconnects an AWS CloudHSM key store\. Before running this example, replace the example ID with a valid one\.

```
$ aws kms disconnect-custom-key-store --custom-key-store-id cks-1234567890abcdef0
```

To verify that the AWS CloudHSM key store is disconnected, use the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation\. By default, this operation returns all custom keys stores in your account and Region\. But you can use either the `CustomKeyStoreId` and `CustomKeyStoreName` parameter \(but not both\) to limit the response to particular custom key stores\. The `ConnectionState` value of `DISCONNECTED` indicates that this example AWS CloudHSM key store is not connected to its AWS CloudHSM cluster\.

```
$ aws kms describe-custom-key-stores --custom-key-store-id cks-1234567890abcdef0
{
   "CustomKeyStores": [
      "CloudHsmClusterId": "cluster-1a23b4cdefg",
      "ConnectionState": "DISCONNECTED",
      "CreationDate": "1.499288695918E9",
      "CustomKeyStoreId": "cks-1234567890abcdef0",
      "CustomKeyStoreName": "ExampleKeyStore",
      "CustomKeyStoreType": "AWS_CLOUDHSM",
      "TrustAnchorCertificate": "<certificate string appears here>"
   ],
}
```