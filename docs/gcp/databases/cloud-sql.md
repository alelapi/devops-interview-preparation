# Cloud SQL

## Core Concepts

Cloud SQL is a fully managed relational database service for MySQL, PostgreSQL, and SQL Server. Managed backups, replication, failover, and scaling.

**Key Principle**: Use for traditional relational workloads; Google manages infrastructure, you manage schema and queries.

## Cloud SQL Engines

| Engine | Versions | Use Case | Max Size |
|--------|----------|----------|----------|
| **MySQL** | 5.6, 5.7, 8.0 | Web apps, general purpose | 64 TB |
| **PostgreSQL** | 9.6-16 | Advanced SQL, compliance | 64 TB |
| **SQL Server** | 2017, 2019, 2022 | Windows apps, legacy | 64 TB |

## 1st Gen vs 2nd Gen

### 1st Generation (MySQL 5.5/5.6 only)

**Status**: Deprecated, legacy only

**Limitations**:

- Max 500 GB storage
- No automatic storage increase
- Limited performance
- No private IP

**Migration required**: Google recommends migrating to 2nd gen

### 2nd Generation (Current)

**All new instances**: MySQL 5.7/8.0, PostgreSQL, SQL Server

**Features**:

- Up to 64 TB storage
- Automatic storage increase
- Better performance (up to 7x faster)
- Private IP support
- Point-in-time recovery
- High availability configuration

**Recommendation**: Always use 2nd gen for new deployments

## When to Use Cloud SQL

### ✅ Use When

- Relational data with ACID requirements
- Existing MySQL/PostgreSQL/SQL Server app
- Structured data with relationships
- Transactional workloads (OLTP)
- Need SQL features (JOINs, transactions, constraints)
- Migrating from on-premises databases

### ❌ Don't Use When

- Global scale needed → Cloud Spanner
- Analytics workload → BigQuery
- NoSQL better fit → Firestore, Bigtable
- Serverless preferred → Firestore
- Extremely high throughput → Bigtable

## Machine Types

### Shared-Core

**Types**: db-f1-micro (0.6 GB), db-g1-small (1.7 GB)

**Use for**: Development, testing, very light workloads

**Limitations**: Burstable CPU, not for production

### Dedicated-Core

**Types**: Standard (up to 96 vCPUs, 624 GB RAM)

**Use for**: Production workloads

**High-memory**: More RAM per vCPU (db-custom-*)

**Pattern**: Start small, scale up based on monitoring

## Storage

### Types

**SSD** (default, recommended):

- Better performance
- $0.17/GB/month

**HDD** (legacy):

- Slower, cheaper
- $0.09/GB/month
- Not recommended

### Automatic Storage Increase

**Enabled by default**: Automatically grows when approaching capacity

**Thresholds**: Increases when <10% free or <6 GB free

**Benefits**: Prevents downtime from full disk

**Cost**: Pay for actual usage

**Recommendation**: Always enable for production

## High Availability (HA)

### Regional HA Configuration

**Architecture**:

```
Primary instance (zone A)
  ↓ Synchronous replication
Standby instance (zone B, same region)
```

**Automatic failover**: Minutes (typically <60 seconds)

**RPO**: Zero (synchronous replication)

**RTO**: < 1 minute (automatic)

**Cost**: 2x instance cost + network

### How It Works

**Normal operation**: Primary handles all traffic

**Failure detection**: Heartbeat monitoring

**Failover**: Standby promoted to primary automatically

**Reconnection**: Applications reconnect to same IP (unchanged)

### Configuration

**Enable HA**: At creation or after (requires restart)

**Important**: Must use private IP or public IP with authorized networks

**SLA**: 99.95% uptime with HA enabled

### When to Use HA

✅ **Production databases requiring high availability**

❌ **Development/test** (save cost, use snapshots for recovery)

**Trade-off**: 2x cost for better availability (RPO zero, RTO <1 min)

## Backups

### Automated Backups

**Frequency**: Daily, during maintenance window

**Retention**: 7-365 days (default 7)

**Type**: Full backup + transaction logs

**Location**: Same region or custom location

**Cost**: $0.08/GB/month (cheaper than disk)

**Important**: Enabled by default, always enable for production

