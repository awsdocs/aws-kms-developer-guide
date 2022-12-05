# Resource quotas<a name="resource-limits"></a>

AWS KMS establishes resource quotas to ensure that it can provide fast and resilient service to all of our customers\. Some resource quotas apply only to resources that you create, but not to resources that AWS services create for you\. Resources that you use, but that aren't in your AWS account, such as [AWS owned keys](concepts.md#aws-owned-cmk), do not count against these quotas\.

If you have exceeded a resource limit, requests to create an additional resource of that type generate an `LimitExceededException` error message\. 

All AWS KMS resource quotas are adjustable, except for the [key policy document size quota](#key-policy-limit) and the [custom key stores resource quota](#cks-resource-quota)\. To request a quota increase, see [Requesting a quota increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html) in the *Service Quotas User Guide*\. To request a quota decrease, to change a quota that is not listed in Service Quotas, or to change a quota in an AWS Region where Service Quotas for AWS KMS is not available, please visit [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case\. 

The following table lists and describes the AWS KMS resource quotas in each AWS account and Region\. 


| Quota name | Default value | Applies to | Adjustable | 
| --- | --- | --- | --- | 
| [AWS KMS keys](#kms-keys-limit) | 100,000 | Customer managed keys | Yes | 
| [Aliases per KMS key](#aliases-per-key) | 50 | Customer created aliases | Yes | 
| [Grants per KMS key](#grants-per-key) | 50,000 | Customer managed keys | Yes | 
| [Key policy document size](#key-policy-limit) | 32 KB \(32,768 bytes\) |  Customer managed keys AWS managed keys  | No | 
| [Custom key stores](#cks-resource-quota) | 10 | AWS account and Region | No | 

In addition to resource quotas, AWS KMS uses request quotas to ensure the responsiveness of the service\. For details, see [Request quotas](requests-per-second.md)\.

## AWS KMS keys: 100,000<a name="kms-keys-limit"></a>

You can have up to 100,000 [customer managed keys](concepts.md#customer-cmk) in each Region of your AWS account\. This quota applies to all customer managed keys in all AWS Regions regardless of their [key spec](concepts.md#key-spec) or [key state](key-state.md)\. Each KMS key is considered to be one resource\. [AWS managed keys](concepts.md#aws-managed-cmk) and [AWS owned keys](concepts.md#aws-owned-cmk) do not count against this quota\.

## Aliases per KMS key: 50<a name="aliases-per-key"></a>

You can associate up to 50 [aliases](kms-alias.md) with each [customer managed key](concepts.md#customer-cmk)\. Aliases that AWS associates with [AWS managed keys](concepts.md#aws-managed-cmk) do not count against this quota\. You might encounter this quota when you [create](alias-manage.md#alias-create) or [update](alias-manage.md#alias-update) an alias\.

**Note**  
The [kms:ResourceAliases](conditions-kms.md#conditions-kms-resource-aliases) condition is effective only when the KMS key conforms to this quota\. If a KMS key exceeds this quota, principals who are authorized to use the KMS key by the `kms:ResourceAliases` condition are denied access to the KMS key\. For details, see [Access denied due to alias quota](abac.md#access-denied-alias-quota)\.

The Aliases per KMS key quota replaces the Aliases per Region quota that limited the total number of aliases in each Region of an AWS account\. AWS KMS has eliminated the Aliases per Region quota\.

## Grants per KMS key: 50,000<a name="grants-per-key"></a>

Each [customer managed key](concepts.md#customer-cmk) can have up to 50,000 [grants](grants.md), including the grants created by [AWS services that are integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration)\. This quota does not apply to [AWS managed keys](concepts.md#aws-managed-cmk) or [AWS owned keys](concepts.md#aws-owned-cmk)\.

One effect of this quota is that you cannot perform more than 50,000 grant\-authorized operations that use the same KMS key at the same time\. After you reach the quota, you can create new grants on the KMS key only when an active grant is retired or revoked\.

For example, when you attach an Amazon Elastic Block Store \(Amazon EBS\) volume to an Amazon Elastic Compute Cloud \(Amazon EC2\) instance, the volume is decrypted so you can read it\. To get permission to decrypt the data, Amazon EBS creates a grant for each volume\. Therefore, if all of your Amazon EBS volumes use the same KMS key, you cannot attach more than 50,000 volumes at one time\.

## Key policy document size: 32 KB<a name="key-policy-limit"></a>

The maximum length of each [key policy document](key-policy-overview.md) is 32 KB \(32,768 bytes\)\. If you use a larger policy document to create or update the key policy for a KMS key, the operation fails\. 

This quota is not adjustable\. You cannot increase it by using Service Quotas or by creating a case in AWS Support\. If your key policy is approaching the limit, consider using [grants](grants.md) instead of policy statements\. Grants are particularly well suited to temporary or very specific permissions\.

You use a key policy document whenever you create or change a key policy by using the [default view](key-policy-modifying.md#key-policy-modifying-how-to-console-default-view) or [policy view](key-policy-modifying.md#key-policy-modifying-how-to-console-policy-view) in the AWS Management Console, or the [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operation\. This quota applies to your key policy document, even if you use the [default view](key-policy-modifying.md#key-policy-modifying-how-to-console-default-view) in the AWS KMS console, where you don't edit the JSON statements directly\.

## Custom key stores resource quota: 10<a name="cks-resource-quota"></a>

**Note**  
The AWS KMS custom key store resource quota does not yet appear in the Service Quotas console\. You cannot view or manage this quota by using Service Quotas API operations\.

You can create up to 10 [custom key stores](custom-key-store-overview.md) in each AWS account and Region\. If you try to create more, the [CreateCustomKeyStore](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateCustomKeyStore.html) operation fails\.

This quota applies to the total number of custom key stores in each account and region, including all [AWS CloudHSM key stores](keystore-cloudhsm.md) and [external key stores](keystore-external.md), regardless of their connection state\. 

This quota is not adjustable\. You cannot increase it by using Service Quotas or by creating a case in AWS Support\. If you are not using all of your custom key stores, consider deleting an unused, long\-disconnected custom key store\. Before you do, verify that you do not need any of the KMS keys in the custom key store to decrypt ciphertext\. 