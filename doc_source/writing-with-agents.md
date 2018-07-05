# Writing to Amazon Kinesis Data Firehose Using Kinesis Agent<a name="writing-with-agents"></a>

Kinesis Agent is a stand\-alone Java software application that offers an easy way to collect and send data to Kinesis Data Firehose\. The agent continuously monitors a set of files and sends new data to your Kinesis data delivery stream\. The agent handles file rotation, checkpointing, and retry upon failures\. It delivers all of your data in a reliable, timely, and simple manner\. It also emits Amazon CloudWatch metrics to help you better monitor and troubleshoot the streaming process\.

By default, records are parsed from each file based on the newline \(`'\n'`\) character\. However, the agent can also be configured to parse multi\-line records \(see [Agent Configuration Settings](#agent-config-settings)\)\. 

You can install the agent on Linux\-based server environments such as web servers, log servers, and database servers\. After installing the agent, configure it by specifying the files to monitor and the delivery stream for the data\. After the agent is configured, it durably collects data from the files and reliably sends it to the delivery stream\.

**Topics**
+ [Prerequisites](#prereqs)
+ [Download and Install the Agent](#download-install)
+ [Configure and Start the Agent](#config-start)
+ [Agent Configuration Settings](#agent-config-settings)
+ [Monitor Multiple File Directories and Write to Multiple Streams](#sim-writes)
+ [Use the Agent to Pre\-process Data](#pre-processing)
+ [Agent CLI Commands](#cli-commands)

## Prerequisites<a name="prereqs"></a>
+ Your operating system must be either Amazon Linux AMI with version 2015\.09 or later, or Red Hat Enterprise Linux version 7 or later\.
+ If you are using Amazon EC2 to run your agent, launch your EC2 instance\.
+ Manage your AWS credentials using one of the following methods:
  + Specify an IAM role when you launch your EC2 instance\.
  + Specify AWS credentials when you configure the agent \(see [awsAccessKeyId](#awsAccessKeyId) and [awsSecretAccessKey](#awsSecretAccessKey)\)\.
  + Edit `/etc/sysconfig/aws-kinesis-agent` to specify your region and AWS access keys\.
  + If your EC2 instance is in a different AWS account, create an IAM role to provide access to the Kinesis Data Firehose service, and specify that role when you configure the agent \(see [assumeRoleARN](#assumeRoleARN) and [assumeRoleExternalId](#assumeRoleExternalId)\)\. Use one of the previous methods to specify the AWS credentials of a user in the other account who has permission to assume this role\.
+ The IAM role or AWS credentials that you specify must have permission to perform the Kinesis Data Firehose [PutRecordBatch](http://docs.aws.amazon.com/firehose/latest/APIReference/API_PutRecordBatch.html) operation for the agent to send data to your delivery stream\. If you enable CloudWatch monitoring for the agent, permission to perform the CloudWatch [PutMetricData](http://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_PutMetricData.html) operation is also needed\. For more information, see [Controlling Access with Amazon Kinesis Data Firehose ](controlling-access.md), [Monitoring Kinesis Agent Health](agent-health.md), and [Authentication and Access Control for Amazon CloudWatch](http://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/auth-and-access-control-cw.html)\.

## Download and Install the Agent<a name="download-install"></a>

First, connect to your instance\. For more information, see [Connect to Your Instance](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-connect-to-instance-linux.html) in the *Amazon EC2 User Guide for Linux Instances*\. If you have trouble connecting, see [Troubleshooting Connecting to Your Instance](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstancesConnecting.html) in the *Amazon EC2 User Guide for Linux Instances*\.

Next, install the agent using one of the following methods\.

**To set up the agent using the Amazon Linux AMI**  
Use the following command to download and install the agent:

```
sudo yum install –y aws-kinesis-agent
```

**To set up the agent using Red Hat Enterprise Linux**  
Use the following command to download and install the agent:

```
sudo yum install –y https://s3.amazonaws.com/streaming-data-agent/aws-kinesis-agent-latest.amzn1.noarch.rpm
```

**To set up the agent using GitHub**

1. Download the agent from [awslabs/amazon\-kinesis\-agent](https://github.com/awslabs/amazon-kinesis-agent)\. 

1. Install the agent by navigating to the download directory and running the following command:

   ```
   sudo ./setup --install
   ```

## Configure and Start the Agent<a name="config-start"></a>

**To configure and start the agent**

1. Open and edit the configuration file \(as superuser if using default file access permissions\): `/etc/aws-kinesis/agent.json` 

   In this configuration file, specify the files \( `"filePattern"` \) from which the agent collects data, and the name of the delivery stream \( `"deliveryStream"` \) to which the agent sends data\. Note that the file name is a pattern, and the agent recognizes file rotations\. You can rotate files or create new files no more than once per second\. The agent uses the file creation timestamp to determine which files to track and tail into your delivery stream; creating new files or rotating files more frequently than once per second does not allow the agent to differentiate properly between them\.

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

   Note that the default region is `us-east-1`\. If you are using a different region, add the `firehose.endpoint` setting to the configuration file, specifying the endpoint for your region\. For more information, see [Agent Configuration Settings](#agent-config-settings)\.

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
| <a name="assumeRoleARN"></a>assumeRoleARN |  The ARN of the role to be assumed by the user\. For more information, see [Delegate Access Across AWS Accounts Using IAM Roles](http://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html) in the *IAM User Guide*\.  | 
| <a name="assumeRoleExternalId"></a>assumeRoleExternalId |  An optional identifier that determines who can assume the role\. For more information, see [How to Use an External ID](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user_externalid.html) in the *IAM User Guide*\.  | 
| <a name="awsAccessKeyId"></a>awsAccessKeyId |  AWS access key ID that overrides the default credentials\. This setting takes precedence over all other credential providers\.  | 
| <a name="awsSecretAccessKey"></a>awsSecretAccessKey |  AWS secret key that overrides the default credentials\. This setting takes precedence over all other credential providers\.  | 
| cloudwatch\.emitMetrics |  Enables the agent to emit metrics to CloudWatch if set \(true\)\. Default: true  | 
| cloudwatch\.endpoint |  The regional endpoint for CloudWatch\. Default: `monitoring.us-east-1.amazonaws.com`  | 
| firehose\.endpoint |  The regional endpoint for Kinesis Data Firehose\. Default: `firehose.us-east-1.amazonaws.com`  | 
| kinesis\.endpoint |  The regional endpoint for Kinesis Data Stream\. Default: `kinesis.us-east-1.amazonaws.com`  | 

The following are the flow configuration settings\.


| Configuration Setting | Description | 
| --- | --- | 
| dataProcessingOptions |  The list of processing options applied to each parsed record before it is sent to the delivery stream\. The processing options are performed in the specified order\. For more information, see [Use the Agent to Pre\-process Data](#pre-processing)\.  | 
| deliveryStream |  \[Required\] The name of the delivery stream\.  | 
| filePattern |  \[Required\] A glob for the files that need to be monitored by the agent\. Any file that matches this pattern is picked up by the agent automatically and monitored\. For all files matching this pattern, read permission must be granted to `aws-kinesis-agent-user`\. For the directory containing the files, read and execute permissions must be granted to `aws-kinesis-agent-user`\.  | 
| initialPosition |  The initial position from which the file started to be parsed\. Valid values are `START_OF_FILE` and `END_OF_FILE`\. Default: `END_OF_FILE`  | 
| maxBufferAgeMillis |  The maximum time, in milliseconds, for which the agent buffers data before sending it to the delivery stream\. Value range: 1,000 to 900,000 \(1 second to 15 minutes\) Default: 60,000 \(1 minute\)  | 
| maxBufferSizeBytes |  The maximum size, in bytes, for which the agent buffers data before sending it to the delivery stream\. Value range: 1 to 4,194,304 \(4 MB\) Default: 4,194,304 \(4 MB\)  | 
| maxBufferSizeRecords |  The maximum number of records for which the agent buffers data before sending it to the delivery stream\. Value range: 1 to 500 Default: 500  | 
| minTimeBetweenFilePollsMillis |  The time interval, in milliseconds, at which the agent polls and parses the monitored files for new data\. Value range: 1 or more Default: 100  | 
| multiLineStartPattern |  The pattern for identifying the start of a record\. A record is made of a line that matches the pattern and any following lines that don't match the pattern\. The valid values are regular expressions\. By default, each new line in the log files is parsed as one record\.  | 
| skipHeaderLines |  The number of lines for the agent to skip parsing at the beginning of monitored files\. Value range: 0 or more Default: 0 \(zero\)  | 
| truncatedRecordTerminator |  The string that the agent uses to truncate a parsed record when the record size exceeds the Kinesis Firehose record size limit\. \(1,000 KB\) Default: `'\n'` \(newline\)  | 

## Monitor Multiple File Directories and Write to Multiple Streams<a name="sim-writes"></a>

By specifying multiple flow configuration settings, you can configure the agent to monitor multiple file directories and send data to multiple streams\. In the following configuration example, the agent monitors two file directories and sends data to an Kinesis stream and a Kinesis Firehose delivery stream respectively\. Note that you can specify different endpoints for Kinesis Data Streams and Kinesis Firehose so that your Kinesis stream and Kinesis Firehose delivery stream don’t need to be in the same region\.

```
{
    "cloudwatch.emitMetrics": true,
    "kinesis.endpoint": "kinesis.your-region.amazonaws.com", 
    "firehose.endpoint": "firehose.your-region.amazonaws.com", 
    "flows": [
        {
            "filePattern": "/tmp/app1.log*", 
            "kinesisStream": "yourkinesisstreamname"
        }, 
        {
            "filePattern": "/tmp/app2.log*",
            "deliveryStream": "yourfirehosedeliverystreamname" 
        }
    ] 
}
```

Sample

```
{
  "cloudwatch.emitMetrics": true,
  "kinesis.endpoint": "kinesis.eu-west-1.amazonaws.com",
  "firehose.endpoint": "firehose.eu-west-1.amazonaws.com",

  "flows": [
    {
      "filePattern": "/tmp/sings.log*",
      "kinesisStream": "singers",
    },
    {
      "filePattern": "/tmp/sings.log*",
      "deliveryStream": "singers",
    }
  ]
}
```


For more detailed information about using the agent with Amazon Kinesis Data Streams, see [Writing to Amazon Kinesis Data Streams with Kinesis Agent](http://docs.aws.amazon.com/kinesis/latest/dev/writing-with-agents.html)\.

## Use the Agent to Pre\-process Data<a name="pre-processing"></a>

The agent can pre\-process the records parsed from monitored files before sending them to your delivery stream\. You can enable this feature by adding the `dataProcessingOptions` configuration setting to your file flow\. One or more processing options can be added and they will be performed in the specified order\.

The agent supports the following processing options\. Because the agent is open\-source, you can further develop and extend its processing options\. You can download the agent from [Kinesis Agent](https://github.com/awslabs/amazon-kinesis-agent)\.Processing Options

`SINGLELINE`  
Converts a multi\-line record to a single line record by removing newline characters, leading spaces, and trailing spaces\.  

```
{
    "optionName": "SINGLELINE"
}
```

`CSVTOJSON`  
Converts a record from delimiter separated format to JSON format\.  

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
The following flow configuration converts an Apache Common Log entry to a single line record in JSON format:  

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
The following flow configuration parses multi\-line records whose first line starts with "`[SEQUENCE=`"\. Each record is first converted to a single line record\. Then, values are extracted from the record based on a tab delimiter\. Extracted values are mapped to specified `customFieldNames` values to form a single\-line record in JSON format\.  

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

## Agent CLI Commands<a name="cli-commands"></a>

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
