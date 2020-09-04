# Request quotas<a name="requests-per-second"></a>

AWS KMS establishes quotas for the number of API operations requested in each second\. For a table that lists the per\-second request quota for each API operation, see [Request quotas for each AWS KMS API operation](#rps-table)\.

When you exceed an API request quota, AWS KMS *throttles* the request, that is, it rejects an otherwise valid request and returns a `ThrottlingException` error like the following one\. To respond, use a [backoff and retry strategy](https://docs.aws.amazon.com/general/latest/gr/api-retries.html)\. 

```
You have exceeded the rate at which you may call KMS. Reduce the frequency of your calls. 
(Service: AWSKMS; Status Code: 400; Error Code: ThrottlingException; Request ID: <ID>
```

The request quotas differ with the API operation, the AWS Region, and other factors, such as the CMK type\.

**Note**  
If you need to exceed a quota, you can request a quota increase in Service Quotas\. Use the [Service Quotas console](https://console.aws.amazon.com/servicequotas) or the [RequestServiceQuotaIncrease](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/API_RequestServiceQuotaIncrease.html) operation\. For details, see [Requesting a quota increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html) in the *Service Quotas User Guide*\. If Service Quotas for AWS KMS are not available in the AWS Region, please visit the [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case\.   
For help requesting an increase in an AWS KMS quota, see [Request an AWS KMS Quota Increase](increase-quota.md)\.  
If you are exceeding the request quota for the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation, consider using the [data key caching](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/data-key-caching.html) feature of the AWS Encryption SDK\. Reusing data keys might reduce the frequency of your requests to AWS KMS\. 

In addition to request quotas, AWS KMS uses resource quotas to ensure capacity for all users\. For details, see [Resource quotas](resource-limits.md)\.

**Topics**
+ [Applying request quotas](#about-rate-limits)
+ [Shared quotas for cryptographic operations](#rps-shared-limit)
+ [API requests made on your behalf](#rps-from-service)
+ [Cross\-account requests](#rps-cross-account)
+ [Custom key store quota](#rps-key-stores)
+ [Request quotas for each AWS KMS API operation](#rps-table)

## Applying request quotas<a name="about-rate-limits"></a>

When reviewing request quotas, keep in mind the following information\.
+ Request quotas apply to both [customer managed CMKs](concepts.md#customer-cmk) and [AWS managed CMKs](concepts.md#aws-managed-cmk)\. The use of [AWS owned CMKs](concepts.md#aws-owned-cmk) does not count against request quotas for your AWS account, even when they are used to protect resources in your account\.
+ Throttling is based on all requests on CMKs of all types in the Region\. This total includes requests from all principals in the AWS account, including requests from AWS services on your behalf\.
+ Each request quota is calculated independently\. For example, requests for the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation have no effect on the request quota for the [CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html) operation\. If your `CreateAlias` requests are throttled, your `CreateKey` requests can still complete successfully\.
+ Although cryptographic operations share a quota, the shared quota is calculated independently of quotas for other operations\. For example, calls to the [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operations share a request quota, but that quota is independent of the quota for management operations, such as [EnableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKey.html)\. For example, in the Europe \(London\) Region, you can perform 10,000 cryptographic operations on symmetric CMKs *plus* 5 `EnableKey` operations per second without being throttled\.

## Shared quotas for cryptographic operations<a name="rps-shared-limit"></a>

AWS KMS [cryptographic operations](concepts.md#cryptographic-operations) share request quotas\. You can request any combination of the cryptographic operations that are supported by the CMK, just so the total number of cryptographic operations doesn't exceed the request quota for that type of CMK\. The exceptions are [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html) and [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html), which share a separate quota\. 

The quotas for different types of CMKs are calculated independently\. Each quota applies to all requests for these operations in the AWS account and Region with the given key type in each one\-second interval\.
+ *Cryptographic operations \(symmetric\) request rate* is the shared request quota for cryptographic operations using symmetric CMKs in an account and region\.

  For example, you might be using [symmetric CMKs](symm-asymm-concepts.md#symmetric-cmks) in an AWS Region with a shared quota of 10,000 requests per second\. When you make 7,000 [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) requests per second and 2,000 [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) requests per second, AWS KMS doesn't throttle your requests\. However, when you make 9,500 `GenerateDataKey` requests and 1,000 [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) and requests per second, AWS KMS throttles your requests because they exceed the shared quota\.
+ *Cryptographic operations \(RSA\) request rate* is the shared request quota for cryptographic operations using [RSA asymmetric CMKs](symm-asymm-choose.md#key-spec-rsa)\. 

  For example, with a request quota of 500 operations per second, you can make 200 [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) requests and 100 [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) requests with RSA CMKs that can encrypt and decrypt, plus 50 [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html) requests and 150 [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html) requests with RSA CMKs that can sign and verify\.
+ *Cryptographic operations \(ECC\) request rate* is the shared request quota for cryptographic operations using [elliptic curve \(ECC\) asymmetric CMKs](symm-asymm-choose.md#key-spec-ecc)\. 

  For example, with a request quota of 300 operations per second, you can make 100 Sign requests and 200 Verify requests with RSA CMKs that can sign and verify\.

The quotas for different key types are also calculated independently\. For example, in the Asia Pacific \(Singapore\) Region, if you use both symmetric and asymmetric CMKs, you can make up to 10,000 calls per second with symmetric CMKs *plus* up to 500 additional calls per second with your RSA asymmetric CMKs, *plus* up to 300 additional requests per second with your ECC\-based CMKs\.

## API requests made on your behalf<a name="rps-from-service"></a>

You can make API requests directly or by using an integrated AWS service that makes API requests to AWS KMS on your behalf\. The quota applies to both kinds of requests\.

For example, you might store data in Amazon S3 using server\-side encryption with AWS KMS \(SSE\-KMS\)\. Each time you upload or download an S3 object that's encrypted with SSE\-KMS, Amazon S3 makes a `GenerateDataKey` \(for uploads\) or `Decrypt` \(for downloads\) request to AWS KMS on your behalf\. These requests count toward your quota, so AWS KMS throttles the requests if you exceed a combined total of 5,500 \(or 10,000 or 30,000 depending upon your AWS Region\) uploads or downloads per second of S3 objects encrypted with SSE\-KMS\.

## Cross\-account requests<a name="rps-cross-account"></a>

When an application in one AWS account uses a CMK owned by a different account, it's known as a *cross\-account request*\. For cross\-account requests, AWS KMS throttles the account that makes the requests, not the account that owns the CMK\. For example, if an application in account A uses a CMK in account B, the CMK use applies only to the quotas in account A\.

## Custom key store quota<a name="rps-key-stores"></a>

AWS KMS custom key stores support only symmetric CMKs\. The cryptographic operations that use the CMKs in a [custom key store](custom-key-store-overview.md) share a request quota of 1,800 operations per second for each custom key store\. However, not all operations use the quota equally\. The `GenerateDataKey`, `GenerateDataKeyWithoutPlaintext`, and `GenerateRandom` operations use approximately three times as much of the per\-second quota as the `Encrypt`, `Decrypt`, and `ReEncrypt` operations\.

For example, if you are requesting only `Encrypt` and `Decrypt` operations, you can perform approximately 1,800 operations per second\. If, instead, you request repeated `GenerateDataKey` operations, your performance might be closer to 600 operations per second\. For applications patterns that consist of roughly equal numbers of `GenerateDataKey` and `Decrypt` operations, you can expect about 1,200 operations per second\.

Unlike other AWS KMS quotas, the custom key store quota is not adjustable\. You cannot increase it by using Service Quotas or by creating a case in AWS Support\.

**Note**  
If the AWS CloudHSM cluster that is associated with the custom key store is processing numerous commands, including those unrelated to the custom key store, you might get an AWS KMS `ThrottlingException` at a lower\-than\-expected rate\. If this occurs, lower your request rate to AWS KMS, reduce the unrelated load, or use a dedicated AWS CloudHSM cluster for your custom key store\.

## Request quotas for each AWS KMS API operation<a name="rps-table"></a>

This table lists the [Service Quotas](https://docs.aws.amazon.com/servicequotas/latest/userguide/) quota code and the default value for each AWS KMS request quota\.


| Quota name | Default value \(per second\) | 
| --- | --- | 
|  `Cryptographic operations (symmetric) request rate` Applies to: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/requests-per-second.html)  |  These shared quotas vary with the AWS Region and the type of CMK used in the request\. Each quota is calculated separately\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/requests-per-second.html) Custom key stores quota \(symmetric CMKs\): [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/requests-per-second.html)  | 
| `Cryptographic operations (RSA) request rate` Applies to:[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/requests-per-second.html) |  500 \(shared\) for RSA CMKs  | 
| `Cryptographic operations (ECC) request rate` Applies to:[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/requests-per-second.html) |  300 \(shared\) for elliptic curve \(ECC\) CMKs  | 
| `CancelKeyDeletion request rate` | 5 | 
| `ConnectCustomKeyStore request rate` | 5 | 
| `CreateAlias request rate` | 5 | 
| `CreateCustomKeyStore request rate` | 5 | 
| `CreateGrant request rate` | 50 | 
| `CreateKey request rate` | 5 | 
| `DeleteAlias request rate` | 5 | 
| `DeleteCustomKeyStore request rate` | 5 | 
| `DeleteImportedKeyMaterial request rate` | 5 | 
| `DescribeCustomKeyStores request rate` | 5 | 
| `DescribeKey request rate` | 1000 | 
| `DisableKey request rate` | 5 | 
| `DisableKeyRotation request rate` | 5 | 
| `DisconnectCustomKeyStore request rate` | 5 | 
| `EnableKey request rate` | 5 | 
| `EnableKeyRotation request rate` | 5 | 
|  `GenerateDataKeyPair (ECC_NIST_P256) request rate` Applies to: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/requests-per-second.html)  |  25  | 
|  `GenerateDataKeyPair (ECC_NIST_P384) request rate` Applies to: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/requests-per-second.html)  |  10  | 
|  `GenerateDataKeyPair (ECC_NIST_P521) request rate` Applies to: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/requests-per-second.html)  |  5  | 
|  `GenerateDataKeyPair (ECC_SECG_P256K1) request rate` Applies to: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/requests-per-second.html)  |  25  | 
|  `GenerateDataKeyPair (RSA_2048) request rate` Applies to: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/requests-per-second.html)  |  1  | 
|  `GenerateDataKeyPair (RSA_3072) request rate` Applies to: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/requests-per-second.html)  |  0\.5 \(1 in each 2\-second interval\)  | 
|  `GenerateDataKeyPair (RSA_4096) request rate` Applies to: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/requests-per-second.html)  |  0\.1 \(1 in each 10\-second interval\)  | 
| `GetKeyPolicy request rate` | 1000 | 
| `GetKeyRotationStatus request rate` | 1000 | 
| `GetParametersForImport request rate` | 0\.25 \(1 in each 4\-second interval\) | 
| `GetPublicKey request rate` | 5 | 
| `ImportKeyMaterial request rate` | 5 | 
| `ListAliases request rate` | 100 | 
| `ListGrants request rate` | 100 | 
| `ListKeyPolicies request rate` | 100 | 
| `ListKeys request rate` | 100 | 
| `ListResourceTags request rate` | 100 | 
| `ListRetirableGrants request rate` | 5 | 
| `PutKeyPolicy request rate` | 5 | 
| `RetireGrant request rate` | 15 | 
| `RevokeGrant request rate` | 15 | 
| `ScheduleKeyDeletion request rate` | 5 | 
| `TagResource request rate` | 5 | 
| `UntagResource request rate` | 5 | 
| `UpdateAlias request rate` | 5 | 
| `UpdateCustomKeyStore request rate` | 5 | 
| `UpdateKeyDescription request rate` | 5 | 