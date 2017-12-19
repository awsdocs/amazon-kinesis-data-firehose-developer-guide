# Name and source<a name="create-name"></a>

This topic describes the **Name and source** page of the **Create Delivery Stream** wizard\.

**Name and source**

1. Open the Kinesis Firehose console at [https://console\.aws\.amazon\.com/firehose/](https://console.aws.amazon.com/firehose/)\.

1. Choose **Create Delivery Stream**\. On the **Name and source** page, enter values for the following fields:  
****Delivery stream name****  
The name of your Kinesis Firehose delivery stream\.  
****Source****  

   + **Direct PUT or other sources:** Choose this option to create a Kinesis Firehose delivery stream that producer applications write directly to\.

   + **Kinesis stream:** Choose this option to configure a Kinesis Firehose delivery stream that uses a Kinesis stream as a data source\. You can then use Amazon Kinesis Firehose to read data easily from an existing Kinesis stream and load it into destinations\. For more information about using Kinesis Streams as your data source, see [Writing to Amazon Kinesis Firehose Using Kinesis Streams](http://docs.aws.amazon.com/firehose/latest/dev/writing-with-kinesis-streams.html)\.

1. Choose **Next** to advance to the [Transform records](create-transform.md) page\.