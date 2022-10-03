# Monitoring with Amazon EventBridge<a name="kms-events"></a>

You can use Amazon EventBridge \(formerly Amazon CloudWatch Events\) to alert you to the following important events in the lifecycle of your KMS keys\.
+ The key material in a KMS key was automatically rotated\.
+ The imported key material in a KMS key expired\.
+ A KMS key that had been scheduled for deletion was deleted\.

AWS KMS integrates with Amazon EventBridge to notify you of important events that affect your KMS keys\. Each event is represented in [JSON \(JavaScript Object Notation\)](http://json.org) and includes the event name, the date and time when the event occurred, and the affected\. You can collect these events and establish rules that route them to one or more *targets* such as AWS Lambda functions, Amazon SNS topics, Amazon SQS queues, streams in Amazon Kinesis Data Streams, or built\-in targets\.

For more information about using EventBridge with other kinds of events, including those emitted by AWS CloudTrail when it records a read/write API request, see the [Amazon EventBridge User Guide](https://docs.aws.amazon.com/eventbridge/latest/userguide/)\.

The following topics describe the EventBridge events that AWS KMS generates\.

## KMS CMK Rotation<a name="kms-events-rotation"></a>

AWS KMS supports [automatic rotation](rotate-keys.md) of the key material in symmetric encryption KMS keys\. Annual key material rotation is optional for [customer managed keys](concepts.md#customer-cmk)\. The key material for [AWS managed keys](concepts.md#aws-managed-cmk) is automatically rotated every year\.

Whenever AWS KMS rotates key material, it sends a `KMS CMK Rotation` event to EventBridge\. AWS KMS generates this event on a best\-effort basis\.

The following is an example of this event\.

```
{
  "version": "0",
  "id": "6a7e8feb-b491-4cf7-a9f1-bf3703467718",
  "detail-type": "KMS CMK Rotation",
  "source": "aws.kms",
  "account": "111122223333",
  "time": "2022-08-10T16:37:50Z",
  "region": "us-west-2",
  "resources": [
    "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
  ],
  "detail": {
    "key-id": "1234abcd-12ab-34cd-56ef-1234567890ab"
  }
}
```

## KMS Imported Key Material Expiration<a name="kms-events-expiration"></a>

When you [import key material into a KMS key](importing-keys.md), you can optionally specify a time at which the key material expires\. When the key material expires, AWS KMS deletes the key material and sends a corresponding `KMS Imported Key Material Expiration` event to EventBridge\. AWS KMS generates this event on a best\-effort basis\.

The following is an example of this event\.

```
{
  "version": "0",
  "id": "9da9af57-9253-4406-87cb-7cc400e43465",
  "detail-type": "KMS Imported Key Material Expiration",
  "source": "aws.kms",
  "account": "111122223333",
  "time": "2022-08-10T16:37:50Z",
  "region": "us-west-2",
  "resources": [
    "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
  ],
  "detail": {
    "key-id": "1234abcd-12ab-34cd-56ef-1234567890ab"
  }
}
```

## KMS CMK Deletion<a name="kms-events-deletion"></a>

When you [schedule deletion](deleting-keys.md) of a KMS key, AWS KMS enforces a waiting period before deleting the KMS key\. After the waiting period ends, AWS KMS deletes the KMS key and sends a `KMS CMK Deletion` event to EventBridge\. AWS KMS guarantees this EventBridge event\. Due to retries, it might generate multiple events within a few seconds that delete the same KMS key\.

 The following is an example of this event\.

```
{
  "version": "0",
  "id": "e9ce3425-7d22-412a-a699-e7a5fc3fbc9a",
  "detail-type": "KMS CMK Deletion",
  "source": "aws.kms",
  "account": "111122223333",
  "time": "2022-08-10T16:37:50Z",
  "region": "us-west-2",
  "resources": [
    "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
  ],
  "detail": {
    "key-id": "1234abcd-12ab-34cd-56ef-1234567890ab"
  }
}
```