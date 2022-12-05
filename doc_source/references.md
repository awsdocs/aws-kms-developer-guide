# References<a name="references"></a>

The following references provide useful information about using and managing KMS keys\. 
+ [Key type reference](symm-asymm-compare.md)\. Lists the type of KMS key that supports each AWS KMS API operation\.

  To find: Can I enable and disable an RSA signing KMS key?
+ [Key state table](key-state.md#key-state-table)\. Shows how the key state of a KMS key affects its use in AWS KMS API operations\.

  To find: Can I change the alias of a KMS key that is pending deletion?
+ [AWS KMS API permissions reference](kms-api-permissions-reference.md)\. Provides information about the permissions required for each AWS KMS API operation\.

  To find: Can I run [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) on a key in a different AWS account? Can I allow `kms:Decrypt` permission in an IAM policy?
  + [ViaService reference](conditions-kms.md#viaService_table)\. Lists the AWS services that support the `kms:ViaService` condition key\.

    To find: Can I use the `kms:ViaService` condition key to allow a permission only when it comes from Amazon ElastiCache? What about Amazon Neptune?
+ [AWS KMS pricing](https://aws.amazon.com/kms/pricing/)\. Lists and explains the price of KMS keys\.

  To find: How much does it cost to use my asymmetric keys?
+ [AWS KMS request quotas](requests-per-second.md#requests-per-second-table)\. Lists the per\-second quotas for AWS KMS API requests in each account and Region\.

  To find: How many [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) requests can I run in each second? How many [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) requests can I run on KMS keys in my custom key store? 
+ [AWS KMS resource quotas](resource-limits.md)\. Lists the quotas on AWS KMS resources\.

  To find: How many KMS key can I have in each Region of my account? How many aliases can I have on each KMS key?
+ [AWS services integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration)\. Lists the AWS services that use KMS keys to protect the resources that they create, store, and manage\.

  To find: Does Amazon Connect use KMS keys to protect my Connect resources? 