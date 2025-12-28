# Disk Snapshots

## Core Concepts

Snapshots provide incremental, point-in-time backups of persistent disks. Understanding snapshot architecture is essential for designing cost-effective backup and recovery strategies.

**Key Principle**: Snapshots are incremental after the first full copy, making them ideal for frequent backups.

## Incremental Architecture

### How Snapshots Work

**First Snapshot**:

- Full copy of all disk data
- Stored in Cloud Storage
- Time proportional to disk size
- Baseline for future snapshots

**Subsequent Snapshots**:

- Only changed blocks since previous snapshot
- Significantly faster creation
- Space-efficient storage
- Automatic deduplication

**Chain Management**:

- Google maintains snapshot chain automatically
- Deleting middle snapshot is safe
- Required blocks preserved
- No user management needed

### Storage Efficiency Example

**Scenario**: 1 TB disk, 5% daily change, 30-day retention

**Traditional Full Backups**:

- 30 copies × 1000 GB = 30,000 GB

**Incremental Snapshots**:

- Day 1: 1000 GB (full)
- Days 2-30: 50 GB each
- Total: 1000 + (29 × 50) = 2,450 GB
- **92% storage savings**

## Snapshot Storage Locations

### Regional Storage

**Characteristics**:

- Stored in single region
- Same cost as multi-regional
- Faster restore (same region)
- Region-specific availability

**Use Cases**:

- Fast restore requirements
- Region-bound compliance
- Cost optimization (same-region egress free)
- Non-critical data

**Limitation**: Single region failure risk

### Multi-Regional Storage

**Characteristics**:

- Stored across multiple regions
- Geographic redundancy
- Same cost as regional
- Protected from regional failure

**Use Cases**:

- Disaster recovery
- Cross-region restore capability
- Critical data protection
- Compliance requirements

**Trade-off**: Slightly slower initial creation, cross-region restore incurs egress costs

### Architecture Decision Matrix

| Data Criticality | Restore Frequency | Storage Location | Rationale |
|-----------------|-------------------|------------------|-----------|
| Critical | Daily | Multi-regional | DR protection |
| Important | Weekly | Multi-regional | Balance cost/protection |
| Normal | Daily | Regional | Cost optimization |
| Development | On-demand | Regional | Lowest cost |

## Scheduled Snapshots

### Snapshot Schedules

**Benefits**:

- Automated backup lifecycle
- Consistent backup timing
- Retention policy enforcement
- Reduced operational overhead
- Guaranteed backup frequency

**Schedule Types**:

- **Hourly**: Every N hours (critical data, 4-hour RPO)
- **Daily**: Specific time (standard backups, 24-hour RPO)
- **Weekly**: Specific day and time (less critical, 7-day RPO)

**Retention Policies**:

- Automatic deletion of old snapshots
- Maximum retention: ~7.5 years
- Compliance-driven retention
- Cost control

### Design Patterns

**Tiered Backup Strategy**:

- Hourly snapshots: Retain 24 hours (critical systems)
- Daily snapshots: Retain 7 days (standard)
- Weekly snapshots: Retain 4 weeks (long-term)
- Monthly snapshots: Retain 12 months (compliance)

**Pattern**: Multiple schedules on same disk for different retention tiers

## Snapshot Best Practices

### Application-Consistent Snapshots

**Problem**: Snapshot at disk level, may capture inconsistent state

**Solutions**:

- **Quiesce filesystem**: Flush pending writes
- **Database checkpoint**: Ensure consistent state
- **Application-level backup**: Use native tools first
- **--guest-flush flag**: Windows VSS integration

**Architecture Consideration**: Snapshots are crash-consistent by default; application consistency requires coordination

### Backup Windows

**Considerations**:

- Schedule during low-usage periods
- First snapshot has performance impact
- Subsequent snapshots minimal impact
- Background operation

**Pattern**: 2-4 AM local time for daily snapshots (minimize business impact)

## Recovery Strategies

### Point-in-Time Recovery

**Capabilities**:

- Restore to any snapshot point
- Multiple recovery points (based on schedule)
- Granular recovery options
- Test restores without affecting production

**RTO Considerations**:

- Disk creation from snapshot: 10-30 minutes
- VM recreation: 5-15 minutes
- Application startup: Varies
- **Total RTO: 30-60 minutes typically**

### Partial Recovery

**Options**:

- Restore single disk (not entire VM)
- Attach restored disk as secondary
- Copy needed files
- Detach and delete disk

**Use Case**: Accidental deletion, specific file recovery

## Cost Optimization

### Snapshot Cost Structure

**Storage Pricing**:

- ~$0.026/GB/month (regional and multi-regional)
- Incremental charges (only changed data)
- No retrieval fees
- No operation charges

**Cost Factors**:

- Frequency of snapshots (more = more changed data)
- Change rate (higher = more storage)
- Retention period (longer = more total storage)
- Number of disks

### Optimization Strategies

**Retention Policies**:

- Delete old snapshots automatically
- Balance recovery needs vs cost
- Different policies for different data tiers
- Regular review and adjustment

