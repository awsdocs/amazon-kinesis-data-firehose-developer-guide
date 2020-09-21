# Troubleshooting HTTP Endpoints<a name="http_troubleshooting"></a>

This section describes common troubleshooting steps when dealing with Kinesis Data Firehose delivering data to generic HTTP Endpoints destinations and to partner destinations, including Datadog, MongoDB, and New Relic\. For the purposes of this section, all applicable destinations are referred to as HTTP endpoints\. 

**Note**  
The information in this section does not apply to the following destinations: Splunk, ElasticSearch, S3, and Redshift\.

## CloudWatch Logs<a name="cloudwatch_logs"></a>

It is highly recommended that you enable [CloudWatch Logging for Firehose](https://docs.aws.amazon.com/firehose/latest/dev/monitoring-with-cloudwatch-logs.html)\. Logs are only published when there are errors delivering to your destination\.

### Destination Exceptions<a name="destination_exceptions"></a>

**ErrorCode: HttpEndpoint\.DestinationException**

```
{
    "deliveryStreamARN": "arn:aws:firehose:us-east-1:123456789012:deliverystream/ronald-test",
    "destination": "custom.firehose.endpoint.com...",
    "deliveryStreamVersionId": 1,
    "message": "The following response was received from the endpoint destination. 413: {\"requestId\": \"43b8e724-dbac-4510-adb7-ef211c6044b9\", \"timestamp\": 1598556019164, \"errorMessage\": \"Payload too large\"}",
    "errorCode": "HttpEndpoint.DestinationException",
    "processor": "arn:aws:lambda:us-east-1:379522611494:function:httpLambdaProcessing"
}
```

Destination exceptions indicate that Firehose **is** able to establish a connection to your endpoint and make an HTTP request, but **did not** receive a 200 response code\. 2xx responses that are not 200s will also result in a destination exception\. Kinesis Data Firehose logs the response code and a truncated response payload received from the configured endpoint to CloudWatch Logs\. Because Kinesis Data Firehose logs the response code and payload without modification or interpretation, it is up to the endpoint to provide the exact reason why it rejected Kinesis Data Firehose's HTTP delivery request\. The following are the most common troubleshooting recommendations for these exceptions: 
+ **400**: Indicates that you are sending a bad request due to a misconfiguration of your Kinesis Data Firehose\. Make sure that you have the correct [url](https://docs.aws.amazon.com/firehose/latest/APIReference/API_HttpEndpointConfiguration.html#Firehose-Type-HttpEndpointConfiguration-Url), [common attributes](https://docs.aws.amazon.com/firehose/latest/APIReference/API_HttpEndpointRequestConfiguration.html#Firehose-Type-HttpEndpointRequestConfiguration-CommonAttributes), [content encoding](https://docs.aws.amazon.com/firehose/latest/APIReference/API_HttpEndpointRequestConfiguration.html#Firehose-Type-HttpEndpointRequestConfiguration-ContentEncoding), [access key](https://docs.aws.amazon.com/firehose/latest/APIReference/API_HttpEndpointConfiguration.html#Firehose-Type-HttpEndpointConfiguration-AccessKey), and [buffering hints](https://docs.aws.amazon.com/firehose/latest/APIReference/API_HttpEndpointDestinationConfiguration.html#Firehose-Type-HttpEndpointDestinationConfiguration-BufferingHints) for your destination\. See the destination specific documentation on the required configuration\.
+ **401**: Indicates that the access key you configured for your delivery stream is incorrect or missing\.
+ **403**: Indicates that the access key you configured for your delivery stream does not have permissions to deliver data to the configured endpoint\.
+ **413**: Indicates that the request payload that Kinesis Data Firehose sends to the endpoint is too large for the endpoint to handle\. Try [lowering the buffering hint](https://docs.aws.amazon.com/firehose/latest/APIReference/API_HttpEndpointBufferingHints.html#Firehose-Type-HttpEndpointBufferingHints-SizeInMBs) to the recommended size for your destination\. 
+ **429**: Indicates that Kinesis Data Firehose is sending requests at a greater rate than the destination can handle\. Fine tune your buffering hint by increasing your buffering time and/or increasing your buffering size \(but still within the limit of your destination\)\.
+ **5xx**: Indicates that there is a problem with the destination\. The Kinesis Data Firehose service is still working properly\.

**Important**  
Important: While these are the common troubleshooting recommendations, specific endpoints may have different reasons for providing the response codes and the endpoint specific recommendations should be followed first\. 

### Invalid Response<a name="invalid_response"></a>

**ErrorCode: HttpEndpoint\.InvalidResponseFromDestination**

```
{
    "deliveryStreamARN": "arn:aws:firehose:us-east-1:123456789012:deliverystream/ronald-test",
    "destination": "custom.firehose.endpoint.com...",
    "deliveryStreamVersionId": 1,
    "message": "The response received from the specified endpoint is invalid. Contact the owner of the endpoint to resolve the issue. Response for request 2de9e8e9-7296-47b0-bea6-9f17b133d847 is not recognized as valid JSON or has unexpected fields. Raw response received: 200 {\"requestId\": null}",
    "errorCode": "HttpEndpoint.InvalidResponseFromDestination",
    "processor": "arn:aws:lambda:us-east-1:379522611494:function:httpLambdaProcessing"
}
```

Invalid response exceptions indicate that Kinesis Data Firehose received an invalid response from the endpoint destination\. The response must conform to the [response specifications](https://docs.aws.amazon.com/firehose/latest/dev/httpdeliveryrequestresponse.html) or Kinesis Data Firehose will consider the delivery attempt a failure and will redeliver the same data until the configured retry duration is exceeded\. Kinesis Data Firehose treats responses that do not follow the response specifications as failures even if the response has a 200 status\. If you are developing a Kinesis Data Firehose compatible endpoint, follow the response specifications to ensure data is successfully delivered\. 

Below are some of the common types of invalid responses and how to fix them: 
+ **Invalid JSON or Unexpected Fields**: Indicates that the response can not be properly deserialized as JSON or has unexpected fields\. Ensure that the response is not content\-encoded\.
+ **Missing RequestId**: Indicates that the response does not contain a requestId\.
+ **RequestId does not match**: Indicates that the requestId in the response does not match the outgoing requestId\.
+ **Missing Timestamp**: Indicates that the response does not contain a timestamp field\. The timestamp field must be a number and not a string\.
+ **Missing Content\-Type Header**: Indicates that the response does not contain a “content\-type: application/json” header\. No other content\-type is accepted\.

**Important**  
Important: Kinesis Data Firehose can only deliver data to endpoints that follow the Firehose request and [response specifications](https://docs.aws.amazon.com/firehose/latest/dev/httpdeliveryrequestresponse.html)\. If you are configuring your destination to a third party service, ensure that you are using the correct Kinesis Data Firehose compatible endpoint which will likely be different than the public ingestion endpoint\. For example Datadog’s Kinesis Data Firehose endpoint is [https://aws\-kinesis\-http\-intake\.logs\.datadoghq\.com/](http://aws-kinesis-http-intake.logs.datadoghq.com/) while its public endpoint is [https://api\.datadoghq\.com/](https://api.datadoghq.com/)\.

### Other Common Errors<a name="others"></a>

Additional error codes and definitions are listed below\.
+ **Error Code: HttpEndpoint\.RequestTimeout** \- Indicates that the endpoint took longer than 3 minutes to respond\. If you are the owner of the destination, decrease the response time of the destination endpoint\. If you are not the owner of the destination, contact the owner and ask if anything can be done to lower the response time \(i\.e\. decrease the buffering hint so there is less data being processed per request\)\. 
+ **Error Code: HttpEndpoint\.ResponseTooLarge** \- Indicates that the response is too large\. The response must be less than 1 MiB including headers\.
+ **Error Code: HttpEndpoint\.ConnectionFailed** \- Indicates a connection could not be established with the configured endpoint\. This could be due to a typo in the configured url, the endpoint not being accessible to Kinesis Data Firehose, or the endpoint taking too long to respond to the connection request\.
+ **Error Code: HttpEndpoint\.ConnectionReset** \- Indicates a connection was made but reset or prematurely closed by the endpoint\.
+ **Error Code: HttpEndpoint\.SSLHandshakeFailure** \- Indicates an SSL handshake could not be successfully completed with the configured endpoint\.