# Monitoring with Amazon CloudWatch Metrics<a name="monitoring-with-cloudwatch-metrics"></a>

Kinesis Data Firehose integrates with CloudWatch metrics so that you can collect, view, and analyze CloudWatch metrics for your Kinesis data delivery streams\. For example, you can monitor the `IncomingBytes` and `IncomingRecords` metrics to keep track of data ingested into Kinesis Data Firehose from data producers\.

The metrics that you configure for your Kinesis data delivery streams and agents are automatically collected and pushed to CloudWatch every five minutes\. Metrics are archived for two weeks; after that period, the data is discarded\.

The metrics collected for Kinesis data delivery streams are free of charge\. For information about Kinesis agent metrics, see [Monitoring Kinesis Agent Health](agent-health.md)\.

**Topics**
+ [Service\-level CloudWatch Metrics](#fh-metrics-cw)
+ [API\-Level CloudWatch Metrics](#fh-metrics-api-cw)
+ [Data Transformation CloudWatch Metrics](#fh-metrics-data-transformation)
+ [Format Conversion CloudWatch Metrics](#fh-metrics-format-conversion)
+ [Dimensions for Kinesis Data Firehose](#firehose-metric-dimensions)
+ [Accessing CloudWatch Metrics for Kinesis Data Firehose](#cloudwatch-metrics)

## Service\-level CloudWatch Metrics<a name="fh-metrics-cw"></a>

The `AWS/Firehose` namespace includes the following service\-level metrics\.


| Metric | Description | 
| --- | --- | 
| BackupToS3\.Bytes |  The number of bytes delivered to Amazon S3 for backup over the specified time period\. Kinesis Data Firehose emits this metric when data transformation is enabled for Amazon S3 or Amazon Redshift destinations\. Units: Bytes  | 
| BackupToS3\.DataFreshness |  Age \(from getting into Kinesis Data Firehose to now\) of the oldest record in Kinesis Data Firehose\. Any record older than this age has been delivered to the Amazon S3 bucket for backup\. Kinesis Data Firehose emits this metric when data transformation is enabled for Amazon S3 or Amazon Redshift destinations\. Units: Seconds  | 
| BackupToS3\.Records |  The number of records delivered to Amazon S3 for backup over the specified time period\. Kinesis Data Firehose emits this metric when data transformation is enabled for Amazon S3 or Amazon Redshift destinations\. Units: Count  | 
| BackupToS3\.Success |  Sum of successful Amazon S3 put commands for backup over sum of all Amazon S3 backup put commands\. Kinesis Data Firehose emits this metric when data transformation is enabled for Amazon S3 or Amazon Redshift destinations\.  | 
| DeliveryToElasticsearch\.Bytes |  The number of bytes indexed to Amazon ES over the specified time period\. Units: Bytes  | 
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
| DeliveryToSplunk\.DataFreshness |  Age \(from getting into Kinesis Firehose to now\) of the oldest record in Kinesis Firehose\. Any record older than this age has been delivered to Splunk\. Units: Seconds  | 
| DeliveryToSplunk\.Records |  The number of records delivered to Splunk over the specified time period\. Units: Count  | 
| DeliveryToSplunk\.Success |  The sum of the successfully indexed records over the sum of records that were attempted\.  | 
| IncomingBytes |  The number of bytes ingested into the Kinesis Data Firehose stream over the specified time period\. Units: Bytes  | 
| IncomingRecords |  The number of records ingested into the Kinesis Data Firehose stream over the specified time period\. Units: Count  | 

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
| UpdateDeliveryStream\.Latency |  The time taken per `UpdateDeliveryStream` operation, measured over the specified time period\. Units: Milliseconds  | 
| UpdateDeliveryStream\.Requests |  The total number of `UpdateDeliveryStream` requests\. Units: Count  | 

## Data Transformation CloudWatch Metrics<a name="fh-metrics-data-transformation"></a>

If data transformation with Lambda is enabled, the `AWS/Firehose` namespace includes the following metrics\.


| Metric | Description | 
| --- | --- | 
| ExecuteProcessing\.Duration |  The time it takes for each Lambda function invocation performed by Kinesis Firehose\. Units: Milliseconds  | 
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

## Dimensions for Kinesis Data Firehose<a name="firehose-metric-dimensions"></a>

To filter metrics by delivery stream, use the `DeliveryStreamName` dimension\.

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
Use the [list\-metrics](http://docs.aws.amazon.com/cli/latest/reference/cloudwatch/list-metrics.html) and [get\-metric\-statistics](http://docs.aws.amazon.com/cli/latest/reference/cloudwatch/get-metric-statistics.html) commands\.

```
aws cloudwatch list-metrics --namespace "AWS/Firehose"
```

```
aws cloudwatch get-metric-statistics --namespace "AWS/Firehose" \
--metric-name DescribeDeliveryStream.Latency --statistics Average --period 3600 \
--start-time 2017-06-01T00:00:00Z --endtime 2017-06-30T00:00:00Z
```
