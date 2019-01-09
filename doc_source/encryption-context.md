# Encryption Context<a name="encryption-context"></a>

*Encryption context* is a set of non\-secret key\-value pairs that you can pass to AWS KMS when you call the [https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), [https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html), [https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), or [https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html) APIs\.

When an encryption context is provided in an encryption request, it is cryptographically bound to the ciphertext such that the same encryption context is required to decrypt \(or decrypt and re\-encrypt\) the data\. If the encryption context provided in the decryption request is not an exact, case\-sensitive match, the decrypt request fails\. Only the order of the encryption context pairs can vary\.

The encryption context is never included in the ciphertext that AWS KMS returns for any encryption operation\. However, it logged by AWS CloudTrail\.

To learn how to use encryption context to protect the integrity of encrypted data, see the post [How to Protect the Integrity of Your Encrypted Data by Using AWS Key Management Service and EncryptionContext](https://aws.amazon.com/blogs/security/how-to-protect-the-integrity-of-your-encrypted-data-by-using-aws-key-management-service-and-encryptioncontext/) on the AWS Security Blog\.

Encryption context can consist of any values that you want\. However, because it is not secret, not encrypted, and appears in CloudTrail logs, your encryption context should not include sensitive information\. We recommend that your encryption context describe the data being encrypted or decrypted so that you can better understand the CloudTrail log entries produced by AWS KMS\. For example, Amazon EBS uses the ID of the encrypted volume as the encryption context for server\-side encryption\. If you are encrypting a file, you might use part of the file path as encryption context\.

## Encryption Context in Grants and Key Policies<a name="encryption-context-authorization"></a>

In addition to its primary use in verifying integrity and authenticity, you can also use the encryption context as a condition for authorizing use of customer master keys \(CMKs\) in IAM and key policies, and grants\. This element can limit the permissions to very specific types of data or data from a limited set of sources\.
+ In key policies and IAM policies that control access to AWS KMS CMKs, you can include condition keys that limit the permission to requests that include particular [encryption context keys](policy-conditions.md#conditions-kms-encryption-context-keys) or [key\-value pairs](policy-conditions.md#conditions-kms-encryption-context)\.
+ When you [create a grant](grants.md), you can include [grant constraints](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) that allow access only when a request includes a particular encryption context or encryption context keys\.

For example, when an Amazon EBS volume is attached to an Amazon EC2 instance, a grant is created that allows only that instance to decrypt only that volume\. This is accomplished by including the volume ID in the encryption context, then adding a [grant constraint](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) that requires an encryption context with that volume ID\. If the grant did not include the encryption context constraint, the Amazon EC2 instance could decrypt any volume that was encrypted under the customer master key \(CMK\), rather than a specific volume\.

## Logging Encryption Context<a name="encryption-context-auditing"></a>

AWS KMS uses AWS CloudTrail to log the encryption context so you can determine which CMKs and data have been accessed\. The log entry shows exactly which CMK was used to encrypt or decrypt specific data referenced by the encryption context in the log entry\.

**Important**  
Because the encryption context is logged, it must not contain sensitive information\.

## Storing Encryption Context<a name="encryption-context-storing"></a>

To simplify use of any encryption context when you call the [https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) \(or [https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)\) API, you can store the encryption context alongside the encrypted data\. We recommend that you store only enough of the encryption context to help you create the full encryption context when you need it for encryption or decryption\. 

For example, if the encryption context is the fully qualified path to a file, store only part of that path with the encrypted file contents\. Then, when you need the full encryption context, reconstruct it from the stored fragment\. If someone tampers with the file, such as renaming it or moving it to a different location, the encryption context value changes and the decryption request fails\.