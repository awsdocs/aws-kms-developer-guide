# Working with key policies<a name="programming-key-policies"></a>

The examples in this topic use the AWS KMS API to view and change the key policies of AWS KMS customer master keys \(CMKs\)\. 

For details about how to use key policies, IAM policies, and grants to manage access to your CMKs, see [Authentication and access control for AWS KMS](control-access.md)\. For help writing and formatting a JSON policy document, see the [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) in the *IAM User Guide*\.

**Topics**
+ [Listing key policy names](#list-policies)
+ [Getting a key policy](#get-policy)
+ [Setting a key policy](#put-policy)

## Listing key policy names<a name="list-policies"></a>

To get the names of key policies for a customer master key, use the [ListKeyPolicies](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeyPolicies.html) operation\. The only key policy name it returns is **default**\.

In languages that require a client object, these examples use the AWS KMS client object that you created in [Creating a client](programming-client.md)\.

------
#### [ Java ]

For details about the Java implementation, see the [listKeyPolicies method](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#listKeyPolicies-com.amazonaws.services.kms.model.ListKeyPoliciesRequest-) in the *AWS SDK for Java API Reference*\.

```
// List key policies
//
// Replace the following example key ARN with a valid key ID or key ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

ListKeyPoliciesRequest req = new ListKeyPoliciesRequest().withKeyId(keyId);
ListKeyPoliciesResult result = kmsClient.listKeyPolicies(req);
```

------
#### [ C\# ]

For details, see the [ListKeyPolicies method](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceListKeyPoliciesListKeyPoliciesRequest.html) in the *AWS SDK for \.NET*\.

```
// List key policies
//
// Replace the following example key ARN with a valid key ID or key ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

ListKeyPoliciesRequest listKeyPoliciesRequest = new ListKeyPoliciesRequest()
{
    KeyId = keyId
};
ListKeyPoliciesResponse listKeyPoliciesResponse = kmsClient.ListKeyPolicies(listKeyPoliciesRequest);
```

------
#### [ Python ]

For details, see the [list\_key\_policies method](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.list_key_policies) in the AWS SDK for Python \(Boto3\)\.

```
# List key policies

# Replace the following example key ARN with a valid key ID or key ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

response = kms_client.list_key_policies(
    KeyId=key_id
)
```

------
#### [ Ruby ]

For details, see the [list\_key\_policies](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#list_key_policies-instance_method) instance method in the [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# List key policies

# Replace the following example key ARN with a valid key ID or key ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

response = kmsClient.list_key_policies({
  key_id: key_id
})
```

------
#### [ PHP ]

For details, see the [ListKeyPolicies method](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-kms-2014-11-01.html#listkeypolicies) in the *AWS SDK for PHP*\.

```
// List key policies
//
// Replace the following example key ARN with a valid key ID or key ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';

$result = $KmsClient->listKeyPolicies([
    'KeyId' => $keyId
]);
```

------
#### [ Node\.js ]

For details, see the [listKeyPolicies property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#listKeyPolicies-property) in the *AWS SDK for JavaScript in Node\.js*\.

```
// List key policies
//
// Replace the following example key ARN with a valid key ID or key ARN
const KeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';

kmsClient.listKeyPolicies({ KeyId }, (err, data) => {
  ...
});
```

------
#### [ PowerShell ]

To list the name of the default key policy, use the [Get\-KMSKeyPolicyList](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-KMSKeyPolicyList.html) cmdlet\.

```
# List key policies

# Replace the following example key ARN with a valid key ID or key ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
$response = Get-KMSKeyPolicyList -KeyId $keyId
```

To use the AWS KMS PowerShell cmdlets, install the [AWS\.Tools\.KeyManagementService](https://www.powershellgallery.com/packages/AWS.Tools.KeyManagementService/) module\. For more information, see the [AWS Tools for Windows PowerShell User Guide](https://docs.aws.amazon.com/powershell/latest/userguide/)\.

------

## Getting a key policy<a name="get-policy"></a>

To get the key policy for a customer master key, use the [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation\.

GetKeyPolicy requires a policy name\. The only valid policy name is **default**\.

In languages that require a client object, these examples use the AWS KMS client object that you created in [Creating a client](programming-client.md)\.

------
#### [ Java ]

For details, see the [getKeyPolicy method](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#getKeyPolicy-com.amazonaws.services.kms.model.GetKeyPolicyRequest-) in the *AWS SDK for Java API Reference*\.

```
// Get the policy for a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String policyName = "default";

GetKeyPolicyRequest req = new GetKeyPolicyRequest().withKeyId(keyId).withPolicyName(policyName);
GetKeyPolicyResult result = kmsClient.getKeyPolicy(req);
```

------
#### [ C\# ]

For details, see the [GetKeyPolicy method](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceGetKeyPolicyGetKeyPolicyRequest.html) in the *AWS SDK for \.NET*\.

```
// Get the policy for a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String policyName = "default";

GetKeyPolicyRequest getKeyPolicyRequest = new GetKeyPolicyRequest()
{
    KeyId = keyId,
    PolicyName = policyName
};
GetKeyPolicyResponse getKeyPolicyResponse = kmsClient.GetKeyPolicy(getKeyPolicyRequest);
```

------
#### [ Python ]

For details, see the [get\_key\_policy method](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.get_key_policy) in the AWS SDK for Python \(Boto3\)\.

```
# Get the policy for a CMK

# Replace the following example key ARN with a valid key ID or key ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
policy_name = 'default'

response = kms_client.get_key_policy(
    KeyId=key_id,
    PolicyName=policy_name
)
```

------
#### [ Ruby ]

For details, see the [get\_key\_policy](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#get_key_policy-instance_method) instance method in the [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Get the policy for a CMK

# Replace the following example key ARN with a valid key ID or key ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
policy_name = 'default'

response = kmsClient.get_key_policy({
  key_id: key_id,
  policy_name: policy_name
})
```

------
#### [ PHP ]

For details, see the [GetKeyPolicy method](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-kms-2014-11-01.html#getkeypolicy) in the *AWS SDK for PHP*\.

```
// Get the policy for a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
$policyName = "default";

$result = $KmsClient->getKeyPolicy([
    'KeyId' => $keyId, 
    'PolicyName' => $policyName
]);
```

------
#### [ Node\.js ]

For details, see the [getKeyPolicy property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#getKeyPolicy-property) in the *AWS SDK for JavaScript in Node\.js*\.

```
// Get the policy for a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
const KeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
const PolicyName = 'default';
kmsClient.getKeyPolicy({ KeyId, PolicyName }, (err, data) => {
  ...
});
```

------
#### [ PowerShell ]

To get the key policy for a CMK, use the [Get\-KMSKeyPolicy](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-KMSKeyPolicy.html) cmdlet\. This cmdlet returns the key policy as a string \(System\.String\) that you can use in a [Write\-KMSKeyPolicy](https://docs.aws.amazon.com/powershell/latest/reference/items/Write-KMSKeyPolicy.html) \(`PutKeyPolicy`\) command\. To convert the policies in the JSON string to `PSCustomObject` objects, use the [ConvertFrom\-JSON](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/convertfrom-json) cmdlet\.

```
# Get the policy for a CMK

# Replace the following example key ARN with a valid key ID or key ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
$policyName = 'default'

$response = Get-KMSKeyPolicy -KeyId $keyId -PolicyName $policyName
```

To use the AWS KMS PowerShell cmdlets, install the [AWS\.Tools\.KeyManagementService](https://www.powershellgallery.com/packages/AWS.Tools.KeyManagementService/) module\. For more information, see the [AWS Tools for Windows PowerShell User Guide](https://docs.aws.amazon.com/powershell/latest/userguide/)\.

------

## Setting a key policy<a name="put-policy"></a>

To create or replace the key policy for a CMK, use the [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operation\.

`PutKeyPolicy` requires a policy name\. The only valid policy name is **default**\.

In languages that require a client object, these examples use the AWS KMS client object that you created in [Creating a client](programming-client.md)\.

------
#### [ Java ]

For details, see the [putKeyPolicy method](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#putKeyPolicy-com.amazonaws.services.kms.model.PutKeyPolicyRequest-) in the *AWS SDK for Java API Reference*\.

```
// Set a key policy for a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String policyName = "default";
String policy = "{" +
                "  \"Version\": \"2012-10-17\"," +
                "  \"Statement\": [{" +
                "    \"Sid\": \"Allow access for ExampleUser\"," +
                "    \"Effect\": \"Allow\"," +
                // Replace the following example user ARN with a valid one
                "    \"Principal\": {\"AWS\": \"arn:aws:iam::111122223333:user/ExampleUser\"}," +
                "    \"Action\": [" +
                "      \"kms:Encrypt\"," +
                "      \"kms:GenerateDataKey*\"," +
                "      \"kms:Decrypt\"," +
                "      \"kms:DescribeKey\"," +
                "      \"kms:ReEncrypt*\"" +
                "    ]," +
                "    \"Resource\": \"*\"" +
                "  }]" +
                "}";
                
PutKeyPolicyRequest req = new PutKeyPolicyRequest().withKeyId(keyId).withPolicy(policy).withPolicyName(policyName);
kmsClient.putKeyPolicy(req);
```

------
#### [ C\# ]

For details, see the [PutKeyPolicy method](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServicePutKeyPolicyPutKeyPolicyRequest.html) in the *AWS SDK for \.NET*\.

```
// Set a key policy for a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String policyName = "default";
String policy = "{" +
                "  \"Version\": \"2012-10-17\"," +
                "  \"Statement\": [{" +
                "    \"Sid\": \"Allow access for ExampleUser\"," +
                "    \"Effect\": \"Allow\"," +
                // Replace the following example user ARN with a valid one
                "    \"Principal\": {\"AWS\": \"arn:aws:iam::111122223333:user/ExampleUser\"}," +
                "    \"Action\": [" +
                "      \"kms:Encrypt\"," +
                "      \"kms:GenerateDataKey*\"," +
                "      \"kms:Decrypt\"," +
                "      \"kms:DescribeKey\"," +
                "      \"kms:ReEncrypt*\"" +
                "    ]," +
                "    \"Resource\": \"*\"" +
                "  }]" +
                "}";
                
PutKeyPolicyRequest putKeyPolicyRequest = new PutKeyPolicyRequest()
{
    KeyId = keyId,
    Policy = policy,
    PolicyName = policyName
};
kmsClient.PutKeyPolicy(putKeyPolicyRequest);
```

------
#### [ Python ]

For details, see the [put\_key\_policy method](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.put_key_policy) in the AWS SDK for Python \(Boto3\)\.

```
# Set a key policy for a CMK

# Replace the following example key ARN with a valid key ID or key ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
policy_name = 'default'
policy = """
{
    "Version": "2012-10-17",
    "Statement": [{
        "Sid": "Allow access for ExampleUser",
        "Effect": "Allow",
        "Principal": {"AWS": "arn:aws:iam::111122223333:user/ExampleUser"},
        "Action": [
            "kms:Encrypt",
            "kms:GenerateDataKey*",
            "kms:Decrypt",
            "kms:DescribeKey",
            "kms:ReEncrypt*"
        ],
        "Resource": "*"
    }]
}"""

response = kms_client.put_key_policy(
    KeyId=key_id,
    Policy=policy,
    PolicyName=policy_name
)
```

------
#### [ Ruby ]

For details, see the [put\_key\_policy](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS/Client.html#put_key_policy-instance_method) instance method in the [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/KMS.html)\.

```
# Set a key policy for a CMK

# Replace the following example key ARN with a valid key ID or key ARN
key_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
policy_name = 'default'
policy = "{" +
  "  \"Version\": \"2012-10-17\"," +
  "  \"Statement\": [{" +
  "    \"Sid\": \"Allow access for ExampleUser\"," +
  "    \"Effect\": \"Allow\"," +
  # Replace the following example user ARN with a valid one
  "    \"Principal\": {\"AWS\": \"arn:aws:iam::111122223333:user/ExampleUser\"}," +
  "    \"Action\": [" +
  "      \"kms:Encrypt\"," +
  "      \"kms:GenerateDataKey*\"," +
  "      \"kms:Decrypt\"," +
  "      \"kms:DescribeKey\"," +
  "      \"kms:ReEncrypt*\"" +
  "    ]," +
  "    \"Resource\": \"*\"" +
  "  }]" +
  "}"

response = kmsClient.put_key_policy({
  key_id: key_id,
  policy: policy,
  policy_name: policy_name
})
```

------
#### [ PHP ]

For details, see the [PutKeyPolicy method](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-kms-2014-11-01.html#putkeypolicy) in the *AWS SDK for PHP*\.

```
// Set a key policy for a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
$policyName = "default";

$result = $KmsClient->putKeyPolicy([
    'KeyId' => $keyId, 
    'PolicyName' => $policyName, 
    'Policy' => '{ 
        "Version": "2012-10-17", 
        "Id": "custom-policy-2016-12-07", 
        "Statement": [ 
            { "Sid": "Enable IAM User Permissions", 
            "Effect": "Allow", 
            "Principal": 
               { "AWS": "arn:aws:iam::111122223333:user/root" }, 
            "Action": [ "kms:*" ], 
            "Resource": "*" }, 
            { "Sid": "Enable IAM User Permissions", 
            "Effect": "Allow", 
            "Principal":
               { "AWS": "arn:aws:iam::111122223333:user/ExampleUser" }, 
            "Action": [
                "kms:Encrypt*",
                "kms:GenerateDataKey*",
                "kms:Decrypt*",
                "kms:DescribeKey*",
                "kms:ReEncrypt*"
            ], 
            "Resource": "*" }
        ]
    } ' 
]);
```

------
#### [ Node\.js ]

For details, see the [putKeyPolicy property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/KMS.html#putKeyPolicy-property) in the *AWS SDK for Node\.js*\.

```
// Set a key policy for a CMK
//
// Replace the following example key ARN with a valid key ID or key ARN
const KeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab';
const PolicyName = 'default';
const Policy = `{
    "Version": "2012-10-17",
    "Id": "custom-policy-2016-12-07",
    "Statement": [
        {
            "Sid": "Enable IAM User Permissions",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::111122223333:root"
            },
            "Action": "kms:*",
            "Resource": "*"
        },
        {
            "Sid": "Enable IAM User Permissions",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
            },
            "Action": [
                "kms:Encrypt*",
                "kms:GenerateDataKey*",
                "kms:Decrypt*",
                "kms:DescribeKey*",
                "kms:ReEncrypt*"
            ],
            "Resource": "*"
        } 
    ]
}`; // The key policy document
  
kmsClient.putKeyPolicy({ KeyId, Policy, PolicyName }, (err, data) => {
  ...
});
```

------
#### [ PowerShell ]

To set a key policy for a CMK, use the [Write\-KMSKeyPolicy](https://docs.aws.amazon.com/powershell/latest/reference/items/Write-KMSKeyPolicy.html) cmdlet\. This cmdlet doesn't return any output\. To verify that the command was effective, use the [Get\-KMSKeyPolicy](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-KMSKeyPolicy.html) cmdlet\.

The `Policy` parameter takes a string\. Enclose the string in single quotes to make it a literal string\. You don't have to use continuation characters or escape characters in the literal string\.

```
# Set a key policy for a CMK

# Replace the following example key ARN with a valid key ID or key ARN
$keyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
$policyName = 'default'
$policy = '{
    "Version": "2012-10-17", 
    "Statement": [
        {
            "Sid": "Enable IAM User Permissions",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::111122223333:root"
            },
            "Action": "kms:*",
            "Resource": "*"
        },
        {
            "Sid": "Enable IAM User Permissions",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
            },
            "Action": [
                "kms:Encrypt*",
                "kms:GenerateDataKey*",
                "kms:Decrypt*",
                "kms:DescribeKey*",
                "kms:ReEncrypt*"
            ],
            "Resource": "*"
        }]
    }'

Write-KMSKeyPolicy -KeyId $keyId -PolicyName $policyName -Policy $policy
```

To use the AWS KMS PowerShell cmdlets, install the [AWS\.Tools\.KeyManagementService](https://www.powershellgallery.com/packages/AWS.Tools.KeyManagementService/) module\. For more information, see the [AWS Tools for Windows PowerShell User Guide](https://docs.aws.amazon.com/powershell/latest/userguide/)\.

------