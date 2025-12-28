# Persistent Disks

## Core Concepts

Persistent Disks are durable, network-attached block storage independent of VM lifecycle. Understanding disk types and their performance characteristics is crucial for designing performant and cost-effective solutions.

**Key Principle**: Performance scales with disk size; separate boot and data disks for flexibility.

## Disk Types Comparison

| Type | IOPS/GB | Max IOPS | Throughput | Use Case | Cost |
|------|---------|----------|------------|----------|------|
| **pd-standard** | 0.75-1.5 | 7,500-15,000 | Low | Archives, backups | Lowest |
| **pd-balanced** | 6 | 80,000 | 28 MB/s/GB | General purpose | Medium |
| **pd-ssd** | 30 | 100,000 | 48 MB/s/GB | Databases, high I/O | High |
| **pd-extreme** | Custom | 120,000 | 2,400 MB/s | Mission-critical | Highest |
| **hyperdisk-balanced** | Configurable | 160,000 | Dynamic | Next-gen balanced | Medium-High |
| **hyperdisk-extreme** | Configurable | 350,000 | Dynamic | Highest performance | Highest |
| **Local SSD** | N/A | 2,400,000 | 9,360 MB/s | Ultra-high perf | Medium (ephemeral) |

## Architectural Decision Criteria

### pd-standard (HDD)

**Appropriate for**:

- Sequential access patterns (logs, media files)
- Large datasets with infrequent access
- Backup target storage
- Cost-sensitive workloads
- Data archiving

**Not appropriate for**:

- Database storage
- Random I/O workloads
- Low-latency requirements
- Applications needing >15,000 IOPS

### pd-balanced (SSD) - Recommended Default

**Appropriate for**:

- General-purpose workloads (should be default choice)
- Small to medium databases
- Boot disks
- Most enterprise applications
- Development environments
- Balance of price and performance

**Architecture Pattern**: Start with pd-balanced, upgrade to pd-ssd only if I/O bottleneck identified

### pd-ssd

**Appropriate for**:

- High-performance databases
- I/O-intensive applications
- OLTP workloads
- Applications requiring low latency
- Random I/O patterns

**Cost Consideration**: 2.5x cost of pd-balanced; verify I/O requirements justify cost

### pd-extreme

**Appropriate for**:

- Mission-critical databases (SAP HANA, Oracle)
- Very large databases requiring >100,000 IOPS
- Real-time analytics
- High-frequency trading
- When performance is more important than cost

**Limitations**:

- Requires N2, C2, or M-series VMs
- Minimum size: 500 GB
- Much higher cost than pd-ssd
- Custom IOPS configuration required

### Local SSDs

**Characteristics**:

- Physically attached to server
- Ephemeral (data lost on VM stop/delete)
- Ultra-high performance
- 375 GB per device, up to 24 devices (9 TB max)
- No persistent disk performance limits apply

**Appropriate for**:

- Temporary cache
- Scratch space for computation
- Data that can be rebuilt (replicated databases)
- High-performance computing
- When highest IOPS/throughput needed

**Not appropriate for**:

- Primary database storage without replication
- Data that must survive VM stop
- Critical data without backups
- Single point of failure scenarios

**Architecture Pattern**: Use for performance, replicate critical data externally

## Performance Scaling

### Size-Based Performance

**pd-balanced Example**:

- 100 GB: 600 read IOPS, 600 write IOPS
- 500 GB: 3,000 IOPS
- 13,334 GB: 80,000 IOPS (maximum)

**Design Implication**: May need larger disk for performance, not just capacity

### VM-Level Limits

**Performance is limited by**:

1. Disk type and size
2. VM machine type (CPU count)
3. Number of vCPUs determines max I/O

**Example**:

- n2-standard-2 (2 vCPUs): Max 15,000 read IOPS
- n2-standard-32 (32 vCPUs): Max 100,000 read IOPS

**Architecture Consideration**: Right-size VM for I/O requirements, not just compute

## Regional Persistent Disks

### Synchronous Replication

**Characteristics**:

- Replicated across two zones in same region
- Synchronous writes (both zones acknowledge)
- Automatic failover on zone failure
- RPO: Near-zero (synchronous)
- RTO: Minutes (automatic failover)
- 2x cost of zonal disks

**Appropriate for**:

- High-availability databases
- Mission-critical applications
- Zone failure protection required
- Applications needing automatic failover
- When cost of 2x storage justified by availability

**Not appropriate for**:

- Cost-sensitive workloads
- Single-zone deployments
- Applications with external replication
- Development/testing

**Architecture Pattern**: Use for stateful tier in multi-zone deployment

### Failover Behavior

**Automatic Failover**:

- Regional disk automatically attaches to VM in surviving zone
- Application must handle brief I/O pause
- No data loss (synchronous replication)
- Requires regional MIG for automatic VM recreation

**Limitations**:

