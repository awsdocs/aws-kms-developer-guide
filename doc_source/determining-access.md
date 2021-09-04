# Determining access to AWS KMS keys<a name="determining-access"></a>

To determine the full extent of who or what currently has access to a AWS KMS key, you must examine the key policy of the KMS key, all [grants](grants.md) that apply to the KMS key, and potentially all AWS Identity and Access Management \(IAM\) policies\. You might do this to determine the scope of potential usage of a KMS key, or to help you meet compliance or auditing requirements\. The following topics can help you generate a complete list of the AWS principals \(identities\) that currently have access to a KMS key\.

**Topics**
+ [Examining the key policy](determining-access-key-policy.md)
+ [Examining IAM policies](determining-access-iam-policies.md)
+ [Examining grants](determining-access-grants.md)
+ [Troubleshooting key access](policy-evaluation.md)