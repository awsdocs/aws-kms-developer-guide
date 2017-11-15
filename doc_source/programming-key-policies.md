# Working with Key Policies<a name="programming-key-policies"></a>

Use the [AWS SDK for Java](https://aws.amazon.com/sdk-for-java/) and the following sample code to list, get, and set key policies for AWS KMS customer master keys \(CMKs\)\. This sample code requires that you previously instantiated an `AWSKMSClient` as `kms`\.


+ [Listing Key Policy Names](#list-policies)
+ [Getting a Key Policy](#get-policy)
+ [Setting a Key Policy](#put-policy)

## Listing Key Policy Names<a name="list-policies"></a>

To list the names of key policies for a customer master key, use the [ListKeyPolicies](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeyPolicies.html) operation\. Currently, the only key policy name it returns is **default**\. For details about the Java implementation, see the [listKeyPolicies method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#listKeyPolicies-com.amazonaws.services.kms.model.ListKeyPoliciesRequest-) in the *AWS SDK for Java API Reference*\.

```
// List key policy names
//
// Replace the fictitious key ARN with a valid key ID
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

ListKeyPoliciesRequest req = new ListKeyPoliciesRequest().withKeyId(keyId);
ListKeyPoliciesResult result = kms.listKeyPolicies(req);
```

## Getting a Key Policy<a name="get-policy"></a>

To get information about a particular key policy of a customer master key, use the [GetKeyPolicy](http://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation\. For details about the Java implementation, see the [getKeyPolicy method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#getKeyPolicy-com.amazonaws.services.kms.model.GetKeyPolicyRequest-) in the *AWS SDK for Java API Reference*\.

GetKeyPolicy requires a policy name\. You can use the [ListKeyPolicies](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeyPolicies.html) operation to get the policy name, but currently, the only policy name is **default**\. 

```
// Get the policy for a CMK
//
// Replace the following fictitious key ARN with a valid key ID
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String policyName = "default";

GetKeyPolicyRequest req = new GetKeyPolicyRequest().withKeyId(keyId).withPolicyName(policyName);
GetKeyPolicyResult result = kms.getKeyPolicy(req);
```

## Setting a Key Policy<a name="put-policy"></a>

To establish or change a key policy for a CMK, use the [PutKeyPolicy](http://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operation\. For details about the Java implementation, see the [putKeyPolicy method](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/AWSKMSClient.html#putKeyPolicy-com.amazonaws.services.kms.model.PutKeyPolicyRequest-) in the *AWS SDK for Java API Reference*\.

PutKeyPolicy requires a policy name\. You can use the [ListKeyPolicies](http://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeyPolicies.html) operation to get the policy name, but currently, the only policy name is **default**\. 

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
kms.putKeyPolicy(req);
```