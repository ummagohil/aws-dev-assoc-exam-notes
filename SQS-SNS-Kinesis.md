# AWS Integration & Messaging: SQS, SNS & Kinesis

## Amazon SQS: Standard Queues

- Oldest offer (over 10 years old)
- Fully managed service, used to decouple applications
- Attributes:
  - Unlimited throughput, unlimited number of messages in queue
  - Default retention of messages: 4 days, maximum of 14 days
  - Low latency (< 10ms on publish and receive)
  - Limitation of 256KB per message sent
- Can have duplicate messages (at least once delivery, occasionally)
- Can have out of order messages (best effort ordering)

### Producing Messages

- Produced to SQS using the SDK (SendMessage API)
- The message is persisted in SQS until a consumer deletes it
- Message retention: default 4 days, up to 14 days
- Example: send an order to be processed
  - Order ID
  - Customer ID
  - Any attributes you want

### Consuming Messages

- Consumers (running on EC2 instances, servers or AWS Lambda)
- Poll SQS for messages (receive up to 10 messages at a time)
- Process the messages (example: insert the message into an RDS database)
- Delete the messages using the DeleteMessage API

### Multiple EC2 Instances Consumers

- Consumers receive and process messages in parallel
- At least once delivery
- Best-effort message ordering
- Consumers delete messages after processing them
- We can scale consumers horizontally to improve throughput of processing

### Security

- Encryption
  - In-flight encryption using HTTPS API
  - At-rest encryption using KMS keys
  - Client-side encryption if the client wants to perform encryption/decryption itself
- Access Controls: IAM policies to regulate access to the SQS API
- SQS Access Policies (similar to S3 bucket policies)
  - Useful for cross-account access to SQS queues
  - Useful for allowing other services (SNS, S3 etc.) to write an SQS queue

### Message Visibility Timeout

- After a message is polled by a consumer, it becomes invisible to other consumers
- By default, the “message visibility timeout” is 30 seconds
- That means the message has 30 seconds to be processed
- After the message visibility timeout is over, the message is “visible” in SQS
- If a message is not processed within the visibility timeout, it will be processed twice
- A consumer could call the ChangeMessageVisibility API to get more time
- If visibility timeout is high (hours), and consumer crashes, re-processing will take time
- If visibility timeout is too low (seconds), we may get duplicates

## Dead Letter Queues (DLQ)

- If a consumer fails to process a message within the Visibility Timeout the message goes back to the queue
- We can set a threshold of how many times a message can go back to the queue
- After the MaximumReceives threshold is exceeded, the message goes into a dead letter queue (DLQ)
- Useful for debugging
- DLQ of a FIFO queue must also be a FIFO queue
- DLQ of a Standard queue must also be a Standard queue
- Make sure to process the message in the DQL before they expire
  - Good to set a retention of 14 days in the DLQ
- Redrive to source:
  - Feature to help consume messages in the DLQ to understand what is wrong with them
  - When our code is fixed, we can reprice the messages from the DLQ back into the source queue (or any other queue) in batches without writing custom code

### Delay Queues

- Delay a message (consumers don’t see it immediately) up to 15 minutes
- Default is 0 seconds (message is available right away)
- Can set a default at queue level
- Can override the default on send using the DelaySeconds parameter

## Certified Developer Concepts

- Long Polling
  - When a consumer requests messages from the queue, it can optionally “wait” for messages to arrive if there are none in the queue, this is known as Long Polling
  - LongPolling decreases the number of API calls made to SQS while increasing the efficiency and decreasing the latency of your application
  - The wait time can be between 1 second to 20 seconds (20 seconds being preferable)
  - Long Polling is preferable to Short Polling
  - Long polling can be enabled at the queue level or at the API level using ReceiveMessageWaitTimeSeconds
- SQS Extended Client
  - Message size is 256KB, how to send large messages, eg. 1GB?
  - Using the SQS Extended Client (Java Library)
