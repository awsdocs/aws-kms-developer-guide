# Controlling access to your AWS CloudHSM key store<a name="authorize-key-store"></a>

You use IAM policies to control access to your AWS CloudHSM key store and your AWS CloudHSM cluster\. You can use key policies, IAM policies, and grants to control access to the AWS KMS keys in your AWS CloudHSM key store\. We recommend that you provide users, groups, and roles only the permissions that they require for the tasks that they are likely to perform\.

**Topics**
+ [Authorizing AWS CloudHSM key store managers and users](#authorize-users)
+ [Authorizing AWS KMS to manage AWS CloudHSM and Amazon EC2 resources](#authorize-kms)

## Authorizing AWS CloudHSM key store managers and users<a name="authorize-users"></a>

When designing your AWS CloudHSM key store, be sure that the principals who use and manage it have only the permissions that they require\. The following list describes the minimum permissions required for AWS CloudHSM key store managers and users\.
+ Principals who create and manage your AWS CloudHSM key store require the following permission to use the AWS CloudHSM key store API operations\.
  + `cloudhsm:DescribeClusters`
  + `kms:CreateCustomKeyStore`
  + `kms:ConnectCustomKeyStore`
  + `kms:DeleteCustomKeyStore`
  + `kms:DescribeCustomKeyStores`
  + `kms:DisconnectCustomKeyStore`
  + `kms:UpdateCustomKeyStore`
  + `iam:CreateServiceLinkedRole`
+ Principals who create and manage the AWS CloudHSM cluster that is associated with your AWS CloudHSM key store need permission to create and initialize an AWS CloudHSM cluster\. This includes permission to create or use an Amazon Virtual Private Cloud \(VPC\), create subnets, and create an Amazon EC2 instance\. They might also need to create and delete HSMs, and manage backups\. For lists of the required permissions, see [Identity and access management for AWS CloudHSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/identity-access-management.html) in the *AWS CloudHSM User Guide*\.
+ Principals who create and manage AWS KMS keys in your AWS CloudHSM key store require [the same permissions](create-keys.md#create-key-permissions) as those who create and manage any KMS key in AWS KMS\. The [default key policy](key-policy-default.md) for a KMS key in an AWS CloudHSM key store is identical to the default key policy for KMS keys in AWS KMS\. [Attribute\-based access control](abac.md) \(ABAC\), which uses tags and aliases to control access to KMS keys, is also effective on KMS keys in AWS CloudHSM key stores\.
+ Principals who use the KMS keys in your AWS CloudHSM key store for [cryptographic operations](use-cmk-keystore.md) need permission to perform the cryptographic operation with the KMS key, such as [kms:Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)\. You can provide these permissions in a key policy, IAM policy\. But, they do not need any additional permissions to use a KMS key in an AWS CloudHSM key store\.

## Authorizing AWS KMS to manage AWS CloudHSM and Amazon EC2 resources<a name="authorize-kms"></a>

To support your AWS CloudHSM key stores, AWS KMS needs permission to get information about your AWS CloudHSM clusters\. It also needs permission to create the network infrastructure that connects your AWS CloudHSM key store to its AWS CloudHSM cluster\. To get these permissions, AWS KMS creates the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role in your AWS account\. Users who create AWS CloudHSM key stores must have the `iam:CreateServiceLinkedRole` permission that allows them to create service\-linked roles\.

**Topics**
+ [About the AWS KMS service\-linked role](#about-key-store-slr)
+ [Create the service\-linked role](#create-key-store-slr)
+ [Edit the service\-linked role description](#edit-key-store-slr)
+ [Delete the service\-linked role](#delete-key-store-slr)

### About the AWS KMS service\-linked role<a name="about-key-store-slr"></a>

A [service\-linked role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) is an IAM role that gives one AWS service permission to call other AWS services on your behalf\. It's designed to make it easier for you to use the features of multiple integrated AWS services without having to create and maintain complex IAM policies\.

For AWS CloudHSM key stores, AWS KMS creates the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role with the **AWSKeyManagementServiceCustomKeyStoresServiceRolePolicy** policy\. This policy grants the role the following permissions:
+ [cloudhsm:DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html)
+ [ec2:AuthorizeSecurityGroupIngress](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_AuthorizeSecurityGroupIngress.html)
+ [ec2:CreateNetworkInterface](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateNetworkInterface.html)
+ [ec2:CreateSecurityGroup](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateSecurityGroup.html)
+ [ec2:DeleteSecurityGroup](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DeleteSecurityGroup.html)
+ [ec2:DescribeSecurityGroups](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeSecurityGroups.html)
+ [ec2:RevokeSecurityGroupEgress](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_RevokeSecurityGroupEgress.html)

Because the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role trusts only `cks.kms.amazonaws.com`, only AWS KMS can assume this service\-linked role\. This role is limited to the operations that AWS KMS needs to view your AWS CloudHSM clusters and to connect an AWS CloudHSM key store to its associated AWS CloudHSM cluster\. It does not give AWS KMS any additional permissions\. For example, AWS KMS does not have permission to create, manage, or delete your AWS CloudHSM clusters, HSMs, or backups\.

**Regions**

Like the AWS CloudHSM key stores feature, the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** role is supported in all AWS Regions where AWS KMS and AWS CloudHSM are available\. For a list of AWS Regions that each service supports, see [AWS Key Management Service Endpoints and Quotas](https://docs.aws.amazon.com/general/latest/gr/kms.html) and [AWS CloudHSM endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/cloudhsm.html) in the *Amazon Web Services General Reference*\.

For more information about how AWS services use service\-linked roles, see [Using service\-linked roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) in the IAM User Guide\.

### Create the service\-linked role<a name="create-key-store-slr"></a>

AWS KMS automatically creates the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role in your AWS account when you create an AWS CloudHSM key store, if the role does not already exist\. You cannot create or re\-create this service\-linked role directly\. 

### Edit the service\-linked role description<a name="edit-key-store-slr"></a>

You cannot edit the role name or the policy statements in the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role, but you can edit role description\. For instructions, see [Editing a service\-linked role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#edit-service-linked-role) in the *IAM User Guide*\.

### Delete the service\-linked role<a name="delete-key-store-slr"></a>

AWS KMS does not delete the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role from your AWS account even if you have [deleted all of your AWS CloudHSM key stores](delete-keystore.md)\. Although there is currently no procedure for deleting the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role, AWS KMS does not assume this role or use its permissions unless you have active AWS CloudHSM key stores\.