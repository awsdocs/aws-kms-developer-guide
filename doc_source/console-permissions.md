# Permissions required to use the AWS KMS console<a name="console-permissions"></a>

To work with the AWS KMS console, users must have a minimum set of permissions that allow them to work with the AWS KMS resources in their AWS account\. In addition to these AWS KMS permissions, users must also have permissions to list IAM users and roles\. If you create an IAM policy that is more restrictive than the minimum required permissions, the AWS KMS console won't function as intended for users with that IAM policy\.

For the minimum permissions required to allow a user read\-only access to the AWS KMS console, see [Allow a user to view KMS keys in the AWS KMS console](customer-managed-policies.md#iam-policy-example-read-only-console)\.

To allow users to work with the AWS KMS console to create and manage KMS keys, attach the **AWSKeyManagementServicePowerUser** managed policy to the user, as described in the following section\.

You don't need to allow minimum console permissions for users that are working with the AWS KMS API through the [AWS SDKs](https://aws.amazon.com/tools/#sdk), [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/) or [AWS Tools for PowerShell](https://docs.aws.amazon.com/powershell/latest/userguide/)\. However, you do need to grant these users permission to use the API\. For more information, see [AWS KMS API permissions reference](kms-api-permissions-reference.md)\.