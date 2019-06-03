# Writing to Kinesis Data Firehose Using Kinesis Data Streams<a name="writing-with-kinesis-streams"></a>

You can configure Amazon Kinesis Data Streams to send information to a Kinesis Data Firehose delivery stream\.

**Important**  
If you use the Kinesis Producer Library \(KPL\) to write data to a Kinesis data stream, you can use aggregation to combine the records that you write to that Kinesis data stream\. If you then use that data stream as a source for your Kinesis Data Firehose delivery stream, Kinesis Data Firehose de\-aggregates the records before it delivers them to the destination\. If you configure your delivery stream to transform the data, Kinesis Data Firehose de\-aggregates the records before it delivers them to AWS Lambda\. For more information, see [Developing Amazon Kinesis Data Streams Producers Using the Kinesis Producer Library](https://docs.aws.amazon.com/streams/latest/dev/developing-producers-with-kpl.html) and [Aggregation](https://docs.aws.amazon.com/streams/latest/dev/kinesis-kpl-concepts.html#kinesis-kpl-concepts-aggretation)\.

1. Sign in to the AWS Management Console and open the Kinesis Data Firehose console at [https://console\.aws\.amazon\.com/firehose/](https://console.aws.amazon.com/firehose/)\.

1. Choose **Create Delivery Stream**\. On the **Name and source** page, provide values for the following fields:  
****Delivery stream name****  
The name of your Kinesis Data Firehose delivery stream\.  
****Source****  
Choose **Kinesis stream** to configure a Kinesis Data Firehose delivery stream that uses a Kinesis data stream as a data source\. You can then use Kinesis Data Firehose to read data easily from an existing data stream and load it into destinations\.   
To use a Kinesis data stream as a source, choose an existing stream in the **Kinesis stream** list, or choose **Create new** to create a new Kinesis data stream\. After you create a new stream, choose **Refresh** to update the **Kinesis stream** list\. If you have a large number of streams, filter the list using **Filter by name**\.   
When you configure a Kinesis data stream as the source of a Kinesis Data Firehose delivery stream, the Kinesis Data Firehose `PutRecord` and `PutRecordBatch` operations are disabled\. To add data to your Kinesis Data Firehose delivery stream in this case, use the Kinesis Data Streams `PutRecord` and `PutRecords` operations\.
Kinesis Data Firehose starts reading data from the `LATEST` position of your Kinesis stream\. For more information about Kinesis Data Streams positions, see [GetShardIterator](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetShardIterator.html)\. Kinesis Data Firehose calls the Kinesis Data Streams [GetRecords](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetRecords.html) operation once per second for each shard\.  
More than one Kinesis Data Firehose delivery stream can read from the same Kinesis stream\. Other Kinesis applications \(consumers\) can also read from the same stream\. Each call from any Kinesis Data Firehose delivery stream or other consumer application counts against the overall throttling limit for the shard\. To avoid getting throttled, plan your applications carefully\. For more information about Kinesis Data Streams limits, see [Amazon Kinesis Streams Limits](https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html)\. 

1. Choose **Next** to advance to the [Process records](create-transform.md) page\.