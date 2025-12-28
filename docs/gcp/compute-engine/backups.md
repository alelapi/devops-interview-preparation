# Backup and Disaster Recovery Strategy

## Core Concepts

A comprehensive backup and disaster recovery strategy minimizes data loss (RPO) and downtime (RTO) through a combination of technologies and procedures. Understanding trade-offs between different approaches is critical for architecture decisions.

**Key Principle**: Design for failure; assume resources will fail and plan accordingly.

## Recovery Objectives

### RPO (Recovery Point Objective)

**Definition**: Maximum acceptable data loss (time between last backup and disaster)

**Common Targets**:

- **Tier 1 (Critical)**: RPO < 1 hour
  - Real-time replication or hourly snapshots
  - Regional persistent disks (RPO near-zero)
  - Highest cost

- **Tier 2 (Important)**: RPO < 4 hours
  - 4-hourly snapshots
  - Balanced cost/protection

- **Tier 3 (Normal)**: RPO < 24 hours
  - Daily snapshots
  - Standard for most workloads

- **Tier 4 (Low Priority)**: RPO < 7 days
  - Weekly backups
  - Cost-optimized

**Architecture Decision**: RPO directly impacts backup frequency and cost

### RTO (Recovery Time Objective)

**Definition**: Maximum acceptable downtime (time to restore and resume operations)

**Common Targets**:

- **Tier 1 (Critical)**: RTO < 1 hour
  - Regional persistent disks with automatic failover
  - Active-active architecture
  - Highest cost

- **Tier 2 (Important)**: RTO < 4 hours
  - Hot standby in DR region
  - Automated restore procedures

- **Tier 3 (Normal)**: RTO < 24 hours
  - Warm standby or documented procedures
  - Standard for most workloads

- **Tier 4 (Low Priority)**: RTO < 48 hours
  - Cold standby or manual procedures
  - Cost-optimized

**Architecture Decision**: RTO determines DR strategy (active-active, hot/warm/cold standby)

## Backup Strategies Comparison

### Strategy 1: Snapshot-Only

**Architecture**:

- Daily disk snapshots
- Automated snapshot schedules
- Retention based on requirements
- Multi-regional storage for DR

**Characteristics**:

- RPO: 24 hours (daily snapshots)
- RTO: 30-60 minutes (disk restore + VM recreation)
- Cost: Low (incremental storage)
- Complexity: Low

**Appropriate for**:

- Standard workloads (Tier 3)
- Cost-sensitive environments
- Predictable, documented recovery procedures
- Acceptable 24-hour data loss

**Limitations**:

- VM configuration not captured
- Manual recreation required
- 24-hour RPO minimum (daily snapshots)

### Strategy 2: Machine Image-Only

**Architecture**:

- Weekly complete VM backups
- All disks + configuration
- Multi-regional storage

**Characteristics**:

- RPO: 7 days (weekly images)
- RTO: 15-30 minutes (fast VM restore)
- Cost: High (full copies)
- Complexity: Low

**Appropriate for**:

- Infrequent changes
- Configuration preservation critical
- Fast recovery more important than RPO
- Pre-maintenance backups

**Limitations**:

- Expensive for frequent backups
- 7-day RPO not acceptable for most production
- Large storage footprint

### Strategy 3: Hybrid Approach (Recommended)

**Architecture**:

- Daily snapshots (data protection, 24-hour RPO)
- Weekly machine images (full VM backup)
- Combines best of both

**Characteristics**:

- RPO: 24 hours (daily snapshots)
- RTO: 30-45 minutes
- Cost: Medium (balanced)
- Complexity: Medium

**Appropriate for**:

- Most production workloads
- Balance of cost and protection
- Flexible recovery options
- Standard enterprise practice

**Pattern**: 

- Snapshots for operational recovery (fast, frequent)
- Machine images for disaster recovery (complete, infrequent)

### Strategy 4: Regional Persistent Disks

**Architecture**:

- Synchronous replication across zones
- Automatic failover
- No backup needed for zone failures

**Characteristics**:

- RPO: Near-zero (synchronous replication)
- RTO: Minutes (automatic failover with MIG)
- Cost: High (2x disk cost)
- Complexity: Medium

**Appropriate for**:

- Critical databases (Tier 1)
- Zero data loss requirement
- Zone failure protection
- HA applications

