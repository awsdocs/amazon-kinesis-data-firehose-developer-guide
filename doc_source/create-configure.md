# Configure settings<a name="create-configure"></a>

This topic describes the **Configure settings** page of the **Create Delivery Stream** wizard\.

**Configure settings**

1. On the **Configure settings** page, provide values for the following fields:  
**Buffer size, Buffer interval**  <a name="buffer"></a>
Kinesis Data Firehose buffers incoming data before delivering it to Amazon S3\. You can choose a buffer size \(1–128 MBs\) or buffer interval \(60–900 seconds\)\. The condition that is satisfied first triggers data delivery to Amazon S3\. If you enable data transformation, the buffer interval applies from the time transformed data is received by Kinesis Data Firehose to the data delivery to Amazon S3\. If data delivery to the destination falls behind data writing to the delivery stream, Kinesis Data Firehose raises the buffer size dynamically to catch up\. This action helps ensure that all data is delivered to the destination\.  
**Compression**  <a name="compression-encryption"></a>
Choose GZIP, Snappy, or Zip data compression, or no data compression\. Snappy or Zip compression is not available for delivery streams with Amazon Redshift as the destination\.   
**Encryption**  
Kinesis Data Firehose supports Amazon S3 server\-side encryption with AWS Key Management Service \(AWS KMS\) for encrypting delivered data in Amazon S3\. You can choose to not encrypt the data or to encrypt with a key from the list of AWS KMS keys that you own\. For more information, see [Protecting Data Using Server\-Side Encryption with AWS KMS–Managed Keys \(SSE\-KMS\)](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html)\.  
**Error logging**  
If data transformation is enabled, Kinesis Data Firehose can log the Lambda invocation, and send data delivery errors to CloudWatch Logs\. Then you can view the specific error logs if the Lambda invocation or data delivery fails\. For more information, see [Monitoring Kinesis Data Firehose Using CloudWatch Logs](monitoring-with-cloudwatch-logs.md)\.  
**IAM role**  
You can choose to create a new role where required permissions are assigned automatically, or choose an existing role created for Kinesis Data Firehose\. The role is used to grant Kinesis Data Firehose access to your S3 bucket, AWS KMS key \(if data encryption is enabled\), and Lambda function \(if data transformation is enabled\)\. The console might create a role with placeholders\. You can safely ignore or safely delete lines with `%FIREHOSE_BUCKET_NAME%`, `%FIREHOSE_DEFAULT_FUNCTION%`, or `%FIREHOSE_DEFAULT_VERSION%`\. For more information, see [Grant Kinesis Data Firehose Access to an Amazon S3 Destination](controlling-access.md#using-iam-s3)\.

1. Review the settings and choose **Create Delivery Stream**\.

The new Kinesis Data Firehose delivery stream takes a few moments in the **Creating** state before it is available\. After your Kinesis Data Firehose delivery stream is in an **Active** state, you can start sending data to it from your producer\.