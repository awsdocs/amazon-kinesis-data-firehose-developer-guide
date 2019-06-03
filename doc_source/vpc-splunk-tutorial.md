# Tutorial: Sending VPC Flow Logs to Splunk Using Amazon Kinesis Data Firehose<a name="vpc-splunk-tutorial"></a>

In this tutorial, you learn how to capture information about the IP traffic going to and from network interfaces in an Amazon Virtual Private Cloud \(Amazon VPC\)\. You then use Amazon Kinesis Data Firehose to send that information to Splunk\. For more information about VPC network traffic, see [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html) in the *Amazon VPC User Guide*\.

The following diagram shows the flow of data that is demonstrated in this tutorial\. 

![\[Diagram showing data logs flowing from VPC to CloudWatch to Kinesis Data Firehose, then to AWS Lambda, then to Splunk.\]](http://docs.aws.amazon.com/firehose/latest/dev/images/vpc-to-splunk-tutorial-flow.png)

As the diagram shows, first you send the Amazon VPC flow logs to Amazon CloudWatch\. Then from CloudWatch, the data goes to a Kinesis Data Firehose delivery stream\. Kinesis Data Firehose then invokes an AWS Lambda function to decompress the data, and sends the decompressed log data to Splunk\.

**Prerequisites**  
Before you begin, ensure that you have the following prerequisites:
+ **AWS account** — If you don't have an AWS account, create one at [http://aws\.amazon\.com](https://aws.amazon.com/)\. For more information, see [Setting Up for Amazon Kinesis Data Firehose](before-you-begin.md)\.
+ **AWS CLI** — Parts of this tutorial require that you use the AWS Command Line Interface \(AWS CLI\)\. To install the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\.
+ **HEC token** — In your Splunk deployment, set up an HTTP Event Collector \(HEC\) token with the source type `aws:cloudwatchlogs:vpcflow`\. For more information, see [ Installation and configuration overview for the Splunk Add\-on for Amazon Kinesis Firehose](http://docs.splunk.com/Documentation/AddOns/released/Firehose/Installationoverview) in the Splunk documentation\.

**Topics**
+ [Step 1: Send Log Data from Amazon VPC to Amazon CloudWatch](log-data-from-vpc-to-cw.md)
+ [Step 2: Create a Kinesis Data Firehose Delivery Stream with Splunk as a Destination](creating-the-stream-to-splunk.md)
+ [Step 3: Send the Data from Amazon CloudWatch to Kinesis Data Firehose](cw-to-delivery-stream.md)
+ [Step 4: Check the Results in Splunk and in Kinesis Data Firehose](check-vpc-to-splunk-results.md)