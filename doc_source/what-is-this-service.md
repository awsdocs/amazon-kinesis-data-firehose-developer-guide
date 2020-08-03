# What Is Amazon Kinesis Data Firehose?<a name="what-is-this-service"></a>

Amazon Kinesis Data Firehose is a fully managed service for delivering real\-time [streaming data](http://aws.amazon.com/streaming-data/) to destinations such as Amazon Simple Storage Service \(Amazon S3\), Amazon Redshift, Amazon Elasticsearch Service \(Amazon ES\), Splunk, and any custom HTTP endpoint or HTTP endpoints owned by supported third\-party service providers, including Datadog, MongoDB, and New Relic\. Kinesis Data Firehose is part of the Kinesis streaming data platform, along with [Kinesis Data Streams](https://docs.aws.amazon.com/kinesis/latest/dev/), [Kinesis Video Streams](https://docs.aws.amazon.com/kinesisvideostreams/latest/dg/), and [Amazon Kinesis Data Analytics](https://docs.aws.amazon.com/kinesisanalytics/latest/dev/)\. With Kinesis Data Firehose, you don't need to write applications or manage resources\. You configure your data producers to send data to Kinesis Data Firehose, and it automatically delivers the data to the destination that you specified\. You can also configure Kinesis Data Firehose to transform your data before delivering it\.

For more information about AWS big data solutions, see [Big Data on AWS](http://aws.amazon.com/big-data/)\. For more information about AWS streaming data solutions, see [What is Streaming Data?](http://aws.amazon.com/streaming-data/)

## Key Concepts<a name="key-concepts"></a>

As you get started with Kinesis Data Firehose, you can benefit from understanding the following concepts:

**Kinesis Data Firehose delivery stream**  
The underlying entity of Kinesis Data Firehose\. You use Kinesis Data Firehose by creating a Kinesis Data Firehose delivery stream and then sending data to it\. For more information, see [Creating an Amazon Kinesis Data Firehose Delivery Stream](basic-create.md) and [Sending Data to an Amazon Kinesis Data Firehose Delivery Stream](basic-write.md)\.

**record**  
The data of interest that your data producer sends to a Kinesis Data Firehose delivery stream\. A record can be as large as 1,000 KB\.

**data producer**  
Producers send records to Kinesis Data Firehose delivery streams\. For example, a web server that sends log data to a delivery stream is a data producer\. You can also configure your Kinesis Data Firehose delivery stream to automatically read data from an existing Kinesis data stream, and load it into destinations\. For more information, see [Sending Data to an Amazon Kinesis Data Firehose Delivery Stream](basic-write.md)\.

**buffer size and buffer interval**  
Kinesis Data Firehose buffers incoming streaming data to a certain size or for a certain period of time before delivering it to destinations\. Buffer Size is in MBs and Buffer Interval is in seconds\.

## Data Flow<a name="data-flow-diagrams"></a>

For Amazon S3 destinations, streaming data is delivered to your S3 bucket\. If data transformation is enabled, you can optionally back up source data to another Amazon S3 bucket\.

![\[Amazon Kinesis Data Firehose data flow for Amazon S3\]](http://docs.aws.amazon.com/firehose/latest/dev/images/fh-flow-s3.png)

For Amazon Redshift destinations, streaming data is delivered to your S3 bucket first\. Kinesis Data Firehose then issues an Amazon Redshift COPY command to load data from your S3 bucket to your Amazon Redshift cluster\. If data transformation is enabled, you can optionally back up source data to another Amazon S3 bucket\.

![\[Amazon Kinesis Data Firehose data flow for Amazon Redshift\]](http://docs.aws.amazon.com/firehose/latest/dev/images/fh-flow-rs.png)

For Amazon ES destinations, streaming data is delivered to your Amazon ES cluster, and it can optionally be backed up to your S3 bucket concurrently\.

![\[Amazon Kinesis Data Firehose data flow for Amazon ES\]](http://docs.aws.amazon.com/firehose/latest/dev/images/fh-flow-es.png)

For Splunk destinations, streaming data is delivered to Splunk, and it can optionally be backed up to your S3 bucket concurrently\. 

![\[Amazon Kinesis Data Firehose data flow for Splunk\]](http://docs.aws.amazon.com/firehose/latest/dev/images/fh-flow-splunk.png)