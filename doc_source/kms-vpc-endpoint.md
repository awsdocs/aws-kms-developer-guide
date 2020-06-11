# Connecting to AWS KMS through a VPC endpoint<a name="kms-vpc-endpoint"></a>

You can connect directly to AWS KMS through a private endpoint in your VPC instead of connecting over the internet\. When you use a VPC endpoint, communication between your VPC and AWS KMS is conducted entirely within the AWS network\.

AWS KMS supports [Amazon Virtual Private Cloud](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Introduction.html) \(Amazon VPC\) [interface endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html) that are powered by [AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Introduction.html#what-is-privatelink)\. Each VPC endpoint is represented by one or more [Elastic Network Interfaces](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html) \(ENIs\) with private IP addresses in your VPC subnets\. 

The VPC interface endpoint connects your VPC directly to AWS KMS without an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection\. The instances in your VPC do not need public IP addresses to communicate with AWS KMS\. 

**Supported AWS Regions**  
AWS KMS supports VPC endpoints in all AWS Regions where both [Amazon VPC](https://docs.aws.amazon.com/general/latest/gr/vpc-service.html) and [AWS KMS](https://docs.aws.amazon.com/general/latest/gr/kms.html) are available\.

**Topics**
+ [Considerations for AWS KMS VPC endpoints](#vpc-endpoint-considerations)
+ [Creating a VPC endpoint for AWS KMS](#create-vpc-endpoint)
+ [Connecting to an AWS KMS VPC endpoint](#connecting-vpc-endpoint)
+ [Using a VPC endpoint in a policy statement](#vpce-policy)
+ [Auditing your VPC endpoint](#vpce-logging)

## Considerations for AWS KMS VPC endpoints<a name="vpc-endpoint-considerations"></a>

Before you set up an interface VPC endpoint for AWS KMS, review the [Interface endpoint properties and limitations](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#vpce-interface-limitations) topic in the *Amazon VPC User Guide*\.

AWS KMS supports the following features to support a VPC endpoint\.
+ You can use your VPC interface endpoint to call all [AWS KMS API operations](https://docs.aws.amazon.com/kms/latest/APIReference/) from your VPC\.
+ AWS KMS does not support creating a VPC interface endpoint to an AWS KMS FIPS endpoint\.
+ You can use AWS CloudTrail logs to audit your use of AWS KMS customer master keys \(CMKs\) through the VPC endpoint\. For details, see [Auditing your VPC endpoint](#vpce-logging)\.

## Creating a VPC endpoint for AWS KMS<a name="create-vpc-endpoint"></a>

You can create a VPC endpoint for AWS KMS by using the Amazon VPC console or the Amazon VPC API\. For more information, see [Creating an interface endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#create-interface-endpoint) in the *Amazon VPC User Guide*\.

To create a VPC endpoint for AWS KMS, use the following service name: 

```
com.amazonaws.region.kms
```

For example, in the US West \(Oregon\) Region \(`us-west-2`\), the service name would be:

```
com.amazonaws.us-west-2.kms
```

To make it easier to use the VPC endpoint, you can enable a [private DNS hostname](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#vpce-private-dns) for your VPC endpoint\. If you do, the standard AWS KMS DNS hostname \(`https://kms.<region>.amazonaws.com`\) resolves to your VPC endpoint\. Also, the AWS CLI and AWS SDKs use the standard AWS KMS DNS hostname by default, so you do not need to specify the VPC endpoint URL in applications and commands\.

You can enable private DNS hostnames when you create or modify your VPC endpoint\. For more information about using private DNS hostnames, see [Private DNS names for endpoint services](https://docs.aws.amazon.com/vpc/latest/userguide/verify-domains.html) in the *Amazon VPC User Guide*\.

## Connecting to an AWS KMS VPC endpoint<a name="connecting-vpc-endpoint"></a>

You can connect to AWS KMS through the VPC endpoint by using an AWS SDK, the AWS CLI or AWS Tools for PowerShell\. To specify the VPC endpoint, use its DNS name\. 

For example, this [list\-keys](https://docs.aws.amazon.com/cli/latest/reference/kms/list-keys.html) command uses the `endpoint-url` parameter to specify the VPC endpoint\. To use a command like this, replace the example VPC endpoint ID with one in your account\.

```
aws kms list-keys --endpoint-url https://vpce-0295a3caf8414c94a-dfm9tr04.kms.us-east-1.vpce.amazonaws.com
```

If private DNS hostnames is enabled on the endpoint, you do not need to specify the VPC endpoint URL in your CLI commands or application configuration\. For more information about using private DNS hostnames, see [Private DNS names for endpoint services](https://docs.aws.amazon.com/vpc/latest/userguide/verify-domains.html) in the *Amazon VPC User Guide*\.

## Using a VPC endpoint in a policy statement<a name="vpce-policy"></a>

You can use IAM policies and AWS KMS key policies to control access to your AWS KMS customer master keys \(CMKs\)\. You can also use [global condition keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#AvailableKeys) to restrict these policies based on VPC endpoint or VPC in the request\.
+ Use the `aws:sourceVpce` condition key to grant or restrict access to an AWS KMS CMK based on the VPC endpoint\.
+ Use the `aws:sourceVpc` condition key to grant or restrict access to an AWS KMS CMK based on the VPC that hosts the private endpoint\.

**Note**  
Use caution when creating IAM and key policies based on your VPC endpoint\. If a policy statement requires that requests come from a particular VPC or VPC endpoint, requests from integrated AWS services that use the CMK on your behalf might fail\. For help, see [Using VPC endpoint conditions in policies with AWS KMS permissions](policy-conditions.md#conditions-aws-vpce)\.  
Also, the `aws:sourceIP` condition key is not effective when the request comes from an [Amazon VPC endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html)\. To restrict requests to a VPC endpoint, use the `aws:sourceVpce` or `aws:sourceVpc` condition keys\. For more information, see [VPC Endpoints \- Controlling the Use of Endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html#vpc-endpoints-iam-access) in the *Amazon VPC User Guide*\. 

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

## Auditing your VPC endpoint<a name="vpce-logging"></a>

When a request to AWS KMS uses a VPC endpoint, the VPC endpoint ID appears in the [AWS CloudTrail log](logging-using-cloudtrail.md) entry that records the request\. You can use the endpoint ID to audit the use of your AWS KMS VPC endpoint\. 

For example, this sample log entry records a [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) request that used the VPC endpoint\. The `vpcEndpointId` field appears at the end of the log entry\.

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