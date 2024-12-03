
# How do you handle configuration drift in infrastructure management to maintain consistency across environments?

## Answer

## Answer

### Managing Infrastructure as Code with Terraform
1. **Lifecycle Management**:
   - Define infrastructure using HCL (HashiCorp Configuration Language).
   - Use version control to track changes.

2. **Preventing Drift**:
   - Regularly use `terraform plan` to detect discrepancies between the desired and actual state.

3. **Multi-Cloud Support**:
   - Terraform supports providers for AWS, GCP, Azure, and more.

### Tools:
- Ansible for configuration management.
- Terraform for provisioning.