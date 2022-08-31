# Resilience in AWS Key Management Service<a name="disaster-recovery-resiliency"></a>

The AWS global infrastructure is built around AWS Regions and Availability Zones\. AWS Regions provide multiple physically separated and isolated Availability Zones, which are connected with low\-latency, high\-throughput, and highly redundant networking\. With Availability Zones, you can design and operate applications and databases that automatically fail over between Availability Zones without interruption\. Availability Zones are more highly available, fault tolerant, and scalable than traditional single or multiple data center infrastructures\.

In addition to the AWS global infrastructure, AWS KMS offers several features to help support your data resiliency and backup needs\. For more information about AWS Regions and Availability Zones, see [AWS Global Infrastructure](http://aws.amazon.com/about-aws/global-infrastructure/)\.

## Regional isolation<a name="regional-isolation"></a>

AWS Key Management Service \(AWS KMS\) is a self\-sustaining Regional service that is available in all AWS Regions\. The Regionally isolated design of AWS KMS ensures that an availability issue in one AWS Region cannot affect AWS KMS operation in any other Region\. AWS KMS is designed to ensure *zero planned downtime*, with all software updates and scaling operations performed seamlessly and imperceptibly\.

The AWS KMS [Service Level Agreement](https://aws.amazon.com/kms/sla/) \(SLA\) includes a service commitment of 99\.999% for all KMS APIs\. To fulfill this commitment, AWS KMS ensures that all data and authorization information required to execute an API request is available on all regional hosts that receive the request\. 

The AWS KMS infrastructure is replicated in at least three Availability Zones \(AZs\) in each Region\. To ensure that multiple host failures do not affect AWS KMS performance, AWS KMS is designed to service customer traffic from any of the AZs in a Region\.

Changes that you make to the properties or permissions of a KMS key are replicated to all hosts in the Region to ensure that subsequent request can be processed correctly by any host in the Region\. Requests for [cryptographic operations](concepts.md#cryptographic-operations) using your KMS key are forwarded to a fleet of AWS KMS hardware security modules \(HSMs\), any of which can perform the operation with the KMS key\.

## Multi\-tenant design<a name="multi-tenant"></a>

The multi\-tenant design of AWS KMS enables it to fulfill the 99\.999% availability SLA, and to sustain high request rates, while protecting the confidentiality of your keys and data\. 

Multiple integrity\-enforcing mechanisms are deployed to ensure that the KMS key that you specified for the cryptographic operation is always the one that is used\. 

The plaintext key material for your KMS keys is protected extensively\. The key material is encrypted in the HSM as soon as it is created, and the encrypted key material is immediately moved to secure, low latency storage\. The encrypted key is retrieved and decrypted within the HSM just in time for use\. The plaintext key remains in HSM memory only for the time needed to complete the cryptographic operation\. Then it is re\-encrypted in the HSM and the encrypted key is returned to storage\. Plaintext key material never leaves the HSMs; it is never written to persistent storage\.

For more information about the mechanisms that AWS KMS uses to secure your keys, see [AWS Key Management Service Cryptographic Details](https://docs.aws.amazon.com/kms/latest/cryptographic-details/)\.

## Resilience best practices in AWS KMS<a name="customer-action.title"></a>

To optimize resilience for your AWS KMS resources, consider the following strategies\.
+ To support your backup and disaster recovery strategy, consider *multi\-Region keys*, which are KMS keys created in one AWS Region and replicated only to Regions that you specify\. With multi\-Region keys, you can move encrypted resources between AWS Regions \(within the same partition\) without ever exposing the plaintext, and decrypt the resource, when needed, in any of its destination Regions\. Related multi\-Region keys are interoperable because they share the same key material and key ID, but they have independent key policies for high\-resolution access control\. For details, see [Multi\-Region keys in AWS KMS](multi-region-keys-overview.md)\.
+ To protect your keys in a multi\-tenant service like AWS KMS, be sure to use access controls, including [key policies](key-policies.md) and [IAM policies](ControllingAccess5-IAMPolicies.xml)\. In addition, you can send your requests to AWS KMS using a *VPC interface endpoint* powered by AWS PrivateLink\. When you do, all communication between your Amazon VPC and AWS KMS is conducted entirely within the AWS network using a dedicated AWS KMS endpoint restricted to your VPC\. You can further secure these requests by creating an additional authorization layer using [VPC endpoint policies](kms-vpc-endpoint.md#vpce-policy)\. For details, see [Connecting to AWS KMS through a VPC endpoint](kms-vpc-endpoint.md)\.