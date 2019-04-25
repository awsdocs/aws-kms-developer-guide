# Working with Aliases<a name="programming-aliases"></a>

The examples in this topic use the AWS KMS API to create, view, update, and delete aliases\.

An *alias* is an optional display name for a [customer master key \(CMK\)](concepts.md#master_keys)\. Each CMK can have multiple aliases, but each alias points to only one CMK\. The alias name must be unique in the AWS account and region\. To simplify code that runs in multiple regions, you can use the same alias name but point it to a different CMK in each region\. 

You can use AWS KMS API operations to list, create, and delete aliases\. You can also update an alias, which associates an existing alias with a different CMK\. There is no operation to edit or change an alias name\. If you create an alias for a CMK that already has an alias, the operation creates another alias for the same CMK\. To change an alias name, delete the current alias and then create a new alias for the CMK\.

Because an alias is not a property of a CMK, it can be associated with and disassociated from an existing CMK without changing the properties of the CMK\. Deleting an alias does not delete the underlying CMK\.

You can use an alias as the value of the `KeyId` parameter only in the following operations:
+ `DescribeKey`
+ `Encrypt`
+ `GenerateDataKey`
+ `GenerateDataKeyWithoutPlaintext`
+ `ReEncrypt`

Aliases are created in an AWS account and are known only to the account in which you create them\. You cannot use an alias name or alias ARN to identify a CMK in a different AWS account\.

To specify an alias, use the alias name or alias ARN, as shown in the following example\. In either case, be sure to prepend `"alias/"` to the alias name\.

```
// Fully specified ARN
arn:aws:kms:us-west-2:111122223333:alias/ExampleAlias
```

**Topics**
+ [Creating an Alias](#create-alias)
+ [Listing Aliases](#list-aliases)
+ [Updating an Alias](#update-alias)
+ [Deleting an Alias](#delete-alias)

## Creating an Alias<a name="create-alias"></a>

To create an alias, use the [CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html) operation\. The alias must be unique in the account and region\. If you create an alias for a CMK that already has an alias, `CreateAlias` creates another alias to the same CMK\. It does not replace the existing alias\.

You cannot create an alias that begins with `aws/`\. The `aws/` prefix is reserved by Amazon Web Services for [AWS managed CMKs](concepts.md#master_keys)\.

This example uses the AWS KMS client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [createAlias method](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#createAlias-com.amazonaws.services.kms.model.CreateAliasRequest-) in the *AWS SDK for Java API Reference*\.

```
// Create an alias for a CMK
//
String aliasName = "alias/projectKey1";
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
String targetKeyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

CreateAliasRequest req = new CreateAliasRequest().withAliasName(aliasName).withTargetKeyId(targetKeyId);
kmsClient.createAlias(req);
```

------
#### [ C\# ]

For details, see the [CreateAlias method](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceCreateAliasCreateAliasRequest.html) in the *AWS SDK for \.NET*\.

```
// Create an alias for a CMK
//
String aliasName = "alias/projectKey1";
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
String targetKeyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

CreateAliasRequest createAliasRequest = new CreateAliasRequest()
{
    AliasName = aliasName,
    TargetKeyId = targetKeyId
};
kmsClient.CreateAlias(createAliasRequest);
```

------
#### [ Python ]

For details, see the [create\_alias method](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.create_alias) in the AWS SDK for Python \(Boto 3\)\.

```
# Create an alias for a CMK

alias_name = 'alias/projectKey1'
# Replace the following fictitious CMK ARN with a valid CMK ID or ARN
target_key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

response = kms_client.create_alias(
    AliasName=alias_name,
    TargetKeyId=key_id
)
```

------
#### [ Ruby ]

For details, see the [create\_alias](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#create_alias-instance_method) instance method in the [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Create an alias for a CMK

aliasName = 'alias/projectKey1'
# Replace the following fictitious CMK ARN with a valid CMK ID or ARN
targetKeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

response = kmsClient.create_alias({
  alias_name: aliasName,
  target_key_id: targetKeyId
})
```

------
#### [ PHP ]

For details, see the [CreateAlias method](https://docs.aws.amazon.com/aws-sdk-php/latest/api-kms-2014-11-01.html#createalias) in the *AWS SDK for PHP*\.

```
// Create an alias for a CMK
//
$aliasName = "alias/projectKey1";
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';


$result = $KmsClient->createAlias([
    'AliasName' => $aliasName, 
    'TargetKeyId' =>  $keyId, 
]);
```

------
#### [ Node\.js ]

For details, see the [createAlias property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#createAlias-property) in the *AWS SDK for JavaScript in Node\.js*\.

```
// Create an alias for a CMK
//
const AliasName = 'alias/projectKey1';

// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
const TargetKeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
kmsClient.createAlias({ AliasName, TargetKeyId }, (err, data) => {
  ...
});
```

------

## Listing Aliases<a name="list-aliases"></a>

To list aliases in the account and region, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\.

By default, the ListAliases command returns all aliases in the account and region\. This includes aliases that you created and associated with your [customer managed CMKs](concepts.md#master_keys), and aliases that AWS created and associated with your [AWS managed CMKs](concepts.md#master_keys)\. The response might also include aliases that have no `TargetKeyId` field\. These are predefined aliases that AWS has created but has not yet associated with a CMK\.

This example uses the AWS KMS client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details about the Java implementation, see the [listAliases method](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#listAliases-com.amazonaws.services.kms.model.ListAliasesRequest-) in the *AWS SDK for Java API Reference*\.

```
// List the aliases in this AWS account
//
Integer limit = 10;

ListAliasesRequest req = new ListAliasesRequest().withLimit(limit);
ListAliasesResult result = kmsClient.listAliases(req);
```

------
#### [ C\# ]

For details, see the [ListAliases method](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceListAliasesListAliasesRequest.html) in the *AWS SDK for \.NET*\.

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
#### [ Python ]

For details, see the [list\_aliases method](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.list_aliases) in the AWS SDK for Python \(Boto 3\)\.

```
# List the aliases in this AWS account

response = kms_client.list_aliases(
    Limit=10
)
```

------
#### [ Ruby ]

For details, see the [list\_aliases](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#list_aliases-instance_method) instance method in the [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# List the aliases in this AWS account

response = kmsClient.list_aliases({
  limit: 10
})
```

------
#### [ PHP ]

For details, see the [List Aliases method](https://docs.aws.amazon.com/aws-sdk-php/latest/api-kms-2014-11-01.html#listaliases) in the *AWS SDK for PHP*\.

```
// List the aliases in this AWS account
//
$limit = 10;

$result = $KmsClient->listAliases([
    'Limit' => $limit,
]);
```

------
#### [ Node\.js ]

For details, see the [listAliases property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#listAliases-property) in the *AWS SDK for JavaScript in Node\.js*\.

```
// List the aliases in this AWS account
//
const Limit = 10;
kmsClient.listAliases({ Limit }, (err, data) => {
  ...
});
```

------

To list only the aliases that are associated with a particular CMK, use the `KeyId` parameter\. Its value can be the ID or Amazon Resource Name \(ARN\) of any CMK in the region\. You cannot specify an alias name or alias ARN\.

------
#### [ Java ]

For details about the Java implementation, see the [listAliases method](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#listAliases-com.amazonaws.services.kms.model.ListAliasesRequest-) in the *AWS SDK for Java API Reference*\.

```
// List the aliases for one CMK
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

ListAliasesRequest req = new ListAliasesRequest().withKeyId(keyId);
ListAliasesResult result = kmsClient.listAliases(req);
```

------
#### [ C\# ]

For details, see the [ListAliases method](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceListAliasesListAliasesRequest.html) in the *AWS SDK for \.NET*\.

```
// List the aliases for one CMK
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

ListAliasesRequest listAliasesRequest = new ListAliasesRequest()
{
    KeyId = keyId
};
ListAliasesResponse listAliasesResponse = kmsClient.ListAliases(listAliasesRequest);
```

------
#### [ Python ]

For details, see the [list\_aliases method](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.list_aliases) in the AWS SDK for Python \(Boto 3\)\.

```
# List the aliases for one CMK

# Replace the following fictitious CMK ARN with a valid CMK ID or ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

response = kms_client.list_aliases(
    KeyId=key_id
)
```

------
#### [ Ruby ]

For details, see the [list\_aliases](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#list_aliases-instance_method) instance method in the [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# List the aliases for one CMK

# Replace the following fictitious CMK ARN with a valid CMK ID or ARN
keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

response = kmsClient.list_aliases({
  key_id: keyId
})
```

------
#### [ PHP ]

For details, see the [List Aliases method](https://docs.aws.amazon.com/aws-sdk-php/latest/api-kms-2014-11-01.html#listaliases) in the *AWS SDK for PHP*\.

```
// List the aliases for one CMK
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';

$result = $KmsClient->listAliases([
    'KeyId' => $keyId,
]);
```

------
#### [ Node\.js ]

For details, see the [listAliases property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#listAliases-property) in the *AWS SDK for JavaScript in Node\.js*\.

```
// List the aliases for one CMK
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
const KeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
kmsClient.listAliases({ KeyId }, (err, data) => {
  ...
});
```

------

## Updating an Alias<a name="update-alias"></a>

To associate an existing alias with a different CMK, use the [UpdateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html) operation\. 

This example uses the AWS KMS client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details about the Java implementation, see the [updateAlias method](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#updateAlias-com.amazonaws.services.kms.model.UpdateAliasRequest-) in the *AWS SDK for Java API Reference*\.

```
// Updating an alias
//
String aliasName = "alias/projectKey1";
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
String targetKeyId = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321";

UpdateAliasRequest req = new UpdateAliasRequest()
     .withAliasName(aliasName)
     .withTargetKeyId(targetKeyId);
     
kmsClient.updateAlias(req);
```

------
#### [ C\# ]

For details, see the [UpdateAlias method](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceUpdateAliasUpdateAliasRequest.html) in the *AWS SDK for \.NET*\.

```
// Updating an alias
//
String aliasName = "alias/projectKey1";
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
String targetKeyId = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321";

UpdateAliasRequest updateAliasRequest = new UpdateAliasRequest()
{
    AliasName = aliasName,
    TargetKeyId = targetKeyId
};

kmsClient.UpdateAlias(updateAliasRequest);
```

------
#### [ Python ]

For details, see the [update\_alias method](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.update_alias) in the AWS SDK for Python \(Boto 3\)\.

```
# Updating an alias

alias_name = 'alias/projectKey1'
# Replace the following fictitious CMK ARN with a valid CMK ID or ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321'

response = kms_client.update_alias(
    AliasName=alias_name,
    TargetKeyID=key_id
)
```

------
#### [ Ruby ]

For details, see the [update\_alias](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#update_alias-instance_method) instance method in the [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Updating an alias

aliasName = 'alias/projectKey1'
# Replace the following fictitious CMK ARN with a valid CMK ID or ARN
keyId = 'arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321'

response = kmsClient.update_alias({
  alias_name: aliasName,
  target_key_id: keyId
})
```

------
#### [ PHP ]

For details, see the [UpdateAlias method](https://docs.aws.amazon.com/aws-sdk-php/latest/api-kms-2014-11-01.html#updatealias) in the *AWS SDK for PHP*\.

```
// Updating an alias
//
$aliasName = "alias/projectKey1";

// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321';

$result = $KmsClient->updateAlias([
    'AliasName' => $aliasName, 
    'TargetKeyId' =>  $keyId, 
]);
```

------
#### [ Node\.js ]

For details, see the [updateAlias property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#updateAlias-property) in the *AWS SDK for JavaScript in Node\.js*\.

```
// Updating an alias
//
const AliasName = 'alias/projectKey1';
 
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
const TargetKeyId = 'arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321';
kmsClient.updateAlias({ AliasName, TargetKeyId }, (err, data) => {
  ...
});
```

------

## Deleting an Alias<a name="delete-alias"></a>

To delete an alias, use the [DeleteAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_DeleteAlias.html) operation\. Deleting an alias has no effect on the underlying CMK\. 

This example uses the AWS KMS client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [deleteAlias method](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#deleteAlias-com.amazonaws.services.kms.model.DeleteAliasRequest-) in the *AWS SDK for Java API Reference*\.

```
// Delete an alias for a CMK
//
String aliasName = "alias/projectKey1";

DeleteAliasRequest req  = new DeleteAliasRequest().withAliasName(aliasName);
kmsClient.deleteAlias(req);
```

------
#### [ C\# ]

For details, see the [DeleteAlias method](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceDeleteAliasDeleteAliasRequest.html) in the *AWS SDK for \.NET*\.

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
#### [ Python ]

For details, see the [delete\_alias method](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.delete_alias) in the AWS SDK for Python \(Boto 3\)\.

```
# Delete an alias for a CMK

alias_name = 'alias/projectKey1'

response = kms_client.delete_alias(
    AliasName=alias_name
)
```

------
#### [ Ruby ]

For details, see the [delete\_alias](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#delete_alias-instance_method) instance method in the [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Delete an alias for a CMK

aliasName = 'alias/projectKey1'

response = kmsClient.delete_alias({
  alias_name: aliasName
})
```

------
#### [ PHP ]

For details, see the [DeleteAlias method](https://docs.aws.amazon.com/aws-sdk-php/latest/api-kms-2014-11-01.html#deletealias) in the *AWS SDK for PHP*\.

```
// Delete an alias for a CMK
//
$aliasName = "alias/projectKey1";

$result = $KmsClient->deleteAlias([
    'AliasName' => $aliasName, 
]);
```

------
#### [ Node\.js ]

For details, see the [deleteAlias property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#deleteAlias-property)\) in the *AWS SDK for JavaScript in Node\.js*\.

```
// Delete an alias for a CMK
//
const AliasName = 'alias/projectKey1';
kmsClient.deleteAlias({ AliasName }, (err, data) => {
  ...
});
```

------