- Must know APIs
  - CreateQueue (MessageRetentionPeriod), DeleteQueue
  - PurgeQueue: delete all the messages in the queue
  - SendMessage (DelaySeconds), ReceiveMessage, DeleteMessage
  - MaxNumberOfMessages: default 1, max 10 (for ReceiveMessage API)
  - ReceiveMessageWaitTimeSeconds: Long Polling
  - ChangeMessageVisibility: change the message timeout
  - Batch APIs for SendMessage, DeleteMessage, ChangeMessageVisibility helps decrease your costs

## FIFO Queues

- FIFO = First In First Out (ordering of messages in the queue)
- Limited throughput: 300 msg/s without batching, 3000 msg/s with
- Exactly-once send capability (by removing duplicates)
- Messages are processed in order by the consumer
- Deduplication
  - De-duplication interval is 5 mins
  - Two de-duplication methods:
    - Content-based deduplication: will do a SHA-256 hash of the message body
    - Explicitly provide a Message Deduplication ID
- Message Grouping
  - If you specify the same value of MessageGroupId in an SQS FIFO queue, you can only have one consumer, and all the messages are in order
  - To get ordering at the level of a subset of messages, specify different values for MessageGroupID
    - Messages that share a common Message Group ID will be in order within the group
    - Each Group ID can have a different consumer (parallel processing)
    - Ordering across groups is not guaranteed

## Amazon SNS

- The “event producer” only sends messages to one SNS topic
- As many “event receivers” (subscriptions) as we want to listen to the SNS topic notifications
- Each subscriber to the topic will get all the messages (note: new features to filter messages)
- Up to 12,500,000 subscriptions per topic
- 100,000 topics limit
- Many AWS services can send data directly to SNS for notifications
  - CloudWatch Alarms
  - AWS Budgets
  - Lambda
  - Auto Scaling Group (notifications)
  - S3 Buckets (events)
  - DynamoDB
  - CloudFormation (state changes)
  - AWS DMS (new replic)
  - RDS Events

### How to Publish

- Topic Publish (using the SDK)
  - Create a topic
  - Create a subscription (or many)
  - Publish to the topic
- Direct Publish (for mobile apps SDK)
  - Create a platform application
  - Create a platform endpoint
  - Publish to the platform endpoint
  - Works with Google GCM, Apple APNS, Amazon ADM

### Security

- Encryption
  - In-flight encryption using HTTPS API
  - At-rest encryption using KMS keys
  - Client-side encryption if the client wants to perform encryption/decryption itself
- Access controls: IAM policies to regulate access to the SNS API
- SNS Access Policies (similar to S3 bucket policies)
  - Useful for cross-account access to SNS topics
  - Useful for allowing other services (S3) to write to an SNS topic

## Amazon SNS and SQS: Fan Out Pattern

- Push once in SNS, receive in all SQS queues that are subscribers
- Fully decoupled, no data loss
- SQS allows for: data persistence, delayed processing and retries of work
- Ability to add more SQS subscribers over time
- Make sure your SQS queue access policy allows for SNS to write
- Cross-Region Delivery: works with SQS Queues in other regions

## Application: S3 events to multiple queues

- For the same combination of:event type (Eg. Object create) and prefix (eg. Images/) you can only have one S3 event rule
- If you want to send the same S3 event to many SQS queues, use fan-out

## Application: SNS to Amazon S3 through Kinesis Data Firehose

- SNS can send to Kinesis and therefore we can have the following solutions architecture
  - Buying service => SNS Topic => Kinesis Data Firehose => Amazon S3 + any supported KDF destination

## SNS: Message Filtering

- JSON policy used to filter messages sent to SNS topic’s subscriptions
- If a subscription doesn’t have a filter policy, it receives every message

## Kinesis

- Makes it easy to collect, process and analyse streaming data in real-time
- Ingest real-time data such as Application logs, metrics, website clickstreams, IoT telemetry data
- Kinesis Data Streams: capture, process and store data streams
- Kinesis Data Firehose: load data streams into AWS data stores
- Kinesis Data Analytics: analyse data streams with SQL or Apache Link
- Kinesis Video Streams: capture, process and store video streams

## Data Streams

