# CreateGrant<a name="ct-creategrant"></a>

The following example shows an AWS CloudTrail log entry for the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation\. For information about creating grants in AWS KMS, see [Using grants](grants.md)\.

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
        "eventTime": "2014-11-04T00:53:12Z",
        "eventSource": "kms.amazonaws.com",
        "eventName": "CreateGrant",
        "awsRegion": "us-east-1",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "AWS Internal",
        "requestParameters": {
            "keyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "constraints": {
                "encryptionContextSubset": {
                    "ContextKey1": "Value1"
                }
            },
            "operations": ["Encrypt",
            "RetireGrant"],
            "granteePrincipal": "EX_PRINCIPAL_ID"
        },
        "responseElements": {
            "grantId": "f020fe75197b93991dc8491d6f19dd3cebb24ee62277a05914386724f3d48758"
        },
        "requestID": "f3c08808-63bc-11e4-bc2b-4198b6150d5c",
        "eventID": "5d529779-2d27-42b5-92da-91aaea1fc4b5",
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