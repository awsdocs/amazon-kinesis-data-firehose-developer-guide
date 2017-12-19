# Writing to Amazon Kinesis Firehose Using Kinesis Streams<a name="writing-with-kinesis-streams"></a>

You can configure Kinesis Streams to send information to a Kinesis Firehose delivery stream\.

1. Open the Kinesis Firehose console at [https://console\.aws\.amazon\.com/firehose/](https://console.aws.amazon.com/firehose/)\.

1. Choose **Create Delivery Stream**\. On the **Name and source** page, enter values for the following fields:  
****Delivery stream name****  
The name of your Kinesis Firehose delivery stream\.  
****Source****  
Choose the **Kinesis stream** option to configure a Kinesis Firehose delivery stream that uses a Kinesis stream as a data source\. You can then use Amazon Kinesis Firehose to read data easily from an existing Kinesis stream and load it into destinations\.   
To use a Kinesis stream as a source, choose an existing stream in the **Kinesis stream** list, or choose **Create new** to create a new Kinesis stream\. After you create a new stream, choose **Refresh** to update the **Kinesis stream** list\. If you have a large number of streams, filter the list using **Filter by name**\.   
When you configure a Kinesis stream as the source of a Kinesis Firehose delivery stream, the Kinesis Firehose `PutRecord` and `PutRecordBatch` operations are disabled\. To add data to your Kinesis Firehose delivery stream in this case, use the Kinesis Streams `PutRecord` and `PutRecords` operations\.
Kinesis Firehose starts reading data from the `LATEST` position of your Kinesis stream\. For more information about Kinesis Streams positions, see [GetShardIterator](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetShardIterator.html)\. Kinesis Firehose calls the Kinesis Streams [GetRecords](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetRecords.html) operation once per second for each shard\.  
More than one Kinesis Firehose delivery stream can read from the same Kinesis stream\. Other Kinesis applications \(consumers\) can also read from the same stream\. Each call from any Kinesis Firehose delivery stream or other consumer application counts against the overall throttling limit for the shard\. To avoid getting throttled, plan your applications carefully\. For more information about Kinesis Streams limits, see [Amazon Kinesis Streams Limits](http://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html)\. 

1. Choose **Next** to advance to the [Transform records](create-transform.md) page\.