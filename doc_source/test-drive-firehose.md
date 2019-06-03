# Testing Your Delivery Stream Using Sample Data<a name="test-drive-firehose"></a>

You can use the AWS Management Console to ingest simulated stock ticker data\. The console runs a script in your browser to put sample records in your Kinesis Data Firehose delivery stream\. This enables you to test the configuration of your delivery stream without having to generate your own test data\.

The following is an example from the simulated data:

```
{"TICKER_SYMBOL":"QXZ","SECTOR":"HEALTHCARE","CHANGE":-0.05,"PRICE":84.51}
```

Note that standard Amazon Kinesis Data Firehose charges apply when your delivery stream transmits the data, but there is no charge when the data is generated\. To stop incurring these charges, you can stop the sample stream from the console at any time\.

**Topics**
+ [Prerequisites](#test-drive-requirements)
+ [Test Using Amazon S3 as the Destination](#test-drive-destination-s3)
+ [Test Using Amazon Redshift as the Destination](#test-drive-destination-redshift)
+ [Test Using Amazon ES as the Destination](#test-drive-destination-elasticsearch)
+ [Test Using Splunk as the Destination](#test-drive-destination-splunk)

## Prerequisites<a name="test-drive-requirements"></a>

Before you begin, create a delivery stream\. For more information, see [Creating an Amazon Kinesis Data Firehose Delivery Stream](basic-create.md)\.

## Test Using Amazon S3 as the Destination<a name="test-drive-destination-s3"></a>

Use the following procedure to test your delivery stream using Amazon Simple Storage Service \(Amazon S3\) as the destination\.

**To test a delivery stream using Amazon S3**

1. Open the Kinesis Data Firehose console at [https://console\.aws\.amazon\.com/firehose/](https://console.aws.amazon.com/firehose/)\.

1. Choose the delivery stream\.

1. Under **Test with demo data**, choose **Start sending demo data** to generate sample stock ticker data\.

1. Follow the onscreen instructions to verify that data is being delivered to your S3 bucket\. Note that it might take a few minutes for new objects to appear in your bucket, based on the buffering configuration of your bucket\.

1. When the test is complete, choose **Stop sending demo data** to stop incurring usage charges\.

## Test Using Amazon Redshift as the Destination<a name="test-drive-destination-redshift"></a>

Use the following procedure to test your delivery stream using Amazon Redshift as the destination\.

**To test a delivery stream using Amazon Redshift**

1. Your delivery stream expects a table to be present in your Amazon Redshift cluster\. [Connect to Amazon Redshift through a SQL interface](https://docs.aws.amazon.com/redshift/latest/mgmt/connecting-to-cluster.html) and run the following statement to create a table that accepts the sample data\.

   ```
   create table firehose_test_table
   (
   	TICKER_SYMBOL varchar(4),
   	SECTOR varchar(16),
   	CHANGE float,
   	PRICE float
   );
   ```

1. Open the Kinesis Data Firehose console at [https://console\.aws\.amazon\.com/firehose/](https://console.aws.amazon.com/firehose/)\.

1. Choose the delivery stream\.

1. Edit the destination details for your delivery stream to point to the newly created `firehose_test_table` table\.

1. Under **Test with demo data**, choose **Start sending demo data** to generate sample stock ticker data\.

1. Follow the onscreen instructions to verify that data is being delivered to your table\. Note that it might take a few minutes for new rows to appear in your table, based on the buffering configuration\.

1. When the test is complete, choose **Stop sending demo data** to stop incurring usage charges\.

1. Edit the destination details for your Kinesis Data Firehose delivery stream to point to another table\.

1. \(Optional\) Delete the `firehose_test_table` table\.

## Test Using Amazon ES as the Destination<a name="test-drive-destination-elasticsearch"></a>

Use the following procedure to test your delivery stream using Amazon Elasticsearch Service \(Amazon ES\) as the destination\.

**To test a delivery stream using Amazon ES**

1. Open the Kinesis Data Firehose console at [https://console\.aws\.amazon\.com/firehose/](https://console.aws.amazon.com/firehose/)\.

1. Choose the delivery stream\.

1. Under **Test with demo data**, choose **Start sending demo data** to generate sample stock ticker data\.

1. Follow the onscreen instructions to verify that data is being delivered to your Amazon ES domain\. For more information, see [Searching Documents in an Amazon ES Domain](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-gsg-search.html) in the *Amazon Elasticsearch Service Developer Guide*\.

1. When the test is complete, choose **Stop sending demo data** to stop incurring usage charges\.

## Test Using Splunk as the Destination<a name="test-drive-destination-splunk"></a>

Use the following procedure to test your delivery stream using Splunk as the destination\.

**To test a delivery stream using Splunk**

1. Open the Kinesis Data Firehose console at [https://console\.aws\.amazon\.com/firehose/](https://console.aws.amazon.com/firehose/)\.

1. Choose the delivery stream\.

1. Under **Test with demo data**, choose **Start sending demo data** to generate sample stock ticker data\.

1. Check whether the data is being delivered to your Splunk index\. Example search terms in Splunk are `sourcetype="aws:firehose:json"` and `index="name-of-your-splunk-index"`\. For more information about how to search for events in Splunk, see [Search Manual](http://docs.splunk.com/Documentation/Splunk/latest/Search/GetstartedwithSearch) in the Splunk documentation\.

   If the test data doesn't appear in your Splunk index, check your Amazon S3 bucket for failed events\. Also see [Data Not Delivered to Splunk](https://docs.aws.amazon.com/firehose/latest/dev/troubleshooting.html#data-not-delivered-to-splunk)\.

1. When you finish testing, choose **Stop sending demo data** to stop incurring usage charges\.