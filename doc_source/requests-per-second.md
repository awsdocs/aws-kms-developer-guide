# Rate Limits<a name="requests-per-second"></a>

AWS KMS establishes limits on the number of API operations requested in each second\. For a table that lists the per\-second rate limit for each API operation, see [Rate Limits for Each AWS KMS API Operation](#rps-table)\.

When you exceed an API rate limit, AWS KMS *throttles* the request, that is, it rejects an otherwise valid request and returns a `ThrottlingException` error like the following one\. To respond, use a [backoff and retry strategy](https://docs.aws.amazon.com/general/latest/gr/api-retries.html)\. 

```
You have exceeded the rate at which you may call KMS. Reduce the frequency of your calls. 
(Service: AWSKMS; Status Code: 400; Error Code: ThrottlingException; Request ID: <ID>
```

The rate limits differ with the API operation, the AWS Region, and other factors, such as the CMK type\.

**Note**  
If you need to exceed these limits, you can request a limit increase in Service Quotas\. Use the [Service Quotas console](https://console.aws.amazon.com/servicequotas) or the [RequestServiceQuotaIncrease](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/API_RequestServiceQuotaIncrease.html) operation\. For details, see [Requesting a Quota Increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html) in the *Service Quotas User Guide*\. If Service Quotas for AWS KMS are not available in the AWS Region, please visit the [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case\.   
If you are exceeding the rate limit for the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation, consider using the [data key caching](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/data-key-caching.html) feature of the AWS Encryption SDK\. Reusing data keys might reduce the frequency of your requests to AWS KMS\. 

In addition to rate limits, AWS KMS uses resource limits to ensure capacity for all users\. For details, see [Resource Limits](resource-limits.md)\.

**Topics**
+ [Applying Rate Limits](#about-rate-limits)
+ [Shared Limits for Cryptographic Operations](#rps-shared-limit)
+ [API Requests Made on Your Behalf](#rps-from-service)
+ [Cross\-Account Requests](#rps-cross-account)
+ [Custom Key Store Limits](#rps-key-stores)
+ [Rate Limits for Each AWS KMS API Operation](#rps-table)

## Applying Rate Limits<a name="about-rate-limits"></a>

When reviewing rate limits, keep in mind the following information\.
+ Rate limits apply to both [customer managed CMKs](concepts.md#customer-cmk) and [AWS managed CMKs](concepts.md#aws-managed-cmk)\. The use of [AWS owned CMKs](concepts.md#aws-owned-cmk) does not count against rate limits for your AWS account, even when they are used to protect resources in your account\.

   
+ Throttling is based on all requests on CMKs of all types in the Region\. This total includes requests from all principals in the AWS account, including requests from AWS services on your behalf\.

   
+ Each rate limit is calculated independently\. For example, requests for the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation have no effect on the limit of requests to the [CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html) operation\. If your `CreateAlias` requests are throttled, your `CreateKey` requests can still complete successfully\.

   
+ Although cryptographic operations share a limit, the shared limit is calculated independently of limits for other operations\. For example, calls to the [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operations share a rate limit, but that limit is independent of the limit for management operations, such as [EnableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKey.html)\. For example, in the Europe \(London\) Region, you can perform 10,000 cryptographic operations on symmetric CMKs *plus* 5 `EnableKey` operations per second without being throttled\.

## Shared Limits for Cryptographic Operations<a name="rps-shared-limit"></a>

AWS KMS cryptographic operations share throttle limits\. These limits are displayed in the first row of the [Rate Limit table](#rps-table)\. The limits for different types of CMKs are calculated independently\. Each limit applies to all requests for these operations in the AWS account and Region with the given key type in each one\-second interval\.

For example, you might be using [symmetric CMKs](symm-asymm-concepts.md#symmetric-cmks) in an AWS Region with a shared limit of 10,000 requests per second\. When you make 7,000 [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) requests per second and 2,000 [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) requests per second, AWS KMS doesn't throttle your requests\. However, when you make 9,500 `GenerateDataKey` requests and 1,000 [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) and requests per second, AWS KMS throttles your requests because they exceed the shared limit\.

Similarly, if you are using [asymmetric CMKs](symm-asymm-concepts.md#asymmetric-cmks), you can request any combination of the cryptographic operations that are supported by the CMK, just so the total number of cryptographic operations doesn't exceed the requests\-per\-second limit for that type of CMK\. For example, you can make 300 [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) requests and 200 [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) requests on your RSA\-based encryption CMKs without being throttled\.

**Note**  
Asymmetric CMKs and asymmetric data key pairs are supported by AWS KMS only in the following AWS Regions: US East \(N\. Virginia\), US West \(Oregon\), Asia Pacific \(Sydney\), Asia Pacific \(Tokyo\), and Europe \(Ireland\)\.

The limits for different key types are also calculated independently\. For example, in the Asia Pacific \(Singapore\) Region, if you use both symmetric and asymmetric CMKs, you can make up to 10,000 calls per second with symmetric CMKs *plus* up to 500 additional calls per second with your RSA\-based asymmetric CMKs, *plus* up to 300 additional requests per second with your ECC\-based CMKs\.

## API Requests Made on Your Behalf<a name="rps-from-service"></a>

You can make API requests directly or by using an integrated AWS service that makes API requests to AWS KMS on your behalf\. The limit applies to both kinds of requests\.

For example, you might store data in Amazon S3 using server\-side encryption with AWS KMS \(SSE\-KMS\)\. Each time you upload or download an S3 object that's encrypted with SSE\-KMS, Amazon S3 makes a `GenerateDataKey` \(for uploads\) or `Decrypt` \(for downloads\) request to AWS KMS on your behalf\. These requests count toward your limit, so AWS KMS throttles the requests if you exceed a combined total of 5,500 \(or 10,000 or 30,000 depending upon your AWS Region\) uploads or downloads per second of S3 objects encrypted with SSE\-KMS\.

## Cross\-Account Requests<a name="rps-cross-account"></a>

When an application in one AWS account uses a CMK owned by a different account, that's known as a *cross\-account request*\. For cross\-account requests, AWS KMS throttles the account that makes the requests, not the account that owns the CMK\. For example, if an application in account A uses a CMK in account B, the CMK use is applied only to the limits in account A\.

## Custom Key Store Limits<a name="rps-key-stores"></a>

Cryptographic operations that use CMKs in a [custom key store](custom-key-store-overview.md) share a throttle limit of 1,800 operations per second for each custom key store\. However, not all operations use the limit equally\. The `GenerateDataKey`, `GenerateDataKeyWithoutPlaintext`, and `GenerateRandom` operations use approximately three times as much of the per\-second limit as the `Encrypt`, `Decrypt`, and `ReEncrypt` operations\.

For example, if you are requesting only `Encrypt` and `Decrypt` operations, you can perform approximately 1,800 operations per second\. If, instead, you request repeated `GenerateDataKey` operations, your performance might be closer to 600 operations per second\. For applications patterns that consist of roughly equal numbers of `GenerateDataKey` and `Decrypt` operations, you can expect about 1,200 operations per second\.

Unlike other AWS KMS limits, you cannot raise this limit by using Service Quotas or by creating a case in AWS Support\.

**Note**  
If the AWS CloudHSM cluster that is associated with the custom key store is processing numerous commands, including those unrelated to the custom key store, you might get an AWS KMS `ThrottlingException` at a lower\-than\-expected rate\. If this occurs, lower your request rate to AWS KMS, reduce the unrelated load, or use a dedicated AWS CloudHSM cluster for your custom key store\.

## Rate Limits for Each AWS KMS API Operation<a name="rps-table"></a>


| API Operation | Rate Limits \(per second\) | 
| --- | --- | 
|  `Decrypt` `Encrypt` `GenerateDataKey` \(symmetric\) `GenerateDataKeyWithoutPlaintext` \(symmetric\) `GenerateRandom` `ReEncrypt` `Sign` `Verify`  |  These shared limits vary with the AWS Region and the type of CMK used in the request\. Each limit is calculated separately\. Symmetric CMK limit: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/requests-per-second.html) Asymmetric CMK limit:  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/requests-per-second.html) Custom key stores limit: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/requests-per-second.html)  | 
| CancelKeyDeletion | 5 | 
| ConnectCustomKeyStore | 5 | 
| CreateAlias | 5 | 
| CreateCustomKeyStore | 5 | 
| CreateGrant | 50 | 
| CreateKey | 5 | 
| DeleteAlias | 5 | 
| DeleteCustomKeyStore | 5 | 
| DeleteImportedKeyMaterial | 5 | 
| DescribeCustomKeyStores | 5 | 
| DescribeKey | 100 | 
| DisableKey | 5 | 
| DisableKeyRotation | 5 | 
| DisconnectCustomKeyStore | 5 | 
| EnableKey | 5 | 
| EnableKeyRotation | 5 | 
| `GenerateDataKeyPair``GenerateDataKeyPairWithoutPlaintext` |  These shared limits vary with the type of data keys that are requested\. Each is calculated separately\.1 — RSA\_20480\.5 — RSA\_3072 \(1 in a 2\-second interval\)0\.1 — RSA\_4096 \(1 in a 10\-second interval\)25 — ECC\_NIST\_P25610 — ECC\_NIST\_P3845 — ECC\_NIST\_P52125 — ECC\_SECG\_P256K1 | 
| GetKeyPolicy | 30 | 
| GetKeyRotationStatus | 30 | 
| GetParametersForImport | 0\.25 \(1 in a 4\-second interval\) | 
| GetPublicKey | 5 | 
| ImportKeyMaterial | 5 | 
| ListAliases | 100 | 
| ListGrants | 100 | 
| ListKeyPolicies | 100 | 
| ListKeys | 100 | 
| ListResourceTags | 100 | 
| ListRetirableGrants | 5 | 
| PutKeyPolicy | 5 | 
| RetireGrant | 15 | 
| RevokeGrant | 15 | 
| ScheduleKeyDeletion | 5 | 
| TagResource | 5 | 
| UntagResource | 5 | 
| UpdateAlias | 5 | 
| UpdateCustomKeyStore | 5 | 
| UpdateKeyDescription | 5 | 