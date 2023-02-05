# Configuring VPC endpoint service connectivity<a name="vpc-connectivity"></a>

Use the guidance in this section to create and configure the AWS resources and related components that are required for an external key store that uses [VPC endpoint service connectivity](plan-xks-keystore.md#xks-vpc-connectivity)\. The resources listed for this connectivity option are a supplement to the [resources required for all external key stores](create-xks-keystore.md#xks-requirements)\. After you create and configure the required resources, you can [create your external key store](create-xks-keystore.md)\.

You can locate your external key store proxy in your Amazon VPC or locate the proxy outside of AWS and use your VPC endpoint service for communication\.

Before you begin, [confirm that you need an external key store](keystore-external.md#do-i-need-xks)\. Most customer can use KMS keys backed by AWS KMS key material\.

**Note**
Some of the elements required for VPC endpoint service connectivity might be included in your external key manager\. Also, your software might have additional configuration requirements\. Before creating and configuring the AWS resources in this section, consult your proxy and key manager documentation\.

**Topics**
+ [Requirements for VPC endpoint service connectivity](#xks-vpce-service-requirements)
+ [Creating an Amazon VPC and subnets](#xks-create-vpc)
+ [Creating a target group](#xks-target-group)
+ [Creating a network load balancer](#xks-nlb)
+ [Creating a VPC endpoint service](#xks-vpc-svc)
+ [Verifying your private DNS name domain](#xks-private-dns)
+ [Authorizing AWS KMS to connect to the VPC endpoint service](#xks-vpc-authorize-kms)

## Requirements for VPC endpoint service connectivity<a name="xks-vpce-service-requirements"></a>

If you choose VPC endpoint service connectivity for your external key store, the following resources are required\.

To minimize network latency, create your AWS components in the [supported AWS Region](keystore-external.md#xks-regions) that is closest to your [external key manager](keystore-external.md#concept-ekm)\. If possible, choose a Region with a network round\-trip time \(RTT\) of 35 milliseconds or less\.
+ An Amazon VPC that is connected to your external key manager\. It must have at least two private [subnets](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) in two different Availability Zones\.

  You can use an existing Amazon VPC for your external key store, provided that it [fulfills the requirements](#xks-vpc-requirements) for use with an external key store\. Multiple external key stores can share an Amazon VPC, but each external key store must have its own VPC endpoint service and private DNS name\.
+ An [Amazon VPC endpoint service powered by AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/privatelink-share-your-services.html) with a [network load balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html) and [target group](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html)\.

  The endpoint service cannot require acceptance\. Also, you must add AWS KMS as an allowed principal\. This allows AWS KMS to create interface endpoints so it can communicate with your external key store proxy\.
+ A private DNS name for the VPC endpoint service that is unique in its AWS Region\.

  The private DNS name must be a subdomain of a higher\-level public domain\. For example, if the private DNS name is `myproxy-private.xks.example.com`, it must be a subdomain of a public domain such as `xks.example.com` or `example.com`\.

  You must [verify ownership](#xks-private-dns) of the DNS domain for private DNS name\.
+ A TLS certificate issued by a [supported public certificate authority](https://github.com/aws/aws-kms-xksproxy-api-spec/blob/main/TrustedCertificateAuthorities) for your external key store proxy\.

  The subject common name \(CN\) on the TLS certificate must match the private DNS name\. For example, if the private DNS name is `myproxy-private.xks.example.com`, the CN on the TLS certificate must be `myproxy-private.xks.example.com` or `*.xks.example.com`\.

For all requirements for an external key store, see the [Assemble the prerequisites](create-xks-keystore.md#xks-requirements)\.

## Creating an Amazon VPC and subnets<a name="xks-create-vpc"></a>

VPC endpoint service connectivity requires an Amazon VPC that is connected to your external key manager with at least two private subnets\. You can create an Amazon VPC or use an existing Amazon VPC that fulfills the requirements for external key stores\. For help with creating a new Amazon VPC, see [Create a VPC](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-vpcs.html#Create-VPC) in the *Amazon Virtual Private Cloud User Guide*\.

### Requirements for your Amazon VPC<a name="xks-vpc-requirements"></a>

To work with external key stores using VPC endpoint service connectivity, the Amazon VPC must have the following properties:
+ Must be in the same AWS account and [supported Region](keystore-external.md#xks-regions) as your external key store\.
+ Requires at least two private subnets, each in a different Availability Zone\.
+ The private IP address range of your Amazon VPC must not overlap with the private IP address range of the data center hosting your [external key manager](keystore-external.md#concept-ekm)\.
+ All components must use IPv4\.

You have many options for connecting the Amazon VPC to your external key store proxy\. Choose an option that meets your performance and security needs\. For a list, see [Connect your VPC to other networks](https://docs.aws.amazon.com/vpc/latest/userguide/extend-intro.html) and [Network\-to\-Amazon VPC connectivity options](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/network-to-amazon-vpc-connectivity-options.html)\. For more details, see [AWS Direct Connect](https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html), and the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/)\.

### Creating an Amazon VPC for your external key store<a name="xks-vpc-create"></a>

Use the following instructions to create the Amazon VPC for your external key store\. An Amazon VPC is required only if you choose the [VPC endpoint service connectivity](plan-xks-keystore.md#choose-xks-connectivity) option\. You can use an existing Amazon VPC that fulfills the requirements for an external key store\.

Follow the instructions in the [Create a VPC, subnets, and other VPC resources](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-vpcs.html#create-vpc-and-other-resources) topic using the following required values\. For other fields, accept the default values and provide names as requested\.


| Field | Value |
| --- | --- |
| IPv4 CIDR block | Enter the IP addresses for your VPC\. The private IP address range of your Amazon VPC must not overlap with the private IP address range of the data center hosting your [external key manager](keystore-external.md#concept-ekm)\. |
| Number of Availability Zones \(AZs\) | 2 or more |
| Number of public subnets |  None are required \(0\)  |
| Number of private subnets | One for each AZ |
| NAT gateways | None are required\. |
| VPC endpoints | None are required\. |
| Enable DNS hostnames | Yes |
| Enable DNS resolution | Yes |

Be sure to test your VPC communication\. For example, if your external key store proxy is not located in your Amazon VPC, create an Amazon EC2 instance in your Amazon VPC, verify that the Amazon VPC can communicate with your external key store proxy\.

### Connecting the VPC to the external key manager<a name="xks-vpc-to-ekm"></a>

Connect the VPC to the data center that hosts your external key manager using any of the [network connectivity options](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/network-to-amazon-vpc-connectivity-options.html) that Amazon VPC supports\. Ensure that the Amazon EC2 instance in the VPC \(or the external key store proxy, if it is in the VPC\), can communicate with the data center and the external key manager\.

## Creating a target group<a name="xks-target-group"></a>

Before you create the required VPC endpoint service, create its required components, a network load balancer \(NLB\) and a target group\. The network load balancer \(NLB\) distributes requests among multiple healthy targets, any of which can service the request\. In this step, you create a target group with at least two hosts for your external key store proxy, and register your IP addresses with the target group\.

Follow the instructions in the [Configure a target group](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/create-network-load-balancer.html#configure-target-group) topic using the following required values\. For other fields, accept the default values and provide names as requested\.


| Field | Value |
| --- | --- |
| Target type | IP addresses |
| Protocol | TLS |
| Port |  443  |
| IP address type | IPv4 |
| VPC | Choose the VPC where you will create the VPC endpoint service for your external key store\. |
| Health check protocol and path | Your health check protocol and path will differ with your external key store proxy configuration\. Consult the documentation for your external key manager or external key store proxy\.For general information about configuring health checks for your target groups, see [Health checks for your target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-health-checks.html) in the Elastic Load Balancing User Guide for Network Load Balancers\. |
| Network | Other private IP address |
| IPv4 address | The private addresses of your external key store proxy |
| Ports | 443 |

## Creating a network load balancer<a name="xks-nlb"></a>

The network load balancer distributes the network traffic, including requests from AWS KMS to your external key store proxy, to the configured targets\.

Follow the instructions in the [Configure a load balancer and a listener](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/create-network-load-balancer.html#configure-load-balancer) topic to configure and add a listener and create a load balancer using the following required values\. For other fields, accept the default values and provide names as requested\.


| Field | Value |
| --- | --- |
| Scheme | Internal |
| IP address type | IPv4 |
| Network mapping |  Choose the VPC where you will create the VPC endpoint service for your external key store\.  |
| Mapping | Choose both of the availability zones \(at least two\) that you configured for your VPC subnets\. Verify the subnet names and private IP address\. |
| Protocol | TLS |
| Port | 443 |
| Default action: Forward to | Choose the [target group](#xks-target-group) for your network load balancer\. |

## Creating a VPC endpoint service<a name="xks-vpc-svc"></a>

Typically, you create an endpoint to a service\. However, when you create a VPC endpoint service, you are the provider, and AWS KMS creates an endpoint to your service\. For an external key store, create a VPC endpoint service with the network load balancer that you created in the previous step\. The VPC endpoint service must must be in the same AWS account and [supported Region](keystore-external.md#xks-regions) as your external key store\.

Multiple external key stores can share an Amazon VPC, but each external key store must have its own VPC endpoint service and private DNS name\.

Follow the instructions in the [Create an endpoint service](https://docs.aws.amazon.com/vpc/latest/privatelink/create-endpoint-service.html#create-endpoint-service-nlb) topic to create your VPC endpoint service with the following required values\. For other fields, accept the default values and provide names as requested\.


| Field | Value |
| --- | --- |
| Load balancer type | Network |
| Available load balancers | Choose the [network load balancer](#xks-nlb) that you created in the previous step\.If your new load balancer does not appear in the list, verify that its state is active\. It might take a few minutes for the load balancer state to change from provisioning to active\. |
| Acceptance required | False\. Uncheck the check box\.*Do not require acceptance*\. AWS KMS cannot connect to the VPC endpoint service without a manual acceptance\. If acceptance is required, attempts to [create the external key store](create-xks-keystore.md) fail with an `XksProxyInvalidConfigurationException` exception\.  |
| Enable private DNS name | Associate a private DNS name with the service |
| Private DNS name | Enter a private DNS name that is unique in its AWS Region\. The private DNS name must be a subdomain of a higher level public domain\. For example, if the private DNS name is `myproxy-private.xks.example.com`, it must be a subdomain of a public domain such as `xks.example.com` or `example.com`\.This private DNS name must match the subject common name \(CN\) in the TLS certificate configured on your external key store proxy\. For example, if the private DNS name is `myproxy-private.xks.example.com`, the CN on the TLS certificate must be `myproxy-private.xks.example.com` or `*.xks.example.com`\.If the certificate and private DNS name do not match, attempts to connect an external key store to its external key store proxy fail with a connection error code of `XKS_PROXY_INVALID_TLS_CONFIGURATION`\. For details, see [General configuration errors](xks-troubleshooting.md#fix-xks-gen-configuration)\. |
| Supported IP address types | IPv4 |

## Verifying your private DNS name domain<a name="xks-private-dns"></a>

When you create your VPC endpoint service, its domain verification status is `pendingVerification`\. Before using the VPC endpoint service to create an external key store, this status must be `verified`\. To verify that you own the domain associated with your private DNS name, you must create a TXT record in a public DNS server\.

For example, if the private DNS name for your VPC endpoint service is `myproxy-private.xks.example.com`, you must create a TXT record in a public domain, such as `xks.example.com` or `example.com`, whichever is public\. AWS PrivateLink looks for the TXT record first on `xks.example.com` and then on `example.com`\.

**Tip**
After you add a TXT record, it might take a few minutes for the **Domain verification status** value to change from `pendingVerification` to `verify`\.

To begin, find the verification status of your domain using either of the following methods\. Valid values are `verified`, `pendingVerification`, and `failed`\.
+ In the [Amazon VPC console](https://console.aws.amazon.com/vpc), choose **Endpoint services**, and choose your endpoint service\. In the detail pane, see **Domain verification status**\.
+ Use the [DescribeVpcEndpointServiceConfigurations](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeVpcEndpointServiceConfigurations.html) operation\. The `State` value is in the `ServiceConfigurations.PrivateDnsNameConfiguration.State` field\.

If the verification status is not `verified`, follow the instructions in the [Domain ownership verification](https://docs.aws.amazon.com/vpc/latest/privatelink/manage-dns-names.html#verify-domain-ownership) topic to add a TXT record to your domain's DNS server and verify that the TXT record is published\. Then check your verification status again\.

You are not required to create an A record for the private DNS domain name\. When AWS KMS creates an interface endpoint to your VPC endpoint service, AWS PrivateLink automatically creates a hosted zone with the required A record for the private domain name in the AWS KMS VPC\. For external key stores with VPC endpoint service connectivity, this happens when you [connect your external key store](xks-connect-disconnect.md) to its external key store proxy\.

## Authorizing AWS KMS to connect to the VPC endpoint service<a name="xks-vpc-authorize-kms"></a>

You must add AWS KMS to the **Allow principals** list for your VPC endpoint service\. This allows AWS KMS to create interface endpoints to your VPC endpoint service\. If AWS KMS is not an allowed principal, attempts to create an external key store will fail with an `XksProxyVpcEndpointServiceNotFoundException` exception\.

Follow the instructions in the [Manage permissions](https://docs.aws.amazon.com/vpc/latest/privatelink/configure-endpoint-service.html#add-remove-permissions) topic in the *AWS PrivateLink Guide*\. Use the following required value\.


| Field | Value |
| --- | --- |
| ARN | cks\.kms\.<region>\.amazonaws\.comFor example, `cks.kms.us-east-1.amazonaws.com` |

**Next**: [Creating an external key store](create-xks-keystore.md)
