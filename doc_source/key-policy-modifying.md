# Changing a Key Policy<a name="key-policy-modifying"></a>

To change the permissions for a customer master key \(CMK\) in AWS KMS, you change the CMK's [key policy](key-policies.md)\.

When changing a key policy, keep in mind the following rules:
+ You can add or remove IAM users, IAM roles, and AWS accounts \(root users\) in the key policy, and change the actions that are allowed or denied for those principals\. For more information about the ways to specify principals and permissions in a key policy, see [Using Key Policies](key-policies.md)\.

   
+ You cannot add IAM groups to a key policy, but you can add multiple IAM users\. For more information, see [Allowing Multiple IAM Users to Access a CMK](#key-policy-modifying-multiple-iam-users)\.

   
+ If you add external AWS accounts to a key policy, you must also use IAM policies in the external accounts to give permissions to IAM users, groups, or roles in those accounts\. For more information, see [Allowing Users in Other Accounts to Use a CMK](key-policy-modifying-external-accounts.md)\.

   
+ The resulting key policy document cannot exceed 32 KB \(32,768 bytes\)\.

**Topics**
+ [How to Change a Key Policy](#key-policy-modifying-how-to)
+ [Allowing Multiple IAM Users to Access a CMK](#key-policy-modifying-multiple-iam-users)

## How to Change a Key Policy<a name="key-policy-modifying-how-to"></a>

You can change a key policy in three different ways, each of which is explained in the following sections\.

**Topics**
+ [Using the AWS Management Console Default View](#key-policy-modifying-how-to-console-default-view)
+ [Using the AWS Management Console Policy View](#key-policy-modifying-how-to-console-policy-view)
+ [Using the AWS KMS API](#key-policy-modifying-how-to-api)

### Using the AWS Management Console Default View<a name="key-policy-modifying-how-to-console-default-view"></a>

You can use the console to change a key policy with a graphical interface called the *default view*\.

If the following steps don't match what you see in the console, it might mean that this key policy was not created by the console\. Or it might mean that the key policy has been modified in a way that the console's default view does not support\. In that case, follow the steps at [Using the AWS Management Console Policy View](#key-policy-modifying-how-to-console-policy-view) or [Using the AWS KMS API](#key-policy-modifying-how-to-api)\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\. \(You cannot change the key policies of AWS managed keys\.\)

1. Choose the alias or key ID of the CMK whose key policy you want to change\.

1. Scroll down to the **Key policy** tab\.

1. Decide what to change\.
   + To add or remove [key administrators](key-policies.md#key-policy-default-allow-administrators), and to allow or prevent key administrators from [deleting the CMK](deleting-keys.md), use the controls in the **Key administrators** section of the page\. Key administrators manage the CMK, including enabling and disabling it, setting key policy, and [enabling key rotation](rotate-keys.md)\.
   + To add or remove [key users](key-policies.md#key-policy-default-allow-users), and to allow or disallow external AWS accounts to use the CMK, use the controls in the **Key users** section of the page\. Key users can use the CMK in cryptographic operations, such as encrypting, decrypting, re\-encrypting, and generating data keys\.

### Using the AWS Management Console Policy View<a name="key-policy-modifying-how-to-console-policy-view"></a>

You can use the console to change a key policy document with the console's *policy view*\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\. \(You cannot change the key policy of an AWS managed CMK\.

1. Choose the alias or key ID of the CMK that you want to change\.

1. Scroll down to the **Key policy** tab\.

1. In the **Key Policy** section, choose **Switch to policy view**\.

1. Edit the key policy document, and then choose **Save changes**\.

### Using the AWS KMS API<a name="key-policy-modifying-how-to-api"></a>

You can use the AWS KMS API to change a key policy document\. The following steps use the [AWS KMS HTTP API](https://docs.aws.amazon.com/kms/latest/APIReference/)\. You can perform the same operations with the [AWS SDKs](https://aws.amazon.com/tools/#sdk) or [AWS command line tools](https://aws.amazon.com/tools/#cli), which is often easier than using the HTTP API\. For the operations and syntax to use for other SDKs and tools, consult the reference documentation for that particular SDK or tool\. For sample code that uses the AWS SDK for Java, see [Working with Key Policies](programming-key-policies.md)\.

**To change a key policy document \(KMS API\)**

1. Use [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) to retrieve the existing key policy document, and then save the key policy document to a file\.

1. Open the key policy document in your preferred text editor, edit the key policy document, and then save the file\.

1. Use [PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) to apply the updated key policy document to the CMK\.

## Allowing Multiple IAM Users to Access a CMK<a name="key-policy-modifying-multiple-iam-users"></a>

IAM groups are not valid principals in a key policy\. To allow multiple IAM users to access a CMK, do one of the following:
+ Add each IAM user to the key policy\. This approach requires that you update the key policy each time the list of authorized users changes\.
+ Ensure that the key policy includes the statement that [enables IAM policies to allow access to the CMK](key-policies.md#key-policy-default-allow-root-enable-iam)\. Then [create an IAM policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-using.html#create-managed-policy-console) that allows access to the CMK, and then [attach that policy to an IAM group](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-using.html#attach-managed-policy-console) that contains the authorized IAM users\. Using this approach, you don't need to change any policies when the list of authorized users changes\. Instead, you only need to add or remove those users from the appropriate IAM group\.

For more information about how AWS KMS key policies and IAM policies work together, see [Troubleshooting Key Access](policy-evaluation.md)\.