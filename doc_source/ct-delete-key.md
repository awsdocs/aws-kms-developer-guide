# DeleteKey<a name="ct-delete-key"></a>

The following example shows an AWS CloudTrail log entry generated when a customer master key \(CMK\) is deleted\. To delete a CMK, you use the [ScheduleKeyDeletion](https://docs.aws.amazon.com/kms/latest/APIReference/API_ScheduleKeyDeletion.html) operation\. After the specified waiting period expires, AWS KMS deletes the key\. AWS KMS records an entry like the following one in your CloudTrail log to record that event\. 

For an example of the CloudTrail log entry for the `ScheduleKeyDeletion` operation, see [ScheduleKeyDeletion](ct-schedule-key-deletion.md)\. For information about deleting CMKs, see [Deleting customer master keys](deleting-keys.md)\.

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "accountId": "111122223333",
        "invokedBy": "AWS Internal"
    },
    "eventTime": "2020-07-31T00:07:00Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "DeleteKey",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "AWS Internal",
    "userAgent": "AWS Internal",
    "requestParameters": null,
    "responseElements": null,
    "eventID": "b25f9cda-74e1-4458-847b-4972a0bf9668",
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