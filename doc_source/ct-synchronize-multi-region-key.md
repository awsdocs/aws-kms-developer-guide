# SynchronizeMultiRegionKey<a name="ct-synchronize-multi-region-key"></a>

The following example shows an AWS CloudTrail log entry generated when AWS KMS synchronizes a [multi\-Region key](multi-region-keys-overview.md)\. Synchronizing involves cross\-Region calls to copy the [shared properties](multi-region-keys-overview.md#mrk-sync-properties) of a multi\-Region primary key to its replica keys\. AWS KMS synchronizes multi\-Region keys periodically to assure that all related multi\-Region keys have the same key material\.

The `resources` element of the CloudTrail log entry includes the key ARN of the multi\-Region primary key, including its AWS Region\. The related multi\-Region replica keys and their Regions are not listed in this log entry\.

```
{
    "eventVersion": "1.08",
    "userIdentity": {
        "accountId": "111122223333",
        "invokedBy": "AWS Internal"
    },
    "eventTime": "2020-11-18T02:04:37Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "SynchronizeMultiRegionKey",
    "awsRegion": "us-west-2",
    "sourceIPAddress": "AWS Internal",
    "userAgent": "AWS Internal",
    "requestParameters": null,
    "responseElements": null,
    "requestID": "12345681-de97-42e9-bed0-b02ae1abd8dc",
    "eventID": "abcdec99-2b5c-4670-9521-ddb8f031e146",
    "readOnly": false,
    "resources": [
        {
            "accountId": "111122223333",
            "type": "AWS::KMS::Key",
            "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
        }
    ],
    "eventType": "AwsApiCall",
    "managementEvent": true,
    "recipientAccountId": "111122223333",
    "eventCategory": "Management"
}
```