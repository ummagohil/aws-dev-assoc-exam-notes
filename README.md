# AWS Developer Associate Exam Notes

# Advanced Identity

## AWS STS - Security token Service

- Allows you to grant limited ad temporary access to AWS resources (up to one hour)
- AssumeRole: Assume roles within your account or cross account
- AssumeRoleWithSAML: return credentials for users logged in with SAML
- AssumeRoleWithWebIdentity
  - Return creds for users logged with an IdP (Facebook, Google, OIDC compatible)
  - AWS recommends against using this, and using Cognito Identity Pools instead
- GetSessionToken: for MFA, from a user or AWS account root user
- GetFederationToken: obtain temporary creds for a federated user
- GetCallerIdentity: return details about the IAM user or role used in the API call
- DecodeAuthorisationMessage: decode error message when AWS API is denied

## Using STS to Assume a Role

- Define an IAM Role within your account or cross-account
- Define which principals can access this IAM Role
- Use AWS STS (security token service) to retrieve credential sand impersonate the IAM Role you have access to (AssumeRole API)
- Temporary credentials can be valid between 15 minutes to 1 hour

## STS with MFA

- Use GetSessionToken from STS
- Appropriate IAM policy using IAM Conditions
- aws:MultiFactorAuthPresent:true
- Reminder: Get SessionToken returns access ID, secret key, session token, expiration date

## Advanced IAM

### Authorisation Model Evaluation of Policies

1. If there’s an explicit DENy, end decision and DENY
2. If there’s an ALLOW, end decision with ALLOW
3. Else DENY

### IAM Policies and S3 Bucket Policies

- IAM Policies are attached to users, roles, groups
- S3 Bucket Policies are attached to buckets
- When evaluating if an IAM Principal can perform an operation X on a bucket, the union of its assigned IAM Policies and S3 Bucket Policies will be evaluated

### Dynamic Policies with IAM

- How do you assigned each user a /home/<user> folder pin an S3 bucket?
- Option one
  - Create am IAM policy allowing each user to have accent to /home/<user-name>
  - Not very scalable
- Option two
  - Create one dynamic policy with IAM
  - Leverage the special policy variable ${aws:username}

### Inline vs Managed Policies

- AWS Managed Policy
  - Maintained by AWS
  - Good for power users and admins
  - Updated in case of new services/new APIs
- Customer Managed Policy
  - Best practice, reusable, can be applied to many principals
  - Version controlled + rollback, central change management
- Inline
- Strict one-to-one relationship between policy and principal
- Policy is delete Dif you delete the IAM principal

### Granting a User Permissions to pass a Role to an AWS Service

- To configure many AWS services, you must pass an IAM role to the service (this happens only once during setup)
- The service will later assume the role and perform actions
- Example of passing a role
  - To an EC2 instance, a Lambda function, an ECS task, CodePipeline to allow it to invoke other services
- For this, you may need the IAM permission iam:PassRole
- It often comes with iam:GetRole to view the role being passed
- Can a role be passed to any service?
  - No: roles can only be passed to what their trust allows
  - A trust policy for the role that allows the service to assume the role

## AWS Directory Services

- AWS Managed Microsoft AD
  - Create your own AD in AWS, manage users locally, supports MFA
  - Establish “trust” connections with your on premise AD
- AD Connector
  - Directory Gateway (proxy) to redirect to on-premise AD, supports MFA
- Simple AD
  - AD-compatible managed directory on AWS
  - Cannot be joined with on-premise AD

## What is Microsoft Active Directory (AD)?

- Found on any windows Server with AD Domain Services
- Database of objects: user accounts, computers, printers, file shares, security groups
- Centralised security management, create account, assign permission
- Objects are organised in trees
- A group of trees is a forest

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

# Amazon S3 Security

## S3 Encryption

### Object Encryption

- You can encrypt objects in S3 buckets using one of 4 methods
- Server-Side Encryption (SSE)
  - Server-side encryption with Amazon S3-Managed Keys (SSE-S3) - enabled by default
    - Encrypts S3 objects using keys handled, managed and owned by AWS
  - Service-Side Encryption with LMS Keys stored in AWS KMS (SSE-KMS)
    - Leverage AWS Key Management Service (AWS KMS) to manage encryption keys
  - Server-Side Encryption with Customer-Provided Keys (SSE-C)
    - When you want to manage your own encryption keys
- Client-Side Encryption

## SSE-S3

- Encryption using keys handled, managed and owned by AWS
- Object is encrypted server-side
- Encryption type is AES-256
- Must set header “x-amz-server-side-encryption”:”AES256”
- Enabled by default for new buckets & new objects

## SSE-KMS

- Encryption using keys handled and managed by AWS KMS (Key Management Service)
- KMS advantages: user control + audit key usage using CloudTrail
- Object is encrypted server side
- Must set header “x-amz-server-side-encryption”:”as:kms”
- Limitations:
  - If you see SSE-KMS, you may be impacted by the KMS limits
  - When you upload, it calls the GenerateDataKey KMS API
  - When you download, it calls the Decrypt KMS API
  - Count towards the KMS quote per second (5500, 10000, 30000 req/s based on region)
  - You can request a quote increase using the Service Quotas Console

## SSE-C

- Server-Side Encryption using keys fully managed by the customer outside of AWS
- Amazon S3 does not store the encryption key you provide
- HTTPS must be used
- Encryption key must be provided in HTTP headers, for every HTTP request made

## Client-Side Encryption

- Use client libraries such as Amazon S3 Client-Side Encryption Library
- Clients must encrypt data themselves before sending to Amazon S3
- Clients must decrypt data themselves when retrieving from Amazon S3
- Customer fully manages the keys and encryption cycle

## Encryption in Transit (SSL/TLS)

- Encryption in flight is also called SSL/TLS
- Amazon S3 exposes two endpoints
  - HTTP Endpoint: non encrypted
  - HTTPS Endpoint: encryption in flight
- HTTPS is recommended
- HTTPS is mandatory for SSE-C
- Most clients would use the HTTPS endpoint by default

## About DDS-KMS

- “Double encryption based on KMS”

## S3 Default Encryption

### Default Encryption vs. Bucket Policies

- SSE-S3 encryption is automatically applied to new objects stored in S3 bucket
- Optionally, you can “force encryption: using a bucket policy and refuse any API call to PUT an S3 object without encryption headers (SSE-KMS or SSE-C)
- Note: Bucket Policies are evaluated before “Default Encryption”

### S3 CORS

- Cross-Origin Resource Sharing (CORS)
- Origin = scheme (protocol) + host(domain) + port
  - example: (implied port is 443 for HTTPS, 80 for HTTP)
- Web browser based mechanism to allow requests to other origins while visiting the main origin
- Same origin: &
- Different origins: &
- The requests won’t be fulfilled unless the other origin allows for the requests, using CORS Headers (example: Access-Control-Allow-Origin)
- If a client makes a cross-origin request on our S3 bucket, we need to enable the correct CORS headers
- It’s a popular exam question
- You can allow for a specific origin or for \* (all origins)

### S3 MFA Delete

- MFA (multi-factor authentication): force users to generate a code on a device (usually a mobile phone or hardware) before doing important operations on S3
- MFA will be required to:
  - Permanently delete an object version
  - Suspend versioning on the bucket
- MFA won’t be required to:
  - Enable versioning
  - List deleted versions
- To use MFA Delete, versioning must be enabled on the bucket
- Only the bucket owner (root account) can be enable/disable MFA Delete

### S3 Access Logs

- For audit purposes, you may want to log all access to S3 buckets
- Any requests made to S3, from any account, authorised or denied, will be logged into another S3 bucket
- That data can be analysed using data analysis tools
- The target logging bucket must bin the same AWS region
- The log format is at:
- Warning:
  - Do not set your logging bucket to be the monitored bucket
  - It will create a logging loop and your bucket will grow exponentially

### S3 Pre-signed URLs

- Generate pre-signed URLs using the S3 Console, AWS CLI or SDK
- URL Expiration
  - S3 Console - 1 min up to 720 mins (12 hours)
  - AWS CLI - configure expiration with —expires-in parameter in seconds (default is 3600 secs, max is 604800 secs ~168 hours)
- Users given a pre-signed URL inherit the permissions of the user that generated the URL for GET/PUT
- Examples
  - Allow only logged-in users to download a premium video form your S3 bucket
  - Allow an ever-changing list of users to download files by generating URLs dynamically
  - Allow temporarily a user to upload a file to a precise location in your bucket

## S3 Access Points

### VPC Origin

- We can define the access point to be accessible only from within the VPC
- You must create a VPC Endpoint to access the Access Point (Gateway or Interface Endpoint)
- The VPC Endpoint Policy must allow access to the target bucket and Access Point

## S3 Object Lambda

- Use AWS Lambda Function storage change the object before it is retrieved by the caller application
- Only one S3 bucket is needed, on top of which we create S3 Access Point and S3 Object Lambda Access Points
- Use Cases
  - Redacting personally identifiable information for analytics or non-production environments
  - Converting across data formats, such as converting XML to JSON
  - Resizing and watermarking images on the fly using caller-specific details, such as the user who requested the object

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

# AWS Serverless: API Gateway

- AWS Lambda + API Gateway: no infrastructure to manage
- Support for the WebSocket Protocol
- Handles API versioning (v1, v2 etc.)
- Handle different environments (dev, test, prod)
- Handle security (authentication and authorisation)
- Create API keys, handle request throttling
- Swagger/Open API import to quickly define APIs
- Transform and validate requests and responses
- Generate SDK and API specifications
- Cache API responses

## Integrations (High Level)

- Lambda Function
  - Lambda functions are invoked
  - Easy way to expose REST API backed by AWS Lambda
- HTTP
  - Expose HTTP endpoints in the backend
  - Example: internal HTTP API on premise, Application Load Balancer etc.
  - Why? Can add rate limiting, caching, user authentication, API keys etc.
- AWS Service
  - Expose any AWS API through the API Gateway
  - Example: start an AWS Step Function workflow, post a message to SQS
  - Why? Add authentication, deploy publicly, rate control etc.

## Endpoint Types

- Edge-optimised (default): for global clients
  - Requests are routed through the CloudFront Edge locations (improves latency)
  - The API Gateway still lives in only one region
- Regional
  - For clients within the same region
  - Could manually combine with CloudFront (more control over the caching strategies and the distribution)
- Private
  - Can only be accessed from your VPC using an interface VPC endpoint (ENI)

## Security

- User authentication through
  - IAM Roles: useful for internal applications
  - Cognito: identity for external uses - eg. Mobile users
  - Custom authoriser (your own logic)
- Custom Domain Name HTTPS security through integration with AWS Certificate Manage (ACM)
  - If using Edge-Optimised endpoint, then the certificate must be in us-east-1
  - If using Regional endpoint, the certificate must the in the API Gateway region
  - Must setup CNAME or A-alias record in Route 53

## Stages and Deployments

### Deployment Stages

- Making changes in the API Gateway does not mean they’re effective
- You need to make a “deployment” for them to be in effect
- It’s a common source of confusion
- Changes are deployed to “Stages” (as many as you want)
- Use the naming you like for stages (dev, test, prod)
- Each stage has it’s own configuration parameters
- Stages can be rolled back as a history of deployment is kept

### Stage Variables

- Stage variables are like environment variables for API Gateway
- Use them to change often changing configuration values
- They can be used in:
  - Lambda function ARN
  - HTTP Endpoint
  - Parameter mapping templates
- Use cases
  - Configure HTTP endpoints your stages talk to (dev, test, prod, etc.)
  - Pass configuration parameters to AWS Lambda through mapping templates
- Stage variables are passed to the “context” object in AWS Lambda
- Format: ${stageVariables.variableName}

### API Gateway Stage Variables and Lambda Aliases

- We create a stage variable to indicate the corresponding Lambda alias
- Our API gateway will automatically invoke the right Lambda function

### Canary Deployments

- Possibility to enable canary deployments for any stage (usually prod)
- Choose the % of traffic the canary channel receives
- Metrics and logs are separate (for better monitoring)
- Possibility to override stage variables for canary
- This is blue/green deployment with AWS Lambda & API Gateway

## Integration Types and Mappings

- Integration Type MOCK
  - API Gateway returns a response without sending the request to the backend
- Integration Type HTTP/AWS (Lambda & AWS Services)
  - You must configure both the integration request and integration response
  - Set up data mapping using mapping templates for the request and response
- Integration Type AWS_PROXY (Lambda Proxy)
  - Incoming request from the client is the input to Lambda
  - The function is responsible for the logic of request/response
  - No mapping template, headers, query, string parameters are passed as arguments
- Integration type HTTP_PROXY
  - No mapping template
  - The HTTP request is passed to the backend
  - The HTTP response from the backend is forward by API Gateway
  - Possibility to add HTTP Headers if need be (eg. API key)

### Mapping Templates (AWS & HTTP Integration)

- Mapping templates can be used to modify request/responses
- Rename/modify query string parameters
- Modify body content
- Add headers
- Uses Velocity Template Language (VTL)
- Filter output results (remove unnecessary data)
- Content-Type can be set to application/json or application/xml

### Mapping Example: JSON to XML with SOAP

- SOAP API are XML based, whereas REST API are JSON based
- In this case, API Gateway should:
  - Extract data from the request (either path, payload or header)
  - Build SOAP message based on request data (mapping template)
  - Call SOAP service and receive XML response
  - Transform XML response to desired format (like JSON) and respond to the user

### Open API

- Common way of defining REST APIs, using API definition as code
- Import existing OpenAPI 3.0 spec to API Gateway
  - Method
  - Method Request
  - Integration Request
  - Method Response
  - - AWS extensions for API gateway and setup every single option
- Can export current API on OpenAPI spec
- OpenAPI specs can be written in YAML or JSON
- Using OpenAPI we can generate SDK for our applications

### REST API: Request Validation

- You can configure API Gateway to perform basic validation of an API request before proceeding with the integration request
- When the validation fails, API Gateway immediately fails the request
  - Returns a 400-error response to the caller and this reduces unnecessary calls to the backend
- Checks
  - The required request parameters in the URI, query string ad headers of an incoming request are included and non-blank
  - The applicable request payload adheres to the configured JSON Schema request model of the method
- OpenAPI
  - Set up request validation by importing OpenAPI definitions file

## Caching

### Caching API Responses

- Caching reduces the number of calls made to the backend
- Default TTL (time to live) is 300 seconds (min: 0s, max:3600s)
- Caches are defined per stage
- Possible to override cache settings per method
- Cache encryption option
- Cache capacity between 0.5GB to 237GB
- Cache is expensive, makes sense in production, may not make sense in dev/test

### API Gateway Cache Invalidation

- Able to flush the entire cache (invalidate it) immediately
- Clients can invalidate the cache with header: Cache-Control:max-age=0 (with proper IAM authorisation)
- If you don’t impose an InvalidateCache policy (or choose the require authorisation checkbox in the console), any client can invalidate the API cache

## Usage Plans & API Keys

- If you want to make an API available as an offering ($) to your customers
- Usage plan
  - Who can access one or more deployed API stages and methods
  - How much and how fast they can access them
  - Uses API keys to identify API clients and meter access
  - Configure throttling limits and quote limits that are enforced on individual client
- API keys
  - Alphanumeric string values to distribute to your customers, eg. WbjAjigERer2954GDFgsdfeE
  - Can use with the usage plans to control access
  - Throttling limits are applied to the API keys
  - Quotas limits is the overall number of maximum requests

### Correct Order for API Keys (to configure a usage plan)

1. Create one or more APIs, configure the methods to require an API key, and deploy the APIs to stages
2. Generate or import API keys to distribute to application developers (your customers) who will be using your API
3. Create the usage plan with the desired throttle and quotas limits
4. Associate API stages and API keys with the usage plan

- Callers of the API must supply an assigned API key in the z-api-key header in the request to the API

## Monitoring, Logging and Tracking

- CloudWatch Logs
  - Log contains information about request/response body
  - Enable CloudWatch logging at the Stage level (with Log Level - ERROR, DEBUG, INFO)
  - Can override settings on a per API basis

### CloudWatch Metrics

- Metrics are by stage, possibility to enable detailed metrics
- CacheHitCount and CacheMissCount: efficiency of the cache
- Count: the total number of API requests in a given period
- IntegrationLatency: the time between when API Gateway relays a request to the backend and when it receives a response from the backend
- Latency: the time between when API Gateway receives a request from a client and when it returns a response to the client. The latency includes the integration latency and other API Gateway overhead
- 4xx error (client side) and 5xx error (server side)

### API Gateway Throttling

- Account limit
  - API Gateway throttles requests at 10,000 res across all APIs
  - Soft limit that can be increased upon request
- In case of throttling => 429 Too Many Requests (retriable error)
- Can set stage limit and method limits to improve performance
- Or you can define Usage Plans to throttle per customer
- Just like Lambda Concurrency, one aPI that tis overloaded, if not limited can cause the other APIs to be throttled

### Errors

- 4xx means client errors
  - 400: bad request
  - 403: access denies, WAF filtered
  - 429: quota exceeded, throttle
- 4xx means server errors
  - 502: bad gateway exception, usually for an incompatible output returned from a Lambda proxy integration backend and occasionally for out-of-order invocations due to heavy loads
  - 503: service unavailable exception
  - 504: integration failure - eg. Endpoint request timed out exception
    - API Gateway requests time out after 29 seconds max

### CORs

- CORs must be enabled when you receive API calls from another domain
- The OPTIONS pre-flight request must contain the following headers:
  - Access-Control-Allow-Methods
  - Access-Control-Allow-Headers
  - Access-Control-Allow-Origin
- CORs can be enabled through the console

## Authentication and Authorisation

### IAM Permissions

- Create an IAM policy authorisation and attach to User/Role
- Authentication = IAM
- Authorisation = IAM Policy
- Good to provide access within AWS (EC2, Lambda, IAM users etc.)
- Leverage Sig v4 capability where IAM credentials are in headers

### Resource Policies

- Resource policies (similar to Lambda Resource Policy)
- Allow for Cross Account Access (combined with IAM Security)
- Allow for a specific source IP address
- Allow for a VPC Endpoint

## API Gateway - Security

