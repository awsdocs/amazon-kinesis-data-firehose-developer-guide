# Sending Data to an Amazon Kinesis Firehose Delivery Stream<a name="basic-write"></a>

You can send data to your Kinesis Firehose delivery stream using different types of sources: You can use a Kinesis stream, the Kinesis Agent, or the Kinesis Firehose API using the AWS SDK\. You can also use Amazon CloudWatch Logs, CloudWatch Events, or AWS IoT as your data source\. If you are new to Kinesis Firehose, take some time to become familiar with the concepts and terminology presented in [What Is Amazon Kinesis Firehose?](what-is-this-service.md)\.

**Note**  
Some AWS services can only send messages and events to a Kinesis Firehose delivery stream that is in the same Region\. If your Kinesis Firehose delivery stream doesn't appear as an option when you're configuring a target for Amazon CloudWatch Logs, CloudWatch Events, or AWS IoT, verify that your Kinesis Firehose delivery stream is in the same Region as your other services\.


+ [Writing to Amazon Kinesis Firehose Using Kinesis Streams](writing-with-kinesis-streams.md)
+ [Writing to Amazon Kinesis Firehose Using Kinesis Agent](writing-with-agents.md)
+ [Writing to a Kinesis Firehose Delivery Stream Using the AWS SDK](writing-with-sdk.md)
+ [Writing to Amazon Kinesis Firehose Using CloudWatch Logs](writing-with-cloudwatch-logs.md)
+ [Writing to Amazon Kinesis Firehose Using CloudWatch Events](writing-with-cloudwatch-events.md)
+ [Writing to Amazon Kinesis Firehose Using AWS IoT](writing-with-iot.md)