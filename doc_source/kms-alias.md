# Using aliases<a name="kms-alias"></a>

An *alias* is a friendly name for a [AWS KMS key](concepts.md#kms_keys)\. For example, an alias lets you refer to a KMS key as `test-key` instead of `1234abcd-12ab-34cd-56ef-1234567890ab`\. 

You can use an alias to identify a KMS key in the AWS KMS console, in the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation, and in [cryptographic operations](concepts.md#cryptographic-operations), such as [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) and [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)\. 

Aliases also make it easy to recognize an [AWS managed key](concepts.md#aws-managed-cmk)\. Aliases for these KMS keys always have the form `aws/<service-name>`\. For example, the alias for the AWS managed key for Amazon DynamoDB is `aws/dynamodb`\. You can establish similar alias standards for your projects, such as prefacing your aliases with the name of a project or category\.

You can also allow and deny access to KMS keys based on their aliases without editing policies or managing grants\. This feature is part of AWS KMS support for [attribute\-based access control](abac.md) \(ABAC\)\. For details, see [Using aliases to control access to KMS keys](alias-authorization.md)\.

Much of the power of aliases come from your ability to change the KMS key associated with an alias at any time\. Aliases can make your code easier to write and maintain\. For example, suppose you use an alias to refer to a particular KMS key and you want to change the KMS key\. In that case, just associate the alias with a different KMS key\. You don't need to change your code\. 

Aliases also make it easier to reuse the same code in different AWS Regions\. Create aliases with the same name in multiple Regions and associate each alias with a KMS key in its Region\. When the code runs in each Region, the alias refers to the associated KMS key in that Region\. For an example, see [Using aliases in your applications](alias-using.md)\.

The AWS KMS API provides full control of aliases in each account and Region\. The API includes operations to create an alias \([CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html)\), view alias names and alias ARNs \([ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html)\), change the KMS key associated with an alias \([UpdateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html)\), and delete an alias \([DeleteAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteAlias.html)\)\. For examples of managing aliases multiple programming languages, see [Working with aliases](programming-aliases.md)\.

The following resources can help you learn more:
+ For information about KMS key identifiers, including aliases, see [Key identifiers \(KeyId\)](concepts.md#key-id)\.
+ For help finding the aliases associated with a KMS key, see [Finding the alias name and alias ARN](find-cmk-alias.md)
+ For information about resource quotas for aliases and rate quotas for API operations related to aliases, see [Quotas](limits.md)\.
+ For examples of creating and managing aliases in multiple programming languages, see [Working with aliases](programming-aliases.md)\.

**Topics**
+ [About aliases](alias-about.md)
+ [Managing aliases](alias-manage.md)
+ [Using aliases in your applications](alias-using.md)
+ [Controlling access to aliases](alias-access.md)
+ [Using aliases to control access to KMS keys](alias-authorization.md)
+ [Finding aliases in AWS CloudTrail logs](alias-ct.md)