# Connecting to AWS KMS through a VPC endpoint<a name="kms-vpc-endpoint"></a>

You can connect directly to AWS KMS through a private endpoint in your VPC instead of connecting over the internet\. When you use a VPC endpoint, communication between your VPC and AWS KMS is conducted entirely within the AWS network\.

AWS KMS supports [Amazon Virtual Private Cloud](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Introduction.html) \(Amazon VPC\) [interface endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html) that are powered by [AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Introduction.html#what-is-privatelink)\. Each VPC endpoint is represented by one or more [Elastic Network Interfaces](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html) \(ENIs\) with private IP addresses in your VPC subnets\. 

The VPC interface endpoint connects your VPC directly to AWS KMS without an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection\. The instances in your VPC do not need public IP addresses to communicate with AWS KMS\. 

**Supported AWS Regions**  
AWS KMS supports VPC endpoints in all AWS Regions where both [Amazon VPC](https://docs.aws.amazon.com/general/latest/gr/vpc-service.html) and [AWS KMS](https://docs.aws.amazon.com/general/latest/gr/kms.html) are available\.

**Topics**
+ [Considerations for AWS KMS VPC endpoints](#vpce-considerations)
+ [Creating a VPC endpoint for AWS KMS](#vpce-create-endpoint)
+ [Connecting to an AWS KMS VPC endpoint](#vpce-connect)
+ [Controlling access to a VPC endpoint](#vpce-policy)
+ [Using a VPC endpoint in a policy statement](#vpce-policy-condition)
+ [Logging your VPC endpoint](#vpce-logging)

## Considerations for AWS KMS VPC endpoints<a name="vpce-considerations"></a>

Before you set up an interface VPC endpoint for AWS KMS, review the [Interface endpoint properties and limitations](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#vpce-interface-limitations) topic in the *Amazon VPC User Guide*\.

AWS KMS supports the following features to support a VPC endpoint\.
+ You can use your VPC interface endpoint to call all [AWS KMS API operations](https://docs.aws.amazon.com/kms/latest/APIReference/) from your VPC\.
+ AWS KMS does not support creating a VPC interface endpoint to an AWS KMS FIPS endpoint\.
+ You can use AWS CloudTrail logs to audit your use of AWS KMS customer master key \(CMKs\) through the VPC endpoint\. For details, see [Logging your VPC endpoint](#vpce-logging)\.

## Creating a VPC endpoint for AWS KMS<a name="vpce-create-endpoint"></a>

You can create a VPC endpoint for AWS KMS by using the Amazon VPC console or the Amazon VPC API\. For more information, see [Creating an interface endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#create-interface-endpoint) in the *Amazon VPC User Guide*\.

To create a VPC endpoint for AWS KMS, use the following service name: 

```
com.amazonaws.region.kms
```

For example, in the US West \(Oregon\) Region \(`us-west-2`\), the service name would be:

```
com.amazonaws.us-west-2.kms
```

To make it easier to use the VPC endpoint, you can enable a [private DNS hostname](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#vpce-private-dns) for your VPC endpoint\. If you select the **Enable Private DNS Name** option, the standard AWS KMS DNS hostname \(`https://kms.<region>.amazonaws.com`\) resolves to your VPC endpoint\.

This option makes it easier to use the VPC endpoint\. The AWS SDKs and AWS CLI use the standard AWS KMS DNS hostname by default, so you do not need to specify the VPC endpoint URL in applications and commands\.

For more information, see [Accessing a service through an interface endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#access-service-though-endpoint) in the *Amazon VPC User Guide*\.

## Connecting to an AWS KMS VPC endpoint<a name="vpce-connect"></a>

You can connect to AWS KMS through the VPC endpoint by using an AWS SDK, the AWS CLI or AWS Tools for PowerShell\. To specify the VPC endpoint, use its DNS name\. 

For example, this [list\-keys](https://docs.aws.amazon.com/cli/latest/reference/kms/list-keys.html) command uses the `endpoint-url` parameter to specify the VPC endpoint\. To use a command like this, replace the example VPC endpoint ID with one in your account\.

```
$ aws kms list-keys --endpoint-url https://vpce-1234abcdf5678c90a-09p7654s-us-east-1a.ec2.us-east-1.vpce.amazonaws.com
```

If you enabled private hostnames when you created your VPC endpoint, you do not need to specify the VPC endpoint URL in your CLI commands or application configuration\. The standard AWS KMS DNS hostname \(https://kms\.*<region>*\.amazonaws\.com\) resolves to your VPC endpoint\. The AWS CLI and SDKs use this hostname by default, so you can begin using the VPC endpoint without changing anything in your scripts and application\. 

To use private hostnames, the` enableDnsHostnames` and `enableDnsSupport` attributes of your VPC must be set to true\. To set these attributes, use the [ModifyVpcAttribute](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_ModifyVpcAttribute.html) operation\. 

## Controlling access to a VPC endpoint<a name="vpce-policy"></a>

To control access to your VPC endpoint for AWS KMS, attach a *VPC endpoint policy* to your VPC endpoint\. The endpoint policy determines whether principals can use the VPC endpoint to call AWS KMS operations on AWS KMS resources\.

You can create a VPC endpoint policy when you create your endpoint, and you can change the VPC endpoint policy at any time\. Use the VPC management console, or the [CreateVpcEndpoint](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateVpcEndpoint.html) or [ModifyVpcEndpoint](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_ModifyVpcEndpoint.html) operations\. You can also create and change a VPC endpoint policy by [using an AWS CloudFormation template](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-vpcendpoint.html)\. For help using the VPC management console, see [Creating an interface endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#create-interface-endpoint) and [Modifying an interface endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#modify-interface-endpoint) in the *Amazon VPC User Guide*\.

**Note**  
AWS KMS supports VPC endpoint policies beginning in July 2020\. VPC endpoints for AWS KMS that were created before that date have the [default VPC endpoint policy](#vpce-default-policy), but you can change it at any time\.

For help writing and formatting a JSON policy document, see the [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) in the *IAM User Guide*\.

**Topics**
+ [About VPC endpoint policies](#vpce-policy-about)
+ [Default VPC endpoint policy](#vpce-default-policy)
+ [Creating a VPC endpoint policy](#vpce-policy-create)
+ [Viewing a VPC endpoint policy](#vpce-policy-get)

### About VPC endpoint policies<a name="vpce-policy-about"></a>

For an AWS KMS request that uses a VPC endpoint to be successful, the principal requires permissions from two sources:
+ A [key policy](key-policies.md), [IAM policy](iam-policies.md), or [grant](grants.md) must give principal permission to call the operation on the resource \(CMK or alias\)\.
+ A VPC endpoint policy must give the principal permission to use the endpoint to make the request\.

For example, a key policy might give a principal permission to call [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) on a particular CMK\. However, the VPC endpoint policy might not allow that principal to call `Decrypt` on that CMK by using the endpoint\.

Or a VPC endpoint policy might allow a principal to use the endpoint to call [DisableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) on certain CMKs\. But if the principal doesn't have those permissions from a key policy, IAM policy, or grant, the request fails\.

### Default VPC endpoint policy<a name="vpce-default-policy"></a>

Every VPC endpoint has a VPC endpoint policy, but you are not required to specify the policy\. If you don't specify a policy, the default endpoint policy allows all operations by all principals on all resources over the endpoint\. 

However, for AWS KMS resources, the principal must also have permission to call the operation from a [key policy](key-policies.md), [IAM policy](iam-policies.md), or [grant](grants.md)\. Therefore, in practice, the default policy says that if a principal has permission to call an operation on a resource, they can also call it by using the endpoint\.

```
{
  "Statement": [
    {
      "Action": "*", 
      "Effect": "Allow", 
      "Principal": "*", 
      "Resource": "*"
    }
  ]
}
```

 To allow principals to use the VPC endpoint for only a subset of their permitted operations, [create or modify the VPC endpoint policy](#vpce-policy-create)\.

### Creating a VPC endpoint policy<a name="vpce-policy-create"></a>

A VPC endpoint policy determines whether a principal has permission to use the VPC endpoint to perform operations on a resource\. For AWS KMS resources, the principal must also have permission to perform the operations from a [key policy](key-policies.md), [IAM policy](iam-policies.md), or [grant](grants.md)\.

Each VPC endpoint policy statement requires the following elements:
+ The principal that can perform actions
+ The actions that can be performed
+ The resources on which actions can be performed

The policy statement doesn't specify the VPC endpoint\. Instead, it applies to any VPC endpoint to which the policy is attached\. For more information, see [Controlling access to services with VPC endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints-access.html) in the *Amazon VPC User Guide*\. 

The following is an example of a VPC endpoint policy for AWS KMS\. When attached to a VPC endpoint, this policy allows `ExampleUser` to use the VPC endpoint to call the specified operations on the specified CMK\. Before using a policy like this one, replace the example principal and [key ARN](concepts.md#key-id-key-ARN) with valid values from your account\.

```
{
   "Statement":[
      {
         "Sid": "Allow decrypt and view permission",
         "Principal": {"AWS": "arn:aws:iam::111122223333:user/ExampleUser"},
         "Effect":"Allow",
         "Action": [ 
             "kms:Decrypt",
             "kms:DescribeKey",  
             "kms:ListAliases", 
             "kms:ListKeys"
          ],
         "Resource": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
      }
   ]
}
```

AWS CloudTrail logs all operations that use the VPC endpoint\. However, your CloudTrail logs don’t include operations requested by principals in other accounts or operations for CMKs in other accounts\.

As such, you might want to create a VPC endpoint policy that prevents principals in external accounts from using the VPC endpoint to call any KMS operations on any keys in the local account\.

The following example uses the [aws:PrincipalAccount](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-principalaccount) global condition key to deny access to all principals for all operations on all CMKs unless the principal is in the local account\. Before using a policy like this one, replace the example account ID with a valid one\.

```
{
  "Statement": [
    {
      "Sid": "Access for a specific account",
      "Principal": {"AWS": "*"},
      "Action": "kms:*",
      "Effect": "Deny",
      "Resource": "arn:aws:kms:*:111122223333:key/*",
      "Condition": {
        "StringNotEquals": {
          "aws:PrincipalAccount": "111122223333"
        }
      }
    }
  ]
}
```

### Viewing a VPC endpoint policy<a name="vpce-policy-get"></a>

To view the VPC endpoint policy for an endpoint, use the [VPC management console](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#describe-interface-endpoint) or the [DescribeVpcEndpoints](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeVpcEndpoints.html) operation\.

The following AWS CLI command gets the policy for the endpoint with the specified VPC endpoint ID\. 

Before using this command, replace the example endpoint ID with a valid one from your account\.

```
$ aws ec2 describe-vpc-endpoints \
--query 'VpcEndpoints[?VpcEndpointId==`vpce-1234abcdf5678c90a`].[PolicyDocument]'
--output text
```

## Using a VPC endpoint in a policy statement<a name="vpce-policy-condition"></a>

You can control access to AWS KMS resources and operations when the request comes from VPC or uses a VPC endpoint\. To do so, use one of the following [global condition keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#AvailableKeys) in a [key policy](key-policies.md) or [IAM policy](iam-policies.md)\.
+ Use the `aws:sourceVpce` condition key to grant or restrict access based on the VPC endpoint\.
+ Use the `aws:sourceVpc` condition key to grant or restrict access based on the VPC that hosts the private endpoint\.

**Note**  
Use caution when creating key policies and IAM policies based on your VPC endpoint\. If a policy statement requires that requests come from a particular VPC or VPC endpoint, requests from integrated AWS services that use an AWS KMS resource on your behalf might fail\. For help, see [Using VPC endpoint conditions in policies with AWS KMS permissions](policy-conditions.md#conditions-aws-vpce)\.  
Also, the `aws:sourceIP` condition key is not effective when the request comes from an [Amazon VPC endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html)\. To restrict requests to a VPC endpoint, use the `aws:sourceVpce` or `aws:sourceVpc` condition keys\. For more information, see [VPC Endpoints \- Controlling the Use of Endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html#vpc-endpoints-iam-access) in the *Amazon VPC User Guide*\. 

You can use these global condition keys to control access to customer master keys \(CMKs\), aliases, and to operations like [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) that don't depend on any particular resource\.

For example, the following sample key policy allows a user to perform some cryptographic operations with a CMK only when the request uses the specified VPC endpoint\. When a user makes a request to AWS KMS, the VPC endpoint ID in the request is compared to the `aws:sourceVpce` condition key value in the policy\. If they do not match, the request is denied\. 

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
                    "aws:sourceVpce": "vpce-1234abcdf5678c90a"
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

## Logging your VPC endpoint<a name="vpce-logging"></a>

AWS CloudTrail logs all operations that use the VPC endpoint\. When a request to AWS KMS uses a VPC endpoint, the VPC endpoint ID appears in the [AWS CloudTrail log](logging-using-cloudtrail.md) entry that records the request\. You can use the endpoint ID to audit the use of your AWS KMS VPC endpoint\.

However, your CloudTrail logs don't include operations requested by principals in other accounts or requests for AWS KMS operations on CMKs and aliases in other accounts\. Also, to protect your VPC, requests that are denied by a [VPC endpoint policy](#vpce-policy), but otherwise would have been allowed, are not recorded in [AWS CloudTrail](logging-using-cloudtrail.md)\.

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
	"vpcEndpointId": "vpce-1234abcdf5678c90a"
}
```