# Integration and Messaging

Two patterns of application communication

* Synchronous communications
	* application to application
* Asynchronous/Event based
	* application to queue to application

* Synchronous between apps can be problematic if there are sudden spikes of traffic
* Decoupling applications
	* using **SQS**: queue model
	* using **SNS**: pub/sub model
	* using **Kinesis**: real-time streaming model
* These services can scale independently from our application

## SQS
### What's a queue?
* Producers send messages to a queue
* Consumers poll messages from a queue
* Both can scale independently

### Standard Queue
* Fully managed
* Scales from 1 messages per second to 10,000 per second
* Default retention of messages: 4 days, maximum of 14 days
* No limit to how many messages can be in the queue
* Low latency(<10mson publish and receive)
* Horizontal scaling in terms of number of consumeers
* Can have duplicate messages
* Can have out of order messages(best effort ordering)
* Limitation f 256kb per message sent

### Delay Queue
* Delay a message(consumers dont see it immediately) up to 15 minutes
* Default is 0 seconds(message is available right away)
* Can set a default at the queue level
* Can override the default using the DelaySeconds parameter

### Producing Messages
* Define Body
	* string, up to 256kb
* Add message attributes(metadata - optional)
	* Name
	* Type
	* Value
* Provide Delay Delivery(optional)
* Get back:
	* Message identifier
	* MD5 hash of the body

### Consuming Messages
* Poll SQS for messages (receive up to 10 messages at a time)
* Process the message within the visibility timeout
* Delete the message using the message ID & receipt handle
* Process means:
	* perform something using the message

### Visibility Timeout
* When a consumer polls a message from a queue, the message is "invisible" to the other consumers for a defined period.
	* Set between 0 seconds and 12 hours(default 30 seconds) 
	* If too high(15 minutes) and consumer fails to process the message, yo umust wait a long time before processing the message again
	* If too low(30 seconds) and consumer need time to process the message(2 minutes), another consumer will receive the message and the message will be processed more than once
* **ChangeMessageVisibility** API to change the visibility while processing a message
* **DeleteMessage** API to tell SQS the message was successfully processed

### Dead Letter Queue
* If a consumer fails to process a message within the VisibilityTimeout, the message goes back to the queue!
* We can set a threshold of how many times a message can go back to the queue - its called a **redrive policy**
* After the threshold is exceeded, the message goes into a dead letter queue(DLQ)
* We have to create a DLQ first and then designate it dead letter queue
* Make sure to process the messages in the DLQ before they expire

### Long Polling
* Long polling means that consumers can optionally **wait** for messages to arrive if there are non in the queue
* Long polling decreases the number of API calls made to SQS while increasing the efficiency and latency of your application
* The wait time can be between 1 sec to 20 sec(20 sec preferrable)
* Long Polling is preferable to Short Polling
* Long Polling can be enabled at the queue level or at the API level using `WaitTimeSeconds`

### FIFO Queue
* **First In - First Out** - not available in all regions
* Name of the queue must end in .fifo
* Lower throughput(up to 3000/s with batching, 300/s without)
* Messaged are processed in order by the consumer
* Messages are sent exactly once
* No per message delay(only per queue delay) because fifo order might be wrong

### FIFO Features
* **Deduplication**: not send the same message twice
	* Provide a MessageDeduplicationId with your message
	* De-duplication interval is 5 minutes
	* Content based duplication: the MessageDeduplicationId is generated as the SHA-256 of the message body(not the attributes)
* **Sequencing**:
	* To ensure strict ordering between messages, specify a MessageGroupId
	* Messages with different Group ID may be received out of order
	* E.g. to order messages for user, you could use the "user_id" as a group_id
	* Messages with the same Group ID are delivered to one consumer at a time

## SQS Extended Client
* Message size limit is 256kb
* Using the SQS Extended Client, we can send larger messages(Java library)
* Producer sends large message to S3, send small metadata message to queue, consumer polls for messages,  reads small metadata message, retrieve large message from S3

## SQS Security
* Encryption in flight using the HTTPS endpoint
* Can enable SSE(Server Side Encryption) using KMS
	* Can set the CMK(Customer Managed Key) we want to use
	* Can set the data key reuse period(between 1 minute and 24 hours)
	* SSE only encrypts the body, not the metadata(message ID, timestamp, attributes)
* IAM policy must allow usage of SQS
* SQS queue access policy
* No VPC Enpoint, must have internet access to access SQS

### Must know API
* CreateQueue, DeleteQueue
* PurgeQueue: delete all the messages in queue
* SendMessage, ReceiveMessage, DeleteMessage
* ChangeMessageVisibility: change the timeout

* Batch APIs for SendMessage, DeleteMessage, ChangeMessageVisibility helps decrease your cost

## AWS SNS
* Pub/sub, send one message to multiple consumers
* The **event producer** only sends message to one SNS topic
* As many **event receivers** (subscriptions) as we want to listen to the SNS topic notifications
* Each subscriber to the topic will get all the messages
	* note: new feature to filter messages
