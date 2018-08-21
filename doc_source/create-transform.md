# Transform records<a name="create-transform"></a>

This topic describes the **Transform records** page of the **Create Delivery Stream** wizard in Amazon Kinesis Data Firehose\.

**Transform records**

1. On the **Transform records with AWS Lambda** page, provide values for the following fields:  
**Record transformation**  
To create a Kinesis Data Firehose delivery stream that does not transform incoming data, choose **Disabled**\.   
To specify a Lambda function that Kinesis Data Firehose can invoke to transform incoming data before delivering it, choose **Enabled**\. You can configure a new Lambda function using one of the Lambda blueprints or choose an existing Lambda function\. Your Lambda function must contain the status model that is required by Kinesis Data Firehose\. For more information, see [Amazon Kinesis Data Firehose Data Transformation](data-transformation.md)\.

1. Choose **Next** to go to the [Choose destination](create-destination.md) page\.