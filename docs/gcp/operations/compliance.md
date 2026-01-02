# Compliance

## Core Concepts

Compliance ensures GCP usage meets regulatory, industry, and organizational requirements. GCP provides certifications, controls, and tools to support compliance efforts.

**Key Principle**: Shared responsibility - GCP provides compliant infrastructure, customers ensure compliant usage.

## Shared Responsibility Model

| Layer | Google's Responsibility | Customer's Responsibility |
|-------|------------------------|--------------------------|
| **Infrastructure** | Physical security, hardware | N/A |
| **Platform** | Service certifications, controls | Configuration, access control |
| **Data** | Encryption at rest/transit | Data classification, handling |
| **Access** | IAM service availability | User permissions, authentication |
| **Compliance** | Infrastructure certifications | Workload compliance validation |

## Common Compliance Frameworks

### HIPAA (Healthcare)

**Requirements**:

- Encryption at rest and in transit
- Access controls and audit logging
- Business Associate Agreement (BAA)
- Physical safeguards

**GCP Features**:

- BAA available for covered services
- CMEK for additional control
- Access Transparency logs
- VPC Service Controls (data perimeter)
- Audit logs (Admin Activity, Data Access)

**Customer Actions**:

- Sign BAA with Google
- Enable data access logs
- Implement access controls
- Regular access reviews
- Use covered services only

### PCI-DSS (Payment Card Industry)

**Requirements**:

- Network segmentation
- Encryption of cardholder data
- Access controls
- Regular security testing
- Maintain security policies

**GCP Features**:

- PCI-DSS Level 1 certification
- VPC for network isolation
- Cloud Armor for protection
- Web Security Scanner
- Cloud KMS for encryption

**Customer Actions**:

- Scope PCI environment (minimize)
- Network segmentation (VPC, firewall rules)
- Quarterly vulnerability scans
- No cardholder data in logs
- Implement application controls

### GDPR (EU Data Protection)

**Requirements**:

- Data residency (EU data stays in EU)
- Right to erasure
- Data protection by design
- Breach notification
- Data Processing Agreement (DPA)

**GCP Features**:

- EU regions available
- Organization Policies (location restrictions)
- DPA available
- Data deletion APIs
- Audit logs for compliance evidence

**Customer Actions**:

- Data residency: Use EU regions only
- Organization Policy: Restrict resource locations
- Data inventory and classification
- Implement deletion processes
- Privacy impact assessments

### SOX (Sarbanes-Oxley)

**Requirements**:

- Financial data controls
- Audit trails
- Segregation of duties
- 7-year data retention

**GCP Features**:

- SOC 1/2/3 reports available
- Audit logs (400-day retention for admin)
- IAM for segregation of duties
- Log export for long-term retention

**Customer Actions**:

- Enable audit logs
- Export logs to Storage (7-year retention)
- Implement least privilege
- Separate production access
- Document controls

### ISO 27001 (Information Security)

**Requirements**:

- Information security management system
- Risk assessment
- Security controls
- Continuous improvement

**GCP Features**:

- ISO 27001 certified
- ISO 27017 (cloud security)
- ISO 27018 (privacy)
- Security Command Center

**Customer Actions**:

- Implement ISMS
- Regular risk assessments
- Use GCP security features
- Document security controls

## GCP Compliance Tools

### Compliance Reports Manager

**Purpose**: Access compliance reports and certifications

**Available**:

- SOC 1/2/3
- ISO 27001/27017/27018
- PCI-DSS AOC
- HIPAA attestation

**Access**: Through GCP Console

### Security Command Center

**Purpose**: Centralized security and compliance dashboard

**Features**:

- Asset inventory and discovery
- Vulnerability scanning
- Compliance monitoring (CIS benchmarks)
- Security findings aggregation
- Policy violations

**Tiers**: Standard (free), Premium (paid)

### Policy Intelligence

**Tools**:

