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
