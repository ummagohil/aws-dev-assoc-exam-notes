# AWS Fundamentals: RDS + Aurora + ElastiCache

## Amazon RDS

- RDS stands for Relational Database Service
- It’s a managed DB service for DB using SQL as a query language
- It allows you to create databases in the cloud that are managed by AWS
  - Postgres
  - MySQL
  - MariaDB
  - Oracle
  - Microsoft SQL Server
  - IBM DB2
  - Aurora (AWS Proprietary database)

## Advantage Over Using RDS vs Deploying DB on EC2

- RDS is a managed service:
  - Automated provisioning, OS patching
  - Continuous backups and restore to specific timestamp (point in time restore)
  - Monitoring dashboards
  - Read replicas for improved read performance
  - Multi AZ set up for DR (Disaster recovery)
  - Maintenance windows for upgrades
  - Scaling capability (vertical and horizontal)
  - Storage backed by EBS (gp2 or io1)
  - Note: you can’t SSH into your instances

## Storage Auto Scaling

- Helps you increase storage on your RDS DB instance dynamically
- When RDS detects you are running out of free database storage, it scales automatically
- Avoid manually scaling your database storage
- You have to set a Maximum Storage Threshold (max limit for DB storage)
- Automatically modify storage if:
  - Free storage is less than 10% of allocated storage
  - Low-storage lasts at least 5 minutes
  - 6 hours have passed since last modification
- Useful for applications with unpredictable workloads
- Supports all RDS database engines

## RDS Read Replicas vs Multi AZ

### Read Scalability

- Up to 15 read replicas
- Within AZ, Cross AZ or Cross Region
- Replication is ASYNC, so reads are eventually consistent
- Replicas can be promoted to their own DB
- Applications must update the connection string to leverage read replicas

### Use Cases

- You have a production database that is taking on normal load
- You want to run a reporting application to run some analytics
- You create a Read Replica to run the new workload there
- The production application is unaffected
- Read replicas are used for SELECT (read) only kind of statements (not INSERT/UPDATED/DELETE)

### Network Cost

- In AWS there’s a network cost when data goes from one AZ to another
- For RDS Read Replicas within the same region, you don’t pay a fee

### Multi AZ (Distater Recovery)

- SYNC replication
- One DNS name - automatic app failover to standby
- Increase availability
- Failover in case of loss of AZ, loss of network, instance or storage failure
- No manual intervention in apps
- Not used for scaling
- Note: the read replicas can be set up as multi AZ for Disaster Recovery (DR)

### From Single-AZ to Multi-AZ

- Zero downtime operation (no need to stop the DB)
- Just click on “modify” for the database
- The following happens internally:
  - A snapshot is taken
  - A new DB is restored from the snapshot in a new AZ
  - Synchronisation is established between the two databases

## Amazon Aurora

- Aurora is a proprietary technology from AWS (not open source)
- Postgres and MySQL are both supported as Aurora DB (that means your drivers will work as if Aurora was a Postgres or MySQL database)
- Aurora is “AWS Cloud Optimised” and claims 5x performance improvement over MySQL on RDS, over 3x the performance of Postgres on RDS
- Aurora storage automatically grows in increments of 10GB, up to 128 TB
- Aurora can have up to 15 replicas and the replication process is faster than MySQL (sub 10 ms replica lag)
- Failover in Aurora is instantaneous - its HA native
- Aurora costs more than RDS (20% more) but it’s more efficient

## High Availability and Read Scaling

- 6 copies of your data across 3AZ:
  - 5 copies out of 6 needed for writes
  - 3 copies out of 6 needed for reads
  - Self healing with peer-to-peer replication
  - Storage is striped across 100s of volume
- One Aurora Instance takes writes (master)
- Automated failover for master in less than 30 seconds
- Master + up to 15 Aurora Read Replicas serve reads
- Support for Cross Region Replication

### Features

- Automatic fail-over
- Backup and recovery
- Isolation and security
- Industry compliance
- Push-button scaling
- Automated patching with zero downtime
- Advanced monitoring
- Routine maintenance
- Backtrack: restore data at any point of time without using backups

## RDS & Aurora Security

- At-rest encryption:
  - Database master & replicas encryption using AWS KMS - must be defined as launch time
  - If the master is not encrypted, the read replicas cannot be encrypted
  - To encrypt an un-encrypted database, go through a DB snapshot and restore as encrypted