- Retention between 1 day to 365 days
- Ability to reprocess (replay) data
- Once data is inserted in Kinesis, it can’t be delete d(immutability)
- Data that shares the same partition goes to the same shard (ordering)
- Producers: AWS SDK, Kinesis Producer Library (KPL), Kinesis Agent
- Consumers
  - Write your own: Kinesis Client Library (KCL), AWS SDK
  - Managed: AWS Lambda, Kinesis Data Firehose, Kinesis Data Analytics
- Capacity Modes
  - Provisioned mode
    - You choose the number of shards provisioned, scale manually or using API
    - Each shard gets 1MB/s in (or 1,000 records per second)
    - Each shard gets 2MB/s out (classic or enhanced fan-out consumer)
    - You pay per shard provisioned per hour
  - On-demand mode
    - No need to provision or manage the capacity
    - Default capacity provisioned (4 MB/s in or 4000 records per second)
    - Scales automatically based on observed throughput peak during the last 30 days
    - Pay per stream per house & data in/out per GB
- Security
  - Control access/authorisation using IAM policies
  - Encryption in flight using HTTPS endpoints
  - Encryption at rest using KMS
  - You can implement encryption/decryption of data on client side (harder)
  - VPC endpoints available for Kinesis to access within VPC
  - Monitor API calls using CloudTrail

## Producers

- Puts data records into data streams
- Data records consist of:
  - Sequence number (unique per partition-key within shard)
  - Partition key(must specify while put records into stream)
  - Data blob (up to 1MB)
- Producers
  - AWS SDK: simple producer
  - Kinesis Producer Library (KPL): C++, Java, batch, compression, retries
- Write throughput: 1 MB/sec or 1000 records/sec per shard
- PutRecord API
- Use batching with PutRecords API to reduce costs & increase throughput

## Consumers

- Get data records from data streams and process them
- AWS Lambda
- Kinesis Data Analytics
- Kinesis Data Firehose
- Custom Consumer (AWS SDK): classic or enhanced fan-out
- Kinesis Client Library (KCL): library to simplify reading from data stream
- Consumer Types
  - Shared (classic) fan-out consumer - pull
    - Low number of consuming applications
    - Read throughput: 2 MB/sec per shard across all consumers
    - Max. 5 GetRecords API call/sec
    - Latency ~200 ms
    - Minimise cost ($)
    - Consumers poll data from Kinesis using GetRecords API call
    - Returns up to 10MB (then throttle for 5 seconds) or up to 10,000 records
  - Enhanced fan-out consumer - push
    - Multiple consuming applications for the same stream
    - 2 MB/sec per consumer per shard
    - Latency ~70ms
    - Higher costs ($$$)
    - Kinesis pushes data to consumers over HTTP/2 (SubscribeToShardAPI)
    - Soft limit of 5 consumer applications (KCL) per data stream (default)
- AWS Lambda
  - Supports Classic & Enhanced fan-out consumers
  - Read records in batches
  - Can configure batch size and batch window
  - If error occurs Lambda retires until succeeds or data expired
  - Can process up to 10 batches per shard simultaneously

## Client Library

- A Java library that helps read records from a Kinesis Data Stream with distributed applications sharing the read workload
- Each shard is to be read by only one KCL instance
  - 4 shards = max. 4 KCL instances
  - 6 shards = max. 6 KCL instances
- Progress is checkpointed into DynamoDB (needs IAM access)
- Track other workers and share the work amongst shards using DynamoDB
- KCL can run on EC2, Elastic Beanstalk and on-premises
- Records are read in order at the shard level
- Versions:
  - KCL 1.x (supports shared consumer)

## Operations

- Shard splitting
  - Used to increase the Stream capacity (1MB/s data in per shard)
  - Used to divide a “hot shard”
  - The old shard is closed and will be deleted once the data is expired
  - No automatic scaling (manually increase/decrease capacity)
  - Can’t split into more than two shards in a single operation
- Merging shards
  - Decrease the stream capacity and save costs
  - Can be used to group two shards with low traffic (cold shards)
  - Old shards are closed and will be deleted once the data is expired
  - Can’t merge more than two shards in a single operation

## Data Firehose

