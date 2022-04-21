# Viewing HMAC KMS keys<a name="hmac-view"></a>

You can view HMAC KMS keys in the AWS KMS console or by using the [https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) API\. You can monitor the use of your HMAC KMS keys in [AWS CloudTrail logs](logging-using-cloudtrail.md) and in [Amazon CloudWatch](monitoring-cloudwatch.md)\. For basic instructions on viewing KMS keys, see [Viewing keys](viewing-keys.md)\.

You can distinguish HMAC KMS keys from other types of KMS keys by their key spec, which begins with `HMAC`, or their key usage, which is always **Generate and verify MAC** \(`GENERATE_VERIFY_MAC`\)\.

HMAC KMS keys are included in the table on the **Customer managed keys** page of the AWS KMS console\. However, you cannot [sort or filter](viewing-keys-console.md#viewing-console-filter) KMS keys by key spec or key usage\. To make it easier to find your HMAC keys, assign them a distinctive alias or tag\. Then you can sort or filter by the alias or tag\.

On the [key details page](viewing-keys-console.md#viewing-details-navigate) for a HMAC KMS key, you can find its configuration details on the **Cryptographic configuration** tab\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/hmac-crypto-config.png)