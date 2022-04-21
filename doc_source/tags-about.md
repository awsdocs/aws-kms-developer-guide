# About tags in AWS KMS<a name="tags-about"></a>

A *tag* is an optional metadata label that you can assign \(or AWS can assign\) to an AWS resource\. Each tag consists of a *tag key* and a *tag value*, both of which are case\-sensitive strings\. The tag value can be an empty \(null\) string\. Each tag on a resource must have a different tag key, but you can add the same tag to multiple AWS resources\. Each resource can have up to 50 user\-created tags\. 

In AWS KMS, you can add tags to a [customer managed key](concepts.md#customer-cmk) when you [create the KMS key](create-keys.md), and [tag or untag existing KMS keys](manage-tags-api.md#tagging-keys-tag-resource) unless they are [pending deletion](key-state.md)\. You cannot tag aliases, [custom key stores](concepts.md#keystore-concept), [AWS managed keys](concepts.md#aws-managed-cmk), [AWS owned keys](concepts.md#aws-owned-cmk), or KMS keys in other AWS accounts\. Tags are optional, but they can be very useful\.

For example, you can add a `"Project"="Alpha"` tag to all KMS keys and Amazon S3 buckets that you use for the Alpha project\.

```
TagKey   = "Project"
TagValue = "Alpha"
```

For general information about tags, including the format and syntax, see [Tagging AWS resources](https://docs.aws.amazon.com/general/latest/gr/aws_tagging.html) in the *Amazon Web Services General Reference*\.

Tags help you do the following:
+ Identify and organize your AWS resources\. Many AWS services support tagging, so you can assign the same tag to resources from different services to indicate that the resources are related\. For example, you can assign the same tag to an [KMS key](concepts.md#kms_keys) and an Amazon Elastic Block Store \(Amazon EBS\) volume or AWS Secrets Manager secret\. You can also use tags to identify KMS keys for automation\.
+ Track your AWS costs\. When you add tags to your AWS resources, AWS generates a cost allocation report with usage and costs aggregated by tags\. You can use this feature to track AWS KMS costs for a project, application, or cost center\.

  For more information about using tags for cost allocation, see [Using Cost Allocation Tags](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html) in the *AWS Billing User Guide*\. For information about the rules for tag keys and tag values, see [User\-Defined Tag Restrictions](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/allocation-tag-restrictions.html) in the *AWS Billing User Guide*\.
+ Control access to your AWS resources\. Allowing and denying access to KMS keys based on their tags is part of AWS KMS support for [attribute\-based access control](abac.md) \(ABAC\)\. For information about controlling access to AWS KMS keys based on their tags, see [Using tags to control access to KMS keys](tag-authorization.md)\. For more general information about using tags to control access to AWS resources, see [Controlling Access to AWS Resources Using Resource Tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_tags.html) in the *IAM User Guide*\.

AWS KMS writes an entry to your AWS CloudTrail log when you use the [TagResource](ct-tagresource.md), [UntagResource](ct-untagresource.md), or [ListResourceTags](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListResourceTags.html) operations\.