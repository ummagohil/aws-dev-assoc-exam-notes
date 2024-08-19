# Amazon S3

- Amazon S3 is one of the main building blocks of AWS
- It’s advertised as “infinitely scaling” storage
- Many websites use Amazon S3 as a backbone
- Many AWS services use Amazon S3 as an integration tool too

## Use Cases

- Backup and storage
- Disaster Recovery
- Archive
- Hybrid Cloud storage
- Application hosting
- Media hosting
- Data lakes and big data analytics
- Software Delivery
- Static website

## Buckets

- Amazon S3 allows people to store objects (files) in “buckets” (directories)
- Buckets must have a globally unique name (across all region all accounts)
- Buckets are defined at the region level
- S3 looks like a global service but buckets are created in a region
- Naming convention
  - No upper case, no underscore
  - 3-63 characters long
  - Not an IP
  - Must start with lowercase letter or number
  - Must not start with the prefix xn—
  - Must not end with the suffix -s3alias

## Objects

- Objects (files) have a Key
- The key is the FULL path
  - s3://my-bucket/my_file.txt
  - s3://my-bucket/my_folder1/another_folder/my_file.txt
- The key is composed of prefix + object name
  - s3://my-bucket/my_folder/my_folder_two/my_filter.txt
- There’s no concept of “directories” within buckets
  - Although the UI will trick you into thinking otherwise
- Just keys with very long names that contain slashes (“/“)
- Object values are the content of the body
  - Max object size is 5TB (5000GB)
  - If uploading more than 5GB, must use “multi-part upload”
- Metadata (list of text key/value pairs - system or user metadata)
- Tags (unicode key/value pair - up to 10) - useful for security/lifecycle
- Version ID (if versioning is enabled)

## S3 Security

- User-based
  - IAM policies: which API calls should be allowed for a specific user from IAM
- Resource-based
  - Bucket policies: bucket wide rules from the S3 console - allows cross account
  - Object access control list (ACL): finer grain (can be disabled)
  - Bucket access control list (ACL): less common (can be disabled)
- Note: an IAM principal can access an S3 object if:
  - The user IAM permissions ALLOW it OR the resource policy ALLOWS it
  - AND there’s not explicit DENY
- Encryption: encrypt objects in Amazon S3 using encryption keys

## Bucket Policy

- JSON based policies
  - Resources: buckets and objects
  - Effect: allow/deny
  - Actions: set of API to allow/deny
  - Principal: the account or user to apply the policy to
- Use S3 bucket for policy to:
  - Grant public access to the bucket
  - Force objects to be encrypted to upload
  - Grant access to another account (cross account)

## S3 Website

- S3 can host static websites and have them accessible on the Internet
- The website URL will be (depending on the region)
  - http://bucket-name.s3-website-aws-region.amazonaws.com
  - OR
  - http://bucket-name.s3-website.aws-region.amazonaws.com
- If you get a 403 Forbidden error, make sure the bucket policy allows public reads

## S3 Versioning

- You can version your files in Amazon S3
- It Is enabled at bucket level
- Same key overwrite will change the version 1, 2, 3
- It is best practice to version your buckets
  - Protect against unintended deleted (ability to restore a version)
  - Easy roll back to previous version
- Any file is not versioned prior to enabling versioning will have a version “null”
- Suspending versioning does not delete the previous versions

## S3 Replication

- Must enable versioning in source and destination buckets
- Cross-region replication (CRR)
- Same-region replication (SRR)
- Buckets can be different AWS accounts
- Copying is asynchronous
- Must give proper IAM permissions to S3
- After you enable Replication, only new objects are replicated
- Optionally, you can replicate existing objects using S3 Batch Replication
  - Replicates existing objects and objects that failed replication
- For DELETE operations
  - Can replicate delete markers from source to target (optional setting)
  - Deletions with a version ID are not replicated (to avoid malicious deletes)
- There is no “chaining” of replication
  - If the bucket one has replication into bucket two, which has replication into bucket three, then objects created in bucket one are not replicated to bucket three

### Use Cases

- CRR: compliance, lower latency access, replication across accounts
- SRR: log aggregation, live replication between production and test accounts

## S3 Storage Classes

- Amazon S3 Standard - general purpose
- Amazon S3 Standard -Infrequent Access (IA)
- Amazon S3 One Zone-Infrequent Access
- Amazon S3 Glacier Instant Retrieval
- Amazon S3 Glacier Flexible Retrieval
- Amazon S3 Glacier Deep Archive
- Amazon S3 Intelligent tTiering
- Can move between classes manually or using S3 Lifecycle configurations

## Durability and Availability

- Durability:
  - High durability (99.99999999999% 11 9’s) of objects across multiple AZ
  - If you store 10,000,000 objects with Amazon S3, you can on average expect to incur a loss of a single object oncer ever 10,000 years
  - Same for all storage classes
- Availability:
  - Measures how readily available a service is
  - Varies depending on storage class
  - Example: S3 standard has 99.99% availability = not available 53 minutes a year

## S3 Standard - General Purpose

- 99.99% availability
- Used for frequently access data
- Low latency and high throughput
- Sustain 2 concurrent facility failures
- Use cases: big data analytics, mobile and gaming applications, content distribution etc.

## S3 Infrequent Access

- For data that is less frequently accessed but requires rapid access when needed
- Lower cost than S3 Standard
- Amazon S3 Standard-Infrequent Access (S3 Standard-IA)
  - 00.99% availability
  - Use cases: disaster recovery, backups
- Amazon S3 One Zone-Infrequent Access (S3 One Zone-IA)
  - High durability (99.99999999999%) in a single AZ, data lost when AZ is destroyed
  - 99.5% availability
  - Use cases: storing secondary backup copes of on-premise data or data you can recreate

## S3 Glacier Storage

- Low-cost object storage meant for archiving/back up
- Pricing: price for storage + object retrieval cost
- Amazon S3 Glacier Instant Retrieval
  - Millisecond retrieval, great for data accessed once a quarter
  - Minimum storage duration of 90 days
- Amazon S3 Glacier Flexible Retrieval (formerly Amazon S3 Glacier)
  - Expedited (1 to 5 mins), Standard (3 to 5 hours), Bulk (5 to 12 hours) - free
  - Minimum storage duration of 90 days
- Amazon S3 Glacier Deep Archive - for long term storage
  - Standard (12 hours), Bulk (48 hours)
  - Minimum storage duration of 180 days

## S3 Intelligent-Tiering

- Small monthly monitoring and auto-tiering fee
- Moves objects automatically between Access Tiers based on usage
- There are no retrieval charges in S3 Intelligent-Tiering
- Frequent Access tier (automatic): default tier
- Infrequent Access tier (automatic): objects not accessed for 30 days
- Archive Instant Access tier (automatic): objects not accessed for 90 days
- Archive Access tier (optional): configurable from 90 days to 700+ days
- Deep Archive Access tier (optional): configurable from 190 days to 700+ days
