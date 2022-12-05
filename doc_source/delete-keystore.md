# Deleting an AWS CloudHSM key store<a name="delete-keystore"></a>

When you delete an AWS CloudHSM key store, AWS KMS deletes all metadata about the AWS CloudHSM key store from KMS, including information about its association with an AWS CloudHSM cluster\. This operation does not affect the AWS CloudHSM cluster, its HSMs, or its users\. You can create a new AWS CloudHSM key store that is associated with the same AWS CloudHSM cluster, but you cannot undo the delete operation\.

You can only delete an AWS CloudHSM key store that is disconnected from its AWS CloudHSM cluster and does not contain any AWS KMS keys\. Before you delete a custom key store, do the following\.
+ Verify that you will never need to use any of the KMS keys in the key store for any [cryptographic operations](use-cmk-keystore.md)\. Then [schedule deletion](delete-cmk-keystore.md) of all of the KMS keys from the key store\. For help finding the KMS keys in an AWS CloudHSM key store, see [Find the KMS keys in an AWS CloudHSM key store](find-key-material.md#find-cmk-in-keystore)\.
+ Confirm that all KMS keys have been deleted\. To view the KMS keys in an AWS CloudHSM key store, see [Viewing KMS keys in an AWS CloudHSM key store](view-cmk-keystore.md)\.
+ [Disconnect the AWS CloudHSM key store](disconnect-keystore.md) from its AWS CloudHSM cluster\.

Instead of deleting the AWS CloudHSM key store, consider [disconnecting it](disconnect-keystore.md) from its associated AWS CloudHSM cluster\. While an AWS CloudHSM key store is disconnected, you can manage the AWS CloudHSM key store and its AWS KMS keys\. But you cannot create or use KMS keys in the AWS CloudHSM key store\. You can reconnect the AWS CloudHSM key store at any time\.

**Topics**
+ [Delete an AWS CloudHSM key store \(console\)](#delete-keystore-console)
+ [Delete an AWS CloudHSM key store \(API\)](#delete-keystore-api)

## Delete an AWS CloudHSM key store \(console\)<a name="delete-keystore-console"></a>

To delete an AWS CloudHSM key store in the AWS Management Console, begin by selecting the AWS CloudHSM key store from the **Custom key stores** page\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Custom key stores**, **AWS CloudHSM key stores**\.

1. Find the row that represents the AWS CloudHSM key store that you want to delete\. If the **Connection state** of the AWS CloudHSM key store is not **DISCONNECTED**, you must [disconnect the AWS CloudHSM key store](disconnect-keystore.md) before you delete it\.

1. From the **Key store actions** menu, choose **Delete**\.

When the operation completes, a success message appears and the AWS CloudHSM key store no longer appears in the key stores list\. If the operation is unsuccessful, an error message appears that describes the problem and provides help on how to fix it\. If you need more help, see [Troubleshooting a custom key store](fix-keystore.md)\.

## Delete an AWS CloudHSM key store \(API\)<a name="delete-keystore-api"></a>

To delete an AWS CloudHSM key store, use the [DeleteCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteCustomKeyStore.html) operation\. If the operation is successful, AWS KMS returns an HTTP 200 response and a JSON object with no properties\.

To begin, verify that the AWS CloudHSM key store does not contain any AWS KMS keys\. You cannot delete a custom key store that contains KMS keys\. The first example command uses [ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html) and [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) to search for AWS KMS keys in the AWS CloudHSM key store with the example *cks\-1234567890abcdef0* custom key store ID\. In this case, the command does not return any KMS keys\. If it does, use the [ScheduleKeyDeletion](https://docs.aws.amazon.com/kms/latest/APIReference/API_ScheduleKeyDeletion.html) operation to schedule deletion of each of the KMS keys\.

------
#### [ Bash ]

```
for key in $(aws kms list-keys --query 'Keys[*].KeyId' --output text) ; 
do aws kms describe-key --key-id $key | 
grep '"CustomKeyStoreId": "cks-1234567890abcdef0"' --context 100; done
```

------
#### [ PowerShell ]

```
PS C:\> Get-KMSKeyList | Get-KMSKey | where CustomKeyStoreId -eq 'cks-1234567890abcdef0'
```

------

Next, disconnect the AWS CloudHSM key store\. This example command uses the [DisconnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisconnectCustomKeyStore.html) operation to disconnect an AWS CloudHSM key store from its AWS CloudHSM cluster\. Before running this command, replace the example custom key store ID with a valid one\.

------
#### [ Bash ]

```
$ aws kms disconnect-custom-key-store --custom-key-store-id cks-1234567890abcdef0
```

------
#### [ PowerShell ]

```
PS C:\> Disconnect-KMSCustomKeyStore -CustomKeyStoreId cks-1234567890abcdef0
```

------

After the custom key store is disconnected, you can use the [DeleteCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteCustomKeyStore.html) operation to delete it\. 

------
#### [ Bash ]

```
$ aws kms delete-custom-key-store --custom-key-store-id cks-1234567890abcdef0
```

------
#### [ PowerShell ]

```
PS C:\> Remove-KMSCustomKeyStore -CustomKeyStoreId cks-1234567890abcdef0
```

------