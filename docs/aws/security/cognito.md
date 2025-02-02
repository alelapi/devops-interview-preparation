# Cognito

## Introduction

Amazon Cognito is a service that enables user identity and access management for web and mobile applications. It provides two main components: Cognito User Pools (CUP) and Cognito Identity Pools (Federated Identities), each serving distinct purposes in the authentication and authorization workflow.

## Cognito User Pools (CUP)

### Overview

Cognito User Pools provide a serverless database solution for managing user identities in web and mobile applications. As the primary authentication mechanism, CUP handles user sign-up, sign-in, and integrates seamlessly with various identity providers to create a comprehensive identity management system.

### Core Features

The User Authentication system in Cognito User Pools offers a comprehensive set of features for secure user management. Users can sign in using either username or email combined with their password. The system includes built-in password reset functionality and supports both email and phone number verification to ensure user authenticity. Multi-factor authentication (MFA) adds an additional layer of security, while federated identity support enables integration with providers such as Facebook, Google, and SAML. The system actively protects against compromised credentials and issues JSON Web Tokens (JWT) upon successful authentication.

### Lambda Triggers Integration

Cognito User Pools can invoke Lambda functions at various stages of the authentication process. During authentication events, pre-authentication triggers enable custom validation of sign-in requests, while post-authentication triggers facilitate event logging for analytics purposes. The pre-token generation phase allows for token claim modification.

For sign-up operations, the system provides pre-sign-up validation capabilities, post-confirmation processing for welcome messages, and user migration support from existing directories. Additional customization options include message personalization and token attribute modification through pre-token generation triggers.

### Hosted Authentication UI

Cognito provides a hosted authentication interface that includes pre-built sign-up and sign-in workflows. This UI seamlessly integrates with social logins, OIDC, and SAML providers. Organizations can customize the interface with their own logos and CSS styles. Custom domain support is available, though it requires an ACM certificate in the us-east-1 region.

### Adaptive Authentication

The adaptive authentication system provides sophisticated security measures by analyzing login attempts and assigning risk scores (low, medium, high) based on various factors. The system monitors device usage, location patterns, and IP addresses to detect suspicious activities. It includes protection against compromised credentials and account takeover attempts. For monitoring and analysis, the system integrates with CloudWatch Logs to track sign-in attempts, risk scores, and authentication challenges.

### JWT Token Structure

The JSON Web Token issued by Cognito consists of three main components: a header, payload, and signature. The payload contains essential user information, including the user UUID (sub), given name, email, phone number, and any additional custom attributes. The signature verification ensures the token's authenticity and trustworthiness.

## Application Load Balancer Integration

### Authentication Capabilities

The Application Load Balancer can handle user authentication, removing this burden from application servers. It supports various identity providers that are OIDC compliant and integrates directly with Cognito User Pools. Authentication rules require HTTPS listeners, and administrators can configure how the system handles unauthenticated requests through authenticate, deny, or allow options.

### Implementation Requirements

Implementing ALB authentication requires proper setup of the Cognito User Pool, Client, and Domain. The configuration must ensure proper ID token return and handle URL redirections appropriately. When integrating social or corporate identity providers, careful attention must be paid to callback URL configuration.

## Cognito Identity Pools (Federated Identities)

### Core Functionality

Identity Pools extend Cognito's capabilities by providing temporary AWS credentials to users. This system supports multiple identity sources and enables both authenticated and guest access to AWS services. Users can interact directly with AWS services or through API Gateway, with access controlled by customizable IAM policies based on user identity.

### Identity Source Support

The system accepts identities from various sources, including public providers like Amazon, Facebook, Google, and Apple. It integrates with Cognito User Pools and supports OpenID Connect and SAML identity providers. Organizations can also implement custom login servers through Developer Authenticated Identities.

### IAM Role Management

Identity Pools manage AWS access through IAM roles, with separate default roles for authenticated and guest users. Role selection can be customized based on user attributes, and access can be partitioned using policy variables. The system obtains credentials through AWS STS and requires appropriate trust policies.

## Comparing User Pools and Identity Pools

Understanding the distinction between User Pools and Identity Pools is crucial for implementing effective authentication and authorization strategies. User Pools focus on authentication, providing user directory management, federated login support, and customizable interfaces. Identity Pools handle authorization by managing AWS credentials and access control through IAM policies.

## Security Best Practices

Security in Cognito requires a comprehensive approach that includes proper IAM policy implementation, MFA enablement for sensitive operations, and regular monitoring through CloudWatch. Organizations should carefully configure trust relationships, implement least privilege access principles, and conduct regular security assessments to maintain a robust security posture.
