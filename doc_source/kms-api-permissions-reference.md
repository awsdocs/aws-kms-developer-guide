# AWS KMS API Permissions: Actions and Resources Reference<a name="kms-api-permissions-reference"></a>

The Actions and Resources Table is designed to help you define [access control](control-access.md#authorization) in [key policies](key-policies.md) and [IAM policies](iam-policies.md)\. The columns provide the following information:
+ **API Operations and Actions \(Permissions\) **lists each AWS KMS API operation and the corresponding action \(permission\) that allows the operation\. You specify actions in a policy's `Action` element\. 
+ **Policy Type** indicates whether the permission can be used in a key policy or IAM policy\. When the type is *key policy*, you can specify the permission explicitly in the key policy\. Or, if the key policy contains the [policy statement that enables IAM policies](key-policies.md#key-policy-default-allow-root-enable-iam), you can specify the permission in an IAM policy\. When the type is *IAM policy*, you can specify the permission only in an IAM policy\.
+ **Resources and ARNs** lists the resources for which you can allow the operation\. To specify a resource in an IAM policy, type the Amazon Resource Name \(ARN\) in the `Resource` element\. Because a key policy applies only to the CMK that it is attached to, the value of its `Resource` element is always `"*"`\. 
+ **AWS KMS Condition Keys** lists the AWS KMS condition keys that you can use to control access to the operation\. You specify conditions in a policy's `Condition` element\. For more information, see [AWS KMS Condition Keys](policy-conditions.md#conditions-kms)\. This column also includes [AWS global condition keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) that are supported by AWS KMS, but not by all AWS services\.

If you see an expand arrow \(**↗**\) in the upper\-right corner of the table, you can open the table in a new window\. To close the window, choose the close button \(**X**\) in the lower\-right corner\.


**AWS KMS API Operations and Permissions**  
<a name="kms-api-permissions-reference-table"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/kms-api-permissions-reference.html)