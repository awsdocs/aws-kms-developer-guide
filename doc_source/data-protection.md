# Data protection in AWS Key Management Service<a name="data-protection"></a>

AWS Key Management Service stores and protects your encryption keys to make them highly available while providing you with strong and flexible access control\.

**Topics**
+ [Data encryption](#data-encryption)
+ [Internetwork traffic privacy](#inter-network-privacy)

## Data encryption<a name="data-encryption"></a>

The data in AWS KMS consists of [AWS KMS keys](concepts.md#kms_keys) and the encryption key material they represent\. This key material exists in plaintext only within AWS KMS hardware security modules \(HSMs\) and only when in use\. Otherwise, the key material is encrypted and stored in durable persistent storage\. 

The key material that AWS KMS generates for KMS keys never leaves the boundary of AWS KMS HSMs unencrypted\. It is not exported or transmitted in any AWS KMS API operations\.

**Topics**
+ [Encryption at rest](#encryption-at-rest)
+ [Encryption in transit](#encryption-in-transit)
+ [Key management](#encryption-key-mgmt)

### Encryption at rest<a name="encryption-at-rest"></a>

AWS KMS generates key material for AWS KMS keys in FIPS 140\-2 Level 2–compliant hardware security modules \(HSMs\)\. When not in use, key material is encrypted by an HSM and written to durable, persistent storage\. The key material for KMS keys and the encryption keys that protect the key material never leave the HSMs in plaintext form\. 

Encryption and management of key material for KMS keys is handled entirely by AWS KMS\.

For more details, see [Working with AWS KMS keys](https://docs.aws.amazon.com/kms/latest/cryptographic-details/kms-keys.html) in AWS Key Management Service Cryptographic Details\.

### Encryption in transit<a name="encryption-in-transit"></a>

Key material that AWS KMS generates for KMS keys is never exported or transmitted in AWS KMS API operations\. AWS KMS uses [key identifiers](concepts.md#key-id) to represent the KMS keys in API operations\. Similarly, key material for KMS keys in AWS KMS [custom key stores](custom-key-store-overview.md) is non\-exportable and never transmitted in AWS KMS or AWS CloudHSM API operations\.

However, some AWS KMS API operations return [data keys](concepts.md#data-keys)\. Also, customers can use API operations to [import key material](importing-keys.md) for selected KMS keys\. 

All AWS KMS API calls must be signed and be transmitted using Transport Layer Security \(TLS\)\. AWS KMS supports TLS 1\.0—1\.3 and hybrid post\-quantum TLS for AWS KMS service endpoints in all regions, except AWS GovCloud \(US\) and China Regions\. The AWS GovCloud \(US\) region only supports TLS 1\.0—1\.2 for AWS KMS service endpoints\. AWS KMS does not support hybrid post\-quantum TLS for endpoints in AWS GovCloud \(US\)\. AWS KMS recommends you always use the latest supported TLS version\. Calls to AWS KMS also require a modern cipher suite that supports *perfect forward secrecy*, which means that compromise of any secret, such as a private key, does not also compromise the session key\.

If you require FIPS 140\-2 validated cryptographic modules when accessing AWS through a command line interface or an API, use a FIPS endpoint\. AWS KMS supports version 1\.2 of Transport Layer Security \(TLS\) for FIPS endpoints for select regions\. For more information about the available FIPS endpoints, see [Federal Information Processing Standard \(FIPS\) 140\-2](http://aws.amazon.com/compliance/fips/)\. For a list of AWS KMS FIPS endpoints, see [AWS Key Management Service endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/kms.html) in the AWS General Reference\.

Communications between AWS KMS service hosts and HSMs are protected using Elliptic Curve Cryptography \(ECC\) and Advanced Encryption Standard \(AES\) in an authenticated encryption scheme\. For more details, see [Internal communication security](https://docs.aws.amazon.com/kms/latest/cryptographic-details/internal-communication-security.html) in AWS Key Management Service Cryptographic Details\.

### Key management<a name="encryption-key-mgmt"></a>

AWS KMS does not directly store customer data\. Instead, AWS KMS is responsible for storing and protecting AWS KMS keys, which are logical entities backed by encryption key material\.

Key material for KMS keys is supported by a distributed fleet of FIPS 140\-2 Level\-2–validated hardware security modules \(HSMs\)\. Each AWS KMS HSM is a standalone cryptographic hardware appliance designed to provide dedicated cryptographic functions to meet the security and scalability requirements of AWS KMS\.

The key material for KMS keys exists in plaintext only inside the HSMs and only when the key material is generated or being used in a cryptographic operation\.

When not in use, key material is encrypted on the HSMs and the encrypted key material is written to durable, low\-latency persistent storage\. The encryption keys that protect the key material never leave the HSMs in plaintext form\. There are no mechanisms for anyone, including AWS service operators, to export or view the key material or HSM encryption keys in plaintext\.

[Custom key stores](custom-key-store-overview.md), an optional AWS KMS feature, lets you create KMS keys backed by key material generated in AWS CloudHSM hardware security modules that you control\. These HSMs are certified at [FIPS 140\-2 Level 3](https://docs.aws.amazon.com/cloudhsm/latest/userguide/compliance.html)\. 

Another optional feature lets you [import the key material](importing-keys.md) for a KMS key\. During transport from its source to AWS KMS, the imported key material must be encrypted using RSA key pairs generated in AWS KMS HSMs\. The imported key material is decrypted on an AWS KMS HSM and re\-encrypted under symmetric keys in the HSM\. These operations are performed before the imported key material is stored with key material generated by AWS KMS\. Once it is imported, the imported key material never leaves the HSMs unencrypted\. The customer who provided the key material is responsible for secure use, durability, and maintenance of the key material outside of AWS KMS\.

For details about the management of KMS keys and key material, see [AWS Key Management Service Cryptographic Details](https://docs.aws.amazon.com/kms/latest/cryptographic-details/)

## Internetwork traffic privacy<a name="inter-network-privacy"></a>

AWS KMS supports an AWS Management Console and a set of API operations that enable you to create and manage AWS KMS keys and use them in cryptographic operations\.

AWS KMS supports two network connectivity options from your private network to AWS\.
+ An IPSec VPN connection over the internet
+ [AWS Direct Connect](https://aws.amazon.com/directconnect/), which links your internal network to an AWS Direct Connect location over a standard Ethernet fiber\-optic cable\.

All AWS KMS API calls must be signed and be transmitted using Transport Layer Security \(TLS\)\. The calls also require a modern cipher suite that supports [perfect forward secrecy](https://en.wikipedia.org/wiki/Forward_secrecy)\. Traffic to the hardware security modules \(HSMs\) that store key material for KMS keys is permitted only from known AWS KMS API hosts over the AWS internal network\.

To connect directly to AWS KMS from your virtual private cloud \(VPC\) without sending traffic over the public internet, use VPC endpoints, powered by AWS PrivateLink\. For more information, see [Connecting to AWS KMS through a VPC endpoint](kms-vpc-endpoint.md)\.

AWS KMS also supports a [hybrid post\-quantum key exchange](pqtls.md) option for the Transport Layer Security \(TLS\) network encryption protocol\. You can use this option with TLS when you connect to AWS KMS API endpoints\.