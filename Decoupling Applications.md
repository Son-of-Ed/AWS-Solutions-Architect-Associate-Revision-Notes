# Decoupling Applications
Services need to communicate with each other. The are two types of app communication:
1. Synchronous (app to app) communication
2. Asynchronous (app to queue to app) communication
Sudden spikes in traffic can cause issues with synchronous traffic as apps can get overloaded without any throttling.
Can decouple apps from each other using:
	- SQS queue model
	- SNS pub/sub model
	- Kinesis real time data streaming model

## SQS
Fully managed service used to decouple applications with unlimited throughput + unlimited messages within queue.
Helps to decouple by putting send + receive queues between applications which can then be used to scale your frontend and backend applications according to CloudWatch metrics/alarms.
It also acts as a buffer between different components of your application by storing messages within a queue before pushing them to a destination in order to not overwhelm the destination component (especially useful when dealing with large spikes in traffic).
Multiple publishers/producers can send messages to an SQS queue which can then send messages to many consumers (many to one to many).
Messages exist on queues for up to 4 days (can increase up to 14) and are deleted from the queue after they are read.
Each consumer polls the queue and can receive up to 10 messages at a time. After a consumer has processed the message, it delete the message from the queue so other consumers do not attempt to process it.
Consumers poll/receive messages in parallel. To guarantee message delivery, SQS uses "at-least-once" delivery which means that every message will be processed but it may happen multiple times. This means you applications must be idempotent to prevent multiple copies of the same message affecting the function of your app.
Sometimes there can be duplicate messages or messages that are out of order.
#### Security
Use IAM controls to regulate who can access the SQS API (push/pull).
Enable SQS access policies (similar to S3 access policies) to control which AWS subscriptions/services can write to your queues.
Messages are encrypted in-flight using HTTPS and at rest using KMS or client-side encryption if the client wants to handle encryption/decryption.
#### Message Visibility Timeout
Timer set so that when a consumer polls a message, that message is then made invisible to other consumers so that they cannot poll it while the first consumer is processing the message. After the time has elapsed, if the message has not been deleted from the queue by the first consumer, other consumers can then poll the message.
If the message has not been deleted from the queue by the end of the timer, the message will get picked up by a second consumer and will be processed a second time.
To get more time before a message is made visible again, a consumer can call the ChangeMessageVisibility API.
If the visibility timeout is too long, consumers can crash and re-processing messages can be time-consuming.
#### Long Polling
When a consumer is polling a queue, it can wait and poll the queue for a longer time if there are currently no messages in the queue.
This reduces the number of polls that need to be made to a queue which can improve the latency of your application as consumers are watching for new messages to come in.
Can be set to be within 1s and 20s, the longer the better.
Long polling can be enabled at the queue for all consumers or at the consumer.
#### FIFO Queues
First in, first out - maintains the order of messages and so that they are processed in the same sequence as when the queue received them.
An alternative to the standard queues which work on the principle of at-least-once delivery.
Comes with limited throughput when compared to standard queues of 300msg/s without batching and 3000msg/s with batching.
Has exactly-once capability so duplicate messages are removed and never processed more than once.
Can use Group IDs in your messages to make sure that like data is always grouped together if you want it to all go to one consumer i.e. you can have one Group ID assigned per consumer.

