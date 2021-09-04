# How Amazon Relational Database Service \(Amazon RDS\) uses AWS KMS<a name="services-rds"></a>

You can use the [Amazon Relational Database Service \(Amazon RDS\)](https://aws.amazon.com/rds/) to set up, operate, and scale a relational database in the cloud\. Optionally, you can choose to encrypt the data stored on your Amazon RDS [DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.DBInstance.html) under a [AWS KMS key](concepts.md#kms_keys) \(KMS key\) in AWS KMS\. To learn how to encrypt your Amazon RDS resources under an KMS key, see [Encrypting Amazon RDS Resources](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.html) in the *Amazon RDS User Guide*\.

**Important**  
Amazon RDS supports only [symmetric KMS keys](symm-asymm-concepts.md#symmetric-cmks)\. You cannot use an [asymmetric KMS key](symm-asymm-concepts.md#asymmetric-cmks) to encrypt data in an Amazon RDS database\. For help determining whether a KMS key is symmetric or asymmetric, see [Identifying symmetric and asymmetric KMS keys](find-symm-asymm.md)\.

Amazon RDS builds on [Amazon Elastic Block Store \(Amazon EBS\) encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html) to provide full disk encryption for database volumes\. For more information about how Amazon EBS uses AWS KMS to encrypt volumes, see [How Amazon Elastic Block Store \(Amazon EBS\) uses AWS KMS](services-ebs.md)\.

When you create an encrypted DB instance with Amazon RDS, Amazon RDS creates an encrypted EBS volume on your behalf to store the database\. Data stored at rest on the volume, database snapshots, automated backups, and read replicas are all encrypted under the KMS key that you specified when you created the DB instance\. 

## Amazon RDS encryption context<a name="rds-encryptioncontext"></a>

When Amazon RDS uses your KMS key, or when Amazon EBS uses the KMS key on behalf of Amazon RDS, the service specifies an [encryption context](concepts.md#encrypt_context)\. The encryption context is [additional authenticated data](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#term-aad) \(AAD\) that AWS KMS uses to ensure data integrity\. When an encryption context is specified for an encryption operation, the service must specify the same encryption context for the decryption operation\. Otherwise, decryption fails\. The encryption context is also written to your [AWS CloudTrail](https://aws.amazon.com/cloudtrail/) logs to help you understand why a given KMS key was used\. Your CloudTrail logs might contain many entries describing the use of a KMS key, but the encryption context in each log entry can help you determine the reason for that particular use\.

At minimum, Amazon RDS always uses the DB instance ID for the encryption context, as in the following JSON\-formatted example:

```
{ "aws:rds:db-id": "db-CQYSMDPBRZ7BPMH7Y3RTDG5QY" }
```

This encryption context can help you identify the DB instance for which your KMS key was used\.

When your KMS key is used for a specific DB instance and a specific EBS volume, both the DB instance ID and the EBS volume ID are used for the encryption context, as in the following JSON\-formatted example:

```
{
  "aws:rds:db-id": "db-BRG7VYS3SVIFQW7234EJQOM5RQ",
  "aws:ebs:id": "vol-ad8c6542"
}
```