# AWS Lambda

## Serverless

- Serverless is a new paradigm in which the developers don’t have to manage servers anymore
- They just deploy code (in the form of functions)
- Initially… Serverless == FaaS (Function as a Service)
- Serverless was pioneered by AWS Lambda but now also includes anything that’s managed (databases, messaging, storage etc.)
- Serverless does not mean there are no servers - it just means you don’t manage/provision/see them

### Serverless in AWS

- AWS Lambda
- DynamoDB
- AWS Cognito
- AWS API Gateway
- Amazon S3
- AWS SNS & SQS
- AWS Kinesis Data Firehose
- Aurora Serverless
- Step Functions
- Fargate

## AWS Lambda

### (EC2)

- Virtual servers in the cloud
- Limited by RAM and CPU
- Continuously running
- Scaling means intervention add/remove servers
  (Amazon Lambda)
- Virtual functions - no servers to manage
- Limited by time - short executions
- Run on-demand
- Scaling is automated

### Benefits of AWS Lambda

- Easy pricing
  - Pay per request and compute time
  - Free tier of 1,000,000 AWS Lambda requests and 400,000 GBs of compute time
- Integrated with the whole AWS suite of services
- Integrated with many programming languages
- Easy monitoring through AWS CloudWatch
- Easy to get more resources per functions (up to 10GB of RAM)
- Increasing RAM will also improve CPU and network

### AWS Lambda Language Support

- Node.js (JavaScript)
- Python
- Java
- C# (.NET Core) / Powershell
- Ruby
- Customer runtime API (community supported, example Rust or Golang)
- Lambda Container Image
  - The container image must implement the Lambda Runtime API
  - ECS/Fargate is preferred for running arbitrary Docker images

### AWS Lambda Integrations (main ones)

- API Gateway
- Kinesis
- DynamoDB
- S3
- CloudFront
- CloudWatch Events EventBridge
- CloudWatch Logs
- SNS
- SQS
- Cognito

### Lambda Pricing Example

- You can find overall pricing information here: https://aws.amazon.com/lambda/pricing
- Pay per calls
  - First 1,000,000 request are free
  - $0.20 per 1 million request thereafter ($0.0000002 per request)
- Pay per duration (in increments of 1ms)
- 400,000 GB-seconds of compute time per month if FREE
- === 400,000 seconds if function is 1 GB RAM
- == 3,200,000 seconds if function is 128 MB RAM
- After that $1.00 for 600,000 GB-seconds

### Lambda Synchronous Invocations

- Synchronous: CLI, SDK, API Gateway, Application Load Balancer
  - Results are returned right away
  - Error handling must happen client Sid e(retries, exponential backoff etc.)

### Services

- User invoked
  - Elastic Load Balancing (application load balancer)
  - Amazon API Gateway
  - Amazon CloudFront (Lambda@Edge)
  - Amazon S3 Batch
- Service Invoked
  - Amazon Cognito
  - AWS Step Functions
- Other services
  - Amazon Lex
  - Amazon Alexa
  - Amazon Kinesis Data Firehose

## Application Load Balancer

- To expose a Lambda function as an HTTP(S) endpoint you can use the ALB (or an API Gateway)
- The Lambda function must be registered in a target group

### ALB Multi-Header Values

- ALB can support multi header values (ALB setting)
- Wen you enable multi-value headers, HTTP headers and query string parameters that are sent with multiple values are shown as arrays within the AWS Lambda event and response objects

## Asynchronous Invocations & DLQ

### Asynchronous Invocations

- S3, SNS, CloudWatch Events
- The events are placed in an Event Queue
- Lambda attempts to retry on errors
  - 3 tries total
  - 1 minute wait after 1st, then 2 minutes wait
- Make sure the processing is idempotent (in case of retires)
- If the function is retried, you will see duplicate logs entries in CloudWatch Logs
- Can define a DLQ (dead-letter queue) - SNS or SQS - for failed processing (need correct IAM permissions)
- Asynchronous innovations allow you to speed up the processing if you don’t need to wait for the result - eg. You need 1,000 files processed

