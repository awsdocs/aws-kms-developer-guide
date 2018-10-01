# How Amazon WorkSpaces Uses AWS KMS<a name="services-workspaces"></a>

You can use [Amazon WorkSpaces](https://aws.amazon.com/workspaces/) to provision a cloud\-based desktop \(a *WorkSpace*\) for each of your end users\. When you launch a new WorkSpace, you can choose to encrypt its volumes and decide which AWS KMS customer master key \(CMK\) to use for the encryption\. You can choose your account's default CMK for Amazon WorkSpaces \(use the alias **aws/workspaces**\), or you can choose a custom CMK that you created separately in AWS KMS\.

For more information about creating WorkSpaces with encrypted volumes, go to [Encrypt a WorkSpace](https://docs.aws.amazon.com/workspaces/latest/adminguide/wsp_encrypt_workspace.html) in the *Amazon WorkSpaces Administration Guide*\.

**Topics**
+ [Overview of Amazon WorkSpaces Encryption Using AWS KMS](#services-workspaces-overview)
+ [Amazon WorkSpaces Encryption Context](#services-workspaces-encryptioncontext)
+ [Giving Amazon WorkSpaces Permission to Use A CMK On Your Behalf](#services-workspaces-permissions)

## Overview of Amazon WorkSpaces Encryption Using AWS KMS<a name="services-workspaces-overview"></a>

When you create WorkSpaces with encrypted volumes, Amazon WorkSpaces uses Amazon Elastic Block Store \(Amazon EBS\) to create and manage those volumes\. Both services use your KMS customer master key \(CMK\) to work with the encrypted volumes\. For more information about EBS volume encryption, see the following documentation:
+ [How Amazon Elastic Block Store \(Amazon EBS\) Uses AWS KMS](services-ebs.md) in this guide
+ [Amazon EBS Encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html) in the *Amazon EC2 User Guide for Windows Instances*

When you launch WorkSpaces with encrypted volumes, the end\-to\-end process works like this:

1. <a name="WSP-you-specify-CMK"></a>You specify the CMK to use for encryption as well as the WorkSpace's user and directory\. This action creates a [grant](grants.md) that allows Amazon WorkSpaces to use your CMK only for this WorkSpace—that is, only for the WorkSpace associated with the specified user and directory\.

1. Amazon WorkSpaces creates an encrypted EBS volume for the WorkSpace and specifies the CMK to use as well as the volume's user and directory \(the same information that you specified at [Step 1](#WSP-you-specify-CMK)\)\. This action creates a [grant](grants.md) that allows Amazon EBS to use your CMK only for this WorkSpace and volume—that is, only for the WorkSpace associated with the specified user and directory, and only for the specified volume\.

1. <a name="WSP-EBS-requests-encrypted-volume-data-key"></a>Amazon EBS requests a volume data key that is encrypted under your CMK and specifies the WorkSpace user's `Sid` and directory ID as well as the volume ID as encryption context\.

1. <a name="WSP-KMS-creates-data-key"></a>AWS KMS creates a new data key, encrypts it under your CMK, and then sends the encrypted data key to Amazon EBS\.

1. <a name="WSP-uses-EBS-to-attach-encrypted-volume"></a>Amazon WorkSpaces uses Amazon EBS to attach the encrypted volume to your WorkSpace, at which time Amazon EBS sends the encrypted data key to AWS KMS with a [https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) request and specifies the WorkSpace user's `Sid` and directory ID as well as the volume ID as encryption context\.

1. AWS KMS uses your CMK to decrypt the data key, and then sends the plaintext data key to Amazon EBS\.

1. Amazon EBS uses the plaintext data key to encrypt all data going to and from the encrypted volume\. Amazon EBS keeps the plaintext data key in memory for as long as the volume is attached to the WorkSpace\.

1. Amazon EBS stores the encrypted data key \(received at [Step 4](#WSP-KMS-creates-data-key)\) with the volume metadata for future use in case you reboot or rebuild the WorkSpace\.

1. When you use the AWS Management Console to remove a WorkSpace \(or use the [https://docs.aws.amazon.com/workspaces/latest/devguide/API_TerminateWorkspaces.html](https://docs.aws.amazon.com/workspaces/latest/devguide/API_TerminateWorkspaces.html) action in the Amazon WorkSpaces API\), Amazon WorkSpaces and Amazon EBS retire the grants that allowed them to use your CMK for that WorkSpace\.

## Amazon WorkSpaces Encryption Context<a name="services-workspaces-encryptioncontext"></a>

Amazon WorkSpaces doesn't use your customer master key \(CMK\) directly for cryptographic operations \(such as [https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), [https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), etc\.\), which means Amazon WorkSpaces doesn't send requests to AWS KMS that include encryption context\. However, when Amazon EBS requests an encrypted data key for the encrypted volumes of your WorkSpaces \([Step 3](#WSP-EBS-requests-encrypted-volume-data-key) in the [Overview of Amazon WorkSpaces Encryption Using AWS KMS](#services-workspaces-overview)\) and when it requests a plaintext copy of that data key \([Step 5](#WSP-uses-EBS-to-attach-encrypted-volume)\), it includes encryption context in the request\. The encryption context provides additional authenticated information that AWS KMS uses to ensure data integrity\. The encryption context is also written to your AWS CloudTrail log files, which can help you understand why a given customer master key \(CMK\) was used\. Amazon EBS uses the following for the encryption context:
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

For more information about encryption context, see [Encryption Context](encryption-context.md)\.

## Giving Amazon WorkSpaces Permission to Use A CMK On Your Behalf<a name="services-workspaces-permissions"></a>

You can use your account's default customer master key \(CMK\) for Amazon WorkSpaces with the alias **aws/workspaces**, or you can use a custom CMK that you create\. If you use the default CMK for Amazon WorkSpaces, you don't need to perform any steps to give Amazon WorkSpaces permission to use it\. AWS KMS automatically specifies the necessary permissions in the [key policy](key-policies.md) for the default CMK\.

To use a custom CMK, the WorkSpaces administrators who create WorkSpaces with encrypted volumes must have permission to use the CMK\. The WorkSpaces administrators don't use the CMK directly\. Simply creating a WorkSpace with encrypted volumes implicitly creates the [grant](grants.md) that gives Amazon WorkSpaces permission to use the CMK on the administrator's behalf\.

Even though the WorkSpaces administrators don't use the CMK directly, they need permission to use the CMK because they can only grant permissions that they have\. To give WorkSpaces administrators permission to use a CMK, do these things:

1. [Add the WorkSpaces administrators to the list of key users in the CMK's key policy](#workspaces-permissions-key-users)

1. [Give the WorkSpaces administrators extra permissions with an IAM policy](#workspaces-permissions-iam-policy)

WorkSpaces administrators also need permission to use Amazon WorkSpaces\. For more information about these permissions, go to [Controlling Access to Amazon WorkSpaces Resources](https://docs.aws.amazon.com/workspaces/latest/adminguide/wsp_iam.html) in the *Amazon WorkSpaces Administration Guide*\.

### Part 1: Adding WorkSpaces Administrators to a CMK's Key Users<a name="workspaces-permissions-key-users"></a>

To add WorkSpaces administrators to the list of key users in a CMK's key policy, you can use the AWS Management Console or the AWS Command Line Interface \(AWS CLI\)\.

**To add WorkSpaces administrators as key users for a CMK \(console\)**

1. Sign in to the AWS Management Console and open the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Choose the alias of the CMK that WorkSpaces administrators will use\.

1. In the **Key Policy** section, under **Key Users**, choose **Add**\.

1. In the list of IAM users and roles, select the users and roles that correspond to your WorkSpaces administrators, and then choose **Attach**\.

**To add WorkSpaces administrators as key users for a CMK \(AWS CLI\)**

1. Use the [https://docs.aws.amazon.com/cli/latest/reference/kms/get-key-policy.html](https://docs.aws.amazon.com/cli/latest/reference/kms/get-key-policy.html) command to retrieve the existing key policy, and then save the policy document to a file\.

1. Open the policy document in your preferred text editor\. Add the IAM users and roles that correspond to your WorkSpaces administrators to the policy statements that [give permission to key users](key-policies.md#key-policy-default-allow-users)\. Then save the file\.

1. Use the [https://docs.aws.amazon.com/cli/latest/reference/kms/put-key-policy.html](https://docs.aws.amazon.com/cli/latest/reference/kms/put-key-policy.html) command to apply the key policy to the CMK\.

### Part 2: Giving WorkSpaces Administrators Extra Permissions with an IAM Policy<a name="workspaces-permissions-iam-policy"></a>

In addition to the permissions in the key users section of the [default key policy](key-policies.md#key-policy-default), WorkSpaces administrators need some permissions in an IAM user policy that applies to them\. For information about creating and editing IAM user policies, go to [Working with Managed Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-using.html) and [Working with Inline Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_inline-using.html) in the *IAM User Guide*\.

At minimum, WorkSpaces administrators need permission to create [grants](grants.md) for the custom CMK\(s\) that they will use with Amazon WorkSpaces\. To use the [AWS Management Console](https://console.aws.amazon.com/console/home) to create WorkSpaces with encrypted volumes, WorkSpaces administrators also need permission to list aliases and list keys, which are actions that the console performs on behalf of WorkSpaces administrators to display a list of available CMKs\.

To give these permissions to your WorkSpaces administrators, add an IAM policy similar to the following example to your WorkSpaces administrators\. Replace `arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab` in the first policy statement with the ARN\(s\) of the CMK\(s\) that WorkSpaces administrators will use when they create WorkSpaces with encrypted volumes\. If your WorkSpaces administrators will launch WorkSpaces with only the Amazon WorkSpaces API \(not with the console\), you can omit the second statement with the `"kms:ListAliases"` and `"kms:ListKeys"` permissions\.

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