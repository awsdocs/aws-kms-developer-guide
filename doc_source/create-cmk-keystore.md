# Creating CMKs in a Custom Key Store<a name="create-cmk-keystore"></a>

After you have created a custom key store, you can create [customer master keys](concepts.md#master_keys) \(CMKs\) in your key store\. Then you can use and manage these CMKs very much like you would use any CMK in AWS KMS\. For example, you can do any of the following:
+ Create aliases that point to the CMKs\. 
+ Set IAM and key policies on the CMKs\.
+ Enable and disable the CMKs\. 
+ Schedule deletions of the CMKs\. 
+ Use the CMKs for cryptographic operations\.

To create a CMK in a custom key store, the custom key store must be [connected to its associated AWS CloudHSM cluster](disconnect-keystore.md) and the cluster must contain at least two active HSMs in different Availability Zones\. To find the connection status and number of HSMs, view the [custom key stores page](view-keystore.md#view-keystore-console) in the AWS Management Console\. When using the API operations, use the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation to verify that the custom key store is connected\. Use the AWS CloudHSM [DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html) operation to get the number of active HSMs in the cluster and their Availability Zones\. 

When you create a CMK in your custom key store, AWS KMS creates the CMK in AWS KMS\. But, it creates the key material for the CMK in the associated AWS CloudHSM cluster\. Specifically, AWS KMS signs into the cluster as the [`kmsuser` CU that you created](create-keystore.md#before-keystore)\. Then it creates a persistent, non\-extractable, 256\-bit Advanced Encryption Standard \(AES\) symmetric key in the cluster\. AWS KMS sets the value of the [key label attribute](https://docs.aws.amazon.com/cloudhsm/latest/userguide/key-attribute-table.html), which is visible only in the cluster, to Amazon Resource Name \(ARN\) of the CMK\.

When the command succeeds, the [key state](key-state.md) of the new CMK is `Enabled` and its origin is `AWS_CLOUDHSM`\. You cannot change the origin of any CMK after you create it\. When you view a CMK in a custom key store in the console or by using the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) API operation, you can see typical properties, like its key ID, key state, and creation date\. But you can also see the custom key store ID and \(optionally\) the AWS CloudHSM cluster ID\. For details, see [Viewing CMKs in a Custom Key Store](view-cmk-keystore.md)\.

If your attempt to create a CMK in your custom key store fails, use the error message to help you determine the cause\. It might indicate that the custom key store is not connected \(`CustomKeyStoreInvalidStateException`\) or the associated AWS CloudHSM cluster doesn't have the two active HSMs that are required for this operation \(`CloudHsmClusterInvalidConfigurationException`\)\. For help see [Troubleshooting a Custom Key Store](fix-keystore.md)\.

**Topics**
+ [Create a CMK in a Custom Key Store \(Console\)](#create-cmk-keystore-console)
+ [Create a CMK in a Custom Key Store \(API\)](#create-cmk-keystore-api)

## Create a CMK in a Custom Key Store \(Console\)<a name="create-cmk-keystore-console"></a>

Use the following procedure to create a customer master key \(CMK\) in a custom key store\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Choose **Create key**\.

1. Type an alias and an optional description for the CMK\.

1. Choose **Advanced options**\.

1. For **Key material origin**, choose **Custom key store \(CloudHSM\)**\.

1. Choose **Next**\.

1. Select a custom key store for your new CMK\. To create a new custom key store, choose **Create custom key store**\.

   The custom key store that you select must have a status of **CONNECTED**\. Its associated AWS CloudHSM cluster must be active and contain at least two active HSMs in different Availability Zones\. 

   For help with connecting a custom key store, see [Connecting and Disconnecting a Custom Key Store](disconnect-keystore.md)\. For help with adding HSMs, see [Adding an HSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/add-remove-hsm.html#add-hsm) in the *AWS CloudHSM User Guide*\.

1. Choose **Next**\.

1. \(Optional\)\. On the **Add Tags** page, add tags that identify or categorize your CMK\.

   When you add tags to your AWS resources, AWS generates a cost allocation report with usage and costs aggregated by tags\. For information about tagging CMKs, see [Tagging Keys](tagging-keys.md)\.

1. Choose **Next**\.

1. In the **Key Administrators** section, select the IAM users and roles who can manage the CMK\. For more information, see [Allows Key Administrators to Administer the CMK](key-policies.md#key-policy-default-allow-administrators)\.
**Note**  
IAM policies can give other IAM users and roles permission to use the CMK\.

1. \(Optional\) To prevent these key administrators from deleting this CMK, clear the box at the bottom of the page for **Allow key administrators to delete this key\.**

1. Choose **Next**\.

1. In the **This account** section, select the IAM users and roles in this AWS account who can use the CMK in cryptographic operations\. For more information, see [Allows Key Users to Use the CMK](key-policies.md#key-policy-default-allow-users)\.
**Note**  
IAM policies can give other IAM users and roles permission to use the CMK\.

1. \(Optional\) You can allow other AWS accounts to use this CMK for cryptographic operations\. To do so, in the Other AWS accounts section at the bottom of the page, choose Add another AWS account and enter the AWS account identification number of an external account\. To add multiple external accounts, repeat this step\.
**Note**  
Administrators of the other AWS accounts must also allow access to the CMK by creating IAM policies for their users\. For more information, see [ Allowing Users in Other Accounts to Use a CMKAllowing Cross\-Account Access to a CMK  You can allow IAM users or roles in one AWS account to use a customer master key \(CMK\) in a different AWS account\. You can add these permissions when you create the CMK or change the permissions for an existing CMK\. To give permission to users and roles in another account, you must use two different types of policies:   The **key policy** for the CMK must give the external account \(or users and roles in the external account\) permission to use the CMK\. The key policy is in the account that owns the CMK\.   You must attach **IAM policies** to IAM users and roles in the external account\. These IAM policies delegate the permissions that are specified in the key policy\.   In this scenario, the key policy determines who *can* have access to the CMK\. The IAM policy determines who *does* have access to the CMK\. Neither the key policy nor the IAM policy alone is sufficient—you must change both\.  To edit the key policy, you can use the [Policy View](key-policy-modifying.md#key-policy-modifying-how-to-console-policy-view) in the AWS Management Console or use the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) or [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operations\. For help setting the key policy when creating a CMK, see [Creating CMKs that Other Accounts Can Use](#cross-account-console)\. For help with editing IAM policies, see [Using IAM Policies with AWS KMS](iam-policies.md)\.  For an example that shows how the key policy and IAM policies work together to allow use of a CMK in a different account, see [ Example 2: User Assumes Role with Permission to Use a CMK in a Different AWS Account   Bob is a user in account 1 \(111122223333\)\. He is allowed to use a CMK in account 2 \(444455556666\) in cryptographic operations\. How is this possible?  When evaluating cross\-account permissions, remember that the key policy is specified in the CMK's account\. The IAM policy is specified in the caller's account, even when the caller is in a different account\.    The key policy for the CMK in account 2 allows account 2 to use IAM policies to control access to the CMK\.    The key policy for the CMK in account 2 allows account 1 to use the CMK in cryptographic operations\. However, account 1 must use IAM policies to give its principals access to the CMK\.   An IAM policy in account 1 allows the `ExampleRole` role to use the CMK in account 2 for cryptographic operations\.   Bob, a user in account 1, has permission to assume the `ExampleRole` role\.   Bob can trust this CMK, because even though it is not in his account, an IAM policy in his account gives him explicit permission to use this CMK\.   

![\[Flowchart that describes the policy evaluation process\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/kms-auth-flow-Bob.png) Consider the policies that let Bob, a user in account 1, use the CMK in account 2\.   The key policy for the CMK allows account 2 \(444455556666, the account that owns the CMK\) to use IAM policies to control access to the CMK\. This key policy also allows account 1 \(111122223333\) to use the CMK in cryptographic operations \(specified in the `Action` element of the policy statement\)\. However, no one in account 1 can use the CMK in account 2 until account 1 defines IAM policies that give the principals access to the CMK\. In the flowchart, this key policy in account 2 satisfies the *Does the key policy ALLOW the caller's account to use IAM policies to control access to the key?* condition\.  

  ```
  {
      "Id": "key-policy-acct-2",
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "Permission to use IAM policies",
              "Effect": "Allow",
              "Principal": {
                  "AWS": "arn:aws:iam::444455556666:root"
              },
              "Action": "kms:*",
              "Resource": "*"
          },
          {
              "Sid": "Allow account 1 to use this CMK",
              "Effect": "Allow",
              "Principal": {
                  "AWS": "arn:aws:iam::111122223333:root"
              },
              "Action": [
                  "kms:Encrypt",
                  "kms:Decrypt",
                  "kms:ReEncryptFrom",
                  "kms:ReEncryptTo",
                  "kms:GenerateDataKey",
                  "kms:GenerateDataKeyWithoutPlaintext",
                  "kms:DescribeKey"
              ],
              "Resource": "*"
          }
      ]
  }
  ```   An IAM policy in the caller's AWS account \(account 1, 111122223333\) gives the `ExampleRole` role in account 1 permission to perform cryptographic operations using the CMK in account 2 \(444455556666\)\. The `Action` element gives the role the same permissions that the key policy in account 2 gave to account 1\. Cross\-account IAM policies like this one are effective only when the key policy for the CMK in account 2 gives account 1 permission to use the CMK\. Also, account 1 can only give its principals permission to perform the actions that the key policy gave to the account\. In the flowchart, this satisfies the *Does an IAM policy allow the caller to perform this action?* condition\. 

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Principal": { "arn:aws:iam::111122223333:role/ExampleRole" }
              "Effect": "Allow",
              "Action": [
                  "kms:Encrypt",
                  "kms:Decrypt",
                  "kms:ReEncryptFrom",
                  "kms:ReEncryptTo",
                  "kms:GenerateDataKey",
                  "kms:GenerateDataKeyWithoutPlaintext",
                  "kms:DescribeKey"
              ],
              "Resource": [
                  "arn:aws:kms:us-west-2:444455556666:key/1234abcd-12ab-34cd-56ef-1234567890ab"
              ]
          }
      ]
  }
  ```   The last required element is the definition of the `ExampleRole` role in account 1\. The `AssumeRolePolicyDocument` in the role allows Bob to assume the `ExampleRole` role\. 

  ```
  {
      "Role": {
          "Arn": "arn:aws:iam::111122223333:role/ExampleRole",
          "CreateDate": "2019-05-16T00:09:25Z",
          "AssumeRolePolicyDocument": {
              "Version": "2012-10-17",
              "Statement": {
                  "Principal": {
                      "AWS": "arn:aws:iam::111122223333:user/bob"
                  },
                  "Effect": "Allow",
                  "Action": "sts:AssumeRole"
              }
          },
          "Path": "/",
          "RoleName": "ExampleRole",
          "RoleId": "AROA4KJY2TU23Y7NK62MV"
      }
  }
  ```    ](policy-evaluation.md#example-cross-acct)\.   Step 1: Add a Key Policy Statement in the Local Account  The key policy for a CMK is the primary determinant of who can access the CMK and which operations they can perform\. The key policy is always in the account that owns the CMK\. Unlike IAM policies, key policies do specify a resource\. The resource is the CMK that is associated with the key policy\. To give an external account permission to use the CMK, add a statement to the key policy that specifies the external account\. In the `Principal` element of the key policy, enter the Amazon Resource Name \(ARN\) of the external account\. When you specify an external account in a key policy, IAM administrators in the external account can use IAM policies to delegate those permissions to any users and roles in the external account\. They can also decide which of the actions specified in the key policy the users and roles can perform\.  For example, assume that you want to allow account 444455556666 to use a CMK in account 111122223333\. To do that, add a policy statement like the one in the following example to the key policy for the CMK in account 111122223333\. This policy statement gives the external account, 444455556666, permission to use the CMK in cryptographic operations\.  

```
{
    "Sid": "Allow an external account to use this CMK",
    "Effect": "Allow",
    "Principal": {
        "AWS": [
            "arn:aws:iam::444455556666:root"
        ]
    },
    "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
    ],
    "Resource": "*"
}
``` If you prefer, you can specify particular external users and roles in the key policy instead of giving permission to the external account\. However, those users and roles cannot use the CMK until IAM administrators in the external account attach the proper IAM policies to their identities\. The IAM policies can give permission to all or a subset of the external users and roles that are specified in the key policy\. And they can allow all or a subset of the actions specified in the key policy\.  Specifying identities in a key policy restricts the permissions that IAM administrators in the external account can provide\. However, it makes policy management with two accounts more complex\. For example, assume that you need to add a user or role\. You must add that identity to the key policy in the account that owns the CMK and create IAM policies in the identity's account\. To specify particular external users or roles in a key policy, in the `Principal` element, enter the Amazon Resource Name \(ARN\) of a user or role in the external account\. For example, the following example key policy statement allows `ExampleRole` and ExampleUser in account 444455556666 to use a CMK in account 111122223333\. This key policy statement gives the external account, 444455556666, permission to use the CMK in cryptographic operations\.  

```
{
    "Sid": "Allow an external account to use this CMK",
    "Effect": "Allow",
    "Principal": {
        "AWS": [
            "arn:aws:iam::444455556666:role/ExampleRole"
            "arn:aws:iam::444455556666:user/ExampleUser"
        ]
    },
    "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
    ],
    "Resource": "*"
}
``` You also need to decide which permissions you want to give to the external account\. For a list of permissions on CMKs, see [AWS KMS API Permissions: Actions and Resources Reference](kms-api-permissions-reference.md)\. You can give the external account permission to use the CMK in cryptographic operations and use the CMK with AWS services that are integrated with AWS KMS\. To do that, use the **Key Users** section of the AWS Management Console\. For details, see [Creating CMKs that Other Accounts Can Use](#cross-account-console)\. To specify other permissions in key policies, edit the key policy document\. For example, you might want to give users permission to decrypt but not encrypt, or permission to view the CMK but not use it\. To edit the key policy document, you can use the [Policy View](key-policy-modifying.md#key-policy-modifying-how-to-console-policy-view) in the AWS Management Console or the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) or [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operations\.   Step 2: Add IAM Policies in the External Account  The key policy in the account that owns the CMK sets the valid range for permissions\. But, users and roles in the external account cannot use the CMK until you attach IAM policies that delegate those permissions, or use grants to manage access to the CMK\. The IAM policies are set in the external account\.  If the key policy gives permission to the external account, you can attach IAM policies to any user or role in the account\. But if the key policy gives permission to specified users or roles, the IAM policy can only give those permissions to all or a subset of the specified users and roles\. If an IAM policy gives CMK access to other external users or roles, it has no effect\. The key policy also limits the actions in the IAM policy\. The IAM policy can delegate all or a subset of the actions specified in the key policy\. If the IAM policy lists actions that are not specified in the key policy, those permissions are not effective\. The following example IAM policy allows the principal to use the CMK in account 111122223333 for cryptographic operations\. To give this permission to users and roles in account 444455556666, [attach the policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-using.html#attach-managed-policy-console) to the users or roles in account 444455556666\. 

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Allow Use Of CMK In Account 111122223333",
      "Effect": "Allow",
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
      ],
      "Resource": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
    }
  ]
}
``` Note the following details about this policy:   Unlike key policies, IAM policy statements do not contain the `Principal` element\. In IAM policies, the principal is the identity to which the policy is attached\.    The `Resource` element in the IAM policy identifies the CMK that the principal can use\. To specify a CMK, add its [Amazon Resource Name \(ARN\)](control-access-overview.md#kms-resources-operations) to the `Resource` element\. You can specify more than one CMK in the policy statement\. But if you don't specify particular CMKs in the `Resource` element, you might inadvertently give access to more CMKs than you intend\.   To allow the external user to use the CMK with [AWS services that integrate with AWS KMS,](https://aws.amazon.com//kms/features/#AWS_Service_Integration) you might need to add permissions to the key policy or the IAM policy\. For details, see [Using External CMKs with AWS Services](#cross-account-service)\.   For more information about working with IAM policies, see [Using IAM Policies](iam-policies.md)\.   Creating CMKs that Other Accounts Can Use  When you use the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation create a CMK, you can use its `Policy` parameter to specify a [key policy](#cross-account-key-policy) that gives an external account, or external users and roles, permission to use the CMK\. You must also add [IAM policies](#cross-account-iam-policy) in the external account that delegate these permissions to the account's users and roles, even when users and roles are specified in the key policy\. You can change the key policy at any time by using the [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operation\. When you create a CMK in the AWS Management Console, you also create its key policy\. When you select identities in the **Key Administrators** and **Key Users** sections, AWS KMS adds policy statements for those identities to the CMK's key policy\.  The **Key Users** section also lets you add external accounts as key users\. 

![\[The console element that adds external accounts to the key policy for a CMK.\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-external-accounts-sm.png) When you enter the account ID of an external account, AWS KMS adds two statements to the key policy\. This action only affects the key policy\. Users and roles in the external account cannot use the CMK until you attach [IAM policies](#cross-account-iam-policy) to give them some or all of these permissions\. The first policy statement gives the external account permission to use the CMK in cryptographic operations\.  

```
{
    "Sid": "Allow use of the key",
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::444455556666:root"
    },
    "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
    ],
    "Resource": "*"
}
``` The second policy statement allows the external account to create, view, and revoke grants on the CMK, but only when the request comes from an [AWS service that is integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration)\. These permissions allow other AWS services, such as that encrypt user data to use the CMK\.  These permissions are designed for CMKs that encrypt user data in AWS services, such as [Amazon WorkMail](services-wm.md)\. These services typically use grants to get the permissions they need to use the CMK on the user's behalf\. For details, see [Using External CMKs with AWS Services](#cross-account-service)\. 

```
{
    "Sid": "Allow attachment of persistent resources",
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::444455556666:root"
    },
    "Action": [
        "kms:CreateGrant",
        "kms:ListGrants",
        "kms:RevokeGrant"
    ],
    "Resource": "*",
    "Condition": {
        "Bool": {
            "kms:GrantIsForAWSResource": "true"
        }
    }
}
``` If these permissions don't meet your needs, you can edit them in the console [policy view](key-policy-modifying.md#key-policy-modifying-how-to-console-policy-view) or by using the [PutKeyPolicy]() operation\. You can specify particular external users and role instead of giving permission to the external account\. You can change the actions that the policy specifies\. And you can use global and AWS KMS policy conditions to refine the permissions\.   Using External CMKs with AWS Services  You can give a user in a different account permission to use your CMK with a service that is integrated with AWS KMS\. For example, a user in an external account can use your CMK to [encrypt the objects in an Amazon S3 bucket](services-s3.md) or to [encrypt the secrets they store in AWS Secrets Manager](services-secrets-manager.md)\. The key policy must give the external user or the external user's account permission to use the CMK\. In addition, you need to attach IAM policies to the identity that gives the user permission to use the AWS service\.  Also, the service might require that users have additional permissions in the key policy\. For example, it might require permission to create, list, and revoke grants on the CMK\. Or it might require particular IAM policies\. For details, see the documentation for the service\.  Finally, the lists of CMKs displayed in the AWS Management Console for an integrated service do not include CMKs in external accounts\. This is true even when the user or role has permission to use them\. To use an external account's CMK, the user must enter the ID or ARN of the CMK\. For details, see the service's console documentation\.  ](key-policy-modifying-external-accounts.md)\.

1. Choose **Next**\.

1. On the **Review and edit key policy** page, review and edit the policy document for the new CMK\. When you're done, choose **Finish**\.

When the procedure succeeds, the display shows the new CMK in the custom key store that you chose\. When you choose the name or alias of the new CMK, its detail page displays the origin of the CMK \(**CloudHSM**\), the name and ID of the custom key store, and the ID of the AWS CloudHSM cluster\. If the procedure fails, an error message appears that describes the failure\.

**Tip**  
To make it easier to identify CMKs in a custom key store, on the **Customer managed keys** page, add the **Custom key store ID** column to the display\. Click the gear icon in the upper\-right and select **Custom key store ID**\.

## Create a CMK in a Custom Key Store \(API\)<a name="create-cmk-keystore-api"></a>

To create a new [customer master key](concepts.md#master_keys) \(CMK\) in your custom key store, use the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation\. Use the `CustomKeyStoreId` parameter to identify your custom key store and specify an `Origin` value of `AWS_CLOUDHSM`\. 

You might also want to use the `Policy` parameter to specify a key policy\. You can change the key policy \([PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html)\) and add optional elements, such as a [description](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) and [tags](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) at any time\.

The examples in this section use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

The following example begins with a call to the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation to verify that the custom key store is connected to its associated AWS CloudHSM cluster\. By default, this operation returns all custom keys stores in your account and Region\. To describe only a particular custom key store, use the `CustomKeyStoreId` or `CustomKeyStoreName` parameter \(but not both\)\.

Before running this command, replace the example custom key store ID with a valid ID\.

```
$ aws kms describe-custom-key stores --custom-key-store-id cks-1234567890abcdef0
{
   "CustomKeyStores": [
      "CustomKeyStoreId": "cks-1234567890abcdef0",
      "CustomKeyStoreName": "ExampleKeyStore",
      "CloudHsmClusterId": "cluster-1a23b4cdefg",
      "TrustAnchorCertificate": "<certificate string appears here>",
      "CreationDate": "1.499288695918E9",
      "ConnectionState": "CONNECTED"
   ],
}
```

The next example command uses the [DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html) operation to verify that the AWS CloudHSM cluster that is associated with the `ExampleKeyStore` \(cluster\-1a23b4cdefg\) has at least two active HSMs\. If the cluster has fewer than two HSMs, the `CreateKey` operation fails\.

```
$ aws cloudhsmv2 describe-clusters
{
    "Clusters": [
        {
            "SubnetMapping": {
               ...
            },
            "CreateTimestamp": 1507133412.351,
            "ClusterId": "cluster-1a23b4cdefg",
            "SecurityGroup": "sg-865af2fb",
            "HsmType": "hsm1.medium",
            "VpcId": "vpc-1a2b3c4d",
            "BackupPolicy": "DEFAULT",
            "Certificates": {
                "ClusterCertificate": "-----BEGIN CERTIFICATE-----\...\n-----END CERTIFICATE-----\n"
            },
            "Hsms": [
                {
                    "AvailabilityZone": "us-west-2a",
                    "EniIp": "10.0.1.11",
                    "ClusterId": "cluster-1a23b4cdefg",
                    "EniId": "eni-ea8647e1",
                    "StateMessage": "HSM created.",
                    "SubnetId": "subnet-a6b10bd1",
                    "HsmId": "hsm-abcdefghijk",
                    "State": "ACTIVE"
                },
                {
                    "AvailabilityZone": "us-west-2b",
                    "EniIp": "10.0.0.2",
                    "ClusterId": "cluster-1a23b4cdefg",
                    "EniId": "eni-ea8647e1",
                    "StateMessage": "HSM created.",
                    "SubnetId": "subnet-b6b10bd2",
                    "HsmId": "hsm-zyxwvutsrqp",
                    "State": "ACTIVE"
                },
            ],
            "State": "ACTIVE"
        }
    ]
}
```

This example command uses the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation to create a CMK the custom key store\. To create a CMK in a custom key store, you must provide the ID of the custom key store name and specify an `Origin` value of `AWS_CLOUDHSM`\.

The response includes the IDs of the custom key store and the AWS CloudHSM cluster\. 

Before running this command, replace the example custom key store ID with a valid ID\.

```
$ aws kms create-key --origin AWS_CLOUDHSM --custom-key-store-id cks-1234567890abcdef0
{
  "KeyMetadata": {
    "AWSAccountId": "111122223333",
    "Arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
    "CreationDate": 1.499288695918E9,
    "Description": "Example key",
    "Enabled": true,
    "KeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
    "KeyManager": "CUSTOMER",
    "KeyState": "Enabled",
    "KeyUsage": "ENCRYPT_DECRYPT",
    "Origin": "AWS_CLOUDHSM"
    "CloudHsmClusterId": "cluster-1a23b4cdefg",
    "CustomKeyStoreId": "cks-1234567890abcdef0"
  }
}
```