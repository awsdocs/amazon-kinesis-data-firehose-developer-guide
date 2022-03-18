# Backup and Advanced Settings<a name="create-configure"></a>

This topic describes how to configure the backup and the advanced settings for your Kinesis Data Firehose delivery stream\.

## Backup Settings<a name="create-configure-backup"></a>

Kinesis Data Firehose uses Amazon S3 to backup all or failed only data that it attempts to deliver to your chosen destination\. You can specify the S3 backup settings for your Kinesis Data Firehose delivery stream if you made one of the following choices:
+ If you set Amazon S3 as the destination for your Kinesis Data Firehose delivery stream and you choose to specify an AWS Lambda function to transform data records or if you choose to convert data record formats for your delivery stream\.
+ If you set Amazon Redshift as the destination for your Kinesis Data Firehose delivery stream and you choose to specify an AWS Lambda function to transform data records\.
+ If you set any of the following services as the destination for your Kinesis Data Firehose delivery stream: Amazon OpenSearch Service, Datadog, Dynatrace, HTTP Endpoint, LogicMonitor, MongoDB Cloud, New Relic, Splunk, or Sumo Logic\.

The following are the backup settings for your Kinesis Data Firehose delivery stream:
+ Source record backup in Amazon S3 \- if S3 or Amazon Redshift is your selected destination, this setting indicates whether you want to enable source data backup or keep it disabled\. If any other supported service \(other than S3 or Amazon Redshift\) is set as your selected destination, then this setting indicates if you want to backup all your source data or failed data only\.
+ S3 backup bucket \- this is the S3 bucket where Kinesis Data Firehose backs up your data\.
+ S3 backup bucket prefix \- this is the prefix where Kinesis Data Firehose backs up your data\.
+ S3 backup bucket error output prefix \- all failed data is backed up in the this S3 bucket error output prefix\.
+ Buffer hints, compression and encryption for backup \- Kinesis Data Firehose uses Amazon S3 to backup all or failed only data that it attempts to deliver to your chosen destination\. Kinesis Data Firehose buffers incoming data before delivering it \(backing it up\) to Amazon S3\. You can choose a buffer size of 1–128 MiBs and a buffer interval of 60–900 seconds\. The condition that is satisfied first triggers data delivery to Amazon S3\. If you enable data transformation, the buffer interval applies from the time transformed data is received by Kinesis Data Firehose to the data delivery to Amazon S3\. If data delivery to the destination falls behind data writing to the delivery stream, Kinesis Data Firehose raises the buffer size dynamically to catch up\. This action helps ensure that all data is delivered to the destination\. 
+ S3 compressions and encryption \- choose GZIP, Snappy, Zip, or Hadoop\-Compatible Snappy data compression, or no data compression\. Snappy, Zip, and Hadoop\-Compatible Snappy compression is not available for delivery streams with Amazon Redshift as the destination\. 

  Kinesis Data Firehose supports Amazon S3 server\-side encryption with AWS Key Management Service \(AWS KMS\) for encrypting delivered data in Amazon S3\. You can choose to not encrypt the data or to encrypt with a key from the list of AWS KMS keys that you own\. For more information, see [Protecting Data Using Server\-Side Encryption with AWS KMS\-Managed Keys \(SSE\-KMS\)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html)\. 

## Advanced Settings<a name="create-configure-advanced"></a>

The following are the advanced settings for your Kinesis Data Firehose delivery stream:
+ Server\-side encryption \- Kinesis Data Firehose supports Amazon S3 server\-side encryption with AWS Key Management Service \(AWS KMS\) for encrypting delivered data in Amazon S3\. For more information, see [Protecting Data Using Server\-Side Encryption with AWS KMS–Managed Keys \(SSE\-KMS\)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html)\.
+ Error logging \- If data transformation is enabled, Kinesis Data Firehose can log the Lambda invocation, and send data delivery errors to CloudWatch Logs\. Then you can view the specific error logs if the Lambda invocation or data delivery fails\. For more information, see [Monitoring Kinesis Data Firehose Using CloudWatch Logs](https://docs.aws.amazon.com/firehose/latest/dev/monitoring-with-cloudwatch-logs.html)\.
+ Permissions \- Kinesis Data Firehose uses IAM roles for all the permissions that the delivery stream needs\. You can choose to create a new role where required permissions are assigned automatically, or choose an existing role created for Kinesis Data Firehose\. The role is used to grant Kinesis Data Firehose access to various services, including your S3 bucket, AWS KMS key \(if data encryption is enabled\), and Lambda function \(if data transformation is enabled\)\. The console might create a role with placeholders\. For more information, see [What is IAM?](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)\.
+ Tags \- You can add tags to organize your AWS resources, track costs, and control access\.

Once you've chosen your backup and advanced settings, review your choices, and then choose **Create delivery stream**\.

The new Kinesis Data Firehose delivery stream takes a few moments in the **Creating** state before it is available\. After your Kinesis Data Firehose delivery stream is in an **Active** state, you can start sending data to it from your producer\.