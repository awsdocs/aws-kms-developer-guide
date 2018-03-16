# Importing Key Material Step 2: Download the Public Key and Import Token<a name="importing-keys-get-public-key-and-token"></a>

After you [create a customer master key \(CMK\) with no key material](importing-keys-create-cmk.md), you download a public key and import token for that CMK\. You need these items to import your key material, and you can download both items in one step using the AWS Management Console or the AWS KMS API\.

You also download these items when you want to reimport key material into a CMK\. You might do this to change the expiration time for the key material, or to restore a CMK after the key material has expired or been deleted\.

**Use of the public key**  
When you import key material, you don't upload the raw key material to AWS KMS\. You must first encrypt the key material with the public key that you download in this step, and then upload the encrypted key material to AWS KMS\. When AWS KMS receives your encrypted key material, it uses the corresponding private key to decrypt it\. The public key you receive from AWS KMS is a 2048\-bit RSA public key and is always unique to your AWS account\.

**Use of the import token**  
The import token contains metadata to ensure that your key material is imported correctly\. When you upload your encrypted key material to AWS KMS, you must upload the same import token that you download in this step\.

**Plan ahead**  
Before you download a public key and import token, you must determine how you will encrypt your key material\. Typically, you choose an option based on the capabilities of the hardware security module \(HSM\) or key management system that protects your key material\. You must use the RSA PKCS \#1 encryption scheme with one of three padding options, represented by the following choices\. These choices are listed in order of AWS preference\. The technical details of the schemes represented by these choices are explained in section 7 of the [PKCS \#1 Version 2\.1](https://tools.ietf.org/html/rfc3447) standard\.

+ **RSAES\_OAEP\_SHA\_256** – The RSA encryption algorithm with Optimal Asymmetric Encryption Padding \(OAEP\) with the SHA\-256 hash function\.

+ **RSAES\_OAEP\_SHA\_1** – The RSA encryption algorithm with Optimal Asymmetric Encryption Padding \(OAEP\) with the SHA\-1 hash function\.

+ **RSAES\_PKCS1\_V1\_5** – The RSA encryption algorithm with the padding format defined in PKCS \#1 Version 1\.5\.

