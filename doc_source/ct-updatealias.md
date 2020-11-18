# UpdateAlias<a name="ct-updatealias"></a>

The following example shows an AWS CloudTrail log entry for the [UpdateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html) operation\. The `resources` element includes fields for the alias and CMK resources\. For information about creating aliases in AWS KMS, see [Creating an alias](kms-alias.md#alias-create)\.

```
{
    "Records": [
    {
        "eventVersion": "1.05",
        "userIdentity": {
            "type": "IAMUser",
            "principalId": ""EX_PRINCIPAL_ID",",
            "arn": ""arn:aws:iam::111122223333:user/Alice",
            "accountId": "111122223333",
            "accessKeyId": "EXAMPLE_KEY_ID",
            "userName": "Alice"
        },
        "eventTime": "2020-11-13T23:18:15Z",
        "eventSource": "kms.amazonaws.com",
        "eventName": "UpdateAlias",
        "awsRegion": "us-west-2",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "AWS Internal",
        "requestParameters": {
            "aliasName": "alias/my_alias",
            "targetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab"
        },
        "responseElements": null,
        "requestID": "d9472f40-63bc-11e4-bc2b-4198b6150d5c",
        "eventID": "f72d3993-864f-48d6-8f16-e26e1ae8dff0",
        "readOnly": false,
        "resources": [
            {
                "accountId": "111122223333",
                "type": "AWS::KMS::Key",
                "ARN": "arn:aws:kms:us-west-2:111122223333:alias/my_alias"
            },
            {
                "accountId": "111122223333",
                "type": "AWS::KMS::Key",
                "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
            }
        ],
        "eventType": "AwsApiCall",
        "recipientAccountId": "111122223333"
    }
  ]
}
```