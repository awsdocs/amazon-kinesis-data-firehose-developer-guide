# Resilience in Amazon Kinesis Data Firehose<a name="disaster-recovery-resiliency"></a>

The AWS global infrastructure is built around AWS Regions and Availability Zones\. AWS Regions provide multiple physically separated and isolated Availability Zones, which are connected with low\-latency, high\-throughput, and highly redundant networking\. With Availability Zones, you can design and operate applications and databases that automatically fail over between Availability Zones without interruption\. Availability Zones are more highly available, fault tolerant, and scalable than traditional single or multiple data center infrastructures\. 

For more information about AWS Regions and Availability Zones, see [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/)\.

In addition to the AWS global infrastructure, Kinesis Data Firehose offers several features to help support your data resiliency and backup needs\.

## Disaster Recovery<a name="disaster-recovery"></a>

Kinesis Data Firehose runs in a serverless mode, and takes care of host degradations, Availability Zone availability, and other infrastructure related issues by performing automatic migration\. When this happens, Kinesis Data Firehose ensures that the delivery stream is migrated without any loss of data\. 