# Using IAM policies with AWS KMS<a name="iam-policies"></a>

You can use IAM policies, along with [key policies](key-policies.md), [grants](grants.md), and [VPC endpoint policies](kms-vpc-endpoint.md#vpce-policy), to control access to your customer master keys \(CMKs\) in AWS KMS\. 

**Note**  
To use an IAM policy to control access to a CMK, the key policy for the CMK must give the account permission to use IAM policies\. Specifically, the key policy must include the [policy statement that enables IAM policies](key-policies.md#key-policy-default-allow-root-enable-iam)\.  
This section explains how to use IAM policies to control access to AWS KMS operations\. For more general information about IAM, see the [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/)\.

All CMKs must have a key policy\. IAM policies are optional\. To use an IAM policy to control access to a CMK, the key policy for the CMK must give the account permission to use IAM policies\. Specifically, the key policy must include the [policy statement that enables IAM policies](key-policies.md#key-policy-default-allow-root-enable-iam)\.

IAM policies can control access to any AWS KMS operation\. Unlike key policies, IAM policies can control access to multiple CMKs and provide permissions for the operations of several related AWS services\. But IAM policies are particularly useful for controlling access to operations, such as [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html), that can't be controlled by a key policy because they don't involve any particular CMK\.

If you access AWS KMS through an Amazon Virtual Private Cloud \(Amazon VPC\) endpoint, you can also use a VPC endpoint policy to limit access to your AWS KMS resources when using the endpoint\. For example, when using the VPC endpoint, you might only allow the principals in your AWS account to access your CMKs\. For details, see [Controlling access to a VPC endpoint](kms-vpc-endpoint.md#vpce-policy) \.

For help writing and formatting a JSON policy document, see the [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) in the *IAM User Guide*\.

**Topics**
+ [Overview of IAM policies](iam-policies-overview.md)
+ [Best practices for IAM policies](iam-policies-best-practices.md)
+ [Specifying CMKs in IAM policy statements](cmks-in-iam-policies.md)
+ [Permissions required to use the AWS KMS console](console-permissions.md)
+ [AWS managed policy for power users](aws-managed-policies.md)
+ [Customer managed policy examples](customer-managed-policies.md)