# Using multi\-Region keys<a name="multi-region-keys-overview"></a>

AWS KMS supports *multi\-Region keys*, which are AWS KMS keys in different AWS Regions that can be used interchangeably – as though you had the same key in multiple Regions\. Each set of *related* multi\-Region keys has the same [key material](concepts.md#key-material) and [key ID](concepts.md#key-id-key-id), so you can encrypt data in one AWS Region and decrypt it in a different AWS Region without re\-encrypting or making a cross\-Region call to AWS KMS\. 

Like all KMS keys, multi\-Region keys never leave AWS KMS unencrypted\. You can create symmetric or asymmetric multi\-Region keys for encryption or signing, and create [multi\-Region keys with imported key material](multi-region-keys-import.md) or key material that AWS KMS generates\. You must [manage each multi\-Region key](multi-region-keys-manage.md) independently, including creating aliases and tags, setting their key policies and grants, and enabling and disabling them selectively\. You can use multi\-Region keys in all cryptographic operations that you can do with single\-Region keys\.

Multi\-Region keys are a flexible and powerful solution for many common data security scenarios\.

Disaster recovery   
In a backup and recovery architecture, multi\-Region keys let you process encrypted data without interruption even in the event of an AWS Region outage\. Data maintained in backup Regions can be decrypted in the backup Region, and data newly encrypted in the backup Region can be decrypted in the primary Region when that Region is restored\.

Global data management  
Businesses that operate globally need globally distributed data that is available consistently across AWS Regions\. You can create multi\-Region keys in all Regions where your data resides, then use the keys as though they were a single\-Region key without the latency of a cross\-Region call or the cost of re\-encrypting data under a different key in each Region\.

Distributed signing applications  
Applications that require cross\-Region signature capabilities can use multi\-Region asymmetric signing keys to generate identical digital signatures consistently and repeatedly in different AWS Regions\.   
If you use certificate chaining with a single global trust store \(for a single root certification authority \(CA\), and Regional intermediate CAs signed by the root CA, you don't need multi\-Region keys\. However, if your system doesn't support intermediate CAs, such as application signing, you can use multi\-Region keys to bring consistency to Regional certifications\.

Active\-active applications that span multiple Regions  
Some workloads and applications can span multiple Regions in active\-active architectures\. For these applications, multi\-Region keys can reduce complexity by providing the same key material for concurrent encrypt and decrypt operations on data that might be moving across Region boundaries\.

You can use multi\-Region keys with client\-side encryption libraries, such as the [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/), the [DynamoDB Encryption Client](https://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/), and [Amazon S3 client\-side encryption](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/UsingClientSideEncryption.html)\. For an example of using multi\-Region keys with Amazon DynamoDB global tables and the DynamoDB Encryption Client, see [Encrypt global data client\-side with AWS KMS multi\-Region keys](http://aws.amazon.com/blogs/security/encrypt-global-data-client-side-with-aws-kms-multi-region-keys/) in the AWS Security Blog\.

[AWS services that integrate with AWS KMS](https://aws.amazon.com/kms/features/) for encryption at rest or digital signatures currently treat multi\-Region keys as though they were single\-Region keys\. They might re\-wrap or re\-encrypt data moved between Regions\. For example, Amazon S3 cross\-region replication decrypts and re\-encrypts data under a KMS key in the destination Region, even when replicating objects protected by a multi\-Region key\.

Multi\-Region keys are not global\. You create a multi\-Region primary key and then replicate it into Regions that you select within an [AWS partition](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html)\. Then you manage the multi\-Region key in each Region independently\. Neither AWS nor AWS KMS ever automatically creates or replicates multi\-Region keys into any Region on your behalf\. [AWS managed keys](concepts.md#aws-managed-cmk), the KMS keys that AWS services create in your account for you, are always single\-Region keys\.

You cannot convert an existing single\-Region key to a multi\-Region key\. This design ensures that all data protected with existing single\-Region keys maintain the same data residency and data sovereignty properties\.

For most data security needs, the Regional isolation and fault tolerance of Regional resources make standard AWS KMS single\-Region keys a best\-fit solution\. However, when you need to encrypt or sign data in client\-side applications across multiple Regions, multi\-Region keys might be the solution\.

**Regions**

Multi\-Region keys are supported in all AWS Regions that AWS KMS supports except for China \(Beijing\) and China \(Ningxia\)\.

**Pricing and quotas**

Every key in a set of related multi\-Region keys counts as one KMS key for pricing and quotas\. [AWS KMS quotas](limits.md) are calculated separately for each Region of an account\. Use and management of the multi\-Region keys in each Region count toward the quotas for that Region\.

**Topics**
+ [Controlling access to multi\-Region keys](multi-region-keys-auth.md)
+ [Creating multi\-Region keys](multi-region-keys-create.md)
+ [Viewing multi\-Region keys](multi-region-keys-view.md)
+ [Managing multi\-Region keys](multi-region-keys-manage.md)
+ [Importing key material into multi\-Region keys](multi-region-keys-import.md)
+ [Deleting multi\-Region keys](multi-region-keys-delete.md)

## Security considerations for multi\-Region keys<a name="mrk-when-to-use"></a>

Use an AWS KMS multi\-Region key only when you need one\. Multi\-Region keys provide a flexible and scalable solution for workloads that move encrypted data between AWS Regions or need cross\-Region access\. Consider a multi\-Region key if you must share, move, or back up protected data across Regions or need to create identical digital signatures of applications operating in different Regions\.

However, the process of creating a multi\-Region key moves your key material across AWS Region boundaries within AWS KMS\. The ciphertext generated by a multi\-Region key can potentially be decrypted by multiple related keys in multiple geographic locations\. There are also significant benefits to Regionally\-isolated services and resources\. Each AWS Region is isolated and independent of the other Regions\. Regions provide fault tolerance, stability, and resilience, and can also reduce latency\. They enable you to create redundant resources that remain available and unaffected by an outage in another Region\. In AWS KMS, they also ensure that every ciphertext can be decrypted by only one key\.

Multi\-Region keys also raise new security considerations:
+ Controlling access and enforcing data security policy is more complex with multi\-Region keys\. You need to ensure that policy is audited consistently on key across multiple, isolated regions\. And you need to use policy to enforce boundaries, instead of relying on separate keys\.

  For example, you need to set policy conditions on data to prevent payroll teams in one Region from being able to read payroll data for a different Region\. Also, you must use access control to prevent a scenario where a multi\-Region key in one Region protects one tenant's data and a related multi\-Region key in another Region protects a different tenant's data\.
+ Auditing keys across Regions is also more complex\. With multi\-Region keys, you need to examine and reconcile audit activities across multiple Regions to gain a complete understanding of key activities on protected data\.
+ Compliance with data residency mandates can be more complex\. With isolated Regions, you can ensure data residency and data sovereignty compliance\. KMS keys in a given Region can decrypt sensitive data only in that Region\. Data encrypted in one Region can remain completely protected and inaccessible in any other Region\.

  To verify data residency and data sovereignty with multi\-Region keys, you need to implement access policies and compile AWS CloudTrail events across multiple Regions\.

To make it easier for you to manage access control on multi\-Region keys, the permission to replicate a multi\-Region key \([kms:ReplicateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReplicateKey.html)\) is separate from the standard permission to create keys \([kms:CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html)\)\. Also, AWS KMS supports several policy conditions for multi\-Region keys, including `kms:MultiRegion`, which allows or denies permission to create, use, or manage multi\-Region keys and `kms:ReplicaRegion`, which restricts the Regions into which a multi\-Region key can be replicated\. For details, see [Controlling access to multi\-Region keys](multi-region-keys-auth.md)\.

## How multi\-Region keys work<a name="mrk-how-it-works"></a>

You begin by creating a symmetric or asymmetric [multi\-Region primary key](#mrk-primary-key) in an AWS Region that AWS KMS supports, such as US East \(N\. Virginia\)\. You decide whether a key is single\-Region or multi\-Region only when you create it; you can't change this property later\. As with any KMS key, you set a key policy for the multi\-Region key, and you can create grants, and add aliases and tags for categorization and authorization\. \(These are [independent properties](#mrk-sync-properties) that aren't shared or synchronized with other keys\.\) You can use your multi\-Region primary key in cryptographic operations for encryption or signing\.

You can [create a multi\-Region primary key](create-primary-keys.md) in the AWS KMS console or by using the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) API with the `MultiRegion` parameter set to `true`\. Notice that multi\-Region keys have a distinctive key ID that begins with `mrk-`\. You can use the `mrk-` prefix to identify MRKs programmatically\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/multi-region-primary-key.png)

If you choose, you can [replicate](#replicate) the multi\-Region primary key into one or more different AWS Regions in the same [AWS partition](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html), such as Europe \(Ireland\)\. When you do, AWS KMS creates a [replica key](#mrk-replica-key) in the specified Region with the same key ID and other [shared properties](#mrk-sync-properties) as the primary key\. Then it securely transports the key material across the Region boundary and associates it with the new KMS key in the destination Region, all within AWS KMS\. The result is two *related* multi\-Region keys — a primary key and a replica key — that can be used interchangeably\.

You can [create a multi\-Region replica key](multi-region-keys-replicate.md) in the AWS KMS console or by using the [ReplicateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReplicateKey.html) API\. 

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/multi-region-replica-key.png)

The resulting [multi\-Region replica key](#mrk-replica-key) is a fully\-functional KMS key with the same [shared properties](#mrk-sync-properties) as the primary key\. In all other respects, it is an independent KMS key with its own description, key policy, grants, aliases, and tags\. Enabling or disabling a multi\-Region key has no effect on related multi\-Region keys\. You can use the primary and replica keys independently in cryptographic operations or coordinate their use\. For example, you can encrypt data with the primary key in the US East \(N\. Virginia\) Region, move the data to the Europe \(Ireland\) Region and use the replica key to decrypt the data\. 

Related multi\-Region keys have the same key ID\. Their key ARNs \(Amazon Resource Names\) differ only in the Region field\. For example, the multi\-Region primary key and replica keys might have the following example key ARNs\. The key ID – the last element in the key ARN – is identical\. Both keys have the distinctive key ID of multi\-Region keys, which begins with *mrk\-*\.

```
Primary key: arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef12345678990ab
Replica key: arn:aws:kms:eu-west-1:111122223333:key/mrk-1234abcd12ab34cd56ef12345678990ab
```

Having the same key ID is required for interoperability\. When encrypting, AWS KMS binds the key ID of the KMS key to the ciphertext so the ciphertext can be decrypted only with that KMS key or a KMS key with the same key ID\. This feature also makes related multi\-Region keys easy to recognize, and it makes it easier to use them interchangeably\. For example, when using them in an application, you can refer to related multi\-Region keys by their shared key ID\. Then, if necessary, specify the Region or ARN to distinguish them\. 

As your data needs change, you can replicate the primary key to other AWS Regions in the same partition, such as US West \(Oregon\) and Asia Pacific \(Sydney\)\. The result is four *related* multi\-Region keys with the same key material and key IDs, as shown in the following diagram\. You manage the keys independently\. You can use them independently or in a coordinated fashion\. For example, you can encrypt data with the replica key in Asia Pacific \(Sydney\), move the data to US West \(Oregon\), and decrypt it with the replica key in US West \(Oregon\)\. 

![\[The primary and replica keys in a multi-region key\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/multi-region-keys.png)

Other considerations for multi\-Region keys include the following\.

*Synchronizing shared properties* — If a [shared property](#mrk-sync-properties) of the multi\-Region keys changes, AWS KMS automatically synchronizes the change from the [primary key](#mrk-primary-key) to all of its [replica keys](#mrk-replica-key)\. You cannot request or force a synchronization of shared properties\. AWS KMS detects and synchronizes all changes for you\. However, you can audit synchronization by using the [SynchronizeMultiRegionKey](ct-synchronize-multi-region-key.md) event in CloudTrail logs\.

For example, if you enable automatic key rotation on a symmetric multi\-Region primary key, AWS KMS copies that setting to all of its replica keys\. When the key material is rotated, the rotation is synchronized among all of the related multi\-Region keys, so they continue to have the same current key material, and access to all older versions of the key material\. If you create a new replica key, it has the same current key material of all related multi\-Region keys and access to all previous versions of the key material\. For details, see [Rotating multi\-Region keys](multi-region-keys-manage.md#multi-region-rotate)\.

*Changing the primary key* — Every set of multi\-Region keys must have exactly one primary key\. The [primary key](#mrk-primary-key) is the only key that can be replicated\. It's also the source of the shared properties of its replica keys\. But you can change the primary key to a replica and promote one of the replica keys to primary\. You might do this so you can delete a multi\-Region primary key from a particular Region, or locate the primary key in a Region closer to project administrators\. For details, see [Updating the primary Region](multi-region-keys-manage.md#multi-region-update)\.

*Deleting multi\-Region keys* — Like all KMS keys, you must schedule the deletion of multi\-Region keys before AWS KMS deletes them\. While the key is pending deletion, you cannot use it in any cryptographic operations\. However, AWS KMS will not delete a multi\-Region primary key until all of its replica keys are deleted\. For details, see [Deleting multi\-Region keys](multi-region-keys-delete.md)\.

## Concepts<a name="multi-region-concepts"></a>

The following terms and concepts are used with multi\-Region keys\.

### Multi\-Region key<a name="multi-Region-concept"></a>

A *multi\-Region key* is one of a set of KMS keys with the same key ID and key material \(and other [shared properties](#mrk-replica-key)\) in different AWS Regions\. Each multi\-Region key is a fully functioning KMS key that can be used entirely independently of its related multi\-Region keys\. Because all *related* multi\-Region keys have the same key ID and key material, they are *interoperable*, that is, any related multi\-Region key in any AWS Region can decrypt ciphertext encrypted by any other related multi\-Region key\.

You set the multi\-Region property of a KMS key when you create it\. You cannot change this property on an existing key\. Therefore, to move existing workloads into multi\-Region scenarios, you must re\-encrypt your data or create new signatures with new multi\-Region keys\.

A multi\-Region key can be [symmetric or asymmetric](symmetric-asymmetric.md) and it can use AWS KMS key material or [imported key material](importing-keys.md)\. You cannot create multi\-Region keys in a [custom key store](custom-key-store-overview.md)\.

In a set of related multi\-Region keys, there is exactly one [primary key](#mrk-primary-key) at any time\. You can create [replica keys](#mrk-replica-key) of that primary key in other AWS Regions\. You can also [update the primary region](multi-region-keys-manage.md#update-primary-console), which changes the primary key to a replica key and changes a specified replica key to the primary key\. However, you can maintain only one primary key or replica key in each AWS Region\. All of the Regions must be in the same [AWS partition](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html)\.

You can have multiple sets of related multi\-Region keys in the same or different AWS Regions\. Although related multi\-Region keys are interoperable, unrelated multi\-Region keys are not interoperable\.

### Primary key<a name="mrk-primary-key"></a>

A multi\-Region *primary key* is a KMS key that can be replicated into other AWS Regions in the same partition\. Each set of multi\-Region keys has just one primary key\.

A primary key differs from a replica key in the following ways:
+ Only a primary key can be [replicated](multi-region-keys-replicate.md)\.
+ The primary key is the source for [shared properties](#mrk-replica-key) of its [replica keys](#mrk-replica-key), including the key material and key ID\. 
+ You can enable and disable [automatic key rotation](rotate-keys.md) only on a primary key\.
+ You can [schedule the deletion of a primary key](multi-region-keys-delete.md) at any time\. But AWS KMS will not delete a primary key until all of its replica keys are deleted\.

However, primary and replica keys don't differ in any cryptographic properties\. You can use a primary key and its replica keys interchangeably\. 

You are not required to replicate a primary key\. You can use it just as you would any KMS key and replicate it if and when it is useful\. However, because multi\-Region keys have different security properties than single\-Region keys, we recommend that you create a multi\-Region key only when you plan to replicate it\.

### Replica key<a name="mrk-replica-key"></a>

A multi\-Region *replica key* is a KMS key that has the same [key ID](concepts.md#key-id-key-id) and [key material](concepts.md#key-material) as its [primary key](#mrk-primary-key) and related replica keys, but exists in a different AWS Region\. 

A replica key is a fully functional KMS key with it own key policy, grants, alias, tags, and other properties\. It is not a copy of or pointer to the primary key or any other key\. You can use a replica key even if its primary key and all related replica keys are disabled\. You can also convert a replica key to a primary key and a primary key to a replica key\. Once it is created, a replica key relies on its primary key only for [key rotation](multi-region-keys-manage.md#multi-region-rotate) and [updating the primary Region](multi-region-keys-manage.md#multi-region-update)\. 

Primary and replica keys don't differ in any cryptographic properties\. You can use a primary key and its replica keys interchangeably\. Data encrypted by a primary or replica key can be decrypted by the same key, or by any related primary or replica key\.

### Replicate<a name="replicate"></a>

You can *replicate* a multi\-Region [primary key](#mrk-primary-key) into a different AWS Region in the same partition\. When you do, AWS KMS creates a multi\-Region [replica key](#mrk-replica-key) in the specified Region with the same [key ID](concepts.md#key-id-key-id) and other [shared properties](#mrk-sync-properties) as its primary key\. Then it securely transports the key material across the Region boundary and associates it with the new replica key, all within AWS KMS\. 

### Shared properties<a name="mrk-sync-properties"></a>

*Shared properties* are properties of a multi\-Region primary key that are shared with its replica keys\. AWS KMS creates the replica keys with the same shared property values as those of the primary key\. Then, it periodically synchronizes the shared property values of the primary key to its replica keys\. You cannot set these properties on a replica key\. 

The following are the shared properties of multi\-Region keys\. 
+ [Key ID](concepts.md#key-id-key-id) — \(The `Region` element of the [key ARN](concepts.md#key-id-key-ARN) differs\.\)
+ [Key material](concepts.md#key-material)
+ [Key material origin](concepts.md#key-origin)
+ [Key spec](concepts.md#key-spec) and encryption algorithms
+ [Key usage](concepts.md#key-usage)
+ [Automatic key rotation](rotate-keys.md) — You can enable and disable automatic key rotation only on the primary key\. New replica keys are created with all versions of the shared key material\. For details, see [Rotating multi\-Region keys](multi-region-keys-manage.md#multi-region-rotate)\.

You can also think of the primary and replica designations of related multi\-Region keys as shared properties\. When you [create new replica keys](#mrk-replica-key) or [update the primary key](multi-region-keys-manage.md#update-primary-console), AWS KMS synchronizes the change to all related multi\-Region keys\. When these changes are complete, all related multi\-Region keys list their primary key and replica keys accurately\.

All other properties of multi\-Region keys are *independent properties*, including the description, [key policy](key-policies.md), [grants](grants.md), [enabled and disabled key states](enabling-keys.md), [aliases](kms-alias.md), and [tags](tagging-keys.md)\. You can set the same values for these properties on all related multi\-Region keys, but if you change the value of an independent property, AWS KMS does not synchronize it\.

You can track the synchronization of the shared properties of your multi\-Region keys\. In your AWS CloudTrail log, look for the [SynchronizeMultiRegionKey](ct-synchronize-multi-region-key.md) event\.