- Fully managed service, no administration, automatic scaling, server less
  - AWS: Redshift/Amazon S3 OpenSearch
  - 3rd part partner: Splunk/MongoDb/DataDog/NewRelic
  - Custom: send to any HTTP endpoint
- Pay for data going through Firehose
- Near real time
  - Buffer interval: 0 seconds (no buffering) to 900 seconds
  - Buffer size: minimum 1MB
- Supports many data formats, conversions, transformations, compression
- Supports custom data transformations using AWS Lambda
- Can send failed or all data to a backup S3 bucket
- Kinesis Data Streams vs Firehose
  - Kinesis Data Streams
    - Streaming service for ingest at scale
    - Write custom code (producer/consumer)
    - Real-time (~200 ms)
    - Manage scaling (shard splitting/merging)
    - Data storage for 1 to 165 days
    - Supports replay capability
  - Kinesis Data Firehose
    - Load streaming data into S3/Redshift/OpenSearch/3rd party/customer HTTP
    - Fully managed
    - Near real-time
    - Automatic scaling
    - No data storage
    - Doesn’t support replay capability

## Data Analysis

- SQL Application
  - Real-time analytics on Kinesis Data Streams & firehose using SQL
  - Add reference data from Amazon S3 to enrich streaming data
  - Fully managed, no servers to provision
  - Automatic scaling
  - Pay for actual consumption rate
  - Output
    - Kinesis Data Streams: create streams out of the real-time analytics queries
    - Kinesis Data Firehose: send analytics query results to destinations
  - Use cases
    - Time-series analytics
    - Real-time dashboards
    - Real-time metrics
- Apache Flink
  - Use Flink (Java, Scala or SQL) to process and analyse streaming data
  - Run any Apache Flink application on a managed cluster on AWS
    - Provisioning compute resource, parallel computation, automatic scaling
    - Application backups (implemented as checkpoints and snapshots)
    - Use any Apache Flink programming features
    - Flink does not read from Firehose (use Kinesis Analytics for SQL instead)

## Data Ordering for Kinesis vs SQS FIFO

### Into Kinesis

- Imagine you have a 100 trucks (truck_1, truck_2,… truck_100) on the road sending their GPS positions regularly into AWS
- You want to consume tend at a in order for each trust, so that you can track their movements accurately
- How should you send that data into Kinesis
- Answer: send using a “partition key” value of the “track_id”
- The same key will always go to the same shard

### Into SQS

- For SQS standard, there is no ordering
- For SQS FIFO, if you do’t use a Group ID, messages are consumed in the order they are sent, with only one consumer
- You want to scale the number of consumers, but you want messages to be “grouped” when they are related to each other
- Then you use a Group ID (similar to partition key in Kinesis)

### Kinesis vs SQS

- Let’s assume 100 trucks, 5 kinesis shards, 1 SQS FIFO
- Kinesis data streams:
  - On average you’ll have 20 trucks per shard
  - Trucks will have their data ordered within each shard
  - The maximum amount of consumers in parallel we can have is 5
  - Can receive up to 5 MB/s of data
- SQS FIFO
  - You only have one SQS FIFO queue
  - You will have 100 Group ID
  - You can have up to 100 consumers (due to the 100 Group ID)
  - You have up to 300 messages per second (or 3,000 if using batch)

## SQS vs SNS vs Kinesis

- SQS
  - Consumer “pull data”
  - Data is deleted after being consumed
  - Can have as many workers (consumers) as we want
  - No need to provision throughput
  - Ordering guarantees only FIFO queues
  - Individual message delay capability
- SNS
  - Push data to many subscribers
  - Up to 12,500,000 subscribers
  - Data is not persisted (lost if not delivered)
  - Pub/sub
  - Up to 100,000 topics
  - No need to provision throughput
  - Integrates with SQS for fan-out architecture pattern
  - FIFO capability for SQS FIFO
- Kinesis
  - Standard: pull data
    - 2 MB per shard
  - Enhanced-fan out: push data
    - 2 MB per shard per consumer
  - Possibility to reply data
  - Meant for real-time big data, analytics and ETL
  - Ordering at the shard level
  - Data expires after X days
  - Provisioned mode or on-demand capacity mode
