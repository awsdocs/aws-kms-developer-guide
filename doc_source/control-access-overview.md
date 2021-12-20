# Managing access to your AWS KMS resources<a name="control-access-overview"></a>

Every AWS resource belongs to an AWS account\. Permissions to create or access the resources are defined in permissions policies in that account\. An account administrator can attach permissions policies to IAM identities \(that is, users, groups, and roles\), and some services \(including AWS KMS\) also support attaching permissions policies to other kinds of resources\.

**Note**  
An *account administrator* \(or administrator user\) is a user with administrator permissions\. For more information, see [Creating an Admin User and Group](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html) in the *IAM User Guide*\.

When managing permissions, you decide who gets the permissions, the resources they get permissions for, and the specific actions allowed\.

**Topics**
+ [AWS KMS resources and operations](#kms-resources-operations)
+ [Managing access to KMS keys](#managing-access)
+ [Specifying permissions in a policy](#overview-policy-elements)
+ [Specifying conditions in a policy](#overview-policy-conditions)

## AWS KMS resources and operations<a name="kms-resources-operations"></a>

In AWS KMS, the primary resource is a [AWS KMS keys](concepts.md#kms_keys)\. AWS KMS also supports an [alias](kms-alias.md), an independent resource that provides a friendly name for a KMS key\. Some AWS KMS operations allow you to use an alias to identify a KMS key\.

Each instance of a KMS key or alias has a unique [Amazon Resource Name](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html#arns-syntax) \(ARN\) with a standard format\. In AWS KMS resources, the AWS service name is `kms`\. 
+ **AWS KMS key**

  ARN format:

  `arn:AWS partition name:AWS service name:AWS Region:AWS account ID:key/key ID`

  Example ARN:

  `arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab`
+ **Alias**

  ARN format:

  `arn:AWS partition name:AWS service name:AWS Region:AWS account ID:alias/alias name`

  Example ARN:

  `arn:aws:kms:us-west-2:111122223333:alias/example-alias`

AWS KMS provides a set of API operations to work with your AWS KMS resources\. For more information about identifying KMS keys in the AWS Management Console and AWS KMS API operations, see [Key identifiers \(KeyId\)](concepts.md#key-id)\. For a list of AWS KMS operations, see the [AWS Key Management Service API Reference](https://docs.aws.amazon.com/kms/latest/APIReference/)\.

## Managing access to KMS keys<a name="managing-access"></a>

The primary way to manage access to your KMS keys is with *policies*\. Policies are documents that describe which principals can access which resources\. Policies attached to an IAM identity are called *identity\-based policies* \(or *IAM policies*\), and policies attached to other kinds of resources are called *resource\-based policies*\. In AWS KMS, you must attach resource\-based policies to your KMS keys\. These are called *key policies*\. All KMS keys have a key policy\.

You can control access to your KMS keys in these ways:
+ **Use the key policy** – You must use the key policy to control access to a KMS key\. You can use the key policy alone to control access, which means the full scope of access to the KMS key is defined in a single document \(the key policy\)\.
+ **Use IAM policies in combination with the key policy** – You can use IAM policies in combination with the key policy to control access to a KMS key\. Controlling access this way enables you to manage all of the permissions for your IAM identities in IAM\.
+ **Use grants in combination with the key policy** – You can use grants in combination with the key policy to allow access to a KMS key\. Controlling access this way enables you to allow access to the KMS key in the key policy, and to allow users to delegate their access to others\.

For most AWS services, IAM policies are the only way to control access to the service's resources\. Some services offer resource\-based policies or other access control mechanisms to complement IAM policies, but these are generally optional and you can control access to the resources in these services with only IAM policies\. This is not the case for AWS KMS, however\. To allow access to a KMS key, you must use the key policy, either alone or in combination with IAM policies or grants\. IAM policies by themselves are not sufficient to allow access to a KMS key, though you can use them in combination with a key policy\.

KMS keys belong to the AWS account in which they were created\. The IAM user who creates a KMS key is not considered to be the key owner and they don't automatically have permission to use or manage the KMS key that they created\. Like any other principal, the key creator needs to get permission through a key policy, IAM policy, or grant\. However, principals who have the `kms:CreateKey` permission can set the initial key policy and give themselves permission to use or manage the key\.

For more information about using key policies, see [Key policies](key-policies.md)\.

For more information about using IAM policies, see [IAM policies](iam-policies.md)\.

For more information about using grants, see [Grants in AWS KMS](grants.md)\.

## Specifying permissions in a policy<a name="overview-policy-elements"></a>

AWS KMS provides a set of API operations\. To control access to these API operations, AWS KMS provides a set of *actions* that you can specify in a policy\. For more information, see [Permissions reference](kms-api-permissions-reference.md)\.

A policy is a document that describes a set of permissions\. The following are the basic elements of a policy\.
+ **Resource** – In an IAM policy, you use an Amazon Resource Name \(ARN\) to specify the resource that the policy applies to\. For more information, see [AWS KMS resources and operations](#kms-resources-operations)\. In a key policy, you use `"*"` for the resource, which effectively means "this KMS key\." A key policy applies only to the KMS key it is attached to\.
+ **Action** – You use actions to specify the API operations you want to allow or deny\. For example, the `kms:Encrypt` action corresponds to the AWS KMS [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation\.
+ **Effect** – You use the effect to specify whether to allow or deny the permissions\. If you don't explicitly allow access to a resource, access is implicitly denied\. You can also explicitly deny access to a resource, which you might do to make sure that a user cannot access it, even when a different policy allows access\.
+ **Principal** – In an IAM policy, you don't specify an [AWS principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/intro-structure.html#intro-structure-principal)\. Instead, the identity \(the IAM user, group, or role\) that the policy is attached to is the implicit principal\. In a key policy, you must specify the principal \(the identity\) that the permissions apply to\. You can specify AWS accounts \(root\), IAM users, IAM roles, and some AWS services as principals in a key policy\. IAM groups are not valid principals in a key policy\.

For more information, see [Key policies](key-policies.md) and [IAM policies](iam-policies.md)\.

## Specifying conditions in a policy<a name="overview-policy-conditions"></a>

You can use another policy element called a *condition key* to specify the circumstances in which a policy takes effect\. For example, you might want a policy statement to take effect only after a specific date\. Or, you might want a policy statement to control access based on whether a specific value exists in the API request\.

To specify conditions, you use predefined *condition keys*\. Some condition keys apply generally to AWS, and some are specific to AWS KMS\. For more information, see [Policy conditions](policy-conditions.md)\.

To support attribute\-based access control \(ABAC\), AWS KMS provides condition keys that control access to a KMS key based on tags and aliases\. For details, see [ABAC for AWS KMS](abac.md)\.