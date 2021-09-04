# How AWS Secrets Manager uses AWS KMS<a name="services-secrets-manager"></a>

[AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/Introduction.html) is an AWS service that encrypts and stores your secrets, and transparently decrypts and returns them to you in plaintext\. It's designed especially to store application secrets, such as login credentials, that change periodically and should not be hard\-coded or stored in plaintext in the application\. In place of hard\-coded credentials or table lookups, your application calls Secrets Manager\.

Secrets Manager also supports features that periodically rotate the secrets associated with commonly used databases\. It always encrypts newly rotated secrets before they are stored\.

Secrets Manager integrates with AWS Key Management Service \(AWS KMS\) to encrypt every version of every secret value with a unique [data key](concepts.md#data-keys) that is protected by an AWS KMS key\. This integration protects your secrets under encryption keys that never leave AWS KMS unencrypted\. It also enables you to set custom permissions on the KMS key and audit the operations that generate, encrypt, and decrypt the data keys that protect your secrets\.

For information about how Secrets Manager uses KMS keys to protect your secrets, see [Encrypting and decrypting secrets](https://docs.aws.amazon.com/secretsmanager/latest/userguide/security-encryption.html) in the *AWS Secrets Manager User Guide*\.