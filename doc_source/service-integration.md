# How AWS Services use AWS KMS<a name="service-integration"></a>

Many AWS services use AWS KMS to support encryption of your data\. When an AWS service is integrated with AWS KMS, you can use the customer master keys \(CMKs\) in your account to protect the data that the service receives, stores, or manages for you\. For the complete list of AWS services that are integrated with AWS KMS, see [AWS Service Integration](https://aws.amazon.com/kms/details/#integration)\.

The following topics discuss in detail how particular services use AWS KMS, including the CMKs they support, how they manage data keys, the permissions they require, and how to track each service's use of the CMKs in your account\.

The first topic explains [envelope encryption](workflow.md) and the methods that many integrated services use to [encrypt](workflow.md#encrypting_user_data) and [decrypt](workflow.md#decrypting_user_data) your data transparently\.

**Topics**
+ [How Envelope Encryption Works with Supported AWS Services](workflow.md)
+ [AWS CloudTrail](services-cloudtrail.md)
+ [Amazon Elastic Block Store \(Amazon EBS\)](services-ebs.md)
+ [Amazon Elastic Transcoder](services-et.md)
+ [Amazon EMR](services-emr.md)
+ [Amazon Redshift](services-redshift.md)
+ [Amazon Relational Database Service \(Amazon RDS\)](services-rds.md)
+ [Amazon Simple Email Service \(Amazon SES\)](services-ses.md)
+ [Amazon Simple Storage Service \(Amazon S3\)](services-s3.md)
+ [AWS Systems Manager Parameter Store](services-parameter-store.md)
+ [Amazon WorkMail](services-wm.md)
+ [Amazon WorkSpaces](services-workspaces.md)