# How AWS services use AWS KMS<a name="service-integration"></a>

Many AWS services use AWS KMS to support encryption of your data\. When an AWS service is integrated with AWS KMS, you can use the customer master keys \(CMKs\) in your account to protect the data that the service receives, stores, or manages for you\. For the complete list of AWS services that are integrated with AWS KMS, see [AWS Service Integration](https://aws.amazon.com/kms/features/#AWS_Service_Integration)\.

The following topics discuss in detail how particular services use AWS KMS, including the CMKs they support, how they manage data keys, the permissions they require, and how to track each service's use of the CMKs in your account\.

**Important**  
[AWS services that are integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration) use symmetric CMKs to encrypt your data\. These services do not support encryption with asymmetric CMKs\. For help determining whether a CMK is symmetric or asymmetric, see [Identifying symmetric and asymmetric CMKs](find-symm-asymm.md)\.