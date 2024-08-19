# AWS Serverless: API Gateway

- AWS Lambda + API Gateway: no infrastructure to manage
- Support for the WebSocket Protocol
- Handles API versioning (v1, v2 etc.)
- Handle different environments (dev, test, prod)
- Handle security (authentication and authorisation)
- Create API keys, handle request throttling
- Swagger/Open API import to quickly define APIs
- Transform and validate requests and responses
- Generate SDK and API specifications
- Cache API responses

## Integrations (High Level)

- Lambda Function
  - Lambda functions are invoked
  - Easy way to expose REST API backed by AWS Lambda
- HTTP
  - Expose HTTP endpoints in the backend
  - Example: internal HTTP API on premise, Application Load Balancer etc.
  - Why? Can add rate limiting, caching, user authentication, API keys etc.
- AWS Service
  - Expose any AWS API through the API Gateway
  - Example: start an AWS Step Function workflow, post a message to SQS
  - Why? Add authentication, deploy publicly, rate control etc.

## Endpoint Types

- Edge-optimised (default): for global clients
  - Requests are routed through the CloudFront Edge locations (improves latency)
  - The API Gateway still lives in only one region
- Regional
  - For clients within the same region
  - Could manually combine with CloudFront (more control over the caching strategies and the distribution)
- Private
  - Can only be accessed from your VPC using an interface VPC endpoint (ENI)

## Security

- User authentication through
  - IAM Roles: useful for internal applications
  - Cognito: identity for external uses - eg. Mobile users
  - Custom authoriser (your own logic)
- Custom Domain Name HTTPS security through integration with AWS Certificate Manage (ACM)
  - If using Edge-Optimised endpoint, then the certificate must be in us-east-1
  - If using Regional endpoint, the certificate must the in the API Gateway region
  - Must setup CNAME or A-alias record in Route 53

## Stages and Deployments

### Deployment Stages

- Making changes in the API Gateway does not mean they’re effective
- You need to make a “deployment” for them to be in effect
- It’s a common source of confusion
- Changes are deployed to “Stages” (as many as you want)
- Use the naming you like for stages (dev, test, prod)
- Each stage has it’s own configuration parameters
- Stages can be rolled back as a history of deployment is kept

### Stage Variables

- Stage variables are like environment variables for API Gateway
- Use them to change often changing configuration values
- They can be used in:
  - Lambda function ARN
  - HTTP Endpoint
  - Parameter mapping templates
- Use cases
  - Configure HTTP endpoints your stages talk to (dev, test, prod, etc.)
  - Pass configuration parameters to AWS Lambda through mapping templates
- Stage variables are passed to the “context” object in AWS Lambda
- Format: ${stageVariables.variableName}

### API Gateway Stage Variables and Lambda Aliases

- We create a stage variable to indicate the corresponding Lambda alias
- Our API gateway will automatically invoke the right Lambda function

### Canary Deployments

- Possibility to enable canary deployments for any stage (usually prod)
- Choose the % of traffic the canary channel receives
- Metrics and logs are separate (for better monitoring)
- Possibility to override stage variables for canary
- This is blue/green deployment with AWS Lambda & API Gateway

## Integration Types and Mappings

- Integration Type MOCK
  - API Gateway returns a response without sending the request to the backend
- Integration Type HTTP/AWS (Lambda & AWS Services)
  - You must configure both the integration request and integration response
  - Set up data mapping using mapping templates for the request and response
- Integration Type AWS_PROXY (Lambda Proxy)
  - Incoming request from the client is the input to Lambda
  - The function is responsible for the logic of request/response
  - No mapping template, headers, query, string parameters are passed as arguments
- Integration type HTTP_PROXY
  - No mapping template
  - The HTTP request is passed to the backend
  - The HTTP response from the backend is forward by API Gateway
  - Possibility to add HTTP Headers if need be (eg. API key)

### Mapping Templates (AWS & HTTP Integration)

