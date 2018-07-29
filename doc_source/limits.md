# Limits<a name="limits"></a>

AWS KMS objects have limits that apply to each region and each AWS account\. Some limits apply to all objects\. Others apply only to objects that you create, but not to objects that AWS services create in your account\. 

**Important**  
If you need to exceed these limits, please visit the [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case\. 


| Resource | Default Limit | Applies To | 
| --- | --- | --- | 
| [Customer Master Keys \(CMKs\)](#customer-master-keys-limit) | 1000 | Customer\-managed CMKs | 
| [Aliases](#aliases-limit) | 1100 | Customer\-created aliases | 
| [Grants per CMK](#grants-per-key) | 2500 | Customer\-managed CMKs | 
| [Grants for a given principal per CMK](#grants-per-principal-per-key) | 500 | All CMKs | 
| [Requests per second](#requests-per-second) | Varies by API operation; see [table](#requests-per-second-table)\. | All CMKs | 

**Note**  
If you are exceeding the [requests per second](#requests-per-second) limit, consider using the [data key caching](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/data-key-caching.html) feature of the AWS Encryption SDK\. Reusing data keys, rather than requesting a new data key for every encryption operation, might reduce the frequency of your requests to AWS KMS\. 

## Customer Master Keys \(CMKs\): 1000<a name="customer-master-keys-limit"></a>

You can have up to 1000 [customer managed CMKs](concepts.md#master_keys) per region\. This limit applies to all customer managed CMKs regardless of their [key state](key-state.md)\. [AWS managed CMKs](concepts.md#master_keys) do not count against this limit\.

You can create a [support case](https://console.aws.amazon.com/support/home) to request more CMKs in a region; however, managing a large number of CMKs from the AWS Management Console may be slower than acceptable\. If you have a large number of CMKs in a region, we recommend managing them programmatically with the [AWS SDKs](https://aws.amazon.com/tools/#sdk) or [AWS Command Line Tools](https://aws.amazon.com/tools/#cli)\.

## Aliases: 1100<a name="aliases-limit"></a>

You can create up to 1100 aliases in your account\. Aliases that AWS creates in your account, such as aws/*<service\-name>*, do not count against this limit\. 

An *alias* is a display name that you can map to a CMK\. Each alias is mapped to exactly one CMK and multiple aliases can map to the same CMK\. 

If you use a [support case](https://console.aws.amazon.com/support/home) to increase your CMK limit, you might also need to request an increase in the number of aliases\.

## Grants per CMK: 2500<a name="grants-per-key"></a>

Each [customer managed CMK](concepts.md#master_keys) can have up to 2500 grants, including the grants created by [AWS services that are integrated with AWS KMS](https://aws.amazon.com/kms/details/#integration)\. This limit does not apply to [AWS managed CMKs](concepts.md#master_keys)\.

One effect of this limit is that you cannot create more than 2500 resources that use the same CMK\. For example, you cannot create more than 2500 [encrypted EBS volumes](services-ebs.md) that use the same CMK\.

[Grants](grants.md) are an alternative to [key policy](key-policies.md)\. They are advanced mechanisms for specifying permissions\. 

You or an AWS service integrated with AWS KMS can use a grant to limit how and when the grantee can use a CMK\. Grants are attached to a CMK, and each grant includes the principal who receives permission to use the CMK, the ID of the CMK, and a list of operations that the grantee can perform\. 

## Grants for a given principal per CMK: 500<a name="grants-per-principal-per-key"></a>

For a given CMK, no more than 500 grants can specify the same grantee principal\. This limit applies to all CMKs, including [AWS managed CMKs](concepts.md#master_keys)\.

For example, you might want to encrypt multiple Amazon EBS volumes and attach them to a single Amazon Elastic Compute Cloud \(Amazon EC2\) instance\. A unique grant is created for each encrypted volume and all of these grants have the same grantee principal \(an IAM assumed\-role user associated with the EC2 instance\)\. Each grant gives permission to use the specified CMK to decrypt an EBS volume's unique data encryption key\. For each CMK, you can have up to 500 grants that specify the same EC2 instance as the grantee principal\. This effectively means that you can have no more than 500 encrypted EBS volumes per EC2 instance for a given CMK\.

## Requests per second: varies<a name="requests-per-second"></a>

AWS KMS throttles API requests at different limits depending on the API operation\. Throttling means that AWS KMS rejects an otherwise valid request because the request exceeds the limit for the number of requests per second\. When a request is throttled, AWS KMS returns a `ThrottlingException` error\. [The following table](#requests-per-second-table) lists each API operation and the point at which AWS KMS throttles requests for that operation\.

This limit applies to all CMKs, including [AWS managed CMKs](concepts.md#master_keys)\.

**Note**  
If you need to exceed these limits, please visit the [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case\.   
If you are exceeding the requests per second limit for the [GenerateDataKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) API operation, consider using the [data key caching](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/data-key-caching.html) feature of the AWS Encryption SDK\. Reusing data keys might reduce the frequency of your requests to AWS KMS\. 

**Shared limit**

The API operations in the first row of the following table share a limit of 1200 requests per second\. For example, when you make 600 `GenerateDataKey` and 400 `Decrypt` requests per second, AWS KMS doesn't throttle your requests\. However, when you make 200 `Encrypt` and 1100 `GenerateDataKey` requests per second, AWS KMS throttles your requests because you are making more than 1200 requests per second for operations with the shared limit\.

The remaining API operations have a unique limit for requests per second, which means the limit is not shared\.

**API requests made on your behalf**

You can make API requests directly or by using an integrated AWS service that makes API requests to AWS KMS on your behalf\. The limit applies to both kinds of requests\.

For example, you might store data in Amazon S3 using server\-side encryption with AWS KMS \(SSE\-KMS\)\. Each time you upload or download an S3 object that's encrypted with SSE\-KMS, Amazon S3 makes a `GenerateDataKey` \(for uploads\) or `Decrypt` \(for downloads\) request to AWS KMS on your behalf\. These requests count toward your limit, so AWS KMS throttles the requests if you exceed a combined total of 1200 uploads or downloads per second of S3 objects encrypted with SSE\-KMS\.

**Cross\-account requests**

When an application in one AWS account uses a CMK owned by a different account, that's known as a cross\-account request\. For cross\-account requests, AWS KMS throttles the account that makes the requests, not the account that owns the CMK\. For example, you might have applications in accounts A and B that both use a CMK in account C\. In this scenario, the limit for requests per second applies separately to accounts A and B, not to account C\.


**Requests per second limit for each AWS KMS API operation**  

| API operation | Requests per second limit | 
| --- | --- | 
|  `Decrypt` `Encrypt` `GenerateDataKey` `GenerateDataKeyWithoutPlaintext` `GenerateRandom` `ReEncrypt`  | 1200 \(shared\) | 
| CancelKeyDeletion | 5 | 
| CreateAlias | 5 | 
| CreateGrant | 50 | 
| CreateKey | 5 | 
| DeleteAlias | 5 | 
| DeleteImportedKeyMaterial | 5 | 
| DescribeKey | 30 | 
| DisableKey | 5 | 
| DisableKeyRotation | 5 | 
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
| UpdateKeyDescription | 5 | 