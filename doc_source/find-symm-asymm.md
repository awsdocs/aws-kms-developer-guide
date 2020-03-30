# Identifying symmetric and asymmetric CMKs<a name="find-symm-asymm"></a>

To determine whether a particular CMK is [symmetric or asymmetric](symmetric-asymmetric.md), find its *key type* or [key spec](concepts.md#key-spec)\. You can use the AWS KMS console or AWS KMS API\. 

Some of these methods will also show you other aspects of the cryptographic configuration of a CMK, including its key usage and the encryption or signing algorithms that the CMK supports\. You can view the cryptographic configuration of an existing CMK, but you cannot change it\.

For general information about viewing CMKs, including sorting, filtering, and choosing columns for your console display, see [Viewing CMKs in the console](viewing-keys-console.md)\.

**Topics**
+ [Finding the key type in the CMK table](#find-key-type-table)
+ [Finding the key type on the details page](#find-key-type-details)
+ [Finding the key spec using the AWS KMS API](#find-key-type-api)

## Finding the key type in the CMK table<a name="find-key-type-table"></a>

In the AWS KMS console, the **Key type** column shows whether each CMK is symmetric or asymmetric\. You can add a **Key type** column to the CMK table on the **Customer managed keys** or **AWS managed keys** pages in the console\.

To identify symmetric and asymmetric CMKs in your CMK table, use the following procedure\.

1. Open the AWS KMS console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. To view the keys in your account that you create and manage, in the navigation pane choose **Customer managed keys**\. To view the keys in your account that AWS creates and manages for you, in the navigation pane, choose **AWS managed keys**\.

1. The **Key type** columns shows whether each CMK is symmetric or asymmetric\. You can also [sort and filter](viewing-keys-console.md#viewing-console-filter) by the **Key type** value\. 

   If the **Key type** column does not appear in your CMK table, choose the gear icon in the upper right corner of the page, choose **Key type**, and then choose **Confirm**\. You can also add the **Key spec** and **Key usage** columns\.  
![\[The Key type column in a CMK table\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-table-key-type.png)

## Finding the key type on the details page<a name="find-key-type-details"></a>

In the AWS KMS console, the details page for each CMK includes a **Cryptographic Configuration** section that displays the key type \(symmetric or asymmetric\) and other cryptographic details about the CMK\. 

To identify symmetric and asymmetric CMKs on the details page for a CMK, use the following procedure\.

1. Open the AWS KMS console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. To view the keys in your account that you create and manage, in the navigation pane choose **Customer managed keys**\. To view the keys in your account that AWS creates and manages for you, in the navigation pane, choose **AWS managed keys**\.

1. Choose the alias or key ID of a CMK\.

1. Choose **Cryptographic configuration**\.

   The **Cryptographic configuration** section includes the **Key Type**, which indicates whether it is symmetric or asymmetric\. It also displays other details about the CMK, including the **Key Usage**, which tells whether a CMK can be used for encryption and decryption or signing and verification\. For asymmetric CMKs, it displays the encryption algorithms or signing algorithms that the CMK supports\.

   For example, the following is an example **Cryptographic configuration** section for a symmetric CMK\.  
![\[The Cryptographic configuration section for a symmetric CMK\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-cryptographic-config-symmetric.png)

   The following is an example **Cryptographic configuration** section for an asymmetric RSA CMK that's used for signing and verification\.  
![\[The Cryptographic configuration section for an asymmetric CMK\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-cryptographic-configuration.png)

## Finding the key spec using the AWS KMS API<a name="find-key-type-api"></a>

To determine whether a CMK is symmetric or asymmetric, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. The `CustomerMasterKeySpec` field in the response contains the [key spec](concepts.md#key-spec) of the CMK\. For a symmetric CMK, the value of `CustomerMasterKeySpec` is `SYMMETRIC_DEFAULT`\. All other values indicate an asymmetric CMK\.

For example, `DescribeKey` returns the following response for a symmetric CMK\. The `CustomerMasterKeySpec` value is `SYMMETRIC_DEFAULT`\.

```
{
  "KeyMetadata": {
    "AWSAccountId": "111122223333",
    "KeyId": "0987dcba-09fe-87dc-65ba-ab0987654321",
    "Arn": "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321",
    "CreationDate": 1496966810.831,
    "Enabled": true,
    "Description": "",
    "KeyState": "Enabled",
    "Origin": "AWS_KMS",
    "KeyManager": "CUSTOMER",
    "CustomerMasterKeySpec": "SYMMETRIC_DEFAULT",
    "KeyUsage": "ENCRYPT_DECRYPT",
    "EncryptionAlgorithms": [
        "SYMMETRIC_DEFAULT"
    ]
  }
}
```

The `DescribeKey` response for an asymmetric RSA CMK used in signing and verification looks similar to this example\. The key spec, as shown in the `CustomerMasterKeySpec` value is [RSA\_2048](symm-asymm-choose.md#key-spec-rsa)\. The `KeyUsage` is `SIGN_VERIFY`, and the `SigningAlgorithms` element lists the valid signing algorithms for the CMK\.

```
{
  "KeyMetadata": {
    "AWSAccountId": "111122223333",
    "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
    "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
    "CreationDate": 1571767572.317,
    "Enabled": false,
    "Description": "",
    "KeyState": "Disabled",
    "Origin": "AWS_KMS",
    "KeyManager": "CUSTOMER",
    "CustomerMasterKeySpec": "RSA_2048",
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