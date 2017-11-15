# How Envelope Encryption Works with Supported AWS Services<a name="workflow"></a>

This topic describes how and when AWS KMS generates, encrypts, and decrypts keys that can be used to encrypt your data in a supported AWS service\.

## Envelope Encryption<a name="envelope_encryption"></a>

AWS KMS supports two kinds of keys—*master keys* and *data keys*\. Master keys can be used to directly encrypt and decrypt up to 4 kilobytes \(4096 bytes\) of data and can also be used to protect data keys\. The data keys are then used to encrypt and decrypt customer data\.

Customer master keys are stored securely in AWS KMS\. They can never be exported from AWS KMS\. Data keys created inside of AWS KMS can be exported and are protected for export by being encrypted under a master key\. The data key encryption process is illustrated by the following diagram:

![\[Envelope encryption diagram\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/Workflow_EnvelopeEncryption.png)

## Encrypting User Data<a name="encrypting_user_data"></a>

When a data key is requested, AWS KMS returns both the encrypted and plaintext key back to the service or application that requested it\. The plaintext data key is used to encrypt the user's data in memory\. This key should never be written to disk and should be deleted from memory as soon as practical\. The encrypted copy of the data key should be written to disk alongside of the encrypted data\. This is acceptable and simplifies management of the encrypted data key\.

![\[Encrypting user data diagram\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/Workflow_EncryptingContent.png)

## Decrypting User Data<a name="decrypting_user_data"></a>

Decryption reverses the encryption process\. When a service or application needs to decrypt data, it sends AWS KMS the encrypted data key\. AWS KMS decrypts the data key by automatically using the correct customer master key and then sends the plaintext key back to the service or application that requested it\. The plaintext key is used to decrypt the data\. This key should never be written to disk and should be deleted as soon as is practical\. The following illustration shows the customer master key used with the symmetric decryption algorithm to decrypt the data key\.

![\[Decrypting user data diagram\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/Workflow_DecryptingContentPt1.png)

The next illustration shows the plaintext data key and symmetric algorithm used together to decrypt the user's encrypted data\. The plaintext data key should be removed from memory as soon as is practical\.

![\[Decrypting user data diagram\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/Workflow_DecryptingContentPt2.png)

## Managed Keys in AWS Services and in Custom Applications<a name="aws-managed-keys"></a>

You can choose to encrypt data in one of the services integrated with AWS KMS by using the AWS\-managed key for that service under your account\. In this case, all users who have access to that service can use the key\. For more granular control, you can choose to create a customer\-managed key and set policies that define who can use the key and what actions the users can perform\.