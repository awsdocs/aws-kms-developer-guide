# Managing CMK tags with API operations<a name="manage-tags-api"></a>

You can use the [AWS Key Management Service \(AWS KMS\) API](https://docs.aws.amazon.com/kms/latest/APIReference/) to add, delete, and list tags for the CMKs that you manage\. These examples use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/), but you can use any supported programming language\. You cannot tag AWS managed CMKs\.

To add, edit, view, and delete tags for a CMK, you must have the required permissions\. For details, see [Controlling access to tags](tag-permissions.md)\.

**Topics**
+ [CreateKey: Add tags to a new CMK](#tagging-keys-create-key)
+ [TagResource: Add or change tags for a CMK](#tagging-keys-tag-resource)
+ [ListResourceTags: Get the tags for a CMK](#tagging-keys-list-resource-tags)
+ [UntagResource: Delete tags from a CMK](#tagging-keys-untag-resource)

## CreateKey: Add tags to a new CMK<a name="tagging-keys-create-key"></a>

You can add tags when you create a customer managed CMK\. To specify the tags, use the `Tags` parameter of the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation\. 

To add tags when creating a CMK, the caller must have `kms:TagResource` permission in an IAM policy\. At a minimum, the permission must cover all CMKs in the account and Region\. For details, see [Controlling access to tags](tag-permissions.md)\.

The value of the `Tags` parameter of `CreateKey` is a collection of case\-sensitive tag key and tag value pairs\. Each tag on a CMK must have a different tag name\. The tag value can be a null or empty string\.

For example, the following AWS CLI command creates a symmetric CMK with a `Project:Alpha` tag\. When specifying more than one key\-value pair, use a space to separate each pair\. 

```
$ aws kms create-key --tags TagKey=Project,TagValue=Alpha
```

When this command is successful, it returns a `KeyMetadata` object with information about the new CMK\. However, the `KeyMetadata` does not include tags\. To get the tags, use the [ListResourceTags](#tagging-keys-list-resource-tags) operation\.

## TagResource: Add or change tags for a CMK<a name="tagging-keys-tag-resource"></a>

The [TagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_TagResource.html) operation adds one or more tags to a CMK\. You cannot use this operation to add or edit tags in a different AWS account\.

To add a tag, specify a new tag key and a tag value\. To edit a tag, specify an existing tag key and a new tag value\. Each tag on a CMK must have a different tag key\. The tag value can be a null or empty string\.

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

## ListResourceTags: Get the tags for a CMK<a name="tagging-keys-list-resource-tags"></a>

The [ListResourceTags](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListResourceTags.html) operation gets the tags for a CMK\. The `KeyId` parameter is required\. You cannot use this operation to view the tags on CMKs in a different AWS account\.

For example, the following command gets the tags for an example CMK\.

```
$ aws kms list-resource-tags --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
       
  "Truncated": false,
  "Tags": [
      {
        "TagKey": "Project",
        "TagValue": "Alpha"
     },
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

## UntagResource: Delete tags from a CMK<a name="tagging-keys-untag-resource"></a>

The [UntagResource](https://docs.aws.amazon.com/kms/latest/APIReference/API_UntagResource.html) operation deletes tags from a CMK\. To identify the tags to delete, specify the tag keys\. You cannot use this operation to delete tags from CMKs a different AWS account\.

When it succeeds, the `UntagResource` operation doesn't return any output\. Also, if the specified tag key isn't found on the CMK, it doesn't throw an exception or return a response\. To confirm that the operation worked, use the [ListResourceTags](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListResourceTags.html) operation\.

For example, this command deletes the **Purpose** tag and its value from the specified CMK\.

```
$ aws kms untag-resource --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --tag-keys Purpose
```

When this command is successful, it does not return any output\.