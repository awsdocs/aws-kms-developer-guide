# Connecting and disconnecting an external key store<a name="xks-connect-disconnect"></a>

New external key stores are not connected\. To create and use AWS KMS keys in your external key store, you need to connect your external key store to its [external key store proxy](keystore-external.md#concept-xks-proxy)\. You can connect and disconnect your external key store at any time, and [view its connection state](view-xks-keystore.md)\.

While your external key store is disconnected, AWS KMS cannot communicate with your external key store proxy\. As a result, you can view and manage your external key store and its existing KMS keys\. However, you cannot create KMS keys in your external key store, or use its KMS keys in cryptographic operations\. You might need to disconnect your external key store at some point, such as when editing its properties, but plan accordingly\. Disconnecting the key store might disrupt the operation of AWS services that use its KMS keys\. 

You are not required to connect your external key store\. You can leave an external key store in a disconnected state indefinitely and connect it only when you need to use it\. However, you might want to test the connection periodically to verify that the settings are correct and it can be connected\.

When you disconnect a custom key store, the KMS keys in the key store become unusable right away \(subject to eventual consistency\)\. However, resources encrypted with [data keys](concepts.md#data-keys) protected by the KMS key are not affected until the KMS key is used again, such as to decrypt the data key\. This issue affects AWS services, many of which use data keys to protect your resources\. For details, see [How unusable KMS keys affect data keys](concepts.md#unusable-kms-keys)\.

**Note**  
External key stores are in a `DISCONNECTED` state only when the key store has never been connected or you explicitly disconnect it\. A `CONNECTED` state does not indicate that external key store or its supporting components are operating efficiently\. For information about the performance of your external key store components, see the graphs in **Monitoring** section of the detail page for each external key store\. For details, see [Monitoring an external key store](xks-monitoring.md)\.  
Your external key manager might provide additional methods of stopping and restarting communication between your AWS KMS external key store and your external key store proxy, or between your external key store proxy and external key manager\. For details, see your external key manager documentation\.

**Topics**
+ [Connecting an external key store](#about-xks-connecting)
+ [Disconnecting an external key store](#about-xks-disconnecting)
+ [Connection state](#xks-connection-state)
+ [Connect an external key store \(console\)](#connect-xks-console)
+ [Connect an external key store \(API\)](#connect-xks-api)
+ [Disconnect an external key store \(console\)](#disconnect-xks-console)
+ [Disconnect an external key store \(API\)](#disconnect-xks-api)

## Connecting an external key store<a name="about-xks-connecting"></a>

When your external key store is connected to its external key store proxy, you can [create KMS keys in your external key store](create-cmk-keystore.md) and use its existing KMS keys in [cryptographic operations](use-cmk-keystore.md)\. 

The process that connects an external key store to its external key store proxy differs based on the connectivity of the external key store\.
+ When you connect an external key store with [public endpoint connectivity](keystore-external.md#concept-xks-connectivity), AWS KMS sends a [GetHealthStatus request](keystore-external.md#concept-proxy-apis) to the external key store proxy to validate the [proxy URI endpoint](create-xks-keystore.md#require-endpoint), [proxy URI path](create-xks-keystore.md#require-path), and [proxy authentication credential](keystore-external.md#concept-xks-credential)\. A successful response from the proxy confirms that the [proxy URI endpoint](create-xks-keystore.md#require-endpoint) and [proxy URI path](create-xks-keystore.md#require-path) are accurate and accessible, and that the proxy authenticated the request signed with the [proxy authentication credential](keystore-external.md#concept-xks-credential) for the external key store\.
+ When you connect an external key store with [VPC endpoint service connectivity](plan-xks-keystore.md#xks-vpc-connectivity) to its external key store proxy, AWS KMS does the following: 
  + Confirms that the domain for the private DNS name specified in the [proxy URI endpoint](create-xks-keystore.md#require-endpoint) is [verified](vpc-connectivity.md#xks-private-dns)\. 
  + Creates an interface endpoint from an AWS KMS VPC to your VPC endpoint service\.
  + Creates a private hosted zone for the private DNS name specified in the proxy URI endpoint
  + Sends a [GetHealthStatus request](keystore-external.md#concept-proxy-apis) to the external key store proxy\. A successful response from the proxy confirms that the [proxy URI endpoint](create-xks-keystore.md#require-endpoint) and [proxy URI path](create-xks-keystore.md#require-path) are accurate and accessible, and that the proxy authenticated the request signed with the [proxy authentication credential](keystore-external.md#concept-xks-credential) for the external key store\.

The connect operation begins the process of connecting your custom key store, but connecting an external key store it its external proxy takes approximately five minutes\. A success response from the connect operation does not indicate that the external key store is connected\. To confirm that the connection was successful, use the AWS KMS console or the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/DescribeCustomKeyStores.html) operation to view the [connection state](#xks-connection-state) of external your key store\.

When the connection state is `FAILED`, a connection error code is displayed in the AWS KMS console and is added to the `DescribeCustomKeyStore` response\. For help interpreting connection error codes, see [Connection error codes for external key stores](xks-troubleshooting.md#xks-connection-error-codes)\.

## Disconnecting an external key store<a name="about-xks-disconnecting"></a>

When you disconnect an external key store with [VPC endpoint service connectivity](plan-xks-keystore.md#xks-vpc-connectivity) from its external key store proxy, AWS KMS deletes its interface endpoint to the VPC endpoint service and removes the network infrastructure that it created to support the connection\. No equivalent process is required for external key stores with public endpoint connectivity\. This action does not affect the VPC endpoint service or any of its supporting components, and it does not affect the external key store proxy or any external components\.

While the external key store is disconnected, AWS KMS does not send any requests to the external key store proxy\. The connection state of the external key store is `DISCONNECTED`\. The KMS keys in the disconnected external key store are in an [`UNAVAILABLE` key state](key-state.md) \(unless they are [pending deletion](deleting-keys.md)\), which means that they cannot be used in cryptographic operations\. However, you can still view and manage your external key store and its existing KMS keys\. 

The disconnected state is designed to be temporary and reversible\. You can reconnect your external key store at any time\. Typically, no reconfiguration is necessary\. However, if any properties of the associated external key store proxy have changed while it was disconnected, such as rotation of its [proxy authentication credential](keystore-external.md#concept-xks-credential), you must [edit the external key store settings](update-xks-keystore.md) before reconnecting\. 

**Note**  
While a custom key store is disconnected, all attempts to create KMS keys in the custom key store or to use existing KMS keys in cryptographic operations will fail\. This action can prevent users from storing and accessing sensitive data\.

To better estimate the effect of disconnecting your external key store, identify the KMS keys in the external key store and [determine their past use](deleting-keys-determining-usage.md)\.

You might disconnect an external key store for reasons such as the following:
+ **To edit its properties\.** You can edit the custom key store name, proxy URI path, and proxy authentication credential while the external key store is connected\. However, to edit the proxy connectivity type, proxy URI endpoint, or VPC endpoint service name, you must first disconnect the external key store\. For details, see [Editing external key store properties](update-xks-keystore.md)\.
+ **To stop all communication** between AWS KMS and the external key store proxy\. You can also stop communication between AWS KMS and your proxy by disabling your endpoint or VPC endpoint service\. In addition, your external key store proxy or key management software might provide additional mechanisms to prevent AWS KMS from communicating with the proxy or to prevent the proxy from accessing your external key manager\.
+ **To disable all KMS keys** in the external key store\. You can [disable and re\-enable KMS keys](enabling-keys.md) in an external key store by using the AWS KMS console or the [DisableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) operation\. These operations complete quickly \(subject to eventual consistency\), but they act on one KMS key at a time\. Disconnecting the external key store changes the key state of all KMS keys in the external key store to `Unavailable`, which prevents them from being used in any cryptographic operation\.
+ **To repair a failed connection attempt**\. If an attempt to connect an external key store fails \(the connection state of the custom key store is `FAILED`\), you must disconnect the external key store before you try to connect it again\.

## Connection state<a name="xks-connection-state"></a>

Connecting and disconnecting changes the *connection state* of your custom key store\. Connection state values are the same for AWS CloudHSM key stores and external key stores\. 

To view the connection state of your custom key store, use the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/DescribeCustomKeyStores.html) operation or AWS KMS console\. **Connection state** appears in each custom key store table, in the **General configuration** section of the detail page for each custom key store, and on the **Cryptographic configuration** tab of KMS keys in a custom key store\. For details, see [Viewing an AWS CloudHSM key store](view-keystore.md) and [Viewing an external key store](view-xks-keystore.md)\.

An custom key store can have one of the following connection states:
+ `CONNECTED`: The custom key store is connected to its backing key store\. You can create and use KMS keys in the custom key store\.

  The *backing key store* for an AWS CloudHSM key store is its associated AWS CloudHSM cluster\. The *backing key store* for an external key store is external key store proxy and the external key manager that it supports\.

  A CONNECTED state means that a connection succeeded and the custom key store has not been intentionally disconnected\. It does not indicate that the connection is operating properly\. For information about the status of the AWS CloudHSM cluster associated with your AWS CloudHSM key store, see [Getting CloudWatch metrics for AWS CloudHSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/hsm-metrics-cw.html) in the AWS CloudHSM User Guide\. For information about the status and operation of your external key store, see the graphs in the **Monitoring** section of the detail page for each external key store\. For details, see [Monitoring an external key store](xks-monitoring.md)\.
+ `CONNECTING`: The process of connecting an custom key store is in progress\. This is a transient state\.
+ `DISCONNECTED`: The custom key store has never been connected to its backing, or it was intentionally disconnected by using the AWS KMS console or the [DisconnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/DisconnectCustomKeyStores.html) operation\. 
+ `DISCONNECTING`: The process of disconnecting an custom key store is in progress\. This is a transient state\.
+ `FAILED`: An attempt to connect the custom key store failed\. The `ConnectionErrorCode` in the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/DescribeCustomKeyStores.html) response indicates the problem\.

To connect an custom key store, its connection state must be `DISCONNECTED`\. If the connection state is `FAILED`, use the `ConnectionErrorCode` to identify and resolve the problem\. Then disconnect the custom key store before trying to connect it again\. For help with connection failures, see [External key store connection errors](xks-troubleshooting.md#fix-xks-connection)\. For help responding to a connection error code, see [Connection error codes for external key stores](xks-troubleshooting.md#xks-connection-error-codes)\.

To view the connection error code:
+ In the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) response, view the value of the `ConnectionErrorCode` element\. This element appears in the `DescribeCustomKeyStores` response only when the `ConnectionState` is `FAILED`\.
+ To view the connection error code in the AWS KMS console, on detail page for the external key store and hover over the **Failed** value\.  
![\[Connection error code on the custom key store details page\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/connection-error-code.png)

## Connect an external key store \(console\)<a name="connect-xks-console"></a>

You can use the AWS KMS console to connect an external key store to its external key store proxy\. 

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Custom key stores**, **External key stores**\.

1. Choose the row of the external key store you want to connect\. 

   If the [connection state](#xks-connection-state) of the external key store is **FAILED**, you must [disconnect the external key store](disconnect-keystore.md#disconnect-keystore-console) before you connect it\.

1. From the **Key store actions** menu, choose **Connect**\.

The connection process typically takes about five minutes to complete\.When the operation completes, the [connection state](#xks-connection-state) changes to **CONNECTED**\. 

If the connection state is **Failed**, hover over the connection state to see the *connection error code*, which explains the cause of the error\. For help responding to a connection error code, see [ Connection error codes for external key stores  The following connection error codes apply to external key stores  

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
AWS KMS can't find the VPC endpoint service that it uses to communicate with the external key store proxy\. Verify that the `XksProxyVpcEndpointServiceName` is correct and the AWS KMS service principal has service consumer permissions on the Amazon VPC endpoint service\.  ](xks-troubleshooting.md#xks-connection-error-codes)\. To connect an external key store with a **Failed** connection state, you must first [disconnect the custom key store](disconnect-keystore.md#disconnect-keystore-console)\.

## Connect an external key store \(API\)<a name="connect-xks-api"></a>

To connect a disconnected external key store, use the [ConnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_ConnectCustomKeyStore.html) operation\. 

Before connecting, the [connection state](#xks-connection-state) of the external key store must be `DISCONNECTED`\. If the current connection state is `FAILED`, [disconnect the external key store](#disconnect-xks-api), and then connect it\. 

The connection process takes about five minutes to complete\. Unless it fails quickly, `ConnectCustomKeyStore` returns an HTTP 200 response and a JSON object with no properties\. However, this initial response does not indicate that the connection was successful\. To determine whether the external key store is connected, see the connection state in the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) response\. 

The examples in this section use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

To identify the external key store, use its custom key store ID\. You can find the ID on the **Custom key stores** page in the console or by using the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation\. Before running this example, replace the example ID with a valid one\.

```
$ aws kms connect-custom-key-store --custom-key-store-id cks-1234567890abcdef0
```

The `ConnectCustomKeyStore` operation does not return the `ConnectionState` in its response\. To verify that the external key store is connected, use the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation\. By default, this operation returns all custom keys stores in your account and Region\. But you can use either the `CustomKeyStoreId` or `CustomKeyStoreName` parameter \(but not both\) to limit the response to particular custom key stores\. A `ConnectionState` value of `CONNECTED` indicates that the external key store is connected to its external key store proxy\.

```
$ aws kms describe-custom-key-stores --custom-key-store-name ExampleXksVpc
{
    "CustomKeyStores": [
    {
      "CustomKeyStoreId": "cks-9876543210fedcba9",
      "CustomKeyStoreName": "ExampleXksVpc",
      "ConnectionState": "CONNECTED",
      "CreationDate": "2022-12-13T18:34:10.675000+00:00",
      "CustomKeyStoreType": "EXTERNAL_KEY_STORE",
      "XksProxyConfiguration": { 
        "AccessKeyId": "ABCDE98765432EXAMPLE",
        "Connectivity": "VPC_ENDPOINT_SERVICE",
        "UriEndpoint": "https://example-proxy-uri-endpoint-vpc",
        "UriPath": "/example/prefix/kms/xks/v1",
        "VpcEndpointServiceName": "com.amazonaws.vpce.us-east-1.vpce-svc-example"
      }
    }
  ]
}
```

If the `ConnectionState` value in the `DescribeCustomKeyStores` response is `FAILED`, the `ConnectionErrorCode` element indicates the reason for the failure\. 

In the following example, the `XKS_VPC_ENDPOINT_SERVICE_NOT_FOUND` value for the `ConnectionErrorCode` indicates that AWS KMS can't find the VPC endpoint service that it uses to communicate with the external key store proxy\. Verify that the `XksProxyVpcEndpointServiceName` is correct, the AWS KMS service principal is an allowed principal on the Amazon VPC endpoint service, and that the VPC endpoint service does not require acceptance of connection requests\. For help responding to a connection error code, see [Connection error codes for external key stores](xks-troubleshooting.md#xks-connection-error-codes)\.

```
$ aws kms describe-custom-key-stores --custom-key-store-name ExampleXksVpc
{
    "CustomKeyStores": [
    {
      "CustomKeyStoreId": "cks-9876543210fedcba9",
      "CustomKeyStoreName": "ExampleXksVpc",
      "ConnectionState": "FAILED",
      "ConnectionErrorCode": "XKS_VPC_ENDPOINT_SERVICE_NOT_FOUND",
      "CreationDate": "2022-12-13T18:34:10.675000+00:00",
      "CustomKeyStoreType": "EXTERNAL_KEY_STORE",
      "XksProxyConfiguration": { 
        "AccessKeyId": "ABCDE98765432EXAMPLE",
        "Connectivity": "VPC_ENDPOINT_SERVICE",
        "UriEndpoint": "https://example-proxy-uri-endpoint-vpc",
        "UriPath": "/example/prefix/kms/xks/v1",
        "VpcEndpointServiceName": "com.amazonaws.vpce.us-east-1.vpce-svc-example"
      }
    }
  ]
}
```

## Disconnect an external key store \(console\)<a name="disconnect-xks-console"></a>

You can use the AWS KMS console to connect an external key store to its external key store proxy\. This process takes about 5 minutes to complete\. 

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Custom key stores**, **External key stores**\.

1. Choose the row of the external key store you want to disconnect\. 

1. From the **Key store actions** menu, choose **Disconnect**\.

When the operation completes, the connection state changes from **DISCONNECTING** to **DISCONNECTED**\. If the operation fails, an error message appears that describes the problem and provides help on how to fix it\. If you need more help, see [External key store connection errors](xks-troubleshooting.md#fix-xks-connection)\.

## Disconnect an external key store \(API\)<a name="disconnect-xks-api"></a>

To disconnect a connected external key store, use the [DisconnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisconnectCustomKeyStore.html) operation\. If the operation is successful, AWS KMS returns an HTTP 200 response and a JSON object with no properties\. The process takes about five minutes to complete\. To find the connection state of the external key store, use the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation\.

The examples in this section use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

This example disconnects an external key store with VPC endpoint service connectivity\. Before running this example, replace the example custom key store ID with a valid one\.

```
$ aws kms disconnect-custom-key-store --custom-key-store-id cks-1234567890abcdef0
```

To verify that the external key store is disconnected, use the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation\. By default, this operation returns all custom keys stores in your account and Region\. But you can use either the `CustomKeyStoreId` and `CustomKeyStoreName` parameter \(but not both\) to limit the response to particular custom key stores\. The `ConnectionState` value of `DISCONNECTED` indicates that this example external key store is no longer connected to its external key store proxy\.

```
$ aws kms describe-custom-key-stores --custom-key-store-name ExampleXksVpc
{
    "CustomKeyStores": [
    {
      "CustomKeyStoreId": "cks-9876543210fedcba9",
      "CustomKeyStoreName": "ExampleXksVpc",
      "ConnectionState": "DISCONNECTED",
      "CreationDate": "2022-12-13T18:34:10.675000+00:00",
      "CustomKeyStoreType": "EXTERNAL_KEY_STORE",
      "XksProxyConfiguration": { 
        "AccessKeyId": "ABCDE98765432EXAMPLE",
        "Connectivity": "VPC_ENDPOINT_SERVICE",
        "UriEndpoint": "https://example-proxy-uri-endpoint-vpc",
        "UriPath": "/example/prefix/kms/xks/v1",
        "VpcEndpointServiceName": "com.amazonaws.vpce.us-east-1.vpce-svc-example"
      }
    }
  ]
}
```