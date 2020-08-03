# Step 1: Send Log Data from Amazon VPC to Amazon CloudWatch<a name="log-data-from-vpc-to-cw"></a>

In the first part of this [Kinesis Data Firehose tutorial](vpc-splunk-tutorial.md), you create an Amazon CloudWatch log group to receive your Amazon VPC flow logs\. Then, you create flow logs for your Amazon VPC and send them to the CloudWatch log group that you created\.

**To create a CloudWatch log group to receive your Amazon VPC flow logs**

1. Sign in to the AWS Management Console and open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. In the navigation pane, choose **Log groups**\.

1. Choose **Actions**, and then choose **Create log group**\.

1. Enter the name **VPCtoSplunkLogGroup**, and choose **Create log group**\.

**To create a VPC flow log**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Your VPCs**\. Then choose your VPC from the list by selecting the check box next to it\.  
![\[Screenshot of the VPC dashboard showing the actions menu and where to check the box to select your VPC.\]](http://docs.aws.amazon.com/firehose/latest/dev/images/firehose-selectvpc-console.png)

1. Choose **Actions**, and then choose **Create flow log**\.

1. In the **Filter\*** list, choose **All**\.

1. Keep the destination set to **Send to CloudWatch Logs**\.

1. For **Destination log group\***, choose **VPCtoSplunkLogGroup**, which is the log group that you created in the previous procedure\.

1. To set up an IAM role, choose **Set Up Permissions**\.   
![\[Screenshot showing the Set Up Permissions link.\]](http://docs.aws.amazon.com/firehose/latest/dev/images/firehose-createflowlog-console.png)

1. In the new window that appears, keep **IAM Role** set to **Create a new IAM Role**\. In the **Role Name** box, enter **VPCtoSplunkWritetoCWRole**\. Then choose **Allow**\.  
![\[Screenshot showing creating the role named VPCtoSplunkWritetoCWRole.\]](http://docs.aws.amazon.com/firehose/latest/dev/images/firehose-cwflowlogrole-console.png)

1. Return to the **Create flow log** browser tab, and refresh the **IAM role\*** box\. Then choose **VPCtoSplunkWritetoCWRole** in the list\.

1. Choose **Create**, and then choose **Close**\.

1. Back on the Amazon VPC dashboard, choose **Your VPCs** in the navigation pane\. Then select the check box next to your VPC\.

1. Scroll down and choose the **Flow Logs** tab, and look for the flow log that you created in the preceding steps\. Ensure that its status is **Active**\. If it is not, review the previous steps\.

Proceed to [Step 2: Create a Kinesis Data Firehose Delivery Stream with Splunk as a Destination](creating-the-stream-to-splunk.md)\.