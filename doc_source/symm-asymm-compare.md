# Comparing symmetric and asymmetric CMKs<a name="symm-asymm-compare"></a>

You can create and manage symmetric and asymmetric CMKs by using the AWS KMS console and the AWS KMS API\. However, AWS KMS supports different features for CMKs of different types\. 

For example, you can only use symmetric CMKs to [generate symmetric data keys](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [asymmetric data key pairs](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairs.html)\. Also, [importing key material](importing-keys.md) and [automatic key rotation](rotate-keys.md) are supported only for symmetric CMKs, and you can create only symmetric CMKs in a [custom key store](custom-key-store-overview.md)\. 

The following table lists the AWS KMS operations that you can use to create and manage CMKs of each type\. If you use the operation on a CMK that doesn't not support it, the operation fails\.


**AWS KMS operations with symmetric and asymmetric CMKs**  
<a name="symm-asymm-table"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/symm-asymm-compare.html)

\[1\] `GenerateDataKeyPair` and `GenerateDataKeyPairWithoutPlaintext` generate an asymmetric data key pair that is protected by a symmetric CMK\.