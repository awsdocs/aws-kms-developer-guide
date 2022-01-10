# How Amazon Simple Storage Service \(Amazon S3\) uses AWS KMS<a name="services-s3"></a>

[Amazon Simple Storage Service \(Amazon S3\)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) is an object storage service that stores data as objects within buckets\. Buckets and the objects in them are private and can be accessed only if you explicitly grant access permissions\. 

Amazon S3 integrates with AWS Key Management Service \(AWS KMS\) to provide server side encryption of Amazon S3 objects\. Amazon S3 uses AWS KMS keys to encrypt your Amazon S3 objects\. This integration protects your objects under encryption keys that never leave AWS KMS unencrypted\. It also enables you to set permissions on the AWS KMS key and audit the operations that generate, encrypt, and decrypt the data keys that protect your secrets\. 

For more information about how Amazon S3 uses AWS KMS, see [Protecting data using server\-side encryption with KMS keys \(SSE\-KMS\)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html) in the *Amazon S3 User Guide*\.