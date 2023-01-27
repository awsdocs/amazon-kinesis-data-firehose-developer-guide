# Dynamic Partitioning in Kinesis Data Firehose<a name="dynamic-partitioning"></a>

Dynamic partitioning enables you to continuously partition streaming data in Kinesis Data Firehose by using keys within data \(for example, `customer_id` or `transaction_id`\) and then deliver the data grouped by these keys into corresponding Amazon Simple Storage Service \(Amazon S3\) prefixes\. This makes it easier to run high performance, cost\-efficient analytics on streaming data in Amazon S3 using various services such as Amazon Athena, Amazon EMR, Amazon Redshift Spectrum, and Amazon QuickSight\. In addition, AWS Glue can perform more sophisticated extract, transform, and load \(ETL\) jobs after the dynamically partitioned streaming data is delivered to Amazon S3, in use\-cases where additional processing is required\.

Partitioning your data minimizes the amount of data scanned, optimizes performance, and reduces costs of your analytics queries on Amazon S3\. It also increases granular access to your data\. Kinesis Data Firehose delivery streams are traditionally used in order to capture and load data into Amazon S3\. To partition a streaming data set for Amazon S3\-based analytics, you would need to run partitioning applications between Amazon S3 buckets prior to making the data available for analysis, which could become complicated or costly\. 

With dynamic partitioning, Kinesis Data Firehose continuously groups in\-transit data using dynamically or statically defined data keys, and delivers the data to individual Amazon S3 prefixes by key\. This reduces time\-to\-insight by minutes or hours\. It also reduces costs and simplifies architectures\. 

