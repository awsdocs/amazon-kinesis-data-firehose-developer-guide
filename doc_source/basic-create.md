# Creating an Amazon Kinesis Data Firehose Delivery Stream<a name="basic-create"></a>

You can use the AWS Management Console or an AWS SDK to create a Kinesis Data Firehose delivery stream to your chosen destination\. 

You can update the configuration of your delivery stream at any time after it’s created, using the Kinesis Data Firehose console or [UpdateDestination](https://docs.aws.amazon.com/firehose/latest/APIReference/API_UpdateDestination.html)\. Your Kinesis Data Firehose delivery stream remains in the `ACTIVE` state while your configuration is updated, and you can continue to send data\. The updated configuration normally takes effect within a few minutes\. The version number of a Kinesis Data Firehose delivery stream is increased by a value of `1` after you update the configuration\. It is reflected in the delivered Amazon S3 object name\. For more information, see [Amazon S3 Object Name Format](basic-deliver.md#s3-object-name)\.

The following topics describe how to create a Kinesis Data Firehose delivery stream:

**Topics**
+ [Name and source](create-name.md)
+ [Process records](create-transform.md)
+ [Choose destination](create-destination.md)
+ [Configure settings](create-configure.md)