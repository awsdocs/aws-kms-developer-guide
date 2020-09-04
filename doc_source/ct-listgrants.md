# ListGrants<a name="ct-listgrants"></a>

The following example shows an AWS CloudTrail log entry for the [ListGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation\. For information about grants in AWS KMS, see [Using grants](grants.md)\.

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
        "eventTime": "2014-11-04T00:52:49Z",
        "eventSource": "kms.amazonaws.com",
        "eventName": "ListGrants",
        "awsRegion": "us-east-1",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "AWS Internal",
        "requestParameters": {
            "keyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "marker": "eyJncmFudElkIjoiMWY4M2U2ZmM0YTY2NDgxYjQ2Yzc4MTdhM2Y4YmQwMDFkZDNiYmQ1MGVlYTMyY2RmOWFiNWY1Nzc1NDNjYmNmMyIsImtleUFybiI6ImFybjphd3M6dHJlbnQtc2FuZGJveDp1cy1lYXN0LTE6NTc4Nzg3Njk2NTMwOmtleS9lYTIyYTc1MS1lNzA3LTQwZDAtOTJhYy0xM2EyOGZhOWViMTEifQ\u003d\u003d",
            "limit": 10
        },
        "responseElements": null,
        "requestID": "e5c23960-63bc-11e4-bc2b-4198b6150d5c",
        "eventID": "d24380f5-1b20-4253-8e92-dd0492b3bd3d",
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