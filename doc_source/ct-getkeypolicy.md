# GetKeyPolicy<a name="ct-getkeypolicy"></a>

The following example shows an AWS CloudTrail log entry for the [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation\. For information about viewing the key policy for an AWS KMS customer master key \(CMK\), see [Viewing a key policy](key-policy-viewing.md)\.

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
        "eventTime": "2014-11-04T00:50:30Z",
        "eventSource": "kms.amazonaws.com",
        "eventName": "GetKeyPolicy",
        "awsRegion": "us-east-1",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "AWS Internal",
        "requestParameters": {
            "keyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "policyName": "default"
        },
        "responseElements": null,
        "requestID": "93746dd6-63bc-11e4-bc2b-4198b6150d5c",
        "eventID": "4aa7e4d5-d047-452a-a5a6-2cce282a7e82",
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