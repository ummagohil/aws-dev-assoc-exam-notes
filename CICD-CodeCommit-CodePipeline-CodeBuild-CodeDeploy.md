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
