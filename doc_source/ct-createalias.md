# CreateAlias<a name="ct-createalias"></a>

The following example shows an AWS CloudTrail log entry for the [CreateAlias](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateAlias.html) operation\. The resources element includes fields for the alias and CMK resources\. For information about creating aliases in AWS KMS, see [Creating an alias](kms-alias.md#alias-create)\.

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
            "userName": "Alice",
            "sessionContext": {
                "attributes": {
                    "mfaAuthenticated": "false",
                    "creationDate": "2014-11-04T00:52:27Z"
                }
            }
        },
        "eventTime": "2014-11-04T00:52:27Z",
        "eventSource": "kms.amazonaws.com",
        "eventName": "CreateAlias",
        "awsRegion": "us-east-1",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "AWS Internal",
        "requestParameters": {
            "aliasName": "alias/my_alias",
            "targetKeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
        },
        "responseElements": null,
        "requestID": "d9472f40-63bc-11e4-bc2b-4198b6150d5c",
        "eventID": "f72d3993-864f-48d6-8f16-e26e1ae8dff0",
        "readOnly": false,
        "resources": [{
            "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "accountId": "111122223333"
        },
        {
            "ARN": "arn:aws:kms:us-east-1:123456789012:alias/my_alias",
            "accountId": "111122223333"
        }],
        "eventType": "AwsApiCall",
        "recipientAccountId": "111122223333"
    }
  ]
}
```