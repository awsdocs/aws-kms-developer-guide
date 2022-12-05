# Monitoring with Amazon CloudWatch<a name="monitoring-cloudwatch"></a>

You can monitor your AWS KMS keys using [Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/), an AWS service that collects and processes raw data from AWS KMS into readable, near real\-time metrics\. These data are recorded for a period of two weeks so that you can access historical information and gain a better understanding of the usage of your KMS keys and their changes over time\.

You can use Amazon CloudWatch to alert you to important events, such as the following ones\.
+ The imported key material in a KMS key is nearing its expiration date\.
+ A KMS key that is pending deletion is still being used\. 
+ The key material in a KMS key was automatically rotated\.
+ A KMS key was deleted\.

You can also create an [Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/) alarm that alerts you when your request rate reaches a certain percentage of a quota value\. For details, see [Manage your AWS KMS API request rates using Service Quotas and Amazon CloudWatch](http://aws.amazon.com/blogs/security/manage-your-aws-kms-api-request-rates-using-service-quotas-and-amazon-cloudwatch/) in the *AWS Security Blog*\.

**Topics**
+ [AWS KMS metrics and dimensions](#kms-metrics)
+ [Viewing AWS KMS metrics](#how-to-view-kms-metrics)
+ [Creating CloudWatch alarms to monitor KMS keys](#creating-alarms)

## AWS KMS metrics and dimensions<a name="kms-metrics"></a>

AWS KMS predefines Amazon CloudWatch metrics to make it easier for you to monitor critical data and create alarms\. You can view the AWS KMS metrics using the AWS Management Console and the Amazon CloudWatch API\. 

This section lists each AWS KMS metrics and the dimensions for each metric, and provides some basic guidance for creating CloudWatch alarms based on these metrics and dimensions\.

**Note**  
**Dimension group name**:   
To view a metric in the Amazon CloudWatch console, in the **Metrics** section, select the dimension group name\. Then you can filter by the **Metric name**\. This topic includes the metric name and dimension group name for each AWS KMS metric\.

**Topics**
+ [SecondsUntilKeyMaterialExpiration](#key-material-expiration-metric)
+ [ExternalKeyStoreThrottle](#metric-throttling)
+ [XksProxyCertificateDaysToExpire](#metric-xks-proxy-certificate-days-to-expire)
+ [XksProxyCredentialAge](#metric-xks-proxy-credential-age)
+ [XksProxyErrors](#metric-xks-proxy-errors)
+ [XksExternalKeyManagerStates](#metric-xks-ekm-states)
+ [XksProxyLatency](#metric-xks-proxy-latency)

### SecondsUntilKeyMaterialExpiration<a name="key-material-expiration-metric"></a>

The number of seconds remaining until the [imported key material](importing-keys.md) in a KMS key expires\. This metric is valid only for KMS keys with imported key material \(a [key material origin](concepts.md#key-origin) of `EXTERNAL`\) and an expiration date\.

Use this metric to track the time that remains until your imported key material expires\. When that time falls below a threshold that you define, you might want to reimport the key material with a new expiration date\. The `SecondsUntilKeyMaterialExpiration` metric is specific to a KMS key\. You cannot use this metric to monitor multiple KMS keys or KMS keys that you might create in the future\. For help with creating a CloudWatch alarm to monitor this metric, see [Creating a CloudWatch alarm for expiration of imported key material](importing-keys.md#imported-key-material-expiration-alarm)\.

The most useful statistic for this metric is `Minimum`, which tells you the smallest amount of time remaining for all data points in the specified statistical period\. The only valid unit for this metric is `Seconds`\.

**Dimension group name**: **Per\-Key Metrics**


**Dimensions for `SecondsUntilKeyMaterialExpiration`**  

| Dimension | Description; related to AWS | 
| --- | --- | 
| KeyId | Value for each KMS key\. | 

### ExternalKeyStoreThrottle<a name="metric-throttling"></a>

The number of requests for cryptographic operations on KMS keys in each external key store that AWS KMS throttles \(responds with a `ThrottlingException`\)\. This metric applies only to [external key stores](keystore-external.md)\.

The `ExternalKeyStoreThrottle` metric applies only to KMS keys in an external key store and only to requests for [cryptographic operations](concepts.md#cryptographic-operations) and the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. AWS KMS [throttles these requests](throttling.md) when the request rate exceeds the [custom key store request quota](requests-per-second.md#rps-key-stores)for your external key store\.

Use this metric to review and adjust the value of the your custom key store request quota\. If this metric indicates that AWS KMS is frequently throttling your requests for these KMS keys, you might consider [requesting an increase](increase-quota.md) in your custom key store request quota value\. 

This metric does not include throttling by your external key store proxy or external key manager\. If you are getting very frequent `KMSInvalidStateException` errors with a message that explains that the request was rejected "due to a very high request rate" or the request was rejected "because the external key store proxy did not respond in time," it might indicate that your external key manager or external key store proxy cannot keep pace with the current request rate\. In that case, consider [requesting a decrease](increase-quota.md) in your custom key store request quota value\. Decreasing this quota value might increase throttling \(and the `ExternalKeyStoreThrottle` metric value\), but it indicates that AWS KMS is rejecting excess requests quickly before they are sent to your external key store proxy or external key manager\.

**Dimension group name**: **Keystore Throttle Metrics**


| Dimension | Description | 
| --- | --- | 
| CustomKeyStoreId | Value for each external key store\. | 
| KmsOperation | Value for each AWS KMS API operation\. This metric applies only to cryptographic operations and the DescribeKey operation on KMS keys in an external key store\. | 
| KeySpec | Value for each type of KMS key\. The only supported [key spec](concepts.md#key-spec) for KMS keys in an external key store is SYMMETRIC\_DEFAULT\. | 

### XksProxyCertificateDaysToExpire<a name="metric-xks-proxy-certificate-days-to-expire"></a>

The number of days until the TLS certificate for your [external key store proxy endpoint](create-xks-keystore.md#require-endpoint) \(`XksProxyUriEndpoint`\) expires\. This metric applies only to [external key stores](keystore-external.md)\.

Use this metric to create a CloudWatch alarm that notifies you about the upcoming expiration of your TLS certificate\. When the certificate expires, AWS KMS cannot communicate with the external key store proxy\. All data protected by KMS keys in your external key store becomes inaccessible until you renew the certificate\. 

A certificate alarm prevents certificate expiration that might prevent you from accessing your encrypted resources\. Set the alarm to give your organization time to renew the certificate before it expires\.

**Dimension group name**: **XKS Proxy Certificate Metrics**


| Dimension | Description | 
| --- | --- | 
| CustomKeyStoreId | Value for each external key store\. | 
| CertificateName | Subject name \(CN\) in the TLS certificate\. | 

### XksProxyCredentialAge<a name="metric-xks-proxy-credential-age"></a>

The number of days since the current external key store [proxy authentication credential](keystore-external.md#concept-xks-credential) \(`XksProxyAuthenticationCredential`\) was associated with the external key store\. This count begins when you enter the authentication credential as part of creating or updating your external key store\. This metric applies only to [external key stores](keystore-external.md)\.

This value is designed to remind you about the age of your authentication credential\. However, because we begin the count when you associate the credential with your external key store, not when you create your authentication credential on your external key store proxy, this might not be an accurate indicator of the credential age on the proxy\.

Use this metric to create a CloudWatch alarm that reminds you to rotate your external key store proxy authentication credential\.

**Dimension group name**: **Per\-Keystore Metrics**


| Dimension | Description | 
| --- | --- | 
| CustomKeyStoreId | Value for each external key store\. | 

### XksProxyErrors<a name="metric-xks-proxy-errors"></a>

The number of exceptions related to AWS KMS requests to your [external key store proxy](keystore-external.md#concept-xks-proxy)\. This count includes exceptions that the external key store proxy returns to AWS KMS and timeout errors that occur when the external key store proxy does not respond to AWS KMS within the 250 millisecond timeout interval\. This metric applies only to [external key stores](keystore-external.md)\.

Use this metric to track the error rate of KMS keys in your external key store\. It reveals the most frequent errors, so you can prioritize your engineering effort\. For example, KMS keys that are generating high rates of non\-retryable errors might indicate a problem with the configuration of your external key store\. To view your external key store configuration, see [Viewing an external key store](view-xks-keystore.md)\. To edit your external key store settings, see [Editing external key store properties](update-xks-keystore.md)\.

**Dimension group name**: **XKS Proxy Error Metrics**


| Dimension | Description | 
| --- | --- | 
| CustomKeyStoreId | Value for each external key store\. | 
| KmsOperation | Value for each AWS KMS API operation that generated a request to the XKS proxy\. | 
| XksOperation | Value for each [external key store proxy API operation](keystore-external.md#concept-proxy-apis)\. | 
| KeySpec | Value for each type of KMS key\. The only supported [key spec](concepts.md#key-spec) for KMS keys in an external key store is SYMMETRIC\_DEFAULT\. | 
| ErrorType | Values:[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/monitoring-cloudwatch.html) | 
| ExceptionName |  Values: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/monitoring-cloudwatch.html)  | 

### XksExternalKeyManagerStates<a name="metric-xks-ekm-states"></a>

A count of the number of [external key manager instances](keystore-external.md#concept-ekm) in each of the following health states: `Active`, `Degraded`, and `Unavailable`\. The information for this metric comes from the external key store proxy associated with each external key store\. This metric applies only to [external key stores](keystore-external.md)\.

The following are the health states for the external key manager instances associated with an external key store\. Each external key store proxy might use different indicators to measure the health states of your external key manager\. For details, see the documentation for your external key store proxy\.
+ `Active`: The external key manager is healthy\.
+ `Degraded`: The external key manager is unhealthy, but can still serve traffic
+ `Unavailable`: The external key manager cannot serve traffic\.

Use this metric to create a CloudWatch alarm that alerts you to degraded and unavailable external key manager instances\. To determine which external key manager instances are in each state, consult your external key store proxy logs\.

**Dimension group name**: **XKS External Key Manager Metrics**


| Dimension | Description | 
| --- | --- | 
| CustomKeyStoreId | Value for each external key store\. | 
| XksExternalKeyManagerState | Value for each health state\. | 

### XksProxyLatency<a name="metric-xks-proxy-latency"></a>

The number of milliseconds it takes for an external key store proxy to respond to an AWS KMS request\. If the request timed out, the recorded value is the 250 millisecond timeout limit\. This metric applies only to [external key stores](keystore-external.md)\.

Use this metric to evaluate the performance of your external key store proxy and external key manager\. For example, if the proxy is frequently timing out on encryption and decryption operations, consult your external proxy administrator\. 

Slow responses might also indicate that your external key manager cannot handle the current request traffic\. AWS KMS recommends that your external key manager be able to handle up to 1800 requests for cryptographic operations per second\. If your external key manager cannot handle the 1800 requests per second rate, consider requesting a decrease in your [request quota for KMS keys in a custom key store](requests-per-second.md#rps-key-stores)\. Requests for cryptographic operations using the KMS keys in your external key store will fail fast with a [throttling exception](throttling.md), rather than being processed and later rejected by your external key store proxy or external key manager\.

**Dimension group name**: **XKS Proxy Latency Metrics**


| Dimension | Description | 
| --- | --- | 
| CustomKeyStoreId | Value for each external key store\. | 
| KmsOperation | Value for each AWS KMS API operation that generated a request to the XKS proxy\. | 
| XksOperation | Value for each [external key store proxy API operation](keystore-external.md#concept-proxy-apis)\. | 
| KeySpec | Value for each type of KMS key\. The only supported [key spec](concepts.md#key-spec) for KMS keys in an external key store is SYMMETRIC\_DEFAULT\. | 

## Viewing AWS KMS metrics<a name="how-to-view-kms-metrics"></a>

You can view the AWS KMS metrics using the AWS Management Console and the Amazon CloudWatch API\.

**To view metrics using the CloudWatch console**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. If necessary, change the region\. From the navigation bar, select the region where your AWS resources reside\.

1. In the navigation pane, choose **Metrics**, **All metrics**\.

1. On the **Browse** tab, search for `KMS`, and them choose **KMS**\.

1. Choose the dimension group name of the metric you want to view\. 

   For example, for the `SecondsUntilKeyMaterialExpiration` metric, choose **Per\-Key Metrics**\.

1. For a graph of the metric value, choose the metric name, then choose `Add to graph`\. To convert the line graph to a value, choose **Line**, then choose **Number**\. 

**To view metrics using the Amazon CloudWatch API**  
To view AWS KMS metrics using the CloudWatch API, send a [ListMetrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_ListMetrics.html) request with `Namespace` set to `AWS/KMS`\. The following example shows how to do this with the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/)\.

```
$  aws cloudwatch list-metrics --namespace AWS/KMS

{
    "Metrics": [
        {
            "Namespace": "AWS/KMS",
            "MetricName": "SecondsUntilKeyMaterialExpiration",
            "Dimensions": [
                {
                    "Name": "KeyId",
                    "Value": "1234abcd-12ab-34cd-56ef-1234567890ab"
                }
            ]
        },
        {
            "Namespace": "AWS/KMS",
            "MetricName": "ExtenalKeyStoreThrottle",
            "Dimensions": [
                {
                    "Name": "CustomKeyStoreId",
                    "Value": "cks-1234567890abcdef0"
                },
                {
                    "Name": "KmsOperation",
                    "Value": "Encrypt"
                },
                {
                    "Name": "KeySpec",
                    "Value": "SYMMETRIC_DEFAULT"
                }    
            ]
        },
        {
            "Namespace": "AWS/KMS",
            "MetricName": "XksProxyCertificateDaysToExpire",
            "Dimensions": [
                {
                    "Name": "CustomKeyStoreId",
                    "Value": "cks-1234567890abcdef0"
                },
                {
                    "Name": "CertificateName",
                    "Value": "myproxy.xks.example.com"
                }    
            ]
        },
        {
            "Namespace": "AWS/KMS",
            "MetricName": "XksProxyCredentialAge",
            "Dimensions": [
                {
                    "Name": "CustomKeyStoreId",
                    "Value": "cks-1234567890abcdef0"
                }
            ]
        },
        {
            "Namespace": "AWS/KMS",
            "MetricName": "XksProxyErrors",
            "Dimensions": [
                {
                    "Name": "CustomKeyStoreId",
                    "Value": "cks-1234567890abcdef0"
                },
                {
                    "Name": "KmsOperation",
                    "Value": "Decrypt"
                },
                {
                    "Name": "XksOperation",
                    "Value": "Decrypt"
                },
                {
                     "Name": "KeySpec",
                     "Value": "SYMMETRIC_DEFAULT"
                },
                {
                    "Name": "ErrorType",
                    "Value": "Retryable errors"
                },
                {
                     "Name": "ExceptionName",
                     "Value": "KMSInvalidStateException"
                }                
            ]
        },
        {
            "Namespace": "AWS/KMS",
            "MetricName": "XksProxyHsmStates",
            "Dimensions": [
                {
                    "Name": "CustomKeyStoreId",
                    "Value": "cks-1234567890abcdef0"
                },
                {
                    "Name": "XksProxyHsmState",
                    "Value": "Active"
                }
            ]
        },
        {
            "Namespace": "AWS/KMS",
            "MetricName": "XksProxyLatency",
            "Dimensions": [
                {
                    "Name": "CustomKeyStoreId",
                    "Value": "cks-1234567890abcdef0"
                },
                {
                    "Name": "KmsOperation",
                    "Value": "Decrypt"
                },
                {
                    "Name": "XksOperation",
                    "Value": "Decrypt"
                },
                {
                     "Name": "KeySpec",
                     "Value": "SYMMETRIC_DEFAULT"
                }                
            ]
        }
    ]
}
```

## Creating CloudWatch alarms to monitor KMS keys<a name="creating-alarms"></a>

You can create an Amazon CloudWatch alarm based on an AWS KMS metric\. The alarm sends an email message when a metric value exceeds a threshold specified in the alarm configuration\. The alarm can send the email message to an [Amazon Simple Notification Service \(Amazon SNS\) topic](https://docs.aws.amazon.com/sns/latest/dg/sns-create-topic.html) or an [Amazon EC2 Auto Scaling policy](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scale-based-on-demand.html#as-how-scaling-policies-work)\. For detailed information about CloudWatch alarms, see [Using Amazon CloudWatch alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html) in the Amazon CloudWatch User Guide

### Create an alarm for expiring imported key material<a name="key-material-expiration-alarm"></a>

You can use the [`SecondsUntilKeyMaterialExpiration`](#key-material-expiration-metric) metric to create a CloudWatch alarm that notifies you when the imported key material in a KMS key is about to expire\.

When you [import key material into a KMS key](importing-keys.md), you can optionally specify a date and time when the key material expires\. When the key material expires, AWS KMS deletes the key material and the KMS key becomes unusable\. To use the KMS key again, you must [reimport the key material](importing-keys.md#reimport-key-material)\. 

For instructions, see [Creating a CloudWatch alarm for expiration of imported key material](importing-keys.md#imported-key-material-expiration-alarm)\.

### Create an alarm for use of KMS keys that are pending deletion<a name="cmk-pending-deletion-alarm"></a>

When you [schedule deletion](deleting-keys.md) of a KMS key, AWS KMS enforces a waiting period before deleting the KMS key\. You can use the waiting period to ensure that you don't need the KMS key now or in the future\. You can also configure a CloudWatch alarm to warn you if a person or application attempts to use the KMS key in a [cryptographic operation](concepts.md#cryptographic-operations) during the waiting period\. If you receive a notification from such an alarm, you might want to cancel deletion of the KMS key\.

For instructions, see [Creating an alarm that detects use of a KMS key pending deletion](deleting-keys-creating-cloudwatch-alarm.md)\.

### Create an alarm to monitor an external key store<a name="xks-metric-alarm"></a>

You can create CloudWatch alarms based on the metrics for external key stores and KMS keys in external key stores\.

For example, we recommend that you set a CloudWatch alarm to notify you when the TLS certificate for your external key store is about to expire \(XksProxyCertificateDaysToExpire\), when your and when your external key store proxy reports that your external key manager instances are in a degraded or unavailable state \(XksProxyHsmStates\)\.

For instructions, see [Monitoring an external key store](xks-monitoring.md)\.