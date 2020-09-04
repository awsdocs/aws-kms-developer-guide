# Tagging keys<a name="tagging-keys"></a>

A *tag* is a metadata label that you can assign \(or AWS can assign\) to an AWS resource\. Each tag consists of a *tag key* and a *tag value*, both of which are case\-sensitive strings\. The tag value can be an empty \(null\) string\. Each tag on a resource must have a different tag key, but you can add the same tag to multiple AWS resources\. Each resource can have up to 50 user\-created tags\. 

In AWS KMS, you can add tags to a [customer managed CMK](concepts.md#master_keys) when you [create the CMK](create-keys.md), and [add or edit tags](editing-keys.md#edit-tags) on existing CMKs unless they are [pending deletion](key-state.md)\. You cannot tag aliases, [AWS managed CMKs](concepts.md#master_keys), or [AWS owned CMKs](concepts.md#aws-owned-cmk)\.

For example, you can add a `"Project"="Alpha"` tag to all CMKs and Amazon S3 buckets that you use for the Alpha project\.

```
TagKey   = "Project"
TagValue = "Alpha"
```

 For more information, see [Creating keys](create-keys.md) and [Editing keys](editing-keys.md)\. For general information about tags, including the format and syntax, see [Tagging AWS resources](https://docs.aws.amazon.com/general/latest/gr/aws_tagging.html) in the *Amazon Web Services General Reference*\.

Tags help you do the following:
+ Identify and organize your AWS resources\. Many AWS services support tagging, so you can assign the same tag to resources from different services to indicate that the resources are related\. For example, you can assign the same tag to an AWS KMS [customer master key](concepts.md#master_keys) \(CMK\) and an Amazon Elastic Block Store \(Amazon EBS\) volume or AWS Secrets Manager secret\. You can also use tags to identify CMK for automation\.
+ Track your AWS costs\. When you add tags to your AWS resources, AWS generates a cost allocation report with usage and costs aggregated by tags\. You can use this feature to track AWS KMS costs for a project, application, or cost center\.

  For more information about using tags for cost allocation, see [Using Cost Allocation Tags](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html) in the *AWS Billing and Cost Management User Guide*\. For information about the rules for tag keys and tag values, see [User\-Defined Tag Restrictions](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/allocation-tag-restrictions.html) in the *AWS Billing and Cost Management User Guide*\.

AWS KMS writes an entry to your AWS CloudTrail log when you use the [TagResource](ct-tagresource.md), [UntagResource](ct-untagresource.md), or `ListResourceTags` operations\.

**Topics**
+ [Managing CMK tags \(console\)](#manage-tags)
+ [Managing CMK Tags \(AWS KMS API\)](#manage-tags-api)

## Managing CMK tags \(console\)<a name="manage-tags"></a>

You can add tags to a CMK when you [create the CMK](create-keys.md) in the AWS KMS console\. You can also use the console to add, edit, and delete tags on customer managed CMKs\. 

**To add, edit, or delete a tag for an existing CMK**

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\. \(You cannot manage the tags of an AWS managed CMK\.\)

1. Select the check box next to the alias of a CMK\.

1. Choose **Key actions**, **Add or edit tags**\.

1. On the details page for CMK, choose the **Tags** tab\.
   + To create your first tag, choose **Create tag**, type a tag name \(required\) and tag value \(optional\), and then choose **Save**\.

     If you leave the tag value blank, the actual tag value is a null or empty string\.
   + To add a tag, choose **Edit**, choose **Add tag**, type a tag name and tag value, and then choose **Save**\.
   + To change the name or value of a tag, choose **Edit**, make your changes, and then choose **Save**\.
   + To delete a tag, choose **Edit**\. On the tag row, choose **Remove**, and then choose **Save**\.

1. To save your changes, choose **Save changes**\.

## Managing CMK Tags \(AWS KMS API\)<a name="manage-tags-api"></a>

You can use the [AWS Key Management Service \(AWS KMS\) API](https://docs.aws.amazon.com/kms/latest/APIReference/) to add, delete, and list tags for the CMKs that you manage\. These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

You cannot tag AWS managed CMKs\.

**Topics**
+ [CreateKey: Add tags to a new CMK](#tagging-keys-create-key)
+ [TagResource: Add or change tags for a CMK](#tagging-keys-tag-resource)
+ [ListResourceTags: Get the tags for a CMK](#tagging-keys-list-resource-tags)
+ [UntagResource: Delete tags from a CMK](#tagging-keys-untag-resource)

### CreateKey: Add tags to a new CMK<a name="tagging-keys-create-key"></a>

You can add tags when you create a customer managed CMK\. To specify the tags, use the `Tags` parameter of the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation\. The value of the `Tags` parameter is a collection of case\-sensitive tag key and tag value pairs\. 

Each tag on a CMK must have a different tag name\. The tag value can be a null or empty string\.

For example, the following AWS CLI command creates a symmetric CMK with a `Project:Alpha` tag\. When specifying more than one key\-value pair, use a space to separate each pair\. 

```
$ aws kms create-key --tags TagKey=Project,TagValue=Alpha
```

When this command is successful, it returns a `KeyMetadata` object with information about the new CMK\. However, the `KeyMetadata` does not include tags\. To get the tags, use the [ListResourceTags](#tagging-keys-list-resource-tags) operation\.

### TagResource: Add or change tags for a CMK<a name="tagging-keys-tag-resource"></a>

The [TagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) operation adds one or more tags to a CMK\. 

To add a tag, specify a new tag key and a tag value\. To edit a tag, specify an existing tag key and a new tag value\. Each tag on a CMK must have a different tag name\. The tag value can be a null or empty string\.

For example, the following command adds **Purpose** and **Department** tags to an example CMK\.

```
$ aws kms tag-resource \
         --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
         --tags TagKey=Purpose,TagValue=Pretest TagKey=Department,TagValue=Finance
```

When this command is successful, it does not return any output\. To view the tags on a CMK, use the [ListResourceTags](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListResourceTags.html) operation\.

You can also use **TagResource** to change the tag value of an existing tag\. To replace a tag value, specify the same tag key with a different value\.

For example, this command changes the value of the `Purpose` tag from `Pretest` to `Test`\.

```
$ aws kms tag-resource \
         --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
         --tags TagKey=Purpose,TagValue=Test
```

### ListResourceTags: Get the tags for a CMK<a name="tagging-keys-list-resource-tags"></a>

The [ListResourceTags](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListResourceTags.html) operation gets the tags for a CMK\. The `key-id` parameter is required\.

For example, the following command gets the tags for an example CMK\.

```
$ aws kms list-resource-tags --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
       
    "Truncated": false,
    "Tags": [
        {
            "TagKey": "Project",
            "TagValue": "Alpha"
        }
        {
            "TagKey": "Purpose",
            "TagValue": "Test"
        },
        {
            "TagKey": "Department",
            "TagValue": "Finance"
        }
    ]
}
```

### UntagResource: Delete tags from a CMK<a name="tagging-keys-untag-resource"></a>

The [UntagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_UntagResource.html) operation deletes tags from a CMK\. The `key-id` and `tag-keys` parameters are required\.

When it succeeds, the `UntagResource` operation doesn't return any output\. Also, if the specified tag key isn't found on the CMK, it doesn't throw an exception or return a response\. To confirm that the operation worked, use the [ListResourceTags](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListResourceTags.html) operation\.

For example, this command deletes the **Purpose** tag and its value from the specified CMK\.

```
$ aws kms untag-resource --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --tag-keys Purpose
```

When this command is successful, it does not return any output\.