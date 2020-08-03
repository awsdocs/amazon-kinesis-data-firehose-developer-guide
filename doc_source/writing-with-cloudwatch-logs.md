# Writing to Kinesis Data Firehose Using CloudWatch Logs<a name="writing-with-cloudwatch-logs"></a>

For information about how to create a CloudWatch Logs subscription that sends log events to Kinesis Data Firehose, see [Subscription Filters with Amazon Kinesis Firehose](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/SubscriptionFilters.html#FirehoseExample)\. 

**Important**  
CloudWatch log events are compressed with gzip level 6\. If you want to specify Amazon ES or Splunk as the destination for the delivery stream, use a Lambda function to uncompress the records to UTF\-8\.and single\-line JSON\. For information about using Lambda functions with a delivery stream, see [Amazon Kinesis Data Firehose Data Transformation](data-transformation.md)\.