# Custom Prefixes for Amazon S3 Objects<a name="s3-prefixes"></a>

If your destination is Amazon S3, Amazon Elasticsearch Service, or Splunk, you can configure the Amazon S3 object keys used for delivering data from Kinesis Data Firehose\. To do this, you specify expressions that Kinesis Data Firehose evaluates at delivery time\. The final object keys have the format `<evaluated prefix><suffix>`, where the suffix has the format `<delivery stream name>-<delivery stream version>-<year>-<month>-<day>-<hour>-<minute>-<second>-<uuid><file extension>`\. You can't change the suffix field\.

You can use expressions of the following forms in your custom prefix: `!{namespace:value}`, where `namespace` can be either `firehose` or `timestamp`, as explained in the following sections\.

If a prefix ends with a slash, it appears as a folder in the Amazon S3 bucket\. For more information, see [Amazon S3 Object Name Format](https://docs.aws.amazon.com/firehose/latest/dev/basic-deliver.html#s3-object-name) in the *Amazon Kinesis Data Firehose Developer Guide*\.

## The `timestamp` namespace<a name="timestamp-namespace"></a>

Valid values for this namespace are strings that are valid [Java DateTimeFormatter](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html) strings\. As an example, in the year 2018, the expression `!{timestamp:yyyy}` evaluates to `2018`\. 

When evaluating timestamps, Kinesis Data Firehose uses the approximate arrival timestamp of the oldest record that's contained in the Amazon S3 object being written\. 

The timestamp is always in UTC\. 

If you use the `timestamp` namespace more than once in the same prefix expression, every instance evaluates to the same instant in time\.

## The `firehose` namespace<a name="firehose-namespace"></a>

There are two values that you can use with this namespace: `error-output-type` and `random-string`\. The following table explains how to use them\.


**The `firehose` namespace values**  

| Conversion | Description | Example input | Example output | Notes | 
| --- | --- | --- | --- | --- | 
| error\-output\-type | Evaluates to one of the following strings, depending on the configuration of your delivery stream, and the reason of failure: \{processing\-failed, elasticsearch\-failed, splunk\-failed, format\-conversion\-failed, http\-endpoint\-failed\}\.If you use it more than once in the same expression, every instance evaluates to the same error string\.\. | myPrefix/result=\!\{firehose:error\-output\-type\}/\!\{timestamp:yyyy/MM/dd\} | myPrefix/result=processing\-failed/2018/08/03 | The error\-output\-type value can only be used in the ErrorOutputPrefix field\. | 
| random\-string |  Evaluates to a random string of 11 characters\. If you use it more than once in the same expression, every instance evaluates to a new random string\.  | myPrefix/\!\{firehose:random\-string\}/ | myPrefix/046b6c7f\-0b/ | You can use it with both prefix types\.You can place it at the beginning of the format string to get a randomized prefix, which is sometimes necessary for attaining extremely high throughput with Amazon S3\. | 

## Semantic rules<a name="prefix-rules"></a>

The following rules apply to `Prefix` and `ErrorOutputPrefix` expressions\.
+ For the `timestamp` namespace, any character that isn't in single quotes is evaluated\. In other words, any string escaped with single quotes in the value field is taken literally\.
+ If you specify a prefix that doesn't contain a timestamp namespace expression, Kinesis Data Firehose appends the expression `!{timestamp:yyyy/MM/dd/HH/}`to the value in the `Prefix` field\.
+ The sequence `!{` can only appear in `!{namespace:value}` expressions\.
+ `ErrorOutputPrefix` can be null only if `Prefix` contains no expressions\. In this case, `Prefix` evaluates to `<specified-prefix>YYYY/MM/DDD/HH/` and `ErrorOutputPrefix` evaluates to `<specified-prefix><error-output-type>YYYY/MM/DDD/HH/`\. `DDD` represents the day of the year\.
+ If you specify an expression for `ErrorOutputPrefix`, you must include at least one instance of `!{firehose:error-output-type}`\.
+ `Prefix` can't contain `!{firehose:error-output-type}`\.
+ Neither `Prefix` nor `ErrorOutputPrefix` can be greater than 512 characters after they're evaluated\.
+ If the destination is Amazon Redshift, `Prefix` must not contain expressions and `ErrorOutputPrefix` must be null\.
+ When the destination is Amazon Elasticsearch Service or Splunk, and no `ErrorOutputPrefix` is specified, Kinesis Data Firehose uses the `Prefix` field for failed records\. 
+ When the destination is Amazon S3, the `Prefix` and `ErrorOutputPrefix` in the Amazon S3 destination configuration are used for successful records and failed records, respectively\. If you use the AWS CLI or the API, you can use `ExtendedS3DestinationConfiguration` to specify an Amazon S3 *backup* configuration with its own `Prefix` and `ErrorOutputPrefix`\.
+ When you use the AWS Management Console and set the destination to Amazon S3, Kinesis Data Firehose uses the `Prefix` and `ErrorOutputPrefix` in the destination configuration for successful records and failed records, respectively\. If you specify a prefix but no error prefix, Kinesis Data Firehose automatically sets the error prefix to `!{firehose:error-output-type}/`\.
+ When you use `ExtendedS3DestinationConfiguration` with the AWS CLI, the API, or AWS CloudFormation, if you specify a `S3BackupConfiguration`, Kinesis Data Firehose doesn't provide a default `ErrorOutputPrefix`\.

## Example prefixes<a name="s3-prefix-examples"></a>


**`Prefix` and `ErrorOutputPrefix` examples**  

| Input | Evaluated prefix \(at 10:30 AM UTC on Aug 27, 2018\) | 
| --- | --- | 
|  `Prefix`: Unspecified `ErrorOutputPrefix`: `myFirehoseFailures/!{firehose:error-output-type}/`  |  `Prefix`: `2018/08/27/10` `ErrorOutputPrefix`: `myFirehoseFailures/processing-failed/`  | 
|  `Prefix`: `!{timestamp:yyyy/MM/dd}` `ErrorOutputPrefix`: Unspecified  | Invalid input: ErrorOutputPrefix can't be null when Prefix contains expressions | 
|  `Prefix`: `myFirehose/DeliveredYear=!{timestamp:yyyy}/anyMonth/rand=!{firehose:random-string}` `ErrorOutputPrefix`: `myFirehoseFailures/!{firehose:error-output-type}/!{timestamp:yyyy}/anyMonth/!{timestamp:dd}`  |  `Prefix`: `myFirehose/DeliveredYear=2018/anyMonth/rand=5abf82daaa5` `ErrorOutputPrefix`: `myFirehoseFailures/processing-failed/2018/anyMonth/10`  | 
| `Prefix`: `myPrefix/year=!{timestamp:yyyy}/month=!{timestamp:MM}/day=!{timestamp:dd}/hour=!{timestamp:HH}/` `ErrorOutputPrefix`: `myErrorPrefix/year=!{timestamp:yyyy}/month=!{timestamp:MM}/day=!{timestamp:dd}/hour=!{timestamp:HH}/!{firehose:error-output-type}`  | `Prefix`: `myPrefix/year=2018/month=07/day=06/hour=23/` `ErrorOutputPrefix`: `myErrorPrefix/year=2018/month=07/day=06/hour=23/processing-failed` | 
|  `Prefix`: `myFirehosePrefix` `ErrorOutputPrefix`: Unspecified  |  `Prefix`: `myFirehosePrefix/2018/08/27/` `ErrorOutputPrefix`: `myFirehosePrefix/processing-failed/2018/08/27/`  | 