# Importing key material step 4: Import the key material<a name="importing-keys-import-key-material"></a>

After you [encrypt your key material](importing-keys-encrypt-key-material.md), you can import the key material to use with an AWS KMS customer master key \(CMK\)\. To import key material, you upload the encrypted key material from [Step 3: Encrypt the key material](importing-keys-encrypt-key-material.md) and the import token that you downloaded at [Step 2: Download the public key and import token](importing-keys-get-public-key-and-token.md)\. You must import key material into the same CMK that you specified when you downloaded the public key and import token\.

When you import key material, you can optionally specify a time at which the key material expires\. When the key material expires, AWS KMS deletes the key material and the CMK becomes unusable\. To use the CMK again, you must reimport key material\.

After you successfully import key material, the CMK's key state changes to enabled, and you can use the CMK\.

To import key material, you can use the AWS Management Console or the AWS KMS API\. You can use the API directly by making HTTP requests, or through one of the [AWS SDKs](https://aws.amazon.com/tools/#sdk) or [command line tools](https://aws.amazon.com/tools/#cli)\.

When you import the key material, an [ImportKeyMaterial entry](ct-importkeymaterial.md) is added to your AWS CloudTrail log to record the [ImportKeyMaterial](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html) operation\. The CloudTrail entry is the same whether you use the AWS KMS console or the AWS KMS API\.

## Import key material \(console\)<a name="importing-keys-import-key-material-console"></a>

You can use the AWS Management Console to import key material\.

1. If you are on the **Download wrapping key and import token** page, skip to [Step 8](#id-key-materials-step)\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Choose the key ID or alias of the CMK for which you downloaded the public key and import token\.

1. Expand the **Cryptographic configuration** section and view its values\.

   You can only import key material into CMKs with a **Key type** of **Symmetric** and an **Origin** of **EXTERNAL**\. For information about creating CMKs with imported key material, see [Importing key material in AWS Key Management Service \(AWS KMS\)](importing-keys.md)\.

1. Expand the **Key material** section and then choose **Upload key material**\.

   The **Key material** section appears only for CMKs with a **Key type** of **Symmetric** and an **Origin** value of **EXTERNAL**\.

1. <a name="id-key-materials-step"></a>In the **Encrypted key material and import token** section, under **Wrapped key material**, choose **Choose file**\. Then upload the file that contains your wrapped \(encrypted\) key material\. 

1. In the **Encrypted key material and import token** section, under **Import token**, choose **Choose file**\. Upload the file that contains the import token that you [downloaded](importing-keys-get-public-key-and-token.md#importing-keys-get-public-key-and-token-console)\.

1. In the **Expiration option** section, you determine whether the key material expires\. To set an expiration date and time, choose **Key material expires**, and use the calendar to select a date and time\.

1. Choose **Finish** or **Upload key material**\.

## Import key material \(AWS KMS API\)<a name="importing-keys-import-key-material-api"></a>

To use the [AWS KMS API](https://docs.aws.amazon.com/kms/latest/APIReference/) to import key material, send an [ImportKeyMaterial](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html) request\. The following example shows how to do this with the [AWS CLI](https://aws.amazon.com/cli/)\.

This example specifies an expiration time for the key material\. To import key material with no expiration, replace `KEY_MATERIAL_EXPIRES` with `KEY_MATERIAL_DOES_NOT_EXPIRE` and omit the `--valid-to` parameter\.

To use this example:

1. Replace `1234abcd-12ab-34cd-56ef-1234567890ab` with the key ID of the CMK that you used when you downloaded the public key and import token\. To identify the CMK, use its key ID or ARN\. You cannot use an alias for this operation\.

1. Replace `EncryptedKeyMaterial.bin` with the name of the file that contains the encrypted key material\.

1. Replace `ImportToken.bin` with the name of the file that contains the import token\.

```
$ aws kms import-key-material --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
                              --encrypted-key-material fileb://EncryptedKeyMaterial.bin \
                              --import-token fileb://ImportToken.bin \
                              --expiration-model KEY_MATERIAL_EXPIRES \
                              --valid-to 2019-09-17T12:00:00-08:00
```