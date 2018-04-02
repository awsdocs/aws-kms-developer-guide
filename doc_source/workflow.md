# How Envelope Encryption Works with Supported AWS Services<a name="workflow"></a>

This topic describes how and when AWS KMS generates, encrypts, and decrypts keys that can be used to encrypt your data in a supported AWS service\.

## Envelope Encryption<a name="envelope_encryption"></a>

AWS KMS supports two kinds of keys— [customer master keys](concepts.md#master_keys) \(CMKs\) and [data keys](concepts.md#data-keys)\. You can use CMKs to encrypt and decrypt up to 4 kilobytes \(4096 bytes\) of data, use them to generate, encrypt, and decrypt data keys\. The data keys are used to encrypt and decrypt customer data of any size\.

Customer master keys are stored securely in AWS KMS\. They can never be exported from AWS KMS unencrypted\. Data keys created in AWS KMS are exported\. They are protected for export by being encrypted under a master key\. 

The following diagram show how to use a master key to generate a data key\. The master key returns two copies of the data key; one in plaintext and one that is encrypted by the master key\.

![\[Envelope encryption diagram\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/Workflow_EnvelopeEncryption.png)

## Encrypting User Data<a name="encrypting_user_data"></a>

When an AWS application or service requests a data key, AWS KMS returns both the encrypted and plaintext key\. The service uses the plaintext data key to encrypt the user's data in memory\. The plaintext key should never be written to disk and should be deleted from memory as soon as practical\. The encrypted copy of the data key should be written to disk alongside of the encrypted data\. This is an acceptable security practice that simplifies management of the encrypted data key\.

![\[Encrypting user data diagram\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/Workflow_EncryptingContent.png)

## Decrypting User Data<a name="decrypting_user_data"></a>

Decryption reverses the encryption process\. When a service or application decrypts data, it sends AWS KMS the encrypted data key\. AWS KMS automatically uses the correct customer master key to decrypt the data key\. Then, it sends the plaintext key back to the service or application that requested it\. The application or service uses the plaintext key to decrypt the data\. The plaintext key should never be written to disk and should be deleted as soon as is practical\. The following illustration shows the customer master key used with a symmetric decryption algorithm to decrypt the data key\.

![\[Decrypting user data diagram\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/Workflow_DecryptingContentPt1.png)

The next illustration shows the plaintext data key and symmetric algorithm used together to decrypt the user's encrypted data\. The plaintext data key should be removed from memory as soon as is practical\.

![\[Decrypting user data diagram\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/Workflow_DecryptingContentPt2.png)

## Managed Keys in AWS Services and in Custom Applications<a name="aws-managed-keys"></a>

You can choose to encrypt data in one of the services integrated with AWS KMS by using the AWS\-managed key for that service under your account\. In this case, all users who have access to that service can use the key\. For more granular control, you can choose to create a customer\-managed key and set policies that define who can use the key and what actions the users can perform\.