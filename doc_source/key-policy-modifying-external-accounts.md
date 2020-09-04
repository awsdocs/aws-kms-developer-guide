# Allowing users in other accounts to use a CMK<a name="key-policy-modifying-external-accounts"></a>

You can allow IAM users or roles in one AWS account to use a customer master key \(CMK\) in a different AWS account\. You can add these permissions when you create the CMK or change the permissions for an existing CMK\.

**Warning**  
Be cautious about giving principals permissions to use your CMKs\. Whenever possible, follow the *least privilege* principle\. Give users access only to the CMKs they need for only the operations they require\.  
Also, be cautious about using any unfamiliar CMK, especially a CMK in a different account\. Malicious users might give you permissions to use their CMK to get information about you or your account\.   
For information about using policies to protect the resources in your account, see [Best practices for IAM policies](iam-policies.md#iam-policies-best-practices)\.

To give permission to use a CMK to users and roles in another account, you must use two different types of policies:
+ The **key policy** for the CMK must give the external account \(or users and roles in the external account\) permission to use the CMK\. The key policy is in the account that owns the CMK\.
+ **IAM policies** in the external account must delegate the key policy permissions to its users and roles\. These policies are set in the external account and give permissions to users and roles in that account\.

The key policy determines who *can* have access to the CMK\. The IAM policy determines who *does* have access to the CMK\. Neither the key policy nor the IAM policy alone is sufficient—you must change both\. 

To edit the key policy, you can use the [Policy View](key-policy-modifying.md#key-policy-modifying-how-to-console-policy-view) in the AWS Management Console or use the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) or [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operations\. For help setting the key policy when creating a CMK, see [Creating CMKs that other accounts can use](#cross-account-console)\.

For help with editing IAM policies, see [Using IAM policies with AWS KMS](iam-policies.md)\. 

For an example that shows how the key policy and IAM policies work together to allow use of a CMK in a different account, see [Example 2: User assumes role with permission to use a CMK in a different AWS account](policy-evaluation.md#example-cross-acct)\.

You can view the resulting cross\-account AWS KMS operations on the CMK in your [AWS CloudTrail logs](logging-using-cloudtrail.md)\. Operations that use CMKs in other accounts are logged in both the caller's account and the CMK owner's account\.

**Topics**
+ [Step 1: Add a key policy statement in the local account](#cross-account-key-policy)
+ [Step 2: Add IAM policies in the external account](#cross-account-iam-policy)
+ [Creating CMKs that other accounts can use](#cross-account-console)
+ [Using external CMKs with AWS services](#cross-account-service)

## Step 1: Add a key policy statement in the local account<a name="cross-account-key-policy"></a>

The key policy for a CMK is the primary determinant of who can access the CMK and which operations they can perform\. The key policy is always in the account that owns the CMK\. Unlike IAM policies, key policies do not specify a resource\. The resource is the CMK that is associated with the key policy\.

To give an external account permission to use the CMK, add a statement to the key policy that specifies the external account\. In the `Principal` element of the key policy, enter the Amazon Resource Name \(ARN\) of the external account\.

When you specify an external account in a key policy, IAM administrators in the external account can use IAM policies to delegate those permissions to any users and roles in the external account\. They can also decide which of the actions specified in the key policy the users and roles can perform\. 

For example, assume that you want to allow account 444455556666 to use a symmetric CMK in account 111122223333\. To do that, add a policy statement like the one in the following example to the key policy for the CMK in account 111122223333\. This policy statement gives the external account, 444455556666, permission to use the CMK in cryptographic operations for symmetric CMKs\. 

```
{
    "Sid": "Allow an external account to use this CMK",
    "Effect": "Allow",
    "Principal": {
        "AWS": [
            "arn:aws:iam::444455556666:root"
        ]
    },
    "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
    ],
    "Resource": "*"
}
```

Instead of giving permission to the external account, you can specify particular external users and roles in the key policy \. However, those users and roles cannot use the CMK until IAM administrators in the external account attach the proper IAM policies to their identities\. The IAM policies can give permission to all or a subset of the external users and roles that are specified in the key policy\. And they can allow all or a subset of the actions specified in the key policy\. 

Specifying identities in a key policy restricts the permissions that IAM administrators in the external account can provide\. However, it makes policy management with two accounts more complex\. For example, assume that you need to add a user or role\. You must add that identity to the key policy in the account that owns the CMK and create IAM policies in the identity's account\.

To specify particular external users or roles in a key policy, in the `Principal` element, enter the Amazon Resource Name \(ARN\) of a user or role in the external account\.

For example, the following example key policy statement allows `ExampleRole` and ExampleUser in account 444455556666 to use a CMK in account 111122223333\. This key policy statement gives the external account, 444455556666, permission to use the CMK in cryptographic operations for symmetric CMKs\. 

```
{
    "Sid": "Allow an external account to use this CMK",
    "Effect": "Allow",
    "Principal": {
        "AWS": [
            "arn:aws:iam::444455556666:role/ExampleRole",
            "arn:aws:iam::444455556666:user/ExampleUser"
        ]
    },
    "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
    ],
    "Resource": "*"
}
```

**Note**  
Do not set the Principal to an asterisk \(\*\) in any key policy statement that allows permissions\. An asterisk gives every identity in every AWS account permission to use the CMK, unless another policy statement explicitly denies it\. Users in other AWS accounts just need corresponding IAM permissions in their own accounts to use the CMK\.

You also need to decide which permissions you want to give to the external account\. For a list of permissions on CMKs, see [AWS KMS API permissions: Actions and resources reference](kms-api-permissions-reference.md)\.

You can give the external account permission to use the CMK in [cryptographic operations](concepts.md#cryptographic-operations) and use the CMK with AWS services that are integrated with AWS KMS\. To do that, use the **Key Users** section of the AWS Management Console\. For details, see [Creating CMKs that other accounts can use](#cross-account-console)\.

To specify other permissions in key policies, edit the key policy document\. For example, you might want to give users permission to decrypt but not encrypt, or permission to view the CMK but not use it\. To edit the key policy document, you can use the [Policy View](key-policy-modifying.md#key-policy-modifying-how-to-console-policy-view) in the AWS Management Console or the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) or [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operations\.

## Step 2: Add IAM policies in the external account<a name="cross-account-iam-policy"></a>

The key policy in the account that owns the CMK sets the valid range for permissions\. But, users and roles in the external account cannot use the CMK until you attach IAM policies that delegate those permissions, or use grants to manage access to the CMK\. The IAM policies are set in the external account\. 

If the key policy gives permission to the external account, you can attach IAM policies to any user or role in the account\. But if the key policy gives permission to specified users or roles, the IAM policy can only give those permissions to all or a subset of the specified users and roles\. If an IAM policy gives CMK access to other external users or roles, it has no effect\.

The key policy also limits the actions in the IAM policy\. The IAM policy can delegate all or a subset of the actions specified in the key policy\. If the IAM policy lists actions that are not specified in the key policy, those permissions are not effective\.

The following example IAM policy allows the principal to use the CMK in account 111122223333 for cryptographic operations\. To give this permission to users and roles in account `444455556666`, [attach the policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-using.html#attach-managed-policy-console) to the users or roles in account `444455556666`\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Allow Use Of CMK In Account 111122223333",
      "Effect": "Allow",
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
      ],
      "Resource": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
    }
  ]
}
```

Note the following details about this policy:
+ Unlike key policies, IAM policy statements do not contain the `Principal` element\. In IAM policies, the principal is the identity to which the policy is attached\. 
+ The `Resource` element in the IAM policy identifies the CMK that the principal can use\. To specify a CMK, add its [key ARN](concepts.md#key-id-alias-ARN) to the `Resource` element\.
+ You can specify more than one CMK in the `Resource` element\. But if you don't specify particular CMKs in the `Resource` element, you might inadvertently give access to more CMKs than you intend\.
+ To allow the external user to use the CMK with [AWS services that integrate with AWS KMS,](https://aws.amazon.com/kms/features/#AWS_Service_Integration) you might need to add permissions to the key policy or the IAM policy\. For details, see [Using external CMKs with AWS services](#cross-account-service)\.

For more information about working with IAM policies, see [Using IAM policies](iam-policies.md)\.

## Creating CMKs that other accounts can use<a name="cross-account-console"></a>

When you use the [CreateKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html) operation create a CMK, you can use its `Policy` parameter to specify a [key policy](#cross-account-key-policy) that gives an external account, or external users and roles, permission to use the CMK\. You must also add [IAM policies](#cross-account-iam-policy) in the external account that delegate these permissions to the account's users and roles, even when users and roles are specified in the key policy\. You can change the key policy at any time by using the [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) operation\.

When you create a CMK in the AWS Management Console, you also create its key policy\. When you select identities in the **Key Administrators** and **Key Users** sections, AWS KMS adds policy statements for those identities to the CMK's key policy\. 

The **Key Users** section also lets you add external accounts as key users\.

![\[The console element that adds external accounts to the key policy for a CMK.\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-external-accounts-sm.png)

When you enter the account ID of an external account, AWS KMS adds two statements to the key policy\. This action only affects the key policy\. Users and roles in the external account cannot use the CMK until you attach [IAM policies](#cross-account-iam-policy) to give them some or all of these permissions\.

The first policy statement gives the external account permission to use the CMK in cryptographic operations\. 

```
{
    "Sid": "Allow use of the key",
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::444455556666:root"
    },
    "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
    ],
    "Resource": "*"
}
```

The second policy statement allows the external account to create, view, and revoke grants on the CMK, but only when the request comes from an [AWS service that is integrated with AWS KMS](https://aws.amazon.com/kms/features/#AWS_Service_Integration)\. These permissions allow other AWS services, such as that encrypt user data to use the CMK\. 

These permissions are designed for CMKs that encrypt user data in AWS services, such as [Amazon WorkMail](services-wm.md)\. These services typically use grants to get the permissions they need to use the CMK on the user's behalf\. For details, see [Using external CMKs with AWS services](#cross-account-service)\.

```
{
    "Sid": "Allow attachment of persistent resources",
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::444455556666:root"
    },
    "Action": [
        "kms:CreateGrant",
        "kms:ListGrants",
        "kms:RevokeGrant"
    ],
    "Resource": "*",
    "Condition": {
        "Bool": {
            "kms:GrantIsForAWSResource": "true"
        }
    }
}
```

If these permissions don't meet your needs, you can edit them in the console [policy view](key-policy-modifying.md#key-policy-modifying-how-to-console-policy-view) or by using the [PutKeyPolicy]() operation\. You can specify particular external users and role instead of giving permission to the external account\. You can change the actions that the policy specifies\. And you can use global and AWS KMS policy conditions to refine the permissions\.

## Using external CMKs with AWS services<a name="cross-account-service"></a>

You can give a user in a different account permission to use your CMK with a service that is integrated with AWS KMS\. For example, a user in an external account can use your CMK to [encrypt the objects in an Amazon S3 bucket](services-s3.md) or to [encrypt the secrets they store in AWS Secrets Manager](services-secrets-manager.md)\.

The key policy must give the external user or the external user's account permission to use the CMK\. In addition, you need to attach IAM policies to the identity that gives the user permission to use the AWS service\. 

Also, the service might require that users have additional permissions in the key policy\. For example, it might require permission to create, list, and revoke grants on the CMK\. Or it might require particular IAM policies\. For details, see the documentation for the service\. 

Finally, the lists of CMKs displayed in the AWS Management Console for an integrated service do not include CMKs in external accounts\. This is true even when the user or role has permission to use them\. To use an external account's CMK, the user must enter the ID or ARN of the CMK\. For details, see the service's console documentation\.