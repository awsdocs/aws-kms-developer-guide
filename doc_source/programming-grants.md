# Working with Grants<a name="programming-grants"></a>

The examples in this topic use the AWS KMS API to create, view, retire, and revoke grants on AWS KMS customer master keys \(CMKs\)\.

**Topics**
+ [Creating a Grant](#create-grant)
+ [Viewing a Grant](#list-grants)
+ [Retiring a Grant](#retire-grant)
+ [Revoking a Grant](#revoke-grant)

## Creating a Grant<a name="create-grant"></a>

To create a grant for an AWS KMS customer master key, use the [CreateGrant](http://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation\.

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [createGrant method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#createGrant-com.amazonaws.services.kms.model.CreateGrantRequest-) in the *AWS SDK for Java API Reference*\.

```
// Create a grant
//
// Replace the following fictitious key ARN with a valid key ID
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
// Replace the following fictitious key ARN with a valid key ID
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

## Viewing a Grant<a name="list-grants"></a>

To get detailed information about the grants on an AWS KMS customer master key, use the [ListGrants](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation\. 

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details about the Java implementation, see the [listGrants method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#listGrants-com.amazonaws.services.kms.model.ListGrantsRequest-) in the *AWS SDK for Java API Reference*\.

```
// Listing grants on a CMK
//
// Replace the following fictitious key ARN with a valid key ID
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
Integer limit = 10;
String marker = null;

ListGrantsRequest req = new ListGrantsRequest().withKeyId(keyId).withMarker(marker).withLimit(limit);
ListGrantsResult result = kmsClient.listGrants(req);
```

------
#### [ C\# ]

For details, see the [EnableKey method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceListGrantsListGrantsRequest.html) in the *AWS SDK for \.NET*\.

```
// Listing grants on a CMK
//
// Replace the following fictitious key ARN with a valid key ID
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
int limit = 10;
String marker = null;

ListGrantsRequest listGrantsRequest = new ListGrantsRequest()
{
    KeyId = keyId,
    Limit = limit,
    Marker = marker
};
ListGrantsResponse listGrantsResponse = kmsClient.ListGrants(listGrantsRequest);
```

------

## Retiring a Grant<a name="retire-grant"></a>

To retire a grant for an AWS KMS customer master key, use the [RetireGrant](http://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html) operation\. You should retire a grant to clean up after you are done using it\.

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

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

For details, see the [EnableKey method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceRetireGrantRetireGrantRequest.html) in the *AWS SDK for \.NET*\.

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

## Revoking a Grant<a name="revoke-grant"></a>

To revoke a grant to an AWS KMS customer master key, use the [RevokeGrant](http://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) operation\. You can revoke a grant to explicitly deny operations that depend on it\. 

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [revokeGrant method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#revokeGrant-com.amazonaws.services.kms.model.RevokeGrantRequest-) in the *AWS SDK for Java API Reference*\.

```
// Revoke a grant on a CMK
//
// Replace the following fictitious key ARN with a valid key ID
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String grantId = "grant1";

RevokeGrantRequest req = new RevokeGrantRequest().withKeyId(keyId).withGrantId(grantId);
kmsClient.revokeGrant(req);
```

------
#### [ C\# ]

For details, see the [EnableKey method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceRevokeGrantRevokeGrantRequest.html) in the *AWS SDK for \.NET*\.

```
// Revoke a grant on a CMK
//
// Replace the following fictitious key ARN with a valid key ID
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