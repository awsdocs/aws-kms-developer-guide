# Using hybrid post\-quantum TLS with AWS KMS<a name="pqtls"></a>

AWS Key Management Service \(AWS KMS\) now supports a hybrid post\-quantum key exchange option for the Transport Layer Security \(TLS\) network encryption protocol\. You can use this TLS option when you connect to KMS API endpoints\. We're offering this feature before post\-quantum algorithms are standardized so you can begin testing the effect of these key exchange protocols on AWS KMS calls\. These optional hybrid post\-quantum key exchange features are at least as secure as the TLS encryption we use today and are likely to provide additional security benefits\. However, they affect latency and throughput compared to the classic key exchange protocols in use today\.

The data that you send to AWS Key Management Service \(AWS KMS\) is protected in transit by the encryption provided by a Transport Layer Security \(TLS\) connection\. The classic cipher suites that AWS KMS supports for TLS sessions make brute force attacks on the key exchange mechanisms infeasible with current technology\. However, if large\-scale quantum computing becomes practical in the future, the classic cipher suites used in TLS key exchange mechanisms will be susceptible to these attacks\. If you’re developing applications that rely on the long\-term confidentiality of data passed over a TLS connection, you should consider a plan to migrate to post\-quantum cryptography before large\-scale quantum computers become available for use\. AWS is working to prepare for this future, and we want you to be well\-prepared, too\.

To protect data encrypted today against potential future attacks, AWS is participating with the cryptographic community in the development of quantum\-resistant or *post\-quantum* algorithms\. We've implemented hybrid post\-quantum key exchange cipher suites in AWS KMS endpoints\. These hybrid cipher suites, which combine classic and post\-quantum elements, ensure that your TLS connection is at least as strong as it would be with classic cipher suites\.