## Kinesis
Takes data streams, e.g. metrics, IoT, click streams, and collects them with Kinesis Streams, analyse them with Kinesis Analytics, and output findings to S3 buckets or Redshift using Kinesis Firehose.
#### Data Streams
Captures, processes, and stores data streams.
Provision a stream consisting of N shards which must be provisioned ahead of use, but you can scale them later as well.
The maximum number of consumers you can have is equal to your max number of shards.
You write customised code for your producers & consumers.
Real-time data transfer (~200ms).
Producers push records which consist of a partition key (dictates which shard to send data to) and a data blob (up to 1MB of data). Records have a throughput of 1MB/s/shard or 1000msg/s/shard.
Consumers receive records from Kinesis Data Streams which are similar to producer records except they also contain a sequence number which tells the consumer the position of the data within the shard. The throughput can operate in one of two modes: shared (2MB/s/shard for all consumers) or enhanced (2MB/s/shard/consumer).
Data is retained for between 1-365 days so that you can replay data if you need to.
Data inserted into Kinesis is immutable, therefore it cannot be deleted.
The partition keys provide key based ordering because data gets grouped into shards. This is useful if you want data coming from a particular source to always be grouped together e.g. managing a fleet of lorries and wanting all the data for each lorry to be grouped appropriately.
Producers include: AWS SDK (applications & clients will call the API at a low level), Kinesis Product Library, or the Kinesis Agent.
Consumers include: Lambda, Data Firehose, Data Analytics, as well as any client you create with the Kinesis Client Library and the AWS SDK.
More appropriate for applications that require high throughput, low latency data streams and don't necessarily need strict time ordering or deduplication of the data e.g. metric collection, real-time analytics.
There are 2 capacity modes available:
- Provisioned - pay per shard provisioned per hour and manage how many shards you have provisioned.
- On-demand - AWS manages the capacity of shards with a default capacity of 4MB/s or 4000msg/s which is the equivalent of 4 shards. It will scale your number of shards automatically based on the observed throughput peak of the last 30 days. However, you now pay per stream provisioned per hour + data in/out of your stream per GB.
##### Security
Deployed per region.
Use IAM to control access to the stream.
Data encrypted in-flight using HTTPS and at rest using KMS (can use client-side encryption but much harder).
Can use VPC Endpoints to allow traffic to pass from your private subnets to at data stream without passing through the public Internet.
Can monitor API calls using CloudTrail.
#### Data Firehose
Loads data streams into AWS data stores.
Producers send records of data of up to 1MB to Firehose which can perform optional transformations on the data using Lambdas before sending the data in batches to specified destinations.
Producers: AWS SDK, Kinesis Product Library, Kinesis Agent, or Kinesis Data Streams.
Destinations: AWS (S3, Redshift - writes data to S3 initially before loading, or OpenSearch), 3rd parties (Datadog, Splunk, New Relic, MongoDB), or custom HTTP endpoints.
Data is also written to a separate backup S3 bucket but you can specify whether you want all data to be written their or just failed data.
Just pay for the data going through Firehose.
Offers near real-time data transfer (60s latency for batches less than 1MB as it waits for the batch to fill up or 1MB of data must be sent).
No data stored in Firehose so doesn't support a replay capability.
Can optionally convert data output format to Apache Parquet or Apache ORC.
Setting a larger buffer size will result in a lower cost but higher latency.
#### Data Analytics
Analyses data streams with SQL or Apache Flink that can read from Kinesis Data Streams or Firehose for SQL apps or MSK for Flink apps.
Can apply SQL statements to your data or apply reference data in real-time.
Pay for the actual consumption rate.
#### Video Streams
Captures, processes, stores video streams.

## SNS (Simple Notification Service)
A publisher, e.g. an AWS service, will send a message to a single SNS topic, then the topic will push the message to any subscribers that subscribe to that topic which reduces the number of integrations required (one pub to one topic to many subs).
Can send notifications to subscribers that include SQS, Lambda, Kinesis Firehose, Emails, SMS & push notifications, HTTP endpoints.
No message retention unlike SQS.
Can have up to 12,500,000 subs to single topic and up to 100,000 topics.
Also has the ability to use Direct Publish instead of Topic Publish which allows you to publish directly to an application's endpoint
Has the same security features as SQS.
Able to push messages to destinations in other regions subject to appropriate security allowances.
Message filtering allows SNS to use a JSON policy to restrict the types of messages sent to subs. This means you can have certain messages sent to some subs that are subscribed to a topic but to other subs.

## SQS + SNS Fan-out
One way to easily push to multiple SQS queues that may be monitored by individual services is to push messages to an SNS topic which is monitored by multiple SQS queues. This provides an easy way to scale if you want to add more services + SQS queues to your SNS topic.
This provides faster processing for time-critical messages by removing the need for SQS queues to poll for messages and also guarantees message persistence because of SQS.

## Amazon MQ
Message broker that uses open protocols, e.g. MQTT, AMQP, STOMP, Openwire, WSS, which may already be put into practice in your on-prem applications.
This allows you to migrate to the cloud without re-engineering how your applications communicate.
Not as scalable as SQS/SNS.
Designed to work with RabbitMQ, ActiveMQ.
Runs on servers which can make use of multiple AZs for failover. Works by using standby servers in other AZs that are connected to the same backend storage (EFS) as your active server.
Has queue (SQS) and topic (SNS) features.

## Managed Streaming for Kafka (MSK)
An alternative to Kinesis that allows you to create Kafka clusters made up of broker and zookeeper nodes.
Gets deployed into your VPC in multiple AZs.
Has automatic recovery for common Kafka failures and all the data is stored in EBS volumes for as long as you want unlike Kinesis.
Can be deployed as a managed cluster or as serverless.
"Producers" write to a topic which is hosted on a broker node and replicated to other broker nodes.
"Consumers" then poll a topic and consume data from it.
Has 1MB max message size as the default but can be increased.
Topic and partitions in MSK work in a similar way to data streams + shards in Kinesis.
Where Kinesis may allow for shard splitting/merging, MSK only allows for adding partitions to topics.
Can make use of PLAINTEXT in-flight encryption as well as TLS

#aws
