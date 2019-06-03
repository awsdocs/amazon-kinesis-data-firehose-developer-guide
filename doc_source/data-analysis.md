# Using Amazon Kinesis Data Analytics<a name="data-analysis"></a>

## Create a Kinesis Data Analytics Application That Reads from a Delivery Stream<a name="create-application"></a>

1. Sign in to the AWS Management Console and open the Kinesis Data Analytics console at [ https://console\.aws\.amazon\.com/kinesisanalytics](https://console.aws.amazon.com/kinesisanalytics)\.

1. Choose **Create application**

1. Specify a name for the application and an optional description\. Then choose **Create application**\.

1. Choose **Connect streaming data**\.

1. For **Source**, choose **Kinesis Firehose delivery stream**\.

1. In the list labeled **Kinesis Firehose delivery stream**, choose the delivery stream that you want your Kinesis Data Analytics to process\. Alternatively, choose **Create new** to set up a new delivery stream\.

1. To finish setting up your Kinesis Data Analytics application, see [Getting Started with Amazon Kinesis Data Analytics for SQL Applications](https://docs.aws.amazon.com//kinesisanalytics/latest/dev/getting-started.html)\.

## Write Data from a Kinesis Data Analytics Application to a Delivery Stream<a name="kda-api"></a>

1. To create a Kinesis Data Analytics application, follow the instructions under [Getting Started with Amazon Kinesis Data Analytics for SQL Applications](https://docs.aws.amazon.com//kinesisanalytics/latest/dev/getting-started.html)\. 

1. Open the Kinesis Data Analytics console at [ https://console\.aws\.amazon\.com/kinesisanalytics](https://console.aws.amazon.com/kinesisanalytics)\.

1. In the list of application, choose the application that you want to configure to write to a delivery stream\.

1. Choose **Application details**\.

1. At the bottom of the page, choose **Connect to a destination**\.

1. Choose **Kinesis Firehose delivery stream**, and then choose an existing delivery stream in the list or choose **Create new** to create a new delivery stream\.