### Services

- Amazon Simple Storage Service (S3)
- Amazon Simple Notification Service (SNS)
- Amazon CloudWatch Events/EventBridge
- AWS CodeCommit (CodeCommit Trigger: new branch, new tag, new push)
- AWS CodePipeline (income a Lambda function during the pipeline, Lambda must callback)
  —other—
- Amazon CloudWatch Logs (log processing)
- Amazon Simple Email Service
- AWS CloudFormation
- AWS Config
- AWS IoT
- AWS IoT Events

## S3 Event Notifications

- S3:ObjectCreated, S3:ObjectRemoved, S3:ObjectRestore, S3:Replication
- Object name filtering possible (\*.jpeg)
- Use case: generate thumbnails of images uploaded to S3
- S3 even notifications typically deliver events in seconds but can sometimes take a minute or longer
- If two writes are made to a single non-versioned object at the same time, it is possible that only a single event notification will be sent
- If you want to ensure that an event notification is sent for every successful write, you can enable versioning on your bucket

### Event Source Mapping

- Kinesis Data Streams
- SQS & SQS FIFO queue
- DynamoDB Streams
- Common denominator: records need to be polled from the source
- Your Lambda function is invoked synchronously

### Steams and Lambda - Kinesis & DynamoDB

- An event source mapping creates an iterator for each shard, processes item in order
- Start with new items, from the beginning or from timestamp
- Processed items aren’t removed from the stream (other consumers can read them)
- Low traffic: use batch window to accumulate records before processing
- You can process multiple batches in parallel
  - Up to 10 batches per shard
  - In-order processing is still guaranteed for each partition key

### Error Handling

- By default, if your function returns an error, the entire batch is reprocessed until the function succeeds, or the items in the batch expire
- To ensure in-order processing, processing for the affected shard is paused until the error is resolved
- You can configure the event source mapping to:
  - Discard old events
  - Restrict the number of retries
  - Split the batch on error (to work around Lambda timeout issues)
- Discarded events go to a Destination

### SQS & SQS FIFO

- Event Source Mapping will poll SQS (Long Polling)
- Specify batch size (1-10 messages)
- Recommended: set the queue visibility timeout 6x the timeout of your Lambda function
- To use a DLQ
  - Set up on the SQS queue, not Lambda (DLQ for Lambda is only for asynchronous invocations)
  - Or use a Lambda destination for failures

## Queues & Lambda

- Lambda also supports in-order processing for FIFO queues, scaling u pro the number of active message groups
- For standard queues, items aren’t necessarily processed in order
- Lambda scales up to process a standard queue as quickly as possible
- When an error occurs, batches are returned to the queue as individual items and might be processed in a different grouping than the original batch
- Occasionally, the event source mapping might receive the same item from the queue twice, even if no function error occurred
- Lambda deletes items from the queue after they’re processed successfully
- You can configure the source queue to send items to a dead-letter queue if they can’t be processed

## Lambda Event Mapper Scaling

- Kinesis Data Streams & DynamoDB Streams
  - One Lambda inception per stream shard
  - If you use parallelisation, up to 10 batches processed per shard simultaneously
- SQS Standard
  - Lambda adds 60 more instances per minute to scale up
  - Up to 1000 batches of messages processed simultaneously
- SQS FIFO
  - Messages with the same GroupID will be processed in order
  - The Lambda function scales to the number of active message groups

## Lambda Event and Context Objects

### Event Object

- JSON-formatted document contains data for the function to process
- Contains information from the invoking service (eg. EventBridge, custom)
- Lambda runtime converts the event to an object (eg. Dict type in Python)
- Example: input arguments, invoking service arguments…

### Context Object

