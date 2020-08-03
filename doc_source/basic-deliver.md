# Amazon Kinesis Data Firehose Data Delivery<a name="basic-deliver"></a>

After data is sent to your delivery stream, it is automatically delivered to the destination you choose\.

**Important**  
If you use the Kinesis Producer Library \(KPL\) to write data to a Kinesis data stream, you can use aggregation to combine the records that you write to that Kinesis data stream\. If you then use that data stream as a source for your Kinesis Data Firehose delivery stream, Kinesis Data Firehose de\-aggregates the records before it delivers them to the destination\. If you configure your delivery stream to transform the data, Kinesis Data Firehose de\-aggregates the records before it delivers them to AWS Lambda\. For more information, see [Developing Amazon Kinesis Data Streams Producers Using the Kinesis Producer Library](https://docs.aws.amazon.com/streams/latest/dev/developing-producers-with-kpl.html) and [Aggregation](https://docs.aws.amazon.com/streams/latest/dev/kinesis-kpl-concepts.html#kinesis-kpl-concepts-aggretation) in the *Amazon Kinesis Data Streams Developer Guide*\.

**Topics**
+ [Data Delivery Format](#format)
+ [Data Delivery Frequency](#frequency)
+ [Data Delivery Failure Handling](#retry)
+ [Amazon S3 Object Name Format](#s3-object-name)
+ [Index Rotation for the Amazon ES Destination](#es-index-rotation)
+ [Delivery Across AWS Accounts and Across AWS Regions for HTTP Endpoint Destinations](#across)
+ [Duplicated Records](#duplication)

## Data Delivery Format<a name="format"></a>

For data delivery to Amazon Simple Storage Service \(Amazon S3\), Kinesis Data Firehose concatenates multiple incoming records based on the buffering configuration of your delivery stream\. It then delivers the records to Amazon S3 as an Amazon S3 object\. You might want to add a record separator at the end of each record before you send it to Kinesis Data Firehose\. Then you can divide a delivered Amazon S3 object to individual records\. 

For data delivery to Amazon Redshift, Kinesis Data Firehose first delivers incoming data to your S3 bucket in the format described earlier\. Kinesis Data Firehose then issues an Amazon Redshift COPY command to load the data from your S3 bucket to your Amazon Redshift cluster\. Ensure that after Kinesis Data Firehose concatenates multiple incoming records to an Amazon S3 object, the Amazon S3 object can be copied to your Amazon Redshift cluster\. For more information, see [Amazon Redshift COPY Command Data Format Parameters](https://docs.aws.amazon.com/redshift/latest/dg/copy-parameters-data-format.html)\.

For data delivery to Amazon ES, Kinesis Data Firehose buffers incoming records based on the buffering configuration of your delivery stream\. It then generates an Elasticsearch bulk request to index multiple records to your Elasticsearch cluster\. Make sure that your record is UTF\-8 encoded and flattened to a single\-line JSON object before you send it to Kinesis Data Firehose\. Also, the `rest.action.multi.allow_explicit_index` option for your Elasticsearch cluster must be set to true \(default\) to take bulk requests with an explicit index that is set per record\. For more information, see [Amazon ES Configure Advanced Options](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-createupdatedomains.html#es-createdomain-configure-advanced-options) in the *Amazon Elasticsearch Service Developer Guide*\. 

For data delivery to Splunk, Kinesis Data Firehose concatenates the bytes that you send\. If you want delimiters in your data, such as a new line character, you must insert them yourself\. Make sure that Splunk is configured to parse any such delimiters\.

When delivering data to an HTTP endpoint owned by a supported third\-party service provider, you can use the integrated Amazon Lambda service to create a function to transform the incoming record\(s\) to the format that matches the format the service provider's integration is expecting\. Contact the third\-party service provider whose HTTP endpoint you've chosen for your destination to learn more about their accepted record format\. 

## Data Delivery Frequency<a name="frequency"></a>

Each Kinesis Data Firehose destination has its own data delivery frequency\.

**Amazon S3**  
The frequency of data delivery to Amazon S3 is determined by the Amazon S3 **Buffer size** and **Buffer interval** value that you configured for your delivery stream\. Kinesis Data Firehose buffers incoming data before it delivers it to Amazon S3\. You can configure the values for Amazon S3 **Buffer size** \(1–128 MB\) or **Buffer interval** \(60–900 seconds\)\. The condition satisfied first triggers data delivery to Amazon S3\. When data delivery to the destination falls behind data writing to the delivery stream, Kinesis Data Firehose raises the buffer size dynamically\. It can then catch up and ensure that all data is delivered to the destination\. 

**Amazon Redshift**  
The frequency of data COPY operations from Amazon S3 to Amazon Redshift is determined by how fast your Amazon Redshift cluster can finish the COPY command\. If there is still data to copy, Kinesis Data Firehose issues a new COPY command as soon as the previous COPY command is successfully finished by Amazon Redshift\. 

**Amazon Elasticsearch Service**  
The frequency of data delivery to Amazon ES is determined by the Elasticsearch **Buffer size** and **Buffer interval** values that you configured for your delivery stream\. Kinesis Data Firehose buffers incoming data before delivering it to Amazon ES\. You can configure the values for Elasticsearch **Buffer size** \(1–100 MB\) or **Buffer interval** \(60–900 seconds\), and the condition satisfied first triggers data delivery to Amazon ES\. 

**Splunk**  
Kinesis Data Firehose buffers incoming data before delivering it to Splunk\. The buffer size is 5 MB, and the buffer interval is 60 seconds\. The condition satisfied first triggers data delivery to Splunk\. The buffer size and interval aren't configurable\. These numbers are optimal\.

**HTTP endpoint destination**  
Kinesis Data Firehose buffers incoming data before delivering it to the specified HTTP endpoint destination\. For the HTTP endpoint destinations, including Datadog, MongoDB, and New Relic you can choose a buffer size of 1\-64 MiBs and a buffer interval \(60\-900 seconds\)\. The recommended buffer size for the destination varies from service provider to service provider\. For example, the recommended buffer size for Datadog is 4 MiBs and the recommended buffer size for New Relic is 1 MiB\. Contact the third\-party service provider whose endpoint you've chosen as your data destination for more information about their recommended buffer size\.

## Data Delivery Failure Handling<a name="retry"></a>

Each Kinesis Data Firehose destination has its own data delivery failure handling\.

**Amazon S3**  
Data delivery to your S3 bucket might fail for various reasons\. For example, the bucket might not exist anymore, the IAM role that Kinesis Data Firehose assumes might not have access to the bucket, the network failed, or similar events\. Under these conditions, Kinesis Data Firehose keeps retrying for up to 24 hours until the delivery succeeds\. The maximum data storage time of Kinesis Data Firehose is 24 hours\. If data delivery fails for more than 24 hours, your data is lost\.

**Amazon Redshift**  
For an Amazon Redshift destination, you can specify a retry duration \(0–7200 seconds\) when creating a delivery stream\.  
Data delivery to your Amazon Redshift cluster might fail for several reasons\. For example, you might have an incorrect cluster configuration of your delivery stream, a cluster under maintenance, or a network failure\. Under these conditions, Kinesis Data Firehose retries for the specified time duration and skips that particular batch of Amazon S3 objects\. The skipped objects' information is delivered to your S3 bucket as a manifest file in the `errors/` folder, which you can use for manual backfill\. For information about how to COPY data manually with manifest files, see [Using a Manifest to Specify Data Files](https://docs.aws.amazon.com/redshift/latest/dg/loading-data-files-using-manifest.html)\. 

**Amazon Elasticsearch Service**  
For the Amazon ES destination, you can specify a retry duration \(0–7200 seconds\) when creating a delivery stream\.  
Data delivery to your Amazon ES cluster might fail for several reasons\. For example, you might have an incorrect Amazon ES cluster configuration of your delivery stream, an Amazon ES cluster under maintenance, a network failure, or similar events\. Under these conditions, Kinesis Data Firehose retries for the specified time duration and then skips that particular index request\. The skipped documents are delivered to your S3 bucket in the `elasticsearch_failed/` folder, which you can use for manual backfill\. Each document has the following JSON format:  

```
{
    "attemptsMade": "(number of index requests attempted)",
    "arrivalTimestamp": "(the time when the document was received by Firehose)",
    "errorCode": "(http error code returned by Elasticsearch)",
    "errorMessage": "(error message returned by Elasticsearch)",
    "attemptEndingTimestamp": "(the time when Firehose stopped attempting index request)",
    "esDocumentId": "(intended Elasticsearch document ID)",
    "esIndexName": "(intended Elasticsearch index name)",
    "esTypeName": "(intended Elasticsearch type name)",
    "rawData": "(base64-encoded document data)"
}
```

**Splunk**  
When Kinesis Data Firehose sends data to Splunk, it waits for an acknowledgment from Splunk\. If an error occurs, or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time Kinesis Data Firehose sends data to Splunk, whether it's the initial attempt or a retry, it restarts the acknowledgement timeout counter\. It then waits for an acknowledgement to arrive from Splunk\. Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout is reached\. If the acknowledgment times out, Kinesis Data Firehose checks to determine whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
A failure to receive an acknowledgement isn't the only type of data delivery error that can occur\. For information about the other types of data delivery errors, see [Splunk Data Delivery Errors](https://docs.aws.amazon.com/firehose/latest/dev/monitoring-with-cloudwatch-logs.html#monitoring-splunk-errors)\. Any data delivery error triggers the retry logic if your retry duration is greater than 0\.  
The following is an example error record\.  

```
{
  "attemptsMade": 0,
  "arrivalTimestamp": 1506035354675,
  "errorCode": "Splunk.AckTimeout",
  "errorMessage": "Did not receive an acknowledgement from HEC before the HEC acknowledgement timeout expired. Despite the acknowledgement timeout, it's possible the data was indexed successfully in Splunk. Kinesis Firehose backs up in Amazon S3 data for which the acknowledgement timeout expired.",
  "attemptEndingTimestamp": 13626284715507,
  "rawData": "MiAyNTE2MjAyNzIyMDkgZW5pLTA1ZjMyMmQ1IDIxOC45Mi4xODguMjE0IDE3Mi4xNi4xLjE2NyAyNTIzMyAxNDMzIDYgMSA0MCAxNTA2MDM0NzM0IDE1MDYwMzQ3OTQgUkVKRUNUIE9LCg==",
  "EventId": "49577193928114147339600778471082492393164139877200035842.0"
}
```

**HTTP endpoint destination**  
When Kinesis Data Firehose sends data to an HTTP endpoint destination, it waits for a response from this destination\. If an error occurs, or the response doesn’t arrive within the response timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time Kinesis Data Firehose sends data to an HTTP endpoint destination, whether it's the initial attempt or a retry, it restarts the response timeout counter\. It then waits for a response to arrive from the HTTP endpoint destination\. Even if the retry duration expires, Kinesis Data Firehose still waits for the response until it receives it or the response timeout is reached\. If the response times out, Kinesis Data Firehose checks to determine whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives a response or determines that the retry time has expired\.  
A failure to receive a response isn't the only type of data delivery error that can occur\. For information about the other types of data delivery errors, see [HTTP Endpoint Data Delivery Errors](https://docs.aws.amazon.com/firehose/latest/dev/monitoring-with-cloudwatch-logs.html#monitoring-http-errors)  
The following is an example error record\.  

```
{
	"attemptsMade":5,
	"arrivalTimestamp":1594265943615,
	"errorCode":"HttpEndpoint.DestinationException",
	"errorMessage":"Received the following response from the endpoint destination. {"requestId": "109777ac-8f9b-4082-8e8d-b4f12b5fc17b", "timestamp": 1594266081268, "errorMessage": "Unauthorized"}", 
	"attemptEndingTimestamp":1594266081318,
	"rawData":"c2FtcGxlIHJhdyBkYXRh",
	"subsequenceNumber":0,
	"dataId":"49607357361271740811418664280693044274821622880012337186.0"
}
```

## Amazon S3 Object Name Format<a name="s3-object-name"></a>

Kinesis Data Firehose adds a UTC time prefix in the format `YYYY/MM/dd/HH` before writing objects to Amazon S3\. This prefix creates a logical hierarchy in the bucket, where each forward slash \(`/`\) creates a level in the hierarchy\. You can modify this structure by specifying a custom prefix\. For information about how to specify a custom prefix, see [Custom Prefixes for Amazon S3 Objects](s3-prefixes.md)\. 

The Amazon S3 object name follows the pattern `DeliveryStreamName-DeliveryStreamVersion-YYYY-MM-dd-HH-MM-SS-RandomString`, where `DeliveryStreamVersion` begins with `1` and increases by 1 for every configuration change of the Kinesis Data Firehose delivery stream\. You can change delivery stream configurations \(for example, the name of the S3 bucket, buffering hints, compression, and encryption\)\. You can do so by using the Kinesis Data Firehose console or the [UpdateDestination](https://docs.aws.amazon.com/firehose/latest/APIReference/API_UpdateDestination.html) API operation\.

## Index Rotation for the Amazon ES Destination<a name="es-index-rotation"></a>

For the Amazon ES destination, you can specify a time\-based index rotation option from one of the following five options: NoRotation, OneHour, OneDay, OneWeek, or OneMonth\.

Depending on the rotation option you choose, Kinesis Data Firehose appends a portion of the UTC arrival timestamp to your specified index name\. It rotates the appended timestamp accordingly\. The following example shows the resulting index name in Amazon ES for each index rotation option, where the specified index name is myindex and the arrival timestamp is `2016-02-25T13:00:00Z`\. 


| RotationPeriod | IndexName | 
| --- | --- | 
| NoRotation | myindex | 
| OneHour | myindex\-2016\-02\-25\-13 | 
| OneDay | myindex\-2016\-02\-25 | 
| OneWeek | myindex\-2016\-w08 | 
| OneMonth | myindex\-2016\-02 | 

## Delivery Across AWS Accounts and Across AWS Regions for HTTP Endpoint Destinations<a name="across"></a>

Kinesis Data Firehose supports data delivery to HTTP endpoint destinations across AWS accounts\. Kinesis Data Firehose delivery stream and the HTTP endpoint that you've chosen as your destination can be in different AWS accounts\.

Kinesis Data Firehose also supports data delivery to HTTP endpoint destinations across AWS regions\. You can deliver data from a delivery stream in one AWS region to an HTTP endpoint in another AWS region\. You can also delivery data from a delivery stream to an HTTP endpoint destination outside of AWS regions, for example to your own on\-premises server by setting the HTTP endpoint URL to your desired destination\. For these scenarios, additional data transfer charges are added to your delivery costs\. For more information, see the [Data Transfer](https://aws.amazon.com/ec2/pricing/on-demand/#Data_Transfer) section in the "On\-Demand Pricing" page\.

## Duplicated Records<a name="duplication"></a>

Kinesis Data Firehose uses at\-least\-once semantics for data delivery\. In some circumstances, such as when data delivery times out, delivery retries by Kinesis Data Firehose might introduce duplicates if the original data\-delivery request eventually goes through\. This applies to all destination types that Kinesis Data Firehose supports\.