# Troubleshooting a custom key store<a name="fix-keystore"></a>

Custom key stores are designed to be available and resilient\. However, there are some error conditions that you might have to repair to keep your custom key store operational\.

**Topics**
+ [How to fix unavailable CMKs](#fix-unavailable-cmks)
+ [How to fix a failing CMK](#fix-cmk-failed)
+ [How to fix a connection failure](#fix-keystore-failed)
+ [How to fix invalid `kmsuser` credentials](#fix-keystore-password)
+ [How to delete orphaned key material](#fix-keystore-orphaned-key)
+ [How to recover deleted key material for a CMK](#fix-keystore-recover-backing-key)
+ [How to log in as `kmsuser`](#fix-login-as-kmsuser)

## How to fix unavailable CMKs<a name="fix-unavailable-cmks"></a>

The [key state](key-state.md) of customer master keys \(CMKs\) in a custom key store is typically `Enabled`\. Like all CMKs, the key state changes when you disable the CMKs in a custom key store or schedule them for deletion\. However, unlike other CMKs, the CMKs in a custom key store can also have a [key state](key-state.md) of `Unavailable`\. 

A key state of `Unavailable` indicates that the CMK is in a custom key store that was intentionally [disconnected from its AWS CloudHSM cluster](disconnect-keystore.md) and attempts to reconnect it, if any, failed\. While a CMK is unavailable, you can view and manage the CMK, but you cannot use it for [cryptographic operations](use-cmk-keystore.md)\.

To find the key state of a CMK, on the **Customer managed keys** page, view the **Status** field of the CMK\. Or, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation and view the `KeyState` element in the response\. For details, see [Viewing keys](viewing-keys.md)\.

The CMKs in a disconnected custom key store will have a key state of `Unavailable` or `PendingDeletion`\. CMKs that are scheduled for deletion from a custom key store have a `Pending Deletion` key state, even when the custom key store is disconnected from its AWS CloudHSM cluster\. This allows you to cancel the scheduled key deletion without reconnecting the custom key store\. 

To fix an unavailable CMK, [reconnect the custom key store](disconnect-keystore.md)\. After the custom key store is reconnected, the key state of the CMKs in the custom key store is automatically restored to its previous state, such as `Enabled` or `Disabled`\. CMKs that are pending deletion remain in the `PendingDeletion` state\. However, while the problem persists, [enabling and disabling an unavailable CMK](enabling-keys.md) does not change its key state\. The enable or disable action takes effect only when the key becomes available\.

For help with failed connections, see [How to fix a connection failure](#fix-keystore-failed)\. 

## How to fix a failing CMK<a name="fix-cmk-failed"></a>

Problems with creating and using CMKs in custom key stores can be caused by a problem with your custom key store, its associated AWS CloudHSM cluster, the CMK, or its key material\. 

When a custom key store is disconnected from its AWS CloudHSM cluster, the key state of CMKs in the custom key store is `Unavailable`\. All requests to create CMKs in a disconnected custom key store return a `CustomKeyStoreInvalidStateException` exception\. All requests to encrypt, decrypt, re\-encrypt, or generate data keys return a `KMSInvalidStateException` exception\. To fix the problem, [reconnect the custom key store](disconnect-keystore.md)\.

However, your attempts to use a custom key store CMK for [cryptographic operations](use-cmk-keystore.md) might fail even when its key state is `Enabled` and the connection status of the custom key store is `Connected`\. This might be caused by any of the following conditions\.
+ The key material for the CMK might have been deleted from the associated AWS CloudHSM cluster\. To investigate, [find the key handle](view-cmk-keystore.md) of the key material for a CMK and, if necessary, try to [recover the key material](#fix-keystore-recover-backing-key)\.
+ All HSMs were deleted from the AWS CloudHSM cluster that is associated with the custom key store\. To use a CMK in a custom key store in a cryptographic operation, its AWS CloudHSM cluster must contain at least one active HSM\. To verify the number and state of HSMs in an AWS CloudHSM cluster, [use the AWS CloudHSM console](https://docs.aws.amazon.com/cloudhsm/latest/userguide/add-remove-hsm.html) or the [DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html) operation\. To add an HSM to the cluster, use the AWS CloudHSM console or the [CreateHsm](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_CreateHsm.html) operation\.
+ The AWS CloudHSM cluster associated with the custom key store was deleted\. To fix the problem, [create a cluster from a backup](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-cluster-from-backup.html) that is related to the original cluster, such as a backup of the original cluster, or a backup that was used to create the original cluster\. Then, [edit the cluster ID](update-keystore.md) in the custom key store settings\. For instructions, see [How to recover deleted key material for a CMK](#fix-keystore-recover-backing-key)\.

## How to fix a connection failure<a name="fix-keystore-failed"></a>

If you try to [connect a custom key store](disconnect-keystore.md) to its AWS CloudHSM cluster, but the operation fails, the connection status of the custom key store changes to `FAILED`\. To find the status of a custom key store, view the **Status** column of the custom key store in the AWS Management Console or the `ConnectionState` element the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) response\. 

Alternatively, some connection attempts fail quickly due to easily detected cluster configuration errors\. In this case, the **Status** or `ConnectionState` is still `DISCONNECTED`\. These failures return an error message or [exception](https://docs.aws.amazon.com/kms/latest/APIReference/API_ConnectCustomKeyStore.html#API_ConnectCustomKeyStore_Errors) that explains why the attempt failed\. Review the exception description and [cluster requirements](create-keystore.md#before-keystore), fix the problem, [update the custom key store](update-keystore.md), if necessary, and try to connect again\.

When the connection status is `FAILED`, run the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation and see the `ConnectionErrorCode` element in the response\.

**Note**  
When the connection status of a custom key store is `FAILED`, you must [disconnect the custom key store](disconnect-keystore.md) before attempting to reconnect it\. You cannot connect a custom key store with a `FAILED` connection status\.
+ `CLUSTER_NOT_FOUND` indicates that AWS KMS cannot find an AWS CloudHSM cluster with the specified cluster ID\. This might occur because the wrong cluster ID was provided to an API operation or the cluster was deleted and not replaced\. To fix this error, verify the cluster ID, such as by using the AWS CloudHSM console or the [DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html) operation\. If the cluster was deleted, [create a cluster from a recent backup](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-cluster-from-backup.html) of the original\. Then, [disconnect the custom key store](disconnect-keystore.md), [edit the custom key store](update-keystore.md) cluster ID setting, and [reconnect the custom key store](disconnect-keystore.md) to the cluster\.
+ `INSUFFICIENT_CLOUDHSM_HSMS` indicates that the associated AWS CloudHSM cluster does not contain any HSMs\. To connect, the cluster must have at least one HSM\. To find the number of HSMs in the cluster, use the [DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html) operation\. To resolve this error, [add at least one HSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-hsm.html) to the cluster\. If you add multiple HSMs, it's best to create them in different Availability Zones\.
+ `INTERNAL_ERROR` indicates that AWS KMS could not complete the request due to an internal error\. Retry the request\. For `ConnectCustomKeyStore` requests, disconnect the custom key store before trying to connect again\.
+ `INVALID_CREDENTIALS` indicates that AWS KMS cannot log into the associated AWS CloudHSM cluster because it doesn't have the correct `kmsuser` account password\. For help with this error, see [How to fix invalid `kmsuser` credentials](#fix-keystore-password)\.
+ `NETWORK_ERRORS` usually indicates transient network issues\. [Disconnect the custom key store](disconnect-keystore.md), wait a few minutes, and try to connect again\.
+ `SUBNET_NOT_FOUND` indicates that at least one subnet in the AWS CloudHSM cluster configuration was deleted\. If AWS KMS cannot find all of the subnets in the cluster configuration, attempts to connect the custom key store to the AWS CloudHSM cluster fail\. 

  To fix this error, [create a cluster from a recent backup](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-cluster-from-backup.html) of the same AWS CloudHSM cluster\. \(This process creates a new cluster configuration with a VPC and private subnets\.\) Verify that the new cluster meets the [requirements for a custom key store](create-keystore.md#before-keystore), and note the new cluster ID\. Then, to associate the new cluster with your custom key store, [disconnect the custom key store](disconnect-keystore.md), [change the cluster ID](update-keystore.md) of the custom key store to the ID of the new cluster, and try to connect again\.
**Tip**  
To avoid [resetting the `kmsuser` password](#fix-keystore-password), use the most recent backup of the AWS CloudHSM cluster\.
+ `USER_LOCKED_OUT` indicates that the [`kmsuser` crypto user \(CU\) account](key-store-concepts.md#concept-kmsuser) is locked out of the associated AWS CloudHSM cluster due to too many failed password attempts\. For help with this error, see [How to fix invalid `kmsuser` credentials](#fix-keystore-password)\.

  To fix this error, [disconnect the custom key store](disconnect-keystore.md) and use the [changePswd](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-changePswd.html) command in cloudhsm\_mgmt\_util to change the `kmsuser` account password\. Then, [edit the `kmsuser` password setting](update-keystore.md) for the custom key store, and try to connect again\. For help, use the procedure described in the [How to fix invalid `kmsuser` credentials](#fix-keystore-password) topic\.
+ `USER_LOGGED_IN` indicates that the `kmsuser` CU account is logged into the associated AWS CloudHSM cluster\. This prevents AWS KMS from rotating the `kmsuser` account password and logging into the cluster\. To fix this error, log the `kmsuser` CU out of the cluster\. If you changed the `kmsuser` password to log into the cluster, you must also and update the key store password value for the custom key store\. For help, see [How to log out and reconnect](#login-kmsuser-2)\.
+ `USER_NOT_FOUND` indicates that AWS KMS cannot find a `kmsuser` CU account in the associated AWS CloudHSM cluster\. To fix this error, [create a kmsuser CU account](create-keystore.md#kmsuser-concept) in the cluster, and then [update the key store password value](update-keystore.md) for the custom key store\. For help, see [How to fix invalid `kmsuser` credentials](#fix-keystore-password)\.

## How to fix invalid `kmsuser` credentials<a name="fix-keystore-password"></a>

When you [connect a custom key store](disconnect-keystore.md), AWS KMS logs into the associated AWS CloudHSM cluster as the [`kmsuser` crypto user](key-store-concepts.md#concept-kmsuser) \(CU\)\. It remains logged in until the custom key store is disconnected\. The [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) response shows a `ConnectionState` of `FAILED` and `ConnectionErrorCode` value of `INVALID_CREDENTIALS`, as shown in the following example\.

If you disconnect the custom key store and change the `kmsuser` password, AWS KMS cannot log into the AWS CloudHSM cluster with the credentials of the `kmsuser` CU account\. As a result, all attempts to connect the custom key store fail\. The `DescribeCustomKeyStores` response shows a `ConnectionState` of `FAILED` and `ConnectionErrorCode` value of `INVALID_CREDENTIALS`, as shown in the following example\.

```
$ aws kms describe-custom-key-stores --custom-key-store-name ExampleKeyStore
{
   "CustomKeyStores": [
      "CloudHsmClusterId": "cluster-1a23b4cdefg",
      "ConnectionErrorCode": "INVALID_CREDENTIALS"
      "CustomKeyStoreId": "cks-1234567890abcdef0",
      "CustomKeyStoreName": "ExampleKeyStore",
      "TrustAnchorCertificate": "<certificate string appears here>",
      "CreationDate": "1.499288695918E9",
      "ConnectionState": "FAILED"
   ],
}
```

Also, after five failed attempts to log into the cluster with an incorrect password, AWS CloudHSM locks the user account\. To log into the cluster, you must change the account password\. 

If AWS KMS gets a lockout response when it tries to log into the cluster as the `kmsuser` CU, the request to connect the custom key store fails\. The [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) response includes a `ConnectionState` of `FAILED` and `ConnectionErrorCode` value of `USER_LOCKED_OUT`, as shown in the following example\.

```
$ aws kms describe-custom-key-stores --custom-key-store-name ExampleKeyStore
{
   "CustomKeyStores": [
      "CloudHsmClusterId": "cluster-1a23b4cdefg",
      "ConnectionErrorCode": "USER_LOCKED_OUT"
      "CustomKeyStoreId": "cks-1234567890abcdef0",
      "CustomKeyStoreName": "ExampleKeyStore",
      "TrustAnchorCertificate": "<certificate string appears here>",
      "CreationDate": "1.499288695918E9",
      "ConnectionState": "FAILED"
   ],
}
```

To repair any of these conditions, use the following procedure\. 

1. [Disconnect the custom key store](disconnect-keystore.md)\. 

1. Run the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation and view the value of the `ConnectionErrorCode` element in the response\. 
   + If the `ConnectionErrorCode` value is `INVALID_CREDENTIALS`, determine the current password for the `kmsuser` account\. If necessary, use the [changePswd](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-changePswd.html) command in cloudhsm\_mgmt\_util to set the password to a known value\.
   + If the `ConnectionErrorCode` value is `USER_LOCKED_OUT`, you must use the [changePswd](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-changePswd.html) command in cloudhsm\_mgmt\_util to change the `kmsuser` password\.

1. [Edit the `kmsuser` password setting](update-keystore.md) so it matches the current `kmsuser` password in the cluster\. This action tells AWS KMS which password to use to log into the cluster\. It does not change the `kmsuser` password in the cluster\.

1. [Connect the custom key store](disconnect-keystore.md)\.

## How to delete orphaned key material<a name="fix-keystore-orphaned-key"></a>

After scheduling deletion of a CMK from a custom key store, you might need to manually delete the corresponding key material from the associated cluster\. 

When you create a CMK in a custom key store, AWS KMS creates the CMK metadata in AWS KMS and generates the key material in the associated AWS CloudHSM cluster\. When you schedule deletion of a CMK in a custom key store, after the waiting period, AWS KMS deletes the CMK metadata\. Then AWS KMS makes a best effort to delete the corresponding key material from the cluster\. AWS KMS does not attempt to delete key material from cluster backups\.

If AWS KMS cannot delete the key material, such as when the custom key store is disconnected, AWS KMS writes an entry to your AWS CloudTrail logs\. The entry includes the CMK ID, the AWS CloudHSM cluster ID, and the key handle of the key material\.

To delete the key material from the associated AWS CloudHSM cluster, use a procedure like the following one\. This example uses the AWS CLI and AWS CloudHSM command line tools, but you can use the AWS Management Console instead of the CLI\.

1. Disconnect the custom key store, if it is not already disconnected, then log into the key\_mgmt\_util, as explained in [How to disconnect and log in](#login-kmsuser-1)\.

1. Use the [deleteKey](https://docs.aws.amazon.com/cloudhsm/latest/userguide/key_mgmt_util-deleteKey.html) command in key\_mgmt\_util to delete the key from the HSMs in the cluster\.

   For example, this command deletes key `262162` from the HSMs in the cluster\. The key handle is listed in the CloudTrail log entry\.

   ```
   Command: deleteKey -k 262162
   
           Cfm3DeleteKey returned: 0x00 : HSM Return: SUCCESS
   
           Cluster Error Status
           Node id 0 and err state 0x00000000 : HSM Return: SUCCESS
           Node id 1 and err state 0x00000000 : HSM Return: SUCCESS
           Node id 2 and err state 0x00000000 : HSM Return: SUCCESS
   ```

1. Log out of key\_mgmt\_util and reconnect the custom key store as described in [How to log out and reconnect](#login-kmsuser-2)\.

## How to recover deleted key material for a CMK<a name="fix-keystore-recover-backing-key"></a>

If the key material for a customer master key is deleted, the CMK is unusable and all ciphertext that was encrypted under the CMK cannot be decrypted\. This can happen if the key material for a CMK in a custom key store is deleted from the associated AWS CloudHSM cluster\. However, it might be possible to recover the key material\.

When you create a customer master key \(CMK\) in a custom key store, AWS KMS logs into the associated AWS CloudHSM cluster and creates the key material for the CMK\. It also changes the password to a value that only it knows and remains logged in as long as the custom key store is connected\. Because only the key owner, that is, the CU who created a key, can delete the key, it is unlikely that the key will be deleted from the HSMs accidentally\. 

However, if the key material for a CMK is deleted from the HSMs in a cluster, the CMK key state eventually changes to `UNAVAILABLE`\. If you attempt to use the CMK for a cryptographic operation, the operation fails with a **KMSInvalidStateException** exception\. Most importantly, any data that was encrypted under the CMK cannot be decrypted\.

Under certain circumstances, you can recover deleted key material by [creating a cluster from a backup](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-cluster-from-backup.html) that contains the key material\. This strategy works only when at least one backup was created while the key existed and before it was deleted\. 

Use the following process to recover the key material\.

1. Find a cluster backup that contains the key material\. The backup must also contain all users and keys that you need to support the cluster and its encrypted data\.

   Use the [DescribeBackups](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeBackups.html) operation to list the backups for a cluster\. Then use the backup timestamp to help you select a backup\. To limit the output to the cluster that is associated with the custom key store, use the `Filters` parameter, as shown in the following example\. 

   ```
   $ aws cloudhsmv2 describe-backups --filters clusterIds=<cluster ID>
   {
       "Backups": [
           {
               "ClusterId": "cluster-1a23b4cdefg",
               "BackupId": "backup-9g87f6edcba",
               "CreateTimestamp": 1536667238.328,
               "BackupState": "READY"
           },
                ...
       ]
   }
   ```

1. [Create a cluster from the selected backup](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-cluster-from-backup.html)\. Verify that the backup contains the deleted key and other users and keys that the cluster requires\. 

1. [Disconnect the custom key store](disconnect-keystore.md) so you can edit its properties\.

1. [Edit the cluster ID](update-keystore.md) of the custom key store\. Enter the cluster ID of the cluster that you created from the backup\. Because the cluster shares a backup history with the original cluster, the new cluster ID should be valid\. 

1. [Reconnect the custom key store](disconnect-keystore.md)\.

## How to log in as `kmsuser`<a name="fix-login-as-kmsuser"></a>

To create and manage the key material in the AWS CloudHSM cluster for your custom key store, AWS KMS uses the [kmsuser crypto user \(CU\) account](key-store-concepts.md#concept-kmsuser)\. You [create the `kmsuser` CU account](create-keystore.md#before-keystore) in your cluster and provide its password to AWS KMS when you create your custom key store\.

In general, AWS KMS manages the `kmsuser` account\. However, for some tasks, you need to disconnect the custom key store, log into the cluster as the `kmsuser` CU, and use the cloudhsm\_mgmt\_util and key\_mgmt\_util command line tools\.

**Note**  
While a custom key store is disconnected, all attempts to create customer master keys \(CMKs\) in the custom key store or to use existing CMKs in cryptographic operations will fail\. This action can prevent users from storing and accessing sensitive data\.

This topic explains how to [disconnect your custom key store and log in](#login-kmsuser-1) as `kmsuser`, run the AWS CloudHSM command line tool, and [log out and reconnect your custom key store](#login-kmsuser-2)\.

**Topics**
+ [How to disconnect and log in](#login-kmsuser-1)
+ [How to log out and reconnect](#login-kmsuser-2)

### How to disconnect and log in<a name="login-kmsuser-1"></a>

Use the following procedure each time to need to log into an associated cluster as the `kmsuser` CU\.

1. Disconnect the custom key store, if it is not already disconnected\. You can use the AWS Management Console or AWS KMS API\. 

   While your custom key is connected, AWS KMS is logged in as the `kmsuser`\. This prevents you from logging in as `kmsuser` or changing the `kmsuser` password\.

   For example, this command uses [DisconnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisconnectCustomKeyStore.html) to disconnect an example key store\. Replace the example custom key store ID with a valid one\.

   ```
   $ aws kms disconnect-custom-key-store --custom-key-store-id cks-1234567890abcdef0
   ```

1. Start cloudhsm\_mgmt\_util\. Use the procedure described in [Prepare to run cloudhsm\_mgmt\_util](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-getting-started.html#cloudhsm_mgmt_util-setup) section of the *AWS CloudHSM User Guide*\.

1. Log into cloudhsm\_mgmt\_util on the AWS CloudHSM cluster as a [crypto officer](https://docs.aws.amazon.com/cloudhsm/latest/userguide/hsm-users.html#crypto-officer) \(CO\)\. 

   For example, this command logs in as a CO named admin\. Replace the example CO user name and password with valid values\.

   ```
   aws-cloudhsm>loginHSM CO admin <password>
   loginHSM success on server 0(10.0.2.9)
   loginHSM success on server 1(10.0.3.11)
   loginHSM success on server 2(10.0.1.12)
   ```

1. Use the [changePswd](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-changePswd.html) command to change the password of the `kmsuser` account to one that you know\. \(AWS KMS rotates the password when you connect your custom key store\.\) The password must consist of 7\-32 alphanumeric characters\. It is case\-sensitive and cannot contain any special characters\.

   For example, this command changes the `kmsuser` password to `tempPassword`\.

   ```
   aws-cloudhsm>changePswd CU kmsuser tempPassword
   
   *************************CAUTION********************************
   This is a CRITICAL operation, should be done on all nodes in the
   cluster. Cav server does NOT synchronize these changes with the
   nodes on which this operation is not executed or failed, please
   ensure this operation is executed on all nodes in the cluster.
   ****************************************************************
   
   Do you want to continue(y/n)?y
   Changing password for kmsuser(CU) on 3 nodes
   ```

1. Log into key\_mgmt\_util or cloudhsm\_mgmt\_util as `kmsuser` using the password that you set\. For detailed instructions, see [Getting Started with cloudhsm\_mgmt\_util](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-getting-started.html) and [Getting Started with key\_mgmt\_util](https://docs.aws.amazon.com/cloudhsm/latest/userguide/key_mgmt_util-getting-started.html)\. The tool that you use depends on your task\.

   For example, this command logs into key\_mgmt\_util\.

   ```
   Command: loginHSM -u CU -s kmsuser -p tempPassword
   Cfm3LoginHSM returned: 0x00 : HSM Return: SUCCESS
   
   Cluster Error Status
   Node id 0 and err state 0x00000000 : HSM Return: SUCCESS
   Node id 1 and err state 0x00000000 : HSM Return: SUCCESS
   Node id 2 and err state 0x00000000 : HSM Return: SUCCESS
   ```

### How to log out and reconnect<a name="login-kmsuser-2"></a>

1. Perform the task, then log out of the command line tool\. If you do not log out, attempts to reconnect your custom key store will fail\.

   ```
   Command:  logoutHSM
   Cfm3LogoutHSM returned: 0x00 : HSM Return: SUCCESS
   
   Cluster Error Status
   Node id 0 and err state 0x00000000 : HSM Return: SUCCESS
   Node id 1 and err state 0x00000000 : HSM Return: SUCCESS
   ```

1. [Edit the `kmsuser` password setting](update-keystore.md) for the custom key store\. 

   This tells AWS KMS the current password for `kmsuser` in the cluster\. If you omit this step, AWS KMS will not be able to log into the cluster as `kmsuser`, and all attempts to reconnect your custom key store will fail\. You can use the AWS Management Console or the `KeyStorePassword` parameter of the [UpdateCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateCustomKeyStore.html) operation\.

   For example, this command tells AWS KMS that the current password is `tempPassword`\. Replace the example password with the actual one\. 

   ```
   $ aws kms update-custom-key-store --custom-key-store-id cks-1234567890abcdef0 --key-store-password tempPassword
   ```

1. Reconnect the custom key store to AWS KMS\. Replace the example custom key store ID with a valid one\. During the connection process, AWS KMS changes the `kmsuser` password to a value that only it knows\.

   The [ConnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_ConnectCustomKeyStore.html) operation returns quickly, but the connection process can take an extended period of time\. The initial response does not indicate the success of the connection process\.

   ```
   $ aws kms connect-custom-key-store --custom-key-store-id cks-1234567890abcdef0
   ```

1. Use the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation to verify that the custom key store is connected\. Replace the example custom key store ID with a valid one\.

   In this example, the connection state field shows that the custom key store is now connected\.

   ```
   $ aws kms describe-custom-key-stores --custom-key-store-id cks-1234567890abcdef0
   {
      "CustomKeyStores": [
         "CustomKeyStoreId": "cks-1234567890abcdef0",
         "CustomKeyStoreName": "ExampleKeyStore",
         "CloudHsmClusterId": "cluster-1a23b4cdefg",
         "TrustAnchorCertificate": "<certificate string appears here>",
         "CreationDate": "1.499288695918E9",
         "ConnectionState": "CONNECTED"
      ],
   }
   ```