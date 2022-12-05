# Deleting an external key store<a name="delete-xks"></a>

When you delete an external key store, AWS KMS deletes all metadata about the external key store from AWS KMS, including information about its external key store proxy\. This operation does not affect the [external key store proxy](keystore-external.md#concept-xks-proxy), [external key manager](keystore-external.md#concept-ekm), [external keys](keystore-external.md#concept-external-key), or any AWS resources that you created to support the external key store, such as an Amazon VPC or a VPC endpoint service\.

Before you delete an external key store, you must [delete all of the KMS keys](delete-xks-key.md) from the key store and [disconnect the key store](xks-connect-disconnect.md) from its external key store proxy\. Otherwise, attempts to delete the key store fail\.

Deleting an external key store is irreversible, but you can create a new external key store and associate it with the same external key store proxy and external key manager\. However, you cannot recreate the symmetric encryption KMS keys in the external key store, even you have access to the same external key material\. AWS KMS includes metadata in the symmetric ciphertext unique to each KMS key\. This security feature ensures that only the KMS key that encrypted the data can decrypt it\. 

Instead of deleting the external key store, consider disconnecting it\. While an external key store is disconnected, you can manage the external key store and its AWS KMS keys but you cannot create or use KMS keys in the external key store\. You can reconnect the external key store at any time and resume using its KMS keys to encrypt and decrypt data\. There is no cost for a disconnected external key store proxy or its unavailable KMS keys\.

**Topics**
+ [Delete an external key store \(console\)](#delete-xks-console)
+ [Delete an external key store \(API\)](#delete-xks-api)

## Delete an external key store \(console\)<a name="delete-xks-console"></a>

You can use the AWS KMS console to delete an external key store\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Custom key stores**, **External key stores**\.

1. Find the row that represents the external key store that you want to delete\. If the **Connection state** of the external key store is not **DISCONNECTED**, you must [disconnect the external key store](xks-connect-disconnect.md#disconnect-xks-console) before you delete it\.

1. From the **Key store actions** menu, choose **Delete**\.

When the operation completes, a success message appears and the external key store no longer appears in the key store list\. If the operation is unsuccessful, an error message appears that describes the problem and provides help on how to fix it\. If you need more help, see [Troubleshooting external key stores](xks-troubleshooting.md)\.

## Delete an external key store \(API\)<a name="delete-xks-api"></a>

To delete an external key store, use the [DeleteCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteCustomKeyStore.html) operation\. If the operation is successful, AWS KMS returns an HTTP 200 response and a JSON object with no properties\.

To begin, disconnect the external key store\. Before running this command, replace the example custom key store ID with a valid one\.

```
$ aws kms disconnect-custom-key-store --custom-key-store-id cks-1234567890abcdef0
```

After the external key store is disconnected, you can use the [DeleteCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteCustomKeyStore.html) operation to delete it\. 

```
$ aws kms delete-custom-key-store --custom-key-store-id cks-1234567890abcdef0
```

To confirm that the external key store is deleted, use the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation\.

```
$ aws kms describe-custom-key-stores
            
{
    "CustomKeyStores": []
}
```

If you specify a custom key store name or ID that no longer exists, AWS KMS returns a `CustomKeyStoreNotFoundException` exception\.

```
$ aws kms describe-custom-key-stores --custom-key-store-id cks-1234567890abcdef0

An error occurred (CustomKeyStoreNotFoundException) when calling the DescribeCustomKeyStore operation:
```