- Cognito User Pools
  - Congnito fully manages user lifecycle, token expires automatically
  - API gateway verifies identity automatically from AWS Cognito
  - No custom implementation required
  - Authentication = cognito user pools
  - Authorisation = API gateway methods
- Lambda Authoriser (formerly Custom Authorisers)
  - Token-based authoriser (bearer token) - eg. JWT (JSON Web Token) or OAuth
  - A request parameter-based Lambda authoriser (headers, query string, stage var)
  - Lambda must return an IAM policy for the user, result policy is cached
  - Authentication = external
  - Authorisation = lambda function
- IAM
  - Great for users.roles already within your AWS account (+ resource policy for cross account)
  - Handle authentication + authorisation
  - Leverages Signature v4
- Custom Authoriser
  - Great for 3rd party tokens
  - Very flexible in terms of what IAM policy is returned
  - Handle authentication verification + authorisation in the Lambda function
  - Pay per Lambda invocation, results are cached
- Cognito User Pool
  - You manage your own user pool (can be backed by Facebook, Google Log in etc.)
  - No n ed to write any custom code
  - Must implement authorisation in the backend

## REST API vs HTTP API

- HTTP APIs
  - Low latency, cost-effective AWS Lambda proxy, HTTP proxy APIs and private integration (no data mapping)
  - Supports OIDC and OAuth 2.0 authorisation and built in support for CORs
  - No usage plans and Api keys
- REST APIs
  - All features (except Native OpenID Connect.OAuth 2.0)

## WebSocket API

- What’s WebSocket?
  - Two-way interactive communication between a user’s browser and a server
  - Server can push information to the client, this enables stateful application use cases
- WebSocket APIs are often used in real-time applications such as chat applications, collaboration platforms, multiplayer games and financial trading platforms
- Works with AWS Services (Lambda, DynamoDB) or HTTP endpoints

## Routing

- Incoming JSON messages are routed to different backed
- If no routes => sent to $default
- You request a route selection expression to select the field on JSON to route from
- Sample expression: $request.body.action
- The result is evaluated against the route keys viable in your API Gateway
- The route is then connection ed the backend you’ve setup through API Gateway

## Architecture

- Create a single interface for all the micro-services in your company
- Use API endpoints with various resources
- Apply a simple domain name and SSL certificates
- Can apply forwarding and transformation rules at the API Gateway level

# AWS Cloud Development Kit (CDK)

- Define your cloud infrastructure using a familiar language
  - JavaScript/TypeScript, Python, Java and .NET
- Contains high level components called constructs
- The code is “complied” into a CloudFormation template (JSON/YAML)
- You can therefore deploy infrastructure and application runtime code together
  - Great for Lambda functions
  - Great for Docker containers in ECS/EKS

## CDK vs SAM

- SAM
  - Serverless focused
  - Write your template declaratively in JSON or YAML
  - Great for quickly getting started with Lambda
  - Leverage CloudFormation
- CDK
  - All AWS services
  - Write infra in a programming language such as JavaScript/TypeScript, Python, Java and .NET
  - Leverages CloudFormation

## CDK + SAM

- You can use SAM CLI to locally test your CDK app
- You must first run cdk synth

## Constructs

- CDK Construct is a component that encapsulates everything the CDK needs to create the final CloudFormation stack
- Can represent a single AWS resource (eg. S3 bucket) or multiple related resources (eg. Worker queue with compute)
- AWS Construct Library
  - A collect of Constructs included in AWS CDK, which contains constructs for every AWS resource
  - Contains 3 different levels of Constructs available (L1, L2, L3)
- Construct Hub: contains additional Constructs from AWS, 3rd parties and open-source CDK community

### Layer 1 Constructs (L1)

- Can be called CFN Resource which represents all resources directly available in CloudFormation
- Constructs are periodically generated from CloudFormation Resource Specification
- Construct names start with Cfn (eg. CfnBucket)
- You must explicitly configure all resource properties

### Layer 2 Constructs (L2)

- Represents AWS resources but with a higher level (intent-based API)
- Similar functionality as L1 but with convenient defaults and boilerplate
  - You don’t need to know all the details about the resource properties
- Provide methods that make it simpler to work with the resource (eg. bucket.addLifeCycleRule())

### Layer 3 Constructs (L3)

- Can be called Patterns, which represents multiple related resources
- Helps you compute common tasks in AWS
- Examples
  - Aws-apigateway.LambdaRestAPI represents an API Gateway backed by a Lambda function
  - Aws-ec-patterns.ApplicationLoadBalancerFargatService which represents an architecture that includes a Fargate cluster with Application Load Balancer

## Commands and Bootstrapping

npm install -g aws-cdk-lib //install the CDK CLI and libraries
cdk init app //create a new CDK project from specified template
cdk synth //syntehsizes and prints the CloudFormation template
cdk bootstrap //deploys the CDK Toolkit staging stack
cdk deploy //deploy the stack(s)
cdk diff //view differences of local CDK and deployed stack
cdk destroy //destroy the stack(s)

- The process of provisioning resources for CDK before you can deploy CDK apps into an AWS environment
- AWS Environment = account & Region
- CloudFormation Stack called CDKToolkit contains:
  - S3 bucket to store files
  - IAM Roles to grant permissions to perform deployments
- You must run the following command for each new environment
  cdk bootstrap aws://<aws_account>/<aws_region>
- Otherwise, you will get an error: policy contains a statement with one or move invalid principal

## Unit Testing

- To test CDK apps, use CDK Assertions Module combined with popular test frameworks such as Jest (JavaScript) or Pytest (Python)
- Verify we have specific resources, rules, conditions, parameters
- Two types of tests
  - Fine-grained Assertions (common): test specific aspects of the CloudFormation template (eg. Check if a resource has this property with this value)
  - Snapshot tests: test the synthesised CloudFormation template against a previously stored baseline template
- To import a template
  - Template.fromStack(MyStack): stack built in CDK
  - Template.fromString(MyString): stack built outside CDK

# AWS CLI, SDK, IAM Roles and Policies

## AWS EC2 Instance Metadata (IMDS)

- AWS EC2 Instance Metadata (IMDS) is powerful but one of the least known features to developers
- It allows AWS EC2 instances to “learn about themselves” without using an IAM Role for that purpose
- The URL is
- You can retrieve the IAM Role name from the metadata but you cannot retrieve the IAM Policy
- Metadata = Info about the EC2 Instance
- Userdata = launch script of the EC2 instance

### IMDSv2 vs. IMDSv1

- IMDSv1 is accessing directly
- IMDSv2 is more secure and is done in two-steps:
  - 1. Get Session Token (limited validity): using headers & PUT
  - 2. Use Session Token in IMDSv2 calls: using headers

## AWS CLI

### With MFA

- To use MFA with the CLI, you must create a temporary session
- To do so, you must run the STS GetSessionToken API call
- aws sts get-session-token —serial-number arn-of-the-mfa-device —token-code code-from-token —duration-seconds 3600

## AWS SDK

- What if you want to perform actions on AWS directly from your applications code? (Without CLI)
- You can use an SDK (Software Development Kit)
- Official SKDs are:
  - Java
  - .NET
  - Node.js
  - PHP
  - Python (named boto3/botocore)
  - Go
  - Ruby
  - C++
- We have to use the AWS SDK when coding against AWS services such as DynamoDB
- AWS CLI uses the Python SDK (boto 3)

## Exponential Backoff & Service Limit Increase

### AWS Limits (Quotas)

- API Rate Limits
  - DescribeInstances API for EC2 has a limit of 100 calls per seconds
  - GetObject on S3 has a limit of 5500 GET per second per prefix
  - For Intermittent Errors: implement Exponential Backoff
  - For Consistent Errors: request an API throttling limit increase
- Service Quotas (Service Limits)
  - Running On-Demand Standard Instances: 1 152 vCPU
  - You can request a service limit increase by opening a ticket
  - You can request a service quota increase by using the Service Quotas API

### Exponential Backoff (any AWS service)

- If you get ThrottlingException intermittently, use exponential backoff
- Retry mechanism already included in AWS SDK API calls
- Must implement yourself if using the AWS API as-is or in specific cases:
  - Must only implement the retries on 5xx server errors and throttling
  - Do not implement on the 4xx client errors

## AWS Credentials Provider and Chain

### The CLI will look for credentials in this order:

1. Command line options: —region, —output and —profile
2. Environment variables: AWS_ACCESS_KEY_ID, AWS_SERCRET_ACCESS_KEY and AWS_SESSION_TOKEN
3. CLI credentials file: aws configure ~/.aws/credentials on Linux/Mac & C:\Users\user\.aws\credentials on Windows
4. CLI configuration file: aws configure ~/.aws/config on Linux/macOS & C:\Users\USERNAME\.aws\config on Windows
5. Container credentials: for ECS tasks
6. Instance profile credentials: for EC2 Instance Profiles

### Best Practices

- Overall, NEVER EVER STORE AWS CREDENTIALS IN YOUR CODE
- Best practice is for credentials to be inherited from the credentials chain
- If using working with AWS, use IAM Roles
  - EC2 Instances Roles for EC2 Instances
  - ECS Roles for ECS tasks
  - Lambda Roles for Lambda function
- If working outside of AWS, use environment variables/named profiles

### AWS Signature v4 Signing (Sigv4)

- When you call the AWS HTTP API, you sign the request so that AWS can identify you, using your AWS credentials (access key & secret key)
- Note: some requests to Amazon S3 don’t need to be signed
- If you use the SDK or CLI, the HTTP requests are signed for you
- You should sign an AWS HTTP request using Signature v4 (SigV4)

# AWS CloudFormation

- CloudFormation is a declarative way of outlining your AWS Infrastructure, for any resources (most them are supported)
- For example, within a CloudFormation template, you say:
  - I want a security group
  - I want two EC2 instances using this security group
  - I want two Elastic IPs for these EC2 Instances
  - I want an S3 bucket
  - I want a load balancer (ELB) in front of me too these EC2 instances
- Then CloudFormation creates those for you, in the right order, with the exact configuration that you specify

## Benefits

- Infrastructure as code
  - No resources are manually created, which is excellent for control
  - The code can be version controlled for example using Git
  - Changes to the infrastructure are reviewed through code
- Cost
  - Each resources within the stack is tagged with an identifier so you can easily see how much a stack costs you
  - You can estimate the costs of your resources using the CloudFormation template
  - Savings strategy: in dev, you could automation deletion of templates at 5pm and recreated at 8am, safely
- Productivity
  - Ability to destroy and re-create an infrastructure on the cloud on the fly
  - Automated generation of Diagram for your templates
  - Declarative programming (no need to figure out ordering and orchestration)
- Separation of concern (create many stacks for many apps, and many layers)
  - VPC stacks
  - Network stacks
  - App stacks
- Don’t re-invent the wheel
  - Leverage existing templates on the web
  - Leverage the documentation

## How CloudFormation Works

- Templates must be uploaded in S3 and then referenced in CloudFormation
- To update a template, we can’t edit previous ones. We have to re-upload a new version of the templates to AWS
- Stacks are identified by name
- Deleting a stack deletes every single artifact that was created by CloudFormation

## Deploying CloudFormation Templates

- Manual way
  - Editing templates in Application Composer or code editor
  - Using the console to input parameters etc.
  - We’ll mostly do this way in the course for learning purposes
- Automated way
  - Editing templates in a YAML file
  - Using the AWS CLI (command line interface) to deploy the templates, or using a Continuous Delivery (CD) tool
  - Recommended way when you fully want to automate your flow

## Building Blocks

- Template’s components
  - AWSTemplateFormatVersion: identifies the capabilities of the template “2010-09-09”
  - Description: comments about the template
  - Resources (mandatory): your AWS resources declared in the template
  - Parameters: the dynamic inputs for your template
  - Mappings: the static variables for your template
  - Outputs: references to what has been created
  - Conditions: list of conditions to perform resource creation
- Template’s Helpers
  - References
  - Functions

## YAML

- YAML and JSON are the languages you can use for CloudFormation
- JSON is horrible for CF and YAML is great in so many ways
- Key value pairs
- Nested objects
- Support arrays
- Multi-line strings
- Can include comments

## Resources

- Resources are the core of your CloudFormation template (mandatory)
- They represent the different AWS Components that will be created and configured
- Resources are declared and can reference each other
- AWS figures out creation, updates and deletes resources for us
- There are over 700 types of resources
- Resource type identifiers are of the form: service-provider::service-name::data-type-name

### Can I Create a Dynamic Number of Resources

- Yes, you can by using CloudFormation Macros and Transform
- It is not in the scope of this course

### Is every AWS Service supported?

- Almost. Only a select few niches are not there yet
- You can work around that using CloudFormation Custom Resources

## Parameters

- Parameters are a way to provide inputs to your AWS CloudFormation template
- They’re important to know about if:
  - You want to reuse your templates across the company
  - Some inputs can not be determined ahead of time
- Parameters are extremely powerful, controlled, and can prevent errors from happening in your templates, thanks to types

### Why Should You Use a Parameter?

- Ask yourself this:
  - If this CloudFormation resource configuration likely to change int he future? If so, make it a parameter
- You won’t have to re-upload a template to change its content

### Parameters Settings

- Parameters can be controlled by these settings:
  - Type
    - String
    - Number
    - CommaDelimitedList
    - List<Number>
    - AWS-Specific Parameter (to help catch invalid values - match against existing values in the AWS account)
    - List<AWS-Specific Parameter>
    - SSM Parameter (get parameter value from SSM Parameter store)
  - Description
  - ConstraintDescription (string)
  - Min/MaxLength
  - Min/MaxValue
  - Default
  - allowedValues (array)
  - AllowedPatter (regex)
  - NoEcho (Boolean)

### How to reference a parameter

- The Fn::Reg function can be leveraged to reference parameters
- Parameters can be used anywhere in a template
- The shorthand for this in YAML is !Ref
- The function can also reference other elements within the template

### Pseudo Parameters

- AWS offers us Pseudo Parameters in any CloudFormation template
- These can be used at any time and are enabled by default
- Import pseudo parameters:
  - AWS::AccountId: 1234567890
  - AWS::Region: us-east-1
  - AWS::StackId: arn:aws:cloudformation:us-east-1:1234567890:stack/MyStack/29uefj93-jef9348-0394urfj
  - AWS::StackName: MyStack
  - AWS::NotificationARNs: [arn:aws:sns:us-east-1:1234567890:MyTopic]
  - AWS::NoValue: Doesn’t return a value

## Mappings

- Mappings are fixed variables within your CloudFormation template
- They’re very hand to differentiate between different environments (dev vs prod), regions (aws regions), AMI types…
- All the values are hardcoded within the template

### Accessing Mapping Values (Fn::FindInMap)

- We use Fn::FindInMap to return a named value from a specific key
- !FindInMap [MapName, TopLevelKey, SecondLevelKey]

## When Would youUse Mappings vs. Parameters?

- Mappings are great when you know in advance all the values that can be taken and that they can be deduced from variables such as:
  - Region
  - Availability Zone
  - AWS account
  - Env (dev vs prod)
- They allow safer control over the template
- Use parameters when the values are really user specific

## Outputs & Exports

- The Outputs section declares optional outputs values that we can import into other stacks (if you export them first)
- You can also view the outputs in the AWS Console or in using the AWS CLI
- They’re very useful for example if you define a network CloudFormation, and output the variables such as VPC ID and your Subnet IDs
- It’s the best way to perform some collaboration cross stack, as you let an export handle their own part of the stack
- Creating a SSH Security Group as part of the one template
- We create an output that references that security group

### Outputs Cross-Stack Reference

- We then create a second template that leverages that security group
- For this, we use the Fn::ImportValue function
- You can’t delete the underlying stack until all the references are deleted

## Conditions

- Conditions are used to control the creation of resources or outputs based on a condition
- Conditions can be whatever you want them to be, but common ones are:
  - Environment (dev/test/prod)
  - AWS Region
  - Any parameter value
- Each condition can reference another condition, parameter value or mapping

### How to Define a Condition

- The logical ID is for you to choose - it’s how you name a condition
- The intrinsic function (logical) can be nay of the following:
  - Fn::And
  - Fn::Equal
  - Fn::If
  - Fn::Not
  - Fn::Or

### How to Use a Condition

- Conditions can be applied to resources, outputs etc.

## Intrinsic Functions

- Ref
- Fn::GetAtt
- Fn:FindInMap
- Fn::ImportValue
- Fn::Base64
- Fn::Join
- Fn::Sub
- Fn::ForEach
- Fn::ToJsonString
- Fn::Cidr
- Fn::GetAZs
- Fn::Select
- Fn::Split
- Fn::Transform
- Fn::Length
- Conditional Functions (Fn::If, Fn::Not, Fn::Equals etc.)

### Fn:Ref

- The Fn::Ref function can be leveraged to reference
  - Parameters: returns the value of the parameter
  - Resources: returns the physical ID of the underlying resources (eg. EC2 ID)
- The shorthand for this in YMAL is !Ref

### Fn::GetAtt

- Attributes are attached to any resources you create
- To know the attributes of your resources, the best place to look at is the documentation
- Example: the AZ of an EC2 instance

### Fn:FindInMap

- We use Fn::FindInMap to return a named value from a specific key
- !FindInMap [MapName, TopLevelKey, SecondLevelKey]

### Fn::ImportValue

- Import values that are exported in other stacks
- For this, we use the Fn::ImportValue function

### Fn::Base64

- Convert String to its Based 64 representation (!Base64 “ValueToEncode”)
- Example: pass encoded data to EC2 Instance’s UserData property

### Condition Functions

- The logical ID is for you to choose - its how you name a condition
- The intrinsic function (logical) can be any of the following:
  - Fn::And
  - Fn::Equals
  - Fn::If
  - Fn::Not
  - Fn::Or

## Rollbacks

- Stack creation fails
  - Default: everything rolls back (gets deleted) - can look at the logs
  - Option to disable rollback and troubleshoot what happened
- Stack update fails
  - The stack automatically rolls back to the previous known working state
  - Ability to see in the log what happened and error messages
- Rollback failure? Fix resources manually then issue ContinueUpdateRollback API from Console
  - Or from the CLI using continue-update-rollback API call

## Service Role

