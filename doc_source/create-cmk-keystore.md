# Creating CMKs in a custom key store<a name="create-cmk-keystore"></a>

After you have created a custom key store, you can create [customer master keys](concepts.md#master_keys) \(CMKs\) in your key store\. They must be [symmetric CMKs](symm-asymm-concepts.md#symmetric-cmks) with key material that AWS KMS generates\. You cannot create [asymmetric CMKs](symm-asymm-concepts.md#asymmetric-cmks) or CMKs with [imported key material](importing-keys.md), and you cannot use symmetric CMKs in a custom key store to generate asymmetric data key pairs\.

Use and manage the CMKs in your custom key store the same way that you use and manage any CMK in AWS KMS\. For example, you can do any of the following:
+ Use the CMKs for [cryptographic operations](concepts.md#cryptographic-operations)\.
+ Set IAM and key policies on the CMKs\.
+ Create aliases are associated with the CMKs\.
+ Attach tags to the CMKs\.
+ Enable and disable the CMKs\. 
+ Schedule deletions of the CMKs\.

To create a CMK in a custom key store, the custom key store must be [connected to its associated AWS CloudHSM cluster](disconnect-keystore.md) and the cluster must contain at least two active HSMs in different Availability Zones\. To find the connection status and number of HSMs, view the [custom key stores page](view-keystore.md#view-keystore-console) in the AWS Management Console\. When using the API operations, use the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation to verify that the custom key store is connected\. Use the AWS CloudHSM [DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html) operation to get the number of active HSMs in the cluster and their Availability Zones\. 

When you create a CMK in your custom key store, AWS KMS creates the CMK in AWS KMS\. But, it creates the key material for the CMK in the associated AWS CloudHSM cluster\. Specifically, AWS KMS signs into the cluster as the [`kmsuser` CU that you created](create-keystore.md#before-keystore)\. Then it creates a persistent, non\-extractable, 256\-bit Advanced Encryption Standard \(AES\) symmetric key in the cluster\. AWS KMS sets the value of the [key label attribute](https://docs.aws.amazon.com/cloudhsm/latest/userguide/key-attribute-table.html), which is visible only in the cluster, to Amazon Resource Name \(ARN\) of the CMK\.

When the command succeeds, the [key state](key-state.md) of the new CMK is `Enabled` and its origin is `AWS_CLOUDHSM`\. You cannot change the origin of any CMK after you create it\. When you view a CMK in a custom key store in the console or by using the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation, you can see typical properties, like its key ID, key state, and creation date\. But you can also see the custom key store ID and \(optionally\) the AWS CloudHSM cluster ID\. For details, see [Viewing CMKs in a custom key store](view-cmk-keystore.md)\.

If your attempt to create a CMK in your custom key store fails, use the error message to help you determine the cause\. It might indicate that the custom key store is not connected \(`CustomKeyStoreInvalidStateException`\) or the associated AWS CloudHSM cluster doesn't have the two active HSMs that are required for this operation \(`CloudHsmClusterInvalidConfigurationException`\)\. For help see [Troubleshooting a custom key store](fix-keystore.md)\.

**Topics**
+ [Create a CMK in a custom key store \(console\)](#create-cmk-keystore-console)
+ [Create a CMK in a custom key store \(API\)](#create-cmk-keystore-api)

## Create a CMK in a custom key store \(console\)<a name="create-cmk-keystore-console"></a>

Use the following procedure to create a customer master key \(CMK\) in a custom key store\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Choose **Create key**\.

1. Choose **Symmetric**\.

   You cannot create an asymmetric CMK in a custom key store\. 

1. Choose **Advanced options**\.

1. For **Key material origin**, choose **Custom key store \(CloudHSM\)**\.

1. Choose **Next**\.

1. Select a custom key store for your new CMK\. To create a new custom key store, choose **Create custom key store**\.

   The custom key store that you select must have a status of **CONNECTED**\. Its associated AWS CloudHSM cluster must be active and contain at least two active HSMs in different Availability Zones\. 

   For help with connecting a custom key store, see [Connecting and disconnecting a custom key store](disconnect-keystore.md)\. For help with adding HSMs, see [Adding an HSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/add-remove-hsm.html#add-hsm) in the *AWS CloudHSM User Guide*\.

1. Choose **Next**\.

1. Type an alias and an optional description for the CMK\.

1. \(Optional\)\. On the **Add Tags** page, add tags that identify or categorize your CMK\.

   When you add tags to your AWS resources, AWS generates a cost allocation report with usage and costs aggregated by tags\. For information about tagging CMKs, see [Tagging keys](tagging-keys.md)\.

1. Choose **Next**\.

1. In the **Key Administrators** section, select the IAM users and roles who can manage the CMK\. For more information, see [Allows Key Administrators to Administer the CMK](key-policies.md#key-policy-default-allow-administrators)\.
**Note**  
IAM policies can give other IAM users and roles permission to use the CMK\.

1. \(Optional\) To prevent these key administrators from deleting this CMK, clear the box at the bottom of the page for **Allow key administrators to delete this key\.**

1. Choose **Next**\.

1. In the **This account** section, select the IAM users and roles in this AWS account who can use the CMK in [cryptographic operations](concepts.md#cryptographic-operations)\. For more information, see [Allows Key Users to Use the CMK](key-policies.md#key-policy-default-allow-users)\.
**Note**  
IAM policies can give other IAM users and roles permission to use the CMK\.

1. \(Optional\) You can allow other AWS accounts to use this CMK for cryptographic operations\. To do so, in the Other AWS accounts section at the bottom of the page, choose Add another AWS account and enter the AWS account identification number of an external account\. To add multiple external accounts, repeat this step\.
**Note**  
Administrators of the other AWS accounts must also allow access to the CMK by creating IAM policies for their users\. For more information, see [Allowing users in other accounts to use a CMK](key-policy-modifying-external-accounts.md)\.

1. Choose **Next**\.

1. On the **Review and edit key policy** page, review and edit the policy document for the new CMK\. When you're done, choose **Finish**\.

When the procedure succeeds, the display shows the new CMK in the custom key store that you chose\. When you choose the name or alias of the new CMK, its detail page displays the origin of the CMK \(**CloudHSM**\), the name and ID of the custom key store, and the ID of the AWS CloudHSM cluster\. If the procedure fails, an error message appears that describes the failure\.

**Tip**  
To make it easier to identify CMKs in a custom key store, on the **Customer managed keys** page, add the **Custom key store ID** column to the display\. Click the gear icon in the upper\-right and select **Custom key store ID**\.

## Create a CMK in a custom key store \(API\)<a name="create-cmk-keystore-api"></a>

To create a new [customer master key](concepts.md#master_keys) \(CMK\) in your custom key store, use the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation\. Use the `CustomKeyStoreId` parameter to identify your custom key store and specify an `Origin` value of `AWS_CLOUDHSM`\. 

You might also want to use the `Policy` parameter to specify a key policy\. You can change the key policy \([PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html)\) and add optional elements, such as a [description](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) and [tags](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) at any time\.

The examples in this section use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

The following example begins with a call to the [DescribeCustomKeyStores](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeCustomKeyStores.html) operation to verify that the custom key store is connected to its associated AWS CloudHSM cluster\. By default, this operation returns all custom keys stores in your account and Region\. To describe only a particular custom key store, use the `CustomKeyStoreId` or `CustomKeyStoreName` parameter \(but not both\)\.

Before running this command, replace the example custom key store ID with a valid ID\.

```
$ aws kms describe-custom-key-stores --custom-key-store-id cks-1234567890abcdef0
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
    "CustomerMasterKeySpec": "SYMMETRIC_DEFAULT",
    "EncryptionAlgorithms": [
        "SYMMETRIC_DEFAULT"
    ]
  }
}
```