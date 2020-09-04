# GenerateDataKey<a name="ct-generatedatakey"></a>

The following example shows an AWS CloudTrail log entry for the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation\.

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
        "eventTime": "2014-11-04T00:52:40Z",
        "eventSource": "kms.amazonaws.com",
        "eventName": "GenerateDataKey",
        "awsRegion": "us-east-1",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "AWS Internal",
        "requestParameters": {
            "keyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "numberOfBytes": 32
        },
        "responseElements": null,
        "requestID": "e0eb83e3-63bc-11e4-bc2b-4198b6150d5c",
        "eventID": "a9dea4f9-8395-46c0-942c-f509c02c2b71",
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