# Tagging keys<a name="tagging-keys"></a>

In AWS KMS, you can add tags to a [customer managed key](concepts.md#kms_keys) when you [create the KMS key](create-keys.md), and [tag or untag existing KMS keys](manage-tags-api.md#tagging-keys-tag-resource) unless they are [pending deletion](key-state.md)\. You cannot tag aliases, [custom key stores](concepts.md#keystore-concept), [AWS managed keys](concepts.md#kms_keys), [AWS owned keys](concepts.md#aws-owned-cmk), or KMS keys in other AWS accounts\. Tags are optional, but they can be very useful\.

For more information, see [Creating keys](create-keys.md) and [Editing keys](editing-keys.md)\. For general information about tags, including best practices, tagging strategies, and the format and syntax of tags, see [Tagging AWS resources](https://docs.aws.amazon.com/general/latest/gr/aws_tagging.html) in the *Amazon Web Services General Reference*\. 

**Topics**
+ [About tags in AWS KMS](tags-about.md)
+ [Managing KMS key tags in the console](manage-tags-console.md)
+ [Managing KMS key tags with API operations](manage-tags-api.md)
+ [Controlling access to tags](tag-permissions.md)
+ [Using tags to control access to KMS keys](tag-authorization.md)