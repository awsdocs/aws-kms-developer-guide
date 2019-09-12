# Limits<a name="limits"></a>

AWS KMS resources have limits that apply to each region and each AWS account\. Some limits apply to all resources\. Others apply only to resources that you create, but not to resources that AWS services create in your account\. Resources that you use, but that aren't in your AWS account, such as [AWS owned CMKs](concepts.md#aws-owned-cmk), do not count against these limits\.

The limits for AWS KMS resources and the rate limits for AWS KMS API operations are described in this topic\. You can also view the AWS KMS limits for most AWS Regions in Service Quotas\. Use the [Service Quotas console](https://console.aws.amazon.com/servicequotas), or the [ListServiceQuotas](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/API_ListServiceQuotas.html) and [GetServiceQuota](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/API_GetServiceQuota.html) API operations\. For details, see [Viewing a Service Quota](https://docs.aws.amazon.com/servicequotas/latest/userguide//gs-request-quota.html) in the *Service Quotas User Guide*\.

**Important**  
If you need to exceed these limits, you can request a limit increase in Service Quotas\. Use the [Service Quotas console](https://console.aws.amazon.com/servicequotas) or the [RequestServiceQuotaIncrease](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/API_RequestServiceQuotaIncrease.html) API\. For details, see [Requesting a Quota Increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html) in the *Service Quotas User Guide*\. If Service Quotas for AWS KMS are not available in the AWS Region, please visit the [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case\.


| Resource | Default Limit | Applies To | 
| --- | --- | --- | 
| [Customer Master Keys \(CMKs\)](#customer-master-keys-limit) | 10,000 | Customer managed CMKs | 
| [Aliases](#aliases-limit) | 10,000 | Customer created aliases | 
| [Grants per CMK](#grants-per-key) | 10,000 | Customer managed CMKs | 
| [Grants for a given principal per CMK](#grants-per-principal-per-key) | 500 | Customer managed CMKsAWS managed CMKs | 
| [Key policy document size](#key-policy-limit) | 32 KB \(32,768 bytes\) | Customer managed CMKsAWS managed CMKs | 
| [Requests per second](#requests-per-second) | Varies by API operation; see [table](#requests-per-second-table)\. | Customer managed CMKsAWS managed CMKs | 

**Note**  
If you are exceeding the [requests per second](#requests-per-second) limit, consider using the [data key caching](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/data-key-caching.html) feature of the AWS Encryption SDK\. Reusing data keys, rather than requesting a new data key for every encryption operation, might reduce the frequency of your requests to AWS KMS\. 

## Customer Master Keys \(CMKs\): 10,000<a name="customer-master-keys-limit"></a>

You can have up to 10,000 [customer managed CMKs](concepts.md#customer-cmk) in each Region of your AWS account\. This limit applies to all customer managed CMKs regardless of their [key state](key-state.md)\. [AWS managed CMKs](concepts.md#aws-managed-cmk) and [AWS owned CMKs](concepts.md#aws-owned-cmk) do not count against this limit\. To request more CMKs in an AWS Region, use [Service Quotas](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html)\. If Service Quotas for AWS KMS are not available in the AWS Region, please visit the [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case\. 

However, managing a large number of CMKs from the AWS Management Console may be slower than acceptable\. If you have a large number of CMKs in a region, we recommend managing them programmatically with the [AWS SDKs](https://aws.amazon.com/tools/#sdk) or [AWS Command Line Tools](https://aws.amazon.com/tools/#cli)\.

## Aliases: 10,000<a name="aliases-limit"></a>

You can create up to 10,000 aliases in each region of your account\. Aliases that AWS creates in your account, such as aws/*<service\-name>*, do not count against this limit\. 

An *alias* is a display name that you can map to a CMK\. Each alias is mapped to exactly one CMK and multiple aliases can map to the same CMK\. 

If you increase your [CMK limit](#customer-master-keys-limit), such as by using [Service Quotas](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html) or by creating a case in the [AWS Support Center](https://console.aws.amazon.com/support/home), you might also need to request an increase in the number of aliases\.

## Grants per CMK: 10,000<a name="grants-per-key"></a>

Each [customer managed CMK](concepts.md#customer-cmk) can have up to 10,000 grants, including the grants created by [AWS services that are integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration)\. This limit does not apply to [AWS managed CMKs](concepts.md#aws-managed-cmk)\.

One effect of this limit is that you cannot perform more than 10,000 grant\-authorized operations that use the same CMK at the same time\. After you reach the limit, you can create new grants on the CMK only when an active grant is retired or revoked\.

For example, Amazon Elastic Block Store \(Amazon EBS\) uses a separate grant for permission to decrypt the data key for each Amazon EBS volume\. Therefore, you cannot attach more than 10,000 Amazon EBS volumes that use a given CMK to Amazon EC2 instances at one time\. 

[Grants](grants.md) are an alternative to [key policy](key-policies.md)\. They are advanced mechanisms for specifying permissions\. 

Each grant is attached to one CMK\. The grant includes the principal who receives permission to use the CMK, the ID of the CMK, and a list of operations that the grantee can perform\. You \(or [https://aws.amazon.com/kms/features/#AWS_Service_Integration](https://aws.amazon.com/kms/features/#AWS_Service_Integration)\) can use a grant to determine how and when the grantee can use a CMK\. 

## Grants for a Given Principal per CMK: 500<a name="grants-per-principal-per-key"></a>

For a given CMK, no more than 500 grants can specify the same grantee principal\. This limit applies to all CMKs, including [AWS managed CMKs](concepts.md#aws-managed-cmk)\.

For example, you might want to encrypt multiple Amazon EBS volumes and attach them to a single Amazon Elastic Compute Cloud \(Amazon EC2\) instance\. A unique grant is created for each encrypted volume and all of these grants have the same grantee principal \(an IAM assumed\-role user associated with the EC2 instance\)\. Each grant gives permission to use the specified CMK to decrypt an EBS volume's unique data encryption key\. For each CMK, you can have up to 500 grants that specify the same EC2 instance as the grantee principal\. This effectively means that you can have no more than 500 encrypted EBS volumes per EC2 instance for a given CMK\.

## Key Policy Document Size: 32 KB<a name="key-policy-limit"></a>

The maximum length of each key policy document is 32 KB \(32,768 bytes\)\. If the document exceeds this length, operations that use the key policy document to set or change the key policy fail\. If you must exceed this limit, you can request a limit increase by using [Service Quotas](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html)\. If Service Quotas for AWS KMS are not available in the AWS Region, please visit the [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case\.

A [key policy document](key-policies.md#key-policy-overview) is a collection of policy statements in JSON format\. The statements in the key policy document determine who has permission to use the CMK and how they can use it\. You may also use IAM policies and grants to control access to the CMK, but every CMK must have a key policy document\. 

You can create a key policy document by using the [default view](key-policy-modifying.md#key-policy-modifying-how-to-console-default-view) or [policy view](key-policy-modifying.md#key-policy-modifying-how-to-console-policy-view) in the AWS Management Console, or by using the [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) API operation\. All of these techniques involve an underlying key policy document\.

## Requests per Second: Varies<a name="requests-per-second"></a>

AWS KMS throttles API requests at different limits depending on the API operation\. Throttling means that AWS KMS rejects an otherwise valid request because the request exceeds the limit for the number of requests per second\. When a request is throttled, AWS KMS returns a `ThrottlingException` error\. [The following table](#requests-per-second-table) lists each API operation and the point at which AWS KMS throttles requests for that operation\.

This limit applies to all CMKs, including [AWS managed CMKs](concepts.md#aws-managed-cmk)\.

**Note**  
If you must exceed this limit, you can request a limit increase by using [Service Quotas](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html)\. If Service Quotas for AWS KMS are not available in the AWS Region, please visit the [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case\.  
If you are exceeding the requests per second limit for the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) API operation, consider using the [data key caching](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/data-key-caching.html) feature of the AWS Encryption SDK\. Reusing data keys might reduce the frequency of your requests to AWS KMS\. 

### Shared Limit<a name="rps-shared-limit"></a>

The API operations in the first row of the following table share a limit of 5,500 \(or 10,000\) requests per second\. For example, with a shared limit of 5,500 requests per second, when you make 3,000 `GenerateDataKey` requests per second and 1,000 `Decrypt` requests per second, AWS KMS doesn't throttle your requests\. However, when you make 5,000 `GenerateDataKey` and 1,000 `Encrypt` and requests per second, AWS KMS throttles your requests because you are making more than 5,500 requests per second for operations with the shared limit\.

The remaining API operations have a unique limit for requests per second, which means the limit is not shared\.

### API Requests Made on Your Behalf<a name="rps-from-service"></a>

You can make API requests directly or by using an integrated AWS service that makes API requests to AWS KMS on your behalf\. The limit applies to both kinds of requests\.

For example, you might store data in Amazon S3 using server\-side encryption with AWS KMS \(SSE\-KMS\)\. Each time you upload or download an S3 object that's encrypted with SSE\-KMS, Amazon S3 makes a `GenerateDataKey` \(for uploads\) or `Decrypt` \(for downloads\) request to AWS KMS on your behalf\. These requests count toward your limit, so AWS KMS throttles the requests if you exceed a combined total of 5,500 \(or 10,000\) uploads or downloads per second of S3 objects encrypted with SSE\-KMS\.

### Cross\-Account Requests<a name="rps-cross-account"></a>

When an application in one AWS account uses a CMK owned by a different account, that's known as a cross\-account request\. For cross\-account requests, AWS KMS throttles the account that makes the requests, not the account that owns the CMK\. For example, you might have applications in accounts A and B that both use a CMK in account C\. In this scenario, the limit for requests per second applies separately to accounts A and B, not to account C\.

### Custom Key Store Limits<a name="rps-key-stores"></a>

Cryptographic operations that use CMKs in a [custom key store](custom-key-store-overview.md) share a throttle limit of 1,800 operations per second for each custom key store\. However, not all operations use the limit equally\. The `GenerateDataKey`, `GenerateDataKeyWithoutPlaintext`, and `GenerateRandom` operations use approximately three times as much of the per\-second limit as the `Encrypt`, `Decrypt`, and `ReEncrypt` operations\.

For example, if you are requesting only `Encrypt` and `Decrypt` operations, you can perform approximately 1,800 operations per second\. If, instead, you request repeated `GenerateDataKey` operations, your performance might be closer to 600 operations per second\. For applications patterns that consist of roughly equal numbers of `GenerateDataKey` and `Decrypt` operations, you can expect about 1,200 operations per second\.

Unlike other AWS KMS limits, you cannot raise this limit by using Service Quotas or by creating a case in AWS Support\.

**Note**  
If the AWS CloudHSM cluster that is associated with the custom key store is processing numerous commands, including those unrelated to the custom key store, you might get an AWS KMS `ThrottlingException` at a lower\-than\-expected rate\. If this occurs, lower your request rate to AWS KMS, reduce the unrelated load, or use a dedicated AWS CloudHSM cluster for your custom key store\.

### Requests\-per\-Second Limit for Each AWS KMS API Operation<a name="rps-table"></a>


| API Operation | Requests\-perSecond Limit | 
| --- | --- | 
|  `Decrypt` `Encrypt` `GenerateDataKey` `GenerateDataKeyWithoutPlaintext` `GenerateRandom` `ReEncrypt`  | 5,500 \(shared\)10,000 \(shared\) only in the following regions:[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/limits.html)1,800 \(shared\) for each custom key store\. For details, see [Custom Key Store Limits](#rps-key-stores)\. | 
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
| DescribeKey | 30 | 
| DisableKey | 5 | 
| DisableKeyRotation | 5 | 
| DisconnectCustomKeyStore | 5 | 
| EnableKey | 5 | 
| EnableKeyRotation | 5 | 
| GetKeyPolicy | 30 | 
| GetKeyRotationStatus | 30 | 
| GetParametersForImport | 0\.25 \(AWS KMS throttles requests when the rate is more than 1 per 4 seconds\) | 
| ImportKeyMaterial | 5 | 
| ListAliases | 5 | 
| ListGrants | 5 | 
| ListKeyPolicies | 5 | 
| ListKeys | 5 | 
| ListResourceTags | 5 | 
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