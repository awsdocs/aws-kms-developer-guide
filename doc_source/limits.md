# Quotas<a name="limits"></a>

To make AWS KMS responsive and performant for all users, AWS KMS applies two types of quotas, resource quotas and request quotas\. Each quota is calculated independently for each Region of each AWS account\.

All AWS KMS quotas are adjustable, except for the [key policy document size quota](resource-limits.md#key-policy-limit), and the request quota for KMS keys in a [custom key store](requests-per-second.md#rps-key-stores)\. To request a quota increase, use the [Service Quotas console](https://console.aws.amazon.com/servicequotas) or the [RequestServiceQuotaIncrease](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/API_RequestServiceQuotaIncrease.html) operation\. For instructions, see [Requesting an AWS KMS quota increase](increase-quota.md)\. For details, see [Requesting a quota increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html) in the *Service Quotas User Guide*\. If Service Quotas for AWS KMS are not available in your AWS Region, please visit the [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case\. 

To request an increase in an AWS KMS quota, see [Requesting an AWS KMS quota increase](increase-quota.md)\.

**Topics**
+ [Resource quotas](resource-limits.md)
+ [Request quotas](requests-per-second.md)
+ [Throttling AWS KMS requests](throttling.md)
+ [Requesting an AWS KMS quota increase](increase-quota.md)