**Note**  
If you plan to try the [Encrypt Key Material with OpenSSL](importing-keys-encrypt-key-material.md#importing-keys-encrypt-key-material-openssl) proof\-of\-concept example in [Step 3](importing-keys-encrypt-key-material.md), use RSAES\_OAEP\_SHA\_1\.

If your HSM or key management system supports it, we recommending using RSAES\_OAEP\_SHA\_256 to encrypt your key material\. If that option is not available, you should use RSAES\_OAEP\_SHA\_1\. If neither of the OAEP options are available, you must use RSAES\_PKCS1\_V1\_5\. For information about how to encrypt your key material, see the documentation for the hardware security module or key management system that protects your key material\.

The public key and import token are valid for 24 hours\. If you don't use them to import key material within 24 hours of downloading them, you must download new ones\.

To download the public key and import token, you can use the AWS Management Console or the AWS KMS API\. You can use the API directly by making HTTP requests, or through one of the [AWS SDKs](https://aws.amazon.com/tools/#sdk) or [command line tools](https://aws.amazon.com/tools/#cli)\.


+ [Download the Public Key and Import Token \(AWS Management Console\)](#importing-keys-get-public-key-and-token-console)
+ [Download the Public Key and Import Token \(AWS KMS API\)](#importing-keys-get-public-key-and-token-api)

## Download the Public Key and Import Token \(AWS Management Console\)<a name="importing-keys-get-public-key-and-token-console"></a>

You can use the AWS Management Console to download the public key and import token\. If you just completed the steps to [create a CMK with no key material](importing-keys-create-cmk.md#importing-keys-create-cmk-console), skip to [Step 6](#id-wrap-step)\.

**To download the public key and import token \(console\)**

1. Sign in to the AWS Management Console and open the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Choose the alias of the CMK for which you are downloading the public key and import token\.
**Tip**  
The CMK's **Origin** must be **External**\. If you don't see the **Origin** column, choose the settings button \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-icon-settings.png)\) in the upper\-right corner of the page\. Select the check box next to **Origin**, and then choose **Close**\.

1. In the **Key Material** section of the page, choose **Download wrapping key and import token**\.

1. <a name="id-wrap-step"></a>For **Select wrapping algorithm**, choose the option that you will use to encrypt your key material\. For more information about the options, see the preceding section\.

   If you plan to try the [ Encrypt Key Material with OpenSSL](importing-keys-encrypt-key-material.md#importing-keys-encrypt-key-material-openssl) proof\-of\-concept example in [Step 3](importing-keys-encrypt-key-material.md), choose RSAES\_OAEP\_SHA\_1\.

1. Choose **Download wrapping key and import token**, and then save the file\.

1. Decompress the `.zip` file that you saved in the previous step \(`ImportParameters.zip`\)\.

   The folder contains the following files:

   + The wrapping key \(public key\), in a file named wrappingKey\_*CMK\_key\_ID*\_*timestamp* \(for example, `wrappingKey_f44c4e20-f83c-48f4-adc6-a1ef38829760_0809092909`\)\. This is a 2048\-bit RSA public key\.

   + The import token, in a file named importToken\_*CMK\_key\_ID*\_*timestamp* \(for example, `importToken_f44c4e20-f83c-48f4-adc6-a1ef38829760_0809092909`\)\.

   + A text file named README\_*CMK\_key\_ID*\_*timestamp*\.txt \(for example, `README_f44c4e20-f83c-48f4-adc6-a1ef38829760_0809092909.txt`\)\. This file contains information about the wrapping key \(public key\), the wrapping algorithm to use to encrypt your key material, and the date and time when the wrapping key \(public key\) and import token expire\.

   To continue the process now, proceed to the next step\. Otherwise, choose **Skip and do this later** and then proceed to [Step 3: Encrypt the Key Material](importing-keys-encrypt-key-material.md)\.

1. \(Optional\) To continue the process now, [encrypt your key material](importing-keys-encrypt-key-material.md)\. Then do one of the following:

   + If you are in the **Import key material** wizard, select the check box for **I am ready to upload my exported key material** and choose **Next**\.

   + If you are in the key details page, choose **Upload key material**\.

After you complete this step, proceed to [Step 3: Encrypt the Key Material](importing-keys-encrypt-key-material.md)\.

## Download the Public Key and Import Token \(AWS KMS API\)<a name="importing-keys-get-public-key-and-token-api"></a>

To use the [AWS KMS API](http://docs.aws.amazon.com/kms/latest/APIReference/) to download the public key and import token, send a [GetParametersForImport](http://docs.aws.amazon.com/kms/latest/APIReference/API_GetParametersForImport.html) request that specifies the CMK for which you are downloading these items\. The following example shows how to do this with the [AWS CLI](https://aws.amazon.com/cli/)\.

This example specifies `RSAES_OAEP_SHA_1` as the encryption option\. To specify a different option, replace `RSAES_OAEP_SHA_1` with `RSAES_OAEP_SHA_256` or `RSAES_PKCS1_V1_5`\. Replace `1234abcd-12ab-34cd-56ef-1234567890ab` with the key ID of the CMK for which to download the public key and import token\. You can use the CMK's key ID or Amazon Resource Name \(ARN\), but you cannot use an alias for this operation\.

**Note**  
If you plan to try the [Encrypt Key Material with OpenSSL](importing-keys-encrypt-key-material.md#importing-keys-encrypt-key-material-openssl) proof\-of\-concept example in [Step 3](importing-keys-encrypt-key-material.md), specify RSAES\_OAEP\_SHA\_1\.

```
$ aws kms get-parameters-for-import --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
                                    --wrapping-algorithm RSAES_OAEP_SHA_1 \
                                    --wrapping-key-spec RSA_2048
```

When the command is successful, you see output similar to the following:

```
{
    "ParametersValidTo": 1470933314.949,
    "PublicKey": "public key base64 encoded data",
    "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
    "ImportToken": "import token base64 encoded data"
}
```

When you receive this output, save the base64 encoded public key and import token in separate files\. Then base64 decode each file into binary data and save the binary data in new files\. Doing so prepares these items for later steps\. See the following example\.

**To prepare the public key and import token for later steps**

1. Copy the public key's base64 encoded data \(represented by *public key base64 encoded data* in the example output\), paste it into a new file, and then save the file\. Give the file a descriptive name, for example `PublicKey.b64`\.

1. Use [OpenSSL](https://openssl.org/) to base64 decode the file's contents and save the decoded data to a new file\. The following example decodes the data in the file that you saved in the previous step \(`PublicKey.b64`\) and saves the output to a new file named `PublicKey.bin`\.

   ```
   $ openssl enc -d -base64 -A -in PublicKey.b64 -out PublicKey.bin
   ```

Repeat these two steps for the import token, and then proceed to [Step 3: Encrypt the Key Material](importing-keys-encrypt-key-material.md)\.