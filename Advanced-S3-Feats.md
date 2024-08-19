# Advanced Amazon S3

## S3 Lifecycle Rules (with S3 Analytics)

### Moving Between Storage Classes

- You can transition objects between storage classes
- For infrequently accessed objects, move them to Standard IA
- For archive objects that you don’t need fast access to, move them Glacier or Glacier Deep Archive
- Moving objects can be automated using a Lifecycle Rule

### Lifecycle Rules

- Transition Actions: configure objects to transition to another storage class
  - Move objects to Standard IA class 60 days after creation
  - Move to Glacier for archiving after 6 months
- Expiration actions: configure objects to expire (delete) after some time
  - Access log files can be set to delete after 365 days
  - Can be used to delete old versions of files (if versioning is enabled)
  - Can be used to delete incomplete multi-part uploads
- Rules can be created a for certain prefix (example: s3//mybucket.mp3/\*)
- Rules can be created for certain objects Tags (example: Department: Finance)

## Storage Class Analysis

- Help you decide when to transition objects to the right storage class
- Recommendations for Standard and Standard IA
  - Does not work for One-Zone IA or Glacier
- Report is updated daily
- 24 to 48 hours to start seeing data analysis
- Good first step to put together Lifecycle rules (or improve them)

## S3 Event Notifications

- S3:ObjectCreated, S3:ObjectRemoved, S3:ObjectRestore, S3:Replication
- Object name filtering possible (\*.jpg)
- Use case: generate thumbnails of images uploaded to S3
- Can create as many “S3 eventS” as desired
- S3 event notifications typically deliver events in seconds but can sometimes take a minute or two longer
- Need a Lambda/SNS/SQS resource function attached to the policy to enable IAM permissions

### With Amazon EventBridge

- Advanced filtering options with JSON rules (metadata, object size, name)
- Multiple destinations: eg. Step Functions, Kinesis Streams/Firehose
- EventBridge capabilities: archive, replay events, reliable delivery

## S3 Performance

### Baseline Performance

- Amazon S3 automatically scales to high request rates, latency 100-200ms
- You application can achieve at least 3,500 PUT/COPY/POST/DELETE or 5,500 GET/HEAD requests per second per prefix in a bucket
- There is no limits to the number of prefixes in a bucket
- Example (object path => prefix)
  - Bucket/folder1/sub1/file => /folder1/sub1/
  - Bucket/folder/sub2/file => /folder1/sub2/
  - Bucket/1/file => /1/
  - Bucket/2/file => /2/
- If you spread reads across all four prefixes evenly, you can achieve 22,000 requests per second for GET and HEAD

### Multi-Part Upload

- Recommended for files > 100MB, must use for files > 5GB
- Can help parallelise uploads (speed up transfers)

### S3 Transfer Acceleration

- Increase transfer speed by transferring file to an AWS edge location which will forward the data to the S3 bucket in the target region
- Compatible with multipart upload

### S3 Byte-Range Fetches

- Parallelise GETs by requesting specific byte ranges
- Better resilience in case of failures
- Can be used to speed up downloads
- Can be used to retrieve only partial data (for example the head of a file)

### S3 Select & Glacier Select

- Retrieve less data using SQL by performing server-side filtering
- Can filter by rows and columns (simple SQL statements)
- Less network transfer, less CPU cost client-side

## S3 Object Tag & Metadata

### S3 User-Defined object Metadata

- When uploading an object, you can also assign metadata
- Name-value (key-value) pairs
- Use-defined metadata names must be begin with “x-amz-meta-“
- Amazon S3 stores user-defined metadata keys in lowercase
- Metadata can be retrieved while retrieving the object

### S3 Object Tags

- Key-value Paris for objects in Amazon S3
- Useful for fine-grained permissions (only access specific objects with specific tags)
- Useful for analytics purposes (using S3 Analytics to group by tags)

- You cannot search the object metadata or object tags
- Instead, you must use an external DB as a search index such as DynamoDB
