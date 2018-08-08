# Working with Grants<a name="programming-grants"></a>

The examples in this topic use the AWS KMS API to create, view, retire, and revoke grants on AWS KMS customer master keys \(CMKs\)\.

**Topics**
+ [Creating a Grant](#create-grant)
+ [Viewing a Grant](#list-grants)
+ [Retiring a Grant](#retire-grant)
+ [Revoking a Grant](#revoke-grant)

## Creating a Grant<a name="create-grant"></a>

To create a grant for an AWS KMS customer master key, use the [CreateGrant](http://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation\.

This example uses the KMS client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [createGrant method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#createGrant-com.amazonaws.services.kms.model.CreateGrantRequest-) in the *AWS SDK for Java API Reference*\.

```
// Create a grant
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String granteePrincipal = "arn:aws:iam::111122223333:user/Alice";
String operation = GrantOperation.Encrypt;

CreateGrantRequest req = new CreateGrantRequest();
req.setKeyId(keyId);
req.setGranteePrincipal(granteePrincipal);
req.setOperation(operation);

CreateGrantResult result = kmsClient.createGrant(req);
```

------
#### [ C\# ]

For details, see the [CreateGrant method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceCreateGrantCreateGrantRequest.html) in the *AWS SDK for \.NET*\.

```
// Create a grant
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String granteePrincipal = "arn:aws:iam::111122223333:user/Alice";
String operation = GrantOperation.Encrypt;

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

For details, see the [create\_grant method](http://boto3.readthedocs.org/en/latest/reference/services/kms.html#KMS.Client.create_grant) in the AWS SDK for Python \(Boto 3\)\.

```
# Create a grant

# Replace the following fictitious CMK ARN with a valid CMK ID or ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
grantee_principal = 'arn:aws:iam::111122223333:user/Alice'
operation = 'Encrypt'

response = kms_client.create_grant(
    KeyId=key_id,
    GranteePrincipal=grantee_principal,
    Operations=operation
)
```

------
#### [ Ruby ]

For details, see the [create\_grant](http://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#create_grant-instance_method) instance method in the [AWS SDK for Ruby](http://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Create a grant

# Replace the following fictitious CMK ARN with a valid CMK ID or ARN
keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
granteePrincipal = 'arn:aws:iam::111122223333:user/Alice'
operation = ['Encrypt']

response = kmsClient.create_grant({
  key_id: keyId,
  grantee_principal: granteePrincipal,
  operations: operation
})
```

------
#### [ PHP ]

For details, see the [CreateGrant method](http://docs.aws.amazon.com/aws-sdk-php/v3/api/api-kms-2014-11-01.html#creategrant) in the *AWS SDK for PHP *\.

```
// Create a grant
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
$granteePrincipal = "arn:aws:iam::111122223333:user/Alice";
$operation =  ['Encrypt', 'Decrypt']

$result = $KmsClient->createGrant([
    'GranteePrincipal' => $granteePrincipal,
    'KeyId' => $keyId, 
    'Operations' => $operation 
]);
```

------
#### [ Node.js ]

For details, see the [CreateGrant property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#createGrant-property) in the *AWS SDK for Node.js*\.

```js
// Create a grant
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN

const KeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
const GranteePrincipal = 'arn:aws:iam::111122223333:user/Alice';
const Operations: [ "Encrypt", "Decrypt"];

kmsClient.createGrant({ KeyId, GranteePrincipal, Operations }, (err, data) => {
  ...
});
```

------

## Viewing a Grant<a name="list-grants"></a>

To get detailed information about the grants on an AWS KMS customer master key, use the [ListGrants](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation\. 

This example uses the KMS client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details about the Java implementation, see the [listGrants method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#listGrants-com.amazonaws.services.kms.model.ListGrantsRequest-) in the *AWS SDK for Java API Reference*\.

```
// Listing grants on a CMK
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
Integer limit = 10;

ListGrantsRequest req = new ListGrantsRequest().withKeyId(keyId).withLimit(limit);
ListGrantsResult result = kmsClient.listGrants(req);
```

------
#### [ C\# ]

For details, see the [ListGrants method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceListGrantsListGrantsRequest.html) in the *AWS SDK for \.NET*\.

```
// Listing grants on a CMK
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
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

For details, see the [list\_grants method](http://boto3.readthedocs.org/en/latest/reference/services/kms.html#KMS.Client.list_grants) in the AWS SDK for Python \(Boto 3\)\.

```
# Listing grants on a CMK

# Replace the following fictitious CMK ARN with a valid CMK ID or ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

response = kms_client.list_grants(
    KeyId=key_id,
    Limit=10
)
```

------
#### [ Ruby ]

For details, see the [list\_grants](http://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#list_grants-instance_method) instance method in the [AWS SDK for Ruby](http://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Listing grants on a CMK

# Replace the following fictitious CMK ARN with a valid CMK ID or ARN
keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

response = kmsClient.list_grants({
  key_id: keyId,
  limit: 10
})
```

------
#### [ PHP ]

For details, see the [ListGrants method](http://docs.aws.amazon.com/aws-sdk-php/v3/api/api-kms-2014-11-01.html#listgrants) in the *AWS SDK for PHP *\.

```
// Listing grants on a CMK
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
$limit = 10;

$result = $KmsClient->listGrants([
    'KeyId' => $keyId, 
    'Limit' => $limit,
]);
```

------
#### [ Node.js ]

For details, see the [ListGrants property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#listGrants-property) in the *AWS SDK for Node.js*\.

```js
// Listing grants on a CMK
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN

const KeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
const Limit = 10;

kmsClient.listGrants({ KeyId, Limit }, (err, data) => {
  ...
});
```

------

## Retiring a Grant<a name="retire-grant"></a>

To retire a grant for an AWS KMS customer master key, use the [RetireGrant](http://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html) operation\. You should retire a grant to clean up after you are done using it\.

This example uses the KMS client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [retireGrant method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#retireGrant-com.amazonaws.services.kms.model.RetireGrantRequest-) in the *AWS SDK for Java API Reference*\.

```
// Retire a grant
//
String grantToken = Place your grant token here;

RetireGrantRequest req = new RetireGrantRequest().withGrantToken(grantToken);
kmsClient.retireGrant(req);
```

------
#### [ C\# ]

For details, see the [RetireGrant method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceRetireGrantRetireGrantRequest.html) in the *AWS SDK for \.NET*\.

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

For details, see the [retire\_grant method](http://boto3.readthedocs.org/en/latest/reference/services/kms.html#KMS.Client.retire_grant) in the AWS SDK for Python \(Boto 3\)\.

```
# Retire a grant

grant_token = Place your grant token here

response = kms_client.retire_grant(
    GrantToken=grant_token
)
```

------
#### [ Ruby ]

For details, see the [retire\_grant](http://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#retire_grant-instance_method) instance method in the [AWS SDK for Ruby](http://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Retire a grant

grantToken = Place your grant token here

response = kmsClient.retire_grant({
  grant_token: grantToken
})
```

------
#### [ PHP ]

For details, see the [RetireGrant method](http://docs.aws.amazon.com/aws-sdk-php/v3/api/api-kms-2014-11-01.html#retiregrant) in the *AWS SDK for PHP *\.

```
// Retire a grant
//
$grantToken = 'Place your grant token here';

$result = $KmsClient->retireGrant([
    'GrantToken' => $grantToken,
]);
```

------
#### [ Node.js ]

For details, see the [RetireGrant property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#retireGrant-property) in the *AWS SDK for Node.js*\.

```js
// Retire a grant
//

const GrantToken = 'Place your grant token here';

kmsClient.retireGrant({ GrantToken }, (err, data) => {
  ...
});
```

------

## Revoking a Grant<a name="revoke-grant"></a>

To revoke a grant to an AWS KMS customer master key, use the [RevokeGrant](http://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) operation\. You can revoke a grant to explicitly deny operations that depend on it\. 

This example uses the KMS client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [revokeGrant method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#revokeGrant-com.amazonaws.services.kms.model.RevokeGrantRequest-) in the *AWS SDK for Java API Reference*\.

```
// Revoke a grant on a CMK
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String grantId = "grant1";

RevokeGrantRequest req = new RevokeGrantRequest().withKeyId(keyId).withGrantId(grantId);
kmsClient.revokeGrant(req);
```

------
#### [ C\# ]

For details, see the [RevokeGrant method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceRevokeGrantRevokeGrantRequest.html) in the *AWS SDK for \.NET*\.

```
// Revoke a grant on a CMK
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String grantId = "grant1";

RevokeGrantRequest revokeGrantRequest = new RevokeGrantRequest()
{
    KeyId = keyId,
    GrantId = grantId
};
kmsClient.RevokeGrant(revokeGrantRequest);
```

------
#### [ Python ]

For details, see the [revoke\_grant method](http://boto3.readthedocs.org/en/latest/reference/services/kms.html#KMS.Client.revoke_grant) in the AWS SDK for Python \(Boto 3\)\.

```
# Revoke a grant on a CMK

// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
grant_id = 'grant1'

response = kms_client.revoke_grant(
    KeyId=key_id,
    GrantId=grant_id
)
```

------
#### [ Ruby ]

For details, see the [revoke\_grant](http://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#revoke_grant-instance_method) instance method in the [AWS SDK for Ruby](http://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Revoke a grant on a CMK

# Replace the following fictitious CMK ARN with a valid CMK ID or ARN
keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
grantId = 'grant1'

response = kmsClient.revoke_grant({
  key_id: keyId,
  grant_id: grantId
})
```

------
#### [ PHP ]

For details, see the [RevokeGrant method](http://docs.aws.amazon.com/aws-sdk-php/v3/api/api-kms-2014-11-01.html#revokegrant) in the *AWS SDK for PHP *\.

```
// Revoke a grant on a CMK
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
$grantId = "grant1";

$result = $KmsClient->revokeGrant([
    'KeyId' => $keyId, 
    'GrantId' => $grantId,
]);
```

------
#### [ Node.js ]

For details, see the [RevokeGrant property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#revokeGrant-property) in the *AWS SDK for Node.js*\.

```js
// Revoke a grant on a CMK
//
// Replace the following fictitious CMK ARN with a valid CMK ID or ARN

const KeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
const GrantId = 'grant1';

kmsClient.revokeGrant({ GrantId, KeyId }, (err, data) => {
  ...
});
```

------
