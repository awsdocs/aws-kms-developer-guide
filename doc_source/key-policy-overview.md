# Creating a key policy<a name="key-policy-overview"></a>

You can create and manage key policies in the AWS KMS console, by using AWS KMS API operations, such as [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html), [ReplicateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReplicateKey.html), and [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html), or by using an [AWS CloudFormation template](creating-resources-with-cloudformation.md)\. 

When you create a KMS key in the AWS KMS console, the console walks you through the steps of creating a key policy based on the [default key policy for the console](key-policy-default.md)\. When you use the `CreateKey` or `ReplicateKey` APIs, if you don't specify a key policy, these APIs apply the [default key policy for keys created programmatically](key-policy-default.md)\. When you use the `PutKeyPolicy` API or create a KMS key by using a AWS CloudFormation template, you are required to specify a key policy\.

Each policy document can have one or more policy statements\. The following example shows a valid key policy document with one policy statement\.

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

**Topics**
+ [Key policy format](#key-policy-format)
+ [Elements in a key policy](#key-policy-elements)
+ [Example key policy](#key-policy-example)

## Key policy format<a name="key-policy-format"></a>

A key policy document must conform to the following rules:
+ Up to 32 kilobytes \(32,768 bytes\)
+ The `Sid` element in a key policy statement can include spaces\. \(Spaces are prohibited in the `Sid` element of an IAM policy document\.\)

A key policy document can include only the following characters:
+ Printable ASCII characters
+ Printable characters in the Basic Latin and Latin\-1 Supplement character set
+ The tab \(`\u0009`\), line feed \(`\u000A`\), and carriage return \(`\u000D`\) special characters

## Elements in a key policy<a name="key-policy-elements"></a>

A key policy document must have the following elements:

**Version**  
Specifies the key policy document version\. Set the version to `2012-10-17` \(the latest version\)\.

**Statement**  
Encloses the policy statements\. A key policy document must have at least one statement\.  
Each key policy statement consists of up to six elements\. The `Effect`, `Principal`, `Action`, and `Resource` elements are required\.    
**Sid **  
\(Optional\) The statement identifier \(`Sid`\) an arbitrary string you can use to describe the statement\. The `Sid` in a key policy can include spaces\. \(You can't include spaces in an IAM policy `Sid` element\.\)  
**Effect**  
\(Required\) Determines whether to allow or deny the permissions in the policy statement\. Valid values are `Allow` or `Deny`\. If you don't explicitly allow access to a KMS key, access is implicitly denied\. You can also explicitly deny access to a KMS key\. You might do this to make sure that a user cannot access it, even when a different policy allows access\.  
**Principal**  
\(Required\) The [principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#Principal_specifying) is the identity that gets the permissions specified in the policy statement\. You can specify AWS accounts, IAM users, IAM roles, and some AWS services as principals in a key policy\. IAM [user groups](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups.html) are not a valid principal in any policy type\.   
When the principal in a key policy statement is an [https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-accounts](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-accounts) expressed as `arn:aws:iam::111122223333:root"`, the policy statement doesn't give permission to any IAM principal\. Instead, it gives the AWS account permission to use IAM policies to delegate the permissions specified in the key policy\. \(A principal in `arn:aws:iam::111122223333:root"` format does *not* represent the [AWS account root user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html), despite the use of "root" in the account identifier\. However, the account principal represents the account and its administrators, including the account root user\.\)  
When the principal is another AWS account or its principals, the permissions are effective only when the account is enabled in the Region with the KMS key and key policy\. For information about Regions that are not enabled by default \("opt\-in Regions"\), see [Managing AWS Regions](https://docs.aws.amazon.com/general/latest/gr/rande-manage.html) in the *AWS General Reference*\.  
Do not set the Principal to an asterisk \(\*\) in any key policy statement that allows permissions unless you use conditions to limit the key policy\. An asterisk gives every identity in every AWS account permission to use the KMS key, unless another policy statement explicitly denies it\. Users in other AWS accounts just need corresponding IAM permissions in their own accounts to use the KMS key\.
To allow a different AWS account or its principals to use a KMS key, you must provide permission in a key policy and in an IAM policy in the other account\. For details, see [Allowing users in other accounts to use a KMS key](key-policy-modifying-external-accounts.md)\.  
**Action**  
\(Required\) Specify the API operations to allow or deny\. For example, the `kms:Encrypt` action corresponds to the AWS KMS [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation\. You can list more than one action in a policy statement\. For more information, see [Permissions reference](kms-api-permissions-reference.md)\.  
**Resource**  
\(Required\) In a key policy, the value of the Resource element is `"*"`, which means "this KMS key\." The asterisk \(`"*"`\) identifies the KMS key to which the key policy is attached\.  
If the required `Resource` element is missing from a key policy statement, the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) and [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) APIs succeed, but the policy statement has no effect\. A key policy statement without a `Resource` element doesn't apply to any KMS key\.   
**Condition**  
\(Optional\) Conditions specify requirements that must be met for a key policy to take effect\. With conditions, AWS can evaluate the context of an API request to determine whether or not the policy statement applies\.   
To specify conditions, you use predefined *condition keys*\. AWS KMS supports [AWS global condition keys](policy-conditions.md#conditions-aws) and [AWS KMS condition keys](policy-conditions.md#conditions-kms)\. To support attribute\-based access control \(ABAC\), AWS KMS provides condition keys that control access to a KMS key based on tags and aliases\. For details, see [ABAC for AWS KMS](abac.md)\.  
The format for a condition is:  

```
"Condition": {"condition operator": {"condition key": "condition value"}}
```
such as:  

```
"Condition": {"StringEquals": {"kms:CallerAccount": "111122223333"}}
```

For more information about AWS policy syntax, see [AWS IAM Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) in the *IAM User Guide*\.

## Example key policy<a name="key-policy-example"></a>

The following example shows a complete key policy for a symmetric encryption KMS key\. You can use it for reference as you read about the key policy concepts in this chapter\. This key policy combines the example policy statements from the preceding [default key policy](key-policy-default.md) section into a single key policy that accomplishes the following:
+ Allows the example AWS account, 111122223333, full access to the KMS key\. It allows the account and its administrators, including the account root user, to use IAM policies in the account to allow access to the KMS key\.
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