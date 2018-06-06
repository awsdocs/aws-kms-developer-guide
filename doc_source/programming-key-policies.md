# Working with Key Policies<a name="programming-key-policies"></a>

The examples in this topic use the AWS KMS API to view and change the key policies of AWS KMS customer master keys \(CMKs\)\. For details about how to use key policies and IAM policies to manage access to your CMKs, see [Authentication and Access Control for AWS KMS](control-access.md)\.

**Topics**
+ [Listing Key Policy Names](#list-policies)
+ [Getting a Key Policy](#get-policy)
+ [Setting a Key Policy](#put-policy)

## Listing Key Policy Names<a name="list-policies"></a>

To get the names of key policies for a customer master key, use the [ListKeyPolicies](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeyPolicies.html) operation\. The only key policy name it returns is **default**\.

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details about the Java implementation, see the [listKeyPolicies method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#listKeyPolicies-com.amazonaws.services.kms.model.ListKeyPoliciesRequest-) in the *AWS SDK for Java API Reference*\.

```
// List key policy names
//
// Replace the fictitious key ARN with a valid key ID
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

ListKeyPoliciesRequest req = new ListKeyPoliciesRequest().withKeyId(keyId);
ListKeyPoliciesResult result = kmsClient.listKeyPolicies(req);
```

------
#### [ C\# ]

For details, see the [ListKeyPolicies method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceListKeyPoliciesListKeyPoliciesRequest.html) in the *AWS SDK for \.NET*\.

```
// List key policy names
//
// Replace the fictitious key ARN with a valid key ID
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

ListKeyPoliciesRequest listKeyPoliciesRequest = new ListKeyPoliciesRequest()
{
    KeyId = keyId
};
ListKeyPoliciesResponse listKeyPoliciesResponse = kmsClient.ListKeyPolicies(listKeyPoliciesRequest);
```

------

## Getting a Key Policy<a name="get-policy"></a>

To get the key policy for a customer master key, use the [GetKeyPolicy](http://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation\.

GetKeyPolicy requires a policy name\. The only valid policy name is **default**\.

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [getKeyPolicy method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#getKeyPolicy-com.amazonaws.services.kms.model.GetKeyPolicyRequest-) in the *AWS SDK for Java API Reference*\.

```
// Get the policy for a CMK
//
// Replace the following fictitious key ARN with a valid key ID
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String policyName = "default";

GetKeyPolicyRequest req = new GetKeyPolicyRequest().withKeyId(keyId).withPolicyName(policyName);
GetKeyPolicyResult result = kmsClient.getKeyPolicy(req);
```

------
#### [ C\# ]

For details, see the [GetKeyPolicy method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServiceGetKeyPolicyGetKeyPolicyRequest.html) in the *AWS SDK for \.NET*\.

```
// Get the policy for a CMK
//
// Replace the following fictitious key ARN with a valid key ID
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

## Setting a Key Policy<a name="put-policy"></a>

To establish or change a key policy for a CMK, use the [PutKeyPolicy](http://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operation\.

PutKeyPolicy requires a policy name\. The only valid policy name is **default**\.

This example uses the `kmsClient` client object that you created in [Creating a Client](programming-client.md)\.

------
#### [ Java ]

For details, see the [putKeyPolicy method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#putKeyPolicy-com.amazonaws.services.kms.model.PutKeyPolicyRequest-) in the *AWS SDK for Java API Reference*\.

```
// Set a key policy for a CMK
//

// Replace the following fictitious key ARN with a valid key ID
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String policyName = "default";
String policy = "{" +
                "  \"Version\": \"2012-10-17\"," +
                "  \"Statement\": [{" +
                "    \"Sid\": \"Allow access for ExampleUser\"," +
                "    \"Effect\": \"Allow\"," +
                // Replace the following user ARN with one for a real user.
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

For details, see the [PutKeyPolicy method](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/MKeyManagementServicePutKeyPolicyPutKeyPolicyRequest.html) in the *AWS SDK for \.NET*\.

```
// Set a key policy for a CMK
//

// Replace the following fictitious key ARN with a valid key ID
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String policyName = "default";
String policy = "{" +
                "  \"Version\": \"2012-10-17\"," +
                "  \"Statement\": [{" +
                "    \"Sid\": \"Allow access for ExampleUser\"," +
                "    \"Effect\": \"Allow\"," +
                // Replace the following user ARN with one for a real user.
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