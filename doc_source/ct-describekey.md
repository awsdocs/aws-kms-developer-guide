# DescribeKey<a name="ct-describekey"></a>

The following example shows a log file that records multiple calls to [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html)\. These calls were the result of [viewing keys](viewing-keys.md) in the AWS KMS management console\.

```
{
  "Records": [
    {
      "eventVersion": "1.02",
      "userIdentity": {
        "type": "IAMUser",
        "principalId": "EX_PRINCIPAL_ID",
        "arn": "arn:aws:iam::123456789012:user/Alice",
        "accountId": "123456789012",
        "accessKeyId": "EXAMPLE_KEY_ID",
        "userName": "Alice",
        "sessionContext": {
          "attributes": {
            "mfaAuthenticated": "false",
            "creationDate": "2014-11-05T20:51:21Z"
          }
        },
        "invokedBy": "signin.amazonaws.com"
      },
      "eventTime": "2014-11-05T20:51:34Z",
      "eventSource": "kms.amazonaws.com",
      "eventName": "DescribeKey",
      "awsRegion": "us-east-1",
      "sourceIPAddress": "192.0.2.0",
      "userAgent": "signin.amazonaws.com",
      "requestParameters": {
        "keyId": "30a9a1e7-2a84-459d-9c61-04cbeaebab95"
      },
      "responseElements": null,
      "requestID": "874d4823-652d-11e4-9a87-01af2a1ddecb",
      "eventID": "f715da9b-c52c-4824-99ae-88aa1bb58ae4",
      "readOnly": true,
      "resources": [
        {
          "ARN": "arn:aws:kms:us-east-1:123456789012:key/30a9a1e7-2a84-459d-9c61-04cbeaebab95",
          "accountId": "123456789012"
        }
      ],
      "eventType": "AwsApiCall",
      "recipientAccountId": "123456789012"
    },
    {
      "eventVersion": "1.02",
      "userIdentity": {
        "type": "IAMUser",
        "principalId": "EX_PRINCIPAL_ID",
        "arn": "arn:aws:iam::123456789012:user/Alice",
        "accountId": "123456789012",
        "accessKeyId": "EXAMPLE_KEY_ID",
        "userName": "Alice",
        "sessionContext": {
          "attributes": {
            "mfaAuthenticated": "false",
            "creationDate": "2014-11-05T20:51:21Z"
          }
        },
        "invokedBy": "signin.amazonaws.com"
      },
      "eventTime": "2014-11-05T20:51:55Z",
      "eventSource": "kms.amazonaws.com",
      "eventName": "DescribeKey",
      "awsRegion": "us-east-1",
      "sourceIPAddress": "192.0.2.0",
      "userAgent": "signin.amazonaws.com",
      "requestParameters": {
        "keyId": "e7b6d35a-b551-4c8f-b51a-0460ebc04565"
      },
      "responseElements": null,
      "requestID": "9400c720-652d-11e4-9a87-01af2a1ddecb",
      "eventID": "939fcefb-dc14-4a52-b918-73045fe97af3",
      "readOnly": true,
      "resources": [
        {
          "ARN": "arn:aws:kms:us-east-1:123456789012:key/e7b6d35a-b551-4c8f-b51a-0460ebc04565",
          "accountId": "123456789012"
        }
      ],
      "eventType": "AwsApiCall",
      "recipientAccountId": "123456789012"
    }
  ]
}
```