# How AWS Nitro Enclaves uses AWS KMS<a name="services-nitro-enclaves"></a>

AWS Nitro Enclaves is an Amazon EC2 capability that allows you to create isolated compute environments from Amazon EC2 instances\. 

Applications running in AWS Nitro Enclaves can use the [https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt) to call the AWS KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt), [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey), and [GenerateRandom](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateRandom) operations\. The Nitro Enclaves SDK adds the attestation document from the enclave to each AWS KMS API request\. Instead of returning plaintext data, the AWS KMS operations encrypt the plaintext with the public key from the attestation document\. This design allows for the ciphertext to be decrypted only by the corresponding private key within the enclave\.

To support AWS Nitro Enclaves, AWS KMS adds a `Recipient` request parameter with the `RecipientInfo` object type and a `CiphertextForRecipient` response field to the standard request and response fields for these operations\. These enclave\-specific elements are valid only in the supported API operations and only when the request is signed using the AWS Nitro Enclaves Development Kit\. AWS KMS relies on the digital signature for the enclave’s attestation document to prove that the public key in the request came from a valid AWS Nitro enclave\. You cannot supply your own certificate to digitally sign the attestation document\.

AWS KMS also supports policy condition keys that you can use to allow enclave operations on an AWS KMS key only when the attestation document has the specified content\. For details, see [AWS KMS condition keys for AWS Nitro Enclaves](policy-conditions.md#conditions-nitro-enclaves)\.

For information about AWS Nitro Enclaves, see [What is AWS Nitro Enclaves](https://docs.aws.amazon.com/enclaves/latest/user/nitro-enclave.html) in the *AWS Nitro Enclaves Developer Guide*\. For information about setting up your data and data keys for encryption, see [Using cryptographic attestation with AWS KMS](https://docs.aws.amazon.com/enclaves/latest/user/kms.html)\.

## Recipient<a name="nitro-recipient"></a>

```
"Recipient": { 
   "AttestationDocument": blob,
   "KeyEncryptionAlgorithm": "string"
}
```

A request parameter that contains the signed attestation document from an enclave and an encryption algorithm\. The only valid encryption algorithm is `RSAES_OAEP_SHA_256`\.

This parameter is valid only when the request comes from the AWS Nitro Enclaves Development Kit\.

**Type**: RecipientInfo object

## RecipientInfo<a name="recipient-info"></a>

This type contains information about the enclave that receives the response from the API operation\.

**AttestationDocument**  
A document with measurements that describe the state of the Nitro enclave\. This document also includes the enclave's public key\. AWS KMS will encrypt any plaintext in the response under this public key so that it can be decrypted later only by the corresponding private key in the enclave\.  
**Type**: Base64\-encoded binary data object  
**Length Constraints**: Minimum length of 1\. Maximum length of 262144\.  
**Required**: No

**KeyEncryptionAlgorithm**  
The encryption algorithm that AWS KMS should use with the public key\. The only valid value is `RSAES_OAEP_SHA_256`\.  
**Type**: String  
**Valid Values**: RSAES\_OAEP\_SHA\_256  
**Required**: No

## CiphertextForRecipient<a name="ciphertext-for-recipient"></a>

```
{
   "CiphertextForRecipient": blob
}
```

This response field contains a ciphertext encrypted with the public key from the attestation document in the request\. This field is populated only when the request includes a `Recipient` parameter with a valid attestation document and encryption algorithm\. When this field is populated, the `Plaintext` field in the response is null\.

**Type**: Base64\-encoded binary data object

**Length Constraints**: Minimum length of 1\. Maximum length of 6144\.

## AWS KMS operations for AWS Nitro Enclaves<a name="recipient-operations"></a>

The following AWS KMS operations support Nitro Enclaves\. This topic explains how these API operations behave when a request comes from the AWS Nitro Enclaves Development Kit and the `Recipient` parameter includes a valid attestation document\. These operations support the `Recipient` parameter and the `CiphertextForRecipient` response field\. 

### Decrypt<a name="recipient-decrypt"></a>

To call the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt) operation from an enclave, use the [kms\-decrypt](https://github.com/aws/aws-nitro-enclaves-sdk-c/blob/main/docs/kms-apis/Decrypt.md) operation in the AWS Nitro Enclaves Development Kit\.

After using the specified AWS KMS key to decrypt the ciphertext blob in the request, the `Decrypt` operation re\-encrypts the resulting plaintext using the public key from the attestation document and the specified encryption algorithm\. It returns the resulting ciphertext in the `CiphertextForRecipient` field in the response\. The `Plaintext` field in the response is null\.

### GenerateDataKey<a name="recipient-generate-data-key"></a>

To call the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey) operation from an enclave, use the [kms\-generate\-data\-key](https://github.com/aws/aws-nitro-enclaves-sdk-c/blob/main/docs/kms-apis/GenerateDataKey.md) operation in the AWS Nitro Enclaves Development Kit\.

After generating the data key, the `GenerateDataKey` operation encrypts one copy of the data key under the specified AWS KMS key and returns it in the `CiphertextBlob` field\. It encrypts the other copy of the data key under the public key from the attestation document and returns it in the `CiphertextForRecipient` field\. The `Plaintext` field in the response is null\. 

Your use of the two encrypted data keys in the `GenerateDataKey` response depends on your use of the enclave\.
+ If you want to use the data key to encrypt data within the enclave, decrypt the value in the `CiphertextForRecipient` field using the private key within your enclave\. If you want to persist this newly encrypted data outside of the enclave, you can store it with either of the two encrypted data key copies in the `GenerateDataKey` \(`kms-generate-data-key`\) response\.
+ If you intend to keep the enclave running and can rely on the durability of the private key in enclave memory, you can include the `CiphertextForRecipient` object with the newly encrypted data when you move it outside of the enclave\. When you’re ready to decrypt the `CiphertextForRecipient` object, you must use the corresponding private key in the enclave\. 

  If you don’t intend to keep your enclave running or you don't want to rely on the durability of the private key in enclave memory, you should include the `CiphertextBlob` object with your encrypted data\. To decrypt this copy of the data key, you must send it to AWS KMS in a `Decrypt` \(`kms-decrypt`\) request\. 

  You can also pass the `CiphertextBlob` object to a different enclave, and then decrypt it by calling the `kms-decrypt` \(`Decrypt`\) operation in the AWS Nitro Enclaves Development Kit\. This request will include the attestation document for the new enclave with a new public key\. AWS KMS will decrypt the data key that was encrypted under the KMS key, then re\-encrypt it under the public key of the new enclave\. This data key can be decrypted only by using the corresponding private key within the new enclave\.

### GenerateRandom<a name="recipient-generate-random"></a>

To call the [GenerateRandom](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateRandom) operation from an enclave, use the [kms\-generate\-random](https://github.com/aws/aws-nitro-enclaves-sdk-c/blob/main/docs/kms-apis/GenerateRandom.md) operation in the AWS Nitro Enclaves Development Kit\.

After generating the random byte string, the [GenerateRandom](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateRandom) operation encrypts the random byte string using the public key in the attestation document and the specified encryption algorithm\. It returns the encrypted byte string in the `CiphertextForRecipient` field\. The `Plaintext` field in the response is null\.