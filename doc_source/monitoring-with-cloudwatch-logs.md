# Monitoring Kinesis Data Firehose Using CloudWatch Logs<a name="monitoring-with-cloudwatch-logs"></a>

Kinesis Data Firehose integrates with Amazon CloudWatch Logs so that you can view the specific error logs when the Lambda invocation for data transformation or data delivery fails\. You can enable Kinesis Data Firehose error logging when you create your delivery stream\.

If you enable Kinesis Data Firehose error logging in the Kinesis Data Firehose console, a log group and corresponding log streams are created for the delivery stream on your behalf\. The format of the log group name is `/aws/kinesisfirehose/delivery-stream-name`, where `delivery-stream-name` is the name of the corresponding delivery stream\. The log stream name is **S3Delivery**, **RedshiftDelivery**, or **ElasticsearchDelivery**, depending on the delivery destination\. Lambda invocation errors for data transformation are also logged to the log stream used for data delivery errors\.

For example, if you create a delivery stream "MyStream" with Amazon Redshift as the destination and enable Kinesis Data Firehose error logging, the following are created on your behalf: a log group named `aws/kinesisfirehose/MyStream` and two log streams named **S3Delivery** and **RedshiftDelivery**\. In this example, the **S3Delivery** log stream is used for logging errors related to delivery failure to the intermediate S3 bucket\. The **RedshiftDelivery** log stream is used for logging errors related to Lambda invocation failure and delivery failure to your Amazon Redshift cluster\.

You can enable Kinesis Data Firehose error logging through the AWS CLI, the API, or AWS CloudFormation using the `CloudWatchLoggingOptions` configuration\. To do so, create a log group and a log stream in advance\. We recommend reserving that log group and log stream for Kinesis Data Firehose error logging exclusively\. Also ensure that the associated IAM policy has `"logs:putLogEvents"` permission\. For more information, see [Controlling Access with Amazon Kinesis Data Firehose ](controlling-access.md)\.

Note that Kinesis Data Firehose does not guarantee that all delivery error logs are sent to CloudWatch Logs\. In circumstances where delivery failure rate is high, Kinesis Data Firehose samples delivery error logs before sending them to CloudWatch Logs\.

