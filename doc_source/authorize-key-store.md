# Controlling access to your custom key store<a name="authorize-key-store"></a>

You use IAM policies to control access to your AWS KMS custom key store and your AWS CloudHSM cluster\. You can use IAM policies and key policies to control access to the customer master keys \(CMKs\) in your custom key store\. We recommend that you provide users, groups, and roles only the permissions that they require for the tasks that they are likely to perform\.

**Topics**
+ [Authorizing custom key store managers and users](#authorize-users)
+ [Authorizing AWS KMS to manage AWS CloudHSM and Amazon EC2 resources](#authorize-kms)

## Authorizing custom key store managers and users<a name="authorize-users"></a>

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
+ Principals who use the CMKs in your custom key store for [cryptographic operations](use-cmk-keystore.md) need permission to perform the cryptographic operation with the CMK, such as [kms:Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)\. You can provide these permissions in an IAM or key policy\. But, they do not need any additional permissions to use a CMK in a custom key store\.

## Authorizing AWS KMS to manage AWS CloudHSM and Amazon EC2 resources<a name="authorize-kms"></a>

To support your custom key stores, AWS KMS needs permission to get information about your AWS CloudHSM clusters\. It also needs permission to create the network infrastructure that connects your custom key store to its AWS CloudHSM cluster\. To get these permissions, AWS KMS creates the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role in your AWS account\. Users who create custom key stores must have the `iam:CreateServiceLinkedRole` permission that allows them to create service\-linked roles\.

**Topics**
+ [About the AWS KMS service\-linked role](#about-key-store-slr)
+ [Create the service\-linked role](#create-key-store-slr)
+ [Edit the service\-linked role description](#edit-key-store-slr)
+ [Delete the service\-linked role](#delete-key-store-slr)

### About the AWS KMS service\-linked role<a name="about-key-store-slr"></a>

A [service\-linked role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) is an IAM role that gives one AWS service permission to call other AWS services on your behalf\. It's designed to make it easier for you to use the features of multiple integrated AWS services without having to create and maintain complex IAM policies\.

For custom key stores, AWS KMS creates the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role with the **AWSKeyManagementServiceCustomKeyStoresServiceRolePolicy** policy\. This policy grants the role the following permissions:
+ [cloudhsm:DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html)
+ [ec2:AuthorizeSecurityGroupIngress](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_AuthorizeSecurityGroupIngress.html)
+ [ec2:CreateNetworkInterface](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateNetworkInterface.html)
+ [ec2:CreateSecurityGroup](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateSecurityGroup.html)
+ [ec2:DeleteSecurityGroup](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DeleteSecurityGroup.html)
+ [ec2:DescribeSecurityGroups](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeSecurityGroups.html)
+ [ec2:RevokeSecurityGroupEgress](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_RevokeSecurityGroupEgress.html)

Because the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role trusts only `cks.kms.amazonaws.com`, only AWS KMS can assume this service\-linked role\. This role is limited to the operations that AWS KMS needs to view your AWS CloudHSM clusters and to connect a custom key store to its associated AWS CloudHSM cluster\. It does not give AWS KMS any additional permissions\. For example, AWS KMS does not have permission to create, manage, or delete your AWS CloudHSM clusters, HSMs, or backups\.

**Regions**

Like the custom key stores feature, the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** role is supported in all AWS Regions where AWS KMS and AWS CloudHSM are available\. For a list of AWS Regions that each service supports, see [AWS Key Management Service Endpoints and Quotas](https://docs.aws.amazon.com/general/latest/gr/kms.html) and [AWS CloudHSM Endpoints and Quotas](https://docs.aws.amazon.com/general/latest/gr/cloudhsm.html) in the *Amazon Web Services General Reference*\.

For more information about how AWS services use service\-linked roles, see [Using Service\-Linked Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) in the IAM User Guide\.

### Create the service\-linked role<a name="create-key-store-slr"></a>

AWS KMS automatically creates the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role in your AWS account when you create a custom key store, if the role does not already exist\. You cannot create or re\-create this service\-linked role directly\. 

### Edit the service\-linked role description<a name="edit-key-store-slr"></a>

You cannot edit the role name or the policy statements in the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role, but you can edit role description\. For instructions, see [Editing a Service\-Linked Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#edit-service-linked-role) in the *IAM User Guide*\.

### Delete the service\-linked role<a name="delete-key-store-slr"></a>

AWS KMS does not delete the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role from your AWS account\.  If you have [deleted all of your custom key stores](delete-keystore.md) and do not plan to create any new ones, you no longer need this service\-linked role\. AWS KMS does not assume this role or use its permissions unless you have active custom key stores\. However, there is currently no procedure for deleting the **AWSServiceRoleForKeyManagementServiceCustomKeyStores** service\-linked role\.