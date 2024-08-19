# AWS Serverless: DynamoDB

## Traditional Architecture

- Traditional applications leverage RDBMS database
- These databases have the SQL query language
- Strong requirements about how the data should be modelled
- Ability to do query joins, aggregations, complex computations
- Vertical scaling (getting a more powerful CPU/RAM/IO)
- Horizontal scaling (increasing reading capability by adding EC2/RDS Read Replicas)

## NoSQL Databases

- Non-relational database and are distributed
- Incudes MongoDB, DynamoDB
- Do not support query joins (or just limited support)
- All the data that is needed for a query is present in one row
- Don’t preform aggregations such as SUM, AVG etc.
- These databases scale horizontally
- There’s no right/wrong for NoSQL vs SQL, they just require to model the data differently and think about user queries differently

## Amazon DynamoDB

- Fully managed, highly available with replication across multiple AZs
- NoSQL database - not a relational database
- Scales to massive workloads, distributed database
- Millions of requests per seconds, trillions of rows, 100s of TB of storage
- Fast dance consistent in performance (low latency on retrieval)
- Integrated with IAM for security, authorisation and administration
- Enables event driven programming with DynamoDB Streams
- Low cost and auto-scaling capabilities
- Standard & Infrequent Access (IA) Table Class
- Basics
  - DynamoDB is made up of tables
  - Each table has a Primary Key (must be decided at creation time)
  - Each table can have an infinite number of items (=rows)
  - Each item has attributes (can be added over time - can be null)
  - Maximum size of an item is 400KB
  - Data types supported
    - Scalar: strings, numbers, binary, boolean, null
    - Document: list, map
    - Set types: string set, number set, binary set
- Primary keys
  - Option 1: Partition Key (HASH)
    - Partition key must be the unique for each item
    - Partition key must be “diverse” so that the data is distributed
    - Example: “USED_ID” for a users table
  - Option 2: Partition Key + Sort Key (HASH + RANGE)
    - The combination must be unique for each item
    - Data is grouped by partition key
    - Example: users-game table, “USER_ID” for Partition Key and “Game_ID” for Sort Key

## DynamoD WCU & RCU - Throughput

### Read/Write Capacity Modes

- Control how you manage your table’s capacity (read/write throughput)
- Provisioned mode (default)
  - You specify the number of read/writes per second
  - You need to plan capacity beforehand
  - Pay for provisioned read and write capacity units
- On-Demand mode
  - Read/writes automatically scale up/down with your workloads
  - No capacity planning needed
  - Pay for what you use, more expensive ($$$)
- You can switch between different modes once every 24 hours

### Provisioned

- Table must have provisioned read and write capacity units
- Read Capacity Units (RCU): throughput for reads
- Write Capacity Units (WCU): throughput for writes
- Option to set up auto-scaling of throughput to meet demand
- Throughput can be exceeded temporarily using “Burst Capacity”
- If Burst Capacity has been consumed, you’ll get a “ProvisionedThroughputExceedException”
- It’s then advised to do an exponential backoff retry

### Write Capacity Units (WCU)

- One write capacity unit (WCU) represents one write per second for an item up to 1KB in size
- If the items are larger than 1KB, more WCUs are consumed
- Examples
  - We write 10 items per second, with items size 2KB: 10\*(2kb/1kb) = 20 WCUs
  - We write 6 items per second, with items size 4.5KB: 6\*(5kb/1kb) = 30 WCUs (4.5 gets rounded up to the upper KB)

Strongly Consistent Read vs. Eventually Consistent Read

- Eventually Consistent Read (default)
  - If we read just after a write, it’s possible we’ll get some stale data because of replication
- Strongly Consistent Read
  - If we read just after a write, we will get the correct data
  - Set “ConsistentRead” parameter to True in API calls (GetItem, BatchGetItem, Query, Scan)
  - Consumes twice the RCU

### Read Capacity Units (RCU)

- One Read Capacity Unit (RCU) represents one strongly consistent read per second, or two Eventually Consistent Reads per second, for an item up to 4KB in size
- If the items are larger than 4KB, more RCUs are consumed
  - Examples
    - 10 strongly consistent reads per second with item size of 4KB: 10\*(4kb/4kb) = 10 RCUs
    - 16 eventually consistent reads per second with item size 12 KB: (16/2)\*(12kb/4kb) = 24 RCUs

