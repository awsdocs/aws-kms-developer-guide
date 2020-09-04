# Working with grants<a name="programming-grants"></a>

The examples in this topic use the AWS KMS API to create, view, retire, and revoke grants on AWS KMS customer master keys \(CMKs\)\. For more details about using grants in AWS KMS, see [Using grants](grants.md)\.

**Topics**
+ [Creating a grant](#create-grant)
+ [Viewing a grant](#list-grants)
+ [Retiring a grant](#retire-grant)
+ [Revoking a grant](#revoke-grant)

## Creating a grant<a name="create-grant"></a>

To create a grant for an AWS KMS customer master key, use the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation\. The response includes only the grant ID and grant token\. To get detailed information about the grant, use the [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation, as shown in [Viewing a grant](#list-grants)\.

These examples create a grant that allows Alice, an IAM user in the account, to call the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation on the CMK identified by the `KeyId` parameter\.

In languages that require a client object, these examples use the AWS KMS client object that you created in [Creating a client](programming-client.md)\.

------
#### [ Java ]

For details, see the [createGrant method](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#createGrant-com.amazonaws.services.kms.model.CreateGrantRequest-) in the *AWS SDK for Java API Reference*\.

```
// Create a grant
//
// Replace the following example key ARN with a valid key ID or key ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String granteePrincipal = "arn:aws:iam::111122223333:user/Alice";
String operation = GrantOperation.GenerateDataKey.toString();

CreateGrantRequest request = new CreateGrantRequest()
    .withKeyId(keyId)
    .withGranteePrincipal(granteePrincipal)
    .withOperations(operation);

CreateGrantResult result = kmsClient.createGrant(request);
```

------
#### [ C\# ]

For details, see the [CreateGrant method](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceCreateGrantCreateGrantRequest.html) in the *AWS SDK for \.NET*\.

```
// Create a grant
//
// Replace the following example key ARN with a valid key ID or key ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String granteePrincipal = "arn:aws:iam::111122223333:user/Alice";
String operation = GrantOperation.GenerateDataKey;

CreateGrantRequest createGrantRequest = new CreateGrantRequest()
{
    KeyId = keyId,
    GranteePrincipal = granteePrincipal,
    Operations = new List<string>() { operation }
};

CreateGrantResponse createGrantResult = kmsClient.CreateGrant(createGrantRequest);
```

------
#### [ Python ]

For details, see the [create\_grant method](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.create_grant) in the AWS SDK for Python \(Boto3\)\.

```
# Create a grant

# Replace the following example key ARN with a valid key ID or key ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
grantee_principal = 'arn:aws:iam::111122223333:user/Alice'
operation = ['GenerateDataKey']

response = kms_client.create_grant(
    KeyId=key_id,
    GranteePrincipal=grantee_principal,
    Operations=operation
)
```

------
#### [ Ruby ]

For details, see the [create\_grant](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#create_grant-instance_method) instance method in the [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Create a grant

# Replace the following example key ARN with a valid key ID or key ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
grantee_principal = 'arn:aws:iam::111122223333:user/Alice'
operation = ['GenerateDataKey']

response = kmsClient.create_grant({
  key_id: key_id,
  grantee_principal: grantee_principal,
  operations: operation
})
```

------
#### [ PHP ]

For details, see the [CreateGrant method](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-kms-2014-11-01.html#creategrant) in the *AWS SDK for PHP*\.

```
// Create a grant
//
// Replace the following example key ARN with a valid key ID or key ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
$granteePrincipal = "arn:aws:iam::111122223333:user/Alice";
$operation = ['GenerateDataKey']

$result = $KmsClient->createGrant([
    'GranteePrincipal' => $granteePrincipal,
    'KeyId' => $keyId, 
    'Operations' => $operation 
]);
```

------
#### [ Node\.js ]

For details, see the [createGrant property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#createGrant-property) in the *AWS SDK for JavaScript in Node\.js*\.

```
// Create a grant
//
// Replace the following example key ARN with a valid key ID or key ARN
const KeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
const GranteePrincipal = 'arn:aws:iam::111122223333:user/Alice';
const Operations: ["GenerateDataKey"];
kmsClient.createGrant({ KeyId, GranteePrincipal, Operations }, (err, data) => {
  ...
});
```

------
#### [ PowerShell ]

To create a grant, use the [New\-KMSGrant](https://docs.aws.amazon.com/powershell/latest/reference/items/New-KMSGrant.html) cmdlet\.

```
# Create a grant
 
# Replace the following example key ARN with a valid key ID or key ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
$granteePrincipal = 'arn:aws:iam::111122223333:user/Alice'
$operation = 'GenerateDataKey'

$response = New-KMSGrant -GranteePrincipal $granteePrincipal -KeyId $keyId -Operation $operation
```

To use the AWS KMS PowerShell cmdlets, install the [AWS\.Tools\.KeyManagementService](https://www.powershellgallery.com/packages/AWS.Tools.KeyManagementService/) module\. For more information, see the [AWS Tools for Windows PowerShell User Guide](https://docs.aws.amazon.com/powershell/latest/userguide/)\.

------

## Viewing a grant<a name="list-grants"></a>

To get detailed information about the grants on an AWS KMS customer master key, use the [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation\. These examples use the optional `Limits` parameter, which determines how many grants the operation returns\.

**Note**  
The `GranteePrincipal` field in the `ListGrants` response usually contains the grantee principal of the grant\. However, when the grantee principal in the grant is an AWS service, the `GranteePrincipal` field contains the [service principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services), which might represent several different grantee principals\.

In languages that require a client object, these examples use the AWS KMS client object that you created in [Creating a client](programming-client.md)\.

------
#### [ Java ]

For details about the Java implementation, see the [listGrants method](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#listGrants-com.amazonaws.services.kms.model.ListGrantsRequest-) in the *AWS SDK for Java API Reference*\.

```
// Listing grants on a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
Integer limit = 10;

ListGrantsRequest req = new ListGrantsRequest().withKeyId(keyId).withLimit(limit);
ListGrantsResult result = kmsClient.listGrants(req);
```

------
#### [ C\# ]

For details, see the [ListGrants method](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceListGrantsListGrantsRequest.html) in the *AWS SDK for \.NET*\.

```
// Listing grants on a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
int limit = 10;

ListGrantsRequest listGrantsRequest = new ListGrantsRequest()
{
    KeyId = keyId,
    Limit = limit
};
ListGrantsResponse listGrantsResponse = kmsClient.ListGrants(listGrantsRequest);
```

------
#### [ Python ]

For details, see the [list\_grants method](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.list_grants) in the AWS SDK for Python \(Boto3\)\.

```
# Listing grants on a CMK

# Replace the following example key ARN with a valid key ID or key ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

response = kms_client.list_grants(
    KeyId=key_id,
    Limit=10
)
```

------
#### [ Ruby ]

For details, see the [list\_grants](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#list_grants-instance_method) instance method in the [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Listing grants on a CMK

# Replace the following example key ARN with a valid key ID or key ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

response = kmsClient.list_grants({
  key_id: key_id,
  limit: 10
})
```

------
#### [ PHP ]

For details, see the [ListGrants method](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-kms-2014-11-01.html#listgrants) in the *AWS SDK for PHP*\.

```
// Listing grants on a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
$limit = 10;

$result = $KmsClient->listGrants([
    'KeyId' => $keyId, 
    'Limit' => $limit,
]);
```

------
#### [ Node\.js ]

For details, see the [listGrants property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#listGrants-property) in the *AWS SDK for JavaScript in Node\.js*\.

```
// Listing grants on a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
const KeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
const Limit = 10;
kmsClient.listGrants({ KeyId, Limit }, (err, data) => {
  ...
});
```

------
#### [ PowerShell ]

To view the details of all AWS KMS grants for a CMK, use the [Get\-KMSGrantList](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-KMSGrantList.html) cmdlet\. 

To limit the number of output objects, this example uses the [Select\-Object](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/select-object) cmdlet, instead of the `Limit` parameter, which is being deprecated in list cmdlets\. For help with paginating output in AWS Tools for PowerShell, see [Output Pagination with AWS Tools for PowerShell](http://aws.amazon.com/blogs/developer/output-pagination-with-aws-tools-for-powershell/)\.

```
# Listing grants on a CMK

# Replace the following example key ARN with a valid key ID or key ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
$limit = 10

$response = Get-KMSGrantList -KeyId $keyId | Select-Object -First $limit
```

To use the AWS KMS PowerShell cmdlets, install the [AWS\.Tools\.KeyManagementService](https://www.powershellgallery.com/packages/AWS.Tools.KeyManagementService/) module\. For more information, see the [AWS Tools for Windows PowerShell User Guide](https://docs.aws.amazon.com/powershell/latest/userguide/)\.

------

## Retiring a grant<a name="retire-grant"></a>

To retire a grant for an AWS KMS customer master key, use the [RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html) operation\. You should retire a grant to clean up after you are done using it\. 

To retire a grant, provide the grant token, or both the grant ID and CMK ID\. For this operation, the CMK ID must be [Amazon Resource Name \(ARN\) of the CMK](find-cmk-id-arn.md)\. The grant token is returned by the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation\. The grant ID is returned by the CreateGrant and [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operations\.

RetireGrant doesn't return a response\. To verify that it was effective, use the [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation\.

In languages that require a client object, these examples use the AWS KMS client object that you created in [Creating a client](programming-client.md)\.

------
#### [ Java ]

For details, see the [retireGrant method](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#retireGrant-com.amazonaws.services.kms.model.RetireGrantRequest-) in the *AWS SDK for Java API Reference*\.

```
// Retire a grant
//
String grantToken = Place your grant token here;

RetireGrantRequest req = new RetireGrantRequest().withGrantToken(grantToken);
kmsClient.retireGrant(req);
```

------
#### [ C\# ]

For details, see the [RetireGrant method](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceRetireGrantRetireGrantRequest.html) in the *AWS SDK for \.NET*\.

```
// Retire a grant
//
String grantToken = "Place your grant token here";

RetireGrantRequest retireGrantRequest = new RetireGrantRequest()
{
    GrantToken = grantToken
};
kmsClient.RetireGrant(retireGrantRequest);
```

------
#### [ Python ]

For details, see the [retire\_grant method](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.retire_grant) in the AWS SDK for Python \(Boto3\)\.

```
# Retire a grant

grant_token = Place your grant token here

response = kms_client.retire_grant(
    GrantToken=grant_token
)
```

------
#### [ Ruby ]

For details, see the [retire\_grant](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#retire_grant-instance_method) instance method in the [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Retire a grant

grant_token = Place your grant token here

response = kmsClient.retire_grant({
  grant_token: grant_token
})
```

------
#### [ PHP ]

For details, see the [RetireGrant method](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-kms-2014-11-01.html#retiregrant) in the *AWS SDK for PHP*\.

```
// Retire a grant
//
$grantToken = 'Place your grant token here';

$result = $KmsClient->retireGrant([
    'GrantToken' => $grantToken,
]);
```

------
#### [ Node\.js ]

For details, see the [retireGrant property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#retireGrant-property) in the *AWS SDK for JavaScript in Node\.js*\.

```
// Retire a grant
//
const GrantToken = 'Place your grant token here';
kmsClient.retireGrant({ GrantToken }, (err, data) => {
  ...
});
```

------
#### [ PowerShell ]

To retire a grant, use the [Disable\-KMSGrant](https://docs.aws.amazon.com/powershell/latest/reference/items/Disable-KMSGrant.html) cmdlet\. To get the grant token, use the [New\-KMSGrant](https://docs.aws.amazon.com/powershell/latest/reference/items/New-KMSGrant.html) cmdlet\. The `GrantToken` parameter takes a string, so you don't need to convert output that the [Read\-Host](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/read-host) cmdlet returns\.

```
# Retire a grant

$grantToken = Read-Host -Message Place your grant token here
Disable-KMSGrant -GrantToken $grantToken
```

To use the AWS KMS PowerShell cmdlets, install the [AWS\.Tools\.KeyManagementService](https://www.powershellgallery.com/packages/AWS.Tools.KeyManagementService/) module\. For more information, see the [AWS Tools for Windows PowerShell User Guide](https://docs.aws.amazon.com/powershell/latest/userguide/)\.

------

## Revoking a grant<a name="revoke-grant"></a>

To revoke a grant to an AWS KMS customer master key, use the [RevokeGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) operation\. You can revoke a grant to explicitly deny operations that depend on it\. 

In languages that require a client object, these examples use the AWS KMS client object that you created in [Creating a client](programming-client.md)\.

------
#### [ Java ]

For details, see the [revokeGrant method](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#revokeGrant-com.amazonaws.services.kms.model.RevokeGrantRequest-) in the *AWS SDK for Java API Reference*\.

```
// Revoke a grant on a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

// &fake-grant-id;
String grantId = "grant1";

RevokeGrantRequest req = new RevokeGrantRequest().withKeyId(keyId).withGrantId(grantId);
kmsClient.revokeGrant(req);
```

------
#### [ C\# ]

For details, see the [RevokeGrant method](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceRevokeGrantRevokeGrantRequest.html) in the *AWS SDK for \.NET*\.

```
// Revoke a grant on a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

// &fake-grant-id;
String grantId = "grant1";

RevokeGrantRequest revokeGrantRequest = new RevokeGrantRequest()
{
    KeyId = keyId,
    GrantId = grantId
};
kmsClient.RevokeGrant(revokeGrantRequest);
```

To use the AWS KMS PowerShell cmdlets, install the [AWS\.Tools\.KeyManagementService](https://www.powershellgallery.com/packages/AWS.Tools.KeyManagementService/) module\. For more information, see the [AWS Tools for Windows PowerShell User Guide](https://docs.aws.amazon.com/powershell/latest/userguide/)\.

------
#### [ Python ]

For details, see the [revoke\_grant method](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.revoke_grant) in the AWS SDK for Python \(Boto3\)\.

```
# Revoke a grant on a CMK

# Replace the following example key ARN with a valid key ID or key ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

# &fake-grant-id;
grant_id = 'grant1'

response = kms_client.revoke_grant(
    KeyId=key_id,
    GrantId=grant_id
)
```

------
#### [ Ruby ]

For details, see the [revoke\_grant](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#revoke_grant-instance_method) instance method in the [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Revoke a grant on a CMK

# Replace the following example key ARN with a valid key ID or key ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

# &fake-grant-id;
grant_id = 'grant1'

response = kmsClient.revoke_grant({
  key_id: key_id,
  grant_id: grant_id
})
```

------
#### [ PHP ]

For details, see the [RevokeGrant method](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-kms-2014-11-01.html#revokegrant) in the *AWS SDK for PHP*\.

```
// Revoke a grant on a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';

// Replace the following example grant ID with a valid one
$grantId = "grant1";

$result = $KmsClient->revokeGrant([
    'KeyId' => $keyId, 
    'GrantId' => $grantId,
]);
```

------
#### [ Node\.js ]

For details, see the [revokeGrant property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#revokeGrant-property) in the *AWS SDK for JavaScript in Node\.js*\.

```
// Revoke a grant on a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
const KeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';

// Replace the following example grant ID with a valid one
const GrantId = 'grant1';
kmsClient.revokeGrant({ GrantId, KeyId }, (err, data) => {
  ...
});
```

------
#### [ PowerShell ]

To revoke a grant, use the [Revoke\-KMSGrant](https://docs.aws.amazon.com/powershell/latest/reference/items/Revoke-KMSGrant.html) cmdlet\. 

```
# Revoke a grant on a CMK

# Replace the following example key ARN with a valid key ID or key ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

# Replace the following example grant ID with a valid one
$grantId = 'grant1'

Revoke-KMSGrant -KeyId $keyId -GrantId $grantId
```

To use the AWS KMS PowerShell cmdlets, install the [AWS\.Tools\.KeyManagementService](https://www.powershellgallery.com/packages/AWS.Tools.KeyManagementService/) module\. For more information, see the [AWS Tools for Windows PowerShell User Guide](https://docs.aws.amazon.com/powershell/latest/userguide/)\.

------