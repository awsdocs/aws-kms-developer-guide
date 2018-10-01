# Deleting Imported Key Material<a name="importing-keys-delete-key-material"></a>

When you import key material, you can specify an expiration date\. When the key material expires, AWS KMS deletes the key material and the customer master key \(CMK\) becomes unusable\. You can also delete key material on demand\. Whether you wait for the key material to expire or you delete it manually, the effect is the same\. AWS KMS deletes the key material, the CMK's [key state](key-state.md) changes to *pending import*, and the CMK is unusable\. To use the CMK again, you must reimport the same key material\.

Deleting key material affects the CMK immediately, but you can reverse the deletion of key material by reimporting the same key material into the CMK\. In contrast, deleting a CMK is irreversible\. If you [schedule key deletion](deleting-keys.md#deleting-keys-how-it-works) and the required waiting period expires, AWS KMS deletes the key material and all metadata associated with the CMK\.

To delete key material, you can use the AWS Management Console or the AWS KMS API\. You can use the API directly by making HTTP requests, or through one of the [AWS SDKs](https://aws.amazon.com/tools/#sdk) or [command line tools](https://aws.amazon.com/tools/#cli)\.

**Topics**
+ [How Deleting Key Material Affects AWS Services Integrated With AWS KMS](#importing-keys-delete-key-material-services)
+ [Delete Key Material \(AWS Management Console\)](#importing-keys-delete-key-material-console)
+ [Delete Key Material \(AWS KMS API\)](#importing-keys-delete-key-material-api)

## How Deleting Key Material Affects AWS Services Integrated With AWS KMS<a name="importing-keys-delete-key-material-services"></a>

When you delete key material, the CMK becomes unusable right away\. However, *data encryption keys* that are actively in use by AWS services are not immediately affected\. This means that deleting key material might not immediately affect all of the data and AWS resources protected under the CMK, though they are affected eventually\.

Several AWS services integrate with AWS KMS to protect your data\. Some of these services, such as [Amazon EBS](https://docs.aws.amazon.com/kms/latest/developerguide/services-ebs.html) and [Amazon Redshift](https://docs.aws.amazon.com/kms/latest/developerguide/services-redshift.html), use a [customer master key](concepts.md#master_keys) \(CMK\) in AWS KMS to generate a [data key](concepts.md#data-keys), and then use the data key to encrypt your data\. These plaintext data keys persist in memory as long as the data they are protecting is actively in use\.

For example, consider this scenario:

1. You create an encrypted EBS volume and specify a CMK with imported key material\. Amazon EBS asks AWS KMS to use your CMK to [generate an encrypted data key](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html) for the volume\. Amazon EBS stores the encrypted data key with the volume\.

    

1. When you attach the EBS volume to an EC2 instance, Amazon EC2 asks AWS KMS to use your CMK to decrypt the EBS volume's encrypted data key\. Amazon EC2 stores the plaintext data key in hypervisor memory and uses it to encrypt disk I/O to the EBS volume\. The data key persists in memory as long as the EBS volume is attached to the EC2 instance\.

    

1. You delete the imported key material from the CMK, which makes it unusable\. This has no immediate effect on the EC2 instance or the EBS volume, because Amazon EC2 is using the plaintext data key—not the CMK—to encrypt all disk I/O while the volume is attached to the instance\.

    

1. However, when the encrypted EBS volume is detached from the EC2 instance, Amazon EBS removes the plaintext key from memory\. The next time the encrypted EBS volume is attached to an EC2 instance, the attachment fails, because Amazon EBS cannot use the CMK to decrypt the volume's encrypted data key\. To use the EBS volume again, you must reimport the same key material into the CMK\. 

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

To use the [AWS KMS API](https://docs.aws.amazon.com/kms/latest/APIReference/) to delete key material, send a [DeleteImportedKeyMaterial](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteImportedKeyMaterial.html) request\. The following example shows how to do this with the [AWS CLI](https://aws.amazon.com/cli/)\.

Replace `1234abcd-12ab-34cd-56ef-1234567890ab` with the key ID of the CMK whose key material you want to delete\. You can use the CMK's key ID or ARN but you cannot use an alias for this operation\.

```
$ aws kms delete-imported-key-material --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
```