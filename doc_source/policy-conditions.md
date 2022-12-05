# Condition keys for AWS KMS<a name="policy-conditions"></a>

You can specify conditions in the key policies and AWS Identity and Access Management policies \([IAM policies](iam-policies.md)\) that control access to AWS KMS resources\. The policy statement is effective only when the conditions are true\. For example, you might want a policy statement to take effect only after a specific date\. Or, you might want a policy statement to control access only when a specific value appears in an API request\.

To specify conditions, you use *condition keys* in the [`Condition` element](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html) of a policy statement with [IAM condition operators](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html)\. Some condition keys apply generally to AWS; others are specific to AWS KMS\.

**Note**  
Condition key values must adhere to the character and encoding rules for AWS KMS key policies and IAM policies\. For details about key policy document rules, see [Key policy format](key-policy-overview.md#key-policy-format)\. For details about IAM policy document rules, see [IAM name requirements](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_iam-quotas.html#reference_iam-quotas-names) in the *IAM User Guide*\.\.

**Topics**
+ [AWS global condition keys](conditions-aws.md)
+ [AWS KMS condition keys](conditions-kms.md)
+ [AWS KMS condition keys for AWS Nitro Enclaves](conditions-nitro-enclaves.md)