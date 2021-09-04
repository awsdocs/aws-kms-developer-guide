# Creating AWS KMS resources with AWS CloudFormation<a name="creating-resources-with-cloudformation"></a>

AWS Key Management Service is integrated with AWS CloudFormation, a service that helps you to model and set up your AWS resources so that you can spend less time creating and managing your resources and infrastructure\. You create a template that describes the AWS resources that you want, including KMS keys, aliases, and [multi\-Region replica keys](multi-region-keys-overview.md#mrk-replica-key), and AWS CloudFormation provisions and configures those resources for you\. For information about AWS KMS support for CloudFormation, see the [KMS resource type reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/AWS_KMS.html) in the *AWS CloudFormation User Guide*\.

When you use AWS CloudFormation, you can reuse your template to set up your AWS KMS resources consistently and repeatedly\. Describe your resources once, and then provision the same resources over and over in multiple AWS accounts and Regions\. 

To provision and configure resources for AWS KMS and other AWS services, you must understand [AWS CloudFormation templates](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-guide.html)\. Templates are formatted text files in JSON or YAML\. These templates describe the resources that you want to provision in your AWS CloudFormation stacks\. If you're unfamiliar with JSON or YAML, you can use AWS CloudFormation Designer to help you get started with AWS CloudFormation templates\. For more information, see [What is AWS CloudFormation Designer?](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/working-with-templates-cfn-designer.html) in the *AWS CloudFormation User Guide*\.

## AWS KMS resources in AWS CloudFormation templates<a name="working-with-templates"></a>

AWS KMS supports the following AWS CloudFormation resources\. 
+ `AWS::KMS::Key` creates a symmetric or asymmetric [KMS key](concepts.md#kms_keys)\. You cannot use this resource to create KMS keys with [imported key material](importing-keys.md) or KMS keys in a [custom key store](custom-key-store-overview.md)\. 
+ `AWS::KMS::Alias` creates an [alias](kms-alias.md) and associates it with a KMS key\. The KMS key can be defined in the template, or created by another mechanism\.
+ `AWS::KMS::ReplicaKey` creates a [multi\-Region replica key](multi-region-keys-overview.md#mrk-replica-key)\. To create a multi\-Region primary key, use the `AWS::KMS::Key` resource\. You cannot use this resource to create multi\-Region keys with [imported key material](multi-region-keys-import.md)\. For details about multi\-Region keys, see [Using multi\-Region keys](multi-region-keys-overview.md)\.

The KMS keys that the template creates are actual resources in your AWS account\. Authorized principals can use and manage the KMS keys that the template creates, either by using the template, the AWS KMS console, or the AWS KMS APIs\. When you delete a KMS key from your template, the KMS key is scheduled for deletion using a waiting period that you specify in advance\. 

For example, you can use an AWS CloudFormation template to create a test KMS key with a key policy, key spec, key usage, aliases, and tags you prefer\. You can run it through your test suite, review your results, and then use the template to schedule the test key for deletion\. Later, you can run the template again to create a test key with the same properties\. 

Or you can use an AWS CloudFormation template to define a particular KMS key configuration that satisfies your business rules and security standards\. Then you can use that template any time you need to create a KMS key\. You don't have to worry about misconfigured keys\. If your preferred configuration changes, you can use your template to update your KMS keys\. For example, the template makes it easy to programmatically enable automatic key rotation on all KMS keys that the template defines\.

For more information about AWS KMS resources, including examples, see the [KMS resource type reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/AWS_KMS.html) in the *AWS CloudFormation User Guide*\.

## Learn more about AWS CloudFormation<a name="learn-more-cloudformation"></a>

To learn more about AWS CloudFormation, see the following resources:
+ [AWS CloudFormation](http://aws.amazon.com/cloudformation/)
+ [AWS CloudFormation User Guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)
+ [AWS CloudFormation API Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/Welcome.html)
+ [AWS CloudFormation Command Line Interface User Guide](https://docs.aws.amazon.com/cloudformation-cli/latest/userguide/what-is-cloudformation-cli.html)