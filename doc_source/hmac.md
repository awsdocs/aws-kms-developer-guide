# HMAC keys in AWS KMS<a name="hmac"></a>

Hash\-Based Message Authentication Code \(HMAC\) KMS keys are symmetric keys that you use to generate and verify HMACs within AWS KMS\. The unique key material associated with each HMAC KMS key provides the secret key that HMAC algorithms require\. You can use an HMAC KMS key with the `[GenerateMac](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateMac.html)` and [https://docs.aws.amazon.com/kms/latest/APIReference/API_VerifyMac.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_VerifyMac.html) operations to verify the integrity and authenticity of data within AWS KMS\.

HMAC algorithms combine a cryptographic hash function and a shared secret key\. They take a message and a secret key, such as the key material in an HMAC KMS key, and return a unique, fixed\-size code or *tag*\. If even one character of the message changes, or if the secret key is not identical, the resulting tag is entirely different\. By requiring a secret key, HMAC also provides authenticity; it is impossible to generate an identical HMAC tag without the secret key\. HMACs are sometimes called *symmetric signatures*, because they work like digital signatures, but use a single key for both signing and verification\.

HMAC KMS keys and the HMAC algorithms that AWS KMS uses conform to industry standards defined in [RFC 2104](https://datatracker.ietf.org/doc/html/rfc2104)\. The AWS KMS [GenerateMac](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateMac.html) operation generates standard HMAC tags\. HMAC KMS keys are generated in AWS KMS hardware security modules that are certified under the [FIPS 140\-2 Cryptographic Module Validation Program](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/4177) \(except in China \(Beijing\) and China \(Ningxia\) Regions\) and never leave AWS KMS unencrypted\. To use an HMAC KMS key, you must call AWS KMS\.

You can use HMAC KMS keys to determine the authenticity of a message, such as a JSON Web Token \(JWT\), tokenized credit card information, or a submitted password\. They can also be used as secure Key Derivation Functions \(KDFs\), especially in applications that require deterministic keys\.

HMAC KMS keys provide an advantage over HMACs from application software because the key material is generated and used entirely within AWS KMS, subject to the access controls that you set on the key\.

**Tip**  
Best practices recommend that you limit the time during which any signing mechanism, including an HMAC, is effective\. This deters an attack where the actor uses a signed message to establish validity repeatedly or long after the message is superseded\. HMAC tags do not include a timestamp, but you can include a timestamp in the token or message to help you detect when its time to refresh the HMAC\. 

You can create, manage, and use the HMAC KMS keys in your AWS account\. This includes [enabling and disabling keys](enabling-keys.md), setting and changing [aliases](kms-alias.md) and [tags](tagging-keys.md), and [scheduling deletion](deleting-keys.md) of HMAC KMS keys\. You can also control access to HMAC KMS keys using [key policies](hmac-authz.md), [IAM policies](iam-policies.md), and [grants](grants.md)\. You can audit all operations that use or manage your HMAC KMS keys within AWS in [AWS CloudTrail logs](logging-using-cloudtrail.md)\. You can also create HMAC [multi\-Region KMS keys](multi-region-keys-overview.md) that behave like copies of the same HMAC KMS key in multiple AWS Regions\. 

HMAC KMS keys support only the [https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateMac.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateMac.html) and [https://docs.aws.amazon.com/kms/latest/APIReference/API_VerifyMac.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_VerifyMac.html) cryptographic operations\. You cannot use HMAC KMS keys to encrypt data or sign messages, or use any other type of KMS key in HMAC operations\. When you use the `GenerateMac` operation, you supply a message of up to 4,096 bytes, an HMAC KMS key, and the MAC algorithm that is compatible with the HMAC key spec, and `GenerateMac` computes the HMAC tag\. To verify an HMAC tag, you must supply the HMAC tag, and the same message, HMAC KMS key, and MAC algorithm that `GenerateMac` used to compute the original HMAC tag\. The `VerifyMac` operation computes the HMAC tag and verifies that it is identical to the supplied HMAC tag\. If the input and computed HMAC tags are not identical, verification fails\. 

HMAC KMS keys *do not* support [automatic key rotation](rotate-keys.md) or [imported key material](importing-keys.md), and you cannot create an HMAC KMS key in a [custom key store](custom-key-store-overview.md)\.

If you are creating a KMS key to encrypt data in an AWS service, use a symmetric encryption key\. You cannot use an HMAC KMS key\. 

**Regions**

HMAC KMS keys are supported in all AWS Regions that AWS KMS supports except for China \(Beijing\) Region and China \(Ningxia\) Region\.

**Learn more**
+ For help with choosing a type of KMS key, see [Choosing a KMS key type](key-types.md#symm-asymm-choose)\.
+ For a table that compares the AWS KMS API operations supported by each type of KMS key, see [Key type reference](symm-asymm-compare.md)\.
+ For information about creating multi\-Region HMAC KMS keys, see [Multi\-Region keys in AWS KMS](multi-region-keys-overview.md)\.
+ To examine the difference in the default key policy that the AWS KMS console sets for HMAC KMS keys, see [Allows key users to use the KMS key with AWS services](key-policy-default.md#key-policy-service-integration)\.
+ For information about pricing of HMAC KMS keys, see [AWS Key Management Service pricing](https://aws.amazon.com/kms/pricing/)\.
+ For information about quotas that apply to HMAC KMS keys, see [Resource quotas](resource-limits.md) and [Request quotas](requests-per-second.md)\.
+ For information about deleting HMAC KMS keys, see [Deleting AWS KMS keys](deleting-keys.md)\.
+ To learn about using HMACs to create JSON web tokens, see [How to protect HMACs inside AWS KMS](http://aws.amazon.com/blogs/security/how-to-protect-hmacs-inside-aws-kms/) in the *AWS Security Blog*\.
+ Listen to a podcast: [Introducing HMACs for AWS Key Management Service](https://aws.amazon.com/podcasts/introducing-hmacs-apis-in-aws-key-management-service) on *The Official AWS Podcast*\.

**Topics**
+ [Key specs for HMAC KMS keys](#hmac-key-specs)
+ [Creating HMAC KMS keys](hmac-create-key.md)
+ [Controlling access to HMAC KMS keys](hmac-authz.md)
+ [Viewing HMAC KMS keys](hmac-view.md)

## Key specs for HMAC KMS keys<a name="hmac-key-specs"></a>

AWS KMS supports symmetric HMAC keys in varying lengths\. The key spec that you select can depend on your security, regulatory, or business requirements\. The length of the key determines the MAC algorithm that is used in [GenerateMac](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateMac.html) and [VerifyMac](https://docs.aws.amazon.com/kms/latest/APIReference/API_VerifyMac.html) operations\. In general, longer keys are more secure\. Use the longest key that is practical for your use case\.


| HMAC key spec | MAC algorithm | 
| --- | --- | 
| HMAC\_224 | HMAC\_SHA\_224 | 
| HMAC\_256 | HMAC\_SHA\_256 | 
| HMAC\_384 | HMAC\_SHA\_384 | 
| HMAC\_512 | HMAC\_SHA\_512 | 