# Managing KMS keys in a CloudHSM key store<a name="manage-cmk-keystore"></a>

You can create, view, manage, use, and schedule deletion of the AWS KMS keys in an AWS CloudHSM key store\. The procedures that you use are very similar to those you use for other KMS keys\. The only difference is that you specify an AWS CloudHSM key store when you create the KMS key\. Then, AWS KMS creates non\-extractable key material for the KMS key in the AWS CloudHSM cluster that is associated with the AWS CloudHSM key store\. When you use a KMS key in an AWS CloudHSM key store, the [cryptographic operations](use-cmk-keystore.md) are performed in the HSMs in the cluster\.

**Supported features**

In addition to the procedures discussed in this section, you can do the following with KMS keys in an AWS CloudHSM key store:
+ Use key policies, IAM policies, and grants to [authorize access](control-access.md) to the KMS keys\.
+ [Enable and disable](enabling-keys.md) the KMS keys\. 
+ Assign [tags](tagging-keys.md) and create [aliases](programming-aliases.md), and use attribute\-based access control \(ABAC\) to authorize access to the KMS keys\.
+ Use the KMS keys for [cryptographic operations](concepts.md#cryptographic-operations), including encrypting, decrypting, re\-encrypting, and generating data keys\.
+ Use the KMS keys with [AWS services that integrate with AWS KMS](service-integration.md) and support customer managed keys\.
+ Track use of your KMS keys in [AWS CloudTrail logs](logging-using-cloudtrail.md) and [Amazon CloudWatch monitoring tools](monitoring-overview.md)\.

**Unsupported features**
+ AWS CloudHSM key stores support only symmetric encryption KMS keys\. You cannot create HMAC KMS keys, asymmetric KMS keys, or asymmetric data key pairs in an AWS CloudHSM key store\.
+ You cannot [import key material](importing-keys.md) into a KMS key in an AWS CloudHSM key store\. AWS KMS generates the key material for the KMS key in the AWS CloudHSM cluster\.
+ You cannot enable or disable [automatic rotation](rotate-keys.md) of the key material for a KMS key in an AWS CloudHSM key store\.

**Topics**
+ [Creating KMS keys in an AWS CloudHSM key store](create-cmk-keystore.md)
+ [Viewing KMS keys in an AWS CloudHSM key store](view-cmk-keystore.md)
+ [Using KMS keys in an AWS CloudHSM key store](use-cmk-keystore.md)
+ [Finding KMS keys and key material](find-key-material.md)
+ [Scheduling deletion of KMS keys from an AWS CloudHSM key store](delete-cmk-keystore.md)