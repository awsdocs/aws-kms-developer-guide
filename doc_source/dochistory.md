# Document History<a name="dochistory"></a>

This topic describes significant updates to the *AWS Key Management Service Developer Guide*\.

**Topics**
+ [Recent Updates](#recent-updates)
+ [Earlier Updates](#earlier-updates)

## Recent Updates<a name="recent-updates"></a>

The following table describes significant changes to this documentation since January 2018\. In addition to major changes listed here, we also update the documentation frequently to improve the descriptions and examples, and to address the feedback that you send to us\. To be notified about significant changes, use the link in the upper right corner to subscribe to the RSS feed\.

**Current API version**: 2014\-11\-01

| Change | Description | Date | 
| --- |--- |--- |
| [Updated feature](https://docs.aws.amazon.com/kms/latest/developerguide/key-policy-viewing.html) | You can view the key policy of AWS managed CMKs in the AWS KMS console\. This feature used to be limited to customer managed CMKs\. | November 15, 2019 | 
| [New feature](https://docs.aws.amazon.com/kms/latest/developerguide/pqtls.html) | Explains how to use [hybrid post\-quantum key exchange](https://docs.aws.amazon.com/kms/latest/developerguide/pqtls.html) algorithms in TLS for your calls to AWS KMS\. | November 4, 2019 | 
| [Limit change](https://docs.aws.amazon.com/kms/latest/developerguide/limits.html) | Increased the resource limits for some APIs that manage CMKs\. | September 18, 2019 | 
| [Limit change](https://docs.aws.amazon.com/kms/latest/developerguide/limits.html) | Changed the resource limits on customer master keys \(CMKs\), aliases, and grants per CMK\. | March 27, 2019 | 
| [Limit change](https://docs.aws.amazon.com/kms/latest/developerguide/limits.html#rps-key-stores) | Changed the shared requests\-per\-second throttle limit on cryptographic operations that use customer master keys \(CMKs\) in a custom key store\. | March 7, 2019 | 
| [New feature](https://docs.aws.amazon.com/kms/latest/developerguide/custom-key-store-overview.html) | Explains how to create and manage AWS KMS [custom key stores](https://docs.aws.amazon.com/kms/latest/developerguide/custom-key-store-overview.html)\. Each key store is backed by an AWS CloudHSM cluster that you own and control\. | November 26, 2018 | 
| [New console](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html#create-keys-console) | Explains how to use the new AWS KMS console, which is independent of the IAM console\. The original console, and instructions for using it, will remain available for a brief period to give you time to familiarize yourself with the new console\. | November 7, 2018 | 
| [Limit change](https://docs.aws.amazon.com/kms/latest/developerguide/limits.html#requests-per-second) | Changed the shared [requests\-per\-second](https://docs.aws.amazon.com/kms/latest/developerguide/limits.html#requests-per-second) limit on customer master keys\. | August 21, 2018 | 
| [New content](https://docs.aws.amazon.com/kms/latest/developerguide/services-secrets-manager.html) | Explains [how AWS Secrets Manager uses AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/services-secrets-manager.html) customer master keys to encrypt the secret value in a secret\. | July 13, 2018 | 
| [New content](https://docs.aws.amazon.com/kms/latest/developerguide/services-dynamodb.html) | Explains [how DynamoDB uses AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/services-dynamodb.html) customer master keys to support its server\-side encryption option\. | May 23, 2018 | 
| [New feature](https://docs.aws.amazon.com/kms/latest/developerguide/kms-vpc-endpoint.html) | Explains how to [use a private endpoint in your VPC](https://docs.aws.amazon.com/kms/latest/developerguide/kms-vpc-endpoint.html) to connect directly to AWS KMS, instead of connecting over the internet\. | January 22, 2018 | 

## Earlier Updates<a name="earlier-updates"></a>

The following table describes the important changes to the AWS Key Management Service Developer Guide prior to 2018\.


| Change | Description | Date | 
| --- | --- | --- | 
| New content | Added documentation about [Tagging Keys](tagging-keys.md)\. | February 15, 2017 | 
| New content | Added documentation about [Monitoring Customer Master Keys](monitoring-overview.md) and [Monitoring with Amazon CloudWatch](monitoring-cloudwatch.md)\. | August 31, 2016 | 
| New content | Added documentation about [Importing Key Material](importing-keys.md)\. | August 11, 2016 | 
| New content | Added the following documentation: [Overview of Managing Access](control-access-overview.md), [Using IAM Policies](iam-policies.md), [AWS KMS API Permissions Reference](kms-api-permissions-reference.md), and [Using Policy Conditions](policy-conditions.md)\. | July 5, 2016 | 
| Update | Updated portions of the documentation in the [Authentication and Access Control](control-access.md) chapter\. | July 5, 2016 | 
| Update | Updated the [Limits](limits.md) page to reflect new default limits\. | May 31, 2016 | 
| Update | Updated the [Limits](limits.md) page to reflect new default limits, and updated the [Grant Tokens](concepts.md#grant_token) documentation to improve clarity and accuracy\. | April 11, 2016 | 
| New content | Added documentation about [Allowing Multiple IAM Users to Access a CMK](key-policy-modifying.md#key-policy-modifying-multiple-iam-users) and [Using the IP Address Condition](policy-conditions.md#conditions-aws-ip-address)\. | February 17, 2016 | 
| Update | Updated the [Using Key Policies in AWS KMS](key-policies.md) and [Changing a Key Policy](key-policy-modifying.md) pages to improve clarity and accuracy\. | February 17, 2016 | 
| Update | Updated the [Getting Started](getting-started.md) topic pages to improve clarity\. | January 5, 2016 | 
| New content | Added documentation about [How AWS CloudTrail Uses AWS KMS](services-cloudtrail.md)\. | November 18, 2015 | 
| New content | Added instructions for [Changing a Key Policy](key-policy-modifying.md)\. | November 18, 2015 | 
| Update | Updated the documentation about [How Amazon Relational Database Service \(Amazon RDS\) Uses AWS KMS](services-rds.md)\. | November 18, 2015 | 
| New content | Added documentation about [How Amazon WorkSpaces Uses AWS KMS](services-workspaces.md)\. | November 6, 2015 | 
| Update | Updated the [Using Key Policies in AWS KMS](key-policies.md) page to improve clarity\. | October 22, 2015 | 
| New content | Added documentation about [Deleting Customer Master Keys](deleting-keys.md), including supporting documentation about [Creating an Amazon CloudWatch Alarm](deleting-keys-creating-cloudwatch-alarm.md) and [Determining Past Usage of a Customer Master Key](deleting-keys-determining-usage.md)\. | October 15, 2015 | 
| New content | Added documentation about [Determining Access to an AWS KMS Customer Master Key](determining-access.md)\. | October 15, 2015 | 
| New content | Added documentation about [How Key State Affects Use of a Customer Master Key](key-state.md)\. | October 15, 2015 | 
| New content | Added documentation about [How Amazon Simple Email Service \(Amazon SES\) Uses AWS KMS](services-ses.md)\. | October 1, 2015 | 
| Update | Updated the [Limits](limits.md) page to explain the new requests per second limits\. | August 31, 2015 | 
| New content | Added information about the charges for using AWS KMS\. See [AWS KMS Pricing](overview.md#pricing)\. | August 14, 2015 | 
| New content | Added requests per second to the AWS KMS [Limits](limits.md)\. | June 11, 2015 | 
| New content | Added a new Java code sample demonstrating use of the [https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html](https://docs.aws.amazon.com/kms/latest/APIReference/API_UpdateAlias.html) API\. See [Updating an Alias](programming-aliases.md#update-alias)\. | June 1, 2015 | 
| Update | Moved the [AWS Key Management Service regions table](https://docs.aws.amazon.com/general/latest/gr/rande.html#kms_region) to the AWS General Reference\. | May 29, 2015 | 
| New content | Added documentation about [How Amazon EMR Uses AWS KMS](services-emr.md)\. | January 28, 2015 | 
| New content | Added documentation about [How Amazon WorkMail Uses AWS KMS](services-wm.md)\. | January 28, 2015 | 
| New content | Added documentation about [How Amazon Relational Database Service \(Amazon RDS\) Uses AWS KMS](services-rds.md)\. | January 6, 2015 | 
| New content | Added documentation about [How Amazon Elastic Transcoder Uses AWS KMS](services-et.md)\. | November 24, 2014 | 
| New guide | Introduced the AWS Key Management Service Developer Guide\. | November 12, 2014 | 