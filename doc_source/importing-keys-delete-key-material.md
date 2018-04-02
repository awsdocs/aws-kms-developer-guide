# Deleting Imported Key Material<a name="importing-keys-delete-key-material"></a>

When you import key material, you have the option of specifying a time at which the key material expires\. When the key material expires, AWS KMS deletes the key material and the customer master key \(CMK\) becomes unusable\. You can also delete key material on demand\. Whether you wait for the key material to expire or you delete it manually, the effect is the same\. AWS KMS deletes the key material, the CMK's [key state](key-state.md) changes to *pending import*, and the CMK is unusable\. To use the CMK again, you must reimport the same key material\.

Deleting key material affects the CMK right away, but you can reverse the deletion of key material by reimporting the same key material into the CMK\. In contrast, [scheduling key deletion](deleting-keys.md#deleting-keys-how-it-works) for a CMK is irreversible\. It deletes the key material and all metadata associated with the CMK, and requires a waiting period of between 7 and 30 days\.

To delete key material, you can use the AWS Management Console or the AWS KMS API\. You can use the API directly by making HTTP requests, or through one of the [AWS SDKs](https://aws.amazon.com/tools/#sdk) or [command line tools](https://aws.amazon.com/tools/#cli)\.

**Topics**
+ [How Deleting Key Material Affects AWS Services Integrated With AWS KMS](#importing-keys-delete-key-material-services)
+ [Delete Key Material \(AWS Management Console\)](#importing-keys-delete-key-material-console)
+ [Delete Key Material \(AWS KMS API\)](#importing-keys-delete-key-material-api)

## How Deleting Key Material Affects AWS Services Integrated With AWS KMS<a name="importing-keys-delete-key-material-services"></a>

When you delete key material, the CMK becomes unusable right away\. However, *data encryption keys* that are actively in use by AWS services are not immediately affected\. This means that deleting key material might not immediately affect all of the data and AWS resources protected under the CMK, though they are affected eventually\.

Several [AWS services integrate with AWS KMS](service-integration.md) to protect your data, such as Amazon Elastic Block Store \(Amazon EBS\), Amazon Relational Database Service \(Amazon RDS\), Amazon Simple Storage Service \(Amazon S3\), and others\. Most of these services use *envelope encryption* to protect your data\. Envelope encryption means that the CMK protects a data encryption key \(or *data key*\), and the data key protects your data\. These data keys persist in memory on the AWS service host while the data they are protecting are actively in use\. For more information about how envelope encryption works, see [How Envelope Encryption Works with Supported AWS Services](workflow.md)\.

For example, consider this scenario:

1. You create an encrypted EBS volume, protected under a CMK with imported key material\. This action creates a corresponding request to AWS KMS to generate a unique data key for that volume\.

    

1. AWS KMS generates a new data key, encrypts it with the specified CMK, and then sends the encrypted data key to Amazon EBS to store with the volume until you attach the volume to an Amazon Elastic Compute Cloud \(Amazon EC2\) instance\.

    

1. You attach the EBS volume to an EC2 instance\. This action creates a corresponding request to AWS KMS to decrypt the EBS volume's encrypted data key\.

    

1. AWS KMS decrypts the encrypted data key and sends the decrypted \(plaintext\) data key to Amazon EC2\.

    

1. Amazon EC2 uses the plaintext data key in hypervisor memory to encrypt disk I/O to the EBS volume while the volume is attached to the EC2 instance\.

    

1. You delete the CMK's imported key material\. This has no immediate effect on the EC2 instance or the EBS volume, because Amazon EC2 is using the plaintext data key—not the CMK—to encrypt all disk I/O while the volume is attached to the instance\.

    

1. When the EBS volume is detached \(due to your explicit request or an event that affects the EBS volume or the EC2 instance\), the plaintext data key is deleted from memory\. The next attempt to attach the encrypted EBS volume to an EC2 instance fails\. It fails because the CMK needed to decrypt the EBS volume's encrypted data key \(step 3 in this scenario\) is unusable \(it has no key material\)\. To use the EBS volume again, you must reimport the same key material into the CMK\.

## Delete Key Material \(AWS Management Console\)<a name="importing-keys-delete-key-material-console"></a>

You can use the AWS Management Console to delete key material\.

**To delete key material \(console\)**

1. Sign in to the AWS Management Console and open the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Choose one of the following:
   + Select the check box for the CMK whose key material you want to delete\. Choose **Key actions**, **Delete key material**\.
   + Choose the alias of the CMK whose key material you want to delete\. In the **Key Material** section of the page, choose **Delete key material**\.

1. Confirm that you want to delete the key material and then choose **Delete key material**\. The CMK's key state changes to `Pending Import`\.

## Delete Key Material \(AWS KMS API\)<a name="importing-keys-delete-key-material-api"></a>

To use the [AWS KMS API](http://docs.aws.amazon.com/kms/latest/APIReference/) to delete key material, send a [DeleteImportedKeyMaterial](http://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteImportedKeyMaterial.html) request\. The following example shows how to do this with the [AWS CLI](https://aws.amazon.com/cli/)\.

Replace `1234abcd-12ab-34cd-56ef-1234567890ab` with the key ID of the CMK whose key material you want to delete\. You can use the CMK's key ID or ARN but you cannot use an alias for this operation\.

```
$ aws kms delete-imported-key-material --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
```