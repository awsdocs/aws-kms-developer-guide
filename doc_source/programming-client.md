# Creating a Client<a name="programming-client"></a>

To use the [AWS SDK for Java](https://aws.amazon.com/sdk-for-java/) to write code that uses the [AWS Key Management Service \(AWS KMS\) API](http://docs.aws.amazon.com/kms/latest/APIReference/), you start by creating a client\. The following code snippet shows you how to do this using the SDK's client builder\. For more information about using the client builder, see the following resources\.

+ [Fluent Client Builders](https://aws.amazon.com/blogs/developer/fluent-client-builders/) on the AWS Developer Blog

+ [Creating Service Clients](http://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/creating-clients.html) in the *AWS SDK for Java Developer Guide*

+ [AWSKMSClientBuilder](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/index.html?com/amazonaws/services/kms/AWSKMSClientBuilder.html) in the *AWS SDK for Java API Reference*

The client object, `kms`, is used in the example code in the topics that follow\.

```
AWSKMS kms = AWSKMSClientBuilder.defaultClient();
```