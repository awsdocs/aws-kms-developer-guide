# Troubleshooting external key stores<a name="xks-troubleshooting"></a>

The resolution for most problems with external key stores are indicated by the error message that AWS KMS displays with each exception, or by the [connection error code](#fix-xks-connection) that AWS KMS returns when an attempt to [connect the external key store](xks-connect-disconnect.md) to its external key store proxy fails\. However, some issues are a bit more complex\. 

When diagnosing an issue with an external key store, first locate the cause\. This will narrow the range of remedies and make your troubleshooting more efficient\.
+ AWS KMS — The problem might be within AWS KMS, such as an incorrect value in your [external key store configuration](create-xks-keystore.md#xks-requirements)\.
+ External — The problem might originate outside of AWS KMS, including problems with the configuration or operation of the external key store proxy, external key manager, external keys, or VPC endpoint service\.
+ Networking — It might be a problem with connectivity or networking, such as a problem with your proxy endpoint, port, or your private DNS name or domain\.

**Note**  
When management operations on external key stores fail, they generate several different exceptions\. But when cryptographic operations on external key stores fail, they always return `KMSInvalidStateException`\. To identify the problem, use the accompanying error message text\.  
The [ConnectCustomKeyStore](xks-connect-disconnect.md) operation succeeds quickly before the connection process is complete\. To determine whether the connection process is successful, view the [connection state](xks-connect-disconnect.md#xks-connection-state) of the external key store\. If the connection process fails, AWS KMS returns a [connection error code](#xks-connection-error-codes) that explains the cause and suggests a remedy\.

**Topics**
+ [Troubleshooting tools for external key stores](#xks-troubleshooting-tools)
+ [Configuration errors](#fix-xks-configuration)
+ [External key store connection errors](#fix-xks-connection)
+ [Latency and timeout errors](#fix-xks-latency)
+ [Authentication credential errors](#fix-xks-credentials)
+ [Key state errors](#fix-unavailable-xks-keys)
+ [Decryption errors](#fix-xks-decrypt)
+ [External key errors](#fix-external-key)
+ [Proxy issues](#fix-xks-proxy)
+ [Proxy authorization issues](#fix-xks-authorization)

## Troubleshooting tools for external key stores<a name="xks-troubleshooting-tools"></a>

AWS KMS provides several tools to help you identify and resolve problems with your external key store and its keys\. Use these tools in conjunction with the tools provided with your external key store proxy and external key manager\.

**Note**  
Your external key store proxy and external key manager might provide easier methods of creating and maintaining your external key store and its KMS keys\. For details, see the documentation for your external tools\. 

**AWS KMS exceptions and error messages**  
AWS KMS provides a detailed error message about any problem it encounters\. You can find additional information about AWS KMS exceptions in the [https://docs.aws.amazon.com/kms/latest/APIReference/](https://docs.aws.amazon.com/kms/latest/APIReference/) and AWS SDKs\. Even if you are using the AWS KMS console, you might find these references to be helpful\. For example, see the [Errors](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateCustomKeyStore.html#API_CreateCustomKeyStore_Errors) list for the `CreateCustomKeyStores` operation\.  
If the problem surfaces in a different AWS service, such as when you use a KMS key in your external key store to protect a resource in another AWS service, the AWS service might provide additional information to help you identify the problem\. If the AWS service doesn't provide the message, you can view the error message in the [CloudTrail logs](logging-using-cloudtrail.md) that record the use of your KMS key\.

**[CloudTrail logs](logging-using-cloudtrail.md)**  
Every AWS KMS API operation, including actions in the AWS KMS console, is recorded in AWS CloudTrail logs\. AWS KMS records a log entry for successful and failed operations\. For failed operations, the log entry includes the AWS KMS exception name \(`errorCode`\) and the error message \(`errorMessage`\)\. You can use this information to help you identify and resolve the error\. For an example, see [Decrypt failure with a KMS key in an external key store](ct-decrypt.md#ct-decrypt-xks-fail)\.  
The log entry also includes the request ID\. If the request reached your external key store proxy, you can use the request ID in the log entry to find the corresponding request in your proxy logs, if your proxy provides them\.

**[CloudWatch metrics](monitoring-cloudwatch.md#kms-metrics)**  
AWS KMS records detailed Amazon CloudWatch metrics about the operation and performance of your external key store, including latency, throttling, proxy errors, external key manager status, the number of days until your TLS certificate expires, and the reported age of your proxy authentication credentials\. You can use these metrics to develop data models for the operation of your external key store and CloudWatch alarms that alert you to impending problems before they occur\.   
AWS KMS recommends that you create CloudWatch alarms to monitor the external key store metrics\. These alarms will alert you to early signs of problems before they develop\.

**[Monitoring graphs](xks-monitoring.md)**  
AWS KMS displays graphs of the external key store CloudWatch metrics on the detail page for each external key store in the AWS KMS console\. You can use the data in the graphs to help locate the source of errors, detect impending problems, establish baselines, and refine your CloudWatch alarm thresholds\. For details about interpreting the monitoring graphs and using their data, see [Monitoring an external key store](xks-monitoring.md)\.

**Displays of external key stores and KMS keys**  
AWS KMS displays detailed information about your external key stores and the KMS keys in the external key store in the AWS KMS console, and in the response to the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) and [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operations\. These displays include special fields for external key stores and KMS keys with information that you can use for troubleshooting, such as the [connection state](xks-connect-disconnect.md#xks-connection-state) of the external key store and the ID of the external key that is associated with the KMS key\. For details, see [Viewing an external key store](view-xks-keystore.md) and [Viewing KMS keys in an external key store](view-xks-key.md)\.

**[XKS Proxy Test Client](https://github.com/aws-samples/aws-kms-xksproxy-test-client)**  
AWS KMS provides an open source test client that verifies that your external key store proxy conforms to the [AWS KMS External Key Store Proxy API Specification](https://github.com/aws/aws-kms-xksproxy-api-spec/)\. You can use this test client to identify and resolve problems with your external key store proxy\.

## Configuration errors<a name="fix-xks-configuration"></a>

When you create an external key store, you specify property values that comprise the *configuration* of your external key store, such as the [proxy authentication credential](create-xks-keystore.md#require-credential), [proxy URI endpoint](create-xks-keystore.md#require-endpoint), [proxy URI path](create-xks-keystore.md#require-path), and [VPC endpoint service name](create-xks-keystore.md#require-vpc-service-name)\. When AWS KMS detects an error in a property value, the operation fails and returns an error that indicates the faulty value\. 

Many configuration issues can be resolved by fixing the incorrect value\. You can fix an invalid proxy URI path or proxy authentication credential without disconnecting the external key store\. For definitions of these values, including uniqueness requirements, see [Assemble the prerequisites](create-xks-keystore.md#xks-requirements)\. For instructions about updating these values, see [Editing external key store properties](update-xks-keystore.md)\.

To avoid errors with your proxy URI path and proxy authentication credential values, when creating or updating your external key store, upload a [proxy configuration file](create-xks-keystore.md#proxy-configuration-file) to the AWS KMS console\. This is a JSON\-based file with proxy URI path and proxy authentication credential values that is provided by your external key store proxy or external key manager\. You can't use a proxy configuration file with AWS KMS API operations, but you can use the values in the file to help you provide parameter values for your API requests that match the values in your proxy\.

### General configuration errors<a name="fix-xks-gen-configuration"></a>

**Exceptions**: `CustomKeyStoreInvalidStateException` \(`CreateKey`\), `KMSInvalidStateException` \(cryptographic operations\), `XksProxyInvalidConfigurationException` \(management operations, except for `CreateKey`\)

[**Connection error codes**](#xks-connection-error-codes): `XKS_PROXY_INVALID_CONFIGURATION`, `XKS_PROXY_INVALID_TLS_CONFIGURATION`

For external key stores with [public endpoint connectivity](plan-xks-keystore.md#xks-connectivity-public-endpoint), AWS KMS tests the property values when you create and update the external key store\. For external key stores with [VPC endpoint service connectivity](plan-xks-keystore.md#xks-vpc-connectivity), AWS KMS tests the property values when you connect and update the external key store\. 

**Note**  
The `ConnectCustomKeyStore` operation, which is asynchronous, might succeed even though the attempt to connect the external key store to its external key store proxy fails\. In that case, there is no exception, but the connection state of the external key store is Failed, and a connection error code explains the error message\. For more information, see [External key store connection errors](#fix-xks-connection)\.

If AWS KMS detects an error in a property value, the operation fails returns an `XksProxyInvalidConfigurationException` with one of the following error messages\.


|  | 
| --- |
| The external key store proxy rejected the request because of an invalid URI path\. Verify the URI path for your external key store and update if necessary\. | 
+ The [proxy URI path](create-xks-keystore.md#require-path) is the base path for AWS KMS requests to the proxy APIs\. If this path is incorrect, all requests to the proxy fail\. To [view the current proxy URI path](view-xks-keystore.md) for your external key store, use the AWS KMS console or the `DescribeCustomKeyStores` operation\. To find the correct proxy URI path, see your external key store proxy documentation\. For help correcting your proxy URI path value, see [Editing external key store properties](update-xks-keystore.md)\.
+ The proxy URI path for your external key store proxy can change with updates to your external key store proxy or external key manager\. For information about these changes, see the documentation for your external key store proxy or external key manager\.


|  | 
| --- |
| `XKS_PROXY_INVALID_TLS_CONFIGURATION`AWS KMS cannot establish a TLS connection to the external key store proxy\. Verify the TLS configuration, including its certificate\. | 
+ All external key store proxies require a TLS certificate\. The TLS certificate must be issued by a public certificate authority \(CA\) that is supported for external key stores\. For list of supported CAs, see [Trusted Certificate Authorities](https://github.com/aws/aws-kms-xksproxy-api-spec/blob/main/TrustedCertificateAuthorities) in the AWS KMS External Key Store Proxy API Specification\.
+ For public endpoint connectivity, the subject common name \(CN\) on the TLS certificate must match the domain name in the [proxy URI endpoint](create-xks-keystore.md#require-endpoint) for the external key store proxy\. For example, if the public endpoint is https://myproxy\.xks\.example\.com, the TLS, the CN on the TLS certificate must be `myproxy.xks.example.com` or `*.xks.example.com`\.
+ For VPC endpoint service connectivity, the subject common name \(CN\) on the TLS certificate must match the private DNS name for your [VPC endpoint service](create-xks-keystore.md#require-vpc-service-name)\. For example, if the private DNS name is myproxy\-private\.xks\.example\.com, the CN on the TLS certificate must be `myproxy-private.xks.example.com` or `*.xks.example.com`\.
+ The TLS certificate cannot be expired\. To get the expiration date of a TLS certificate, use SSL tools, such as [OpenSSL](https://www.openssl.org/)\. To monitor the expiration date of a TLS certificate associated with an external key store, use the [XksProxyCertificateDaysToExpire](monitoring-cloudwatch.md#metric-xks-proxy-certificate-days-to-expire) CloudWatch metric\. The number of days to your TLS certification expiration date also appears in the [**Monitoring** section](xks-monitoring.md) of the AWS KMS console\. 
+ If you are using [public endpoint connectivity](plan-xks-keystore.md#xks-connectivity-public-endpoint), use SSL test tools to test your SSL configuration\. TLS connection errors can result from incorrect certificate chaining\. 

### VPC endpoint service connectivity configuration errors<a name="fix-xks-vpc-configuration"></a>

**Exceptions**: `XksProxyVpcEndpointServiceNotFoundException`, `XksProxyVpcEndpointServiceInvalidConfigurationException`

In addition to general connectivity issues, you might encounter the following issues while creating, connecting, or updating an external key store with VPC endpoint service connectivity\. AWS KMS tests the property values of an external key store with VPC endpoint service connectivity while [creating](create-xks-keystore.md), [connecting](xks-connect-disconnect.md), and [updating](update-xks-keystore.md) the external key store\. When management operations fail due to configuration errors, they generate the following exceptions:


|  | 
| --- |
| XksProxyVpcEndpointServiceNotFoundException | 

The cause might be one of the following:
+ An incorrect VPC endpoint service name\. Verify that the VPC endpoint service name for the external key store is correct and matches the proxy URI endpoint value for the external key store\. To find the VPC endpoint service name, use the [Amazon VPC console](https://console.aws.amazon.com/vpc) or the [DescribeVpcEndpointServices](https://docs.aws.amazon.com/AmazonVPC/latest/APIReference/DescribeVpcEndpointServices.html) operation\. To find the VPC endpoint service name and proxy URI endpoint of an existing external key store, use the AWS KMS console or the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation\. For details, see [Viewing an external key store](view-xks-keystore.md)\.
+ The VPC endpoint service might be in a different AWS Region than the external key store\. Verify that the VPC endpoint service and external key store are in same Region\. \(The external name of the Region name, such as `us-east-1`, is part of the VPC endpoint service name, such as com\.amazonaws\.vpce\.us\-east\-1\.vpce\-svc\-example\.\) For a list of requirements for the VPC endpoint service for an external key store, see [VPC endpoint service](create-xks-keystore.md#require-vpc-service-name)\. You cannot move a VPC endpoint service or an external key store to a different Region\. However, you can create a new external key store in the same Region as the VPC endpoint service\. For details, see [Configuring VPC endpoint service connectivity](vpc-connectivity.md) and [Creating an external key store](create-xks-keystore.md)\.
+ AWS KMS is not an allowed principal for the VPC endpoint service\. The **Allow principals** list for the VPC endpoint service must include the `cks.kms.<region>.amazonaws.com` value, such as `cks.kms.eu-west-3.amazonaws.com`\. For instructions about adding this value, see [Manage permissions](https://docs.aws.amazon.com/vpc/latest/privatelink/configure-endpoint-service.html#add-remove-permissions) in the *AWS PrivateLink Guide*\.


|  | 
| --- |
| XksProxyVpcEndpointServiceInvalidConfigurationException | 

This error occurs when the VPC endpoint service fails to meet one of the following requirements:
+ The VPC requires at least two private subnets, each in a different Availability Zone\. For help adding a subnet to your VPC, see [Create a subnet in your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-subnets.html#create-subnets) in the *Amazon VPC User Guide*\.
+ Your [VPC endpoint service type](https://docs.aws.amazon.com/vpc/latest/privatelink/create-endpoint-service.html) must use a network load balancer, not a gateway load balancer\.
+ Acceptance must not be required for the VPC endpoint service \(**Acceptance required** must be false\.\)\. If manual acceptance of each connection request is required, AWS KMS cannot use the VPC endpoint service to connect to the external key store proxy\. For details, see [Accept or reject connection requests](https://docs.aws.amazon.com/vpc/latest/privatelink/configure-endpoint-service.html#accept-reject-connection-requests) in the *AWS PrivateLink Guide*\.
+ The VPC endpoint service must have a private DNS name that is a subdomain of a public domain\. For example, if the private DNS name is `https://myproxy-private.xks.example.com`, the `xks.example.com` or `example.com` domains must have a public DNS server\. To view or change the private DNS name for your VPC endpoint service, see [Manage DNS names for VPC endpoint services](https://docs.aws.amazon.com/vpc/latest/privatelink/manage-dns-names.html) in the *AWS PrivateLink Guide*\.
+ The **Domain verification status** of the domain for your private DNS name must be `verified`\. To view and update the verification status of the private DNS name domain, see [Verifying your private DNS name domain](vpc-connectivity.md#xks-private-dns)\. It might take a few minutes for the updated verification status to appear after you've added the required text record\. 
**Note**  
A private DNS domain can be verified only if it is the subdomain of a public domain\. Otherwise, the verification status of the private DNS domain does not change, even after you add the required TXT record\. 
+ The private DNS name of the VPC endpoint service must match the [proxy URI endpoint](create-xks-keystore.md#require-endpoint) value for the external key store\. For an external key store with VPC endpoint service connectivity, the proxy URI endpoint must be `https://` followed by the private DNS name of the VPC endpoint service\. To view the proxy URI endpoint value, see [Viewing an external key store](view-xks-keystore.md)\. To change the proxy URI endpoint value, see [Editing external key store properties](update-xks-keystore.md)\.

## External key store connection errors<a name="fix-xks-connection"></a>

The [process of connecting an external key store](xks-connect-disconnect.md#about-xks-connecting) to its external key store proxy takes about five minutes to complete\. Unless it fails quickly, the `ConnectCustomKeyStore` operation returns an HTTP 200 response and a JSON object with no properties\. However, this initial response does not indicate that the connection was successful\. To determine whether the external key store is connected, see its [connection state](xks-connect-disconnect.md#xks-connection-state)\. If the connection fails, the connection state of the external key store changes to `FAILED` and AWS KMS generates a [connection error code](#xks-connection-error-codes) that explains the cause of the failure\.

**Note**  
When the connection state of a custom key store is `FAILED`, you must disconnect the custom key store before attempting to reconnect it\. You cannot connect a custom key store with a `FAILED` connection status\.

To view the connection state of an external key store:
+ In the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) response, view the value of the `ConnectionState` element\.
+ In the AWS KMS console, the **Connection state** appears in the external key store table\. Also, on the detail page for each external key store, the **Connection state** appears in the **General configuration** section\.

When the connection state is `FAILED`, the connection error code helps to explains the error\. 

To view the connection error code:
+ In the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) response, view the value of the `ConnectionErrorCode` element\. This element appears in the `DescribeCustomKeyStores` response only when the `ConnectionState` is `FAILED`\.
+ To view the connection error code in the AWS KMS console, on detail page for the external key store and hover over the **Failed** value\.  
![\[Connection error code on the custom key store details page\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/connection-error-code.png)

### Connection error codes for external key stores<a name="xks-connection-error-codes"></a>

The following connection error codes apply to external key stores

`INTERNAL_ERROR`  
AWS KMS could not complete the request due to an internal error\. Retry the request\. For `ConnectCustomKeyStore` requests, disconnect the custom key store before trying to connect again\.

`INVALID_CREDENTIALS`  
One or both of the `XksProxyAuthenticationCredential` values is not valid on the specified external key store proxy\.

`NETWORK_ERRORS`  
Network errors are preventing AWS KMS from connecting the custom key store to its backing key store\.

`XKS_PROXY_ACCESS_DENIED`  
AWS KMS requests are denied access to the external key store proxy\. If the external key store proxy has authorization rules, verify that they permit AWS KMS to communicate with the proxy on your behalf\.

`XKS_PROXY_INVALID_CONFIGURATION`  
A configuration error is preventing the external key store from connecting to its proxy\. Verify the value of the `XksProxyUriPath`\.

`XKS_PROXY_INVALID_RESPONSE`  
AWS KMS cannot interpret the response from the external key store proxy\. If you see this connection error code repeatedly, notify your external key store proxy vendor\.

`XKS_PROXY_INVALID_TLS_CONFIGURATION`  
AWS KMS cannot connect to the external key store proxy because the TLS configuration is invalid\. Verify that the external key store proxy supports TLS 1\.2 or 1\.3\. Also, verify that the TLS certificate is not expired, that it matches the hostname in the `XksProxyUriEndpoint` value, and that it is signed by a trusted certificate authority included in the [Trusted Certificate Authorities](https://github.com/aws/aws-kms-xksproxy-api-spec/blob/main/TrustedCertificateAuthorities) list\.

`XKS_PROXY_NOT_REACHABLE`  
AWS KMS can't communicate with your external key store proxy\. Verify that the `XksProxyUriEndpoint` and `XksProxyUriPath` are correct\. Use the tools for your external key store proxy to verify that the proxy is active and available on its network\. Also, verify that your external key manager instances are operating properly\. Connection attempts fail with this connection error code if the proxy reports that all external key manager instances are unavailable\.

`XKS_PROXY_TIMED_OUT`  
AWS KMS can connect to the external key store proxy, but the proxy does not respond to AWS KMS in the time allotted\. If you see this connection error code repeatedly, notify your external key store proxy vendor\.

`XKS_VPC_ENDPOINT_SERVICE_INVALID_CONFIGURATION`  
The Amazon VPC endpoint service configuration doesn't conform to the requirements for an AWS KMS external key store\.  
+ The VPC endpoint service must be an endpoint service for interface endpoints in the caller's AWS account\.
+ It must have a network load balancer \(NLB\) connected to at least two subnets, each in a different Availability Zone\.
+ The `Allow principals` list must include the AWS KMS service principal for the Region, `cks.kms.<region>.amazonaws.com`, such as `cks.kms.us-east-1.amazonaws.com`\.
+ It must *not* require [acceptance](https://docs.aws.amazon.com/vpc/latest/privatelink/create-endpoint-service.html) of connection requests\.
+ It must have a private DNS name\. The private DNS name for an external key store with `VPC_ENDPOINT_SERVICE` connectivity must be unique in its AWS Region\.
+ The domain of the private DNS name must have a [verification status](https://docs.aws.amazon.com/vpc/latest/privatelink/verify-domains.html) of `verified`\.
+ The [TLS certificate](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/create-tls-listener.html) specifies the private DNS hostname at which the endpoint is reachable\.

`XKS_VPC_ENDPOINT_SERVICE_NOT_FOUND`  
AWS KMS can't find the VPC endpoint service that it uses to communicate with the external key store proxy\. Verify that the `XksProxyVpcEndpointServiceName` is correct and the AWS KMS service principal has service consumer permissions on the Amazon VPC endpoint service\.

## Latency and timeout errors<a name="fix-xks-latency"></a>

**Exceptions**: `CustomKeyStoreInvalidStateException` \(`CreateKey`\), `KMSInvalidStateException` \(cryptographic operations\), `XksProxyUriUnreachableException` \(management operations\)

[**Connection error codes**](#xks-connection-error-codes): `XKS_PROXY_NOT_REACHABLE`, `XKS_PROXY_TIMED_OUT`

When AWS KMS can't contact the proxy within the 250 millisecond timeout interval, it throws an exception\. `CreateCustomKeyStore` and `UpdateCustomKeyStore` generate a `XksProxyUriUnreachableException`\. [Cryptographic operations](use-xks-key.md) generate the standard `KMSInvalidStateException` with an error message that describes the problem\. If `ConnectCustomKeyStore` fails, AWS KMS returns a [connection error code](#fix-xks-connection) that describes the problem\. 

Timeout errors might be transient issues that can be resolved by retrying the request\. If the problem persists, verify that your external key store proxy is active and is connected to the network, and that its proxy URI endpoint, proxy URI path, and VPC endpoint service name \(if any\) are correct in your external key store\. Also, verify that your external key manager is close to the AWS Region for your external key store\. If you need to update any of these values, see [Editing external key store properties](update-xks-keystore.md)\.

To track latency patterns, use the [`XksProxyLatency`](monitoring-cloudwatch.md#metric-xks-proxy-latency) CloudWatch metric and the **Average latency** graph \(based on that metric\) in the [**Monitoring** section](xks-monitoring.md) of the AWS KMS console\. Your external key store proxy might also generate logs and metrics that track latency and timeouts\.


|  | 
| --- |
| `XksProxyUriUnreachableException`AWS KMS cannot communicate with the external key store proxy\. This might be a transient network issue\. If you see this error repeatedly, verify that your external key store proxy is active and is connected to the network, and that its endpoint URI is correct in your external key store\. | 
+ The external key store proxy didn't respond to an AWS KMS proxy API request within the 250 millisecond timeout interval\. This might indicate a transient network problem or an operational or performance problem with the proxy\. If retrying doesn't solve the problem, notify your external key store proxy administrator\.

Latency and timeout errors often manifest as connection failures\. When the [ConnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_ConnectCustomKeyStore.html) operation fails, the *connection state* of the external key store changes to `FAILED` and AWS KMS generates a *connection error code* that explains the error\. For a list of connection error codes and suggestions for resolving the errors, see [Connection error codes for external key stores](#xks-connection-error-codes)\. The connection codes lists for **All custom key stores** and **External key stores** apply to external key stores\. The following connection errors are related to latency and timeouts\.


|  | 
| --- |
| `XKS_PROXY_NOT_REACHABLE`\-or\-`CustomKeyStoreInvalidStateException`, `KMSInvalidStateException`, `XksProxyUriUnreachableException`AWS KMS cannot communicate with the external key store proxy\. Verify that your external key store proxy is active and is connected to the network, and that its URI path and endpoint URI or VPC service name are correct in your external key store\. | 

This error might occur for the following reasons:
+ The external key store proxy is not active and or not connected to the network\. 
+ or `KMSInvalidStateException`
+ There is an error in the [proxy URI endpoint](create-xks-keystore.md#require-endpoint), [proxy URI path](create-xks-keystore.md#require-path), or [VPC endpoint service name](create-xks-keystore.md#require-vpc-service-name) \(if applicable\) values in the external key store configuration\. To view the external key store configuration, use the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation or [view the detail page](view-xks-keystore.md) for the external key store in the AWS KMS console\.
+ There might be a network configuration error, such as a port error, on the network path between AWS KMS and the external key store proxy\. AWS KMS communicates with the external key store proxy on port 443\. This value is not configurable\.
+ When the external key store proxy reports \(in a [GetHealthStatus](keystore-external.md#concept-proxy-apis) response\) that all external key manager instances are `UNAVAILABLE`, the [ConnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_ConnectCustomKeyStore.html) operation fails with a `ConnectionErrorCode` of `XKS_PROXY_NOT_REACHABLE`\. For help, see your external key manager documentation\.
+ This error can result from a long physical distance between the external key manager and the AWS Region with the external key store\. The ping latency \(network round\-trip time \(RTT\)\) between the AWS Region and the external key manager must be no more than 35 milliseconds\. You might have to create an external key store in an AWS Region that is closer to the external key manager, or move the external key manager to a data center that is closer to the AWS Region\.


|  | 
| --- |
| `XKS_PROXY_TIMED_OUT`\-or\-`CustomKeyStoreInvalidStateException`, `KMSInvalidStateException`, `XksProxyUriUnreachableException`AWS KMS rejected the request because the external key store proxy did not respond in time\. Retry the request\. If you see this error repeatedly, report it to your external key store proxy administrator\. | 

This error might occur for the following reasons:
+ This error can result from a long physical distance between the external key manager and the external key store proxy\. If possible, move the external key store proxy closer to the external key manager\.
+ Timeout errors can occur when the proxy is not designed to handle the volume and frequency of requests from AWS KMS\. If your CloudWatch metrics indicate a persistent problem, notify your external key store proxy administrator\.


|  | 
| --- |
| `XKS_PROXY_TIMED_OUT`\-or\-`CustomKeyStoreInvalidStateException`, `KMSInvalidStateException`, `XksProxyUriUnreachableException` The external key store proxy did not respond to the request in the time allotted\. Retry the request\. If you see this error repeatedly, report it to your external key store proxy administrator\. | 
+ This error can result from a long physical distance between the external key manager and the external key store proxy\. If possible, move the external key store proxy closer to the external key manager\.

## Authentication credential errors<a name="fix-xks-credentials"></a>

**Exceptions**: `CustomKeyStoreInvalidStateException` \(`CreateKey`\), `KMSInvalidStateException` \(cryptographic operations\), `XksProxyIncorrectAuthenticationCredentialException` \(management operations other than `CreateKey`\)

You establish and maintain an authentication credential for AWS KMS on your external key store proxy\. Then you tell AWS KMS the credential values when you create an external key store\. To change the authentication credential, make the change on your external key store proxy\. Then [update the credential](update-xks-keystore.md#xks-edit-name) for your external key store\. If your proxy rotates the credential, you must [update the credential](update-xks-keystore.md#xks-edit-name) for your external key store\. 

If the external key store proxy won't authenticate a request signed with the [proxy authentication credential](keystore-external.md#concept-xks-credential) for your external key store, the effect depends on the request:
+ `CreateCustomKeyStore` and `UpdateCustomKeyStore` fail with an `XksProxyIncorrectAuthenticationCredentialException`\.
+ `ConnectCustomKeyStore` succeeds, but the connection fails\. The connection state is `FAILED` and the connection error code is `INVALID_CREDENTIALS`\. For details, see [External key store connection errors](#fix-xks-connection)\.
+ [Cryptographic operations](use-xks-key.md) fail with the standard `KMSInvalidStateException` that AWS KMS uses for all KMS keys in a custom key store\. The accompanying error message describes the problem\.


|  | 
| --- |
| The external key store proxy rejected the request because it could not authenticate AWS KMS\. Verify the credentials for your external key store and update if necessary\.  | 

This error might occur for the following reasons:
+ The access key ID or the secret access key for the external key store doesn't match the values established on the external key store proxy\. 

  To fix this error, [update the proxy authentication credential](update-xks-keystore.md#xks-edit-name) for your external key store\. You can make this change without disconnecting your external key store\.
+ A reverse proxy between AWS KMS and the external key store proxy could be manipulating HTTP headers in a manner that invalidates the SigV4 signatures\. To fix this error, notify your proxy administrator\.

## Key state errors<a name="fix-unavailable-xks-keys"></a>

**Exceptions**: `KMSInvalidStateException`

`KMSInvalidStateException` is used for two distinct purposes for KMS keys in custom key stores\. 
+ When a management operation, such as `CancelKeyDeletion`, fails and returns this exception, it indicates that the [key state](key-state.md) of the KMS key is not compatible with the operation\.
+ When a [cryptographic operation](concepts.md#cryptographic-operations) on a KMS key in a custom key store fails with `KMSInvalidStateException`, it can indicate a problem with the key state of the KMS key\. But AWS KMS cryptographic operation return `KMSInvalidStateException` for all networking and configuration errors with a KMS key in a custom key store\. To identify the problem, use the error message that accompanies the exception\.

To find the required key state for an AWS KMS API operations, see [Key states of AWS KMS keys](key-state.md)\. To find the key state of a KMS key, on the **Customer managed keys** page, view the **Status** field of the KMS key\. Or, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation and view the `KeyState` element in the response\. For details, see [Viewing keys](viewing-keys.md)\.

**Note**  
The key state of a KMS key in an external key store does not indicate anything about the status of its associated [external key](keystore-external.md#concept-external-key)\. For information about the external key status, use your external key manager and external key store proxy tools\.   
The `CustomKeyStoreInvalidStateException` refers to the [connection state](xks-connect-disconnect.md#xks-connection-state) of the external key store, not the [key state](key-state.md) of a KMS key\.

A cryptographic operation on a KMS key in a custom key store might fail because the key state of the KMS key is `Unavailable` or `PendingDeletion`\. \(Disabled keys generate a `DisabledException`\.\)
+ A KMS key has a `Disabled` key state only when you intentionally disable the KMS key in the AWS KMS console or by using the [DisableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) operation\. While a KMS key is disabled, you can view and manage the key, but you cannot use it in cryptographic operations\. To fix this problem, enable the key\. For details, see [Enabling and disabling keys](enabling-keys.md)\.
+ A KMS key has an `Unavailable` key state when the external key store is disconnected from its external key store proxy\. To fix an unavailable KMS key, [reconnect the external key store](xks-connect-disconnect.md)\. After the external key store is reconnected, the key state of the KMS keys in the external key store is automatically restored to its previous state, such as `Enabled` or `Disabled`\.
+ A KMS key has a `PendingDeletion` key state when it has been scheduled for deletion and is in its waiting period\. A key state error on a KMS key that is pending deletion indicates that the key should not be deleted, either because it's being used for encryption, or it is required for decryption\. To re\-enable the KMS key, cancel the scheduled deletion, and then [enable the key](enabling-keys.md)\. For details, see [Scheduling and canceling key deletion](deleting-keys-scheduling-key-deletion.md)\.

## Decryption errors<a name="fix-xks-decrypt"></a>

**Exceptions**: `KMSInvalidStateException`

When a [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation with a KMS key in an external key store fails, AWS KMS throws the standard `KMSInvalidStateException` that cryptographic operations use for all errors with KMS keys in a custom key store\. An error message that indicates the problem\.

To decrypt a ciphertext that was encrypted using [double encryption](keystore-external.md#concept-double-encryption), the external key manager uses the external key to decrypt the outer layer of ciphertext\. Then AWS KMS uses the AWS KMS key material in the KMS key to decrypt the inner layer of ciphertext\. An invalid or corrupt ciphertext can be rejected by the external key manager or AWS KMS\.

The following error messages accompany the `KMSInvalidStateException` when decryption fails\. It indicates a problem with the ciphertext or the optional encryption context in the request\.


|  | 
| --- |
| The external key store proxy rejected the request because the specified ciphertext or additional authenticated data is corrupted, missing, or otherwise invalid\. | 
+ When the external key store proxy or external key manager report that a ciphertext or its encryption context is invalid, it typically indicates a problem with the ciphertext or encryption context in the `Decrypt` request sent to AWS KMS\. For `Decrypt` operations, AWS KMS sends the proxy the same ciphertext and encryption context it receives in the `Decrypt` request\. 

  This error might be caused by a networking problem in transit, such as a flipped bit\. Retry the `Decrypt` request\. If the problem persists, verify that the ciphertext was not altered or corrupted\. Also, verify that the encryption context in the `Decrypt` request to AWS KMS matches the encryption context in the request that encrypted the data\.


|  | 
| --- |
| The ciphertext that the external key store proxy submitted for decryption, or the encryption context, is corrupted, missing, or otherwise invalid\. | 
+ When AWS KMS rejects the ciphertext that it received from the proxy, it indicates that the external key manager or proxy returned an invalid or corrupt ciphertext to AWS KMS\.

  This error might be caused by a networking problem in transit, such as a flipped bit\. Retry the `Decrypt` request\. If the problem persists, verify that the external key manager is operating properly, and that the external key store proxy does not alter the ciphertext that it receives from the external key manager before it returns it to AWS KMS\.

## External key errors<a name="fix-external-key"></a>

An [external key](keystore-external.md#concept-external-key) is a cryptographic key in the external key manager that serves as the external key material for a KMS key\. AWS KMS cannot directly access the external key\. It must ask the external key manager \(via the external key store proxy\) to use the external key to encrypt data or decrypt a ciphertext\.

You specify the ID of the external key in its external key manager when you create a KMS key in your external key store\. You cannot change the external key ID after the KMS key is created\. To prevent problems with the KMS key, the `CreateKey` operation asks the external key store proxy to verify the ID and configuration of the external key\. If the external key doesn't [fulfill the requirements](create-xks-keys.md#xks-key-requirements) for use with a KMS key, the `CreateKey` operation fails with an exception and error message that identifies the problem\. 

However, issues can occur after the KMS key is created\. If a cryptographic operation fails because of a problem with the external key, the operation fails and generates an KMSInvalidStateException with an error message that indicates the problem\.

### CreateKey errors for the external key<a name="fix-external-key-create"></a>

**Exceptions**: `XksKeyAlreadyInUseException`, `XksKeyNotFoundException`, `XksKeyInvalidConfigurationException`

The [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation attempts to verify the ID and properties of the external key that you provide in the **External key ID** \(console\) or `XksKeyId` \(API\) parameter\. This practice is designed to detect errors early before you try to use the external key with the KMS key\.

**External key in use** 

Each KMS key in an external key store must use a different external key\. When `CreateKey` recognizes that the external key ID \(XksKeyId\) for a KMS key is not unique in the external key store, it fails with an `XksKeyAlreadyInUseException`\. 

If you use multiple IDs for the same external key, `CreateKey` won't recognize the duplicate\. However, KMS keys with the same external key are not interoperable because they have different AWS KMS key material and metadata\. 

**External key not found** 

When the external key store proxy reports that it cannot find the external key using the external key ID \(XksKeyId\) for the KMS key, the `CreateKey` operation fails and generates an `XksKeyNotFoundException` with the following error message\.


|  | 
| --- |
| The external key store proxy rejected the request because it could not find the external key\. | 

This error might occur for the following reasons:
+ The ID of the external key \(`XksKeyId`\) for the KMS key might be invalid\. To find the ID for your external key proxy uses to identify the external key, see your external key store proxy or external key manager documentation\. 
+ The external key might have been deleted from your external key manager\. To investigate, use your external key manager tools\. If the external key is permanently deleted, use a different external key with the KMS key\. For a list or requirements for the external key, see [Requirements for a KMS key in an external key store](create-xks-keys.md#xks-key-requirements)\.

**External key requirements not met**

When the external key store proxy reports that the external key does not [fulfill the requirements](create-xks-keys.md#xks-key-requirements) for use with a KMS key, the `CreateKey` operation fails and generates an `XksKeyInvalidConfigurationException` with one of the following error messages\.


|  | 
| --- |
| The key spec of the external key must be AES\_256\. The key spec of specified external key is <key\-spec>\. | 
+ The external key must be a 256\-bit symmetric encryption key with a key spec of AES\_256\. If the specified external key is a different type, specify the ID of an external key that fulfills this requirement\. 


|  | 
| --- |
| The status of the external key must be ENABLED\. The status of specified external key is <status>\. | 
+ The external key must be enabled in the external key manager\. If the specified external key is not enabled, use your external key manager tools to enable it, or specify an enabled external key\.


|  | 
| --- |
| The key usage of the external key must include ENCRYPT and DECRYPT\. The key use of specified external key is <key\-usage>\. | 
+ The external key must be configured for encryption and decryption in the external key manager\. If the specified external key does not include these operations, use your external key manager tools to change the operations, or specify a different external key\.

### Cryptographic operation errors for the external key<a name="fix-external-key-crypto"></a>

**Exceptions**: `KMSInvalidStateException`

When the external key store proxy cannot find the external key associated with the KMS key, or the external key doesn't [fulfill the requirements](create-xks-keys.md#xks-key-requirements) for use with a KMS key, the cryptographic operation fails\. 

External key issues that are detected during a cryptographic operation are more difficult to resolve than external key issues detected before creating the KMS key\. You cannot change the external key ID after the KMS key is created\. If the KMS key has not yet encrypted any data, you can delete the KMS key and create a new one with a different external key ID\. However, ciphertext generated with the KMS key cannot be decrypted by any other KMS key, even one with the same external key, because keys will have different key metadata and different AWS KMS key material\. Instead, to the extent possible, use your external key manager tools to resolve the problem with the external key\. 

When the external key store proxy reports a problem with the external key, cryptographic operations return a `KMSInvalidStateException` with an error message that identifies the problem\.

**External key not found**

When the external key store proxy reports that it cannot find the external key using the external key ID \(XksKeyId\) for the KMS key, cryptographic operations return a `KMSInvalidStateException` with the following error message\. 


|  | 
| --- |
| The external key store proxy rejected the request because it could not find the external key\. | 

This error might occur for the following reasons:
+ The ID of the external key \(`XksKeyId`\) for the KMS key is no longer valid\. 

  To find the external key ID associated with your KMS key, [view the details of the KMS key](view-xks-key.md)\. To find the ID that your external key proxy uses to identify the external key, see your external key store proxy or external key manager documentation\.

  AWS KMS verifies the external key ID when it creates a KMS key in an external key store\. However, the ID might become invalid, especially if the external key ID value is an alias or mutable name\. You cannot change the external key ID associated with an existing KMS key\. To decrypt any ciphertext encrypted under the KMS key, you must re\-associate the external key with the existing external key ID\.

  If you have not yet used the KMS key to encrypt data, you can create a new KMS key with a valid external key ID\. However, if you have generated ciphertext with the KMS key, you cannot use any other KMS key to decrypt the ciphertext, even if uses the same external key\.
+ The external key might have been deleted from your external key manager\. To investigate, use your external key manager tools\. If possible, try to [recover the key material](fix-keystore.md#fix-keystore-recover-backing-key) from a copy or backup of your external key manager\. If the external key is permanently deleted, any ciphertext encrypted under the associated KMS key is unrecoverable\.

**External key configuration errors**

When the external key store proxy reports that the external key doesn't [fulfill the requirements](create-xks-keys.md#xks-key-requirements) for use with a KMS key, the cryptographic operation a `KMSInvalidStateException` with the one of the following error messages\. 


|  | 
| --- |
| The external key store proxy rejected the request because the external key does not support the requested operation\. | 
+ The external key must support both encryption and decryption\. If the key usage does not include encryption and decryption, use your external key manager tools to change the key usage\.


|  | 
| --- |
| The external key store proxy rejected the request because the external key is not enabled in the external key manager\. | 
+ The external key must be enabled and available for use in the external key manager\. If the status of the external key is not `Enabled`, use your external key manager tools to enable it\.

## Proxy issues<a name="fix-xks-proxy"></a>

**Exceptions**: 

 `CustomKeyStoreInvalidStateException` \(`CreateKey`\), `KMSInvalidStateException` \(cryptographic operations\), `UnsupportedOperationException`, `XksProxyUriUnreachableException`, `XksProxyInvalidResponseException` \(management operations other than `CreateKey`\)

The external key store proxy mediates all communication between AWS KMS and the external key manager\. It translates generic AWS KMS requests into a format that your external key manager can understand\. If the external key store proxy doesn't conform to the [AWS KMS External Key Store Proxy API Specification](https://github.com/aws/aws-kms-xksproxy-api-spec/), or if isn't operating properly, or can't communicate with AWS KMS, you won't be able to create or use KMS keys in your external key store\. 

While many errors mention the external key store proxy because of its critical role in the external key store architecture, those problem might originate in the external key manager or external key\. 

The issues in this section relate to problems with the design or operation of the external key store proxy\. Resolving these issues might require a change to the proxy software\. Consult your proxy administrator\. To help diagnose proxy issues, AWS KMS provides [XKS Proxy Text Client](https://github.com/aws-samples/aws-kms-xksproxy-test-client), an open source test client that verifies that your external key store proxy conforms to the [AWS KMS External Key Store Proxy API Specification](https://github.com/aws/aws-kms-xksproxy-api-spec/)\.


|  | 
| --- |
| `CustomKeyStoreInvalidStateException,`, `KMSInvalidStateException` or `XksProxyUriUnreachableException`The external key store proxy is in an unhealthy state\. If you see this message repeatedly, notify your external key store proxy administrator\. | 
+ This error can indicate an operational problem or software error in the external key store proxy\. You can find CloudTrail log entries for the AWS KMS API operation that generated each error\. This error might be resolved by retrying the operation\. However, if it persists, notify your external key store proxy administrator\.
+ When the external key store proxy reports \(in a [GetHealthStatus](keystore-external.md#concept-proxy-apis) response\) that all external key manager instances are `UNAVAILABLE`, attempts to create or update an external key store fail with this exception\. If this error persists, consult your external key manager documentation\.


|  | 
| --- |
| `CustomKeyStoreInvalidStateException`, `KMSInvalidStateException` or `XksProxyInvalidResponseException`AWS KMS cannot interpret the response from the external key store proxy\. If you see this error repeatedly, consult your external key store proxy administrator\. | 
+ AWS KMS operations generate this exception when the proxy returns an undefined response that AWS KMS cannot parse or interpret\. This error might occur occasionally due to temporarily external issues or sporadic network errors\. However, if it persists, it might indicate that the external key store proxy doesn't conform to the [AWS KMS External Key Store Proxy API Specification](https://github.com/aws/aws-kms-xksproxy-api-spec/)\. Notify your external key store administrator or vendor\.


|  | 
| --- |
|  `CustomKeyStoreInvalidStateException`, `KMSInvalidStateException` or `UnsupportedOperationException` The external key store proxy rejected the request because it does not support the requested cryptographic operation\. | 
+ The external key store proxy should support all [proxy APIs](keystore-external.md#concept-proxy-apis) defined in the [AWS KMS External Key Store Proxy API Specification](https://github.com/aws/aws-kms-xksproxy-api-spec/)\. This error indicates that the proxy does not support the operation that is related to the request\. Notify your external key store administrator or vendor\.

## Proxy authorization issues<a name="fix-xks-authorization"></a>

**Exceptions**: `CustomKeyStoreInvalidStateException`, `KMSInvalidStateException`

Some external key store proxies implement authorization requirements for the use of its external keys\. An external key store proxy is permitted, but not required, to design and implement an authorization scheme that allows particular users to request particular operations under certain conditions\. For example, a proxy might allow a user to encrypt with a particular external key, but not to decrypt with it\. For more information, see [External key store proxy authorization \(optional\)](authorize-xks-key-store.md#xks-proxy-authorization)\.

Proxy authorization is based on metadata that AWS KMS includes in its requests to the proxy\. The `awsSourceVpc` and `awsSourceVpce` fields are included in the metadata only when the request is from a VPC endpoint and only when the caller is in the same account as the KMS key\. 

```
"requestMetadata": {
    "awsPrincipalArn": string,
    "awsSourceVpc": string, // optional
    "awsSourceVpce": string, // optional
    "kmsKeyArn": string,
    "kmsOperation": string,
    "kmsRequestId": string,
    "kmsViaService": string // optional
}
```

When the proxy rejects a request due to an authorization failure, the related AWS KMS operation fails\. `CreateKey` generates a `CustomKeyStoreInvalidStateException`\. AWS KMS cryptographic operations generate a `KMSInvalidStateException`\. Both use the following error message:


|  | 
| --- |
| The external key store proxy denied access to the operation\. Verify that the user and the external key are both authorized for this operation, and try the request again\. | 
+ To resolve the error, use your external key manager or external key store proxy tools to determine why authorization failed\. Then, update the procedure that caused the unauthorized request or use your external key store proxy tools to update the authorization policy\. You cannot resolve this error in AWS KMS\.