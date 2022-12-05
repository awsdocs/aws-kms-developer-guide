# AWS global condition keys<a name="conditions-aws"></a>

AWS defines [global condition keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#AvailableKeys), a set of policy conditions keys for all AWS services that use IAM for access control\. AWS KMS supports all global condition keys\. You can use them in AWS KMS key policies and IAM policies\.

For example, you can use the [aws:PrincipalArn](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-principalarn) global condition key to allow access to an AWS KMS key \(KMS key\) only when the principal in the request is represented by the Amazon Resource Name \(ARN\) in the condition key value\. To support [attribute\-based access control](abac.md) \(ABAC\) in AWS KMS, you can use the [aws:ResourceTag/*tag\-key*](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-resourcetag) global condition key in an IAM policy to allow access to KMS keys with a particular tag\.

To help prevent an AWS service from being used as a confused deputy in a policy where the principal is an [AWS service principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services), you can use the [https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourcearn](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourcearn) or [https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourceaccount](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourceaccount) global condition keys\. For details, see [Using `aws:SourceArn` or `aws:SourceAccount` condition keys](key-policy-services.md#least-privilege-source-arn)\.

For information about AWS global condition keys, including the types of requests in which they are available, see [AWS Global Condition Context Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) in the *IAM User Guide*\. For examples of using global condition keys in IAM policies, see [Controlling Access to Requests](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_tags.html#access_tags_control-requests) and [Controlling Tag Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_tags.html#access_tags_control-tag-keys) in the *IAM User Guide*\.

The following topics provide special guidance for using condition keys based on IP addresses and VPC endpoints\.

**Topics**
+ [Using the IP address condition in policies with AWS KMS permissions](#conditions-aws-ip-address)
+ [Using VPC endpoint conditions in policies with AWS KMS permissions](#conditions-aws-vpce)

## Using the IP address condition in policies with AWS KMS permissions<a name="conditions-aws-ip-address"></a>

You can use AWS KMS to protect your data in an [integrated AWS service](service-integration.md)\. But use caution when specifying the [IP address condition operators](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_IPAddress) or the `aws:SourceIp` condition key in the same policy statement that allows or denies access to AWS KMS\. For example, the policy in [AWS: Denies Access to AWS Based on the Source IP](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_aws_deny-ip.html) restricts AWS actions to requests from the specified IP range\.

Consider this scenario:

1. You attach a policy like the one shown at [AWS: Denies Access to AWS Based on the Source IP](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_aws_deny-ip.html) to an IAM user\. You set the value of the `aws:SourceIp` condition key to the range of IP addresses for the user's company\. This IAM user has other policies attached that allow it to use Amazon EBS, Amazon EC2, and AWS KMS\.

1. The user attempts to attach an encrypted EBS volume to an EC2 instance\. This action fails with an authorization error even though the user has permission to use all the relevant services\.

Step 2 fails because the request to AWS KMS to decrypt the volume's encrypted data key comes from an IP address that is associated with the Amazon EC2 infrastructure\. To succeed, the request must come from the IP address of the originating user\. Because the policy in step 1 explicitly denies all requests from IP addresses other than those specified, Amazon EC2 is denied permission to decrypt the EBS volume's encrypted data key\.

Also, the `aws:sourceIP` condition key is not effective when the request comes from an [Amazon VPC endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html)\. To restrict requests to a VPC endpoint, including an [AWS KMS VPC endpoint](kms-vpc-endpoint.md), use the `aws:sourceVpce` or `aws:sourceVpc` condition keys\. For more information, see [VPC Endpoints \- Controlling the Use of Endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html#vpc-endpoints-iam-access) in the *Amazon VPC User Guide*\. 

## Using VPC endpoint conditions in policies with AWS KMS permissions<a name="conditions-aws-vpce"></a>

[AWS KMS supports Amazon Virtual Private Cloud \(Amazon VPC\) endpoints](kms-vpc-endpoint.md) that are powered by [AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Introduction.html#what-is-privatelink)\. You can use the following [global condition keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#AvailableKeys) in key policies and IAM policies to control access to AWS KMS resources when the request comes from a VPC or uses a VPC endpoint\. For details, see [Using a VPC endpoint in a policy statement](kms-vpc-endpoint.md#vpce-policy-condition)\.
+ `aws:SourceVpc` limits access to requests from the specified VPC\. 
+ `aws:SourceVpce` limits access to requests from the specified VPC endpoint\. 

If you use these condition keys to control access to KMS keys, you might inadvertently deny access to AWS services that use AWS KMS on your behalf\. 

Take care to avoid a situation like the [IP address condition keys](#conditions-aws-ip-address) example\. If you restrict requests for a KMS key to a VPC or VPC endpoint, calls to AWS KMS from an integrated service, such as Amazon S3 or Amazon EBS, might fail\. This can happen even if the source request ultimately originates in the VPC or from the VPC endpoint\. 