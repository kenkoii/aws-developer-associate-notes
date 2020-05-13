# DynamoDB

NoSQL databases

* Means Not only SQL databases
* non relational databases and are distributed
* include MongoDB, DynamoDB, etc.
* Do not support join
* All the data that is needed for a query is present in one row
* NoSQL db don't perform aggregation such as "SUM"
* NoSQL databases scale horizontally

## Overview
* Full Managed, Highly available with replication across 3AZ
* Scales to massive workloads, distributed database
* Millions of requests per seconds, trillions of row, 100s of TB of storage
* Fast and consistent in performance(low latency on retrieval)
* Integraded with IAM for security, authorization and administration
* Enables event driven programming with DynamoDB Streams
* Low cost and auto scaling capabilities

Basics:

* DynamoDB is made of **tables**
* Each table has a **primary key**(must be decided at creation time)
* Each table can have an infinite number of items(=rows)
* Each item has **attributes** (can be added over time - can be null)
* Maximum size of an item is 400kb
* Data type supported are:
	* Scalar Types: String, Number, Binary, Boolean, Null
	* Document Types: List, Map
	* Set Types: String Set, Number Set, Binary Set

## Primary Keys
* Option 1: **Partition key only(HASH)**
	* Partition key must be:
		* unique for each item
		* diverse so that data is distributed
		* ex. `user_id` for a users table
* Option 2: **Partition key + Sort key**
	* The combination must be unique
	* Data is grouped by partition key
	* Sort key == Range 

## Provisioned Throughput
* Table must have provisioned read and write capacity units
* **Read Capacity Units(RCU)**
* **Write Capacity Units(WCU)** 
* Option to setup auto-scaling of throughput to meet demand
* Throughput can be exceeded temporarily using "burst credit"
* If burst credit are empty, you'll get a `ProvisionedThrougputException`
* It's advised then to do an exponential back-off retry

### WCU
* One `Write capacity unit` represents one write per second per second for an item up
* to 1 KB in size
* If the items are larger than 1KB, more WCU are consumed
* Examples:
	* 10 objects per second of 2KB each
		* `2 * 10 = 20 WCU`
	* 6 objects per second of 4.5KB each
		* `6 * 5 = 30 WCU` (4.5 gets rounded up)
	* 120 objects per minute of 2KB each
		* `120 / 60 * 2 = 4 WCU`

### Strongly Consistent Read
* If we read just after a write, we will get the correct data

### Eventually Consistent Read
* If we read just after a write, it's possible we'll get unexpected response because replication takes time


* **By default:** DynamoDB uses `Eventually Consistent Reads` but **GetItem**, **Query** & **Scan** provide a `ConsistentRead` parameter you can set to true.

## RCU
* One `read capacity unit` represents one strongly consistent read per second, or two eventually consistent reads per second, for an item up to 4KB in size
* If Items are larger than 4KB, more RCU are consumed
* Examples:
	* 10 strongly consistent reads per second of 4KB each
		* `10 * 4KB / 4KB = 10 RCU`
	* 16 eventually consistent reads per seconds of 12KB each
		* `(16 / 2) * (12 / 4) = 24 RCU`
	* 10 strongly consistent reads per seconds of 6KB each
		* `10 * 8KB / 4 = 20 RCU` (we have to round up 6KB to 8KB)

### Partitions Internal
* Data is divided in partitions
* Partition keys go through a hashing algorithm to know which partition they go to
* To compute the number of partitions:
	* By capacity: (Total RCU/3000) + (Total WCU / 1000)
	* By size: Total Size / 10GB
	* Total Partitions = CEILING(MAX(Capacity, Size))
* **WCU and RCU** are spread **evenly** between partitions

### Throttling
* If we exceed our RCU or WCU, we get **ProvisionedThroughputExceededExceptions**
* Reasons:
	* Hot keys: one partition key is being read too many times
	* Hot partitions
	* Very large items: RCU and WCU depends on size of items
* Solutions:
	* Exponential Back-off
	* Distribute partition keys as much as possible
	* If RCU issue, we can use DynamoDB Accelerator(DAX)

## API
### Writing Data
* **PutItem** - Write data to DynamoDB(create data or full replace)
	* Consumes WCU
* **UpdateItem** - Update data in DynamoDB(partial update of attributes)
	* Possibility to use Atomic Counters and increase them
* **ConditionalWrites**:
	* Accept a write/update only if conditions are respected, otherwise rejected
	* Helps with concurrent access to items
	* No performance impact

### Deleting data
* **DeleteItem**
	* Delete an individual row
	* ability to perform a conditional delete
* **DeleteTable**
	* Delete a whole table and all its items
	* Much quicker deletion than calling DeleteItem on all items
	
### Batching Writes
* **BatchWriteItem**
	* Up to 25 **PutItem** and/or **DeleteItem** in one call
	* Up to 16MB of data written
	* Up to 400KB of data per item
* Batching allows you to save in latency by reducing the number of API calls done against DynamoDB
* Operations are done in parallel for better efficiency
* It's possible for part of a batch to fail, in which case we have the chance to retry the failed items(using exponential back-off algorithm)

