# How Symmetric Key Cryptography Works<a name="crypto_overview"></a>

This topic provides a high\-level introduction to how symmetric key cryptography uses algorithms to encrypt and decrypt data, the difference between block and stream ciphers, and how block ciphers use encryption modes to expand the effectiveness of the generic encryption schemes\.

## Encryption and Decryption<a name="symmetric"></a>

AWS KMS uses symmetric key cryptography to perform encryption and decryption\. Symmetric key cryptography uses the same algorithm and key to both encrypt and decrypt digital data\. The unencrypted data is typically called plaintext whether it is text or not\. The encrypted data is typically called ciphertext\. The following illustration shows a secret \(symmetric\) key and a symmetric algorithm being used to turn plaintext into ciphertext\.

![\[Symmetric key encryption diagram\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/Symmetric_Key_Encryption_sm.png)

The next illustration shows the same secret key and symmetric algorithm being used to turn ciphertext back into plaintext\.

![\[Symmetric key decryption diagram\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/Symmetric_Key_Decryption_sm.png)