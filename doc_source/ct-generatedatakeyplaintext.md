# GenerateDataKeyWithoutPlaintext<a name="ct-generatedatakeyplaintext"></a>

The following example shows an AWS CloudTrail log entry for the [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html) operation\.

```
{
    "Records": [
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
        "eventTime": "2014-11-04T00:52:23Z",
        "eventSource": "kms.amazonaws.com",
        "eventName": "GenerateDataKeyWithoutPlaintext",
        "awsRegion": "us-east-1",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "AWS Internal",
        "errorCode": "InvalidKeyUsageException",
        "requestParameters": {
            "keyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "numberOfBytes": 16
        },
        "responseElements": null,
        "requestID": "d6b8e411-63bc-11e4-bc2b-4198b6150d5c",
        "eventID": "f7734272-9ec5-4c80-9f36-528ebbe35e4a",
        "readOnly": true,
        "resources": [{
            "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "accountId": "111122223333"
        }],
        "eventType": "AwsApiCall",
        "recipientAccountId": "111122223333"
    }
  ]
}
```