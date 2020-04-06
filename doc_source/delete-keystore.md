# Deleting a custom key store<a name="delete-keystore"></a>

When you delete a custom key store, AWS KMS deletes all metadata about the custom key store from KMS, including information about its association with an AWS CloudHSM cluster\. This operation does not affect the AWS CloudHSM cluster, its HSMs, or its users\. You can create a new custom key store that is associated with the specified cluster, but you cannot undo the delete operation\.

You can only delete a custom key store that is disconnected from AWS KMS and does not contain any customer master keys \(CMKs\)\. Before you delete a custom key store, do the following\.
+ Verify that you will never need to use any of the CMKs in the key store for any [cryptographic operations](use-cmk-keystore.md)\. Then [schedule deletion](delete-cmk-keystore.md) of all of the CMKs from the key store\. For help finding the CMKs in a custom key store, see [Find the CMKs in a custom key store](find-key-material.md#find-cmk-in-keystore)\.
+ Confirm that all CMKs have been deleted\. To view the CMKs in a custom key store, see [Viewing CMKs in a custom key store](view-cmk-keystore.md)\.
+ [Disconnect the custom key store](disconnect-keystore.md) from AWS KMS\.

Instead of deleting the custom key store, consider [disconnecting it](disconnect-keystore.md) from its associated AWS CloudHSM cluster\. While a custom key store is disconnected, you can manage the custom key store and its customer master keys \(CMKs\)\. But you cannot create or use CMKs in the custom key store\. You can reconnect the custom key store at any time\.

If you have deleted all custom key stores from all Regions of your AWS account and you do not plan to create any more, you should [delete the service\-linked role](authorize-key-store.md#authorize-kms) that AWS KMS uses for custom key stores\.

**Topics**
+ [Delete a custom key store \(console\)](#delete-keystore-console)
+ [Delete a custom key store \(API\)](#delete-keystore-api)

## Delete a custom key store \(console\)<a name="delete-keystore-console"></a>

To delete a custom key store in the AWS Management Console, begin by selecting the custom key store from the **Custom key stores** page\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Custom key stores**\.

1. Find the row that represents the custom key store that you want to remove\. If the status of the custom key store is not **DISCONNECTED**, you must [disconnect the custom key store](disconnect-keystore.md) before you delete the custom key store\.

1. From the **Key store actions** menu, select **Delete custom key store**\.

When the operation completes, a success message appears and the custom key store no longer appears in the custom key store list\. If the operation is unsuccessful, an error message appears that describes the problem and provides help on how to fix it\. If you need more help, see [Troubleshooting a custom key store](fix-keystore.md)\.

## Delete a custom key store \(API\)<a name="delete-keystore-api"></a>

To delete a custom key store, use the [DeleteCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteCustomKeyStore.html) operation\. If the operation is successful, AWS KMS returns an HTTP 200 response and a JSON object with no properties\.

To begin, verify that the custom key store does not contain any AWS KMS customer master keys \(CMKs\)\. You cannot delete a custom key store that contains CMKs\. The first example command uses [ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html) and [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) to search for AWS KMS customer master keys in the custom key store with the *cks\-1234567890abcdef0* fictitious key store ID\. In this case, the command does not return any CMKs\. If it does, use the [ScheduleKeyDeletion](https://docs.aws.amazon.com/kms/latest/APIReference/API_ScheduleKeyDeletion.html) operation to schedule deletion of each of the CMKs\.

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
PS C:\> (Get-KMSKeyList).KeyArn | foreach {Get-KMSKey -KeyId $_} | where CustomKeyStoreId -eq 'cks-1234567890abcdef0'
```

------

Next, disconnect the custom key store\. This example command uses the [DisconnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisconnectCustomKeyStore.html) operation to disconnect the custom key store from its AWS CloudHSM cluster\. Before running this command, replace the example custom key store ID with a valid one\.

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