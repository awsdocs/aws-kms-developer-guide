# Working With Keys<a name="programming-keys"></a>

The examples in this topic use the AWS KMS API to create, view, enable, and disable AWS KMS customer master keys, and to generate data keys\.

**Topics**
+ [Creating a Customer Master Key](#creating-keys)
+ [Generating a Data Key](#generate-datakeys)
+ [Viewing a Custom Master Key](#describing-keys)
+ [Getting Key IDs and Key ARNs of Customer Master Keys](#listing-keys)
+ [Enabling Customer Master Keys](#enable-keys)
+ [Disabling Customer Master Keys](#disable-keys)

## Creating a Customer Master Key<a name="creating-keys"></a>

To create a [customer master key](concepts.md#master_keys), use the [CreateKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation\.

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [createKey method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#createKey-com.amazonaws.services.kms.model.CreateKeyRequest-) in the *AWS SDK for Java API Reference*\.

```
// Create a CMK
//
String desc = "Key for protecting critical data";
    
CreateKeyRequest req = new CreateKeyRequest().withDescription(desc);
CreateKeyResult result = kmsClient.createKey(req);
```

------
#### [ C\# ]

For details, see the [CreateKey method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceCreateKeyCreateKeyRequest.html) in the *AWS SDK for \.NET*\.

```
// Create a CMK
//
String desc = "Key for protecting critical data";

CreateKeyRequest req = new CreateKeyRequest()
{
    Description = desc
};
CreateKeyResponse response = kmsClient.CreateKey(req);
```

------

## Generating a Data Key<a name="generate-datakeys"></a>

To generate a data key, use the [GenerateDataKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation\. This operation returns plaintext and encrypted copies of the data key that it creates\. 

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [generateDataKey method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#generateDataKey-com.amazonaws.services.kms.model.GenerateDataKeyRequest-) in the *AWS SDK for Java API Reference*\.

```
// Generate a data key
//
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
GenerateDataKeyRequest dataKeyRequest = new GenerateDataKeyRequest();
dataKeyRequest.setKeyId(keyId);
dataKeyRequest.setKeySpec("AES_128");

GenerateDataKeyResult dataKeyResult = kmsClient.generateDataKey(dataKeyRequest);

ByteBuffer plaintextKey = dataKeyResult.getPlaintext();

ByteBuffer encryptedKey = dataKeyResult.getCiphertextBlob();
```

------
#### [ C\# ]

For details, see the [GenerateDataKey method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceGenerateDataKeyGenerateDataKeyRequest.html) in the *AWS SDK for \.NET*\.

```
// Generate a data key
//
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
GenerateDataKeyRequest dataKeyRequest = new GenerateDataKeyRequest()
{
    KeyId = keyId,
    KeySpec = DataKeySpec.AES_128
};

GenerateDataKeyResponse dataKeyResponse = kmsClient.GenerateDataKey(dataKeyRequest);

MemoryStream plaintextKey = dataKeyResponse.Plaintext;

MemoryStream encryptedKey = dataKeyResponse.CiphertextBlob;
```

------

## Viewing a Custom Master Key<a name="describing-keys"></a>

To get detailed information about a customer master key \(CMK\), including the CMK ARN and [key state](key-state.md), use the [DescribeKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. 

DescribeKey does not get aliases\. To get aliases, use the [ListAliases](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. 

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [describeKey method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMS.html#describeKey-com.amazonaws.services.kms.model.DescribeKeyRequest-) in the *AWS SDK for Java API Reference*\.

```
// Describe a CMK
//
// Replace the fictitious key ARN with a valid one.
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

DescribeKeyRequest req = new DescribeKeyRequest().withKeyId(keyId);
DescribeKeyResult result = kmsClient.describeKey(req);
```

------
#### [ C\# ]

For details, see the [DescribeKey method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceDescribeKeyDescribeKeyRequest.html) in the *AWS SDK for \.NET*\. 

```
// Describe a CMK
//
// Replace the fictitious key ARN with a valid one.
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

DescribeKeyRequest describeKeyRequest = new DescribeKeyRequest()
{
    KeyId = keyId
};

DescribeKeyResponse describeKeyResponse = kmsClient.DescribeKey(describeKeyRequest);
```

------

## Getting Key IDs and Key ARNs of Customer Master Keys<a name="listing-keys"></a>

To get the IDs and ARNs of the customer master keys, use the [ListKeys](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html) operation\. 

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [listKeys method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#listKeys-com.amazonaws.services.kms.model.ListKeysRequest-) in the *AWS SDK for Java API Reference*\.

```
// List CMKs in this account
//
Integer limit = 10;
String marker = null;

ListKeysRequest req = new ListKeysRequest().withMarker(marker).withLimit(limit);
ListKeysResult result = kmsClient.listKeys(req);
```

------
#### [ C\# ]

For details, see the [ListKeys method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceListKeysListKeysRequest.html) in the *AWS SDK for \.NET*\.

```
// List CMKs in this account
//
int limit = 10;
String marker = null;

ListKeysRequest listKeysRequest = new ListKeysRequest()
{
    Marker = marker,
    Limit = limit
};
ListKeysResponse listKeysResponse = kmsClient.ListKeys(listKeysRequest);
```

------

## Enabling Customer Master Keys<a name="enable-keys"></a>

To enable a disabled customer master key \(CMK\), use the [EnableKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKey.html) operation\.

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details about the Java implementation, see the [enableKey method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#enableKey-com.amazonaws.services.kms.model.EnableKeyRequest-) in the *AWS SDK for Java API Reference*\.

```
// Enable a CMK
//
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

EnableKeyRequest req = new EnableKeyRequest().withKeyId(keyId);
kmsClient.enableKey(req);
```

------
#### [ C\# ]

For details, see the [EnableKey method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceEnableKeyEnableKeyRequest.html) in the *AWS SDK for \.NET*\.

```
// Enable a CMK
//
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

EnableKeyRequest enableKeyRequest = new EnableKeyRequest()
{
    KeyId = keyId
};
kmsClient.EnableKey(enableKeyRequest);
```

------

## Disabling Customer Master Keys<a name="disable-keys"></a>

To disable a CMK, use the [DisableKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) operation\. Disabling a CMK prevents it from being used\.

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [disableKey method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#disableKey-com.amazonaws.services.kms.model.DisableKeyRequest-) in the *AWS SDK for Java API Reference*\.

```
// Disable a CMK
//
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

DisableKeyRequest req = new DisableKeyRequest().withKeyId(keyId);
kmsClient.disableKey(req);
```

------
#### [ C\# ]

For details, see the [EnableKey method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceDisableKeyDisableKeyRequest.html) in the *AWS SDK for \.NET*\.

```
// Disable a CMK
//
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

DisableKeyRequest disableKeyRequest = new DisableKeyRequest()
{
    KeyId = keyId
};
kmsClient.DisableKey(disableKeyRequest);
```

------