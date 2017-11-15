# GenerateDataKey<a name="ct-generatedatakey"></a>

The following example shows a log file created by calling `GenerateDataKey`\.

```
{
    "Records": [
    {
        "eventVersion": "1.02",
        "userIdentity": {
            "type": "IAMUser",
            "principalId": "EX_PRINCIPAL_ID",
            "arn": "arn:aws:iam::123456789012:user/Alice",
            "accountId": "123456789012",
            "accessKeyId": "EXAMPLE_KEY_ID",
            "userName": "Alice"
        },
        "eventTime": "2014-11-04T00:52:40Z",
        "eventSource": "kms.amazonaws.com",
        "eventName": "GenerateDataKey",
        "awsRegion": "us-east-1",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "AWS Internal",
        "requestParameters": {
            "keyId": "637e8678-3d08-4922-a650-e77eb1591db5",
            "numberOfBytes": 32
        },
        "responseElements": null,
        "requestID": "e0eb83e3-63bc-11e4-bc2b-4198b6150d5c",
        "eventID": "a9dea4f9-8395-46c0-942c-f509c02c2b71",
        "readOnly": true,
        "resources": [{
            "ARN": "arn:aws:kms:us-east-1:123456789012:key/637e8678-3d08-4922-a650-e77eb1591db5",
            "accountId": "123456789012"
        }],
        "eventType": "AwsApiCall",
        "recipientAccountId": "123456789012"
    }
  ]
}
```