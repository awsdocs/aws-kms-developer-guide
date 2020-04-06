# Editing custom key store settings<a name="update-keystore"></a>

You can change the settings of an existing [custom key store](key-store-concepts.md#concept-custom-key-store)\. The custom key store must be disconnected from its AWS CloudHSM cluster\.

To edit custom key store settings:

1. [Disconnect the custom key store](disconnect-keystore.md) from its AWS CloudHSM cluster\. While the custom key store is disconnected, you cannot create [customer master keys](concepts.md#master_keys) \(CMKs\) in the custom key store and you cannot use the CMKs it contains for [cryptographic operations](use-cmk-keystore.md)\. 

1. Edit one or more of the custom key store settings\.

1. [Reconnect the custom key store](disconnect-keystore.md) to its AWS CloudHSM cluster\.

You can edit the following settings in a custom key store:

**The friendly name of the custom key store\.**  
Enter a new friendly name\. The new name must be unique in your AWS account\.

**The cluster ID of the associated AWS CloudHSM cluster\.**  
Edit this value to substitute a related AWS CloudHSM cluster for the original one\. You can use this feature to repair a custom key store if its AWS CloudHSM cluster becomes corrupted or is deleted\.   
Specify an AWS CloudHSM cluster that shares a backup history with the original cluster and [fulfills the requirements](create-keystore.md#before-keystore) for association with a custom key store, including two active HSMs in different Availability Zones\. Clusters that share a backup history have the same cluster certificate\. To view the cluster certificate of a cluster, use the [DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html) operation\. You cannot use the edit feature to associate the custom key store with an unrelated AWS CloudHSM cluster\. 

**The current password of the [`kmsuser` crypto user](key-store-concepts.md#concept-kmsuser) \(CU\)\.**  
Tells AWS KMS the current password of the `kmsuser` CU in the AWS CloudHSM cluster\. This action does not change the password of the `kmsuser` CU in the AWS CloudHSM cluster\.  
If you change the password of the `kmsuser` CU in the AWS CloudHSM cluster, use this feature to tell AWS KMS the new `kmsuser` password\. Otherwise, AWS KMS cannot log into the cluster and all attempts to connect the custom key store to the cluster fail\. 

**Topics**
+ [Edit a custom key store \(console\)](#update-keystore-console)
+ [Edit a custom key store \(API\)](#update-keystore-api)

## Edit a custom key store \(console\)<a name="update-keystore-console"></a>

When you edit the custom key store, you can change any or of the configurable values\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Custom key stores**\.

1. Choose the custom key store you want to edit\. 

1. If the value in the **Status** column is not **DISCONNECTED**, you must disconnect the custom key store before you can edit it\. From the **Key store actions** menu, select **Disconnect custom key store**\.

1. From the **Key store actions** menu, select **Edit custom key store settings**\.

1. Do one or more of the following actions\.
   + Type a new friendly name for the custom key store\.
   + Type the cluster ID of a related AWS CloudHSM cluster\.
   + Type the current password of the `kmsuser` crypto user in the associated AWS CloudHSM cluster\.

1. Choose **Save**\.

   When the procedure is successful, a message describes the settings that you edited\. When it is unsuccessful, an error message appears that describes the problem and provides help on how to fix it\. If you need more help, see [Troubleshooting a custom key store](fix-keystore.md)\.

1. [Reconnect the custom key store\.](disconnect-keystore.md)

   To use the custom key store, you must reconnect it after editing\. You can leave the custom key store disconnected\. But while it is disconnected, you cannot create CMKs in the custom key store or use the CMKs in the custom key store in [cryptographic operations](use-cmk-keystore.md)\.

## Edit a custom key store \(API\)<a name="update-keystore-api"></a>

To change the properties of a custom key store, use the [UpdateCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateCustomKeyStore.html) operation\. You can change multiple properties of a custom key store in the same command\. If the operation is successful, AWS KMS returns an HTTP 200 response and a JSON object with no properties\.

The examples in this section use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

Begin by using [DisconnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisconnectCustomKeyStore.html) to [disconnect the custom key store](disconnect-keystore.md) from AWS KMS\. Replace the example custom key store ID, cks\-1234567890abcdef0, with an actual ID\.

```
$ aws kms disconnect-custom-key-store --custom-key-store-id cks-1234567890abcdef0
```

The first example uses [UpdateCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateCustomKeyStore.html) to change the friendly name of the custom key store to `DevelopmentKeys`\. The command uses the `CustomKeyStoreId` parameter to identify the custom key store and the `CustomKeyStoreName` to specify the new name for the custom key store\.

```
$ aws kms update-custom-key-store --custom-key-store-id cks-1234567890abcdef0 --new-custom-key-store-name DevelopmentKeys
```

The following example changes the cluster that is associated with a custom key store to another backup of the same cluster\. The command uses the `CustomKeyStoreId` parameter to identify the custom key store and the `CloudHsmClusterId` parameter to specify the new cluster ID\. 

```
$ aws kms update-custom-key-store --custom-key-store-id cks-1234567890abcdef0 --cloud-hsm-cluster-id cluster-1a23b4cdefg
```

The following example tells AWS KMS that the current `kmsuser` password is `ExamplePassword`\. The command uses the `CustomKeyStoreId` parameter to identify the custom key store and the `KeyStorePassword` parameter to specify the current password\.

```
$ aws kms update-custom-key-store --custom-key-store-id cks-1234567890abcdef0 --key-store-password ExamplePassword
```

The final command reconnects the custom key store to AWS KMS\. You can leave the custom key store in the disconnected state, but you must connect it before you can create new CMKs or use existing CMKs for [cryptographic operations](use-cmk-keystore.md)\. Replace the example custom key store ID with an actual ID\.

```
$ aws kms connect-custom-key-store --custom-key-store-id cks-1234567890abcdef0
```