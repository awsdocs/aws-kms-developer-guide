# Creating a Client<a name="programming-client"></a>

To use the [AWS SDK for Java](https://aws.amazon.com/sdk-for-java/), the [AWS SDK for \.NET](https://aws.amazon.com/sdk-for-net/), or the [AWS SDK for Python \(Boto3\)](https://aws.amazon.com/sdk-for-python/) to write code that uses the [AWS Key Management Service \(AWS KMS\) API](http://docs.aws.amazon.com/kms/latest/APIReference/), start by creating an AWS KMS client\.

The client object that you create is used in the example code in the topics that follow\.

------
#### [ Java ]

To create an AWS KMS client in Java, use the client builder\.

```
AWSKMS kmsClient = AWSKMSClientBuilder.defaultClient();
```

For more information about using the Java client builder, see the following resources\.
+ [Fluent Client Builders](https://aws.amazon.com/blogs/developer/fluent-client-builders/) on the AWS Developer Blog
+ [Creating Service Clients](http://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/creating-clients.html) in the *AWS SDK for Java Developer Guide*
+ [AWSKMSClientBuilder](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/index.html?com/amazonaws/services/kms/AWSKMSClientBuilder.html) in the *AWS SDK for Java API Reference*

------
#### [ C\# ]

```
AmazonKeyManagementServiceClient kmsClient = new AmazonKeyManagementServiceClient();
```

------
#### [ Python ]

```
kms_client = boto3.client('kms')
```

------