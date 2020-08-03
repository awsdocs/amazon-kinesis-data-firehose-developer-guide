# Process records<a name="create-transform"></a>

This topic describes the **Process records** page of the **Create Delivery Stream** wizard in Amazon Kinesis Data Firehose\.

**Process records**

1. In the **Transform source records with AWS Lambda** section, provide values for the following field:  
**Record transformation**  
To create a Kinesis Data Firehose delivery stream that doesn't transform incoming data, choose **Disabled**\.   
To specify a Lambda function for Kinesis Data Firehose to invoke and use to transform incoming data before delivering it, choose **Enabled**\. You can configure a new Lambda function using one of the Lambda blueprints or choose an existing Lambda function\. Your Lambda function must contain the status model that is required by Kinesis Data Firehose\. For more information, see [Amazon Kinesis Data Firehose Data Transformation](data-transformation.md)\.

1. In the **Convert record format** section, provide values for the following field:  
**Record format conversion**  
To create a Kinesis Data Firehose delivery stream that doesn't convert the format of the incoming data records, choose **Disabled**\.   
To convert the format of the incoming records, choose **Enabled**, then specify the output format you want\. You need to specify an AWS Glue table that holds the schema that you want Kinesis Data Firehose to use to convert your record format\. For more information, see [Converting Your Input Record Format in Kinesis Data Firehose](record-format-conversion.md)\.  
For an example of how to set up record format conversion with AWS CloudFormation, see [AWS::KinesisFirehose::DeliveryStream](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-kinesisfirehose-deliverystream.html#aws-resource-kinesisfirehose-deliverystream--examples)\.

1. Choose **Next** to go to the **Select destination** page\.