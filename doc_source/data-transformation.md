# Amazon Kinesis Firehose Data Transformation<a name="data-transformation"></a>

Kinesis Firehose can invoke your Lambda function to transform incoming source data and deliver the transformed data to destinations\. You can enable Kinesis Firehose data transformation when you create your delivery stream\.

## Data Transformation Flow<a name="data-transformation-flow"></a>

When you enable Kinesis Firehose data transformation, Kinesis Firehose buffers incoming data up to 3 MB or the buffering size you specified for the delivery stream, whichever is smaller\. Kinesis Firehose then invokes the specified Lambda function with each buffered batch asynchronously\. The transformed data is sent from Lambda to Kinesis Firehose for buffering\. Transformed data is delivered to the destination when the specified buffering size or buffering interval is reached, whichever happens first\.

## Data Transformation and Status Model<a name="data-transformation-status-model"></a>

All transformed records from Lambda must contain the following parameters or Kinesis Firehose rejects them and treats that as a data transformation failure\.

**recordId**  
The record ID is passed from Kinesis Firehose to Lambda during the invocation\. The transformed record must contain the same record ID\. Any mismatch between the ID of the original record and the ID of the transformed record is treated as a data transformation failure\.

**result**  
The status of the data transformation of the record\. The possible values are: "Ok" \(the record was transformed successfully\), "Dropped" \(the record was dropped intentionally by your processing logic\), and "ProcessingFailed" \(the record could not be transformed\)\. If a record has a status of "Ok" or "Dropped", Kinesis Firehose considers it successfully processed\. Otherwise, Kinesis Firehose considers it unsuccessfully processed\.

**data**  
The transformed data payload, after base64\-encoding\.

## Lambda Blueprints<a name="lambda-blueprints"></a>

Kinesis Firehose provides the following Lambda blueprints that you can use to create a Lambda function for data transformation\.

+ **General Kinesis Firehose Processing** — Contains the data transformation and status model described in the previous section\. Use this blueprint for any custom transformation logic\.

+ **Apache Log to JSON** — Parses and converts Apache log lines to JSON objects, using predefined JSON field names\.

+ **Apache Log to CSV** — Parses and converts Apache log lines to CSV format\.

+ **Syslog to JSON** — Parses and converts Syslog lines to JSON objects, using predefined JSON field names\.

+ **Syslog to CSV** — Parses and converts Syslog lines to CSV format\.

+ **CloudWatch Logs Processor** — Parses and extracts individual log events from records sent by CloudWatch Logs subscription filters\.

**Note**  
Lambda blueprints are only available in the Node\.js and Python languages\. You can implement your own functions in other supported languages\. For information on AWS Lambda supported languages, see [Introduction: Building Lambda Functions](http://docs.aws.amazon.com/lambda/latest/dg/lambda-app.html)\.

## Data Transformation Failure Handling<a name="data-transformation-failure-handling"></a>

If your Lambda function invocation fails because of a network timeout or because you've reached the Lambda invocation limit, Kinesis Firehose retries the invocation three times by default\. If the invocation does not succeed, Kinesis Firehose then skips that batch of records\. The skipped records are treated as unsuccessfully processed records\. You can specify or override the retry options using the [CreateDeliveryStream](http://docs.aws.amazon.com/firehose/latest/APIReference/API_CreateDeliveryStream.html) or [UpdateDestination](http://docs.aws.amazon.com/firehose/latest/APIReference/API_UpdateDestination.html) API\. For this type of failure, you can log invocation errors to Amazon CloudWatch Logs\. For more information, see [Monitoring with Amazon CloudWatch Logs](monitoring-with-cloudwatch-logs.md)\.

If the status of the data transformation of a record is `ProcessingFailed`, Kinesis Firehose treats the record as unsuccessfully processed\. For this type of failure, you can emit error logs to Amazon CloudWatch Logs from your Lambda function\. For more information, see [Accessing Amazon CloudWatch Logs for AWS Lambda](http://docs.aws.amazon.com/lambda/latest/dg/monitoring-functions-logs.html) in the *AWS Lambda Developer Guide*\.

If data transformation fails, the unsuccessfully processed records are delivered to your S3 bucket in the `processing_failed` folder\. The records have the following format:

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
The time that the record was received by Kinesis Firehose\.

`errorCode`  
The HTTP error code returned by Lambda\.

`errorMessage`  
The error message returned by Lambda\.

`attemptEndingTimestamp`  
The time that Kinesis Firehose stopped attempting Lambda invocations\.

`rawData`  
The base64\-encoded record data\.

`lambdaArn`  
The Amazon Resource Name \(ARN\) of the Lambda function\.

## Source Record Backup<a name="data-transformation-source-record-backup"></a>

Kinesis Firehose can back up all untransformed records to your S3 bucket concurrently while delivering transformed records to the destination\. You can enable source record backup when you create or update your delivery stream\. You cannot disable source record backup after you enable it\.