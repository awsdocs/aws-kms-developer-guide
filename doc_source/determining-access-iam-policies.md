# Examining IAM policies<a name="determining-access-iam-policies"></a>

In addition to the key policy and grants, you can also use IAM policies in combination with a CMK's key policy to allow access to a CMK\. For more information about how IAM policies and key policies work together, see [Troubleshooting key access](policy-evaluation.md)\.

To determine which principals currently have access to a CMK through IAM policies, you can use the browser\-based [IAM Policy Simulator](https://policysim.aws.amazon.com/) tool, or you can make requests to the IAM API\.

**Contents**
+ [Examining IAM policies with the IAM policy simulator](#determining-access-iam-policy-simulator)
+ [Examining IAM policies with the IAM API](#determining-access-iam-api)

## Examining IAM policies with the IAM policy simulator<a name="determining-access-iam-policy-simulator"></a>

The IAM Policy Simulator can help you learn which principals have access to a KMS CMK through an IAM policy\.

**To use the IAM policy simulator to determine access to a KMS CMK**

1. Sign in to the AWS Management Console and then open the IAM Policy Simulator at [https://policysim.aws.amazon.com/](https://policysim.aws.amazon.com/)\.

1. In the **Users, Groups, and Roles** pane, choose the user, group, or role whose policies you want to simulate\.

1. \(Optional\) Clear the check box next to any policies that you want to omit from the simulation\. To simulate all policies, leave all policies selected\.

1. In the **Policy Simulator** pane, do the following:

   1. For **Select service**, choose **Key Management Service**\.

   1. To simulate specific AWS KMS actions, for **Select actions**, choose the actions to simulate\. To simulate all AWS KMS actions, choose **Select All**\.

1. \(Optional\) The Policy Simulator simulates access to all KMS CMKs by default\. To simulate access to a specific KMS CMK, choose **Simulation Settings**and then type the Amazon Resource Name \(ARN\) of the KMS CMK to simulate\.

1. Choose **Run Simulation**\.

You can view the results of the simulation in the **Results** section\. Repeat steps 2 through 6 for every IAM user, group, and role in the AWS account\.

## Examining IAM policies with the IAM API<a name="determining-access-iam-api"></a>

You can use the IAM API to examine IAM policies programmatically\. The following steps provide a general overview of how to do this:

1. For each AWS account listed as a principal in the CMK's key policy \(that is, each *root account* listed in this format: `"Principal": {"AWS": "arn:aws:iam::111122223333:root"}`\), use the [ListUsers](https://docs.aws.amazon.com/IAM/latest/APIReference/API_ListUsers.html) and [ListRoles](https://docs.aws.amazon.com/IAM/latest/APIReference/API_ListRoles.html) operations in the IAM API to retrieve a list of every IAM user and role in the account\.

1. For each IAM user and role in the list, use the [SimulatePrincipalPolicy](https://docs.aws.amazon.com/IAM/latest/APIReference/API_SimulatePrincipalPolicy.html) operation in the IAM API, passing in the following parameters:
   + For `PolicySourceArn`, specify the Amazon Resource Name \(ARN\) of a user or role from your list\. You can specify only one `PolicySourceArn` for each `SimulatePrincipalPolicy` request, so you must call this operation multiple times, once for each IAM user and role in your list\.
   + For the `ActionNames` list, specify every AWS KMS API action to simulate\. To simulate all AWS KMS API actions, use `kms:*`\. To test individual AWS KMS API actions, precede each API action with "`kms:`", for example "`kms:ListKeys`"\. For a complete list of all AWS KMS API actions, see [Actions](https://docs.aws.amazon.com/kms/latest/APIReference/API_Operations.html) in the *AWS Key Management Service API Reference*\.
   + \(Optional\) To determine whether the IAM users or roles have access to specific KMS CMKs, use the `ResourceArns` parameter to specify a list of the Amazon Resource Names \(ARNs\) of the CMKs\. To determine whether the IAM users or roles have access to any CMK, do not use the `ResourceArns` parameter\.

IAM responds to each `SimulatePrincipalPolicy` request with an evaluation decision: `allowed`, `explicitDeny`, or `implicitDeny`\. For each response that contains an evaluation decision of `allowed`, the response includes the name of the specific AWS KMS API operation that is allowed\. It also includes the ARN of the CMK that was used in the evaluation, if any\.