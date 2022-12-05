# Managing KMS keys in an external key store<a name="keystore-external-key-manage"></a>

To create, view, manage, use, and schedule deletion of the KMS keys in an external key store, you use procedures that are very similar to those you use for other KMS keys\. However, when you create a KMS key in an external key store, you specify an [external key store](keystore-external.md#concept-external-key-store) and an [external key](keystore-external.md#concept-external-key)\. When you use a KMS key in an external key store, [encryption and decryption operations](keystore-external.md#xks-how-it-works) are performed by your external key manager using the specified external key\. 

AWS KMS cannot create, view, update, or delete any cryptographic keys in your external key manager\. AWS KMS never directly accesses your external key manager or any external key\. All requests for cryptographic operations are mediated by your [external key store proxy](keystore-external.md#concept-xks-proxy)\. To use a KMS key in an external key store, the external key store that hosts the KMS key must be [connected](xks-connect-disconnect.md) to its external key store proxy\.

**Supported features **

In addition to the procedures discussed in this section, you can do the following with KMS keys in an external key store: 
+ Use [key policies](key-policies.md), [IAM policies](iam-policies.md), and [grants](grants.md) to control access to the KMS keys\.
+ [Enable and disable](enabling-keys.md) the KMS keys\. These actions do not affect the external key in your external key manager\.
+ Assign [tags](tagging-keys.md) and create [aliases](kms-alias.md), and use [attribute\-based access control](abac.md) \(ABAC\) to authorize access to the KMS keys\.
+ Use the KMS keys with [AWS services that integrate with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration) and support [customer managed keys](concepts.md#customer-cmk)\.

**Unsupported features** 
+ External key stores support only [symmetric encryption KMS keys](concepts.md#symmetric-cmks)\. You cannot create HMAC KMS keys or asymmetric KMS keys in an external key store\.
+ [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html) and [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html) are not supported on KMS keys in an external key store\.
+ You cannot use an [AWS CloudFormation template](creating-resources-with-cloudformation.md) to create an external key store or a KMS key in an external key store\.
+ [Multi\-Region keys](multi-region-keys-overview.md) are not supported in an external key store\.
+ KMS keys with [imported key material](importing-keys.md) are not supported in an external key store\.
+ [Automatic key rotation](rotate-keys.md) is not supported for KMS keys in an external key store\.

**Topics**
+ [Creating KMS keys in an external key store](create-xks-keys.md)
+ [Viewing KMS keys in an external key store](view-xks-key.md)
+ [Using KMS keys in an external key store](use-xks-key.md)
+ [Scheduling deletion of KMS keys from an external key store](delete-xks-key.md)