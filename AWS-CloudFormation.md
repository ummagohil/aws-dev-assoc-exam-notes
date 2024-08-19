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
