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
