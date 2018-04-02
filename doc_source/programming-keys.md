# Working With Keys<a name="programming-keys"></a>

This topic discusses how to create, describe, list, enable, and disable keys in Java\. For detailed information, see the [AWS SDK for Java API Reference](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/)\. 

**Topics**
+ [Creating a Customer Master Key](#creating-keys)
+ [Creating a Data Key](#generate-datakeys)
+ [Getting Information About a Custom Master Key](#describing-keys)
+ [Getting Key IDs and Key ARNs of Customer Master Keys](#listing-keys)
+ [Enabling Customer Master Keys](#enable-keys)
+ [Disabling Customer Master Keys](#disable-keys)

## Creating a Customer Master Key<a name="creating-keys"></a>

To create a [customer master key](concepts.md#master_keys), use the [CreateKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation\. For details about the Java implementation, see the [createKey method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#createKey-com.amazonaws.services.kms.model.CreateKeyRequest-) in the *AWS SDK for Java API Reference*\.

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

```
// Create a CMK
//
String desc = "Key for protecting critical data";
    
CreateKeyRequest req = new CreateKeyRequest().withDescription(desc);
CreateKeyResult result = kmsClient.createKey(req);
```

## Creating a Data Key<a name="generate-datakeys"></a>

To generate a data key, use the [GenerateDataKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation\. This operation returns plaintext and encrypted copies of the data key that it creates\. For details about the Java implementation, see the [generateDataKey method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#generateDataKey-com.amazonaws.services.kms.model.GenerateDataKeyRequest-) in the *AWS SDK for Java API Reference*\.

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

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

## Getting Information About a Custom Master Key<a name="describing-keys"></a>

To get detailed information about a CMK, including the key ARN and key state, use the [DescribeKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. For details about the Java implementation of DescribeKey, see the [describeKey method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMS.html#describeKey-com.amazonaws.services.kms.model.DescribeKeyRequest-) in the *AWS SDK for Java API Reference*\.

DescribeKey does not get aliases\. To get aliases, use the [ListAliases](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. 

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

```
// Describe a CMK
//
// Replace the fictitious key ARN with a valid one.
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

DescribeKeyRequest req = new DescribeKeyRequest().withKeyId(keyId);
DescribeKeyResult result = kmsClient.describeKey(req);
```

## Getting Key IDs and Key ARNs of Customer Master Keys<a name="listing-keys"></a>

To get the key IDs and key ARNs of the customer master keys, use the [ListKeys](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html) operation\. For details about the Java implementation, see the [listKeys method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#listKeys-com.amazonaws.services.kms.model.ListKeysRequest-) in the *AWS SDK for Java API Reference*\.

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

```
// List CMKs in this account
//
Integer limit = 10;
String marker = null;

ListKeysRequest req = new ListKeysRequest().withMarker(marker).withLimit(limit);
ListKeysResult result = kmsClient.listKeys(req);
```

## Enabling Customer Master Keys<a name="enable-keys"></a>

To enable a disabled CMK, use the [EnableKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKey.html) operation\. For details about the Java implementation, see the [enableKey method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#enableKey-com.amazonaws.services.kms.model.EnableKeyRequest-) in the *AWS SDK for Java API Reference*\.

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

```
// Enable a CMK
//
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

EnableKeyRequest req = new EnableKeyRequest().withKeyId(keyId);
kmsClient.enableKey(req);
```

## Disabling Customer Master Keys<a name="disable-keys"></a>

To disable a CMK, use the [DisableKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) operation\. Disabling a CMK prevents it from being used\. For details about the Java implementation, see the [disableKey method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#disableKey-com.amazonaws.services.kms.model.DisableKeyRequest-) in the *AWS SDK for Java API Reference*\.

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

```
// Disable a CMK
//
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

DisableKeyRequest req = new DisableKeyRequest().withKeyId(keyId);
kmsClient.disableKey(req);
```