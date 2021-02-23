# About aliases<a name="alias-about"></a>

Learn how aliases work in AWS KMS\.

**An alias is an independent AWS resource**  
An alias is not a property of a CMK\. The actions that you take on the alias don't affect its associated CMK\. You can create an alias for a CMK and then update the alias so it's associated with a different CMK\. You can even delete the alias without any effect on the associated CMK\. However, if you delete a CMK, all aliases associated with that CMK are deleted\.  
If you specify an alias as the resource in an IAM policy, the policy refers to the alias, not to the associated CMK\.

**Each alias has two formats**  
When you create an alias, you specify the alias name\. AWS KMS creates the alias ARN for you\.  
+ An [alias ARN](concepts.md#key-id-alias-ARN) is an Amazon Resource Name \(ARN\) that uniquely identifies the alias\. 

  ```
  # Alias ARN
  arn:aws:kms:us-west-2:111122223333:alias/<alias-name>
  ```
+ An [alias name](concepts.md#key-id-alias-name) that is unique in the account and Region\. In the AWS KMS API, the alias name is always prefixed by `alias/`\. That prefix is omitted in the AWS KMS console\.

  ```
  # Alias name
  alias/<alias-name>
  ```

**Each alias is associated with one CMK at a time**  
The alias and its CMK must be in the same account and Region\.   
You can associate an alias with any [customer managed CMK](concepts.md#customer-cmk) in the same AWS account and Region\. However, you do not have permission to associate an alias with an [AWS managed CMK](concepts.md#aws-managed-cmk)\.  
For example, this [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) output shows that the `test-key` alias is associated with exactly one target CMK, which is represented by the `TargetKeyId` property\.  

```
{
     "AliasName": "alias/test-key",
     "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/test-key",
     "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
     "CreationDate": 1593622000.191,
     "LastUpdatedDate: 1593622000.191
}
```

**Multiple aliases can be associated with the same CMK**  
For example, you can associate the `test-key` and `project-key` aliases with the same CMK\.  

```
{
     "AliasName": "alias/test-key",
     "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/test-key",
     "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
     "CreationDate": 1593622000.191,
     "LastUpdatedDate: 1593622000.191
},
{
     "AliasName": "alias/project-key",
     "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/project-key",
     "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
     "CreationDate": 1516435200.399,
     "LastUpdatedDate: 1516435200.399
}
```

**An alias must be unique in an account and Region**  
For example, you can have only one `test-key` alias in each account and Region\. Aliases are case\-sensitive, but aliases that differ only in their capitalization are very prone to error\. You cannot change an alias name\. However, you can delete the alias and create a new alias with the desired name\.

**You can create an alias with the same name in different Regions**  
For example, you can have a `finance-key` alias in US East \(N\. Virginia\) and a `finance-key` alias in Europe \(Frankfurt\)\. Each alias would be associated with a CMK in its Region\. If your code refers to an alias name like `alias/finance-key`, you can run it in multiple Regions\. In each Region, it uses a different CMK\. For details, see [Using aliases in your applications](alias-using.md)\.

**You can change the CMK associated with an alias**  
You can use the [UpdateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html) operation to associate an alias with a different CMK\. For example, if the `finance-key` alias is associated with the `1234abcd-12ab-34cd-56ef-1234567890ab` CMK, you can update it so it is associated with the `0987dcba-09fe-87dc-65ba-ab0987654321` CMK\.  
However, the current and new CMK must be the same type \(both symmetric or both asymmetric\), and they must have the same [key usage](concepts.md#key-usage) \(ENCRYPT\_DECRYPT or SIGN\_VERIFY\)\. This restriction prevents errors in code that uses aliases\. If you must associate an alias with a different type of key, and you have mitigated the risks, you can delete and recreate the alias\.

**Some CMKs don't have aliases**  
When you create a CMK in the AWS KMS console, you must give it a new alias\. But an alias is not required when you use the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation to create a CMK\. Also, you can use the [UpdateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html) operation to change the CMK that's associated with an alias and the[DeleteAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteAlias.html) operation to delete an alias\. As a result, some CMKs might have several aliases, and some might have none\.

**AWS creates aliases in your account**  
AWS creates aliases in your account for [AWS managed CMKs](concepts.md#aws-managed-cmk)\. These aliases have names of the form `alias/aws/<service-name>`, such as `alias/aws/s3`\.   
Some AWS aliases have no CMK\. These predefined aliases are usually associated with an AWS managed CMK when you start using the service\.

**Use aliases to identify CMKs**  
You can use an [alias name](concepts.md#key-id-alias-name) or [alias ARN](concepts.md#key-id-alias-ARN) to identify a CMK in AWS KMS [cryptographic operations](concepts.md#cryptographic-operations) and in the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. However, you cannot use an alias name or alias ARN in API operations that manage CMKs, such as [DisableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) or [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html)\. For information about the valid [key identifiers](concepts.md#key-id) for each AWS KMS API operation, see the descriptions of the `KeyId` parameters in the *AWS Key Management Service API Reference*\.  
You cannot use an alias name or alias ARN to identify a CMK in the `Resource` element of an [IAM policy](iam-policies.md)\. This restriction prevents errors that might occur if you intended to control access to a particular CMK, but the alias is deleted or updated to a different CMK\.  