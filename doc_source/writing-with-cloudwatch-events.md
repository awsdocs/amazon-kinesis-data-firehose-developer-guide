# Writing to Amazon Kinesis Data Firehose Using CloudWatch Events<a name="writing-with-cloudwatch-events"></a>

You can configure Amazon CloudWatch to send events to a Kinesis Data Firehose delivery stream by adding a target to a CloudWatch Events rule\.

**To create a target for a CloudWatch Events rule that sends events to an existing Kinesis Data Firehose delivery stream**

1. When creating your rule, in the **Step 1: Create Rule** page, for Targets, choose Add target, and then choose **Firehose delivery stream**\.

1. For Delivery stream, select an existing Kinesis Data Firehose delivery stream\. 

For more information on creating CloudWatch Events rules, see [Getting Started with Amazon CloudWatch Events](http://docs.aws.amazon.com/AmazonCloudWatch/latest/events/CWE_GettingStarted.html)\.