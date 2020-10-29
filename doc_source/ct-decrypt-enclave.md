# Decrypt \(from an enclave\)<a name="ct-decrypt-enclave"></a>

The following example shows an AWS CloudTrail log entry for a `kms-decrypt` operation in the [Nitro Enclaves SDK](https://github.com/aws/aws-nitro-enclaves-sdk-c)\. The `kms-decrypt` API calls the AWS KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation with a parameter that includes a signed [attestation document](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/nitro-enclave-concepts.html#term-attestdoc) from the enclave\. 

AWS Nitro Enclaves is an Amazon EC2 capability that lets you create isolated compute environments called [enclaves](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/nitro-enclave-concepts.html#term-enclave) to protect and process highly sensitive data\. For more information about AWS Nitro Enclaves and its integration with AWS KMS, see [Nitro Enclaves](https://docs.aws.amazon.com/enclaves/latest/user/) in the Amazon EC2 User Guide for Linux Instances\.

When the call originates in an enclave, the CloudTrail log includes recipient data that represents the measurements of the enclave\.

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "EX_PRINCIPAL_ID",
        "arn": "arn:aws:iam::111122223333:user/Alice",
        "accountId": "111122223333",
        "accessKeyId": "EXAMPLE_KEY_ID",
        "userName": "Alice"
    },
    "eventTime": "2020-07-27T22:58:24Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "Decrypt",
    "awsRegion": "us-west-2",
    "sourceIPAddress": "192.0.2.0",
    "userAgent": "AWS Internal",
    "requestParameters": {
        "encryptionAlgorithm": "SYMMETRIC_DEFAULT",
        "keyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
    },
    "responseElements": null,
    "additionalEventData": {
        "recipient": {
            "attestationDocumentModuleId": "i-123456789abcde123-enc123456789abcde12",
            "attestationDocumentEnclaveImageDigest": "ee0d451a2ff9aaaa9bccd07700b9cab123a0ac2386ef7e88ad5ea6c72ebabea840957328e2ec890b408c9b06cb8ebe6a",
        }
    },
    "requestID": "b4a65126-30d5-4b28-98b9-9153da559963",
    "eventID": "e5a2f202-ba1a-467c-b4ba-f729d45ae521",
    "readOnly": true,
    "resources": [
        {
            "accountId": "111122223333",
            "type": "AWS::KMS::Key",
            "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
        }
    ],
    "eventType": "AwsApiCall",
    "recipientAccountId": "111122223333"
}
```