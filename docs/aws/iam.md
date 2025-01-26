# Identity and Access Management (IAM)

AWS Identity and Access Management (IAM) secures AWS resources by managing access through policies. Policies are JSON documents that define permissions, ensuring only authorized entities can perform actions. By default, all requests are denied unless explicitly allowed by a policy.

## Overview of Policies in AWS IAM

**Policy Types**: AWS supports seven types of policies:

- **Identity-based policies**: Attached to users, groups, or roles.
- **Resource-based policies**: Attached to resources like S3 buckets.
- **Permissions boundaries**: Limit the maximum permissions for an identity.
- **Organizations SCPs (Service Control Policies)**: Limit permissions across accounts in an organization.
- **Organizations RCPs (Resource Control Policies)**: Limit permissions for resources across accounts.
- **Access Control Lists (ACLs)**: Control access to resources from other accounts.
- **Session policies**: Limit permissions for temporary sessions.

## Policy Structure and Evaluation

- **JSON Policy Documents**: Most policies are stored as JSON documents.
- **Policy Elements**: Include `Version`, `Statement`, `Sid`, `Effect`, `Principal`, `Action`, `Resource`, and `Condition`.
- **Policy Evaluation**: AWS applies a logical `OR` across multiple statements and policies. An explicit deny overrides any allow.

## Key Concepts

- **Managed Policies**: Standalone policies that can be attached to multiple identities.
  - **AWS Managed Policies**: Created and managed by AWS.
  - **Customer Managed Policies**: Created and managed by users.
- **Inline Policies**: Policies added directly to a single identity.
- **Permissions Boundaries**: Define the maximum permissions an identity can have.

## Policy Use Cases

- **Identity-Based Policies**: Grant permissions to users, groups, or roles.
- **Resource-Based Policies**: Grant permissions to access specific resources.
- **Cross-Account Access**: Use resource-based policies to allow access from other accounts.

## Tools

- **IAM Access Analyzer**: AWS IAM Access Analyzer helps you identify the resources in your organization and accounts, such as Amazon S3 buckets or IAM roles, that are shared with an external entity. This lets you identify unintended access to your resources and data, which is a security risk.

You can set the scope for the analyzer to an organization or an AWS account. This is your zone of trust. The analyzer scans all of the supported resources within your zone of trust. When Access Analyzer finds a policy that allows access to a resource from outside of your zone of trust, it generates an active finding.

- **Access Advisor**: To help identify the unused roles, IAM reports the last-used timestamp that represents when a role was last used to make an AWS request. Your security team can use this information to identify, analyze, and then confidently remove unused roles. This helps improve the security posture of your AWS environments. This does not provide information about non-IAM entities such as S3, hence it's not a correct choice here.

## Best Practices

- Use the AWS Management Console's visual editor to create policies without needing to write JSON.
- Use IAM Access Analyzer for policy validation and recommendations.
- Organize policies by resource type for clarity and manageability.

### Root Account Security
Protect your AWS root account by:
- Never using root credentials for daily operations
- Enabling multi-factor authentication (MFA)
- Creating dedicated administrative IAM users for management tasks

### Principle of Least Privilege
Implement strict access control by:
- Granting minimal permissions required for each task
- Avoiding blanket "*" service access permissions
- Regularly reviewing and refining user, group, and role permissions

### Credential Handling
Maintain robust security for access credentials:
- Store IAM credentials only on secure personal computers or on-premise servers
- Utilize AWS Security Token Service (STS) for temporary security credentials
- Avoid hardcoding or sharing access keys

## Service-Specific Role Management

### Dedicated Roles for Services
Create isolated, purpose-specific roles for:
- EC2 instances
- Lambda functions
- ECS tasks
- CodeBuild projects

### Role Design Principles
- Generate least-privileged roles for each service
- Create unique roles per application or function
- Avoid role reuse across different applications

## Cross-Account Access Strategy

### Secure Inter-Account Permissions
- Define specific IAM roles for cross-account access
- Explicitly specify which accounts can assume these roles
- Use STS to generate temporary credentials
- Limit credential validity to 15-60 minutes

## Policy Evaluation and Management

### IAM and S3 Bucket Policies
- IAM Policies are attached to users, roles, and groups
- S3 Bucket Policies are attached directly to buckets
- Access evaluation combines (unions) IAM Policies and S3 Bucket Policies

### Policy Interaction
When determining access permissions, AWS evaluates:
- IAM policies attached to users, roles, and groups
- Resource-specific policies (such as S3 bucket policies)

### Monitoring and Auditing
- Continuously monitor API calls using CloudTrail
- Pay special attention to denied access attempts
- Regularly audit and update access permissions

## Continuous Security Improvement

- Implement regular security reviews
- Use AWS IAM Access Analyzer to identify unintended resource access
- Stay updated on AWS security best practices and features


This summary covers the core concepts and types of policies in AWS IAM, providing a foundation for managing access and permissions effectively within AWS environments.
