# What is AWS Key Management Service?<a name="overview"></a>

AWS Key Management Service \(AWS KMS\) is a managed service that makes it easy for you to create and control the encryption keys used to encrypt your data\. AWS KMS is integrated with other AWS services including Amazon Elastic Block Store \(Amazon EBS\), Amazon Simple Storage Service \(Amazon S3\), Amazon Redshift, Amazon Elastic Transcoder, Amazon WorkMail, Amazon Relational Database Service \(Amazon RDS\), and others to make it simple to encrypt your data with encryption keys that you manage\. AWS KMS is also integrated with AWS CloudTrail to provide you with key usage logs to help meet your auditing, regulatory and compliance needs\.

AWS KMS lets you create master keys that can never be exported from the service and which can be used to encrypt and decrypt data based on policies you define\.

You can perform the following management actions on master keys by using AWS KMS:

+ Create, describe, and list master keys

+ Enable and disable master keys

+ Set and retrieve master key usage policies \(access control\)

+ Create, delete, list, and update *aliases*, which are friendly names that point to your master keys

+ Delete master keys to complete the key lifecycle

With AWS KMS you can also perform the following cryptographic functions using master keys:

+ Encrypt, decrypt, and re\-encrypt data

+ Generate data encryption keys that you can export from the service in plaintext or encrypted under a master key that doesn't leave the service

+ Generate random numbers suitable for cryptographic applications

By using AWS KMS, you gain more control over access to data you encrypt\. You can use the key management and cryptographic features directly in your applications or through AWS services that are integrated with AWS KMS\. Whether you are writing applications for AWS or using AWS services, AWS KMS enables you to maintain control over who can use your master keys and gain access to your encrypted data\.

AWS KMS is integrated with AWS CloudTrail, a service that delivers log files to an Amazon S3 bucket that you designate\. By using CloudTrail you can monitor and investigate how and when your master keys have been used and by whom\.

For a more detailed introduction to AWS KMS, see AWS KMS Concepts\.

To learn more about how AWS KMS uses cryptography and secures master keys, see the [AWS Key Management Service Cryptographic Details](https://d0.awsstatic.com/whitepapers/KMS-Cryptographic-Details.pdf) whitepaper\.

**AWS KMS Pricing**  
As with other AWS products, there are no contracts or minimum commitments for using AWS KMS\. For more information about AWS KMS pricing, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/)\.