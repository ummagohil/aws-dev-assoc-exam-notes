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
