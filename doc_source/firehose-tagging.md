# Tagging Your Delivery Streams in Amazon Kinesis Data Firehose<a name="firehose-tagging"></a>

You can assign your own metadata to delivery streams that you create in Amazon Kinesis Data Firehose in the form of *tags*\. A tag is a key\-value pair that you define for a stream\. Using tags is a simple yet powerful way to manage AWS resources and organize data, including billing data\. 

**Topics**
+ [Tag Basics](#firehose-tagging-basics)
+ [Tracking Costs Using Tagging](#firehose-tagging-billing)
+ [Tag Restrictions](#firehose-tagging-restrictions)
+ [Tagging Delivery Streams Using the Amazon Kinesis Data Firehose API](#firehose-tagging-howto)

## Tag Basics<a name="firehose-tagging-basics"></a>

You can use the Amazon Kinesis Data Firehose API to complete the following tasks:
+ Add tags to a delivery stream\.
+ List the tags for your delivery streams\.
+ Remove tags from a delivery stream\.

You can use tags to categorize your Kinesis Data Firehose delivery streams\. For example, you can categorize delivery streams by purpose, owner, or environment\. Because you define the key and value for each tag, you can create a custom set of categories to meet your specific needs\. For example, you might define a set of tags that helps you track delivery streams by owner and associated application\. 

The following are several examples of tags:
+ `Project: Project name`
+ `Owner: Name`
+ `Purpose: Load testing` 
+ `Application: Application name`
+ `Environment: Production` 

## Tracking Costs Using Tagging<a name="firehose-tagging-billing"></a>

You can use tags to categorize and track your AWS costs\. When you apply tags to your AWS resources, including Kinesis Data Firehose delivery streams, your AWS cost allocation report includes usage and costs aggregated by tags\. You can organize your costs across multiple services by applying tags that represent business categories \(such as cost centers, application names, or owners\)\. For more information, see [Use Cost Allocation Tags for Custom Billing Reports](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html) in the *AWS Billing and Cost Management User Guide*\.

## Tag Restrictions<a name="firehose-tagging-restrictions"></a>

The following restrictions apply to tags in Kinesis Data Firehose\.

**Basic restrictions**
+ The maximum number of tags per resource \(stream\) is 50\.
+ Tag keys and values are case\-sensitive\.
+ You can't change or edit tags for a deleted stream\.

**Tag key restrictions**
+ Each tag key must be unique\. If you add a tag with a key that's already in use, your new tag overwrites the existing key\-value pair\. 
+ You can't start a tag key with `aws:` because this prefix is reserved for use by AWS\. AWS creates tags that begin with this prefix on your behalf, but you can't edit or delete them\.
+ Tag keys must be between 1 and 128 Unicode characters in length\.
+ Tag keys must consist of the following characters: Unicode letters, digits, white space, and the following special characters: `_ . / = + - @`\.

**Tag value restrictions**
+ Tag values must be between 0 and 255 Unicode characters in length\.
+ Tag values can be blank\. Otherwise, they must consist of the following characters: Unicode letters, digits, white space, and any of the following special characters: `_ . / = + - @`\.

## Tagging Delivery Streams Using the Amazon Kinesis Data Firehose API<a name="firehose-tagging-howto"></a>

You can specify tags when you invoke [CreateDeliveryStream](https://docs.aws.amazon.com/firehose/latest/APIReference/API_CreateDeliveryStream.html) to create a new delivery stream\. For existing delivery streams, you can add, list, and remove tags using the following three operations: 
+ [TagDeliveryStream](https://docs.aws.amazon.com/firehose/latest/APIReference/API_TagDeliveryStream.html)
+ [ListTagsForDeliveryStream](https://docs.aws.amazon.com/firehose/latest/APIReference/API_ListTagsForDeliveryStream.html)
+ [UntagDeliveryStream](https://docs.aws.amazon.com/firehose/latest/APIReference/API_UntagDeliveryStream.html)