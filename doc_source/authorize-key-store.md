# Controlling Access to Your Custom Key Store<a name="authorize-key-store"></a>

You use IAM policies to control access to your AWS KMS custom key store and your AWS CloudHSM cluster\. You can use IAM policies and key policies to control access to the customer master keys \(CMKs\) in your custom key store\. We recommend that you provide users, groups, and roles only the permissions that they require for the tasks that they are likely to perform\.

**Topics**
+ [Authorizing Custom Key Store Managers and Users](#authorize-users)
+ [Authorizing AWS KMS to Manage AWS CloudHSM and Amazon EC2 Resources](#authorize-kms)

## Authorizing Custom Key Store Managers and Users<a name="authorize-users"></a>

When designing your custom key store, be sure that the principals who use and manage it have only the permissions that they require\. The following list describes the minimum permissions required for custom key store managers and users\.
+ Principals who create and manage your custom key store require the following permission to use the custom key store API operations\.
  + `cloudhsm:DescribeClusters`
  + `kms:CreateCustomKeyStore`
  + `kms:ConnectCustomKeyStore`
  + `kms:DisconnectCustomKeyStore`
  + `kms:UpdateCustomKeyStore`
  + `kms:DeleteCustomKeyStore`
  + `kms:DescribeCustomKeyStores`
  + `iam:CreateServiceLinkedRole`

   
+ Principals who create and manage the AWS CloudHSM cluster that is associated with your custom key store need permission to create and initialize an AWS CloudHSM cluster\. This includes permission to create or use a virtual private cloud, create subnets, and create an Amazon EC2 instance\. They might also need to create and delete HSMs, and manage backups\. For lists of the required permissions, see [Restrict User Permissions to What's Necessary for AWS CloudHSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-iam-user.html#permissions-for-cloudhsm) in the *AWS CloudHSM User Guide*\.

   
+ Principals who create and manage customer master keys \(CMKs\) in your custom key store require the same permissions as those who create and manage any CMK in AWS KMS\. For example, those principals need an IAM policy with [kms:CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) permission\. No additional permissions are required\. The [default key policy](key-policies.md#key-policy-default) for CMKs in a custom key store is identical to the default key policy for CMKs in AWS KMS\.

   
+ Principals who use the CMKs in your custom key store for cryptographic operations need permission to perform the cryptographic operation with the CMK, such as [kms:Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)\. You can provide these permissions in an IAM or key policy\. But, they do not need any additional permissions to use a CMK in a custom key store\.

## Authorizing AWS KMS to Manage AWS CloudHSM and Amazon EC2 Resources<a name="authorize-kms"></a>

To support your custom key stores, AWS KMS needs permission to get information about your AWS CloudHSM clusters\. It also needs permission to create the network infrastructure that connects your custom key store to its AWS CloudHSM cluster\. To get these permissions, AWS KMS creates the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role in your AWS account\. Users who create custom key stores must have permission to create service\-linked roles\.

A [service\-linked role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) is an IAM role that gives one AWS service permission to call other AWS services on your behalf\. It's designed to make it easier for you to use the features of multiple integrated AWS services without having to create and maintain complex IAM policies\.

For custom key stores, AWS KMS creates the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role with the **AWSKeyManagementServiceCustomKeyStoresServiceRolePolicy** policy\. This policy grants the role the following permissions:
+ [cloudhsm:DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html)
+ [ec2:AuthorizeSecurityGroupIngress](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_AuthorizeSecurityGroupIngress.html)
+ [ec2:CreateNetworkInterface](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateNetworkInterface.html)
+ [ec2:CreateSecurityGroup](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateSecurityGroup.html)
+ [ec2:DeleteSecurityGroup](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DeleteSecurityGroup.html)
+ [ec2:DescribeSecurityGroups](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeSecurityGroups.html)
+ [ec2:RevokeSecurityGroupEgress](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_RevokeSecurityGroupEgress.html)

Only AWS KMS can assume this service\-linked role\. It is limited to the operations that AWS KMS needs in order to view your clusters and connect a custom key store to its AWS CloudHSM cluster\. It does not give AWS KMS any additional permissions\. For example, AWS KMS does not have permission to create, manage, or delete your AWS CloudHSM clusters, HSMs, or backups\.

Like the custom key stores feature, the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** role is supported in all AWS Regions where both AWS KMS and AWS CloudHSM are available, except for AWS GovCloud \(US\-West\)\. For a list of AWSRegions that each service supports, see [AWS Key Management Service](https://docs.aws.amazon.com/general/latest/gr/rande.html#kms_region) and [AWS CloudHSM](https://docs.aws.amazon.com/general/latest/gr/rande.html#cloudhsm_region)\.

**Create the Service\-Linked Role**

AWS KMS automatically creates the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role in your AWS account when you create a custom key store, if the role does not already exist\. You cannot create this service\-linked role directly\.

**Edit the Service\-Linked Role Description**

You cannot edit the role name or the policy statements in the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role, but you can edit role description\. For instructions, see [Editing a Service\-Linked Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#edit-service-linked-role) in the *IAM User Guide*\.

**Delete the Service\-Linked Role**

AWS KMS does not delete the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role\. 

If you have [deleted all of your custom key stores](delete-keystore.md) and do not plan to create any new ones, we recommend that you delete the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** role\. AWS KMS prevents you from deleting this service\-linked role if you have any custom key stores in any Region of your AWS account\. 

To delete the service\-linked role:

1. [Delete all AWS KMS custom key stores](delete-keystore.md) from all Regions of your AWS account\.

1. Use the IAM console, the IAM CLI, or the IAM API to delete the service\-linked role\. For instructions, see [Deleting a Service\-Linked Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#delete-service-linked-role) in the *IAM User Guide*\.
**Note**  
If AWS KMS is using the role when you try to delete it, the deletion might fail\. If that happens, wait for a few minutes and try the operation again\.

For more information about how AWS services use service\-linked roles, see [Using Service\-Linked Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) in the IAM User Guide\.