# What is AWS Key Management Service?<a name="overview"></a>

AWS Key Management Service \(AWS KMS\) is a managed service that makes it easy for you to create and control the encryption keys used to encrypt your data\. The master keys that you create in AWS KMS are protected by [FIPS 140\-2 validated cryptographic modules](https://csrc.nist.gov/projects/cryptographic-module-validation-program/Certificate/3139)\.

AWS KMS is integrated with most [other AWS services](https://aws.amazon.com/kms/features/#AWS_Service_Integration) that encrypt your data with encryption keys that you manage\. AWS KMS is also integrated with [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/) to provide encryption key usage logs to help meet your auditing, regulatory and compliance needs\.

You can perform the following management actions on your AWS KMS master keys:
+ Create, describe, and list master keys
+ Enable and disable master keys
+ Create and view grants and access control policies for your master keys
+ Enable and disable automatic rotation of the cryptographic material in a master key
+ Import cryptographic material into an AWS KMS master key
+ Tag your master keys for easier identification, categorizing, and tracking
+ Create, delete, list, and update *aliases*, which are friendly names associated with your master keys
+ Delete master keys to complete the key lifecycle

With AWS KMS you can also perform the following cryptographic functions using master keys:
+ Encrypt, decrypt, and re\-encrypt data
+ Generate data encryption keys that you can export from the service in plaintext or encrypted under a master key that doesn't leave the service
+ Generate random numbers suitable for cryptographic applications

By using AWS KMS, you gain more control over access to data you encrypt\. You can use the key management and cryptographic features directly in your applications or through AWS services that are integrated with AWS KMS\. Whether you are writing applications for AWS or using AWS services, AWS KMS enables you to maintain control over who can use your master keys and gain access to your encrypted data\.

AWS KMS is integrated with AWS CloudTrail, a service that delivers log files to an Amazon S3 bucket that you designate\. By using CloudTrail you can monitor and investigate how and when your master keys have been used and by whom\.

**Learn More**
+ For a more detailed introduction to AWS KMS, see [AWS KMS Concepts](concepts.md)\.
+ For information about the AWS KMS API, see the [AWS Key Management Service API Reference](https://docs.aws.amazon.com/kms/latest/APIReference/)\.
+ For detailed technical information about how AWS KMS uses cryptography and secures master keys, see the [AWS Key Management Service Cryptographic Details](https://d0.awsstatic.com/whitepapers/KMS-Cryptographic-Details.pdf) whitepaper\.
<a name="pricing"></a>
**AWS KMS Pricing**  
As with other AWS products, there are no contracts or minimum commitments for using AWS KMS\. For more information about AWS KMS pricing, see [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/)\.

**Service Level Agreement**  
AWS Key Management Service is backed by a [service level agreement](https://aws.amazon.com/kms/sla/) that defines our service availability policy\.