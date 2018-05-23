# Amazon Kinesis Data Firehose Data Delivery<a name="basic-deliver"></a>

After data is sent to your delivery stream, it is automatically delivered to the destination you choose\.

**Topics**
+ [Data Delivery Format](#format)
+ [Data Delivery Frequency](#frequency)
+ [Data Delivery Failure Handling](#retry)
+ [Amazon S3 Object Name Format](#s3-object-name)
+ [Index Rotation for the Amazon ES Destination](#es-index-rotation)

## Data Delivery Format<a name="format"></a>

For data delivery to Amazon S3, Kinesis Firehose concatenates multiple incoming records based on buffering configuration of your delivery stream, and then delivers them to Amazon S3 as an S3 object\. You may want to add a record separator at the end of each record before you send it to Kinesis Firehose so that you can divide a delivered S3 object to individual records\. 

For data delivery to Amazon Redshift, Kinesis Firehose first delivers incoming data to your S3 bucket in the format described earlier\. Kinesis Firehose then issues an Amazon Redshift COPY command to load the data from your S3 bucket to your Amazon Redshift cluster\. You need to make sure that after Kinesis Firehose concatenates multiple incoming records to an S3 object, the S3 object can be copied to your Amazon Redshift cluster\. For more information, see [Amazon Redshift COPY Command Data Format Parameters](http://docs.aws.amazon.com/redshift/latest/dg/copy-parameters-data-format.html)\.

For data delivery to Amazon ES, Kinesis Firehose buffers incoming records based on buffering configuration of your delivery stream\. It then generates an Elasticsearch bulk request to index multiple records to your Elasticsearch cluster\. You need to make sure that your record is UTF\-8 encoded and flattened to a single\-line JSON object before you send it to Kinesis Firehose\. Also, the `rest.action.multi.allow_explicit_index` option for your Elasticsearch cluster must be set to true \(default\) in order to take bulk requests with an explicit index that is set per record\. For more information, see [Amazon ES Configure Advanced Options](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide//es-createupdatedomains.html#es-createdomain-configure-advanced-options) in the *Amazon Elasticsearch Service Developer Guide*\. 

For data delivery to Splunk, Kinesis Data Firehose concatenates the bytes that you send\. If you want delimiters in your data, such as a new line character, you must insert them yourself\. Make sure that Splunk is configured to parse any such delimiters\.

## Data Delivery Frequency<a name="frequency"></a>

Each Kinesis Data Firehose destination has its own data delivery frequency\.

**Amazon S3**  
The frequency of data delivery to Amazon S3 is determined by the S3 **Buffer size** and **Buffer interval** value you configured for your delivery stream\. Kinesis Data Firehose buffers incoming data before delivering it to Amazon S3\. You can configure the values for S3 **Buffer size** \(1–128 MB\) or **Buffer interval** \(60–900 seconds\), and the condition satisfied first triggers data delivery to Amazon S3\. In circumstances where data delivery to the destination is falling behind data writing to the delivery stream, Kinesis Data Firehose raises the buffer size dynamically to catch up and make sure that all data is delivered to the destination\. 

**Amazon Redshift**  
The frequency of data COPY operations from Amazon S3 to Amazon Redshift is determined by how fast your Amazon Redshift cluster can finish the COPY command\. If there is still data to copy, Kinesis Data Firehose issues a new COPY command as soon as the previous COPY command is successfully finished by Amazon Redshift\. 

**Amazon Elasticsearch Service**  
The frequency of data delivery to Amazon ES is determined by the Elasticsearch **Buffer size** and **Buffer interval** values that you configured for your delivery stream\. Kinesis Data Firehose buffers incoming data before delivering it to Amazon ES\. You can configure the values for Elasticsearch **Buffer size** \(1–100 MB\) or **Buffer interval** \(60–900 seconds\), and the condition satisfied first triggers data delivery to Amazon ES\. 

**Splunk**  
Kinesis Data Firehose buffers incoming data before delivering it to Splunk\. The buffer size is 5 MB and the buffer interval is 60 seconds\. The condition satisfied first triggers data delivery to Splunk\. The buffer size and interval aren't configurable\. These numbers are optimal\.

## Data Delivery Failure Handling<a name="retry"></a>

Each Kinesis Data Firehose destination has its own data delivery failure handling\.

**Amazon S3**  
Data delivery to your S3 bucket might fail for reasons such as the bucket doesn’t exist anymore, the IAM role that Kinesis Firehose assumes doesn’t have access to the bucket, the network failed, or similar events\. Under these conditions, Kinesis Firehose keeps retrying for up to 24 hours until the delivery succeeds\. The maximum data storage time of Kinesis Firehose is 24 hours, and your data is lost if data delivery fails for more than 24 hours\.

**Amazon Redshift**  
For the Amazon Redshift destination, you can specify a retry duration \(0–7200 seconds\) when creating a delivery stream\.  
Data delivery to your Amazon Redshift cluster might fail for reasons such as an incorrect Amazon Redshift cluster configuration of your delivery stream, an Amazon Redshift cluster under maintenance, network failure, or similar events\. Under these conditions, Kinesis Data Firehose retries for the specified time duration and skips that particular batch of S3 objects\. The skipped objects' information is delivered to your S3 bucket as a manifest file in the `errors/` folder, which you can use for manual backfill\. For information about how to COPY data manually with manifest files, see [Using a Manifest to Specify Data Files](http://docs.aws.amazon.com/redshift/latest/dg/loading-data-files-using-manifest.html)\. 

**Amazon Elasticsearch Service**  
For the Amazon ES destination, you can specify a retry duration \(0–7200 seconds\) when creating a delivery stream\.  
Data delivery to your Amazon ES cluster might fail for reasons such as an incorrect Amazon ES cluster configuration of your delivery stream, an Amazon ES cluster under maintenance, network failure, or similar events\. Under these conditions, Kinesis Data Firehose retries for the specified time duration and then skips that particular index request\. The skipped documents are delivered to your S3 bucket in the `elasticsearch_failed/` folder, which you can use for manual backfill\. Each document has the following JSON format:  

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
When Kinesis Data Firehose sends data to Splunk, it waits for acknowledgment from Splunk\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Firehose starts the retry duration counter and keeps retrying until the retry duration expires, after which Kinesis Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time Kinesis Firehose sends data to Splunk, whether it's the initial attempt or a retry, it restarts the acknowledgement timeout counter and waits for an acknowledgement to arrive from Splunk\. Even if the retry duration expires, Kinesis Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout is reached\. If the acknowledgment times out, Kinesis Firehose checks to determine whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
A failure to receive an acknowledgement isn't the only kind of data delivery error that can occur\. For information about the other types of data delivery errors that you might encounter, see [Splunk Data Delivery Errors](http://docs.aws.amazon.com/firehose/latest/dev/monitoring-with-cloudwatch-logs.html#monitoring-splunk-errors)\. Any data delivery error triggers the retry logic if your retry duration is greater than 0\.  
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

## Amazon S3 Object Name Format<a name="s3-object-name"></a>

Kinesis Data Firehose adds a UTC time prefix in the format `YYYY/MM/DD/HH` before writing objects to Amazon S3\. This prefix creates a logical hierarchy in the bucket, where each forward slash \(`/`\) creates a level in the hierarchy\. You can modify this structure by adding to the start of the prefix when you create the Kinesis data delivery stream\. For example, add `myApp/` to use the `myApp/YYYY/MM/DD/HH` prefix or `myApp` to use the `myApp YYYY/MM/DD/HH` prefix\.

The S3 object name follows the pattern `DeliveryStreamName-DeliveryStreamVersion-YYYY-MM-DD-HH-MM-SS-RandomString`, where `DeliveryStreamVersion` begins with `1` and increases by 1 for every configuration change of the Kinesis data delivery stream\. You can change Kinesis data delivery stream configurations \(for example, the name of the S3 bucket, buffering hints, compression, and encryption\) using the Kinesis Data Firehose console, or by using the [UpdateDestination](http://docs.aws.amazon.com/firehose/latest/APIReference/API_UpdateDestination.html) API operation\.

## Index Rotation for the Amazon ES Destination<a name="es-index-rotation"></a>

For the Amazon ES destination, you can specify a time\-based index rotation option from one of the following five options: NoRotation, OneHour, OneDay, OneWeek, or OneMonth\.

Depending on the rotation option you choose, Kinesis Data Firehose appends a portion of the UTC arrival time stamp to your specified index name, and rotates the appended time stamp accordingly\. The following example shows the resulting index name in Amazon ES for each index rotation option, where the specified index name is myindex and the arrival time stamp is 2016\-02\-25T13:00:00Z\. 


| RotationPeriod | IndexName | 
| --- | --- | 
| NoRotation | myindex | 
| OneHour | myindex\-2016\-02\-25\-13 | 
| OneDay | myindex\-2016\-02\-25 | 
| OneWeek | myindex\-2016\-w08 | 
| OneMonth | myindex\-2016\-02 | 