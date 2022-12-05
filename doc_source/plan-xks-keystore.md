# Planning an external key store<a name="plan-xks-keystore"></a>

Before creating your external key store, choose the connectivity option that determines how AWS KMS communicates with your external key store components\. The connectivity option that you choose determines the remainder of the planning process\.

**Learn more**:
+ Review the process for creating an external key store, including [assembling the prerequisites](create-xks-keystore.md#xks-requirements)\. It will help you to ensure that you have all of the components you need when you create your external key store\.
+ Learn how to [control access to your external key store](authorize-xks-key-store.md), including the permissions that external key store administrators and users require\. 
+ Learn about the [Amazon CloudWatch metrics and dimensions](monitoring-cloudwatch.md#kms-metrics) that AWS KMS records for external key stores\. We strongly recommend that you create alarms to monitor your external key store so you can detect the early signs of performance and operational problems\.

## Choosing a proxy connectivity option<a name="choose-xks-connectivity"></a>

If you are creating an external key store, you need to determine how AWS KMS communicates with your [external key store proxy](keystore-external.md#concept-xks-proxy)\. This choice will determine which components you need and how you configure them\. AWS KMS supports the following connectivity options\. Choose the option that meets your performance and security goals\.

Before you begin, [confirm that you need an external key store](keystore-external.md#do-i-need-xks)\. Most customer can use KMS keys backed by AWS KMS key material\.

**Note**  
If your external key store proxy is built into your external key manager, your connectivity might be predetermined\. For guidance, consult the documentation for your external key manager or external key store proxy\.

You can [change your external key store proxy connectivity option](update-xks-keystore.md) even on an operating external key store\. However, the process must be carefully planned and executed to minimize disruption, avoid errors, and ensure continued access to the cryptographic keys that encrypt your data\.

### Public endpoint connectivity<a name="xks-connectivity-public-endpoint"></a>

AWS KMS connects to the external key store proxy \(XKS proxy\) over the internet using a public endpoint\.

This connectivity option is easier to set up and maintain, and it aligns well with some models of key management\. However, it might not fulfill the security requirements of some organizations\.

![\[Public endpoint connectivity\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/xks-public-endpoint-60.png)

**Requirements**

If you choose public endpoint connectivity, the following are required\. 
+ Your external key store proxy must be reachable at a publicly routable endpoint\. 
+ You can use the same public endpoint for multiple external key stores provided that they use different [proxy URI path](create-xks-keystore.md#require-path) values\. 
+ You cannot use the same endpoint for an external key store with public endpoint connectivity and any external key store with VPC endpoint services connectivity in the same AWS Region, even if the key stores are in different AWS accounts\.
+ You must obtain a TLS certificate issued by a public certificate authority supported for external key stores\. For a list, see [Trusted Certificate Authorities](https://github.com/aws/aws-kms-xksproxy-api-spec/blob/main/TrustedCertificateAuthorities)\. 

  The subject common name \(CN\) on the TLS certificate must match the domain name in the [proxy URI endpoint](create-xks-keystore.md#require-endpoint) for the external key store proxy\. For example, if the public endpoint is `https://myproxy.xks.example.com`, the TLS, the CN on the TLS certificate must be `myproxy.xks.example.com` or `*.xks.example.com`\.
+ Ensure that any firewalls between AWS KMS and the external key store proxy allow traffic to and from port 443 on the proxy\. AWS KMS communicates on port 443\. This value is not configurable\.

For all requirements for an external key store, see the [Assemble the prerequisites](create-xks-keystore.md#xks-requirements)\.

### VPC endpoint service connectivity<a name="xks-vpc-connectivity"></a>

AWS KMS connects to the external key store proxy \(XKS proxy\) by creating an interface endpoint to an Amazon VPC endpoint service that you create and configure\. You are responsible for [creating the VPC endpoint service](vpc-connectivity.md) and for connecting your VPC to your external key manager\.

Your endpoint service can use any of the [supported network\-to\-Amazon VPC options](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/network-to-amazon-vpc-connectivity-options.html) for communications, including [AWS Direct Connect](https://docs.aws.amazon.com/directconnect/latest/UserGuide/)\. 

This connectivity option is more complicated to set up and maintain\. But it uses AWS PrivateLink, which enables AWS KMS to privately connect to your Amazon VPC and your external key store proxy without using the public internet\.

You can locate your external key store proxy in your Amazon VPC\.

![\[VPC endpoint service connectivity - XKS proxy in your VPC\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/xks-proxy-in-vpc-60.png)

Or, locate your external key store proxy outside of AWS and use your Amazon VPC endpoint service only for secure communication with AWS KMS\.

![\[VPC endpoint service connectivity - XKS proxy outside of AWS\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/xks-proxy-via-vpc-60.png)