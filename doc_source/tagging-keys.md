# Tagging Keys<a name="tagging-keys"></a>

Many AWS services support resource tags\. You can use tags to label, identify, and categorize your resources across AWS services\.

AWS KMS supports resource tags on [customer master keys](concepts.md#master_keys) \(CMKs\)\. You can add, change, and delete resource tags for [customer managed CMKs](concepts.md#master_keys)\. AWS KMS doesn't support resource tags on aliases and you cannot create or manage the tags on [AWS managed CMKs](concepts.md#master_keys)\.

Each resource tag consists of a *tag key* and a *tag value* that you define\. For example, the tag key might be "Cost Center" and the tag value might be "87654\." You can attach the same tag, with the same tag key and tag value, to multiple CMKs\. However, every tag on a CMK must have a different tag key\. For example, you cannot have more than one "Cost Center" tag key on the same CMK\. For detailed information about resource tags, see [Tagging AWS Resources](https://docs.aws.amazon.com/general/latest/gr/aws_tagging.html) in the *AWS General Reference*\. 

When you add tags to your AWS resources, AWS generates a cost allocation report with usage and costs aggregated by tags\. You can use this feature to track AWS KMS costs for a project, application, or cost center\. For more information about using tags for cost allocation, see [Using Cost Allocation Tags](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html) in the *AWS Billing and Cost Management User Guide*\. For information about the rules that apply to tag keys and tag values, see [User\-Defined Tag Restrictions](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/allocation-tag-restrictions.html) in the *AWS Billing and Cost Management User Guide*\.

**Topics**
+ [Managing CMK Tags \(Console\)](#manage-tags)
+ [Managing CMK Tags \(KMS API\)](#manage-tags-api)

## Managing CMK Tags \(Console\)<a name="manage-tags"></a>

You can add, edit, and delete tags for your customer managed CMKs in the AWS Management Console\. You can add tags to a CMK when you [create it](create-keys.md) and edit them at any time\. You cannot edit the tags of CMKs that are pending deletion\. For more information, see [Creating Keys](create-keys.md) and [Editing Keys](editing-keys.md)\.

The following procedure shows how to manage tags using the check boxes on the **Customer managed keys** page\. You can use the **Tags** tab on the [detail page](viewing-keys-console.md#viewing-console-details) for any customer managed CMK\.

**To add, edit, or delete a tag for an existing CMK**

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\. \(You cannot manage the tags of an AWS managed CMK\.\)

1. Select the check box next to the alias of a CMK\.

1. Choose **Key actions**, **Add or edit tags**\.

1. Use the controls to add, edit, or delete tags\.

1. To save your changes, choose **Save changes**\.

## Managing CMK Tags \(KMS API\)<a name="manage-tags-api"></a>

You can use the [AWS Key Management Service \(AWS KMS\) API](https://docs.aws.amazon.com/kms/latest/APIReference/) to add, delete, and list tags for the CMKs that you manage\. These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

You cannot tag [AWS managed CMKs](concepts.md#aws-managed-cmk)\.

**Topics**
+ [CreateKey: Add Tags to a New CMK](#tagging-keys-create-key)
+ [TagResource: Add or Change Tags for a CMK](#tagging-keys-tag-resource)
+ [ListResourceTags: Get the Tags for a CMK](#tagging-keys-list-resource-tags)
+ [UntagResource: Delete Tags from a CMK](#tagging-keys-untag-resource)

### CreateKey: Add Tags to a New CMK<a name="tagging-keys-create-key"></a>

You can add tags when you create a [customer managed CMK](concepts.md#customer-cmk)\. To specify the tags, use the `Tags` parameter of the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation\. The value of the `Tags` parameter is a collection of tag key\-value pairs\.

For example, the following AWS CLI command creates a symmetric CMK with a `Project:Alpha` tag\. When specifying more than one key\-value pair, use a space to separate each pair\. 

```
$ aws kms create-key --tags TagKey=Project,TagValue=Alpha
```

When this command is successful, it returns a `KeyMetadata` object with information about the new CMK\. However, the `KeyMetadata` does not include tags\. To get the tags, use the [ListResourceTags](#tagging-keys-list-resource-tags) operation\.

### TagResource: Add or Change Tags for a CMK<a name="tagging-keys-tag-resource"></a>

The [TagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) operation adds one or more tags to a CMK\.

You can also use **TagResource** to change the values for an existing tag\. To replace tag values, specify the same tag key with different values\. To add values to a tag, specify the tag key with both new and existing values\.

For example, the following command adds **Purpose** and **Department** tags to an example CMK\.

```
$ aws kms tag-resource \
         --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
         --tags TagKey=Purpose,TagValue=Test TagKey=Department,TagValue=Finance
```

When this command is successful, it does not return any output\. To view the tags on a CMK, use the [ListResourceTags](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListResourceTags.html) operation\.

### ListResourceTags: Get the Tags for a CMK<a name="tagging-keys-list-resource-tags"></a>

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

### UntagResource: Delete Tags from a CMK<a name="tagging-keys-untag-resource"></a>

The [UntagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_UntagResource.html) operation deletes tags from a CMK\. The `key-id` and `tag-keys` parameters are required\.

For example, this command deletes the **Purpose** tag and all of its values from the specified CMK\.

```
$ aws kms untag-resource --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --tag-keys Purpose
```

When this command is successful, it does not return any output\.