# Other AWS Services

## AWS SES (Simple Email Service)

- Send emails to people using SMTP interface or AWS SDK
- Ability to receive email. Integrates with:
  - S3
  - SNS
  - Lambda
- Integrated with IAM for allowing to send emails

## Amazon OpenSearch Service

- Amazon OpenSearch is successor to Amazon ElasticSearch
- In DynamoDB, queries only exist by primary key or indexes
- With OpenSearch, you can search any field, even partially matches
- It’s common to use OpenSearch as a complement to another database
- Two modes: managed cluster or serverless cluster
- Does not natively support SQL (can be enabled via a plugin)
- Ingestion from Kinesis Data Firehose, AWS IoT and CloudWatch Logs
- Security through Cognito and IAM, KMS encryption, TLS
- Comes with OpenSearch Dashboards (visualisation)

## Amazon Athena

- Serverless query service to analyse data stored in Amazon S3
- Uses standard SQL language to query the files (built on Presto)
- Supports CSV, JSON, ORC, Avro and Parquet
- Pricing: $5 per TB of data scanned
- Commonly used with Amazon Quicksight for reporting/dashboards
- Use cases: business intelligence/analytics/reporting, analyse and query VPC Flow Logs, ELB Logs, CloudTrail trails etc.
- Exam tip: analyse data in S3 using serverelss SQL, use Athena

## Performance Improvement

- Use columnar data for cost-savings (less scan)
  - Apache Parquet or ORC ir commended
  - Huge performance improvement
  - Use Glue to convert your data to Parquet or ORC
- Compress data for smaller retrievals (zip, gzip, Iz4, snappy, slip, std)
- Partition datasets in S3 for easy query on virtual columns
- Use larger files (> 128MB) to minimise overhead

## Federated Query

- Allows you to run SQL queries across data stored in relational, non-relational object and customer data spruces (AWS or on-premises)
- Uses Data Source Connectors that run on AWS Lambda to run Federated Queries (eg. CloudWatch Logs, DynamoDB, RDS etc.)
- Store the results back in Amazon S3

## Amazon MSK (Amazon Managed Streaming for Apache Kafka)

- Alternative to Amazon Kinesis
- Fully managed Apache Kafka on AWS
  - Allow you to create, update, delete clusters
  - MSK creates and manages Kafka brokers nodes and Zookeeper nodes for you
  - Deploy the MSK cluster in your VPC, multi-AZ (ump to 3 for HA)
  - Automatic recovery from common Apache Kafka failures
  - Data is stored in EBS volumes for as long as you want
- MSK Serverless
  - Run Apache Kafka on MSK without managing the capacity
  - MSK automatically provisions resource and scales compute and storage

Kinesis Data Streams vs. Amazon MSK
Kinesis Data Streams Amazon MSK
1MB message size limit 1MB default, configure for higher (eg. 10MB)
Data Streams with Shards Kafka topics with partitions
Shard splitting and merging PLAINTEXT or TLS In-Flight encryption
TLS in-flight encryption Can only add partitions to a topic
KMS at-rest encryption KMS at-rest encryption

## Amazon Certificate Manager (ACM)

- Let’s you easily provision, manage and deploy SSL/TLS certificates
- Used to provide in-slight encryption for websites (https)
- Supports both public and private TLS certificates
- Free of charge for public TLS certificates
- Automatic TLS certificate renewal
- Integrations with (load TLS certificates on)
  - Elastic Load Balancers
  - CloudFront Distributions
  - APIs on API Gateway

## ACM Private CA (AWS Private Certificate Authority)

- Managed service allows you to create private Certificate Authorities (CA), including root and subordinaries CA
- Can issue and deploy end-entity X.509 certificates
- Certificates are trust only by your organisation (not the public Internet)
- Works for AWS services that are integrated with ACM
- Use cases
  - Encrypted TLS communication, Cryptographically signing code
  - Authenticate users, computers, API endpoints and IoT devices
  - Enterprise customers building a Public Key Infrastructure (PKI)

## Amazon Macie

- Amazon Macie is a fully managed data security and data privacy service that uses machine learning and pattern matching to discover and protect your sensitive data in AWS
- Macie helps identify and alert you to sensitive data, such as personally identifiable information (PII)

## AWS AppConfig

- Configure, validate and deploy dynamic configurations
- Deploy dynamic configuration changes to your application independently of any code deployments
  - You don’t need to restart the application
- Feature flags, application turning, allow/block listing
- Use the apps on EC2 instances, Lambda, ECS, EKS
- Gradually deploy the configuration changes and rollback if issues occur
- Validate configuration changes before deployment using:
  - JSON Schema (syntactic check) or Lambda Function (run code to perform validation - semantic check)

## CloudWatch Evidently

- Safely validate new features by serving them to a specified % of your answers
  - Reduce risk and identify unintended consequences
  - Collect experiment data, analyse using stats, monitor performance
- Launches (feature flags): enable and disable features for a subnet of users
- Experiments (A/B testing): compare multiple versions of the same feature
- Overrides: pre-define a variation of a specific user
- Store evaluation events in CloudWatch Logs or S3
