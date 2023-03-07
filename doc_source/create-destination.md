# Destination Settings<a name="create-destination"></a>

This topic describes the destination settings for your delivery stream\.

**Topics**
+ [Choose Amazon S3 for Your Destination](#create-destination-s3)
+ [Choose Amazon Redshift for Your Destination](#create-destination-redshift)
+ [Choose OpenSearch Service for Your Destination](#create-destination-elasticsearch)
+ [Choose OpenSearch Serverless for Your Destination](#create-destination-opensearch-serverless)
+ [Choose HTTP Endpoint for Your Destination](#create-destination-http)
+ [Choose Datadog for Your Destination](#create-destination-datadog)
+ [Choose Honeycomb for Your Destination](#create-destination-honeycomb)
+ [Choose Coralogix for Your Destination](#create-destination-coralogix)
+ [Choose Dynatrace for Your Destination](#create-destination-dynatrace)
+ [Choose LogicMonitor for Your Destination](#create-destination-logicmonitor)
+ [Choose Logz\.io for Your Destination](#create-destination-logz)
+ [Choose MongoDB Cloud for Your Destination](#create-destination-mongodb)
+ [Choose New Relic for Your Destination](#create-destination-new-relic)
+ [Choose Splunk for Your Destination](#create-destination-splunk)
+ [Choose Sumo Logic for Your Destination](#create-destination-sumo-logic)
+ [Choose Elastic for Your Destination](#create-destination-elastic)

## Choose Amazon S3 for Your Destination<a name="create-destination-s3"></a>

You must specify the following settings in order to use Amazon S3 as the destination for your Kinesis Data Firehose delivery stream:

****
+ Enter values for the following fields:  
 **S3 bucket**   
Choose an S3 bucket that you own where the streaming data should be delivered\. You can create a new S3 bucket or choose an existing one\.  
 **S3 bucket prefix \- optional**   
If you don't enable dynamic partitioning, this is an optional field\. If you choose to enable dynamic partitioning, you MUST specify an S3 error bucket prefix for Kinesis Data Firehose to use when delivering data to Amazon S3 in error conditions\. If Kinesis Data Firehose fails to dynamically partition your incoming data, those data records are delivered to this S3 error bucket prefix\. For more information, see [Amazon S3 Object Name Format](basic-deliver.md#s3-object-name) and [Custom Prefixes for Amazon S3 Objects](s3-prefixes.md)  
 **Dynamic partitioning**   
Choose **Enabled** to enable and configure dynamic partitioning\.   
 **Multi record deagreggation**   
This is the process of parsing through the records in the delivery stream and separating them based either on valid JSON or on the specified new line delimiter\.  
If you aggregate multiple events, logs, or records into a single PutRecord and PutRecordBatch API call, you can still enable and configure dynamic partitioning\. With aggregated data, when you enable dynamic partitioning, Kinesis Data Firehose parses the records and looks for multiple valid JSON objects within each API call\. When the delivery stream is configured with Kinesis Data Stream as a source, you can also use the built\-in aggregation in the Kinesis Producer Library \(KPL\)\. Data partition functionality is executed after data is de\-aggregated\. Therefore, each record in each API call can be delivered to different Amazon S3 prefixes\. You can also leverage the Lambda function integration to perform any other de\-aggregation or any other transformation before the data partitioning functionality\.  
If your data is aggregated, dynamic partitioning can be applied only after data deaggregation is performed\. So if you enable dynamic partitioning to your aggregated data, you must choose **Enabled** to enable multi record deaggregation\. 
Kinesis Data Firehose delivery stream preforms the following processing steps in the following order: KPL \(protobuf\) de\-aggregation, JSON or delimiter de\-aggregation, Lambda processing, data partitioning, data format conversion, and Amazon S3 delivery\.  
 **Multi record deaggreation type**   
If you enabled multi recrord deaggregation, you must specify the method for Kinesis Data Firehose to deaggregate your data\. Use the drop\-down menu to choose either **JSON** or **Delimited**\.   
 **New line delimiter**   
When you enable dynamic partitioning, you can configure your delivery stream to add a new line delimiter between records in objects that are delivered to Amazon S3\. To do so, choose **Enabled**\. To not add a new line delimiter between records in objects that are delivered to Amazon S3, choose **Disabled**\.  
 **Inline parsing**   
This is one of the supported mechanisms to dynamically partition your data that is bound for Amazon S3\. To use inline parsing for dynamic partitioning of your data, you must specify data record parameters to be used as partitioning keys and provide a value for each specified partitioning key\. Choose **Enabled** to enable and configure inline parsing\.  
If you specified an AWS Lambda function in the steps above for transforming your source records, you can use this function to dyncamically partition your data that is bound to S3 and you can still create your partitioning keys with inline parsing\. With dynamic partitioning, you can use either inline parsing or your AWS Lambda function to create your partitioning keys\. Or you can use both inline parsing and your AWS Lambda function at the same time to create your partitioning keys\.   
 **Dynamic partitioning keys**   
You can use the **Key** and **Value** fields to specify the data record parameters to be used as dynamic partitioning keys and jq queries to generate dynamic partitioning key values\. Kinesis Data Firehose supports jq 1\.6 only\. You can specify up to 50 dynamic partitioning keys\. You must enter valid jq expressions for your dynamic partitioning key values in order to successfully configure dynamic partitioning for your delivery stream\.  
 **S3 bucket prefix**   
When you enable and configure dynamic partitioning, you must specify the S3 bucket prefixes to which Kinesis Data Firehose is to deliver partitioned data\.  
In order for dynamic partitioning to be configured correctly, the number of the S3 bucket prefixes must be identical to the number of the specified partitioning keys\.  
 You can partition your source data with inline parsing or with your specified AWS Lambda function\. If you specified an AWS Lambda function to create partitioning keys for your source data, you must manually type in the S3 bucket prefix value\(s\) using the following format: "partitionKeyFromLambda:keyID"\. If you are using inline parsing to specify the partitioning keys for your source data, you can either manually type in the S3 bucket preview values using the following format: "partitionKeyFromQuery:keyID" or you can choose the **Apply dynamic partitioning keys** button to use your dynamic partitioning key/value pairs to auto\-generate your S3 bucket prefixes\. While partitioning your data with either inline parsing or AWS Lambda, you can also use the following expression forms in your S3 bucket prefix: \!\{namespace:value\}, where namespace can be either partitionKeyFromQuery or partitionKeyFromLambda\.   
 **S3 buffer hints**   
Kinesis Data Firehose buffers incoming data before delivering it to the specified destination\. The recommended buffer size for the destination varies from service provider to service provider\.   
 **S3 compressions and encryption**   
Choose GZIP, Snappy, Zip, or Hadoop\-Compatible Snappy data compression, or no data compression\. Snappy, Zip, and Hadoop\-Compatible Snappy compression is not available for delivery streams with Amazon Redshift as the destination\.   
Kinesis Data Firehose supports Amazon S3 server\-side encryption with AWS Key Management Service \(AWS KMS\) for encrypting delivered data in Amazon S3\. You can choose to not encrypt the data or to encrypt with a key from the list of AWS KMS keys that you own\. For more information, see [Protecting Data Using Server\-Side Encryption with AWS KMS–Managed Keys \(SSE\-KMS\)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html)\. 

## Choose Amazon Redshift for Your Destination<a name="create-destination-redshift"></a>

This section describes settings for using Amazon Redshift as your delivery stream destination\.

****
+ Enter values for the following fields:  
 **Cluster**   
The Amazon Redshift cluster to which S3 bucket data is copied\. Configure the Amazon Redshift cluster to be publicly accessible and unblock Kinesis Data Firehose IP addresses\. For more information, see [Grant Kinesis Data Firehose Access to an Amazon Redshift Destination ](controlling-access.md#using-iam-rs)\.  
 **User name**   
An Amazon Redshift user with permissions to access the Amazon Redshift cluster\. This user must have the Amazon Redshift `INSERT` permission for copying data from the S3 bucket to the Amazon Redshift cluster\.  
 **Password**   
The password for the user who has permissions to access the cluster\.  
 **Database**   
The Amazon Redshift database to where the data is copied\.  
 **Table**   
The Amazon Redshift table to where the data is copied\.  
 **Columns**   
\(Optional\) The specific columns of the table to which the data is copied\. Use this option if the number of columns defined in your Amazon S3 objects is less than the number of columns within the Amazon Redshift table\.   
 **Intermediate S3 destination**   <a name="redshift-s3-bucket"></a>
Kinesis Data Firehose delivers your data to your S3 bucket first and then issues an Amazon Redshift COPY command to load the data into your Amazon Redshift cluster\. Specify an S3 bucket that you own where the streaming data should be delivered\. Create a new S3 bucket, or choose an existing bucket that you own\.  
Kinesis Data Firehose doesn't delete the data from your S3 bucket after loading it to your Amazon Redshift cluster\. You can manage the data in your S3 bucket using a lifecycle configuration\. For more information, see [Object Lifecycle Management](https://docs.aws.amazon.com/AmazonS3/latest/dev/object-lifecycle-mgmt.html) in the *Amazon Simple Storage Service User Guide*\.  
 **Intermediate S3 prefix**   
\(Optional\) To use the default prefix for Amazon S3 objects, leave this option blank\. Kinesis Data Firehose automatically uses a prefix in "`YYYY/MM/dd/HH`" UTC time format for delivered Amazon S3 objects\. You can add to the start of this prefix\. For more information, see [Amazon S3 Object Name Format](basic-deliver.md#s3-object-name)\.  
 **COPY options**   <a name="redshift-copy-parameters"></a>
Parameters that you can specify in the Amazon Redshift COPY command\. These might be required for your configuration\. For example, "`GZIP`" is required if Amazon S3 data compression is enabled\. "`REGION`" is required if your S3 bucket isn't in the same AWS Region as your Amazon Redshift cluster\. For more information, see [COPY](https://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html) in the *Amazon Redshift Database Developer Guide*\.  
 **COPY command**   <a name="redshift-copy-command"></a>
The Amazon Redshift COPY command\. For more information, see [COPY](https://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html) in the *Amazon Redshift Database Developer Guide*\.  
 **Retry duration**   
Time duration \(0–7200 seconds\) for Kinesis Data Firehose to retry if data COPY to your Amazon Redshift cluster fails\. Kinesis Data Firehose retries every 5 minutes until the retry duration ends\. If you set the retry duration to 0 \(zero\) seconds, Kinesis Data Firehose does not retry upon a COPY command failure\.  
 **S3 buffer hints**   
Kinesis Data Firehose buffers incoming data before delivering it to the specified destination\. The recommended buffer size for the destination varies from service provider to service provider\.  
 **S3 compressions and encryption**   
Choose GZIP or no data compression\.   
Kinesis Data Firehose supports Amazon S3 server\-side encryption with AWS Key Management Service \(AWS KMS\) for encrypting delivered data in Amazon S3\. You can choose to not encrypt the data or to encrypt with a key from the list of AWS KMS keys that you own\. For more information, see [Protecting Data Using Server\-Side Encryption with AWS KMS–Managed Keys \(SSE\-KMS\)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html)\. 

## Choose OpenSearch Service for Your Destination<a name="create-destination-elasticsearch"></a>

This section describes options for using OpenSearch Service for your destination\.

****
+ Enter values for the following fields:  
** **OpenSearch Service domain** **  
The OpenSearch Service domain to which your data is delivered\.  
** **Index** **  
The OpenSearch Service index name to be used when indexing data to your OpenSearch Service cluster\.  
** **Index rotation** **  
Choose whether and how often the OpenSearch Service index should be rotated\. If index rotation is enabled, Kinesis Data Firehose appends the corresponding timestamp to the specified index name and rotates\. For more information, see [Index Rotation for the OpenSearch Service Destination](basic-deliver.md#es-index-rotation)\.  
** **Type** **  
The OpenSearch Service type name to be used when indexing data to your OpenSearch Service cluster\. For Elasticsearch 7\.x and OpenSearch 1\.x, there can be only one type per index\. If you try to specify a new type for an existing index that already has another type, Kinesis Data Firehose returns an error during runtime\.   
For Elasticsearch 7\.x, leave this field empty\.  
** **Retry duration** **  
Time duration \(0–7200 seconds\) for Kinesis Data Firehose to retry if an index request to your OpenSearch Service cluster fails\. Kinesis Data Firehose retries every 5 minutes until the retry duration ends\. If you set the retry duration to 0 \(zero\) seconds, Kinesis Data Firehose does not retry upon an index request failure\.  
** **Destination VPC connectivity** **  
If your OpenSearch Service domain is in a private VPC, use this section to specify that VPC\. Also specify the subnets and subgroups that you want Kinesis Data Firehose to use when it sends data to your OpenSearch Service domain\. You can use the same security group that the doma OpenSearch Servicein uses or different ones\. If you specify different security groups, ensure that they allow outbound HTTPS traffic to the OpenSearch Service domain's security group\. Also ensure that the OpenSearch Service domain's security group allows HTTPS traffic from the security groups that you specified when you configured your delivery stream\. If you use the same security group for both your delivery stream and the OpenSearch Service domain, make sure the security group's inbound rule allows HTTPS traffic\. For more information about security group rules, see [Security group rules](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#SecurityGroupRules) in the Amazon VPC documentation\.  
**Buffer hints**  
Kinesis Data Firehose buffers incoming data before delivering it to the specified destination\. The recommended buffer size for the destination varies from service provider to service provider\.

## Choose OpenSearch Serverless for Your Destination<a name="create-destination-opensearch-serverless"></a>

This section describes options for using OpenSearch Serverless for your destination\.

****
+ Enter values for the following fields:  
** **OpenSearch Serverless collection** **  
The endpoint for a group of OpenSearch Serverless indexes to which your data is delivered\.  
** **Index** **  
The OpenSearch Serverless index name to be used when indexing data to your OpenSearch Serverless collection\.  
** **Destination VPC connectivity** **  
If your OpenSearch Serverless collection is in a private VPC, use this section to specify that VPC\. Also specify the subnets and subgroups that you want Kinesis Data Firehose to use when it sends data to your OpenSearch Serverless collection\.  
** **Retry duration** **  
Time duration \(0–7200 seconds\) for Kinesis Data Firehose to retry if an index request to your OpenSearch Serverless collection fails\. Kinesis Data Firehose retries every 5 minutes until the retry duration ends\. If you set the retry duration to 0 \(zero\) seconds, Kinesis Data Firehose does not retry upon an index request failure\.  
**Buffer hints**  
Kinesis Data Firehose buffers incoming data before delivering it to the specified destination\. The recommended buffer size for the destination varies from service provider to service provider\.

## Choose HTTP Endpoint for Your Destination<a name="create-destination-http"></a>

This section describes options for using **HTTP endpoint** for your destination\.

**Important**  
If you choose an HTTP endpoint as your destination, review and follow the instructions in [Appendix \- HTTP Endpoint Delivery Request and Response Specifications](httpdeliveryrequestresponse.md)\.

****
+ Provide values for the following fields:  
 **HTTP endpoint name \- optional**   
Specify a user friendly name for the HTTP endpoint\. For example, `My HTTP Endpoint Destination`\.  
 **HTTP endpoint URL**   
Specify the URL for the HTTP endpoint in the following format: `https://xyz.httpendpoint.com`\. The URL must be an HTTPS URL\.  
 **Access key \- optional**   
Contact the endpoint owner to obtain the access key \(if it is required\) to enable data delivery to their endpoint from Kinesis Data Firehose\.  
 **Content encoding**   
Kinesis Data Firehose uses content encoding to compress the body of a request before sending it to the destination\. Choose **GZIP** or **Disabled** to enable/disable content encoding of your request\.   
 **Retry duration**   
Specify how long Kinesis Data Firehose retries sending data to the selected HTTP endpoint\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from the HTTP endpoint\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to the HTTP endpoint \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from the HTTP endpoint\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.  
 **Parameters \- optional**   
Kinesis Data Firehose includes these key\-value pairs in each HTTP call\. These parameters can help you identify and organize your destinations\.  
 **S3 buffer hints**   
Kinesis Data Firehose buffers incoming data before delivering it to the specified destination\. The recommended buffer size for the destination varies from service provider to service provider\.  
For the HTTP endpoint destinations, if you are seeing 413 response codes from the destination endpoint in CloudWatch Logs, lower the buffering hint size on your delivery stream and try again\.

## Choose Datadog for Your Destination<a name="create-destination-datadog"></a>

This section describes options for using **Datadog** for your destination\. For more information about Datadog, see [https://docs\.datadoghq\.com/integrations/amazon\_web\_services/](https://docs.datadoghq.com/integrations/amazon_web_services/)\.

****
+ Provide values for the following fields:  
 **HTTP endpoint URL**   
Choose the HTTP endpoint URL from the following options in the drop down menu:  
  + **Datadog logs \- US1**
  + **Datadog logs \- US5**
  + **Datadog logs \- EU**
  + **Datadog logs \- GOV**
  + **Datadog metrics \- US**
  + **Datadog metrics \- EU**  
 **API key**   
Contact Datadog to obtain the API key required to enable data delivery to this endpoint from Kinesis Data Firehose\.  
 **Content encoding**   
Kinesis Data Firehose uses content encoding to compress the body of a request before sending it to the destination\. Choose **GZIP** or **Disabled** to enable/disable content encoding of your request\.   
 **Retry duration**   
Specify how long Kinesis Data Firehose retries sending data to the selected HTTP endpoint\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from the HTTP endpoint\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to the HTTP endpoint \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from the HTTP endpoint\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.  
 **Parameters \- optional**   
Kinesis Data Firehose includes these key\-value pairs in each HTTP call\. These parameters can help you identify and organize your destinations\.   
 **S3 buffer hints**   
Kinesis Data Firehose buffers incoming data before delivering it to the specified destination\. The recommended buffer size for the destination varies from service provider to service provider\.

## Choose Honeycomb for Your Destination<a name="create-destination-honeycomb"></a>

This section describes options for using **Honeycomb** for your destination\. For more information about Honeycomb, see [https://docs\.honeycomb\.io/getting\-data\-in/metrics/aws\-cloudwatch\-metrics/ ](https://docs.honeycomb.io/getting-data-in/metrics/aws-cloudwatch-metrics/ )\.

****
+ Provide values for the following fields:  
 **Honeycomb Kinesis endpoint**   
Specify the URL for the HTTP endpoint in the following format: https://api\.honeycomb\.io/1/kinesis\_events/\{\{dataset\}\}   
 **API key**   
Contact Honeycomb to obtain the API key required to enable data delivery to this endpoint from Kinesis Data Firehose\.  
 **Content encoding**   
Kinesis Data Firehose uses content encoding to compress the body of a request before sending it to the destination\. Choose **GZIP** to enable content encoding of your request\. This is the recommended option for the Honeycomb destination\.  
 **Retry duration**   
Specify how long Kinesis Data Firehose retries sending data to the selected HTTP endpoint\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from the HTTP endpoint\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to the HTTP endpoint \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from the HTTP endpoint\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.  
 **Parameters \- optional**   
Kinesis Data Firehose includes these key\-value pairs in each HTTP call\. These parameters can help you identify and organize your destinations\.   
 **S3 buffer hints**   
Kinesis Data Firehose buffers incoming data before delivering it to the specified destination\. The recommended buffer size for the destination varies from service provider to service provider\.

## Choose Coralogix for Your Destination<a name="create-destination-coralogix"></a>

This section describes options for using **Coralogix** for your destination\. For more information about Coralogix, see [https://coralogix\.com/integrations/aws\-firehose ](https://coralogix.com/integrations/aws-firehose)\.

****
+ Provide values for the following fields:  
 **HTTP endpoint URL**   
Choose the HTTP endpoint URL from the following options in the drop down menu:  
  + **Coralogix \- US**
  + **Coralogix \- SINGAPORE**
  + **Coralogix \- IRELAND**
  + **Coralogix \- INDIA**
  + **Coralogix \- STOCKHOLM**  
 **Private key**   
Contact Coralogix to obtain the private key required to enable data delivery to this endpoint from Kinesis Data Firehose\.  
 **Content encoding**   
Kinesis Data Firehose uses content encoding to compress the body of a request before sending it to the destination\. Choose **GZIP** to enable content encoding of your request\. This is the recommended option for the Coralogix destination\.  
 **Retry duration**   
Specify how long Kinesis Data Firehose retries sending data to the selected HTTP endpoint\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from the HTTP endpoint\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to the HTTP endpoint \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from the HTTP endpoint\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.  
 **Parameters \- optional**   
Kinesis Data Firehose includes these key\-value pairs in each HTTP call\. These parameters can help you identify and organize your destinations\.   
  + applicationName: the environment where you are running Data Firehose
  + subsystemName: the name of the Data Firehose integration
  + computerName: the name of the delivery stream in use  
 **S3 buffer hints**   
Kinesis Data Firehose buffers incoming data before delivering it to the specified destination\. The recommended buffer size for the destination varies from service provider to service provider\.

## Choose Dynatrace for Your Destination<a name="create-destination-dynatrace"></a>

This section describes options for using **Dynatrace** for your destination\. For more information, see [https://www\.dynatrace\.com/support/help/technology\-support/cloud\-platforms/amazon\-web\-services/integrations/cloudwatch\-metric\-streams/](https://www.dynatrace.com/support/help/technology-support/cloud-platforms/amazon-web-services/integrations/cloudwatch-metric-streams/)\.

****
+ Provide values for the following fields:  
 **HTTP endpoint URL**   
Choose the HTTP endpoint URL \(**Dynatrace US**, **Dynatrace EU**, or **Dynatrace Global**\) from the drop down menu\.  
 **API token**   
Generate the Dynatrace API token required for data delivery from Kinesis Data Firehose\. For more information, see [https://www\.dynatrace\.com/support/help/dynatrace\-api/basics/dynatrace\-api\-authentication/](https://www.dynatrace.com/support/help/dynatrace-api/basics/dynatrace-api-authentication/)\.  
 **API URL**   
Provide the API URL of your Dynatrace environment\.  
 **Content encoding**   
Kinesis Data Firehose uses content encoding to compress the body of a request before sending it to the destination\. Choose **GZIP** or **Disabled** to enable/disable content encoding of your request\.   
 **Retry duration**   
Specify how long Kinesis Data Firehose retries sending data to the selected HTTP endpoint\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from the HTTP endpoint\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to the HTTP endpoint \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from the HTTP endpoint\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.  
 **Parameters \- optional**   
Kinesis Data Firehose includes these key\-value pairs in each HTTP call\. These parameters can help you identify and organize your destinations\.   
When using Dynatrace as your specified destination, you must specify at least one parameter key\-value pair\. You must name this key `dt-url` and set its value to the URL of your Dynatrace environment \(for example, `https://xyzab123456.dynatrace.live.com`\)\. You can then optionally specify additional parameter key\-value pairs and set them to custom names and values of your choosing\.  
S3 buffer hints  
Kinesis Data Firehose buffers incoming data before delivering it to the specified destination\. The recommended buffer size for the destination varies from service provider to service provider\.

## Choose LogicMonitor for Your Destination<a name="create-destination-logicmonitor"></a>

This section describes options for using **LogicMonitor** for your destination\. For more information, see [https://www\.logicmonitor\.com](https://www.logicmonitor.com)\.

****
+ Provide values for the following fields:  
 **HTTP endpoint URL**   
Specify the URL for the HTTP endpoint in the following format: https://ACCOUNT\.logicmonitor\.com  
 **API key**   
Contact LogicMonitor to obtain the API key required to enable data delivery to this endpoint from Kinesis Data Firehose\.  
 **Content encoding**   
Kinesis Data Firehose uses content encoding to compress the body of a request before sending it to the destination\. Choose **GZIP** or **Disabled** to enable/disable content encoding of your request\.   
 **Retry duration**   
Specify how long Kinesis Data Firehose retries sending data to the selected HTTP endpoint\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from the HTTP endpoint\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to the HTTP endpoint \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from the HTTP endpoint\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.  
 **Parameters \- optional**   
Kinesis Data Firehose includes these key\-value pairs in each HTTP call\. These parameters can help you identify and organize your destinations\.   
 **S3 buffer hints**   
Kinesis Data Firehose buffers incoming data before delivering it to the specified destination\. The recommended buffer size for the destination varies from service provider to service provider\.

## Choose Logz\.io for Your Destination<a name="create-destination-logz"></a>

This section describes options for using **Logz\.io** for your destination\. For more information, see [https://logz\.io/](https://logz.io/)\.

**Note**  
In the Europe \(Milan\) region, Logz\.io is not supported as an Amazon Kinesis Data Firehose destination\.

****
+ Provide values for the following fields:  
 **HTTP endpoint URL**   
Specify the URL for the HTTP endpoint in the following format: `https://listener-aws-metrics-stream-<region>.logz.io/`\. For example, `https://listener-aws-metrics-stream-us.logz.io/`\. The URL must be an HTTPS URL\.   
 **Content encoding**   
Kinesis Data Firehose uses content encoding to compress the body of a request before sending it to the destination\. Choose **GZIP** or **Disabled** to enable/disable content encoding of your request\.   
 **Retry duration**   
Specify how long Kinesis Data Firehose retries sending data to Logz\.io\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from the HTTP endpoint\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to the HTTP endpoint \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from the HTTP endpoint\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.  
 **Parameters \- optional**   
Kinesis Data Firehose includes these key\-value pairs in each HTTP call\. These parameters can help you identify and organize your destinations\.   
 **S3 buffer hints**   
Kinesis Data Firehose buffers incoming data before delivering it to the specified destination\. The recommended buffer size for the destination varies from service provider to service provider\.

## Choose MongoDB Cloud for Your Destination<a name="create-destination-mongodb"></a>

This section describes options for using **MongoDB Cloud** for your destination\. For more information, see [https://www\.mongodb\.com](https://www.mongodb.com)\.

****
+ Provide values for the following fields:  
**MongoDB Realm webhook URL**  
Specify the URL for the HTTP endpoint in the following format: `https://webhooks.mongodb-realm.com`\. The URL must be an HTTPS URL\.   
**API key**  
Contact MongoDB Cloud to obtain the API key required to enable data delivery to this endpoint from Kinesis Data Firehose\.  
**Content encoding**  
Kinesis Data Firehose uses content encoding to compress the body of a request before sending it to the destination\. Choose **GZIP** or **Disabled** to enable/disable content encoding of your request\.   
**Retry duration**  
Specify how long Kinesis Data Firehose retries sending data to the selected third\-party provider\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from the HTTP endpoint\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to the HTTP endpoint \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from the HTTP endpoint\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.  
**S3 buffer hints**  
Kinesis Data Firehose buffers incoming data before delivering it to the specified destination\. The recommended buffer size for the destination varies from service provider to service provider\.  
**Parameters \- optional**  
Kinesis Data Firehose includes these key\-value pairs in each HTTP call\. These parameters can help you identify and organize your destinations\. 

## Choose New Relic for Your Destination<a name="create-destination-new-relic"></a>

This section describes options for using **New Relic** for your destination\. For more information, see [https://newrelic\.com](https://newrelic.com)\. 

****
+ Provide values for the following fields:  
 **HTTP endpoint URL**   
Choose the HTTP endpoint URL from the following options in the drop down menu:  
  + **New Relic logs \- US**
  + **New Relic metrics \- US**
  + **New Relic metrics \- EU**  
 **API key**   
Enter your License Key \(40\-characters hexadecimal string\) from your New Relic One Account settings\. This API key is required to enable data delivery to this endpoint from Kinesis Data Firehose\.  
 **Content encoding**   
Kinesis Data Firehose uses content encoding to compress the body of a request before sending it to the destination\. Choose **GZIP** or **Disabled** to enable/disable content encoding of your request\.   
 **Retry duration**   
Specify how long Kinesis Data Firehose retries sending data to the New Relic HTTP endpoint\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from the HTTP endpoint\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to the HTTP endpoint \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from the HTTP endpoint\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.  
 **Parameters \- optional**   
Kinesis Data Firehose includes these key\-value pairs in each HTTP call\. These parameters can help you identify and organize your destinations\.   
 **S3 buffer hints**   
Kinesis Data Firehose buffers incoming data before delivering it to the specified destination\. The recommended buffer size for the destination varies from service provider to service provider\.

## Choose Splunk for Your Destination<a name="create-destination-splunk"></a>

This section describes options for using Splunk for your destination\.

****
+ Provide values for the following fields:  
 **Splunk cluster endpoint**   
To determine the endpoint, see [Configure Amazon Kinesis Firehose to Send Data to the Splunk Platform](http://docs.splunk.com/Documentation/AddOns/latest/Firehose/ConfigureFirehose) in the Splunk documentation\.  
 **Splunk endpoint type**   
Choose `Raw endpoint` in most cases\. Choose `Event endpoint` if you preprocessed your data using AWS Lambda to send data to different indexes by event type\. For information about what endpoint to use, see [Configure Amazon Kinesis Firehose to send data to the Splunk platform](http://docs.splunk.com/Documentation/AddOns/released/Firehose/ConfigureFirehose) in the Splunk documentation\.  
 **Authentication token**   
To set up a Splunk endpoint that can receive data from Kinesis Data Firehose, see [Installation and configuration overview for the Splunk Add\-on for Amazon Kinesis Firehose](http://docs.splunk.com/Documentation/AddOns/released/Firehose/Installationoverview) in the Splunk documentation\. Save the token that you get from Splunk when you set up the endpoint for this delivery stream, and add it here\.  
 **HEC acknowledgement timeout**   
Specify how long Kinesis Data Firehose waits for the index acknowledgement from Splunk\. If Splunk doesn’t send the acknowledgment before the timeout is reached, Kinesis Data Firehose considers it a data delivery failure\. Kinesis Data Firehose then either retries or backs up the data to your Amazon S3 bucket, depending on the retry duration value that you set\.   
 **Retry duration**   
Specify how long Kinesis Data Firehose retries sending data to Splunk\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from Splunk\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to Splunk \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from Splunk\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.

## Choose Sumo Logic for Your Destination<a name="create-destination-sumo-logic"></a>

This section describes options for using **Sumo Logic** for your destination\. For more information, see [https://www\.sumologic\.com](https://www.sumologic.com)\.

****
+ Provide values for the following fields:  
 **HTTP endpoint URL**   
Specify the URL for the HTTP endpoint in the following format: `https://deployment name.sumologic.net/receiver/v1/kinesis/dataType/access token`\. The URL must be an HTTPS URL\.   
 **Content encoding**   
Kinesis Data Firehose uses content encoding to compress the body of a request before sending it to the destination\. Choose **GZIP** or **Disabled** to enable/disable content encoding of your request\.   
 **Retry duration**   
Specify how long Kinesis Data Firehose retries sending data to Sumo Logic\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from the HTTP endpoint\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to the HTTP endpoint \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from the HTTP endpoint\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.  
 **Parameters \- optional**   
Kinesis Data Firehose includes these key\-value pairs in each HTTP call\. These parameters can help you identify and organize your destinations\.   
 **S3 buffer hints**   
Kinesis Data Firehose buffers incoming data before delivering it to the specified destination\. The recommended buffer size for the Elastic destination varies from service provider to service provider\.

## Choose Elastic for Your Destination<a name="create-destination-elastic"></a>

This section describes options for using **Elastic** for your destination\. 

****
+ Provide values for the following fields:  
 **Elastic endpoint URL**   
Specify the URL for the HTTP endpoint in the following format: `https://<cluster-id>.es.<region>.aws.elastic-cloud.com`\. The URL must be an HTTPS URL\.   
 **API key**   
Contact Elastic service to obtain the API key required to enable data delivery to their service from Kinesis Data Firehose\.  
 **Content encoding**   
Kinesis Data Firehose uses content encoding to compress the body of a request before sending it to the destination\. Choose **GZIP** \(which is what selected by default\) or **Disabled** to enable/disable content encoding of your request\.   
 **Retry duration**   
Specify how long Kinesis Data Firehose retries sending data to Elastic\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from the HTTP endpoint\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to the HTTP endpoint \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from the HTTP endpoint\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.  
 **Parameters \- optional**   
Kinesis Data Firehose includes these key\-value pairs in each HTTP call\. These parameters can help you identify and organize your destinations\.   
 **S3 buffer hints**   
Kinesis Data Firehose buffers incoming data before delivering it to the specified destination\. The recommended buffer size for the Elastic destination is 1 MiB\.