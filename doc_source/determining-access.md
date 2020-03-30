# Determining access to an AWS KMS customer master key<a name="determining-access"></a>

To determine the full extent of who or what currently has access to a customer master key \(CMK\) in AWS KMS, you must examine the CMK's key policy, all [grants](grants.md) that apply to the CMK, and potentially all AWS Identity and Access Management \(IAM\) policies\. You might do this to determine the scope of potential usage of a CMK, or to help you meet compliance or auditing requirements\. The following topics can help you generate a complete list of the AWS principals \(identities\) that currently have access to a CMK\.

**Topics**
+ [Examining the key policy](determining-access-key-policy.md)
+ [Examining IAM policies](determining-access-iam-policies.md)
+ [Examining grants](determining-access-grants.md)
+ [Troubleshooting key access](policy-evaluation.md)