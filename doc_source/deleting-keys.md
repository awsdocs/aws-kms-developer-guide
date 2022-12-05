# Deleting AWS KMS keys<a name="deleting-keys"></a>

Deleting an AWS KMS key is destructive and potentially dangerous\. It deletes the key material and all metadata associated with the KMS key and is irreversible\. After a KMS key is deleted, you can no longer decrypt the data that was encrypted under that KMS key, which means that data becomes unrecoverable\. You should delete a KMS key only when you are sure that you don't need to use it anymore\. If you are not sure, consider [disabling the KMS key](enabling-keys.md) instead of deleting it\. You can re\-enable a disabled KMS key and [cancel the scheduled deletion](deleting-keys-scheduling-key-deletion.md) of a KMS key, but you cannot recover a deleted KMS key\.

You can only schedule the deletion of a customer managed key\. You cannot delete AWS managed keys or AWS owned keys\.

Before deleting a KMS key, you might want to know how many ciphertexts were encrypted under that KMS key\. AWS KMS does not store this information and does not store any of the ciphertexts\. To get this information, you must determine past usage of a KMS key\. For help, go to [Determining past usage of a KMS key](deleting-keys-determining-usage.md)\.

AWS KMS never deletes your KMS keys unless you explicitly schedule them for deletion and the mandatory waiting period expires\.

However, you might choose to delete a KMS key for one or more of the following reasons:
+ To complete the key lifecycle for KMS keys that you no longer need
+ To avoid the management overhead and [costs](https://aws.amazon.com/kms/pricing/) associated with maintaining unused KMS keys
+ To reduce the number of KMS keys that count against your [KMS key resource quota](resource-limits.md#kms-keys-limit)

**Note**  
If you [close or delete your AWS account](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/close-account.html), your KMS keys become inaccessible and you are no longer billed for them\. You do not need to schedule deletion of your KMS keys separate from closing the account\.

AWS KMS records an entry in your AWS CloudTrail log when you [schedule deletion](ct-schedule-key-deletion.md) of the KMS key and when the [KMS key is actually deleted](ct-delete-key.md)\. 

For information about deleting multi\-Region primary and replica keys, see [Deleting multi\-Region keys](multi-region-keys-delete.md)\.

**Topics**
+ [About the waiting period](#deleting-keys-how-it-works)
+ [Deleting asymmetric KMS keys](#deleting-asymmetric-cmks)
+ [Deleting multi\-Region keys](#deleting-mrks)
+ [Controlling access to key deletion](deleting-keys-adding-permission.md)
+ [Scheduling and canceling key deletion](deleting-keys-scheduling-key-deletion.md)
+ [Creating an alarm that detects use of a KMS key pending deletion](deleting-keys-creating-cloudwatch-alarm.md)
+ [Determining past usage of a KMS key](deleting-keys-determining-usage.md)

## About the waiting period<a name="deleting-keys-how-it-works"></a>

Because it is destructive and potentially dangerous to delete a KMS key, AWS KMS requires you to set a waiting period of 7 – 30 days\. The default waiting period is 30 days\.

However, the actual waiting period might be up to 24 hours longer than the one you scheduled\. To get the actual date and time when the KMS key will be deleted, use the [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html) operation\. Or in the AWS KMS console, on [detail page](viewing-keys-console.md#viewing-details-navigate) for the KMS key, in the **General configuration** section, see the **Scheduled deletion date**\. Be sure to note the time zone\.

During the waiting period, the KMS key status and key state is **Pending deletion**\.
+ A KMS key pending deletion cannot be used in any [cryptographic operations](concepts.md#cryptographic-operations)\. 
+ AWS KMS does not [rotate the key material](rotate-keys.md#rotate-keys-how-it-works) of KMS keys that are pending deletion\.

After the waiting period ends, AWS KMS deletes the KMS key, its aliases, and all related AWS KMS metadata\.

Scheduling the deletion of a KMS key might not immediately affect data keys encrypted by the KMS key\. For details, see [How unusable KMS keys affect data keys](concepts.md#unusable-kms-keys)\.

Use the waiting period to ensure that you don't need the KMS key now or in the future\. You can [configure an Amazon CloudWatch alarm](deleting-keys-creating-cloudwatch-alarm.md) to warn you if a person or application attempts to use the KMS key during the waiting period\. To recover the KMS key, you can cancel key deletion before the waiting period ends\. After the waiting period ends you cannot cancel key deletion, and AWS KMS deletes the KMS key\.

## Deleting asymmetric KMS keys<a name="deleting-asymmetric-cmks"></a>

Users [who are authorized](deleting-keys-adding-permission.md) can delete symmetric or asymmetric KMS keys\. The procedure to schedule the deletion of these KMS keys is the same for both types of keys\. However, because the [public key of an asymmetric KMS key can be downloaded](download-public-key.md) and used outside of AWS KMS, the operation poses significant additional risks, especially for asymmetric KMS keys used for encryption \(the key usage is `ENCRYPT_DECRYPT`\)\.
+ When you schedule the deletion of a KMS key, the key state of KMS key changes to **Pending deletion**, and the KMS key cannot be used in [cryptographic operations](concepts.md#cryptographic-operations)\. However, scheduling deletion has no effect on public keys outside of AWS KMS\. Users who have the public key can continue to use them to encrypt messages\. They do not receive any notification that the key state is changed\. Unless the deletion is canceled, ciphertext created with the public key cannot be decrypted\.
+ Alarms, logs, and other strategies that detect attempted use of KMS key that is pending deletion cannot detect use of the public key outside of AWS KMS\.
+ When the KMS key is deleted, all AWS KMS actions involving that KMS key fail\. However, users who have the public key can continue to use them to encrypt messages\. These ciphertexts cannot be decrypted\.

If you must delete an asymmetric KMS key with a key usage of `ENCRYPT_DECRYPT`, use your CloudTrail Log entries to determine whether the public key has been downloaded and shared\. If it has, verify that the public key is not being used outside of AWS KMS\. Then, consider [disabling the KMS key](enabling-keys.md) instead of deleting it\.

## Deleting multi\-Region keys<a name="deleting-mrks"></a>

Users [who are authorized](deleting-keys-adding-permission.md) can schedule the deletion of multi\-Region primary and replica keys\. However, AWS KMS will not delete a multi\-Region primary key that has replica keys\. Also, as long as its primary key exists, you can recreate a deleted multi\-Region replica key\. For details, see [Deleting multi\-Region keys](multi-region-keys-delete.md)\.