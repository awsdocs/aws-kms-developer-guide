# Creating a custom key store<a name="create-keystore"></a>

You can create one or several [custom key stores](key-store-concepts.md#concept-custom-key-store) in your account\. Each custom key store is associated with one AWS CloudHSM cluster in the same AWS Region\. Before you create your custom key store, you need to [assemble the prerequisites](#before-keystore)\. Then, before you can use your custom key store, you must [connect it](disconnect-keystore.md) to its AWS CloudHSM cluster\. 

**Tip**  
You do not have to connect your custom key store immediately\. You can leave it in a disconnected state until you are ready to use it\. However, to verify that it is configured properly, you might want to [connect it](disconnect-keystore.md),[ view its connection status](view-keystore.md), and then [disconnect it](disconnect-keystore.md)\.

**Topics**
+ [Assemble the prerequisites](#before-keystore)
+ [Create a custom key store \(console\)](#create-keystore-console)
+ [Create a custom key store \(API\)](#create-keystore-api)

## Assemble the prerequisites<a name="before-keystore"></a>

Each AWS KMS custom key store is backed by an AWS CloudHSM cluster\. To create a custom key store, you must specify an active AWS CloudHSM cluster that is not already associated with another key store\. You also need to create a dedicated crypto user \(CU\) in the cluster's HSMs that AWS KMS can use to create and manage keys on your behalf\.

Before you create a custom key store, do the following:

**Select an AWS CloudHSM cluster**  
Every custom key store is [associated with exactly one AWS CloudHSM cluster](key-store-concepts.md#concept-cluster)\. When you create a [customer master key](concepts.md#master_keys) \(CMK\) in your custom key store, AWS KMS creates the CMK metadata, such as an ID and Amazon Resource Name \(ARN\) in AWS KMS\. It then creates the key material in the HSMs of the associated cluster\. You can [create a new AWS CloudHSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/getting-started.html) cluster or use an existing one\. AWS KMS does not require exclusive access to the cluster\.  
The AWS CloudHSM cluster that you select is permanently associated with the custom key store\. After you create the custom key store, you can [change the cluster ID](update-keystore.md) of the associated cluster, but the cluster that you specify must share a backup history with the original cluster\. To use an unrelated cluster, you need to create a new custom key store\.  
The AWS CloudHSM cluster that you select must have the following characteristics:  
+ **The cluster must be active**\. 

  You must create the cluster, initialize it, install the AWS CloudHSM client software for your platform, and then activate the cluster\. For detailed instructions, see the[ Getting Started](https://docs.aws.amazon.com/cloudhsm/latest/userguide/getting-started.html) section of the *AWS CloudHSM User Guide*\.
+ **The cluster must be in the same account and Region** as the AWS KMS custom key store\. You cannot associate a custom key store in one region with a cluster in a different region\. To create a key infrastructure in multiple regions, you must create key stores and clusters in each region\.
+ **The cluster cannot be associated with** another custom key store in the account\. Each custom key store must be associated with a different AWS CloudHSM cluster\. You cannot specify a cluster that is already associated with a custom key store or a cluster that shares a backup history with an associated cluster\. Clusters that share a backup history have the same cluster certificate\. To view the cluster certificate of a cluster, use the AWS CloudHSM console or the [DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html) operation\.
+ The cluster must be configured with [private subnets](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-subnets.html) in **at least two Availability Zones** in the Region\. Because AWS CloudHSM is not supported in all Availability Zones, we recommend that you create private subnets in all Availability Zones in the region\. You cannot reconfigure the subnets for an existing cluster, but you can [create a cluster from a backup](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-cluster-from-backup.html) with different subnets in the cluster configuration\.
**Important**  
After you create your custom key store, do not delete any of the private subnets configured for its AWS CloudHSM cluster\. If AWS KMS cannot find all of the subnets in the cluster configuration, attempts to [connect to the custom key store](disconnect-keystore.md) fail with a `SUBNET_NOT_FOUND` connection error state\. For details, see [How to fix a connection failure](fix-keystore.md#fix-keystore-failed)\.
+ The [security group for the cluster](https://docs.aws.amazon.com/cloudhsm/latest/userguide/configure-sg.html) \(cloudhsm\-cluster\-*<cluster\-id>*\-sg\) must include inbound rules and outbound rules that allow TCP traffic on ports 2223\-2225\. The **Source** in the inbound rules and the **Destination** in the outbound rules must match the security group ID\. These rules are set by default when you create the cluster\. Do not delete or change them\.
+ **The cluster must contain at least two active HSMs** in different Availability Zones\. To verify the number of HSMs, use the AWS CloudHSM console or the [DescribeClusters](https://docs.aws.amazon.com/cloudhsm/latest/APIReference/API_DescribeClusters.html) operation\. If necessary, you can [add an HSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/add-remove-hsm.html#add-hsm)\.

**Find the trust anchor certificate**  
When you create a custom key store, you must upload the trust anchor certificate for the AWS CloudHSM cluster to AWS KMS\. AWS KMS needs the trust anchor certificate to connect the custom key store to the cluster\.  
Every active AWS CloudHSM cluster has a *trust anchor certificate*\. When you [initialize the cluster](https://docs.aws.amazon.com/cloudhsm/latest/userguide/initialize-cluster.html#sign-csr), you generate this certificate, save it in the `customerCA.crt` file, and copy it to hosts that connect to the cluster\.

**Create the `kmsuser` crypto user for AWS KMS**  <a name="kmsuser-concept"></a>
To administer your custom key store, AWS KMS logs into the [`kmsuser` crypto user](key-store-concepts.md#concept-kmsuser) \(CU\) account in the selected cluster\. Before you create your custom key store, you must create the `kmsuser` CU\. Then when you create your custom key store, you provide the password for `kmsuser` to AWS KMS\. AWS KMS rotates the `kmsuser` password whenever you connect the custom key store to its associated AWS CloudHSM cluster\.  
Do not specify the `2FA` option when you create the `kmsuser` CU\. If you do, AWS KMS cannot log in and your custom key store cannot be connected to this AWS CloudHSM cluster\. Once you specify 2FA, you cannot undo it\. Instead, you must delete the CU and recreate it\.
To create the `kmsuser` CU, use the following procedure\.  

1. Start cloudhsm\_mgmt\_util as described in the [Prepare to run cloudhsm\_mgmt\_util](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-getting-started.html#cloudhsm_mgmt_util-setup) section of the *AWS CloudHSM User Guide*\.

1. Use the [createUser](https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm_mgmt_util-createUser.html) command in cloudhsm\_mgmt\_util to create a CU named `kmsuser`\. The password must consist of 7\-32 alphanumeric characters\. It is case\-sensitive and cannot contain any special characters\.

   For example, the following example command creates a `kmsuser` CU with a password of `kmsPswd`\. 

   ```
   aws-cloudhsm> createUser CU kmsuser kmsPswd
   ```

## Create a custom key store \(console\)<a name="create-keystore-console"></a>

When you create a [custom key store](key-store-concepts.md#concept-custom-key-store) in the AWS Management Console, you can add and create the [prerequisites](#before-keystore) as part of your workflow\. However, the process is quicker when you have assembled them in advance\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Custom key stores**\.

1. Choose **Create key store**\.

1. Enter a friendly name for the custom key store\. The name must be unique in the account\.

1. Select [an AWS CloudHSM cluster](key-store-concepts.md#concept-cluster) for the custom key store\. Or, to create a new AWS CloudHSM cluster, choose the **Create an AWS CloudHSM cluster** link\.

   The cluster must [fulfill the requirements](#before-keystore) for association with a custom key store\. The menu displays custom key stores in your account and region that are not already associated with a custom key store\.

1. Choose **Upload file**, and then upload the trust anchor certificate for the AWS CloudHSM cluster that you chose\. This is the `customerCA.crt` file that you created when you [initialized the cluster](https://docs.aws.amazon.com/cloudhsm/latest/userguide/initialize-cluster.html#sign-csr)\.

1. Enter the password of [the `kmsuser` crypto user](key-store-concepts.md#concept-kmsuser) \(CU\) that you created in the selected cluster\. 

1. Choose **Create**\.

When the procedure is successful, the new custom key store appears in the list of custom key stores in the account and Region\. If it is unsuccessful, an error message appears that describes the problem and provides help on how to fix it\. If you need more help, see [Troubleshooting a custom key store](fix-keystore.md)\.

**Next**: New custom key stores are not automatically connected\. Before you can create customer master keys \(CMKs\) in the custom key store, you must [connect the custom key store](disconnect-keystore.md) to its associated AWS CloudHSM cluster\.

## Create a custom key store \(API\)<a name="create-keystore-api"></a>

The [CreateCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateCustomKeyStore.html) operation creates a new [custom key store](key-store-concepts.md#concept-custom-key-store) that is associated with an AWS CloudHSM cluster in the account and Region\. These examples use the AWS Command Line Interface \(AWS CLI\), but you can use any supported programming language\.

The `CreateCustomKeyStore` operation requires the following parameter values\.
+ CustomKeyStoreName – A friendly name for the custom key store that is unique in the account\.
+ CloudHsmClusterId – The cluster ID of a cluster that [fulfills the requirements](#before-keystore) for association with a custom key store\.
+ KeyStorePassword – The password of `kmsuser` CU account in the specified cluster\. 
+ TrustAnchorCertificate – The content of the `customerCA.crt` file that you created when you [initialized the cluster](https://docs.aws.amazon.com/cloudhsm/latest/userguide/initialize-cluster.html)\.

The following example uses a fictitious cluster ID\. Before running the command, replace it with a valid cluster ID\.

```
$ aws kms create-custom-key-store
        --custom-key-store-name ExampleKeyStore \
        --cloud-hsm-cluster-id cluster-1a23b4cdefg \
        --key-store-password kmsPswd \
        --trust-anchor-certificate <certificate-goes-here>
```

If you are using the AWS CLI, you can specify the trust anchor certificate file, instead of its contents\. In the following example, the `customerCA.crt` file is in the root directory\.

```
$ aws kms create-custom-key-store
        --custom-key-store-name ExampleKeyStore \
        --cloud-hsm-cluster-id cluster-1a23b4cdefg \
        --key-store-password kmsPswd \
        --trust-anchor-certificate file://customerCA.crt
```

When the operation is successful, `CreateCustomKeyStore` returns the custom key store ID, as shown in the following example response\.

```
{
    "CustomKeyStoreId": cks-1234567890abcdef0
}
```

If the operation fails, correct the error indicated by the exception, and try again\. For additional help, see [Troubleshooting a custom key store](fix-keystore.md)\.

Next, to use the custom key store, [connect it to its AWS CloudHSM cluster](disconnect-keystore.md)\.