# Comparing symmetric and asymmetric KMS keys<a name="symm-asymm-compare"></a>

You can create and manage symmetric and asymmetric KMS keys by using the AWS KMS console and the AWS KMS API\. However, AWS KMS supports different features for KMS keys of different types\. 

For example, you can only use symmetric KMS keys to [generate symmetric data keys](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [asymmetric data key pairs](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairs.html)\. Also, [importing key material](importing-keys.md) and [automatic key rotation](rotate-keys.md) are supported only for symmetric KMS keys, and you can create only symmetric KMS keys in a [custom key store](custom-key-store-overview.md)\. 

The following table lists the AWS KMS operations that you can use to create and manage KMS keys of each type\. If you use the operation on a KMS key that doesn't not support it, the operation fails\.


**AWS KMS operations with symmetric and asymmetric KMS keys**  
<a name="symm-asymm-table"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/symm-asymm-compare.html)

\[1\] `GenerateDataKeyPair` and `GenerateDataKeyPairWithoutPlaintext` generate an asymmetric data key pair that is protected by a symmetric KMS key\.