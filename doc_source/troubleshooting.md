# Troubleshooting Amazon Kinesis Data Firehose<a name="troubleshooting"></a>

To monitor the freshness of your data delivery, check the `DataFreshness` metric under the **Monitoring** tab in the Kinesis Data Firehose console\. `DataFreshness` indicates how current your data is within your Kinesis Data Firehose delivery stream\. If the value of `DataFreshness` doesn't increase over time, this means that your delivery stream is successfully delivering your data\. When Kinesis Data Firehose encounters an error, it uses Amazon S3 to back up all data that it can't deliver to your primary destination\. If you enable CloudWatch Logs for your delivery stream, you can see all delivery errors\. Kinesis Data Firehose automatically retries all failed deliveries until the configured retry duration expires\. For more information, see [Monitoring with CloudWatch Logs](monitoring-with-cloudwatch-logs.md)\.

**Topics**
+ [Data Not Delivered to Amazon S3](#data-not-delivered-to-s3)
+ [Data Not Delivered to Amazon Redshift](#data-not-delivered-to-rs)
+ [Data Not Delivered to Amazon Elasticsearch Service](#data-not-delivered-to-es)
+ [Data Not Delivered to Splunk](#data-not-delivered-to-splunk)
+ [Delivery Stream Not Available as a Target for CloudWatch Logs, CloudWatch Events, or AWS IoT Action](#delivery-stream-not-available)

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
+ Make sure that the IAM role that is specified in your delivery stream can access your Amazon ES cluster and Lambda function \(if data transformation is enabled\)\. For more information, see [Grant Kinesis Data Firehose Access to an Amazon ES Destination](controlling-access.md#using-iam-es)\.
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