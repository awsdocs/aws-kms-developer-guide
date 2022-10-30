# Importing key material step 2: Download the public key and import token<a name="importing-keys-get-public-key-and-token"></a>

After you [create a symmetric encryption AWS KMS key with no key material](importing-keys-create-cmk.md), download a public key and an import token for that KMS key\. You can download both items in one step by using the AWS KMS console or the [GetParametersForImport](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetParametersForImport.html) API\. The public key and import token are valid for 24 hours\. If you don't use them to import key material within 24 hours of downloading them, you must download new ones\.

You can also download these items when you want to reimport the same key material into a KMS key\. You might do this to change the expiration time for the key material, or to restore expired or deleted key material\.

**Use of the public key**  
The download includes a public key, also called a *wrapping key*\.  
Before you import key material, you encrypt the key material with the public key, and then upload the encrypted key material to AWS KMS\. When AWS KMS receives your encrypted key material, it uses the corresponding private key to decrypt it\. The public key that AWS KMS provides is a 2048\-bit RSA public key that is unique to your AWS account\.

**Use of the import token**  
The download includes an import token with metadata that ensures that your key material is imported correctly\. When you upload your encrypted key material to AWS KMS, you must upload the same import token that you download in this step\.

**Select a wrapping algorithm**  <a name="select-wrapping-algorithm"></a>
To protect your key material during import, you encrypt it using the downloaded public key and a supported wrapping algorithm\. You must use RSA PKCS \#1 encryption with one of three padding options, represented by the following choices\. These choices are listed in order of AWS preference\. Typically, you choose an algorithm that is supported by the hardware security module \(HSM\) or key management system that protects your key material\. For technical details about these algorithms, see section 7 of the [PKCS \#1 Version 2\.1](https://tools.ietf.org/html/rfc3447) standard\.  
If your HSM or key management system supports it, we recommend using `RSAES_OAEP_SHA_256` to encrypt your key material\. If that option is not available, use `RSAES_OAEP_SHA_1`\. If neither of the OAEP options are available, you must use `RSAES_PKCS1_V1_5`\. For information about how to encrypt your key material, see the documentation for the hardware security module or key management system that protects your key material\.  
To run the [Encrypt Key Material with OpenSSL](importing-keys-encrypt-key-material.md#importing-keys-encrypt-key-material-openssl) proof\-of\-concept example in [Step 3](importing-keys-encrypt-key-material.md), use `RSAES_OAEP_SHA_1`\.
+ `RSAES_OAEP_SHA_256` – The RSA encryption algorithm with Optimal Asymmetric Encryption Padding \(OAEP\) with the SHA\-256 hash function\.
+ `RSAES_OAEP_SHA_1` – The RSA encryption algorithm with Optimal Asymmetric Encryption Padding \(OAEP\) with the SHA\-1 hash function\.
+ `RSAES_PKCS1_V1_5` – The RSA encryption algorithm with the padding format defined in PKCS \#1 Version 1\.5\.

**Topics**
+ [Downloading the public key and import token \(console\)](#importing-keys-get-public-key-and-token-console)
+ [Downloading the public key and import token \(AWS KMS API\)](#importing-keys-get-public-key-and-token-api)

## Downloading the public key and import token \(console\)<a name="importing-keys-get-public-key-and-token-console"></a>

You can use the AWS KMS console to download the public key and import token\.

1. If you just completed the steps to [create a KMS key with no key material](importing-keys-create-cmk.md#importing-keys-create-cmk-console) and you are on the **Download wrapping key and import token** page, skip to [Step 8](#id-wrap-step)\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.
**Tip**  
You can import key material only into a symmetric encryption KMS key with an **Origin** of **EXTERNAL**\. This indicates that the KMS key was created with no key material\. To add the **Origin** column to your table, in the upper\-right corner of the page, choose the settings icon \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-icon-settings-new.png)\)\. Turn on **Origin**, and then choose **Confirm**\.

1. Choose the alias or key ID of the KMS key that is pending import\.

1. Choose the **Cryptographic configuration** tab and view its values\. The tabs are below the **General configuration** section\.

   You can only import key material into KMS keys with a **Key type** of **Symmetric** and an **Origin** of **EXTERNAL**\. For information about creating KMS keys with imported key material, see, [Importing key material in AWS KMS keys](importing-keys.md)\.

1. Choose the **Key material** tab and then choose **Download wrapping key and import token**\. 

   The **Key material** tab appears only for symmetric encryption KMS keys that have an **Origin** value of **EXTERNAL**\.

1. <a name="id-wrap-step"></a>For **Select wrapping algorithm**, choose the option that you will use to encrypt your key material\. For more information about the options, see [Select a Wrapping Algorithm](#select-wrapping-algorithm)\.

   If you plan to try the [ Encrypt Key Material with OpenSSL](importing-keys-encrypt-key-material.md#importing-keys-encrypt-key-material-openssl) proof\-of\-concept example in [Step 3](importing-keys-encrypt-key-material.md), choose RSAES\_OAEP\_SHA\_1\.

1. Choose **Download wrapping key and import token**, and then save the file\. 

   If you have a **Next** option, to continue the process now, choose **Next**\. To continue later, choose **Cancel**\. Otherwise, to close the window, choose **Cancel** or click the **X**\. 

1. Decompress the `.zip` file that you saved in the previous step \(`ImportParameters.zip`\)\.

   The folder contains the following files:
   + A 2048\-bit RSA public key in a file named wrappingKey\_*KMS key\_key\_ID*\_*timestamp* \(for example, `wrappingKey_f44c4e20-f83c-48f4-adc6-a1ef38829760_0809092909`\)\.
   + An import token in a file named importToken\_*KMS key\_key\_ID*\_*timestamp* \(for example, `importToken_f44c4e20-f83c-48f4-adc6-a1ef38829760_0809092909`\)\.
   + A text file named README\_*KMS key\_key\_ID*\_*timestamp*\.txt \(for example, `README_f44c4e20-f83c-48f4-adc6-a1ef38829760_0809092909.txt`\)\. This file contains information about the public key, the wrapping algorithm to use to encrypt your key material, and the date and time when the wrapping key \(public key\) and import token expire\.

1. To continue the process, see [encrypt your key material](importing-keys-encrypt-key-material.md)\. 

## Downloading the public key and import token \(AWS KMS API\)<a name="importing-keys-get-public-key-and-token-api"></a>

To download the public key and import token, use the [GetParametersForImport](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetParametersForImport.html) API\. Specify the KMS key that will be associated with the imported key material\. This KMS key must have an [Origin](concepts.md#key-origin) value of `EXTERNAL`\.

This example specifies a wrapping algorithm value of `RSAES_OAEP_SHA_1`\. To specify a different option, replace `RSAES_OAEP_SHA_1` with `RSAES_OAEP_SHA_256` or `RSAES_PKCS1_V1_5`\. Replace `1234abcd-12ab-34cd-56ef-1234567890ab` with the key ID of the KMS key for which to download the public key and import token\. You can use the [key ID](concepts.md#key-id-key-id) or [key ARN](concepts.md#key-id-key-ARN), but you cannot use an [alias name](concepts.md#key-id-alias-name) or [alias ARN](concepts.md#key-id-alias-ARN) for this operation\.

**Note**  
To use the [Encrypt Key Material with OpenSSL](importing-keys-encrypt-key-material.md#importing-keys-encrypt-key-material-openssl) proof\-of\-concept example in [Step 3](importing-keys-encrypt-key-material.md), specify RSAES\_OAEP\_SHA\_1\.

```
$ aws kms get-parameters-for-import \
    --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
    --wrapping-algorithm RSAES_OAEP_SHA_1 \
    --wrapping-key-spec RSA_2048
```

When the command is successful, you see output similar to the following:

```
{
    "ParametersValidTo": 1568290320.0,
    "PublicKey": "public key (base64 encoded)",
    "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
    "ImportToken": "import token (base64 encoded)"
}
```

To prepare the data for the next step, base64 decode the public key and import token and save the decoded values in files\.

To base64 decode the public key and import token:

1. Copy the base64 encoded public key \(represented by *public key \(base64 encoded\)* in the example output\), paste it into a new file, and then save the file\. Give the file a descriptive name, such as `PublicKey.b64`\.

1. Use [OpenSSL](https://openssl.org/) to base64 decode the file's contents and save the decoded data to a new file\. The following example decodes the data in the file that you saved in the previous step \(`PublicKey.b64`\) and saves the output to a new file named `PublicKey.bin`\.

   ```
   $ openssl enc -d -base64 -A -in PublicKey.b64 -out PublicKey.bin
   ```

1. Copy the base64 encoded import token \(represented by *import token \(base64 encoded\)* in the example output\), paste it into a new file, and then save the file\. Give the file a descriptive name, for example `ImportToken.b64`\.

1. Use [OpenSSL](https://openssl.org/) to base64 decode the file's contents and save the decoded data to a new file\. The following example decodes the data in the file that you saved in the previous step \(`ImportToken.b64`\) and saves the output to a new file named `ImportToken.bin`\.

   ```
   $ openssl enc -d -base64 -A -in ImportToken.b64 -out ImportToken.bin
   ```

Proceed to [Step 3: Encrypt the key material](importing-keys-encrypt-key-material.md)\.