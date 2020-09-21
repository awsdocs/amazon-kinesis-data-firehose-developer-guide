# Choose destination<a name="create-destination"></a>

This topic describes the **Choose destination** page of the **Create Delivery Stream** wizard in Amazon Kinesis Data Firehose\.

Kinesis Data Firehose can send records to Amazon Simple Storage Service \(Amazon S3\), Amazon Redshift, Amazon Elasticsearch Service \(Amazon ES\), and any HTTP enpoint owned by you or any of your third\-party service providers, including Datadog, New Relic, and Splunk\.

**Topics**
+ [Choose Amazon S3 for Your Destination](#create-destination-s3)
+ [Choose Amazon Redshift for Your Destination](#create-destination-redshift)
+ [Choose Amazon ES for Your Destination](#create-destination-elasticsearch)
+ [Choose HTTP Endpoint for Your Destination](#create-destination-http)
+ [Choose Datadog for Your Destination](#create-destination-datadog)
+ [Choose MongoDB Cloud for Your Destination](#create-destination-mongodb)
+ [Choose New Relic for Your Destination](#create-destination-new-relic)
+ [Choose Splunk for Your Destination](#create-destination-splunk)

## Choose Amazon S3 for Your Destination<a name="create-destination-s3"></a>

This section describes options for using Amazon S3 for your destination\.

**To choose Amazon S3 for your destination**
+ On the **Choose destination** page, enter values for the following fields:  
 **Destination**   
Choose **Amazon S3**\.   
 **S3 bucket**   
Choose an S3 bucket that you own where the streaming data should be delivered\. You can create a new S3 bucket or choose an existing one\.  
 **Prefix**   
\(Optional\) To use the default prefix for Amazon S3 objects, leave this option blank\. Kinesis Data Firehose automatically uses a prefix in the "`YYYY/MM/dd/HH`" UTC time format for delivered Amazon S3 objects\. You can also override this default by specifying a custom prefix\. For more information, see [Amazon S3 Object Name Format](basic-deliver.md#s3-object-name) and [Custom Prefixes for Amazon S3 Objects](s3-prefixes.md)  
 **Error prefix**   
\(Optional\) You can specify a prefix for\. Kinesis Data Firehose to use when delivering data to Amazon S3 in error conditions\. For more information, see [Amazon S3 Object Name Format](basic-deliver.md#s3-object-name) and [Custom Prefixes for Amazon S3 Objects](s3-prefixes.md)

## Choose Amazon Redshift for Your Destination<a name="create-destination-redshift"></a>

This section describes options for using Amazon Redshift for your destination\.

**To choose Amazon Redshift for your destination**
+ On the **Choose destination** page, enter values for the following fields:  
 **Destination**   
Choose **Amazon Redshift**\.   
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
 **Intermediate S3 bucket**   <a name="redshift-s3-bucket"></a>
Kinesis Data Firehose delivers your data to your S3 bucket first and then issues an Amazon Redshift COPY command to load the data into your Amazon Redshift cluster\. Specify an S3 bucket that you own where the streaming data should be delivered\. Create a new S3 bucket, or choose an existing bucket that you own\.  
Kinesis Data Firehose doesn't delete the data from your S3 bucket after loading it to your Amazon Redshift cluster\. You can manage the data in your S3 bucket using a lifecycle configuration\. For more information, see [Object Lifecycle Management](https://docs.aws.amazon.com/AmazonS3/latest/dev/object-lifecycle-mgmt.html) in the *Amazon Simple Storage Service Developer Guide*\.  
 **Prefix**   
\(Optional\) To use the default prefix for Amazon S3 objects, leave this option blank\. Kinesis Data Firehose automatically uses a prefix in "`YYYY/MM/dd/HH`" UTC time format for delivered Amazon S3 objects\. You can add to the start of this prefix\. For more information, see [Amazon S3 Object Name Format](basic-deliver.md#s3-object-name)\.  
 **COPY options**   <a name="redshift-copy-parameters"></a>
Parameters that you can specify in the Amazon Redshift COPY command\. These might be required for your configuration\. For example, "`GZIP`" is required if Amazon S3 data compression is enabled\. "`REGION`" is required if your S3 bucket isn't in the same AWS Region as your Amazon Redshift cluster\. For more information, see [COPY](https://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html) in the *Amazon Redshift Database Developer Guide*\.  
 **COPY command**   <a name="redshift-copy-command"></a>
The Amazon Redshift COPY command\. For more information, see [COPY](https://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html) in the *Amazon Redshift Database Developer Guide*\.  
 **Retry duration**   
Time duration \(0–7200 seconds\) for Kinesis Data Firehose to retry if data COPY to your Amazon Redshift cluster fails\. Kinesis Data Firehose retries every 5 minutes until the retry duration ends\. If you set the retry duration to 0 \(zero\) seconds, Kinesis Data Firehose does not retry upon a COPY command failure\.

## Choose Amazon ES for Your Destination<a name="create-destination-elasticsearch"></a>

This section describes options for using Amazon ES for your destination\.

**To choose Amazon ES for your destination**

1. On the **Choose destination** page, enter values for the following fields:  
** **Destination** **  
Choose **Amazon Elasticsearch Service**\.   
** **Domain** **  
The Amazon ES domain to which your data is delivered\.  
** **Index** **  
The Elasticsearch index name to be used when indexing data to your Amazon ES cluster\.  
** **Index rotation** **  
Choose whether and how often the Elasticsearch index should be rotated\. If index rotation is enabled, Kinesis Data Firehose appends the corresponding timestamp to the specified index name and rotates\. For more information, see [Index Rotation for the Amazon ES Destination](basic-deliver.md#es-index-rotation)\.  
** **Type** **  
The Amazon ES type name to be used when indexing data to your Amazon ES cluster\. For Elasticsearch 6\.x, there can be only one type per index\. If you try to specify a new type for an existing index that already has another type, Kinesis Data Firehose returns an error during runtime\.   
For Elasticsearch 7\.x, leave this field empty\.  
** **Retry duration** **  
Time duration \(0–7200 seconds\) for Kinesis Data Firehose to retry if an index request to your Amazon ES cluster fails\. Kinesis Data Firehose retries every 5 minutes until the retry duration ends\. If you set the retry duration to 0 \(zero\) seconds, Kinesis Data Firehose does not retry upon an index request failure\.  
** **Destination VPC connectivity** **  
If your Amazon ES domain is in a private VPC, use this section to specify that VPC\. Also specify the subnets and subgroups that you want Kinesis Data Firehose to use when it sends data to your Amazon ES domain\. You can use the same security group that the Amazon ES domain uses or different ones\. If you specify different security groups, ensure that they allow outbound HTTPS traffic to the Amazon ES domain's security group\. Also ensure that the Amazon ES domain's security group allows HTTPS traffic from the security groups that you specified when you configured your delivery stream\. If you use the same security group for both your delivery stream and the Amazon ES domain, make sure the security group's inbound rule allows HTTPS traffic\. For more information about security group rules, see [Security group rules](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#SecurityGroupRules) in the Amazon VPC documentation\.  
**Backup mode**  
You can choose to either back up failed records only or all records\. If you choose failed records only, any data that Kinesis Data Firehose can't deliver to your Amazon ES cluster or that your Lambda function can't transform is backed up to the specified S3 bucket\. If you choose all records, Kinesis Data Firehose backs up all incoming source data to your S3 bucket concurrently with data delivery to Amazon ES\. For more information, see [Data Delivery Failure Handling](basic-deliver.md#retry) and [Data Transformation Failure Handling](data-transformation.md#data-transformation-failure-handling)\.  
**Backup S3 bucket**  
An S3 bucket you own that is the target of the backup data\. Create a new S3 bucket, or choose an existing bucket that you own\.  
**Backup S3 bucket prefix**  
\(Optional\) To use the default prefix for Amazon S3 objects, leave this option blank\. Kinesis Data Firehose automatically uses a prefix in "`YYYY/MM/dd/HH`" UTC time format for delivered Amazon S3 objects\. You can add to the start of this prefix\. For more information, see [Amazon S3 Object Name Format](basic-deliver.md#s3-object-name)\.

1. Choose **Next** to go to the [Configure settings](create-configure.md) page\.

## Choose HTTP Endpoint for Your Destination<a name="create-destination-http"></a>

This section describes options for using **HTTP endpoint** for your destination\.

**Important**  
If you choose an HTTP endpoint as your destination, review and follow the instructions in [Appendix \- HTTP Endpoint Delivery Request and Response Specifications](httpdeliveryrequestresponse.md)\.

**To choose **HTTP endpoint** for your destination**
+ On the **Choose destination** page, provide values for the following fields:  
 **Destination**   
Choose **HTTP endpoint**\.  
 **HTTP endpoint name \- optional**   
Specify a user friendly name for the HTTP endpoint\. For example, `My HTTP Endpoint Destination`\.  
 **HTTP endpoint URL**   
Specify the URL for the HTTP endpoint in the following format: `https://xyz.httpendpoint.com`\. The URL must be an HTTPS URL\.  
 **Access key \- optional**   
Contact the endpoint owner to obtain the access key \(if it is required\) to enable data delivery to their endpoint from Kinesis Data Firehose\.  
 **Content encoding**   
Kinesis Data Firehose uses content encoding to compress the body of a request before sending it to the destination\. Choose **GZIP** or **Disabled** to enable/disable content encoding of your request\.   
 **Parameters \- optional**   
Kinesis Data Firehose includes these key\-value pairs in each HTTP call\. These parameters can help you identify and organize your destinations\.  
 **Retry duration**   
Specify how long Kinesis Data Firehose retries sending data to the selected HTTP endpoint\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from the HTTP endpoint\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to the HTTP endpoint \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from the HTTP endpoint\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.  
 **S3 backup mode**   
Choose whether to back up all the events that Kinesis Data Firehose sends to the specified HTTP endpoint or only the ones for which delivery to the HTTP endpoint fails\. If you require high data durability, turn on this backup mode for all events\. Also consider backing up all events initially, until you verify that your data is getting indexed correctly in the specified HTTP endpoint service\.  
 **S3 backup bucket**   
Choose an existing backup bucket or create a new one\.  
 **Backup S3 bucket prefix**   
\(Optional\) To use the default prefix for Amazon S3 objects, leave this option blank\. Kinesis Data Firehose automatically uses a prefix in "`YYYY/MM/dd/HH`" UTC time format for delivered Amazon S3 objects\. You can add to the start of this prefix\. For more information, see [Amazon S3 Object Name Format](basic-deliver.md#s3-object-name)\.

## Choose Datadog for Your Destination<a name="create-destination-datadog"></a>

This section describes options for using **Datadog** for your destination\.

**To choose **Datadog** for your destination**
+ On the **Choose destination** page, provide values for the following fields:  
 **Destination**   
Choose **Third\-party service provider** and then in the drop\-down menu, choose **Datadog**\.  
 **HTTP endpoint URL**   
Choose the HTTP endpoint URL \(**Datadog EU** or **Datadog US**\) from the drop down menu\.  
 **API key**   
Contact Datadog to obtain the API key required to enable data delivery to this endpoint from Kinesis Data Firehose\.  
 **Content encoding**   
Kinesis Data Firehose uses content encoding to compress the body of a request before sending it to the destination\. Choose **GZIP** or **Disabled** to enable/disable content encoding of your request\.   
 **Parameters \- optional**   
Kinesis Data Firehose includes these key\-value pairs in each HTTP call\. These parameters can help you identify and organize your destinations\.   
 **Retry duration**   
Specify how long Kinesis Data Firehose retries sending data to the selected HTTP endpoint\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from the HTTP endpoint\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to the HTTP endpoint \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from the HTTP endpoint\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.  
 **S3 backup mode**   
Choose whether to back up all the events that Kinesis Data Firehose sends to the specified HTTP endpoint or only the ones for which delivery to the HTTP endpoint fails\. If you require high data durability, turn on this backup mode for all events\. Also consider backing up all events initially, until you verify that your data is getting indexed correctly in the specified HTTP endpoint service\.  
 **S3 backup bucket**   
Choose an existing backup bucket or create a new one\.  
 **Backup S3 bucket prefix**   
\(Optional\) To use the default prefix for Amazon S3 objects, leave this option blank\. Kinesis Data Firehose automatically uses a prefix in "`YYYY/MM/dd/HH`" UTC time format for delivered Amazon S3 objects\. You can add to the start of this prefix\. For more information, see [Amazon S3 Object Name Format](basic-deliver.md#s3-object-name)\.

## Choose MongoDB Cloud for Your Destination<a name="create-destination-mongodb"></a>

This section describes options for using **MongoDB Cloud** for your destination\.

**To choose **MongoDB Cloud** for your destination**
+ On the **Choose destination** page, provide values for the following fields:  
**Destination**  
Choose **Third\-party service provider** and then in the drop\-down menu, choose **MongoDB Cloud**\.  
**MongoDB Real webhook URL**  
Specify the URL for the HTTP endpoint in the following format: `https://webhooks.mongodb-realm.com`\. The URL must be an HTTPS URL\.   
**API key**  
Contact MongoDB Cloud to obtain the API key required to enable data delivery to this endpoint from Kinesis Data Firehose\.  
**Content encoding**  
Kinesis Data Firehose uses content encoding to compress the body of a request before sending it to the destination\. Choose **GZIP** or **Disabled** to enable/disable content encoding of your request\.   
**Parameters \- optional**  
Kinesis Data Firehose includes these key\-value pairs in each HTTP call\. These parameters can help you identify and organize your destinations\.   
**Retry duration**  
Specify how long Kinesis Data Firehose retries sending data to the selected third\-party provider\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from the HTTP endpoint\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to the HTTP endpoint \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from the HTTP endpoint\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.  
**S3 backup mode**  
Choose whether to back up all the events that Kinesis Data Firehose sends to the specified HTTP endpoint or only the ones for which delivery to the HTTP endpoint fails\. If you require high data durability, turn on this backup mode for all events\. Also consider backing up all events initially, until you verify that your data is getting indexed correctly in the specified HTTP endpoint service\.  
**S3 backup bucket**  
Choose an existing backup bucket or create a new one\.  
**Backup S3 bucket prefix**  
\(Optional\) To use the default prefix for Amazon S3 objects, leave this option blank\. Kinesis Data Firehose automatically uses a prefix in "`YYYY/MM/dd/HH`" UTC time format for delivered Amazon S3 objects\. You can add to the start of this prefix\. For more information, see [Amazon S3 Object Name Format](basic-deliver.md#s3-object-name)\.

## Choose New Relic for Your Destination<a name="create-destination-new-relic"></a>

This section describes options for using **New Relic** for your destination\.

**To choose **New Relic** for your destination**
+ On the **Choose destination** page, provide values for the following fields:  
 **Destination**   
Choose **Third\-party service provider** and then in the drop\-down menu, choose **New Relic**\.  
 **HTTP endpoint URL**   
Specify the URL for the HTTP endpoint in the following format: `https://xyz.httpendpoint.com`\. The URL must be an HTTPS URL\.   
 **API key**   
Contact New Relic to obtain the API key required to enable data delivery to this endpoint from Kinesis Data Firehose\.  
 **Content encoding**   
Kinesis Data Firehose uses content encoding to compress the body of a request before sending it to the destination\. Choose **GZIP** or **Disabled** to enable/disable content encoding of your request\.   
 **Parameters \- optional**   
Kinesis Data Firehose includes these key\-value pairs in each HTTP call\. These parameters can help you identify and organize your destinations\.   
 **Retry duration**   
Specify how long Kinesis Data Firehose retries sending data to New Relic\.   
After sending data, Kinesis Data Firehose first waits for an acknowledgment from the HTTP endpoint\. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Data Firehose starts the retry duration counter\. It keeps retrying until the retry duration expires\. After that, Kinesis Data Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket\.   
Every time that Kinesis Data Firehose sends data to the HTTP endpoint \(either the initial attempt or a retry\), it restarts the acknowledgement timeout counter and waits for an acknowledgement from the HTTP endpoint\.   
Even if the retry duration expires, Kinesis Data Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached\. If the acknowledgment times out, Kinesis Data Firehose determines whether there's time left in the retry counter\. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired\.  
If you don't want Kinesis Data Firehose to retry sending data, set this value to 0\.  
 **S3 backup mode**   
Choose whether to back up all the events that Kinesis Data Firehose sends to the specified New Relic or only the ones for which delivery to the HTTP endpoint fails\. If you require high data durability, turn on this backup mode for all events\. Also consider backing up all events initially, until you verify that your data is getting indexed correctly in New Relic\.  
 **S3 backup bucket**   
Choose an existing backup bucket or create a new one\.  
 **Backup S3 bucket prefix**   
\(Optional\) To use the default prefix for Amazon S3 objects, leave this option blank\. Kinesis Data Firehose automatically uses a prefix in "`YYYY/MM/dd/HH`" UTC time format for delivered Amazon S3 objects\. You can add to the start of this prefix\. For more information, see [Amazon S3 Object Name Format](basic-deliver.md#s3-object-name)\.

## Choose Splunk for Your Destination<a name="create-destination-splunk"></a>

This section describes options for using Splunk for your destination\.

**To choose Splunk for your destination**
+ On the **Choose destination** page, provide values for the following fields:  
 **Destination**   
Choose **Third\-party service provider** and then choose **Splunk**\.  
 **Splunk cluster endpoint**   
To determine the endpoint, see [Configure Amazon Kinesis Firehose to Send Data to the Splunk Platform](http://docs.splunk.com/Documentation/AddOns/latest/Firehose/ConfigureFirehose) in the Splunk documentation\.  
 **Splunk endpoint type**   
Choose `Raw` in most cases\. Choose `Event` if you preprocessed your data using AWS Lambda to send data to different indexes by event type\. For information about what endpoint to use, see [Configure Amazon Kinesis Firehose to send data to the Splunk platform](http://docs.splunk.com/Documentation/AddOns/released/Firehose/ConfigureFirehose) in the Splunk documentation\.  
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
 **S3 backup mode**   
Choose whether to back up all the events that Kinesis Data Firehose sends to Splunk or only the ones for which delivery to Splunk fails\. If you require high data durability, turn on this backup mode for all events\. Also consider backing up all events initially, until you verify that your data is getting indexed correctly in Splunk\.  
 **S3 backup bucket**   
Choose an existing backup bucket or create a new one\.  
Backup S3 bucket prefix  
\(Optional\) To use the default prefix for Amazon S3 objects, leave this option blank\. Kinesis Data Firehose automatically uses a prefix in "`YYYY/MM/dd/HH`" UTC time format for delivered Amazon S3 objects\. You can add to the start of this prefix\. For more information, see [Amazon S3 Object Name Format](basic-deliver.md#s3-object-name)\.