### On-Demand Backups

**Manual**: Create anytime

**Retention**: Until manually deleted (no automatic expiration)

**Use for**: Before major changes, migrations

**Cost**: Same as automated ($0.08/GB/month)

### Point-in-Time Recovery (PITR)

**Requires**: Automated backups + binary logging enabled

**Capability**: Restore to any point in time (within retention period)

**Granularity**: Down to the second

**Use case**: Recover from accidental DELETE/UPDATE

**Example**: Accidentally deleted data at 2 PM, restore to 1:59 PM

### Backup Best Practices

- Enable automated backups (production)
- 30-day retention minimum (compliance may require more)
- Test restoration quarterly
- Cross-region backups for DR
- On-demand backup before schema changes

## Disaster Recovery

### Cross-Region DR Strategies

**Backup and Restore**:

- Export backups to Cloud Storage (multi-regional)
- Restore in DR region on disaster
- RPO: 24 hours (daily backup)
- RTO: 1-2 hours (restore time)
- Cost: Lowest (backup storage only)

**Read Replica in DR Region**:

- Async replication to DR region
- Promote replica to standalone on disaster
- RPO: Minutes (replication lag)
- RTO: < 30 minutes (manual promotion)
- Cost: Medium (replica instance cost)

**Managed Cross-Region Replica**:

- Automatic async replication
- Manual failover
- Easier management than manual replica

### Failover Process

**Regional HA** (automatic):

1. Primary fails
2. Standby promoted (seconds)
3. Applications reconnect automatically

**Cross-Region** (manual):

1. Declare disaster
2. Promote read replica to standalone
3. Update DNS/connection strings
4. Point applications to new region

### DR Testing

**Important**: Test DR procedures regularly

**Process**:

1. Create read replica in DR region
2. Simulate failover (promote replica)
3. Verify application connectivity
4. Measure actual RTO
5. Document gaps and update procedures

## Read Replicas

### Purpose

**Read scaling**: Offload read queries from primary

**Analytics**: Run heavy queries without impacting production

**DR**: Promote to standalone on disaster

### Architecture

```
Primary (read/write)
  ↓ Async replication
Read Replica 1 (read-only)
Read Replica 2 (read-only)
```

**Replication**: Asynchronous (eventual consistency)

**Lag**: Typically seconds, monitor replication lag

### Configuration

**Location**: Same region, different region, or external (on-premises)

**Max replicas**: 10 per primary

**Cascading**: Replica of replica (reduce load on primary)

**Failover**: Manual promotion to standalone instance

### Use Cases

**Read scaling**: Distribute read traffic across replicas

**Reporting**: Analytics on replica (not primary)

**DR**: Cross-region replica for disaster recovery

**Migration**: Replicate to new region, then migrate

## Scalability

### Vertical Scaling

**Machine type**: Upgrade vCPUs and memory

**Process**: Requires restart (brief downtime)

**Pattern**: Start small, scale up based on CPU/memory metrics

**Downtime**: Seconds to minutes (failover to HA standby if configured)

### Storage Scaling

**Automatic**: Storage increases automatically (no downtime)

**Manual**: Can increase anytime (no downtime)

**Cannot decrease**: Storage size cannot shrink

**Max**: 64 TB per instance

### Connection Scaling

**Max connections**: Based on machine type

**Formula**: ~4 connections per GB of RAM

**Pooling**: Use connection pooling in application (PgBouncer, ProxySQL)

**Cloud SQL Proxy**: Recommended for secure connections

### Horizontal Scaling (Read Replicas)

**Read traffic**: Distribute across replicas

**Write traffic**: Only to primary (cannot scale writes this way)

**Sharding**: Application-level for write scaling

**Alternative**: Cloud Spanner for horizontal write scaling

## Maintenance

### Maintenance Windows

**Purpose**: Apply patches, updates

**Frequency**: Weekly, monthly, or as needed

**Duration**: Minutes (with HA, seamless failover)

**Configuration**: Choose day and hour

**Deny maintenance**: Defer up to 365 days (not recommended)

### Updates

**Minor versions**: Automatic (during maintenance window)

**Major versions**: Manual (e.g., MySQL 5.7 → 8.0)

