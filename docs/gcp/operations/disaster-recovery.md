# Disaster Recovery

## Core Concepts

Disaster Recovery (DR) is the ability to recover IT systems and data after a disaster. Understanding RPO/RTO requirements drives DR strategy selection.

**Key Principle**: Plan for failure; the goal is not to prevent disasters, but to minimize impact.

## RPO and RTO

### Recovery Point Objective (RPO)

**Definition**: Maximum acceptable data loss (time between last backup and disaster)

**Examples**:

- RPO 24 hours: Lose up to 24 hours of data
- RPO 1 hour: Lose up to 1 hour of data
- RPO near-zero: Minimal to no data loss

**Determines**: Backup frequency

### Recovery Time Objective (RTO)

**Definition**: Maximum acceptable downtime (time to restore operations)

**Examples**:

- RTO 4 hours: System down max 4 hours
- RTO 1 hour: System down max 1 hour
- RTO minutes: Near-instant recovery

**Determines**: DR strategy and architecture

### RTO vs RPO Matrix

| Tier | RPO | RTO | Strategy | Cost |
|------|-----|-----|----------|------|
| Tier 0 (Critical) | Near-zero | < 1 hour | Active-active | Very High |
| Tier 1 (Important) | < 1 hour | < 4 hours | Hot standby | High |
| Tier 2 (Standard) | < 24 hours | < 24 hours | Warm standby | Medium |
| Tier 3 (Low priority) | < 7 days | < 48 hours | Backup/restore | Low |

## Disaster Recovery Strategies

### Backup and Restore (Cheapest, Slowest)

**Architecture**: Backup data, restore on disaster

**RPO**: Hours to days (backup frequency)
**RTO**: Hours (restoration time)
**Cost**: Lowest (storage only)

**Implementation**:

- Compute Engine: Snapshots, machine images
- Databases: Cloud SQL backups, manual dumps
- Storage: Cloud Storage versioning, replication

**Use when**: Cost-critical, acceptable downtime hours/days

### Pilot Light (Low Cost, Faster)

**Architecture**: Minimal infrastructure always running in DR region

**Core components**:

- Database replicas (standby)
- Pre-configured but minimal compute
- Network infrastructure ready

**RPO**: 1-4 hours (replication lag)
**RTO**: 1-2 hours (scale up DR environment)
**Cost**: Low (minimal running resources)

**Implementation**:

- Cloud SQL read replicas in DR region
- Load balancer pre-configured
- Startup scripts ready
- Scale compute on failover

**Use when**: Balance cost and recovery time

### Warm Standby (Medium Cost, Fast)

**Architecture**: Scaled-down version running in DR region

**Running**:

- Database active (replication)
- Reduced compute capacity
- Can handle reduced load immediately
- Scale up for full capacity

**RPO**: Minutes to 1 hour (near real-time replication)
**RTO**: 30-60 minutes (scale up)
**Cost**: Medium (running infrastructure at reduced capacity)

**Implementation**:

- Cloud SQL with HA and replicas
- GKE cluster with fewer nodes
- Cloud Run with min instances
- Regional persistent disks

**Use when**: Important applications, can tolerate brief reduced capacity

### Hot Standby / Active-Passive (High Cost, Very Fast)

**Architecture**: Full capacity in DR region, ready but not serving traffic

**Characteristics**:

- Everything running at full capacity
- Immediate failover
- No scale-up needed
- Single-digit minute RTO

**RPO**: Near-zero (synchronous or near-synchronous replication)
**RTO**: < 5 minutes (DNS/load balancer switch)
**Cost**: High (2x infrastructure, mostly idle in DR)

**Implementation**:

- Global load balancer with health checks
- Full compute capacity in both regions
- Database with synchronous replication
- Automatic failover configured

**Use when**: Mission-critical, minimal downtime acceptable

### Active-Active / Multi-Region (Highest Cost, Zero RTO)

**Architecture**: Full capacity in multiple regions, all serving traffic

**Characteristics**:

- No failover needed (both active)
- Zero RTO (automatic)
- Highest availability
- Most complex (data consistency)

**RPO**: Near-zero (multi-region replication)
**RTO**: No planned downtime (automatic)
**Cost**: Highest (2x+ infrastructure, all active)

**Implementation**:

- Global HTTP(S) Load Balancer
- Cloud Spanner (global database) or eventual consistency
- Multi-region Cloud Storage
- Cloud CDN for static content