These hybrid cipher suites are available for use on your production workloads in [most AWS Regions](#pqtls-regions)\. However, because the performance characteristics and bandwidth requirements of hybrid cipher suites are different from those of classic key exchange mechanisms, we recommend that you [test them on your AWS KMS API calls](#pqtls-testing) under different conditions\. 

**Feedback**

As always, we welcome your feedback and participation in our open\-source repositories\. We’d especially like to hear how your infrastructure interacts with this new variant of TLS traffic\. 
+ To provide feedback on this topic, use the **Feedback** link in the lower right corner of this page\. You can also [create an issue](https://github.com/awsdocs/aws-kms-developer-guide/issues) or a pull request in the [aws\-kms\-developer\-docs](https://github.com/awsdocs/aws-kms-developer-guide/) repository in GitHub\.
+ We're developing these hybrid cipher suites in open source in the [s2n](https://github.com/awslabs/s2n) repository on GitHub\. To provide feedback on the usability of the cipher suites, or share novel test conditions or results, [create an issue](https://github.com/awslabs/s2n/issues) in the s2n repository\.
+ We're writing code samples for using hybrid post\-quantum TLS with AWS KMS in the [aws\-kms\-pq\-tls\-example](https://github.com/aws-samples/aws-kms-pq-tls-example) GitHub repository\. To ask questions or share ideas about configuring your HTTP client or AWS KMS client to use the hybrid cipher suites, [create an issue](https://github.com/aws-samples/aws-kms-pq-tls-example/issues) in the `aws-kms-pq-tls-example` repository\.

**Supported AWS Regions**

Post\-quantum TLS for AWS KMS is available in all AWS Regions except for AWS GovCloud \(US\-East\), AWS GovCloud \(US\-West\), China \(Beijing\), and China \(Ningxia\)\. 

For a list of AWS KMS endpoints for each AWS Region, see [AWS Key Management Service Endpoints and Quotas](https://docs.aws.amazon.com/general/latest/gr/kms.html) in the *Amazon Web Services General Reference*\. For information about FIPS endpoints, see [AWS Service Endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html) in the *Amazon Web Services General Reference*\.\.

## About hybrid post\-quantum key exchange in TLS<a name="PQTLS-concepts"></a>

AWS KMS supports hybrid post\-quantum key exchange cipher suites\. You can use the AWS SDK for Java 2\.x and AWS common runtime to configure an HTTP client to use these cipher suites\. Then, whenever you connect to a AWS KMS endpoint with your HTTP client, the hybrid cipher suites are used\.

This HTTP client uses [s2n](https://github.com/awslabs/s2n), which is an open source implementation of the TLS protocol\. s2n includes the [pq\-crypto](https://github.com/awslabs/s2n/tree/master/pq-crypto) module, which includes implementations of hybrid post\-quantum algorithms for encryption in transit\.

The hybrid cipher suites in s2n are implemented only for key exchange, not for direct data encryption\. During *key exchange*, the client and server calculate the key they will use to encrypt and decrypt the data on the wire\.

The algorithms that s2n uses are a *hybrid* that combines [Elliptic Curve Diffie\-Hellman](https://en.wikipedia.org/wiki/Elliptic-curve_Diffie%E2%80%93Hellman) \(ECDH\), a classic key exchange algorithm used today in TLS, with [Bit Flipping Key Encapsulation](https://bikesuite.org/) \(BIKE\), a proposed post\-quantum algorithm\. This mechanism uses each of the algorithms independently to generate a key\. Then it combines the two keys cryptographically\. With s2n, you can configure an HTTP client with a *cipher preference* that places ECDH with BIKE first in the preference list\. Classic key exchange algorithms are included in the preference list to ensure compatibility, but they are lower in the preference order\.

If ongoing research reveals that the BIKE algorithm lacks the anticipated post\-quantum strength, the hybrid key is still at least as strong as the single ECDH key currently in use\. The National Institute for Standards and Technology \(NIST\) has [not yet standardized](https://csrc.nist.gov/Projects/Post-Quantum-Cryptography/Post-Quantum-Cryptography-Standardization) post\-quantum algorithms\. They are still in the process of evaluating candidate approaches\. Until that process is complete, we recommend using hybrid algorithms, rather than using post\-quantum algorithms alone\.

## Using hybrid post\-quantum TLS with AWS KMS<a name="pqtls-details"></a>

You can use hybrid post\-quantum TLS for your calls to AWS KMS\. When setting up your HTTP client test environment, be aware of the following information:

**Encryption in Transit**

The hybrid cipher suites in s2n are used only for encryption in transit\. They protect your data while it is traveling from your client to the AWS KMS endpoint\. AWS KMS does not use these cipher suites to encrypt data under customer master keys \(CMKs\)\. 

Instead, when AWS KMS encrypts your data under CMKs, it uses symmetric cryptography with 256\-bit keys and the Advanced Encryption Standard in Galois Counter Mode \(AES\-GCM\) algorithm, which is already quantum resistant\. Theoretical future, large\-scale quantum computing attacks on ciphertexts created under 256\-bit AES\-GCM keys [reduce the effective security of the key to 128 bits](https://www.etsi.org/images/files/ETSIWhitePapers/QuantumSafeWhitepaper.pdf)\. This security level is sufficient to make brute force attacks on AWS KMS ciphertexts infeasible\. 

**Supported Systems**

Use of the hybrid cipher suites in s2n is currently supported only on Linux systems\. In addition, these cipher suites are supported only in SDKs that support the AWS common runtime, such as the AWS SDK for Java 2\.x\. For an example, see [How to configure hybrid post\-quantum TLS](#pqtls-how-to)\.

**AWS KMS Endpoints**

When using the hybrid cipher suites, use the standard AWS KMS endpoint\. The hybrid cipher suites in s2n are not compatible with the [FIPS 140\-2 validated endpoints for AWS KMS](https://docs.aws.amazon.com/general/latest/gr/rande.html#kms_region)\. Post\-quantum algorithms are not allowed in a validated cryptographic module\. 

When you configure a HTTP client with the hybrid post\-quantum cipher preference in s2n, the post\-quantum ciphers are first in the cipher preference list\. However, the preference list includes the classic, non\-hybrid ciphers lower in the preference order for compatibility\. If you were to use this cipher preference with an AWS KMS FIPS 140\-2 validated endpoint, s2n negotiates a classic, non\-hybrid key exchange cipher\.

For a list of AWS KMS endpoints for each AWS Region, see [AWS Key Management Service Endpoints and Quotas](https://docs.aws.amazon.com/general/latest/gr/kms.html) in the *Amazon Web Services General Reference*\. For information about FIPS endpoints, see [AWS Service Endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html) in the *Amazon Web Services General Reference*\.

**Expected Performance**

Our early benchmark testing shows that the hybrid cipher suites in s2n are slower than classic TLS cipher suites\. The effect varies based on the network profile, CPU speed, the number of cores, and your call rate\. For performance test results, see [Post\-quantum TLS now supported in AWS KMS](http://aws.amazon.com/blogs/security/post-quantum-tls-now-supported-in-aws-kms/)\.

## How to configure hybrid post\-quantum TLS<a name="pqtls-how-to"></a>

In this procedure, you get the `aws-crt-dev-preview` \(developer preview\) branch of the [AWS SDK for Java 2\.x](https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk) from its GitHub repository\. Next, you build an AWS common runtime client and add the AWS common runtime to your dependencies\. Then you can configure an HTTP client that uses the hybrid post\-quantum cipher preference and create an AWS KMS client that uses the HTTP client\.

To see a complete working examples of configuring and using hybrid post\-quantum TLS with AWS KMS, see the [aws\-kms\-pq\-tls\-example](https://github.com/aws-samples/aws-kms-pq-tls-example) repository\.

1. Clone the developer preview branch \([https://github.com/aws/aws-sdk-java-v2/tree/aws-crt-dev-preview](https://github.com/aws/aws-sdk-java-v2/tree/aws-crt-dev-preview)\) of the AWS SDK for Java 2\.x\.

   The AWS SDK for Java 2\.x is being developed in the [aws\-sdk\-java\-v2](https://github.com/aws/aws-sdk-java-v2) GitHub repository\. 
**Note**  
The `aws-crt-dev-preview` branch is a beta release\. Your use of this library is subject to Section 1\.10 \("Beta Service Participation"\) of the [AWS Service Terms](https://aws.amazon.com/service-terms/)\.

   ```
   $ git clone git@github.com:aws/aws-sdk-java-v2.git --branch aws-crt-dev-preview
   ```

1. Install and build the AWS common runtime client \(`aws-crt-client`\) JAR\. 

   ```
   $ cd aws-sdk-java-v2
   $ mvn install -Pquick
   ```

1. Add the AWS common runtime client to your Maven dependencies\. We recommend using the latest available version\. 

   For example, this statement adds version `2.10.7-SNAPSHOT` of the AWS common runtime client to your Maven dependencies\. 

   ```
   <dependency>
       <groupId>software.amazon.awssdk</groupId>
       <artifactId>aws-crt-client</artifactId>
       <version>2.10.7-SNAPSHOT</version>
   </dependency>
   ```

1. To enable the hybrid post\-quantum cipher suites, add the AWS SDK for Java 2\.x to your project and initialize it\. Then enable the hybrid cipher suites as shown in the following example\.

   This code ensures that you are working on a system that supports the hybrid cipher suite\. The code then creates an HTTP client with the `TLS_CIPHER_KMS_PQ_TLSv1_0_2019_06` cipher preference that prioritizes the ECDH with BIKE hybrid cipher suite\. Finally, it creates an AWS KMS client that uses the HTTP client for data transmission\.

   This code uses the AWS KMS asynchronous client, `KmsAsyncClient`, which calls AWS KMS asynchronously\. For information about this client, see the [KmsAsyncClient Javadoc](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/kms/KmsAsyncClient.html)\.

   After this code completes, your [AWS KMS API](https://docs.aws.amazon.com/kms/latest/APIReference/) requests on the AWS KMS asynchronous client use the hybrid cipher suite for TLS\.

   ```
   // Check platform support
   if(!TLS_CIPHER_KMS_PQ_TLSv1_0_2019_06.isSupported()){
       throw new RuntimeException("Hybrid post-quantum cipher suites are not supported on this platform");
   }
   
   // Configure HTTP client   
   SdkAsyncHttpClient awsCrtHttpClient = AwsCrtAsyncHttpClient.builder()
             .tlsCipherPreference(TLS_CIPHER_KMS_PQ_TLSv1_0_2019_06)
             .build();
   
   // Create the KMS async client
   KmsAsyncClient kmsAsync = KmsAsyncClient.builder()
            .httpClient(awsCrtHttpClient)
            .build();
   ```

1. Test your KMS calls with post\-quantum TLS\.

   When you call AWS KMS API operations on the configured AWS KMS client, your calls are transmitted to the AWS KMS endpoint using hybrid post\-quantum TLS\. To test your configuration, run a simple KMS API call, such as `[ListKeys](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListKeys.html)`\.

   ```
   ListKeysReponse keys = kmsAsync.listKeys().get();
   ```

## Testing hybrid post\-quantum TLS with AWS KMS<a name="pqtls-testing"></a>

Consider running the following tests with hybrid cipher suites on your applications that call AWS KMS\.
+ Run load tests and benchmarks\. The hybrid cipher suites perform differently than traditional key exchange algorithms\. You might need to adjust your connection timeouts to allow for the longer handshake times\. If you’re running inside an AWS Lambda function, extend the execution timeout setting\.
+ Try connecting from different locations\. Depending on the network path your request takes, you might discover that intermediate hosts, proxies, or firewalls with deep packet inspection \(DPI\) block the request\. This might result from using the new cipher suites in the [ClientHello](https://tools.ietf.org/html/rfc5246#section-7.4.1.2) part of the TLS handshake, or from the larger key exchange messages\. If you have trouble resolving these issues, work with your security team or IT administrators to update the relevant configuration and unblock the new TLS cipher suites\. 

## Learn more about post\-quantum TLS in AWS KMS<a name="pqtls-see-also"></a>

For more information about using hybrid post\-quantum TLS in AWS KMS, see the following resources\.
+ For more information about using hybrid post\-quantum TLS cipher suites with AWS KMS, including performance data, see [Post\-quantum TLS now supported in AWS KMS](http://aws.amazon.com/blogs/security/post-quantum-tls-now-supported-in-aws-kms/)\.
+ For information about the AWS SDK for Java 2\.x, see the [AWS SDK for Java 2\.x Developer Guide](https://docs.aws.amazon.com/sdk-for-java/v2/developer-guide/) and the [AWS SDK for Java 2\.x released](http://aws.amazon.com/blogs/developer/aws-sdk-for-java-2-x-released/) blog post\.
+ For information about s2n, see [Introducing s2n, a New Open Source TLS Implementation](http://aws.amazon.com/blogs/security/introducing-s2n-a-new-open-source-tls-implementation/) and [Using s2n](https://github.com/awslabs/s2n/blob/master/docs/USAGE-GUIDE.md)\.
+ For information about the post\-quantum cryptography project at the National Institute for Standards and Technology \(NIST\), see [Post\-Quantum Cryptography](https://csrc.nist.gov/Projects/Post-Quantum-Cryptography)\.
+ For technical information about using hybrid post\-quantum key exchange in TLS, see [Hybrid Post\-Quantum Key Encapsulation Methods \(PQ KEM\) for Transport Layer Security 1\.2 \(TLS\)](https://tools.ietf.org/html/draft-campagna-tls-bike-sike-hybrid-01)\.