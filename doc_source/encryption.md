# Data Protection in Amazon Kinesis Data Firehose<a name="encryption"></a>

If you have sensitive data, you can enable server\-side data encryption when you use Amazon Kinesis Data Firehose\. How you do this depends on the source of your data\.

## Server\-Side Encryption with Kinesis Data Streams as the Data Source<a name="sse-with-data-stream-as-source"></a>

When you configure a Kinesis data stream as the data source of a Kinesis Data Firehose delivery stream, Kinesis Data Firehose no longer stores the data at rest\. Instead, the data is stored in the data stream\. 

When you send data from your data producers to your data stream, Kinesis Data Streams encrypts your data using an AWS Key Management Service \(AWS KMS\) key before storing the data at rest\. When your Kinesis Data Firehose delivery stream reads the data from your data stream, Kinesis Data Streams first decrypts the data and then sends it to Kinesis Data Firehose\. Kinesis Data Firehose buffers the data in memory based on the buffering hints that you specify\. It then delivers it to your destinations without storing the unencrypted data at rest\.

For information about how to enable server\-side encryption for Kinesis Data Streams, see [Using Server\-Side Encryption](https://docs.aws.amazon.com/streams/latest/dev/server-side-encryption.html) in the *Amazon Kinesis Data Streams Developer Guide*\.

## Server\-Side Encryption with Direct PUT or Other Data Sources<a name="sse-with-direct-put"></a>

If you send data to your delivery stream using [PutRecord](https://docs.aws.amazon.com/firehose/latest/APIReference/API_PutRecord.html) or [PutRecordBatch](https://docs.aws.amazon.com/firehose/latest/APIReference/API_PutRecordBatch.html), or if you send the data using AWS IoT, Amazon CloudWatch Logs, or CloudWatch Events, you can turn on server\-side encryption by using the [StartDeliveryStreamEncryption](https://docs.aws.amazon.com/firehose/latest/APIReference/API_StartDeliveryStreamEncryption.html) operation\. 

To stop server\-side\-encryption, use the [StopDeliveryStreamEncryption](https://docs.aws.amazon.com/firehose/latest/APIReference/API_StopDeliveryStreamEncryption.html) operation\.

You can also enable SSE when you create the delivery stream\. To do that, specify [DeliveryStreamEncryptionConfigurationInput](https://docs.aws.amazon.com/firehose/latest/APIReference/API_DeliveryStreamEncryptionConfigurationInput.html) when you invoke [CreateDeliveryStream](https://docs.aws.amazon.com/firehose/latest/APIReference/API_CreateDeliveryStream.html)\.

When the CMK is of type `CUSTOMER_MANAGED_CMK`, if the Amazon Kinesis Data Firehose service is unable to decrypt records because of a `KMSNotFoundException`, a `KMSInvalidStateException`, a `KMSDisabledException`, or a `KMSAccessDeniedException`, the service waits up to 24 hours \(the retention period\) for you to resolve the problem\. If the problem persists beyond the retention period, the service skips those records that have passed the retention period and couldn't be decrypted, and then discards the data\. Amazon Kinesis Data Firehose provides the following four CloudWatch metrics that you can use to track the four AWS KMS exceptions:
+ `KMSKeyAccessDenied`
+ `KMSKeyDisabled`
+ `KMSKeyInvalidState`
+ `KMSKeyNotFound`

For more information about these four metrics, see [Monitoring Kinesis Data Firehose Using CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.

**Important**  
To encrypt your delivery stream, use symmetric CMKs\. Kinesis Data Firehose doesn't support asymmetric CMKs\. For information about symmetric and asymmetric CMKs, see [About Symmetric and Asymmetric CMKs](https://docs.aws.amazon.com/kms/latest/developerguide/symm-asymm-concepts.html) in the AWS Key Management Service developer guide\.