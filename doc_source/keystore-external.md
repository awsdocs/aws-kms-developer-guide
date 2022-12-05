# External key stores<a name="keystore-external"></a>

External key stores allow you to protect your AWS resources using cryptographic keys outside of AWS\. This advanced feature is designed for regulated workloads that you must protect with encryption keys stored in an external key management system that you control\.

An *external key store* is a [custom key store](custom-key-store-overview.md) backed by an *external key manager* that you own and manage outside of AWS\. Your external key manager can be a physical or virtual hardware security modules \(HSMs\), or any hardware\-based or software\-based system capable of generating and using cryptographic keys\. Encryption and decryption operations that use a KMS key in an external key store are performed by your external key manager using your cryptographic key material, a feature known as *hold your own keys* \(HYOKs\)\. 

AWS KMS never interacts directly with your external key manager, and cannot create, manage, or delete your keys\. Instead, AWS KMS interacts only with [external key store proxy](#concept-xks-proxy) \(XKS proxy\) software that you provide\. Your external key store proxy mediates all communication between AWS KMS and your external key manager\. It transmits all requests from AWS KMS to your external key manager, and transmits responses from your external key manager back to AWS KMS\. The external key store proxy also translates generic requests from AWS KMS into a vendor\-specific format that your external key manager can understand, allowing you to use external key stores with key managers from a variety of vendors\.

You can use KMS keys in an external key store for client\-side encryption, including with the [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/)\. But external key stores are an important resource for server\-side encryption, allowing you to protect your AWS resources with your cryptographic keys located outside of AWS\. Most AWS services that support [customer managed keys](concepts.md#customer-cmk) for symmetric encryption also support KMS keys in an external key store\. For a list of AWS services that do *not* support KMS keys in an external key store, see [AWS Service Integration](https://aws.amazon.com/kms/features/#AWS_Service_Integration)\.

External key stores allow you to use AWS KMS for regulated workloads where encryption keys must be stored and used outside of AWS\. But they are a major departure from the standard shared responsibility model, and require additional operational burdens\. The greater risk to availability and latency will, for most customers, exceed the perceived security benefits of external key stores\.

External key stores let you control the root of trust\. Data encrypted under KMS keys in your external key store can be decrypted only by using the external key manager that you control\. If you temporarily revoke access to your external key manager, such as by disconnecting the external key store or disconnecting your external key manager from the external key store proxy, AWS loses all access to your cryptographic keys until you restore it\. During that interval, ciphertext encrypted under your KMS keys can't be decrypted\. If you permanently revoke access to your external key manager, all ciphertext encrypted under a KMS key in your external key store becomes unrecoverable\. The only exceptions are AWS services that briefly cache the [data keys](concepts.md#data-keys) protected by your KMS keys\. These data keys continue to work until your deactivate the resource or the cache expires\. For details, see [How unusable KMS keys affect data keys](concepts.md#unusable-kms-keys)\.

External key stores unblock the few use cases for regulated workloads where encryption keys must remain solely under your control and inaccessible to AWS\. But this is a major change in the way you operate cloud\-based infrastructure and a signiﬁcant shift in the shared responsibility model\. For most workloads, the additional operational burden and greater risks to availability and performance will exceed the perceived security benefits of external key stores\.

**Learn more**:
+ [Announcing AWS KMS External Key Store](http://aws.amazon.com/blogs/aws/announcing-aws-kms-external-key-store-xks/) in the *AWS News Blog*\.



**Do I need an external key store?**

For most users, the default AWS KMS key store, which is protected by [FIPS 140\-2 validated cryptographic modules](https://csrc.nist.gov/projects/cryptographic-module-validation-program/Certificate/3139), fulfills their security, control, and regulatory requirements\. External key store users incur substantial cost, maintenance and troubleshooting burden, and risks to latency, availability and reliability\.

When considering an external key store, you should also take some time to understand the alternatives, including an [AWS CloudHSM key store](keystore-cloudhsm.md) backed by an AWS CloudHSM cluster that you own and manage, and KMS keys with [imported key material](importing-keys.md) that you generate in your own HSMs and can delete from KMS keys on demand\. 

An external key store might be the right solution for your organization if you have the following requirements:
+ You are required to use cryptographic keys in your on\-premises key manager or a key manager outside of AWS that you control\.
+ You must demonstrate that your cryptographic keys are retained solely under your control outside of the cloud\.
+ You must encrypt and decrypt using cryptographic keys with independent authorization\.
+ Key material must be subject to a secondary, independent audit path\.



**Shared responsibility model**

Standard KMS keys use key material that is generated and used in HSMs that AWS KMS owns and manages\. You establish the access control policies on your KMS keys and configure AWS services that use KMS keys to protect your resources\. AWS KMS assumes responsibility for the security, availability, latency, and durability of the key material in your KMS keys\.

KMS keys in external key stores rely on key material and operations in your external key manager\. As such, the balance of responsibility shifts in your direction\. You are responsible for the security, reliability, durability, and performance of the cryptographic keys in your external key manager\. AWS KMS is responsible for responding promptly to requests and communicating with your external key store proxy, and for maintaining our security standards\.To ensure that every external key store ciphertext at least as strong than standard AWS KMS ciphertext, AWS KMS first encrypts all plaintext with AWS KMS key material specific to your KMS key, and then sends it to your external key manager for encryption with your external key, a procedure known as [*double encryption*](#concept-double-encryption)\. As a result, neither AWS KMS nor the owner of the external key material can decrypt double\-encrypted ciphertext alone\.

You are responsible for maintaining an external key manager that meet your regulatory and performance standards, for supplying and maintaining an external key store proxy that conforms to the AWS KMS [AWS KMS External Key Store Proxy API Specification](https://github.com/aws/aws-kms-xksproxy-api-spec/), and for ensuring the availability and durability of your key material\. You must also create, configure, and maintain an external key store\. When errors arise that are caused by components that you maintain, you must be prepared to identify and resolve the errors so that AWS services can access your resources without undue disruption\. AWS KMS provides [troubleshooting guidance](xks-troubleshooting.md) to help you determine the cause of problems and the most likely resolutions\. 

Review the [Amazon CloudWatch metrics and dimensions](monitoring-cloudwatch.md#kms-metrics) that AWS KMS records for external key stores\. AWS KMS strongly recommends that you create CloudWatch alarms to monitor yourexternal key store so you can detect the early signs of performance and operational problems before they occur\.

**Where do I start?**

To create and manage an external key store, you need to [choose your external key store proxy connectivity option](plan-xks-keystore.md), [assemble the prerequisites](create-xks-keystore.md#xks-requirements), and [create and configure your external key store](create-xks-keystore.md)\. To begin, see [Planning an external key store](plan-xks-keystore.md)\.

**Quotas**

AWS KMS allows up to [10 custom key stores](resource-limits.md) in each AWS account and Region, including both [AWS CloudHSM key stores](https://docs.aws.amazon.com/kms/latest/developerguide/keystore-cloudhsm.html) and [external key stores](https://docs.aws.amazon.com/kms/latest/developerguide/keystore-external.html), regardless of their connection state\. In addition, there are AWS KMS request quotas on the [use of KMS keys in a custom key store](requests-per-second.md#rps-key-stores)\. 

If you choose [VPC proxy connectivity](#concept-xks-connectivity) for your external key store proxy, there might also be quotas on the required components, such as VPCs, subnets, and network load balancers\. For information about these quotas, use the [Service Quotas console](https://console.aws.amazon.com/servicequotas/home)\.



**Regions**

To minimize network latency, create your external key store components in the AWS Region closest to your [external key manager](#concept-ekm)\. If possible, choose a Region with a network round\-trip time \(RTT\) of 35 milliseconds or less\.

External key stores are supported in all AWS Regions in which AWS KMS is supported except for Asia Pacific \(Hyderabad\), China \(Beijing\), China \(Ningxia\), Europe \(Spain\), Europe \(Zurich\), Middle East \(UAE\), AWS GovCloud \(US\-East\), and AWS GovCloud \(US\-West\)\. 

**Unsupported features**

AWS KMS does not support the following features in custom key stores\.
+ [Asymmetric KMS keys](symmetric-asymmetric.md)
+ [Asymmetric data key pairs](concepts.md#data-key-pairs)
+ [HMAC KMS keys](hmac.md)
+ [KMS keys with imported key material](importing-keys.md)
+ [Automatic key rotation](rotate-keys.md)
+ [Multi\-Region keys](multi-region-keys-overview.md)

**Topics**
+ [External key store concepts](#xks-concepts)
+ [How external key stores work](#xks-how-it-works)
+ [Controlling access to your external key store](authorize-xks-key-store.md)
+ [Planning an external key store](plan-xks-keystore.md)
+ [Managing an external key store](keystore-external-manage.md)
+ [Managing KMS keys in an external key store](keystore-external-key-manage.md)
+ [Troubleshooting external key stores](xks-troubleshooting.md)

## External key store concepts<a name="xks-concepts"></a>

This topic explains some of the concepts used in external key stores\.

**Topics**
+ [External key store](#concept-external-key-store)
+ [External key manager](#concept-ekm)
+ [External key](#concept-external-key)
+ [External key store proxy](#concept-xks-proxy)
+ [External key store proxy connectivity](#concept-xks-connectivity)
+ [External key store proxy authentication credential](#concept-xks-credential)
+ [Proxy APIs](#concept-proxy-apis)
+ [Double encryption](#concept-double-encryption)

### External key store<a name="concept-external-key-store"></a>

An *external key store* is an AWS KMS [custom key store](custom-key-store-overview.md) backed by an external key manager outside of AWS that you own and manage\. Each KMS key in an external key store is associated with an [external key](#concept-external-key) in your external key manager\. When you use a KMS key in an external key store for encryption or decryption, the operation is performed in your external key manager using your external key, an arrangement known as *Hold your Own Keys* \(HYOK\)\. This feature is designed for organizations that are required to maintain their cryptographic keys in their own external key manager\.

External key stores ensure that the cryptographic keys and operations that protect your AWS resources remain in your external key manager under your control\. AWS KMS sends requests to your external key manager to encrypt and decrypt data, but AWS KMS cannot create, delete, or manage any external keys\. All requests from AWS KMS to your external key manager are mediated by an [external key store proxy](#concept-xks-proxy) software component that you supply, own, and manage\. 

AWS services that support AWS KMS [customer managed keys](concepts.md) can use the KMS keys in your external key store to protect your data\. As a result, your data is ultimately protected by your keys using your encryption operations in your external key manager\.

The KMS keys in an external key store have fundamentally different trust models, [shared responsibility arrangements](#xks-shared-responsibility), and performance expectations than standard KMS keys\. With external key stores, you are responsible for the security and integrity of the key material and the cryptographic operations\. The availability and latency of KMS keys in an external key store are affected by the hardware, software, networking components, and the distance between AWS KMS and your external key manager\. You are also likely to incur additional costs for your external key manager and for the networking and load balancing infrastructure you need for your external key manager to communicate with AWS KMS

You can use your external key store as part of your broader data protection strategy\. For each AWS resource that you protect, you can decide which require a KMS key in an external key store and which can be protected by a standard KMS key\. This gives you the flexibility to chose KMS keys for specific data classifications, applications, or projects\.

### External key manager<a name="concept-ekm"></a>

An *external key manager* is a component outside of AWS that can generate 256\-bit AES symmetric keys and perform symmetric encryption and decryption\. The external key manager for an external key store can be a physical hardware security module \(HSM\), a virtual HSM, or a software key manager with or without an HSM component\. It can be located anywhere outside of AWS, including on your premises, in a local or remote data center, or in any cloud\. Your external key store can be backed by a single external key manager or multiple related key manager instances that share cryptographic keys, such as an HSM cluster\. External key stores are designed to support a variety of external managers from different vendors\. For details about the requirements for your external key manager, see [Planning an external key store](plan-xks-keystore.md)\.

### External key<a name="concept-external-key"></a>

Each KMS key in an external key store is associated with a cryptographic key in your [external key manager](#concept-ekm) known as an *external key*\. When you encrypt or decrypt with a KMS key in your external key store, the cryptographic operation is performed in your [external key manager](#concept-ekm) using your external key\.

**Warning**  
The external key is essential to the operation of the KMS key\. If the external key is lost or deleted, ciphertext encrypted under the associated KMS key is unrecoverable\.

For external key stores, an external key must be a 256\-bit AES key that is enabled and can perform encryption and decryption\. For detailed external key requirements, see [Requirements for a KMS key in an external key store](create-xks-keys.md#xks-key-requirements)\.

AWS KMS cannot create, delete, or manage any external keys\. Your cryptographic key material never leaves your external key manager\.When you create a KMS key in an external key store, you provide the ID of an external key \(`XksKeyId`\)\. You cannot change the external key ID associated with a KMS key, although your external key manager can rotate the key material associated with the external key ID\.

In addition to your external key, a KMS key in an external key store also has AWS KMS key material\. Data protected by the KMS key is encrypted first by AWS KMS using the AWS KMS key material, and then by your external key manager using your external key\. This [double encryption](#concept-double-encryption) process ensures that ciphertext protected by your KMS key is always at least as strong as ciphertext protected only by AWS KMS\. 

Many cryptographic keys have different types of identifiers\. When creating a KMS key in an external key store, provide the ID of the external key that the [external key store proxy](#concept-xks-proxy) uses to refer to the external key\. If you use the wrong identifier, your attempt to create a KMS key in your external key store fails\.

### External key store proxy<a name="concept-xks-proxy"></a>

The *external key store proxy* \("XKS proxy"\) is a customer\-owned and customer\-managed software application that mediates all communication between AWS KMS and your external key manager\. It also translates generic AWS KMS requests into a format that your vendor\-specific external key manager understand\. An external key store proxy is required for an external key store\. Each external key store is associated with one external key store proxy\.

![\[External key store proxy\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/xks-proxy-concept-40.png)

AWS KMS cannot create, delete, or manage any external keys\. Your cryptographic key material never leaves your external key manager\. All communication between AWS KMS and your external key manager is mediated by your external key store proxy\. AWS KMS sends requests to the external key store proxy and receives responses from the external key store proxy\. The external key store proxy is responsible for transmitting requests from AWS KMS to your external key manager and transmitting responses from your external key manager back to AWS KMS

You own and manage the external key store proxy for your external key store, and you are responsible for its maintenance and operation\. You can develop your external key store proxy based on the open\-source [external key store proxy API specification](https://github.com/aws/aws-kms-xksproxy-api-spec/) that AWS KMS publishes or purchase a proxy application from a vendor\. Your external key store proxy might be included in your external key manager\. To support proxy development, AWS KMS also provides a sample external key store proxy \([aws\-kms\-xks\-proxy](https://github.com/aws-samples/aws-kms-xks-proxy)\) and a test client \([xks\-kms\-xksproxy\-test\-client](https://github.com/aws-samples/aws-kms-xksproxy-test-client)\) that verifies that your external key store proxy conforms to the specification\.

To authenticate to AWS KMS, the proxy uses server\-side TLS certificates\. To authenticate to your proxy, AWS KMS signs all requests to your external key store proxy with a SigV4 [proxy authentication credential](#concept-xks-credential)\. Optionally, your proxy can enable mutual TLS \(mTLS\) for additional assurance that it only accepts requests from AWS KMS\. 

To create and use the KMS keys in your external key store, you must first [connect the external key store](xks-connect-disconnect.md) to its external key store proxy\. You can also disconnect your external key store from its proxy on demand\. When you do, all KMS keys in the external key store become [unavailable](key-state.md); they cannot be used in any cryptographic operation\.

### External key store proxy connectivity<a name="concept-xks-connectivity"></a>

The external key store proxy connectivity \("XKS proxy connectivity"\) describes the method that AWS KMS uses to communicate with your external key store proxy\. 

You specify your proxy connectivity option when you create your external key store, and it becomes a property of the external key store\. You can change your proxy connectivity option by updating the custom key store property, but you must be certain that your external key store proxy can still access the same external keys\.

AWS KMS supports the following connectivity options:
+ [Public endpoint connectivity](plan-xks-keystore.md#xks-connectivity-public-endpoint) — AWS KMS sends requests for your external key store proxy over the internet to a public endpoint that you control\. This option is simple to create and maintain, but it might not fulfill the security requirements for every installation\.
+ [VPC endpoint service connectivity](plan-xks-keystore.md#xks-vpc-connectivity) — AWS KMS sends requests to a Amazon Virtual Private Cloud \(Amazon VPC\) endpoint service that you create and maintain\. You can host your external key store proxy inside your Amazon VPC, or host your external key store proxy outside of AWS and use the Amazon VPC only for communication\. 

For details about the external key store proxy connectivity options, see [Choosing a proxy connectivity option](plan-xks-keystore.md#choose-xks-connectivity)\.

### External key store proxy authentication credential<a name="concept-xks-credential"></a>

To authenticate to your external key store proxy, AWS KMS signs all requests to your external key store proxy with a [Signature V4 \(SigV4\)](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) authentication credential\. You establish and maintain the authentication credential on your proxy, then provide this credential to AWS KMS when you create your external store\. 

**Note**  
The SigV4 credential that AWS KMS uses to sign requests to the XKS proxy is unrelated to any SigV4 credentials associated with AWS Identity and Access Management principals in your AWS accounts\. Do not reuse any IAM SigV4 credentials for your external key store proxy\.

Each proxy authentication credential has two parts\. You must provide both parts when creating an external key store or updating the authentication credential for your external key store\.
+ Access key ID: Identifies the secret access key\. You can provide this ID in plaintext\.
+ Secret access key: The secret part of the credential\. AWS KMS encrypts the secret access key in the credential before storing it\.

You can [edit the credential setting](update-xks-keystore.md) at any time, such as when you enter incorrect values, when you change your credential on the proxy, or when your proxy rotates the credential\. For technical details about AWS KMS authentication to the external key store proxy, see [Authentication](https://github.com/aws/aws-kms-xksproxy-api-spec/blob/main/xks_proxy_api_spec.md#authentication) in the AWS KMS External Key Store Proxy API Specification\.

To allow you to rotate your credential without disrupting the AWS services that use KMS keys in your external key store, we recommend that the external key store proxy support at least two valid authentication credentials for AWS KMS\. This ensures that your previous credential continues to work while you provide your new credential to AWS KMS\.

To help you track the age of your proxy authentication credential, AWS KMS defines an Amazon CloudWatch metric, [XksProxyCredentialAge](monitoring-cloudwatch.md#metric-xks-proxy-credential-age)\. You can use this metric to create a CloudWatch alarm that notifies you when the age of your credential reaches a threshold you establish\.

To provide additional assurance that your external key store proxy responds only to AWS KMS, some external key proxies support mutual Transport Layer Security \(mTLS\)\. For details, see [mTLS authentication \(optional\)](authorize-xks-key-store.md#xks-mtls)\.

### Proxy APIs<a name="concept-proxy-apis"></a>

To support an AWS KMS external key store, an [external key store proxy](#concept-xks-proxy) must implement the required proxy APIs as described in the [AWS KMS External Key Store Proxy API Specification](https://github.com/aws/aws-kms-xksproxy-api-spec/)\. These proxy API requests are the only requests that AWS KMS sends to the proxy\. Although you never send these requests directly, knowing about them might help you fix any issues that might arise with your external key store or its proxy\. For example, AWS KMS includes information about the latency and success rates of these API calls in its [Amazon CloudWatch metrics](monitoring-cloudwatch.md) for external key stores\. For details, see [Monitoring an external key store](xks-monitoring.md)\.

The following table lists and describes each of the proxy APIs\. It also includes the AWS KMS operations that trigger a call to the proxy API and any AWS KMS operation exceptions related to the proxy API\.


| Proxy API | Description | Related AWS KMS operations | 
| --- | --- | --- | 
| Decrypt | AWS KMS sends the ciphertext to be decrypted, and the ID of the [external key](#concept-external-key) to use\. The required encryption algorithm is AES\_GCM\.  | [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) | 
| Encrypt | AWS KMS sends data to be encrypted, and the ID of the [external key](#concept-external-key) to use\. The required encryption algorithm is AES\_GCM\.  | [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) | 
| GetHealthStatus | AWS KMS requests information about the status of the proxy and your external key manager\. The status of each external key manager can be one of the following\.[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kms/latest/developerguide/keystore-external.html) | [CreateCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateCustomKeyStore.html) \(for [public endpoint connectivity](plan-xks-keystore.md#xks-connectivity-public-endpoint)\), [ConnectCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_ConnectCustomKeyStore.html) \(for [VPC endpoint service connectivity](plan-xks-keystore.md#xks-vpc-connectivity)\)If all external key manager instances are `Unavailable`, attempts to create or connect the key store fail with [`XksProxyUriUnreachableException`](xks-troubleshooting.md#fix-xks-latency)\. | 
| GetKeyMetadata | AWS KMS requests information about the [external key](#concept-external-key) associated with a KMS key in your external key store\. The response includes the key spec \(`AES_256`\), the key usage \(`[ENCRYPT, DECRYPT]`\), and the whether the external key is `ENABLED` or `DISABLED`\. | [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html)If the key spec is not `AES_256`, or the key usage is not `[ENCRYPT, DECRYPT]`, or the status is `DISABLED`, the `CreateKey` operation fails with `XksKeyInvalidConfigurationException`\. | 

### Double encryption<a name="concept-double-encryption"></a>

Data encrypted by a KMS key in an external key store is encrypted twice\. First, AWS KMS encrypts the data with AWS KMS key material specific to the KMS key\. Then the AWS KMS\-encrypted ciphertext is encrypted by your [external key manager](#concept-ekm) using your [external key](#concept-external-key)\. This process is known as *double encryption*\.

Double encryption ensures that data encrypted by a KMS key in an external key store is at least as strong as ciphertext encrypted by a standard KMS key\. It also protects your plaintext in transit from AWS KMS to your external key store proxy\. With double encryption, you retain full control of your ciphertexts\. If you permanently revoke AWS access to your external key through your external proxy, any ciphertext remaining in AWS is effectively crypto\-shredded\.

![\[Double encryption of data protected by a KMS key in an external key store\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/xks-double-encrypt-40.png)

To enable double encryption, each KMS key in an external key store has *two* cryptographic backing keys:
+ An AWS KMS key material unique to the KMS key\. This key material is generated and used in AWS KMS FIPS 140\-2 level\-2 certified hardware security managers \(HSMs\)\.
+ An [external key](#concept-external-key) in your external key manager\.

Double encryption has the following effects:
+ AWS KMS cannot decrypt any ciphertext encrypted by a KMS key in an external key store without access to your external keys via your external key store proxy\.
+ You cannot decrypt any ciphertext encrypted by a KMS key in an external key store outside of AWS, even if you have its external key material\.
+ You cannot recreate a KMS key that was deleted from an external key store, even if you have its external key material\. Each KMS key has unique metadata that it includes in the symmetric ciphertext\. A new KMS key would not be able to decrypt ciphertext encrypted by the original key, even if it used the same external key material\.

For an example of double encryption in practice, see [How external key stores work](#xks-how-it-works)\.

## How external key stores work<a name="xks-how-it-works"></a>

Your [external key store](#concept-external-key-store), [external key store proxy,](#concept-xks-proxy) and [external key manager](#concept-ekm) work together to protect your AWS resources\. The following procedure depicts the encryption workflow of a typical AWS service that encrypts each object under a unique data key protected by a KMS key\. In this case, you've chosen a KMS key in an external key store to protect the object\. The example shows how AWS KMS uses [double encryption](#concept-double-encryption) to protect the data key in transit and ensure that ciphertext generated by a KMS key in an external key store is always at least as strong as ciphertext encrypted by a standard symmetric KMS key with key material in AWS KMS\.

The encryption methods used by each actual AWS service that integrates with AWS KMS vary\. For details, see the "Data protection" topic in the Security chapter of the AWS service documentation\.

![\[How external key stores work\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/xks-how-it-works-60.png)

1. You add a new object to your AWS service resource\. To encrypt the object, the AWS service sends a [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) request to AWS KMS using a KMS key in your external key store\.

1. AWS KMS generates a 256\-bit symmetric [data key](concepts.md#data-keys) and prepares to send a copy of the plaintext data key to your external key manager via your external key store proxy\. AWS KMS begins the [double encryption](#concept-double-encryption) process by encrypting the plaintext data key with the [AWS KMS key material](#concept-double-encryption) associated with the KMS key in the external key store\. 

1. AWS KMS sends an [encrypt](#concept-proxy-apis) request to the external key store proxy associated with the external key store\. The request includes the data key ciphertext to be encrypted and the ID of the [external key](#concept-external-key) that is associated with the KMS key\. AWS KMS signs the request using the [proxy authentication credential](#concept-xks-credential) for your external key store proxy\. 

   The plaintext copy of the data key remains within AWS KMS\.

1. The external key store proxy authenticates the request, and then passes the encrypt request to your external key manager\. 

   Some external key store proxies also implement an optional [authorization policy](authorize-xks-key-store.md#xks-proxy-authorization) that allows only selected principals to perform operations under specific conditions\.

1. Your external key manager encrypts the data key ciphertext using the specified external key\. The external key manager returns the double\-encrypted data key to your external key store proxy, which returns it to AWS KMS\.

1. AWS KMS returns the plaintext data key and the double\-encrypted copy of that data key to the AWS service\. 

1. The AWS service uses the plaintext data key to encrypt the resource object, destroys the plaintext data key, and stores the encrypted data key with the encrypted object\. 

   Some AWS services might cache the plaintext data key to use for multiple objects, or to reuse while the resource is in use\. For details, see [How unusable KMS keys affect data keys](concepts.md#unusable-kms-keys)\.

To decrypt the encrypted object, the AWS service must send the encrypted data key back to AWS KMS in a [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) request\. To decrypt the encrypted data key, AWS KMS must send the encrypted data key back to your external key store proxy with the ID of the external key\. If the decrypt request to the external key store proxy fails for any reason, AWS KMS cannot decrypt the encrypted data key, and the AWS service cannot decrypt the encrypted object\.