# Amazon Kinesis Firehose Limits<a name="limits"></a>

Amazon Kinesis Firehose has the following limits\. 

+ By default, each account can have up to 20 Kinesis Firehose delivery streams per Region\. This limit can be increased using the [Amazon Kinesis Firehose Limits form](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-kinesis-firehose)\.

+ When **Direct PUT** is configured as the data source, each Kinesis Firehose delivery stream accepts by default a maximum of 2,000 transactions/second, 5,000 records/second, and 5 MB/second\. You can submit a limit increase request using the [Amazon Kinesis Firehose Limits form](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-kinesis-firehose)\. The three limits scale proportionally\. For example, if you increase the throughput limit to 10 MB/second, the other two limits increase to 4,000 transactions/second and 10,000 records/second\.
**Important**  
If the increased limit is much higher than the running traffic, it causes very small delivery batches to destinations, which is inefficient and can result in higher costs at the destination services\. Be sure to increase the limit only to match current running traffic, and increase the limit further if traffic increases\.
**Note**  
When Kinesis Streams is configured as the data source, this limit doesn't apply, and Kinesis Firehose scales up and down with no limit\. 

+ Each Kinesis Firehose delivery stream stores data records for up to 24 hours in case the delivery destination is unavailable\.

+ The maximum size of a record sent to Kinesis Firehose, before base64\-encoding, is 1000 KB\.

+ The [PutRecordBatch](http://docs.aws.amazon.com/firehose/latest/APIReference/API_PutRecordBatch.html) operation can take up to 500 records per call or 4 MB per call, whichever is smaller\. This limit cannot be changed\.

+ The following operations can provide up to 5 transactions per second: [CreateDeliveryStream](http://docs.aws.amazon.com/firehose/latest/APIReference/API_CreateDeliveryStream.html), [DeleteDeliveryStream](http://docs.aws.amazon.com/firehose/latest/APIReference/API_DeleteDeliveryStream.html), [DescribeDeliveryStream](http://docs.aws.amazon.com/firehose/latest/APIReference/API_DescribeDeliveryStream.html), [ListDeliveryStreams](http://docs.aws.amazon.com/firehose/latest/APIReference/API_ListDeliveryStreams.html), and [UpdateDestination](http://docs.aws.amazon.com/firehose/latest/APIReference/API_UpdateDestination.html)\.

+ The buffer sizes hints range from 1 MB to 128 MB for Amazon S3 delivery, and 1 MB to 100 MB for Amazon Elasticsearch Service delivery\. The size threshold is applied to the buffer before compression\.

+ The buffer interval hints range from 60 seconds to 900 seconds\.

+ For Kinesis Firehose to Amazon Redshift delivery, only publicly accessible Amazon Redshift clusters are supported\.

+ The retry duration range is from 0 seconds to 7200 seconds for Amazon Redshift and Amazon ES delivery\.

+ Kinesis Firehose doesn't currently support Elasticsearch 6\.0\. Data delivery from Kinesis Firehose to Elasticsearch 6\.0 fails\.