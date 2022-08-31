# Importing key material step 4: Import the key material<a name="importing-keys-import-key-material"></a>

After you [encrypt your key material](importing-keys-encrypt-key-material.md), you can import the key material to use with an AWS KMS key\. To import key material, you upload the encrypted key material from [Step 3: Encrypt the key material](importing-keys-encrypt-key-material.md) and the import token that you downloaded at [Step 2: Download the public key and import token](importing-keys-get-public-key-and-token.md)\. You must import key material into the same KMS key that you specified when you [downloaded the public key and import token](importing-keys-get-public-key-and-token.md)\. When key material is imported, the [key state](key-state.md) of the KMS key changes to `Enabled`, and you can use the KMS key in cryptographic operations\.

When you import key material, you can set an optional expiration date for the key material\. When the key material expires, AWS KMS deletes the key material and the KMS key becomes unusable\. To use the KMS key in cryptographic operations, you must reimport the same key material\. After you import your key material, you cannot set, change, or cancel the expiration date for the current import\. To change these values, you must [delete](importing-keys-delete-key-material.md) and [reimport](importing-keys.md#reimport-key-material) the same key material\.

To import key material, you can use the AWS KMS console or the [ImportKeyMaterial](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html) API\. You can use the API directly by making HTTP requests, or by using an [AWS SDKs](https://aws.amazon.com/tools/#sdk), [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/) or [AWS Tools for PowerShell](https://docs.aws.amazon.com/powershell/latest/userguide/)\.

When you import the key material, an [ImportKeyMaterial entry](ct-importkeymaterial.md) is added to your AWS CloudTrail log to record the `ImportKeyMaterial` operation\. The CloudTrail entry is the same whether you use the AWS KMS console or the AWS KMS API\.

## Import key material \(console\)<a name="importing-keys-import-key-material-console"></a>

You can use the AWS Management Console to import key material\.

1. If you are on the **Download wrapping key and import token** page, skip to [Step 8](#id-key-materials-step)\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Choose the key ID or alias of the KMS key for which you downloaded the public key and import token\.

1. Choose the **Cryptographic configuration** tab and view its values\. The tabs are on the detail page for a KMS key below the **General configuration** section\.

   You can only import key material into KMS keys with a **Key type** of **Symmetric** and an **Origin** of **EXTERNAL**\. For information about creating KMS keys with imported key material, see [Importing key material in AWS KMS keys](importing-keys.md)\.

1. Choose the **Key material** tab and then choose **Upload key material**\.

   The **Key material** tab appears only for KMS keys with a **Key type** of **Symmetric** and an **Origin** value of **EXTERNAL**\.

1. <a name="id-key-materials-step"></a>In the **Encrypted key material and import token** section, under **Wrapped key material**, choose **Choose file**\. Then upload the file that contains your wrapped \(encrypted\) key material\. 

1. In the **Encrypted key material and import token** section, under **Import token**, choose **Choose file**\. Upload the file that contains the import token that you [downloaded](importing-keys-get-public-key-and-token.md#importing-keys-get-public-key-and-token-console)\.

1. In the **Expiration option** section, you determine whether the key material expires\. To set an expiration date and time, choose **Key material expires**, and use the calendar to select a date and time\. You can specify a date up to 365 days from the current date and time\.

1. Choose **Finish** or **Upload key material**\.

## Import key material \(AWS KMS API\)<a name="importing-keys-import-key-material-api"></a>

To import key material, use the [ImportKeyMaterial](https://docs.aws.amazon.com/kms/latest/APIReference/API_ImportKeyMaterial.html) operation\. The following example uses the [AWS CLI](https://aws.amazon.com/cli/), but you can use any supported programming language\.

To use this example:

1. Replace `1234abcd-12ab-34cd-56ef-1234567890ab` with a key ID of the KMS key that you specified when you downloaded the public key and import token\. To identify the KMS key, use its [key ID](concepts.md#key-id-key-id) or [key ARN](concepts.md#key-id-key-ARN)\. You cannot use an [alias name](concepts.md#key-id-alias-name) or [alias ARN](concepts.md#key-id-alias-ARN) for this operation\.

1. Replace `EncryptedKeyMaterial.bin` with the name of the file that contains the encrypted key material\.

1. Replace `ImportToken.bin` with the name of the file that contains the import token\.

1. If you want the imported key material to expire, set the value of the `expiration-model` parameter to its default value, `KEY_MATERIAL_EXPIRES`, or omit the `expiration-model` parameter\. Then, replace the value of the `valid-to` parameter with the date and time that you want the key material to expire\. The date and time can be up to 365 days from the time of the request\. 

   ```
   $ aws kms import-key-material --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
       --encrypted-key-material fileb://EncryptedKeyMaterial.bin \
       --import-token fileb://ImportToken.bin \
       --expiration-model KEY_MATERIAL_EXPIRES \
       --valid-to 2022-09-17T12:00:00-08:00
   ```

   If you do not want the imported key material to expire, set the value of the `expiration-model` parameter to `KEY_MATERIAL_DOES_NOT_EXPIRE` and omit the `valid-to` parameter from the command\.

   ```
   $ aws kms import-key-material --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \ 
       --encrypted-key-material fileb://EncryptedKeyMaterial.bin \
       --import-token fileb://ImportToken.bin \
       --expiration-model KEY_MATERIAL_DOES_NOT_EXPIRE
   ```