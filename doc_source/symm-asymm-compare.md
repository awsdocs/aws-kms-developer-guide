# Key type reference<a name="symm-asymm-compare"></a>

AWS KMS supports different features for different types of KMS keys\. For example, you can only use symmetric encryption KMS keys to [generate symmetric data keys](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [asymmetric data key pairs](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairs.html)\. Also, [importing key material](importing-keys.md) and [automatic key rotation](rotate-keys.md) are supported only for symmetric encryption KMS keys, and you can create only symmetric encryption KMS keys in a [custom key store](custom-key-store-overview.md)\. 

In addition to the information in this table, KMS keys can be used in the following AWS KMS special features\.
+ [Multi\-Region keys](multi-region-keys-overview.md): 
  + All API operations that support symmetric KMS keys also support multi\-Region symmetric KMS keys\. All API operations that support asymmetric KMS keys also support multi\-Region asymmetric KMS keys\.
  + You can't create multi\-Region keys in a custom key store\.
+ [Imported key material](importing-keys.md)
  + Only symmetric encryption KMS keys can have imported key material\.
  + Asymmetric KMS keys, HMAC KMS keys, and KMS keys in custom key stores cannot have imported key material\. 
  + Multi\-Region symmetric encryption keys can have imported key material\.
  + Automatic key rotation \(EnableKeyRotation, DisableKeyRotation\) is not supported for keys with imported key material\.
+ [Custom key stores](custom-key-store-overview.md)
  + Custom key stores support only symmetric KMS keys\.
  + Automatic key rotation \(EnableKeyRotation, DisableKeyRotation\) is not supported for keys in custom key stores\.
  + You can't create multi\-Region keys in custom key stores\.

The following table lists the AWS KMS operations that you can use to create and manage KMS keys of each type\. If you use the operation on a KMS key that doesn't not support it, the operation fails\.

You might need to scroll horizontally or vertically to see all of the data in this table\.

<a name="symm-asymm-table"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/symm-asymm-compare.html)

\[1\] `GenerateDataKeyPair` and `GenerateDataKeyPairWithoutPlaintext` generate an asymmetric data key pair that is protected by a symmetric encryption KMS key\.