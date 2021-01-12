# AWS KMS permissions<a name="kms-api-permissions-reference"></a>

The [AWS KMS permissions table](#kms-api-permissions-reference-table) lists the permissions for each AWS KMS operation so you can use them correctly in [key policies](key-policies.md) and [IAM policies](iam-policies.md)\. 

**Important**  
Be cautious when giving principals permission to create and manage the policies and grants that control access to your customer master keys \(CMKs\)\. Principals who have permission to manage tags and aliases can also control access to a CMK\. For details, see [Using ABAC for AWS KMS](abac.md)\.

**Note**  
You might need to scroll horizontally and vertically to see all of the data in this table\.

<a name="kms-api-permissions-reference-table"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/kms-api-permissions-reference.html)

The columns in this table provide the following information:
+ **Actions and permissions **lists each AWS KMS API operation and the permission that allows the operation\. You specify the operation in `Action` element of a policy statement\.
+ **Policy type** indicates whether the permission can be used in a key policy or IAM policy\. 

  *Key policy* means that you can specify the permission in the key policy\. When the key policy contains the [policy statement that enables IAM policies](key-policies.md#key-policy-default-allow-root-enable-iam), you can specify the permission in an IAM policy\. 

  *IAM policy* means that you can specify the permission only in an IAM policy\.
+ **Resources** lists the AWS KMS resources to which the permissions apply\. AWS KMS supports two resource types: a customer master key \(CMK\) and an alias\. In a key policy, the value of the `Resource` element is always `*`, which indicates the CMK to which the key policy is attached\. 

  Use the following values to represent an AWS KMS resource in an IAM policy\.  
**CMK**  
When the resource is a customer master key \(CMK\), use its [key ARN](concepts.md#key-id-key-ARN)\. For help, see [Finding the key ID and ARN](find-cmk-id-arn.md)\.  
`arn:AWS_partition_name:kms:AWS_Region:AWS_account_ID:key/key_ID`  
For example:  
arn:aws:kms:us\-west\-2:111122223333:key/1234abcd\-12ab\-34cd\-56ef\-1234567890ab  
**Alias**  
When the resource is an alias, use its [alias ARN](concepts.md#key-id-alias-ARN)\. For help, see [Finding the alias name and alias ARN](find-cmk-alias.md)\.  
`arn:AWS_partition_name:kms:AWS_region:AWS_account_ID:alias/alias_name`  
For example:  
arn:aws:kms:us\-west\-2:111122223333:alias/ExampleAlias  
**`*` \(asterisk\)**  
When the permission doesn't apply to a particular resource \(CMK or alias\), use an asterisk \(`*`\)\.  
In an IAM policy for an AWS KMS permission, an asterisk in the `Resource` element indicates all AWS KMS resources \(CMKs and aliases\)\. You can also use an asterisk in the `Resource` element when the AWS KMS permission doesn't apply to any particular CMKs or aliases\. For example, when allowing or denying `kms:CreateKey` or `kms:ListKeys` permission, you can set the `Resource` element to `*` or to an account\-specific variation, such as `arn:AWS_partition_name:kms:AWS_region:AWS_account_ID:*`\.
+ **AWS KMS condition keys** lists the [AWS KMS condition keys](policy-conditions.md) that you can use to limit a permission\. You specify conditions in a policy's `Condition` element\. This column also includes [AWS global condition keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) that are supported by AWS KMS, but not by all AWS services\.