**Limitations**:

- 2x storage cost
- Regional only (not multi-region)
- Still need snapshots for data corruption/deletion
- Slight latency increase (cross-zone sync)

**Important**: Regional disks protect against zone failure, not against data corruption or deletion - still need snapshots

## Disaster Recovery Patterns

### DR Strategy 1: Backup and Restore (Lowest Cost)

**Architecture**:

- Production in primary region
- Snapshots/images in multi-regional storage
- Restore on-demand in DR region
- Network pre-configured but inactive

**Characteristics**:

- RTO: 2-4 hours
- RPO: 24 hours (daily snapshots)
- Cost: Lowest (pay for storage only)
- Complexity: Low

**Appropriate for**:

- Tier 3-4 workloads
- Cost-sensitive environments
- Acceptable multi-hour outage
- Regional disaster only scenario

### DR Strategy 2: Pilot Light

**Architecture**:

- Minimal infrastructure in DR region (always running)
- Core components ready (networking, databases at small scale)
- Scale up on failover
- Regular snapshots for data sync

**Characteristics**:

- RTO: 1-2 hours
- RPO: 4-24 hours
- Cost: Low-Medium
- Complexity: Medium

**Appropriate for**:

- Tier 2 workloads
- Balance cost and recovery time
- Predictable scale-up process
- Partial availability acceptable during failover

### DR Strategy 3: Warm Standby

**Architecture**:

- Reduced capacity in DR region (always running)
- Database replication active
- Can handle reduced load immediately
- Scale up for full capacity

**Characteristics**:

- RTO: 30-60 minutes
- RPO: 1-4 hours (replication lag)
- Cost: Medium-High
- Complexity: Medium-High

**Appropriate for**:

- Tier 1-2 workloads
- Need fast recovery
- Can operate at reduced capacity
- Critical business applications

### DR Strategy 4: Hot Standby / Active-Active

**Architecture**:

- Full capacity in multiple regions
- Active-active or active-passive
- Real-time data replication
- Global load balancing

**Characteristics**:

- RTO: < 5 minutes (automatic failover)
- RPO: Near-zero (real-time replication)
- Cost: Highest (2x infrastructure)
- Complexity: Highest

**Appropriate for**:

- Tier 1 workloads only
- No tolerance for downtime
- Global services
- High availability requirement

**Considerations**:

- Data consistency challenges
- Application must handle multi-region
- Complex failover procedures
- Significant cost

## 3-2-1 Backup Rule

### Rule Definition

**3 Copies**: Original data + 2 backups
**2 Media Types**: Different storage types
**1 Off-Site**: Geographic separation

### GCP Implementation

**3 Copies**:

1. Original: Data on persistent disk
2. Backup 1: Regional snapshots (same region)
3. Backup 2: Multi-regional machine images

**2 Media Types**:

1. Persistent disk (block storage)
2. Snapshots in Cloud Storage (object storage)

**1 Off-Site**:

- Multi-regional snapshot storage
- Cross-region machine images
- Geographic redundancy

**Architecture Pattern**: Standard for critical data (Tier 1-2)

## Backup Testing

### Regular DR Drills

**Frequency**:

- Critical systems (Tier 1): Monthly
- Important systems (Tier 2): Quarterly
- Normal systems (Tier 3): Semi-annually

**Test Scope**:

- Complete restoration procedure
- Network connectivity verification
- Application functionality testing
- Performance validation
- Documentation update

**Automation**:

- Scripted restoration
- Automated testing
- Monitoring and alerting
- Regular execution (CI/CD)

### Validation Strategy

**Verification Points**:

- Snapshot creation success
- Backup completeness
- Restoration time (actual RTO)
- Data integrity
- Application functionality

**Documentation**:

- Runbooks for each scenario
- Contact information
- Escalation procedures
- Last test date and results

## Cost Optimization

### Tiered Backup Strategy

**Pattern**:

| Tier | Snapshot Frequency | Retention | Machine Image | Total Cost |
|------|-------------------|-----------|---------------|------------|
| Tier 1 | Hourly | 7 days | Weekly, 30 days | Highest |
| Tier 2 | Daily | 30 days | Weekly, 30 days | High |
| Tier 3 | Daily | 7 days | Monthly, 90 days | Medium |
| Tier 4 | Weekly | 30 days | Quarterly, 1 year | Low |

