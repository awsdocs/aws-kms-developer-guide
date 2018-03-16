# Modifying a Key Policy<a name="key-policy-modifying"></a>

To change the permissions for a customer master key \(CMK\) in AWS KMS you modify the CMK's [key policy](key-policies.md)\. You can add or remove IAM users, IAM roles, and AWS accounts \(root users\) in the key policy, and change the actions that are allowed or denied for those principals\. For more information about the ways to specify principals and permissions in a key policy, see [Using Key Policies](key-policies.md)\.

You cannot add IAM groups to a key policy, though you can add multiple IAM users\. For more information, see [Allowing Multiple IAM Users to Access a CMK](#key-policy-modifying-multiple-iam-users)\.

When you add external AWS accounts to a key policy, you must also use IAM policies in the external accounts to give permissions to IAM users, groups, or roles in those accounts\. For more information, see [Allowing External AWS Accounts to Access a CMK](#key-policy-modifying-external-accounts)\.


+ [How to Modify a Key Policy](#key-policy-modifying-how-to)
+ [Allowing Multiple IAM Users to Access a CMK](#key-policy-modifying-multiple-iam-users)
+ [Allowing External AWS Accounts to Access a CMK](#key-policy-modifying-external-accounts)

## How to Modify a Key Policy<a name="key-policy-modifying-how-to"></a>

You can modify a key policy in three different ways, each of which is explained in the following sections\.


+ [Using the AWS Management Console's Default View](#key-policy-modifying-how-to-console-default-view)
+ [Using the AWS Management Console's Policy View](#key-policy-modifying-how-to-console-policy-view)
+ [Using the AWS KMS API](#key-policy-modifying-how-to-api)

### Using the AWS Management Console's Default View<a name="key-policy-modifying-how-to-console-default-view"></a>

You can use the console to modify a key policy with a graphical interface called the *default view*\.

**Note**  
If the following steps don't match what you see in the console, it means that this key policy was not created by the console, or the key policy has been modified in a way that the console's default view does not support\. In that case, follow the steps at [Using the AWS Management Console's Policy View](#key-policy-modifying-how-to-console-policy-view) or [Using the AWS KMS API](#key-policy-modifying-how-to-api)\.

**To modify a key policy \(console default view\)**

1. Sign in to the AWS Management Console and open the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Choose the alias of the CMK whose key policy you want to modify\.

1. Decide what to modify\.

   + To add or remove [key administrators](key-policies.md#key-policy-default-allow-administrators), and to allow or disallow key administrators to [delete the CMK](deleting-keys.md), use the controls in the **Key Administrators** area in the **Key Policy** section of the page\.  
![\[Key administrators area in the console's key policy section\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-administrators.png)

   + To add or remove [key users](key-policies.md#key-policy-default-allow-users), and to allow or disallow external AWS accounts to use the CMK, use the controls in the **Key Users** area in the **Key Policy** section of the page\.  
![\[Key users area in the console's key policy section\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-users.png)

### Using the AWS Management Console's Policy View<a name="key-policy-modifying-how-to-console-policy-view"></a>

You can use the console to modify a key policy document with the console's *policy view*\.

**To modify a key policy document \(console policy view\)**

1. Sign in to the AWS Management Console and open the AWS Identity and Access Management \(IAM\) console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Encryption keys**\.

1. For **Region**, choose the appropriate AWS Region\. Do not use the region selector in the navigation bar \(top right corner\)\.

1. Choose the alias of the CMK whose key policy document you want to edit\.

1. On the **Key Policy** line, choose **Switch to policy view**\.  
![\[Switch to policy view in the console\]](http://docs.aws.amazon.com/kms/latest/developerguide/images/console-key-policy-switch-to-policy-view.png)

1. Edit the key policy document, and then choose **Save Changes**\.

### Using the AWS KMS API<a name="key-policy-modifying-how-to-api"></a>

You can use the AWS KMS API to modify a key policy document\. The following steps use the [AWS KMS HTTP API](http://docs.aws.amazon.com/kms/latest/APIReference/)\. You can perform the same operations with the [AWS SDKs](https://aws.amazon.com/tools/#sdk) or [AWS command line tools](https://aws.amazon.com/tools/#cli), which is often easier than using the HTTP API\. For the operations and syntax to use for other SDKs and tools, consult the reference documentation for that particular SDK or tool\. For sample code that uses the AWS SDK for Java, see [Working with Key Policies](programming-key-policies.md)\.

**To modify a key policy document \(API\)**

1. Use [GetKeyPolicy](http://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) to retrieve the existing key policy document, and then save the key policy document to a file\.

1. Open the key policy document in your preferred text editor, edit the key policy document, and then save the file\.

1. Use [PutKeyPolicy](http://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) to apply the updated key policy document to the CMK\.

## Allowing Multiple IAM Users to Access a CMK<a name="key-policy-modifying-multiple-iam-users"></a>

IAM groups are not valid principals in a key policy\. To allow multiple IAM users to access a CMK, do one of the following:

+ Add each IAM user to the key policy\. This approach requires that you update the key policy each time the list of authorized users changes\.

+ Ensure that the key policy includes the statement that [enables IAM policies to allow access to the CMK](key-policies.md#key-policy-default-allow-root-enable-iam)\. Then [create an IAM policy](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-using.html#create-managed-policy-console) that allows access to the CMK, and then [attach that policy to an IAM group](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-using.html#attach-managed-policy-console) that contains the authorized IAM users\. Using this approach, you don't need to modify any policies when the list of authorized users changes\. Instead, you only need to add or remove those users from the appropriate IAM group\.

For more information about how AWS KMS key policies and IAM policies work together, see [Understanding Policy Evaluation](determining-access.md#policy-evaluation)\.

## Allowing External AWS Accounts to Access a CMK<a name="key-policy-modifying-external-accounts"></a>

You can allow IAM users or roles in one AWS account to access a CMK in another account\. For example, suppose that users or roles in account 111122223333 need to use a CMK in account 444455556666\. To allow this, you must do two things:

1. Modify the key policy for the CMK in account 444455556666\.

1. Add an IAM policy \(or modify an existing one\) for the users or roles in account 111122223333\.

Neither step by itself is sufficient to give access to a CMK across accounts—you must do both\.

### Modifying the CMK's Key Policy to Allow External Accounts<a name="key-policy-modifying-external-accounts-cmk"></a>

To allow IAM users or roles in one AWS account to use a CMK in a different account, you first add the external account \(root user\) to the CMK's key policy\. Note that you don't add the individual IAM users or roles to the key policy, only the external account that owns them\.

Decide what permissions you want to give to the external account:

+ To add the external account to a key policy as a *key user*, you can use the AWS Management Console's default view for the key policy\. For more information, see [Using the AWS Management Console's Default View](#key-policy-modifying-how-to-console-default-view)\.

  You can also modify the key policy document directly using the console's policy view or the AWS KMS API, as described in [Using the AWS Management Console's Policy View](#key-policy-modifying-how-to-console-policy-view) and [Using the AWS KMS API](#key-policy-modifying-how-to-api)\.

+ To add the external account to a key policy as a *key administrator* or give custom permissions, you must modify the key policy document directly using the console's policy view or the AWS KMS API\. For more information, see [Using the AWS Management Console's Policy View](#key-policy-modifying-how-to-console-policy-view) or [Using the AWS KMS API](#key-policy-modifying-how-to-api)\.

For an example of JSON syntax that adds an external account to the `Principal` element of a key policy document, see [the policy statement in the default console key policy](key-policies.md#key-policy-default-allow-users) that allows key users to use the CMK\.

### Adding or modifying an IAM Policy to Allow Access to a CMK in Another AWS Account<a name="key-policy-modifying-external-accounts-iam"></a>

After you add the external account to the CMK's key policy, you then add an IAM policy \(or modify an existing one\) to the users or roles in the external account\. In this scenario, users or roles in account 111122223333 need to use a CMK that is in account 444455556666\. To allow this, you [create an IAM policy](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-using.html#create-managed-policy-console) in account 111122223333 that allows access to the CMK in account 444455556666, and then [attach the policy](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-using.html#attach-managed-policy-console) to the users or roles in account 111122223333\. The following example shows a policy that allows access to a CMK in account 444455556666\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowUseOfCMKInAccount444455556666",
      "Effect": "Allow",
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
      ],
      "Resource": "arn:aws:kms:us-west-2:444455556666:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d"
    },
    {
      "Sid": "AllowUseofCMKToCreateEncryptedResourcesInAccount444455556666",
      "Effect": "Allow",
      "Action": "kms:CreateGrant",
      "Resource": "arn:aws:kms:us-west-2:444455556666:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d",
      "Condition": {
        "Bool": {
          "kms:GrantIsForAWSResource": true
        }
      }
    }
  ]
}
```

This policy allows users and roles in account 111122223333 to use the CMK in account 444455556666 directly for encryption and decryption, and to delegate a subset of their own permissions to some of the [AWS services that are integrated with AWS KMS](service-integration.md), specifically the services that use grants\. Note the following details about this policy:

+ The policy allows the use of a specific CMK in account 444455556666, identified by the [CMK's Amazon Resource Name \(ARN\)](control-access-overview.md#kms-resources-operations) in the `Resource` element of the policy statements\. When you give access to CMKs with an IAM policy, always list the specific CMK ARNs in the policy's `Resource` element\. Otherwise, you might inadvertently give access to more CMKs than you intend\.

+ IAM policies do not contain the `Principal` element, which differs from KMS key policies\. In IAM policies, the principal is implied by the identity to which the policy is attached\.

+ The policy gives key users permissions to allow integrated services to use the CMK, but these users also need permission to use the integrated services themselves\. For details about giving users access to an AWS service that integrates with AWS KMS, consult the documentation for the integrated service\. Also, note that for users or roles in account 111122223333, the CMK in account 444455556666 will not appear in the AWS Management Console to select when creating encrypted resources, even when the users or roles have a policy like this attached\. The console does not show CMKs in other accounts\.

For more information about working with IAM policies, see [Using IAM Policies](iam-policies.md)\.