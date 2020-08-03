# Monitoring Kinesis Data Firehose Using CloudWatch Metrics<a name="monitoring-with-cloudwatch-metrics"></a>

Kinesis Data Firehose integrates with Amazon CloudWatch metrics so that you can collect, view, and analyze CloudWatch metrics for your Kinesis Data Firehose delivery streams\. For example, you can monitor the `IncomingBytes` and `IncomingRecords` metrics to keep track of data ingested into Kinesis Data Firehose from data producers\.

The metrics that you configure for your Kinesis Data Firehose delivery streams and agents are automatically collected and pushed to CloudWatch every five minutes\. Metrics are archived for two weeks; after that period, the data is discarded\.

The metrics collected for Kinesis Data Firehose delivery streams are free of charge\. For information about Kinesis agent metrics, see [Monitoring Kinesis Agent Health](agent-health.md)\.

**Topics**
+ [Data Delivery CloudWatch Metrics](#fh-metrics-cw)
+ [Data Ingestion Metrics](#fh-ingestion-metrics)
+ [API\-Level CloudWatch Metrics](#fh-metrics-api-cw)
+ [Data Transformation CloudWatch Metrics](#fh-metrics-data-transformation)
+ [Format Conversion CloudWatch Metrics](#fh-metrics-format-conversion)
+ [Server\-Side Encryption \(SSE\) CloudWatch Metrics](#fh-metrics-sse)
+ [Dimensions for Kinesis Data Firehose](#firehose-metric-dimensions)
+ [Kinesis Data Firehose Usage Metrics](#fh-metrics-usage)
+ [Accessing CloudWatch Metrics for Kinesis Data Firehose](#cloudwatch-metrics)
+ [Best Practices with CloudWatch Alarms](#firehose-cloudwatch-metrics-best-practices)
+ [Monitoring Kinesis Data Firehose Using CloudWatch Logs](monitoring-with-cloudwatch-logs.md)
+ [Monitoring Kinesis Agent Health](agent-health.md)
+ [Logging Kinesis Data Firehose API Calls with AWS CloudTrail](monitoring-using-cloudtrail.md)

## Data Delivery CloudWatch Metrics<a name="fh-metrics-cw"></a>

The `AWS/Firehose` namespace includes the following service\-level metrics\. If you see small drops in the average for `BackupToS3.Success`, `DeliveryToS3.Success`, `DeliveryToSplunk.Success`, `DeliveryToElasticsearch.Success`, or `DeliveryToRedshift.Success`, that doesn't indicate that there's data loss\. Kinesis Data Firehose retries delivery errors and doesn't move forward until the records are successfully delivered either to the configured destination or to the backup S3 bucket\. 

### Delivery to Amazon ES<a name="fh-es-metrics"></a>


| Metric | Description | 
| --- | --- | 
| DeliveryToElasticsearch\.Bytes |  The number of bytes indexed to Amazon ES over the specified time period\. Units: Bytes  | 
| DeliveryToElasticsearch\.DataFreshness |  The age \(from getting into Kinesis Data Firehose to now\) of the oldest record in Kinesis Data Firehose\. Any record older than this age has been delivered to Amazon ES\. Units: Seconds  | 
| DeliveryToElasticsearch\.Records |  The number of records indexed to Amazon ES over the specified time period\. Units: Count  | 
| DeliveryToElasticsearch\.Success |  The sum of the successfully indexed records over the sum of records that were attempted\.  | 
| DeliveryToS3\.Bytes |  The number of bytes delivered to Amazon S3 over the specified time period\. Kinesis Data Firehose emits this metric only when you enable backup for all documents\. Units: Count  | 
| DeliveryToS3\.DataFreshness |  The age \(from getting into Kinesis Data Firehose to now\) of the oldest record in Kinesis Data Firehose\. Any record older than this age has been delivered to the S3 bucket\. Kinesis Data Firehose emits this metric only when you enable backup for all documents\. Units: Seconds  | 
| DeliveryToS3\.Records |  The number of records delivered to Amazon S3 over the specified time period\. Kinesis Data Firehose emits this metric only when you enable backup for all documents\. Units: Count  | 
| DeliveryToS3\.Success |  The sum of successful Amazon S3 put commands over the sum of all Amazon S3 put commands\. Kinesis Data Firehose always emits this metric regardless of whether backup is enabled for failed documents only or for all documents\.  | 

### Delivery to Amazon Redshift<a name="fh-redshift-metrics"></a>


| Metric | Description | 
| --- | --- | 
| DeliveryToRedshift\.Bytes |  The number of bytes copied to Amazon Redshift over the specified time period\. Units: Count  | 
| DeliveryToRedshift\.Records |  The number of records copied to Amazon Redshift over the specified time period\. Units: Count  | 
| DeliveryToRedshift\.Success |  The sum of successful Amazon Redshift COPY commands over the sum of all Amazon Redshift COPY commands\.  | 
| DeliveryToS3\.Bytes |  The number of bytes delivered to Amazon S3 over the specified time period\. Units: Bytes  | 
| DeliveryToS3\.DataFreshness |  The age \(from getting into Kinesis Data Firehose to now\) of the oldest record in Kinesis Data Firehose\. Any record older than this age has been delivered to the S3 bucket\. Units: Seconds  | 
| DeliveryToS3\.Records |  The number of records delivered to Amazon S3 over the specified time period\. Units: Count  | 
| DeliveryToS3\.Success |  The sum of successful Amazon S3 put commands over the sum of all Amazon S3 put commands\.  | 
| BackupToS3\.Bytes |  The number of bytes delivered to Amazon S3 for backup over the specified time period\. Kinesis Data Firehose emits this metric when backup to Amazon S3 is enabled\. Units: Count  | 
| BackupToS3\.DataFreshness |  Age \(from getting into Kinesis Data Firehose to now\) of the oldest record in Kinesis Data Firehose\. Any record older than this age has been delivered to the Amazon S3 bucket for backup\. Kinesis Data Firehose emits this metric when backup to Amazon S3 is enabled\. Units: Seconds  | 
| BackupToS3\.Records |  The number of records delivered to Amazon S3 for backup over the specified time period\. Kinesis Data Firehose emits this metric when backup to Amazon S3 is enabled\. Units: Count  | 
| BackupToS3\.Success |  Sum of successful Amazon S3 put commands for backup over sum of all Amazon S3 backup put commands\. Kinesis Data Firehose emits this metric when backup to Amazon S3 is enabled\.  | 

### Delivery to Amazon S3<a name="fh-s3-metrics"></a>

The metrics in the following table are related to delivery to Amazon S3 when it is the main destination of the delivery stream\.


| Metric | Description | 
| --- | --- | 
| DeliveryToS3\.Bytes |  The number of bytes delivered to Amazon S3 over the specified time period\. Units: Bytes  | 
| DeliveryToS3\.DataFreshness |  The age \(from getting into Kinesis Data Firehose to now\) of the oldest record in Kinesis Data Firehose\. Any record older than this age has been delivered to the S3 bucket\. Units: Seconds  | 
| DeliveryToS3\.Records |  The number of records delivered to Amazon S3 over the specified time period\. Units: Count  | 
| DeliveryToS3\.Success |  The sum of successful Amazon S3 put commands over the sum of all Amazon S3 put commands\.  | 
| BackupToS3\.Bytes |  The number of bytes delivered to Amazon S3 for backup over the specified time period\. Kinesis Data Firehose emits this metric when backup is enabled \(which is only possible when data transformation is also enabled\)\. Units: Count  | 
| BackupToS3\.DataFreshness |  Age \(from getting into Kinesis Data Firehose to now\) of the oldest record in Kinesis Data Firehose\. Any record older than this age has been delivered to the Amazon S3 bucket for backup\. Kinesis Data Firehose emits this metric when backup is enabled \(which is only possible when data transformation is also enabled\)\. Units: Seconds  | 
| BackupToS3\.Records |  The number of records delivered to Amazon S3 for backup over the specified time period\. Kinesis Data Firehose emits this metric when backup is enabled \(which is only possible when data transformation is also enabled\)\. Units: Count  | 
| BackupToS3\.Success |  Sum of successful Amazon S3 put commands for backup over sum of all Amazon S3 backup put commands\. Kinesis Data Firehose emits this metric when backup is enabled \(which is only possible when data transformation is also enabled\)\.  | 

### Delivery to Splunk<a name="fh-splunk-metrics"></a>


| Metric | Description | 
| --- | --- | 
| DeliveryToSplunk\.Bytes |  The number of bytes delivered to Splunk over the specified time period\. Units: Bytes  | 
| DeliveryToSplunk\.DataAckLatency |  The approximate duration it takes to receive an acknowledgement from Splunk after Kinesis Data Firehose sends it data\. The increasing or decreasing trend for this metric is more useful than the absolute approximate value\. Increasing trends can indicate slower indexing and acknowledgement rates from Splunk indexers\. Units: Seconds  | 
| DeliveryToSplunk\.DataFreshness |  Age \(from getting into Kinesis Data Firehose to now\) of the oldest record in Kinesis Data Firehose\. Any record older than this age has been delivered to Splunk\. Units: Seconds  | 
| DeliveryToSplunk\.Records |  The number of records delivered to Splunk over the specified time period\. Units: Count  | 
| DeliveryToSplunk\.Success |  The sum of the successfully indexed records over the sum of records that were attempted\.  | 
| DeliveryToS3\.Success |  The sum of successful Amazon S3 put commands over the sum of all Amazon S3 put commands\. This metric is emitted when backup to Amazon S3 is enabled\.  | 
| BackupToS3\.Bytes |  The number of bytes delivered to Amazon S3 for backup over the specified time period\. Kinesis Data Firehose emits this metric when the delivery stream is configured to back up all documents\. Units: Count  | 
| BackupToS3\.DataFreshness |  Age \(from getting into Kinesis Data Firehose to now\) of the oldest record in Kinesis Data Firehose\. Any record older than this age has been delivered to the Amazon S3 bucket for backup\. Kinesis Data Firehose emits this metric when the delivery stream is configured to back up all documents\. Units: Seconds  | 
| BackupToS3\.Records |  The number of records delivered to Amazon S3 for backup over the specified time period\. Kinesis Data Firehose emits this metric when the delivery stream is configured to back up all documents\. Units: Count  | 
| BackupToS3\.Success |  Sum of successful Amazon S3 put commands for backup over sum of all Amazon S3 backup put commands\. Kinesis Data Firehose emits this metric when the delivery stream is configured to back up all documents\.  | 

### Delivery to HTTP Endpoints<a name="fh-http-metrics"></a>


| Metric | Description | 
| --- | --- | 
| DeliveryToHttpEndpoint\.Bytes |  The number of bytes delivered successfully to the HTTP endpoint\. Units: Bytes  | 
| DeliveryToHttpEndpoint\.Records |  The number of records delivered successfully to the HTTP endpoint\. Units: Counts  | 
| DeliveryToHttpEndpoint\.DataFreshness |  Age of the oldest record in Kinesis Data Firehose\. Units: Seconds  | 
| DeliveryToHttpEndpoint\.Success |  The sum of all successful data delivery requests to the HTTP endpoint Units: Count  | 
| DeliveryToHttpEndpoint\.ProcessedBytes |  The number of attempted processed bytes, including retries\.   | 
| DeliveryToHttpEndpoint\.ProcessedRecords |  The number of attempted records including retries\.  | 

## Data Ingestion Metrics<a name="fh-ingestion-metrics"></a>

### Data Ingestion Through Kinesis Data Streams<a name="fh-ingestion-kds-metrics"></a>


| Metric | Description | 
| --- | --- | 
| DataReadFromKinesisStream\.Bytes |  When the data source is a Kinesis data stream, this metric indicates the number of bytes read from that data stream\. This number includes rereads due to failovers\. Units: Bytes  | 
| DataReadFromKinesisStream\.Records |  When the data source is a Kinesis data stream, this metric indicates the number of records read from that data stream\. This number includes rereads due to failovers\. Units: Count  | 
| ThrottledDescribeStream |  The total number of times the `DescribeStream` operation is throttled when the data source is a Kinesis data stream\. Units: Count  | 
| ThrottledGetRecords |  The total number of times the `GetRecords` operation is throttled when the data source is a Kinesis data stream\. Units: Count  | 
| ThrottledGetShardIterator |  The total number of times the `GetShardIterator` operation is throttled when the data source is a Kinesis data stream\. Units: Count  | 

### Data Ingestion Through Direct PUT<a name="fh-ingestion-directput-metrics"></a>


| Metric | Description | 
| --- | --- | 
| BackupToS3\.Bytes |  The number of bytes delivered to Amazon S3 for backup over the specified time period\. Kinesis Data Firehose emits this metric when data transformation is enabled for Amazon S3 or Amazon Redshift destinations\. Units: Bytes  | 
| BackupToS3\.DataFreshness |  Age \(from getting into Kinesis Data Firehose to now\) of the oldest record in Kinesis Data Firehose\. Any record older than this age has been delivered to the Amazon S3 bucket for backup\. Kinesis Data Firehose emits this metric when data transformation is enabled for Amazon S3 or Amazon Redshift destinations\. Units: Seconds  | 
| BackupToS3\.Records |  The number of records delivered to Amazon S3 for backup over the specified time period\. Kinesis Data Firehose emits this metric when data transformation is enabled for Amazon S3 or Amazon Redshift destinations\. Units: Count  | 
| BackupToS3\.Success |  Sum of successful Amazon S3 put commands for backup over sum of all Amazon S3 backup put commands\. Kinesis Data Firehose emits this metric when data transformation is enabled for Amazon S3 or Amazon Redshift destinations\.  | 
| BytesPerSecondLimit | The current maximum number of bytes per second that a delivery stream can ingest before throttling\. To request an increase to this limit, go to the [AWS Support Center](https://console.aws.amazon.com/support/home) and choose Create case, then choose Service limit increase\. | 
| DataReadFromKinesisStream\.Bytes |  When the data source is a Kinesis data stream, this metric indicates the number of bytes read from that data stream\. This number includes rereads due to failovers\. Units: Bytes  | 
| DataReadFromKinesisStream\.Records |  When the data source is a Kinesis data stream, this metric indicates the number of records read from that data stream\. This number includes rereads due to failovers\. Units: Count  | 
| DeliveryToElasticsearch\.Bytes |  The number of bytes indexed to Amazon ES over the specified time period\. Units: Bytes  | 
| DeliveryToElasticsearch\.DataFreshness |  The age \(from getting into Kinesis Data Firehose to now\) of the oldest record in Kinesis Data Firehose\. Any record older than this age has been delivered to Amazon ES\. Units: Seconds  | 
| DeliveryToElasticsearch\.Records |  The number of records indexed to Amazon ES over the specified time period\. Units: Count  | 
| DeliveryToElasticsearch\.Success |  The sum of the successfully indexed records over the sum of records that were attempted\.  | 
| DeliveryToRedshift\.Bytes |  The number of bytes copied to Amazon Redshift over the specified time period\. Units: Bytes  | 
| DeliveryToRedshift\.Records |  The number of records copied to Amazon Redshift over the specified time period\. Units: Count  | 
| DeliveryToRedshift\.Success |  The sum of successful Amazon Redshift COPY commands over the sum of all Amazon Redshift COPY commands\.  | 
| DeliveryToS3\.Bytes |  The number of bytes delivered to Amazon S3 over the specified time period\. Units: Bytes  | 
| DeliveryToS3\.DataFreshness |  The age \(from getting into Kinesis Data Firehose to now\) of the oldest record in Kinesis Data Firehose\. Any record older than this age has been delivered to the S3 bucket\. Units: Seconds  | 
| DeliveryToS3\.Records |  The number of records delivered to Amazon S3 over the specified time period\. Units: Count  | 
| DeliveryToS3\.Success |  The sum of successful Amazon S3 put commands over the sum of all Amazon S3 put commands\.  | 
| DeliveryToSplunk\.Bytes |  The number of bytes delivered to Splunk over the specified time period\. Units: Bytes  | 
| DeliveryToSplunk\.DataAckLatency |  The approximate duration it takes to receive an acknowledgement from Splunk after Kinesis Data Firehose sends it data\. The increasing or decreasing trend for this metric is more useful than the absolute approximate value\. Increasing trends can indicate slower indexing and acknowledgement rates from Splunk indexers\. Units: Seconds  | 
| DeliveryToSplunk\.DataFreshness |  Age \(from getting into Kinesis Data Firehose to now\) of the oldest record in Kinesis Data Firehose\. Any record older than this age has been delivered to Splunk\. Units: Seconds  | 
| DeliveryToSplunk\.Records |  The number of records delivered to Splunk over the specified time period\. Units: Count  | 
| DeliveryToSplunk\.Success |  The sum of the successfully indexed records over the sum of records that were attempted\.  | 
| IncomingBytes |  The number of bytes ingested successfully into the delivery stream over the specified time period after throttling\. Units: Bytes  | 
| IncomingPutRequests | The number of successful PutRecord and PutRecordBatch requests over the specified period of time after throttling\. Units: Count | 
| IncomingRecords |  The number of records ingested successfully into the delivery stream over the specified time period after throttling\. Units: Count  | 
| KinesisMillisBehindLatest |  When the data source is a Kinesis data stream, this metric indicates the number of milliseconds that the last read record is behind the newest record in the Kinesis data stream\. Units: Millisecond  | 
| RecordsPerSecondLimit | The current maximum number of records per second that a delivery stream can ingest before throttling\. Units: Count | 
| ThrottledRecords | The number of records that were throttled because data ingestion exceeded one of the delivery stream limits\. Units: Count | 

## API\-Level CloudWatch Metrics<a name="fh-metrics-api-cw"></a>

The `AWS/Firehose` namespace includes the following API\-level metrics\.


| Metric | Description | 
| --- | --- | 
| DescribeDeliveryStream\.Latency |  The time taken per `DescribeDeliveryStream` operation, measured over the specified time period\. Units: Milliseconds  | 
| DescribeDeliveryStream\.Requests |  The total number of `DescribeDeliveryStream` requests\. Units: Count  | 
| ListDeliveryStreams\.Latency |  The time taken per `ListDeliveryStream` operation, measured over the specified time period\. Units: Milliseconds  | 
| ListDeliveryStreams\.Requests |  The total number of `ListFirehose` requests\. Units: Count  | 
| PutRecord\.Bytes |  The number of bytes put to the Kinesis Data Firehose delivery stream using `PutRecord` over the specified time period\. Units: Bytes  | 
| PutRecord\.Latency |  The time taken per `PutRecord` operation, measured over the specified time period\. Units: Milliseconds  | 
| PutRecord\.Requests |  The total number of `PutRecord` requests, which is equal to total number of records from `PutRecord` operations\. Units: Count  | 
| PutRecordBatch\.Bytes |  The number of bytes put to the Kinesis Data Firehose delivery stream using `PutRecordBatch` over the specified time period\. Units: Bytes  | 
| PutRecordBatch\.Latency |  The time taken per `PutRecordBatch` operation, measured over the specified time period\. Units: Milliseconds  | 
| PutRecordBatch\.Records |  The total number of records from `PutRecordBatch` operations\. Units: Count  | 
| PutRecordBatch\.Requests |  The total number of `PutRecordBatch` requests\. Units: Count  | 
| PutRequestsPerSecondLimit | The maximum number of put requests per second that a delivery stream can handle before throttling\. This number includes PutRecord and PutRecordBatch requests\. Units: Count | 
| ThrottledDescribeStream |  The total number of times the `DescribeStream` operation is throttled when the data source is a Kinesis data stream\. Units: Count  | 
| ThrottledGetRecords |  The total number of times the `GetRecords` operation is throttled when the data source is a Kinesis data stream\. Units: Count  | 
| ThrottledGetShardIterator |  The total number of times the `GetShardIterator` operation is throttled when the data source is a Kinesis data stream\. Units: Count  | 
| UpdateDeliveryStream\.Latency |  The time taken per `UpdateDeliveryStream` operation, measured over the specified time period\. Units: Milliseconds  | 
| UpdateDeliveryStream\.Requests |  The total number of `UpdateDeliveryStream` requests\. Units: Count  | 

## Data Transformation CloudWatch Metrics<a name="fh-metrics-data-transformation"></a>

If data transformation with Lambda is enabled, the `AWS/Firehose` namespace includes the following metrics\.


| Metric | Description | 
| --- | --- | 
| ExecuteProcessing\.Duration |  The time it takes for each Lambda function invocation performed by Kinesis Data Firehose\. Units: Milliseconds  | 
| ExecuteProcessing\.Success |  The sum of the successful Lambda function invocations over the sum of the total Lambda function invocations\.  | 
| SucceedProcessing\.Records |  The number of successfully processed records over the specified time period\. Units: Count  | 
| SucceedProcessing\.Bytes |  The number of successfully processed bytes over the specified time period\. Units: Bytes  | 

## Format Conversion CloudWatch Metrics<a name="fh-metrics-format-conversion"></a>

If format conversion is enabled, the `AWS/Firehose` namespace includes the following metrics\.


| Metric | Description | 
| --- | --- | 
| SucceedConversion\.Records |  The number of successfully converted records\. Units: Count  | 
| SucceedConversion\.Bytes |  The size of the successfully converted records\. Units: Bytes  | 
| FailedConversion\.Records |  The number of records that could not be converted\. Units: Count  | 
| FailedConversion\.Bytes |  The size of the records that could not be converted\. Units: Bytes  | 

## Server\-Side Encryption \(SSE\) CloudWatch Metrics<a name="fh-metrics-sse"></a>

The `AWS/Firehose` namespace includes the following metrics that are related to SSE\.


| Metric | Description | 
| --- | --- | 
| KMSKeyAccessDenied |  The number of times the service encounters a `KMSAccessDeniedException` for the delivery stream\. Units: Count  | 
| KMSKeyDisabled |  The number of times the service encounters a `KMSDisabledException` for the delivery stream\. Units: Count  | 
| KMSKeyInvalidState |  The number of times the service encounters a `KMSInvalidStateException` for the delivery stream\. Units: Count  | 
| KMSKeyNotFound |  The number of times the service encounters a `KMSNotFoundException` for the delivery stream\. Units: Count  | 

## Dimensions for Kinesis Data Firehose<a name="firehose-metric-dimensions"></a>

To filter metrics by delivery stream, use the `DeliveryStreamName` dimension\.

## Kinesis Data Firehose Usage Metrics<a name="fh-metrics-usage"></a>

You can use CloudWatch usage metrics to provide visibility into your account's usage of resources\. Use these metrics to visualize your current service usage on CloudWatch graphs and dashboards\. 

Service quota usage metrics are in the AWS/Usage namespace and are collected every minute\. 

Currently, the only metric name in this namespace that CloudWatch publishes is `ResourceCount`\. This metric is published with the dimensions `Service`, `Class`, `Type`, and `Resource`\.


| Metric | Description | 
| --- | --- | 
| ResourceCount |  The number of the specified resources running in your account\. The resources are defined by the dimensions associated with the metric\.  The most useful statistic for this metric is MAXIMUM, which represents the maximum number of resources used during the 1\-minute period\.   | 

The following dimensions are used to refine the usage metrics that are published by Kinesis Data Firehose\. 


| Dimension | Description | 
| --- | --- | 
| Service |  The name of the AWS service containing the resource\. For Kinesis Data Firehose usage metrics, the value for this dimension is `Firehose`\.   | 
| Class |  The class of resource being tracked\. Kinesis Data Firehose API usage metrics use this dimension with a value of `None`\.   | 
| Type |  The type of resource being tracked\. Currently, when the Service dimension is `Firehose`, the only valid value for Type is `Resource`\.   | 
| Resource |  The name of the AWS resource\. Currently, when the Service dimension is `Firehose`, the only valid value for Resource is `DeliveryStreams`\.   | 

## Accessing CloudWatch Metrics for Kinesis Data Firehose<a name="cloudwatch-metrics"></a>

You can monitor metrics for Kinesis Data Firehose using the CloudWatch console, command line, or CloudWatch API\. The following procedures show you how to access metrics using these different methods\. 

**To access metrics using the CloudWatch console**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. On the navigation bar, choose a region\.

1. In the navigation pane, choose **Metrics**\.

1. Choose the **Firehose** namespace\.

1. Choose **Delivery Stream Metrics** or **Firehose Metrics**\.

1. Select a metric to add to the graph\.

**To access metrics using the AWS CLI**  
Use the [list\-metrics](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/list-metrics.html) and [get\-metric\-statistics](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/get-metric-statistics.html) commands\.

```
aws cloudwatch list-metrics --namespace "AWS/Firehose"
```

```
aws cloudwatch get-metric-statistics --namespace "AWS/Firehose" \
--metric-name DescribeDeliveryStream.Latency --statistics Average --period 3600 \
--start-time 2017-06-01T00:00:00Z --end-time 2017-06-30T00:00:00Z
```

## Best Practices with CloudWatch Alarms<a name="firehose-cloudwatch-metrics-best-practices"></a>

Add CloudWatch alarms for when the following metrics exceed the buffering limit \(a maximum of 15 minutes\): 
+ `DeliveryToS3.DataFreshness`
+ `DeliveryToSplunk.DataFreshness`
+ `DeliveryToElasticsearch.DataFreshness`

Also, create alarms based on the following metric math expressions\.
+ `IncomingBytes (Sum per 5 Minutes) / 300` approaches a percentage of `BytesPerSecondLimit`\.
+ `IncomingRecords (Sum per 5 Minutes) / 300` approaches a percentage of `RecordsPerSecondLimit`\.
+ `IncomingPutRequests (Sum per 5 Minutes) / 300` approaches a percentage of `PutRequestsPerSecondLimit`\.

Another metric for which we recommend an alarm is `ThrottledRecords`\.

For information about troubleshooting when alarms go to the `ALARM` state, see [Troubleshooting Amazon Kinesis Data Firehose](troubleshooting.md)\.