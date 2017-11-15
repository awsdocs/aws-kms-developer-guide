# GenerateRandom<a name="ct-generaterandom"></a>

The following example shows a log file created by calling `GenerateRandom`\.

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
        "eventTime": "2014-11-04T00:52:37Z",
        "eventSource": "kms.amazonaws.com",
        "eventName": "GenerateRandom",
        "awsRegion": "us-east-1",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "AWS Internal",
        "requestParameters": null,
        "responseElements": null,
        "requestID": "df1e3de6-63bc-11e4-bc2b-4198b6150d5c",
        "eventID": "239cb9f7-ae05-4c94-9221-6ea30eef0442",
        "readOnly": true,
        "resources": [],
        "eventType": "AwsApiCall",
        "recipientAccountId": "123456789012"
    }
  ]
}
```