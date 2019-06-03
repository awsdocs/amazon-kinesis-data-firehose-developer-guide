# Name and source<a name="create-name"></a>

This topic describes the **Name and source** page of the **Create Delivery Stream** wizard in Amazon Kinesis Data Firehose\.

**Name and source**

1. Sign in to the AWS Management Console and open the Kinesis console at [https://console\.aws\.amazon\.com/kinesis](https://console.aws.amazon.com/kinesis)\.

1. Choose **Data Firehose** in the navigation pane\.

1. Choose **Create delivery stream**\.

1.  Enter values for the following fields:  
****Delivery stream name****  
The name of your Kinesis Data Firehose delivery stream\.  
****Source****  
   + **Direct PUT or other sources:** Choose this option to create a Kinesis Data Firehose delivery stream that producer applications write to directly\.
   + **Kinesis stream:** Choose this option to configure a Kinesis Data Firehose delivery stream that uses a Kinesis data stream as a data source\. You can then use Kinesis Data Firehose to read data easily from an existing Kinesis data stream and load it into destinations\. For more information about using Kinesis Data Streams as your data source, see [Writing to Amazon Kinesis Data Firehose Using Kinesis Data Streams](https://docs.aws.amazon.com/firehose/latest/dev/writing-with-kinesis-streams.html)\.

1. Choose **Next** to go to the [Process records](create-transform.md) page\.