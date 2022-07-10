# Editing keys<a name="editing-keys"></a>

You can change the following properties of your [customer managed keys ](concepts.md#customer-cmk)in the AWS KMS console and by using AWS KMS API\. 

You cannot edit any properties of [AWS managed keys](concepts.md#aws-managed-cmk) or [AWS owned keys](concepts.md#aws-owned-cmk)\. These keys are managed by the AWS services that created them\.

**Description**  
You can change the description of your customer managed key on the [details page](viewing-keys-console.md#viewing-details-navigate) for the KMS key or by using the [UpdateKeyDescription](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateKeyDescription.html) operation\.  
To edit the key description in the console, in the upper right corner of the details page for the KMS key, choose **Edit**\.

**Key policy**  
You can change the [key policy](key-policies.md) on the **Key policy** tab of the [details page](viewing-keys-console.md#viewing-details-navigate) for the customer managed key or by using the [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operation\.  
For details, see [Changing a key policy](key-policy-modifying.md)\.

**Tags**  
You can create and delete [tags](tagging-keys.md) on the **Customer managed keys** page of the AWS KMS console, or on the **Tags** tab of the [details page](viewing-keys-console.md#viewing-details-navigate) for the customer managed key\. Or you can use the [TagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) and [UntagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_UntagResource.html) operations\.  
For details, see [Tagging keys](tagging-keys.md)\.

**Enable and disable**  
You can enable and disable KMS keys on the **Customer managed keys** page of the AWS KMS console, or on the [details page](viewing-keys-console.md#viewing-details-navigate) for the customer managed key\. Or you can use the [EnableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKey.html) and [DisableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) operations\.  
For details, see [Enabling and disabling keys](enabling-keys.md)\.

**Automatic key rotation**  
You can enable and disable automatic key rotation on the **Key rotation** tab of the [details page](viewing-keys-console.md#viewing-details-navigate) for the customer managed key or by using the [EnableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKeyRotation.html) and [DisableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKeyRotation.html) operations\.  
For details, see [Rotating AWS KMS keys](rotate-keys.md)\.

**See also**  
[Updating aliases](alias-manage.md#alias-update)