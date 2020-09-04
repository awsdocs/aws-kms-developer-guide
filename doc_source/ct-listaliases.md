# ListAliases<a name="ct-listaliases"></a>

The following example shows an AWS CloudTrail log entry for the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. Because this operation doesn't use any particular alias or customer master key, the `resources` field is empty\. For information about viewing aliases in AWS KMS, see [ Viewing aliases  Aliases make it easy to recognize CMKs in the AWS KMS console\. But the console displays only one alias for each CMK\. To see all aliases for a CMK, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\. The [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation, which returns detailed information about a CMK, doesn't include aliases\.  Viewing aliases \(console\)  The **Customer managed keys** and **AWS managed keys** pages in the AWS KMS console display the alias associated with each CMK\. You can also search, sort, and filter CMKs based on their alias\. The following image of the AWS KMS console shows the aliases on the **Customer managed keys** page of an example account\. As shown in the image, some CMKs don't have an alias\.  

![\[Aliases in the Customer managed keys page of the AWS KMS console\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/alias-console-sm.png) The details page for each CMK displays one of the aliases for the CMK\. To open the details page for a CMK, in the CMK table, choose its alias or key ID\. This page displays one alias name but doesn't display the alias ARN\. For help with finding the alias ARN, see [Finding the alias name and alias ARN](find-cmk-alias.md)\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/alias-console-key-detail.png) You can use the alias to recognize an AWS managed CMK, as shown in this example **AWS managed keys** page\. The aliases for AWS managed CMKs always have the format: `aws/<service-name>`\. For example, the alias for the AWS managed CMK for Amazon DynamoDB is `aws/dynamodb`\.  

![\[Aliases in the AWS managed keys page of the AWS KMS console\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/alias-console-aws-managed-sm.png) The AWS KMS console requires that you specify an alias when you create a CMK\. It also displays the alias name \(if any\) that is associated with the CMK\. But you cannot use the console to create additional aliases, change the CMK associated with an alias, or delete an alias\. Those tasks can be done only with the AWS KMS API\. Also, the AWS KMS console displays only one alias for each CMK and it does not display the alias ARN\. To view all aliases for a particular CMK or to view the alias ARN for any alias in the account and Region, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation\.   Viewing aliases \(AWS KMS API\)  The [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) operation returns the alias name and alias ARN of aliases in the account and Region\. The output includes aliases for AWS managed CMKs and for customer managed CMKs\. The aliases for AWS managed CMKs have the format `aws/<service-name>`, such as `aws/dynamodb`\. The response might also include aliases that have no `TargetKeyId` field\. These are predefined aliases that AWS has created but has not yet associated with a CMK\. Aliases that AWS creates in your account, including predefined aliases, do not count against your [AWS KMS aliases quota](resource-limits.md#aliases-limit)\. 

```
$ aws kms list-aliases
{
    "Aliases": [
        {
            "AliasName": "alias/access-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/access-key",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321"
        },        
        {
            "AliasName": "alias/ECC-P521-Sign",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/ECC-P521-Sign",
            "TargetKeyId": "1234abcd-12ab-34cd-56ef-1234567890ab"
        },
        {
            "AliasName": "alias/ImportedKey",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/ImportedKey",
            "TargetKeyId": "1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d"
        },
        {
            "AliasName": "alias/finance-project",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/finance-project",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321"
        },
        {
            "AliasName": "alias/aws/dynamodb",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/aws/dynamodb",
            "TargetKeyId": "0987ab65-43cd-21ef-09ab-87654321cdef"
        },
        {
            "AliasName": "alias/aws/ebs",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/aws/ebs",
            "TargetKeyId": "abcd1234-09fe-ef90-09fe-ab0987654321"        
        }
    ]
}
``` To get all aliases that are associated with a particular CMK, use the optional `KeyId` parameter of the `ListAliases` operation\. The `KeyId` parameter takes the [key ID](concepts.md#key-id-key-id) or [key ARN](concepts.md#key-id-key-ARN) of the CMK\.  This example gets all aliases associated with the `0987dcba-09fe-87dc-65ba-ab0987654321` CMK\. 

```
$ aws kms list-aliases --key-id 0987dcba-09fe-87dc-65ba-ab0987654321
{
    "Aliases": [
        {
            "AliasName": "alias/access-key",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/access-key",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321"
        },
        {
            "AliasName": "alias/finance-project",
            "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/finance-project",
            "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321"
        }
    ]
}
``` The `KeyId` parameter doesn't take wildcard characters, but you can use the features of your programming language to filter the response\.  For example, the following AWS CLI command gets only the aliases for AWS managed CMKs\. 

```
$ aws kms list-aliases --query 'Aliases[?starts_with(AliasName, `alias/aws/`)]'
``` The following command gets only the `access-key` alias\. The alias name is case\-sensitive\. 

```
$ aws kms list-aliases --query 'Aliases[?AliasName==`alias/access-key`]'
[
    {
        "AliasName": "alias/access-key",
        "AliasArn": "arn:aws:kms:us-west-2:111122223333:alias/access-key",
        "TargetKeyId": "0987dcba-09fe-87dc-65ba-ab0987654321"
    }
]
```  ](kms-alias.md#alias-view)\.

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
        "eventTime": "2014-11-04T00:51:45Z",
        "eventSource": "kms.amazonaws.com",
        "eventName": "ListAliases",
        "awsRegion": "us-east-1",
        "sourceIPAddress": "192.0.2.0",
        "userAgent": "AWS Internal",
        "requestParameters": {
            "limit": 5,
            "marker": "eyJiIjoiYWxpYXMvZTU0Y2MxOTMtYTMwNC00YzEwLTliZWItYTJjZjA3NjA2OTJhIiwiYSI6ImFsaWFzL2U1NGNjMTkzLWEzMDQtNGMxMC05YmViLWEyY2YwNzYwNjkyYSJ9"
        },
        "responseElements": null,
        "requestID": "bfe6c190-63bc-11e4-bc2b-4198b6150d5c",
        "eventID": "a27dda7b-76f1-4ac3-8b40-42dfba77bcd6",
        "readOnly": true,
        "resources": [],
        "eventType": "AwsApiCall",
        "recipientAccountId": "111122223333"
    }
  ]
}
```