**Use when**: Zero-tolerance for downtime, global applications

## Disaster Scenarios

### Scenario 1: Zone Failure

**Impact**: Resources in single zone unavailable

**Protection**:

- Regional persistent disks (automatic failover)
- Regional MIGs (distribute across zones)
- GKE regional clusters
- Multi-zone database (Cloud SQL HA)

**RTO**: Minutes (automatic)
**RPO**: Zero (synchronous replication within region)

### Scenario 2: Regional Disaster

**Impact**: Entire region unavailable

**Protection**:

- Multi-region deployment
- Cross-region snapshots/backups
- Global load balancer
- Multi-region database (Spanner) or cross-region replication

**RTO**: Depends on strategy (minutes to hours)
**RPO**: Depends on replication method

### Scenario 3: Data Corruption/Deletion

**Impact**: Data corrupted or accidentally deleted

**Protection**:

- Cloud Storage versioning
- Persistent disk snapshots
- Database backups and PITR (point-in-time recovery)
- Retention policies (prevent deletion)

**RTO**: Minutes to hours (restore time)
**RPO**: Backup frequency

### Scenario 4: Application Bug/Bad Deployment

**Impact**: Application malfunction from bad release

**Protection**:

- Blue-green deployments
- Canary deployments
- Rollback capability
- Immutable infrastructure

**RTO**: Minutes (rollback)
**RPO**: Zero (no data loss)

## DR for Specific Services

### Compute Engine

**Backup**:

- Machine images (full VM)
- Persistent disk snapshots (incremental)
- Custom images for OS

**RPO**: Snapshot frequency (hourly, daily)
**RTO**: 15-60 minutes (restore and boot)

**Cross-region DR**: Snapshots in multi-regional storage, restore in DR region

### Cloud SQL

**Backup**:

- Automated backups (daily)
- Point-in-time recovery (7-35 days)
- On-demand backups

**HA Configuration**:

- Regional HA (automatic failover within region)
- Cross-region read replicas
- External replica (on-premises)

**RPO**: 

- HA: Zero (synchronous)
- Read replicas: Seconds (near real-time)
- Backups: 24 hours

**RTO**:

- HA: < 1 minute (automatic)
- Cross-region failover: Minutes (manual promotion)
- Backup restore: 30-60 minutes

### GKE

**Backup**:

- Persistent volume snapshots
- Etcd backups (cluster state)
- Workload configuration (GitOps)

**HA Configuration**:

- Regional cluster (multi-zone control plane)
- Multi-cluster setup (cross-region)
- Config Sync (disaster recovery clusters)

**RPO**: Depends on replication strategy
**RTO**: Minutes (regional cluster), hours (multi-region)

### Cloud Storage

**Backup**:

- Object versioning
- Cross-region replication (Turbo Replication)
- Dual-region or multi-region storage

**RPO**:

- Standard: Eventual consistency (versioning)
- Turbo Replication: < 15 minutes

**RTO**: Immediate (automatic failover for multi/dual-region)

### BigQuery

**Backup**:

- Table snapshots (point-in-time)
- Export to Cloud Storage
- Cross-region dataset copy

**RPO**: Snapshot frequency
**RTO**: Minutes to hours (restore table)

**Multi-region**: BigQuery datasets in US/EU automatically multi-region

## Testing DR Plans

### Importance

**Untested DR plan = No DR plan**

**Common failures**:

- Backup restores never tested
- Failover process not documented
- Insufficient permissions in DR region
- Network connectivity issues
- Dependencies not identified

### Testing Types

**Tabletop Exercise** (Cheapest):

- Walk through DR procedure
- Identify gaps in documentation
- No actual failover

**Simulated Disaster** (Best):

- Actual failover to DR environment
- Test restoration procedures
- Measure RTO/RPO
- Identify issues before real disaster

**Chaos Engineering**:

- Intentional failures (zone, service)
- Verify automatic recovery
- Test resilience continuously

### Testing Frequency

- **Critical systems**: Quarterly
- **Important systems**: Semi-annually
- **Standard systems**: Annually

### DR Drill Checklist

1. Document current state
2. Initiate DR procedure
3. Time each step (measure RTO)
4. Verify data integrity (validate RPO)
5. Test application functionality
6. Document issues and gaps
7. Update DR plan
8. Conduct post-mortem

## DR Plan Documentation

### Essential Components

**Contact Information**:

