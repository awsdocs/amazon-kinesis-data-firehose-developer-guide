# Step 3: Send the Data from Amazon CloudWatch to Kinesis Data Firehose<a name="cw-to-delivery-stream"></a>

In this step of this [Kinesis Data Firehose tutorial](vpc-splunk-tutorial.md), you subscribe the delivery stream to the Amazon CloudWatch log group\. This step causes the log data to flow from the log group to the delivery stream\.

**To send log data from CloudWatch Logs to your delivery stream**

In this procedure, you use the [AWS Command Line Interface \(AWS CLI\)](https://aws.amazon.com/cli/) to create a CloudWatch Logs subscription that sends log events to your delivery stream\. 

1. Save the following trust policy to a local file, and name the file `VPCtoSplunkCWtoFHTrustPolicy.json`\. Be sure to replace the *your\-region* placeholder with your AWS Region code\.

   ```
   {
     "Statement": {
       "Effect": "Allow",
       "Principal": { "Service": "logs.your-region.amazonaws.com" },
       "Action": "sts:AssumeRole"
     }
   }
   ```

1. In a command window, go to the directory where you saved `VPCtoSplunkCWtoFHPolicy.json`, and run the following AWS CLI command\. 

   ```
   aws iam create-role --role-name VPCtoSplunkCWtoFHRole --assume-role-policy-document file://VPCtoSplunkCWtoFHTrustPolicy.json
   ```

1. Save the following access policy to a local file, and name the file `VPCtoSplunkCWtoFHAccessPolicy.json`\. Be sure to replace the *your\-region* and *your\-aws\-account\-id* placeholders with your AWS Region code and account ID\.

   ```
   {
       "Statement":[
         {
           "Effect":"Allow",
           "Action":["firehose:*"],
           "Resource":["arn:aws:firehose:your-region:your-aws-account-id:deliverystream/VPCtoSplunkStream"]
         },
         {
           "Effect":"Allow",
           "Action":["iam:PassRole"],
           "Resource":["arn:aws:iam::your-aws-account-id:role/VPCtoSplunkCWtoFHRole"]
         }
       ]
   }
   ```

1. In a command window, go to the directory where you saved `VPCtoSplunkCWtoFHAccessPolicy.json`, and run the following AWS CLI command\.

   ```
   aws iam put-role-policy --role-name VPCtoSplunkCWtoFHRole --policy-name VPCtoSplunkCWtoFHAccessPolicy --policy-document file://VPCtoSplunkCWtoFHAccessPolicy.json
   ```

1. Replace the *your\-region* and *your\-aws\-account\-id* placeholders in the following AWS CLI command with your AWS Region code and account ID, and then run the command\.

   ```
   aws logs put-subscription-filter --log-group-name "VPCtoSplunkLogGroup" --filter-name "Destination" --filter-pattern "" --destination-arn "arn:aws:firehose:your-region:your-aws-account-id:deliverystream/VPCtoSplunkStream" --role-arn "arn:aws:iam::your-aws-account-id:role/VPCtoSplunkCWtoFHRole"
   ```

Proceed to [Step 4: Check the Results in Splunk and in Kinesis Data Firehose](check-vpc-to-splunk-results.md)\.