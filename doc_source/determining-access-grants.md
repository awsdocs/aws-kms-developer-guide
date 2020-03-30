# Examining grants<a name="determining-access-grants"></a>

Grants are advanced mechanisms for specifying permissions that you or an AWS service integrated with AWS KMS can use to specify how and when a CMK can be used\. Grants are attached to a CMK, and each grant contains the principal who receives permission to use the CMK and a list of operations that are allowed\. Grants are an alternative to the key policy, and are useful for specific use cases\. For more information, see [Using grants](grants.md)\.

To get a list of grants for a CMK, use the AWS KMS [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation\. You can examine the grants for a CMK to determine who or what currently has access to use the CMK via those grants\. For example, the following is a JSON representation of a grant that was obtained from the [list\-grants](https://docs.aws.amazon.com/cli/latest/reference/kms/list-grants.html) command in the AWS CLI\.

```
{"Grants": [{
  "Operations": ["Decrypt"],
  "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
  "Name": "0d8aa621-43ef-4657-b29c-3752c41dc132",
  "RetiringPrincipal": "arn:aws:iam::123456789012:root",
  "GranteePrincipal": "arn:aws:sts::111122223333:assumed-role/aws:ec2-infrastructure/i-5d476fab",
  "GrantId": "dc716f53c93acacf291b1540de3e5a232b76256c83b2ecb22cdefa26576a2d3e",
  "IssuingAccount": "arn:aws:iam::111122223333:root",
  "CreationDate": 1.444151834E9,
  "Constraints": {"EncryptionContextSubset": {"aws:ebs:id": "vol-5cccfb4e"}}
}]}
```

To find out who or what has access to use the CMK, look for the `"GranteePrincipal"` element\. In the preceding example, the grantee principal is an assumed role user that is associated with the EC2 instance i\-5d476fab\. The EC2 infrastructure uses this role to attach the encrypted EBS volume vol\-5cccfb4e to the instance\. In this case, the EC2 infrastructure role has permission to use the CMK because you previously created an encrypted EBS volume that is protected by this CMK\. You then attached the volume to an EC2 instance\.

The following is another example of a JSON representation of a grant that was obtained from the [list\-grants](https://docs.aws.amazon.com/cli/latest/reference/kms/list-grants.html) command in the AWS CLI\. In the following example, the grantee principal is another AWS account\.

```
{"Grants": [{
  "Operations": ["Encrypt"],
  "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
  "Name": "",
  "GranteePrincipal": "arn:aws:iam::444455556666:root",
  "GrantId": "f271e8328717f8bde5d03f4981f06a6b3fc18bcae2da12ac38bd9186e7925d11",
  "IssuingAccount": "arn:aws:iam::111122223333:root",
  "CreationDate": 1.444151269E9
}]}
```