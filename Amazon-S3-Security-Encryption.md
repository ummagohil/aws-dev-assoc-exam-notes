# Amazon S3 Security

## S3 Encryption

### Object Encryption

- You can encrypt objects in S3 buckets using one of 4 methods
- Server-Side Encryption (SSE)
  - Server-side encryption with Amazon S3-Managed Keys (SSE-S3) - enabled by default
    - Encrypts S3 objects using keys handled, managed and owned by AWS
  - Service-Side Encryption with LMS Keys stored in AWS KMS (SSE-KMS)
    - Leverage AWS Key Management Service (AWS KMS) to manage encryption keys
  - Server-Side Encryption with Customer-Provided Keys (SSE-C)
    - When you want to manage your own encryption keys
- Client-Side Encryption

## SSE-S3

- Encryption using keys handled, managed and owned by AWS
- Object is encrypted server-side
- Encryption type is AES-256
- Must set header “x-amz-server-side-encryption”:”AES256”
- Enabled by default for new buckets & new objects

## SSE-KMS

- Encryption using keys handled and managed by AWS KMS (Key Management Service)
- KMS advantages: user control + audit key usage using CloudTrail
- Object is encrypted server side
- Must set header “x-amz-server-side-encryption”:”as:kms”
- Limitations:
  - If you see SSE-KMS, you may be impacted by the KMS limits
  - When you upload, it calls the GenerateDataKey KMS API
  - When you download, it calls the Decrypt KMS API
  - Count towards the KMS quote per second (5500, 10000, 30000 req/s based on region)
  - You can request a quote increase using the Service Quotas Console

## SSE-C

- Server-Side Encryption using keys fully managed by the customer outside of AWS
- Amazon S3 does not store the encryption key you provide
- HTTPS must be used
- Encryption key must be provided in HTTP headers, for every HTTP request made

## Client-Side Encryption

- Use client libraries such as Amazon S3 Client-Side Encryption Library
- Clients must encrypt data themselves before sending to Amazon S3
- Clients must decrypt data themselves when retrieving from Amazon S3
- Customer fully manages the keys and encryption cycle

## Encryption in Transit (SSL/TLS)

- Encryption in flight is also called SSL/TLS
- Amazon S3 exposes two endpoints
  - HTTP Endpoint: non encrypted
  - HTTPS Endpoint: encryption in flight
- HTTPS is recommended
- HTTPS is mandatory for SSE-C
- Most clients would use the HTTPS endpoint by default

## About DDS-KMS

- “Double encryption based on KMS”

## S3 Default Encryption

### Default Encryption vs. Bucket Policies

- SSE-S3 encryption is automatically applied to new objects stored in S3 bucket
- Optionally, you can “force encryption: using a bucket policy and refuse any API call to PUT an S3 object without encryption headers (SSE-KMS or SSE-C)
- Note: Bucket Policies are evaluated before “Default Encryption”

### S3 CORS

- Cross-Origin Resource Sharing (CORS)
- Origin = scheme (protocol) + host(domain) + port
  - example: (implied port is 443 for HTTPS, 80 for HTTP)
- Web browser based mechanism to allow requests to other origins while visiting the main origin
- Same origin: &
- Different origins: &
- The requests won’t be fulfilled unless the other origin allows for the requests, using CORS Headers (example: Access-Control-Allow-Origin)
- If a client makes a cross-origin request on our S3 bucket, we need to enable the correct CORS headers
- It’s a popular exam question
- You can allow for a specific origin or for \* (all origins)

### S3 MFA Delete

- MFA (multi-factor authentication): force users to generate a code on a device (usually a mobile phone or hardware) before doing important operations on S3
- MFA will be required to:
  - Permanently delete an object version
  - Suspend versioning on the bucket
- MFA won’t be required to:
  - Enable versioning
  - List deleted versions
- To use MFA Delete, versioning must be enabled on the bucket
- Only the bucket owner (root account) can be enable/disable MFA Delete

### S3 Access Logs

- For audit purposes, you may want to log all access to S3 buckets
- Any requests made to S3, from any account, authorised or denied, will be logged into another S3 bucket
- That data can be analysed using data analysis tools
- The target logging bucket must bin the same AWS region
- The log format is at:
- Warning:
  - Do not set your logging bucket to be the monitored bucket
  - It will create a logging loop and your bucket will grow exponentially

### S3 Pre-signed URLs

- Generate pre-signed URLs using the S3 Console, AWS CLI or SDK
- URL Expiration
  - S3 Console - 1 min up to 720 mins (12 hours)
  - AWS CLI - configure expiration with —expires-in parameter in seconds (default is 3600 secs, max is 604800 secs ~168 hours)
- Users given a pre-signed URL inherit the permissions of the user that generated the URL for GET/PUT
- Examples
  - Allow only logged-in users to download a premium video form your S3 bucket
  - Allow an ever-changing list of users to download files by generating URLs dynamically
  - Allow temporarily a user to upload a file to a precise location in your bucket

## S3 Access Points

### VPC Origin

- We can define the access point to be accessible only from within the VPC
- You must create a VPC Endpoint to access the Access Point (Gateway or Interface Endpoint)
- The VPC Endpoint Policy must allow access to the target bucket and Access Point

## S3 Object Lambda

- Use AWS Lambda Function storage change the object before it is retrieved by the caller application
- Only one S3 bucket is needed, on top of which we create S3 Access Point and S3 Object Lambda Access Points
- Use Cases
  - Redacting personally identifiable information for analytics or non-production environments
  - Converting across data formats, such as converting XML to JSON
  - Resizing and watermarking images on the fly using caller-specific details, such as the user who requested the object
