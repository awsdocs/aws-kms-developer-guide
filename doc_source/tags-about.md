# About tags in AWS KMS<a name="tags-about"></a>

A *tag* is an optional metadata label that you can assign \(or AWS can assign\) to an AWS resource\. Each tag consists of a *tag key* and a *tag value*, both of which are case\-sensitive strings\. The tag value can be an empty \(null\) string\. Each tag on a resource must have a different tag key, but you can add the same tag to multiple AWS resources\. Each resource can have up to 50 user\-created tags\. 

In AWS KMS, you can add tags to a [customer managed CMK](concepts.md#master_keys) when you [create the CMK](create-keys.md), and [tag or untag existing CMKs](manage-tags-api.md#tagging-keys-tag-resource) unless they are [pending deletion](key-state.md)\. You cannot tag aliases, [custom key stores](concepts.md#keystore-concept), [AWS managed CMKs](concepts.md#master_keys), [AWS owned CMKs](concepts.md#aws-owned-cmk), or CMKs in other AWS accounts\. Tags are optional, but they can be very useful\.

For example, you can add a `"Project"="Alpha"` tag to all CMKs and Amazon S3 buckets that you use for the Alpha project\.

```
TagKey   = "Project"
TagValue = "Alpha"
```

For general information about tags, including the format and syntax, see [Tagging AWS resources](https://docs.aws.amazon.com/general/latest/gr/aws_tagging.html) in the *Amazon Web Services General Reference*\.

Tags help you do the following:
+ Identify and organize your AWS resources\. Many AWS services support tagging, so you can assign the same tag to resources from different services to indicate that the resources are related\. For example, you can assign the same tag to an AWS KMS [customer master key](concepts.md#master_keys) \(CMK\) and an Amazon Elastic Block Store \(Amazon EBS\) volume or AWS Secrets Manager secret\. You can also use tags to identify CMK for automation\.
+ Track your AWS costs\. When you add tags to your AWS resources, AWS generates a cost allocation report with usage and costs aggregated by tags\. You can use this feature to track AWS KMS costs for a project, application, or cost center\.

  For more information about using tags for cost allocation, see [Using Cost Allocation Tags](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html) in the *AWS Billing and Cost Management User Guide*\. For information about the rules for tag keys and tag values, see [User\-Defined Tag Restrictions](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/allocation-tag-restrictions.html) in the *AWS Billing and Cost Management User Guide*\.
+ Control access to your AWS resources\. Allowing and denying access to CMKs based on their tags is part of AWS KMS support for [attribute\-based access control](abac.md) \(ABAC\)\. For information about controlling access to customer master keys based on their tags, see [Using tags to control access to CMKs](tag-authorization.md)\. For more general information about using tags to control access to AWS resources, see [Controlling Access to AWS Resources Using Resource Tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_tags.html) in the *IAM User Guide*\.

AWS KMS writes an entry to your AWS CloudTrail log when you use the [TagResource](ct-tagresource.md), [UntagResource](ct-untagresource.md), or [ListResourceTags](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListResourceTags.html) operations\.