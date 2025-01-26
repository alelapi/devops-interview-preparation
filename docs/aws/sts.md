# Security Token Service (STS)

## Core Purpose

AWS Security Token Service enables organizations to grant limited and temporary access to AWS resources, with credential validity up to one hour.

## Credential Generation Methods

STS provides multiple mechanisms for obtaining temporary credentials:

### Role Assumption Methods
- AssumeRole: Enables role assumption within or across AWS accounts
- AssumeRoleWithSAML: Returns credentials for SAML-authenticated users
- AssumeRoleWithWebIdentity: Generates credentials for users logged via identity providers (Facebook, Google, OIDC)
- GetSessionToken: Supports multi-factor authentication for users and root accounts
- GetFederationToken: Obtains temporary credentials for federated users
- GetCallerIdentity: Returns details about the IAM user or role used in an API call
- DecodeAuthorizationMessage: Decodes error messages when an AWS API is denied

## Role Assumption Process

To assume a role using STS:
- Define an IAM Role within your account or across accounts
- Specify which principals can access the role
- Use the AssumeRole API to retrieve credentials
- Utilize temporary credentials valid between 15 minutes to 1 hour

## Multi-Factor Authentication (MFA) Support
STS provides robust MFA capabilities through the `GetSessionToken` method:
- Requires appropriate IAM policy with condition `aws:MultiFactorAuthPresent:true`
- Generates temporary credentials including:
    - Access ID
    - Secret Key
    - Session Token
    - Expiration date

## Important Note

For web identity authentication, AWS recommends using Cognito Identity Pools instead of direct STS web identity credential generation.
