# Writing to Kinesis Data Firehose Using AWS IoT<a name="writing-with-iot"></a>

You can configure AWS IoT to send information to a Amazon Kinesis Data Firehose delivery stream by adding an action\.

**To create an action that sends events to an existing Kinesis Data Firehose delivery stream**

1. When creating a rule in the AWS IoT console, on the **Create a rule** page, under **Set one or more actions**, choose **Add action**\.

1. Choose **Send messages to an Amazon Kinesis Firehose stream**\.

1. Choose **Configure action**\.

1. For **Stream name**, choose an existing Kinesis Data Firehose delivery stream\. 

1. For **Separator**, choose a separator character to be inserted between records\.

1. For **IAM role name**, choose an existing IAM role or choose **Create a new role**\.

1. Choose **Add action**\.

For more information about creating AWS IoT rules, see [AWS IoT Rule Tutorials](https://docs.aws.amazon.com/iot/latest/developerguide/iot-rules-tutorial.html)\.