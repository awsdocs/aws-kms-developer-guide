# Authentication and access control for AWS KMS<a name="control-access"></a>

To use AWS KMS, you must have credentials that AWS can use to authenticate your requests\. The credentials must include permissions to access AWS resources: [AWS KMS keys](concepts.md#kms_keys) and [aliases](kms-alias.md)\. No AWS principal has any permissions to a KMS key unless that permission is provided explicitly and never denied\. There are no implicit or automatic permission to use or manage a KMS key\. 

The primary way to manage access to your AWS KMS resources is with *policies*\. Policies are documents that describe which principals can access which resources\. Policies attached to an IAM identity are called *identity\-based policies* \(or *IAM policies*\), and policies attached to other kinds of resources are called *resource policies*\. AWS KMS resource policies for KMS keys are called *key policies*\. All KMS keys have a key policy\.

To control access to your AWS KMS aliases, use IAM policies\. To allow principals to create aliases, you must provide the permission to the alias in an IAM policy and permission to the key in a key policy\. For details, see [Controlling access to aliases](alias-access.md)\.

To control access to your KMS keys, you can use the following policy mechanisms\.
+ **Key policy** – Every KMS key has a key policy\. It is the primary mechanism for controlling access to a KMS key\. You can use the key policy alone to control access, which means the full scope of access to the KMS key is defined in a single document \(the key policy\)\. For more information about using key policies, see [Key policies](key-policies.md)\.
+ **IAM policies** – You can use IAM policies in combination with the key policy and grants to control access to a KMS key\. Controlling access this way enables you to manage all of the permissions for your IAM identities in IAM\. To use an IAM policy to allow access to a KMS key, the key policy must explicitly allow it\. For more information about using IAM policies, see [IAM policies](iam-policies.md)\. 
+ **Grants** – You can use grants in combination with the key policy and IAM policies to allow access to a KMS key\. Controlling access this way enables you to allow access to the KMS key in the key policy, and to allow users to delegate their access to others\. For more information about using grants, see [Grants in AWS KMS](grants.md)\.

KMS keys belong to the AWS account in which they were created\. However, no identity or principal, including the AWS account root user, has permission to use or manage a KMS key unless that permission is explicitly provided in a key policy, IAM policy or grant\. The IAM user who creates a KMS key is not considered to be the key owner and they don't automatically have permission to use or manage the KMS key that they created\. Like any other principal, the key creator needs to get permission through a key policy, IAM policy, or grant\. However, principals who have the `kms:CreateKey` permission can set the initial key policy and give themselves permission to use or manage the key\.

The following topics provide details about how you can use AWS Identity and Access Management \(IAM\) and AWS KMS permissions to help secure your resources by controlling who can access them\.

**Topics**
+ [Concepts](#permission-concepts)
+ [Key policies](key-policies.md)
+ [IAM policies](iam-policies.md)
+ [Grants](grants.md)
+ [VPC endpoint](kms-vpc-endpoint.md)
+ [Condition keys](policy-conditions.md)
+ [Attribute\-based access control \(ABAC\)](abac.md)
+ [Cross\-account access](key-policy-modifying-external-accounts.md)
+ [Service\-linked roles](using-service-linked-roles.md)
+ [Hybrid post\-quantum TLS](pqtls.md)
+ [Determining access](determining-access.md)
+ [Permissions reference](kms-api-permissions-reference.md)

## Concepts in AWS KMS access control<a name="permission-concepts"></a>

Learn the concepts used in discussions of access control in AWS KMS\.

**Topics**
+ [Authentication](#authentication)
+ [Authorization](#authorization)
+ [AWS KMS resources](#kms-resources-operations)

### Authentication<a name="authentication"></a>

You can access AWS as any of the following types of identities:
+ **AWS account root user** – When you sign up for AWS, you provide an email address and password for your AWS account\. These are your *root credentials* and they provide complete access to all of your AWS resources\.
**Important**  
For security reasons, we recommend that you use the root credentials only to create an *administrator user*, which is an *IAM user* with full permissions to your AWS account\. Then, you can use this administrator user to create other IAM users and roles with limited permissions\. For more information, see [Create Individual IAM Users \(IAM Best Practices\)](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#create-iam-users) and [Creating An Admin User and Group](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html) in the *IAM User Guide*\.
**Note**  
No AWS principal, including the account root user or key creator, has any permissions to a KMS key unless they are explicitly allowed, and never denied, in a key policy, IAM policy, or grant\.
+ **IAM user** – An [IAM user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html) is an identity in your AWS account that has specific permissions \(for example, to use a KMS key\)\. You can use an IAM user name and password to sign in to secure AWS web pages like the [AWS Management Console](https://console.aws.amazon.com/), [AWS Discussion Forums](https://forums.aws.amazon.com/), or the [AWS Support Center](https://console.aws.amazon.com/support/home#/)\.

  In addition to a user name and password, you can also create [access keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html) for each user to enable the user to access AWS services programmatically, by using an [AWS SDK](https://aws.amazon.com/tools/#sdk), the [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/), or [AWS Tools for PowerShell](https://docs.aws.amazon.com/powershell/latest/userguide/)\. The SDKs and command line tools use the access keys to cryptographically sign API requests\. If you don't use the AWS tools, you must sign API requests yourself\. AWS KMS supports *Signature Version 4*, an AWS protocol for authenticating API requests\. For more information about authenticating API requests, see [Signature Version 4 Signing Process](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) in the *AWS General Reference*\.
+ **IAM role** – An [IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) is another IAM identity you can create in your account that has specific permissions\. It is similar to an IAM user, but it is not associated with a specific person\. An IAM role enables you to obtain temporary access keys to access AWS services and resources programmatically\. IAM roles are useful in the following situations:
  + **Federated user access** – Instead of creating an IAM user, you can use preexisting user identities from [AWS Directory Service](https://aws.amazon.com/directoryservice/), your enterprise user directory, or a web identity provider\. These are known as *federated users*\. Federated users use IAM roles through an [identity provider](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers.html)\. For more information about federated users, see [Federated Users and Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_access-management.html#intro-access-roles) in the *IAM User Guide*\.
  + **Cross\-account access** – You can use an IAM role in your AWS account to allow another AWS account permissions to access your account's resources\. For an example, see [Tutorial: Delegate Access Across AWS Accounts Using IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html) in the *IAM User Guide*\.
  + **AWS service access** – You can use an IAM role in your account to allow an AWS service permissions to access your account's resources\. For example, you can create a role that allows Amazon Redshift to access an S3 bucket on your behalf and then load data stored in the S3 bucket into an Amazon Redshift cluster\. For more information, see [Creating a Role to Delegate Permissions to an AWS Service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) in the *IAM User Guide*\.
  + **Applications running on EC2 instances** – Instead of storing access keys on an EC2 instance for use by applications that run on the instance and make AWS API requests, you can use an IAM role to provide temporary access keys for these applications\. To assign an IAM role to an EC2 instance, you create an *instance profile* and then attach it when you launch the instance\. An instance profile contains the role and enables applications running on the EC2 instance to get temporary access keys\. For more information, see [Using Roles for Applications on Amazon EC2](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html) in the *IAM User Guide*\.

### Authorization<a name="authorization"></a>

You can have valid credentials to authenticate your requests, but you also need permissions to make AWS KMS API requests to create, manage, or use AWS KMS resources\. For example, you must have permissions to create, manage, and use a KMS key for [cryptographic operations](concepts.md#cryptographic-operations)\.

Use key policies, IAM policies, and grants to control access to your AWS KMS resources\. You can use policy condition keys to grant access only when a request or resource meets the conditions you specify\. You can allow access to principals you trust in other AWS accounts\.

### AWS KMS resources<a name="kms-resources-operations"></a>

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