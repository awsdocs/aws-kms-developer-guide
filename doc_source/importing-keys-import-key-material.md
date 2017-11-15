# Importing Key Material Step 4: Import the Key Material<a name="importing-keys-import-key-material"></a>

After you encrypt your key material, you can import the key material to use with an AWS KMS customer master key \(CMK\)\. To import key material, you upload the encrypted key material from [Step 3: Encrypt the Key Material](importing-keys-encrypt-key-material.md) and the import token that you downloaded at [Step 2: Download the Public Key and Import Token](importing-keys-get-public-key-and-token.md)\. You must import key material into the same CMK that you specified when you downloaded the public key and import token\.

When you import key material, you can optionally specify a time at which the key material expires\. When the key material expires, AWS KMS deletes the key material and the CMK becomes unusable\. To use the CMK again, you must reimport key material\.

After you successfully import key material, the CMK's key state changes to enabled, and you can use the CMK\.

To import key material, you can use the AWS Management Console or the AWS KMS API\. You can use the API directly by making HTTP requests, or through one of the [AWS SDKs](https://aws.amazon.com/tools/#sdk) or [command line tools](https://aws.amazon.com/tools/#cli)\.


+ [Import Key Material \(AWS Management Console\)](#importing-keys-import-key-material-console)
+ [Import Key Material \(AWS KMS API\)](#importing-keys-import-key-material-api)

## Import Key Material \(AWS Management Console\)<a name="importing-keys-import-key-material-console"></a>

You can use the AWS Management Console to import key material\. If you just completed the optional final step of downloading the public key and import token with the console, skip to [[ERROR] BAD/MISSING LINK TEXT](#id-key-materials-step)\.

**To import key material \(console\)**

1. Sign in to the AWS Management Console and open the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Choose the alias of the CMK for which you downloaded the public key and import token\.

1. In the **Key Material** section of the page, choose **Upload key material**\.

1. In the **Specify key material details** section, for **Encrypted key material**, choose the file that contains your encrypted key material\. For **Import token**, choose the file that contains the import token that you downloaded previously\.

1. In the **Choose an expiration option** section, choose whether the key material expires\. If you choose expiration, type a date and a time in the corresponding boxes\.

1. Choose **Finish** or **Upload key material**\.

## Import Key Material \(AWS KMS API\)<a name="importing-keys-import-key-material-api"></a>

To use the [AWS KMS API](http://docs.aws.amazon.com/kms/latest/APIReference/) to import key material, send an [ImportKeyMaterial](http://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html) request\. The following example shows how to do this with the [AWS CLI](https://aws.amazon.com/cli/)\.

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
                              --valid-to 2016-11-08T12:00:00-08:00
```