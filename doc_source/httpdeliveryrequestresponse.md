# Appendix \- HTTP Endpoint Delivery Request and Response Specifications<a name="httpdeliveryrequestresponse"></a>

For Kinesis Data Firehose to successfully deliver data to custom HTTP endpoints, these endpoints must accept requests and send responses using certain Kinesis Data Firehose request and response formats\. This section describes the format specifications of the HTTP requests that the Kinesis Data Firehose service sends to custom HTTP endpoints, as well as the format specifications of the HTTP responses that the Kinesis Data Firehose service expects\. HTTP endpoints have 3 minutes to respond to a request before Kinesis Data Firehose times out that request\. Kinesis Data Firehose treats responses that do not adhere to the proper format as delivery failures\.

**Topics**
+ [Request Format](#requestformat)
+ [Response Format](#responseformat)
+ [Examples](#examples)

## Request Format<a name="requestformat"></a>

**Path and URL Parameters**  
These are configured directly by you as part of a single URL field\. Kinesis Data Firehose sends them as configured without modification\. Only https destinations are supported\. URL restrictions are applied during delivery\-stream configuration\.

**HTTP Headers \- X\-Amz\-Firehose\-Protocol\-Version**  
This header is used to indicate the version of the request/response formats\. Currently the only version is 1\.0\.

**HTTP Headers \- X\-Amz\-Firehose\-Request\-Id**  
The value of this header is an opaque GUID that can be used for debugging and deduplication purposes\. Endpoint implementations should log the value of this header if possible, for both successful and unsuccessful requests\. The request ID is kept the same between multiple attempts of the same request\.

**HTTP Headers \- Content\-Type**  
The value of the Content\-Type header is always `application/json`\.

**HTTP Headers \- Content\-Encoding**  
A Kinesis Data Firehose delivery stream can be configured to use GZIP to compress the body when sending requests\. When this compression is enabled, the value of the Content\-Encoding header is set to gzip, as per standard practice\. If compression is not enabled, the Content\-Encoding header is absent altogether\.

**HTTP Headers \- Content\-Length**  
This is used in the standard way\.

**HTTP Headers \- X\-Amz\-Firehose\-Source\-Arn:**  
The ARN of the Kinesis Data Firehose delivery stream represented in ASCII string format\. The ARN encodes region, AWS account ID and the stream name\. For example, `arn:aws:firehose:us-east-1:123456789:deliverystream/testStream`\. 

**HTTP Headers \- X\-Amz\-Firehose\-Access\-Key**  
This header carries an API key or other credentials\. You have the ability to create or update the API\-key \(aka authorization token\) when creating or updating your delivery\-stream\. Kinesis Data Firehose restricts the size of the access key to 4096 bytes\. Kinesis Data Firehose does not attempt to interpret this key in any way\. The configured key is copied verbatim into the value of this header\.   
The contents can be arbitrary and can potentially represent a JWT token or an ACCESS\_KEY\. If an endpoint requires multi\-field credentials \(for example, username and password\), the values of all of the fields should be stored together within a single access\-key in a format that the endpoint understands \(JSON or CSV\)\. This field can be base\-64 encoded if the original contents are binary\. Kinesis Data Firehose does not modifiy and/or encode the configured value and uses the contents as is\.

**HTTP Headers \- X\-Amz\-Firehose\-Common\-Attributes**  
This header carries the common attributes \(metadata\) that pertain to the entire request, and/or to all records within the request\. These are configured directly by you when creating a delivery stream\. The value of this attribute is encoded as a JSON object with the following schema:   

```
"$schema": http://json-schema.org/draft-07/schema#

properties:
  commonAttributes:
    type: object
    minProperties: 0
    maxProperties: 50
    patternProperties:
      "^.{1,256}$":
        type: string
        minLength: 0
        maxLength: 1024
```
Here's an example:  

```
"commonAttributes": {
    "deployment -context": "pre-prod-gamma",
    "device-types": ""
  }
```

**Body \- Max Size**  
The maximum body size is configured by you, and can be up to a maximum of 64 MiB, before compression\.

**Body \- Schema**  
The body carries a single JSON document with the following JSON Schema \(written in YAML\):  

```
"$schema": http://json-schema.org/draft-07/schema#

title: FirehoseCustomHttpsEndpointRequest
description: >
  The request body that the Firehose service sends to
  custom HTTPS endpoints.
type: object
properties:
  requestId:
    description: >
      Same as the value in the X-Amz-Firehose-Request-Id header,
      duplicated here for convenience.
    type: string
  timestamp:
    description: >
      The timestamp (milliseconds since epoch) at which the Firehose
      server generated this request.
    type: integer
  records:
    description: >
      The actual records of the Delivery Stream, carrying 
      the customer data.
    type: array
    minItems: 1
    maxItems: 10000
    items:
      type: object
      properties:
        data:
          description: >
            The data of this record, in Base64. Note that empty
            records are permitted in Firehose. The maximum allowed
            size of the data, before Base64 encoding, is 1024000
            bytes; the maximum length of this field is therefore
            1365336 chars.
          type: string
          minLength: 0
          maxLength: 1365336

required:
  - requestId
  - records
```
Here's an example:  

```
{
  "requestId": "ed4acda5-034f-9f42-bba1-f29aea6d7d8f",
  "timestamp": 1578090901599
  "records": [
    {
      "data": "aGVsbG8="
    },
    {
      "data": "aGVsbG8gd29ybGQ="
    }
  ]
}
```

## Response Format<a name="responseformat"></a>

**Default Behavior on Error**  
If a response fails to conform to the requirements below, the Kinesis Firehose server treats it as though it had a 500 status code with no body\.

**Status Code**  
The HTTP status code MUST be in the 2XX, 4XX or 5XX range\.  
The Kinesis Data Firehose server does NOT follow redirects \(3XX status codes\)\. Only response code 200 is considered as a successful delivery of the records to HTTP/EP\. Response code 413 \(size exceeded\) is considered as a permanent failure and the record batch is not sent to error bucket if configured\. All other response codes are considered as retriable errors and are subjected to back\-off retry algorithm explained later\. 

**Headers \- Content Type**  
The only acceptable content type is application/json\.

**HTTP Headers \- Content\-Encoding**  
Content\-Encoding MUST NOT be used\. The body MUST be uncompressed\.

**HTTP Headers \- Content\-Length**  
The Content\-Length header MUST be present if the response has a body\.

**Body \- Max Size**  
The response body must be 1 MiB or less in size\.  

```
"$schema": http://json-schema.org/draft-07/schema#

title: FirehoseCustomHttpsEndpointResponse

description: >
  The response body that the Firehose service sends to
  custom HTTPS endpoints.
type: object
properties:
  requestId:
    description: >
      Must match the requestId in the request.
    type: string
  
  timestamp:
    description: >
      The timestamp (milliseconds since epoch) at which the
      server processed this request.
    type: integer
   
  errorMessage:
    description: >
      For failed requests, a message explaining the failure.
      If a request fails after exhausting all retries, the last 
      Instance of the error message is copied to error output
      S3 bucket if configured.
    type: string
    minLength: 0
    maxLength: 8192
required:
  - requestId
  - timestamp
```
Here's an example:  

```
Failure Case (HTTP Response Code 4xx or 5xx)
{
  "requestId": "ed4acda5-034f-9f42-bba1-f29aea6d7d8f",
  "timestamp": "1578090903599",
  "errorMessage": "Unable to deliver records due to unknown error."
}
Success case (HTTP Response Code 200)
{
  "requestId": "ed4acda5-034f-9f42-bba1-f29aea6d7d8f",
  "timestamp": 1578090903599
}
```

**Error Response Handling**  
In all error cases the Kinesis Data Firehose server reattempts delivery of the same batch of records using an exponential back\-off algorithm\. The retries are backed off using an initial back\-off time \(1 second\) with a jitter factor of \(15%\) and each subsequent retry is backed off using the formula \(initial\-backoff\-time \* \(multiplier\(2\) ^ retry\_count\)\) with added jitter\. The backoff time is capped by a maximum interval of 2 minutes\. For example on the ‘n’\-th retry the back off time is = MAX\(120sec, \(1 \* \(2^n\)\) \* random\(0\.85, 1,15\)\.  
The parameters specified in the previous equation are subject to change\. Refer to the AWS Firehose documentation for exact initial back off time, max backoff time, multiplier and jitter percentages used in exponential back off algorithm\.  
In each subsequent retry attempt the access key and/or destination to which records are delivered might change based on updated configuration of the delivery stream\. Kinesis Data Firehose service uses the same request\-id across retries in a best\-effort manner\. This last feature can be used for deduplication purpose by the HTTP end point server\. If the request is still not delivered after the maximum time allowed \(based on delivery stream configuration\) the batch of records can optionally be delivered to an error bucket based on stream configuration\.

## Examples<a name="examples"></a>

 Example of a CWLog sourced request:

```
{
  "requestId": "ed4acda5-034f-9f42-bba1-f29aea6d7d8f",
  "timestamp": 1578090901599,
  "records": [
   {
    "data": {
      "messageType": "DATA_MESSAGE",
      "owner": "123456789012",
      "logGroup": "log_group_name",
      "logStream": "log_stream_name",
      "subscriptionFilters": [
        "subscription_filter_name"
      ],
      "logEvents": [
        {
          "id": "0123456789012345678901234567890123456789012345",
          "timestamp": 1510109208016,
          "message": "log message 1"
        },
        {
          "id": "0123456789012345678901234567890123456789012345",
          "timestamp": 1510109208017,
          "message": "log message 2"
        }
      ]
    }
   }
  ]
}
```