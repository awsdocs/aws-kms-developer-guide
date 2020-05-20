# Request an AWS KMS Quota Increase<a name="increase-quota"></a>

AWS KMS resource quotas and request quotas are adjustable, except for the [custom key store quota](requests-per-second.md#rps-key-stores)\.

If you need to exceed a quota, you can request a quota increase in Service Quotas\. Use the [Service Quotas console](https://console.aws.amazon.com/servicequotas) or the [RequestServiceQuotaIncrease](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/API_RequestServiceQuotaIncrease.html) operation\. For details, see [Requesting a quota increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html) in the *Service Quotas User Guide*\. If Service Quotas for AWS KMS are not available in the AWS Region, please visit the [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case\. 

## Using the Service Quotas console<a name="quota-increase-console"></a>

To request an increase for an AWS KMS quota, you can use the [Service Quotas console](https://console.aws.amazon.com/servicequotas)\. For instructions, see [Requesting a quota increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-quota-increase.html) in the *Service Quotas User Guide*\. 

1. For service name, choose **AWS Key Management Service \(AWS KMS\)**\.

1. Choose the quota name of the quota you want to increase\. Use the detailed information about the quota to confirm that you have chosen the quota you want to increase\. 

   You can search for the quota name in the Service Quotas console\. There are several pages of AWS KMS quotas\. You can also find the quota names and descriptions of AWS KMS quotas in the [resource quota](resource-limits.md) and [request quota](requests-per-second.md) tables\.

   For example, to request an increase to the quota for cryptographic operations on a symmetric CMK, choose **Cryptographic operations \(symmetric\) request rate**\.

1. Choose **Request quota increase**\.

## Using the Service Quotas API<a name="quota-increase-api"></a>

To request an increase in an AWS KMS quota, you can use the [Service Quotas API](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/)\. The `RequestServiceQuotaIncrease` operation, which submits the request, requires the quota code for the quota\. So begin by getting the quota code\.

1. Find the quota name of the quota you want to increase\. You can find the quota names and descriptions of AWS KMS quotas in the [resource quota](resource-limits.md) and [request quota](requests-per-second.md) tables\. 

1. To get the quota code for an AWS KMS quota, use the [ListServiceQuotas](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/API_ListServiceQuotas.html) operation\.

   Set `ServiceCode` to `kms`\.

   The response includes the `QuotaName` and `QuotaCode` for each quota\. 

   For example, to get only the quota information for the `Cryptographic operations (RSA) request rate` quota, use a command like the following one\. It uses the `query` parameter in the AWS Command Line Interface \(AWS CLI\) to get only the quota with the specified quota name\.

   ```
   $ aws service-quotas list-service-quotas \
           --service-code kms \
           --query 'Quotas[?QuotaName==`Cryptographic operations (RSA) request rate`]'
   
   {
       "Quotas": [
           {
               "ServiceCode": "kms",
               "ServiceName": "AWS Key Management Service (AWS KMS)",
               "QuotaArn": "arn:aws:servicequotas:us-east-2:111122223333:kms/L-2AC98190",
               "QuotaCode": "L-2AC98190",
               "QuotaName": "Cryptographic operations (RSA) request rate",
               "Value": 500,
               "Unit": "None",
               "Adjustable": true,
               "GlobalQuota": false
           }
       ]
   }
   ```

1. To request an increase for an AWS KMS quota, use the [RequestServiceQuotaIncrease](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/API_RequestServiceQuotaIncrease.html) operation\. To identify the quota, use the quota code\.

   For example, the following command requests an increase in the `Cryptographic operations (RSA) request rate` quota to `700` requests per second\. It uses the required quota code, `L-2AC98190`, to identify the quota\.

   If the command completes successfully, the `Status` field displays the current status of the request\. To get the updated status of the request, use the [GetRequestedServiceQuotaChange](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/API_GetRequestedServiceQuotaChange.html), [ListRequestedServiceQuotaChangeHistory](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/API_ListRequestedServiceQuotaChangeHistory.html) or [ListRequestedServiceQuotaChangeHistoryByQuota](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/API_ListRequestedServiceQuotaChangeHistoryByQuota.html) operations\.

   ```
   $ aws service-quotas request-service-quota-increase \
           --service-code kms \
           --quota-code L-2AC98190 \
           --desired-value 700
   
   {
       "RequestedQuota": {
           "Id": "a12345",
           "ServiceCode": "kms",
           "ServiceName": "AWS Key Management Service (AWS KMS)",
           "QuotaCode": "L-2AC98190",
           "QuotaName": "Cryptographic operations (RSA) request rate",
           "DesiredValue": 700,
           "Status": "PENDING",
           "Created": 1580446904.067,
           "Requester": "{\"accountId\":\"111122223333\",\"callerArn\":\"arn:aws:iam::111122223333:root\"}",
           "QuotaArn": "arn:aws:servicequotas:us-east-2:111122223333:kms/L-2AC98190",
           "GlobalQuota": false,
           "Unit": "None"
       }
   }
   ```