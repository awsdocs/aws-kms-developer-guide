# How Amazon EMR uses AWS KMS<a name="services-emr"></a>

When you use an [Amazon EMR](https://aws.amazon.com/emr/) cluster, you can configure the cluster to encrypt data *at rest* before saving it to a persistent storage location\. You can encrypt data at rest on the EMR File System \(EMRFS\), on the storage volumes of cluster nodes, or both\. To encrypt data at rest, you can use a customer master key \(CMK\) in AWS KMS\. The following topics explain how an Amazon EMR cluster uses a CMK to encrypt data at rest\.

**Important**  
Amazon EMR supports only [symmetric CMKs](symm-asymm-concepts.md#symmetric-cmks)\. You cannot use an [asymmetric CMK](symm-asymm-concepts.md#asymmetric-cmks) to encrypt data at rest in an Amazon EMR cluster\. For help determining whether a CMK is symmetric or asymmetric, see [Identifying symmetric and asymmetric CMKs](find-symm-asymm.md)\.

Amazon EMR clusters also encrypt data *in transit*, which means the cluster encrypts data before sending it through the network\. You cannot use a CMK to encrypt data in transit\. For more information, see [In\-Transit Data Encryption](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-data-encryption-options.html#emr-encryption-intransit) in the *Amazon EMR Management Guide*\.

For more information about all the encryption options available in Amazon EMR, see [Encryption Options](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-data-encryption-options.html) in the *Amazon EMR Management Guide*\.

**Topics**
+ [Encrypting data on the EMR file system \(EMRFS\)](#emrfs-encryption)
+ [Encrypting data on the storage volumes of cluster nodes](#emr-local-disk-encryption)
+ [Encryption context](#emr-encryption-context)

## Encrypting data on the EMR file system \(EMRFS\)<a name="emrfs-encryption"></a>

Amazon EMR clusters use two distributed files systems:
+ The Hadoop Distributed File System \(HDFS\)\. HDFS encryption does not use a CMK in AWS KMS\.
+ The EMR File System \(EMRFS\)\. EMRFS is an implementation of HDFS that allows Amazon EMR clusters to store data in Amazon Simple Storage Service \(Amazon S3\)\. EMRFS supports four encryption options, two of which use a CMK in AWS KMS\. For more information about all four of the EMRFS encryption options, see [Encryption Options](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-data-encryption-options.html) in the *Amazon EMR Management Guide*\.

The two EMRFS encryption options that use a CMK use the following encryption features offered by Amazon S3:
+ [Server\-Side Encryption with AWS KMS\-Managed Keys \(SSE\-KMS\)](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html)\. With SSE\-KMS, the Amazon EMR cluster sends data to Amazon S3, and then Amazon S3 uses a CMK to encrypt the data before saving it to an S3 bucket\. For more information about how this works, see [Process for encrypting data on EMRFS with SSE\-KMS](#emrfs-encryption-sse-kms)\.
+ [Client\-Side Encryption with AWS KMS\-Managed Keys \(CSE\-KMS\)](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingClientSideEncryption.html)\. With CSE\-KMS, the Amazon EMR cluster uses a CMK to encrypt data before sending it to Amazon S3 for storage\. For more information about how this works, see [Process for encrypting data on EMRFS with CSE\-KMS](#emrfs-encryption-cse-kms)\.

When you configure an Amazon EMR cluster to encrypt data on EMRFS with SSE\-KMS or CSE\-KMS, you choose the CMK in AWS KMS that you want Amazon S3 or the Amazon EMR cluster to use\. With SSE\-KMS, you can choose the AWS managed CMK for Amazon S3 with the alias **aws/s3**, or a symmetric customer managed CMK that you create\. With CSE\-KMS, you must choose a symmetric customer managed CMK that you create\. When you choose a customer managed CMK, you must ensure that your Amazon EMR cluster has permission to use the CMK\. For more information, see [Using AWS KMS customer master keys \(CMKs\) for encryption](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-encryption-enable.html#emr-awskms-keys) in the *Amazon EMR Management Guide*\.

For both SSE\-KMS and CSE\-KMS, the CMK you choose is the master key in an [envelope encryption](concepts.md#enveloping) workflow\. The data is encrypted with a unique data encryption key \(or *data key*\), and this data key is encrypted under the CMK in AWS KMS\. The encrypted data and an encrypted copy of its data key are stored together as a single encrypted object in an S3 bucket\. For more information about how this works, see the following topics\.

**Topics**
+ [Process for encrypting data on EMRFS with SSE\-KMS](#emrfs-encryption-sse-kms)
+ [Process for encrypting data on EMRFS with CSE\-KMS](#emrfs-encryption-cse-kms)

### Process for encrypting data on EMRFS with SSE\-KMS<a name="emrfs-encryption-sse-kms"></a>

When you configure an Amazon EMR cluster to use SSE\-KMS, the encryption process works like this:

1. The cluster sends data to Amazon S3 for storage in an S3 bucket\.

1. Amazon S3 sends a [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) request to AWS KMS, specifying the key ID of the CMK that you chose when you configured the cluster to use SSE\-KMS\. The request includes encryption context; for more information, see [Encryption context](#emr-encryption-context)\.

1. AWS KMS generates a unique data encryption key \(data key\) and then sends two copies of this data key to Amazon S3\. One copy is unencrypted \(plaintext\), and the other copy is encrypted under the CMK\.

1. Amazon S3 uses the plaintext data key to encrypt the data that it received in step 1, and then removes the plaintext data key from memory as soon as possible after use\.

1. Amazon S3 stores the encrypted data and the encrypted copy of the data key together as a single encrypted object in an S3 bucket\.

The decryption process works like this:

1. The cluster requests an encrypted data object from an S3 bucket\.

1. Amazon S3 extracts the encrypted data key from the S3 object, and then sends the encrypted data key to AWS KMS with a [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) request\. The request includes an [encryption context](concepts.md#encrypt_context)\.

1. AWS KMS decrypts the encrypted data key using the same CMK that was used to encrypt it, and then sends the decrypted \(plaintext\) data key to Amazon S3\.

1. Amazon S3 uses the plaintext data key to decrypt the encrypted data, and then removes the plaintext data key from memory as soon as possible after use\.

1. Amazon S3 sends the decrypted data to the cluster\.

### Process for encrypting data on EMRFS with CSE\-KMS<a name="emrfs-encryption-cse-kms"></a>

When you configure an Amazon EMR cluster to use CSE\-KMS, the encryption process works like this:

1. When it's ready to store data in Amazon S3, the cluster sends a [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) request to AWS KMS, specifying the key ID of the CMK that you chose when you configured the cluster to use CSE\-KMS\. The request includes encryption context; for more information, see [Encryption context](#emr-encryption-context)\.

1. AWS KMS generates a unique data encryption key \(data key\) and then sends two copies of this data key to the cluster\. One copy is unencrypted \(plaintext\), and the other copy is encrypted under the CMK\.

1. The cluster uses the plaintext data key to encrypt the data, and then removes the plaintext data key from memory as soon as possible after use\.

1. The cluster combines the encrypted data and the encrypted copy of the data key together into a single encrypted object\.

1. The cluster sends the encrypted object to Amazon S3 for storage\.

The decryption process works like this:

1. The cluster requests the encrypted data object from an S3 bucket\.

1. Amazon S3 sends the encrypted object to the cluster\.

1. The cluster extracts the encrypted data key from the encrypted object, and then sends the encrypted data key to AWS KMS with a [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) request\. The request includes [encryption context](concepts.md#encrypt_context)\.

1. AWS KMS decrypts the encrypted data key using the same CMK that was used to encrypt it, and then sends the decrypted \(plaintext\) data key to the cluster\.

1. The cluster uses the plaintext data key to decrypt the encrypted data, and then removes the plaintext data key from memory as soon as possible after use\.

## Encrypting data on the storage volumes of cluster nodes<a name="emr-local-disk-encryption"></a>

An Amazon EMR cluster is a collection of Amazon Elastic Compute Cloud \(Amazon EC2\) instances\. Each instance in the cluster is called a *cluster node* or *node*\. Each node can have two types of storage volumes: instance store volumes, and Amazon Elastic Block Store \(Amazon EBS\) volumes\. You can configure the cluster to use [Linux Unified Key Setup \(LUKS\)](https://gitlab.com/cryptsetup/cryptsetup/blob/master/README.md) to encrypt both types of storage volumes on the nodes \(but not the boot volume of each node\)\. This is called *local disk encryption*\.

When you enable local disk encryption for a cluster, you can choose to encrypt the LUKS master key with a CMK in AWS KMS\. You must choose a [customer managed CMK](concepts.md#customer-cmk) that you create; you cannot use an [AWS managed CMK](concepts.md#aws-managed-cmk)\. If you choose a customer managed CMK, you must ensure that your Amazon EMR cluster has permission to use the CMK\. For more information, see [Using AWS KMS customer master keys \(CMKs\) for encryption](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-encryption-enable.html#emr-awskms-keys) in the *Amazon EMR Management Guide*\.

When you enable local disk encryption using a CMK, the encryption process works like this:

1. When each cluster node launches, it sends a [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) request to AWS KMS, specifying the key ID of the CMK that you chose when you enabled local disk encryption for the cluster\.

1. AWS KMS generates a unique data encryption key \(data key\) and then sends two copies of this data key to the node\. One copy is unencrypted \(plaintext\), and the other copy is encrypted under the CMK\.

1. The node uses a base64\-encoded version of the plaintext data key as the password that protects the LUKS master key\. The node saves the encrypted copy of the data key on its boot volume\.

1. If the node reboots, the rebooted node sends the encrypted data key to AWS KMS with a [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) request\.

1. AWS KMS decrypts the encrypted data key using the same CMK that was used to encrypt it, and then sends the decrypted \(plaintext\) data key to the node\.

1. The node uses the base64\-encoded version of the plaintext data key as the password to unlock the LUKS master key\.

## Encryption context<a name="emr-encryption-context"></a>

Each AWS service that is integrated with AWS KMS can specify an [encryption context](concepts.md#encrypt_context) when it uses AWS KMS to generate data keys or to encrypt or decrypt data\. Encryption context is additional authenticated information that AWS KMS uses to check for data integrity\. When a service specifies encryption context for an encryption operation, it must specify the same encryption context for the corresponding decryption operation or decryption will fail\. Encryption context is also written to AWS CloudTrail log files, which can help you understand why a given CMK was used\. 

The following section explain the encryption context that is used in each Amazon EMR encryption scenario that uses a CMK\.

### Encryption context for EMRFS encryption with SSE\-KMS<a name="emr-encryption-context-sse-kms"></a>

With SSE\-KMS, the Amazon EMR cluster sends data to Amazon S3, and then Amazon S3 uses a CMK to encrypt the data before saving it to an S3 bucket\. In this case, Amazon S3 uses the Amazon Resource Name \(ARN\) of the S3 object as encryption context with each [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) request that it sends to AWS KMS\. The following example shows a JSON representation of the encryption context that Amazon S3 uses\.

```
{ "aws:s3:arn" : "arn:aws:s3:::S3_bucket_name/S3_object_key" }
```

### Encryption context for EMRFS encryption with CSE\-KMS<a name="emr-encryption-context-cse-kms"></a>

With CSE\-KMS, the Amazon EMR cluster uses a CMK to encrypt data before sending it to Amazon S3 for storage\. In this case, the cluster uses the Amazon Resource Name \(ARN\) of the CMK as encryption context with each [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) request that it sends to AWS KMS\. The following example shows a JSON representation of the encryption context that the cluster uses\.

```
{ "kms_cmk_id" : "arn:aws:kms:us-east-2:111122223333:key/0987ab65-43cd-21ef-09ab-87654321cdef" }
```

### Encryption context for local disk encryption with LUKS<a name="emr-encryption-context-luks"></a>

When an Amazon EMR cluster uses local disk encryption with LUKS, the cluster nodes do not specify encryption context with the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) requests that they send to AWS KMS\.