# Authentication and Access Control for AWS KMS<a name="control-access"></a>

Access to AWS KMS requires credentials that AWS can use to authenticate your requests\. The credentials must have permissions to access AWS resources, such as AWS KMS customer master keys \(CMKs\)\. The following sections provide details about how you can use AWS Identity and Access Management \(IAM\) and AWS KMS to help secure your resources by controlling who can access them\.

**Topics**

+ [Authentication](#authentication)

+ [Access Control](#authorization)

## Authentication<a name="authentication"></a>

You can access AWS as any of the following types of identities:

+ **AWS account root user** – When you sign up for AWS, you provide an email address and password for your AWS account\. These are your *root credentials* and they provide complete access to all of your AWS resources\.
**Important**  
For security reasons, we recommend that you use the root credentials only to create an *administrator user*, which is an *IAM user* with full permissions to your AWS account\. Then, you can use this administrator user to create other IAM users and roles with limited permissions\. For more information, see [Create Individual IAM Users \(IAM Best Practices\)](http://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#create-iam-users) and [Creating An Admin User and Group](http://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html) in the *IAM User Guide*\.

+ **IAM user** – An [IAM user](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html) is an identity within your AWS account that has specific permissions \(for example, to use a KMS CMK\)\. You can use an IAM user name and password to sign in to secure AWS webpages like the [AWS Management Console](https://console.aws.amazon.com/), [AWS Discussion Forums](https://forums.aws.amazon.com/), or the [AWS Support Center](https://console.aws.amazon.com/support/home#/)\.

   

  In addition to a user name and password, you can also create [access keys](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html) for each user to enable the user to access AWS services programmatically, through [one of the AWS SDKs](https://aws.amazon.com/tools/#sdk) or the [command line tools](https://aws.amazon.com/tools/#cli)\. The SDKs and command line tools use the access keys to cryptographically sign API requests\. If you don't use the AWS tools, you must sign API requests yourself\. AWS KMS supports *Signature Version 4*, an AWS protocol for authenticating API requests\. For more information about authenticating API requests, see [Signature Version 4 Signing Process](http://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) in the *AWS General Reference*\.

   

+ **IAM role** – An [IAM role](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) is another IAM identity you can create in your account that has specific permissions\. It is similar to an IAM user, but it is not associated with a specific person\. An IAM role enables you to obtain temporary access keys to access AWS services and resources programmatically\. IAM roles are useful in the following situations:

   

  + **Federated user access** – Instead of creating an IAM user, you can use preexisting user identities from [AWS Directory Service](https://aws.amazon.com/directoryservice/), your enterprise user directory, or a web identity provider\. These are known as *federated users*\. Federated users use IAM roles through an [identity provider](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers.html)\. For more information about federated users, see [Federated Users and Roles](http://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_access-management.html#intro-access-roles) in the *IAM User Guide*\.

     

  + **Cross\-account access** – You can use an IAM role in your AWS account to allow another AWS account permissions to access your account's resources\. For an example, see [Tutorial: Delegate Access Across AWS Accounts Using IAM Roles](http://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html) in the *IAM User Guide*\.

     

  + **AWS service access** – You can use an IAM role in your account to allow an AWS service permissions to access your account's resources\. For example, you can create a role that allows Amazon Redshift to access an S3 bucket on your behalf and then load data stored in the S3 bucket into an Amazon Redshift cluster\. For more information, see [Creating a Role to Delegate Permissions to an AWS Service](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) in the *IAM User Guide*\.

     

  + **Applications running on EC2 instances** – Instead of storing access keys on an EC2 instance for use by applications that run on the instance and make AWS API requests, you can use an IAM role to provide temporary access keys for these applications\. To assign an IAM role to an EC2 instance, you create an *instance profile* and then attach it when you launch the instance\. An instance profile contains the role and enables applications running on the EC2 instance to get temporary access keys\. For more information, see [Using Roles for Applications on Amazon EC2](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html) in the *IAM User Guide*\.

## Access Control<a name="authorization"></a>

You can have valid credentials to authenticate your requests, but you also need permissions to make AWS KMS API requests to create, manage, or use AWS KMS resources\. For example, you must have permissions to create a KMS CMK, to manage the CMK, to use the CMK for cryptographic operations \(such as encryption and decryption\), and so on\.

The following pages describe how to manage permissions for AWS KMS\. We recommend that you read the overview first\.

+ [Overview of Managing Access](control-access-overview.md)

+ [Using Key Policies](key-policies.md)

+ [Using IAM Policies](iam-policies.md)

+ [AWS KMS API Permissions Reference](kms-api-permissions-reference.md)

+ [Using Policy Conditions](policy-conditions.md)

+ [Using Grants](grants.md)