- IAM role that allows CloudFormation to create/update/delete stack resources on your behalf
- Gives the ability to users to create/update/delete the stack resources even if they don’t have permissions to work with the resource sin the stack
- Use cases: you want to achieve the least privilege principle but you don’t want to give the user all the required permissions to create the stack resources
- Users must have iam:PassRole permissions

## Capabilities

- CAPABILITY_NAMED_IAM and CAPABILITY_IAM
  - Necessary to enable when your CloudFormation template is creating or updating IAM resources (IAM User, Role, Group, Policy, Access Keys, Instance Profile)
  - Specify CAPABILITY_NAMED_IAM if the resource is named
- CAPABILITY_AUTO_EXPAND
  - Necessary when your CloudFormation template includes Macros or Nested Stacks (stacks within stacks) to perform dynamic transformations
  - You’re acknowledging that your template may change before deploying
- InsufficientCapabilitiesException
  - Exception that will be thrown by CloudFormation if the capabilities haven’t been acknowledged when deploying a template (security measure)

## Deletion Policy

### Delete

- DeletionPolicy:
  - Control what happens when the CloudFormation template is deleted or when a resource is removed from a CloudFormation template
  - Extra safety measure to preserve and backup resources
- Default DeletionPolicy=Delete
  - Delete won’t work on an S3 bucket if the bucket is not empty

### Retain

- DeletionPolicy=Retain
  - Specify on resources to preserve in case of CloudFormation deletes
  - Works with any resources

### Snapshot

- DeletionPolicy=Snapshot
- Create one final snapshot before deleting the resource
- Examples of supported resources:
  - EBS Volume, ElastiCache Cluster, ElastiCache ReplicationGroup
  - RDS DBInstance, RDS DBCluster, Redshift Cluster, Neptune DBCluster, DocumentDB DBCluster

### Stack Policy

- During a CloudFormation Stack update, all update actions are allowed on all resources (Default)
- A Stack Policy is a JSON document that defines the update actions that are allowed on specific resources during stack updates
- Protect resources from unintentional updates
- When you set a Stack Policy, all resources in the stack are protected by default
- Specify an explicit ALLOW for the resources you want to be allowed to be updated

### Termination Protection

- To percent accidental deleted of CloudFormation Stacks, use TerminationProtection

## Custom Resources

- Used to:
  - Define resources not yet supported by CloudFormation
  - Define custom provisioning logic for resources that can be outside of CloudFormation (on-premises resources, 3rd party resources)
  - Have custom scripts run during create/update/delete through Lambda functions (running a Lambda function to empty an S3 bucket before being deleted)
- Defined in the template using AWS::CloudFormation::CustomResource or Custom::MyCustomResourceTypeName (recommended)
- Backed by a Lambda function (most common) or an SNS topic

### How to Define a Custom Resource

- ServiceToken specifies where CloudFormation sends requests to, such as Lambda ARN or SNS ARN (required & most be in the same region)
- Input data parameters (optional)

### Use Case: Delete Content From an S3 Bucket

- You can’t delete a non-empty S3 bucket
- To delete a non-empty S3 bucket, you must first delete all the objects inside it
- We can use a custom resource to empty an S3 bucket before it gets deleted by CloudFormation

## StackSets

- Create, update or delete stacks across multiple accounts and regions with a single operation/template
- Target accounts to create, update, delete stack instances from StackSets
- When you update a stack set, all associated stack instances are updated throughout all accounts and regions
- Can be applied into all accounts of an AWS Organisation
- Only Administrator account (or delegated admin) can create StackSets

# Cognito: User Pools, Identity Pools and Sync

- Give uses an identity to interact with our web or mobile application
- Cognito User pools
  - Sign in functionality for app users
  - Integrate with API Gateway & Application Load Balancer
- Cognito Identity Pools (Federated Identity)
  - Provide AWS credentials to users so that they can access AWS resources directly
  - Integrate with Cognito User Pools as an identity provider
- Cognito vs IAM: “hundreds of users”, “mobile users”, “authenticate with SAML”

## User Pools

### User Features

- Create a server less database of user for you web and mobile apps
- Simple login: username (or email) / password combination
- Password reset
- Email and phone number verification
- Multi-factor authentication (MFA)
- Federated identities: users from Google, Facebook, SAML etc.
- Feature: block users if their credentials are compromised elsewhere
- Login sends back a JWT (JSON Web Token)

## Integrations

- CUP integrates with API Gateway and Application Load Balancer

## Other

- Lambda triggers: CUP can invoke a Lambda function synchronously on the triggers below
  ![alt text](<Screenshot 2024-08-19 at 09.52.51.png>)
- Hosted Authentication UI
  - Cognito has a hosted authentication UI that you can add to your app to handle sign up and sign workflows
  - Using the hosted UI, you have a foundation for integration with social logins, OIDC or SAML
  - Can customise with custom logo or CSS
- Hosted UI Custom Domain
  - For custom domains, you must create an ACM certificate in us-east-1
  - The custom domain must be defined in the “App Integration” section
- Adaptive authentication
  - Block sign-ins or require MFA if the login appears suspicious
  - Cognito examines each sign-in attempt and generates a risk score (low, medium, high) for how likely the sign-in request is to be from a malicious attacker
  - Users are prompted for a second MFA only when risk is detected
  - Risk score is based on different factors such as if the user has used the same device, location or IP address
  - Checks for compromised credentials, account takeover protection and phone and email verification
  - Integration with CloudWatch Logs (sign in attempts, risk score, failed challenges etc.)
- Decoding an ID Token (JWT)
  - CUP issues JWT tokens (Base64 encoded)
    - Header
    - Payload
    - Signature
  - The signature must be verified to ensure the JWT can be trusted
  - Libraries can help you verify the validity of JWT tokens issued by Cognito User Pools
  - The payload will contain the user information (sub UUID, given_name, email, phone_number, attributes)
  - From the sub UUID, you can retrieve all users details from Cognito/OIDC

## Application Load Balancer: User Authentication

### Authenticate Users

- Your Application Load Balancer can securely authenticate users
  - Offload the work of authenticating users to your load balancer
  - Your applications can focus on their business login
- Authenticate users through:
  - Identity Provider (IdP): OpenID Connect (OIDC) compliant
  - Cognito User Pools
    - Social IdPs, such as Amazon, Facebook or Google
- Must use an HTTPS listener to set authenticate-cidc and authenticate-cogito rules
- OnUnauthenticatedRequest - authenticate (default), deny, allow

### Using Cognito User Pools

- Create Cognito User Pool, Client and Domain
- Make sure an ID token is returned
- Add the social or Corporate IdP if needed
- Several URL redirections are necessary
- Allow your Cognito User Pool Domain on your IdP app’s callback URL
  - Eg: https//domain-prefix.auth.region.amazoncognito.com/saml2/idpreresponse

### Through an Identity Provider (IdP) That is OpenID Connect (OIDC) Compliant

- Configure a client ID & Client Secret
- All redirect from OIDC to your Application Load Balancer DNS name (AWS provided) and CNAME (DNS Alias of your app)
  - https://DNS/oauth2/idpresonse
  - https://CNAME/oauth2/idpresponse

### Cognito Identity Pools (Federated Identities)

- Get identities for “users” so they obtain temporary AWS credentials
- Your identity pool (eg. Identity source) can include:
  - Public providers (login with Amazon, Facebook, Google, Apple)
  - Users in an Amazon Cognito user pool
  - Open ID Connect Providers & SAML Identity Providers
  - Developer Authenticated Identities (custom login server)
  - Cognito Identity Pools allow for unauthenticated (guest) access
- Users can then access AWS services directly or through API Gateway
  - The IAM policies applied to the credentials are defined in Cognito
  - They can be customised based on the user_id for fine grained control

### IAM Roles

- Default IAM roles for authenticated and guest users
- Define rules to choose the role for each user based on the user’s ID
- You can partition your users’ access using policy variables
- IAM credentials are obtained by Cognito Identity Pools through STS
- The roles must have a “trust” policy of Cognito Identity Pools

## Cognito User Pools vs Cognito Identity Pools

- Cognito User Pools (for authentication = identity verification)
  - Database of users for your web and mobile application
  - Allows to federate logins through Public Social, OIDC, SAML
  - Can customise the hosted UI for authentication (including the logo)
  - Has triggers with AWS Lambda during the authentication flow
  - Adapt the sign in experience to different risk levels (MFA, adaptive authentication etc.)
- Cognito Identity Pools (for authorisation = access control)
  - Obtain AWS credentials for your users
  - Users can login through Public social, OIDC, SAML and Cognito User Pools
  - Users can unauthenticated (guests)
  - Users are mapped to IAM roles and policies, can leverage policy variables
- CUP + CIP = authentication + authorisation

# ECS, ECR & Fargate - Docker in AWS

## Docker

- Docker is a software development platform to deploy apps
- Apps are packaged in containers that can be run on any OS
- Apps run the same regardless of where they’re run
  - Any machine
  - No compatibility issues
  - Predictable behaviour
  - Less work
  - Easier to main and deploy
  - Works with any language, any OS, any technology
- Use cases: micro services, architecture, lift-and-shift apps from on-premises to the AWS cloud

### Where Are Docker Images Stored?

- Docker images are stored in Docker Repositories
- Docker Hub
  - Public repository
  - Find base images for many technologies or OS (eg. Ubuntu, MySQL)
- Amazon ECR (Amazon Elastic Container Registry)
  - Private repository
  - Public repository (Amazon ECR public gallery)

### Docker vs. Virtual Machines

- Docker is “sort of” a virtualisation technology but not exactly
- Resources are shared with the host => Many containers on one server

### Docker Containers Management on AWS

- Amazon Elastic Container Service (Amazon ECS)
  - Amazon’s own container platform
- Amazon Elastic Kubernetes Service (Amazon EKS)
  - Amazon’s managed Kubernetes (open source)
- AWS Fargate
  - Amazon’s own Serverless container platform
  - Works with ECS and with EKS
- Amazon ECR
  - Store container images

## Amazon ECS

### EC2 Launch Type

- ECS = Elastic Container Service
- Launch Docker containers on AWS = Launch ECS Tasks on ECS Clusters
- EC2 Launch Type: you must provision & maintain the infrastructure (the EC2 instances)
- Each EC2 Instance must run the ECS Agent to register in the ECS Cluster
- AWS takes care of starting/stopping containers

### Fargate launch Type

- Launch Docker containers on AWS
- You do not provision the infrastructure (no EC2 instances to manage)
- It’s all Serverless
- You just create task definitions
- AWS just runs ECS Tasks for you based on the CPU/RAM you need
- To scale, just increase the number of tasks - no more EC2 instances

### IAM Roles for ECS

- EC2 Instance Profile (EC2 Launch Type only)
  - Used by the ECS agent
  - Makes API calls to ECS service
  - Send container logs to CloudWatch Logs
  - Pull Docker image from ECR
  - Reference sensitive data in Secrets Manager or SSM Parameter Store
- ECS Take Role
  - Allows each task to have a specific role
  - Use different roles for the different ECS Services you run
  - Task Role is defined in the task definition

## Load Balancer Integrations

- Application Load Balancer supported and works for most use cases
- Network Load Balancer recommended only for high throughput/high performance use cases, or to pair it with AWS Private Link
- Classic Load Balancer supported but not recommended (no advanced features - no Fargate)

## Data Volumes (EFS)

- Mount EFS file systems onto ECS tasks
- Works for both EC2 and Fargate launch types
- Tasks running in any AZ will share the same data in the EFS file system
- Fargate + EFS = Serverless
- Use cases: persistent multi-AZ shared storage for your containers
- Note: Amazon S3 cannot be mounted as a file system

## Auto Scaling

- Automatically increase/decrease the desired number of ECS tasks
- Amazon ECS Auto Scaling uses AWS Application Auto Scaling
  - ECS Service Average CPU Utilisation
  - ECS Service Average Memory Utilisation - Scale on RAM
  - ALB Request Count Per Target - metric coming from the ALB
- Target tracking: scale based on target value for a specific CloudWatch metric
- Step Scaling: scale based on a specified CloudWatch Alarm
- Scheduled Scaling: scale based on a specified date/time (predictable changes)
- ECS Service Auto Scaling (task level) != EC2 Auto Scaling (EC2 instance level)
- Fargate Auto Scaling is much easier to set up (because Serverless)
- Auto scaling EC2 Instances:
  - Accommodate ECS Service Scaling by adding underlying EC2 Instances
  - Auto Scaling Group Scaling
    - Scale your ASG based on CPU Utilisation
    - Add EC2 instances over time
  - ECS Cluster Capacity Provider
    - Used to automatically provision and scale the infrastructure for your ECS Tasks
    - Capacity Provider paired with an Auto Scaling Group
    - Add EC2 Instances when you’re missing capacity (CPU, RAM…)

## Rolling Updates

- When updating from v1 to v2, we can control how many tasks can be started and stopped, and in which order

## Solutions Architectures

- ECS tasks invoked by Event Bridge
- ECS tasks invoked by Event Bridge Schedule
- SQS Queue
- Intercept Stopped Tasks using EventBridge

## Task Definitions

- Task definitions are metadata in JSON form to tell ECS how to run a Docker container
- It contains crucial information such as:
  - Image Name
  - Port Binding for Container and Host
  - Memory and CPU required
  - Environment variables
  - Networking information
  - IAM Role
  - Logging configurations (eg. CloudWatch)
- Can define up to 10 containers in a Task Definition
- Load Balancing (EC2 Launch Type):
  - We get a Dynamic Host Port Mapping if you define only the container port in the task definition
  - The ALB finds the right sport on your EC2 Instance
  - You must allow on the EC2 Instances Security group any port from the ALB’s Security Group
- Load Balancing (Fargate):
  - Each task has a unique private IP
  - Only define the container port (host port is not applicable)
  - Examples:
    - ECS ENI Security Group
      - Allow port 80 from the ALB
    - ALB Security Group
      - Allow port 80/443 from web
- Environment Variables
  - Hardcoded, eg. URLs
  - SSM Parameter Store: sensitive variables (eg. API keys, shared configs)
  - Secrets Manager: sensitive variables (eg. DB passwords)
- Environment Files (bulk): Amazon S3
- Data Volumes (Bind Mounts)
  - Share data between multiple containers in the same Task Definition
  - Works for both EC2 and Fargate tasks
  - EC2 Tasks: using EC2 Instance Storage
    - Data is tied to the lifecycle of the EC2 instance
  - Fargate Tasks: using ephemeral storage
    - Data is tired to the container(s) using them
    - 20 GiB - 200 GiB (default 20 GiB)
  - Use cases:
    - Share ephemeral data between multiple containers
    - “Sidecar” container pattern, where the “sidecar” container used to send metrics/logs to other destinations (separation of concerns)

## Task Placements

- When a task of type EC2 is launched, ECS must determine where the place it, with the constraints of CPU memory, and available port
- Similarly, when a service scales in, ECS needs to determine which task to terminate
- To assist with this, you can define a task placement strategy and task placement constraints
- Note: this is only for ECS with EC2, not for Fargate
- ECS Task Placement Process:
  - Task placement strategies are a best effort
  - When Amazon ECS places tasks, it uses the following process to select container instances:
    - 1. Identify the instances that satisfy the CPU, memory and port requirements in the task definition
    - 2. Identify the instances that satisfy the task placements constraints
    - 3. Identify the instances that satisfy the task placement strategies
    - 4. Select the instances for task placement
- Strategies:
  - Binpack
    - Place tasks based not eh least available amount of CPU or memory
    - This immunises the number of instances in use (cost savings)
  - Random: place the task randomly
  - Spread
    - Place the task evenly based on the specified value
    - Example: instanceId, attribute:ecs.availability-zone
  - Strategies can be mixed together
- Constraints
  - distinctInstance: place each task on a different container instance
  - memberOf: places task on instances that satisfy an expression
    - Uses the Cluster Query Language (advanced)

## Amazon ECR

- ECR = Elastic Container Registry
- Store and manage Docker images on AWS
- Private and public repository (Amazon ECR Public Gallery
- Fully integrated with ECS, backed by Amazon S3
- Access is controlled through IAM (permission errors => policy)
- Supports image vulnerability scanning, versioning, image tags, image life cycles)
- Using AWS CLI
  - Login command
    - AWS CLI v2
  - Docker Commands
    - Push
    - Pull
    - Docker [push/pull] aws_account_id.dkr.ecr.region.amazonaws.com/demo:latest
  - In case an EC2 Instance (or you) can’t pull a Docker image, check IAM permission

## AWS CoPilot

- CLI tool to build, release and operate production-ready containerised apps
- Run your apps on AppRunner, ECS and Fargate
- Helps you focus on building apps rather than setting up infrastructure
- Provisions all required infrastructure for containerised apps (ECS, VPC, ELB, ECR)
- Automated deployments with one command using CodePipeline
- Deploy to multiple environments
- Troubleshooting, logs, health status etc.

## Amazon EKS

- Amazon EKS = Amazon Elastic Kubernetes Service
- It is a way to launch managed Kubernetes clusters on AWS
- Kubernetes is an open-source system for automatic deployment, scaling and management of containerised (usually Docker) application
- It’s an alternative to ECS, similar goal but different API
- EKS support sEC2 if you want to deploy worker nods or Fargate to deploy serverless containers
- Use case: if you r company is already using Kubernetes on-premises or in another cloud, and wants to migrate to AWS using Kubernetes
- Kubernetes is cloud-agnostic (can be used in any cloud, eg. Azure, GCP)

## Node Types

- Managed Node Groups
  - Creates and manages Nodes (EC2 Instances) for you
  - Nodes are part of an ASG managed by EKS
  - Supports On-Demand or Spot Instances
- Self-Managed Nodes
  - Nodes created by you and registered to the EKS cluster and managed by an ASG
  - You can use prebuilt AMI - Amazon EKS Optimised AMI
  - Supports On-Demand or Spot Instances
- AWS Fargate
  - No maintenance required; no nodes managed

## Data Volumes

- Need to specify SotrageClass manifest on your EKS cluster
- Leverages a Container Storage Interface (CSI) compliant driver
- Support for:
  - Amazon EBS
  - Amazon EFS (works with Fargate)
  - Amazon Fix for Lustre
  - Amazon FSx for NetApp ONTAP

