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
