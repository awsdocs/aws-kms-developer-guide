# Tagging Keys<a name="tagging-keys"></a>

You can add, change, and delete tags for [customer managed CMKs](concepts.md#master_keys)\. Each tag consists of a *tag key* and a *tag value* that you define\. For example, the tag key might be "Cost Center" and the tag value might be "87654\." You cannot tag [AWS managed CMKs](concepts.md#master_keys)\.

When you add tags to your AWS resources, AWS generates a cost allocation report with usage and costs aggregated by tags\. You can use this feature to track AWS KMS costs for a project, application, or cost center\.

For more information about using tags for cost allocation, see [Using Cost Allocation Tags](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html) in the *AWS Billing and Cost Management User Guide*\. For information about the rules that apply to tag keys and tag values, see [User\-Defined Tag Restrictions](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/allocation-tag-restrictions.html) in the *AWS Billing and Cost Management User Guide*\.

**Topics**
+ [Managing CMK Tags \(Console\)](#manage-tags)
+ [Managing CMK Tags \(KMS API\)](#manage-tags-api)

## Managing CMK Tags \(Console\)<a name="manage-tags"></a>

You can add, edit, and delete tags for your customer managed CMKs in the AWS Management Console\. You can add tags to a CMK when you [create it](create-keys.md) and edit them at any time\. You cannot edit the tags of CMKs that are pending deletion\. For more information, see [Editing Keys](editing-keys.md)\. 

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\. \(You cannot manage the tags of an AWS managed CMK\.\)

1. Select the check box next to the alias of a CMK\.

1. Choose **Key actions**, **Add or edit tags**\.

1. Use the controls to add, edit, or delete tags\. The tag name must be unique in the account and region\.

1. To save your changes, choose **Save changes**\.

## Managing CMK Tags \(KMS API\)<a name="manage-tags-api"></a>

You can use the [AWS Key Management Service \(AWS KMS\) API](https://docs.aws.amazon.com/kms/latest/APIReference/) to add, delete, and list tags for the CMKs that you manage\. These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. 

You cannot tag AWS managed CMKs\.

**Topics**
+ [TagResource: Add or Change Tags for a CMK](#tagging-keys-tag-resource)
+ [ListResourceTags: Get the Tags for a CMK](#tagging-keys-list-resource-tags)
+ [UntagResource: Delete Tags from a CMK](#tagging-keys-untag-resource)

### TagResource: Add or Change Tags for a CMK<a name="tagging-keys-tag-resource"></a>

The [TagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) operation adds one or more tags to a CMK\.

You can also use **TagResource** to change the values for an existing tag\. To replace tag values, specify the same tag key with different values\. To add values to a tag, specify the tag key with both new and existing values\.

For example, this call to the **TagResource** operation adds **Purpose** and **Department** tags to the specified CMK\. You can use any keys and values as CMK tags\.

```
$ aws kms tag-resource --key-id 1234abcd-12ab-34cd-56ef-1234567890ab /
                       --tags TagKey=Purpose,TagValue=Test /
                              TagKey=Department,TagValue=Finance
```

When this command is successful, it does not return any output\. To view the tags on a CMK, use the [ListResourceTags](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListResourceTags.html) operation\.

### ListResourceTags: Get the Tags for a CMK<a name="tagging-keys-list-resource-tags"></a>

The [ListResourceTags](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListResourceTags.html) operation gets the tags for a CMK\. The `key-id` parameter is required\.

For example, the following command gets the tags for the specified CMK\.

```
$ aws kms list-resource-tags --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
       
    "Truncated": false,
    "Tags": [
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
$ aws kms untag-resource --tag-keys Purpose --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
```

When this command is successful, it does not return any output\.