# AWS Elastic Beanstalk

- Elastic Beanstalk is a developer centric view of deploying an application on AWS
- It uses all the component’s we’ve seen before; EC2, ASG, ELB, RDS etc.
- Managed service
  - Automatically handles capacity provisioning, load balancing, scaling, application health monitoring, instance configuration
  - Just the application code is the responsibility of the developer
- We still have full control over the configuration
- Beanstalk is free but you pay for the underlying instances

## Components

- Application: collection of Elastic Beanstalk components (environments, versions, configurations)
- Application version: an iteration of your application code
- Environment
  - Collection of AWS resources running an application version (only one version at a time)
  - Tiers: web server environment tier and worker environment tier
  - You can create multiple environments (dev, test, prod)

## Deployment Modes

- Single Instance is great for dev
- High Availability with Load Balancer is great for prod

## Beanstalk Deployment Modes

- All at once (deploy all in one go): fastest but instances aren’t available to serve traffic for a bit (downtime)
  - Fastest deployment
  - Application has downtime
  - Great for quick iterations in development environment
  - No additional cost
- Rolling: update a few instances at a time (bucket) and then move onto the next bucket once the first bucket is healthy
  - Application is running below capacity
  - Can set the bucket size
  - Application is running both versions simultaneously
  - No additional cost
  - Long deployment
- Rolling with additional batches: like rolling, but spins up new instances to move the batch (so that the old application is still available)
  - Application is running at capacity
  - Can set the bucket size
  - Application is running both versions simultaneously
  - Small additional cost
  - Additional batch is removed at the end of the deployment
  - Longer deployment
  - Good for prod
- Immutable: spins up new instances in a new ASG, deploys versions to these instances and then swaps all the instances when everything is healthy
  - Zero downtime
  - New code is deployed to new instances on a temporary ASG
  - High cost, double capacity
  - Longest deployment
  - Quick rollback in case of failures (just terminate new ASG)
  - Great for prod
- Blue Green: create a new new environment and switch over when ready
  - Not a “direct feature” of Elastic Beanstalk
  - Zero downtime and release facility
  - Create a new “stage” environment and deploy v2 there
  - The new environment (green) can be validated independently and roll back if issues
  - Route 53 can be setup using weighted policies to redirect a little bit of traffic to the stage environment
  - Using Beanstalk “swap URLs” when done with the environment test
- Traffic splitting: canary testing - sending a small %of traffic to new deployment
  - Canary testing
  - New application version is deployed to a temporary ASG with the same capacity
  - A small % of traffic is sent o the temporary ASG for a configurable amount of time
  - Deployment health is monitored
  - If there’s a deployment failure, this triggers an automated rollback (very quick)
  - No application downtime
  - New instances are migrated from the temporary to the original ASG
  - Old application version is then terminated

## Beanstalk CLI and Deployment Process

- We can instal an additional CLI called the “EB cli” which makes working with Beanstalk from the CLI easier
- Basic commands are:
  - (eb) create, status, health, events, logs, open, deploy, config, terminate
- It’s helpful for your automated deployment pipelines

## Deployment Process

- Describe dependencies
  - Requirements.txt for Python, package.json for Node.js
- Package code as zip and describe dependencies
  - Python: requirements.txt
  - Node.js: package.json
- Console: upload zip file (creates a new app version) and then deploy
- CLI: create a new app version using CLI (uploads zip) and then deploy
- Elastic Beanstalk will deploy the zip on each EC2 instance, resolve dependencies and start the application

## Beanstalk Lifecycle Policy Overview

- Elastic Beanstalk can store at more 1000 application versions
- If you don’t remove old versions, you won’t be able to deploy anymore
- To phase out old application versions, use a lifecycle policy
  - Based on time (old versions are removed)
  - Based on space (when you have too many versions)
- Versions that are currently used own’ the deleted
- Option not to delete the source bundle in S3 to prevent data loss

## Beanstalk Extensions

- A zip file containing our code must be deployed to Elastic Beanstalk
- All the parameters set in the UI can be configured with code using files
- Requirements:
  - In the .ebextensions/ directory in the root of source code
  - YAML/JSON format
  - .config extensions (example: logging.config)
  - Able to modify some default settings using: option_settings
  - Ability to add resources such as RDS, ElastiCache, DynamoDb etc.
- Resources managed by .ebextensions get deleted if the environment goes away

## Beanstalk & CloudFormation

- Under the hood, Elastic Beanstalk relies on CloudFormation
- CloudFormation is used to provision other AWS services
- Use case: you can define CloudFormation resources in your .ebextensions to provision ElastiCache, an S3, anything you want

## Beanstalk Cloning

- Clone an environment with the exact same configuration
- Useful for deploying a “test” version of your application
- All resources and configurations are preserved
  - Load Balancer type and configuration
  - RDS database type (but the data is not preserved)
  - Environment variables
- After cloning an environment, you can change settings

## Beanstalk Migrations

### Load Balancer

- After creating an Elastic Beanstalk environment, you cannot change the Elastic Load Balancer type (only configuration)
  - To migrate:
    - 1. Create a new environment with the same confirmation except LB (can’t clone one)
    - 2. Deploy your application onto the new environment
    - 3. Perform a CNAME swap or Route 53 update

### RDS with Elastic Beanstalk

- RDS can be provisioned with Beanstalk, which is great for dev/test
- This is not great for prod as the database lifecycle is tied to Beanstalk environment lifecycle
- The best for prod is to separately create an EDS database and provide our EB application with the connection string

#### Decouple RDS

1. Create a snapshot of RDS DB (as a safeguard)
2. Go to the RDS console and protect the RDS database from deletion
3. Create a new Elastic Beanstalk environment, without RDS, point your application to exiting RDS
4. Perform a CNAME swap (blue/green) or Route 53 update, confirm working
5. Terminate the old environments (RDS won’t be deleted)
6. Delete CloudFormation stack (in DELETE_FAILED state)

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

# AWS CICD: CodeCommit, CodePipeline, CodeBuild, CodeDeploy

- This section is all about automating the deployment we’ve done so far while adding increased safety
- CodeCommit: storing code
- CodePipeline: automating pipelines from code to Elastic Beanstalk
- CodeBuild: building and testing code
- CodeDeploy: deploying the code to EC2 instances (not Elastic Beanstalk)
- CodeStar: manage software development activities in one place
- CodeArtifact: store, publish and share software packages
- CodeGuru: automated code reviews using Machine Learning

## Continuous Integration (CI)

- Developers push the code to a repository
- A testing/build server checks the code once it’s pushed (CodeBuild, Jenkins CI, etc.)
- Developer gets feedback about passed/failed tests
- Find bugs early which can be fixed
- Deliver faster, as code is tested
- Deploy often

## Continuous Delivery (CD)

- Ensures that the software can be released reliably whenever needed
- Ensures deployments happen often and are quick
- Shift away from “one release every three months” to “five releases a day”
- Usually means automated deployment (eg. CodeDeploy, Jenkins CD, Spinnaker)

## CodeCommit

- Private Git repos
- No size limit on repos (scale seamlessly)
- Fully managed, highly available
- Code only in AWS Cloud account (increased security and compliance)
- Security (encrypted, access control, etc.)
- Integrated with Jenkins, AWS CodeBuild and other CI tools

## Security

- Interactions are done using Git
- Authentication
  - SSH keys: AWS Users can configure SSH keys in their IAM Console
  - HTTPS: with AWS CLI Credential helper or Git Credentials for IAM user
- Authorisation
  - IAM policies to manage users/roles permissions to repositories
- Encryption
  - Repositories are automatically encrypted at rest using AWS KMS
  - Encrypted in transit (can only use HTTPS or SSH - both secure)
- Cross-account access
  - Do NOT share your SSH keys or AWS credentials
  - Use an IAM Role in your AWS account and AWS STS (AssumeRole API)

NOTE: discontinued on 25th July 2024 (assume there’s a GitHub integration, might still come up in the exam)

## CodePipeline

- Visual Workflow to orchestrate your CICD
- Source: CodeCommit, ECR, S3, Bitbucket, GitHub
- Build: CodeBuild, Jenkins, CloudBees, TeamCity
- Test: CodeBuild, AWS Device Farm, 3rd party tools
- Deploy: CodeDeploy, Elastic Beanstalk, CloudFormation, ECS, S3
- Invoke: Lambda, Step Functions
- Consists of stages
  - Each stage can have sequential actions and/or parallel actions
  - Example: build -> test -> deploy -> load testing
  - Manual approval can be defined at any stage

## Artifacts

- Each pipeline stage can create artefacts
- Artifacts are stored in an S3 bucket and passed on to the next stage

## Troubleshooting

- For CodePipeline Pipeline/Action/Stage Execution State Changes
- Use CloudWatch Events (Amazon EventBridge)
  - Example: you can create events for failed pipelines, cancelled stages etc.
- If CodePipeline fails a stage, your pipeline stops, and you can get information in the console
- If pipeline can’t perform an action, make sure the “IAM Service Role” attached has enough IAM permissions (IAM Policy)
- AWS CloudTrail can be used to audit AWS API calls

## CodeBuild

- Source: CodeCommit, S3, Bitbucket, gitHub
- Build instructions: code file build spec.yml or insert manually in console
- Output logs can be store din Amazon S3 and CLoudWatch Logs
- Use CloudWatch Metrics to monitor build statistics
- Use EventBridge to detect failed builds and trigger notifications
- Use CloudWatch Alarms to notify if you need “thresholds” for failures
- Build projects can be defined within CoedPipeline or CodeBuild

### Supported Environments

- Java
- Ruby
- Python
- Go
- Node.js
- Android
- .NET Core
- PHP
- Docker - extend any environment you like

### buildspec.yml

- buildspec.yml file must be at the root of your code
- Env - define environment variables
  - Variables : plaintext variables
  - Parameter-store: variables store din SSM Parameter Store
  - Secrets-manager: variables stored in AWS Secrets Manager
- Phases - specify commands to run
  - Install: install dependencies you may need for your build
  - Pre_build: final commands to execute before build
  - Build: actual build commands
  - Post_build: finishing touches (eg. Zip cutout)
- Artifacts: what to upload to S3 (encrypted with KMS)
- Cache: files to cache (usually dependencies) to S3 for future build speed up

## CodeDeploy

- Deployment service that automates applicant deployment
- Deploy new application versions to EC2 Instances, On-premises servers, Lambda functions, ECS services
- Automated Rollback capability in case of failed deployments or trigger CloudWatch Alarm
- Gradual deployment control

## EC2/On-Premises Platform

- Can deploy to EC2 Instances and on-premise servers
- Perform in-place deployments or blue/green deployments
- Must run the CodeDeploy Agent on the target instances
- Define deployment speed
  - AllAtOnce: most downtime
  - HalfAtATime: reduced capacity 50%
  - OneAtATime: slowest, lowest availability impact
  - Custom: define you %

## CodeDeploy Agent

- The CodeDeploy Agent must be running on the EC2 instances a pre-requisites
- It can be installed and updated automatically if you’re using Systems Manager
- The EC2 Instances must have sufficient permissions to access Amazon S3 to get deployment bundles

## Lambda Platform

- CodeDeploy can help you automate traffic shift for Lambda aliases
- Feature is integrated within the SAM framework
- Linear: grow traffic every N minutes until 100%
  - LambdaLinear10PercentEvery3Minutes
  - LambdaLinear10PercentEvery10Minutes
- Canary: try X percent them 100%
  - LambdaCanary1-Precent5Minutes
  - LambdaCanary10Percent30Minutes
- AllAtOnce: immediate

## ECS Platform

- CodeDeploy can help you automate the deployment of a new ECS Task Definition
- Only Blue/Green Deployments
- Linear: grow traffic every N minutes until 100% (replace Lambda with ECS like example above)
- Canary: try x percent then 100%
- AllAtOnce: immediate

## EC2 and ASG

- Deploy to EC2
  - Define how to deploy the application using appspec.yml + deployment strategy
  - Will do in-place update to your fleet of EC2 instances
  - Can use hooks to verify the deployment after each deployment phase
- Deploy to an ASG
  - In-place deployment
    - Updates existing EC2 instances
    - Newly created EC2 instances by an aSG will also get automated deployments
  - Blue/Green deployment
    - A new auto-scaling group is created (settings are copied)
    - Choose how long to keep the old EC2 instances (old ASG)
    - Must be using an ELB
- Redeploy and rollbacks
  - Rollback = redeploy a previously deployed revision of your application
  - Deployments can be rolled back
    - Automatically: rollback when. Deployment fails or rollback when a CloudWatch Alarm threshold is met
    - Manually
  - Disable rollbacks: do not perform rollbacks for this deployment
- If a roll back happens, CodeDeploy redeploys the last known good revision as a new deployment (not a restored version)

## CodeArtifact

- Software packages depend on each other to be built (also called code dependencies) and new ones are created
- Storing and retrieving these dependencies is called artifact management
- Traditionally you would need to set up your own artifact management system
- CodeArtifact is a secure, scalable and cost effective artifact management for software development
- Works with common dependency management tools such as Maven, Gradle, npm, yarn, twin, pip and NuGet
- Developers and CodeBuild can then retrieve dependencies straight from CodeArtifact

## Resource Policy

- Can be used to authorise another account to access CodeArtifact
- A given principal can either read all the packages in a repository or none of them

## CodeGuru

- A ML-powered service for automated code reviews and application performance recommendations
- Provides two functionalities
  - CodeGuru Reviewer: automated code reviews for static code analysis (development)
  - CodeGuru Profiler: visibility/recommendations about application performance during runtime (production)

## Reviewer

- Identify critical issues, security vulnerabilities and hard-to-find bugs
- Example: common coding best practices, resource leaks, security detection, input validation
- Uses Machine Learning and automated reasoning
- Hard-learned lessons across millions of code reviews on 1000s of open-source and Amazon repositories
- Support Java and Python
- Integrates with GitHub, Bitbucket and AWS CodeCommit

## Profiler

- Helps understand the runtime behaviour of your application
- Example: identify if you application is consuming excessive CPU capacity on a logging routine
- Features
  - Identify and remove code inefficiencies
  - Improve application performance (eg. Reduce CPU utilisation)
  - Decrease compute costs
  - Provides heap summary (identify which objects are using up memory)
  - Anomaly Detection
- Support applications running on aWS or on-premise
- Minimal overheard on application

## Agent Configuration

- MaxStackDepth: the maximum depth of the stacks in the code that is represented in the profile
  - Example: if CodeGuru Profiler finds a method A, which calls method B, which calls method C, which calls method D, then the depth is 4
  - If the MaxStackDepth is set to 2, then the profiler evaluates A and B
- MemoryUsageLimitPercent: the memory percentage used by the profiler
- MinimumTimeForReportingInMilliseconds: the minimum time between sending reports
- ReportingIntervalInMilliseconds: the reporting interval used to report profiles
- SamplingIntervalInMilliseconds: the sampling interval that is used to profile samples
  - Reduce to have a higher sampling rate

# CloudFront

- Content Delivery Network (CDN)
- Improves read performance, content is cached at the edge
- Improves users experience
- 216 Points of Presence globally (edge locations)
- DDoS protection (because worldwide), integration with Shield, AWS Web Application Firewall

## Origins

- S3 bucket
  - For distributing files and caching them at the edge
  - Enhanced security with CloudFront Origin Access Control (OAC)
  - OAC is replacing Origin Access Identity (OAI)
  - CloudFront can be used as an ingress (to upload files to S3)
- Custom Origin (HTTP)
  - Application Load Balancer
  - EC2 Instance
  - S3 website (must first enable the bucket as as static S3 website)
  - Any HTTP backend you want

## CloudFront vs S3 Cross Region Replication

- CloudFront:
  - Global Edge network
  - Files are cached for a TTL (maybe a day)
  - Great for static content that must be available everywhere
- S3 Cross Region Replication:
  - Must be set up for each region you want replication to happen
  - Files are updated in near real-time
  - Read only
  - Great for dynamic content that needs to be available at low-latency in a few regions

### Caching & Caching Policies

- The cache lives at each CloudFront Edge Location
- CloudFront identifies each object in the cache using the Cache Key
- You want to maximise the Cache Hit ratio to minimise requests to the origin
- You can invalidate part of the cache using the CreateInvalidation API

### What is CloudFront Cache Key?

- A unique identifier for every object in the cache
- By default, consists of hostname + resource portion of the URL
- If you have application that serves up content that varies based on user, device, language, location…
- You can add other elements (HTTP headers, cookies, query strings) to the Cache Key using CloudFront Cache Policies

### Cache Policy

- Cache based on:
  - HTTP Headers: none - whitelist
  - Cookies: none - whitelist - include all-except - all
  - Query Strings: none - whitelist - include all-except - all
- Control the TTL (0 seconds to 1 year), can be set by the origin using the Cache-Control headers, Expires header…
- Create your own policy or use Predefined Managed Policies
- All HTTP headers, cookies and query strings that you include in the Cache Key are automatically included in origin requests

### Cache Policy: HTTP Headers

- None:
  - Don’t include any headers in the Cache Key (except default)
  - Headers are not forward (except default)
  - Best caching performance
- Whitelist:
  - Only specified headers included in the Cache Key
  - Specified headers are also forward to Origin

### Cache Policy: Query Strings

- None:
  - Don’t include any query strings in the Cache Key
  - Query strings are not forwarded
- Whitelist:
  - Only specified query strings included in the Cache Key
  - Only specified query strings are forwarded
- Include All-Except
  - Include all query strings in the Cache Key except the specified list
  - All query strings are forwarded except the specified list
- All
  - Include all query strings in the Cache Key
  - All query strings are forwarded
  - Worst caching performance

### CloudFront Policy: Origin Request Policy

- Specify values that you want to include in origin requests without including them in the Cache Key (no duplicated cached content)
- You can include:
  - HTTP headers: none - whitelist - all viewer headers options
  - Cookies: none - whitelist - all
  - Query strings: none - whitelist - all
- Ability to add CloudFront HTTP headers and Custom Headers to an origin request that were not included in the viewer request
- Create your own policy or use Predefined Managed Policies

### Cache Invalidations

