# Throttling AWS KMS requests<a name="throttling"></a>

To ensure that AWS KMS can provide fast and reliable responses to API requests from all customer, it throttles API requests that exceed certain boundaries\. 

*Throttling* occurs when AWS KMS rejects an otherwise valid request and returns a `ThrottlingException` error like the following one\. 

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