# Sending Data to an Amazon Kinesis Data Firehose Delivery Stream<a name="basic-write"></a>

You can send data to your Kinesis Data Firehose Delivery stream using different types of sources: You can use a Kinesis data stream, the Kinesis Agent, or the Kinesis Data Firehose API using the AWS SDK\. You can also use Amazon CloudWatch Logs, CloudWatch Events, or AWS IoT as your data source\. If you are new to Kinesis Data Firehose, take some time to become familiar with the concepts and terminology presented in [What Is Amazon Kinesis Data Firehose?](what-is-this-service.md)\.

**Note**  
Some AWS services can only send messages and events to a Kinesis Data Firehose delivery stream that is in the same Region\. If your delivery stream doesn't appear as an option when you're configuring a target for Amazon CloudWatch Logs, CloudWatch Events, or AWS IoT, verify that your Kinesis Data Firehose delivery stream is in the same Region as your other services\.

**Topics**
+ [Writing to Kinesis Data Firehose Using Kinesis Data Streams](writing-with-kinesis-streams.md)
+ [Writing to Kinesis Data Firehose Using Kinesis Agent](writing-with-agents.md)
+ [Writing to Kinesis Data Firehose Using the AWS SDK](writing-with-sdk.md)
+ [Writing to Kinesis Data Firehose Using CloudWatch Logs](writing-with-cloudwatch-logs.md)
+ [Writing to Kinesis Data Firehose Using CloudWatch Events](writing-with-cloudwatch-events.md)
+ [Writing to Kinesis Data Firehose Using AWS IoT](writing-with-iot.md)