- In case you update the back-end origin, CloudFront doesn’t know about it and will only get the refreshed content after the TTL has expired
- However, you can force an entire or partial cache refresh (this bypassing the TTL) by performing a CloudFront Invalidation
- You can invalidate all files (_) or a special path (/images/_)

### Cache Behaviours

- Configure different settings from given URL path pattern
- Example: one specific cache behaviour to images/\*.jpg files on your origin web server
- Route to different kind of origins/origin groups based on the content type or path pattern
  - /images/\*
  - /api/\*
  - /\* (default cache behaviour)
- When adding additional Cache Behaviours, the Default Cache Behaviour is always the last to be processed and is always/\*
- Use case: how to gate access to an S3 bucket because of authentication page

### Geo Restriction

- You can restrict who can access your distribution
  - Allowlist: allow your users to access your content only if they’re in one of the countries on a list of approved countries
  - Blocklist: prevent your users from accessing your content if they’re in one of the countries on a list of banned countries
- The “country” is determined using a 3rd party Geo-IP database
- Use case: copyright laws to control access to content

## CloudFront Signed URL/Cookies

- You want to distribute a paid shared content to premium users over the world
- We can use CloudFront Signed URL/Cookie. We can attack policy with:
  - Includes URL expiration
  - Includes IP ranges to access the data from
  - Trusted signers (which AWS accounts can create signed URLs)
- How long should the URL be valid for?
  - Shared content (movie, music): make it short (a few minutes)
  - Private content (private to the user): you can make it last for years
- Signed URL = access to individual files (one signed URL per file)
- Signed Cookies = access to multiple files (one signed cookie for many files)

## CloudFront Signed URL vs S3 Pre-Signed URL

- CloudFront Signed URL:
  - Allow access to a path, no matter the origin
  - Account wide key-pair, only the root can manage it
  - Can filter by IP, path, date and expiration
  - Can leverage caching features
- S3 Pre-Signed URL:
  - Issue a request as the person who pre-signed the URL
  - Uses the IAM key of the signing IAM principal
  - Limited lifetime

## Key Groups

- Two types of signers:
  - Either a trusted key group (recommended)
    - Can leverage APIs to create and rotate keys (and IAM for API security)
  - An AWS account that contains a CloudFront Key Pair
    - Need to manage keys using the root account and the AWS console
    - Not recommended because you shouldn’t use the root account for this
- In your CloudFront distribution, create one or more trusted key groups
- You can generate your own public/private key
  - The private key is used by your application (eg. EC2) to sign URLs
  - The public key (uploaded) is used by CloudFront to verify URLs

## Advanced Concepts

### Pricing

- CloudFront Edge locations are all around the world
- The cost of data out per edge location varies

### Price Classes

- You can reduce the number of edge locations for cost reduction
- Three price classes:
  - Price class All: all regions - best performance
  - Price Class 200: most regions but excludes the most expensive regions
  - Price class 100: only the least expensive regions

### Multiple Origin

- To route to different kind of origins based on the content type
- Based on the pattern:
  - /images/\*
  - /api/\*
  - /\*

### Origin Groups

- To increase high availability and do failover
- Origin Group: one primary and one secondary origin
- If the primary origin fails, the second one is used

### Field Level Encryption

- Protect user sensitive information through application stack
- Adds an additional layer of security along with HTTPS
- Sensitive information encrypted at the edge close to user
- Uses asymmetric encryption
- Usage:
  - Specify set of fields in POST requests that you want to be encrypted (up to 10 fields)
  - Specify the public key to encrypt them

### Real Time Logs

- Get real-time requests received by CloudFront sent to Kinesis Data Streams
- Monitor, analyse and take actions based on content delivery performance
- Allows you to choose:
  - Sampling rate: percentage of requests for which you want to receive
  - Specific fields and specific Cache Behaviours (path patterns)

# AWS Monitoring, Troubleshooting and Audit: CloudWatch, X-Ray and CloudTrail

## Monitoring in AWS

- AWS CloudWatch
  - Metrics: collect and track key metrics
  - Logs: collect, monitor, analyse and store log files
  - Events: send notifications when certain events happen in your AWS
  - Alarms: react in real-time to metrics/events
- AWS X-Ray
  - Troubleshooting application performance and errors
  - Distributed tracing of microservices
- AWS CloudTrail
  - Internal monitoring of API calls being made
  - Audit changes to AWS Resources by your users

## CloudWatch

### Metrics

- CloudWatch provides metrics for every services in AWS
- Metric is a variable to monitor (CPUUtilisation, NetoworkIn etc.)
- Metrics belong to namespaces
- Dimension is an attribute of a metric (instance id, environment etc.)
- Up to 30 dimensions per metric
- Metrics have timestamps
- Can create CloudWatch dashboards of metrics
- EC2 Detailed monitoring
  - EC2 instance metrics have metrics “every 5 mins”
  - With detailed monitoring (for a cost), you get data “every 1 minute”
  - Use detailed monitoring if you want to scale faster for your ASG
  - The AWS Free Tier allows us to have 10 detailed monitoring metrics
  - Note: EC2 Memory usage is by default not push (must be pushed from inside the instance as a custom metric)

### Custom Metrics

- Possibility to define and send your own custom metrics to CloudWatch
- Example: memory (RAM) usage, disk space, number of logged in users
- Use API call PutMetricData
- Ability o use dimensions (attributes) to segment metrics
  - Instance.id
  - Environment.name
- Metric resolution (StorageResolution API parameter - two possible values):
  - Standard: 1 minute
  - High resolution: 1/5/10/30 second(s) - higher cost
  - Important: accepts metrics data points two weeks in the past and two hours in the future (make sure to configure your EC2 instance time correctly)

### Logs

- Log groups: arbitrary name, usually representing an application
- Log stream: instances with application/log files/containers
- Can define log expiration policies (never expire, 1 day to 10 years)
- CloudWatch Logs can send logs to:
  - Amazon S3 (exports)
  - Kinesis Data Streams
  - Kinesis Data Firehose
  - AWS Lambda
  - OpenSearch
- Logs are encrypted by default
- Can set up KMS-based encryption with your own keys
- Sources
  - SKD, CloudWatch Logs Agent, CloudWatch Unified Agent
  - Elastic Beanstalk: collection of logs from application
  - EBS: collection from containers
  - AWS Lambda: collection from function logs
  - VPC Flow Logs: VPC specific logs
  - API Gateway
  - CloudTrail based on filter
  - Route53: Log DNS queries
- Insights
  - Search and analyse log data stored in CloudWatch Logs
  - Example: find a specific IP inside a log, count occurrences of “ERROR” in your logs
  - Provides a purpose-built query language
    - Automatically discovers fields from AWS services and JSON log events
    - Fetch desired event fields, filter based on conditions, calculate aggregate statistics, sort events, limit number of events
  - Can query multiple Log Groups in different AWS accounts
  - It’s a query engine, not a real-time engine
- S3 Export
  - Log data can take up to 12 hours to become available for export
  - The API is called CreateExportTask
  - Not near-real time or real-time… Use Logs Subscriptions instead
- Subscriptions
  - Get a real-time log events from CloudWatch Logs for processing and analysis
  - Send to Kinesis Data Streams, Kinesis Data Firehose or Lambda
  - Subscription Filter - filter which logs are events delivered to your destination
  - Cross-Account Subscription - send log events to resources in a different AWS account (KDS, KDF)

### Logs - Metric Filters

- CloudWatch Logs can use filter expressions
  - For example, find specific IP inside of a log
  - Or count occurrences of “ERROR” in your logs
  - Metric filters can be used to trigger alarms
- Filters do not retroactively filter data. Filters only publish the metric data points for events that happen after the filter was created
- Ability to specify up to 3 Dimensions for the Metric Filter (optional)

## Agent & Logs Agent

- CloudWatch Logs for EC2
  - By default, no logs from your EC2 machine will go to CloudWatch
  - You need to turn a CloudWatch agent on Ec2 to push the log files you want
  - Make sure IAM permissions are correct
  - The CloudWatch log agent can be set up on-premises too
- Agent & Unified Agent
  - For virtual servers (EC2 instances, on premise servers)
  - CloudWatch Logs Agent
    - Old version of the agent
    - Can only send to CloudWatch Logs
  - CloudWatch Unified Agent
    - Collect additional system-level metrics such as RAM, processes etc.
    - Collect logs to send to CloudWatch Logs
    - Centralised configuration using SSM Parameter Store
- Metrics
  - Collected directly on your Linux server/EC2 instance
  - CPU (active, guest, idle, system, user steal)
  - Dis metrics (free, used, total), Disk IO (writes, reads, bytes, iops)
  - RAM (free, inactive, used, total, cached)
  - Netstat (number of TCP and UDP connections, net packets, bytes)
  - Processes (total, dead, bloqued, idle, running, sleep)
  - Swap space (free, used, used %)
  - Reminder: out-of-the box metrics for EC2 - disk, CPU, network (high level)

## Alarms

- Alarms are used to trigger notifications for any metric
- Various options (sampling, %, max, min etc.)
- Alarm states:
  - OK
  - INSUFFICIENT_DATA
  - ALARM
- Period
  - Length of time in seconds to evaluate the metric
  - High resolution custom metrics: 10 seconds, 30 seconds, or multiples of 60 seconds
- Alarm Targets
  - Stop, terminate, reboot or recover an EC2 instance
  - Trigger Auto Scaling Action
  - Send notification to SNS (from which you can do pretty much anything)
- Composite Alarms
  - CloudWatch Alarms are on a single metric
  - Composite Alarms are monitoring the states of multiple other alarms
  - AND and OR conditions
  - Helpful to reduce “alarm noise” by creating complex composite alarms
- EC2 Instance Recovery
  - Status check
    - Instance status = check the EC2 VM
    - System status = check the underlying hardware
    - Recovery: sam private, public, elastic IP metadata, placement group
- Alarms can be created based on CloudWatch Logs Metrics Filters
- To test alarms and notifications, set the alarm state to Alarm using CLI
- aws cloud watch set-alarm-state —alarm-name “my alarm” —state-value ALARM —state-reason “testing”

## Synthetics (canary)

- Configurable script that monitors your APIs, URLs, websites
- Reproduce what your customers do programmatically to find issues before customers are impacted
- Checks the availability and latency of your endpoints and can store load time data and screenshots of the UI
- Integration with CloudWatch Alarms
- Scripts written in Node.js or Python
- Programmatic access to a headless Google Chrome browser
- Can run once or on a regular schedule
- Blueprints
  - Heartbeat monitor: load URL, store screenshot and an HTTP archive file
  - API Canary: test basic read and write functions of REST APIs
  - Broken Link Checker: check all links inside the URL that you are testing
  - Visual monitoring: compare a screenshot taken during a canary run with a baseline screenshot
  - Canary Recorder: used with CloudWatch Synthetics Recorder (record your actions on a website and automatically generates a script for that)
  - GUI Workflow builder: berries that actions can be taken on your webpage (eg. Test a webpage with a login form)

## Amazon EventBridge (formerly CloudWatch Events)

- Schedule: Cron jobs (scheduled scripts)
- Event patter: event rules to react to a service doing something
- Trigger Lambda functions, send SQS/SNS messages
- Schema Registry
  - EventBridge can analyse the events in your bus and infer the schema
  - The Schema Registry allows you to generate code for your application, that will know in advance how data is structured in the event bus
  - Schema can be versioned
- Resource-based policy
  - Manage permissions for a specific Event Bus
  - Example: allow/deny events from another AWS account or AWS region
  - Use case: aggregate all events from your AWS Organisation in a single AWS account or AWS region

## X-Ray

- Debugging in production the good old way
  - Test locally
  - Ad log statements everywhere
  - Re-deploy in production
- Log formats differ across applications using CloudWatch and analytics is hard
- Debugging: monolith “easy”, distributed services “hard”
- No common views of your entire architecture

### Advantages

- Troubleshooting performance (bottlenecks)
- Understand dependencies in a micro service architecture
- Pinpoint service issues
- Review request behaviour
- Find errors and exceptions
- Are we meeting time SLA?
- Where I am throttled?
- Identify users that are impacted

## Leverages Tracing

- Tracing is an end to end way to follow a “request”
- Each component dealing with the request adds it own “trace”
- Tracing is made of segments (+sub segments)
- Annotations can be added to traces to provide extra-information
- Ability to trace
  - Every request
  - Sample requests (as a % for example or a rate per minute)
- X-Ray Security
  - IAM for authorisation
  - KMS for encryption at rest

### How to Enable it

