# Step 4: Check the Results in Splunk and in Kinesis Data Firehose<a name="check-vpc-to-splunk-results"></a>

You can monitor the flow of data at several points in this example\. In this step of the [Kinesis Data Firehose tutorial](vpc-splunk-tutorial.md), you check the data in Splunk, the final destination, and you also monitor its flow through Kinesis Data Firehose\.

**To check the results in AWS and in Splunk**

1. Open the Kinesis Data Firehose console at [https://console\.aws\.amazon\.com/firehose/](https://console.aws.amazon.com/firehose/)\.

1. In the list of delivery streams, choose **VPCtoSplunkStream**\.

1. Choose the **Monitoring** tab, and view the graphs\. Be sure to adjust the time range and to use the **refresh** button periodically\.

1. If you don't see your data in Splunk, see [Data Not Delivered to Splunk](https://docs.aws.amazon.com/firehose/latest/dev/troubleshooting.html#data-not-delivered-to-splunk)\.

**Important**  
After you verify your results, delete any AWS resources that you don't need to keep, so as not to incur ongoing charges\.