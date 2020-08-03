# Amazon Kinesis Data Firehose Data Transformation<a name="data-transformation"></a>

Kinesis Data Firehose can invoke your Lambda function to transform incoming source data and deliver the transformed data to destinations\. You can enable Kinesis Data Firehose data transformation when you create your delivery stream\.

## Data Transformation Flow<a name="data-transformation-flow"></a>

When you enable Kinesis Data Firehose data transformation, Kinesis Data Firehose buffers incoming data up to 3 MB by default\. \(To adjust the buffering size, use the [https://docs.aws.amazon.com/firehose/latest/APIReference/API_ProcessingConfiguration.html](https://docs.aws.amazon.com/firehose/latest/APIReference/API_ProcessingConfiguration.html) API with the [https://docs.aws.amazon.com/firehose/latest/APIReference/API_ProcessorParameter.html](https://docs.aws.amazon.com/firehose/latest/APIReference/API_ProcessorParameter.html) called `BufferSizeInMBs`\.\) Kinesis Data Firehose then invokes the specified Lambda function asynchronously with each buffered batch using the AWS Lambda synchronous invocation mode\. The transformed data is sent from Lambda to Kinesis Data Firehose\. Kinesis Data Firehose then sends it to the destination when the specified destination buffering size or buffering interval is reached, whichever happens first\.

**Important**  
The Lambda synchronous invocation mode has a payload size limit of 6 MB for both the request and the response\. Make sure that your buffering size for sending the request to the function is less than or equal to 6 MB\. Also ensure that the response that your function returns doesn't exceed 6 MB\.

## Data Transformation and Status Model<a name="data-transformation-status-model"></a>

All transformed records from Lambda must contain the following parameters, or Kinesis Data Firehose rejects them and treats that as a data transformation failure\.

**recordId**  
The record ID is passed from Kinesis Data Firehose to Lambda during the invocation\. The transformed record must contain the same record ID\. Any mismatch between the ID of the original record and the ID of the transformed record is treated as a data transformation failure\.

**result**  
The status of the data transformation of the record\. The possible values are: `Ok` \(the record was transformed successfully\), `Dropped` \(the record was dropped intentionally by your processing logic\), and `ProcessingFailed` \(the record could not be transformed\)\. If a record has a status of `Ok` or `Dropped`, Kinesis Data Firehose considers it successfully processed\. Otherwise, Kinesis Data Firehose considers it unsuccessfully processed\.

**data**  
The transformed data payload, after base64\-encoding\.

## Lambda Blueprints<a name="lambda-blueprints"></a>

There are blueprints that you can use to create a Lambda function for data transformation\. Some of these blueprints are in the AWS Lambda console and some are in the AWS Serverless Application Repository\.

**To see the blueprints that are available in the AWS Lambda console**

1. Sign in to the AWS Management Console and open the AWS Lambda console at [https://console\.aws\.amazon\.com/lambda/](https://console.aws.amazon.com/lambda/)\.

1. Choose **Create function**, and then choose **Use a blueprint**\.

1. In the **Blueprints** field, search for the keyword `firehose` to find the Kinesis Data Firehose Lambda blueprints\.

**To see the blueprints that are available in the AWS Serverless Application Repository**

1. Go to [AWS Serverless Application Repository](https://aws.amazon.com/serverless/serverlessrepo)\.

1. Choose **Browse all applications**\.

1. In the **Applications** field, search for the keyword `firehose`\.

You can also create a Lambda function without using a blueprint\. See [Getting Started with AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html)\.

## Data Transformation Failure Handling<a name="data-transformation-failure-handling"></a>

If your Lambda function invocation fails because of a network timeout or because you've reached the Lambda invocation limit, Kinesis Data Firehose retries the invocation three times by default\. If the invocation does not succeed, Kinesis Data Firehose then skips that batch of records\. The skipped records are treated as unsuccessfully processed records\. You can specify or override the retry options using the [CreateDeliveryStream](https://docs.aws.amazon.com/firehose/latest/APIReference/API_CreateDeliveryStream.html) or `[UpdateDestination](https://docs.aws.amazon.com/firehose/latest/APIReference/API_UpdateDestination.html)` API\. For this type of failure, you can log invocation errors to Amazon CloudWatch Logs\. For more information, see [Monitoring Kinesis Data Firehose Using CloudWatch Logs](monitoring-with-cloudwatch-logs.md)\.

If the status of the data transformation of a record is `ProcessingFailed`, Kinesis Data Firehose treats the record as unsuccessfully processed\. For this type of failure, you can emit error logs to Amazon CloudWatch Logs from your Lambda function\. For more information, see [Accessing Amazon CloudWatch Logs for AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/monitoring-functions-logs.html) in the *AWS Lambda Developer Guide*\.

If data transformation fails, the unsuccessfully processed records are delivered to your S3 bucket in the `processing-failed` folder\. The records have the following format:

```
{
    "attemptsMade": "count",
    "arrivalTimestamp": "timestamp",
    "errorCode": "code",
    "errorMessage": "message",
    "attemptEndingTimestamp": "timestamp",
    "rawData": "data",
    "lambdaArn": "arn"
}
```

`attemptsMade`  
The number of invocation requests attempted\.

`arrivalTimestamp`  
The time that the record was received by Kinesis Data Firehose\.

`errorCode`  
The HTTP error code returned by Lambda\.

`errorMessage`  
The error message returned by Lambda\.

`attemptEndingTimestamp`  
The time that Kinesis Data Firehose stopped attempting Lambda invocations\.

`rawData`  
The base64\-encoded record data\.

`lambdaArn`  
The Amazon Resource Name \(ARN\) of the Lambda function\.

## Duration of a Lambda Invocation<a name="data-transformation-execution-duration"></a>

Kinesis Data Firehose supports a Lambda invocation time of up to 5 minutes\. If your Lambda function takes more than 5 minutes to complete, you get the following error: Firehose encountered timeout errors when calling AWS Lambda\. The maximum supported function timeout is 5 minutes\.

For information about what Kinesis Data Firehose does if such an error occurs, see [Data Transformation Failure Handling](#data-transformation-failure-handling)\.

## Source Record Backup<a name="data-transformation-source-record-backup"></a>

Kinesis Data Firehose can back up all untransformed records to your S3 bucket concurrently while delivering transformed records to the destination\. You can enable source record backup when you create or update your delivery stream\. You cannot disable source record backup after you enable it\.