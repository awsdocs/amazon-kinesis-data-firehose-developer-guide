# Amazon Kinesis Data Firehose Developer Guide

-----
*****Copyright &copy; 2020 Amazon Web Services, Inc. and/or its affiliates. All rights reserved.*****

-----
Amazon's trademarks and trade dress may not be used in 
     connection with any product or service that is not Amazon's, 
     in any manner that is likely to cause confusion among customers, 
     or in any manner that disparages or discredits Amazon. All other 
     trademarks not owned by Amazon are the property of their respective
     owners, who may or may not be affiliated with, connected to, or 
     sponsored by Amazon.

-----
## Contents
+ [What Is Amazon Kinesis Data Firehose?](what-is-this-service.md)
+ [Setting Up for Amazon Kinesis Data Firehose](before-you-begin.md)
+ [Creating an Amazon Kinesis Data Firehose Delivery Stream](basic-create.md)
   + [Name and source](create-name.md)
   + [Process records](create-transform.md)
   + [Choose destination](create-destination.md)
   + [Configure settings](create-configure.md)
+ [Testing Your Delivery Stream Using Sample Data](test-drive-firehose.md)
+ [Sending Data to an Amazon Kinesis Data Firehose Delivery Stream](basic-write.md)
   + [Writing to Kinesis Data Firehose Using Kinesis Data Streams](writing-with-kinesis-streams.md)
   + [Writing to Kinesis Data Firehose Using Kinesis Agent](writing-with-agents.md)
   + [Writing to Kinesis Data Firehose Using the AWS SDK](writing-with-sdk.md)
   + [Writing to Kinesis Data Firehose Using CloudWatch Logs](writing-with-cloudwatch-logs.md)
   + [Writing to Kinesis Data Firehose Using CloudWatch Events](writing-with-cloudwatch-events.md)
   + [Writing to Kinesis Data Firehose Using AWS IoT](writing-with-iot.md)
+ [Security in Amazon Kinesis Data Firehose](security.md)
   + [Data Protection in Amazon Kinesis Data Firehose](encryption.md)
   + [Controlling Access with Amazon Kinesis Data Firehose](controlling-access.md)
   + [Monitoring Amazon Kinesis Data Firehose](security-monitoring.md)
   + [Compliance Validation for Amazon Kinesis Data Firehose](akda-java-compliance.md)
   + [Resilience in Amazon Kinesis Data Firehose](disaster-recovery-resiliency.md)
   + [Infrastructure Security in Kinesis Data Firehose](infrastructure-security.md)
   + [Security Best Practices for Kinesis Data Firehose](security-best-practices.md)
+ [Amazon Kinesis Data Firehose Data Transformation](data-transformation.md)
+ [Converting Your Input Record Format in Kinesis Data Firehose](record-format-conversion.md)
+ [Using Amazon Kinesis Data Analytics](data-analysis.md)
+ [Amazon Kinesis Data Firehose Data Delivery](basic-deliver.md)
+ [Monitoring Amazon Kinesis Data Firehose](monitoring.md)
   + [Monitoring Kinesis Data Firehose Using CloudWatch Metrics](monitoring-with-cloudwatch-metrics.md)
      + [Monitoring Kinesis Data Firehose Using CloudWatch Logs](monitoring-with-cloudwatch-logs.md)
      + [Monitoring Kinesis Agent Health](agent-health.md)
      + [Logging Kinesis Data Firehose API Calls with AWS CloudTrail](monitoring-using-cloudtrail.md)
+ [Custom Prefixes for Amazon S3 Objects](s3-prefixes.md)
+ [Using Amazon Kinesis Data Firehose with AWS PrivateLink](vpc.md)
+ [Tagging Your Delivery Streams in Amazon Kinesis Data Firehose](firehose-tagging.md)
+ [Tutorial: Sending VPC Flow Logs to Splunk Using Amazon Kinesis Data Firehose](vpc-splunk-tutorial.md)
   + [Step 1: Send Log Data from Amazon VPC to Amazon CloudWatch](log-data-from-vpc-to-cw.md)
   + [Step 2: Create a Kinesis Data Firehose Delivery Stream with Splunk as a Destination](creating-the-stream-to-splunk.md)
   + [Step 3: Send the Data from Amazon CloudWatch to Kinesis Data Firehose](cw-to-delivery-stream.md)
   + [Step 4: Check the Results in Splunk and in Kinesis Data Firehose](check-vpc-to-splunk-results.md)
+ [Troubleshooting Amazon Kinesis Data Firehose](troubleshooting.md)
   + [Troubleshooting HTTP Endpoints](http_troubleshooting.md)
+ [Amazon Kinesis Data Firehose Quota](limits.md)
+ [Appendix - HTTP Endpoint Delivery Request and Response Specifications](httpdeliveryrequestresponse.md)
+ [Document History](history.md)
+ [AWS glossary](glossary.md)