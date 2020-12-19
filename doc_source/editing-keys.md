# Editing keys<a name="editing-keys"></a>

You can change the following properties of your CMKs in the AWS KMS console and the AWS KMS API\.

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
You can enable and disable CMKs of the **Customer managed keys** or **AWS managed keys** pages of the AWS KMS console\. Or you can use the [EnableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKey.html) and [DisableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) operations\.  
For details, see [Enabling and disabling keys](enabling-keys.md)\.

**Automatic key rotation**  
You can enable and disable automatic key rotation on the Key rotation tab of the [details page for the CMK](viewing-keys-console.md#viewing-console-details) or by using the [EnableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) and [DisableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_UntagResource.html) operations\.  
For details, see [Rotating customer master keys](rotate-keys.md)\.

**See also**  
[Updating aliases](alias-manage.md#alias-update)