# Encrypt<a name="ct-encrypt"></a>

The following example shows an AWS CloudTrail log entry for the [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation\.

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
        "eventTime": "2014-11-04T00:53:11Z",
        "eventSource": "kms.amazonaws.com",
        "eventName": "Encrypt",
        "awsRegion": "us-east-1",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "AWS Internal",
        "requestParameters": {
            "encryptionContext": {
                "ContextKey1": "Value1"
            },
            "keyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
        },
        "responseElements": null,
        "requestID": "f3423043-63bc-11e4-bc2b-4198b6150d5c",
        "eventID": "91235988-eb87-476a-ac2c-0cdc244e6dca",
        "readOnly": true,
        "resources": [{
            "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "accountId": "111122223333"
        }],
        "eventType": "AwsServiceEvent",
        "recipientAccountId": "111122223333"
    }
  ]
}
```