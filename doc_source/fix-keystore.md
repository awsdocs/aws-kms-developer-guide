# Troubleshooting a custom key store<a name="fix-keystore"></a>

AWS CloudHSM key stores are designed to be available and resilient\. However, there are some error conditions that you might have to repair to keep your AWS CloudHSM key store operational\.

**Topics**
+ [How to fix unavailable KMS keys](#fix-unavailable-cmks)
+ [How to fix a failing KMS key](#fix-cmk-failed)
+ [How to fix a connection failure](#fix-keystore-failed)
+ [How to respond to a cryptographic operation failure](#fix-keystore-communication)
+ [How to fix invalid `kmsuser` credentials](#fix-keystore-password)
+ [How to delete orphaned key material](#fix-keystore-orphaned-key)
+ [How to recover deleted key material for a KMS key](#fix-keystore-recover-backing-key)
+ [How to log in as `kmsuser`](#fix-login-as-kmsuser)

## How to fix unavailable KMS keys<a name="fix-unavailable-cmks"></a>

The [key state](key-state.md) of AWS KMS keys in an AWS CloudHSM key store is typically `Enabled`\. Like all KMS keys, the key state changes when you disable the KMS keys in an AWS CloudHSM key store or schedule them for deletion\. However, unlike other KMS keys, the KMS keys in a custom key store can also have a [key state](key-state.md) of `Unavailable`\. 

A key state of `Unavailable` indicates that the KMS key is in a custom key store that was intentionally [disconnected](disconnect-keystore.md) and attempts to reconnect it, if any, failed\. While a KMS key is unavailable, you can view and manage the KMS key, but you cannot use it for [cryptographic operations](use-cmk-keystore.md)\.

To find the key state of a KMS key, on the **Customer managed keys** page, view the **Status** field of the KMS key\. Or, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation and view the `KeyState` element in the response\. For details, see [Viewing keys](viewing-keys.md)\.

The KMS keys in a disconnected custom key store will have a key state of `Unavailable` or `PendingDeletion`\. KMS keys that are scheduled for deletion from a custom key store have a `Pending Deletion` key state, even when the custom key store is disconnected\. This allows you to cancel the scheduled key deletion without reconnecting the custom key store\. 

To fix an unavailable KMS key, [reconnect the custom key store](disconnect-keystore.md)\. After the custom key store is reconnected, the key state of the KMS keys in the custom key store is automatically restored to its previous state, such as `Enabled` or `Disabled`\. KMS keys that are pending deletion remain in the `PendingDeletion` state\. However, while the problem persists, [enabling and disabling an unavailable KMS key](enabling-keys.md) does not change its key state\. The enable or disable action takes effect only when the key becomes available\.

For help with failed connections, see [How to fix a connection failure](#fix-keystore-failed)\. 

## How to fix a failing KMS key<a name="fix-cmk-failed"></a>

Problems with creating and using KMS keys in AWS CloudHSM key stores can be caused by a problem with your AWS CloudHSM key store, its associated AWS CloudHSM cluster, the KMS key, or its key material\. 

When an AWS CloudHSM key store is disconnected from its AWS CloudHSM cluster, the key state of KMS keys in the custom key store is `Unavailable`\. All requests to create KMS keys in a disconnected AWS CloudHSM key store return a `CustomKeyStoreInvalidStateException` exception\. All requests to encrypt, decrypt, re\-encrypt, or generate data keys return a `KMSInvalidStateException` exception\. To fix the problem, [reconnect the AWS CloudHSM key store](disconnect-keystore.md)\.

However, your attempts to use a KMS key in an AWS CloudHSM key store for [cryptographic operations](use-cmk-keystore.md) might fail even when its key state is `Enabled` and the connection state of the AWS CloudHSM key store is `Connected`\. This might be caused by any of the following conditions\.
+ The key material for the KMS key might have been deleted from the associated AWS CloudHSM cluster\. To investigate, [find the key handle](view-cmk-keystore.md) of the key material for a KMS key and, if necessary, try to [recover the key material](#fix-keystore-recover-backing-key)\.
+ All HSMs were deleted from the AWS CloudHSM cluster that is associated with the AWS CloudHSM key store\. To use a KMS key in an AWS CloudHSM key store in a cryptographic operation, its AWS CloudHSM cluster must contain at least one active HSM\. To verify the number and state of HSMs in an AWS CloudHSM cluster, [use the AWS CloudHSM console](https://docs.aws.amazon.com/cloudhsm/latest/userguide/add-remove-hsm.html) or the [DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html) operation\. To add an HSM to the cluster, use the AWS CloudHSM console or the [CreateHsm](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_CreateHsm.html) operation\.
+ The AWS CloudHSM cluster associated with the AWS CloudHSM key store was deleted\. To fix the problem, [create a cluster from a backup](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-cluster-from-backup.html) that is related to the original cluster, such as a backup of the original cluster, or a backup that was used to create the original cluster\. Then, [edit the cluster ID](update-keystore.md) in the custom key store settings\. For instructions, see [How to recover deleted key material for a KMS key](#fix-keystore-recover-backing-key)\.
+ The AWS CloudHSM cluster associated with the custom key store did not have any available PKCS \#11 sessions\. This typically occurs during periods of high burst traffic when additional sessions are needed to service the traffic\. To respond to a `KMSInternalException` with an error message about PKCS \#11 sessions, back off and retry the request again\. 

## How to fix a connection failure<a name="fix-keystore-failed"></a>

If you try to [connect an AWS CloudHSM key store](disconnect-keystore.md) to its AWS CloudHSM cluster, but the operation fails, the connection state of the AWS CloudHSM key store changes to `FAILED`\. To find the connection state of an AWS CloudHSM key store, use the AWS KMS console or the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation\. 

Alternatively, some connection attempts fail quickly due to easily detected cluster configuration errors\. In this case, the connection state is still `DISCONNECTED`\. These failures return an error message or [exception](https://docs.aws.amazon.com/kms/latest/APIReference/API_ConnectCustomKeyStore.html#API_ConnectCustomKeyStore_Errors) that explains why the attempt failed\. Review the exception description and [cluster requirements](create-keystore.md#before-keystore), fix the problem, [update the AWS CloudHSM key store](update-keystore.md), if necessary, and try to connect again\.

When the connection state is `FAILED`, run the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation and see the `ConnectionErrorCode` element in the response\.

**Note**  
When the connection state of an AWS CloudHSM key store is `FAILED`, you must [disconnect the AWS CloudHSM key store](disconnect-keystore.md) before attempting to reconnect it\. You cannot connect an AWS CloudHSM key store with a `FAILED` connection state\.
+ `CLUSTER_NOT_FOUND` indicates that AWS KMS cannot find an AWS CloudHSM cluster with the specified cluster ID\. This might occur because the wrong cluster ID was provided to an API operation or the cluster was deleted and not replaced\. To fix this error, verify the cluster ID, such as by using the AWS CloudHSM console or the [DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html) operation\. If the cluster was deleted, [create a cluster from a recent backup](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-cluster-from-backup.html) of the original\. Then, [disconnect the AWS CloudHSM key store](disconnect-keystore.md), [edit the AWS CloudHSM key store](update-keystore.md) cluster ID setting, and [reconnect the AWS CloudHSM key store](disconnect-keystore.md) to the cluster\.
+ `INSUFFICIENT_CLOUDHSM_HSMS` indicates that the associated AWS CloudHSM cluster does not contain any HSMs\. To connect, the cluster must have at least one HSM\. To find the number of HSMs in the cluster, use the [DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html) operation\. To resolve this error, [add at least one HSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-hsm.html) to the cluster\. If you add multiple HSMs, it's best to create them in different Availability Zones\.
+ `INSUFFICIENT_FREE_ADDRESSES_IN_SUBNET` indicates that AWS KMS could not connect the AWS CloudHSM key store to its AWS CloudHSM cluster because at least one [private subnet associated with the cluster](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-subnets.html) doesn't have any available IP addresses\. An AWS CloudHSM key store connection requires one free IP address in each of the associated private subnets, although two are preferable\.

  You [can't add IP addresses](https://aws.amazon.com/premiumsupport/knowledge-center/vpc-ip-address-range/) \(CIDR blocks\) to an existing subnet\. If possible, move or delete other resources that are using the IP addresses in the subnet, such as unused EC2 instances or elastic network interfaces\. Otherwise, you can [create a cluster from a recent backup](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-cluster-from-backup.html) of the AWS CloudHSM cluster with new or existing private subnets that have [more free address space](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html#subnet-sizing)\. Then, to associate the new cluster with your AWS CloudHSM key store, [disconnect the custom key store](disconnect-keystore.md), [change the cluster ID](update-keystore.md) of the AWS CloudHSM key store to the ID of the new cluster, and try to connect again\.
**Tip**  
To avoid [resetting the `kmsuser` password](#fix-keystore-password), use the most recent backup of the AWS CloudHSM cluster\.
+ `INTERNAL_ERROR` indicates that AWS KMS could not complete the request due to an internal error\. Retry the request\. For `ConnectCustomKeyStore` requests, disconnect the AWS CloudHSM key store before trying to connect again\.
+ `INVALID_CREDENTIALS` indicates that AWS KMS cannot log into the associated AWS CloudHSM cluster because it doesn't have the correct `kmsuser` account password\. For help with this error, see [How to fix invalid `kmsuser` credentials](#fix-keystore-password)\.
+ `NETWORK_ERRORS` usually indicates transient network issues\. [Disconnect the AWS CloudHSM key store](disconnect-keystore.md), wait a few minutes, and try to connect again\.
+ `SUBNET_NOT_FOUND` indicates that at least one subnet in the AWS CloudHSM cluster configuration was deleted\. If AWS KMS cannot find all of the subnets in the cluster configuration, attempts to connect the AWS CloudHSM key store to the AWS CloudHSM cluster fail\. 

  To fix this error, [create a cluster from a recent backup](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-cluster-from-backup.html) of the same AWS CloudHSM cluster\. \(This process creates a new cluster configuration with a VPC and private subnets\.\) Verify that the new cluster meets the [requirements for a custom key store](create-keystore.md#before-keystore), and note the new cluster ID\. Then, to associate the new cluster with your AWS CloudHSM key store, [disconnect the custom key store](disconnect-keystore.md), [change the cluster ID](update-keystore.md) of the AWS CloudHSM key store to the ID of the new cluster, and try to connect again\.
**Tip**  
To avoid [resetting the `kmsuser` password](#fix-keystore-password), use the most recent backup of the AWS CloudHSM cluster\.
+ `USER_LOCKED_OUT` indicates that the [`kmsuser` crypto user \(CU\) account](hsm-key-store-concepts.md#concept-kmsuser) is locked out of the associated AWS CloudHSM cluster due to too many failed password attempts\. For help with this error, see [How to fix invalid `kmsuser` credentials](#fix-keystore-password)\.

  To fix this error, [disconnect the AWS CloudHSM key store](disconnect-keystore.md) and use the [changePswd](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-changePswd.html) command in cloudhsm\_mgmt\_util to change the `kmsuser` account password\. Then, [edit the `kmsuser` password setting](update-keystore.md) for the custom key store, and try to connect again\. For help, use the procedure described in the [How to fix invalid `kmsuser` credentials](#fix-keystore-password) topic\.
+ `USER_LOGGED_IN` indicates that the `kmsuser` CU account is logged into the associated AWS CloudHSM cluster\. This prevents AWS KMS from rotating the `kmsuser` account password and logging into the cluster\. To fix this error, log the `kmsuser` CU out of the cluster\. If you changed the `kmsuser` password to log into the cluster, you must also and update the key store password value for the AWS CloudHSM key store\. For help, see [How to log out and reconnect](#login-kmsuser-2)\.
+ `USER_NOT_FOUND` indicates that AWS KMS cannot find a `kmsuser` CU account in the associated AWS CloudHSM cluster\. To fix this error, [create a `kmsuser` CU account](create-keystore.md#kmsuser-concept) in the cluster, and then [update the key store password value](update-keystore.md) for the AWS CloudHSM key store\. For help, see [How to fix invalid `kmsuser` credentials](#fix-keystore-password)\.

## How to respond to a cryptographic operation failure<a name="fix-keystore-communication"></a>

A cryptographic operation that uses a KMS key in a custom key store might fail with an error such as the following\.

```
KMSInvalidStateException: KMS cannot communicate with your CloudHSM cluster
```

Although this is an HTTPS 400 error, it might result from transient network issues\. To respond, begin by retrying the request\. However, if it continues to fail, examine the configuration of your networking components\. This error is most likely caused by the misconfiguration of a networking component, such as a firewall rule or VPC security group rule that is blocking outgoing traffic\. 

## How to fix invalid `kmsuser` credentials<a name="fix-keystore-password"></a>

When you [connect an AWS CloudHSM key store](disconnect-keystore.md), AWS KMS logs into the associated AWS CloudHSM cluster as the [`kmsuser` crypto user](hsm-key-store-concepts.md#concept-kmsuser) \(CU\)\. It remains logged in until the AWS CloudHSM key store is disconnected\. The [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) response shows a `ConnectionState` of `FAILED` and `ConnectionErrorCode` value of `INVALID_CREDENTIALS`, as shown in the following example\.

If you disconnect the AWS CloudHSM key store and change the `kmsuser` password, AWS KMS cannot log into the AWS CloudHSM cluster with the credentials of the `kmsuser` CU account\. As a result, all attempts to connect the AWS CloudHSM key store fail\. The `DescribeCustomKeyStores` response shows a `ConnectionState` of `FAILED` and `ConnectionErrorCode` value of `INVALID_CREDENTIALS`, as shown in the following example\.

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

If AWS KMS gets a lockout response when it tries to log into the cluster as the `kmsuser` CU, the request to connect the AWS CloudHSM key store fails\. The [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) response includes a `ConnectionState` of `FAILED` and `ConnectionErrorCode` value of `USER_LOCKED_OUT`, as shown in the following example\.

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

1. [Disconnect the AWS CloudHSM key store](disconnect-keystore.md)\. 

1. Run the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation and view the value of the `ConnectionErrorCode` element in the response\. 
   + If the `ConnectionErrorCode` value is `INVALID_CREDENTIALS`, determine the current password for the `kmsuser` account\. If necessary, use the [changePswd](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-changePswd.html) command in cloudhsm\_mgmt\_util to set the password to a known value\.
   + If the `ConnectionErrorCode` value is `USER_LOCKED_OUT`, you must use the [changePswd](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-changePswd.html) command in cloudhsm\_mgmt\_util to change the `kmsuser` password\.

1. [Edit the `kmsuser` password setting](update-keystore.md) so it matches the current `kmsuser` password in the cluster\. This action tells AWS KMS which password to use to log into the cluster\. It does not change the `kmsuser` password in the cluster\.

1. [Connect the custom key store](disconnect-keystore.md)\.

## How to delete orphaned key material<a name="fix-keystore-orphaned-key"></a>

After scheduling deletion of a KMS key from an AWS CloudHSM key store, you might need to manually delete the corresponding key material from the associated AWS CloudHSM cluster\. 

When you create a KMS key in an AWS CloudHSM key store, AWS KMS creates the KMS key metadata in AWS KMS and generates the key material in the associated AWS CloudHSM cluster\. When you schedule deletion of a KMS key in an AWS CloudHSM key store, after the waiting period, AWS KMS deletes the KMS key metadata\. Then AWS KMS makes a best effort to delete the corresponding key material from the AWS CloudHSM cluster\. The attempt might fail if AWS KMS cannot access the cluster, such as when it's disconnected from the AWS CloudHSM key store or the `kmsuser` password changes\. AWS KMS does not attempt to delete key material from cluster backups\.

AWS KMS reports the results of its attempt to delete the key material from the cluster in the `DeleteKey` event entry of your AWS CloudTrail logs\. It appears in the `backingKeysDeletionStatus` element of the `additionalEventData` element, as shown in the following example entry\. The entry also includes the KMS key ARN, the AWS CloudHSM cluster ID, and the key handle of the key material \(`backing-key-id`\)\.

```
{
    "eventVersion": "1.08",
    "userIdentity": {
        "accountId": "111122223333",
        "invokedBy": "AWS Internal"
    },
    "eventTime": "2021-12-10T14:23:51Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "DeleteKey",
    "awsRegion": "eu-west-1",
    "sourceIPAddress": "AWS Internal",
    "userAgent": "AWS Internal",
    "requestParameters": null,
    "responseElements": null,
    "additionalEventData": {
        "customKeyStoreId": "cks-1234567890abcdef0",
        "clusterId": "cluster-1a23b4cdefg",
        "backingKeys": "[{\"keyHandle\":\"01\",\"backingKeyId\":\"backing-key-id\"}]",
        "backingKeysDeletionStatus": "[{\"keyHandle\":\"16\",\"backingKeyId\":\"backing-key-id\",\"deletionStatus\":\"FAILURE\"}]"
    },
    "eventID": "c21f1f47-f52b-4ffe-bff0-6d994403cf40",
    "readOnly": false,
    "resources": [
        {
            "accountId": "111122223333",
            "type": "AWS::KMS::Key",
            "ARN": "arn:aws:kms:eu-west-1:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
        }
    ],
    "eventType": "AwsServiceEvent",
    "recipientAccountId": "111122223333",
    "managementEvent": true,
    "eventCategory": "Management"
}
```

To delete the key material from the associated AWS CloudHSM cluster, use a procedure like the following one\. This example uses the AWS CLI and AWS CloudHSM command line tools, but you can use the AWS Management Console instead of the CLI\.

1. Disconnect the AWS CloudHSM key store, if it is not already disconnected, then log into the key\_mgmt\_util, as explained in [How to disconnect and log in](#login-kmsuser-1)\.

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

1. Log out of key\_mgmt\_util and reconnect the AWS CloudHSM key store as described in [How to log out and reconnect](#login-kmsuser-2)\.

## How to recover deleted key material for a KMS key<a name="fix-keystore-recover-backing-key"></a>

If the key material for an AWS KMS key is deleted, the KMS key is unusable and all ciphertext that was encrypted under the KMS key cannot be decrypted\. This can happen if the key material for a KMS key in an AWS CloudHSM key store is deleted from the associated AWS CloudHSM cluster\. However, it might be possible to recover the key material\.

When you create an AWS KMS key \(KMS key\) in an AWS CloudHSM key store, AWS KMS logs into the associated AWS CloudHSM cluster and creates the key material for the KMS key\. It also changes the password to a value that only it knows and remains logged in as long as the AWS CloudHSM key store is connected\. Because only the key owner, that is, the CU who created a key, can delete the key, it is unlikely that the key will be deleted from the HSMs accidentally\. 

However, if the key material for a KMS key is deleted from the HSMs in a cluster, the key state of the KMS key eventually changes to `UNAVAILABLE`\. If you attempt to use the KMS key for a cryptographic operation, the operation fails with a `KMSInvalidStateException` exception\. Most importantly, any data that was encrypted under the KMS key cannot be decrypted\.

Under certain circumstances, you can recover deleted key material by [creating a cluster from a backup](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-cluster-from-backup.html) that contains the key material\. This strategy works only when at least one backup was created while the key existed and before it was deleted\. 

Use the following process to recover the key material\.

1. Find a cluster backup that contains the key material\. The backup must also contain all users and keys that you need to support the cluster and its encrypted data\.

   Use the [DescribeBackups](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeBackups.html) operation to list the backups for a cluster\. Then use the backup timestamp to help you select a backup\. To limit the output to the cluster that is associated with the AWS CloudHSM key store, use the `Filters` parameter, as shown in the following example\. 

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

1. [Disconnect the AWS CloudHSM key store](disconnect-keystore.md) so you can edit its properties\.

1. [Edit the cluster ID](update-keystore.md) of the AWS CloudHSM key store\. Enter the cluster ID of the cluster that you created from the backup\. Because the cluster shares a backup history with the original cluster, the new cluster ID should be valid\. 

1. [Reconnect the AWS CloudHSM key store](disconnect-keystore.md)\.

## How to log in as `kmsuser`<a name="fix-login-as-kmsuser"></a>

To create and manage the key material in the AWS CloudHSM cluster for your AWS CloudHSM key store, AWS KMS uses the [`kmsuser` crypto user \(CU\) account](hsm-key-store-concepts.md#concept-kmsuser)\. You [create the `kmsuser` CU account](create-keystore.md#before-keystore) in your cluster and provide its password to AWS KMS when you create your AWS CloudHSM key store\.

In general, AWS KMS manages the `kmsuser` account\. However, for some tasks, you need to disconnect the AWS CloudHSM key store, log into the cluster as the `kmsuser` CU, and use the cloudhsm\_mgmt\_util and key\_mgmt\_util command line tools\.

**Note**  
While a custom key store is disconnected, all attempts to create KMS keys in the custom key store or to use existing KMS keys in cryptographic operations will fail\. This action can prevent users from storing and accessing sensitive data\.

This topic explains how to [disconnect your AWS CloudHSM key store and log in](#login-kmsuser-1) as `kmsuser`, run the AWS CloudHSM command line tool, and [log out and reconnect your AWS CloudHSM key store](#login-kmsuser-2)\.

**Topics**
+ [How to disconnect and log in](#login-kmsuser-1)
+ [How to log out and reconnect](#login-kmsuser-2)

### How to disconnect and log in<a name="login-kmsuser-1"></a>

Use the following procedure each time to need to log into an associated cluster as the `kmsuser` CU\.

1. Disconnect the AWS CloudHSM key store, if it is not already disconnected\. You can use the AWS KMS console or AWS KMS API\. 

   While your AWS CloudHSM key is connected, AWS KMS is logged in as the `kmsuser`\. This prevents you from logging in as `kmsuser` or changing the `kmsuser` password\.

   For example, this command uses [DisconnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisconnectCustomKeyStore.html) to disconnect an example key store\. Replace the example AWS CloudHSM key store ID with a valid one\.

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

1. Use the [changePswd](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-changePswd.html) command to change the password of the `kmsuser` account to one that you know\. \(AWS KMS rotates the password when you connect your AWS CloudHSM key store\.\) The password must consist of 7\-32 alphanumeric characters\. It is case\-sensitive and cannot contain any special characters\.

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

1. Perform the task, then log out of the command line tool\. If you do not log out, attempts to reconnect your AWS CloudHSM key store will fail\.

   ```
   Command:  logoutHSM
   Cfm3LogoutHSM returned: 0x00 : HSM Return: SUCCESS
   
   Cluster Error Status
   Node id 0 and err state 0x00000000 : HSM Return: SUCCESS
   Node id 1 and err state 0x00000000 : HSM Return: SUCCESS
   ```

1. [Edit the `kmsuser` password setting](update-keystore.md) for the custom key store\. 

   This tells AWS KMS the current password for `kmsuser` in the cluster\. If you omit this step, AWS KMS will not be able to log into the cluster as `kmsuser`, and all attempts to reconnect your custom key store will fail\. You can use the AWS KMS console or the `KeyStorePassword` parameter of the [UpdateCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateCustomKeyStore.html) operation\.

   For example, this command tells AWS KMS that the current password is `tempPassword`\. Replace the example password with the actual one\. 

   ```
   $ aws kms update-custom-key-store --custom-key-store-id cks-1234567890abcdef0 --key-store-password tempPassword
   ```

1. Reconnect the AWS KMS key store to its AWS CloudHSM cluster\. Replace the example AWS CloudHSM key store ID with a valid one\. During the connection process, AWS KMS changes the `kmsuser` password to a value that only it knows\.

   The [ConnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_ConnectCustomKeyStore.html) operation returns quickly, but the connection process can take an extended period of time\. The initial response does not indicate the success of the connection process\.

   ```
   $ aws kms connect-custom-key-store --custom-key-store-id cks-1234567890abcdef0
   ```

1. Use the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation to verify that the AWS CloudHSM key store is connected\. Replace the example AWS CloudHSM key store ID with a valid one\.

   In this example, the connection state field shows that the AWS CloudHSM key store is now connected\.

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