# GenerateDataKeyWithoutPlaintext<a name="ct-generatedatakeyplaintext"></a>

The following example shows a log file created by calling `GenerateDataKeyWithoutPlaintext`\.

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
        "eventTime": "2014-11-04T00:52:23Z",
        "eventSource": "kms.amazonaws.com",
        "eventName": "GenerateDataKeyWithoutPlaintext",
        "awsRegion": "us-east-1",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "AWS Internal",
        "errorCode": "InvalidKeyUsageException",
        "requestParameters": {
            "keyId": "d4f2a88d-5f9c-4807-b71d-4d0ee5225156",
            "numberOfBytes": 16
        },
        "responseElements": null,
        "requestID": "d6b8e411-63bc-11e4-bc2b-4198b6150d5c",
        "eventID": "f7734272-9ec5-4c80-9f36-528ebbe35e4a",
        "readOnly": true,
        "resources": [{
            "ARN": "arn:aws:kms:us-east-1:123456789012:key/d4f2a88d-5f9c-4807-b71d-4d0ee5225156",
            "accountId": "123456789012"
        }],
        "eventType": "AwsApiCall",
        "recipientAccountId": "123456789012"
    }
  ]
}
```