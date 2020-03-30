# Using service\-linked roles for AWS KMS<a name="using-service-linked-roles"></a>

AWS Key Management Service uses AWS Identity and Access Management \(IAM\)[ service\-linked roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-service-linked-role)\. A service\-linked role is a unique type of IAM role that is linked directly to AWS KMS\. Service\-linked roles are defined by AWS KMS and include all the permissions that the service requires to call other AWS services on your behalf\. 

A service\-linked role makes setting up AWS KMS easier because you don’t have to manually add the necessary permissions\. AWS KMS defines the permissions of its service\-linked roles, and unless defined otherwise, only AWS KMS can assume its roles\. The defined permissions include the trust policy and the permissions policy, and that permissions policy cannot be attached to any other IAM entity\.

You can delete a service\-linked role only after first deleting the related resources\. This protects your AWS KMS resources because you can't inadvertently remove permission to access the resources\.

For information about other services that support service\-linked roles, see [AWS Services That Work with IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html) and look for the services that have **Yes **in the **Service\-Linked Role** column\. Choose a **Yes** with a link to view the service\-linked role documentation for that service\.

## Service\-linked role permissions for AWS KMS custom key stores<a name="slr-permissions"></a>

AWS KMS uses a service\-linked role named **AWSServiceRoleForKeyManagementServiceCustomKeyStores** to support [custom key stores](custom-key-store-overview.md)\. This service\-linked role gives AWS KMS permission to view your AWS CloudHSM clusters and create the network infrastructure to support a connection between your custom key store and its AWS CloudHSM cluster\. AWS KMS creates this role only when you create a [custom key store](custom-key-store-overview.md)\. You cannot create this service\-linked role directly\.

The **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role trusts `cks.kms.amazonaws.com` to assume the role\. As a result, only AWS KMS can assume this service\-linked role\. 

The permissions in the role are limited to the actions that AWS KMS performs to connect a custom key store to an AWS CloudHSM cluster\. It does not give AWS KMS any additional permissions\. For example, AWS KMS does not have permission to create, manage, or delete your AWS CloudHSM clusters, HSMs, or backups\.

For more information about the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** role, including a list of permissions and instructions for how to view the role, edit the role description, delete the role, and have AWS KMS recreate it for you, see [Authorizing AWS KMS to manage AWS CloudHSM and Amazon EC2 resources](authorize-key-store.md#authorize-kms)\.