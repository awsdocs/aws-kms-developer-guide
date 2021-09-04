# Viewing the cryptographic configuration of KMS keys<a name="symm-asymm-crypto-config"></a>

After you create your KMS key, you can view its cryptographic configuration\. You cannot change the configuration of a KMS key after it is created\. If you prefer a different configuration, delete the KMS key and create it again\.

You can find the cryptographic configuration of your KMS keys, include the key spec, key usage, and supported encryption or signing algorithms, in the AWS KMS console or by using the AWS KMS API\. For details, see [Identifying symmetric and asymmetric KMS keys](find-symm-asymm.md)\.

In the AWS KMS console, the [details page for each KMS key](viewing-keys-console.md#viewing-console-details) includes a **Cryptographic configuration** tab that displays cryptographic details about your KMS keys\. For example, the following image shows the **Cryptographic configuration** tab for an RSA KMS key used for signing and verification\.

![\[Generate a data key\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-cryptographic-configuration.png)

In the AWS KMS API, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. The `KeyMetadata` structure in the response includes the cryptographic configuration of the KMS key\. For example, `DescribeKey` returns the following response for an RSA KMS key used for signing and verification\.

```
{
  "KeyMetadata": {

    "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
    "AWSAccountId": "111122223333",
    "CreationDate": 1571767572.317,
    "CustomerMasterKeySpec": "RSA_2048",
    "Description": "",
    "Enabled": true,
    "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
    "KeyManager": "CUSTOMER",
    "KeyState": "Enabled",
    "MultiRegion": false, 
    "Origin": "AWS_KMS",
    "KeySpec": "RSA_2048",
    "KeyUsage": "SIGN_VERIFY",
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