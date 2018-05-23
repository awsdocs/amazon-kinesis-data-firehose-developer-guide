# Transform records<a name="create-transform"></a>

This topic describes the **Transform records** page of the **Create Delivery Stream** wizard\.

**Transform records**

1. On the **Transform records with AWS Lambda** page, enter values for the following fields:  
**Record transformation**  
Choose **Disabled** to create a Kinesis data delivery stream that does not transform incoming data\. Choose **Enabled** to specify a Lambda function that Kinesis Data Firehose can invoke to transform incoming data before delivering it\. You can configure a new Lambda function using one of the Lambda blueprints or choose an existing Lambda function\. Your Lambda function must contain the status model required by Kinesis Data Firehose\. For more information, see [Amazon Kinesis Data Firehose Data Transformation](data-transformation.md)\.

1. Choose **Next** to advance to the [Choose destination](create-destination.md) page\.