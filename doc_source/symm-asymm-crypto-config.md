# Viewing the cryptographic configuration of CMKs<a name="symm-asymm-crypto-config"></a>

After you create your CMK, you can view its cryptographic configuration\. You cannot change the configuration of a CMK after it is created\. If you prefer a different configuration, delete the CMK and create it again\.

You can find the cryptographic configuration of your CMKs, include the key spec, key usage, and supported encryption or signing algorithms, in the AWS KMS console or by using the AWS KMS API\. For details, see [Identifying symmetric and asymmetric CMKs](find-symm-asymm.md)\.

In the AWS KMS console, the details page for each CMK includes a **Cryptographic configuration** section that displays cryptographic details about your CMKs\. For example, the following image shows the **Cryptographic configuration** section for an RSA CMK used for signing and verification\.

![\[Generate a data key\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-cryptographic-configuration.png)

In the AWS KMS API, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. The `KeyMetadata` structure in the response includes the cryptographic configuration of the CMK\. For example, `DescribeKey` returns the following response for an RSA CMK used for signing and verification\.

```
{
  "KeyMetadata": {
    "AWSAccountId": "111122223333",
    "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
    "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
    "CreationDate": 1571767572.317,
    "Enabled": false,
    "Description": "",
    "KeyUsage": "SIGN_VERIFY",
    "KeyState": "Disabled",
    "Origin": "AWS_KMS",
    "KeyManager": "CUSTOMER",
    "CustomerMasterKeySpec": "RSA_2048",
    "SigningAlgorithms": [
        "RSASSA_PKCS1_V1_5_SHA_256",
        "RSASSA_PKCS1_V1_5_SHA_384",
        "RSASSA_PKCS1_V1_5_SHA_512",
        "RSASSA_PSS_SHA_256",
        "RSASSA_PSS_SHA_384",
        "RSASSA_PSS_SHA_512"
    ]
  }
}
```