# Managing KMS keys in a custom key store<a name="manage-cmk-keystore"></a>

You can create, view, manage, use, and schedule deletion of the AWS KMS keys in a custom key store\. The procedures that you use are very similar to those you use for KMS keys in AWS KMS\. The only difference is that you specify a custom key store when you create the KMS key\. Then, AWS KMS creates non\-extractable key material for the KMS key in the AWS CloudHSM cluster that is associated with the custom key store\. When you use a KMS key in a custom key store, the [cryptographic operations](use-cmk-keystore.md) are performed in the HSMs in the cluster\.

**Note**  
AWS KMS custom key stores support only symmetric encryption KMS keys\. You cannot create HMAC KMS keys, asymmetric KMS keys, or asymmetric data key pairs in a custom key store\.  
You cannot [import key material](importing-keys.md) into a KMS key in a custom key store\. AWS KMS generates the key material for the KMS key in the AWS CloudHSM cluster\. 

In addition to the procedures discussed in this section, you can do the following with KMS keys in a custom key store:
+ Use key policies, IAM policies, and grants to [authorize access](control-access.md) to the KMS key\.
+ Assign [tags](tagging-keys.md) to the KMS keys and create [aliases](programming-aliases.md) that refer to the KMS keys\.
+ Use the KMS keys for [cryptographic operations](concepts.md#cryptographic-operations), including encrypting, decrypting, re\-encrypting, and generating data keys\.
+ Use the KMS keys with [AWS services that integrate with AWS KMS](service-integration.md) and support customer managed keys
+ Track your KMS key use in [AWS CloudTrail logs](logging-using-cloudtrail.md) and [Amazon CloudWatch monitoring tools](monitoring-overview.md)\.

However, you cannot import key material into a KMS key in a custom key store\.

**Topics**
+ [Creating KMS keys in a custom key store](create-cmk-keystore.md)
+ [Viewing KMS keys in a custom key store](view-cmk-keystore.md)
+ [Using KMS keys in a custom key store](use-cmk-keystore.md)
+ [Finding KMS keys and key material](find-key-material.md)
+ [Scheduling deletion of KMS keys from a custom key store](delete-cmk-keystore.md)