### Partitions Internal

- Data is stored in partitions
- Partition Keys go through a hashing algorithm to know which partition they go to
- To computer the number of partitions:
  - # of partitions (by capacity) = (Total RCUs/3000) + (Total WCUs/1000)
  - # of partitions (by size) = Total Size/10GB
  - # of partitions = ceil(max(# of partitions by capacity, # of partitions by size))
- WCUs and RCUs are spread evenly across partitions

## Throttling

- If we exceed provisioned RCUs or WCUs, we get “ProvisionedThroughputExceededException”
- Reasons
  - Hot keys: one partition key is being read too many times (eg. Popular item)
  - Hot partitions
  - Very large items, remember RCU and wCU depends on size of items
- Solutions
  - Exponential backoff when exception is encountered (already in SDK)
  - Distribute partition keys as much as possible
  - IF RCU issue, we can use DynamoDB Accelerator (DAX)

## On-Demand

- Read/writes automatically scale up/down with your workloads
- No capacity planning needed (WCU/RCU)
- Unlimited WCU & RCU, no throttle, more expensive
- You’re charged for reads/writes that you can use in terms of RRU and WRU
- Read Request Unit (RRU): throughput for reads (same as RCU)
- Write Request Units (WRU): throughput for write s(same as WCU)
- 2.5x more expensive than provisioned capacity (use with care)
- Use case: unknown workloads, unpredictable application traffic, etc.

## Basic Operations

### Writing Data

- PutItem
  - Creates a new item or fully replace an old item (same Primary Key)
  - Consumers WCUs
- UpadteItem
  - Edits an existing item’s attribute or adds a new item if it doesn’t exist
  - Can be used to implement Atomic Counters - a numeric attribute that’s unconditionally incremented
- Conditional Writes
  - Accept a write/update/delete only if conditions are met, otherwise returns an error
  - Helps with concurrent access to items
  - No performance impact

### Reading Data

- GetItem
  - Read based on Primary key
  - Primary Key can be HASH or HASH+RANGE
  - Eventually Consistent Read (default)
  - Option to use Strongly Consistent Reads (more RCU - might take longer)
  - ProjectExpression can be specified to retrieve only certain attributes
- Query
  - Query returns items based on:
    - KeyConditionExpression
      - Partition Key value (must be = operator) - required
      - Sort Key value (=, <, <=, >, >=, between, begins with) - optional
    - FilterExpression
      - Additional filtering after the Query operation (before data returned to you)
      - Use only with non-key attributes (does not allow HASH or RANGE attributes)
  - Returns:
    - The number of items specified in Limit
    - Or up to 1MB of data
  - Ability yo do pagination on results
  - Can query table, a Local Secondary Index, or a Global Secondary Index
- Scan
  - Scan the entire table and them filter rout data (inefficient)
  - Returns up to 1MB of data - use pagination to keep on read
  - Consumes a lot of RCU
  - Limit pact using Limit or educe the size of the result and pause
  - For faster perfjoamcne, use Parallel Scan
    - Multiple workers scan multiple data segments at the same time
    - Increases the throughput and RCU consumed
    - Limit the impact of parallel scans just like you would for Scans
  - Can use ProjectionExpression & FilterExpression (no changes to RCU)

### Deleting Data

- DeleteItem
  - Delete an individual item
  - Ability other than perform a condition delete
- DeleteTable
  - Delete a whole table and all its items
  - Much quicker deletion than calling DeleteItem on all items

### Batch Operations

- Allows you to save in latency by reducing the number of API calls
- Operations are done in apparel for better efficiency
- Part of a batch can fail; in which case we need to try again for the failed items
- BatchWriteItem
  - Up to 25 PutItem and/or DeleteItem in one call
  - Up to 16MB of data written, up to 400KB of data per item
  - Can’t update items (use UpdateItem)
  - UnprocessedItems for failed write operations (exponential backoff or add WCU)
