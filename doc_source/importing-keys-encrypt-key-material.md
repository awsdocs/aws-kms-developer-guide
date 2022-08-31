# Importing key material step 3: Encrypt the key material<a name="importing-keys-encrypt-key-material"></a>

After you [download the public key and import token](importing-keys-get-public-key-and-token.md), you use the public key to encrypt your key material\. The key material must be in binary format\. 

Typically, you encrypt your key material when you export it from your hardware security module \(HSM\) or key management system\. For information about how to export key material in binary format, see the documentation for your HSM or key management system\. You can also refer to the following section that provides a proof of concept demonstration using OpenSSL\. 

When you encrypt your key material, use the same wrapping algorithm that you specified when you [downloaded the public key and import token](importing-keys-get-public-key-and-token.md) \(RSAES\_OAEP\_SHA\_256, RSAES\_OAEP\_SHA\_1, or RSAES\_PKCS1\_V1\_5\)\.

## Example: Encrypt key material with OpenSSL<a name="importing-keys-encrypt-key-material-openssl"></a>

The following example demonstrates how to use [OpenSSL](https://openssl.org/) to generate a 256\-bit symmetric encryption key and then encrypt this key material for import into a KMS key\.

In China Regions, generate a 128\-bit symmetric encryption key\. 

**Important**  
This example is a proof of concept demonstration only\. For production systems, use a more secure method \(such as a commercial HSM or key management system\) to generate and store your key material\.  
The `RSAES_OAEP_SHA_1` encryption algorithm works best with this example\. Before running the example, make sure that you used RSAES\_OAEP\_SHA\_1 for the wrapping algorithm in [Step 2](importing-keys-get-public-key-and-token.md)\. If necessary, repeat the step to download and import the public key and token\.

**To use OpenSSL to generate binary key material and encrypt it for import into AWS KMS**

1. Use the following command to generate a 256\-bit symmetric key and save it in a file named `PlaintextKeyMaterial.bin`\.

   \(China Regions only\) To generate a 128\-bit symmetric key, change the `32` in the following command to `16`\.

   ```
   $ openssl rand -out PlaintextKeyMaterial.bin 32
   ```

1. Use the following command to encrypt the key material with the [public key that you downloaded](importing-keys-get-public-key-and-token.md) and save it in a file named `EncryptedKeyMaterial.bin`\. Replace `PublicKey.bin` with the name of the file that contains the public key\. If you downloaded the public key from the console, this file is named wrappingKey\_*KMS key\_key\_ID*\_*timestamp* \(for example, `wrappingKey_f44c4e20-f83c-48f4-adc6-a1ef38829760_0809092909`\)\.

   ```
   $ openssl pkeyutl \
       -in PlaintextKeyMaterial.bin \
       -out EncryptedKeyMaterial.bin \
       -inkey PublicKey.bin \
       -keyform DER -pubin \
       -encrypt \
       -pkeyopt rsa_padding_mode:oaep \
       -pkeyopt rsa_oaep_md:sha1
   ```

Proceed to [Step 4: Import the key material](importing-keys-import-key-material.md)\.