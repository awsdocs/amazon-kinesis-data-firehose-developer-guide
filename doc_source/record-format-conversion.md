# Converting Your Input Record Format in Kinesis Data Firehose<a name="record-format-conversion"></a>

Amazon Kinesis Data Firehose can convert the format of your input data from JSON to [Apache Parquet](https://parquet.apache.org/) or [Apache ORC](https://orc.apache.org/) before storing the data in Amazon S3\. Parquet and ORC are columnar data formats that save space and enable faster queries compared to row\-oriented formats like JSON\. If you want to convert an input format other than JSON, such as comma\-separated values \(CSV\) or structured text, you can use AWS Lambda to transform it to JSON first\. For more information, see [Amazon Kinesis Data Firehose Data Transformation](data-transformation.md)\.

**Topics**
+ [Record Format Conversion Requirements](#record-format-conversion-concepts)
+ [Choosing the JSON Deserializer](#record-format-conversion-deserializers)
+ [Choosing the Serializer](#record-format-conversion-serializers)
+ [Converting Input Record Format \(Console\)](#record-format-conversion-using-console)
+ [Converting Input Record Format \(API\)](#record-format-conversion-using-api)
+ [Record Format Conversion Error Handling](#record-format-conversion-error-handling)
+ [Record Format Conversion Example](#record-format-conversion-example)

## Record Format Conversion Requirements<a name="record-format-conversion-concepts"></a>

Kinesis Data Firehose requires the following three elements to convert the format of your record data: 
+ **A deserializer to read the JSON of your input data** – You can choose one of two types of deserializers: [Apache Hive JSON SerDe](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-JSON) or [OpenX JSON SerDe](https://github.com/rcongiu/Hive-JSON-Serde)\.
**Note**  
When combining multiple JSON documents into the same record, make sure that your input is still presented in the supported JSON format\. An array of JSON documents is NOT a valid input\.   
For example, this is the correct input: `{"a":1}{"a":2}`  
And this is the INCORRECT input: `[{"a":1}, {"a":2}]`
+ **A schema to determine how to interpret that data** – Use [AWS Glue](https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html) to create a schema in the AWS Glue Data Catalog\. Kinesis Data Firehose then references that schema and uses it to interpret your input data\. You can use the same schema to configure both Kinesis Data Firehose and your analytics software\. For more information, see [Populating the AWS Glue Data Catalog](https://docs.aws.amazon.com/glue/latest/dg/populate-data-catalog.html) in the *AWS Glue Developer Guide*\.
+ **A serializer to convert the data to the target columnar storage format \(Parquet or ORC\)** – You can choose one of two types of serializers: [ORC SerDe](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC) or [Parquet SerDe](https://cwiki.apache.org/confluence/display/Hive/Parquet)\.

**Important**  
If you enable record format conversion, you can't set your Kinesis Data Firehose destination to be Amazon Elasticsearch Service \(Amazon ES\), Amazon Redshift, or Splunk\. With format conversion enabled, Amazon S3 is the only destination that you can use for your Kinesis Data Firehose delivery stream\.

You can convert the format of your data even if you aggregate your records before sending them to Kinesis Data Firehose\.

## Choosing the JSON Deserializer<a name="record-format-conversion-deserializers"></a>

Choose the [OpenX JSON SerDe](https://github.com/rcongiu/Hive-JSON-Serde) if your input JSON contains time stamps in the following formats:
+  yyyy\-MM\-dd'T'HH:mm:ss\[\.S\]'Z', where the fraction can have up to 9 digits – For example, `2017-02-07T15:13:01.39256Z`\.
+  yyyy\-\[M\]M\-\[d\]d HH:mm:ss\[\.S\], where the fraction can have up to 9 digits – For example, `2017-02-07 15:13:01.14`\.
+  Epoch seconds – For example, `1518033528`\.
+  Epoch milliseconds – For example, `1518033528123`\.
+  Floating point epoch seconds – For example, `1518033528.123`\.

The OpenX JSON SerDe can convert periods \(`.`\) to underscores \(`_`\)\. It can also convert JSON keys to lowercase before deserializing them\. For more information about the options that are available with this deserializer through Kinesis Data Firehose, see [OpenXJsonSerDe](https://docs.aws.amazon.com/firehose/latest/APIReference/API_OpenXJsonSerDe.html)\.

If you're not sure which deserializer to choose, use the OpenX JSON SerDe, unless you have time stamps that it doesn't support\.

If you have time stamps in formats other than those listed previously, use the [Apache Hive JSON SerDe](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-JSON)\. When you choose this deserializer, you can specify the time stamp formats to use\. To do this, follow the pattern syntax of the Joda\-Time `DateTimeFormat` format strings\. For more information, see [Class DateTimeFormat](https://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)\. 

You can also use the special value `millis` to parse time stamps in epoch milliseconds\. If you don't specify a format, Kinesis Data Firehose uses `java.sql.Timestamp::valueOf` by default\.

The Hive JSON SerDe doesn't allow the following:
+ Periods \(`.`\) in column names\.
+ Fields whose type is `uniontype`\.
+ Fields that have numerical types in the schema, but that are strings in the JSON\. For example, if the schema is \(an int\), and the JSON is `{"a":"123"}`, the Hive SerDe gives an error\.

The Hive SerDe doesn't convert nested JSON into strings\. For example, if you have `{"a":{"inner":1}}`, it doesn't treat `{"inner":1}` as a string\.

## Choosing the Serializer<a name="record-format-conversion-serializers"></a>

The serializer that you choose depends on your business needs\. To learn more about the two serializer options, see [ORC SerDe](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC) and [Parquet SerDe](https://cwiki.apache.org/confluence/display/Hive/Parquet)\.

## Converting Input Record Format \(Console\)<a name="record-format-conversion-using-console"></a>

You can enable data format conversion on the console when you create or update a Kinesis delivery stream\. With data format conversion enabled, Amazon S3 is the only destination that you can configure for the delivery stream\. Also, Amazon S3 compression gets disabled when you enable format conversion\. However, Snappy compression happens automatically as part of the conversion process\. The framing format for Snappy that Kinesis Data Firehose uses in this case is compatible with Hadoop\. This means that you can use the results of the Snappy compression and run queries on this data in Athena\. For the Snappy framing format that Hadoop relies on, see [BlockCompressorStream\.java](https://github.com/apache/hadoop/blob/f67237cbe7bc48a1b9088e990800b37529f1db2a/hadoop-common-project/hadoop-common/src/main/java/org/apache/hadoop/io/compress/BlockCompressorStream.java)\.

**To enable data format conversion for a data delivery stream**

1. Sign in to the AWS Management Console, and open the Kinesis Data Firehose console at [https://console\.aws\.amazon\.com/firehose/](https://console.aws.amazon.com/firehose/)\.

1. Choose a Kinesis Data Firehose delivery stream to update, or create a new delivery stream by following the steps in [Creating an Amazon Kinesis Data Firehose Delivery Stream](basic-create.md)\.

1. Under **Convert record format**, set **Record format conversion** to **Enabled**\.

1. Choose the output format that you want\. For more information about the two options, see [Apache Parquet](https://parquet.apache.org/) and [Apache ORC](https://orc.apache.org/)\.

1. Choose an AWS Glue table to specify a schema for your source records\. Set the Region, database, table, and table version\.

## Converting Input Record Format \(API\)<a name="record-format-conversion-using-api"></a>

If you want Kinesis Data Firehose to convert the format of your input data from JSON to Parquet or ORC, specify the optional [DataFormatConversionConfiguration](https://docs.aws.amazon.com/firehose/latest/APIReference/API_DataFormatConversionConfiguration.html) element in [ExtendedS3DestinationConfiguration](https://docs.aws.amazon.com/firehose/latest/APIReference/API_ExtendedS3DestinationConfiguration.html) or in [ExtendedS3DestinationUpdate](https://docs.aws.amazon.com/firehose/latest/APIReference/API_ExtendedS3DestinationUpdate.html)\. If you specify [DataFormatConversionConfiguration](https://docs.aws.amazon.com/firehose/latest/APIReference/API_DataFormatConversionConfiguration.html), the following restrictions apply:
+ In [BufferingHints](https://docs.aws.amazon.com/firehose/latest/APIReference/API_BufferingHints.html), you can't set `SizeInMBs` to a value less than 64 if you enable record format conversion\. Also, when format conversion isn't enabled, the default value is 5\. The value becomes 128 when you enable it\.
+ You must set `CompressionFormat` in [ExtendedS3DestinationConfiguration](https://docs.aws.amazon.com/firehose/latest/APIReference/API_ExtendedS3DestinationConfiguration.html) or in [ExtendedS3DestinationUpdate](https://docs.aws.amazon.com/firehose/latest/APIReference/API_ExtendedS3DestinationUpdate.html) to `UNCOMPRESSED`\. The default value for `CompressionFormat` is `UNCOMPRESSED`\. Therefore, you can also leave it unspecified in [ExtendedS3DestinationConfiguration](https://docs.aws.amazon.com/firehose/latest/APIReference/API_ExtendedS3DestinationConfiguration.html)\. The data still gets compressed as part of the serialization process, using Snappy compression by default\. The framing format for Snappy that Kinesis Data Firehose uses in this case is compatible with Hadoop\. This means that you can use the results of the Snappy compression and run queries on this data in Athena\. For the Snappy framing format that Hadoop relies on, see [BlockCompressorStream\.java](https://github.com/apache/hadoop/blob/f67237cbe7bc48a1b9088e990800b37529f1db2a/hadoop-common-project/hadoop-common/src/main/java/org/apache/hadoop/io/compress/BlockCompressorStream.java)\. When you configure the serializer, you can choose other types of compression\.

## Record Format Conversion Error Handling<a name="record-format-conversion-error-handling"></a>

When Kinesis Data Firehose can't parse or deserialize a record \(for example, when the data doesn't match the schema\), it writes it to Amazon S3 with an error prefix\. If this write fails, Kinesis Data Firehose retries it forever, blocking further delivery\. For each failed record, Kinesis Data Firehose writes a JSON document with the following schema:

```
{
  "attemptsMade": long,
  "arrivalTimestamp": long,
  "lastErrorCode": string,
  "lastErrorMessage": string,
  "attemptEndingTimestamp": long,
  "rawData": string,
  "sequenceNumber": string,
  "subSequenceNumber": long,
  "dataCatalogTable": {
    "catalogId": string,
    "databaseName": string,
    "tableName": string,
    "region": string,
    "versionId": string,
    "catalogArn": string
  }
}
```

## Record Format Conversion Example<a name="record-format-conversion-example"></a>

For an example of how to set up record format conversion with AWS CloudFormation, see [AWS::KinesisFirehose::DeliveryStream](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-kinesisfirehose-deliverystream.html#aws-resource-kinesisfirehose-deliverystream--examples)\.