**Snapshot Consolidation**:

- Fewer snapshots = lower cost
- Balance with RPO requirements
- Use snapshot schedules for consistency

**Right-Sizing Frequency**:

- Critical data: Frequent snapshots justified
- Non-critical data: Less frequent acceptable
- Test environments: Manual/infrequent

## Disaster Recovery Architecture

### Cross-Region DR

**Pattern**:

1. Snapshots in multi-regional storage
2. DR plan documents restore procedure
3. Network and firewall rules pre-configured in DR region
4. Regular DR testing (quarterly)
5. Runbooks and automation

**RTO Targets**:

- Tier 1 (Critical): RTO 1 hour
  - Hourly snapshots, multi-regional
  - Automated restoration scripts
  - Pre-warmed network configuration
  
- Tier 2 (Important): RTO 4 hours
  - Daily snapshots, multi-regional
  - Documented procedures
  
- Tier 3 (Standard): RTO 24 hours
  - Daily snapshots, regional
  - Manual restoration acceptable

### RPO Considerations

**Snapshot Frequency = RPO**:

- Hourly snapshots: 1-hour RPO (max 1 hour data loss)
- Daily snapshots: 24-hour RPO
- Weekly snapshots: 7-day RPO

**Architecture Decision**: Balance frequency cost vs acceptable data loss

## Snapshot Limits

### Operational Limits

| Limit | Value | Impact |
|-------|-------|--------|
| Snapshots per disk | Unlimited | No practical limit |
| Snapshots per project | 10,000 | Request increase if needed |
| Snapshot creation rate | 1 per 10 min per disk | Affects backup frequency |
| Snapshot size | 64 TB | Matches max disk size |

**Architecture Implication**: Large environments may need multiple projects for snapshot quotas

## Comparison: Snapshots vs Machine Images

### When to Use Snapshots

**Advantages**:

- Incremental (cost-effective)
- Frequent backups (daily/hourly)
- Per-disk granularity
- Fast creation (after first)
- Low storage cost

**Use Cases**:

- Regular data protection
- Point-in-time recovery
- Database backups
- Frequent backup requirements
- Cost-sensitive environments

### When to Use Machine Images

**Advantages**:

- Complete VM configuration
- All disks in one operation
- Fast full restoration
- VM migration

**Use Cases**:

- Weekly/monthly full backups
- VM cloning and migration
- Disaster recovery base
- Configuration preservation

**Architecture Pattern**: Use both - snapshots for frequent backups, machine images for complete system backups

## Security Considerations

### Encryption

**Default Encryption**:

- All snapshots encrypted at rest
- Google-managed keys
- Transparent operation
- No configuration needed

**Customer-Managed Keys (CMEK)**:

- Control key lifecycle
- Regulatory compliance
- Key rotation policies
- Cloud KMS integration

**Trade-offs**:

- CMEK: More control, more complexity
- Google-managed: Simpler, less control
- Performance impact minimal

### Access Control

**IAM Roles**:

- `roles/compute.storageAdmin`: Create/delete snapshots
- `roles/compute.viewer`: View snapshots
- Principle of least privilege
- Separate snapshot admin from VM admin

**Snapshot Sharing**:

- Cross-project via IAM
- Service account access
- Controlled sharing
- Audit access

## Compliance and Retention

### Regulatory Requirements

**Common Requirements**:

- **HIPAA**: 6-year retention, encryption, audit logs
- **SOX**: 7-year retention, access controls
- **GDPR**: Data location restrictions, deletion capability
- **PCI-DSS**: Encryption, access controls, retention

**Implementation**:

- Snapshot schedules with appropriate retention
- Storage location selection (region/multi-region)
- CMEK for enhanced security
- Audit logging enabled

### Retention Strategy

**Pattern**:

- **Short-term**: Daily snapshots, 7-30 day retention (operational recovery)
- **Long-term**: Monthly snapshots, multi-year retention (compliance)
- **Archive**: Consider exporting to Cloud Storage for very long-term

## Monitoring and Alerting

### Key Metrics

**Monitor**:

- Snapshot creation success/failure rate
- Time to create snapshots
- Storage costs trending
- Age of oldest snapshot
- Recovery testing results

**Alerts**:

- Snapshot creation failures
- Storage quota approaching limits
- Unexpected cost increases
- Missing scheduled snapshots
- Compliance violations (retention)

## Exam Focus Areas

### Design Decisions

- Snapshot vs machine image trade-offs
- Frequency and retention policies
- Storage location selection (regional vs multi-regional)
- Cost optimization strategies

### Backup Architecture

- Tiered backup strategies
- RPO/RTO planning
- Application-consistent backup methods
- Cross-region DR design

### Cost Management

- Incremental snapshot cost model
- Retention policy optimization
- Storage location cost impact
- Snapshot cleanup strategies

### Operations

- Scheduled snapshots configuration
- Automated lifecycle management
- Monitoring and alerting
- Testing and validation

### Compliance

- Retention requirements
- Encryption options (default vs CMEK)
- Access control design
- Audit logging
