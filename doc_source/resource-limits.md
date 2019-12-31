# Resource Quotas<a name="resource-limits"></a>

AWS KMS establishes resource quotas to ensure that it can provide fast and resilient service to all of our customers\. Some resource quotas apply only to resources that you create, but not to resources that AWS services create for you\. Resources that you use, but that aren't in your AWS account, such as [AWS owned CMKs](concepts.md#aws-owned-cmk), do not count against these quotas\.

If you have reached a resource limit, requests to create an additional resource of that type generate an `LimitExceededException` error message\. 

The following table lists and describes the AWS KMS resource quotas in each AWS account and Region\. If you need to exceed a quota, you can request a quota increase in Service Quotas\. Use the [Service Quotas console](https://console.aws.amazon.com/servicequotas) or the [RequestServiceQuotaIncrease](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/API_RequestServiceQuotaIncrease.html) operation\. For details, see [Requesting a Quota Increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html) in the *Service Quotas User Guide*\. If Service Quotas for AWS KMS are not available in the AWS Region, please visit the [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case\. 


| Resource | Default Limit | Applies To | 
| --- | --- | --- | 
| [Customer Master Keys \(CMKs\)](#customer-master-keys-limit) | 10,000 | Customer managed CMKs | 
| [Aliases](#aliases-limit) | 10,000 | Customer created aliases | 
| [Grants per CMK](#grants-per-key) | 10,000 | Customer managed CMKs | 
| [Grants for a given principal per CMK](#grants-per-principal-per-key) | 500 | Customer managed CMKsAWS managed CMKs | 
| [Key policy document size](#key-policy-limit) | 32 KB \(32,768 bytes\) | Customer managed CMKsAWS managed CMKs | 

In addition to resource quotas, AWS KMS uses request quotas to ensure the responsiveness of the service\. For details, see [Request Quotas](requests-per-second.md)\.

## Customer Master Keys \(CMKs\): 10,000<a name="customer-master-keys-limit"></a>

You can have up to 10,000 [customer managed CMKs](concepts.md#customer-cmk) in each Region of your AWS account\. This quota applies to all symmetric and asymmetric customer managed CMKs regardless of their [key state](key-state.md)\. Each CMK — whether symmetric or asymmetric — is considered to be one resource\. [AWS managed CMKs](concepts.md#aws-managed-cmk) and [AWS owned CMKs](concepts.md#aws-owned-cmk) do not count against this quota\.

If you need to exceed this quota, request a quota increase in Service Quotas\. However, managing a large number of CMKs from the AWS Management Console may be slower than acceptable\. If you have a large number of CMKs in an AWS Region, manage them programmatically with the [AWS SDKs](https://aws.amazon.com/tools/#sdk) or [AWS Command Line Tools](https://aws.amazon.com/tools/#cli)\.

## Aliases: 10,000<a name="aliases-limit"></a>

You can create up to 10,000 aliases in each Region of your account\. Aliases that AWS creates in your account, such as aws/*<service\-name>*, do not count against this quota\. 

An *alias* is a display name that you can map to a CMK\. Each alias is mapped to exactly one CMK and multiple aliases can map to the same CMK\. 

If you increase your CMK resource quota, you might also need to increase your aliases resource quota\. For help with requesting a quota increase, see [Requesting a Quota Increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html) in the *Service Quotas User Guide*\.

## Grants per CMK: 10,000<a name="grants-per-key"></a>

Each [customer managed CMK](concepts.md#customer-cmk) can have up to 10,000 grants, including the grants created by [AWS services that are integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration)\. This quota does not apply to [AWS managed CMKs](concepts.md#aws-managed-cmk) or [AWS owned CMKs](concepts.md#aws-owned-cmk)\.

One effect of this quota is that you cannot perform more than 10,000 grant\-authorized operations that use the same CMK at the same time\. After you reach the quota, you can create new grants on the CMK only when an active grant is retired or revoked\.

For example, when you attach an Amazon Elastic Block Store \(Amazon EBS\) volume to an Amazon Elastic Compute Cloud \(Amazon EC2\) instance, the volume is decrypted so you can read it\. To get permission to decrypt the data, Amazon EBS creates a grant for each volume\. However, you cannot have more than 10,000 grants on each CMK\. Therefore, if all of your Amazon EBS volumes use the same CMK, you cannot attach more than 10,000 volumes at one time\.

[Grants](grants.md) are an alternative to [key policy](key-policies.md)\. Like a key policy, a grant is attached to a CMK\. You \(or an AWS service integrated with AWS KMS\) can use a grant to allow a principal to use or manage the CMK\. Each grant includes the principal who receives permission to use the CMK, the ID of the CMK, and a list of operations that the grantee can perform\. 

## Grants for a Given Principal per CMK: 500<a name="grants-per-principal-per-key"></a>

You cannot have more than 500 grants on a CMK that specify the same grantee principal\. This quota is calculated separately for each CMK in the account\. It applies to [customer managed CMKs](concepts.md#customer-cmk) and [AWS managed CMKs](concepts.md#aws-managed-cmk), but not to [AWS owned CMKs](concepts.md#aws-owned-cmk)\.

For example, when you attach an Amazon Elastic Block Store \(Amazon EBS\) volume to an Amazon Elastic Compute Cloud \(Amazon EC2\) instance, the volume is decrypted so you can read it\. To get permission to decrypt the data, Amazon EBS creates a grant for each volume\. Each grant is unique, but all of the grants have the same grantee principal, a user that assumes a role associated with Amazon EC2 instances\. However, you cannot have more than 500 grants for the same principal on each CMK\. Therefore, if all of your Amazon EBS volumes use the same CMK, you cannot attach more than 500 volumes at one time\.

## Key Policy Document Size: 32 KB<a name="key-policy-limit"></a>

The maximum length of each key policy document is 32 KB \(32,768 bytes\)\. If you use a larger policy document to create or update the key policy for a CMK, the operation fails\. 

If you must exceed this quota, request a quota increase in Service Quotas\. For details, see [Requesting a Quota Increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html) in the *Service Quotas User Guide*\.

A [key policy document](key-policies.md#key-policy-overview) is a collection of policy statements in JSON format\. The statements in the key policy document determine who has permission to use the CMK and how they can use it\. You may also use IAM policies and grants to control access to the CMK, but every CMK must have a key policy document\. 

You use a key policy document whenever you create or change a key policy by using the [default view](key-policy-modifying.md#key-policy-modifying-how-to-console-default-view) or [policy view](key-policy-modifying.md#key-policy-modifying-how-to-console-policy-view) in the AWS Management Console, or the [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operation\. This quota applies to your key policy document, even if you use the [default view](key-policy-modifying.md#key-policy-modifying-how-to-console-default-view) in the AWS KMS console, where you don't edit the JSON statements directly\.