- Provides methods and properties that provide information about the invocation, function, and runtime environment
- Passed to your function by Lambda at runtime
- Example: aws_request_id, function_name, memory_limit_in_mb

## Lambda Destinations

- From Nov 2019 can configure to send result to a destination
- Asynchronous invocations: can define destinations for successful and failed event:
  - Amazon SQS
  - Amazon SNS
  - AWS Lambda
  - Amazon EventBridge bus
- Note: AWS recommends you use destinations instead of DLQ now (but both can be used at the same time)
- Event Source mapping: for discarded event batches
  - Amazon SQS
  - Amazon SNS
- Note: you can send events to a dLQ directly from SQS

## IAM Roles & Resource Policies

- Grants the Lambda function permissions to AWS services/resources
- Sample managed policies in Lambda:
  - AWSLambdaBasicExecutionRole: upload logs to CloudWatch
  - AWSLambdaKinesisExecutionRole: read from Kinesis
  - AWSLambdaDynamoDBExecutionRole: read from Dynamo Streams
  - AWSLambdaSQSQueueExecutionRole: read from SQS
  - AWSLambdaVPCAccessExecutionRole: deploy Lambda function in VPC
  - AWSXRayDaemonWriteAccess: upload trace data to X-Ray
- When you use an event source mapping to invoke your function, Lambda uses the execution role to read event data
- Best practice: create one Lambda Execution Role per function

### Resource Based Policies

- Use resource-based policies to give other accounts and AWS services permissions to use your Lambda resources
- Similar to S3 bucket policies for S3 bucket
- An IAM principal can access Lambda:
  - If the IAM policy attached to the principal authorises it (eg. User access)
  - OR, if the resource-based policies authorises it (eg. Service access)
- When an AWS service like Amazon S3 calls your Lambda function, the resource-based policy gives it access

### Environment Variables

