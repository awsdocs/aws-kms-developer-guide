# CreateKey<a name="ct-createkey"></a>

The following example shows an AWS CloudTrail log entry for a [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation that creates a [symmetric customer master key](symm-asymm-concepts.md#symmetric-cmks)\. For information about creating customer master keys in AWS KMS, see [Creating keys](create-keys.md)\.

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
        "eventTime": "2020-06-30T02:34:07Z",
        "eventSource": "kms.amazonaws.com",
        "eventName": "CreateKey",
        "awsRegion": "us-west-2",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "AWS Internal",
        "requestParameters": {
            "policy": "{\n  \"Version\":\"2012-10-17\",\n  \"Statement\":[{\n    \"Effect\":\"Allow\",\n    \"Principal\":{\"AWS\":\"arn:aws:iam::123456789012:user/Alice\"},\n    \"Action\":\"kms:*\",\n    \"Resource\":\"*\"\n  }, {\n    \"Effect\":\"Allow\",\n    \"Principal\":{\"AWS\":\"arn:aws:iam::012345678901:user/Bob\"},\n    \"Action\":\"kms:CreateGrant\",\n    \"Resource\":\"*\"\n  }, {\n    \"Effect\":\"Allow\",\n    \"Principal\":{\"AWS\":\"arn:aws:iam::012345678901:user/Charlie\"},\n    \"Action\":\"kms:Encrypt\",\n    \"Resource\":\"*\"\n}]\n}",
            "description": "",
            "keyUsage": "ENCRYPT_DECRYPT",
            "customerMasterKeySpec": "SYMMETRIC_DEFAULT",
            "origin": "AWS_KMS",
            "bypassPolicyLockoutSafetyCheck": false
        },
        "responseElements": {
        "keyMetadata": {
            "aWSAccountId": "111122223333",
            "keyId": "1234abcd-12ab-34cd-56ef-1234567890ab",
            "arn": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "creationDate": "Jun 30, 2020 2:34:07 AM",
            "enabled": true,
            "description": "",
            "keyUsage": "ENCRYPT_DECRYPT",
            "keyState": "Enabled",
            "origin": "AWS_KMS",
            "keyManager": "CUSTOMER",
            "customerMasterKeySpec": "SYMMETRIC_DEFAULT",
            "encryptionAlgorithms": [
                "SYMMETRIC_DEFAULT"
            ]
        },
        "requestID": "ebe8ee68-63bc-11e4-bc2b-4198b6150d5c",
        "eventID": "ba116326-1792-4784-87dd-a688d1cb42ec",
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