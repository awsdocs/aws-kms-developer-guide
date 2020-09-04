# How Amazon Elastic Block Store \(Amazon EBS\) uses AWS KMS<a name="services-ebs"></a>

This topic discusses in detail how [Amazon Elastic Block Store \(Amazon EBS\)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html) uses AWS KMS to encrypt volumes and snapshots\. For basic instructions about encrypting Amazon EBS volumes, see [Amazon EBS Encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html)\.

**Topics**
+ [Amazon EBS encryption](#ebs-encrypt)
+ [Using CMKs and data keys](#ebs-cmk)
+ [Amazon EBS encryption context](#ebs-encryption-context)
+ [Detecting Amazon EBS failures](#ebs-failures)
+ [Using AWS CloudFormation to create encrypted Amazon EBS volumes](#ebs-encryption-using-cloudformation)

## Amazon EBS encryption<a name="ebs-encrypt"></a>

When you attach an encrypted Amazon EBS volume to a [supported Amazon Elastic Compute Cloud \(Amazon EC2\) instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#EBSEncryption_supported_instances), data stored at rest on the volume, disk I/O, and snapshots created from the volume are all encrypted\. The encryption occurs on the servers that host Amazon EC2 instances\.

This feature is supported on all [Amazon EBS volume types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html)\. You access encrypted volumes the same way you access other volumes; encryption and decryption are handled transparently and they require no additional action from you, your EC2 instance, or your application\. Snapshots of encrypted volumes are automatically encrypted, and volumes that are created from encrypted snapshots are also automatically encrypted\.

The encryption status of an EBS volume is determined when you create the volume\. You cannot change the encryption status of an existing volume\. However, you can [migrate data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#EBSEncryption_considerations) between encrypted and unencrypted volumes and apply a new encryption status while copying a snapshot\.

Amazon EBS supports optional encryption by default\. You can enable encryption automatically on all new EBS volumes and snapshot copies in your AWS account and Region\. This configuration setting doesn't affect existing volumes or snapshots\. For details, see **Encryption by default** in the [Amazon EC2 User Guide for Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#encryption-by-default) or [Amazon EC2 User Guide for Windows Instances](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/EBSEncryption.html#encryption-by-default)\.

## Using CMKs and data keys<a name="ebs-cmk"></a>

When you [create an encrypted Amazon EBS volume](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-volume.html), you specify an AWS KMS customer master key \(CMK\)\. By default, Amazon EBS uses the [AWS managed CMK](concepts.md#aws-managed-cmk) for Amazon EBS in your account \(`aws/ebs`\)\. However, you can specify a [customer managed CMK](concepts.md#customer-cmk) that you create and manage\. 

To use a customer managed CMK, you must give Amazon EBS permission to use the CMK on your behalf\. For a list of required permissions, see **Permissions for IAM users** in the [Amazon EC2 User Guide for Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#ebs-encryption-permissions) or [Amazon EC2 User Guide for Windows Instances](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/EBSEncryption.html#ebs-encryption-permissions)\.

**Important**  
Amazon EBS supports only [symmetric CMKs](symm-asymm-concepts.md#symmetric-cmks)\. You cannot use an [asymmetric CMK](symm-asymm-concepts.md#asymmetric-cmks) to encrypt an Amazon EBS volume\. For help determining whether a CMK is symmetric or asymmetric, see [Identifying symmetric and asymmetric CMKs](find-symm-asymm.md)\.

For each volume, Amazon EBS asks AWS KMS to generate a unique data key encrypted under the CMK that you specify\. Amazon EBS stores the encrypted data key with the volume\. Then, when you attach the volume to an Amazon EC2 instance, Amazon EBS calls AWS KMS to decrypt the data key\. Amazon EBS uses the plaintext data key in hypervisor memory to encrypt all disk I/O to the volume\. For details, see **How EBS encryption works** in the [Amazon EC2 User Guide for Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#how-ebs-encryption-works) or [Amazon EC2 User Guide for Windows Instances](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/EBSEncryption.html#how-ebs-encryption-works)\.

## Amazon EBS encryption context<a name="ebs-encryption-context"></a>

In its [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) requests to AWS KMS, Amazon EBS uses an encryption context with a name\-value pair that identifies the volume or snapshot in the request\. The name in the encryption context does not vary\.

An [encryption context](concepts.md#encrypt_context) is a set of key–value pairs that contain arbitrary nonsecret data\. When you include an encryption context in a request to encrypt data, AWS KMS cryptographically binds the encryption context to the encrypted data\. To decrypt the data, you must pass in the same encryption context\.

For all volumes and for encrypted snapshots created with the Amazon EBS [CreateSnapshot](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateSnapshot.html) operation, Amazon EBS uses the volume ID as encryption context value\. In the `requestParameters` field of a CloudTrail log entry, the encryption context looks similar to the following:

```
"encryptionContext": {
  "aws:ebs:id": "vol-0cfb133e847d28be9"
}
```

For encrypted snapshots created with the Amazon EC2 [CopySnapshot](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CopySnapshot.html) operation, Amazon EBS uses the snapshot ID as encryption context value\. In the `requestParameters` field of a CloudTrail log entry, the encryption context looks similar to the following:

```
"encryptionContext": {
  "aws:ebs:id": "snap-069a655b568de654f"
}
```

## Detecting Amazon EBS failures<a name="ebs-failures"></a>

To create an encrypted EBS volume or attach the volume to an EC2 instance, Amazon EBS and the Amazon EC2 infrastructure must be able to use the CMK that you specified for EBS volume encryption\. When the CMK is not usable—for example, when its [key state](key-state.md) is not `Enabled` —the volume creation or volume attachment fails\.

 In this case, Amazon EBS sends an *event* to Amazon CloudWatch Events to notify you about the failure\. With CloudWatch Events, you can establish rules that trigger automatic actions in response to these events\. For more information, see [Amazon CloudWatch Events for Amazon EBS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-cloud-watch-events.html) in the *Amazon EC2 User Guide for Linux Instances*, especially the following sections:
+ [Invalid Encryption Key on Volume Attach or Reattach](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-cloud-watch-events.html#attach-fail-key)
+ [Invalid Encryption Key on Create Volume](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-cloud-watch-events.html#create-fail-key)

To fix these failures, ensure that the CMK that you specified for EBS volume encryption is enabled\. To do this, first [view the CMK](viewing-keys.md) to determine its current key state \(the **Status** column in the AWS Management Console\)\. Then, see the information at one of the following links:
+ If the CMK's key state is disabled, [enable it](enabling-keys.md)\.
+ If the CMK's key state is pending import, [import key material](importing-keys.md#importing-keys-overview)\.
+ If the CMK's key state is pending deletion, [cancel key deletion](deleting-keys.md#deleting-keys-scheduling-key-deletion)\.

## Using AWS CloudFormation to create encrypted Amazon EBS volumes<a name="ebs-encryption-using-cloudformation"></a>

You can use [AWS CloudFormation](https://aws.amazon.com/cloudformation/) to create encrypted Amazon EBS volumes\. For more information, see [AWS::EC2::Volume](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-ebs-volume.html) in the *AWS CloudFormation User Guide*\.