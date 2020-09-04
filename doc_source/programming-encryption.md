# Encrypting and decrypting data keys<a name="programming-encryption"></a>

The examples in this topic use the [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html), [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html), and [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) operations in the AWS KMS API\. 

These operations are designed to encrypt and decrypt [data keys](concepts.md#data-keys)\. They use an AWS KMS [customer master key](concepts.md#master_keys) \(CMK\) in the encryption operations and they cannot accept more than 4 KB \(4096 bytes\) of data\. Although you might use them to encrypt small amounts of data, such as a password or RSA key, they are not designed to encrypt application data\.

To encrypt application data, use the server\-side encryption features of an AWS service, or a client\-side encryption library, such as the [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/) or the [Amazon S3 encryption client](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingClientSideEncryption.html)\. 

**Topics**
+ [Encrypting a data key](#encryption)
+ [Decrypting a data key](#decryption)
+ [Re\-encrypting a data key under a different customer master key](#reencryption)

## Encrypting a data key<a name="encryption"></a>

The [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation is designed to encrypt data keys, but it is not frequently used\. The [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html) operations return encrypted data keys\. You might use this method when you are moving encrypted data to a different Region and want to encrypt its data key with a CMK in the new Region\. 

In languages that require a client object, these examples use the AWS KMS client object that you created in [Creating a client](programming-client.md)\.

------
#### [ Java ]

For details, see the [encrypt method](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#encrypt-com.amazonaws.services.kms.model.EncryptRequest-) in the *AWS SDK for Java API Reference*\.

```
// Encrypt a data key
//
// Replace the following example key ARN with any valid key identfier
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
ByteBuffer plaintext = ByteBuffer.wrap(new byte[]{1,2,3,4,5,6,7,8,9,0});

EncryptRequest req = new EncryptRequest().withKeyId(keyId).withPlaintext(plaintext);
ByteBuffer ciphertext = kmsClient.encrypt(req).getCiphertextBlob();
```

------
#### [ C\# ]

For details, see the [Encrypt method](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceEncryptEncryptRequest.html) in the *AWS SDK for \.NET*\.

```
// Encrypt a data key
//
// Replace the following example key ARN with any valid key identfier
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
MemoryStream plaintext = new MemoryStream();
plaintext.Write(new byte[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 0 }, 0, 10);

EncryptRequest encryptRequest = new EncryptRequest()
{
    KeyId = keyId,
    Plaintext = plaintext
};
MemoryStream ciphertext = kmsClient.Encrypt(encryptRequest).CiphertextBlob;
```

------
#### [ Python ]

For details, see the [encrypt method](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.encrypt) in the AWS SDK for Python \(Boto3\)\.

```
# Encrypt a data key

# Replace the following example key ARN with any valid key identfier
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
plaintext = b'\x01\x02\x03\x04\x05\x06\x07\x08\x09\x00'

response = kms_client.encrypt(
    KeyId=key_id,
    Plaintext=plaintext
)

ciphertext = response['CiphertextBlob']
```

------
#### [ Ruby ]

For details, see the [encrypt](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#encrypt-instance_method) instance method in the [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Encrypt a data key

# Replace the following example key ARN with any valid key identfier
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
plaintext = "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x00"

response = kmsClient.encrypt({
  key_id: key_id,
  plaintext: plaintext
})

ciphertext = response.ciphertext_blob
```

------
#### [ PHP ]

For details, see the [Encrypt method](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-kms-2014-11-01.html#encrypt) in the *AWS SDK for PHP*\.

```
// Encrypt a data key
//
// Replace the following example key ARN with any valid key identfier
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
$message = pack('c*',1,2,3,4,5,6,7,8,9,0);

$result = $KmsClient->encrypt([
    'KeyId' => $keyId, 
    'Plaintext' => $message, 
]);

$ciphertext = $result['CiphertextBlob'];
```

------
#### [ Node\.js ]

For details, see the [encrypt property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#encrypt-property) in the AWS SDK for JavaScript in Node\.js\.

```
// Encrypt a data key
//
// Replace the following example key ARN with any valid key identfier
const KeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
const Plaintext = Buffer.from([1, 2, 3, 4, 5, 6, 7, 8, 9, 0]);
kmsClient.encrypt({ KeyId, Plaintext }, (err, data) => {
  if (err) console.log(err, err.stack); // an error occurred
  else {
    const { CiphertextBlob } = data;
      ...
  }
});
```

------
#### [ PowerShell ]

To encrypt a data key under an AWS KMS CMK, use the [Invoke\-KMSEncrypt](https://docs.aws.amazon.com/powershell/latest/reference/items/Invoke-KMSEncrypt.html) cmdlet\. It returns the ciphertext as a `MemoryStream` \([System\.IO\.MemoryStream](https://docs.microsoft.com/en-us/dotnet/api/system.io.memorystream)\) object\. You can use the `MemoryStream` object as the input to the [Invoke\-KMSDecrypt](https://docs.aws.amazon.com/powershell/latest/reference/items/Invoke-KMSDecrypt.html) cmdlet\.

AWS KMS also returns data keys as `MemoryStream` objects\. In this example, to simulate a plaintext data key, we create a byte array and write it to a `MemoryStream` object\. 

Note that the `Plaintext` parameter of `Invoke-KMSEncrypt` takes a byte array \(`byte[]`\); it does not require a `MemoryStream` object\. Beginning in AWSPowerShell version 4\.0, parameters in all AWSPowerShell modules that take byte arrays and `MemoryStream` objects accept byte arrays, `MemoryStream` objects, strings, string arrays, and `FileInfo` \([System\.IO\.FileInfo](https://docs.microsoft.com/en-us/dotnet/api/system.io.fileinfo)\) objects\. You can pass any of these types to `Invoke-KMSEncrypt`\.

```
# Encrypt a data key

# Replace the following example key ARN with any valid key identfier
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

# Simulate a data key
 # Create a byte array
[byte[]] $bytes = 1, 2, 3, 4, 5, 6, 7, 8, 9, 0

 # Create a MemoryStream                    
$plaintext = [System.IO.MemoryStream]::new()

 # Add the byte array to the MemoryStream
$plaintext.Write($bytes, 0, $bytes.length)

# Encrypt the simulated data key
$response = Invoke-KMSEncrypt -KeyId $keyId -Plaintext $plaintext

# Get the ciphertext from the response
$ciphertext = $response.CiphertextBlob
```

To use the AWS KMS PowerShell cmdlets, install the [AWS\.Tools\.KeyManagementService](https://www.powershellgallery.com/packages/AWS.Tools.KeyManagementService/) module\. For more information, see the [AWS Tools for Windows PowerShell User Guide](https://docs.aws.amazon.com/powershell/latest/userguide/)\.

------

## Decrypting a data key<a name="decryption"></a>

To decrypt a data key, use the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\.

The `ciphertextBlob` that you specify must be the value of the `CiphertextBlob` field from a [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), or [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) response, or the `PrivateKeyCiphertextBlob` field from a [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html) or [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html) response\. You can also use the `Decrypt` operation to decrypt data encrypted outside of AWS KMS by the public key in an asymmetric CMK\.

The `KeyId` parameter is not required when decrypting with symmetric CMKs\. AWS KMS can get the CMK that was used to encrypt the data from the metadata in the ciphertext blob\. But it's always a best practice to specify the CMK you are using\. This practice ensures that you use the CMK that you intend, and prevents you from inadvertently decrypting a ciphertext using a CMK you do not trust\.

In languages that require a client object, these examples use the AWS KMS client object that you created in [Creating a client](programming-client.md)\.

------
#### [ Java ]

For details, see the [decrypt method](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#decrypt-com.amazonaws.services.kms.model.DecryptRequest-) in the *AWS SDK for Java API Reference*\.

```
// Decrypt a data key
//
// Replace the following example key ARN with any valid key identfier
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

ByteBuffer ciphertextBlob = Place your ciphertext here;

DecryptRequest req = new DecryptRequest().withCiphertextBlob(ciphertextBlob).withKeyId(keyId);
ByteBuffer plainText = kmsClient.decrypt(req).getPlaintext();
```

------
#### [ C\# ]

For details, see the [Decrypt method](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceDecryptDecryptRequest.html) in the *AWS SDK for \.NET*\.

```
// Decrypt a data key
//
// Replace the following example key ARN with any valid key identfier
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

MemoryStream ciphertextBlob = new MemoryStream();
// Write ciphertext to memory stream

DecryptRequest decryptRequest = new DecryptRequest()
{
    CiphertextBlob = ciphertextBlob,
    KeyId = keyId
};
MemoryStream plainText = kmsClient.Decrypt(decryptRequest).Plaintext;
```

------
#### [ Python ]

For details, see the [decrypt method](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.decrypt) in the AWS SDK for Python \(Boto3\)\.

```
# Decrypt a data key

# Replace the following example key ARN with any valid key identfier
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
ciphertext = 'Place your ciphertext here'

response = kms_client.decrypt(
    CiphertextBlob=ciphertext,
    KeyId=key_id
)

plaintext = response['Plaintext']
```

------
#### [ Ruby ]

For details, see the [decrypt](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#decrypt-instance_method) instance method in the [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Decrypt a data key

# Replace the following example key ARN with any valid key identfier
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

ciphertext = 'Place your ciphertext here'
ciphertext_packed = [ciphertext].pack("H*")

response = kmsClient.decrypt({
  ciphertext_blob: ciphertext_packed,
  key_id: key_id
})

plaintext = response.plaintext
```

------
#### [ PHP ]

For details, see the [Decrypt method](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-kms-2014-11-01.html#decrypt) in the *AWS SDK for PHP*\.

```
// Decrypt a data key
//
// Replace the following example key ARN with any valid key identfier
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
$ciphertext = 'Place your cipher text blob here';

$result = $KmsClient->decrypt([
     'CiphertextBlob' => $ciphertext,
     'KeyId' => $keyId,
]);

$plaintext = $result['Plaintext'];
```

------
#### [ Node\.js ]

For details, see the [decrypt property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#decrypt-property) in the *AWS SDK for JavaScript in Node\.js*\.

```
// Decrypt a data key
//
// Replace the following example key ARN with any valid key identfier
const KeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
const CiphertextBlob = 'Place your cipher text blob here';
kmsClient.decrypt({ CiphertextBlob, KeyId }, (err, data) => {
  if (err) console.log(err, err.stack); // an error occurred
  else {
    const { Plaintext } = data;
    ...
  }
});
```

------
#### [ PowerShell ]

To decrypt a data key, use the [Invoke\-KMSEncrypt](https://docs.aws.amazon.com/powershell/latest/reference/items/Invoke-KMSEncrypt.html) cmdlet\. 

This cmdlet returns the plaintext as a `MemoryStream` \([System\.IO\.MemoryStream](https://docs.microsoft.com/en-us/dotnet/api/system.io.memorystream)\) object\. To convert it to a byte array, use cmdlets or functions that convert `MemoryStream` objects to byte arrays, such as the functions in the [Convert](https://www.powershellgallery.com/packages/Convert) module\.

Because this example uses the ciphertext that an AWS KMS encryption cmdlet returned, it uses a `MemoryStream` object for the value of the `CiphertextBlob` parameter\. However, the `CiphertextBlob` parameter of `Invoke-KMSDecrypt` takes a byte array \(`byte[]`\); it does not require a `MemoryStream` object\. Beginning in AWSPowerShell version 4\.0, parameters in all AWSPowerShell modules that take byte arrays and `MemoryStream` objects accept byte arrays, `MemoryStream` objects, strings, string arrays, and `FileInfo` \([System\.IO\.FileInfo](https://docs.microsoft.com/en-us/dotnet/api/system.io.fileinfo)\) objects\. You can pass any of these types to `Invoke-KMSDecrypt`\.

```
# Decrypt a data key
# Replace the following example key ARN with any valid key identfier
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
 
[System.IO.MemoryStream]$ciphertext = Read-Host 'Place your cipher text blob here'

$response = Invoke-KMSDecrypt -CiphertextBlob $ciphertext -KeyId $keyId
$plaintext = $response.Plaintext
```

To use the AWS KMS PowerShell cmdlets, install the [AWS\.Tools\.KeyManagementService](https://www.powershellgallery.com/packages/AWS.Tools.KeyManagementService/) module\. For more information, see the [AWS Tools for Windows PowerShell User Guide](https://docs.aws.amazon.com/powershell/latest/userguide/)\.

------

## Re\-encrypting a data key under a different customer master key<a name="reencryption"></a>

To decrypt an encrypted data key, and then immediately re\-encrypt the data key under a different customer master key \(CMK\), use the [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) operation\. The operations are performed entirely on the server side within AWS KMS, so they never expose your plaintext outside of AWS KMS\.

The `ciphertextBlob` that you specify must be the value of the `CiphertextBlob` field from a [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html), [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html), or [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) response, or the `PrivateKeyCiphertextBlob` field from a [GenerateDataKeyPair](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPair.html) or [GenerateDataKeyPairWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyPairWithoutPlaintext.html) response\. You can also use the `ReEncrypt` operation to re\-encrypt data encrypted outside of AWS KMS by the public key in an asymmetric CMK\.

The `SourceKeyId` parameter is not required when re\-encrypting with symmetric CMKs\. AWS KMS can get the CMK that was used to encrypt the data from the metadata in the ciphertext blob\. But it's always a best practice to specify the CMK you are using\. This practice ensures that you use the CMK that you intend, and prevents you from inadvertently decrypting a ciphertext using a CMK you do not trust\.

In languages that require a client object, these examples use the AWS KMS client object that you created in [Creating a client](programming-client.md)\.

------
#### [ Java ]

For details, see the [reEncrypt method](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#reEncrypt-com.amazonaws.services.kms.model.ReEncryptRequest-) in the *AWS SDK for Java API Reference*\.

```
// Re-encrypt a data key

ByteBuffer sourceCiphertextBlob = Place your ciphertext here;

// Replace the following example key ARNs with valid key identfiers
String sourceKeyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String destinationKeyId = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321";

ReEncryptRequest req = new ReEncryptRequest();
req.setCiphertextBlob(sourceCiphertextBlob);
req.setSourceKeyId(sourceKeyId);
req.setDestinationKeyId(destinationKeyId);
ByteBuffer destinationCipherTextBlob = kmsClient.reEncrypt(req).getCiphertextBlob();
```

------
#### [ C\# ]

For details, see the [ReEncrypt method](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceReEncryptReEncryptRequest.html) in the *AWS SDK for \.NET*\.

```
// Re-encrypt a data key

MemoryStream sourceCiphertextBlob = new MemoryStream();
// Write ciphertext to memory stream

// Replace the following example key ARNs with valid key identfiers
String sourceKeyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String destinationKeyId = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321";

ReEncryptRequest reEncryptRequest = new ReEncryptRequest()
{
    CiphertextBlob = sourceCiphertextBlob,
    SourceKeyId = sourceKeyId,
    DestinationKeyId = destinationKeyId
};
MemoryStream destinationCipherTextBlob = kmsClient.ReEncrypt(reEncryptRequest).CiphertextBlob;
```

------
#### [ Python ]

For details, see the [re\_encrypt method](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.re_encrypt) in the AWS SDK for Python \(Boto3\)\.

```
# Re-encrypt a data key
ciphertext = 'Place your ciphertext here'

# Replace the following example key ARNs with valid key identfiers
source_key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
destination_key_id = 'arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321'

response = kms_client.re_encrypt(
    CiphertextBlob=ciphertext,
    SourceKeyId=source_key_id,
    DestinationKeyId=destination_key_id
)

destination_ciphertext_blob = response['CiphertextBlob']
```

------
#### [ Ruby ]

For details, see the [re\_encrypt](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#re_encrypt-instance_method) instance method in the [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Re-encrypt a data key

ciphertext = 'Place your ciphertext here'
ciphertext_packed = [ciphertext].pack("H*")

# Replace the following example key ARNs with valid key identfiers
source_key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
destination_key_id = 'arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321'

response = kmsClient.re_encrypt({
  ciphertext_blob: ciphertext_packed,
  source_key_id: source_key_id,
  destination_key_id: destination_key_id
})

destination_ciphertext_blob = response.ciphertext_blob.unpack('H*')
```

------
#### [ PHP ]

For details, see the [ReEncrypt method](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-kms-2014-11-01.html#reencrypt) in the *AWS SDK for PHP*\.

```
// Re-encrypt a data key

$ciphertextBlob = 'Place your ciphertext here';

// Replace the following example key ARNs with valid key identfiers
$sourceKeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
$destinationKeyId = 'arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321';

$result = $KmsClient->reEncrypt([
    'CiphertextBlob' => $ciphertextBlob,
    'SourceKeyId' => $sourceKeyId,
    'DestinationKeyId' => $destinationKeyId, 
]);
```

------
#### [ Node\.js ]

For details, see the [reEncrypt property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#reEncrypt-property) in the *AWS SDK for JavaScript in Node\.js*\.

```
// Re-encrypt a data key
const CiphertextBlob = 'Place your cipher text blob here';
// Replace the following example key ARNs with valid key identfiers
const SourceKeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
const DestinationKeyId = 'arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321';

kmsClient.reEncrypt({ CiphertextBlob, SourceKeyId, DestinationKeyId }, (err, data) => {
  ...
});
```

------
#### [ PowerShell ]

To re\-encrypt a ciphertext under the same or a different CMK, use the [Invoke\-KMSReEncrypt](https://docs.aws.amazon.com/powershell/latest/reference/items/Invoke-KMSReEncrypt.html) cmdlet\.

Because this example uses the ciphertext that an AWS KMS encryption cmdlet returned, it uses a `MemoryStream` object for the value of the `CiphertextBlob` parameter\. However, the `CiphertextBlob` parameter of `Invoke-KMSReEncrypt` takes a byte array \(`byte[]`\); it does not require a `MemoryStream` object\. Beginning in AWSPowerShell version 4\.0, parameters in all AWSPowerShell modules that take byte arrays and `MemoryStream` objects accept byte arrays, `MemoryStream` objects, strings, string arrays, and `FileInfo` \([System\.IO\.FileInfo](https://docs.microsoft.com/en-us/dotnet/api/system.io.fileinfo)\) objects\. You can pass any of these types to `Invoke-KMSReEncrypt`\.

```
# Re-encrypt a data key

[System.IO.MemoryStream]$ciphertextBlob = Read-Host 'Place your cipher text blob here'

# Replace the following example key ARNs with valid key identfiers
$sourceKeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
$destinationKeyId = 'arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321'

$response = Invoke-KMSReEncrypt -Ciphertext $ciphertextBlob -SourceKeyId $sourceKeyId -DestinationKeyId $destinationKeyId
$reEncryptedCiphertext = $response.CiphertextBlob
```

To use the AWS KMS PowerShell cmdlets, install the [AWS\.Tools\.KeyManagementService](https://www.powershellgallery.com/packages/AWS.Tools.KeyManagementService/) module\. For more information, see the [AWS Tools for Windows PowerShell User Guide](https://docs.aws.amazon.com/powershell/latest/userguide/)\.

------