# Using Server\-Side Encryption with Amazon Kinesis Firehose<a name="encryption"></a>

If you have sensitive data, you can enable server\-side data encryption when you use Amazon Kinesis Firehose\. However, this is only possible if you use a Kinesis stream as your data source\. When you configure a Kinesis stream as the data source of a Kinesis Firehose delivery stream, Kinesis Firehose no longer stores the data at rest\. Instead, the data is stored in the Kinesis stream\. 

When you send data from your data producers to your Kinesis stream, the Kinesis Streams service encrypts your data using an AWS KMS key before storing it at rest\. When your Kinesis Firehose delivery stream reads the data from your Kinesis stream, the Kinesis Streams service first decrypts the data and then sends it to Kinesis Firehose\. Kinesis Firehose buffers the data in memory based on the buffering hints that you specify and then delivers it to your destinations without storing the unencrypted data at rest\.

For information about how to enable server\-side encryption for Kinesis Streams, see [Using Server\-Side Encryption](http://docs.aws.amazon.com/streams/latest/dev/server-side-encryption.html)\.