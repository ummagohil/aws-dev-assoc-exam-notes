# Advanced Identity

## AWS STS - Security token Service

- Allows you to grant limited ad temporary access to AWS resources (up to one hour)
- AssumeRole: Assume roles within your account or cross account
- AssumeRoleWithSAML: return credentials for users logged in with SAML
- AssumeRoleWithWebIdentity
  - Return creds for users logged with an IdP (Facebook, Google, OIDC compatible)
  - AWS recommends against using this, and using Cognito Identity Pools instead
- GetSessionToken: for MFA, from a user or AWS account root user
- GetFederationToken: obtain temporary creds for a federated user
- GetCallerIdentity: return details about the IAM user or role used in the API call
- DecodeAuthorisationMessage: decode error message when AWS API is denied

## Using STS to Assume a Role

- Define an IAM Role within your account or cross-account
- Define which principals can access this IAM Role
- Use AWS STS (security token service) to retrieve credential sand impersonate the IAM Role you have access to (AssumeRole API)
- Temporary credentials can be valid between 15 minutes to 1 hour

## STS with MFA

- Use GetSessionToken from STS
- Appropriate IAM policy using IAM Conditions
- aws:MultiFactorAuthPresent:true
- Reminder: Get SessionToken returns access ID, secret key, session token, expiration date

## Advanced IAM

### Authorisation Model Evaluation of Policies

1. If there’s an explicit DENy, end decision and DENY
2. If there’s an ALLOW, end decision with ALLOW
3. Else DENY

### IAM Policies and S3 Bucket Policies

- IAM Policies are attached to users, roles, groups
- S3 Bucket Policies are attached to buckets
- When evaluating if an IAM Principal can perform an operation X on a bucket, the union of its assigned IAM Policies and S3 Bucket Policies will be evaluated

### Dynamic Policies with IAM

- How do you assigned each user a /home/<user> folder pin an S3 bucket?
- Option one
  - Create am IAM policy allowing each user to have accent to /home/<user-name>
  - Not very scalable
- Option two
  - Create one dynamic policy with IAM
  - Leverage the special policy variable ${aws:username}

### Inline vs Managed Policies

- AWS Managed Policy
  - Maintained by AWS
  - Good for power users and admins
  - Updated in case of new services/new APIs
- Customer Managed Policy
  - Best practice, reusable, can be applied to many principals
  - Version controlled + rollback, central change management
- Inline
- Strict one-to-one relationship between policy and principal
- Policy is delete Dif you delete the IAM principal

### Granting a User Permissions to pass a Role to an AWS Service

- To configure many AWS services, you must pass an IAM role to the service (this happens only once during setup)
- The service will later assume the role and perform actions
- Example of passing a role
  - To an EC2 instance, a Lambda function, an ECS task, CodePipeline to allow it to invoke other services
- For this, you may need the IAM permission iam:PassRole
- It often comes with iam:GetRole to view the role being passed
- Can a role be passed to any service?
  - No: roles can only be passed to what their trust allows
  - A trust policy for the role that allows the service to assume the role

## AWS Directory Services

- AWS Managed Microsoft AD
  - Create your own AD in AWS, manage users locally, supports MFA
  - Establish “trust” connections with your on premise AD
- AD Connector
  - Directory Gateway (proxy) to redirect to on-premise AD, supports MFA
- Simple AD
  - AD-compatible managed directory on AWS
  - Cannot be joined with on-premise AD

## What is Microsoft Active Directory (AD)?

- Found on any windows Server with AD Domain Services
- Database of objects: user accounts, computers, printers, file shares, security groups
- Centralised security management, create account, assign permission
- Objects are organised in trees
- A group of trees is a forest
