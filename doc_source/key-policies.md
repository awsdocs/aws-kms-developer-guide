# Key policies in AWS KMS<a name="key-policies"></a>

Key policies are the primary way to control access to AWS KMS keys\. Every KMS key must have exactly one key policy\. The statements in the key policy document determine who has permission to use the KMS key and how they can use it\. You can also use [IAM policies](iam-policies.md) and [grants](grants.md) to control access to the KMS key, but every KMS key must have a key policy\. For more information, see [Managing access to KMS keys](control-access-overview.md#managing-access)\.

For help writing and formatting a JSON policy document, see the [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) in the *IAM User Guide*\.

**Topics**
+ [Overview of key policies](#key-policy-overview)
+ [Example key policy](#key-policy-example)
+ [Default key policy](key-policy-default.md)
+ [Permissions for AWS services](key-policy-services.md)
+ [Managing key policies](key-policy-manage.md)

## Overview of key policies<a name="key-policy-overview"></a>

Every KMS key must have exactly one key policy\. This key policy controls access only to its associated KMS key, along with IAM policies and grants\. Unlike IAM policies, which are global, key policies are Regional\. Each key policy is effective only in the Region that hosts the KMS key\.

A key policy is implemented as a [JSON \(JavaScript Object Notation\)](http://json.org/) document of up to [32 KB](resource-limits.md#key-policy-limit) \(32,768 bytes\)\. You can create and manage key policies in the AWS KMS console or by using AWS KMS API operations, such as [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) and [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html)\. 

Key policy documents use the same JSON syntax as other policy documents in AWS and have the following basic structure:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Describe the policy statement",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111122223333:user/Alice"
      },
      "Action": "kms:DescribeKey",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "kms:KeySpec": "SYMMETRIC_DEFAULT"
        }
      }
    }
  ]
}
```

For information about using the console default view for key policies, see [Default key policy](key-policy-default.md) and [Changing a key policy](key-policy-modifying.md)\. For help writing and formatting a JSON policy document, see the [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) in the *IAM User Guide*\.

A key policy document must have a `Version` element\. We recommend setting the version to `2012-10-17` \(the latest version\)\. In addition, a key policy document must have one or more statements, and each statement consists of up to six elements:
+ **Sid** – \(Optional\) The Sid is a statement identifier, an arbitrary string you can use to identify the statement\.
+ **Effect** – \(Required\) The effect specifies whether to allow or deny the permissions in the policy statement\. The Effect must be Allow or Deny\. If you don't explicitly allow access to a KMS key, access is implicitly denied\. You can also explicitly deny access to a KMS key\. You might do this to make sure that a user cannot access it, even when a different policy allows access\.
+ **Principal** – \(Required\) The [principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#Principal_specifying) is the identity that gets the permissions specified in the policy statement\. You can specify AWS accounts \(root\), IAM users, IAM roles, and some AWS services as principals in a key policy\. IAM groups are not valid principals\. 

  When the principal is another AWS account or its principals, the permissions are effective only when the account is enabled in the Region with the KMS key and key policy\. For information about Regions that are not enabled by default \("opt\-in Regions"\), see [Managing AWS Regions](https://docs.aws.amazon.com/general/latest/gr/rande-manage.html) in the *AWS General Reference*\.
**Note**  
Do not set the Principal to an asterisk \(\*\) in any key policy statement that allows permissions unless you use conditions to limit the key policy\. An asterisk gives every identity in every AWS account permission to use the KMS key, unless another policy statement explicitly denies it\. Users in other AWS accounts just need corresponding IAM permissions in their own accounts to use the KMS key\.
+ **Action** – \(Required\) Actions specify the API operations to allow or deny\. For example, the `kms:Encrypt` action corresponds to the AWS KMS [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation\. You can list more than one action in a policy statement\. For more information, see [Permissions reference](kms-api-permissions-reference.md)\.
+ **Resource** – \(Required\) In a key policy, the value of the Resource element is `"*"`, which means "this KMS key\." The asterisk \(`"*"`\) identifies the KMS key to which the key policy is attached\.
+ **Condition** – \(Optional\) Conditions specify requirements that must be met for a key policy to take effect\. With conditions, AWS can evaluate the context of an API request to determine whether or not the policy statement applies\. 

  The format for a condition is:

  ```
  "Condition": {"condition operator": {"condition key": "condition value"}}
  ```

  such as:

  ```
  "Condition": {"StringEquals": {"kms:CallerAccount": "111122223333"}}
  ```

  For more information, see [Policy conditions](policy-conditions.md)\.

For more information about AWS policy syntax, see [AWS IAM Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) in the *IAM User Guide*\.

## Example key policy<a name="key-policy-example"></a>

The following example shows a complete key policy for a symmetric KMS key\. This key policy combines the example policy statements from the preceding [default key policy](key-policy-default.md) section into a single key policy that accomplishes the following:
+ Allows the AWS account \(root user\) 111122223333 full access to the KMS key, and thus enables IAM policies in the account to allow access to the KMS key\.
+ Allows IAM user KMSAdminUser and IAM role KMSAdminRole to administer the KMS key\.
+ Allows IAM user `ExampleUser`, IAM role `ExampleRole`, and AWS account 444455556666 to use the KMS key\.

```
{
  "Version": "2012-10-17",
  "Id": "key-consolepolicy-2",
  "Statement": [
    {
      "Sid": "Enable IAM policies",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::111122223333:root"},
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow access for Key Administrators",
      "Effect": "Allow",
      "Principal": {"AWS": [
        "arn:aws:iam::111122223333:user/KMSAdminUser",
        "arn:aws:iam::111122223333:role/KMSAdminRole"
      ]},
      "Action": [
        "kms:Create*",
        "kms:Describe*",
        "kms:Enable*",
        "kms:List*",
        "kms:Put*",
        "kms:Update*",
        "kms:Revoke*",
        "kms:Disable*",
        "kms:Get*",
        "kms:Delete*",
        "kms:TagResource",
        "kms:UntagResource",
        "kms:ScheduleKeyDeletion",
        "kms:CancelKeyDeletion"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Allow use of the key",
      "Effect": "Allow",
      "Principal": {"AWS": [
        "arn:aws:iam::111122223333:user/ExampleUser",
        "arn:aws:iam::111122223333:role/ExampleRole",
        "arn:aws:iam::444455556666:root"
      ]},
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Allow attachment of persistent resources",
      "Effect": "Allow",
      "Principal": {"AWS": [
        "arn:aws:iam::111122223333:user/ExampleUser",
        "arn:aws:iam::111122223333:role/ExampleRole",
        "arn:aws:iam::444455556666:root"
      ]},
      "Action": [
        "kms:CreateGrant",
        "kms:ListGrants",
        "kms:RevokeGrant"
      ],
      "Resource": "*",
      "Condition": {"Bool": {"kms:GrantIsForAWSResource": "true"}}
    }
  ]
}
```