- On-call rotation
- Escalation path
- Vendor contacts (Google support)

**System Inventory**:

- Critical services and dependencies
- Data classification
- RPO/RTO requirements per system

**Procedures**:

- Step-by-step recovery instructions
- Failover triggers (when to activate)
- Rollback procedures
- Communication plan

**Architecture Diagrams**:

- Normal operations
- DR configuration
- Network topology
- Data flow

### Runbooks

**Format**: Step-by-step instructions

**Include**:

- Prerequisites and permissions
- Commands to execute
- Expected output
- Verification steps
- Rollback procedure

**Accessibility**: Available offline (printed or local)

## Cost Optimization

### Right-Size DR Strategy

**Anti-pattern**: Same strategy for all systems

**Best practice**: Tier systems by criticality

**Example**:

- Tier 0 (Payment): Active-active (expensive)
- Tier 1 (Order mgmt): Hot standby
- Tier 2 (Reporting): Pilot light
- Tier 3 (Archives): Backup/restore

### Use Appropriate Storage Classes

**Backups**:

- Recent (< 30 days): Standard or Nearline
- Historical (30-90 days): Coldline
- Compliance (> 365 days): Archive

**Savings**: 50-80% on long-term backup storage

### Minimize Idle Resources

**Pilot light**: Minimal resources, scale on disaster

**Scheduled scaling**: Non-24/7 workloads scale down off-hours

### Leverage Managed Services

**Example**: Cloud SQL HA vs self-managed replication

**Benefit**: Lower operational cost, better reliability

## Compliance Considerations

### Retention Requirements

**Regulatory**:

- HIPAA: 6 years
- SOX: 7 years
- PCI-DSS: 1 year

**Implementation**: Backup retention policies, object lifecycle

### Geographic Requirements

**GDPR**: EU data must stay in EU

**Implementation**:

- Organization Policy (location restrictions)
- Snapshots in appropriate regions
- Cross-region DR within compliant regions

### Encryption

**At rest**: Automatic (CMEK for compliance)

**In transit**: Automatic for cross-region replication

## Monitoring and Alerting

### Health Checks

**Purpose**: Detect failures automatically

**Implementation**:

- Load balancer health checks
- Uptime checks (Cloud Monitoring)
- Application health endpoints

### Failover Automation

**Trigger**: Health check failures

**Action**:

- Automatic (load balancer, Cloud SQL HA)
- Semi-automatic (alert + manual approval)
- Manual (runbook execution)

### Backup Monitoring

**Metrics**:

- Backup success/failure rate
- Time since last successful backup
- Backup size trends

**Alerts**:

- Backup failure
- Backup age > RPO threshold
- Restoration test failure

## Best Practices

### 3-2-1 Backup Rule

**3 copies**: Original + 2 backups
**2 media types**: Disk snapshots + object storage
**1 offsite**: Different region

**GCP Implementation**:

- Original: Persistent disks
- Backup 1: Regional snapshots
- Backup 2: Multi-regional Cloud Storage

### Immutable Backups

**Purpose**: Prevent ransomware/malicious deletion

**Implementation**:

- Cloud Storage retention locks
- Separate backup project (no delete permissions)
- Object holds

### Automate Everything

**Automate**:

- Backup creation (scheduled)
- Retention enforcement (lifecycle)
- Testing (scheduled DR drills)
- Monitoring (automatic alerts)

**Why**: Human error is common failure point

### Document and Communicate

**Documentation**: Updated with each change

**Communication**: Team aware of DR procedures

**Training**: Regular DR training for on-call

## Exam Focus

### RPO vs RTO

- Definition and difference
- How they drive strategy selection
- Cost implications

### DR Strategies

- Backup/restore, pilot light, warm standby, hot standby, active-active
- Cost vs RTO/RPO trade-offs
- When to use each

### Service-Specific DR

- Compute Engine (snapshots, machine images)
- Cloud SQL (HA, read replicas, backups)
- Cloud Storage (versioning, replication)
- GKE (regional clusters, multi-cluster)

### Testing

- Importance of testing
- Testing frequency
- DR drill procedures

### Architecture Patterns

- Multi-zone (zone failure)
- Multi-region (regional disaster)
- Global (active-active)
- Tiered approach (different strategies per tier)

### Cost Optimization

- Tier systems by criticality
- Appropriate storage classes
- Minimize idle resources in DR
- Leverage managed services
