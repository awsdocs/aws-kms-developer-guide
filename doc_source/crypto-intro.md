# Cryptography Basics<a name="crypto-intro"></a>

Following are some basic terms and concepts in cryptography that you'll encounter when you work with AWS KMS\.

**Plaintext and Ciphertext**  
*Plaintext* refers to information or data in an unencrypted, or unprotected, form\. *Ciphertext* refers to the output of an encryption algorithm operating on plaintext\. Ciphertext is unreadable without knowledge of the algorithm and a secret key\.

**Algorithms and Keys**  
An * encryption algorithm* is a step\-by\-step set of instructions that specifies precisely how plaintext is transformed into ciphertext\. Encryption algorithms require a secret key\. AWS KMS uses the [Advanced Encryption Standard \(AES\)](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) algorithm in [Galois/Counter Mode \(GCM\)](https://en.wikipedia.org/wiki/Galois/Counter_Mode), known as AES\-GCM\. AWS KMS uses this algorithm with 256\-bit secret keys\.

**Symmetric and Asymmetric Encryption**  
Encryption algorithms are either symmetric or asymmetric\. *Symmetric encryption* uses the same secret key to perform both the encryption and decryption processes\. *Asymmetric encryption*, also known as *public\-key encryption*, uses two keys, a public key for encryption and a corresponding private key for decryption\. The public key and private key are mathematically related so that when the public key is used for encryption, the corresponding private key must be used for decryption\. AWS KMS uses only symmetric encryption\.

For a more detailed introduction to cryptography and AWS KMS, see the following topics\.


+ [How Symmetric Key Cryptography Works](crypto_overview.md)
+ [Authenticated Encryption](crypto_authen.md)
+ [Encryption Context](encryption-context.md)
+ [Reference: AWS KMS and Cryptography Terminology](crypto-terminology.md)