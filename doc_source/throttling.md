# Throttling AWS KMS requests<a name="throttling"></a>

To ensure that AWS KMS can provide fast and reliable responses to API requests from all customer, it throttles API requests that exceed certain boundaries\. 

*Throttling* occurs when AWS KMS rejects a request that might otherwise be valid, and returns a `ThrottlingException` error like the following one\. 

```
You have exceeded the rate at which you may call KMS. Reduce the frequency of your calls. 
(Service: AWSKMS; Status Code: 400; Error Code: ThrottlingException; Request ID: <ID>
```

AWS KMS throttles requests for the following conditions\.
+ The rate of requests per second exceeds the AWS KMS [request quota](requests-per-second.md) for an account and Region\. 

  For example, if users in your account submit 1000 `DescribeKey` requests in a second, AWS KMS throttles all subsequent `DescribeKey` requests in that second\.

  To respond to throttling, use a [backoff and retry strategy](https://docs.aws.amazon.com/general/latest/gr/api-retries.html)\. This strategy is implemented automatically for HTTP 400 errors in some AWS SDKs\.
+ A burst or sustained high rate of requests to change the state of the same KMS key\. This condition is often known as a "hot key\."

  For example, if an application in your account sends a persistent volley of `EnableKey` and `DisableKey` requests for the same KMS key, AWS KMS throttles the requests\. This throttling occurs even if the requests don't exceed the request\-per\-second request limit for the `EnableKey` and `DisableKey` operations\.

  To respond to throttling, adjust your application logic so it makes only required requests or it consolidates the requests of multiple functions\. 
+ Requests for operations on KMS keys in [custom key stores](custom-key-store-overview.md) might be throttled at a lower\-than\-expected rate when the AWS CloudHSM cluster associated with the custom key store is processing numerous commands, including those unrelated to the custom key store\.

  \(AWS KMS no longer throttles requests for operations on KMS keys in a custom key store when there are no available PKCS \#11 sessions for the AWS CloudHSM cluster\. Instead, it throws a `KMSInternalException` and recommends that you retry your request\.\)

To view trends in your request rates, use the [Service Quotas console](https://console.aws.amazon.com/servicequotas)\. You can also create an [Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/) alarm that alerts you when your request rate reaches a certain percentage of a quota value\. For details, see [Manage your AWS KMS API request rates using Service Quotas and Amazon CloudWatch](http://aws.amazon.com/blogs/security/manage-your-aws-kms-api-request-rates-using-service-quotas-and-amazon-cloudwatch/) in the *AWS Security Blog*\.

All AWS KMS quotas are adjustable, except for the [key policy document size resource quota](resource-limits.md#key-policy-limit) and the [custom key stores resource quota](resource-limits.md#cks-resource-quota)\. To request a quota increase, see [Requesting a quota increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html) in the *Service Quotas User Guide*\. To request a quota decrease, to change a quota that is not listed in Service Quotas, or to change a quota in an AWS Region where Service Quotas for AWS KMS is not available, please visit [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case\. 

**Note**  
AWS KMS custom key store quotas do not appear in the Service Quotas console\. You cannot view or manage these quotas by using Service Quotas API operations\. To request a change in your custom key store request quota, visit the [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case\.