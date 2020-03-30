# Finding CMKs and key material<a name="find-key-material"></a>

If you manage a custom key store, you might need to identify the CMKs in each custom key store\. For example, you might need to do some of the following tasks\.
+ Track the CMKs in custom key store in AWS CloudTrail logs\. 
+ Predict the effect on CMKs of disconnecting a custom key store\. 
+ Schedule deletion of CMKs before you delete a custom key store\. 

In addition, you might want to identify the keys in your AWS CloudHSM cluster that serve as key material for your CMKs\. Although AWS KMS manages the CMKs and their key material, you still retain control of and responsibility for the management of your AWS CloudHSM cluster, its HSMs and backups and the keys in the HSMs\. You might need to identify the keys in order to audit the key material, protect it from accidental deletion, or delete it from HSMs and cluster backups after deleting the CMK\.

All key material for the CMKs in your custom key store is owned by the [`kmsuser` crypto user](key-store-concepts.md#concept-kmsuser) \(CU\)\. AWS KMS sets the key label attribute, which is viewable only in AWS CloudHSM, to the Amazon Resource Name \(ARN\) of the CMK\.

To find CMKs and key material, use any of the following techniques\.
+ [Find the CMKs in a custom key store](#find-cmk-in-keystore) — How to identify the CMKs in one or all of your custom key stores\.
+ [Find all keys for a custom key store](#find-all-kmsuser-keys) — How to find all keys in your cluster that serve as key material for the CMKs in your custom key store\.
+ [Find the key for a CMK](#find-handle-for-cmk-id) — How to find the key in your cluster that serves as key material for a particular CMK in your custom key store\.
+ [Find the CMK for a key](#find-label-for-key-handle) — How to find the CMK for a particular key in your cluster\. 

## Find the CMKs in a custom key store<a name="find-cmk-in-keystore"></a>

If you manage a custom key store, you might need to identify the CMKs in each custom key store\. You can use this information track the CMK operations in AWS CloudTrail logs, predict the effect on CMKs of disconnecting a custom key store, or schedule deletion of CMKs before you delete a custom key store\. 

### To find the CMKs in a custom key store \(console\)<a name="find-cmk-in-keystore-console"></a>

To find the CMKs in a particular custom key store, on the **Customer Managed Keys** page, view the values in the **Custom Key Store Name** or **Custom Key Store ID** fields\. To identify CMKs in any custom key store, look for CMKs with an **Origin** value of **CloudHSM**\. To add optional columns to the display, choose the gear icon in the upper right corner of the page\.

### To find the CMKs in a custom key store \(API\)<a name="find-cmk-in-keystore-api"></a>

To find the CMKs in a custom key store, use the [ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html) and [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operations and then filter the `CustomKeyStoreId` value\. Before running the examples, replace the fictitious custom key store ID values with a valid value\.

------
#### [ Bash ]

To find CMKs in a particular custom key store, get all of your CMKs in the account and Region\. Then filter the ID of the custom key store\. 

```
for key in $(aws kms list-keys --query 'Keys[*].KeyId' --output text) ; 
do aws kms describe-key --key-id $key | 
grep '"CustomKeyStoreId": "cks-1234567890abcdef0"' --context 100; done
```

To get CMKs in any custom key store in the account and Region, search for `CustomKeyStoreId` values that begin with `cks-`\.

```
for key in $(aws kms list-keys --query 'Keys[*].KeyId' --output text) ; 
do aws kms describe-key --key-id $key | 
grep '"CustomKeyStoreId": "cks-"' --context 100; done
```

------
#### [ PowerShell ]

To find CMKs in a particular custom key store, use the [Get\-KmsKeyList](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-KMSKeyList.html) [Get\-KmsKey](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-KMSKey.html) cmdlets to get all of your CMKs in the account and Region\. Then filter for the ID of the custom key store\. 

```
PS C:\> (Get-KMSKeyList).KeyArn | foreach {Get-KMSKey -KeyId $_} | where CustomKeyStoreId -eq 'cks-1234567890abcdef0'
```

To get CMKs in any custom key store in the account and Region, use the `-like` comparison operator\. All custom key store identifiers begin with `cks-`\.

```
PS C:\> (Get-KMSKeyList).KeyArn | foreach {Get-KMSKey -KeyId $_} | where CustomKeyStoreId -like 'cks*'
```

------

## Find all keys for a custom key store<a name="find-all-kmsuser-keys"></a>

You can identify the keys in your AWS CloudHSM cluster that serve as key material for your custom key store\. To do that, use the [findAllKeys](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-findAllKeys.html) command in cloudhsm\_mgmt\_util to find the key handles of all keys that `kmsuser` owns or shares\. Unless you have logged in as `kmsuser` and created keys outside of AWS KMS, all of the keys that `kmsuser` owns represent key material for AWS KMS CMKs\. 

Any crypto officer in the cluster can run this command without disconnecting the custom key store\.

1. Start cloudhsm\_mgmt\_util by using the procedure described in the [Prepare to run cloudhsm\_mgmt\_util](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-getting-started.html#cloudhsm_mgmt_util-setup) topic\.

1. [Log into cloudhsm\_mgmt\_util](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-getting-started.html#cloudhsm_mgmt_util-log-in) using a crypto officer \(CO\) account\. 

1. Use the [listUsers](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-listUsers.html) command to find the user ID of the `kmsuser` crypto user\.

   In this example, `kmsuser` has user ID 3\.

   ```
   aws-cloudhsm> listUsers
   Users on server 0(10.0.0.1):
   Number of users found:3
   
       User Id             User Type       User Name            MofnPubKey    LoginFailureCnt         2FA
            1              PCO             admin                      NO               0               NO
            2              AU              app_user                   NO               0               NO
            3              CU              kmsuser                    NO               0               NO
   ```

1. Use the [findAllKeys](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-findAllKeys.html) command to find the key handles of all keys that `kmsuser` owns or shares\. Replace the example user ID with the actual user ID of `kmsuser` in your cluster\.

   The example output shows that `kmsuser` owns keys with key handles 8, 9, and 262162 on both HSMs in the cluster\.

   ```
   aws-cloudhsm> findAllKeys 3 0
   Keys on server 0(10.0.0.1):
   Number of keys found 3
   number of keys matched from start index 0::6
   8,9,262162
   findAllKeys success on server 0(10.0.0.1)
   
   Keys on server 1(10.0.0.2):
   Number of keys found 6
   number of keys matched from start index 0::6
   8,9,262162
   findAllKeys success on server 1(10.0.0.2)
   ```

## Find the CMK for a key<a name="find-label-for-key-handle"></a>

If you know the key handle of a key that `kmsuser` owns in the cluster, you can use the key label to identify the associated CMK in your custom key store\.

When AWS KMS creates the key material for a CMK in your AWS CloudHSM cluster, it writes the Amazon Resource Name \(ARN\) of the CMK in the key label\. Unless you have changed the label value, you can use the [getAttribute](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-getAttribute.html) command in key\_mgmt\_util or cloudhsm\_mgmt\_util to associate the key with its CMK\. 

To run this procedure, you need to disconnect the custom key store temporarily so you can log in as the `kmsuser` CU\.

**Note**  
While a custom key store is disconnected, all attempts to create customer master keys \(CMKs\) in the custom key store or to use existing CMKs in cryptographic operations will fail\. This action can prevent users from storing and accessing sensitive data\.

1. Disconnect the custom key store, if it is not already disconnected\., then log into the key\_mgmt\_util as `kmsuser`, as explained in [How to disconnect and log in](fix-keystore.md#login-kmsuser-1)\.

1. Use the `getAttribute` command in [key\_mgmt\_util](https://docs.aws.amazon.com/cloudhsm/latest/userguide/key_mgmt_util-getAttribute.html) or [cloudhsm\_mgmt\_util](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-getAttribute.html) to get the label attribute \(`OBJ_ATTR_LABEL`, attribute `3`\) for a particular key handle\. 

   For example, this command uses `getAttribute` in cloudhsm\_mgmt\_util to get the label attribute \(attribute `3`\) of the key with key handle `262162`\. The output shows that key `262162` serves as key material for the CMK with ARN `arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab`\. Before running this command, replace the example key handle with a valid one\. 

   For a list of key attributes, use the [listAttributes](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-listAttributes.html) command or see the [Key Attribute Reference](https://docs.aws.amazon.com/cloudhsm/latest/userguide/key-attribute-table.html) in the *AWS CloudHSM User Guide*\.

   ```
   aws-cloudhsm> getAttribute 262162 3
   
   Attribute Value on server 0(10.0.1.10):
   OBJ_ATTR_LABEL
   arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
   
   Attribute Value on server 1(10.0.1.12):
   OBJ_ATTR_EXTRACTABLE
   arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
   ```

1. Log out of key\_mgmt\_util or cloudhsm\_mgmt\_util and reconnect the custom key store as explained in [How to log out and reconnect](fix-keystore.md#login-kmsuser-2)\.

## Find the key for a CMK<a name="find-handle-for-cmk-id"></a>

You can use the CMK ID of a CMK in a custom key store to identify the key in your cluster that serves as its key material\. Then you can use its key handle to identify the key in AWS CloudHSM client commands\. 

When AWS KMS creates the key material for a CMK in your AWS CloudHSM cluster, it writes the Amazon Resource Name \(ARN\) of the CMK in the key label\. Unless you have changed the label value, you can use the [findKey](https://docs.aws.amazon.com/cloudhsm/latest/userguide/key_mgmt_util-findKey.html) command in key\_mgmt\_util to get the key handle of the key material for the CMK\. To run this procedure, you need to disconnect the custom key store temporarily so you can log in as the `kmsuser` CU\. 

**Note**  
While a custom key store is disconnected, all attempts to create customer master keys \(CMKs\) in the custom key store or to use existing CMKs in cryptographic operations will fail\. This action can prevent users from storing and accessing sensitive data\.

1. Disconnect the custom key store, if it is not already disconnected, then log into the key\_mgmt\_util as `kmsuser`, as explained in [How to disconnect and log in](fix-keystore.md#login-kmsuser-1)\.

1. Use the [findKey](https://docs.aws.amazon.com/cloudhsm/latest/userguide/key_mgmt_util-findKey.html) command in key\_mgmt\_util to search for a key with a label that matches the ARN of a CMK in your custom key store\. Replace the example CMK ARN in the value of the `-l` \(lower\-case L for 'label'\) parameter with a valid CMK ARN\. 

   For example, this command finds the key with a label that matches the example CMK ARN, `arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab`\. The example output shows that the key with key handle `262162` has the specified CMK ARN in its label\. You can now use this key handle in other key\_mgmt\_util commands\.

   ```
   Command: findKey -l arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
   Total number of keys present 1
   
    number of keys matched from start index 0::1
   262162
   
           Cluster Error Status
           Node id 1 and err state 0x00000000 : HSM Return: SUCCESS
           Node id 0 and err state 0x00000000 : HSM Return: SUCCESS
   
           Cfm3FindKey returned: 0x00 : HSM Return: SUCCESS
   ```

1. Log out of key\_mgmt\_util and reconnect the custom key store as explained in [How to log out and reconnect](fix-keystore.md#login-kmsuser-2)\.