- In-flight encryption: TLS-ready by default use the AWS TLS root certificates client-side
- IAM authentication: IAM roles to connect your database (instead of using username and password)
- Security groups: control network access to your RDS/Aurora DB
- No SSH available expect on RDS Custom
- Audit Logs can be enabled and sent to CloudWatch Logs for longer retention

## RDS Proxy

- Fully managed databases proxy for RDS
- Allows apps to pool and share DB connections established with the database
- Improving database efficiency by reducing the stress on database resources (Eg. CPU, RAM) and minimise open connections (and timeouts)
- Serverless, autoscaling, highly available (multi-AZ)
- Reduced RDS & Aurora failover time by 66%
- Supports RDS (MySQL, PostgresSQL, MariaDB, MS SQL Server) and Aurora (MySQL, PostgreSQL)
- No code changes required for most apps
- Enforce IAM Authentication for DB, and securely store credentials in AWS Secrets Manager
- RDS Proxy is never publicly accessible (must be accessed from VPC)

## ElastiCache

- The same way RDS is to get managed Relational Databases
- ElastiCache is to get managed Redis or Memcached
- Caches are in-memory databases with really high performance, low latency
- Helps reduce load off of databases for read intensive workloads
- Helps make your application stateless
- AWS takes care of OS maintenance/pathing, optimisations, set up, configurations, monitoring, failure recovery and backups
- Using ElastCache involves heavy application code changes

## DB Cache

- Application queries ElastiCache, if not available, get from RDS and store in ElastiCache
- Helps relieve load in RDS
- Cache must have an invalidation strategy to make sure only the most current data is used in there

## Solution Architecture - User Session Store

- User logs into nay of the applications
- The application writes the session data into ElastiCache
- The user hits another instance of our application
- The instance retrieves the data and the user is already logged in

## Redis vs Memcached

- Redis
  - Multi AZ with Auto-Failover
  - Read Replicas to scale reads and have high availability
  - Data Durability using AOF persistence
  - Backup and restore features
  - Supports Sets and Sorted Sets
- Memcached
  - Multi-node for partitioning data (sharding)
  - No high availability (replication)
  - No backup and restore
  - Multi-threaded architecture

## ElastiCache Strategies

### Caching Implementation Considerations

- Is it safe to cache data? Data maybe out of date, eventually inconsistent
- Is caching effective for that data?
  - Pattern: data changing slowly, few keys are frequently needed
  - Anti patterns: data changing rapidly, all large key space frequently needed
- Is data structured well for caching?
  - Example: key value caching, or caching of aggregation results

### Lazy Loading/Cache-Aside/Lazy Population

- Pros
  - Only requested data is cached (the cache isn’t filled up with unused data)
  - Node failures are not fatal (just increased latency to warm the cache)
- Cons
  - Cache miss penalty that results in three round trips, noticeable delay for that request
  - Stale data: data can be updated in the database and outdated in the cache

### Write Through - Add or Update cache when database is updated

- Pros
  - Data in cache is never stale, reads are quick
  - Write penalty vs Read penalty (each write requires 2 calls)
- Cons
  - Missing Data until it is added/updated in the DB. Mitigation is to implement Lazy Loading strategy as well
  - Cache churn - a lo too the data will never be read

### Cache Evictions and Time-to-Live (TTL)

- Cache eviction can occur in three ways:
  - You delete the item explicitly in the cache
  - Item is evicted because the memory is full and it’ snot recently used (LRU)
  - You set an item time-to-live (TTL)
- TTL are helpful for any kind of data
  - Leaderboards
  - Comments
  - Activity streams
- TTL can range from few seconds to hours or days
- If too many evictions happen due to memory, you should scale up or out

### In a Nutshell

- Lazy loading/cache aside is easy to implement and works for many situations as a foundation, especially on the read side
- Write-through is usually combined with Lazy Loading as targeted for the queries or workloads that benefit from this optimisation
- Setting a TTL is usually not a bad idea, expect when you’re using Write-through. Set it to a sensible value for your application
- Only cache the data that makes sense

## Amazon MemoryDB for Redis

- Redis-compatible, durable, in-memory database service
- Ultra-fast performance with over 160 millions requested/second
- Durable in-memory data storage with Multi-AZ transactional log
- Scale seamlessly from 10s GBs to 100s TBs of storage
- Use cases: web and mobile apps, online gaming, media, streaming etc.
