# AWS KMS API permissions: Actions and resources reference<a name="kms-api-permissions-reference"></a>

The Actions and Resources Table is designed to help you define [access control](control-access.md#authorization) in [key policies](key-policies.md) and [IAM policies](iam-policies.md)\. The columns provide the following information:
+ **API operations and actions \(permissions\) **lists each AWS KMS API operation and the corresponding action \(permission\) that allows the operation\. You specify actions in a policy's `Action` element\. 
+ **Policy type** indicates whether the permission can be used in a key policy or IAM policy\. When the type is *key policy*, you can specify the permission explicitly in the key policy\. Or, if the key policy contains the [policy statement that enables IAM policies](key-policies.md#key-policy-default-allow-root-enable-iam), you can specify the permission in an IAM policy\. When the type is *IAM policy*, you can specify the permission only in an IAM policy\.
+ **Resources** lists the AWS resources to which the permissions apply\. To specify a resource in an IAM policy, add its Amazon Resource Name \(ARN\) to the `Resource` element\. In a key policy, which is attached to a resource, the value of the `Resource` element is always `"*"`\.   
CMK ARNs  
When the resource is a customer master key \(CMK\), use its [key ARN](concepts.md#key-id-key-ARN)\. For help, see [Finding the key ID and ARN](find-cmk-id-arn.md)\.  
`arn:AWS_partition_name:kms:AWS_Region:AWS_account_ID:key/key_ID`  
For example:  
arn:aws:kms:us\-west\-2:111122223333:key/1234abcd\-12ab\-34cd\-56ef\-1234567890ab  
Alias ARNs  
When the resource is an alias, use its [alias ARN](concepts.md#key-id-alias-ARN)\. To get the alias ARN, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\.  
`arn:AWS_partition_name:kms:AWS_region:AWS_account_ID:alias/alias_name`  
For example:  
arn:aws:kms:us\-west\-2:111122223333:alias/ExampleAlias
+ **AWS KMS condition keys** lists the AWS KMS condition keys that you can use to control access to the operation\. You specify conditions in a policy's `Condition` element\. For more information, see [AWS KMS condition keys](policy-conditions.md#conditions-kms)\. This column also includes [AWS global condition keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) that are supported by AWS KMS, but not by all AWS services\.


**AWS KMS API operations and permissions**  
<a name="kms-api-permissions-reference-table"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/kms-api-permissions-reference.html)