- Mapping templates can be used to modify request/responses
- Rename/modify query string parameters
- Modify body content
- Add headers
- Uses Velocity Template Language (VTL)
- Filter output results (remove unnecessary data)
- Content-Type can be set to application/json or application/xml

### Mapping Example: JSON to XML with SOAP

- SOAP API are XML based, whereas REST API are JSON based
- In this case, API Gateway should:
  - Extract data from the request (either path, payload or header)
  - Build SOAP message based on request data (mapping template)
  - Call SOAP service and receive XML response
  - Transform XML response to desired format (like JSON) and respond to the user

### Open API

- Common way of defining REST APIs, using API definition as code
- Import existing OpenAPI 3.0 spec to API Gateway
  - Method
  - Method Request
  - Integration Request
  - Method Response
  - - AWS extensions for API gateway and setup every single option
- Can export current API on OpenAPI spec
- OpenAPI specs can be written in YAML or JSON
- Using OpenAPI we can generate SDK for our applications

### REST API: Request Validation

- You can configure API Gateway to perform basic validation of an API request before proceeding with the integration request
- When the validation fails, API Gateway immediately fails the request
  - Returns a 400-error response to the caller and this reduces unnecessary calls to the backend
- Checks
  - The required request parameters in the URI, query string ad headers of an incoming request are included and non-blank
  - The applicable request payload adheres to the configured JSON Schema request model of the method
- OpenAPI
  - Set up request validation by importing OpenAPI definitions file

## Caching

### Caching API Responses

- Caching reduces the number of calls made to the backend
- Default TTL (time to live) is 300 seconds (min: 0s, max:3600s)
- Caches are defined per stage
- Possible to override cache settings per method
- Cache encryption option
- Cache capacity between 0.5GB to 237GB
- Cache is expensive, makes sense in production, may not make sense in dev/test

### API Gateway Cache Invalidation

- Able to flush the entire cache (invalidate it) immediately
- Clients can invalidate the cache with header: Cache-Control:max-age=0 (with proper IAM authorisation)
- If you don’t impose an InvalidateCache policy (or choose the require authorisation checkbox in the console), any client can invalidate the API cache

## Usage Plans & API Keys

- If you want to make an API available as an offering ($) to your customers
- Usage plan
  - Who can access one or more deployed API stages and methods
  - How much and how fast they can access them
  - Uses API keys to identify API clients and meter access
  - Configure throttling limits and quote limits that are enforced on individual client
- API keys
  - Alphanumeric string values to distribute to your customers, eg. WbjAjigERer2954GDFgsdfeE
  - Can use with the usage plans to control access
  - Throttling limits are applied to the API keys
  - Quotas limits is the overall number of maximum requests

### Correct Order for API Keys (to configure a usage plan)

1. Create one or more APIs, configure the methods to require an API key, and deploy the APIs to stages
2. Generate or import API keys to distribute to application developers (your customers) who will be using your API
3. Create the usage plan with the desired throttle and quotas limits
4. Associate API stages and API keys with the usage plan

- Callers of the API must supply an assigned API key in the z-api-key header in the request to the API

## Monitoring, Logging and Tracking

- CloudWatch Logs
  - Log contains information about request/response body
  - Enable CloudWatch logging at the Stage level (with Log Level - ERROR, DEBUG, INFO)
  - Can override settings on a per API basis

### CloudWatch Metrics

- Metrics are by stage, possibility to enable detailed metrics
- CacheHitCount and CacheMissCount: efficiency of the cache
- Count: the total number of API requests in a given period
- IntegrationLatency: the time between when API Gateway relays a request to the backend and when it receives a response from the backend
- Latency: the time between when API Gateway receives a request from a client and when it returns a response to the client. The latency includes the integration latency and other API Gateway overhead
- 4xx error (client side) and 5xx error (server side)

### API Gateway Throttling

- Account limit
  - API Gateway throttles requests at 10,000 res across all APIs
  - Soft limit that can be increased upon request
