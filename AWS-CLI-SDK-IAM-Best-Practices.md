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