There is a nominal charge for error logs sent to CloudWatch Logs\. For more information, see [Amazon CloudWatch Pricing](https://aws.amazon.com/cloudwatch/pricing/)\.

**Topics**
+ [Data Delivery Errors](#data-delivery-errors)
+ [Lambda Invocation Errors](#lambda-invocation-errors)
+ [Accessing CloudWatch Logs for Kinesis Data Firehose](#accessing-firehose-data-delivery-logs)

## Data Delivery Errors<a name="data-delivery-errors"></a>

The following is a list of data delivery error codes and messages for each Kinesis Data Firehose destination\. Each error message also describes the proper action to take to fix the issue\.

**Topics**
+ [Amazon S3 Data Delivery Errors](#monitoring-s3-errors)
+ [Amazon Redshift Data Delivery Errors](#monitoring-rs-errors)
+ [Splunk Data Delivery Errors](#monitoring-splunk-errors)
+ [HTTPS Endpoint Data Delivery Errors](#monitoring-http-errors)
+ [Amazon Elasticsearch Service Data Delivery Errors](#monitoring-es-errors)

### Amazon S3 Data Delivery Errors<a name="monitoring-s3-errors"></a>

Kinesis Data Firehose can send the following Amazon S3\-related errors to CloudWatch Logs\.


| Error Code | Error Message and Information | 
| --- | --- | 
| S3\.KMS\.NotFoundException |  "The provided AWS KMS key was not found\. If you are using what you believe to be a valid AWS KMS key with the correct role, check if there is a problem with the account to which the AWS KMS key is attached\."  | 
| S3\.KMS\.RequestLimitExceeded |  "The KMS request per second limit was exceeded while attempting to encrypt S3 objects\. Increase the request per second limit\." For more information, see [Limits](https://docs.aws.amazon.com/kms/latest/developerguide/limits.html) in the *AWS Key Management Service Developer Guide*\.  | 
| S3\.AccessDenied | "Access was denied\. Ensure that the trust policy for the provided IAM role allows Kinesis Data Firehose to assume the role, and the access policy allows access to the S3 bucket\." | 
| S3\.AccountProblem | "There is a problem with your AWS account that prevents the operation from completing successfully\. Contact AWS Support\." | 
| S3\.AllAccessDisabled | "Access to the account provided has been disabled\. Contact AWS Support\." | 
| S3\.InvalidPayer | "Access to the account provided has been disabled\. Contact AWS Support\." | 
| S3\.NotSignedUp | "The account is not signed up for Amazon S3\. Sign the account up or use a different account\." | 
| S3\.NoSuchBucket | "The specified bucket does not exist\. Create the bucket or use a different bucket that does exist\." | 
| S3\.MethodNotAllowed | "The specified method is not allowed against this resource\. Modify the bucket’s policy to allow the correct Amazon S3 operation permissions\." | 
| InternalError | "An internal error occurred while attempting to deliver data\. Delivery will be retried; if the error persists, then it will be reported to AWS for resolution\." | 

### Amazon Redshift Data Delivery Errors<a name="monitoring-rs-errors"></a>

Kinesis Data Firehose can send the following Amazon Redshift\-related errors to CloudWatch Logs\.


| Error Code | Error Message and Information | 
| --- | --- | 
| Redshift\.TableNotFound |  "The table to which to load data was not found\. Ensure that the specified table exists\." The destination table in Amazon Redshift to which data should be copied from S3 was not found\. Note that Kinesis Data Firehose does not create the Amazon Redshift table if it does not exist\.  | 
| Redshift\.SyntaxError | "The COPY command contains a syntax error\. Retry the command\." | 
| Redshift\.AuthenticationFailed | "The provided user name and password failed authentication\. Provide a valid user name and password\." | 
| Redshift\.AccessDenied | "Access was denied\. Ensure that the trust policy for the provided IAM role allows Kinesis Data Firehose to assume the role\." | 
| Redshift\.S3BucketAccessDenied | "The COPY command was unable to access the S3 bucket\. Ensure that the access policy for the provided IAM role allows access to the S3 bucket\." | 
| Redshift\.DataLoadFailed | "Loading data into the table failed\. Check STL\_LOAD\_ERRORS system table for details\." | 
| Redshift\.ColumnNotFound | "A column in the COPY command does not exist in the table\. Specify a valid column name\." | 
| Redshift\.DatabaseNotFound | "The database specified in the Amazon Redshift destination configuration or JDBC URL was not found\. Specify a valid database name\." | 
| Redshift\.IncorrectCopyOptions |  "Conflicting or redundant COPY options were provided\. Some options are not compatible in certain combinations\. Check the COPY command reference for more info\." For more information, see the [Amazon Redshift COPY command](https://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html) in the *Amazon Redshift Database Developer Guide*\.  | 
| Redshift\.MissingColumn | "There is a column defined in the table schema as NOT NULL without a DEFAULT value and not included in the column list\. Exclude this column, ensure that the loaded data always provides a value for this column, or add a default value to the Amazon Redshift schema for this table\." | 
| Redshift\.ConnectionFailed | "The connection to the specified Amazon Redshift cluster failed\. Ensure that security settings allow Kinesis Data Firehose connections, that the cluster or database specified in the Amazon Redshift destination configuration or JDBC URL is correct, and that the cluster is available\." | 
| Redshift\.ColumnMismatch | "The number of jsonpaths in the COPY command and the number of columns in the destination table should match\. Retry the command\." | 
| Redshift\.IncorrectOrMissingRegion | "Amazon Redshift attempted to use the wrong region endpoint for accessing the S3 bucket\. Either specify a correct region value in the COPY command options or ensure that the S3 bucket is in the same region as the Amazon Redshift database\."  | 
| Redshift\.IncorrectJsonPathsFile | "The provided jsonpaths file is not in a supported JSON format\. Retry the command\." | 
| Redshift\.MissingS3File | "One or more S3 files required by Amazon Redshift have been removed from the S3 bucket\. Check the S3 bucket policies to remove any automatic deletion of S3 files\." | 
| Redshift\.InsufficientPrivilege | "The user does not have permissions to load data into the table\. Check the Amazon Redshift user permissions for the INSERT privilege\." | 
| Redshift\.ReadOnlyCluster | "The query cannot be executed because the system is in resize mode\. Try the query again later\." | 
| Redshift\.DiskFull | "Data could not be loaded because the disk is full\. Increase the capacity of the Amazon Redshift cluster or delete unused data to free disk space\." | 
| InternalError | "An internal error occurred while attempting to deliver data\. Delivery will be retried; if the error persists, then it will be reported to AWS for resolution\." | 

### Splunk Data Delivery Errors<a name="monitoring-splunk-errors"></a>

Kinesis Data Firehose can send the following Splunk\-related errors to CloudWatch Logs\.


| Error Code | Error Message and Information | 
| --- | --- | 
| Splunk\.ProxyWithoutStickySessions |  "If you have a proxy \(ELB or other\) between Kinesis Data Firehose and the HEC node, you must enable sticky sessions to support HEC ACKs\."  | 
| Splunk\.DisabledToken | "The HEC token is disabled\. Enable the token to allow data delivery to Splunk\." | 
| Splunk\.InvalidToken | "The HEC token is invalid\. Update Kinesis Data Firehose with a valid HEC token\." | 
| Splunk\.InvalidDataFormat | "The data is not formatted correctly\. To see how to properly format data for Raw or Event HEC endpoints, see [Splunk Event Data](http://dev.splunk.com/view/event-collector/SP-CAAAE6P#data)\." | 
| Splunk\.InvalidIndex | "The HEC token or input is configured with an invalid index\. Check your index configuration and try again\." | 
| Splunk\.ServerError | "Data delivery to Splunk failed due to a server error from the HEC node\. Kinesis Data Firehose will retry sending the data if the retry duration in your Kinesis Data Firehose is greater than 0\. If all the retries fail, Kinesis Data Firehose backs up the data to Amazon S3\." | 
| Splunk\.DisabledAck | "Indexer acknowledgement is disabled for the HEC token\. Enable indexer acknowledgement and try again\. For more info, see [Enable indexer acknowledgement](http://dev.splunk.com/view/event-collector/SP-CAAAE8X#enable)\." | 
| Splunk\.AckTimeout | "Did not receive an acknowledgement from HEC before the HEC acknowledgement timeout expired\. Despite the acknowledgement timeout, it's possible the data was indexed successfully in Splunk\. Kinesis Data Firehose backs up in Amazon S3 data for which the acknowledgement timeout expired\." | 
| Splunk\.MaxRetriesFailed |  "Failed to deliver data to Splunk or to receive acknowledgment\. Check your HEC health and try again\."  | 
| Splunk\.ConnectionTimeout | "The connection to Splunk timed out\. This might be a transient error and the request will be retried\. Kinesis Data Firehose backs up the data to Amazon S3 if all retries fail\." | 
| Splunk\.InvalidEndpoint | "Could not connect to the HEC endpoint\. Make sure that the HEC endpoint URL is valid and reachable from Kinesis Data Firehose\." | 
| Splunk\.ConnectionClosed | "Unable to send data to Splunk due to a connection failure\. This might be a transient error\. Increasing the retry duration in your Kinesis Data Firehose configuration might guard against such transient failures\."  | 
| Splunk\.SSLUnverified | "Could not connect to the HEC endpoint\. The host does not match the certificate provided by the peer\. Make sure that the certificate and the host are valid\."  | 
| Splunk\.SSLHandshake | "Could not connect to the HEC endpoint\. Make sure that the certificate and the host are valid\."  | 

### HTTPS Endpoint Data Delivery Errors<a name="monitoring-http-errors"></a>

Kinesis Data Firehose can send the following HTTP Endpoint\-related errors to CloudWatch Logs\. If none of these errors are a match to the problem that you're experiencing, the default error is the following: "An internal error occurred while attempting to deliver data\. Delivery will be retried; if the error persists, then it will be reported to AWS for resolution\."


| Error Code | Error Message and Information | 
| --- | --- | 
| HttpEndpoint\.RequestTimeout |  The delivery timed out before a response was received and will be retried\. If this error persists, contact the AWS Firehose service team\.  | 
| HttpEndpoint\.ResponseTooLarge | "The response received from the endpoint is too large\. Contact the owner of the endpoint to resolve this issue\." | 
| HttpEndpoint\.InvalidResponseFromDestination | "The response received from the specified endpoint is invalid\. Contact the owner of the endpoint to resolve the issue\." | 
| HttpEndpoint\.DestinationException | "The following response was received from the endpoint destination\." | 
| HttpEndpoint\.ConnectionFailed | "Unable to connect to the destination endpoint\. Contact the owner of the endpoint to resolve this issue\." | 
| HttpEndpoint\.ConnectionReset | "Unable to maintain connection with the endpoint\. Contact the owner of the endpoint to resolve this issue\." | 
| HttpEndpoint\.ConnectionReset | "Trouble maintaining connection with the endpoint\. Please reach out to the owner of the endpoint\." | 

### Amazon Elasticsearch Service Data Delivery Errors<a name="monitoring-es-errors"></a>

For the Amazon ES destination, Kinesis Data Firehose sends errors to CloudWatch Logs as they are returned by Elasticsearch\.

## Lambda Invocation Errors<a name="lambda-invocation-errors"></a>

Kinesis Data Firehose can send the following Lambda invocation errors to CloudWatch Logs\.


| Error Code | Error Message and Information | 
| --- | --- | 
| Lambda\.AssumeRoleAccessDenied |  "Access was denied\. Ensure that the trust policy for the provided IAM role allows Kinesis Data Firehose to assume the role\."  | 
| Lambda\.InvokeAccessDenied |  "Access was denied\. Ensure that the access policy allows access to the Lambda function\."  | 
| Lambda\.JsonProcessingException |  "There was an error parsing returned records from the Lambda function\. Ensure that the returned records follow the status model required by Kinesis Data Firehose\." For more information, see [Data Transformation and Status Model](data-transformation.md#data-transformation-status-model)\.  | 
| Lambda\.InvokeLimitExceeded |  "The Lambda concurrent execution limit is exceeded\. Increase the concurrent execution limit\." For more information, see [AWS Lambda Limits](https://docs.aws.amazon.com/lambda/latest/dg/limits.html) in the *AWS Lambda Developer Guide*\.  | 
| Lambda\.DuplicatedRecordId |  "Multiple records were returned with the same record ID\. Ensure that the Lambda function returns unique record IDs for each record\." For more information, see [Data Transformation and Status Model](data-transformation.md#data-transformation-status-model)\.  | 
| Lambda\.MissingRecordId |  "One or more record IDs were not returned\. Ensure that the Lambda function returns all received record IDs\." For more information, see [Data Transformation and Status Model](data-transformation.md#data-transformation-status-model)\.  | 
| Lambda\.ResourceNotFound |  "The specified Lambda function does not exist\. Use a different function that does exist\."  | 
| Lambda\.InvalidSubnetIDException |  "The specified subnet ID in the Lambda function VPC configuration is invalid\. Ensure that the subnet ID is valid\."  | 
| Lambda\.InvalidSecurityGroupIDException |  "The specified security group ID in the Lambda function VPC configuration is invalid\. Ensure that the security group ID is valid\."  | 
| Lambda\.SubnetIPAddressLimitReachedException |  "AWS Lambda was not able to set up the VPC access for the Lambda function because one or more configured subnets have no available IP addresses\. Increase the IP address limit\." For more information, see [Amazon VPC Limits \- VPC and Subnets](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Appendix_Limits.html#vpc-limits-vpcs-subnets) in the *Amazon VPC User Guide*\.  | 
| Lambda\.ENILimitReachedException |  "AWS Lambda was not able to create an Elastic Network Interface \(ENI\) in the VPC, specified as part of the Lambda function configuration, because the limit for network interfaces has been reached\. Increase the network interface limit\." For more information, see [Amazon VPC Limits \- Network Interfaces](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Appendix_Limits.html#vpc-limits-enis) in the *Amazon VPC User Guide*\.  | 

## Accessing CloudWatch Logs for Kinesis Data Firehose<a name="accessing-firehose-data-delivery-logs"></a>

You can view the error logs related to Kinesis Data Firehose data delivery failure using the Kinesis Data Firehose console or the CloudWatch console\. The following procedures show you how to access error logs using these two methods\.

**To access error logs using the Kinesis Data Firehose console**

1. Sign in to the AWS Management Console and open the Kinesis console at [https://console\.aws\.amazon\.com/kinesis](https://console.aws.amazon.com/kinesis)\.

1. Choose **Data Firehose** in the navigation pane\.

1. On the navigation bar, choose an AWS Region\.

1. Choose a delivery stream name to go to the delivery stream details page\.

1. Choose **Error Log** to view a list of error logs related to data delivery failure\.

**To access error logs using the CloudWatch console**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. On the navigation bar, choose a Region\.

1. In the navigation pane, choose **Logs**\.

1. Choose a log group and log stream to view a list of error logs related to data delivery failure\.