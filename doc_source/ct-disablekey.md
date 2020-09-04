# DisableKey<a name="ct-disablekey"></a>

The following example shows an AWS CloudTrail log entry for the [DisableKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DisableKey.html) operation\. For information about enabling and disabling customer master keys in AWS KMS, see [Enabling and disabling keys](enabling-keys.md)\.

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
        "eventTime": "2014-11-04T00:52:43Z",
        "eventSource": "kms.amazonaws.com",
        "eventName": "DisableKey",
        "awsRegion": "us-east-1",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "AWS Internal",
        "requestParameters": {
            "keyId": "1234abcd-12ab-34cd-56ef-1234567890ab"
        },
        "responseElements": null,
        "requestID": "e26552bc-63bc-11e4-bc2b-4198b6150d5c",
        "eventID": "995c4653-3c53-4a06-a0f0-f5531997b741",
        "readOnly": false,
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