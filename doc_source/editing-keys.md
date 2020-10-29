# Editing keys<a name="editing-keys"></a>

You can change the following properties of your [customer managed CMKs](concepts.md#customer-cmk) in the AWS KMS console and the AWS KMS API\.

**Description**  
You can change the key description on the [details page for the CMK](viewing-keys-console.md#viewing-console-details) or by using the [UpdateKeyDescription](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateKeyDescription.html) operation\.  
To edit the key description in the console, in the upper right corner of the details page for the CMK, choose **Edit**\.

**Key policy**  
You can change the [key policy](key-policies.md) on the **Key policy** tab of the [details page for the CMK](viewing-keys-console.md#viewing-console-details) or by using the [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operation\.  
For details, see [Changing a key policy](key-policy-modifying.md)\.

**Tags**  
You can create and delete [tags](tagging-keys.md) on the **Tags** tab of the [details page for the CMK](viewing-keys-console.md#viewing-console-details) or by using the [TagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) and [UntagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_UntagResource.html) operations\.  
For details, see [Tagging keys](tagging-keys.md)\.

**Enable and disable**  
You can enable and disable CMKs on the **Customer managed keys** page of the AWS KMS console\. Or you can use the [EnableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKey.html) and [DisableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) operations\.  
For details, see [Enabling and disabling keys](enabling-keys.md)\.

**Automatic key rotation**  
You can enable and disable automatic key rotation on the **Key rotation** tab of the [details page for the CMK](viewing-keys-console.md#viewing-console-details) or by using the [EnableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) and [DisableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_UntagResource.html) operations\.  
Automatic key rotation is not supported on [asymmetric CMKs](symm-asymm-concepts.md#asymmetric-cmks), CMKs with [imported key material](importing-keys.md), or CMKs in [custom key stores](custom-key-store-overview.md)\.  
For details, see [Rotating customer master keys](rotate-keys.md)\.

**Updating key material**  
For CMK with [imported key material](importing-keys.md), you can delete and reimport the same key material\. Use the **Key material** tab of the [details page for the CMK](viewing-keys-console.md#viewing-console-details) or by using the AWS KMS API\.   
For details, see [How to reimport key material](importing-keys.md#reimport-key-material)\.

**See also**
+ [Updating aliases](kms-alias.md#alias-update)
+ [Deleting customer master keys](deleting-keys.md)
+ [Editing custom key store settings](update-keystore.md)