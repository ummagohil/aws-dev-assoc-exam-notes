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
