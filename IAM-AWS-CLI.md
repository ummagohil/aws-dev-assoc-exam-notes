# IAM & AWS CLI

- IAM: Identity and Access Management
- A global service

## Users, Groups, Policies

- Root account created by default, shouldn't be shared or used
- Users: people within an organisation (can be grouped)
- Groups: only contain users and not other groups
- Users don't have to belong to a group
- One user can belong to multiple groups
- Groups are usually set up for permissions
- Users or groups can be assigned JSON documents called policies (these policies define the permissions set for users)
- Best practise is to apply the least privilege principle (don't give more permission than a user needs)

## IAM Policies

- Inheritance: if a user belongs to two groups, they will inherit policies from both groups
- Policies structure consists of:
  - Version number
  - ID (optional identifier for the policy)
  - Statement (required - one or more individual statements) which consist of:
    - Sid: an optional identifier for the statement
    - Effect: whether the statement allows or denies access (allow/deny)
    - Principal: account/user/role to which this policy applied to
    - Action: list of actions this policy allows or denies
    - Resource: list of resources to which the action applies to
    - Condition: conditions for when this policy is in effect (optional)

## MFA

- Password policy: the stronger the password, the higher the security for your account
  - Set a minimum password length
  - Require specific character types: uppercase, lowercase, numbers and non-alphanumeric characters
  - Allow all IAM users to change their own passwords
  - Require users to change their password after some time (password expiration)
  - Prevent password re-use
- Users have access to your account and can possibly change configurations or delete resources in your AWS account
- You want to protect your Root Accounts and IAM users
- MFA = passwords you know + security device you own
- Main benefit: if a password is stolen or hacked, the account is not compromised
- Device options in AWS: virtual MFA device (authy, google authenticator etc), universal 2nd factor (U2F) security key (eg. YubiKey), hardware key fob MFA device or hardware key fob MFA device for AWS GovCloud (US)
- Virtual MFA devices have support for multiple token on a single device
- Universal 2nd Factor Security Keys support multiple root and IAM users using a single security key

## AWS Access Keys, CLI and SDK

- AWS can be accessed via:
  - AWS Management Console (protected by password + MFA)
  - AWS Command Line Interface (CLI): protected by access keys
  - AWS Software Developer Kit (SDK): for code - protected by access keys
- Access keys are generated through the AWS Console
- Users manage their own access keys
- Access Keys are secret, like password and shouldn't be shared
- Access Key ID is usually the username
- Secret Access Key is usually the password
- AWS CLI:
  - A tool that enables you to interact with AWS services using command in your command-line shell
  - Direct access to the public APIs of AWS services
  - You can develop scripts to manage your resources
  - It's open source (github.com/aws/aws-cli)
  - Alternative to using AWS Management Console
- AWS SDK:
  - AWS Software Development Kit (AWS SDK)
  - Language-specific APIs (set of libraries)
  - Enables you to access and manage AWS services programmatically
  - Embedded within your application
  - Supports:
    - SDKs (JavaScript, Python, PHP, .NET, Ruby, Java, Go, Node.js, C++)
    - Mobile SDKs (Android, iOS)
    - IoT Device SDKs (Embedded C, Arduino)
  - Example: AWS CLI is built on AWS SDK for Python

## AWS CloudShell

- Alternative to using the command line
- The icon on the top nav bar on the right hand side (it's globally available)
- Example commands:
  - aws iam list-users
  - aws iam list-users --region
- Full repo via shell commands like ls, cd, echo etc.
- Can upload your own files

## IAM Roles for AWS Services

- Some AWS services will need to perform actions on your behalf
- To do so, assign permission to AWS services via IAM Roles
- Common roles:
  - EC2 Instance Roles
  - Lambda Function Roles
  - Roles for CloudFormation

## Security Tools (Audit)

- IAM Credentials Report (account-level): a report that lists all your account's users and the status of their various credentials
- IAM Access Advisor (user-level): shows the service permissions granted to a user and when those services were last accessed

## IAM Best Practices

- Don't use the root account except for AWS account setup
- One physical user = one aws user
- Assign users to groups and assign permission to groups
- Create a strong password policy
- Use and enforce the use of Multi Factor Authentication (MFA)
- Create and use Roles for giving permissions to AWS services
- Use Access Keys for Programmatic Access (CLI/SDK)
- Audit permissions of your account using IAM Credentials Report & IAM Access Advisor
- Never share IAM users & Access Keys

## Shared Responsibility Model for IAM

- AWS' responsibilities:
  - Infrastructure (global network security)
  - Configuration and vulnerability analysis
  - Compliance validation
- Your responsibilities:
  - Users, Groups, Roles, Policies management and monitoring
  - Enable MFA on all accounts
  - Enable MFA on all accounts
  - Rotate all your keys often
  - Use IAM tools to apply appropriate permissions
  - Analyse access patterns and review permissions
