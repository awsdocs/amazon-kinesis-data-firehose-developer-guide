# Logging Kinesis Data Firehose API Calls with AWS CloudTrail<a name="monitoring-using-cloudtrail"></a>

Amazon Kinesis Data Firehose is integrated with AWS CloudTrail, a service that provides a record of actions taken by a user, role, or an AWS service in Kinesis Data Firehose\. CloudTrail captures all API calls for Kinesis Data Firehose as events\. The calls captured include calls from the Kinesis Data Firehose console and code calls to the Kinesis Data Firehose API operations\. If you create a trail, you can enable continuous delivery of CloudTrail events to an Amazon S3 bucket, including events for Kinesis Data Firehose\. If you don't configure a trail, you can still view the most recent events in the CloudTrail console in **Event history**\. Using the information collected by CloudTrail, you can determine the request that was made to Kinesis Data Firehose, the IP address from which the request was made, who made the request, when it was made, and additional details\. 

To learn more about CloudTrail, including how to configure and enable it, see the [AWS CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\.

## Kinesis Data Firehose Information in CloudTrail<a name="kinesis-data-firehose-name-info-in-cloudtrail"></a>

CloudTrail is enabled on your AWS account when you create the account\. When supported event activity occurs in Kinesis Data Firehose, that activity is recorded in a CloudTrail event along with other AWS service events in **Event history**\. You can view, search, and download recent events in your AWS account\. For more information, see [Viewing Events with CloudTrail Event History](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events.html)\. 

For an ongoing record of events in your AWS account, including events for Kinesis Data Firehose, create a trail\. A *trail* enables CloudTrail to deliver log files to an Amazon S3 bucket\. By default, when you create a trail in the console, the trail applies to all AWS Regions\. The trail logs events from all Regions in the AWS partition and delivers the log files to the Amazon S3 bucket that you specify\. Additionally, you can configure other AWS services to further analyze and act upon the event data collected in CloudTrail logs\. For more information, see the following: 
+ [Overview for Creating a Trail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail.html)
+ [CloudTrail Supported Services and Integrations](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-aws-service-specific-topics.html#cloudtrail-aws-service-specific-topics-integrations)
+ [Configuring Amazon SNS Notifications for CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/getting_notifications_top_level.html)
+ [Receiving CloudTrail Log Files from Multiple Regions](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/receive-cloudtrail-log-files-from-multiple-regions.html) and [Receiving CloudTrail Log Files from Multiple Accounts](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-receive-logs-from-multiple-accounts.html)

Kinesis Data Firehose supports logging the following actions as events in CloudTrail log files:
+ [CreateDeliveryStream](https://docs.aws.amazon.com/firehose/latest/APIReference/API_CreateDeliveryStream.html)
+ [DeleteDeliveryStream](https://docs.aws.amazon.com/firehose/latest/APIReference/API_DeleteDeliveryStream.html)
+ [DescribeDeliveryStream](https://docs.aws.amazon.com/firehose/latest/APIReference/API_DescribeDeliveryStream.html)
+ [ListDeliveryStreams](https://docs.aws.amazon.com/firehose/latest/APIReference/API_ListDeliveryStreams.html)
+ [ListTagsForDeliveryStream](https://docs.aws.amazon.com/firehose/latest/APIReference/API_ListTagsForDeliveryStream.html)
+ [TagDeliveryStream](https://docs.aws.amazon.com/firehose/latest/APIReference/API_TagDeliveryStream.html)
+ [StartDeliveryStreamEncryption](https://docs.aws.amazon.com/firehose/latest/APIReference/API_StartDeliveryStreamEncryption.html)
+ [StopDeliveryStreamEncryption](https://docs.aws.amazon.com/firehose/latest/APIReference/API_StopDeliveryStreamEncryption.html)
+ [UntagDeliveryStream](https://docs.aws.amazon.com/firehose/latest/APIReference/API_UntagDeliveryStream.html)
+ [UpdateDestination](https://docs.aws.amazon.com/firehose/latest/APIReference/API_UpdateDestination.html)

Every event or log entry contains information about who generated the request\. The identity information helps you determine the following: 
+ Whether the request was made with root or AWS Identity and Access Management \(IAM\) user credentials\.
+ Whether the request was made with temporary security credentials for a role or federated user\.
+ Whether the request was made by another AWS service\.

For more information, see the [CloudTrail userIdentity Element](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-user-identity.html)\.

## Example: Kinesis Data Firehose Log File Entries<a name="understanding-service-name-entries"></a>

 A trail is a configuration that enables delivery of events as log files to an Amazon S3 bucket that you specify\. CloudTrail log files contain one or more log entries\. An event represents a single request from any source and includes information about the requested action, the date and time of the action, request parameters, and so on\. CloudTrail log files aren't an ordered stack trace of the public API calls, so they don't appear in any specific order\.

The following example shows a CloudTrail log entry that demonstrates the `CreateDeliveryStream`, `DescribeDeliveryStream`, `ListDeliveryStreams`, `UpdateDestination`, and `DeleteDeliveryStream` actions\.

```
{
  "Records":[
        {
            "eventVersion":"1.02",
            "userIdentity":{
                "type":"IAMUser",
                "principalId":"AKIAIOSFODNN7EXAMPLE",
                "arn":"arn:aws:iam::111122223333:user/CloudTrail_Test_User",
                "accountId":"111122223333",
                "accessKeyId":"AKIAI44QH8DHBEXAMPLE",
                "userName":"CloudTrail_Test_User"
            },
            "eventTime":"2016-02-24T18:08:22Z",
            "eventSource":"firehose.amazonaws.com",
            "eventName":"CreateDeliveryStream",
            "awsRegion":"us-east-1",
            "sourceIPAddress":"127.0.0.1",
            "userAgent":"aws-internal/3",
            "requestParameters":{
                "deliveryStreamName":"TestRedshiftStream",
                "redshiftDestinationConfiguration":{
                "s3Configuration":{
                    "compressionFormat":"GZIP",
                    "prefix":"prefix",
                    "bucketARN":"arn:aws:s3:::firehose-cloudtrail-test-bucket",
                    "roleARN":"arn:aws:iam::111122223333:role/Firehose",
                    "bufferingHints":{
                        "sizeInMBs":3,
                        "intervalInSeconds":900
                    },
                    "encryptionConfiguration":{
                        "kMSEncryptionConfig":{
                            "aWSKMSKeyARN":"arn:aws:kms:us-east-1:key"
                        }
                    }
                },
                "clusterJDBCURL":"jdbc:redshift://example.abc123.us-west-2.redshift.amazonaws.com:5439/dev",
                "copyCommand":{
                    "copyOptions":"copyOptions",
                    "dataTableName":"dataTable"
                },
                "password":"",
                "username":"",
                "roleARN":"arn:aws:iam::111122223333:role/Firehose"
            }
        },
        "responseElements":{
            "deliveryStreamARN":"arn:aws:firehose:us-east-1:111122223333:deliverystream/TestRedshiftStream"
        },
        "requestID":"958abf6a-db21-11e5-bb88-91ae9617edf5",
        "eventID":"875d2d68-476c-4ad5-bbc6-d02872cfc884",
        "eventType":"AwsApiCall",
        "recipientAccountId":"111122223333"
    },
    {
        "eventVersion":"1.02",
        "userIdentity":{
            "type":"IAMUser",
            "principalId":"AKIAIOSFODNN7EXAMPLE",
            "arn":"arn:aws:iam::111122223333:user/CloudTrail_Test_User",
            "accountId":"111122223333",
            "accessKeyId":"AKIAI44QH8DHBEXAMPLE",
            "userName":"CloudTrail_Test_User"
        },
        "eventTime":"2016-02-24T18:08:54Z",
        "eventSource":"firehose.amazonaws.com",
        "eventName":"DescribeDeliveryStream",
        "awsRegion":"us-east-1",
        "sourceIPAddress":"127.0.0.1",
        "userAgent":"aws-internal/3",
        "requestParameters":{
            "deliveryStreamName":"TestRedshiftStream"
        },
        "responseElements":null,
        "requestID":"aa6ea5ed-db21-11e5-bb88-91ae9617edf5",
        "eventID":"d9b285d8-d690-4d5c-b9fe-d1ad5ab03f14",
        "eventType":"AwsApiCall",
        "recipientAccountId":"111122223333"
    },
    {
        "eventVersion":"1.02",
        "userIdentity":{
            "type":"IAMUser",
            "principalId":"AKIAIOSFODNN7EXAMPLE",
            "arn":"arn:aws:iam::111122223333:user/CloudTrail_Test_User",
            "accountId":"111122223333",
            "accessKeyId":"AKIAI44QH8DHBEXAMPLE",
            "userName":"CloudTrail_Test_User"
        },
        "eventTime":"2016-02-24T18:10:00Z",
        "eventSource":"firehose.amazonaws.com",
        "eventName":"ListDeliveryStreams",
        "awsRegion":"us-east-1",
        "sourceIPAddress":"127.0.0.1",
        "userAgent":"aws-internal/3",
        "requestParameters":{
            "limit":10
        },
        "responseElements":null,
        "requestID":"d1bf7f86-db21-11e5-bb88-91ae9617edf5",
        "eventID":"67f63c74-4335-48c0-9004-4ba35ce00128",
        "eventType":"AwsApiCall",
        "recipientAccountId":"111122223333"
    },
    {
        "eventVersion":"1.02",
        "userIdentity":{
            "type":"IAMUser",
            "principalId":"AKIAIOSFODNN7EXAMPLE",
            "arn":"arn:aws:iam::111122223333:user/CloudTrail_Test_User",
            "accountId":"111122223333",
            "accessKeyId":"AKIAI44QH8DHBEXAMPLE",
            "userName":"CloudTrail_Test_User"
        },
        "eventTime":"2016-02-24T18:10:09Z",
        "eventSource":"firehose.amazonaws.com",
        "eventName":"UpdateDestination",
        "awsRegion":"us-east-1",
        "sourceIPAddress":"127.0.0.1",
        "userAgent":"aws-internal/3",
        "requestParameters":{
            "destinationId":"destinationId-000000000001",
            "deliveryStreamName":"TestRedshiftStream",
            "currentDeliveryStreamVersionId":"1",
            "redshiftDestinationUpdate":{
                "roleARN":"arn:aws:iam::111122223333:role/Firehose",
                "clusterJDBCURL":"jdbc:redshift://example.abc123.us-west-2.redshift.amazonaws.com:5439/dev",
                "password":"",
                "username":"",
                "copyCommand":{
                    "copyOptions":"copyOptions",
                    "dataTableName":"dataTable"
                },
                "s3Update":{
                    "bucketARN":"arn:aws:s3:::firehose-cloudtrail-test-bucket-update",
                    "roleARN":"arn:aws:iam::111122223333:role/Firehose",
                    "compressionFormat":"GZIP",
                    "bufferingHints":{
                        "sizeInMBs":3,
                        "intervalInSeconds":900
                    },
                    "encryptionConfiguration":{
                        "kMSEncryptionConfig":{
                            "aWSKMSKeyARN":"arn:aws:kms:us-east-1:key"
                        }
                    },
                    "prefix":"arn:aws:s3:::firehose-cloudtrail-test-bucket"
                }
            }
        },
        "responseElements":null,
        "requestID":"d549428d-db21-11e5-bb88-91ae9617edf5",
        "eventID":"1cb21e0b-416a-415d-bbf9-769b152a6585",
        "eventType":"AwsApiCall",
        "recipientAccountId":"111122223333"
    },
    {
        "eventVersion":"1.02",
        "userIdentity":{
            "type":"IAMUser",
            "principalId":"AKIAIOSFODNN7EXAMPLE",
            "arn":"arn:aws:iam::111122223333:user/CloudTrail_Test_User",
            "accountId":"111122223333",
            "accessKeyId":"AKIAI44QH8DHBEXAMPLE",
            "userName":"CloudTrail_Test_User"
        },
        "eventTime":"2016-02-24T18:10:12Z",
        "eventSource":"firehose.amazonaws.com",
        "eventName":"DeleteDeliveryStream",
        "awsRegion":"us-east-1",
        "sourceIPAddress":"127.0.0.1",
        "userAgent":"aws-internal/3",
        "requestParameters":{
            "deliveryStreamName":"TestRedshiftStream"
        },
        "responseElements":null,
        "requestID":"d85968c1-db21-11e5-bb88-91ae9617edf5",
        "eventID":"dd46bb98-b4e9-42ff-a6af-32d57e636ad1",
        "eventType":"AwsApiCall",
        "recipientAccountId":"111122223333"
    }
  ]
}
```