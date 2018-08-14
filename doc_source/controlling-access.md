# Controlling Access with Amazon Kinesis Data Firehose<a name="controlling-access"></a>

The following sections cover how to control access to and from your Kinesis Data Firehose resources\. The information they cover includes how to grant your application access so it can send data to your Kinesis Data Firehose delivery stream\. They also describe how you can grant Kinesis Data Firehose access to your Amazon Simple Storage Service \(Amazon S3\) bucket, Amazon Redshift cluster, or Amazon Elasticsearch Service cluster, as well as the access permissions you need if you use Splunk as your destination\. Finally, you'll find in this topic guidance on how to configure Kinesis Data Firehose so it can deliver data to a destination that belongs to a different AWS account\. The technology for managing all these forms of access is AWS Identity and Access Management \(IAM\)\. For more information about IAM, see [What is IAM?](http://docs.aws.amazon.com/IAM/latest/UserGuide/IAM_Introduction.html)\.

**Topics**
+ [Grant Your Application Access to Your Kinesis Data Firehose Resources](#access-to-firehose)
+ [Grant Kinesis Data Firehose Access to an Amazon S3 Destination](#using-iam-s3)
+ [Grant Kinesis Data Firehose Access to an Amazon Redshift Destination](#using-iam-rs)
+ [Grant Kinesis Data Firehose Access to an Amazon ES Destination](#using-iam-es)
+ [Grant Kinesis Data Firehose Access to a Splunk Destination](#using-iam-splunk)
+ [VPC Access to Splunk](#using-iam-splunk-vpc)
+ [Cross\-Account Delivery](#cross-account-delivery)

## Grant Your Application Access to Your Kinesis Data Firehose Resources<a name="access-to-firehose"></a>

To give your application access to your Kinesis Data Firehose delivery stream, use a policy similar to this example\. You can adjust the individual API operations to which you grant access by modifying the `Action` section, or grant access to all operations with `"firehose:*"`\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "firehose:DeleteDeliveryStream",
                "firehose:PutRecord",
                "firehose:PutRecordBatch",
                "firehose:UpdateDestination"
            ],
            "Resource": [
                "arn:aws:firehose:region:account-id:deliverystream/delivery-stream-name"
            ]
        }
    ]
}
```

## Grant Kinesis Data Firehose Access to an Amazon S3 Destination<a name="using-iam-s3"></a>

When you're using an Amazon S3 destination, Kinesis Data Firehose delivers data to your S3 bucket and can optionally use an AWS KMS key that you own for data encryption\. If error logging is enabled, Kinesis Data Firehose also sends data delivery errors to your CloudWatch log group and streams\. You are required to have an IAM role when creating a delivery stream\. Kinesis Data Firehose assumes that IAM role and gains access to the specified bucket, key, and CloudWatch log group and streams\.

Use the following trust policy to enable Kinesis Data Firehose to assume the role\. Edit the policy to replace *account\-id* with your AWS account ID\. This ensures that only you can request Kinesis Data Firehose to assume the IAM role\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "firehose.amazonaws.com"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId":"account-id"
        }
      }
    }
  ]
}
```

Use the following access policy to enable Kinesis Data Firehose to access your S3 bucket and AWS KMS key\. If you don't own the S3 bucket, add `s3:PutObjectAcl` to the list of Amazon S3 actions\. This grants the bucket owner full access to the objects delivered by Kinesis Data Firehose\. This policy also has a statement that allows access to Amazon Kinesis Data Streams\. If you don't use Kinesis Data Streams as your data source, you can remove that statement\.

```
{
    "Version": "2012-10-17",  
    "Statement":
    [    
        {      
            "Effect": "Allow",      
            "Action": [        
                "s3:AbortMultipartUpload",        
                "s3:GetBucketLocation",        
                "s3:GetObject",        
                "s3:ListBucket",        
                "s3:ListBucketMultipartUploads",        
                "s3:PutObject"
            ],      
            "Resource": [        
                "arn:aws:s3:::bucket-name",
                "arn:aws:s3:::bucket-name/*"		    
            ]    
        },        
        {
            "Effect": "Allow",
            "Action": [
                "kinesis:DescribeStream",
                "kinesis:GetShardIterator",
                "kinesis:GetRecords"
            ],
            "Resource": "arn:aws:kinesis:region:account-id:stream/stream-name"
        },
        {
           "Effect": "Allow",
           "Action": [
               "kms:Decrypt",
               "kms:GenerateDataKey"
           ],
           "Resource": [
               "arn:aws:kms:region:account-id:key/key-id"           
           ],
           "Condition": {
               "StringEquals": {
                   "kms:ViaService": "s3.region.amazonaws.com"
               },
               "StringLike": {
                   "kms:EncryptionContext:aws:s3:arn": "arn:aws:s3:::bucket-name/prefix*"
               }
           }
        },
        {
           "Effect": "Allow",
           "Action": [
               "logs:PutLogEvents"
           ],
           "Resource": [
               "arn:aws:logs:region:account-id:log-group:log-group-name:log-stream:log-stream-name"
           ]
        },
        {
           "Effect": "Allow", 
           "Action": [
               "lambda:InvokeFunction", 
               "lambda:GetFunctionConfiguration" 
           ],
           "Resource": [
               "arn:aws:lambda:region:account-id:function:function-name:function-version"
           ]
        }
    ]
}
```

For more information about allowing other AWS services to access your AWS resources, see [Creating a Role to Delegate Permissions to an AWS Service](http://docs.aws.amazon.com/IAM/latest/UserGuide//id_roles_create_for-service.html) in the *IAM User Guide*\.

## Grant Kinesis Data Firehose Access to an Amazon Redshift Destination<a name="using-iam-rs"></a>

Refer to the following when you are granting access to Kinesis Data Firehose when using an Amazon Redshift destination\.

**Topics**
+ [IAM Role and Access Policy](#using-iam-rs-policy)
+ [VPC Access to an Amazon Redshift Cluster](#using-iam-rs-vpc)

### IAM Role and Access Policy<a name="using-iam-rs-policy"></a>

When you're using an Amazon Redshift destination, Kinesis Data Firehose delivers data to your S3 bucket as an intermediate location\. It can optionally use an AWS KMS key you own for data encryption\. Kinesis Data Firehose then loads the data from the S3 bucket to your Amazon Redshift cluster\. If error logging is enabled, Kinesis Data Firehose also sends data delivery errors to your CloudWatch log group and streams\. Kinesis Data Firehose uses the specified Amazon Redshift user name and password to access your cluster, and uses an IAM role to access the specified bucket, key, CloudWatch log group, and streams\. You are required to have an IAM role when creating a delivery stream\.

Use the following trust policy to enable Kinesis Data Firehose to assume the role\. Edit the policy to replace *account\-id* with your AWS account ID\. This is so that only you can request Kinesis Data Firehose to assume the IAM role\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "firehose.amazonaws.com"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId":"account-id"
        }
      }
    }
  ]
}
```

Use the following access policy to enable Kinesis Data Firehose to access your S3 bucket and AWS KMS key\. If you don't own the S3 bucket, add `s3:PutObjectAcl` to the list of Amazon S3 actions, which grants the bucket owner full access to the objects delivered by Kinesis Data Firehose\. This policy also has a statement that allows access to Amazon Kinesis Data Streams\. If you don't use Kinesis Data Streams as your data source, you can remove that statement\.

```
{
"Version": "2012-10-17",  
    "Statement":
    [    
        {      
            "Effect": "Allow",
            "Action": [        
                "s3:AbortMultipartUpload",        
                "s3:GetBucketLocation",        
                "s3:GetObject",        
                "s3:ListBucket",        
                "s3:ListBucketMultipartUploads",        
                "s3:PutObject"
            ],      
            "Resource": [        
                "arn:aws:s3:::bucket-name",
                "arn:aws:s3:::bucket-name/*"		    
            ]    
        },
        {
           "Effect": "Allow",
           "Action": [
               "kms:Decrypt",
               "kms:GenerateDataKey"
           ],
           "Resource": [
               "arn:aws:kms:region:account-id:key/key-id"           
           ],
           "Condition": {
               "StringEquals": {
                   "kms:ViaService": "s3.region.amazonaws.com"
               },
               "StringLike": {
                   "kms:EncryptionContext:aws:s3:arn": "arn:aws:s3:::bucket-name/prefix*"
               }
           }
        },        
        {
           "Effect": "Allow",
           "Action": [
               "kinesis:DescribeStream",
               "kinesis:GetShardIterator",
               "kinesis:GetRecords"
           ],
           "Resource": "arn:aws:kinesis:region:account-id:stream/stream-name"
        },
        {
           "Effect": "Allow",
           "Action": [
               "logs:PutLogEvents"
           ],
           "Resource": [
               "arn:aws:logs:region:account-id:log-group:log-group-name:log-stream:log-stream-name"
           ]
        },
        {
           "Effect": "Allow", 
           "Action": [
               "lambda:InvokeFunction", 
               "lambda:GetFunctionConfiguration" 
           ],
           "Resource": [
               "arn:aws:lambda:region:account-id:function:function-name:function-version"
           ]
        }
    ]
}
```

For more information about allowing other AWS services to access your AWS resources, see [Creating a Role to Delegate Permissions to an AWS Service](http://docs.aws.amazon.com/IAM/latest/UserGuide//id_roles_create_for-service.html) in the *IAM User Guide*\.

### VPC Access to an Amazon Redshift Cluster<a name="using-iam-rs-vpc"></a>

If your Amazon Redshift cluster is in a virtual private cloud \(VPC\), it must be publicly accessible with a public IP address\. Also, grant Kinesis Data Firehose access to your Amazon Redshift cluster by unblocking the Kinesis Data Firehose IP addresses\. Kinesis Data Firehose currently uses one CIDR block for each available Region: 
+ `13.58.135.96/27` for US East \(Ohio\)
+ `52.70.63.192/27` for US East \(N\. Virginia\)
+ `13.57.135.192/27` for US West \(N\. California\)
+ `52.89.255.224/27` for US West \(Oregon\)
+ `13.228.64.192/27` for Asia Pacific \(Singapore\)
+ `13.210.67.224/27` for Asia Pacific \(Sydney\)
+ `13.113.196.224/27` for Asia Pacific \(Tokyo\)
+ `35.158.127.160/27` for EU \(Frankfurt\)
+ `52.19.239.192/27` for EU \(Ireland\)

For more information about how to unblock IP addresses, see the step [Authorize Access to the Cluster](http://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-authorize-cluster-access.html) in the *Amazon Redshift Getting Started* guide\. 

## Grant Kinesis Data Firehose Access to an Amazon ES Destination<a name="using-iam-es"></a>

When you're using an Amazon ES destination, Kinesis Data Firehose delivers data to your Amazon ES cluster, and concurrently backs up failed or all documents to your S3 bucket\. If error logging is enabled, Kinesis Data Firehose also sends data delivery errors to your CloudWatch log group and streams\. Kinesis Data Firehose uses an IAM role to access the specified Elasticsearch domain, S3 bucket, AWS KMS key, and CloudWatch log group and streams\. You are required to have an IAM role when creating a delivery stream\.

Use the following trust policy to enable Kinesis Data Firehose to assume the role\. Edit the following policy to replace *account\-id* with your AWS account ID\. This is so that only you can request Kinesis Data Firehose to assume the IAM role\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "firehose.amazonaws.com"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId":"account-id"
        }
      }
    }
  ]
}
```

Use the following access policy to enable Kinesis Data Firehose to access your S3 bucket, Amazon ES domain, and AWS KMS key\. If you do not own the S3 bucket, add `s3:PutObjectAcl` to the list of Amazon S3 actions, which grants the bucket owner full access to the objects delivered by Kinesis Data Firehose\. This policy also has a statement that allows access to Amazon Kinesis Data Streams\. If you don't use Kinesis Data Streams as your data source, you can remove that statement\.

```
{
    "Version": "2012-10-17",  
    "Statement": [    
        {      
            "Effect": "Allow",      
            "Action": [        
                "s3:AbortMultipartUpload",        
                "s3:GetBucketLocation",        
                "s3:GetObject",        
                "s3:ListBucket",        
                "s3:ListBucketMultipartUploads",        
                "s3:PutObject"
            ],      
            "Resource": [        
                "arn:aws:s3:::bucket-name",
                "arn:aws:s3:::bucket-name/*"		    
            ]    
        },
        {
           "Effect": "Allow",
           "Action": [
               "kms:Decrypt",
               "kms:GenerateDataKey"
           ],
           "Resource": [
               "arn:aws:kms:region:account-id:key/key-id"           
           ],
           "Condition": {
               "StringEquals": {
                   "kms:ViaService": "s3.region.amazonaws.com"
               },
               "StringLike": {
                   "kms:EncryptionContext:aws:s3:arn": "arn:aws:s3:::bucket-name/prefix*"
               }
           }
        },
        {
           "Effect": "Allow",
           "Action": [
               "es:DescribeElasticsearchDomain",
               "es:DescribeElasticsearchDomains",
               "es:DescribeElasticsearchDomainConfig",
               "es:ESHttpPost",
               "es:ESHttpPut"
           ],
          "Resource": [
              "arn:aws:es:region:account-id:domain/domain-name",
              "arn:aws:es:region:account-id:domain/domain-name/*"
          ]
       },
       {
          "Effect": "Allow",
          "Action": [
              "es:ESHttpGet"
          ],
          "Resource": [
              "arn:aws:es:region:account-id:domain/domain-name/_all/_settings",
              "arn:aws:es:region:account-id:domain/domain-name/_cluster/stats",
              "arn:aws:es:region:account-id:domain/domain-name/index-name*/_mapping/type-name",
              "arn:aws:es:region:account-id:domain/domain-name/_nodes",
              "arn:aws:es:region:account-id:domain/domain-name/_nodes/stats",
              "arn:aws:es:region:account-id:domain/domain-name/_nodes/*/stats",
              "arn:aws:es:region:account-id:domain/domain-name/_stats",
              "arn:aws:es:region:account-id:domain/domain-name/index-name*/_stats"
          ]
       },        
       {
          "Effect": "Allow",
          "Action": [
              "kinesis:DescribeStream",
              "kinesis:GetShardIterator",
              "kinesis:GetRecords"
          ],
          "Resource": "arn:aws:kinesis:region:account-id:stream/stream-name"
       },
       {
          "Effect": "Allow",
          "Action": [
              "logs:PutLogEvents"
          ],
          "Resource": [
              "arn:aws:logs:region:account-id:log-group:log-group-name:log-stream:log-stream-name"
          ]
       },
       {
          "Effect": "Allow", 
          "Action": [
              "lambda:InvokeFunction", 
              "lambda:GetFunctionConfiguration" 
          ],
          "Resource": [
              "arn:aws:lambda:region:account-id:function:function-name:function-version"
          ]
       }
    ]
}
```

For more information about allowing other AWS services to access your AWS resources, see [Creating a Role to Delegate Permissions to an AWS Service](http://docs.aws.amazon.com/IAM/latest/UserGuide//id_roles_create_for-service.html) in the *IAM User Guide*\.

## Grant Kinesis Data Firehose Access to a Splunk Destination<a name="using-iam-splunk"></a>

When you're using a Splunk destination, Kinesis Data Firehose delivers data to your Splunk HTTP Event Collector \(HEC\) endpoint\. It also backs up that data to the Amazon S3 bucket that you specify, and you can optionally use an AWS KMS key that you own for Amazon S3 server\-side encryption\. If error logging is enabled, Kinesis Data Firehose sends data delivery errors to your CloudWatch log streams\. You can also use AWS Lambda for data transformation\.

You are required to have an IAM role when creating a delivery stream\. Kinesis Data Firehose assumes that IAM role and gains access to the specified bucket, key, and CloudWatch log group and streams\.

Use the following trust policy to enable Kinesis Data Firehose to assume the role\. Edit the policy to replace *account\-id* with your AWS account ID\. This ensures that only you can request Kinesis Data Firehose to assume the IAM role\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "firehose.amazonaws.com"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId":"account-id"
        }
      }
    }
  ]
}
```

Use the following access policy to enable Kinesis Data Firehose to access your S3 bucket\. If you don't own the S3 bucket, add `s3:PutObjectAcl` to the list of Amazon S3 actions, which grants the bucket owner full access to the objects delivered by Kinesis Data Firehose\. This policy also grants Kinesis Data Firehose access to CloudWatch for error logging and to AWS Lambda for data transformation\. The policy also has a statement that allows access to Amazon Kinesis Data Streams\. If you don't use Kinesis Data Streams as your data source, you can remove that statement\. Kinesis Firehose doesn't use IAM to access Splunk\. For accessing Splunk, it uses your HEC token\.

```
{
    "Version": "2012-10-17",  
    "Statement":
    [    
        {      
            "Effect": "Allow",      
            "Action": [        
                "s3:AbortMultipartUpload",        
                "s3:GetBucketLocation",        
                "s3:GetObject",        
                "s3:ListBucket",        
                "s3:ListBucketMultipartUploads",        
                "s3:PutObject"
            ],      
            "Resource": [        
                "arn:aws:s3:::bucket-name",
                "arn:aws:s3:::bucket-name/*"		    
            ]     
        },
        {
           "Effect": "Allow",
           "Action": [
               "kms:Decrypt",
               "kms:GenerateDataKey"
           ],
           "Resource": [
               "arn:aws:kms:region:account-id:key/key-id"           
           ],
           "Condition": {
               "StringEquals": {
                   "kms:ViaService": "s3.region.amazonaws.com"
               },
               "StringLike": {
                   "kms:EncryptionContext:aws:s3:arn": "arn:aws:s3:::bucket-name/prefix*"
               }
           }
        },     
        {
           "Effect": "Allow",
           "Action": [
               "kinesis:DescribeStream",
               "kinesis:GetShardIterator",
               "kinesis:GetRecords"
           ],
           "Resource": "arn:aws:kinesis:region:account-id:stream/stream-name"
        },
        {
           "Effect": "Allow",
           "Action": [
               "logs:PutLogEvents"
           ],
           "Resource": [
               "arn:aws:logs:region:account-id:log-group:log-group-name:log-stream:*"
           ]
        },
        {
           "Effect": "Allow", 
           "Action": [
               "lambda:InvokeFunction", 
               "lambda:GetFunctionConfiguration" 
           ],
           "Resource": [
               "arn:aws:lambda:region:account-id:function:function-name:function-version"
           ]
        }
    ]
}
```

For more information about allowing other AWS services to access your AWS resources, see [Creating a Role to Delegate Permissions to an AWS Service](http://docs.aws.amazon.com/IAM/latest/UserGuide//id_roles_create_for-service.html) in the *IAM User Guide*\.

## VPC Access to Splunk<a name="using-iam-splunk-vpc"></a>

If your Splunk platform is in a VPC, it must be publicly accessible with a public IP address\. Also, grant Kinesis Data Firehose access to your Splunk platform by unblocking the Kinesis Data Firehose IP addresses\. Kinesis Data Firehose currently uses the following CIDR blocks\.
+ `18.216.68.160/27`, `18.216.170.64/27`, `18.216.170.96/27` for US East \(Ohio\)
+ `34.238.188.128/26`, `34.238.188.192/26`, `34.238.195.0/26` for US East \(N\. Virginia\)
+ `13.57.180.0/26` for US West \(N\. California\)
+ `34.216.24.32/27`, `34.216.24.192/27`, `34.216.24.224/27` for US West \(Oregon\)
+ `13.229.187.128/26` for Asia Pacific \(Singapore\)
+ `13.211.12.0/26` for Asia Pacific \(Sydney\)
+ `13.230.21.0/27`, `13.230.21.32/27` for Asia Pacific \(Tokyo\)
+ `18.194.95.192/27`, `18.194.95.224/27`, `18.195.48.0/27` for EU \(Frankfurt\)
+ `34.241.197.32/27`, `34.241.197.64/27`, `34.241.197.96/27` for EU \(Ireland\)

## Cross\-Account Delivery<a name="cross-account-delivery"></a>

You can configure your Kinesis Data Firehose delivery stream to deliver data to a destination that belongs to a different AWS account, as long as the destination supports resource\-based permissions\. For more information about resource\-based permissions, see [Identity\-Based \(IAM\) Permissions and Resource\-Based Permissions](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_permissions.html#TypesPermissions)\. For a list of AWS services that support resource\-based permissions, see [AWS Services That Work with IAM](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html)\.

The following procedure shows an example of configuring a Kinesis Data Firehose delivery stream owned by account A to deliver data to an Amazon S3 bucket owned by account B\.

1. Create an IAM role under account A using steps described in [Grant Kinesis Firehose Access to an Amazon S3 Destination](http://docs.aws.amazon.com/firehose/latest/dev/controlling-access.html#using-iam-s3)\. 
**Note**  
The Amazon S3 bucket specified in the access policy is owned by account B in this case\. Make sure you add `s3:PutObjectAcl` to the list of Amazon S3 actions in the access policy, which grants account B full access to the objects delivered by Amazon Kinesis Data Firehose\.

1. To allow access from the IAM role previously created, create an S3 bucket policy under account B\. The following code is an example of the bucket policy\. For more information, see [Using Bucket Policies and User Policies](http://docs.aws.amazon.com/AmazonS3/latest/dev/using-iam-policies.html)\. 

   ```
   {
   
       "Version": "2012-10-17",
       "Id": "PolicyID",
       "Statement": [
           {
               "Sid": "StmtID",
               "Effect": "Allow",
               "Principal": {
                   "AWS": "arn:aws:iam::accountA-id:role/iam-role-name"
               },
               "Action": [
                   "s3:AbortMultipartUpload",
                   "s3:GetBucketLocation",
                   "s3:GetObject",
                   "s3:ListBucket",
                   "s3:ListBucketMultipartUploads",
                   "s3:PutObject",
                   "s3:PutObjectAcl"
               ],
               "Resource": [
                   "arn:aws:s3:::bucket-name",
                   "arn:aws:s3:::bucket-name/*"
               ]
           }
       ]
   }
   ```

1. Create a Kinesis Data Firehose delivery stream under account A using the IAM role that you created in step 1\.