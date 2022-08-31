# Source, Destination, and Name<a name="create-name"></a>

****

1. Sign in to the AWS Management Console and open the Kinesis console at [https://console\.aws\.amazon\.com/kinesis](https://console.aws.amazon.com/kinesis)\.

1. Choose **Data Firehose** in the navigation pane\.

1. Choose **Create delivery stream**\.

1.  Enter values for the following fields:  
****Source****  
   + **Direct PUT:** Choose this option to create a Kinesis Data Firehose delivery stream that producer applications write to directly\. Currently, the following are AWS services and agents and open source services that are integrated with Direct PUT in Kinesis Data Firehose:
     + AWS SDK
     + AWS Lambda
     + AWS Cloud Watch Logs
     + AWS Cloud Watch Events
     + AWS Cloud Metric Streams
     + AWS IOT
     + AWS Eventbridge
     + Amazon SNS
     + AWS WAF web ACL logs
     + Amazon API Gateway \- Access logs
     + Amazon Pinpoint
     + Amazon MSK Broker Logs
     + Amazon Route 53 Resolver query logs
     + AWS Network Firewall Alerts Logs
     + AWS Network Firewall Flow Logs
     + Amazon Elasticache Redis SLOWLOG
     + Kinesis Agent \(linux\)
     + Kinesis Tap \(windows\)
     + Fluentbit
     + Fluentd
     + Apache Nifi
   + **Kinesis stream:** Choose this option to configure a Kinesis Data Firehose delivery stream that uses a Kinesis data stream as a data source\. You can then use Kinesis Data Firehose to read data easily from an existing Kinesis data stream and load it into destinations\. For more information about using Kinesis Data Streams as your data source, see [Writing to Amazon Kinesis Data Firehose Using Kinesis Data Streams](https://docs.aws.amazon.com/firehose/latest/dev/writing-with-kinesis-streams.html)\.  
****Delivery stream destination****  
The destination of your Kinesis Data Firehose delivery stream\. Kinesis Data Firehose can send data records to various destinations, including Amazon Simple Storage Service \(Amazon S3\), Amazon Redshift, Amazon OpenSearch Service, and any HTTP endpoint that is owned by you or any of your third\-party service providers\. The following are the supported destinations:  
   + Amazon OpenSearch Service
   + Amazon Redshift
   + Amazon S3
   + Coralogix
   + Datadog
   + Dynatrace
   + HTTP Endpoint
   + Honeycomb
   + Logic Monitor
   + MongoDB Cloud
   + New Relic
   + Splunk
   + Sumo Logic  
****Delivery stream name****  
The name of your Kinesis Data Firehose delivery stream\.