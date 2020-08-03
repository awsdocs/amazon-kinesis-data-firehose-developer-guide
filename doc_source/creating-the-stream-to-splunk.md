# Step 2: Create a Kinesis Data Firehose Delivery Stream with Splunk as a Destination<a name="creating-the-stream-to-splunk"></a>

In this part of the [Kinesis Data Firehose tutorial](vpc-splunk-tutorial.md), you create an Amazon Kinesis Data Firehose delivery stream to receive the log data from Amazon CloudWatch and deliver that data to Splunk\. 

The logs that CloudWatch sends to the delivery stream are in a compressed format\. However, Kinesis Data Firehose can't send compressed logs to Splunk\. Therefore, when you create the delivery stream in the following procedure, you enable data transformation and configure an AWS Lambda function to uncompress the log data\. Kinesis Data Firehose then sends the uncompressed data to Splunk\.

**To create a Kinesis Data Firehose delivery stream with Splunk as a destination**

1. Open the Kinesis Data Firehose console at [https://console\.aws\.amazon\.com/firehose/](https://console.aws.amazon.com/firehose/)\.

1. Choose **Create delivery stream**\.

1. For the name of the delivery stream, enter **VPCtoSplunkStream**\. Then scroll to the bottom, and choose **Next**\.

1. For **Data transformation\***, choose **Enabled**\.

1. For **Lambda function\***, choose **Create new**\.

1. In the **Choose Lambda blueprint** pane, scroll down and choose **Kinesis Firehose Cloudwatch Logs Processor**\. This opens the AWS Lambda console\.  
![\[Screenshot of the Lambda blueprint window showing the Kinesis Firehose Cloudwatch Logs Processor blueprint.\]](http://docs.aws.amazon.com/firehose/latest/dev/images/firehose-lambdablueprint-console.png)

1. On the AWS Lambda console, for the function name, enter **VPCtoSplunkLambda**\.

1. In the description text under **Execution role**, choose the **IAM console** link to create a custom role\. This opens the AWS Identity and Access Management \(IAM\) console\.

1. In the IAM console, choose **Lambda**\.

1. Choose **Next: Permissions**\.

1. Choose **Create policy**\.

1. Choose the **JSON** tab and replace the existing JSON with the following\. Be sure to replace the *your\-region* and *your\-aws\-account\-id* placeholders with your AWS Region code and account ID\. Don't include any hyphens or dashes in the account ID\. For a list of AWS Region codes, see [AWS Regions and Endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html)\.

   ```
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "logs:GetLogEvents"
               ],
               "Resource": "arn:aws:logs:*:*:*"
           },
           {
               "Effect": "Allow",
               "Action": [
                   "firehose:PutRecordBatch"
               ],
               "Resource": [
                   "arn:aws:firehose:your-region:your-aws-account-id:deliverystream/VPCtoSplunkStream"
               ]
           }
       ]
   }
   ```

   This policy allows the Lambda function to put data back into the delivery stream by invoking the `PutRecordBatch` operation\. This step is needed because a Lambda function can only return up to 6 MiB of data every time Kinesis Data Firehose invokes it\. If the size of the uncompressed data exceeds 6 MiB, the function invokes `PutRecordBatch` to put some of the data back into the delivery stream for future processing\. 

1. Back in the **Create role** window, refresh the list of policies, then choose **VPCtoSplunkLambdaPolicy** by selecting the box to its left\.

1. Choose **Next: Tags**\.

1. Choose **Next: Review**\.

1. For **Role Name**, enter **VPCtoSplunkLambdaRole**, then choose **Create role**\.

1. Back in the Lambda console, refresh the list of existing roles, then select **VPCtoSplunkLambdaRole**\.

1. Scroll down and choose **Create function**\.

1. In the Lambda function pane, scroll down to the **Basic settings** section, and increase the timeout to **3** minutes\.

1. Scroll up and choose **Save**\.

1. Back in the **Choose Lambda blueprint** dialog box, choose **Close**\.

1. On the delivery stream creation page, under the **Transform source records with AWS Lambda** section, choose the **refresh** button\. Then choose **VPCtoSplunkLambda** in the list of functions\.

1. Scroll down and choose **Next**\.

1. For **Destination\***, choose **Splunk**\.

1. For **Splunk cluster endpoint**, see the information at [Configure Amazon Kinesis Firehose to send data to the Splunk platform](http://docs.splunk.com/Documentation/AddOns/latest/Firehose/ConfigureFirehose) in the Splunk documentation\.

1. Keep **Splunk endpoint type** set to **Raw endpoint**\.

1. Enter the value \(and not the name\) of your Splunk HTTP Event Collector \(HEC\) token\.

1. For **S3 backup mode\***, choose **Backup all events**\.

1. Choose an existing Amazon S3 bucket \(or create a new one if you want\), and choose **Next**\.

1. On the **Configure settings** page, scroll down to the **IAM role** section, and choose **Create new or choose**\.

1. In the **IAM role** list, choose **Create a new IAM role**\. For **Role Name**, enter **VPCtoSplunkLambdaFirehoseRole**, and then choose **Allow**\.

1. Choose **Next**, and review the configuration that you chose for the delivery stream\. Then choose **Create delivery stream**\.

Proceed to [Step 3: Send the Data from Amazon CloudWatch to Kinesis Data Firehose](cw-to-delivery-stream.md)\.