- Both zones must be available for writes
- Slightly higher latency than zonal (cross-zone sync)
- Cannot span regions
- Requires zone failure, not VM failure (use MIG autohealing for VM failures)

## Disk Architecture Patterns

### Separate Boot and Data Disks

**Benefits**:

- Independent lifecycle management
- Different disk types (balanced boot, SSD data)
- Easier backups (snapshot data disk only)
- Flexibility to resize independently
- Replace boot disk without data loss

**Pattern**:

- Boot disk: 20-50 GB pd-balanced
- Data disk(s): Size and type based on workload
- Attach data disks to multiple VMs (read-only)

### Multi-Disk for Performance

**Stripe Multiple Disks**:

- Combine multiple disks in RAID 0
- Aggregate IOPS and throughput
- Each disk contributes performance
- Better than single large disk for max performance

**Consideration**: Local SSDs better choice for highest performance

### Disk Quotas and Limits

**Per VM Limits**:

- 128 persistent disks (including boot)
- 257 TB total persistent disk size
- 24 Local SSD devices (9 TB)

**Architecture Impact**:

- Plan data architecture within limits
- Consider object storage (Cloud Storage) for > 128 data sources
- Use Filestore for NFS requirements

## Snapshot Architecture

### Incremental Backups

**How It Works**:

- First snapshot: Full copy of disk
- Subsequent snapshots: Only changed blocks
- Snapshot chains maintained automatically
- Deleting intermediate snapshots safe (blocks preserved if needed)

**Cost Efficiency**:

- Daily snapshots of 1 TB disk with 5% daily change
- Month 1: ~1,000 GB + 30 × 50 GB = ~2,500 GB storage
- Much cheaper than 30 full copies (30,000 GB)

### Snapshot Storage Locations

**Regional**:

- Stored in single region
- Lower cost
- Faster creation/restore (same region)
- Use for: Non-critical data, cost optimization

**Multi-Regional**:

- Stored across multiple regions
- Higher cost
- Disaster recovery protection
- Slower initial creation
- Use for: Critical data, DR requirements

**Architecture Decision**: Balance cost vs DR requirements

## Encryption

### Default Encryption

**Google-Managed Keys**:

- All persistent disks encrypted at rest
- Automatic, no configuration
- No performance impact
- Transparent to applications

### Customer-Managed Encryption Keys (CMEK)

**Benefits**:

- Control key lifecycle
- Regulatory compliance
- Audit key usage
- Revoke access immediately

**Considerations**:

- Additional Cloud KMS costs
- Key management responsibility
- Availability dependency on Cloud KMS
- Slight performance impact

**Use Cases**:

- Regulatory requirements (HIPAA, PCI-DSS)
- Enhanced security posture
- Key rotation policies
- Multi-cloud key management

## Cost Optimization Strategies

### Right-Sizing

**Approach**:

- Start with pd-balanced (cost-effective default)
- Monitor I/O metrics
- Upgrade to pd-ssd only if bottleneck identified
- Downgrade to pd-standard for sequential-only workloads

### Snapshot Management

**Best Practices**:

- Implement retention policies (delete old snapshots)
- Use snapshot schedules (automated lifecycle)
- Regional storage for non-critical data
- Delete disk but keep snapshots (cheaper long-term storage)

### Storage Tiering

**Pattern**:

- **Hot data**: pd-ssd (frequent access)
- **Warm data**: pd-balanced (occasional access)
- **Cold data**: pd-standard or Cloud Storage (rare access)
- **Archive**: Cloud Storage Archive class (long-term retention)

## Disaster Recovery Considerations

### Backup Strategy

**Snapshot Frequency**:

- Critical data: Hourly or continuous (regional disks)
- Important data: Daily
- Normal data: Weekly
- Test data: Monthly or manual

**Retention**:

- Compliance requirements dictate minimum
- Balance cost vs recovery point options
- 30 days common for production
- 7 days for development

### Cross-Region DR

**Approach**:

- Snapshots stored in multi-regional location
- Restore disks in DR region from snapshots
- Test DR procedures regularly
- Document recovery procedures

**RTO Considerations**:

- Snapshot restore: 30-60 minutes
- Regional disk failover: Minutes
- Multi-region failover: 1-2 hours (manual)

## Exam Focus Areas

### Design Decisions

- Disk type selection based on workload
- Performance sizing (IOPS/throughput requirements)
- Regional vs zonal disks for HA
- Cost optimization strategies

### Architecture Patterns

- Separate boot and data disks
- Local SSD use cases and limitations
- Multi-disk configurations
- Snapshot strategies

### Performance

- Size-based performance scaling
- VM-level performance limits
- When to use Local SSDs
- Multi-disk striping considerations

### Disaster Recovery

- Snapshot frequency and retention
- Regional disk RPO/RTO
- Cross-region DR strategies
- Backup architecture patterns

### Cost Management

- Disk type cost comparison
- Snapshot storage optimization
- Unused disk identification
- Storage tiering strategies
