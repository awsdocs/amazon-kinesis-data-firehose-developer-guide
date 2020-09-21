# Troubleshooting Amazon Kinesis Data Firehose<a name="troubleshooting"></a>

If Kinesis Data Firehose encounters errors while delivering or processing data, it retries until the configured retry duration expires\. If the retry duration ends before the data is delivered successfully, Kinesis Data Firehose backs up the data to the configured S3 backup bucket\. If the destination is Amazon S3 and delivery fails or if delivery to the backup S3 bucket fails, Kinesis Data Firehose keeps retrying until the retention period ends\. For `DirectPut` delivery streams, Kinesis Data Firehose retains the records for 24 hours\. For a delivery stream whose data source is a Kinesis data stream, you can change the retention period as described in [Changing the Data Retention Period](https://docs.aws.amazon.com/streams/latest/dev/kinesis-extended-retention.html)\. 

If the data source is a Kinesis data stream, Kinesis Data Firehose retries the following operations indefinitely: `DescribeStream`, `GetRecords`, and `GetShardIterator`\.

If the delivery stream uses `DirectPut`, check the `IncomingBytes` and `IncomingRecords` metrics to see if there's incoming traffic\. If you are using the `PutRecord` or `PutRecordBatch`, make sure you catch exceptions and retry\. We recommend a retry policy with exponential back\-off with jitter and several retries\. Also, if you use the `PutRecordBatch` API, make sure your code checks the value of [FailedPutCount](https://docs.aws.amazon.com/firehose/latest/APIReference/API_PutRecordBatch.html#Firehose-PutRecordBatch-response-FailedPutCount) in the response even when the API call succeeds\.

If the delivery stream uses a Kinesis data stream as its source, check the `IncomingBytes` and `IncomingRecords` metrics for the source data stream\. Additionally, ensure that the `DataReadFromKinesisStream.Bytes` and `DataReadFromKinesisStream.Records` metrics are being emitted for the delivery stream\.

For information about tracking delivery errors using CloudWatch, see [Monitoring Kinesis Data Firehose Using CloudWatch Logs](monitoring-with-cloudwatch-logs.md)\.

**Topics**
+ [Data Not Delivered to Amazon S3](#data-not-delivered-to-s3)
+ [Data Not Delivered to Amazon Redshift](#data-not-delivered-to-rs)
+ [Data Not Delivered to Amazon Elasticsearch Service](#data-not-delivered-to-es)
+ [Data Not Delivered to Splunk](#data-not-delivered-to-splunk)
+ [Delivery Stream Not Available as a Target for CloudWatch Logs, CloudWatch Events, or AWS IoT Action](#delivery-stream-not-available)
+ [Data Freshness Metric Increasing or Not Emitted](#data-freshness-metric-not-emitted)
+ [Record Format Conversion to Apache Parquet Fails](#apache-parquet-conversion-fails)
+ [No Data at Destination Despite Good Metrics](#no-date-despite-good-metrics)
+ [Troubleshooting HTTP Endpoints](http_troubleshooting.md)

## Data Not Delivered to Amazon S3<a name="data-not-delivered-to-s3"></a>

Check the following if data is not delivered to your Amazon Simple Storage Service \(Amazon S3\) bucket\.
+ Check the Kinesis Data Firehose `IncomingBytes` and `IncomingRecords` metrics to make sure that data is sent to your Kinesis Data Firehose delivery stream successfully\. For more information, see [Monitoring Kinesis Data Firehose Using CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.
+ If data transformation with Lambda is enabled, check the Kinesis Data Firehose `ExecuteProcessingSuccess` metric to make sure that Kinesis Data Firehose has tried to invoke your Lambda function\. For more information, see [Monitoring Kinesis Data Firehose Using CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.
+ Check the Kinesis Data Firehose `DeliveryToS3.Success` metric to make sure that Kinesis Data Firehose has tried putting data to your Amazon S3 bucket\. For more information, see [Monitoring Kinesis Data Firehose Using CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.
+ Enable error logging if it is not already enabled, and check error logs for delivery failure\. For more information, see [Monitoring Kinesis Data Firehose Using CloudWatch Logs](monitoring-with-cloudwatch-logs.md)\.
+ Make sure that the Amazon S3 bucket that is specified in your Kinesis Data Firehose delivery stream still exists\.
+ If data transformation with Lambda is enabled, make sure that the Lambda function that is specified in your delivery stream still exists\.
+ Make sure that the IAM role that is specified in your Kinesis Data Firehose delivery stream has access to your S3 bucket and your Lambda function \(if data transformation is enabled\)\. For more information, see [Grant Kinesis Data Firehose Access to an Amazon S3 Destination](controlling-access.md#using-iam-s3)\.
+ If you're using data transformation, make sure that your Lambda function never returns responses whose payload size exceeds 6 MB\. For more information, see [Amazon Kinesis Data Firehose Data Transformation](https://docs.aws.amazon.com/firehose/latest/dev/data-transformation.html)\.

## Data Not Delivered to Amazon Redshift<a name="data-not-delivered-to-rs"></a>

Check the following if data is not delivered to your Amazon Redshift cluster\.

Data is delivered to your S3 bucket before loading into Amazon Redshift\. If the data was not delivered to your S3 bucket, see [Data Not Delivered to Amazon S3](#data-not-delivered-to-s3)\.
+ Check the Kinesis Data Firehose `DeliveryToRedshift.Success` metric to make sure that Kinesis Data Firehose has tried to copy data from your S3 bucket to the Amazon Redshift cluster\. For more information, see [Monitoring Kinesis Data Firehose Using CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.
+ Enable error logging if it is not already enabled, and check error logs for delivery failure\. For more information, see [Monitoring Kinesis Data Firehose Using CloudWatch Logs](monitoring-with-cloudwatch-logs.md)\.
+ Check the Amazon Redshift `STL_CONNECTION_LOG` table to see if Kinesis Data Firehose can make successful connections\. In this table, you should be able to see connections and their status based on a user name\. For more information, see [https://docs.aws.amazon.com/redshift/latest/dg/r_STL_CONNECTION_LOG.html](https://docs.aws.amazon.com/redshift/latest/dg/r_STL_CONNECTION_LOG.html) in the *Amazon Redshift Database Developer Guide*\.
+ If the previous check shows that connections are being established, check the Amazon Redshift `STL_LOAD_ERRORS` table to verify the reason for the COPY failure\. For more information, see [https://docs.aws.amazon.com/redshift/latest/dg/r_STL_LOAD_ERRORS.html](https://docs.aws.amazon.com/redshift/latest/dg/r_STL_LOAD_ERRORS.html) in the *Amazon Redshift Database Developer Guide*\.
+ Make sure that the Amazon Redshift configuration in your Kinesis Data Firehose delivery stream is accurate and valid\.
+ Make sure that the IAM role that is specified in your Kinesis Data Firehose delivery stream can access the S3 bucket that Amazon Redshift copies data from, and also the Lambda function for data transformation \(if data transformation is enabled\)\. For more information, see [Grant Kinesis Data Firehose Access to an Amazon S3 Destination](controlling-access.md#using-iam-s3)\.
+ If your Amazon Redshift cluster is in a virtual private cloud \(VPC\), make sure that the cluster allows access from Kinesis Data Firehose IP addresses\. For more information, see [Grant Kinesis Data Firehose Access to an Amazon Redshift Destination ](controlling-access.md#using-iam-rs)\.
+ Make sure that the Amazon Redshift cluster is publicly available\.
+ If you're using data transformation, make sure that your Lambda function never returns responses whose payload size exceeds 6 MB\. For more information, see [Amazon Kinesis Data Firehose Data Transformation](https://docs.aws.amazon.com/firehose/latest/dev/data-transformation.html)\.

## Data Not Delivered to Amazon Elasticsearch Service<a name="data-not-delivered-to-es"></a>

Check the following if data is not delivered to your Elasticsearch domain\.

Data can be backed up to your Amazon S3 bucket concurrently\. If data was not delivered to your S3 bucket, see [Data Not Delivered to Amazon S3](#data-not-delivered-to-s3)\.
+ Check the Kinesis Data Firehose `IncomingBytes` and `IncomingRecords` metrics to make sure that data is sent to your Kinesis Data Firehose delivery stream successfully\. For more information, see [Monitoring Kinesis Data Firehose Using CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.
+ If data transformation with Lambda is enabled, check the Kinesis Data Firehose `ExecuteProcessingSuccess` metric to make sure that Kinesis Data Firehose has tried to invoke your Lambda function\. For more information, see [Monitoring Kinesis Data Firehose Using CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.
+ Check the Kinesis Data Firehose `DeliveryToElasticsearch.Success` metric to make sure that Kinesis Data Firehose has tried to index data to the Amazon ES cluster\. For more information, see [Monitoring Kinesis Data Firehose Using CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.
+ Enable error logging if it is not already enabled, and check error logs for delivery failure\. For more information, see [Monitoring Kinesis Data Firehose Using CloudWatch Logs](monitoring-with-cloudwatch-logs.md)\.
+ Make sure that the Amazon ES configuration in your delivery stream is accurate and valid\.
+ If data transformation with Lambda is enabled, make sure that the Lambda function that is specified in your delivery stream still exists\.
+ Make sure that the IAM role that is specified in your delivery stream can access your Amazon ES cluster and Lambda function \(if data transformation is enabled\)\. For more information, see [Grant Kinesis Data Firehose Access to a Public Amazon ES Destination](controlling-access.md#using-iam-es)\.
+ If you're using data transformation, make sure that your Lambda function never returns responses whose payload size exceeds 6 MB\. For more information, see [Amazon Kinesis Data Firehose Data Transformation](https://docs.aws.amazon.com/firehose/latest/dev/data-transformation.html)\.

## Data Not Delivered to Splunk<a name="data-not-delivered-to-splunk"></a>

Check the following if data is not delivered to your Splunk endpoint\.
+ If your Splunk platform is in a VPC, make sure that Kinesis Data Firehose can access it\. For more information, see [Access to Splunk in VPC](https://docs.aws.amazon.com/firehose/latest/dev/controlling-access.html#using-iam-splunk-vpc)\.
+ If you use an AWS load balancer, make sure that it is a Classic Load Balancer\. Kinesis Data Firehose does not support Application Load Balancers or Network Load Balancers\. Also, enable duration\-based sticky sessions with cookie expiration disabled\. For information about how to do this, see [Duration\-Based Session Stickiness](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-sticky-sessions.html#enable-sticky-sessions-duration)\.
+ Review the Splunk platform requirements\. The Splunk add\-on for Kinesis Data Firehose requires Splunk platform version 6\.6\.X or later\. For more information, see [Splunk Add\-on for Amazon Kinesis Firehose](http://docs.splunk.com/Documentation/AddOns/released/Firehose/Hardwareandsoftwarerequirements)\.
+ If you have a proxy \(Elastic Load Balancing or other\) between Kinesis Data Firehose and the HTTP Event Collector \(HEC\) node, enable sticky sessions to support HEC acknowledgements \(ACKs\)\.
+ Make sure that you are using a valid HEC token\.
+ Ensure that the HEC token is enabled\. See [Enable and disable Event Collector tokens](http://docs.splunk.com/Documentation/SplunkCloud/7.0.0/Data/UsetheHTTPEventCollector#Enable_and_disable_Event_Collector_tokens)\.
+ Check whether the data that you're sending to Splunk is formatted correctly\. For more information, see [Format events for HTTP Event Collector](http://docs.splunk.com/Documentation/Splunk/7.0.3/Data/FormateventsforHTTPEventCollector)\.
+ Make sure that the HEC token and input event are configured with a valid index\.
+ When an upload to Splunk fails due to a server error from the HEC node, the request is automatically retried\. If all retries fail, the data gets backed up to Amazon S3\. Check if your data appears in Amazon S3, which is an indication of such a failure\.
+ Make sure that you enabled indexer acknowledgment on your HEC token\. For more information, see [Enable indexer acknowledgement](http://dev.splunk.com/view/event-collector/SP-CAAAE8X#enable)\.
+ Increase the value of `HECAcknowledgmentTimeoutInSeconds` in the Splunk destination configuration of your Kinesis Data Firehose delivery stream\.
+ Increase the value of `DurationInSeconds` under `RetryOptions` in the Splunk destination configuration of your Kinesis Data Firehose delivery stream\.
+ Check your HEC health\.
+ If you're using data transformation, make sure that your Lambda function never returns responses whose payload size exceeds 6 MB\. For more information, see [Amazon Kinesis Data Firehose Data Transformation](https://docs.aws.amazon.com/firehose/latest/dev/data-transformation.html)\.
+ Make sure that the Splunk parameter named `ackIdleCleanup` is set to `true`\. It is false by default\. To set this parameter to `true`, do the following:
  + For a [managed Splunk Cloud deployment](http://docs.splunk.com/Documentation/AddOns/released/Firehose/RequestFirehose), submit a case using the Splunk support portal\. In this case, ask Splunk support to enable the HTTP event collector, set `ackIdleCleanup` to `true` in `inputs.conf`, and create or modify a load balancer to use with this add\-on\.
  + For a [distributed Splunk Enterprise deployment](http://docs.splunk.com/Documentation/AddOns/released/Firehose/ConfigureHECdistributed), set the `ackIdleCleanup` parameter to true in the `inputs.conf` file\. For \*nix users, this file is located under `$SPLUNK_HOME/etc/apps/splunk_httpinput/local/`\. For Windows users, it is under `%SPLUNK_HOME%\etc\apps\splunk_httpinput\local\`\.
  + For a [single\-instance Splunk Enterprise deployment](http://docs.splunk.com/Documentation/AddOns/released/Firehose/ConfigureHECsingle), set the `ackIdleCleanup` parameter to `true` in the `inputs.conf` file\. For \*nix users, this file is located under `$SPLUNK_HOME/etc/apps/splunk_httpinput/local/`\. For Windows users, it is under `%SPLUNK_HOME%\etc\apps\splunk_httpinput\local\`\.
+ See [Troubleshoot the Splunk Add\-on for Amazon Kinesis Firehose](http://docs.splunk.com/Documentation/AddOns/released/Firehose/Troubleshoot)\.

## Delivery Stream Not Available as a Target for CloudWatch Logs, CloudWatch Events, or AWS IoT Action<a name="delivery-stream-not-available"></a>

Some AWS services can only send messages and events to a Kinesis Data Firehose delivery stream that is in the same AWS Region\. Verify that your Kinesis Data Firehose delivery stream is located in the same Region as your other services\.

## Data Freshness Metric Increasing or Not Emitted<a name="data-freshness-metric-not-emitted"></a>

Data freshness is a measure of how current your data is within your delivery stream\. It is the age of the oldest data record in the delivery stream, measured from the time that Kinesis Data Firehose ingested the data to the present time\. Kinesis Data Firehose provides metrics that you can use to monitor data freshness\. To identify the data\-freshness metric for a given destination, see [Monitoring Kinesis Data Firehose Using CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.

If you enable backup for all events or all documents, monitor two separate data\-freshness metrics: one for the main destination and one for the backup\. 

If the data\-freshness metric isn't being emitted, this means that there is no active delivery for the delivery stream\. This happens when data delivery is completely blocked or when there's no incoming data\.

If the data\-freshness metric is constantly increasing, this means that data delivery is falling behind\. This can happen for one of the following reasons\.
+ The destination can't handle the rate of delivery\. If Kinesis Data Firehose encounters transient errors due to high traffic, then the delivery might fall behind\. This can happen for destinations other than Amazon S3 \(it can happen for Amazon Elasticsearch Service, Amazon Redshift, or Splunk\)\. Ensure that your destination has enough capacity to handle the incoming traffic\.
+ The destination is slow\. Data delivery might fall behind if Kinesis Data Firehose encounters high latency\. Monitor the destination's latency metric\.
+ The Lambda function is slow\. This might lead to a data delivery rate that is less than the data ingestion rate for the delivery stream\. If possible, improve the efficiency of the Lambda function\. For instance, if the function does network IO, use multiple threads or asynchronous IO to increase parallelism\. Also, consider increasing the memory size of the Lambda function so that the CPU allocation can increase accordingly\. This might lead to faster Lambda invocations\. For information about configuring Lambda functions, see [Configuring AWS Lambda Functions](https://docs.aws.amazon.com/lambda/latest/dg/resource-model.html)\.
+ There are failures during data delivery\. For information about how to monitor errors using Amazon CloudWatch Logs, see [Monitoring Kinesis Data Firehose Using CloudWatch Logs](monitoring-with-cloudwatch-logs.md)\.
+ If the data source of the delivery stream is a Kinesis data stream, throttling might be happening\. Check the `ThrottledGetRecords`, `ThrottledGetShardIterator`, and `ThrottledDescribeStream` metrics\. If there are multiple consumers attached to the Kinesis data stream, consider the following:
  + If the `ThrottledGetRecords` and `ThrottledGetShardIterator` metrics are high, we recommend you increase the number of shards provisioned for the data stream\.
  + If the `ThrottledDescribeStream` is high, we recommend you add the `kinesis:listshards` permission to the role configured in [KinesisStreamSourceConfiguration](https://docs.aws.amazon.com/firehose/latest/APIReference/API_CreateDeliveryStream.html#Firehose-CreateDeliveryStream-request-KinesisStreamSourceConfiguration)\.
+ Low buffering hints for the destination\. This might increase the number of round trips that Kinesis Data Firehose needs to make to the destination, which might cause delivery to fall behind\. Consider increasing the value of the buffering hints\. For more information, see [BufferingHints](https://docs.aws.amazon.com/firehose/latest/APIReference/API_BufferingHints.html)\.
+ A high retry duration might cause delivery to fall behind when the errors are frequent\. Consider reducing the retry duration\. Also, monitor the errors and try to reduce them\. For information about how to monitor errors using Amazon CloudWatch Logs, see [Monitoring Kinesis Data Firehose Using CloudWatch Logs](monitoring-with-cloudwatch-logs.md)\.
+ If the destination is Splunk and `DeliveryToSplunk.DataFreshness` is high but `DeliveryToSplunk.Success` looks good, the Splunk cluster might be busy\. Free the Splunk cluster if possible\. Alternatively, contact AWS Support and request an increase in the number of channels that Kinesis Data Firehose is using to communicate with the Splunk cluster\.

## Record Format Conversion to Apache Parquet Fails<a name="apache-parquet-conversion-fails"></a>

This happens if you take DynamoDB data that includes the `Set` type, stream it through Lambda to a delivery stream, and use an AWS Glue Data Catalog to convert the record format to Apache Parquet\.

When the AWS Glue crawler indexes the DynamoDB set data types \(`StringSet`, `NumberSet`, and `BinarySet`\), it stores them in the data catalog as `SET<STRING>`, `SET<BIGINT>`, and `SET<BINARY>`, respectively\. However, for Kinesis Data Firehose to convert the data records to the Apache Parquet format, it requires Apache Hive data types\. Because the set types aren't valid Apache Hive data types, conversion fails\. To get conversion to work, update the data catalog with Apache Hive data types\. You can do that by changing `set` to `array` in the data catalog\.

**To change one or more data types from `set` to `array` in an AWS Glue data catalog**

1. Sign in to the AWS Management Console and open the AWS Glue console at [https://console\.aws\.amazon\.com/glue/](https://console.aws.amazon.com/glue/)\.

1. In the left pane, under the **Data catalog** heading, choose **Tables**\.

1. In the list of tables, choose the name of the table where you need to modify one or more data types\. This takes you to the details page for the table\.

1. Choose the **Edit schema** button in the top right corner of the details page\.

1. In the **Data type** column choose the first `set` data type\.

1. In the **Column type** drop\-down list, change the type from `set` to `array`\.

1. In the **ArraySchema** field, enter `array<string>`, `array<int>`, or `array<binary>`, depending on the appropriate type of data for your scenario\.

1. Choose **Update**\.

1. Repeat the previous steps to convert other `set` types to `array` types\.

1. Choose **Save**\.

## No Data at Destination Despite Good Metrics<a name="no-date-despite-good-metrics"></a>

If there are no data ingestion problems and the metrics emitted for the delivery stream look good, but you don't see the data at the destination, check the reader logic\. Make sure your reader is correctly parsing out all the data\.