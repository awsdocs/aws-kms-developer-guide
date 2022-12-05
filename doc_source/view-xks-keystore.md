# Viewing an external key store<a name="view-xks-keystore"></a>

You can view external key stores in each account and Region by using the AWS KMS console or by using the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation\.

When you view an external key store, you can see the following:
+ Basic information about the key store, including its friendly name, ID, key store type, and creation date\.
+ Configuration information for the [external key store proxy](keystore-external.md#concept-xks-proxy), including the [connectivity type](keystore-external.md#concept-xks-connectivity), [proxy URI endpoint](create-xks-keystore.md#require-endpoint) and [path](create-xks-keystore.md#require-path), and the [access key ID](keystore-external.md#concept-xks-credential) of your current [proxy authentication credential](keystore-external.md#concept-xks-credential)\.
+ If the external key store proxy uses [VPC endpoint service connectivity](plan-xks-keystore.md#xks-vpc-connectivity), the console displays the name of the VPC endpoint service\.
+ The current [connection state](xks-connect-disconnect.md#xks-connection-state)\. 
**Note**  
A connection state value of **Disconnected** indicates that the external key store has never been connected, or it was intentionally disconnected from its external key store proxy\. However, if your attempts to use a KMS key in a connected external key store fail, that might indicate a problem with the external key store or its proxy\. For help, see [External key store connection errors](xks-troubleshooting.md#fix-xks-connection)\.
+ A [Monitoring](xks-monitoring.md) section with graphs of [Amazon CloudWatch metrics](monitoring-cloudwatch.md#kms-metrics) designed to help you detect and resolve issues with your external key store\. For help interpreting the graphs, using them in your planning and troubleshooting, and creating CloudWatch alarms based on the metrics in the graphs, see [Monitoring an external key store](xks-monitoring.md)\.

See also:
+ [Viewing KMS keys in an external key store](view-xks-key.md)
+ [Logging AWS KMS API calls with AWS CloudTrail](logging-using-cloudtrail.md)

**Topics**
+ [External key store properties](#view-xks-properties)
+ [View an external key store \(console\)](#view-xks-keystore-console)
+ [View an external key store \(API\)](#view-xks-keystore-api)

## External key store properties<a name="view-xks-properties"></a>

The following properties of an external key store are visible in the AWS KMS console and the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) response\. 

### Custom key store properties<a name="view-xks-custom-key-store"></a>

The following values appear in the **General configuration** section of the detail page for each custom key store\.These properties apply to all custom key stores, including AWS CloudHSM key stores and external key stores\.

**Custom key store ID**  
A unique ID that AWS KMS assigns to the custom key store\.

**Custom key store name**  
A friendly name that you assign to the custom key store when you create it\. You can change this value at any time\.

**Custom key store type**  
The type of custom key store\. Valid values are AWS CloudHSM \(`AWS_CLOUDHSM`\) or External key store \(`EXTERNAL_KEY_STORE`\)\. You cannot change the type after you create the custom key store\.

**Creation date**  
The date that the custom key store was created\. This date is displayed in local time for the AWS Region\. 

**Connection state**  
Indicates whether the custom key store is connected to its backing key store\. The connection state is `DISCONNECTED` only if the custom key store has never been connected to its backing key store, or it has been intentionally disconnected\. For details, see [Connection state](xks-connect-disconnect.md#xks-connection-state)\.

### External key store configuration properties<a name="view-xks-configuration"></a>

The following values appear in the **External key store proxy configuration** section of the detail page for each external key store and in the `XksProxyConfiguration` element of the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) response\. For a detailed description of each field, including uniqueness requirements and help with determining the correct value for each field, see [Assemble the prerequisites](create-xks-keystore.md#xks-requirements) in the *Creating an external key store* topic\.

**Proxy connectivity**  
Indicates whether the external key store uses [public endpoint connectivity](plan-xks-keystore.md#xks-connectivity-public-endpoint) or [VPC endpoint service connectivity](plan-xks-keystore.md#xks-vpc-connectivity)\.

**Proxy URI endpoint**  
The endpoint that AWS KMS uses to connect to your [external key store proxy](keystore-external.md#concept-xks-proxy)\. 

**Proxy URI path**  
The path from the proxy URI endpoint where AWS KMS sends [proxy API requests](keystore-external.md#concept-proxy-apis)\.

**Proxy credential: Access key ID**  
Part of the [proxy authentication credential](keystore-external.md#concept-xks-credential) that you establish on your external key store proxy\. The access key ID identifies the secret access key in the credential\.   
AWS KMS uses the SigV4 signing process and the proxy authentication credential to sign its requests to your external key store proxy\. The credential in the signature allows the external key store proxy to authenticate requests on your behalf from AWS KMS\.

**VPC endpoint service name**  
The name of the Amazon VPC endpoint service that supports your external key store\. This value appears only when the external key store uses [VPC endpoint service connectivity](plan-xks-keystore.md#xks-vpc-connectivity)\. You can locate your external key store proxy in the VPC or use the VPC endpoint service to communicate securely with your external key store proxy\.

## View an external key store \(console\)<a name="view-xks-keystore-console"></a>

To view the external key stores in a given account and Region, use the following procedure\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Custom key stores**, **External key stores**\.

1. To view detailed information about an external key store, choose the key store name\.

## View an external key store \(API\)<a name="view-xks-keystore-api"></a>

To view your external key stores, use the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation\. By default, this operation returns all custom key stores in the account and Region\. But you can use either the `CustomKeyStoreId` or `CustomKeyStoreName` parameter \(but not both\) to limit the output to a particular custom key store\. 

For custom key stores, the output consists of the custom key store ID, name, and type, and the [connection state](xks-connect-disconnect.md#xks-connection-state) of the key store\. If the connection state is `FAILED`, the output also includes a `ConnectionErrorCode` that describes the reason for the error\. For help interpreting the `ConnectionErrorCode` for an external key store, see [Connection error codes for external key stores](xks-troubleshooting.md#xks-connection-error-codes)\.

For external key stores, the output also includes the `XksProxyConfiguration` element\. This element includes the [connectivity type](create-xks-keystore.md#require-connectivity), [proxy URI endpoint](create-xks-keystore.md#require-endpoint), [proxy URI path](create-xks-keystore.md#require-path), and the access key ID of the [proxy authentication credential](keystore-external.md#concept-xks-credential)\.

The examples in this section use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

For example, the following command returns all custom key stores in the account and Region\. You can use the `Limit` and `Marker` parameters to page through the custom key stores in the output\.

```
$ aws kms describe-custom-key-stores
```

The following command uses the `CustomKeyStoreName` parameter to get only the example external key store with the `ExampleXksPublic` friendly name\. This example key store uses public endpoint connectivity\. It is connected to its external key store proxy\. 

```
$ aws kms describe-custom-key-stores --custom-key-store-name ExampleXksPublic
{
    "CustomKeyStores": [
    {
      "CustomKeyStoreId": "cks-1234567890abcdef0",
      "CustomKeyStoreName": "ExampleXksPublic",
      "ConnectionState": "CONNECTED",    
      "CreationDate": "2022-12-14T20:17:36.419000+00:00",
      "CustomKeyStoreType": "EXTERNAL_KEY_STORE",
      "XksProxyConfiguration": { 
        "AccessKeyId": "ABCDE12345670EXAMPLE",
        "Connectivity": "PUBLIC_ENDPOINT",
        "UriEndpoint": "https://xks.example.com:6443",
        "UriPath": "/example/prefix/kms/xks/v1"
      }
    }
  ]
}
```

The following command gets an example external key store with VPC endpoint service connectivity\. In this example, the external key store is connected to its external key store proxy\. 

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

A [`ConnectionState`](xks-connect-disconnect.md#xks-connection-state) of `Disconnected` indicates that an external key store has never been connected or it was intentionally disconnected from its external key store proxy\. However, if attempts to use a KMS key in a connected external key store fail, that might indicate a problem with the external key store proxy or other external components\.

If the `ConnectionState` of the external key store is `FAILED`, the `DescribeCustomKeyStores` response includes a `ConnectionErrorCode` element that explains the reason for the error\.

For example, in the following output, the `XKS_PROXY_TIMED_OUT` value indicates AWS KMS can connect to the external key store proxy, but the connection failed because the external key store proxy did not respond to AWS KMS in the time allotted\. If you see this connection error code repeatedly, notify your external key store proxy vendor\. For help with this and other connection error failures, see [Troubleshooting external key stores](xks-troubleshooting.md)\.

```
$ aws kms describe-custom-key-stores --custom-key-store-name ExampleXksVpc
{
    "CustomKeyStores": [
    {
      "CustomKeyStoreId": "cks-9876543210fedcba9",
      "CustomKeyStoreName": "ExampleXksVpc",
      "ConnectionState": "FAILED",
      "ConnectionErrorCode": "XKS_PROXY_TIMED_OUT",
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