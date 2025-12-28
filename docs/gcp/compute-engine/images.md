# Machine Images and Disk Images

## Core Concepts

Images capture VM or disk state for replication, backup, and standardization. Understanding the differences between machine images, disk images, and snapshots is critical for architecture decisions.

## Machine Images vs Disk Images vs Snapshots

| Feature | Machine Image | Disk Image | Snapshot |
|---------|--------------|------------|----------|
| **Scope** | Entire VM | Single disk | Single disk |
| **Configuration** | Yes (all settings) | No | No |
| **Multiple Disks** | Yes (all attached) | No | No |
| **Storage Method** | Full copy | Full copy | Incremental |
| **Cost** | Highest | Medium | Lowest (incremental) |
| **Creation Time** | Slowest | Medium | Fast (after first) |
| **Use Case** | VM cloning, DR | OS templates | Regular backups |
| **Frequency** | Weekly/monthly | One-time | Daily/hourly |

## Machine Images

### Architecture Purpose

**Complete VM Backup**:

- All disk data preserved
- VM configuration captured (machine type, labels, network settings)
- Metadata and startup scripts included
- Service account configuration retained

**When to Use**:

- Complete VM backup and restore
- VM migration between projects/regions
- Disaster recovery (weekly/monthly full backups)
- Creating identical VMs across environments
- Compliance snapshots (full system state)

**When Not to Use**:

- Regular incremental backups (use snapshots instead)
- Single disk backup (use snapshots)
- OS templates (use disk images)
- Frequent backups (too expensive)

### Design Patterns

**DR Strategy**:

- Weekly machine images for complete recovery
- Daily snapshots for incremental backups
- Store in multi-regional location
- Test restoration quarterly

**VM Migration**:

- Create machine image in source project
- Share with target project via IAM
- Restore in target project/region
- Update network and external integrations

**Environment Replication**:

- Production machine image
- Restore as staging environment
- Modify labels and metadata
- Separate network configuration

## Disk Images

### Architecture Purpose

**OS and Software Templates**:

- Base operating system
- Pre-installed software
- Security hardening applied
- Configuration management tools
- Company standards enforced

**When to Use**:

- Creating standard VM configurations
- Instance templates for MIGs
- Golden images for teams
- Consistent deployments
- OS patching strategy (update image, redeploy VMs)

**When Not to Use**:

- Backup purposes (use snapshots)
- Capturing running VM state (use machine images)
- Incremental updates (use configuration management)

### Image Families

**Concept**: Logical grouping where latest image used automatically

**Benefits**:

- Version control for images
- Automatic use of latest
- Easy rollback to previous
- Staged rollouts

**Pattern**:

```
Image Family: "web-server"
├── web-server-v1 (deprecated)
├── web-server-v2 (obsolete)
└── web-server-v3 (active, automatically used)
```

**Architecture Decision**: Always use image families in instance templates, never specific image names

### Public Images

**Use Cases**:

- Quick testing/development
- Standard OS deployments
- Community-maintained images
- Starting point for custom images

**Considerations**:

- No customization
- Google or community maintained
- May not meet security standards
- Use as base for custom images

**Pattern**: Public image → Customize → Save as custom image in family

## Image Lifecycle Management

### Deprecation States

**States**:

- **ACTIVE**: Available for use
- **DEPRECATED**: Available but not recommended, shows warning
- **OBSOLETE**: Available but strongly discouraged, shows error
- **DELETED**: No longer available

**Strategy**:

1. Release new image (ACTIVE)
2. Deprecate old image (after testing period)
3. Mark obsolete (after migration period)
4. Delete (after retention period)

**Architecture Pattern**: Gradual deprecation allows staged migrations

## Cross-Project Sharing

### Use Cases

**Organization Pattern**:

- Central image repository project
- Shared across all projects
- Consistent base images
- Centralized maintenance

**Security Pattern**:

- Share via IAM (not public)
- Principle of least privilege
- Service account access
- Audit image usage

