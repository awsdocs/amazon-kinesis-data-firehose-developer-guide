# Data Protection in Amazon Kinesis Data Firehose<a name="encryption"></a>

If you have sensitive data, you can enable server\-side data encryption when you use Amazon Kinesis Data Firehose\. How you do this depends on the source of your data\.

## Server\-Side Encryption with Kinesis Data Streams as the Data Source<a name="sse-with-data-stream-as-source"></a>

When you configure a Kinesis data stream as the data source of a Kinesis Data Firehose delivery stream, Kinesis Data Firehose no longer stores the data at rest\. Instead, the data is stored in the data stream\. 

When you send data from your data producers to your data stream, Kinesis Data Streams encrypts your data using an AWS Key Management Service \(AWS KMS\) key before storing the data at rest\. When your Kinesis Data Firehose delivery stream reads the data from your data stream, Kinesis Data Streams first decrypts the data and then sends it to Kinesis Data Firehose\. Kinesis Data Firehose buffers the data in memory based on the buffering hints that you specify\. It then delivers it to your destinations without storing the unencrypted data at rest\.

For information about how to enable server\-side encryption for Kinesis Data Streams, see [Using Server\-Side Encryption](https://docs.aws.amazon.com/streams/latest/dev/server-side-encryption.html) in the *Amazon Kinesis Data Streams Developer Guide*\.

## Server\-Side Encryption with Direct PUT or Other Data Sources<a name="sse-with-direct-put"></a>

If you send data to your delivery stream using [PutRecord](https://docs.aws.amazon.com/firehose/latest/APIReference/API_PutRecord.html) or [PutRecordBatch](https://docs.aws.amazon.com/firehose/latest/APIReference/API_PutRecordBatch.html), or if you send the data using AWS IoT, Amazon CloudWatch Logs, or CloudWatch Events, you can turn on server\-side encryption by using the [StartDeliveryStreamEncryption](https://docs.aws.amazon.com/firehose/latest/APIReference/API_StartDeliveryStreamEncryption.html) operation\. 

To stop server\-side\-encryption, use the [StopDeliveryStreamEncryption](https://docs.aws.amazon.com/firehose/latest/APIReference/API_StopDeliveryStreamEncryption.html) operation\.