- IAM Recommender (over-permissioned accounts)
- Policy Analyzer (who has access)
- Policy Simulator (test policy changes)
- Policy Troubleshooter (debug access issues)

**Use for**: Least privilege enforcement, access reviews

### Access Transparency

**Purpose**: Logs of Google employee access to your data

**Use case**: Compliance requirement for transparency

**Available**: For Premium support customers

**Content**: Who, what, when, why Google accessed

## Data Residency and Sovereignty

### Resource Locations

**Organization Policy**: `constraints/gcp.resourceLocations`

**Example**: Restrict to EU regions only

```
Allowed values: in:eu-locations
Result: Resources can only be created in EU
```

**Enforcement**: Blocks creation in non-compliant regions

### Data Classification

**Levels**:

- Public: No restrictions
- Internal: Access controlled
- Confidential: Encrypted, strict access
- Restricted: Maximum security (CMEK, VPC-SC)

**Implementation**:

- Labels for classification
- Different projects for different levels
- Appropriate security controls per level

### Assured Workloads

**Purpose**: Enforces location and access restrictions for compliance

**Features**:

- Guaranteed data location
- Personnel access controls (US only, no foreign nationals)
- Encryption requirements
- Regular compliance monitoring

**Use for**: Government (FedRAMP, IL4), highly regulated industries

## Encryption and Key Management

### Encryption Options

**Default** (Google-managed):

- Automatic encryption at rest
- No configuration needed
- Google manages keys

**CMEK** (Customer-managed):

- Customer controls keys in Cloud KMS
- Can revoke access
- Key rotation policies
- Audit key usage

**CSEK** (Customer-supplied):

- Customer provides keys per operation
- Google doesn't store keys
- Maximum control, maximum complexity

**Decision**:

- Default: Most workloads
- CMEK: Compliance requires customer control
- CSEK: Maximum security, rare

### Key Management Best Practices

- Separate keys per environment (dev, prod)
- Regular key rotation (automatic in KMS)
- Least privilege for key access
- Audit key usage
- Destroy keys when no longer needed

## Audit Logging

### Log Types for Compliance

**Admin Activity** (always on):

- Who did what (API calls)
- 400-day retention
- Free

**Data Access** (must enable):

- Who accessed what data
- 30-day default retention
- Chargeable

**System Events**:

- GCP-initiated actions
- Automatic

**Access Transparency**:

- Google employee access
- Premium support only

### Log Retention Requirements

| Framework | Retention | Implementation |
|-----------|-----------|----------------|
| HIPAA | 6 years | Export to Cloud Storage |
| SOX | 7 years | Export to Cloud Storage |
| PCI-DSS | 1 year | Default + export |
| GDPR | Per policy | Configurable |

**Pattern**: Export logs to Cloud Storage with retention policy

## Access Controls for Compliance

### Principle of Least Privilege

**Implementation**:

- Predefined roles (not basic roles)
- Custom roles for specific needs
- Regular access reviews (quarterly)
- Remove unused permissions

**Tools**: IAM Recommender for over-permissioned accounts

### Segregation of Duties

**Pattern**: Separate roles for different functions

**Examples**:

- Compute Admin ≠ Network Admin
- Developer (create resources) ≠ Security Admin (set policies)
- No single person has complete access

**Implementation**: Multiple administrators, separate roles

### MFA (Multi-Factor Authentication)

**Requirement**: Most compliance frameworks require MFA

**Enforcement**:

- Workspace/Cloud Identity policy
- Mandatory for admin accounts
- Context-aware access (IAP)

## Network Security for Compliance

### Network Segmentation

**Methods**:

- Separate VPCs per environment
- VPC firewall rules (default deny)
- Private Google Access (no internet)
- Cloud NAT (controlled egress)
- VPC Service Controls (data perimeter)

**PCI-DSS**: Requires network segmentation for cardholder data environment

### VPC Service Controls

**Purpose**: Create security perimeter around resources

