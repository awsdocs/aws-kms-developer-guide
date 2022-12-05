# Using KMS keys in an external key store<a name="use-xks-key"></a>

After you [create a symmetric encryption KMS key in an external key store](create-xks-keystore.md), you can use it for the following cryptographic operations:
+ [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html)
+ [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)
+ [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)
+ [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html)
+ [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)

The symmetric encryption operations that generate asymmetric data key pairs, [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html) and [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html), are *not* supported in custom key stores\.

An [encryption context](concepts.md#encrypt_context) is supported for all cryptographic operations with KMS keys in an external key store\. As always, using an encryption context is a security best practice that AWS KMS recommends\.

When you use your KMS key in a request, identify the KMS key by its [key ID, key ARN, alias, or alias ARN](concepts.md#key-id)\. You do not need to specify the external key store\. The response includes the same fields that are returned for any symmetric encryption KMS key\. However, when you use a KMS key in an external key store, encryption and decryption operations are performed by your external key manager using the external key that is associated with the KMS key\.

To ensure that ciphertext encrypted by a KMS key in an external key store is at least as secure as any ciphertext encrypted by a standard KMS key, AWS KMS uses [double encryption](keystore-external.md#concept-double-encryption)\. Data is first encrypted in AWS KMS using AWS KMS key material\. Then it is encrypted by your external key manager using the external key for the KMS key\. To decrypt double\-encrypted ciphertext, the ciphertext is first decrypted by your external key manager using the external key for the KMS key\. Then it is decrypted in AWS KMS using the AWS KMS key material for the KMS key\.

To make this possible, the following conditions are required\.
+ The [key state](key-state.md) of the KMS key must be `Enabled`\. To find the key state, see the **Status** field for customer managed keys the [AWS KMS console](view-cmk-keystore.md) or the `KeyState` field in the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) response\.
+ The external key store that hosts the KMS key must be connected to its [external key store proxy](keystore-external.md#concept-xks-proxy), that is, the [connection state](xks-connect-disconnect.md#xks-connection-state) of the external key store must be `CONNECTED`\. 

  You can view the connection state on the **External key stores** page in the AWS KMS console or in the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) response\. The connection state of the external key store is also displayed on the detail page for the KMS key in the AWS KMS console\. On the detail page, choose the **Cryptographic configuration** tab and see the **Connection state** field in the **Custom key store** section\.

  If the connection state is `DISCONNECTED`, you must first connect it\. If the connection state is `FAILED`, you must resolve the problem, disconnect the external key store, and then connect it\. For instructions, see [Connecting and disconnecting an external key store](xks-connect-disconnect.md)\.\.
+ The external key store proxy must be able to find the external key\. 
+ The external key must be enabled and it must perform encryption and decryption\. 

  The status of the external key is independent of and not affected by changes in the [key state](key-state.md) of the KMS key, including enabling and disabling the KMS key\. Similarly, disabling or deleting the external key doesn't change the key state of the KMS key, but cryptographic operations using the associated KMS key will fail\.

If these conditions are not met, the cryptographic operation fails, and AWS KMS returns a `KMSInvalidStateException` exception\. You might need to [reconnect the external key store](xks-connect-disconnect.md) or use your external key manager tools to reconfigure or repair your external key\. For additional help, see [Troubleshooting external key stores](xks-troubleshooting.md)\.

When using the KMS keys in an external key store, be aware that the KMS keys in each external key store share a [custom key store request quota](requests-per-second.md#rps-key-stores) for cryptographic operations\. If you exceed the quota, AWS KMS returns a `ThrottlingException`\. For details about the custom key store request quota, see [Custom key store request quota](requests-per-second.md#rps-key-stores)\.