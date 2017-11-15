# AWS KMS API Permissions: Actions and Resources Reference<a name="kms-api-permissions-reference"></a>

When you are setting up access control with key policies and IAM policies, you can use the following table as a reference\. The first column in the table lists each AWS KMS API operation and the corresponding action \(permission\) that allows the operation\. You specify actions in a policy's `Action` element\. The remaining columns provide the following additional information:

+ The type of policy you must use to allow permissions to perform the operation\. When the key policy is required, you can allow the permissions directly in the key policy, or you can ensure the key policy contains the policy statement that enables IAM policies and then allow the permissions in an IAM policy\.

+ The resource or resources for which you can allow the operation\. You specify resources in a policy's `Resource` element\. For key policies, you always specify `"*"` for the resource, which effectively means "this CMK\." A key policy applies only to the CMK it is attached to\. For IAM policies, you can specify the Amazon Resource Name \(ARN\) for a specific resource or set of resources\.

+ The AWS KMS condition keys you can use to control access to the operation\. You specify conditions in a policy's `Condition` element\. For more information, see [AWS KMS Condition Keys](policy-conditions.md#conditions-kms)\.

If you see an expand arrow \(**↗**\) in the upper\-right corner of the table, you can open the table in a new window\. To close the window, choose the close button \(**X**\) in the lower\-right corner\.


**AWS KMS API Operations and Permissions**  
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/kms-api-permissions-reference.html)