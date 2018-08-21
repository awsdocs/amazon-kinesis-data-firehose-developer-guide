# Monitoring Kinesis Data Firehose API Calls Using AWS CloudTrail<a name="monitoring-with-cloudtrail"></a>

Amazon Kinesis Data Firehose is integrated with AWS CloudTrail, which captures API calls, delivers the log files to an Amazon S3 bucket that you specify, and maintains API call history\. CloudTrail captures API calls made from the Kinesis Data Firehose console or from your code to the Kinesis Data Firehose API\. Use the information collected by CloudTrail to determine the request that was made to Kinesis Data Firehose, the IP address that it came from, who made the request, when it was made, and so on\.

To learn more about CloudTrail, including how to configure and enable it, see the *[AWS CloudTrail User Guide](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/)*\. 

**Topics**
+ [Kinesis Data Firehose and CloudTrail History](#cloudtrail-history)
+ [Kinesis Data Firehose and CloudTrail Logging](#cloudtrail-logging)
+ [CloudTrail Log File Entries for Kinesis Data Firehose](#cloudtrail-log-entries)

## Kinesis Data Firehose and CloudTrail History<a name="cloudtrail-history"></a>

The CloudTrail API activity history feature lets you look up and filter events captured by CloudTrail\. You can look up events related to the creation, modification, or deletion of resources in your AWS account on a per\-region basis\. Events can be looked up by using the CloudTrail console, or programmatically by using the AWS SDKs or AWS CLI\. 

The following actions are supported:
+ [CreateDeliveryStream](http://docs.aws.amazon.com/firehose/latest/APIReference/API_CreateDeliveryStream.html)
+ [DeleteDeliveryStream](http://docs.aws.amazon.com/firehose/latest/APIReference/API_DeleteDeliveryStream.html)
+ [UpdateDestination](http://docs.aws.amazon.com/firehose/latest/APIReference/API_UpdateDestination.html)

## Kinesis Data Firehose and CloudTrail Logging<a name="cloudtrail-logging"></a>

When CloudTrail logging is enabled in your AWS account, API calls made to specific Kinesis Data Firehose actions are tracked in CloudTrail log files\. Kinesis Data Firehose actions are written with other AWS service records\. CloudTrail determines when to create and write to a new file based on the specified time period and file size\.

The following actions are supported:
+ [CreateDeliveryStream](http://docs.aws.amazon.com/firehose/latest/APIReference/API_CreateDeliveryStream.html)
+ [DescribeDeliveryStream](http://docs.aws.amazon.com/firehose/latest/APIReference/API_DescribeDeliveryStream.html)
+ [ListDeliveryStreams](http://docs.aws.amazon.com/firehose/latest/APIReference/API_ListDeliveryStreams.html)
+ [UpdateDestination](http://docs.aws.amazon.com/firehose/latest/APIReference/API_UpdateDestination.html)
+ [DeleteDeliveryStream](http://docs.aws.amazon.com/firehose/latest/APIReference/API_DeleteDeliveryStream.html)

Every log entry contains information about who generated the request\. For example, if a request is made to create a delivery stream \(CreateDeliveryStream\), the identity of the person or service that made the request is logged\. The identity information in the log entry helps you determine the following: 
+ Whether the request was made with root or IAM user credentials
+ Whether the request was made with temporary security credentials for a role or federated user
+ Whether the request was made by another AWS service

 For more information, see the [CloudTrail userIdentity Element](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-user-identity.html)\.

You can store your log files in your bucket for as long as needed but you can also define Amazon S3 lifecycle rules to archive or delete log files automatically\. By default, your log files are encrypted by using Amazon S3 server\-side encryption \(SSE\)\.

You can have CloudTrail publish SNS notifications when new log files are delivered if you want to take quick action upon log file delivery\. For information, see [Configuring Amazon SNS Notifications](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/getting_notifications_top_level.html) in the *AWS CloudTrail User Guide*\.

You can also aggregate Kinesis Data Firehose log files from multiple AWS regions and multiple AWS accounts into a single S3 bucket\. For more information, see [Receiving CloudTrail Log Files from Multiple Regions](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-receive-logs-from-multiple-accounts.html) and [Receiving CloudTrail Log Files from Multiple Accounts](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-receive-logs-from-multiple-accounts.html) in the *AWS CloudTrail User Guide*\.

## CloudTrail Log File Entries for Kinesis Data Firehose<a name="cloudtrail-log-entries"></a>

CloudTrail log files contain one or more log entries\. Each entry lists multiple JSON\-formatted events\. A log entry represents a single request from any source and includes information about the requested action, the date and time of the action, request parameters, and so on\. The log entries are not an ordered stack trace of the public API calls, so they do not appear in any specific order\.

The following is an example CloudTrail log entry\. Note that for security reasons the user name and password for RedshiftDestinationConfiguration will be omitted and returned as an empty string\.

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