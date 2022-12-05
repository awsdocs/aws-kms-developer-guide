# Creating KMS keys in an external key store<a name="create-xks-keys"></a>

After you have [created](create-xks-keystore.md) and [connected](xks-connect-disconnect.md) your external key store, you can create [AWS KMS keys](concepts.md#kms_keys) in your key store\. They must be [symmetric encryption KMS keys](concepts.md#symmetric-cmks) with an origin value of **External key store** \(`EXTERNAL_KEY_STORE`\)\. You cannot create [asymmetric KMS keys](symmetric-asymmetric.md#asymmetric-cmks), [HMAC KMS keys](hmac.md) or KMS keys with [imported key material](importing-keys.md) in a custom key store\. Also, you cannot use symmetric encryption KMS keys in a custom key store to generate asymmetric data key pairs\.

A KMS key in an external key store might have poorer latency, durability and availability than a standard KMS key because it depends on components located outside of AWS\. Before creating or using a KMS key in an external key store, verify that you require a key with external key store properties\.

**Note**  
Some external key managers provide a simpler method for creating KMS keys in an external key store\. For details, see your external key manager documentation\.

To create a KMS key in your external key store, you specify the following:
+ The ID of your external key store\.
+ A [key material origin](concepts.md#key-origin) of External key store \(`EXTERNAL_KEY_STORE`\)\.
+ The ID of an existing [external key](keystore-external.md#concept-external-key) in the [external key manager](keystore-external.md#concept-ekm) associated with your external key store\. This external key serves as key material for the KMS key\. You cannot change the external key ID after you create the KMS key\.

  AWS KMS provides the external key ID to your external key store proxy in requests for encryption and decryption operations\. AWS KMS cannot directly access your external key manager or any of its cryptographic keys\.

In addition to the external key, a KMS key in an external key store also has AWS KMS key material\. All data encrypted under the KMS key is first encrypted in AWS KMS using the key's AWS KMS key material and then by your external key manager using your external key\. This [double encryption](keystore-external.md#concept-double-encryption) process ensures that ciphertext protected by a KMS key in an external key store is at least as strong as ciphertext protected only by AWS KMS\. For details, see [How external key stores work](keystore-external.md#xks-how-it-works)\.

When the `CreateKey` operation succeeds, the [key state](key-state.md) of the new KMS key is `Enabled`\. When you [view a KMS key in an external key store](view-xks-key.md) you can see typical properties, like its key ID, [key spec](concepts.md#key-spec), [key usage](concepts.md#key-usage), [key state](key-state.md), and creation date\. But you can also see the ID and [connection state](xks-connect-disconnect.md#xks-connection-state) of the external key store and the ID of the external key\.

If your attempt to create a KMS key in your external key store fails, use the error message to identify the cause\. It might indicate that the external key store is not connected \(`CustomKeyStoreInvalidStateException`\), that your external key store proxy cannot find an external key with the specified external key ID \(`XksKeyNotFoundException`\), or that the external key is already associated with a KMS key in the same external key store `XksKeyAlreadyInUseException`\.

\.

For an example of the AWS CloudTrail log of the operation that creates a KMS key in an external key store, see [CreateKey](ct-createkey.md)\.

**Topics**
+ [Requirements for a KMS key in an external key store](#xks-key-requirements)
+ [Create a KMS key in an external key store \(console\)](#create-xks-key-console)
+ [Create a KMS key in an external key store \(AWS KMS API\)](#create-xks-key-api)

## Requirements for a KMS key in an external key store<a name="xks-key-requirements"></a>

To create a KMS key in an external key store, the following properties are required of the external key store, the KMS key, and the external key that serves as the external cryptographic key material for the KMS key\.

**External key store requirements**
+ Must be connected to its external key store proxy\.

  To view the [connection state](xks-connect-disconnect.md#xks-connection-state) of your external key store, see [Viewing an external key store](view-xks-keystore.md)\. To connect your external key store, see [Connecting and disconnecting an external key store](xks-connect-disconnect.md)\. 

**KMS key requirements**

You cannot change these properties after you create the KMS key\.
+ Key spec: SYMMETRIC\_DEFAULT
+ Key usage: ENCRYPT\_DECRYPT
+ Key material origin: EXTERNAL\_KEY\_STORE
+ Multi\-Region: FALSE

**External key requirements**
+ 256\-bit AES cryptographic key \(256 random bits\)\. The `KeySpec` of the external key must be `AES_256`\.
+ Enabled and available for use\. The `Status` of the external key must be `ENABLED`\.
+ Configured for encryption and decryption\. The `KeyUsage` of the external key must include `ENCRYPT` and `DECRYPT`\.
+ Used only with this KMS key\. Each `KMS key` in an external key store must be associated with a different external key\.

  AWS KMS also recommends that the external key be used exclusively for the external key store\. This restriction makes it easier to identify and resolve problems with the key\.
+ Accessible by the [external key store proxy](keystore-external.md#concept-xks-proxy) for the external key store\.

  If the external key store proxy can't find the key using the specified external key ID, the `CreateKey` operation fails\.
+ Can handle the anticipated traffic that your use of AWS services generates\. AWS KMS recommends that external keys be prepared to handle up to 1800 requests per second\.

## Create a KMS key in an external key store \(console\)<a name="create-xks-key-console"></a>

There are two ways to create a KMS key in an external key store\. If you choose your external key store before your create your key, AWS KMS chooses all required KMS key properties for you and fills in the ID of your external key store\. This method avoids errors you might make when creating your KMS key\.

**Method 1: Start in your external key store**

To use this method, choose your external key store, then create a KMS key\. The AWS KMS console chooses all required properties for you and fills in the ID of your external key store\. This method avoids many errors you might make when creating your KMS key\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Custom key stores**, **External key stores**\.

1. Choose the name of your external key store\.

1. In the top right corner, choose **Create a KMS key in this key store**\.

   If the external key store is *not* connected, you will be prompted to connect it\. If the connection attempt fails, you need to resolve the problem and connect the external key store before you can create a new KMS key in it\.

   If the external key store is connected, you are redirected to the **Customer managed keys** page for creating a key\. The required **Key configuration** values are already chosen for you\. Also, the custom key store ID of your external key store is filled in, although you can change it\.

1. Enter the key ID of an [external key](keystore-external.md#concept-external-key) in your [external key manager](keystore-external.md#concept-ekm)\. This external key must [fulfill the requirements](#xks-key-requirements) for use with a KMS key\. You cannot change this value after the key is created\.

   If the external key has multiple IDs, enter the key ID that the external key store proxy uses to identify the external key\. 

1. Confirm that you intend to create a KMS key in the specified external key store\.

1. Choose **Next**\.

   The remainder of this procedure is the same as [creating a standard KMS key](create-keys.md)\. 

1. Type an alias \(required\) and a description \(optional\) for the KMS key\.

1. \(Optional\)\. On the **Add Tags** page, add tags that identify or categorize your KMS key\.

   When you add tags to your AWS resources, AWS generates a cost allocation report with usage and costs aggregated by tags\. Tags can also be used to control access to a KMS key\. For information about tagging KMS keys, see [Tagging keys](tagging-keys.md) and [ABAC for AWS KMS](abac.md)\. 

1. Choose **Next**\.

1. In the **Key Administrators** section, select the IAM users and roles who can manage the KMS key\. For more information, see [Allows key administrators to administer the KMS key](key-policy-default.md#key-policy-default-allow-administrators)\.
**Note**  
IAM policies can give other IAM users and roles permission to use the KMS key\.

1. \(Optional\) To prevent these key administrators from deleting this KMS key, clear **Allow key administrators to delete this key** check box\.

   Deleting a KMS key is a destructive and irreversible operation that can render ciphertext unrecoverable\. You cannot recreate a symmetric KMS key in an external key store, even if you have the external key material\. However, deleting a KMS key has no effect on its associated external key\. For information about deleting a KMS key from an external key store, see [Scheduling deletion of KMS keys from an external key store](delete-xks-key.md)\.

1. Choose **Next**\.

1. In the **This account** section, select the IAM users and roles in this AWS account that can use the KMS key in [cryptographic operations](concepts.md#cryptographic-operations)\. For more information, see [Allows key users to use the KMS key](key-policy-default.md#key-policy-default-allow-users)\.
**Note**  
IAM policies can give other IAM users and roles permission to use the KMS key\.

1. \(Optional\) You can allow other AWS accounts to use this KMS key for cryptographic operations\. To do so, in the **Other AWS accounts** section at the bottom of the page, choose **Add another AWS account** and enter the AWS account ID of an external account\. To add multiple external accounts, repeat this step\.
**Note**  
Administrators of the other AWS accounts must also allow access to the KMS key by creating IAM policies for their users\. For more information, see [Allowing users in other accounts to use a KMS key](key-policy-modifying-external-accounts.md)\.

1. Choose **Next**\.

1. Review the key settings that you chose\. You can still go back and change all settings\.

1. When you're done, choose **Finish** to create the key\.

**Method 2: Start in Customer managed keys**

This procedure is the same as the procedure to create a symmetric encryption key with AWS KMS key material\. But, in this procedure, you specify the custom key store ID of the external key store and the key ID of the external key\. You must also specify the [required property values](#xks-key-requirements) for a KMS key in an external key store, such as the key spec and key usage\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Choose **Create key**\.

1. Choose **Symmetric**\.

1. In **Key usage**, the **Encrypt and decrypt** option is selected for you\. Do not change it\. 

1. Choose **Advanced options**\.

1. For **Key material origin**, choose **External key store**\.

1. Confirm that you intend to create a KMS key in the specified external key store\.

1. Choose **Next**\.

1. Choose the row that represents the external key store for your new KMS key\. 

   You cannot choose a disconnected external key store\. To connect a key store that is disconnected, choose the key store name, and then, from **Key store actions**, choose, **Connect**\. For details, see [Connect an external key store \(console\)](xks-connect-disconnect.md#connect-xks-console)\.

1. Enter the key ID of an [external key](keystore-external.md#concept-external-key) in your [external key manager](keystore-external.md#concept-ekm)\. This external key must [fulfill the requirements](#xks-key-requirements) for use with a KMS key\. You cannot change this value after the key is created\.

   If the external key has multiple IDs, enter the key ID that the external key store proxy uses to identify the external key\. 

1. Choose **Next**\.

   The remainder of this procedure is the same as [creating a standard KMS key](create-keys.md)\. 

1. Type an alias and an optional description for the KMS key\.

1. \(Optional\)\. On the **Add Tags** page, add tags that identify or categorize your KMS key\.

   When you add tags to your AWS resources, AWS generates a cost allocation report with usage and costs aggregated by tags\. Tags can also be used to control access to a KMS key\. For information about tagging KMS keys, see [Tagging keys](tagging-keys.md) and [ABAC for AWS KMS](abac.md)\. 

1. Choose **Next**\.

1. In the **Key Administrators** section, select the IAM users and roles who can manage the KMS key\. For more information, see [Allows key administrators to administer the KMS key](key-policy-default.md#key-policy-default-allow-administrators)\.
**Note**  
IAM policies can give other IAM users and roles permission to use the KMS key\.

1. \(Optional\) To prevent these key administrators from deleting this KMS key, clear **Allow key administrators to delete this key** check box\.

   Deleting a KMS key is a destructive and irreversible operation that can render ciphertext unrecoverable\. You cannot recreate a symmetric KMS key in an external key store, even if you have the external key material\. However, deleting a KMS key has no effect on its associated external key\. For information about deleting a KMS key from an external key store, see [Scheduling deletion of KMS keys from an external key store](delete-xks-key.md)\.

1. Choose **Next**\.

1. In the **This account** section, select the IAM users and roles in this AWS account that can use the KMS key in [cryptographic operations](concepts.md#cryptographic-operations)\. For more information, see [Allows key users to use the KMS key](key-policy-default.md#key-policy-default-allow-users)\.
**Note**  
IAM policies can give other IAM users and roles permission to use the KMS key\.

1. \(Optional\) You can allow other AWS accounts to use this KMS key for cryptographic operations\. To do so, in the **Other AWS accounts** section at the bottom of the page, choose **Add another AWS account** and enter the AWS account ID of an external account\. To add multiple external accounts, repeat this step\.
**Note**  
Administrators of the other AWS accounts must also allow access to the KMS key by creating IAM policies for their users\. For more information, see [Allowing users in other accounts to use a KMS key](key-policy-modifying-external-accounts.md)\.

1. Choose **Next**\.

1. Review the key settings that you chose\. You can still go back and change all settings\.

1. When you're done, choose **Finish** to create the key\.

When the procedure succeeds, the display shows the new KMS key in the external key store that you chose\. When you choose the name or alias of the new KMS key, the **Cryptographic configuration** tab on its detail page displays the origin of the KMS key \(**External key store**\), the name, ID, and type of the custom key store, and the ID, key usage, and status of the external key\. If the procedure fails, an error message appears that describes the failure\. For , see [Troubleshooting external key stores](xks-troubleshooting.md)\.

**Tip**  
To make it easier to identify KMS keys in a custom key store, on the **Customer managed keys** page, add the **Origin** and **Custom key store ID** column to the display\. To change the table fields, choose the gear icon in the upper right corner of the page\. For details, see [Customizing your KMS key tables](viewing-keys-console.md#viewing-console-customize)\.

## Create a KMS key in an external key store \(AWS KMS API\)<a name="create-xks-key-api"></a>

To create a new KMS key in an external key store, use the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation\. The following parameters are required:
+ The `Origin` value must be `EXTERNAL_KEY_STORE`\.
+ The `CustomKeyStoreId` parameter identifies your external key store\. The [`ConnectionState`](xks-connect-disconnect.md#xks-connection-state) of the specified external key store must be `CONNECTED`\. To find the `CustomKeyStoreId` and `ConnectionState`, use the `DescribeCustomKeyStores` operation\.
+ The `XksKeyId` parameter identifies the external key\. This external key must [fulfills the requirements](#xks-key-requirements) for association with a KMS key\. 

You can also use any of the optional parameters of the `CreateKey` operation, such as using the `Policy` or [Tags](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) parameters\.

The examples in this section use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

This example command uses the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation to create a KMS key in an external key store\. The response includes the properties of the KMS keys, the ID of the external key store, and the ID, usage, and status of the external key\. For detailed information about these fields, see [Viewing KMS keys in an external key store](view-xks-key.md)\.

Before running this command, replace the example custom key store ID with a valid ID\.

```
$ aws kms create-key --origin EXTERNAL_KEY_STORE --custom-key-store-id cks-1234567890abcdef0 --xks-key-id bb8562717f809024
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