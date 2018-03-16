# How Amazon Elastic Block Store \(Amazon EBS\) Uses AWS KMS<a name="services-ebs"></a>

This topic discusses in detail how [Amazon Elastic Block Store \(Amazon EBS\)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html) uses AWS KMS to encrypt volumes and snapshots\. For basic instructions about encrypting Amazon EBS volumes, see [Amazon EBS Encryption](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html)\.


+ [Amazon EBS Encryption](#ebs-encrypt)
+ [Amazon EBS Encryption Context](#ebs-encryption-context)
+ [Detecting Amazon EBS Failures](#ebs-failures)
+ [Using AWS CloudFormation to Create Encrypted Amazon EBS Volumes](#ebs-encryption-using-cloudformation)

## Amazon EBS Encryption<a name="ebs-encrypt"></a>

When you attach an encrypted Amazon EBS volume to a [supported Amazon Elastic Compute Cloud \(Amazon EC2\) instance type](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#EBSEncryption_supported_instances), data stored at rest on the volume, disk I/O, and snapshots created from the volume are all encrypted\. The encryption occurs on the servers that host Amazon EC2 instances\.

This feature is supported on all [Amazon EBS volume types](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html)\. You access encrypted volumes the same way you access other volumes; encryption and decryption are handled transparently and they require no additional action from you, your EC2 instance, or your application\. Snapshots of encrypted volumes are automatically encrypted, and volumes that are created from encrypted snapshots are also automatically encrypted\.

The encryption status of an EBS volume is determined when you create the volume\. You cannot change the encryption status of an existing volume\. However, you can [migrate data](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#EBSEncryption_considerations) between encrypted and unencrypted volumes and apply a new encryption status while copying a snapshot\.

To [create an encrypted Amazon EBS volume](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-volume.html), select the appropriate box in the Amazon EBS section of the Amazon EC2 console\. You can use a custom [customer master key \(CMK\)](concepts.md#master_keys) by choosing one from the list that appears below the encryption box\. If you do not specify a custom CMK, Amazon EBS uses the AWS\-managed CMK for Amazon EBS in your account\. If there is no AWS\-managed CMK for Amazon EBS in your account, Amazon EBS creates one\.

The following explains how Amazon EBS uses your CMK:

1. When you create an encrypted EBS volume, Amazon EBS sends a [GenerateDataKeyWithoutPlaintext](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html) request to AWS KMS, specifying the CMK that you chose for EBS volume encryption\.

1. AWS KMS generates a new data key, encrypts it under the specified CMK, and then sends the encrypted data key to Amazon EBS to store with the volume metadata\.

1. When you attach the encrypted volume to an EC2 instance, Amazon EC2 sends the encrypted data key to AWS KMS with a [Decrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) request\.

1. AWS KMS decrypts the encrypted data key and then sends the decrypted \(plaintext\) data key to Amazon EC2\.

1. Amazon EC2 uses the plaintext data key in hypervisor memory to encrypt disk I/O to the EBS volume\. The data key persists in memory as long as the EBS volume is attached to the EC2 instance\.

## Amazon EBS Encryption Context<a name="ebs-encryption-context"></a>

Amazon EBS sends [encryption context](encryption-context.md) when making AWS KMS API requests to generate data keys and decrypt\. Amazon EBS uses the volume ID as encryption context for all volumes and for encrypted snapshots created with the [CreateSnapshot](http://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateSnapshot.html) operation in the Amazon EC2 API\. In the `requestParameters` field of a CloudTrail log entry, the encryption context looks similar to the following:

```
"encryptionContext": {
  "aws:ebs:id": "vol-0cfb133e847d28be9"
}
```

Amazon EBS uses the snapshot ID as encryption context for encrypted snapshots created with the [CopySnapshot](http://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CopySnapshot.html) operation in the Amazon EC2 API\. In the `requestParameters` field of a CloudTrail log entry, the encryption context looks similar to the following:

```
"encryptionContext": {
  "aws:ebs:id": "snap-069a655b568de654f"
}
```

## Detecting Amazon EBS Failures<a name="ebs-failures"></a>

To create an encrypted EBS volume or attach the volume to an EC2 instance, Amazon EBS and the Amazon EC2 infrastructure must be able to use the CMK that you specified for EBS volume encryption\. When the CMK is not usable—for example, when it is not in the enabled [key state](key-state.md)—the volume creation or volume attachment fails\. In this case, Amazon EBS sends an *event* to Amazon CloudWatch Events to notify you about the failure\. With CloudWatch Events, you can establish rules that trigger automatic actions in response to these events\. For more information, see [Amazon CloudWatch Events for Amazon EBS](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-cloud-watch-events.html) in the *Amazon EC2 User Guide for Linux Instances*, especially the following sections:

+ [Invalid Encryption Key on Volume Attach or Reattach](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-cloud-watch-events.html#attach-fail-key)

+ [Invalid Encryption Key on Create Volume](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-cloud-watch-events.html#create-fail-key)

To fix these failures, ensure that the CMK that you specified for EBS volume encryption is enabled\. To do this, first [view the CMK](viewing-keys.md) to determine its current key state \(the **Status** column in the AWS Management Console\)\. Then, see the information at one of the following links:

+ If the CMK's key state is disabled, [enable it](enabling-keys.md)\.

+ If the CMK's key state is pending import, [import key material](importing-keys.md#importing-keys-overview)\.

+ If the CMK's key state is pending deletion, [cancel key deletion](deleting-keys.md#deleting-keys-scheduling-key-deletion)\.

## Using AWS CloudFormation to Create Encrypted Amazon EBS Volumes<a name="ebs-encryption-using-cloudformation"></a>

You can use [AWS CloudFormation](https://aws.amazon.com/cloudformation/) to create encrypted Amazon EBS volumes\. For more information, see [AWS::EC2::Volume](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-ebs-volume.html) in the *AWS CloudFormation User Guide*\.