# Viewing KMS keys in the console<a name="viewing-keys-console"></a>

In the AWS Management Console, you can view lists of your KMS keys in the account and Region and details about each KMS key\.

**Note**  
The AWS KMS console displays the KMS keys that you have [permission to view](customer-managed-policies.md#iam-policy-example-read-only-console) in your account and Region\. KMS keys in other AWS accounts do not appear in the console, even if you have permission to view, manage, and use them\. To view KMS keys in other accounts, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\.

**Topics**
+ [Navigating to the key tables](#viewing-console-navigate)
+ [Navigating to key details](#viewing-details-navigate)
+ [Sorting and filtering your KMS keys](#viewing-console-filter)
+ [Displaying KMS key details](#viewing-console-details)
+ [Customizing your KMS key tables](#viewing-console-customize)

## Navigating to the key tables<a name="viewing-console-navigate"></a>

The AWS KMS keys in each account and Region are displayed in tables\. There are separate tables for the KMS keys that you create and the KMS keys that AWS services create for you\. 

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. To view the keys in your account that you create and manage, in the navigation pane choose **Customer managed keys**\. To view the keys in your account that AWS creates and manages for you, in the navigation pane, choose **AWS managed keys**\. For information about the different types of KMS keys, see [AWS KMS keys](concepts.md#kms_keys)\.
**Tip**  
To view [AWS managed keys](concepts.md#aws-managed-cmk) that are missing an alias, use the **Customer managed keys** page\.  
The AWS KMS console also displays the custom key stores in the account and Region\. KMS keys that you create in custom key stores appear on the **Customer managed keys** page\. For information about custom key stores, see [Custom key stores](custom-key-store-overview.md)\.

## Navigating to key details<a name="viewing-details-navigate"></a>

There is a details page for every AWS KMS key in the account and Region\. The details page displays the **General configuration** section for the KMS key and includes tabs that let authorized users view and manage the **Cryptographic configuration** and **Key policy** for the key\. Depending on the type of key, the detail page might also include **Aliases**, **Key material**, **Key rotation**, **Public key**, **Regionality** and **Tags** tabs\.

To navigate to the key details page for a KMS key\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. To view the keys in your account that you create and manage, in the navigation pane choose **Customer managed keys**\. To view the keys in your account that AWS creates and manages for you, in the navigation pane, choose **AWS managed keys**\. For information about the different types of KMS keys, see [AWS KMS key](concepts.md#kms_keys)\.

1. To open the key details page, in the key table, choose the key ID or alias of the KMS key\.

   If the KMS key has multiple aliases, an alias summary \(**\+*n* more**\) appears beside the name of the one of the aliases\. Choosing the alias summary takes you directly to the **Aliases** tab on the key details page\.

## Sorting and filtering your KMS keys<a name="viewing-console-filter"></a>

To make it easier to find your KMS keys in the console, you can sort and filter the key tables\. 

**Sort**  
You can sort KMS keys in ascending or descending order by their column values\. This feature sorts all KMS keys in the table, even if they don't appear on the current table page\.  
Sortable columns are indicated by an arrow beside the column name\. On the **AWS managed keys** page, you can sort by **Aliases** or **Key ID**\. On the **Customer managed keys** page, you can sort by **Aliases**, **Key ID**, or **Key type**\.  
To sort in ascending order, choose the column heading until the arrow points upward\. To sort in descending order, choose the column heading until the arrow points downward\. You can sort by only one column at a time\.  
For example, you can sort KMS keys in ascending order by key ID, instead of aliases, which is the default\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-sort.png)
When you sort KMS keys on the **Customer managed keys** page in ascending order by **Key type**, all asymmetric keys are displayed before all symmetric keys\.

**Filter**  
You can filter KMS keys by their property values or tags\. The filter applies to all KMS keys in the table, even if they don't appear on the current table page\. The filter is not case\-sensitive\.  
Filterable properties are listed in the filter box\. On the **AWS managed keys ** page, you can filter by alias and key ID\. On the **Customer managed keys** page, you can filter by the alias, key ID, and key type properties, and by tags\.  
+ On the **AWS managed keys** page, you can filter by alias and key ID\.
+ On the **Customer managed keys** page, you can filter by tags, or by the alias, key ID, key type, or regionality properties\.
To filter by a property value, choose the filter, choose the property name, and then choose from the list of actual property values\. To filter by a tag, choose the tag key, and then choose from the list of actual tag values\. After choosing a property or tag key, you can also type all or part of the property value or tag value\. You'll see a preview of the results before you make your choice\.   
For example, to display KMS keys with an alias name that contains `aws/e`, choose the filter box, choose **Alias**, type `aws/e`, and then press `Enter` or `Return` to add the filter\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/filter-alias.png)
To display only asymmetric KMS keys on the **Customer managed keys** page, click the filter box, choose **Key type** and then choose **Key type: Asymmetric**\. The **Asymmetric** option appears only when you have asymmetric KMS keys in the table\. For more information about identifying asymmetric KMS keys, see [Identifying asymmetric KMS keys](find-symm-asymm.md)\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/filter-keytype.png)
To display only multi\-Region keys, on the **Customer managed keys** page, choose the filter box, choose **Regionality** and then choose **Regionality: Multi\-Region**\. The **Multi\-Region** option appears only when you have multi\-Region keys in the table\. For more information about identifying multi\-Region keys, see [Viewing multi\-Region keys](multi-region-keys-view.md)\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/mrk-view-regionality-filter.png)
Tag filtering is a bit different\. To display only KMS keys with a particular tag, choose the filter box, choose the tag key, and then choose from among the actual tag values\. You can also type all or part of the tag value\.  
The resulting table displays all KMS keys with the chosen tag\. However, it doesn't display the tag\. To see the tag, choose the key ID or alias of the KMS key and on its detail page, choose the **Tags** tab\. The tabs appear below the **General configuration** section\.  
This filter requires both the tag key and tag value\. It won't find KMS keys by typing only the tag key or only its value\. To filter tags by all or part of the tag key or value, use the [ListResourceTags](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListResourceTags.html) operation to get tagged KMS keys, then use the filtering features of your programming language\. For an example, see [ListResourceTags: Get the tags on KMS keys](viewing-keys-cli.md#viewing-keys-list-resource-tags)\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/filter-tags-2.png)
To search for text, in the filter box, type all or part of an alias, key ID, key type, or tag key\. \(After you select the tag key, you can search for a tag value \)\. You'll see a preview of the results before you make your choice\.  
For example, to display KMS keys with `test` in its tag keys or filterable properties, type `test` in the filter box\. The preview shows the KMS keys that the filter will select\. In this case, `test` appears only in the **Alias** property\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/filter-all-properties.png)
You can use multiple filters at the same time\. When you add additional filters, you can also select a logical operator\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/filter-multi-properties.png)

## Displaying KMS key details<a name="viewing-console-details"></a>

The details page for each KMS key displays the properties of the KMS key\. It differs slightly for the different types of KMS keys\. 

To display detailed information about a KMS key, on the **AWS managed keys ** or **Customer managed keys** page, choose the alias or key ID of the KMS key\. 

The details page for a KMS key includes a **General Configuration** section that displays the basic properties of the KMS key\. It also includes tabs on which you can view and edit properties of the KMS key, such as **Key policy**, **Cryptographic configuration**, **Tags**, **Key material** \(for KMS keys with imported key material\), **Key rotation** \(for symmetric encryption KMS keys\), **Regionality** \(for multi\-Region keys\), and **Public key** \(for asymmetric KMS keys\)\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-detail-view-symmetric-sm.png)

The following list describes the fields in the detailed display, including field in the tabs\. Some of these fields are also available as columns in the table display\.

**Aliases**  
Where: Aliases tab  
A friendly name for the KMS key\. You can use an alias to identify the KMS key in the console and in some AWS KMS APIs\. For details, see [Using aliases](kms-alias.md)\.  
The **Aliases** tab displays all aliases associated with the KMS key in the AWS account and Region\. 

**ARN**  
Where: General configuration section  
The Amazon Resource Name \(ARN\) of the KMS key\. This value uniquely identifies the KMS key\. You can use it to identify the KMS key in AWS KMS API operations\.

**Creation date**  
Where: General configuration section  
The date and time that the KMS key was created\. This value is displayed in local time for the device\. The time zone does not depend on the Region\.  
Unlike **Expiration**, the creation refers only to the KMS key, not its key material\. 

**CloudHSM cluster ID**  
Where: Cryptographic configuration tab  
The cluster ID of the AWS CloudHSM cluster that contains the key material for the KMS key\. This field appears only when the KMS key is created in an AWS KMS [custom key store](custom-key-store-overview.md)\.  
If you choose the CloudHSM cluster ID, it opens the **Clusters** page in the AWS CloudHSM console\.

**Custom key store ID**  
Where: Cryptographic configuration tab  
The ID of the [custom key store](custom-key-store-overview.md) that contains the KMS key\. This field appears only when the KMS key is created in an AWS KMS custom key store\.  
If you choose the custom key store ID, it opens the **Custom key stores** page in the AWS KMS console\.

**Custom key store name**  
Where: Cryptographic configuration tab  
The name of the [custom key store](custom-key-store-overview.md) that contains the KMS key\. This field appears only when the KMS key is created in an AWS KMS custom key store\.

**Description**  
Where: General configuration section  
A brief, optional description of the KMS key that you can write and edit\. To add or update the description of a customer managed key, above **General Configuration**, choose **Edit**\.

**Encryption algorithms**  
Where: Cryptographic configuration tab  
Lists the encryption algorithms that can be used with the KMS key in AWS KMS\. This field appears only when the **Key type** is **Asymmetric** and the **Key usage** is **Encrypt and decrypt**\. For information about the encryption algorithms that AWS KMS supports, see [SYMMETRIC\_DEFAULT key spec](asymmetric-key-specs.md#key-spec-symmetric-default) and [RSA key specs for encryption and decryption](asymmetric-key-specs.md#key-spec-rsa-encryption)\.

**Expiration date**  
Where: Key material tab  
The date and time when the key material for the KMS key expires\. This field appears only for KMS keys with [imported key material](importing-keys.md), that is, when the **Origin** is **External** and the KMS key has key material that expires\.

**Key policy**  
Where: Key policy tab  
Controls access to the KMS key along with [IAM policies](iam-policies.md) and [grants](grants.md)\. Every KMS key has one key policy\. It is the only mandatory authorization element\. To change the key policy of a customer managed key, on the **Key policy** tab, choose **Edit**\. For details, see [Key policies in AWS KMS](key-policies.md)\.

**Key rotation**  
Where: Key rotation tab  
Enables and disables [automatic rotation](rotate-keys.md) of the key material in a [customer managed KMS key](concepts.md#customer-cmk)\. To change the key rotation status of a [customer managed key](concepts.md#customer-cmk), use the check box on the **Key rotation** tab\.   
You can't enable or disable rotation of the key material in an [AWS managed key](concepts.md#aws-managed-cmk)\. AWS managed keys are automatically rotated every year\.

**Key spec**  
Where: Cryptographic configuration tab  
The type of key material in the KMS key\. AWS KMS supports symmetric encryption KMS keys \(SYMMETRIC\_DEFAULT\), HMAC KMS keys of different lengths, KMS keys for RSA keys of different lengths, and elliptic curve keys with different curves\. For details, see [Key spec](concepts.md#key-spec)\.

**Key type**  
Where: Cryptographic configuration tab  
Indicates whether the KMS key is **Symmetric** or **Asymmetric**\.

**Key usage**  
Where: Cryptographic configuration tab  
Indicates whether a KMS key can be used for **Encrypt and decrypt**, **Sign and verify** or **Generate and verify MAC**\. For details, see [Key usage](concepts.md#key-usage)\.

**Origin**  
Where: Cryptographic configuration tab  
The source of the key material for the KMS key\. Valid values are **AWS\_KMS** for key material that AWS KMS generates, **EXTERNAL** for [imported key material](importing-keys.md), and **AWS\_CloudHSM** for KMS keys in [custom key stores](custom-key-store-overview.md)\.

**MAC algorithms**  
Where: Cryptographic configuration tab  
Lists the MAC algorithms that can be used with an HMAC KMS key in AWS KMS\. This field appears only when the **Key spec** is an HMAC key spec \(HMAC\_\*\)\. For information about the MAC algorithms that AWS KMS supports, see [Key specs for HMAC KMS keys](hmac.md#hmac-key-specs)\.

**Primary key**  
Where: Regionality tab  
Indicates that this KMS key is a [multi\-Region primary key](multi-region-keys-overview.md#mrk-primary-key)\. Authorized users can use this section to [change the primary key](multi-region-keys-manage.md) to a different related multi\-Region key\. This field appears only when the KMS key is a multi\-Region primary key\.

**Public key**  
Where: Public key tab  
Displays the public key of an asymmetric KMS key\. Authorized users can use this tab to [copy and download the public key](download-public-key.md)\.

**Regionality**  
Where: General configuration section and Regionality tabs  
Indicates whether a KMS key is a single\-Region key, a [multi\-Region primary key](multi-region-keys-overview.md#mrk-primary-key), or a [multi\-Region replica key](multi-region-keys-overview.md#mrk-replica-key)\. This field appears only when the KMS key is a multi\-Region key\.

**Related multi\-Region keys**  
Where: Regionality tab  
Displays all related [multi\-Region primary and replica keys](multi-region-keys-overview.md), except for the current KMS key\. This field appears only when the KMS key is a multi\-Region key\.  
In the **Related multi\-Region keys** section of a primary key, authorized users can [create new replica keys](multi-region-keys-replicate.md)\.

**Replica key**  
Where: Regionality tab  
Indicates that this KMS key is a [multi\-Region replica key](multi-region-keys-overview.md#mrk-replica-key)\. This field appears only when the KMS key is a multi\-Region replica key\.

**Signing algorithms**  
Where: Cryptographic configuration tab  
Lists the signing algorithms that can be used with the KMS key in AWS KMS\. This field appears only when the **Key type** is **Asymmetric** and the **Key usage** is **Sign and verify**\. For information about the signing algorithms that AWS KMS supports, see [RSA key specs for signing and verification](asymmetric-key-specs.md#key-spec-rsa-sign) and [Elliptic curve key specs](asymmetric-key-specs.md#key-spec-ecc)\.

**Status**  
Where: General configuration section  
The key state of the KMS key\. You can use the KMS key in [cryptographic operations](concepts.md#cryptographic-operations) only when the status is **Enabled**\. For a detailed description of each KMS key status and its effect on the operations that you can run on the KMS key, see [Key states of AWS KMS keys](key-state.md)\.

**Tags**  
Where: Tags tab  
Optional key\-value pairs that describe the KMS key\. To add or change the tags for a KMS key, on the **Tags** tab, choose **Edit**\.  
When you add tags to your AWS resources, AWS generates a cost allocation report with usage and costs aggregated by tags\. Tags can also be used to control access to a KMS key\. For information about tagging KMS keys, see [Tagging keys](tagging-keys.md) and [ABAC for AWS KMS](abac.md)\. 

## Customizing your KMS key tables<a name="viewing-console-customize"></a>

You can customize the tables that appear on the **AWS managed keys** and **Customer managed keys** pages in the AWS Management Console to suit your needs\. You can choose the table columns, the number of AWS KMS keys on each page \(**Page size**\), and the text wrap\. The configuration you choose is saved when you confirm it and reapplied whenever you open the pages\. 

**To customize your KMS key tables**

1. On the **AWS managed keys** or **Customer managed keys** page, choose the settings icon \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-icon-settings-new.png)\) in the upper\-right corner of the page\.

1. On the **Preferences** page, choose your preferred settings, and then choose **Confirm**\.

Consider using the **Page size** setting to increase the number of KMS keys displayed on each page, especially if you typically use a device that's easy to scroll\.

The data columns that you display might vary depending on the table, your job role, and the types of KMS keys in the account and Region\. The following table offers some suggested configurations\. For descriptions of the columns, see [Displaying KMS key details](#viewing-console-details)\.

### Suggested KMS key table configurations<a name="configure-console"></a>

You can customize the columns that appear in your KMS key table to display the information you need about your KMS keys\.

**AWS managed keys**  
By default, the **AWS managed key** table displays the **Aliases**, **Key ID**, and **Status** columns\. These columns are ideal for most use cases\.

**Symmetric encryption KMS keys**  
If you use only symmetric encryption KMS keys with key material generated by AWS KMS, the **Aliases**, **Key ID**, **Status**, and **Creation date** columns are likely to be the most useful\.

**Asymmetric KMS keys**  
If you use asymmetric KMS keys, in addition to the **Aliases**, **Key ID**, and **Status** columns, consider adding the **Key type**, **Key spec**, and **Key usage** columns\. These columns will show you whether a KMS key is symmetric or asymmetric, the type of key material, and whether the KMS key can be used for encryption or signing\.

**HMAC KMS keys**  
If you use HMAC KMS keys, in addition to the **Aliases**, **Key ID**, and **Status** columns, consider adding the **Key spec** and **Key usage** columns\. These columns will show you whether a KMS key is an HMAC key\. Because you can't sort KMS keys by key spec or key usage, use aliases and tags to identify your HMAC keys and then use the [filter features](#viewing-console-filter) of the AWS KMS console to filter by aliases or tags\.

**Imported key material**  
If you have KMS keys with [imported key material](importing-keys.md), consider adding the **Origin** and **Expiration date** columns\. These columns will show you whether the key material in a KMS key is imported or generated by AWS KMS and when the key material expires, if at all\. The **Creation date** field displays the date that the KMS key was created \(without key material\)\. It doesn't reflect any characteristic of the key material\.

**Keys in custom key stores**  
If you have KMS keys in [custom key stores](custom-key-store-overview.md), consider adding the **Custom key store ID** column\. A value in this column indicates that the KMS key is in a custom key store, as well as showing which custom key store it's in\.

**Multi\-Region keys**  
If you have [multi\-Region keys](multi-region-keys-overview.md), consider adding the **Regionality** column\. This shows whether a KMS key is a single\-Region key, a [multi\-Region primary key](multi-region-keys-overview.md#mrk-primary-key) or a [multi\-Region replica key](multi-region-keys-overview.md#mrk-replica-key)\.