**Architecture Decision**: Match backup cost to data criticality

### Storage Location Optimization

**Regional Snapshots**:

- Use for non-critical data
- Same-region restore (free egress)
- Lower recovery time
- Single-region risk

**Multi-Regional Snapshots**:

- Use for critical data
- DR capability
- Cross-region restore (egress charges)
- Geographic redundancy

## Disaster Scenarios and Recovery

### Scenario 1: Data Corruption

**Recovery Approach**:

- Identify corruption time
- Restore from nearest snapshot before corruption
- Create new disk from snapshot
- Attach to VM or create new VM
- Verify data integrity

**RTO**: 30-60 minutes
**Best Prevention**: Frequent snapshots, application-level logging

### Scenario 2: Accidental Deletion

**Recovery Approach**:

- Restore from machine image (complete VM)
- Or restore disk from snapshot + recreate VM
- Update external dependencies (DNS, load balancer)
- Verify functionality

**RTO**: 30-60 minutes
**Best Prevention**: Deletion protection, IAM controls, change management

### Scenario 3: Zone Failure

**Recovery Approach**:

- Regional persistent disks: Automatic failover (minutes)
- Zonal disks: Restore from snapshot in different zone
- MIG autohealing creates new instances
- Load balancer reroutes traffic

**RTO**: 5-30 minutes (depending on architecture)
**Best Prevention**: Regional MIG, regional persistent disks, multi-zone architecture

### Scenario 4: Regional Disaster

**Recovery Approach**:

1. Declare disaster
2. Restore snapshots/images in DR region
3. Update global load balancer
4. Update DNS (if needed)
5. Scale infrastructure
6. Verify functionality
7. Monitor and adjust

**RTO**: 1-4 hours (depending on DR strategy)
**Best Prevention**: Multi-region architecture, regular DR testing

## Compliance and Governance

### Retention Requirements

**Regulatory Standards**:

- **HIPAA**: 6 years minimum
- **SOX**: 7 years minimum
- **GDPR**: As per data processing agreement
- **PCI-DSS**: 3 months minimum, 1 year recommended

**Implementation**:

- Snapshot schedules with appropriate retention
- Separate schedules for compliance backups
- Automated lifecycle management
- Regular audit

### Audit and Monitoring

**Audit Logs**:

- Snapshot creation/deletion
- Machine image operations
- Recovery procedures
- Access to backups

**Monitoring**:

- Backup success/failure rates
- Time to create backups
- Storage costs
- Age of last successful backup
- RTO/RPO compliance

## Architecture Decision Framework

### Questions to Answer

1. **What is acceptable data loss?** (Determines RPO, backup frequency)
2. **What is acceptable downtime?** (Determines RTO, DR strategy)
3. **What is the budget?** (Constrains solutions)
4. **What are compliance requirements?** (Minimum retention, encryption)
5. **What is recovery complexity tolerance?** (Affects automation needs)
6. **Is multi-region DR needed?** (Geographic redundancy)
7. **What is change frequency?** (Affects snapshot efficiency)

### Decision Matrix

| RPO Requirement | RTO Requirement | Recommended Strategy | Cost Level |
|-----------------|-----------------|---------------------|------------|
| < 1 hour | < 1 hour | Regional disks + Active-active | Very High |
| < 4 hours | < 1 hour | Hourly snapshots + Hot standby | High |
| < 24 hours | < 4 hours | Daily snapshots + Warm standby | Medium-High |
| < 24 hours | < 24 hours | Daily snapshots + Pilot light | Medium |
| < 7 days | < 48 hours | Weekly images + Backup/restore | Low |

## Exam Focus Areas

### Strategy Selection

- RPO/RTO requirements and implications
- Backup strategy trade-offs (cost, complexity, protection)
- DR pattern selection criteria
- Multi-region considerations

### Cost Optimization

- Tiered backup strategies
- Snapshot vs machine image economics
- Storage location decisions
- Retention policy optimization

### Architecture Patterns

- 3-2-1 backup rule implementation
- Hybrid backup approaches
- Regional disk HA patterns
- Multi-region DR architectures

### Compliance

- Retention requirements
- Encryption and security
- Audit logging
- Testing requirements

### Operations

- Automation strategies
- Testing procedures
- Monitoring and alerting
- Documentation requirements
