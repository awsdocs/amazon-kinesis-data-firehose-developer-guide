# Controlling Access with Amazon Kinesis Data Firehose<a name="controlling-access"></a>

The following sections cover how to control access to and from your Kinesis Data Firehose resources\. The information they cover includes how to grant your application access so it can send data to your Kinesis Data Firehose delivery stream\. They also describe how you can grant Kinesis Data Firehose access to your Amazon Simple Storage Service \(Amazon S3\) bucket, Amazon Redshift cluster, or Amazon Elasticsearch Service cluster, as well as the access permissions you need if you use Datadog, New Relic, or Splunk as your destination\. Finally, you'll find in this topic guidance on how to configure Kinesis Data Firehose so it can deliver data to a destination that belongs to a different AWS account\. The technology for managing all these forms of access is AWS Identity and Access Management \(IAM\)\. For more information about IAM, see [What is IAM?](https://docs.aws.amazon.com/IAM/latest/UserGuide/IAM_Introduction.html)\.

**Topics**
+ [Grant Your Application Access to Your Kinesis Data Firehose Resources](#access-to-firehose)
+ [Allow Kinesis Data Firehose to Assume an IAM Role](#firehose-assume-role)
+ [Grant Kinesis Data Firehose Access to AWS Glue for Data Format Conversion](#using-iam-glue)
+ [Grant Kinesis Data Firehose Access to an Amazon S3 Destination](#using-iam-s3)
+ [Grant Kinesis Data Firehose Access to an Amazon Redshift Destination](#using-iam-rs)
+ [Grant Kinesis Data Firehose Access to a Public Amazon ES Destination](#using-iam-es)
+ [Grant Kinesis Data Firehose Access to an Amazon ES Destination in a VPC](#using-iam-es-vpc)
+ [Grant Kinesis Data Firehose Access to a Splunk Destination](#using-iam-splunk)
+ [Access to Splunk in VPC](#using-iam-splunk-vpc)
+ [Grant Kinesis Data Firehose Access to an HTTP Endpoint Destination](#using-iam-http)
+ [Cross\-Account Delivery to an Amazon S3 Destination](#cross-account-delivery-s3)
+ [Cross\-Account Delivery to an Amazon ES Destination](#cross-account-delivery-es)
+ [Using Tags to Control Access](#tag-based-access-control)

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

## Allow Kinesis Data Firehose to Assume an IAM Role<a name="firehose-assume-role"></a>

If you use the console to create a delivery stream, and choose the option to create a new role, AWS attaches the required trust policy to the role\. Or if you want Kinesis Data Firehose to use an existing IAM role or if you create a role on your own, attach the following trust policy to that role so that Kinesis Data Firehose can assume it\. 

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "firehose.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

For information about how to modify the trust relationship of a role, see [Modifying a Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_manage_modify.html)\.

## Grant Kinesis Data Firehose Access to AWS Glue for Data Format Conversion<a name="using-iam-glue"></a>

If your delivery stream performs data\-format conversion, Kinesis Data Firehose references table definitions stored in AWS Glue\. To give Kinesis Data Firehose the necessary access to AWS Glue, add the following statement to your policy\. For information on how to find the ARN of the table, see [Specifying AWS Glue Resource ARNs](https://docs.aws.amazon.com/glue/latest/dg/glue-specifying-resource-arns.html)\.

```
{
  "Effect": "Allow",
  "Action": [
    "glue:GetTable",
    "glue:GetTableVersion",
    "glue:GetTableVersions"
  ],
  "Resource": "table-arn"
}
```

## Grant Kinesis Data Firehose Access to an Amazon S3 Destination<a name="using-iam-s3"></a>

When you're using an Amazon S3 destination, Kinesis Data Firehose delivers data to your S3 bucket and can optionally use an AWS KMS key that you own for data encryption\. If error logging is enabled, Kinesis Data Firehose also sends data delivery errors to your CloudWatch log group and streams\. You are required to have an IAM role when creating a delivery stream\. Kinesis Data Firehose assumes that IAM role and gains access to the specified bucket, key, and CloudWatch log group and streams\.

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
                "kinesis:GetRecords",
                "kinesis:ListShards"
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

For more information about allowing other AWS services to access your AWS resources, see [Creating a Role to Delegate Permissions to an AWS Service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) in the *IAM User Guide*\.

To learn how to grant Kinesis Data Firehose access to an Amazon S3 destination in another account, see [Cross\-Account Delivery to an Amazon S3 Destination](#cross-account-delivery-s3)\.

## Grant Kinesis Data Firehose Access to an Amazon Redshift Destination<a name="using-iam-rs"></a>

Refer to the following when you are granting access to Kinesis Data Firehose when using an Amazon Redshift destination\.

**Topics**
+ [IAM Role and Access Policy](#using-iam-rs-policy)
+ [VPC Access to an Amazon Redshift Cluster](#using-iam-rs-vpc)

### IAM Role and Access Policy<a name="using-iam-rs-policy"></a>

When you're using an Amazon Redshift destination, Kinesis Data Firehose delivers data to your S3 bucket as an intermediate location\. It can optionally use an AWS KMS key you own for data encryption\. Kinesis Data Firehose then loads the data from the S3 bucket to your Amazon Redshift cluster\. If error logging is enabled, Kinesis Data Firehose also sends data delivery errors to your CloudWatch log group and streams\. Kinesis Data Firehose uses the specified Amazon Redshift user name and password to access your cluster, and uses an IAM role to access the specified bucket, key, CloudWatch log group, and streams\. You are required to have an IAM role when creating a delivery stream\.

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
               "kinesis:GetRecords",
               "kinesis:ListShards"
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

For more information about allowing other AWS services to access your AWS resources, see [Creating a Role to Delegate Permissions to an AWS Service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) in the *IAM User Guide*\.

### VPC Access to an Amazon Redshift Cluster<a name="using-iam-rs-vpc"></a>

If your Amazon Redshift cluster is in a virtual private cloud \(VPC\), it must be publicly accessible with a public IP address\. Also, grant Kinesis Data Firehose access to your Amazon Redshift cluster by unblocking the Kinesis Data Firehose IP addresses\. Kinesis Data Firehose currently uses one CIDR block for each available Region: 
+ `13.58.135.96/27` for US East \(Ohio\)
+ `52.70.63.192/27` for US East \(N\. Virginia\)
+ `13.57.135.192/27` for US West \(N\. California\)
+ `52.89.255.224/27` for US West \(Oregon\)
+ `18.253.138.96/27` for AWS GovCloud \(US\-East\)
+ `52.61.204.160/27` for AWS GovCloud \(US\-West\)
+ `35.183.92.128/27` for Canada \(Central\)
+ `18.162.221.32/27` for Asia Pacific \(Hong Kong\)
+ `13.232.67.32/27` for Asia Pacific \(Mumbai\)
+ `13.209.1.64/27` for Asia Pacific \(Seoul\)
+ `13.228.64.192/27` for Asia Pacific \(Singapore\)
+ `13.210.67.224/27` for Asia Pacific \(Sydney\)
+ `13.113.196.224/27` for Asia Pacific \(Tokyo\)
+ `52.81.151.32/27` for China \(Beijing\)
+ `161.189.23.64/27` for China \(Ningxia\)
+ `35.158.127.160/27` for Europe \(Frankfurt\)
+ `52.19.239.192/27` for Europe \(Ireland\)
+ `18.130.1.96/27` for Europe \(London\)
+ `35.180.1.96/27` for Europe \(Paris\)
+ `13.53.63.224/27` for Europe \(Stockholm\)
+ `15.185.91.0/27` for Middle East \(Bahrain\)
+ `18.228.1.128/27` for South America \(São Paulo\)
+ `15.161.135.128/27` for Europe \(Milan\)
+ `13.244.121.224/277` for Africa \(Cape Town\)

For more information about how to unblock IP addresses, see the step [Authorize Access to the Cluster](https://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-authorize-cluster-access.html) in the *Amazon Redshift Getting Started* guide\. 

## Grant Kinesis Data Firehose Access to a Public Amazon ES Destination<a name="using-iam-es"></a>

When you're using an Amazon ES destination, Kinesis Data Firehose delivers data to your Amazon ES cluster, and concurrently backs up failed or all documents to your S3 bucket\. If error logging is enabled, Kinesis Data Firehose also sends data delivery errors to your CloudWatch log group and streams\. Kinesis Data Firehose uses an IAM role to access the specified Elasticsearch domain, S3 bucket, AWS KMS key, and CloudWatch log group and streams\. You are required to have an IAM role when creating a delivery stream\.

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
              "kinesis:GetRecords",
              "kinesis:ListShards"
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

For more information about allowing other AWS services to access your AWS resources, see [Creating a Role to Delegate Permissions to an AWS Service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) in the *IAM User Guide*\.

To learn how to grant Kinesis Data Firehose access to an Amazon ES cluster in another account, see [Cross\-Account Delivery to an Amazon ES Destination](#cross-account-delivery-es)\.

## Grant Kinesis Data Firehose Access to an Amazon ES Destination in a VPC<a name="using-iam-es-vpc"></a>

If your Amazon ES domain is in a VPC, make sure you give Kinesis Data Firehose the permissions that are described in the previous section\. In addition, you need to give Kinesis Data Firehose the following permissions to enable it to access your Amazon ES domain's VPC\.
+ `ec2:DescribeVpcs`
+ `ec2:DescribeVpcAttribute`
+ `ec2:DescribeSubnets`
+ `ec2:DescribeSecurityGroups`
+ `ec2:DescribeNetworkInterfaces`
+ `ec2:CreateNetworkInterface`
+ `ec2:CreateNetworkInterfacePermission`
+ `ec2:DeleteNetworkInterface`

If you revoke these permissions after you create the delivery stream, Kinesis Data Firehose can't scale out by creating more ENIs when necessary\. You might therefore see a degradation in performance\.

When you create or update your delivery stream, you specify a security group for Kinesis Data Firehose to use when it sends data to your Amazon ES domain\. You can use the same security group that the Amazon ES domain uses or a different one\. If you specify a different security group, ensure that it allows outbound HTTPS traffic to the Amazon ES domain's security group\. Also ensure that the Amazon ES domain's security group allows HTTPS traffic from the security group you specified when you configured your delivery stream\. If you use the same security group for both your delivery stream and the Amazon ES domain, make sure the security group inbound rule allows HTTPS traffic\. For more information about security group rules, see [Security group rules](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#SecurityGroupRules) in the Amazon VPC documentation\.

## Grant Kinesis Data Firehose Access to a Splunk Destination<a name="using-iam-splunk"></a>

When you're using a Splunk destination, Kinesis Data Firehose delivers data to your Splunk HTTP Event Collector \(HEC\) endpoint\. It also backs up that data to the Amazon S3 bucket that you specify, and you can optionally use an AWS KMS key that you own for Amazon S3 server\-side encryption\. If error logging is enabled, Kinesis Data Firehose sends data delivery errors to your CloudWatch log streams\. You can also use AWS Lambda for data transformation\. If you use an AWS load balancer, make sure that it is a Classic Load Balancer\. Kinesis Data Firehose supports neither Application Load Balancers nor Network Load Balancers\. Also, enable duration\-based sticky sessions with cookie expiration disabled\. For information about how to do this, see [Duration\-Based Session Stickiness](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-sticky-sessions.html#enable-sticky-sessions-duration)\.

You are required to have an IAM role when creating a delivery stream\. Kinesis Data Firehose assumes that IAM role and gains access to the specified bucket, key, and CloudWatch log group and streams\.

Use the following access policy to enable Kinesis Data Firehose to access your S3 bucket\. If you don't own the S3 bucket, add `s3:PutObjectAcl` to the list of Amazon S3 actions, which grants the bucket owner full access to the objects delivered by Kinesis Data Firehose\. This policy also grants Kinesis Data Firehose access to CloudWatch for error logging and to AWS Lambda for data transformation\. The policy also has a statement that allows access to Amazon Kinesis Data Streams\. If you don't use Kinesis Data Streams as your data source, you can remove that statement\. Kinesis Data Firehose doesn't use IAM to access Splunk\. For accessing Splunk, it uses your HEC token\.

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
               "kinesis:GetRecords",
               "kinesis:ListShards"
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

For more information about allowing other AWS services to access your AWS resources, see [Creating a Role to Delegate Permissions to an AWS Service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) in the *IAM User Guide*\.

## Access to Splunk in VPC<a name="using-iam-splunk-vpc"></a>

If your Splunk platform is in a VPC, it must be publicly accessible with a public IP address\. Also, grant Kinesis Data Firehose access to your Splunk platform by unblocking the Kinesis Data Firehose IP addresses\. Kinesis Data Firehose currently uses the following CIDR blocks\.
+ `18.216.68.160/27, 18.216.170.64/27, 18.216.170.96/27` for US East \(Ohio\)
+ `34.238.188.128/26, 34.238.188.192/26, 34.238.195.0/26` for US East \(N\. Virginia\)
+ `13.57.180.0/26` for US West \(N\. California\)
+ `34.216.24.32/27, 34.216.24.192/27, 34.216.24.224/27` for US West \(Oregon\)
+ `18.253.138.192/26` for AWS GovCloud \(US\-East\)
+ `52.61.204.192/26` for AWS GovCloud \(US\-West\)
+ `18.162.221.64/26` for Asia Pacific \(Hong Kong\)
+ `13.232.67.64/26` for Asia Pacific \(Mumbai\)
+ `13.209.71.0/26` for Asia Pacific \(Seoul\)
+ `13.229.187.128/26` for Asia Pacific \(Singapore\)
+ `13.211.12.0/26` for Asia Pacific \(Sydney\)
+ `13.230.21.0/27, 13.230.21.32/27` for Asia Pacific \(Tokyo\)
+ `35.183.92.64/26` for Canada \(Central\)
+ `18.194.95.192/27, 18.194.95.224/27, 18.195.48.0/27` for Europe \(Frankfurt\)
+ `34.241.197.32/27, 34.241.197.64/27, 34.241.197.96/27` for Europe \(Ireland\)
+ `18.130.91.0/26` for Europe \(London\)
+ `35.180.112.0/26` for Europe \(Paris\)
+ `13.53.191.0/26` for Europe \(Stockholm\)
+ `15.185.91.64/26` for Middle East \(Bahrain\)
+ `18.228.1.192/26` for South America \(São Paulo\)
+ `15.161.135.192/26` for Europe \(Milan\)
+ `13.244.165.128/26` for Africa \(Cape Town\)

## Grant Kinesis Data Firehose Access to an HTTP Endpoint Destination<a name="using-iam-http"></a>

You can use Kinesis Data Firehose to deliver data to any HTTP endpoint destination\. Kinesis Data Firehose also backs up that data to the Amazon S3 bucket that you specify, and you can optionally use an AWS KMS key that you own for Amazon S3 server\-side encryption\. If error logging is enabled, Kinesis Data Firehose sends data delivery errors to your CloudWatch log streams\. You can also use AWS Lambda for data transformation\. 

You are required to have an IAM role when creating a delivery stream\. Kinesis Data Firehose assumes that IAM role and gains access to the specified bucket, key, and CloudWatch log group and streams\.

Use the following access policy to enable Kinesis Data Firehose to access the S3 bucket that you specified for data backup\. If you don't own the S3 bucket, add `s3:PutObjectAcl` to the list of Amazon S3 actions, which grants the bucket owner full access to the objects delivered by Kinesis Data Firehose\. This policy also grants Kinesis Data Firehose access to CloudWatch for error logging and to AWS Lambda for data transformation\. The policy also has a statement that allows access to Amazon Kinesis Data Streams\. If you don't use Kinesis Data Streams as your data source, you can remove that statement\. 

**Important**  
Kinesis Data Firehose doesn't use IAM to access HTTP endpoint destinations owned by supported third\-party service providers, including DataDog, MongoDB, and New Relic\. For accessing a specified HTTP endpoint destination owned by a supported third\-party service provider, contact that service provider to obtain the API key or the access key that is required to enable data delivery to that service from Kinesis Data Firehose\.

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
               "kinesis:GetRecords",
               "kinesis:ListShards"
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

For more information about allowing other AWS services to access your AWS resources, see [Creating a Role to Delegate Permissions to an AWS Service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) in the *IAM User Guide*\.

**Important**  
Currently Kinesis Data Firehose does NOT support data delivery to HTTP endpoints in a VPC\.

## Cross\-Account Delivery to an Amazon S3 Destination<a name="cross-account-delivery-s3"></a>

You can use the AWS CLI or the Kinesis Data Firehose APIs to create a delivery stream in one AWS account with an Amazon S3 destination in a different account\. The following procedure shows an example of configuring a Kinesis Data Firehose delivery stream owned by account A to deliver data to an Amazon S3 bucket owned by account B\.

1. Create an IAM role under account A using steps described in [Grant Kinesis Firehose Access to an Amazon S3 Destination](https://docs.aws.amazon.com/firehose/latest/dev/controlling-access.html#using-iam-s3)\. 
**Note**  
The Amazon S3 bucket specified in the access policy is owned by account B in this case\. Make sure you add `s3:PutObjectAcl` to the list of Amazon S3 actions in the access policy, which grants account B full access to the objects delivered by Amazon Kinesis Data Firehose\.

1. To allow access from the IAM role previously created, create an S3 bucket policy under account B\. The following code is an example of the bucket policy\. For more information, see [Using Bucket Policies and User Policies](https://docs.aws.amazon.com/AmazonS3/latest/dev/using-iam-policies.html)\. 

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

## Cross\-Account Delivery to an Amazon ES Destination<a name="cross-account-delivery-es"></a>

You can use the AWS CLI or the Kinesis Data Firehose APIs to create a delivery stream in one AWS account with an Amazon ES destination in a different account\. The following procedure shows an example of how you can create a Kinesis Data Firehose delivery stream under account A and configure it to deliver data to an Amazon ES destination owned by account B\.

1. Create an IAM role under account A using the steps described in [Grant Kinesis Data Firehose Access to a Public Amazon ES Destination](#using-iam-es)\. 

1. To allow access from the IAM role that you created in the previous step, create an Amazon ES policy under account B\. The following JSON is an example\.

   ```
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "AWS": "arn:aws:iam::Account-A-ID:role/firehose_delivery_role "
         },
         "Action": "es:ESHttpGet",
         "Resource": [
           "arn:aws:es:us-east-1:Account-B-ID:domain/cross-account-cluster/_all/_settings",
           "arn:aws:es:us-east-1:Account-B-ID:domain/cross-account-cluster/_cluster/stats",
           "arn:aws:es:us-east-1:Account-B-ID:domain/cross-account-cluster/roletest*/_mapping/roletest",
           "arn:aws:es:us-east-1:Account-B-ID:domain/cross-account-cluster/_nodes",
           "arn:aws:es:us-east-1:Account-B-ID:domain/cross-account-cluster/_nodes/stats",
           "arn:aws:es:us-east-1:Account-B-ID:domain/cross-account-cluster/_nodes/*/stats",
           "arn:aws:es:us-east-1:Account-B-ID:domain/cross-account-cluster/_stats",
           "arn:aws:es:us-east-1:Account-B-ID:domain/cross-account-cluster/roletest*/_stats"
         ]
       }
     ]
   }
   ```

1. Create a Kinesis Data Firehose delivery stream under account A using the IAM role that you created in step 1\. When you create the delivery stream, use the AWS CLI or the Kinesis Data Firehose APIs and specify the `ClusterEndpoint` field instead of `DomainARN` for Amazon ES\.

**Note**  
To create a delivery stream in one AWS account with an Amazon ES destination in a different account, you must use the AWS CLI or the Kinesis Data Firehose APIs\. You can't use the AWS Management Console to create this kind of cross\-account configuration\.

## Using Tags to Control Access<a name="tag-based-access-control"></a>

You can use the optional `Condition` element \(or `Condition` *block*\) in an IAM policy to fine\-tune access to Kinesis Data Firehose operations based on tag keys and values\. The following subsections describe how to do this for the different Kinesis Data Firehose operations\. For more on the use of the `Condition` element and the operators that you can use within it, see [IAM JSON Policy Elements: Condition](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html)\.

### CreateDeliveryStream and TagDeliveryStream<a name="aws-requesttag"></a>

For the `CreateDeliveryStream` and `TagDeliveryStream` operations, use the `aws:RequestTag` condition key\. In the following example, `MyKey` and `MyValue` represent the key and corresponding value for a tag\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "firehose:CreateDeliveryStream",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestTag/MyKey": "MyValue"
                }
            }
        }
    ]
}
```

### UntagDeliveryStream<a name="aws-tagkeys"></a>

For the `UntagDeliveryStream` operation, use the `aws:TagKeys` condition key\. In the following example, `MyKey` is an example tag key\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "firehose:UntagDeliveryStream",
            "Resource": "*",
            "Condition": {
                "ForAnyValue:StringEquals": {
                    "aws:TagKeys": "MyKey"
                 }
            }
        }
    ]
}
```

### ListDeliveryStreams<a name="list-delivery-streams"></a>

You can't use tag\-based access control with `ListDeliveryStreams`\.

### Other Kinesis Data Firehose Operations<a name="firehose-resourcetag"></a>

For all Kinesis Data Firehose operations other than `CreateDeliveryStream`, `TagDeliveryStream`, `UntagDeliveryStream`, and `ListDeliveryStreams`, use the `aws:RequestTag` condition key\. In the following example, `MyKey` and `MyValue` represent the key and corresponding value for a tag\.

```
{
    "Version": "2012-10-17",
    "Statement": [
    {
            "Effect": "Deny",
            "Action": "firehose:DescribeDeliveryStream",
            "Resource": "*",
            "Condition": {
                "Null": {
                    "firehose:ResourceTag/MyKey": "MyValue"
            }
        }
    ]
}
```