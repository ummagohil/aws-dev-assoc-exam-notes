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
