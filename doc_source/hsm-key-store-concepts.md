# AWS CloudHSM key store concepts<a name="hsm-key-store-concepts"></a>

This topic explains some of the concepts used in AWS CloudHSM key stores\.

## AWS CloudHSM key store<a name="concept-hsm-key-store"></a>

An *AWS CloudHSM key store* is a [custom key store](custom-key-store-overview.md) associated with an AWS CloudHSM cluster that you own and manage\. AWS CloudHSM clusters are backed by hardware security modules \(HSMs\) certified at [FIPS 140\-2 Level 3](https://docs.aws.amazon.com/cloudhsm/latest/userguide/compliance.html)\.

When you create a KMS key in your AWS CloudHSM key store, AWS KMS generates a 256\-bit, persistent, non\-exportable Advanced Encryption Standard \(AES\) symmetric key in the associated AWS CloudHSM cluster\. This key material never leaves your HSMs unencrypted\. When you use a KMS key in an AWS CloudHSM key store, the cryptographic operations are performed in the HSMs in the cluster\. 

AWS CloudHSM key stores combine the convenient and comprehensive key management interface of AWS KMS with the additional controls provided by an AWS CloudHSM cluster in your AWS account\. This integrated feature lets you create, manage, and use KMS keys in AWS KMS while maintaining full control of the HSMs that store their key material, including managing clusters, HSMs, and backups\. You can use the AWS KMS console and APIs to manage the AWS CloudHSM key store and its KMS keys\. You can also use the AWS CloudHSM console, APIs, client software, and associated software libraries to manage the associated cluster\.

You can [view and manage](manage-keystore.md) your AWS CloudHSM key store, [edit its properties](update-keystore.md), and [connect and disconnect it](disconnect-keystore.md) from its associated AWS CloudHSM cluster\. If you need to [delete an AWS CloudHSM key store](delete-keystore.md#delete-keystore-console), you must first delete the KMS keys in the AWS CloudHSM key store by scheduling their deletion and waiting until the grace period expires\. Deleting the AWS CloudHSM key store removes the resource from AWS KMS, but it does not affect your AWS CloudHSM cluster\.

## AWS CloudHSM cluster<a name="concept-cluster"></a>

Every AWS CloudHSM key store is associated with one AWS CloudHSM cluster\. When you create an AWS KMS key in your AWS CloudHSM key store, AWS KMS creates its key material in the associated cluster\. When you use a KMS key in your AWS CloudHSM key store, the cryptographic operation is performed in the associated cluster\.

Each AWS CloudHSM cluster can be associated with only one AWS CloudHSM key store\. The cluster that you choose cannot be associated with another AWS CloudHSM key store or share a backup history with a cluster that is associated with another AWS CloudHSM key store\. The cluster must be initialized and active, and it must be in the same AWS account and Region as the AWS CloudHSM key store\. You can create a new cluster or use an existing one\. AWS KMS does not need exclusive use of the cluster\. To create KMS keys in the AWS CloudHSM key store, its associated cluster it must contain at least two active HSMs\. All other operations require only one HSM\.

You specify the AWS CloudHSM cluster when you create the AWS CloudHSM key store, and you cannot change it\. However, you can substitute any cluster that shares a backup history with the original cluster\. This lets you delete the cluster, if necessary, and replace it with a cluster created from one of its backups\. You retain full control of the associated AWS CloudHSM cluster so you can manage users and keys, create and delete HSMs, and use and manage backups\. 

When you are ready to use your AWS CloudHSM key store, you connect it to its associated AWS CloudHSM cluster\. You can [connect and disconnect your custom key store](disconnect-keystore.md) at any time\. When a custom key store is connected, you can create and use its KMS keys\. When it is disconnected, you can view and manage the AWS CloudHSM key store and its KMS keys\. But you cannot create new KMS keys or use the KMS keys in the AWS CloudHSM key store for cryptographic operations\.

## `kmsuser` Crypto user<a name="concept-kmsuser"></a>

To create and manage key material in the associated AWS CloudHSM cluster on your behalf, AWS KMS uses a dedicated AWS CloudHSM [crypto user](https://docs.aws.amazon.com/cloudhsm/latest/userguide/hsm-users.html#crypto-user) \(CU\) in the cluster named `kmsuser`\. The `kmsuser` CU is a standard CU account that is automatically synchronized to all HSMs in the cluster and is saved in cluster backups\. 

Before you create your AWS CloudHSM key store, you [create a `kmsuser` CU account](create-keystore.md#before-keystore) in your AWS CloudHSM cluster using the [createUser](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-createUser.html) command in cloudhsm\_mgmt\_util\. Then when you [create the AWS CloudHSM key store](create-keystore.md), you provide the `kmsuser` account password to AWS KMS\. When you [connect the custom key store](disconnect-keystore.md), AWS KMS logs into the cluster as the `kmsuser` CU and rotates its password\. AWS KMS encrypts your `kmsuser` password before it stores it securely\. When the password is rotated, the new password is encrypted and stored in the same way\.

AWS KMS remains logged in as `kmsuser` as long as the AWS CloudHSM key store is connected\. You should not use this CU account for other purposes\. However, you retain ultimate control of the `kmsuser` CU account\. At any time, you can [find the key handles](find-key-material.md#find-handle-for-cmk-id) of keys that `kmsuser` owns\. If necessary, you can [disconnect the custom key store](disconnect-keystore.md), change the `kmsuser` password, [log into the cluster as `kmsuser`](fix-keystore.md#fix-login-as-kmsuser), and view and manage the keys that `kmsuser` owns\.

For instructions on creating your `kmsuser` CU account, see [Create the `kmsuser` Crypto User](create-keystore.md#before-keystore)\.

## KMS keys in an AWS CloudHSM key store<a name="concept-cmk-key-store"></a>

You can use the AWS KMS or AWS KMS API to create a [AWS KMS keys](concepts.md#kms_keys) in an AWS CloudHSM key store\. You use the same technique that you would use on any KMS key\. The only difference is that you must identify the AWS CloudHSM key store and specify that the origin of the key material is the AWS CloudHSM cluster\. 

When you [create a KMS key in an AWS CloudHSM key store](create-cmk-keystore.md), AWS KMS creates the KMS key in AWS KMS and it generates a 256\-bit, persistent, non\-exportable Advanced Encryption Standard \(AES\) symmetric key material in its associated cluster\. When you use the AWS KMS key in a cryptographic operation, the operation is performed in the AWS CloudHSM cluster using the cluster\-based AES key\. Although AWS CloudHSM supports symmetric and asymmetric keys of different types, AWS CloudHSM key stores support only AES symmetric encryption keys\.

You can view the KMS keys in an AWS CloudHSM key store in the AWS KMS console, and use the console options to display the custom key store ID\. You can also use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation to find the AWS CloudHSM key store ID and AWS CloudHSM cluster ID\.

The KMS keys in an AWS CloudHSM key store work just like any KMS keys in AWS KMS\. Authorized users need the same permissions to use and manage the KMS keys\. You use the same console procedures and API operations to view and manage the KMS keys in an AWS CloudHSM key store\. These include enabling and disabling KMS keys, creating and using tags and aliases, and setting and changing IAM and key policies\. You can use the KMS keys in an AWS CloudHSM key store for cryptographic operations, and use them with [integrated AWS services](service-integration.md) that support the use of customer managed keys However, you cannot enable [automatic key rotation](rotate-keys.md) or [import key material](importing-keys.md) into a KMS key in an AWS CloudHSM key store\. 

You also use the same process to [schedule deletion](delete-cmk-keystore.md) of a KMS key in an AWS CloudHSM key store\. After the waiting period expires, AWS KMS deletes the KMS key from KMS\. Then it makes a best effort to delete the key material for the KMS key from the associated AWS CloudHSM cluster\. However, you might need to manually [delete the orphaned key material](fix-keystore.md#fix-keystore-orphaned-key) from the cluster and its backups\.