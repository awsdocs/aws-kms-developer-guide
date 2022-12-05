# AWS KMS condition keys for AWS Nitro Enclaves<a name="conditions-nitro-enclaves"></a>

[AWS Nitro Enclaves](https://docs.aws.amazon.com/enclaves/latest/user/) is an Amazon EC2 capability that lets you create isolated compute environments called [enclaves](https://docs.aws.amazon.com/enclaves/latest/user/nitro-enclave-concepts.html#term-enclave) to protect and process highly sensitive data\. AWS KMS provides condition keys to support AWS Nitro Enclaves\. These conditions keys work only when a request for an AWS KMS operation originates in an enclave\. 

When you call the `kms-decrypt`, `kms-generate-data-key`, or `kms-generate-random` [AWS Nitro Enclaves SDK](https://github.com/aws/aws-nitro-enclaves-sdk-c) APIs from an enclave, these APIs call the corresponding AWS KMS operation with a parameter that includes a signed [attestation document](https://docs.aws.amazon.com/enclaves/latest/user/nitro-enclave-concepts.html#term-attestdoc) from the enclave\. The signed attestation document proves the enclave's identity to AWS KMS\. 

The following condition keys let you limit the permissions for these operations based on the contents of the signed attestation document\. Before allowing an operation, AWS KMS compares the attestation document from the enclave to the values in these AWS KMS condition keys\.

## kms:RecipientAttestation:ImageSha384<a name="conditions-kms-recipient-image-sha"></a>


| AWS KMS Condition Keys | Condition Type | Value type | API Operations | Policy Type | 
| --- | --- | --- | --- | --- | 
|  `kms:RecipientAttestation:ImageSha384`  |  String  | Single\-valued |  `Decrypt` `GenerateDataKey` `GenerateRandom`  |  Key policies and IAM policies  | 

The `kms:RecipientAttestation:ImageSha384` condition key allows `kms-decrypt`, `kms-generate-data-key`, and `kms-generate-random` requests from an enclave only when the image hash from the signed attestation document in the request matches the value in the condition key\. The `ImageSha384` value corresponds to PCR\[0\] in the attestation document\. This condition key is effective only when you call the AWS Nitro Enclaves SDK APIs from an enclave\.

**Note**  
This condition key is valid in key policy statements and IAM policy statements even though it does not appear in the IAM console or the IAM *Service Authorization Reference*\.

For example, the following key policy statement allows the `data-processing` role to use the KMS key for the `kms-decrypt` \([Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)\), `kms-generate-data-key` \([GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)\), and `kms-generate-random` \([GenerateRandom](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateRandom.html)\) operations\. The `kms:RecipientAttestation:ImageSha384` condition key allows the operations only when the image hash value \(PCR\[0\]\) of the attestation document in the request matches the image hash value in the condition\. 

If the request doesn't include any attestation document, permission is denied because this condition isn't satisfied\.

```
{
  "Sid" : "Enable enclave data processing",
  "Effect" : "Allow",
  "Principal" : {
    "AWS" : "arn:aws:iam::111122223333:role/data-processing"
  },
  "Action": [
    "kms:Decrypt",
    "kms:GenerateDataKey",
    "kms:GenerateRandom"
  ],
  "Resource" : "*",
  "Condition": {
    "StringEqualsIgnoreCase": {
      "kms:RecipientAttestation:ImageSha384": "9fedcba8abcdef7abcdef6abcdef5abcdef4abcdef3abcdef2abcdef1abcdef1abcdef0abcdef1abcdef2abcdef3abcdef4abcdef5abcdef6abcdef7abcdef99"
    }
  }
}
```

## kms:RecipientAttestation:PCR<PCR\_ID><a name="conditions-kms-recipient-pcrs"></a>


| AWS KMS Condition Keys | Condition Type | Value type | API Operations | Policy Type | 
| --- | --- | --- | --- | --- | 
|  `kms:RecipientAttestation:PCR`  |  String  | Single\-valued |  `Decrypt` `GenerateDataKey` `GenerateRandom`  |  Key policies and IAM policies  | 

The `kms:RecipientAttestation:PCR<PCR_ID>` condition key allows `kms-decrypt`, `kms-generate-data-key`, and `kms-generate-random` requests from an enclave only when the platform configuration registers \(PCRs\) from the signed attestation document in the request match the PCRs in the condition key\. This condition key is effective only when you call the AWS Nitro Enclaves SDK APIs from an enclave\.

**Note**  
This condition key is valid in key policy statements and IAM policy statements even though it does not appear in the IAM console or the IAM *Service Authorization Reference*\.

To specify a PCR value, use the following format\. Concatenate the PCR ID to the condition key name\. The PCR value must be a lower\-case hexadecimal string of up to 96 bytes\.

```
"kms:RecipientAttestation:PCRPCR_ID": "PCR_value"
```

For example, the following condition key specifies a particular value for PCR\[1\], which corresponds to the hash of the kernel used for the enclave and the bootstrap process\.

```
kms:RecipientAttestation:PCR1: "0x1abcdef2abcdef3abcdef4abcdef5abcdef6abcdef7abcdef8abcdef9abcdef8abcdef7abcdef6abcdef5abcdef4abcdef3abcdef2abcdef1abcdef0abcde"
```

The following example key policy statement allows the `data-processing` role to use the KMS key for the `kms-decrypt` \([Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)\) operation\.

The `kms:RecipientAttestation:PCR` condition key in this statement allows the operation only when the PCR1 value in the signed attestation document in the request matches `kms:RecipientAttestation:PCR1` value in the condition\. Use the `StringEqualsIgnoreCase` policy operator to require a case\-insensitive comparison of the PCR values\.

If the request doesn't include an attestation document, permission is denied because this condition isn't satisfied\.

```
{
  "Sid" : "Enable enclave data processing",
  "Effect" : "Allow",
  "Principal" : {
    "AWS" : "arn:aws:iam::111122223333:role/data-processing"
  },
  "Action": "kms:Decrypt",
  "Resource" : "*",
  "Condition": {
    "StringEqualsIgnoreCase": {
      "kms:RecipientAttestation:PCR1": "0x1de4f2dcf774f6e3b679f62e5f120065b2e408dcea327bd1c9dddaea6664e7af7935581474844767453082c6f1586116376cede396a30a39a611b9aad7966c87"
    }
  }
}
```