### Reading Data
* **GetItem**
	* Read based on Primary key
	* Primary Key = HASH or HASH-RANGE
	* Eventually consistent read by default
	* Option to use strongly consistent reads(more RCU - might take longer)
	* **ProjectionExpression** can specify what attributes to retrieve
* **BatchGetItem**
	* Up to 100 items
	* Up to 16MB of data
	* Items are retrieved in parallel to minimize latency

### Querying data
* **Query**
	* PartitionKey value (**must be = operator**)
	* SortKey value(=, <, <=, >, >=, Between, Begin) - optional
	* FilterExpression to further filter(client side filtering)
* Returns:
	* Up to 1MB data
	* Or number of items specified in Limit
* Able to do pagination on the results
* Can query table, a local secondary index, or a global secondary index

### Scan
* **Scan** the entire table and then filter out data(inefficient)
* Returns up to 1MB of data - use pagination to keep on reading
* Consumes a lot of RCU
* Limit impact using Limit or reduce the size of the result and pause
* For faster performance, use **parallel scans**:
	* Multiple instances scan multiple partitions at the same time
	* Increases the throughput and RCU Consumed
	* Limit the impact of parallel scans just like you would for Scans
* Can use a **ProjectionExpression** + **FilterExpression**(no change to RCU)

## Indexes
### LSI(Local Secondary Index)
* Alternate range key for your table, **local to the hash key**
* Up to five local secondary indexes per table
* The sort key consists of exactly one scalar attribute
* The attribute that you choose must be a scalar String, Number or Binary
* **LSI** must be defined at table creation time

### GSI(Global Secondary Index)
* To speed up queries on non-key attributes
* GSI = partition key + optional sort key
* The index is a new "table" and we can project attributes on it
	* The partition key and sort key of the original table are always projected(KEYS_ONLY)
	* Can specify extra attributes to project(INCLUDE)
	* Can use all attributes from main table(ALL)
* Must define RCU/WCU for the index
* Possibility to add/modify GSI(not LSI)

### Index and Throttling
* GSI
	* If the writes are throttled on the GSI, then the main table will be throttled
	* Even if the WCU on the main tables are fine
	* Choose your GSI partition key carefully
	* Assign your WCU capacity carefully
* LSI
	* Uses the WCU and RCU of the main table
	* No special throttling considerations 

## DynamoDB Concurrency Model
* Conditional Update/Delete
* That means that you can ensure an item hasn't changed before altering it
* DynamoDB is optimistic locking/concurrency database

## DAX(Accelerator)
* Seamless cache for DynamoDB, no application rewrite
* Writes go through DAX to DynamoDB
* Micro second latency for cached reads & queries
* Solves Hot Key problem(too many reads)
* 5 minutes TTL for cache by default
* Up to 10 nodes in the cluster
* MultiAZ (3 notes minimum recommended for prod)
* Secure(encyption at rest with KMS, VPC, IAM, CloudTrail...)

## DynamoDB Streams
* Changes in DynamoDB(Create, Update Delete) can end up in a DynamoDB Stream
* This stream can be read by AWS Lambda and we can then do:
	* react to changes in real time(welcome email to new users)
	* Analytics
	* Create derivative tables/views
	* Insert into ElasticSearch
* Could implement cross region replication using Streams
* Streams has 24 hours of data retention

## TTL (Time to Live)
* TTL = automatically delete an item after an expiry date/time
* TTL is provided at no extra cost, deletions do not use WCU/RCU
* TTL is a background task operated by the DynamoDB service itself
* Helps reduce storage and manage the table size over time
* Helps adhere to regulatory norms
* TTL is enabled per row(define TTL column and add a date there)
* DynamoDB typically deletes expired items within 48 hours of expiration
* Deleted items due to TTL are also deleted in GSI/LSI
* DynamoDB Streams can help recover expired items

## CLI
* `--projection-expression` attributes to retrieve
* `--filter-expression` filter results
* General CLI pagination options including DynamoDB/S3
	* Optimization:
		* `--page-size` full dataset is still received but each API call will request less data(helps avoid timeouts)
	* Pagination
		* `--max-items` max number of resultsreturned by the CLI. Returns *NextToken*
		* `--starting-token` specify the last received *NextToken* to keep on reading

## Transactions
* Ability to Create/Update/Delete multiple rows in different tables at the same time
* All or nothing type of operation
* Write Modes: Standard, Transactional
* Read Modes: Eventual Consistency, Strong Consistency, Transactional
* Consume 2x of WCU/RCU

## Other features
* Security:
	* VPC Endpoints available to access DynamoDB without internet
	* Access fully controlled by IAM
	* Encryption at rest using KMS
	* Encryption in transit using SSL/TLS
* Backup and Restore
	* Point in time restore like RDS
	* No performance impace
* Global Tables
	* Multi region, fully replicated, high performance
* Amazon DMS can be used to migrate to DynamoDB
* You can launch a local DynamoDB on your computer for dev purposes


# TLDR - Exam Tips:
## Which Secondary Index to choose
* Local Secondary Index
  * Only use if you won't have item collections over 10GB
  * You can define this during table creation
  * You can save money by sharing capacity with base table
  * Secondary index has same index with base table
  * You need strong consistency
* Global Secondary Index
  * Preferred over Local Secondary index
  * If table exists already you must use GSI
  * You don't need strong consistency

