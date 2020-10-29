# GenerateRandom \(from an enclave\)<a name="ct-generate-random-enclave"></a>

The following example shows an AWS CloudTrail log entry for a `kms-generate-random` operation in the [Nitro Enclaves SDK](https://github.com/aws/aws-nitro-enclaves-sdk-c)\. The `kms-generate-random` API calls the AWS KMS [GenerateRandom](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateRandom.html) operation with a parameter that includes a signed [attestation document](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/nitro-enclave-concepts.html#term-attestdoc) from the enclave\. 

AWS Nitro Enclaves is an Amazon EC2 capability that lets you create isolated compute environments called [enclaves](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/nitro-enclave-concepts.html#term-enclave) to protect and process highly sensitive data\. For more information about AWS Nitro Enclaves and its integration with AWS KMS, see [Nitro Enclaves](https://docs.aws.amazon.com/enclaves/latest/user/) in the Amazon EC2 User Guide for Linux Instances\.

When the call originates in an enclave, the CloudTrail log includes recipient data that represents the measurements of the enclave\.

```
{
    "eventVersion": "1.02",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "EX_PRINCIPAL_ID",
        "arn": "arn:aws:iam::111122223333:user/Alice",
        "accountId": "111122223333",
        "accessKeyId": "EXAMPLE_KEY_ID",
        "userName": "Alice"
    },
    "eventTime": "2014-11-04T00:52:37Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "GenerateRandom",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "192.0.2.0",
    "userAgent": "AWS Internal",
    "requestParameters": null,
    "responseElements": null,
    "additionalEventData": {
        "recipient": {
            "attestationDocumentModuleId": "i-123456789abcde123-enc123456789abcde12",
            "attestationDocumentEnclaveImageDigest": "ee0d451a2ff9aaaa9bccd07700b9cab123a0ac2386ef7e88ad5ea6c72ebabea840957328e2ec890b408c9b06cb8ebe6a"
        }
    },
    "requestID": "df1e3de6-63bc-11e4-bc2b-4198b6150d5c",
    "eventID": "239cb9f7-ae05-4c94-9221-6ea30eef0442",
    "readOnly": true,
    "resources": [],
    "eventType": "AwsApiCall",
    "recipientAccountId": "111122223333"
}
```