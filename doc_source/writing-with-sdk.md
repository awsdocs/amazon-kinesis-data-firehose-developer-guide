# Writing to a Kinesis Firehose Delivery Stream Using the AWS SDK<a name="writing-with-sdk"></a>

You can use the [Kinesis Firehose API](http://docs.aws.amazon.com/firehose/latest/APIReference/) to send data to a Kinesis Firehose delivery stream using the [AWS SDK for Java](https://aws.amazon.com/developers/getting-started/java/), [\.NET](https://aws.amazon.com/developers/getting-started/net/), [Node\.js](https://aws.amazon.com/developers/getting-started/nodejs/), [Python](https://aws.amazon.com/developers/getting-started/python/), or [Ruby](https://aws.amazon.com/developers/getting-started/ruby/)\. If you are new to Kinesis Firehose, take some time to become familiar with the concepts and terminology presented in [What Is Amazon Kinesis Firehose?](what-is-this-service.md)\. For more information, see [Start Developing with Amazon Web Services](http://aws.amazon.com/developers/getting-started/)\.

These examples do not represent production\-ready code, in that they do not check for all possible exceptions, or account for all possible security or performance considerations\. 

The Kinesis Firehose API offers two operations for sending data to your Kinesis Firehose delivery stream: [PutRecord](http://docs.aws.amazon.com/firehose/latest/APIReference/API_PutRecord.html) and [PutRecordBatch](http://docs.aws.amazon.com/firehose/latest/APIReference/API_PutRecordBatch.html)\. `PutRecord()` sends one data record within one call and `PutRecordBatch()` can send multiple data records within one call\. 


+ [Single Write Operations Using PutRecord](#putrecord)
+ [Batch Write Operations Using PutRecordBatch](#putrecordbatch)

## Single Write Operations Using PutRecord<a name="putrecord"></a>

Putting data requires only the Kinesis Firehose delivery stream name and a byte buffer \(<=1000 KB\)\. Because Kinesis Firehose batches multiple records before loading the file into Amazon S3, you may want to add a record separator\. To put data one record at a time into a Kinesis Firehose delivery stream, use the following code:

```
PutRecordRequest putRecordRequest = new PutRecordRequest();
putRecordRequest.setDeliveryStreamName(deliveryStreamName);

String data = line + "\n";

Record record = createRecord(data);
putRecordRequest.setRecord(record);

// Put record into the DeliveryStream
firehoseClient.putRecord(putRecordRequest);
```

For more code context, see the sample code included in the AWS SDK\. For information about request and response syntax, see the relevant topic in [Amazon Kinesis Firehose API Operations](http://docs.aws.amazon.com/firehose/latest/APIReference/API_Operations.html)\.

## Batch Write Operations Using PutRecordBatch<a name="putrecordbatch"></a>

Putting data requires only the Kinesis Firehose delivery stream name and a list of records\. Because Kinesis Firehose batches multiple records before loading the file into Amazon S3, you may want to add a record separator\. To put data records in batches into a Kinesis Firehose delivery stream, use the following code:

```
PutRecordBatchRequest putRecordBatchRequest = new PutRecordBatchRequest();
putRecordBatchRequest.setDeliveryStreamName(deliveryStreamName);
putRecordBatchRequest.setRecords(recordList);

// Put Record Batch records. Max No.Of Records we can put in a
// single put record batch request is 500
firehoseClient.putRecordBatch(putRecordBatchRequest);

recordList.clear();
```

For more code context, see the sample code included in the AWS SDK\. For information about request and response syntax, see the relevant topic in [Amazon Kinesis Firehose API Operations](http://docs.aws.amazon.com/firehose/latest/APIReference/API_Operations.html)\.