1. Your code (Java, Python, Go, Node.js, .NET) must import the AWS X-Ray SDK
   - Very little code modification
   - The application SDK will then capture
     - Calls to AWS services
     - HTTP/HTTPS requests
     - Database calls (MySQL, PostgreSQL, DynamoDB)
     - Queue calls (SQS\_
2. Install the X-Ray daemon or enable X-Ray AWS Integration
   - X-Ray daemon works as a low level UDP packet interceptor (Linus/Windows/Mac)
   - AWS Lambda/other AWS services already run the X-Ray Daemon for you
   - Each application musth ave the IAM rights to write data to X-Ray

### Troubleshooting

- If X-Ray is not working on EC2
  - Ensure the EC2 IAM Role has the proper permissions
  - Ensure the EC2 instance is running the X-Ray Daemon
- To enable on AWS Lambda
  - Ensure it has an IAM execution role with proper policy (AWSX-RayWriteOnlyAccess)
  - Ensure that X-Ray is imported in the code
  - Enable Lambda X-Ray Active Tracing

## Instrumentation and Concepts

- Instrumentation means the measure of product’s performance, diagnose errors, and to write trace information
- To instrument you reapplication code, you use the X-Ray SDK
- Many SDK require only configuration changes
- You can modify your application code to customise and annotate the data that the SDK sends to X-Ray, using interceptors, filters, handlers, middleware…
- Segments: each application/service will send them
- Subsegments: if you need more details in your segment
- Trace: segments collected together to form an end-to-end trace
- Sampling: decrease the amount of requests sent to X-ray, reduce costs
- Annotations: key value pairs used to index traces and use with filters
- Metadata: key value pairs, not indexed, not used for search
- The X-Ray daemon/agent has a config to send traces cross account
  - Make sure the IAM permissions are correct - the agent will assume the role
  - This allows you to have a central account for all your application tracing

## Sampling Rules

- With sampling rules, you control the amount of data that you record
- You an modify sampling rules without changing your code
- By default, the X-Ray SDK records the first request each second, and five percent of any additional requests
- One request per second is the reservoir, which ensures that at least one trace is recorded each second as long as the service is serving requests
- Five percent is the rate at thwack additional request beyond the reservoir size are sampled
- Custom sampling rules: you can create your own rules with the reservoir and rate

## APIs

- PutTraceSegements: uploads segment documents to AWS X-Ray
- PutTelemetryRecords: used by the AWS X-RAy daemon to upload telemetry
  - SegmentsReceuvedCount, SegmentsRejectedCounts, BackendConnectionErrors
- GetSamplingRules: Retrieve all sampling rules (to know what/when to send)
- GetSamplingTargets & GetSamplingStatisticSummaries: advanced
- The X-Ray daemon needs to have an IAM policy authorising the correct API calls to function correctly
- GetServiceGraph: main graph
- BatchGetTraces: retrieves a list of traces specified by ID. Each trace is a collection of segment documents that originates from a single request
- GetTraceSummaries: retrieves IDs and annotations for traces available for a specified time frame using an optional filter. To get the full traces, pass the trace IDs to BatchGetTraces
- GetTraceGraph: retrieves a service graph for one or more specific trace IDs

## X-Ray with Beanstalk

- AWS Elastic Beanstalk platforms include the X-Ray daemon
- You can run the daemon by setting an option in the Elastic Beanstalk console or with a configuration file (in .ebextensionxray-daemon.config)
- Make sure to give your instance profile the correct IAM permissions so that the X-Ray daemon can function correctly
- Then make sure your application code is instrumented with the X-Ray SDK
- Note: The X-Ray daemon is not provided for Multicontainer Docker

## AWS Destro for OpenTelemetry

- Secure, production-ready AWS-supported distribution of the open-source OpenTelemetry project
- Provides a single set of APIs, libraries, agents and collector services
- Collects distributed traces and metrics from your apps
- Collects metadata from your AWS resources and services
- Auto-instrumentation Agents to collect traces without changing your code
- Send traces and metrics to multiple AWS services and partner solutions
  - X-Ray, CloudWatch, Prometheus
- Instrument your apps running on AWS (eg. EC2, ECS, EKS, Fargate, Lambda) as well as on-premises
- Migrate from X-Ray to AWS Distro for Telemetry if you want to standardise with open-source APIs from Telemetry or send traces to multiple destinations simultaneously

## CloudTrail

- Provides governance, compliance and audit for your AWS account
- CloudTrail is enabled by default
- Get a history of events/API calls made within your AWS account by:
  - Console
  - SDK
  - CLI
  - AWS services
- Can put logs from CloudTrail into CloudWatch Logs or S3
- A trail can be applied to All Regions (default) or a single Region
- If a resource is deleted in AWS, investigate CloudTrail first

### Events

- Management Events
  - Operations that are performed on resources in your AWS account
  - Examples
    - Configuring security (IAM AttachRolePolicy)
    - Configuring rules for routing data (Amazon EC2 CreateSubnet)
    - Setting up logging (AWS CloudTrail CreateTrail)
  - By default, trails are configured to log management events
  - Can separate Read Event s(that don’t modify resources) from Write Events (that may modify resources)
- Data Events
  - By default, data events are not logged (because high volume operations)
  - Amazon S3 object-level activity (eg. GetObject, DeleteObject, PutObject): can separate Read and Write Events
  - AWS Lambda function execution activity (the Invoke API)
- CloudTrail Insights
  - Enable CloudTrail Insights to detect unusual activity in your account
    - Inaccurate resources provisioning
    - Hitting service limits
    - Bursts of AWS IAM actions
    - Gaps in periodic maintenance activity
  - CloudTrail Insights analyses normal management events to create a baseline
  - And then continuously analyses write events to detect unusual patterns
    - Anomalies appear in the CloudTrail console
    - Even t is sne tot Amazon S3
    - An EventBridge event is generated (for automation needs)
- CloudTrail events retention
  - Evens are stored for 90 days in CLoudTrail
  - To keep events beyond this period, log them to S3 and use Athena

## CloudTrail vs CloudWatch vs X-Ray

### CloudTrail

- Auto API calls made by users, services, AWS console
- Useful to detect unauthorised calls or root cause of changes

## CloudWatch

- CloudWatch Metrics over time for monitoring
- CloudWatch Logs for storing application log
- CloudWatch Alarms to send notifications in case of unexpected metrics

## X-Ray

- Automated Trace Analysis & Central Service Map Visualisation
- Latency, errors and fault analysis
- Request tracking across distributed systems

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

# EC2 Basics

- EC2 is one of the most popular AWS offerings
- Elastic Compute Cloud (infrastructure as a service)
- Capability of:
  - Renting virtual machines (EC2)
  - Storing data on virtual drives (EBS)
  - Distributing load across machines (ELB)
  - Scaling the services using an auto-scaling group (ASG)
- Sizing and configuration options:
  - Operating System (OS): Linux, Windows or Mac OS
  - How much compute power & cores (CPU)
  - How much random-access memory (RAM)
  - How much storage spaces: Network attached (EBS and EFS) and hardware (EC2 instance store)
  - Network card: speed of the card, public IP address
  - Firewall rules: security group
  - Bootstrap script (configure at first launch): EC2 User Data
- EC2 User Data:
  - It is possible to bootstrap our instances using an EC2 User data script
  - Bootstrapping means launching commands when a machine starts
  - That script is only run once at the instance first start
  - EC2 user data is used to automate boot tasks such as:
    - Installing updates
    - Installing software
    - Downloading common files from the Internet
    - Anything you can think of
  - The EC2 User Data Script runs with the root user

## EC2 Instance Types Basics

- You can use different types of EC2 instances that are optimised for different use cases
- AWS has the following naming convention: m5.2xlarge
  - m: instance class
  - 5: generation (AWS improves them over time)
  - 2xlarge: size within the instance class
- General purpose:
  - Great for a diverse workload such as web servers or code repositories
  - Balance between compute, memory and networking
- Compute optimised:
  - Great for compute-intensive tasks that require high performance processors
  - Batch processing workloads
  - Media transcoding
  - High performance web servers
  - High performance computing (HPC)
  - Scientific modelling & machine leaning
  - Dedicated gaming servers
- Memory optimised:
  - Fast performance for workloads that process large data sets in memory
  - Use cases:
    - High performance, relational/non-relational databases
    - Distributed web scale cache stores
    - In-memory databases optimised for BI (business intelligence)
    - Applications performing real-time processing of big unstructured data
- Storage optimised:
  - Great for storage-intensive tasks that require high, sequential read and write access to large data sets on local storage
  - Use cases:
    - High frequency online transaction processing (OLTP) systems
    - Relational & NoSQL databases
    - Cache for in-memory databases (for example, Redis)
    - Data warehousing applications
    - Distributed file systems

## Security Groups and Classic Ports

- Security Groups are fundamental of network security in AWS
- They control how traffic is allowed into or out of our EC2 instances
- Security groups only contain allow rules
- Security groups rules can reference by IP or by security group
- Security groups are acting as a "firewall" on EC2 instances
- They regulate:
  - Access to ports
  - Authorised IP ranges - IPv4 and IPv6
  - Control of inbound network (from other to the instance)
  - Control of outbound network (from the instance to other)
- Security groups:
  - Can be attached to multiple instances
  - Locked down to a region/VPC combination
  - Does live "outside" the EC2 - if traffic is blocked the EC2 instance won't see it
  - It's good to maintain one separate security group for SSH access
  - If your application is not accessible (time out), then it's a security group issue
  - All inbound traffic is blocked by default
  - All outbound traffic is authorised by default
- Classic ports:
  - 22: SSH (secure Shell) - log into a Linux instance
  - 21: FTP (file transfer protocol) - upload files into a file share
  - 22: SFTP (secure file transfer protocol) - upload files using SSH
  - 80: HTTP - access insecure websites
  - 443: HTTPS - access secure websites
  - 3389: RDP (Remote Desktop Protocol) - log into a Windows instance

## EC2 Instance Connect

- Allows you to do a browser based SSH session

## EC2 Purchasing Options

- On-demand instances: short workload, predictable pricing, pay by second
- Reserved (1 & 3 years)
  - Reserved instances - long workloads
  - Convertible reserved instances - long workloads with flexible instances
- Savings plans (1 & 3 years): commitment to an amount of usage, long workload
- Spot instances: short workloads, cheap, can lose instances (less reliable)
- Dedicated hosts: book an entire physical server, control instance placement
- Dedicated instances: no other customers will share your hardware
- EC2 On Demand:
  - Pay for what your use: Linux/windows is billed per second, after the first minute and all other operating systems billed per
    hour
  - Has the highest cost but no upfront payment
  - No long-term commitment
  - Recommended for short-term and un-interrupted workloads, where you can't predict how the application will behave
- EC2 Reserved instances:
  - Up to 72% discount compared to On-Demand
  - You reserve a specific instance attribute (Instance Type, Region, Tenancy, OS)
  - Reservation period: 1 year (+ discount) or 3 years (+++ discount)
  - Payment options: no upfront (+), partial upfront (++), all upfront (+++)
  - Reserved instance's scope: regional or zonal (reserve capacity in an AZ)
  - Recommended for steady-state usage applications (think database)
  - You can buy and sell in the reserved instance market place
  - Convertible Reserved Instance
    - Can change the EC2 instance type, instance family, OS, scope and tenancy
    - Up to 66% discount
- EC2 Savings Plan
  - Get a discount based on long term usage (up to 72% - same as RIs)
  - Commit to a certain type of usage
  - Usage beyond EC2 Savings Plans is billed at the On-Demand price
  - Locked to a specific instance family & AWS Region (eg. M5 in us-east-1)
  - Flexible across:
    - Instance Size (eg.. m5.xlarge, m5.2xlarge)
    - OS (Linux, Windows etc.)
    - Tenancy (Host, Dedicated, Default)
- EC2 Spot Instances
  - Can get discount of up to 90% compared to On-Demand
  - Instances that you can "lose" at any point of time if your max price is less than the current spot price
  - The MOST cost efficient instance in AWS
  - Useful for workloads that are resilient for failure:
    - Batch jobs
    - Data analysis
    - Image processing
    - Any distributed workloads
    - Workloads with a flexible start and end time
  - Note suitable for critical jobs or databases
- EC2 Dedicated Hosts
  - A physical server with EC2 instance capacity fully dedicated to your use
  - Allows you to address compliance requirements and use your existing server-bound software licenses (per-socket, per-core, pe-vm software licenses)
  - Purchasing options:
    - On-demand: pay per second for active Dedicated Host
    - Reserved: 1 or 3 years (no upfront, partial upfront, all upfront)
  - The most expensive option
  - Useful for software that have complicated licensing model (BYOL - Bring Your Own License)
  - Or for companies that have strong regulatory or compliance needs
- EC2 Dedicated Instances
  - Instances run on hardware that's dedicated to you
  - May share hardware with other instances in same account
  - No control over instance placement (can move hardware after stop/start)
- EC2 Capacity Reservations
  - Reserve On-Demand instances capacity in a specific AZ for any duration
  - You always have access to EC2 capacity when you need it
  - No time commitment (create/cancel anytime), no billing discounts
  - Combine with Regional Reserved Instances and Savings Plans to benefit from billing discounts
  - You're charged at On-Demand rate whether you run instances or not
  - Suitable for short-term, uninterrupted workloads that needs to be in a specific AZ
- On demand: coming and staying in a resort whenever we like, we pay the full price
- Reserved: like planning ahead and if we plan to stay for a long time, we may get a good discount
- Savings plan: pay a certain amount per hour for certain period and stay in any room type
- Spot instances: the hotel allows people to bid for the empty rooms and the highest bidder keeps the rooms, and you can get kicked out at any time
- Dedicated Hosts: we book an entire building of the resort
- Capacity reservations: you book a room for a period with full price even when you don't stay in it

# EC2 Instance Storage

## EBS

- An EBS (Elastic Block Store) Volume is a network drive you can attach to your instances while they run
- It allows your instances to persist data, even after their termination
- They can only be mounted to one instance at a time (at the CCP level)
- They are bound to a specific availability zone
- Analogy: think of them as a "network USB stick"
- Free tier: 30 GB of free EBS storage of type General Purpose (SSD) or Magnetic per month

### EBS Volume

- It's a network drive (not a physical drive)
  - It uses the network to communicate the instance, which means there might be a bit of latency
  - It can be detached from an EC2 instance and attached to another one quickly
- It's locked to an availability zone (AZ)
  - An EBS volume in us-east-1a cannot be attached to us-east-1b
  - To move a volume across, you first need to snapshot it
- Have a provisioned capacity (size in GBs and IOPS)
  - You get billed for all the provisioned capacity
  - You can increase the capacity of the drive over time

### EBS - Delete on Termination Attribute

- Controls the EBS behaviour when an EC2 instance terminates
  - By default, the root EBS volume is deleted (attribute enabled)
  - By default, any other attached EBS volume i snot deleted (attribute disabled)
- This can be controlled by the AWS console/AWS CLI
- Use case: preserve root volume when instance is terminated

### EBS Snapshots

- Make a backup (snapshot) of your EBS volume at a point in time
- Not necessary to detach volume to do snapshot, but recommended
- Can copy snapshots across AZ or region
- EBS Snapshot Archive
  - Move a Snapshot to an "archive tier" that is 75% cheaper
  - Takes within 24 to 72 hours for restoring the archive
- Recycle Bin for EBS Snapshots
  - Setup rules to retain deleted snapshots so you can recover them after an accidental deletion
  - Specify retention (for 1 day to 1 year)
- Fast Snapshot Restore (FSR)
  - Force full initialisation of snapshot to have no latency on the first use (£££)

## AMI

- AMI = Amazon Machine Image
- AMI are a customisation of an EC2 instance
  - You add your own software, configuration, operating system, monitoring etc
  - Faster boot/configuration time because all your software is pre-packaged
- AMI are built for a specific region (and can be copied across regions)
- You can launch EC2 instances from:
  - A Public AMI (AWS provided)
  - Your own AMI (you make and maintain them yourself)
  - An AWS Marketplace AMI (an AMI someone else made and potentially sells)
- AMI Process (from an EC2 instance)
  - Start an EC2 instance and customise it
  - Stop the instance (For data integrity)
  - Build an AMI - this will also create EBS snapshots
  - Launch instances from other AMIs

## EC2 Instance Store

-EBS volumes are network drives with good but "limited" performance

- If you need a high-performance hardware disk, use EC2 Instance Store
- Better I/O performance
- EC2 Instance Store lose their storage if they're stopped (ephemeral)
- Good for buffer/cache/scratch data/temporary content
- Risk of data loss if hardware fails
- Backups and replication are your responsibility

## EBS Volume Types

- EBS volumes come in 6 types
  - gp2/gp3 (SSD): general purpose SSD volumes that balances price and performance for a wide variety of workloads
  - io1/io2 Block Express (SSD): Highest-performance SSD volume for mission-critical low-latency or high-throughput workloads
  - st1 (HDD): low cost HDD volume designed for frequently accessed, throughput-intensive workloads
  - sc1 (HDD): lowest cost HDD volume designed for less frequently accessed workloads
- EBS volumes are characterised in size | throughput | IOPS (I/O Ops Per Sec)
- When in doubt always consult the AWS documentation
- Only gp2/gp3 and io I/ oi 2 Block Express can be used as boot volumes
- Use cases:
  - General Purpose SSD
    - Cost effective storage, low-latency
    - System boot volumes, virtual desktops, development and test environments
    - 1 GiB = 16 TiB
    - gp3:
      - Baseline of 3,000 IOPS and throughput of 125 MiB/s
      - Can increase IOPS up to 16,000 and throughput up to 1000 MiB/s independently
    - gp2:
      - Small gp2 volumes can burst IOPS to 3,000
      - Size of the volume and IOPS are linked, max IOPS is 16,000
      - 3 IOPS per GB means 5,334 GB we are at the max IOPS
  - Provisioned IOPS (PIOPS) SSD:
    - Critical business applications with sustained IOPS performance
    - Or applications that need more than 16,000 IOPS
    - Great for databases workloads (sensitive to storage perf and consistency)
    - io 1 (4 GiB - 16 TiB):
      - Max PIOPS: 64,000 for Nitro EC2 instances & 32,000 for other
      - Can increase PIOPS independently from storage size
  - io2 Block Express (4 GiB - 64 TiB):
    - Sub-millisecond latency
    - Max PIOPS: 256,000 with an IOPS: GIB ration of 1,000:1
  - Supports EBS Multi-attach
  - Hard Disk Drives (HDD):
    - Cannot be a boot volume
    - 125 GiB to 16 TiB
    - Throughput optimised HDD (st 1):
      - Big data, data warehouses, log processing
      - Max throughput 500 MiB/s - max IOPS 500
    - Cold HDD (sc 1):
      - For data that is infrequently accessed
      - Scenarios where lowest cost is important
      - Max throughout 250 MiB/s - max IOPS 250

## EBS Multi-Attach

- Attach the same EBS volume to multiple EC2 instances in the same AZ
- Each instance has full read & write permissions to the high-performance volume
- Use cases:
  - Achieve higher application availability in clustered Linux applications (eg. Teradata)
  - Applications must manage concurrent write operations
- Up to 16 EC2 instances at a time
- Must use a file system that's cluster-aware (not XFS, EXT4, etc)

## Amazon EFS - Elastic File System

- Use cases: content management, web serving, data sharing, Wordpress
- Uses NFSv4.1 protocol
- Uses security group to control access to EFS
- Compatible with Linux based AMI (not Windows)
- Encryption at rest using KMS)
- POSIX file system (~Linux) that has a standard file API
- File system scales automatically, pay-per-user, no capacity planning
- Performance & Storage Classes:
  - EFS Scale
    - 1000s of concurrent NFS clients, 10 GB+/s throughput
    - Grow to petabyte-scale network file system, automatically
  - Performance Mode (set at EFS creation time)
    - General Purpose (default) - latency-sensitive use cases (web server, CMS etc.)
    - Max I/O - higher latency, throughput, highly parallel (big data, media processing)
  - Throughput mode
    - Bursting - 1 TB = 50 MiB/s + burst of up to 100MiB/s
    - Provisioned - set your throughput regardless of storage size, eg. 1 GiB/s for 1 TB storage
    - Elastic - automatically scales throughput up or down based on your workloads
      - Up to 3GiB/s for reads and 1GiB/s for writes
      - Used for unpredictable workloads
  - Storage classes
    - Storage Tiers (lifecycle management feature - move files are N days)
      - Standard: for frequently accessed files
      - Infrequent access (EGS-1A): cost to retrieve files, lower price to store
      - Archive: rarely accessed data (few times each year), 50% cheaper
      - Implement lifecycle policies to move files between storage tiers
    - Availability and durability
      - Standard: multi-az, great for prod
      - One zone: one az, great for dev, backup enabled by default, compatible with IA (EFS One Zone-1A)
    - Over 90% in cost savings

## EFS vs EBS

### EBS Volumes...

- One instance (except multi-attach io 1/ io2)
- Are locked at the availability zone (AZ) level
- gp2: IO increases if the disk size increases
- gp3 & io 1: can increase IO independently
- To migrate an EBS volume across AZ:
  - Take a snapshot
  - Restore the snapshot to another AZ
  - EBS backups use IO and you shouldn't run them while your application is handling a lot of traffic
- Root EBS Volumes of instances get terminated by default if the EC2 instance gets terminated (you can disable it)
- Mounting 100s of instances across AZ
- EFS share website files (WordPress)
- Only for Linux Instances (POSIX)
- EFS has a higher price point than EBS
- Can leverage Storage Tiers for cost savings
- Reminder: EFS vs EBS vs Instance Store
  # AWS Fundamentals: ELB + ASG

## High Availability and Scalability

- Scalability means that an application/system can handle greater loads by adapting
- There are two kinds of scalability, vertical and horizontal (elasticity)
- Scalability is linked but different to High Availability

## Vertical Scalability

- Vertically scalability means increasing the size of the instance
- For example, your application runs on t2.micro
- Scaling that application vertically means running it on a t2.large
- Vertical scalability is very common for non distributed systems, such as a database
- RDS, ElastiCache are services that can scale vertically
- There’s usually a limit to how much you can vertically scale (hardware limit)

## Horizontal Scalability

- Horizontal Scalability means increasing the number of instances/systems for your application
- Horizontal scaling implies distributed systems
- This is very common for web applications/modern applications
- It’s easy to horizontally scale thanks to cloud offers such as Amazon EC2

## High Availability

- High availability usually goes hand in hand with horizontal scaling
- High availability means running your application/system in at least 2 data centres (availability zones)
- The goal of high availability is to survive a data centre loss
- The high availability can be passive (for RDS Multi AZ for example)
- The high availability can be active (for horizontal scaling)

### High Availability & Scaling for EC2

- Vertical scaling: increase instance size (scale up/down)
  - From t2.nano to u-12tb1.metal
- Horizontal scaling: increase number of instances (scale out/in)
  - Auto scaling group
  - Load balancer
- High availability: run instances for the same application across multi AZ
  - Auto scaling group multi az
  - Load balancer multi az

## Elastic Load Balancing (ELB)

- Load Balances are servers that forward traffic to multiple servers (ec2 instances) downstream
- Spread load across multiple downstream instances
- Expose a single point of access (DNS) to your application
- Seamlessly handle failures of downstream instances
- Do regular health checks to your instances
- Provide SSL termination (HTTPS) for your websites
- Enforce stickiness with cookies
- High availability across zones
- Separate public traffic from private traffic

### Elastic Load Balancer

- An Elastic Load Balancer is a managed load balancer
  - AWS guarantees that it will be working
  - AWS takes care of upgrades, maintenance, high availability
  - AWS provides only a few configuration knobs
