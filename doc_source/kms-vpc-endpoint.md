# Connecting to AWS KMS Through a VPC Endpoint<a name="kms-vpc-endpoint"></a>

You can connect directly to AWS KMS through a private endpoint in your VPC instead of connecting over the internet\. When you use a VPC endpoint, communication between your VPC and AWS KMS is conducted entirely within the AWS network\.

AWS KMS supports [Amazon Virtual Private Cloud](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html) \(Amazon VPC\) [interface endpoints](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpce-interface.html) that are powered by [AWS PrivateLink](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html#what-is-privatelink)\. Each VPC endpoint is represented by one or more [Elastic Network Interfaces](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html) \(ENIs\) with private IP addresses in your VPC subnets\. 

The VPC interface endpoint connects your VPC directly to AWS KMS without an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection\. The instances in your VPC do not need public IP addresses to communicate with AWS KMS\. 

You can specify the VPC endpoint in [AWS KMS API operations](http://docs.aws.amazon.com/kms/latest/APIReference/) and [AWS CLI commands](http://docs.aws.amazon.com/cli/latest/reference/kms/index.html)\. For example, the following command uses the endpoint\-url parameter to specify a VPC endpoint in an AWS CLI command to AWS KMS\. 

```
$  aws kms list-keys --endpoint-url https://vpce-0295a3caf8414c94a-dfm9tr04.kms.us-east-1.vpce.amazonaws.com
```

If you use the default domain name servers \(**AmazonProvidedDNS**\) and enable [private DNS hostnames](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpce-interface.html#vpce-private-dns) for your VPC endpoint, you do not need to specify the endpoint URL\. AWS populates your VPC name server with private zone data, so the public KMS endpoint \(`https://kms.<region>.amazonaws.com`\) resolves to your private VPC endpoint\. To enable this feature when using your own name servers, forward requests for the KMS domain to the VPC name server\. 

You can also use AWS CloudTrail logs to audit your use of KMS keys through the VPC endpoint\. And you can use the conditions in IAM and key policies to deny access to any request that does not come from a specified VPC or VPC endpoint\.

**Note**  
Use caution when creating IAM and key policies based on your VPC endpoint\. If a policy statement requires that requests come from a particular VPC or VPC endpoint, requests from integrated AWS services that use the CMK on your behalf might fail\. For help, see [Using VPC Endpoint Conditions in Policies with AWS KMS Permissions](policy-conditions.md#conditions-aws-vpce)\.

**Regions**  
AWS KMS supports VPC endpoints in all AWS regions where both [Amazon VPC](http://docs.aws.amazon.com/general/latest/gr/rande.html#vpc_region) and [AWS KMS](http://docs.aws.amazon.com/general/latest/gr/rande.html#kms_region) are available, except for AWS GovCloud \(US\)\.

**Topics**
+ [Create an AWS KMS VPC Endpoint](#create-vpc-endpoint)
+ [Connecting to an AWS KMS VPC Endpoint](#connecting-vpc-endpoint)
+ [Using a VPC Endpoint in a Policy Statement](#vpce-policy)
+ [Audit the CMK Use for your VPC](#vpce-logging)

## Create an AWS KMS VPC Endpoint<a name="create-vpc-endpoint"></a>

You [create an interface endpoint](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpce-interface.html#create-interface-endpoint) in your VPC by using the KMS VPC endpoint service in each region\. You can create a VPC endpoint in the AWS Management Console, or by using the [AWS CLI](http://docs.aws.amazon.com/cli/latest/reference/ec2/create-vpc-endpoint.html) or [Amazon EC2 API](http://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateVpcEndpoint.html)\. 

**Topics**
+ [Creating an AWS KMS VPC Endpoint \(Console\)](#create-vpc-endpoint-console)
+ [Creating an AWS KMS VPC Endpoint \(AWS CLI\)](#create-vpc-endpoint-cli)

### Creating an AWS KMS VPC Endpoint \(Console\)<a name="create-vpc-endpoint-console"></a>

1. Sign in to the AWS Management Console and open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. On the navigation bar, use the region selector to choose your region\.

1. In the navigation pane, choose **Endpoints**\. In the main pane, **Create Endpoint**\.

1. For **Service category**, choose **AWS services**\.

1. In the **Service Name** list, choose the entry for AWS KMS interface endpoint in the region\. For example, in the US East \(N\.Virginia\) Region, the entry name is `com.amazonaws.us-east-1.kms`\. 

1. For **VPC**, select a VPC\. The endpoint is created in the VPC that you select\.

1. For **Subnets**, choose a subnet from each Availability Zone that you want to include\.

   The VPC endpoint can span multiple Availability Zones\. An elastic network interface \(ENI\) for the VPC endpoint is created in each subnet that you choose\. Each ENI has a DNS hostname and a private IP address\.

1. In this step, you can enable a private DNS hostname for your VPC endpoint\. If you select the **Enable Private DNS Name** option, the standard AWS KMS DNS hostname \(`https://kms.<region>.amazonaws.com`\) resolves to your VPC endpoint\.

   This option makes it easier to use the VPC endpoint\. The AWS KMS CLI and SDKs use the standard AWS KMS DNS hostname by default, so you do not need to specify the VPC endpoint URL in applications and commands\.

   This feature works only when the `enableDnsHostnames` and `enableDnsSupport` attributes of your VPC are set to `true`\. To set these attributes, [update DNS support for your VPC](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-dns.html#vpc-dns-updating)\.

   To enable a private DNS hostname, for **Enable Private DNS Name**, select **Enable for this endpoint**\. 

1. For **Security group**, select or create a security group\.

   You can use [security groups](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_SecurityGroups.html) to control access to your endpoint, much like you would use a firewall\.

1. Choose **Create endpoint**\.

The results show the VPC endpoint, including the VPC endpoint ID and the DNS names that you use to [connect to your VPC endpoint](#connecting-vpc-endpoint)\. 

You can also use the Amazon VPC tools to view and manage your endpoint, including creating a notification for an endpoint, changing properties of the endpoint, and deleting the endpoint\. For instructions, see [Interface VPC Endpoints](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpce-interface.html)\.

![\[Creating an endpoint in the VPC console\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/create-vpc-endpoint.png)

### Creating an AWS KMS VPC Endpoint \(AWS CLI\)<a name="create-vpc-endpoint-cli"></a>

You can use the [create\-vpc\-endpoint](http://docs.aws.amazon.com/cli/latest/reference/ec2/create-vpc-endpoint.html) command in the AWS CLI to create a VPC endpoint that connects to AWS KMS\. 

Be sure to use `interface` as the VPC endpoint type and a service name value that includes `kms` and the region where your VPC is located\. 

The command does not include the `PrivateDnsNames` parameter because its default value is true\. To disable this option, you can include the parameter with a value of `false`\. Private DNS names are available only when the `enableDnsHostnames` and `enableDnsSupport` attributes of your VPC are set to `true`\. To set these attributes, use the [ModifyVpcAttribute](http://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_ModifyVpcAttribute.html) API\.

The following diagram shows the syntax of the command\. 

```
aws ec2 create-vpc-endpoint --vpc-id <vpc id> \
                            --vpc-endpoint-type Interface \
                            --service-name com.amazonaws.<region>.kms \
                            --subnet-ids <subnet id> \
                            --security-group-id <security group id>
```

For example, this command creates a VPC endpoint in the VPC with VPC ID `vpc-1a2b3c4d`, which is in the `us-east-1` region\. It specifies just one subnet ID to represent the Availability Zones, but you can specify many\. The security group ID is also required\.

The output includes the VPC endpoint ID and DNS names that you use to connect to your new VPC endpoint\.

```
$  aws ec2 create-vpc-endpoint --vpc-id vpc-1a2b3c4d \
                               --vpc-endpoint-type Interface \
                               --service-name com.amazonaws.us-west-1.kms \
                               --subnet-ids subnet-a6b10bd1 \
                               --security-group-id sg-1a2b3c4d

{
  "VpcEndpoint": {
      "PolicyDocument": "{\n  \"Statement\": [\n    {\n      \"Action\": \"*\", \n      \"Effect\": \"Allow\", \n      \"Principal\": \"*\", \n      \"Resource\": \"*\"\n    }\n  ]\n}",
      "VpcId": "vpc-1a2b3c4d",
      "NetworkInterfaceIds": [
          "eni-bf8aa46b"
      ],
      "SubnetIds": [
          "subnet-a6b10bd1"
      ],
      "PrivateDnsEnabled": true,
      "State": "pending",
      "ServiceName": "com.amazonaws.us-east-1.kms",
      "RouteTableIds": [],
      "Groups": [
          {
              "GroupName": "default",
              "GroupId": "sg-1a2b3c4d"
          }
      ],
      "VpcEndpointId": "vpce-0295a3caf8414c94a",
      "VpcEndpointType": "Interface",
      "CreationTimestamp": "2017-09-05T20:14:41.240Z",
      "DnsEntries": [
          {
              "HostedZoneId": "Z7HUB22UULQXV",
              "DnsName": "vpce-0295a3caf8414c94a-dfm9tr04.kms.us-east-1.vpce.amazonaws.com"
          },
          {
              "HostedZoneId": "Z7HUB22UULQXV",
              "DnsName": "vpce-0295a3caf8414c94a-dfm9tr04-us-east-1a.kms.us-east-1.vpce.amazonaws.com"
          },
          {
              "HostedZoneId": "Z1K56Z6FNPJRR",
              "DnsName": "kms.us-east-1.amazonaws.com"
          }
      ]
  }
}
```

## Connecting to an AWS KMS VPC Endpoint<a name="connecting-vpc-endpoint"></a>

You can connect to AWS KMS through the VPC endpoint by using the AWS CLI or an AWS SDK\. To specify the VPC endpoint, use its DNS name\. 

For example, this [list\-keys](http://docs.aws.amazon.com/cli/latest/reference/kms/list-keys.html) command uses the `endpoint-url` parameter to specify the VPC endpoint\. To use a command like this, replace the example VPC endpoint ID with one in your account\.

```
aws kms list-keys --endpoint-url https://vpce-0295a3caf8414c94a-dfm9tr04.kms.us-east-1.vpce.amazonaws.com
```

If you enabled private hostnames when you created your VPC endpoint, you do not need to specify the VPC endpoint URL in your CLI commands or application configuration\. The standard AWS KMS DNS hostname \(https://kms\.*<region>*\.amazonaws\.com\) resolves to your VPC endpoint\. The AWS CLI and SDKs use this hostname by default, so you can begin using the VPC endpoint without changing anything in your scripts and application\. 

To use private hostnames, the` enableDnsHostnames` and `enableDnsSupport` attributes of your VPC must be set to true\. To set these attributes, use the [ModifyVpcAttribute](http://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_ModifyVpcAttribute.html) API\. 

## Using a VPC Endpoint in a Policy Statement<a name="vpce-policy"></a>

You can use IAM policies and AWS KMS key policies to control access to your AWS KMS customer master keys \(CMKs\)\. You can also use [global condition keys](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#AvailableKeys) to restrict these policies based on VPC endpoint or VPC in the request\.
+ Use the `aws:sourceVpce` condition key to grant or restrict access to an AWS KMS CMK based on the VPC endpoint\.
+ Use the `aws:sourceVpc` condition key to grant or restrict access to an AWS KMS CMK based on the VPC that hosts the private endpoint\.

**Note**  
Use caution when creating IAM and key policies based on your VPC endpoint\. If a policy statement requires that requests come from a particular VPC or VPC endpoint, requests from integrated AWS services that use the CMK on your behalf might fail\. For help, see [Using VPC Endpoint Conditions in Policies with AWS KMS Permissions](policy-conditions.md#conditions-aws-vpce)\.  
Also, the `aws:sourceIP` condition key is not effective when the request comes from an [Amazon VPC endpoint](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-endpoints.html)\. To restrict requests to a VPC endpoint, use the `aws:sourceVpce` or `aws:sourceVpc` condition keys\. For more information, see [VPC Endpoints \- Controlling the Use of Endpoints](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-endpoints.html#vpc-endpoints-iam-access) in the *Amazon VPC User Guide*\. 

For example, the following sample key policy allows a user to perform encryption operations with a CMK only when the request comes through the specified VPC endpoint\. 

When a user makes a request to AWS KMS, the VPC endpoint ID in the request is compared to the `aws:sourceVpce` condition key value in the policy\. If they do not match, then the request is denied\. 

To use a policy like this one, replace the placeholder AWS account ID and VPC endpoint IDs with valid values for your account\.

```
{
    "Id": "example-key-1",
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Enable IAM user permissions",
            "Effect": "Allow",
            "Principal": {"AWS":["111122223333"]},
            "Action": ["kms:*"],
            "Resource": "*"
        },
        {
            "Sid": "Restrict usage to my VPC endpoint",
            "Effect": "Deny",
            "Principal": "*",
            "Action": [
                "kms:Encrypt",
                "kms:Decrypt",
                "kms:ReEncrypt*",
                "kms:GenerateDataKey*"
            ],
            "Resource": "*",
            "Condition": {
                "StringNotEquals": {
                    "aws:sourceVpce": "vpce-0295a3caf8414c94a"
                }
            }
        }

    ]
}
```

You can also use the `aws:sourceVpc` condition key to restrict access to your CMKs based on the VPC in which VPC endpoint resides\. 

The following sample key policy allows commands that manage the CMK only when they come from `vpc-12345678`\. In addition, it allows commands that use the CMK for cryptographic operations only when they come from `vpc-2b2b2b2b`\. You might use a policy like this one if an application is running in one VPC, but you use a second, isolated VPC for management functions\. 

To use a policy like this one, replace the placeholder AWS account ID and VPC endpoint IDs with valid values for your account\.

```
{
    "Id": "example-key-2",
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Allow administrative actions from vpc-12345678",
            "Effect": "Allow",
            "Principal": {"AWS": "111122223333"},
            "Action": [
                "kms:Create*","kms:Enable*","kms:Put*","kms:Update*",
                "kms:Revoke*","kms:Disable*","kms:Delete*",
                "kms:TagResource", "kms:UntagResource"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:sourceVpc": "vpc-12345678"
                }
            }
        },
        {
            "Sid": "Allow key usage from vpc-2b2b2b2b",
            "Effect": "Allow",
            "Principal": {"AWS": "111122223333"},
            "Action": [
                "kms:Encrypt","kms:Decrypt","kms:GenerateDataKey*"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:sourceVpc": "vpc-2b2b2b2b"
                }
            }
        },
        {
            "Sid": "Allow read actions from everywhere",
            "Effect": "Allow",
            "Principal": {"AWS": "111122223333"},
            "Action": [
                "kms:Describe*","kms:List*","kms:Get*"
            ],
            "Resource": "*",
        }
    ]
}
```

## Audit the CMK Use for your VPC<a name="vpce-logging"></a>

When a request to AWS KMS uses a VPC endpoint, the VPC endpoint ID appears in the [AWS CloudTrail log](logging-using-cloudtrail.md) entry that records the request\. You can use the endpoint ID to audit the use of your AWS KMS VPC endpoint\. 

For example, this sample log entry records a `GenerateDataKey` request that used the VPC endpoint\. The `vpcEndpointId` field appears at the end of the log entry\.

```
{
	"eventVersion":"1.05",
	"userIdentity": {
        "type": "IAMUser",
        "principalId": "EX_PRINCIPAL_ID",
        "arn": "arn:aws:iam::111122223333:user/Alice",
        "accessKeyId": "EXAMPLE_KEY_ID",
        "accountId": "111122223333",
        "userName": "Alice"
    },
	"eventTime":"2018-01-16T05:46:57Z",
	"eventSource":"kms.amazonaws.com",
	"eventName":"GenerateDataKey",
	"awsRegion":"eu-west-1",
	"sourceIPAddress":"172.01.01.001",
	"userAgent":"aws-cli/1.14.23 Python/2.7.12 Linux/4.9.75-25.55.amzn1.x86_64 botocore/1.8.27",
	"requestParameters":{
		"keyId":"1234abcd-12ab-34cd-56ef-1234567890ab",
		"numberOfBytes":128
	},
	"responseElements":null,
	"requestID":"a9fff0bf-fa80-11e7-a13c-afcabff2f04c",
	"eventID":"77274901-88bc-4e3f-9bb6-acf1c16f6a7c",
	"readOnly":true,
	"resources":[{
		"ARN":"arn:aws:kms:eu-west-1:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
		"accountId":"111122223333",
		"type":"AWS::KMS::Key"
	}],
	"eventType":"AwsApiCall",
	"recipientAccountId":"111122223333",
	"vpcEndpointId": "vpce-0295a3caf8414c94a"
}
```