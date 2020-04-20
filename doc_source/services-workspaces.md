# How Amazon WorkSpaces uses AWS KMS<a name="services-workspaces"></a>

You can use [Amazon WorkSpaces](https://aws.amazon.com/workspaces/) to provision a cloud\-based desktop \(a *WorkSpace*\) for each of your end users\. When you launch a new WorkSpace, you can choose to encrypt its volumes and decide which AWS KMS [customer master key](concepts.md#master_keys) \(CMK\) to use for the encryption\. You can choose the [AWS managed CMK](concepts.md#aws-managed-cmk) for Amazon WorkSpaces \(**aws/workspaces**\) or a symmetric [customer managed CMK](concepts.md#customer-cmk)\.

**Important**  
Amazon WorkSpaces supports only symmetric CMKs\. You cannot use an asymmetric CMK to encrypt the volumes in an Amazon WorkSpaces\. For help determining whether a CMK is symmetric or asymmetric, see [Identifying symmetric and asymmetric CMKs](find-symm-asymm.md)\.

For more information about creating WorkSpaces with encrypted volumes, go to [Encrypt a WorkSpace](https://docs.aws.amazon.com/workspaces/latest/adminguide/wsp_encrypt_workspace.html) in the *Amazon WorkSpaces Administration Guide*\.

**Topics**
+ [Overview of Amazon WorkSpaces encryption using AWS KMS](#services-workspaces-overview)
+ [Amazon WorkSpaces encryption context](#services-workspaces-encryptioncontext)
+ [Giving Amazon WorkSpaces permission to use a CMK on your behalf](#services-workspaces-permissions)

## Overview of Amazon WorkSpaces encryption using AWS KMS<a name="services-workspaces-overview"></a>

When you create WorkSpaces with encrypted volumes, Amazon WorkSpaces uses Amazon Elastic Block Store \(Amazon EBS\) to create and manage those volumes\. Both services use your KMS customer master key \(CMK\) to work with the encrypted volumes\. For more information about EBS volume encryption, see the following documentation:
+ [How Amazon Elastic Block Store \(Amazon EBS\) uses AWS KMS](services-ebs.md) in this guide
+ [Amazon EBS Encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html) in the *Amazon EC2 User Guide for Windows Instances*

When you launch WorkSpaces with encrypted volumes, the end\-to\-end process works like this:

1. <a name="WSP-you-specify-CMK"></a>You specify the CMK to use for encryption as well as the WorkSpace's user and directory\. This action creates a [grant](grants.md) that allows Amazon WorkSpaces to use your CMK only for this WorkSpace—that is, only for the WorkSpace associated with the specified user and directory\.

1. Amazon WorkSpaces creates an encrypted EBS volume for the WorkSpace and specifies the CMK to use as well as the volume's user and directory \(the same information that you specified at [Step 1](#WSP-you-specify-CMK)\)\. This action creates a [grant](grants.md) that allows Amazon EBS to use your CMK only for this WorkSpace and volume—that is, only for the WorkSpace associated with the specified user and directory, and only for the specified volume\.

1. <a name="WSP-EBS-requests-encrypted-volume-data-key"></a>Amazon EBS requests a volume data key that is encrypted under your CMK and specifies the WorkSpace user's `Sid` and directory ID as well as the volume ID as encryption context\.

1. <a name="WSP-KMS-creates-data-key"></a>AWS KMS creates a new data key, encrypts it under your CMK, and then sends the encrypted data key to Amazon EBS\.

1. <a name="WSP-uses-EBS-to-attach-encrypted-volume"></a>Amazon WorkSpaces uses Amazon EBS to attach the encrypted volume to your WorkSpace\. Amazon EBS sends the encrypted data key to AWS KMS with a [https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) request and specifies the WorkSpace user's `Sid`, its directory ID, and the the volume ID, which is used as the [encryption context](#services-workspaces-encryptioncontext)\.

1. AWS KMS uses your CMK to decrypt the data key, and then sends the plaintext data key to Amazon EBS\.

1. Amazon EBS uses the plaintext data key to encrypt all data going to and from the encrypted volume\. Amazon EBS keeps the plaintext data key in memory for as long as the volume is attached to the WorkSpace\.

1. Amazon EBS stores the encrypted data key \(received at [Step 4](#WSP-KMS-creates-data-key)\) with the volume metadata for future use in case you reboot or rebuild the WorkSpace\.

1. When you use the AWS Management Console to remove a WorkSpace \(or use the [https://docs.aws.amazon.com/workspaces/latest/devguide/API_TerminateWorkspaces.html](https://docs.aws.amazon.com/workspaces/latest/devguide/API_TerminateWorkspaces.html) action in the Amazon WorkSpaces API\), Amazon WorkSpaces and Amazon EBS retire the grants that allowed them to use your CMK for that WorkSpace\.

## Amazon WorkSpaces encryption context<a name="services-workspaces-encryptioncontext"></a>

Amazon WorkSpaces doesn't use your customer master key \(CMK\) directly for cryptographic operations \(such as [https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), [https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), etc\.\), which means Amazon WorkSpaces doesn't send requests to AWS KMS that include an [encryption context](concepts.md#encrypt_context)\. However, when Amazon EBS requests an encrypted data key for the encrypted volumes of your WorkSpaces \([Step 3](#WSP-EBS-requests-encrypted-volume-data-key) in the [Overview of Amazon WorkSpaces encryption using AWS KMS](#services-workspaces-overview)\) and when it requests a plaintext copy of that data key \([Step 5](#WSP-uses-EBS-to-attach-encrypted-volume)\), it includes encryption context in the request\. The encryption context provides [additional authenticated data](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#term-aad) \(AAD\) that AWS KMS uses to ensure data integrity\. The encryption context is also written to your AWS CloudTrail log files, which can help you understand why a given customer master key \(CMK\) was used\. Amazon EBS uses the following for the encryption context:
+ The `sid` of the AWS Directory Service user that is associated with the WorkSpace
+ The directory ID of the AWS Directory Service directory that is associated with the WorkSpace
+ The volume ID of the encrypted volume

The following example shows a JSON representation of the encryption context that Amazon EBS uses:

```
{
  "aws:workspaces:sid-directoryid": "[S-1-5-21-277731876-1789304096-451871588-1107]@[d-1234abcd01]",
  "aws:ebs:id": "vol-1234abcd"
}
```

## Giving Amazon WorkSpaces permission to use a CMK on your behalf<a name="services-workspaces-permissions"></a>

You can protect your workspace data under the AWS managed CMK for Amazon WorkSpaces \(**aws/workspaces**\) or a customer managed CMK\. If you use a customer managed CMK, you need to give Amazon WorkSpaces permission to use the CMK on behalf of the Amazon WorkSpaces administrators in your account\. The AWS managed CMK for Amazon WorkSpaces has the required permissions by default\.

To prepare your customer managed CMK for use with Amazon WorkSpaces, use the following procedure\.

1. [Add the WorkSpaces administrators to the list of key users in the CMK's key policy](#workspaces-permissions-key-users)

1. [Give the WorkSpaces administrators additional permissions with an IAM policy](#workspaces-permissions-iam-policy)

Amazon WorkSpaces administrators also need permission to use Amazon WorkSpaces\. For more information about these permissions, go to [Controlling Access to Amazon WorkSpaces Resources](https://docs.aws.amazon.com/workspaces/latest/adminguide/wsp_iam.html) in the *Amazon WorkSpaces Administration Guide*\.

### Part 1: Adding WorkSpaces administrators to a CMK's key users<a name="workspaces-permissions-key-users"></a>

To give Amazon WorkSpaces administrators the permissions that they require, you can use the AWS Management Console or the AWS KMS API\.

#### To add WorkSpaces administrators as key users for a CMK \(console\)<a name="workspaces-permissions-users-console"></a>

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\.

1. Choose the key ID or alias of your preferred customer managed CMK\.

1. In the **Key policy** section, under **Key users**, choose **Add**\.

1. In the list of IAM users and roles, select the users and roles that correspond to your WorkSpaces administrators, and then choose **Attach**\.

#### To add WorkSpaces administrators as key users for a CMK \(AWS KMS API\)<a name="workspaces-permissions-users-api"></a>

1. Use the [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation to get the existing key policy, and then save the policy document to a file\.

1. Open the policy document in your preferred text editor\. Add the IAM users and roles that correspond to your WorkSpaces administrators to the policy statements that [give permission to key users](key-policies.md#key-policy-default-allow-users)\. Then save the file\.

1. Use the [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operation to apply the key policy to the CMK\.

### Part 2: Giving WorkSpaces administrators extra permissions<a name="workspaces-permissions-iam-policy"></a>

If you are using a customer managed CMK to protect your Amazon WorkSpaces data, in addition to the permissions in the key users section of the [default key policy](key-policies.md#key-policy-default), WorkSpaces administrators need permission to create [grants](grants.md) on the CMK\. Also, if they use the [AWS Management Console](https://console.aws.amazon.com/console/home) to create WorkSpaces with encrypted volumes, WorkSpaces administrators need permission to list aliases and list keys\. For information about creating and editing IAM user policies, see [Managed Policies and Inline Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html) in the *IAM User Guide*\.

To give these permissions to your WorkSpaces administrators, use an IAM policy\. Add an policy statement similar to the following example to the IAM policy for each WorkSpaces administrator\. Replace the example CMK ARN \(`arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab`\) with a valid one\. If your WorkSpaces administrators use only the Amazon WorkSpaces API \(not the console\), you can omit the second policy statement with the `"kms:ListAliases"` and `"kms:ListKeys"` permissions\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "kms:CreateGrant",
      "Resource": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
    },
    {
      "Effect": "Allow",
      "Action": [
        "kms:ListAliases",
        "kms:ListKeys"
      ],
      "Resource": "*"
    }
  ]
}
```