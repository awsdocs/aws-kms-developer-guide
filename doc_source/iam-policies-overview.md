# Overview of IAM policies<a name="iam-policies-overview"></a>

You can use IAM policies in the following ways:
+ **Attach a permissions policy to a user or a group** – You can attach a policy that allows an IAM user or group of users to call AWS KMS operations\.
+ **Attach a permissions policy to a role for federation or cross\-account permissions** – You can attach an IAM policy to an IAM role to enable identity federation, allow cross\-account permissions, or give permissions to applications running on EC2 instances\. For more information about the various use cases for IAM roles, see [IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) in the *IAM User Guide*\.

The following example shows an IAM policy with AWS KMS permissions\. This policy allows the IAM identities to which it is attached to list all KMS keys and aliases\.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
      "kms:ListKeys",
      "kms:ListAliases"
    ],
    "Resource": "*"
  }
}
```

Like all IAM policies, this policy doesn't have a `Principal` element\. When you attach an IAM policy to an IAM user or IAM role, the user or *assumed role user* gets the permissions specified in the policy\.

For a table showing all of the AWS KMS API actions and the resources that they apply to, see the [AWS KMS API permissions reference](kms-api-permissions-reference.md)\.