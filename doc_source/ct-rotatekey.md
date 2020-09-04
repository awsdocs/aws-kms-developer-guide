# RotateKey<a name="ct-rotatekey"></a>

The following example shows an AWS CloudTrail log entry of the operation that rotates a customer master key \(CMK\)\. AWS KMS calls this operation when it is time to rotate a CMK on which automatic key rotation is enabled\. When you enable automatic key rotation \([EnableKeyRotation](https://docs.aws.amazon.com/kms/latest/APIReference/API_EnableKeyRotation.html)\), AWS KMS rotates the CMK 365 days later and every 365 days thereafter\.

For an example of the CloudTrail log entry that records the `EnableKeyRotation` operation, see [EnableKeyRotation](ct-enablekeyrotation.md)\. For information about rotating AWS KMS customer master keys, see [Rotating customer master keys](rotate-keys.md)\.

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "accountId": "111122223333",
        "invokedBy": "AWS Internal"
    },
    "eventTime": "2021-01-14T01:41:59Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "RotateKey",
    "awsRegion": "us-west-2",
    "sourceIPAddress": "AWS Internal",
    "userAgent": "AWS Internal",
    "requestParameters": null,
    "responseElements": null,
    "eventID": "a24b3967-ddad-417f-9b22-2332b918db06",
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