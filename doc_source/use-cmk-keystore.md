# Using KMS keys in an AWS CloudHSM key store<a name="use-cmk-keystore"></a>

After you [create a symmetric encryption KMS key in an AWS CloudHSM key store](create-cmk-keystore.md), you can use it for the following cryptographic operations:
+ [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html)
+ [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)
+ [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)
+ [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html)
+ [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)

The operations that generate asymmetric data key pairs, [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html) and [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html), are *not* supported in custom key stores\.

When you use your KMS key in a request, identify the KMS key by its ID or alias; you do not need to specify the AWS CloudHSM key store or AWS CloudHSM cluster\. The response includes the same fields that are returned for any symmetric encryption KMS key\.

However, when you use a KMS key in an AWS CloudHSM key store, the cryptographic operation is performed entirely within the AWS CloudHSM cluster that is associated with the AWS CloudHSM key store\. The operation uses the key material in the cluster that is associated with the KMS key that you chose\.

To make this possible, the following conditions are required\.
+ The [key state](key-state.md) of the KMS key must be `Enabled`\. To find the key state, use the **Status** field in the [AWS KMS console](view-cmk-keystore.md) or the `KeyState` field in the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) response\.
+ The AWS CloudHSM key store must be connected to its AWS CloudHSM cluster\. Its **Status** in the [AWS KMS console](view-keystore.md) or `ConnectionState` in the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) response must be `CONNECTED`\.
+ The AWS CloudHSM cluster that is associated with the custom key store must contain at least one active HSM\. To find the number of active HSMs in the cluster, use the [AWS KMS console](view-keystore.md), the AWS CloudHSM console, or the [DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html) operation\.
+ The AWS CloudHSM cluster must contain the key material for the KMS key\. If the key material was deleted from the cluster, or an HSM was created from a backup that did not include the key material, the cryptographic operation will fail\.

If these conditions are not met, the cryptographic operation fails, and AWS KMS returns a `KMSInvalidStateException` exception\. Typically, you just need to [reconnect the AWS CloudHSM key store](disconnect-keystore.md)\. For additional help, see [How to fix a failing KMS key](fix-keystore.md#fix-cmk-failed)\.

When using the KMS keys in an AWS CloudHSM key store, be aware that the KMS keys in each AWS CloudHSM key store share a [custom key store request quota](requests-per-second.md#rps-key-stores) for cryptographic operations\. If you exceed the quota, AWS KMS returns a `ThrottlingException`\. If the AWS CloudHSM cluster that is associated with the AWS CloudHSM key store is processing numerous commands, including those unrelated to the AWS CloudHSM key store, you might get a `ThrottlingException` at an even lower rate\. If you get a `ThrottlingException` for any request, lower your request rate and try the commands again\. For details about the custom key store request quota, see [Custom key store request quota](requests-per-second.md#rps-key-stores)\.