# Working with Aliases<a name="programming-aliases"></a>

The examples in this topic use the AWS KMS API to create, view, update, and delete aliases\.

An *alias* is an optional display name for a [customer master key \(CMK\)](concepts.md#master_keys)\. 

Each CMK can have multiple aliases, but each alias points to only one CMK\. The alias name must be unique in the AWS account and region\. To simplify code that runs in multiple regions, you can use the same alias name, but point it to a different CMK in each region\. 

You can use AWS KMS API operations to list, create, and delete aliases\. You can also update an alias, which associates an existing alias with a different CMK\. There is no operation to edit or change an alias name\. If you create an alias for a CMK that already has an alias, the operation creates another alias for the same CMK\. To change an alias name, delete the current alias and then create a new alias for the CMK\.

Because an alias is not a property of a CMK, it can be associated with and disassociated from an existing CMK without changing the properties of the CMK\. Deleting an alias does not delete the underlying CMK\.

You can use an alias as the value of the `KeyId` parameter only in the following operations:
+ `DescribeKey`
+ `Encrypt`
+ `GenerateDataKey`
+ `GenerateDataKeyWithoutPlaintext`
+ `ReEncrypt`

Aliases are created in an AWS account and are known only to the account in which you create them\. You cannot use an alias name or alias ARN to identify a CMK in a different AWS account\.

To specify an alias, use the alias name or alias ARN, as shown in the following example\. In either case, be sure to prepend "alias/" to the alias name\.

```
// Fully specified ARN
arn:aws:kms:us-west-2:111122223333:alias/ExampleAlias
    
// Alias name (prefixed with "alias/")
alias/ExampleAlias
```

**Topics**
+ [Creating an Alias](#create-alias)
+ [Listing Aliases](#list-aliases)
+ [Updating an Alias](#update-alias)
+ [Deleting an Alias](#delete-alias)

## Creating an Alias<a name="create-alias"></a>

To create an alias, use the [CreateAlias](http://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html) operation\. The alias must be unique in the account and region\. If you create an alias for a CMK that already has an alias, CreateAlias creates another alias to the same CMK\. It does not replace the existing alias\.

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [createAlias method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#createAlias-com.amazonaws.services.kms.model.CreateAliasRequest-) in the *AWS SDK for Java API Reference*\.

```
// Create an alias for a CMK
//
String aliasName = "alias/projectKey1";
String targetKeyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

CreateAliasRequest req = new CreateAliasRequest().withAliasName(aliasName).withTargetKeyId(targetKeyId);
kmsClient.createAlias(req);
```

------
#### [ C\# ]

For details, see the [CreateAlias method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceCreateAliasCreateAliasRequest.html) in the *AWS SDK for \.NET*\.

```
// Create an alias for a CMK
//
String aliasName = "alias/projectKey1";
String targetKeyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

CreateAliasRequest createAliasRequest = new CreateAliasRequest()
{
    AliasName = aliasName,
    TargetKeyId = targetKeyId
};
kmsClient.CreateAlias(createAliasRequest);
```

------

## Listing Aliases<a name="list-aliases"></a>

To list all aliases, use the [ListAliases](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. The response includes aliases that are defined by AWS services, but are not associated with a CMK\.

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details about the Java implementation, see the [listAliases method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#listAliases-com.amazonaws.services.kms.model.ListAliasesRequest-) in the *AWS SDK for Java API Reference*\.

```
// List the aliases in this AWS account
//
Integer limit = 10;

ListAliasesRequest req = new ListAliasesRequest().withLimit(limit);
ListAliasesResult result = kmsClient.listAliases(req);
```

------
#### [ C\# ]

For details, see the [EnableKey method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceListAliasesListAliasesRequest.html) in the *AWS SDK for \.NET*\.

```
// List the aliases in this AWS account
//
int limit = 10;

ListAliasesRequest listAliasesRequest = new ListAliasesRequest()
{
    Limit = limit
};
ListAliasesResponse listAliasesResponse = kmsClient.ListAliases(listAliasesRequest);
```

------

## Updating an Alias<a name="update-alias"></a>

To associate an existing alias with a different CMK, use the [UpdateAlias](http://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html) operation\. 

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details about the Java implementation, see the [updateAlias method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#updateAlias-com.amazonaws.services.kms.model.UpdateAliasRequest-) in the *AWS SDK for Java API Reference*\.

```
// Updating an alias
//
String aliasName = "alias/projectKey1";
String targetKeyId = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321";

UpdateAliasRequest req = new UpdateAliasRequest()
     .withAliasName(aliasName)
     .withTargetKeyId(targetKeyId);
     
kmsClient.updateAlias(req);
```

------
#### [ C\# ]

For details, see the [EnableKey method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceUpdateAliasUpdateAliasRequest.html) in the *AWS SDK for \.NET*\.

```
// Updating an alias
//
String aliasName = "alias/projectKey1";
String targetKeyId = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321";

UpdateAliasRequest updateAliasRequest = new UpdateAliasRequest()
{
    AliasName = aliasName,
    TargetKeyId = targetKeyId
};

kmsClient.UpdateAlias(updateAliasRequest);
```

------

## Deleting an Alias<a name="delete-alias"></a>

To delete an alias, use the [DeleteAlias](http://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteAlias.html) operation\. Deleting an alias has no effect on the underlying CMK\. 

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [deleteAlias method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#deleteAlias-com.amazonaws.services.kms.model.DeleteAliasRequest-) in the *AWS SDK for Java API Reference*\.

```
// Delete an alias for a CMK
//
String aliasName = "alias/projectKey1";

DeleteAliasRequest req  = new DeleteAliasRequest().withAliasName(aliasName);
kmsClient.deleteAlias(req);
```

------
#### [ C\# ]

For details, see the [DeleteAlias method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceDeleteAliasDeleteAliasRequest.html) in the *AWS SDK for \.NET*\.

```
// Delete an alias for a CMK
//
String aliasName = "alias/projectKey1";

DeleteAliasRequest deleteAliasRequest = new DeleteAliasRequest()
{
    AliasName = aliasName
};
kmsClient.DeleteAlias(deleteAliasRequest);
```

------