* Up to 10,000,000 subscriptions per topic
* 100,000 topics limit
* Subscribers can be:
	* SQS
	* HTTP/HTTPS(with delivery retries - how many times)
	* Lambda
	* Email/Email Json
	* SMS Messages
	* Mobile Notifications

### How to publish
* Topic Publish(within your AWS Server - using the SDK)
	* Create a topic
	* Create a subscription(or many)
	* Publish to the topic

* Direct Publish(for mobile apps SDK)
	* Create a platform application
	* Create a platform endpoint
	* Publish to the platform endpoint
	* Works with Google GCM, Apple APNS, Amazon ADM
	
### SNS + SQS: Fan Out
* Push once in SNS, receive in many SQS
* Fully decoupled
* No data loss
* Ability to add receivers of data later
* SQS allows for delayed processing
* SQS allows for retries of work
* May have many workers on one queue and one worker on the other queue

## AWS Kinesis
* Alternative to Apache Kafka
* Great for application logs, metrics, IoT, clickstreams
* Great for "real-time" big data
* Great for streaming processing frameworks(Spark, Nifi, etc...)
* Data is automatically replicated to 3 AZs

* **Kinesis Streams**: low latency streaming ingest at scale
* **Kinesis Analytics**: perform real-time analytics on streams using SQL
* **Kinesis Firehose**: load streams into S3, Redshift, ElasticSearch

## Kinesis Streams
* Streams are divided in ordered Shards/Partitions
* Data retention is 1 day by default, can go up to 7 days
* Ability to reprocess/replay data
* Multiple applications can consume the same stream
* Real-time processing with scale of throughput
* Once data is inserted, it cant be deleted(immutability)

### Shards
* One stream is made of many different shards
* 1MB/s or 1000msgs/s at write PER SHARD
* 2MB/s at read PER SHARD
* Billing is per shard provisioned, can have as many shards as you want
* Batching available or per message calls
* The number of shards can evolve over time(reshard/merge)
* **Records are ordered per shard**

### Kinesis API - Put records
* PutRecord API + Partition key that gets hashed
* The same key goes to the same partition(helps with ordering for a specific key)
* Messages sent get a "sequence number"
* Choose a partition key that is highly distributed(helps prevent "hot partition")
	* user_id if many users
* Use batching with PutRecords to reduce costs and increase throughput
* `ProvisionedThroughputExceeded` exception if we go over the limits
* Can use CLI, AWS SDK, or producer libraries from various frameworks

### Kinesis API - Exceptions
* **ProvisionedThroughputExceeded** Exceptions
	* Happens when sending more data(exceeding MB/s or TPS for any shard)
	* Make sure you don't have a hot shard(such as your partition key is bad and too much data goes to that partition)
* Solution
	* Retries with backoff
	* Increase shards(scaling)
	* Ensure your partition key is a good one

### Kinesis API - Consumers
* Can use a normal consumer(CLI, SDK, etc...)
* Can use Kinesis Client library(in java, node, python, ruby, .net)
	* KCL uses DynamoDB to checkpoint offsets
	* KCL uses DynamoDB to track other workers and share the work amongst shards

	
### Kinesis KCL in Depth
* Java library for distributed applications to share read record workload from a Kinesis Stream
* Rule: each shard is to be read by only one KCL instance
* Means 4 shards = max 4 KCL instances
* Progress is checkpointed into DynamoDB(need IAM Access)
* KCL can run on EC2, Elastic Beanstalk, on premise applications
* _Records are read in order at the shard level_

### Kinesis Security
* Control access/authorization usin IAM policies
* Encryption in flight using HTTPS endpoints
* Encryption at rest using KMS
* Possibility to encrypt/decrypt data client side(harder)
* VPC Endpoints available for Kinesis to access within VPC

### Analytics
* Perform real-time analytics on Kinesis Streams using SQL
	* Auto Scaling
	* Managed
	* Continuous
* Pay for actual consumption rate
* Can create streams out of the real-time queries

### Firehose
* Fully managed service, no administration
* Near Real Time(60 sec latency)
* Load data into Redshift/S3/ElasticSearch/Splunk
* Automatic Scaling
* Support many data format (pay for conversion)
* Pay for the amount of data going through Firehose

## Summary
* SQS:
	* consumers pull data
	* data is deleted after being consumed
	* can have as many workers(consumers) as we want
	* No need to provision throughput
	* No ordering guarantee(except FIFO queues)
	* Individual message delay capability
* SNS:
	* Pub/sub
	* Push data to many subscribers
	* Up to 10,000,000 subscribers
	* Data is not persisted(lost if not delivered
	* Up to 100,000 topics
	* no need to provision throughput
	* Integrates with SQS for fan-out architecture pattern
* Kinesis:
	* Consumers pull data
	* As many consumers as we want
	* possibility to replay data
	* Meant for real-time big data, analytics, ETL
	* Ordering at the shard level
	* data expires after X days(1 to 7)
	* must provision throughputo