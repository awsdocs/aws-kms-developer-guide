# Tagging keys<a name="tagging-keys"></a>

In AWS KMS, you can add tags to a [customer managed CMK](concepts.md#master_keys) when you [create the CMK](create-keys.md), and [tag or untag existing CMKs](manage-tags-api.md#tagging-keys-tag-resource) unless they are [pending deletion](key-state.md)\. You cannot tag aliases, [custom key stores](concepts.md#keystore-concept), [AWS managed CMKs](concepts.md#master_keys), [AWS owned CMKs](concepts.md#aws-owned-cmk), or CMKs in other AWS accounts\. Tags are optional, but they can be very useful\.

For more information, see [Creating keys](create-keys.md) and [Editing keys](editing-keys.md)\. For general information about tags, including best practices, tagging strategies, and the format and syntax of tags, see [Tagging AWS resources](https://docs.aws.amazon.com/general/latest/gr/aws_tagging.html) in the *Amazon Web Services General Reference*\. 

**Topics**
+ [About tags in AWS KMS](tags-about.md)
+ [Managing CMK tags in the console](manage-tags-console.md)
+ [Managing CMK tags with API operations](manage-tags-api.md)
+ [Controlling access to tags](tag-permissions.md)
+ [Using tags to control access to CMKs](tag-authorization.md)