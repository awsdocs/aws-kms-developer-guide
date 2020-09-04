# DeleteExpiredKeyMaterial<a name="ct-deleteexpiredkeymaterial"></a>

When you import key material into a customer master key \(CMK\), you can set an expiration date and time for that key material\. AWS KMS records an entry in your CloudTrail log when you [import the key material](ct-importkeymaterial.md) \(with the expiration settings\) and when AWS KMS deletes the expired key material\. For information about creating CMKs with imported key material, see [Importing key material in AWS Key Management Service \(AWS KMS\)](importing-keys.md)\.

The following example shows an AWS CloudTrail log entry generated when AWS KMS deletes the expired key material\.

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "accountId": "111122223333",
        "invokedBy": "AWS Internal"
    },
    "eventTime": "2021-01-01T16:00:00Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "DeleteExpiredKeyMaterial",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "AWS Internal",
    "userAgent": "AWS Internal",
    "requestParameters": null,
    "responseElements": null,
    "eventID": "cfa932fd-0d3a-4a76-a8b8-616863a2b547",
    "readOnly": false,
    "resources": [
        {
            "accountId": "111122223333",
            "type": "AWS::KMS::Key",
            "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
        }
    ],
    "eventType": "AwsServiceEvent",
    "recipientAccountId": "111122223333",
    "serviceEventDetails": {
        "keyId": "1234abcd-12ab-34cd-56ef-1234567890ab"
    }
}
```