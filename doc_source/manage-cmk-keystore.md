# Managing CMKs in a Custom Key Store<a name="manage-cmk-keystore"></a>

You can create, view, manage, use, and schedule deletion of the customer master keys \(CMKs\) in a custom key store\. The procedures that you use are very similar to those that you use for CMKs in AWS KMS\. The only difference is that you specify a custom key store when you create the CMK\. Then, AWS KMS creates non\-extractable key material for the CMK in the AWS CloudHSM cluster that is associated with the custom key store\. When you use a CMK in a custom key store, the cryptographic operations are performed in the HSMs in the cluster\.

In addition to the procedures discussed in this section, you can do the following with CMKs in a custom key store:
+ Use key policies, IAM policies, and grants to [authorize access](control-access.md) to the CMK\.
+ Assign [tags](tagging-keys.md) to the CMKs and create [aliases](programming-aliases.md) that refer to the CMKs\.
+ Use the CMKs for cryptographic operations, including encrypting, decrypting, re\-encrypting, and generating data keys\. For details, see [AWS Key Management Service API Reference](https://docs.aws.amazon.com/kms/latest/APIReference/)\. 
+ Use the CMKs with [AWS services that integrate with AWS KMS](service-integration.md) and support customer managed CMKs\.
+ Track your CMK use in [AWS CloudTrail logs](logging-using-cloudtrail.md) and [Amazon CloudWatch monitoring tools](monitoring-overview.md)\.

However, you cannot import key material into a CMK in a custom key store\.

**Topics**
+ [Creating CMKs in a Custom Key Store](create-cmk-keystore.md)
+ [Viewing CMKs in a Custom Key Store](view-cmk-keystore.md)
+ [Using CMKs in a Custom Key Store](use-cmk-keystore.md)
+ [Finding CMKs and Key Material](find-key-material.md)
+ [Scheduling Deletion of CMKs from a Custom Key Store](delete-cmk-keystore.md)