**Downtime**: With HA, minimal during maintenance

## Connectivity

### Public IP

**Use for**: Development, external access

**Security**: Authorized networks (IP allowlisting)

**Cost**: No additional charge

**Limitation**: Exposed to internet

### Private IP

**Recommended for production**:

- VPC peering-based connection
- No public internet
- Better security
- VPC resources access directly

**Requires**: VPC with allocated IP range

### Cloud SQL Proxy

**Purpose**: Secure connection without whitelisting IPs

**Benefits**:

- Automatic encryption
- IAM-based authentication
- No IP management

**Use for**: Applications in GCP, development from workstation

## Monitoring and Performance

### Key Metrics

**Database**:

- CPU utilization
- Memory utilization
- Disk I/O (IOPS, throughput)
- Connection count
- Replication lag (replicas)

**Application**:

- Query performance
- Slow query log
- Transaction rates

### Query Insights

**Purpose**: Identify slow queries

**Features**:

- Top queries by execution time
- Query execution plan
- Historical performance

**Use for**: Performance optimization, index tuning

### Recommendations

**Automatic**: Google suggests improvements

**Types**:

- Machine type sizing
- Index creation
- Storage optimization

## Cost Optimization

### Instance Right-Sizing

**Start small**: db-n1-standard-1 for most apps

**Scale up**: Based on actual usage, not estimates

**Recommendations**: Use Google's sizing recommendations

### HA Trade-off

**Cost**: 2x instance cost

**Decision**: Production (enable), dev/test (disable)

### Storage

**Auto-increase**: Prevents over-provisioning

**Backups**: Cheaper than live storage ($0.08 vs $0.17/GB)

**Cleanup**: Delete old manual backups

### Read Replicas

**Cost**: Full instance cost per replica

**Use sparingly**: Only for actual read scaling needs

**Alternative**: Cache layer (Memorystore) for reads

## Migration

### Database Migration Service

**Purpose**: Minimal-downtime migration to Cloud SQL

**Supported sources**:

- On-premises MySQL, PostgreSQL
- AWS RDS
- Cloud SQL to Cloud SQL

**Process**:

1. Create migration job
2. Full data copy
3. Continuous replication
4. Promote when ready (cutover)

**Downtime**: Minutes (cutover only)

### Manual Migration

**Export/Import**:

- mysqldump, pg_dump
- Import to Cloud SQL
- Downtime = export + import time

**Replication**:

- Set up replication from source
- Switch over when caught up
- Minimal downtime

## Security

### Encryption

**At rest**: Automatic (Google-managed or CMEK)

**In transit**: TLS enforced (can require SSL)

### IAM Database Authentication

**MySQL, PostgreSQL**: Use IAM users instead of passwords

**Benefits**:

- Centralized identity
- No password management
- Automatic credential rotation

### Best Practices

- Use private IP (production)
- Cloud SQL Proxy for secure access
- Least privilege database users
- Enable SSL/TLS enforcement
- Audit logging (must enable)
- No default users in production

## Exam Focus

### Core Concepts

- Fully managed MySQL, PostgreSQL, SQL Server
- 2nd gen (always use, not 1st gen)
- Regional service (HA within region)
- Automatic backups and maintenance

### High Availability

- Regional HA (synchronous replication, automatic failover)
- RPO: Zero, RTO: <1 minute
- 99.95% SLA with HA
- Cost: 2x instance

### Disaster Recovery

- Point-in-time recovery (PITR)
- Cross-region read replicas
- Backup export to Cloud Storage
- Manual failover to replica

### Scalability

- Vertical: Upgrade machine type (brief downtime)
- Storage: Automatic increase (no downtime)
- Read scaling: Read replicas (async)
- Write scaling: Not supported (use Spanner)

### Backups

- Automated daily backups (7-365 day retention)
- On-demand backups (manual deletion)
- PITR requires automated backups + binary logging
- Test restoration regularly

### Use Cases

- Traditional relational apps
- OLTP workloads
- Migrating from on-premises
- NOT for: Global scale (Spanner), Analytics (BigQuery)

### Cost Optimization

- Right-size machine type
- HA only for production
- Use backups (cheaper than storage)
- Auto-increase storage
- Delete unused replicas/backups
