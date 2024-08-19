# Cognito: User Pools, Identity Pools and Sync

- Give uses an identity to interact with our web or mobile application
- Cognito User pools
  - Sign in functionality for app users
  - Integrate with API Gateway & Application Load Balancer
- Cognito Identity Pools (Federated Identity)
  - Provide AWS credentials to users so that they can access AWS resources directly
  - Integrate with Cognito User Pools as an identity provider
- Cognito vs IAM: “hundreds of users”, “mobile users”, “authenticate with SAML”

## User Pools

### User Features

- Create a server less database of user for you web and mobile apps
- Simple login: username (or email) / password combination
- Password reset
- Email and phone number verification
- Multi-factor authentication (MFA)
- Federated identities: users from Google, Facebook, SAML etc.
- Feature: block users if their credentials are compromised elsewhere
- Login sends back a JWT (JSON Web Token)

## Integrations

- CUP integrates with API Gateway and Application Load Balancer

## Other

- Lambda triggers: CUP can invoke a Lambda function synchronously on the triggers below
  ![alt text](<Screenshot 2024-08-19 at 09.52.51.png>)
- Hosted Authentication UI
  - Cognito has a hosted authentication UI that you can add to your app to handle sign up and sign workflows
  - Using the hosted UI, you have a foundation for integration with social logins, OIDC or SAML
  - Can customise with custom logo or CSS
- Hosted UI Custom Domain
  - For custom domains, you must create an ACM certificate in us-east-1
  - The custom domain must be defined in the “App Integration” section
- Adaptive authentication
  - Block sign-ins or require MFA if the login appears suspicious
  - Cognito examines each sign-in attempt and generates a risk score (low, medium, high) for how likely the sign-in request is to be from a malicious attacker
  - Users are prompted for a second MFA only when risk is detected
  - Risk score is based on different factors such as if the user has used the same device, location or IP address
  - Checks for compromised credentials, account takeover protection and phone and email verification
  - Integration with CloudWatch Logs (sign in attempts, risk score, failed challenges etc.)
- Decoding an ID Token (JWT)
  - CUP issues JWT tokens (Base64 encoded)
    - Header
    - Payload
    - Signature
  - The signature must be verified to ensure the JWT can be trusted
  - Libraries can help you verify the validity of JWT tokens issued by Cognito User Pools
  - The payload will contain the user information (sub UUID, given_name, email, phone_number, attributes)
  - From the sub UUID, you can retrieve all users details from Cognito/OIDC

## Application Load Balancer: User Authentication

### Authenticate Users

- Your Application Load Balancer can securely authenticate users
  - Offload the work of authenticating users to your load balancer
  - Your applications can focus on their business login
- Authenticate users through:
  - Identity Provider (IdP): OpenID Connect (OIDC) compliant
  - Cognito User Pools
    - Social IdPs, such as Amazon, Facebook or Google
- Must use an HTTPS listener to set authenticate-cidc and authenticate-cogito rules
- OnUnauthenticatedRequest - authenticate (default), deny, allow

### Using Cognito User Pools

- Create Cognito User Pool, Client and Domain
- Make sure an ID token is returned
- Add the social or Corporate IdP if needed
- Several URL redirections are necessary
- Allow your Cognito User Pool Domain on your IdP app’s callback URL
  - Eg: https//domain-prefix.auth.region.amazoncognito.com/saml2/idpreresponse

### Through an Identity Provider (IdP) That is OpenID Connect (OIDC) Compliant

- Configure a client ID & Client Secret
- All redirect from OIDC to your Application Load Balancer DNS name (AWS provided) and CNAME (DNS Alias of your app)
  - https://DNS/oauth2/idpresonse
  - https://CNAME/oauth2/idpresponse

### Cognito Identity Pools (Federated Identities)

- Get identities for “users” so they obtain temporary AWS credentials
- Your identity pool (eg. Identity source) can include:
  - Public providers (login with Amazon, Facebook, Google, Apple)
  - Users in an Amazon Cognito user pool
  - Open ID Connect Providers & SAML Identity Providers
  - Developer Authenticated Identities (custom login server)
  - Cognito Identity Pools allow for unauthenticated (guest) access
- Users can then access AWS services directly or through API Gateway
  - The IAM policies applied to the credentials are defined in Cognito
  - They can be customised based on the user_id for fine grained control

### IAM Roles

- Default IAM roles for authenticated and guest users
- Define rules to choose the role for each user based on the user’s ID
- You can partition your users’ access using policy variables
- IAM credentials are obtained by Cognito Identity Pools through STS
- The roles must have a “trust” policy of Cognito Identity Pools

## Cognito User Pools vs Cognito Identity Pools

- Cognito User Pools (for authentication = identity verification)
  - Database of users for your web and mobile application
  - Allows to federate logins through Public Social, OIDC, SAML
  - Can customise the hosted UI for authentication (including the logo)
  - Has triggers with AWS Lambda during the authentication flow
  - Adapt the sign in experience to different risk levels (MFA, adaptive authentication etc.)
- Cognito Identity Pools (for authorisation = access control)
  - Obtain AWS credentials for your users
  - Users can login through Public social, OIDC, SAML and Cognito User Pools
  - Users can unauthenticated (guests)
  - Users are mapped to IAM roles and policies, can leverage policy variables
- CUP + CIP = authentication + authorisation
