# AWS Identity and Access Management (IAM)

AWS Identity and Access Management (IAM) is a service provided by AWS that allows you to manage access to AWS services and resources securely. IAM enables fine-grained control over permissions, ensuring that users, groups, and services have the appropriate level of access.

## Key Characteristics of IAM

### **1. User and Group Management**

- Create and manage individual user accounts.
- Organize users into groups for easier permission management.
- Assign permissions to groups instead of individual users.

### **2. Fine-Grained Permissions**

- Define permissions at a granular level using IAM policies.
- Policies use JSON format to specify which actions are allowed or denied on which resources.
- Support for resource-level permissions.

### **3. Role-Based Access Control**

- Create IAM roles to delegate access without sharing credentials.
- Assign roles to AWS services (e.g., EC2 instances, Lambda functions) to grant them permissions.
- Supports cross-account access by defining trust relationships.

### **4. Temporary Security Credentials**

- Generate temporary credentials for users or applications to access AWS resources.
- Ideal for scenarios requiring short-term access, such as federated users or applications.

### **5. Multi-Factor Authentication (MFA)**

- Enhance security by requiring users to authenticate with an additional factor (e.g., a one-time code).
- Supports both hardware and virtual MFA devices.

### **6. Integration with AWS Services**

- Integrated with all AWS services for access control.
- Provides a consistent mechanism for granting and revoking permissions across AWS.

### **7. Policy Types**

- **Managed Policies**: Predefined policies by AWS or custom policies created by users.
- **Inline Policies**: Policies embedded directly in a user, group, or role.
- **Service Control Policies (SCPs)**: Used with AWS Organizations to set permission boundaries at the organizational level.

### **8. Permission Boundaries**

- Define the maximum permissions a user or role can have.
- Prevent users from escalating privileges or performing unauthorized actions.
- Used to ensure roles or users do not exceed predefined permissions.

### **9. Resource Tags for Access Control**

- Use resource tags to define permissions based on resource metadata.
- Simplifies permission management for large-scale environments.

### **10. Identity Federation**

- Support for Single Sign-On (SSO) using external identity providers (e.g., Microsoft Active Directory, SAML, or OIDC).
- Enables users to log in with corporate credentials.

### **11. Trust Policy**

- A JSON document that defines which entities (e.g., users, roles, accounts) can assume a role.
- Key for cross-account or external access delegation.
- Specifies the principal, conditions, and actions allowed for assuming the role.

### **12. ACL (Access Control List)**

- Used to manage access to specific AWS resources, primarily Amazon S3 buckets and objects.
- Specifies which AWS accounts or users are granted access and the type of operations allowed (e.g., read, write).
- Less granular and flexible compared to IAM policies.

### **13. SCP (Service Control Policy)**

- Policies applied at the organizational level in AWS Organizations.
- Control what services and actions can be used by accounts within an organization or organizational unit (OU).
- Does not grant permissions but sets limits on the maximum permissions available.

### **14. Audit and Compliance**

- Provides detailed logs of IAM-related activities via AWS CloudTrail.
- Simplifies auditing and compliance with built-in monitoring and reporting.

### **15. Free Tier**

- IAM is free of charge; you only pay for the resources accessed using IAM policies.

---

IAM is a foundational service that ensures secure and controlled access to AWS resources. With its fine-grained permissions, role-based access, and integration capabilities, IAM helps organizations manage their AWS environment securely and efficiently.
