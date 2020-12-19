# Using aliases in your applications<a name="alias-using"></a>

You can use an alias to represent a CMK in your application code\. The `KeyId` parameter in the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) and [GetPublicKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetPublicKey.html) operations and in all [cryptographic operations](concepts.md#cryptographic-operations) accepts an alias name or alias ARN\.

For example, the following `GenerateDataKey` command uses an alias name \(`alias/finance`\) to identify a CMK\. The alias name is the value of the `KeyId` parameter\. 

```
$ aws kms generate-data-key --key-id alias/finance --key-spec AES_256
```

One of the most powerful uses of aliases is in applications that run in multiple AWS Regions\. For example, you might have a global application that uses an RSA [asymmetric CMK](symm-asymm-concepts.md#asymmetric-cmks) for signing and verification\. 
+ In US West \(Oregon\) \(us\-west\-2\), you want to use `arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab`\. 
+ In Europe \(Frankfurt\) \(eu\-central\-1\), you want to use `arn:aws:kms:eu-central-1:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321`
+ In Asia Pacific \(Singapore\) \(ap\-southeast\-1\), you want to use `arn:aws:kms:ap-southeast-1:111122223333:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d`\.

You could create a different version of your application in each Region or use a dictionary or switch statement to select the right CMK for each Region\. But it's much easier to create an alias with the same alias name in each Region\. Remember that the alias name is case\-sensitive\.

```
aws --region us-west-2 kms create-alias \
    --alias-name alias/new-app \
    --key-id arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab

aws --region eu-central-1 kms create-alias \
    --alias-name alias/new-app \
    --key-id arn:aws:kms:eu-central-1:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321

aws --region ap-southeast-1 kms create-alias \
    --alias-name alias/new-app \
    --key-id arn:aws:kms:ap-southeast-1:111122223333:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d
```

Then, use the alias in your code\. When your code runs in each Region, the alias will refer to its associated CMK in that Region\. For example, this code calls the [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html) operation with an alias name\.

```
aws kms sign --key-id alias/new-app \
    --message $message \
    --message-type RAW \
    --signing-algorithm RSASSA_PSS_SHA_384
```

However, there is a risk that the alias might be deleted or updated to be associated with a different CMK\. In that case, the application's attempts to verify signatures using the alias name will fail, and you might need to recreate or update the alias\.

To mitigate this risk, be cautious about giving principals permission to manage the aliases that you use in your application\. For details, see [Controlling access to aliases](alias-access.md)\.

There are several other solutions for applications that encrypt data in multiple AWS Regions, including the [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/)\.