- It costs less to set up your own load balancer but it will be a lot more effort on your end
- It is integrated with many AWS offers/services
  - EC2, EC2 Auto Scaling Groups, Amazon ECS
  - AWS Certificate Manager (ACM), CloudWatch
  - Route 53, AWS WAF, AWS Global Accelerator

### Health Checks

- Health Checks are crucial for Load Balancers
- They enable the load balancer to know if instances it forwards traffic to are available to reply to requests
- The health check is done on a port and a route (/health is common)
- If the response is not 200 (OK), then the instance is unhealthy

### Types of Load Balancers on AWS

- Classic load balancer (v1 - old generation) - 2009 - CLB
  - HTTP, HTTPS, TCP, SSL (secure TCP)
- Application Load Balancer (v2 - new generation) - 2016 - ALB
  - HTTP, HTTPS, WebSocket
- Network Load Balancer (v2 - new generation) - 2017 - NLB
  - TCP, TLS (secure TCP), UDP
- Gateway Load Balancer - 2020 - GWLB
  - Operates at layer 3 (network layer) - IP Protocol
- Overall, it’s recommended to use the newer generation load balancer as they provide more features
- Some load balancers can be setup as internal (private) or external (public) ELBs

## Sticky Sessions (Session Affinity)

- It is possible to implement stickiness so that the same client is always redirected to the same instance behind a load balancer
- This works for Classic Load Balancer, Application Load Balancer and Network Load Balancer
- The “cookie” used for stickiness has an expiration date you control
- Use case: make sure the user doesn’t lose his session data
- Enabling stickiness may bring imbalance to the load over the backend EC2 instances
- Cookie names:
  - Application-based cookies
    - Custom cookie
      - Generated by the target
      - Can include any custom attribute required by the application
      - Cookie name must be specified individually for each target group
      - Doesn’t use AWSALB, AWSALBAPP or AWSALBTG (reserved for use by the eLB)
    - Application cookie
      - Generated by the load balancer
      - Cookie name is AWSALAPP
  - Duration-based Cookies
    - Cookie generated by the load balancer
    - Cookie name is AWSALB for ALB, AWSELB for CLB

## Cross Zone Balancing

- With cross zone load balancing: each load balancer instance distributes evenly across all registered instances in all AZ
- Without cross zone load balancing: requests are distributed in the instances of the node of the Elastic Load Balancer
- Application Load Balancer
  - Enabled by default (can be disabled at the target group level)
  - No charge for inter AZ data
- Network Load Balancer & Gateway Load Balancer
  - Disabled by default
  - You pay charge ($) for inter AZ data if enabled
- Classic Load Balancer
  - Disabled by default
  - No charges for inter AZ data if enabled

## SSL Certifications

- An SSL Certificate allows traffic between your clients and your load balancer to be encrypted in transit (in-flight encryption)
- SSL refers to secure sockets layer, used to encrypt connections
- TLS refers to transport layer security, which is a newer version
- Nowadays, TLS certificates are mainly used, but people still refer as SSL
- Public SSL certificates are issued by Certificate Authorities (CA)
- Comodo, Symantec, GoDaddy, GlobalSign, Digicert, Letsencrypt etc.
- SSL certificates have an expiration date (you set) and must be renewed
- The load balancer uses an X.509 certificate (SSL/TLS server certificate)
- You can manage certificates using ACM (AWS Certificate Manager)
- You can create upload your own certificates alternatively
- HTTPS listener:
  - You must specify a default certificate
  - You can add an optional list of certs to support multiple domains
  - Clients can use SNI (server name indication) to specify the hostname they reach
  - Ability to specify a security policy to support older versions of SSL/TLS (legacy clients)
- SNI solves the problem of loading multiple SSL certificates onto one web server (to serve mutable websites)
- It’s a “newer” protocol, and requires the client to indicate the hostname of the target server in the initial SSL handshake
- The server will then find the correct certificate, or return the default one
- Note: only works for ALB & NLB (newer generation) and CloudFront, doesn’t work for CLB (older generation)
- Classic Load Balancer (v1)
  - Support only one SSL certificate
  - Must use multiple CLB for multiple hostname with multiple SSL certificates
- Application Load Balancer (v2)
  - Supports multiple listeners with multiple SSL certificates
  - Uses Server Name Indication (SNI) to make it work
- Network Load Balancer (v2)
  - Supports multiple listeners with multiple SSL certificates

## Connection Draining

- Feature naming
  - Connection Draining - for CLB
  - Deregistration Delay - for ALB & NLB
- Time to complete “in-flight requests” while the instance is de-registering or unhealthy
- Stops sending new requests to the EC2 instance which is de-registering
- Between 1 to 3600 seconds (default is 300 seconds)
- Can be disabled (set value to 0)
- Set to a low value if your requests are short

## Application Load Balancer (ALB)

- Application load balancers is Layer 7 (HTTP)
- Load balancing to multiple HTTP applications across machines (target groups)
- Load balancing to multiple applications on the same machine (eg. Containers)
- Support for HTTP/2 and WebSocket
- Support redirects (from HTTP to HTTPS for example)
- Routing tables to different target groups
  - Routing based on path in URL
  - Routing based on hostname in URL
  - Routing based on query strings and headers
- ALB are a great fit for micro services & container based application
- Has a port mapping feature to redirect to a dynamic port in ECS
- In comparison, we’d need multiple Classic Load Balancer per application
- Fixed hostname (xxx.region.elb.amazonaws.com)
- The application servers don’t see the IP of the client directly
  - The true IP of the client is inserted in the header x-forwarded-for
  - We can also get post (x-forwarded=port) and port (x-forwarded-proto)

## Target Groups

- EC2 instances (can be managed by an Auto Scaling Group) - HTTP
- ECS tasks (managed by ECS itself) - HTTP
- Lambda functions - HTTP request is translated into JSON event
- IP Addresses - must be private IPs
- ALB can route to multiple target groups
- Health checks are at the target group level

## Network Load Balancer (NLB)

- Layer 4
- Forward TCP & UDP traffic to your instances
- Handle millions of requests per seconds
- Less latency ~100 ms (vs 400 ms for ALB)
- NLB has one static IP per AZ and support assigning Elastic IP (helpful for whitelisting specific IP)
- NLBs are used for extreme performance (TCP or UDP traffic)
- Not included in the AWS free tier

## Target Groups

- EC2 instances
- IP addresses - must be private IPs
- Application Load Balancer
- Health Checks support the TCP, HTTP and HTTPS protocols

## Gateway Load Balancer (GLB)

- Deploy, scale and manage a fleet of 3rd party network virtual appliances in AWS
- Example: firewalls, intrusion detection and prevention systems, deep packet inspection systems, payload manipulation
- Operates at Layer 3 (network layer) - IP packets
- Combines the following functions:
  - Transparent Network Gateway: single entry/exit for all traffic
  - Load Balancer: distributes traffic to your virtual appliances
- Uses the GENEVE protocol on port 6081

## Target Groups

- EC2 instances
- IP addresses - must be private

## Auto Scaling Groups (ASG)

- In real life, the load on your websites and applications can change
- In the cloud, you can create and get rid of servers very quickly
- The goal of an auto-scaling group (ASG) is to :
  - Scale out (add EC2 instances) to match an increased load
  - Scale in (remote EC2 instances) to match a decreased load
  - Ensure we have a minimum and a maximum number of EC2 instances running
  - Automatically register new instances to a load balancer
  - Re-create an EC2 instance in case a previous one is terminated (eg. If unhealthy)
- ASG are free (you only pay for the underlying EC2 instances)

## Group Attributes

- A launch template (older “Launch Configurations” are deprecated)
  - AMI + Instance Type
  - EC2 User Data
  - EBS Volumes
  - Security Groups
  - SSH Key Pair
  - IAM Roles for your EC2 instances
  - Network + Subnets information
  - Load Balancer Information
- Min Size/Max Size/Initial Capacity
- Scaling Policies

## CloudWatch Alarms & Scaling

- It is possible to scale an ASG based on CloudWatch alarms
- An alarm monitors a metric (such as Average CPU, or a custom metric)
- Metrics such as Average CPU are computed for the overall ASG instances
- Based on the alarm:
  - We can create scale-out policies (increase the number of instances)
  - We can create scale-in policies (decrease the number of instances)

## Scaling Policies

- Dynamic Scaling
  - Target Tracking Scaling
    - Simple to set-up
    - Example: I want the average ASG CPU to stay at around 40%
  - Simple/Set Scaling
    - When a CloudWatch alarm is triggered (example CPU > 70%), then add 2 units
    - When a CloudWatch alarm is triggered (example CPU < 30%), then remove 1
- Scheduled Scaling
  - Anticipate a scaling based on known usage patterns
  - Example: increase the min capacity to 10 at 5pm on Fridays
- Predictive scaling: continuously forecast load and schedule scaling ahead
- Good metrics to scale on:
  - CPUUtilisation: average CPU utilisation across your instances
  - RequestCountPerTarget: to make sure the number of requests per EC2 instances is stable
  - Average Network In/Out (if you’re application is network bound)
  - Any custom metric (that you push using CloudWatch)
- Scaling cooldowns:
  - After a scaling activity happens, you are in the cooldown period (default 300 seconds)
  - During the cooldown period, the ASG will not launch or terminate additional instances (to allow for metrics to stabilise)
  - Advice: Use a read-to-use AMI to reduce configuration time in order to be serving requests faster and reduce the cooldown period

## Instance Refresh

- Goal: update launch template and then re-creating all EC2 instances
- For this we can use the native feature of Instance Refresh
- Setting of minimum healthy percentage
- Specify warm-up time (how long until the instance is ready to use)

# IAM & AWS CLI

- IAM: Identity and Access Management
- A global service

## Users, Groups, Policies

