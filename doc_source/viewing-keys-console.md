# Viewing CMKs in the console<a name="viewing-keys-console"></a>

In the AWS Management Console, you can view lists of your CMKs and details about each CMK\.

**Topics**
+ [Navigating to the key tables](#viewing-console-navigate)
+ [Sorting and filtering your CMKs](#viewing-console-filter)
+ [Displaying CMK details](#viewing-console-details)
+ [Customizing your CMK tables](#viewing-console-customize)

## Navigating to the key tables<a name="viewing-console-navigate"></a>

The AWS KMS customer master keys \(CMKs\) in each account and region are displayed in tables\. There are separate tables for the CMKs that you create and the CMKs that AWS services create for you\. 

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. To view the keys in your account that you create and manage, in the navigation pane choose **Customer managed keys**\. To view the keys in your account that AWS creates and manages for you, in the navigation pane, choose **AWS managed keys**\. For information about the different types of CMKs, see [Customer master keys](concepts.md#master_keys)\.
**Tip**  
To view [AWS managed CMKs](concepts.md#aws-managed-cmk) that are missing an alias, use the **Customer managed keys** page\.  
The AWS KMS console also displays the custom key stores in the account and Region\. CMKs that you create in custom key stores appear on the **Customer managed keys** page\. For information about custom key stores, see [Using a custom key store](custom-key-store-overview.md)\.

## Sorting and filtering your CMKs<a name="viewing-console-filter"></a>

To make it easier to find your CMKs in the console, you can sort and filter them\. 

**Note**  
The **Key type** column is [displayed optionally](#viewing-console-customize) and is available only in AWS Regions where AWS KMS supports asymmetric CMKs\.

**Sort**  
You can sort customer managed CMKs in ascending or descending order by their column values\. This feature sorts all CMKs in the table, even if they don't appear on the current table page\.  
Sortable columns are indicated by an arrow beside the column name\. On the **AWS managed keys** page, you can sort by **Alias** or **Key ID**\. On the **Customer managed keys** page, you can sort by **Alias**, **Key ID**, or **Key type**\.  
To sort in ascending order, choose the column heading until the arrow points upward\. To sort in descending order, choose the column heading until the arrow points downward\. You can sort by only one column at a time\.  
For example, you can sort CMKs in ascending order by key ID, instead of alias, which is the default\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-sort.png)
When your sort CMKs on the **Customer master keys** page in ascending order by **Key type**, all asymmetric keys are displayed before all symmetric keys\.

**Filter**  
You can filter CMKs by their column values\. The filter applies to all CMKs in the table, even if they don't appear on the current table page\. The filter is not case\-sensitive\.  
Filterable columns are listed in the filter box\. On the **AWS managed keys** page, you can filter by **Alias** and **Key ID**\. On the **Customer managed keys** page, you can filter by **Alias**, **Key ID**, and **Key type**\.  
To filter by the value in a particular column, choose the filter, choose the column name, and then choose from the list of actual column values\. After choosing a column, you can also type all or part of the column value\. You'll see a preview of the results before you make your choice\.  
For example, to display CMKs with an alias name that contains with `aws/e`, choose the filter box, choose **Alias**, type `aws/e`, and then press `Enter` or `Return` to add the filter\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/filter-alias.png)
To display only asymmetric CMKs on the **Customer master keys** page, click the filter box, choose **Key type** and then choose **Key type: Asymmetric**\. The **Asymmetric** option appears only when you have asymmetric CMKs in the table\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/filter-keytype.png)
To search for text in all filterable columns, in the filter box, type all or part of a column value\. You'll see a preview of the results before you make your choice\.  
For example, to display CMKs with `test` in any of the column values, type `test` in the filter box\. The preview shows the CMKs that the filter will select\. In this case, `test` appears only in the **Alias** column\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/filter-all-properties.png)
You can have multiple filters at the same time\. When you add additional filters, you can also select a logical operator\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/filter-multi-properties.png)

## Displaying CMK details<a name="viewing-console-details"></a>

The details page for each CMK displays the properties of the CMK\. It differs slightly for the different types of CMKs\. 

To display detailed information about a CMK:

1. To display the details page for a CMK, on the **AWS managed keys** or **Customer managed keys** page, choose the alias or key ID of the CMK\. 

1. To display all details, expand the **General configuration** and **Cryptographic configuration** sections of the page\. If the CMK is configured for imported key material, the page also has a **Key materials** section that you can expand\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-detail-view-symmetric-sm.png)

The details page for a CMK includes a **General Configuration** section that displays the basic properties of the CMK, a **Cryptographic Configuration** section that displays the cryptographic properties of the CMK, and a tabbed display that includes the key policy, tags, key rotation \(for symmetric CMKs\), and public key \(for asymmetric CMKs\)\.

The following list describes the fields in the detailed display\. Some of these fields are also available as columns in the table display\.

**Alias**  
A friendly name for the CMK\. The **Alias** field in the console lists only one alias\. To find all aliases for the CMK, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\.

**ARN**  
The Amazon Resource Name \(ARN\) of the CMK\. This value uniquely identifies the CMK\. You can use it to identify the CMK in AWS KMS API operations\.

**Creation date**  
The date and time that the CMK was created\. This value is displayed in local time for the device\. The time zone does not depend on the Region\.  
Unlike **Expiration**, the creation refers only to the CMK, not its key material\. 

**CloudHSM cluster ID**  
The cluster ID of the AWS CloudHSM cluster that contains the key material for the CMK\. This field appears only when the CMK is created in an AWS KMS [custom key store](custom-key-store-overview.md)\.  
If you click the CloudHSM cluster ID, it opens the **Clusters** page in the AWS CloudHSM console\.

**Custom key store ID**  
The ID of the [custom key store](custom-key-store-overview.md) that contains the CMK\. This field appears only when the CMK is created in an AWS KMS custom key store\.  
If you click the custom key store ID, it opens the **Custom key stores** page in the AWS KMS console\.

**Custom key store name**  
The name of the [custom key store](custom-key-store-overview.md) that contains the CMK\. This field appears only when the CMK is created in an AWS KMS custom key store\.

**Description**  
A brief, optional description of the CMK\. To add or update the description of a customer managed CMK, above **General Configuration**, choose **Edit**\.

**Encryption algorithms**  
Lists the encryption algorithms that can be used with the CMK in AWS KMS\. This field appears only when the **Key type** is **Asymmetric** and the **Key usage** is **Encrypt and decrypt**\. For information about the encryption algorithms that AWS KMS supports, see [SYMMETRIC\_DEFAULT key spec](symm-asymm-choose.md#key-spec-symmetric-default) and [RSA key specs for encryption and decryption](symm-asymm-choose.md#key-spec-rsa-encryption)\.

**Expiration date**  
The date and time when the key material for the CMK expires\. This field appears only for CMKs with [imported key material](importing-keys.md), that is, when the **Origin** is **External** and the CMK has key material that expires\.

**Key policy**  
Controls access to the CMK along with [IAM policies](iam-policies.md) and [grants](grants.md)\. Every CMK has one key policy\. It is the only mandatory authorization element\. To change the key policy of a customer managed CMK, on the **Key policy** tab, choose **Edit**\. For details, see [Using key policies in AWS KMS](key-policies.md)\.

**Key rotation**  
Enables and disables [automatic key rotation](rotate-keys.md) every year\.   
To change the key rotation status of a [customer managed CMK](concepts.md#customer-cmk), use the checkbox on the **Key rotation** tab\. All [AWS managed CMKs](concepts.md#aws-managed-cmk) are automatically rotated every three years\.

**Key spec**  
The type of key material in the CMK\. AWS KMS supports symmetric CMKs \(SYMMETRIC\_DEFAULT\), CMKs for RSA keys of different lengths, and elliptic curve keys with different curves\. For details, see [Key spec](concepts.md#key-spec)\.

**Key type**  
Indicates whether the CMK is **Symmetric** or **Asymmetric**\.

**Key usage**  
Indicates whether a CMK can be used for **Encrypt and decrypt** or **Sign and verify**\. Only asymmetric CMKs can be used to sign and verify\. For details, see [Key usage](concepts.md#key-usage)\.

**Origin**  
The source of the key material for the CMK\. Valid values are **AWS\_KMS** for key material that AWS KMS generates, **EXTERNAL** for [imported key material](importing-keys.md), and **AWS\_CloudHSM** for CMKs in [custom key stores](custom-key-store-overview.md)\.

**Public key**  
Displays the public key of an asymmetric CMK\. Authorized users can use this tab to [copy and download the public key](download-public-key.md)\.

**Signing algorithms**  
Lists the signing algorithms that can be used with the CMK in AWS KMS\. This field appears only when the **Key type** is **Asymmetric** and the **Key usage** is **Sign and verify**\. For information about the signing algorithms that AWS KMS supports, see [RSA key specs for signing and verification](symm-asymm-choose.md#key-spec-rsa-sign) and [Elliptic curve key specs](symm-asymm-choose.md#key-spec-ecc)\.

**Status**  
The key state of the CMK\. You can use the CMK in [cryptographic operations](concepts.md#cryptographic-operations) only when the status is **Enabled**\. For a detailed description of each CMK status and its effect on the operations that you can run on the CMK, see [Key state: Effect on your CMK](key-state.md)\.

**Tags**  
Optional key\-value pairs that describe the CMK\. To add or change the tags for a CMK, on the **Tags** tab, choose **Edit**\.  
When you add tags to your AWS resources, AWS generates a cost allocation report with usage and costs aggregated by tags\. For information about tagging CMKs, see [Tagging keys](tagging-keys.md)\.

## Customizing your CMK tables<a name="viewing-console-customize"></a>

You can customize the tables that appear on the **AWS managed keys** and **Customer managed keys** pages in the AWS Management Console to suit your needs\. You can choose the table columns, the number of customer master keys \(CMKs\) on each page \(**Page size**\), and the text wrap\. The configuration you choose is saved when you confirm it and reapplied whenever you open the pages\. 

**To customize your CMK tables**

1. On the **AWS managed keys** or **Customer managed keys** page, choose the settings icon \(![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-icon-settings-new.png)\) in the upper\-right corner of the page\.

1. On the **Preferences** page, choose your preferred settings, and then choose **Confirm**\.

Consider using the **Page size** setting to increase the number of CMKs displayed on each page, especially if you typically use a device that's easy to scroll\.

The data columns that you display might vary depending on the table, your job role, and the types of CMKs in the account and Region\. The following table offers some suggested configurations\. For descriptions of the columns, see [Displaying CMK details](#viewing-console-details)\.

### Suggested CMK table configurations<a name="configure-console"></a>

You can customize the columns that appear in your CMK table to display the information you need about your CMKs\.

**AWS managed keys**  
By default, the **AWS managed keys** table displays the **Alias**, **Key ID**, and **Status** columns\. These columns are ideal for most use cases\.

**Symmetric customer managed keys**  
If you use only symmetric CMKs with key material generated by AWS KMS, the **Alias**, **Key ID**, **Status**, and **Creation date** columns are likely to be the most useful\.

**Asymmetric customer managed keys**  
If you use asymmetric CMKs, in addition to the **Alias**, **Key ID**, and **Status** columns, consider adding the **Key type**, **Key spec**, and **Key usage** columns\. These columns will show you whether a CMK is symmetric or asymmetric, the type of key material, and whether the CMK can be used for encryption or signing\.

**Imported key material**  
If you have CMKs with [imported key material](importing-keys.md), in addition to the **Alias**, **Key ID**, and **Status** columns, consider adding the **Origin** and **Expiration date** columns\. These columns will show you whether the key material in a CMK is imported or generated by AWS KMS and when the key material expires, if at all\. The **Creation date** field displays the date that the CMK was created \(without key material\)\. It doesn't reflect any characteristic of the key material\.

**Keys in custom key stores**  
If you have CMKs in [custom key stores](custom-key-store-overview.md), in addition to the **Alias**, **Key ID**, and **Status** columns, consider adding the **Custom key store ID** column\. A value in this column indicates that the CMK is in a custom key store, as well as showing which custom key store it's in\.