**Multi-Project Architecture**:

- Host project: Maintains images
- Service projects: Consume images
- Reduces duplication
- Single source of truth

## Image Creation Strategies

### Build Pipeline Pattern

**Automated Image Creation**:

1. Base public image
2. Packer or Cloud Build
3. Apply configurations
4. Security hardening
5. Testing and validation
6. Promote to image family
7. Deprecate old version

**Benefits**:

- Consistent, repeatable process
- Version controlled configurations
- Automated testing
- Audit trail

### Golden Image Approach

**Single Source of Truth**:

- Create "perfect" VM
- Install and configure software
- Harden and document
- Create image from disk
- Distribute to teams

**Considerations**:

- Manual process initially
- Requires documentation
- Update process needed
- Drift over time

**Recommendation**: Automate with Cloud Build

## Cost Considerations

### Storage Costs

**Machine Images**:

- Charged for all disk data
- Multiple disks = higher cost
- Full copy each time
- Expensive for frequent backups

**Disk Images**:

- Single disk storage
- Full copy
- Long-term storage
- Moderate cost

**Snapshots**:

- Incremental storage
- First snapshot = full copy
- Subsequent = only changes
- Most cost-effective for frequent backups

### Optimization Strategies

**Hybrid Approach**:

- Machine images: Weekly/monthly
- Snapshots: Daily
- Balance cost vs recovery options

**Retention Policies**:

- Keep recent machine images (30 days)
- Archive older snapshots
- Delete obsolete images
- Regular cleanup

## Compliance and Security

### Image Hardening

**Security Baselines**:

- CIS benchmarks applied
- Unnecessary services disabled
- Security patches current
- Logging configured
- Security agents installed

**Compliance Requirements**:

- Approved software list
- Vulnerability scanning
- Configuration standards
- Audit logging

### Scanning and Validation

**Container Analysis**:

- Scan images for vulnerabilities
- Block deployment if critical issues
- Continuous monitoring
- Compliance reporting

**Testing**:

- Automated security tests
- Vulnerability assessments
- Penetration testing
- Regular updates

## Disaster Recovery

### Multi-Region Strategy

**Pattern**:

- Machine images in multi-regional storage
- Snapshots in multiple regions
- Regular DR testing
- Documented procedures

**Considerations**:

- Storage location affects cost
- Restore time varies by location
- Network egress charges
- Compliance requirements

### Recovery Time Objectives

**Machine Images**:

- RTO: 15-30 minutes
- Complete VM restore
- All configuration preserved
- Fastest recovery option

**Snapshots**:

- RTO: 30-60 minutes
- Disk restoration
- VM recreation required
- Configuration must be reapplied

## Instance Templates

### Relationship with Images

**Instance Template Contains**:

- Reference to image (family preferred)
- Machine type specification
- Network configuration
- Metadata and labels
- Startup scripts
- NOT the actual disk data

**Architecture Pattern**:

```
Disk Image (OS + Software)
    ↓
Image Family (versioning)
    ↓
Instance Template (configuration)
    ↓
Managed Instance Group (scaling)
```

### Versioning Strategy

**Pattern**:

- Update image in family
- Instance template references family
- MIG rolling update to new version
- Automated, consistent deployments

**Benefits**:

- Decoupling of image and configuration
- Easy rollbacks
- Gradual rollouts
- Version control

## Exam Focus Areas

### Decision Criteria

- Machine image vs disk image vs snapshot trade-offs
- When to use each type
- Cost comparison and optimization
- Storage location decisions

### Architecture Patterns

- Image pipeline automation
- Golden image management
- Cross-project sharing
- Image family versioning

### DR and Backup

- Hybrid backup strategies (images + snapshots)
- Multi-region DR with images
- RTO/RPO considerations
- Testing and validation

### Security

- Image hardening strategies
- Vulnerability scanning
- Compliance baselines
- Access control (IAM)

### Operations

- Image lifecycle management
- Deprecation strategies
- Instance template integration
- Automated updates
