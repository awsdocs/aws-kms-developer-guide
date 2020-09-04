# Resource quotas<a name="resource-limits"></a>

AWS KMS establishes resource quotas to ensure that it can provide fast and resilient service to all of our customers\. Some resource quotas apply only to resources that you create, but not to resources that AWS services create for you\. Resources that you use, but that aren't in your AWS account, such as [AWS owned CMKs](concepts.md#aws-owned-cmk), do not count against these quotas\.

If you have reached a resource limit, requests to create an additional resource of that type generate an `LimitExceededException` error message\. 

The following table lists and describes the AWS KMS resource quotas in each AWS account and Region\. If you need to exceed a quota, you can request a quota increase in Service Quotas\. Use the [Service Quotas console](https://console.aws.amazon.com/servicequotas) or the [RequestServiceQuotaIncrease](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/API_RequestServiceQuotaIncrease.html) operation\. For details, see [Requesting a quota increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html) in the *Service Quotas User Guide*\. If Service Quotas for AWS KMS are not available in the AWS Region, please visit the [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case\. 

For help requesting an increase in an AWS KMS quota, see [Request an AWS KMS Quota Increase](increase-quota.md)\.


| Quota name | Default value | Applies to | 
| --- | --- | --- | 
| [Customer master keys \(CMKs\)](#customer-master-keys-limit) | 10,000 | Customer managed CMKs | 
| [Aliases per Region](#aliases-limit) | 10,000 | Customer created aliases | 
| [Aliases per CMK](#aliases-per-key) | 50 | Customer created aliases | 
| [Grants per CMK](#grants-per-key) | 10,000 | Customer managed CMKs | 
| [Grants for a given principal per CMK](#grants-per-principal-per-key) | 500 |  Customer managed CMKs AWS managed CMKs  | 
| [Key policy document size](#key-policy-limit) | 32 KB \(32,768 bytes\) |  Customer managed CMKs AWS managed CMKs  | 

In addition to resource quotas, AWS KMS uses request quotas to ensure the responsiveness of the service\. For details, see [Request quotas](requests-per-second.md)\.

## Customer master keys \(CMKs\): 10,000<a name="customer-master-keys-limit"></a>

You can have up to 10,000 [customer managed CMKs](concepts.md#customer-cmk) in each Region of your AWS account\. This quota applies to all [symmetric](symm-asymm-concepts.md#symmetric-cmks) and [asymmetric](symm-asymm-concepts.md#asymmetric-cmks) customer managed CMKs regardless of their [key state](key-state.md)\. Each CMK — whether symmetric or asymmetric — is considered to be one resource\. [AWS managed CMKs](concepts.md#aws-managed-cmk) and [AWS owned CMKs](concepts.md#aws-owned-cmk) do not count against this quota\.

## Aliases per Region: 10,000<a name="aliases-limit"></a>

You can create up to 10,000 [aliases](kms-alias.md) in each AWS Region of your account\. Aliases that AWS creates in your account, such as aws/*<service\-name>*, do not count against this quota\. 

If you increase your [customer master keys quota](#customer-master-keys-limit), you might also need to [request an increase](increase-quota.md) in your aliases per Region quota\.

## Aliases per CMK: 50<a name="aliases-per-key"></a>

You can associate up to 50 [aliases](kms-alias.md) with each [customer managed CMK](concepts.md#customer-cmk)\. Aliases that AWS associates with [AWS managed CMKs](concepts.md#aws-managed-cmk) do not count against this quota\. You might encounter this quota when you [create](kms-alias.md#alias-create) or [update](kms-alias.md#alias-update) an alias\.

## Grants per CMK: 10,000<a name="grants-per-key"></a>

Each [customer managed CMK](concepts.md#customer-cmk) can have up to 10,000 [grants](grants.md), including the grants created by [AWS services that are integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration)\. This quota does not apply to [AWS managed CMKs](concepts.md#aws-managed-cmk) or [AWS owned CMKs](concepts.md#aws-owned-cmk)\.

One effect of this quota is that you cannot perform more than 10,000 grant\-authorized operations that use the same CMK at the same time\. After you reach the quota, you can create new grants on the CMK only when an active grant is retired or revoked\.

For example, when you attach an Amazon Elastic Block Store \(Amazon EBS\) volume to an Amazon Elastic Compute Cloud \(Amazon EC2\) instance, the volume is decrypted so you can read it\. To get permission to decrypt the data, Amazon EBS creates a grant for each volume\. However, you cannot have more than 10,000 grants on each CMK\. Therefore, if all of your Amazon EBS volumes use the same CMK, you cannot attach more than 10,000 volumes at one time\.

## Grants for a given principal per CMK: 500<a name="grants-per-principal-per-key"></a>

A CMK cannot have more than 500 grants for the same grantee principal\. The *grantee principal* is the identity that gets the permissions in the grant\.

This quota is calculated separately for each CMK in the account\. It applies to [customer managed CMKs](concepts.md#customer-cmk) and [AWS managed CMKs](concepts.md#aws-managed-cmk), but not to [AWS owned CMKs](concepts.md#aws-owned-cmk)\.

**Note**  
Be careful when using the output from the [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation to calculate the number of grants with the same grantee principal\.   
The `GranteePrincipal` field in the `ListGrants` response usually contains the grantee principal of the grant\. However, when the grantee principal in the grant is an AWS service, the `GranteePrincipal` field contains the [service principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services), which might represent several different grantee principals\.

This quota can have practical consequences for your use of AWS resources\. For example, it prevents you from launching more than 500 Amazon WorkSpaces encrypted under the same CMK\. When you launch a WorkSpace, Amazon WorkSpaces creates a grant that allows it to decrypt the WorkSpace so you can use it\. Each WorkSpace grant is unique, but all of the grants have the same grantee principal\.

## Key policy document size: 32 KB<a name="key-policy-limit"></a>

The maximum length of each key policy document is 32 KB \(32,768 bytes\)\. If you use a larger policy document to create or update the key policy for a CMK, the operation fails\. 

A [key policy document](key-policies.md#key-policy-overview) is a collection of policy statements in JSON format\. The statements in the key policy document determine who has permission to use the CMK and how they can use it\. You may also use IAM policies and grants to control access to the CMK, but every CMK must have a key policy document\. 

You use a key policy document whenever you create or change a key policy by using the [default view](key-policy-modifying.md#key-policy-modifying-how-to-console-default-view) or [policy view](key-policy-modifying.md#key-policy-modifying-how-to-console-policy-view) in the AWS Management Console, or the [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operation\. This quota applies to your key policy document, even if you use the [default view](key-policy-modifying.md#key-policy-modifying-how-to-console-default-view) in the AWS KMS console, where you don't edit the JSON statements directly\.