**Topics**
+ [Partitioning keys](#dynamic-partitioning-partitioning-keys)
+ [Amazon S3 Bucket Prefix for Dynamic Partitioning](#dynamic-partitioning-s3bucketprefix)
+ [Dynamic partitioning of aggregated data](#dynamic-partitioning-multirecord-deaggergation)
+ [Adding a new line delimiter when delivering data to S3](#dynamic-partitioning-new-line-delimiter)
+ [How to enable dynamic partitioning](#dynamic-partitioning-enable)
+ [Dynamic Partitioning Error Handling](#dynamic-partitioning-error-handling)
+ [Data buffering and dynamic partitioning](#buffering)

## Partitioning keys<a name="dynamic-partitioning-partitioning-keys"></a>

With dynamic partitioning, you create targeted data sets from the streaming S3 data by partitioning the data based on partitioning keys\. Partitioning keys enable you to filter your streaming data based on specific values\. For example, if you need to filter your data based on customer ID and country, you can specify the data field of `customer_id` as one partitioning key and the data field of `country` as another partitioning key\. Then, you specify the expressions \(using the supported formats\) to define the S3 bucket prefixes to which the dynamically partitioned data records are to be delivered\. 

The following are the supported methods of creating partitioning keys:
+ **Inline parsing** \- this method uses Amazon Kinesis Data Firehose built\-in support mechanism, a [jq parser](https://stedolan.github.io/jq/), for extracting the keys for partitioning from data records that are in JSON format\.
+ **AWS Lambda function** \- this method uses a specified AWS Lambda function to extract and return the data fields needed for partitioning\.

**Important**  
When you enable dynamic partitioning, you must configure at least one of these methods to partition your data\. You can configure either of these methods to specify your partitioning keys or both of them at the same time\. 

### Creating partitioning keys with inline parsing<a name="dynamic-partitioning-inline-parsing"></a>

To configure inline parsing as the dynamic partitioning method for your streaming data, you must choose data record parameters to be used as partitioning keys and provide a value for each specified partitioning key\.

Let's look at the following sample data record and and see how you can define partitioning keys for it with inline parsing:

```
{  
   "type": {  
    "device": "mobile",  
    "event": "user_clicked_submit_button" 
  },  
  "customer_id": "1234567890",  
  "event_timestamp": 1565382027,   #epoch timestamp  
  "region": "sample_region"  
}
```

For example, you can choose to partition your data based on the `customer_id` parameter or the `event_timestamp` parameter\. This means that you want the value of the `customer_id` parameter or the `event_timestamp` parameter in each record to be used in determining the S3 prefix to which the record is to be delivered\. You can also choose a nested parameter, like `device` with an expression `.type.device`\. Your dynamic partitioning logic can depend on multiple parameters\.

After selecting data parameters for your partitioning keys, you then map each parameter to a valid jq expression\. The following table shows such a mapping of parameters to jq expressions:


| Parameter | jq expression | 
| --- | --- | 
| customer\_id | \.customer\_id | 
| device |  \.type\.device  | 
| year |  \.event\_timestamp\| strftime\("%Y"\)  | 
| month |  \.event\_timestamp\| strftime\("%m"\)  | 
| day |  \.event\_timestamp\| strftime\("%d"\)  | 
| hour |  \.event\_timestamp\| strftime\("%H"\)  | 

At runtime, Kinesis Data Firehose uses the right column above to evaluate the parameters based on the data in each record\.

### Creating partitioning keys with an AWS Lambda function<a name="dynamic-partitioning-with-lambda"></a>

For compressed or encrypted data records, or data that is in any file format other than JSON, you can use the integrated AWS Lambda function with your own custom code to decompress, decrypt, or transform the records in order to extract and return the data fields needed for partitioning\. This is an expansion of the existing transform Lambda function that is available today with Kinesis Data Firehose\. You can transform, parse and return the data fields that you can then use for dynamic partitioning using the same Lambda function\.

The following is an example Amazon Kinesis Firehose delivery stream processing Lambda function in Python that replays every read record from input to output and extracts partitionig keys from the records\.

```
from __future__ import print_function
import base64
import json
import datetime
 
# Signature for all Lambda functions that user must implement
def lambda_handler(firehose_records_input, context):
    print("Received records for processing from DeliveryStream: " + firehose_records_input['deliveryStreamArn']
          + ", Region: " + firehose_records_input['region']
          + ", and InvocationId: " + firehose_records_input['invocationId'])
 
    # Create return value.
    firehose_records_output = {'records': []}
 
    # Create result object.
    # Go through records and process them
 
    for firehose_record_input in firehose_records_input['records']:
        # Get user payload
        payload = base64.b64decode(firehose_record_input['data'])
        json_value = json.loads(payload)
 
        print("Record that was received")
        print(json_value)
        print("\n")
        # Create output Firehose record and add modified payload and record ID to it.
        firehose_record_output = {}
        event_timestamp = datetime.datetime.fromtimestamp(json_value['eventTimestamp'])
        partition_keys = {"customerId": json_value['customerId'],
                          "year": event_timestamp.strftime('%Y'),
                          "month": event_timestamp.strftime('%m'),
                          "date": event_timestamp.strftime('%d'),
                          "hour": event_timestamp.strftime('%H'),
                          "minute": event_timestamp.strftime('%M')
                          }
 
        # Create output Firehose record and add modified payload and record ID to it.
        firehose_record_output = {'recordId': firehose_record_input['recordId'],
                                  'data': firehose_record_input['data'],
                                  'result': 'Ok',
                                  'metadata': { 'partitionKeys': partition_keys }}
 
        # Must set proper record ID
        # Add the record to the list of output records.
 
        firehose_records_output['records'].append(firehose_record_output)
 
    # At the end return processed records
    return firehose_records_output
```

The following is an example Amazon Kinesis Firehose delivery stream processing Lambda function in Go that replays every read record from input to output and extracts partioninig keys from the records\.

```
package main

import (
	"fmt"
	"encoding/json"
	"time"
	"strconv"

	"github.com/aws/aws-lambda-go/events"
	"github.com/aws/aws-lambda-go/lambda"
)

type KinesisFirehoseEventRecordData struct {
	CustomerId string `json:"customerId"`
}

func handleRequest(evnt events.KinesisFirehoseEvent) (events.KinesisFirehoseResponse, error) {

	fmt.Printf("InvocationID: %s\n", evnt.InvocationID)
	fmt.Printf("DeliveryStreamArn: %s\n", evnt.DeliveryStreamArn)
	fmt.Printf("Region: %s\n", evnt.Region)

	var response events.KinesisFirehoseResponse

	for _, record := range evnt.Records {
		fmt.Printf("RecordID: %s\n", record.RecordID)
		fmt.Printf("ApproximateArrivalTimestamp: %s\n", record.ApproximateArrivalTimestamp)

		var transformedRecord events.KinesisFirehoseResponseRecord
		transformedRecord.RecordID = record.RecordID
		transformedRecord.Result = events.KinesisFirehoseTransformedStateOk
		transformedRecord.Data = record.Data

		var metaData events.KinesisFirehoseResponseRecordMetadata
		var recordData KinesisFirehoseEventRecordData
		partitionKeys := make(map[string]string)

		currentTime := time.Now()
		json.Unmarshal(record.Data, &recordData)
		partitionKeys["customerId"] = recordData.CustomerId
		partitionKeys["year"] = strconv.Itoa(currentTime.Year())
		partitionKeys["month"] = strconv.Itoa(int(currentTime.Month()))
		partitionKeys["date"] = strconv.Itoa(currentTime.Day())
		partitionKeys["hour"] = strconv.Itoa(currentTime.Hour())
		partitionKeys["minute"] = strconv.Itoa(currentTime.Minute())
		metaData.PartitionKeys = partitionKeys
		transformedRecord.Metadata = metaData

		response.Records = append(response.Records, transformedRecord)
	}

	return response, nil
}

func main() {
	lambda.Start(handleRequest)
}
```

## Amazon S3 Bucket Prefix for Dynamic Partitioning<a name="dynamic-partitioning-s3bucketprefix"></a>

When you create a delivery stream that uses Amazon S3 as the destination, you must specify an Amazon S3 bucket where Kinesis Data Firehose is to deliver your data\. Amazon S3 bucket prefixes are used to organize the data that you store in your S3 buckets\. An Amazon S3 bucket prefix is similar to a directory that enables you to group similar objects together\.

With dynamic partitioning, your partitioned data is delivered into the specified Amazon S3 prefixes\. If you don't enable dynamic partitioning, specifying an S3 bucket prefix for your delivery stream is optional\. However, if you choose to enable dynamic partitioning, you MUST specify the S3 bucket prefixes to which Kinesis Data Firehose is to deliver partitioned data\. 

In every delivery stream where you enable dynamic partitioning, the S3 bucket prefix value consists of expressions based on the specified partitioning keys for that delivery stream\. Using the above data record example again, you can build the following S3 prefix value that consists of expressions based on the partitioning keys defined above:

```
"ExtendedS3DestinationConfiguration": {  
"BucketARN": "arn:aws:s3:::my-logs-prod",  
"Prefix": "customer_id=!{partitionKeyFromQuery:customer_id}/ 
    device=!{partitionKeyFromQuery:device}/ 
    year=!{partitionKeyFromQuery:year}/  
    month=!{partitionKeyFromQuery:month}/  
    day=!{partitionKeyFromQuery:day}/  
    hour=!{partitionKeyFromQuery:hour}/"  
}
```

Kinesis Data firehose evaluates the above expression at runtime\. It groups records that match the same evaluated S3 prefix expression into a single data set\. Kinesis Data Firehose then delivers each data set to the evaluated S3 prefix\. The frequency of data set delivery to S3 is determined by the delivery stream buffer setting\. As a result, the record in this example is delivered to the following S3 object key: 

```
s3://my-logs-prod/customer_id=1234567890/device=mobile/year=2019/month=08/day=09/hour=20/my-delivery-stream-2019-08-09-23-55-09-a9fa96af-e4e4-409f-bac3-1f804714faaa
```

For dynamic partitioning, you must use the following expression format in your S3 bucket prefix: `!{namespace:value}`, where namespace can be either `partitionKeyFromQuery` or `partitionKeyFromLambda`, or both\. If you are using inline parsing to create the partitioning keys for your source data, you must specify an S3 bucket prefix value that consists of expressions specified in the following format: `"partitionKeyFromQuery:keyID"`\. If you are using an AWS Lambda function to create partitioning keys for your source data, you must specify an S3 bucket prefix value that consists of expressions specified in the following format: `"partitionKeyFromLambda:keyID"`\. 

**Note**  
You can also specify the S3 bucket prefix value using the hive style format, for example customer\_id=\!\{partitionKeyFromQuery:customer\_id\}\. 

 For more information, see the "Choose Amazon S3 for Your Destination" in [Creating an Amazon Kinesis Data Firehose Delivery Stream](https://docs.aws.amazon.com/firehose/latest/dev/basic-create.html) and [Custom Prefixes for Amazon S3 Objects](https://docs.aws.amazon.com/firehose/latest/dev/s3-prefixes.html)\.

## Dynamic partitioning of aggregated data<a name="dynamic-partitioning-multirecord-deaggergation"></a>

You can apply dynamic partitioning to aggregated data \(for example, multiple events, logs, or records aggregated into a single `PutRecord` and `PutRecordBatch` API call\) but this data must first be deaggregated\. You can deaggregate your data by enabling multi record deaggregation \- the process of parsing through the records in the delivery stream and separating them\. Multi record deaggregation can either be of `JSON` type, meaning that the separation of records is performed based on valid JSON\. Or it can be of the `Delimited` type, meaning that the separation of records is performed based on a specified custom delimiter\. This custom delimiter must be a base\-64 encoded string\. For example, if you want to use the following string as your custom delimeter `####`, you must specify it in the base\-64 ecoded format, which translates it to `IyMjIw==`\.

 With aggregated data, when you enable dynamic partitioning, Kinesis Data Firehose parses the records and looks for either valid JSON objects or delimited records within each API call based on the specified multi record deaggregation type\.

**Important**  
If your data is aggregated, dynamic partitioning can be only be applied if your data is first deaggregated\. 

## Adding a new line delimiter when delivering data to S3<a name="dynamic-partitioning-new-line-delimiter"></a>

When you enable dynamic partitioning, you can configure your delivery stream to add a new line delimiter between records in objects that are delivered to Amazon S3\. This can be helpful for parsing objects in Amazon S3\. This is also particularly useful when dynamic partitioning is applied to aggregated data because multirecord deaggregation \(which must be applied to aggregated data before it can be dynamically partitioned\) removes new lines from records as part of the parsing process\.

## How to enable dynamic partitioning<a name="dynamic-partitioning-enable"></a>

You can configure dynamic partitioning for your delivery streams through the Kinesis Data Firehose Management Console, CLI, or the APIs\.

**Important**  
You can enable dynamic partitioning only when you create a new delivery stream\. You cannot enable dynamic partitioning for an existing delivery stream that does not have dynamic partitioning already enabled\. 

For detailed steps on how to enable and configure dynamic partitioning through the Amazon Kinesis Data Firehose management console while creating a new delivery stream, see [Creating an Amazon Kinesis Data Firehose Delivery Stream](https://docs.aws.amazon.com/firehose/latest/dev/basic-create.html)\. When you get to the task of specifying the destination for your delivery stream, make sure to follow the steps in the [Choose Amazon S3 for Your Destination](https://docs.aws.amazon.com/firehose/latest/dev/create-destination.html#create-destination-s3) section, since currently, dynamic partitioning is only supported for delivery streams that use Amazon S3 as the destination\.

Once dynamic partitioning on an active delivery stream is enabled, you can update the configuration by adding new or removing or updating existing partitioning keys and the S3 prefix expressions\. Once updated, Amazon Kinesis Data Firehose starts using the new keys and the new S3 prefix expressions\. 

**Important**  
Once you enable dynamic partitioning on a delivery stream, it cannot be disabled on this delivery stream\.

## Dynamic Partitioning Error Handling<a name="dynamic-partitioning-error-handling"></a>

If Kinesis Data Firehose is not able to parse data records in your delivery stream or it fails to extract the specified partitioning keys, or to evaluate the expressions included in the S3 prefix value, these data records are delivered to the S3 error bucket prefix that you must specify when you create the delivery stream where you enable dynamic partitioning\. The S3 error bucket prefix contains all the records that Kinesis Data Firehose is not able to deliver to the specified S3 destination\. These records are organized based on the error type\. Along with the record, the delivered object also includes information about the error to help understand and resolve the error\. 

You MUST specify an S3 error bucket prefix for a delivery stream if you want to enable dynamic partitioning for this delivery stream\. If you don't want to enable dynamic partitioning for a delivery stream, specifying an S3 error bucket prefix is optional\. 

## Data buffering and dynamic partitioning<a name="buffering"></a>

Amazon Kinesis Data Firehose buffers incoming streaming data to a certain size and for a certain period of time before delivering it to the specified destinations\. You can configure the buffer size and the buffer interval while creating new delivery streams or update the buffer size and the buffer interval on your existing delivery streams\. A buffer size is measured in MBs and a buffer interval is measured in seconds\.

When dynamic partitioning is enabled, Kinesis Data Firehose internally buffers records that belong to a given partition based on the configured buffering hint \(size and time\) before delivering these records to your Amazon S3 bucket\. In order to deliver maximum size objects, Kinesis Data Firehose uses multi\-stage buffering internally\. Therefore, end\-to\-end delay of a batch of records might be 1\.5 times of the configured buffering hint time\. This affects the data freshness of a delivery stream\. 

The active partition count is the total number of active partitions within the delivery buffer\. For example, if the dynamic partitioning query constructs 3 partitions per second and you have a buffer hint configuration triggering delivery every 60 seconds, then on average you would have 180 active partitions\. If Kinesis Data Firehose cannot deliver the data in a partition to a destination, this partition is counted as active in the delivery buffer until it can be delivered\. 

A new partition is created when an S3 prefix is evaluated to a new value based on the record data fields and the S3 prefix expressions\. A new buffer is created for each active partition\. Every subsequent record with the same evaluated S3 prefix is delivered to that buffer\. Once the buffer meets the buffer size limit or the buffer time interval, Amazon Kinesis Data Firehose creates an object with the buffer data and delivers it to the specified Amazon S3 prefix\. Once the object is delivered, the buffer for that partition and the partition itself are deleted and removed from the active partitions count\. Amazon Kinesis Data Firehose delivers each buffer data as a single object once the buffer size or interval are met for each partition separately\. Once the number of active partitions reaches the limit of 500 per deliver stream, the rest of the records in the delivery stream are delivered to the specified S3 error bucket prefix\.