- Environment variable = key/value pair in “string” format
- Adjust the function behaviour without updating code
- The environment variables are available to your code
- Lambda services add its own system environment variables as well
- Helpful to store secrets (encrypted by KMS(
- Secrets can be encrypted by the Lambda service key, or your own CMK

## Monitoring and X-Ray Tracing

- CloudWatch Logs
  - AWS Lambda execution logs are stored in AWS CloudWatch Logs
  - Make sure your AWS Lambda function has an execution role with an IAM policy that authorises writes to CloudWatchLogs
- CloudWatch Metrics
  - AWS Lambda metrics are displayed in AWS CloudWatch Metrics
  - Invocations, durations, concurrent executions
  - Error count, success rates, throttles
  - Async delivery failures
  - Iterator age (Kinesis and DynamoDB Streams)

### X-Ray

- Enable in Lambda configuration (active tracing)
- Runs the X-Ray daemon for you
- Use AWS X-Ray SDK in Code
- Ensure Lambda Function has a correct IAM Execution Role
  - The managed policy is called AWSXRayDaemonWriteAccess
- Environment variables to communicate with X-Ray
  - \_X_AMZN_TRACE_ID: contains the tracing header
  - AWS_XRAY_CONTEXT_MISSING: by default, LOG_ERROR
  - AWS_XRAY_DAEMON_ADDRESS: the x-ray daemon IP_ADDRESS:PORT

## Lambda@Edge and CloudFront Functions

### Customisation at the Edge

- Many modern applications execute some from of logic at the edge
- Edge Function
  - A code that you write and attach to CloudFront distributions
  - Runs close to your users to minimise latency
- CloudFront provides two types: CloudFront Functions & lambda@Edge
  - Use cases
    - Website security and privacy
    - Dynamic web application at the Edge
    - Search Engine Optimisation (SEO)
    - Intelligently Route Across Origins and Data Centres
    - Bot Mitigation at the Edge
    - Real-time Image Transformation
    - A/B Testing
    - User authentication and authorisation
    - Use prioritisation
    - User tracking and analytics
- You don’t have to manage any servers, deployed globally
- Use case: customers the CDN content
- Pay only for what you use

### CloudFront Functions

- Lightweight functions written in JavaScript
- For high-scale, latency-sensitive CDN customisations
- Sub-ms startup times, millions of request/second
- Used to change Viewer requests and responses
  - Viewer request: after CloudFront receives a request from a viewer
  - Viewer response: before CloudFront forwards the response to the viewer
- Native feature of CloudFront (manage code entirely within CloudFront)

### Lambda@Edge

- Lambda functions written in NodeJS or Python
- Scale to 1000s of requests/second
- Used to change CloudFront requests and responses
  - Viewer request: after CloudFront receives a request from a viewer
  - Origin request: before CloudFront forwards the request to the origin
  - Origin response: after CloudFront receives the response from the origin
  - Viewer response: before CloudFront forwards the response to the viewer
- Author your functions in one AWS Region (us-east-1), then CloudFront replicates to its locations

### CloudFront Functions vs. Lambda@Edge

- CloudFront Functions
  - Cache Key normalisation
    - Transform request attributes (headers, cookies, query strings, URL) to create an optimal Cache Key
  - Header manipulation
    - Insert/modify/delete HTTP headers in the request or response
  - URL rewrites or redirects
  - Request authentication and authorisation
    - Create and validate user-generated tokens (eg. JWT) to allow/deny requests
- Lambda@Edge
  - Longer execution time (several ms)
  - Adjustable CPU or memory
  - Your code depends on 3rd party libraries (eg. AWS SDK to access other AWS services)
  - Network access to us external services for processing
  - File system access or access to the body of HTTP requests

## Lambda in VPC

### Lambda by Default

- By default, your Lambda function is launched outside your own VPC (in an AWS-owned VPC)
- Therefore it cannot access resources in your VPC (RDS, ElastiCache, internal ELB)

### Lambda in VPC

- You must define the VPC ID, the subnets and the security groups
- Lambda will create an ENI (Elastic Network Interface) in your subnets
- AWSLambdaVPCAccessExecutionRole
- Internet Access
  - A Lambda function in your VPC does not have Internet access
  - Deploying a Lambda function in a public subnet does not give it Internet access or a public IP
  - Deploying a Lambda function in a private subnet gives it Internet access if you have a NAT Gateway/Instance
  - You can use VPC endpoints to privately access AWS services without a NAT

## Function Performance

### Function Configuration

- RAM
  - From 128 MB to 10GB in 1MB increments
  - The more RAM you add, the more vCPU credits you get
  - At 1,792 MB a function has the equivalent of one full vCPU
  - After 1,792 MB you get more than one CPU, and you need to use multi-threading in your application to benefit from it
- If your application is CPU-bound (computation heavy), increase RAM
- Timeout: default 3 seconds, maximum is 900 seconds (15 mins)

### Lambda Execution Context

- The execution context is a temporary runtime environment that initialises any external dependencies of your lambda code
- Great for database connections (HTTP clients, SDK clients etc)
- The execution context is maintained for some time in anticipation of another Lambda function innovation
- The next function innovation can “re-use” the context to execution time and save time in initialising connections objects
- The execution context includes /tmp directory

### Lambda Functions /tmp space

- If your Lambda function needs to download a big file to work
- If your Lambda function needs disk space to perform operations
- You can use the /tmp directory
- Max size is 10GB
- The directory content remains when the execution context is frozen, providing transient cache that can be used for multiple invocations (helpful to checkpoint your work)
- For permanent persistence of object (non temporary), use S3
- To encrypt content on /tmp, you must generate KMS Data Keys

## Layers

- Custom Runtimes
  - C++: github.com/awslabs/aws-lambda-cpp
  - Rust: github.com/awslabs/aws-lambda-rust-runtime

## File System Mounting

- Lambda functions can access EFS file systems if they are running in a VPC
- Configure Lambda to mount EFS file systems to local directory during initialisation
- Must leverage EFS Access Points
- Limitations: watch out for the EFS connection limits (one function instance = one connection) and connection burst limits

## Concurrency

- Concurrency limit: up to 1000 concurrent executions
- Can set a “reserved concurrency” at the function level (=limit)
- Each invocation over the concurrency limit will trigger a “throttle”
- Throttle behaviour
  - If synchronous invocation => return ThrottleError 429
  - If asynchronous invocation => retry automatically and then go to DLQ
- If you need a higher limit, open a support ticket
- Lambda concurrency issue: if you don’t reserve (=limit) concurrency, the following happens
  - Throttle

### Concurrency and Asynchronous Invocations

- If the function doesn’t have enough concurrency available to process all events, additional requests are throttled
- For throttling errors (429) and system errors (500-series), Lambda returns the event to the queue and attempts to run the function again for up to 6 hours
- The retry interval increases exponentially from 1 second affect the first attempt to a maximum of 5 mins

### Cold Starts and Provisioned Concurrency

- Cold start
  - New instance: code is loaded and code outside the handler run (init)
  - If the init is large (code, dependencies, SDK etc.) this process can take some time
  - First request served by new instances has higher latency than the rest
- Provisioned concurrency
  - Concurrency is allocated before the function is invoked (in advance)
  - So the cold start never happen and all invocations have low latency
  - Application Auto Scaling can manage concurrency (schedule or target utilisation)
- Note: cold starts in VPC have been dramatically reduced in oct and nov 2019

## External Dependencies

- If your Lambda function depends on external libraries (AWS X-Ray DSK, Database Clients etc.), you will need to install the packages alongside your code and zip it together
  - For Node.js, use npm & “node_modules” directory
  - For Python, use pip —target options
  - For Java, include the relevant .jar files
- Upload the zip straight to Lambda if less than 50MB, else to S3 first
- Native libraries work: they need to be complied on Amazon Linus
- AWS SDK comes by fault with every Lambda function

## Lambda and CloudFormation

### Inline

- Inline functions are very simple
- Use the Code.ZipFile property
- You cannot include function dependencies with inline functions

### Through S3

- You must store the Lambda zip in S3
- You must refer the S3 zip location in the CloudFormation code
  - S3Bucket
  - S3Key: full path to zip
  - S3ObjectVersion: if versioned bucket
- If you update the code in S3 but don’t update S3Bucket, S3Key or S3ObjectVersion, CloudFormation won’t update your function

### Lambda Container Images

- Deploy Lambda function as container images of up to 10GB from ECR
- Pack complex dependencies, large dependencies in a container
- Base images are available for Python, Node.js, Java, .NET, Go, Ruby
- Can create your own image as long as it implements the Lambda Runtime API
- Test the containers locally using the Lambda Runtime Interface Emulator
- Unified workflow to build apps

## Best Practices

- Strategies for optimising container images
  - Use AWS-provided Base Images
    - Stable, built on Amazon Linus 2, cache by Lambda service
  - Use Multi-Stage Builds
    - Builds your code in larger preliminary images, copy only the artifacts you need in your final container image, discard the preliminary steps
  - Build from Stable to Frequently Changing
    - Make your most frequently occurring changes as late in your Dockerfile possible
  - Use a Single Repository for Functions with Large Layers
    - ECR compares each layer of a container image when it is pushed to avoid uploading and storing duplicates
  - Use them to upload large Lambda functions (up to 10GB)

## Versions and Aliases

### Lambda Versions

- When you work on a Lambda function, we work on $LATEST
- When we’re ready to publish a Lambda function, we create a version
- Versions are immutable
- Versions have increasing version numbers
- Versions get their own ARN (Amazon Resource Name)
- Version = code + configuration (nothing can be changed; immutable)
- Each version of the Lambda function can be accessed

### Lambda Aliases

- Aliases are “pointers” to Lambda function versions
- We can define a “dev”, “test”, “port” aliases and have them point at different Lambda versions
- Aliases are mutable
- Aliases enable Canary deployment by assigning weights to Lambda functions
- Aliases enable stable configuration of our event triggers/destinations
- Aliases have their own ARNs
- Aliases cannot reference aliases

## CodeDeploy with Lambda

- CodeDeploy can. Help you automate traffic shift for Lambda aliases
- Feature is integrated within he SAM framework
- Linear: grow traffic every N minutes until 100%
  - Linear10PercentEvery3Minutes
  - Linear10PercentEvery10Minutes
- Canary: try X percent them 100%
  - Canary10Percent5Minutes
  - Canary10Percent30Minutes
- AllAtOnce: immediate
- Can create Pre & Post Traffic hooks to check the health of the Lambda function

### AppSpec.yml

- Name (required): the name of the Lambda function to deploy
- Alias (required): the name of the alias to the Lambda function
- CurrentVersion (required): the version of the Lambda function traffic currently points to
- TargetVersion (required): the version of the Lambda function traffic is shifted to

## Lambda Function URL

- Dedicated HTTP(S) endpoint for your Lambda function
- A unique URL endpoint is generated for you (never changes)
  - htts://<url-id>.lambda-url.<region>.on.aws (dual-stack IPv4 & IPv6)
- Invoke via a web browser, curl, oPostman or any HTTP client
- Access your function URL through the public Internet only
  - Does support Private Link (Lambda functions do support)
- Supports Resource-based policies & CORS configurations
- Can be applied to any function alias or to $LATEST (can’t be applied to other function versions)
- Create and configure using AWS Console or AWS API
- Throttle your function by using Reserved Concurrency

## Security

- Resource-based Policy
  - Authorise other accounts/specific CIDR/IAM principals
- Cross-Origin Resource Sharing (CORS)
  - If you call your Lambda function URL from a different domain
- AuthType NONE (allow public and unauthenticated access)
  - Resource-based policy is always in effect (must grant public access)
- AuthType AWS_IAM (IAM is used to authenticate and authorise requests)
  - Both principals identity-based policy and resource-based policy are evaluated
  - Principal must have lambda:InvokeFunctionUrl permissions
  - Same account: identify-based policy OR resource-based policy as ALLOW
  - Cross account: identify-based policy AND resource-based policy as ALLOW

## Lambda - CodeGuru Integration

- Gain insights into runtime performance of your Lambda functions using codeGuru Profiler
- CodeGuru creates a Profiler Group for your Lambda function
- Supported for Java and Python runtimes
- Activate from AWS Lambda Console
- When activated, Lambda adds:
  - CodeGuru Profiler layer to your function
  - Environment variables to your function
  - AmazonCodeGuruProfilerAgentAccessPolicy to your function

## Lambda Limits (per region)

### Execution

- Memory allocation: 128MB - 10GB (1MB increments)
- Maximum execution time: 900 seconds (15 minutes)
- Environment variables (4 KB)
- Disk capacity in the “function container” (in /tmp): 512 MB to 10GB
- Concurrency executions: 1000 (can be increased)

### Deployment

- Lambda function deployment size (compressed .zip): 50 MB
- Size of uncompressed deployment (code + dependencies): 250 MB
- Can use the /tmp directory to load other files at startup
- Size of environment variables: 4KB

## Lambda Best Practices

- Perform heavy-duty work outside of your function handler
  - Connect to database outside of your function handler
  - Initialise the AWS SDK outside of your function handler
  - Pull in dependencies or datasets outside of your function handler
- Use environment variables for:
  - Database Connection Strings, S3 bucket etc. (don’t put these values in your code)
  - Passwords, sensitive values… They can be encrypted using KMS
- Minimise your deployment package size to its runtime necessities
  - Breakdown the function if need be
  - Remember the AWS Lambda limits
  - Use Layers where necessary
- Avoid using recursive code, never have a Lambda function call itself
