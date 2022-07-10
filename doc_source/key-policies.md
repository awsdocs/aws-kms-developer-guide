# Key policies in AWS KMS<a name="key-policies"></a>

A key policy is a resource policy for an AWS KMS key\. Key policies are the primary way to control access to KMS keys\. Every KMS key must have exactly one key policy\. The statements in the key policy determine who has permission to use the KMS key and how they can use it\. You can also use [IAM policies](iam-policies.md) and [grants](grants.md) to control access to the KMS key, but every KMS key must have a key policy\. 

No AWS principal, including the account root user or key creator, has any permissions to a KMS key unless they are explicitly allowed, and never denied, in a key policy, IAM policy, or grant\. 

Unless the key policy explicitly allows it, you cannot use IAM policies to *allow* access to a KMS key\. Without permission from the key policy, IAM policies that allow permissions have no effect\. \(You can use an IAM policy to *deny* a permission to a KMS key without permission from a key policy\.\) The default key policy enables IAM policies\. To enable IAM policies in your key policy, add the policy statement described in [Allows access to the AWS account and enables IAM policies](key-policy-default.md#key-policy-default-allow-root-enable-iam)\.

Unlike IAM policies, which are global, key policies are Regional\. A key policy controls access only to a KMS key in the same Region\. It has no effect on KMS keys in other Regions\.

**Topics**
+ [Creating a key policy](key-policy-overview.md)
+ [Default key policy](key-policy-default.md)
+ [Viewing a key policy](key-policy-viewing.md)
+ [Changing a key policy](key-policy-modifying.md)
+ [Permissions for AWS services](key-policy-services.md)