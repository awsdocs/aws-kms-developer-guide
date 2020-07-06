# How Amazon Redshift uses AWS KMS<a name="services-redshift"></a>

This topic discusses how Amazon Redshift uses AWS KMS to encrypt data\.

**Topics**
+ [Amazon Redshift encryption](#rs-encryption)
+ [Encryption context](#rs-encryptioncontext)

## Amazon Redshift encryption<a name="rs-encryption"></a>

An Amazon Redshift data warehouse is a collection of computing resources called nodes, which are organized into a group called a cluster\. Each cluster runs an Amazon Redshift engine and contains one or more databases\. 

Amazon Redshift uses a four\-tier, key\-based architecture for encryption\. The architecture consists of data encryption keys, a database key, a cluster key, and a master key\. 

Data encryption keys encrypt data blocks in the cluster\. Each data block is assigned a randomly\-generated AES\-256 key\. These keys are encrypted by using the database key for the cluster\. 

The database key encrypts data encryption keys in the cluster\. The database key is a randomly\-generated AES\-256 key\. It is stored on disk in a separate network from the Amazon Redshift cluster and passed to the cluster across a secure channel\. 

The cluster key encrypts the database key for the Amazon Redshift cluster\. You can use AWS KMS, AWS CloudHSM, or an external hardware security module \(HSM\) to manage the cluster key\. See the [ Amazon Redshift Database Encryption ](https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-db-encryption.html) documentation for more details\. 

The master key encrypts the cluster key\. You can use an AWS KMS [customer master key](concepts.md#master_keys) \(CMK\) as the master key for Amazon Redshift\. You can request encryption by checking the appropriate box in the Amazon Redshift console\. You can specify a [customer managed CMK](concepts.md#customer-cmk) to use by choosing one from the list that appears below the encryption box\. If you do not specify a customer managed CMK, Amazon Redshift uses the [AWS managed CMK](concepts.md#aws-managed-cmk) for Amazon Redshift under your account\. 

**Important**  
Amazon Redshift supports only symmetric CMKs\. You cannot use an asymmetric CMK as the master key in an Amazon Redshift encryption workflow\. For help determining whether a CMK is symmetric or asymmetric, see [Identifying symmetric and asymmetric CMKs](find-symm-asymm.md)\.

## Encryption context<a name="rs-encryptioncontext"></a>

Each service that is integrated with AWS KMS specifies an [encryption context](concepts.md#encrypt_context) when requesting data keys, encrypting, and decrypting\. The encryption context is [additional authenticated data](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#term-aad) \(AAD\) that AWS KMS uses to check for data integrity\. That is, when an encryption context is specified for an encryption operation, the service also specifies it for the decryption operation or decryption will not succeed\. Amazon Redshift uses the cluster ID and the creation time for the encryption context\. In the `requestParameters` field of a CloudTrail log file, the encryption context will look similar to this\. 

```
"encryptionContext": {
    "aws:redshift:arn": "arn:aws:redshift:region:account_ID:cluster:cluster_name",
    "aws:redshift:createtime": "20150206T1832Z"
},
```

 You can search on the cluster name in your CloudTrail logs to understand what operations were performed by using a customer master key \(CMK\)\. The operations include cluster encryption, cluster decryption, and generating data keys\. 