**Benefits**:

- Prevent data exfiltration
- Restrict API access to perimeter
- Complements IAM

**Use case**: Protect sensitive data (PII, PHI, financial)

### Private Connectivity

**Options**:

- Cloud VPN (encrypted tunnel)
- Cloud Interconnect (dedicated connection)
- Private Service Connect (private access to services)

**Use for**: Hybrid compliance, data cannot traverse internet

## Compliance Monitoring

### Continuous Compliance

**Approach**:

- Organization Policies (preventive)
- Security Command Center (detective)
- Audit logs (evidence)
- Regular reviews (corrective)

**Automation**: Policy violations trigger alerts, automated remediation

### Compliance Reporting

**Evidence Collection**:

- Audit logs exported and retained
- Security Command Center reports
- Access reviews documented
- Policy enforcement documented

**Audits**: Provide evidence to auditors via Compliance Reports Manager

## Incident Response

### Data Breach Notification

**GDPR**: 72-hour notification requirement

**Preparation**:

- Incident response plan
- Contact lists (DPO, legal)
- Detection mechanisms (Cloud Monitoring alerts)
- Evidence collection (audit logs)

**GCP Support**: Premium support for incident assistance

### Forensics

**Tools**:

- Cloud Logging (what happened)
- VPC Flow Logs (network traffic)
- Access Transparency (Google access)
- Disk snapshots (preserve evidence)

**Best Practice**: Enable comprehensive logging before incident

## Third-Party Risk Management

### Vendor Assessment

**Questions**:

- What certifications do they have?
- Where is data stored/processed?
- What access do they have?
- How is data encrypted?
- What are their security practices?

**GCP Certifications**: ISO, SOC, PCI, HIPAA, FedRAMP

### Subprocessors

**GDPR requirement**: Know where data is processed

**GCP**: Transparent list of subprocessors

**Action**: Review and approve subprocessor list

## Compliance by Service

### HIPAA-Covered Services

**Covered**: Compute Engine, Cloud Storage, BigQuery, Cloud SQL, GKE

**NOT Covered**: Firebase, Cloud Datastore (some exceptions), App Engine Standard

**Check**: Compliance documentation for current list

### PCI-DSS Scope

**In-scope**: Infrastructure services (GCE, GCS, VPC)

**Customer responsibility**: Application-level controls

**Best practice**: Minimize PCI scope (tokenization, separate environment)

## Best Practices

### Design for Compliance

- Data classification from start
- Encryption by default
- Principle of least privilege
- Comprehensive logging
- Regular security reviews

### Documentation

- Architecture diagrams
- Data flow diagrams
- Security controls documentation
- Policy documentation
- Incident response plan

### Regular Reviews

- Quarterly access reviews
- Annual security assessments
- Policy effectiveness reviews
- Penetration testing (if required)
- Audit log reviews

### Automation

- Organization Policies (enforce)
- Security Command Center (detect)
- Cloud Functions (remediate)
- Infrastructure as Code (consistency)

## Exam Focus

### Shared Responsibility

- What Google provides vs customer responsibilities
- Infrastructure security vs application security
- Compliance certifications vs compliance validation

### Framework Requirements

- HIPAA: BAA, encryption, access logs
- PCI-DSS: Network segmentation, scans, no CHD in logs
- GDPR: Data residency, right to erasure, DPA
- SOX: 7-year retention, segregation of duties

### GCP Features

- Organization Policies for governance
- VPC Service Controls for data perimeter
- CMEK for key control
- Audit logs for evidence
- Security Command Center for monitoring

### Architecture

- Data residency (location restrictions)
- Network segmentation (VPC, firewall rules)
- Encryption options (default, CMEK, CSEK)
- Log retention and export
- Access control patterns

### Common Patterns

- Separate projects per environment
- Export logs for long-term retention
- Organization Policy for location restriction
- VPC Service Controls for sensitive data
- Least privilege with regular reviews
