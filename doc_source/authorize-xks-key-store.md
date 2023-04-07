# Controlling access to your external key store<a name="authorize-xks-key-store"></a>

All AWS KMS access control features — [key policies](key-policies.md), [IAM policies](iam-policies.md), and [grants](grants.md) — that you use with standard KMS keys, work the same way for KMS keys in an external key store\. You can use IAM policies to control access to the API operations that create and manage external key stores\. You use IAM policies and key policies to control access to the AWS KMS keys in your external key store\. You can also use [service control policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html) for your AWS organization and [VPC endpoint policies](kms-vpc-endpoint.md#vpce-policy) to control access to KMS keys in your external key store\.

We recommend that you provide users and roles only the permissions that they require for the tasks that they are likely to perform\.

**Topics**
+ [Authorizing external key store managers](#authorize-xks-managers)
+ [Authorizing users of KMS keys in external key stores](#authorize-xks-users)
+ [Authorizing AWS KMS to communicate with your external key store proxy](#allowlist-kms-xks)
+ [External key store proxy authorization \(optional\)](#xks-proxy-authorization)
+ [mTLS authentication \(optional\)](#xks-mtls)

## Authorizing external key store managers<a name="authorize-xks-managers"></a>

Principals who create and manage an external key store need permissions to the custom key store operations\. The following list describes the minimum permissions required for external key store managers\. Because a custom key store is not an AWS resource, you cannot provide permission to an external key store for principals in other AWS accounts\.
+ `kms:CreateCustomKeyStore`
+ `kms:DescribeCustomKeyStores`
+ `kms:ConnectCustomKeyStore`
+ `kms:DisconnectCustomKeyStore`
+ `kms:UpdateCustomKeyStore`
+ `kms:DeleteCustomKeyStore`

Principals who create an external key store need permission to create and configure the external key store components\. Principals can create external key stores only in their own accounts\. To create an external key store with [VPC endpoint service connectivity](plan-xks-keystore.md#xks-vpc-connectivity), principals must have permission to create the following components:
+ An Amazon VPC
+ Public and private subnets
+ A network load balancer and target group
+ An Amazon VPC endpoint service

For details, see [Identity and access management for Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/security-iam.html), [Identity and access management for VPC endpoints and VPC endpoint services](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-iam.html) and [Elastic Load Balancing API permissions](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/elb-api-permissions.html)\.

## Authorizing users of KMS keys in external key stores<a name="authorize-xks-users"></a>

Principals who create and manage AWS KMS keys in your external key store require [the same permissions](create-keys.md#create-key-permissions) as those who create and manage any KMS key in AWS KMS\. The [default key policy](key-policy-default.md) for KMS key in an external key store is identical to the default key policy for KMS keys in AWS KMS\. [Attribute\-based access control](abac.md) \(ABAC\), which uses tags and aliases to control access to KMS keys, is also effective on KMS keys in an external key stores\.

Principals who use the KMS keys in your custom key store for [cryptographic operations](use-cmk-keystore.md) need permission to perform the cryptographic operation with the KMS key, such as [kms:Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)\. You can provide these permissions in an IAM or key policy\. But, they do not need any additional permissions to use a KMS key in a custom key store\.

To set a permission that applies only to KMS keys in an external key store, use the [`kms:KeyOrigin`](conditions-kms.md#conditions-kms-key-origin) policy condition with a value of `EXTERNAL_KEY_STORE`\. You can use this condition to limit the [kms:CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) permission or any permission that is specific to a KMS key resource\. For example, the following IAM policy allows the identity to which it is attached to call the specified operations on all KMS keys in the account, provided that the KMS keys are in an external key store\. Notice that you can limit the permission to KMS keys in an external key store, and KMS keys in an AWS account, but not to any particular external key store in the account\.

```
{
  "Sid": "AllowKeysInExternalKeyStores",
  "Effect": "Allow",
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:ReEncrypt*",
    "kms:GenerateDataKey*",
    "kms:DescribeKey"
  ],
  "Resource": "arn:aws:kms:us-west-2:111122223333:key/*",
  "Condition": {
    "StringEquals": {
      "kms:KeyOrigin": "EXTERNAL_KEY_STORE"
    }
  }
}
```

## Authorizing AWS KMS to communicate with your external key store proxy<a name="allowlist-kms-xks"></a>

AWS KMS communicates with your external key manager only through the [external key store proxy](keystore-external.md#concept-xks-proxy) that you provide\. AWS KMS authenticates to your proxy by signing its requests using the [Signature Version 4 \(SigV4\) process](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) with the [external key store proxy authentication credential](keystore-external.md#concept-xks-credential) that you specify\. If you are using [public endpoint connectivity](plan-xks-keystore.md#xks-connectivity-public-endpoint) for your external key store proxy, AWS KMS does not require any additional permissions\.

However, if you are using [VPC endpoint service connectivity](plan-xks-keystore.md#xks-vpc-connectivity), you must give AWS KMS permission to create an interface endpoint to your Amazon VPC endpoint service\. This permission is required regardless of whether the external key store proxy is in your VPC or the external key store proxy is located elsewhere, but uses the VPC endpoint service to communicate with AWS KMS\.

To allow AWS KMS to create an interface endpoint, use the [Amazon VPC console](https://docs.aws.amazon.com/vpc/latest/privatelink/configure-endpoint-service.html#add-remove-permissions) or the [ModifyVpcEndpointServicePermissions](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_ModifyVpcEndpointServicePermissions.html) operation\. Allow permissions for the following principal: `cks.kms.<region>.amazonaws.com`\.

For example, the following AWS CLI command allows AWS KMS to connect to the specified VPC endpoint service in the US West \(Oregon\) \(us\-west\-2\) Region\. Before using this command, replace the Amazon VPC service ID and AWS Region with valid values for your configuration\.

```
modify-vpc-endpoint-service-permissions
--service-id vpce-svc-12abc34567def0987
--add-allowed-principals '["cks.kms.us-west-2.amazonaws.com"]'
```

To remove this permission, use the [Amazon VPC console](https://docs.aws.amazon.com/vpc/latest/privatelink/configure-endpoint-service.html#add-remove-permissions) or the [ModifyVpcEndpointServicePermissions](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_ModifyVpcEndpointServicePermissions.html) with the `RemoveAllowedPrincipals` parameter\.

## External key store proxy authorization \(optional\)<a name="xks-proxy-authorization"></a>

Some external key store proxies implement authorization requirements for the use of its external keys\. An external key store proxy is permitted, but not required, to design and implement an authorization scheme that allows particular users to request particular operations only under certain conditions\. For example, a proxy might be configured to allow user A to encrypt with a particular external key, but not to decrypt with it\.

Proxy authorization is independent of the [SigV4\-based proxy authentication](keystore-external.md#concept-xks-credential) that AWS KMS requires for all external key store proxies\. It is also independent of the key policies, IAM policies, and grants that authorize access to operations affecting the external key store or its KMS keys\.

To enable authorization by the external key store proxy, AWS KMS includes metadata in each [proxy API request](keystore-external.md#concept-proxy-apis), including the caller, the KMS key, the AWS KMS operation, the AWS service \(if any\)\. The request metadata for version 1 \(v1\) of the external key proxy API is as follows\.

```
"requestMetadata": {
    "awsPrincipalArn": string,
    "awsSourceVpc": string, // optional
    "awsSourceVpce": string, // optional
    "kmsKeyArn": string,
    "kmsOperation": string,
    "kmsRequestId": string,
    "kmsViaService": string // optional
}
```

For example, you might configure your proxy to allow requests from a particular principal \(`awsPrincipalArn`\), but only when the request is made on the principal's behalf by a particular AWS service \(`kmsViaService`\)\.

If proxy authorization fails, the related AWS KMS operation fails with a message that explains the error\. For details , see [Proxy authorization issues](xks-troubleshooting.md#fix-xks-authorization)\.

## mTLS authentication \(optional\)<a name="xks-mtls"></a>

To enable your external key store proxy to authenticate requests from AWS KMS, AWS KMS signs all requests to your external key store proxy with the Signature V4 \(SigV4\) [proxy authentication credential](keystore-external.md#concept-xks-credential) for your external key store\.

To provide additional assurance that your external key store proxy responds only to AWS KMS requests, some external key proxies support *mutual Transport Layer Security* \(mTLS\), in which both parties to a transaction use certificates to authenticate to each other\. mTLS adds client\-side authentication — where the external key store proxy server authenticates the AWS KMS client — to the server\-side authentication that standard TLS provides\. In the rare case that your proxy authentication credential is compromised, mTLS prevents a third party from making successful API requests to the external key store proxy\.

To implement mTLS, configure your external key store proxy to accept only client\-side TLS certificates with the following properties:
+ The subject common name on the TLS certificate must be `cks.kms.<Region>.amazonaws.com`, for example, `cks.kms.eu-west-3.amazonaws.com`\.
+ The certificate must be chained to a certificate authority associated with [Amazon Trust Services](https://www.amazontrust.com/repository/)\.