- Root account created by default, shouldn't be shared or used
- Users: people within an organisation (can be grouped)
- Groups: only contain users and not other groups
- Users don't have to belong to a group
- One user can belong to multiple groups
- Groups are usually set up for permissions
- Users or groups can be assigned JSON documents called policies (these policies define the permissions set for users)
- Best practise is to apply the least privilege principle (don't give more permission than a user needs)

## IAM Policies

- Inheritance: if a user belongs to two groups, they will inherit policies from both groups
- Policies structure consists of:
  - Version number
  - ID (optional identifier for the policy)
  - Statement (required - one or more individual statements) which consist of:
    - Sid: an optional identifier for the statement
    - Effect: whether the statement allows or denies access (allow/deny)
    - Principal: account/user/role to which this policy applied to
    - Action: list of actions this policy allows or denies
    - Resource: list of resources to which the action applies to
    - Condition: conditions for when this policy is in effect (optional)

## MFA

- Password policy: the stronger the password, the higher the security for your account
  - Set a minimum password length
  - Require specific character types: uppercase, lowercase, numbers and non-alphanumeric characters
  - Allow all IAM users to change their own passwords
  - Require users to change their password after some time (password expiration)
  - Prevent password re-use
- Users have access to your account and can possibly change configurations or delete resources in your AWS account
- You want to protect your Root Accounts and IAM users
- MFA = passwords you know + security device you own
- Main benefit: if a password is stolen or hacked, the account is not compromised
- Device options in AWS: virtual MFA device (authy, google authenticator etc), universal 2nd factor (U2F) security key (eg. YubiKey), hardware key fob MFA device or hardware key fob MFA device for AWS GovCloud (US)
- Virtual MFA devices have support for multiple token on a single device
- Universal 2nd Factor Security Keys support multiple root and IAM users using a single security key

## AWS Access Keys, CLI and SDK

- AWS can be accessed via:
  - AWS Management Console (protected by password + MFA)
  - AWS Command Line Interface (CLI): protected by access keys
  - AWS Software Developer Kit (SDK): for code - protected by access keys
- Access keys are generated through the AWS Console
- Users manage their own access keys
- Access Keys are secret, like password and shouldn't be shared
- Access Key ID is usually the username
- Secret Access Key is usually the password
- AWS CLI:
  - A tool that enables you to interact with AWS services using command in your command-line shell
  - Direct access to the public APIs of AWS services
  - You can develop scripts to manage your resources
  - It's open source (github.com/aws/aws-cli)
  - Alternative to using AWS Management Console
- AWS SDK:
  - AWS Software Development Kit (AWS SDK)
  - Language-specific APIs (set of libraries)
  - Enables you to access and manage AWS services programmatically
  - Embedded within your application
  - Supports:
    - SDKs (JavaScript, Python, PHP, .NET, Ruby, Java, Go, Node.js, C++)
    - Mobile SDKs (Android, iOS)
    - IoT Device SDKs (Embedded C, Arduino)
  - Example: AWS CLI is built on AWS SDK for Python

## AWS CloudShell

- Alternative to using the command line
- The icon on the top nav bar on the right hand side (it's globally available)
- Example commands:
  - aws iam list-users
  - aws iam list-users --region
- Full repo via shell commands like ls, cd, echo etc.
- Can upload your own files

## IAM Roles for AWS Services

- Some AWS services will need to perform actions on your behalf
- To do so, assign permission to AWS services via IAM Roles
- Common roles:
  - EC2 Instance Roles
  - Lambda Function Roles
  - Roles for CloudFormation

## Security Tools (Audit)

- IAM Credentials Report (account-level): a report that lists all your account's users and the status of their various credentials
- IAM Access Advisor (user-level): shows the service permissions granted to a user and when those services were last accessed

## IAM Best Practices

- Don't use the root account except for AWS account setup
- One physical user = one aws user
- Assign users to groups and assign permission to groups
- Create a strong password policy
- Use and enforce the use of Multi Factor Authentication (MFA)
- Create and use Roles for giving permissions to AWS services
- Use Access Keys for Programmatic Access (CLI/SDK)
- Audit permissions of your account using IAM Credentials Report & IAM Access Advisor
- Never share IAM users & Access Keys

## Shared Responsibility Model for IAM

- AWS' responsibilities:
  - Infrastructure (global network security)
  - Configuration and vulnerability analysis
  - Compliance validation
- Your responsibilities:
  - Users, Groups, Roles, Policies management and monitoring
  - Enable MFA on all accounts
  - Enable MFA on all accounts
  - Rotate all your keys often
  - Use IAM tools to apply appropriate permissions
  - Analyse access patterns and review permissions

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

# Route 53

## What is DNS

- Domain Name System which translates the human friendly hostnames into the machine IP addresses
- www.google.com => 172.217.18.36
- DNS is the backbone of the Internet
- DNS uses hierarchical naming structure
  - .com
  - example.com
  - www.example.com
  - api.example.com

## Terminologies

- Domain registrar: Amazon Route 53, GoDaddy etc.
- DNS Records: A, AAAA, CNAME, NS
- Zone file: contains DNS records
- Name server: resolve DNS queries (Authoritative or Non-Authoritative)
- Top Level Domain (TLD): .com, .us, .in, .gov, .org
- Second Level Domain (SLD): amazon.com, google.com
- The root is what comes after the TLD

## Route 53

- A highly available, scalable, fully managed and Authoritative DNS
  - Authoritative = the customer (you) can update the DNS records
- Route 53 is also a Domain Registrar
- Ability to check the health of your resources
- The only AWS service which provides 100% availability SLA
- Why Rout 53? 53 is a reference to the traditional DNS port

## Records

- How you want to route traffic for a domain
- Each record contains:
  - Domain/subdomain Name (example.com)
  - Record Type (A, AAAA etc)
  - Value (12.34.56.78)
  - Routing policy (how Route 53 responds to queries)
  - TTL (amount of time the record cached at DNS resolvers)
- Route 53 supports the following DNS record types: A/AAAA/CNAME/NS

### Record Types

- A: maps a hostname to IPv4
- AAAA: maps a hostname to IPv6
- CNAME: maps a hostname to another hostname
  - The target is a domain name which must be have an A or AAAA record
  - Can’t create a CNAME record for the top node of a DNS namespace (Zone Apex)
  - Example: you can’t create one for example.com but you can create one for www.example.com
- NS: Name Servers for the Hosted Zone
  - Control how traffic is routed for a domain

## Hosted Zones

- A container for records that define how to route traffic to a domain and its subdomains
- Public Hosted Zones: contains records that specify how to route traffic on the Internet (public domain names) such as application1.mypublicdomain.com
- Private Hosted Zones: contains records that specify how your route traffic within one or more VPCs (private domain names) such as application1.company.internal
- You pay $0.50 per month per hosted zone

## TTL (Time To Live)

- High TTL, eg. 24hr
  - Less traffic on Route 53
  - Possibly outdated records
- Low TTL, eg. 60 seconds
  - More traffic on Route 53 (££)
  - Records are outdated for less time
  - Easy to change records
- Except for Alias records, TTL is mandatory for each DNS record

## CNAME vs Alias

- AWS Resources (Load Balancer, CloudFront etc) exposes an AWS host name
  - 1b1-1234.us-east-2.elb.amazonaws.com and you want myapp.mydomain.com
- CNAME points a hostname to any other host name (app.mydoamin.com => somethingelse.example.com)
  - Only for NON ROOT domain (something.domain.com)
- Alias points a hostname to an AWS Resource (app.mydomain.com => something.amazonaws.com)
  - Works for ROOT domain and NON ROOT domain
  - Free of charge
- Alias Records
  - Maps a hostname to an AWS resource
  - An extension to DNS functionality
  - Automatically recognises changes in the resource’s IP addresses
  - Unlike CNAME, it can be used for the top node of a DNS namespace (Zone Apex), eg. example.com
  - Alias Record is always of type A/AAAA for AWS resource (IPv4/IPv6)
- Alias Records Targets
  - Elastic Load Balancers
  - CloudFront Distributions
  - API Gateway
  - Elastic Beanstalk environments
  - S3 Websites
  - VPC Interface Endpoints
  - Global Accelerator accelerator
  - Route 53 record in the same hosted zone
  - You cannot set an ALIAS record for an EC2 DNS name

## Health Checks

- HTTP Health Checks are only for public resources
- Health Check => Automated DNS Failover
  - Health checks that monitor an endpoint (application, server, other AWS resource)
  - Health checks that monitor other health checks (calculated health checks)
  - Health checks that monitor CloudWatch Alarms (full control) - eg. Throttles of DynamicDB, alarms on RDS, custom metrics (helpful for private resources)
- About 15 global health checkers will check the endpoint health
  - Healthy/unhealthy threshold - 3 (default)
  - Interval - 30 sec (can be set to 10 sec - higher cost)
  - Supported protocol: HTTP, HTTPS and TCP
  - If > 18% of health checkers report the endpoint is unhealthy, Route 53 considers it healthy otherwise unhealthy
- Health checks pass only when the endpoint responds with 200 or 300 status codes
- Health checks can be setup to pass/fail based on the text in the first 5120 bytes of the response
- Configure your router/firewall to allow incoming requests from Route 53 Health Checkers
- Combine the result so multiple Health Checks in a single Health Check
- You can use OR, AND or NOT
- Can monitor up to 256 Child Health Checks
- Specify how to make the parent pass
- Usage: perform maintenance to your website without causing all health checks to fail
- Private hosted zones
  - Route 53 health checkers are outside the VPC
  - They can’t access private endpoints (private VPC or on-premises resource)
  - You can create a CloudWatch Metric and associate a CloudWatch Alarm, then create a Health Check that checks the alarm itsel

## Routing Policy

- Define how Route 53 responds to DNS queries
- Don’t get confused by the word “Routing”
  - It’ snot the same as Load balancer routing which routes the traffic
  - DNS does not route any traffic, it only responds to the DNS queries
- Rout 53 Supports the following Routing Policies:
  - Simple, weighted, failover, latency based, geolocation, multi-value answer and geoproximity (using Route 53 Traffic Flow feature)

### Simple

- Typically, route traffic to a single resource
- Can specify multiple values in the same record
- If multiple values are returned, a random one is chosen by the client
- When Alias enabled, specify only one AWS resource
- Can’t be associated with Health Checks

### Weighted

- Control the % of the requests that go to each specific resource
- Assign each record a relative weight:
  - Traffic (%) = weight for a specific record/sum of all the weights for all records
- DNS record must have the same name and type
- Can be associated with Health Checks
- Use cases
  - Load balancing between regions
  - Testing new application versions
- Assign a weight of 0 to a record to stop sending traffic to a resource
- If all records have weight of 0, then all records will be returned equally

### Latency

- Redirect to the resource that has the least latency close to us
- Super helpful when latency for users is a priority
- Latency is based on traffic between users and AWS Regions
- German users may be directed the US (if that has the lowest latency)
- Can be associated with Health Checks (has a failover capability)

### Failover (Active-Passive)

- Automatically goes to the second instance which is known for disaster recovery

## Geolocation

- Different from latency-based
- This routing is based on user location
- Specify location by Continent, Country or by US State (if there’s overlapping, most precise location selected)
- Should create a “Default” record (in case there’s no match on location)
- Use cases: website, localisation, restrict content distribution, load balancing etc.
- Can be associated with Health Checks

## Geoproximity

- Route traffic to your resource based on geographic location of users and resources
- Ability to shift more traffic to resource based on the defined bias
- To change the size of the geographic region, specify bias values
  - To expand (1 to 99): more traffic to the resource
  - To shrink (-1 to -99): less traffic to the resource
- Resources can be:
  - AWS resources (specify AWS region)
  - Non-AWS resources (specify Latitude and Longitude)
- You must use Route 53 Traffic Flow (advanced) to use this feature

### Traffic Flow & Geoproximity

- Simplify the process of creating and maintaining records in large and complex configurations
- Visual editor to manage complex routing decision trees
- Configuration can be saved as Traffic Flow Policy
  - Can be applied to different Route 53 Hosted Zones (different domain names)
  - Supports versioning

## IP-based

- Routing is based on clients’ IP addresses
- You can provide a list of CIDRs for your client and the corresponding endpoints/locations (user-IP-to-endpoint mappings)
- Use cases: optimise performance, reduce network costs etc.
- Example: route end users from a particular ISP to a specific endpoint

## Multi Value

- Use when routing traffic to multiple resources
- Route 53 returns multiple values/resources
- Can be associated with Health Checks (return only values for healthy resources)
- Up to 8 healthy records are returned for each Multi-Value query
- Multi-value is not a substitute for having an ELB

## 3rd Party Domains & Route 53

- You buy or register your domain name with a Domain registrar typically by paying annual charges (eg. GoDaddy, Amazon Registrar Inc. etc.)
- The Domain Registrar usually provides you with a DNS service to manage your DNS records but you can use another DNS service to manage your DNs records
- Example: purchase the domain from GoDaddy and use Route 53 to manage your DNS records
- If you buy your domain on a 3rd party registrar, you can still use Route 53 as the DNS Service provider
  - Create a Hosted Zone in Route 53
  - Update NS Records on 3rd party website to use Route 53 Name Servers
- Domain Registrar != DNS service
- But every Domain Registrar usually comes with some DNS features

# AWS Serverless: SAM - Serverless Application Model

- SAM = Serverless Application Model
- Framework for developing and deploying serverless applications
- All the configuration is YAML code
- Generate complex CloudFormation from simple SAM YAML file
- Supports anything from CodeDeploy to deploy Lambda functions
- SAM can help you to run Lambda, API Gateway, DynamoDB locally

## Recipe

- Transform Header indicates it’s SAM template
  Transform: `AWS::Serverless-2016-10-31`
- Write Code

```
AWS::Serverless::Function
AWS::Serverless::Api
AWS::Serverless::SimpleTable
```

- Package & Deploy
  sam deploy (optionally preceded by sam package”)
- Quickly sync local cages to AWS Lambda (SAM Accelerate)
  `sam sync --watch`

## SAM Accelerate (sam Sync)

- SAM Accelerate is a set of features to reduce latency while deploying resources to AWS
- sam sync
  - Synchronises your project declared in SAM templates to AWS
  - Synchronises code changes to AWS without updating infrastructure (uses service APIs and bypass CloudFormation)
- Examples
  - sam sync (no options)
    - Synchronise code and infrastructure
  - sam sync —code
    - Synchronise code changes without updating infrastructure (bypass CloudFormation, update in seconds)
  - sam sync —code —resource AWSServerless::Function
    - Synchronise all Lambda functions and their dependencies
  - sam sync —code —resource-id HelloWorldLambdaFunction
    - San chronicle only a specific resource by its ID
  - sam sync —watch
    - Monitor for file changes and automatically synchronises when changes are detected
    - If changes include configuration, it uses sam sync
    - If changes are code only, it uses sam sync —code

### SAM Policy Templates

- List of templates to apply permissions to your Lambda functions
- Important examples
  - S3ReadPolicy: gives read only permissions to objects in S3
  - SQSPollerPolicy: allows to poll an SQS queue
  - DynamoDBCrudPolicy: CRUD = create read update delete

### SAM with CodeDeploy

- SAM framework natively uses CodeDeploy to update Lambda function s
- Traffic Shifting feature
- Pre and post traffic hooks features to validate deployment (before the traffic shift starts and after it ends)
- East & automated rollback using CloudWatch Alarms
- AutoPublishAlias
  - Detects when new code is being deployed
  - Creates and publishes an updated version of that function with the latest code
  - Points the alias to the updated version of the Lambda function
- DeploymentPreference
  - Canary, linear, AtAtOnce
- Alarms
  - Alarms that can trigger a rollback
- Hooks
  - Pre and post traffic shifting Lambda functions to test your deployment

### Local Capabilities

- Locally start AWS Lambda
  - Sam local start-lambda
  - Starts a local endpoint that emulates AWS Lambda
  - Can run automated tests against this local endpoint
- Locally invoke Lambda function
  - Sam local invoke
  - Invoke Lambda function with payload once and quite after innovation completes
  - Helpful for generating test cases
  - If the function makes API calls to AWS, make sure you are using the correct —profile option
- Locally start an API Gateway endpoint
  - Sam local start-api
  - Starts a local HTTP server that hosts all your functions
  - Changes to functions are automatically reloaded
- Generate AWS events for Lambda functions
  - Sam local generate-event
  - Generate sample payloads for event sources
  - S3, API Gateway, SNS, Kinesis, DynamoDB…

## Multiple Environments

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

# Other Serverless: Step Functions and AppSync

## Step Functions

- Model for workflows as state machines (one per workflow)
  - Order fulfillment, data processing
  - Web applications, any workflow
- Written JSON
- Visualisation of the workflow and the execution of the flow flow as well as history
- Start workflow with SDK call, API Gateway, Event Bridge (CloudWatch Event)

## Task States

- Do some work in your state machine
- Invoke one AWS service
  - Can invoke a Lambda function
  - Run an AWS Batch job
  - Run an ECS task and wait for it to complete
  - Insert an item from DynamoDB
  - Publish message to SNS, SQS
  - Launch another Step Function workflow
- Run on one activity
  - EC2, Amazon ECS, on-premises
  - Activities poll the Step function for work
  - Activities send results back to Step Functions

## States

- Choice state: test for a condition to send to a brach (or default branch)
- Fail or succeed state: stop execution with failure or success
- Pass state: simply pass its input to its output or inject some fixed data, without performing work
- Wait state: provide a delay for a certain amount of time or until a specified time/date
- Map state: dynamically iterate steps’
- Parallel state: begin parallel branches of execution

## Error Handling

- Any state can encounter runtime errors for various reasons
  - State machine definition issues (eg. No matching rule in a choice state)
  - Task failures (eg. An exception in a Lambda function)
  - Transient issues (eg. Network partition events)
- Use Retry (to retry failed state) and Catch (transition to failure path) in the State Machine to handle the errors instead of inside the application code
- Predefined error codes
  - States.ALL: matches any error name
  - States.Timeout: task ran longer than TimeoutSeconds or no heartbeat received
  - States.TaskFailed: execution failure
  - States.Permissions: insufficient privileges to execute code
- The state may report is own errors
- Retry (task or parallel state)
  - Evaluated from top to bottom
  - ErrorEquals: match a specific kind of error
  - IntervalSeconds: initial delay before retrying
  - BackOffRate: multiple the delay after each retry
  - MaxAttempts: default to 3, set to 0 for never retried
  - When max attempts are reached the Catch kicks in
- Catch (task or parallel state)
  - Evaluated from top to bottom
  - ErrorEquals: match a specific kind of error
  - Next: state to to send to
  - ResultPath: a path that determines what input is sent to the state specified in the Next field
- ResultPath: include the error in the input

## Wait for Task Token

- Allows you to pause Step Functions during a Task until a Task Token is returned
- Task might wait for other AWS services, human approval, 3rd party integration, call legacy systems
- Append .waitForTaskToken to the resource field to tell Step Functions to wait for the Task Token to be returned
  "Resource": "arn:aws:states:::sqs:sendMessage.waitForTaskToken"
- Task will pause until it receives that Task Token back with a SendTaskSuccess or SendTaskFailure API call

## Activity Tasks

- Enables you to have the Task work performed by an Activity Worker
- Activity Worker apps can be running on EC2, Lambda, mobile device etc.
- Activity Worker poll for a Task using GetActivityTask API
- After Activity Worker completes its work, it sends a response of its success/failure using SendTaskSuccess or SendTaskFailure
- To keep the Task active
  - Configure how long a task can wait by setting TimeoutSeconds
  - Periodically send a heartbeat from your Activity Worker using SendTaskHeartBeat within the time you set in HeartBeatSeconds
- By configuring a long TimeoutSeconds and actively sending a heartbeat, Activity Task can wait up to 1 year

## Standard vs Express

- Standard
  - Max duration: up to one year
  - Exec model: exactly-once
  - Exec rate: over 2000/second
  - Exec history: up to 90 days or using CloudWatch
  - Pricing: # of state transitions
  - Use cases: non-idempotent actions (eg. Processing payments)
  - Asynchronous at least one: doesn’t wait for workflow complete (get results from CW logs)
  - You don’t need an immediate response (eg. Messaging services)
  - Must manage idempotence
- Express
  - Max duration: up to 5 mins
  - Exec rate: over 100,000/second
  - Exec history: CloudWatch Logs
  - Pricing: # of elections, duration and memory consumption
  - Use cases: IoT data ingestion, streaming data, mobile app backends
  - Synchronous at most once: wait for workflow to complete
  - You need an immediate response (eg. Orchestrate micro services)
  - Can be invoked from API Gateway or Lambda function

## AppSync

- AppSync is a managed service that uses GraphQL
- GraphQL makes it easy for applications to get exactly the data they need
- This includes combining data from one or more sources
  - NoSQL data stores relational, relational databases, HTTP APIs
  - Integrates with DynamoDB, Aurora, OpenSearch and others
  - Custom sources with AWS Lambda
- Retrieve data in real-time with WebSocket or MQTT on WebSocket
- For mobile apps: local data access & data synchronisation
- It all starts with uploading one GraphQL schema

### Security

- There are four ways you can authorise applications to interact with your AWS AppSync GraphQL API
  - API_KEY
  - AWS_IAM: IAM users/roles/cross-account access
  - OPENID_CONNECT: OpenID Connect provider/JSON Web Token
  - AMAZON_COGNITO_USER_POOLS
- For custom domains & HTTPS, use CloudFront in front of AppSync

## AWS Amplify

- Set of tools to get started with creating mobile and web applications
- Elastic Beanstalk for mobile and web applications
- Must-have features such as data storage, authentication, storage and machine learning, all powered by AWS services
- Front-end libraries with ready-to-use components for react, vue, javascript, iOS, android, flutter etc
- Incorporates AWS best practices for reliability, security and scalability
- Build and deploy with Amplify CLI or Amplify Studio

### Important Features

- Authentication
  - Leverage Amazon Cognito
  - User registration, authentication, account recover and other operations
  - Support Mia, social sign in etc.
  - Pre-build UI components
  - Fine-grained authorisation
- Datastore
  - Leverage Amazon AppSync and Amazon DynamoDB
  - Work with local data and have automatic synchronisation to the cloud without complex code
  - Powered by GraphQL
  - Offline and real-time capabilities
  - Visual data modelling with Amplify Studio

### Hosting

- Build and host modern web apps
- CICI (build, test, deploy)
- Pull request previews
- Custom Domains
- Monitoring
- Redirect and custom headers
- Password protection

### End-to-End (E2E) Testing

- Run end-to-end tests in the test phase in Amplify
- Catch regressions before pushing code to production
- Use the test step to run any test commands at build time (amplify.yml)
- Integrated with Cypress testing framework
  - Allows you to generate UI reports for your tests

# VPC Fundamentals

- At AWS Certified Developer level you should know:
  - VPC, subnets, Internet gateways & NAT gateways
  - Security Groups, Network ACL (NACL), VPC Flow Logs
  - BPC Peering, VPC End points
  - Site to site VPN & Direct Connect

## VPC, Subnets, IGW and NAT

- VPC: private network to deploy your resources (region resource)
- Subnets allow you to partition your network inside of your VPC (availability zone resource)
- A public subnet is a subnet that is accessible from the Internet
- A private subnet is a subnet that is not accessible from the Internet
- To define access to the Internet and between subnets, we use Route Tables
- Internet Gateways help our VPC instances connect to the Internet
- Public Subnets have a route to the Internet Gateway
- NAT Gateways (AWS managed) & NAT Instances (self-managed) allow your instances in your Private Subnets to access the Internet while remaining private

## NACL, SG, VPC Flow Logs

- NACL (Network ACL)
  - A firewall which controls traffic from and to subnet
  - Can have ALLOW or DENY rules
  - Are attached at the Subnet level
  - Rules only include IP addresses
- Security Groups
  - A firewall that controls traffic to and from an ENI/an EC2 instance
  - Can have only ALLOW rules
  - Rules include IP addresses and other security groups

## Network ACLs vs Security Groups

- Security Groups
  - Operates at the instance level
  - Supports allow rules only
  - Is stateful: return traffic is automatically allowed regardless of any rules
  - We evaluate all rules before deciding whether to allow traffic
  - Applies to an instance only if someone specifies the security group when launching the instance or associates the security group with the instance later on
- Network ACL
  - Operates at the subnet level
  - Supports allow rules an deny rules
  - Is stateless: return traffic must be explicitly allowed by rules
  - We process rules in number order when deciding whether to allow traffic
  - Automatically applies to all instances in subnet it’s associated with (thus, you don’t have to rely on users to specify the security group)

## VPC Flow Logs

- Capture information about IP traffic going into your interfaces
  - VPC Flow Logs
  - Subnet Flow Logs
  - Elastic Network Interface Flow Logs
- Helps to monitor & troubleshoot connectivity issues, some examples include:
  - Subnets to Internet
  - Subnets to subnets
  - Internet to subnets
- Captures network information from AWS managed interfaces too: Elastic Load Balancers, ElastiCache, RDS, Aurora etc.
- VPC Flow logs data can go to S3, CloudWatch Logs and Kinesis Data Firehose

## VPC Peering, Endpoints, VPN, DX

### VPC Peering

- Connect tow VPC, privately using AWS’ network
- Make them behave as if they were in the same network
- Must not have overlapping CIDR (IP address range)
- VPC Peering connect is not transitive (must be established for each VPC that need to communicate with one another)

### VPC Endpoints

- Endpoints allow you to connect to AWS Service using a private network instead of the public www network
- This gives you enhanced security and lower latency to access AWS service
- VPC Endpoint Gateway: S3 and DynamoDB
- VPC Endpoint Interface: the rest
- Only used within your VPC

### Site to Site VPN & Direct Connect

- Site to Site VPN
  - Connect an on-premises VPN to AWS
  - The connection is automatically encrypted
  - Goes over the public Internet
- Direct Connect (DX)
  - Establish a physical connection between on-premises and AWS
  - The connection is private, secure and fast
  - Goes over a private network
  - Takes at leas a month to establish

## VPC Cheat Sheet

- VPC: virtual private cloud
- Subnets: tired to an AZ, network partition of the VPC
- Internet Gateway: at the VPC level, provide Internet Access
- NAT Gateway/Instances: give Internet access to private subnets
- NACL: Stateless, subnet rules for inbound and outbound
- Security Groups: Stateful, operate at the EC2 instance level or ENI
- VPC Peering: Connect two VPC with non overlapping IP ranges, non transitive
- VPC Endpoints: Provide private access to AWS services with VPC
- VPC Flow Logs: network traffic logs
- Site to Site VPN: VPN over public Internet between on-premises DC and AWS
- Direct Connect: direct private connection to a AWS
