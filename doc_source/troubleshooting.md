# Troubleshooting Amazon Kinesis Data Firehose<a name="troubleshooting"></a>

You may not see data delivered to your specified destinations\. Use the troubleshooting steps in this topic to solve common issues you might encounter while using Kinesis Data Firehose\. For further troubleshooting, see [Monitoring with CloudWatch Logs](monitoring-with-cloudwatch-logs.md)\.

**Topics**
+ [Data Not Delivered to Amazon S3](#data-not-delivered-to-s3)
+ [Data Not Delivered to Amazon Redshift](#data-not-delivered-to-rs)
+ [Data Not Delivered to Amazon Elasticsearch Service](#data-not-delivered-to-es)
+ [Data Not Delivered to Splunk](#data-not-delivered-to-splunk)
+ [Delivery Stream Not Available as a Target for CloudWatch Logs, CloudWatch Events, or AWS IoT Action](#delivery-stream-not-available)

## Data Not Delivered to Amazon S3<a name="data-not-delivered-to-s3"></a>

Check the following if data is not delivered to your S3 bucket\.
+ Check the Kinesis Data Firehose **IncomingBytes** and **IncomingRecords** metrics to make sure that data is sent to your Kinesis data delivery stream successfully\. For more information, see [Monitoring with Amazon CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.
+ If data transformation with Lambda is enabled, check the Kinesis Data Firehose **ExecuteProcessingSuccess** metric to make sure that Kinesis Data Firehose has attempted to invoke your Lambda function\. For more information, see [Monitoring with Amazon CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.
+ Check the Kinesis Data Firehose **DeliveryToS3\.Success** metric to make sure that Kinesis Data Firehose has attempted putting data to your S3 bucket\. For more information, see [Monitoring with Amazon CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.
+ Enable error logging if it is not already enabled, and check error logs for delivery failure\. For more information, see [Monitoring with Amazon CloudWatch Logs](monitoring-with-cloudwatch-logs.md)\.
+ Make sure that the S3 bucket specified in your Kinesis data delivery stream still exists\.
+ If data transformation with Lambda is enabled, make sure that the Lambda function specified in your delivery stream still exists\.
+ Make sure that the IAM role specified in your Kinesis data delivery stream has access to your S3 bucket and your Lambda function \(if data transformation is enabled\)\. For more information, see [Grant Kinesis Data Firehose Access to an Amazon S3 Destination](controlling-access.md#using-iam-s3)\.
+ If you're using data transformation, make sure that your Lambda function never returns responses whose payload size exceeds 6 MB\. For more information, see [Amazon Kinesis Data Firehose Data Transformation](https://docs.aws.amazon.com/firehose/latest/dev/data-transformation.html)\.

## Data Not Delivered to Amazon Redshift<a name="data-not-delivered-to-rs"></a>

Check the following if data is not delivered to your Amazon Redshift cluster\.

Note that data is delivered to your S3 bucket before loading into Amazon Redshift If the data was not delivered to your S3 bucket, see [Data Not Delivered to Amazon S3](#data-not-delivered-to-s3)\.
+ Check the Kinesis Data Firehose **DeliveryToRedshift\.Success** metric to make sure that Kinesis Data Firehose has attempted to copy data from your S3 bucket to the Amazon Redshift cluster\. For more information, see [Monitoring with Amazon CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.
+ Enable error logging if it is not already enabled, and check error logs for delivery failure\. For more information, see [Monitoring with Amazon CloudWatch Logs](monitoring-with-cloudwatch-logs.md)\.
+ Check the Amazon Redshift STL\_CONNECTION\_LOG table to see if Kinesis Data Firehose is able to make successful connections\. In this table you should be able to see connections and their status based on a user name\. For more information, see [STL\_CONNECTION\_LOG](http://docs.aws.amazon.com/redshift/latest/dg/r_STL_CONNECTION_LOG.html) in the *Amazon Redshift Database Developer Guide*\.
+ If the previous check shows that connections are being established, check the Amazon Redshift STL\_LOAD\_ERRORS table to verify the reason of the COPY failure\. For more information, see [STL\_LOAD\_ERRORS](http://docs.aws.amazon.com/redshift/latest/dg/r_STL_LOAD_ERRORS.html) in the *Amazon Redshift Database Developer Guide*\.
+ Make sure that the Amazon Redshift configuration in your Kinesis data delivery stream is accurate and valid\.
+ Make sure that the IAM role specified in your Kinesis data delivery stream has access to the S3 bucket from which Amazon Redshift copies data and the Lambda function for data transformation \(if data transformation is enabled\)\. For more information, see [Grant Kinesis Data Firehose Access to an Amazon S3 Destination](controlling-access.md#using-iam-s3)\.
+ If your Amazon Redshift cluster is in a VPC, make sure that the cluster allows access from Kinesis Data Firehose IP addresses\. For more information, see [Grant Kinesis Data Firehose Access to an Amazon Redshift Destination ](controlling-access.md#using-iam-rs)\.
+ Make sure that the Amazon Redshift cluster is publicly accessible\.
+ If you're using data transformation, make sure that your Lambda function never returns responses whose payload size exceeds 6 MB\. For more information, see [Amazon Kinesis Data Firehose Data Transformation](https://docs.aws.amazon.com/firehose/latest/dev/data-transformation.html)\.

## Data Not Delivered to Amazon Elasticsearch Service<a name="data-not-delivered-to-es"></a>

Check the following if data is not delivered to your Elasticsearch domain\.

Note that data can be backed up to your S3 bucket concurrently\. If data was not delivered to your S3 bucket, see [Data Not Delivered to Amazon S3](#data-not-delivered-to-s3)\.
+ Check the Kinesis Data Firehose **IncomingBytes** and **IncomingRecords** metrics to make sure that data is sent to your Kinesis data delivery stream successfully\. For more information, see [Monitoring with Amazon CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.
+ If data transformation with Lambda is enabled, check the Kinesis Data Firehose **ExecuteProcessingSuccess** metric to make sure that Kinesis Data Firehose has attempted to invoke your Lambda function\. For more information, see [Monitoring with Amazon CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.
+ Check the Kinesis Data Firehose **DeliveryToElasticsearch\.Success** metric to make sure that Kinesis Data Firehose has attempted to index data to the Amazon ES cluster\. For more information, see [Monitoring with Amazon CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)\.
+ Enable error logging if it is not already enabled, and check error logs for delivery failure\. For more information, see [Monitoring with Amazon CloudWatch Logs](monitoring-with-cloudwatch-logs.md)\.
+ Make sure that the Amazon ES configuration in your delivery stream is accurate and valid\.
+ If data transformation with Lambda is enabled, make sure that the Lambda function specified in your delivery stream still exists\.
+ Make sure that the IAM role specified in your delivery stream has access to your Amazon ES cluster and Lambda function \(if data transformation is enabled\)\. For more information, see [Grant Kinesis Data Firehose Access to an Amazon ES Destination](controlling-access.md#using-iam-es)\.
+ If you're using data transformation, make sure that your Lambda function never returns responses whose payload size exceeds 6 MB\. For more information, see [Amazon Kinesis Data Firehose Data Transformation](https://docs.aws.amazon.com/firehose/latest/dev/data-transformation.html)\.

## Data Not Delivered to Splunk<a name="data-not-delivered-to-splunk"></a>

Check the following if data is not delivered to your Splunk endpoint\.
+ Review the Splunk platform requirements\. The Splunk Add\-on for Kinesis Data Firehose requires Splunk platform version 6\.6\.X or later\. For more information, see [Splunk Add\-on for Amazon Kinesis Firehose](http://docs.splunk.com/Documentation/AddOns/released/Firehose/Hardwareandsoftwarerequirements)\.
+ If you have a proxy \(Elastic Load Balancing or other\) between Kinesis Data Firehose and the HTTP Event Collector \(HEC\) node, be sure to enable sticky sessions in order to support HEC acknowledgements \(ACKs\)\.
+ Make sure that you are using a valid HEC token\.
+ Ensure that the HEC token is enabled\. See [Enable and disable Event Collector tokens]()\.
+ Check to see whether the data that you're sending to Splunk is formatted correctly\. See [Format events for HTTP Event Collector]()\.
+ Make sure that the HEC token and input event are configured with a valid index\.
+ When an upload to Splunk fails due to a server error from the HEC node, the request is automatically retried\. If all retries fail, the data gets backed up to Amazon S3\. Check to see if your data appears in Amazon S3, which is an indication of such a failure\.
+ Make sure that you've enabled indexer acknowledgment on your HEC token\. See [Enable indexer acknowledgement]()\.
+ Increase the value of `HECAcknowledgmentTimeoutInSeconds` in the Splunk destination configuration of your Kinesis Data Firehose delivery stream\.
+ Increase the value of `DurationInSeconds` under `RetryOptions`in the Splunk destination configuration of your Kinesis Data Firehose delivery stream\.
+ Check your HEC health\.
+ If you're using data transformation, make sure that your Lambda function never returns responses whose payload size exceeds 6 MB\. For more information, see [Amazon Kinesis Data Firehose Data Transformation](https://docs.aws.amazon.com/firehose/latest/dev/data-transformation.html)\.
+ Make sure that the Splunk parameter named `ackIdleCleanup` is set to `true`\. It's false by default\. To set this parameter to `true`, do the following:
  + For a [managed Splunk Cloud deployment](http://docs.splunk.com/Documentation/AddOns/released/Firehose/RequestFirehose), submit a case using the Splunk Support Portal\. In the case, ask Splunk Support to enable the HTTP event collector, set `ackIdleCleanup` to `true` in `inputs.conf`, and create or modify an elastic load balancer to use with this add\-on\.
  + For a [distributed Splunk Enterprise deployment](http://docs.splunk.com/Documentation/AddOns/released/Firehose/ConfigureHECdistributed), set the `ackIdleCleanup` parameter to true in the `inputs.conf` file, which is located under `$SPLUNK_HOME/etc/apps/splunk_httpinput/local/` for \*nix users and under `%SPLUNK_HOME%\etc\apps\splunk_httpinput\local\` for Windows users\.
  + For a [single\-instance Splunk Enterprise deployment](http://docs.splunk.com/Documentation/AddOns/released/Firehose/ConfigureHECsingle), set the `ackIdleCleanup` parameter to `true` in the `inputs.conf` file, which is located under `$SPLUNK_HOME/etc/apps/splunk_httpinput/local/` for \*nix users and under `%SPLUNK_HOME%\etc\apps\splunk_httpinput\local\` for Windows users\.
+ See [Troubleshoot the Splunk Add\-on for Amazon Kinesis Firehose](http://docs.splunk.com/Documentation/AddOns/released/Firehose/Troubleshoot)\.

## Delivery Stream Not Available as a Target for CloudWatch Logs, CloudWatch Events, or AWS IoT Action<a name="delivery-stream-not-available"></a>

Some AWS services can only send messages and events to a Kinesis Data Firehose delivery stream that is in the same Region\. Verify that your Kinesis Data Firehose delivery stream is located in the same Region as your other services\.