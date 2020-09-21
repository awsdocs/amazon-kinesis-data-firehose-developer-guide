# Writing to Kinesis Data Firehose Using Kinesis Agent<a name="writing-with-agents"></a>

Amazon Kinesis agent is a standalone Java software application that offers an easy way to collect and send data to Kinesis Data Firehose\. The agent continuously monitors a set of files and sends new data to your Kinesis Data Firehose delivery stream\. The agent handles file rotation, checkpointing, and retry upon failures\. It delivers all of your data in a reliable, timely, and simple manner\. It also emits Amazon CloudWatch metrics to help you better monitor and troubleshoot the streaming process\.

By default, records are parsed from each file based on the newline \(`'\n'`\) character\. However, the agent can also be configured to parse multi\-line records \(see [Agent Configuration Settings](#agent-config-settings)\)\. 

You can install the agent on Linux\-based server environments such as web servers, log servers, and database servers\. After installing the agent, configure it by specifying the files to monitor and the delivery stream for the data\. After the agent is configured, it durably collects data from the files and reliably sends it to the delivery stream\.

**Topics**
+ [Prerequisites](#prereqs)
+ [Credentials](#agent-credentials)
+ [Custom Credential Providers](#custom-cred-provider)
+ [Download and Install the Agent](#download-install)
+ [Configure and Start the Agent](#config-start)
+ [Agent Configuration Settings](#agent-config-settings)
+ [Monitor Multiple File Directories and Write to Multiple Streams](#sim-writes)
+ [Use the agent to Preprocess Data](#pre-processing)
+ [agent CLI Commands](#cli-commands)

## Prerequisites<a name="prereqs"></a>
+ Your operating system must be Amazon Linux, or Red Hat Enterprise Linux version 7 or later\. 
+ Agent version 2\.0\.0 or later runs using JRE version 1\.8 or later\. Agent version 1\.1\.x runs using JRE 1\.7 or later\. 
+ If you are using Amazon EC2 to run your agent, launch your EC2 instance\.
+ The IAM role or AWS credentials that you specify must have permission to perform the Kinesis Data Firehose [PutRecordBatch](https://docs.aws.amazon.com/firehose/latest/APIReference/API_PutRecordBatch.html) operation for the agent to send data to your delivery stream\. If you enable CloudWatch monitoring for the agent, permission to perform the CloudWatch [PutMetricData](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_PutMetricData.html) operation is also needed\. For more information, see [Controlling Access with Amazon Kinesis Data Firehose ](controlling-access.md), [Monitoring Kinesis Agent Health](agent-health.md), and [Authentication and Access Control for Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/auth-and-access-control-cw.html)\.

## Credentials<a name="agent-credentials"></a>

Manage your AWS credentials using one of the following methods:
+ Create a custom credentials provider\. For details, see [Custom Credential Providers](#custom-cred-provider)\.
+ Specify an IAM role when you launch your EC2 instance\.
+ Specify AWS credentials when you configure the agent \(see the entries for `awsAccessKeyId` and `awsSecretAccessKey` in the configuration table under [Agent Configuration Settings](#agent-config-settings)\)\.
+ Edit `/etc/sysconfig/aws-kinesis-agent` to specify your AWS Region and AWS access keys\.
+ If your EC2 instance is in a different AWS account, create an IAM role to provide access to the Kinesis Data Firehose service\. Specify that role when you configure the agent \(see [assumeRoleARN](#assumeRoleARN) and [assumeRoleExternalId](#assumeRoleExternalId)\)\. Use one of the previous methods to specify the AWS credentials of a user in the other account who has permission to assume this role\.

## Custom Credential Providers<a name="custom-cred-provider"></a>

You can create a custom credentials provider and give its class name and jar path to the Kinesis agent in the following configuration settings: `userDefinedCredentialsProvider.classname` and `userDefinedCredentialsProvider.location`\. For the descriptions of these two configuration settings, see [Agent Configuration Settings](#agent-config-settings)\.

To create a custom credentials provider, define a class that implements the `AWSCredentialsProvider` interface, like the one in the following example\.

```
import com.amazonaws.auth.AWSCredentials;
import com.amazonaws.auth.AWSCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;

public class YourClassName implements AWSCredentialsProvider {
    public YourClassName() {
    }

    public AWSCredentials getCredentials() {
        return new BasicAWSCredentials("key1", "key2");
    }

    public void refresh() {
    }
}
```

Your class must have a constructor that takes no arguments\.

AWS invokes the refresh method periodically to get updated credentials\. If you want your credentials provider to provide different credentials throughout its lifetime, include code to refresh the credentials in this method\. Alternatively, you can leave this method empty if you want a credentials provider that vends static \(non\-changing\) credentials\. 

## Download and Install the Agent<a name="download-install"></a>

First, connect to your instance\. For more information, see [Connect to Your Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-connect-to-instance-linux.html) in the *Amazon EC2 User Guide for Linux Instances*\. If you have trouble connecting, see [Troubleshooting Connecting to Your Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstancesConnecting.html) in the *Amazon EC2 User Guide for Linux Instances*\.

Next, install the agent using one of the following methods\.
+ **To set up the agent from the Amazon Linux repositories **

  This method works only for Amazon Linux instances\. Use the following command:

  ```
  sudo yum install –y aws-kinesis-agent
  ```

  Agent v 2\.0\.0 or later is installed on computers with operating system Amazon Linux 2 \(AL2\)\. This agent version requires Java 1\.8 or later\. If required Java version is not yet present, the agent installation process installs it\. For more information regarding Amazon Linux 2 see [https://aws\.amazon\.com/amazon\-linux\-2/](https://aws.amazon.com/amazon-linux-2/)\.
+ **To set up the agent from the Amazon S3 repository**

  This method works for Red Hat Enterprise Linux, as well as Amazon Linux 2 instances because it installs the agent from the publicly available repository\. Use the following command to download and install the latest version of the agent version 2\.x\.x: 

  ```
  sudo yum install –y https://s3.amazonaws.com/streaming-data-agent/aws-kinesis-agent-latest.amzn2.noarch.rpm
  ```

  To install a specific version of the agent, specify the version number in the command\. For example, the following command installs agent v 2\.0\.0\. 

  ```
  sudo yum install –y https://streaming-data-agent.s3.amazonaws.com/aws-kinesis-agent-2.0.0-2.amzn2.noarch.rpm
  ```

  If you have Java 1\.7 and you don’t want to upgrade it, you can download agent version 1\.x\.x, which is compatible with Java 1\.7\. For example, to download agent v1\.1\.6, you can use the following command: 

  ```
  sudo yum install –y https://s3.amazonaws.com/streaming-data-agent/aws-kinesis-agent-1.1.6-1.amzn1.noarch.rpm
  ```

  The latest agent v1\.x\.x can be downloaded using the following command:

  ```
  sudo yum install –y https://s3.amazonaws.com/streaming-data-agent/aws-kinesis-agent-latest.amzn1.noarch.rpm
  ```
+ **To set up the agent from the GitHub repo**

  1. First, make sure that you have required Java version installed, depending on agent version\.

  1.  Download the agent from the [awslabs/amazon\-kinesis\-agent](https://github.com/awslabs/amazon-kinesis-agent) GitHub repo\.

  1. Install the agent by navigating to the download directory and running the following command:

     ```
     sudo ./setup --install
     ```

## Configure and Start the Agent<a name="config-start"></a>

**To configure and start the agent**

1. Open and edit the configuration file \(as superuser if using default file access permissions\): `/etc/aws-kinesis/agent.json` 

   In this configuration file, specify the files \( `"filePattern"` \) from which the agent collects data, and the name of the delivery stream \( `"deliveryStream"` \) to which the agent sends data\. The file name is a pattern, and the agent recognizes file rotations\. You can rotate files or create new files no more than once per second\. The agent uses the file creation time stamp to determine which files to track and tail into your delivery stream\. Creating new files or rotating files more frequently than once per second does not allow the agent to differentiate properly between them\.

   ```
   { 
      "flows": [
           { 
               "filePattern": "/tmp/app.log*", 
               "deliveryStream": "yourdeliverystream"
           } 
      ] 
   }
   ```

   The default AWS Region is `us-east-1`\. If you are using a different Region, add the `firehose.endpoint` setting to the configuration file, specifying the endpoint for your Region\. For more information, see [Agent Configuration Settings](#agent-config-settings)\.

1. Start the agent manually:

   ```
   sudo service aws-kinesis-agent start
   ```

1. \(Optional\) Configure the agent to start on system startup:

   ```
   sudo chkconfig aws-kinesis-agent on
   ```

The agent is now running as a system service in the background\. It continuously monitors the specified files and sends data to the specified delivery stream\. Agent activity is logged in `/var/log/aws-kinesis-agent/aws-kinesis-agent.log`\. 

## Agent Configuration Settings<a name="agent-config-settings"></a>

The agent supports two mandatory configuration settings, `filePattern` and `deliveryStream`, plus optional configuration settings for additional features\. You can specify both mandatory and optional configuration settings in `/etc/aws-kinesis/agent.json`\.

Whenever you change the configuration file, you must stop and start the agent, using the following commands:

```
sudo service aws-kinesis-agent stop
sudo service aws-kinesis-agent start
```

Alternatively, you could use the following command:

```
sudo service aws-kinesis-agent restart
```

The following are the general configuration settings\.


| Configuration Setting | Description | 
| --- | --- | 
| <a name="assumeRoleARN"></a>assumeRoleARN |  The Amazon Resource Name \(ARN\) of the role to be assumed by the user\. For more information, see [Delegate Access Across AWS Accounts Using IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html) in the *IAM User Guide*\.  | 
| <a name="assumeRoleExternalId"></a>assumeRoleExternalId |  An optional identifier that determines who can assume the role\. For more information, see [How to Use an External ID](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user_externalid.html) in the *IAM User Guide*\.  | 
| <a name="awsAccessKeyId"></a>awsAccessKeyId |  AWS access key ID that overrides the default credentials\. This setting takes precedence over all other credential providers\.  | 
| <a name="awsSecretAccessKey"></a>awsSecretAccessKey |  AWS secret key that overrides the default credentials\. This setting takes precedence over all other credential providers\.  | 
| cloudwatch\.emitMetrics |  Enables the agent to emit metrics to CloudWatch if set \(true\)\. Default: true  | 
| cloudwatch\.endpoint |  The regional endpoint for CloudWatch\. Default: `monitoring.us-east-1.amazonaws.com`  | 
| firehose\.endpoint |  The regional endpoint for Kinesis Data Firehose\. Default: `firehose.us-east-1.amazonaws.com`  | 
| userDefinedCredentialsProvider\.classname | If you define a custom credentials provider, provide its fully\-qualified class name using this setting\. Don't include \.class at the end of the class name\.  | 
| userDefinedCredentialsProvider\.location | If you define a custom credentials provider, use this setting to specify the absolute path of the jar that contains the custom credentials provider\. The agent also looks for the jar file in the following location: /usr/share/aws\-kinesis\-agent/lib/\. | 

The following are the flow configuration settings\.


| Configuration Setting | Description | 
| --- | --- | 
| aggregatedRecordSizeBytes |  To make the agent aggregate records and then put them to the delivery stream in one operation, specify this setting\. Set it to the size that you want the aggregate record to have before the agent puts it to the delivery stream\.  Default: 0 \(no aggregation\)  | 
| dataProcessingOptions |  The list of processing options applied to each parsed record before it is sent to the delivery stream\. The processing options are performed in the specified order\. For more information, see [Use the agent to Preprocess Data](#pre-processing)\.  | 
| deliveryStream |  \[Required\] The name of the delivery stream\.  | 
| filePattern |  \[Required\] A glob for the files that need to be monitored by the agent\. Any file that matches this pattern is picked up by the agent automatically and monitored\. For all files matching this pattern, grant read permission to `aws-kinesis-agent-user`\. For the directory containing the files, grant read and execute permissions to `aws-kinesis-agent-user`\.  The agent picks up any file that matches this pattern\. To ensure that the agent doesn't pick up unintended records, choose this pattern carefully\.   | 
| initialPosition |  The initial position from which the file started to be parsed\. Valid values are `START_OF_FILE` and `END_OF_FILE`\. Default: `END_OF_FILE`  | 
| maxBufferAgeMillis |  The maximum time, in milliseconds, for which the agent buffers data before sending it to the delivery stream\. Value range: 1,000–900,000 \(1 second to 15 minutes\) Default: 60,000 \(1 minute\)  | 
| maxBufferSizeBytes |  The maximum size, in bytes, for which the agent buffers data before sending it to the delivery stream\. Value range: 1–4,194,304 \(4 MB\) Default: 4,194,304 \(4 MB\)  | 
| maxBufferSizeRecords |  The maximum number of records for which the agent buffers data before sending it to the delivery stream\. Value range: 1–500 Default: 500  | 
| minTimeBetweenFilePollsMillis |  The time interval, in milliseconds, at which the agent polls and parses the monitored files for new data\. Value range: 1 or more Default: 100  | 
| multiLineStartPattern |  The pattern for identifying the start of a record\. A record is made of a line that matches the pattern and any following lines that don't match the pattern\. The valid values are regular expressions\. By default, each new line in the log files is parsed as one record\.  | 
| skipHeaderLines |  The number of lines for the agent to skip parsing at the beginning of monitored files\. Value range: 0 or more Default: 0 \(zero\)  | 
| truncatedRecordTerminator |  The string that the agent uses to truncate a parsed record when the record size exceeds the Kinesis Data Firehose record size limit\. \(1,000 KB\) Default: `'\n'` \(newline\)  | 

## Monitor Multiple File Directories and Write to Multiple Streams<a name="sim-writes"></a>

By specifying multiple flow configuration settings, you can configure the agent to monitor multiple file directories and send data to multiple streams\. In the following configuration example, the agent monitors two file directories and sends data to a Kinesis data stream and a Kinesis Data Firehose delivery stream respectively\. You can specify different endpoints for Kinesis Data Streams and Kinesis Data Firehose so that your data stream and Kinesis Data Firehose delivery stream don’t need to be in the same Region\.

```
{
    "cloudwatch.emitMetrics": true,
    "kinesis.endpoint": "https://your/kinesis/endpoint", 
    "firehose.endpoint": "https://your/firehose/endpoint", 
    "flows": [
        {
            "filePattern": "/tmp/app1.log*", 
            "kinesisStream": "yourkinesisstream"
        }, 
        {
            "filePattern": "/tmp/app2.log*",
            "deliveryStream": "yourfirehosedeliverystream" 
        }
    ] 
}
```

For more detailed information about using the agent with Amazon Kinesis Data Streams, see [Writing to Amazon Kinesis Data Streams with Kinesis Agent](https://docs.aws.amazon.com/kinesis/latest/dev/writing-with-agents.html)\.

## Use the agent to Preprocess Data<a name="pre-processing"></a>

The agent can pre\-process the records parsed from monitored files before sending them to your delivery stream\. You can enable this feature by adding the `dataProcessingOptions` configuration setting to your file flow\. One or more processing options can be added, and they are performed in the specified order\.

The agent supports the following processing options\. Because the agent is open source, you can further develop and extend its processing options\. You can download the agent from [Kinesis Agent](https://github.com/awslabs/amazon-kinesis-agent)\.Processing Options

`SINGLELINE`  
Converts a multi\-line record to a single\-line record by removing newline characters, leading spaces, and trailing spaces\.  

```
{
    "optionName": "SINGLELINE"
}
```

`CSVTOJSON`  
Converts a record from delimiter\-separated format to JSON format\.  

```
{
    "optionName": "CSVTOJSON",
    "customFieldNames": [ "field1", "field2", ... ],
    "delimiter": "yourdelimiter"
}
```  
`customFieldNames`  
\[Required\] The field names used as keys in each JSON key value pair\. For example, if you specify `["f1", "f2"]`, the record "v1, v2" is converted to `{"f1":"v1","f2":"v2"}`\.  
`delimiter`  
The string used as the delimiter in the record\. The default is a comma \(,\)\.

`LOGTOJSON`  
Converts a record from a log format to JSON format\. The supported log formats are Apache Common Log, Apache Combined Log, Apache Error Log, and RFC3164 Syslog\.  

```
{
    "optionName": "LOGTOJSON",
    "logFormat": "logformat",
    "matchPattern": "yourregexpattern",
    "customFieldNames": [ "field1", "field2", … ]
}
```  
`logFormat`  
\[Required\] The log entry format\. The following are possible values:  
+ `COMMONAPACHELOG` — The Apache Common Log format\. Each log entry has the following pattern by default: "`%{host} %{ident} %{authuser} [%{datetime}] \"%{request}\" %{response} %{bytes}`"\.
+ `COMBINEDAPACHELOG` — The Apache Combined Log format\. Each log entry has the following pattern by default: "`%{host} %{ident} %{authuser} [%{datetime}] \"%{request}\" %{response} %{bytes} %{referrer} %{agent}`"\.
+ `APACHEERRORLOG` — The Apache Error Log format\. Each log entry has the following pattern by default: "`[%{timestamp}] [%{module}:%{severity}] [pid %{processid}:tid %{threadid}] [client: %{client}] %{message}`"\.
+ `SYSLOG` — The RFC3164 Syslog format\. Each log entry has the following pattern by default: "`%{timestamp} %{hostname} %{program}[%{processid}]: %{message}`"\.  
`matchPattern`  
Overrides the default pattern for the specified log format\. Use this setting to extract values from log entries if they use a custom format\. If you specify `matchPattern`, you must also specify `customFieldNames`\.  
`customFieldNames`  
The custom field names used as keys in each JSON key value pair\. You can use this setting to define field names for values extracted from `matchPattern`, or override the default field names of predefined log formats\.

**Example : LOGTOJSON Configuration**  <a name="example-logtojson"></a>
Here is one example of a `LOGTOJSON` configuration for an Apache Common Log entry converted to JSON format:  

```
{
    "optionName": "LOGTOJSON",
    "logFormat": "COMMONAPACHELOG"
}
```
Before conversion:  

```
64.242.88.10 - - [07/Mar/2004:16:10:02 -0800] "GET /mailman/listinfo/hsdivision HTTP/1.1" 200 6291
```
After conversion:  

```
{"host":"64.242.88.10","ident":null,"authuser":null,"datetime":"07/Mar/2004:16:10:02 -0800","request":"GET /mailman/listinfo/hsdivision HTTP/1.1","response":"200","bytes":"6291"}
```

**Example : LOGTOJSON Configuration With Custom Fields**  <a name="example-logtojson-custom-fields"></a>
Here is another example `LOGTOJSON` configuration:  

```
{
    "optionName": "LOGTOJSON",
    "logFormat": "COMMONAPACHELOG",
    "customFieldNames": ["f1", "f2", "f3", "f4", "f5", "f6", "f7"]
}
```
With this configuration setting, the same Apache Common Log entry from the previous example is converted to JSON format as follows:  

```
{"f1":"64.242.88.10","f2":null,"f3":null,"f4":"07/Mar/2004:16:10:02 -0800","f5":"GET /mailman/listinfo/hsdivision HTTP/1.1","f6":"200","f7":"6291"}
```

**Example : Convert Apache Common Log Entry**  <a name="example-apache-common-log-entry"></a>
The following flow configuration converts an Apache Common Log entry to a single\-line record in JSON format:  

```
{ 
    "flows": [
        {
            "filePattern": "/tmp/app.log*", 
            "deliveryStream": "my-delivery-stream",
            "dataProcessingOptions": [
                {
                    "optionName": "LOGTOJSON",
                    "logFormat": "COMMONAPACHELOG"
                }
            ]
        }
    ] 
}
```

**Example : Convert Multi\-Line Records**  <a name="example-convert-multi-line"></a>
The following flow configuration parses multi\-line records whose first line starts with "`[SEQUENCE=`"\. Each record is first converted to a single\-line record\. Then, values are extracted from the record based on a tab delimiter\. Extracted values are mapped to specified `customFieldNames` values to form a single\-line record in JSON format\.  

```
{ 
    "flows": [
        {
            "filePattern": "/tmp/app.log*", 
            "deliveryStream": "my-delivery-stream",
            "multiLineStartPattern": "\\[SEQUENCE=",
            "dataProcessingOptions": [
                {
                    "optionName": "SINGLELINE"
                },
                {
                    "optionName": "CSVTOJSON",
                    "customFieldNames": [ "field1", "field2", "field3" ],
                    "delimiter": "\\t"
                }
            ]
        }
    ] 
}
```

**Example : LOGTOJSON Configuration with Match Pattern**  <a name="example-logtojson-match-pattern"></a>
Here is one example of a `LOGTOJSON` configuration for an Apache Common Log entry converted to JSON format, with the last field \(bytes\) omitted:  

```
{
    "optionName": "LOGTOJSON",
    "logFormat": "COMMONAPACHELOG",
    "matchPattern": "^([\\d.]+) (\\S+) (\\S+) \\[([\\w:/]+\\s[+\\-]\\d{4})\\] \"(.+?)\" (\\d{3})",
    "customFieldNames": ["host", "ident", "authuser", "datetime", "request", "response"]
}
```
Before conversion:  

```
123.45.67.89 - - [27/Oct/2000:09:27:09 -0400] "GET /java/javaResources.html HTTP/1.0" 200
```
After conversion:  

```
{"host":"123.45.67.89","ident":null,"authuser":null,"datetime":"27/Oct/2000:09:27:09 -0400","request":"GET /java/javaResources.html HTTP/1.0","response":"200"}
```

## agent CLI Commands<a name="cli-commands"></a>

Automatically start the agent on system startup:

```
sudo chkconfig aws-kinesis-agent on
```

Check the status of the agent:

```
sudo service aws-kinesis-agent status
```

Stop the agent:

```
sudo service aws-kinesis-agent stop
```

Read the agent's log file from this location:

```
/var/log/aws-kinesis-agent/aws-kinesis-agent.log
```

Uninstall the agent:

```
sudo yum remove aws-kinesis-agent
```