- BatchGetItem
  - Return items from one or more tables
  - Up to 100 items, up to 16MB of data
  - Items are retrieved in parallel to minimise latency
  - UnprocessedKeys for failed read operations (exponential backoff or add RCUs

### PartiQL

- SQL-compatible query language for DynamoDB
- Allows you to select, insert, update and delete data in DynamoDB using SQL
- Run queries across multiple DynamoDB tables
- Run PartiQL queries from:
  - AWS Management Console
  - NoSQL Workbench for DynamoDB
  - DynamoDB APIs
  - AWS CLI
  - AWS SDK

### Conditional Writes

- For PutItem, UpdateItem, DeleteItem and BatchWriteItem
- You can specify a condition expression to determine which items show be modified:
  - attribute_exists
  - attribute_not_exists
  - attiribute_type
  - contains (for string)
  - begins_with (for string)
  - ProductCategory IN (:cat1, :cat2) and Price between :low and :high
- Note: Filter Expression filters the results of read queries, while Condition Expressions are for write operations

### Example on Delete Item

- attribute*note_exits: only succeeds if the attribute doesn’t exist yet (no value*
- attribute_exists: opposite of attribute_not_exits

#### Conditional Writes - Do Not Overwrite Elements

- attribute_not_exists(partition_key): make sure the item isn’t overwritten
- attribute_not_exists(partition_key) and attribute_not_exists(sort_key): make sure the partition/sort key combinations is not overwritten

##### Example of String Comparisons

- begins_with: check if prefix matches
- contains: check if string is contained in another string

## Indexes (GSI + LSI)

### Local Secondary Index (LSI)

- Alternative Sort Key for your table (same Partition Key as that of base table)
- The Sort Key consists of one scalar attribute (string/number/binary)
- Uo to 5 Local Secondary Index per table
- Must be defined at table creation time
- Attribute Projections - can contain some or all the attributes of the base table (KEYS_ONLY, INCLUDE, ALL)

### Global Secondary Index (GSI)

- Alternative Primary Key (HASH or HASH+RANGE) from the base table
- Speed up queries on non-key attributes
- The Index Key consists of scalar attributes (string/number/binary)
- Attribute Projects: some or all the attributes of the base table (KEYS_ONLY, INCLUDE, ALL)
- Must provision RCUs and WCUs for the index
- Can be added/modified after table creation

## Indexes and Throttling

- Global Secondary Index (GSI)
  - If the writes are throttled on the GSI, then the main table will be throttle, even if the WCU on the main table are fine
  - Choose your GSI partition key carefully, assigned your WCU capacity carefully
- Local Secondary Index (LSI)
  - Uses the WCUs and RCUs of the main table
  - No special throttling considerations

## PartiQL

- Use a SQL-like syntax to manipulate DynamoDB tables
- Support some (but not all) statements
  - INSERT
  - UPDATE
  - SELECT
  - DELETE
- It supports Batch operations

## Optimistic Locking

- DynamoDB has a feature called “Conditional Writes”
- A strategy to ensure an item hasn’t changed before you update/delete it
- Each item has an attribute that acts as a version number

## DAX (DynamoDB Accelerator)

- Fully-managed, highly available, seamless in-memory cache for DynamoDB
- Microseconds latency for cached reads & queries
- Doesn’t require application logic modification (compatible with existing DynamoDB APIs)
- Solve the “Hot Key” problem (too many readS)
- 5 minutes TTL for cache (default)
- Up to 10 nodes in the cluster
- Multi-AZ (3 nodes minimum recommended for production)
- Secure (Encryption at rest with KMS, VPC, IAM, CloudTrail…)

## Streams

- Ordered stream of item-level modifications (create/update/delete) in a table
- Stream records can be:
  - Sent to Kinesis data Streams
  - Read by AWS Lambda
  - Read by Kinesis Client Library applications
- Data Retention for up to 24 hours
- Use cases
  - React to changes in real-time (welcome email to users)
  - Analytics
  - Insert into derivative tables
  - Insert into OpenSearch Service
  - Implement cross-region replication

## DynamoDB Streams

- Ability to choose the information that will be written to the stream
  - KEYS_ONLY: only the key attribute of the modified item
  - NEW_IMAGE: the entire item, as it as spears after it was modified
  - OLD_IMAGE: the entire item, as it appeared before it was modified
  - NEW_AND_OLD_IMAGES: both the new and the old images of the item
- DynamoDB Streams are made of shards, just like Kinesis Data Streams
- You don’t provision shards, that is automated by AWS
- Records are not retroactively populated in a stream after enabling it

### DynamoDB Streams & AWS Lambda

- You need to define an Event Source Mapping to read from a DynamoDB Stream
- You need to ensure the Lambda function has the appropriate permissions
- You Lambda function is invoked synchronously

## TTL (Time To Live)

- Automatically delete items after an expiry timestamp
- Doesn’t consumer any WCUs (ie. No extra cost)
- The TTL attribute must be a number data type with Unix Epoch timestamp value
- Expired items detected with 48 hours of expiration
- Expired items that haven’t been deleted, appears in reads/queries/scans (if you do’t want them, filter them out)
- Expired items are deleted from both LSIs and GSIs
- A delete operation for each expired item enters the DynamoDB Streams (can help recover expired items)
- Use cases: reduce stored data by keeping only current items, adhere to regulatory obligations

## CLI

- —project-expression: one or more attributes to retrieve
- —filter-expression: filter items before returned to you
- Generate AWS CLI Pagination options (eg. DynamoDB, S3…)
  - —page-size: specify that AWS CLI retrieves the full list of items but with a larger number of API calls instead of one API call (Default: 1000 items)
  - —max-times: max. Number of items to show in the CLI (returns NextToken)
  - —starting-token: specify the last NextToken to retrieve the next set of items

## Transactions

- Coordinated, all-or-nothing operations (add/updated/delete) to multiple items across one or more tables
- Provides atomicity, consistency, isolation and durability (ACID)
- Read mods: eventual consistency, strong consistency, transactional
- Write modes: standard, transactional
- Consumes 2x WCUs & RCUs
  - DynamoDB performs 2 operations for every item
- Two operations
  - TransactGetItems: one or more GetItem operations
  - TransactWriteItems: one or more PutItem, UpdateItem and DeleteItem operations
- Use cases: financial transactions, managing orders, multiplayer games

## Capacity Computations

- Three transactional writes per second with item size 5KB: 3 _ (5kb/1kb) _ 2(transactional cost) = 30WCUs
- Five transaction reads per second with item size 5KG: 5 _ (8KB/4KB) _ 2(transactional cost) = 20 RCUs

## DynamoDB as Session State Cache

- It’s common to use DynamoDB to store session state
- Vs. ElastiCache
  - ElastiCache is in-memory but DynamoDB is server less
  - Both are key/value stores
- Vs. EFS
  - EFS must be attached to EC2 instances as a network drive
- Vs. EBS & Instance Store
  - EBS & Instance Store can only be used for local caching, not shared caching
- Vs. S3
  - S3 is higher latency and not mean for small objects

## Partitioning Strategies (DynamoDB Write Sharding)

- Imagine we have a voting application with two candidates, candidate A and candidate B
- If Partition Key is “Candidate_ID”, this result into two partition, which will generate issues (eg. Hot partition)
- A strategy that allows better distribution of times evenly across partitions
- Add a suffix to Partition Key value
- Two methods:
  - Sharding Using Random Suffix
  - Sharding Using Calculated Suffix

## Operations

- Table cleanup
  - Option 1: Scan + DeleteItem
    - Very slow, consumes RCU & WCU, expensive
  - Option 2: Drop Table + Recreate table
    - Fast, efficient, cheap
- Copying A DynamoDB Table
  - Option 1: Using AWS Data Pipeline
  - Option 2: Backup and restore into a new table
    - Takes some time
  - Option 3: Scan + PutItem or BatchWriteItem
    - Write your own code

## Security & Other

- Security
  - VPC endpoints available to access DynamoDB without using the Internet
  - Access fully controlled by IAM
  - Encryption at rest using AWS KMS and in-transit using SSL/TLS
- Backup and restore feature available
  - Point-in-time Recovery (PITR) like RDS
  - No performance impact
- Global Tables
  - Multi-region, multi-active, fully replicated, high performance
- DynamoDB Local
  - Develop and test apps locally without accessing the DynamoDB web service (without Internet)
- AWS Database Migration Service (AWS DMS) can be used to migrate to DynamoDB (from MongoDB, Oracle, MYSQL, S3 etc.)

## Fine-Grained Access control

- Using Web Identity Federation or Cognition Identity Pools, each user gets AWS credentials
- You can assign n IAM Role to these users with a Condition to limit their API access to DynamoDB
- LeadingKeys: limit row-level access for users on the Primary Key
- Attributes: limit specific attributes the user can see
