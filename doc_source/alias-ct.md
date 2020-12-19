# Finding aliases in AWS CloudTrail logs<a name="alias-ct"></a>

When you use an alias to represent a customer master key \(CMK\) in an AWS KMS API operation, the alias and the key ARN of the CMK are recorded in the AWS CloudTrail log entry for the event\. The alias appears in the `requestParameters` field\. The key ARN appears in the `resources` field\. This is true even when an AWS service uses an AWS managed CMK in your account\. 

For example, the following [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) request uses the `project-key` alias to represent a CMK\.

```
$ aws kms generate-data-key --key-id alias/project-key --key-spec AES_256
```

When this request is recorded in the CloudTrail log, the log entry includes both the alias and the key ARN of the actual CMK that was used\. 

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "ABCDE",
        "arn": "arn:aws:iam::111122223333:role/ProjectDev",
        "accountId": "111122223333",
        "accessKeyId": "FFHIJ",
        "userName": "example-dev"
    },
    "eventTime": "2020-06-29T23:36:41Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "GenerateDataKey",
    "awsRegion": "us-west-2",
    "sourceIPAddress": "205.205.123.000",
    "userAgent": "aws-cli/1.18.89 Python/3.6.10 Linux/4.9.217-0.1.ac.205.84.332.metal1.x86_64 botocore/1.17.12",
    "requestParameters": {
        "keyId": "alias/project-key",
        "keySpec": "AES_256"
    },
    "responseElements": null,
    "requestID": "d93f57f5-d4c5-4bab-8139-5a1f7824a363",
    "eventID": "d63001e2-dbc6-4aae-90cb-e5370aca7125",
    "readOnly": true,
    "resources": [
        {
            "accountId": "111122223333",
            "type": "AWS::KMS::Key",
            "ARN": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
        }
    ],
    "eventType": "AwsApiCall",
    "recipientAccountId": "111122223333"
}
```

For details about logging AWS KMS operations in CloudTrail logs, see [Logging AWS KMS API calls with AWS CloudTrail](logging-using-cloudtrail.md)\.