- In case of throttling => 429 Too Many Requests (retriable error)
- Can set stage limit and method limits to improve performance
- Or you can define Usage Plans to throttle per customer
- Just like Lambda Concurrency, one aPI that tis overloaded, if not limited can cause the other APIs to be throttled

### Errors

- 4xx means client errors
  - 400: bad request
  - 403: access denies, WAF filtered
  - 429: quota exceeded, throttle
- 4xx means server errors
  - 502: bad gateway exception, usually for an incompatible output returned from a Lambda proxy integration backend and occasionally for out-of-order invocations due to heavy loads
  - 503: service unavailable exception
  - 504: integration failure - eg. Endpoint request timed out exception
    - API Gateway requests time out after 29 seconds max

### CORs

- CORs must be enabled when you receive API calls from another domain
- The OPTIONS pre-flight request must contain the following headers:
  - Access-Control-Allow-Methods
  - Access-Control-Allow-Headers
  - Access-Control-Allow-Origin
- CORs can be enabled through the console

## Authentication and Authorisation

### IAM Permissions

- Create an IAM policy authorisation and attach to User/Role
- Authentication = IAM
- Authorisation = IAM Policy
- Good to provide access within AWS (EC2, Lambda, IAM users etc.)
- Leverage Sig v4 capability where IAM credentials are in headers

### Resource Policies

- Resource policies (similar to Lambda Resource Policy)
- Allow for Cross Account Access (combined with IAM Security)
- Allow for a specific source IP address
- Allow for a VPC Endpoint

## API Gateway - Security

- Cognito User Pools
  - Congnito fully manages user lifecycle, token expires automatically
  - API gateway verifies identity automatically from AWS Cognito
  - No custom implementation required
  - Authentication = cognito user pools
  - Authorisation = API gateway methods
- Lambda Authoriser (formerly Custom Authorisers)
  - Token-based authoriser (bearer token) - eg. JWT (JSON Web Token) or OAuth
  - A request parameter-based Lambda authoriser (headers, query string, stage var)
  - Lambda must return an IAM policy for the user, result policy is cached
  - Authentication = external
  - Authorisation = lambda function
- IAM
  - Great for users.roles already within your AWS account (+ resource policy for cross account)
  - Handle authentication + authorisation
  - Leverages Signature v4
- Custom Authoriser
  - Great for 3rd party tokens
  - Very flexible in terms of what IAM policy is returned
  - Handle authentication verification + authorisation in the Lambda function
  - Pay per Lambda invocation, results are cached
- Cognito User Pool
  - You manage your own user pool (can be backed by Facebook, Google Log in etc.)
  - No n ed to write any custom code
  - Must implement authorisation in the backend

## REST API vs HTTP API

- HTTP APIs
  - Low latency, cost-effective AWS Lambda proxy, HTTP proxy APIs and private integration (no data mapping)
  - Supports OIDC and OAuth 2.0 authorisation and built in support for CORs
  - No usage plans and Api keys
- REST APIs
  - All features (except Native OpenID Connect.OAuth 2.0)

## WebSocket API

- What’s WebSocket?
  - Two-way interactive communication between a user’s browser and a server
  - Server can push information to the client, this enables stateful application use cases
- WebSocket APIs are often used in real-time applications such as chat applications, collaboration platforms, multiplayer games and financial trading platforms
- Works with AWS Services (Lambda, DynamoDB) or HTTP endpoints

## Routing

- Incoming JSON messages are routed to different backed
- If no routes => sent to $default
- You request a route selection expression to select the field on JSON to route from
- Sample expression: $request.body.action
- The result is evaluated against the route keys viable in your API Gateway
- The route is then connection ed the backend you’ve setup through API Gateway

## Architecture

- Create a single interface for all the micro-services in your company
- Use API endpoints with various resources
- Apply a simple domain name and SSL certificates
- Can apply forwarding and transformation rules at the API Gateway level
