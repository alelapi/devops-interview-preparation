# Compute Engine Backup and Disaster Recovery

## Description

A comprehensive backup and disaster recovery (DR) strategy for Compute Engine combines snapshots, machine images, and operational procedures to protect data and ensure business continuity. This guide covers backup methodologies, recovery procedures, and best practices for production environments.

**Goal**: Minimize data loss (RPO) and downtime (RTO) while optimizing costs and operational overhead.

## Key Concepts

### Recovery Objectives

**RPO (Recovery Point Objective)**

- Maximum acceptable data loss
- Time between last backup and disaster
- Measured in time (minutes, hours, days)
- Determines backup frequency

**RTO (Recovery Time Objective)**

- Maximum acceptable downtime
- Time to restore and resume operations
- Measured in time (minutes, hours, days)
- Determines recovery strategy

**Common Targets:**

- **Tier 1 (Critical)**: RPO < 1 hour, RTO < 1 hour
- **Tier 2 (Important)**: RPO < 4 hours, RTO < 4 hours
- **Tier 3 (Normal)**: RPO < 24 hours, RTO < 24 hours
- **Tier 4 (Low Priority)**: RPO < 7 days, RTO < 48 hours

### Backup Types

**Full Backup**

- Complete copy of all data
- First machine image or snapshot
- Longest creation time
- Highest storage cost
- Simplest restoration

**Incremental Backup**

- Only changes since last backup
- Snapshots after the first
- Fast creation
- Low storage cost
- Chain dependency for restore

**Differential Backup**

- Changes since last full backup
- Not natively supported (use snapshots)
- Faster restore than incremental
- Higher storage than incremental

### Disaster Scenarios

**Data Corruption**

- Application bugs
- Human error
- Malware/ransomware
- Database corruption

**Hardware Failure**

- Disk failure (rare with persistent disks)
- Regional outage
- Zone unavailability
- Network failure

**Deletion/Misconfiguration**

- Accidental deletion
- Configuration errors
- Permission changes
- Resource modifications

**Regional Disaster**

- Natural disasters
- Extended outages
- Region-wide failures
- Data center incidents

## Backup Strategies

### Strategy 1: Snapshot-Based Backup

**Characteristics:**

- Incremental backups
- Disk-level granularity
- Fast, efficient
- Cost-effective

**Implementation:**

```bash
# Create snapshot schedule for all production disks
gcloud compute resource-policies create snapshot-schedule prod-backup \
  --region=us-central1 \
  --max-retention-days=30 \
  --daily-schedule \
  --start-time=02:00 \
  --on-source-disk-delete=keep-auto-snapshots

# Apply to disks
for disk in $(gcloud compute disks list \
  --filter="labels.tier=critical" \
  --format="value(name,zone)"); do
  DISK_NAME=$(echo $disk | awk '{print $1}')
  ZONE=$(echo $disk | awk '{print $2}' | awk -F/ '{print $NF}')
  gcloud compute disks add-resource-policies $DISK_NAME \
    --zone=$ZONE \
    --resource-policies=prod-backup
done
```

**RPO/RTO:**

- RPO: 24 hours (daily snapshots)
- RTO: 30-60 minutes (restore time)

**Pros:**

- Low cost
- Automatic
- Incremental
- Cross-region restore

**Cons:**

- Disk-level only (no VM config)
- 24-hour RPO minimum
- Restore requires disk recreation

### Strategy 2: Machine Image Backup

**Characteristics:**

- Complete VM configuration
- All disks included
- VM-level granularity
- Quick VM cloning

**Implementation:**

```bash
# Weekly machine image backup
#!/bin/bash
for vm in $(gcloud compute instances list \
  --filter="labels.backup=required" \
  --format="value(name,zone)"); do
  VM_NAME=$(echo $vm | awk '{print $1}')
  ZONE=$(echo $vm | awk '{print $2}' | awk -F/ '{print $NF}')
  IMAGE_NAME="${VM_NAME}-$(date +%Y%m%d)"
  
  gcloud compute machine-images create $IMAGE_NAME \
    --source-instance=$VM_NAME \
    --source-instance-zone=$ZONE \
    --storage-location=us
done

# Schedule via Cloud Scheduler
```

**RPO/RTO:**

- RPO: 7 days (weekly images)
- RTO: 15-30 minutes (fast VM recreation)

**Pros:**

- Complete VM config
- Fast restore
- Easy VM cloning
- Cross-project restore

**Cons:**

- Higher cost
- Larger storage footprint
- Slower creation
- Less frequent backups

### Strategy 3: Hybrid Approach (Recommended)

**Combines snapshots and machine images:**

```bash
# Daily snapshots for data disks
gcloud compute resource-policies create snapshot-schedule daily-data \
  --region=us-central1 \
  --max-retention-days=7 \
  --daily-schedule \
  --start-time=02:00

# Weekly machine images for complete VM
gcloud compute resource-policies create snapshot-schedule weekly-full \
  --region=us-central1 \
  --max-retention-days=30 \
  --weekly-schedule=sunday \
  --start-time=03:00
```

**RPO/RTO:**

- RPO: 24 hours (daily snapshots)
- RTO: 30-45 minutes

**Benefits:**

- Best of both approaches
- Flexible recovery options
- Optimized cost/protection ratio

### Strategy 4: Regional Persistent Disks

**For critical, zero-downtime requirements:**

```bash
# Create regional disk
gcloud compute disks create critical-data \
  --region=us-central1 \
  --replica-zones=us-central1-a,us-central1-b \
  --size=500GB \
  --type=pd-ssd

# Create regional VM
gcloud compute instances create critical-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --disk=name=critical-data,boot=yes

# Automatic failover in zone failure
```

**RPO/RTO:**

- RPO: Near-zero (synchronous replication)
- RTO: Minutes (automatic failover)

**Pros:**

- Synchronous replication
- Automatic failover
- No data loss
- Zone failure protection

**Cons:**

- 2x disk cost
- Regional only (not multi-region)
- Requires regional disk support

## Disaster Recovery Implementation

### Multi-Region DR Setup

**Primary Region: us-central1**
**DR Region: europe-west1**

```bash
#!/bin/bash
# Cross-region DR setup

# 1. Create regular snapshots in multi-regional storage
gcloud compute resource-policies create snapshot-schedule dr-backup \
  --region=us-central1 \
  --max-retention-days=30 \
  --daily-schedule \
  --start-time=02:00

# Apply to production disks
gcloud compute disks add-resource-policies prod-disk \
  --zone=us-central1-a \
  --resource-policies=dr-backup

# 2. Create machine image in DR location
gcloud compute machine-images create dr-image \
  --source-instance=prod-vm \
  --source-instance-zone=us-central1-a \
  --storage-location=eu

# 3. Test DR restore (quarterly)
# Create VM in DR region from latest snapshot
SNAPSHOT=$(gcloud compute snapshots list \
  --filter="sourceDisk:prod-disk" \
  --sort-by=~creationTimestamp \
  --limit=1 \
  --format="value(name)")

gcloud compute disks create dr-test-disk \
  --zone=europe-west1-b \
  --source-snapshot=$SNAPSHOT

gcloud compute instances create dr-test-vm \
  --zone=europe-west1-b \
  --machine-type=n2-standard-4 \
  --disk=name=dr-test-disk,boot=yes \
  --network=dr-vpc

# Verify and cleanup
```

### Automated Backup Verification

```bash
#!/bin/bash
# verify-backups.sh
# Run weekly to verify backups are restorable

PROJECT_ID="my-project"
TEST_ZONE="us-central1-a"

# Get list of critical VMs
CRITICAL_VMS=$(gcloud compute instances list \
  --filter="labels.tier=critical" \
  --format="value(name)")

for VM in $CRITICAL_VMS; do
  echo "Testing restore for: $VM"
  
  # Get latest snapshot
  SNAPSHOT=$(gcloud compute snapshots list \
    --filter="sourceDisk:$VM" \
    --sort-by=~creationTimestamp \
    --limit=1 \
    --format="value(name)")
  
  if [ -z "$SNAPSHOT" ]; then
    echo "ERROR: No snapshot found for $VM"
    # Send alert
    continue
  fi
  
  # Create test disk
  TEST_DISK="${VM}-verify-$(date +%s)"
  gcloud compute disks create $TEST_DISK \
    --zone=$TEST_ZONE \
    --source-snapshot=$SNAPSHOT \
    --quiet
  
  if [ $? -eq 0 ]; then
    echo "SUCCESS: Snapshot verified for $VM"
    gcloud compute disks delete $TEST_DISK --zone=$TEST_ZONE --quiet
  else
    echo "ERROR: Failed to restore snapshot for $VM"
    # Send alert
  fi
done
```

## Recovery Procedures

### Scenario 1: Single Disk Corruption

**Recovery Steps:**

```bash
# 1. Identify affected disk
DISK_NAME="corrupted-disk"
ZONE="us-central1-a"

# 2. Stop VM (if running)
gcloud compute instances stop my-vm --zone=$ZONE

# 3. Create disk from latest snapshot
SNAPSHOT=$(gcloud compute snapshots list \
  --filter="sourceDisk:$DISK_NAME" \
  --sort-by=~creationTimestamp \
  --limit=1 \
  --format="value(name)")

gcloud compute disks create restored-disk \
  --zone=$ZONE \
  --source-snapshot=$SNAPSHOT

# 4. Detach corrupted disk
gcloud compute instances detach-disk my-vm \
  --zone=$ZONE \
  --disk=$DISK_NAME

# 5. Attach restored disk
gcloud compute instances attach-disk my-vm \
  --zone=$ZONE \
  --disk=restored-disk \
  --device-name=persistent-disk-0

# 6. Start VM
gcloud compute instances start my-vm --zone=$ZONE

# 7. Verify and delete corrupted disk
gcloud compute disks delete $DISK_NAME --zone=$ZONE
```

**RTO:** 30-45 minutes

### Scenario 2: Complete VM Loss

**Recovery Steps:**

```bash
# 1. Identify latest machine image
MACHINE_IMAGE=$(gcloud compute machine-images list \
  --filter="name~my-vm" \
  --sort-by=~creationTimestamp \
  --limit=1 \
  --format="value(name)")

# 2. Restore VM from machine image
gcloud compute instances create my-vm-restored \
  --zone=us-central1-a \
  --source-machine-image=$MACHINE_IMAGE

# 3. Verify VM functionality
gcloud compute ssh my-vm-restored --zone=us-central1-a

# 4. Update DNS/load balancer if needed
gcloud compute forwarding-rules set-target \
  --global \
  --target-http-proxy=my-proxy
```

**RTO:** 15-30 minutes

### Scenario 3: Regional Disaster

**Recovery Steps:**

```bash
#!/bin/bash
# regional-failover.sh
# Restore in DR region

PRIMARY_REGION="us-central1"
DR_REGION="europe-west1"
DR_ZONE="europe-west1-b"

# 1. Get latest snapshots
for disk in $(gcloud compute disks list \
  --filter="region:$PRIMARY_REGION AND labels.tier=critical" \
  --format="value(name)"); do
  
  SNAPSHOT=$(gcloud compute snapshots list \
    --filter="sourceDisk:$disk" \
    --sort-by=~creationTimestamp \
    --limit=1 \
    --format="value(name)")
  
  # 2. Create disks in DR region
  gcloud compute disks create ${disk}-dr \
    --zone=$DR_ZONE \
    --source-snapshot=$SNAPSHOT
done

# 3. Create VMs in DR region
# Use machine images or instance templates
gcloud compute instances create vm1-dr vm2-dr vm3-dr \
  --zone=$DR_ZONE \
  --source-machine-image=latest-machine-image

# 4. Update DNS to point to DR region
gcloud dns record-sets transaction start --zone=my-zone
gcloud dns record-sets transaction remove \
  --zone=my-zone \
  --name=app.example.com \
  --type=A \
  --ttl=300 \
  "OLD_IP"
gcloud dns record-sets transaction add \
  --zone=my-zone \
  --name=app.example.com \
  --type=A \
  --ttl=300 \
  "NEW_DR_IP"
gcloud dns record-sets transaction execute --zone=my-zone

# 5. Verify services
```

**RTO:** 1-2 hours (depending on complexity)

### Scenario 4: Ransomware/Malware

**Recovery Steps:**

```bash
# 1. Isolate infected systems immediately
gcloud compute instances stop infected-vm --zone=us-central1-a

# 2. Identify last known good snapshot (before infection)
# Check timestamps and verify integrity
gcloud compute snapshots list \
  --filter="sourceDisk:infected-disk" \
  --sort-by=creationTimestamp

# 3. Create disk from clean snapshot
gcloud compute disks create clean-disk \
  --zone=us-central1-a \
  --source-snapshot=CLEAN_SNAPSHOT_NAME

# 4. Create new VM instance
gcloud compute instances create clean-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --disk=name=clean-disk,boot=yes

# 5. Verify system is clean
# Run security scans
# Check for persistence mechanisms

# 6. Implement additional security controls
# Update firewall rules
# Enable VPC Service Controls
# Review IAM permissions
```

## Best Practices

### 1. Implement 3-2-1 Backup Rule

**3 Copies:** Original + 2 backups
**2 Media Types:** Disk + Snapshots (different storage types)
**1 Off-Site:** Multi-regional snapshots

```bash
# Production data: On disk
# Backup 1: Regional snapshots (same region)
# Backup 2: Multi-regional machine images

gcloud compute disks snapshot prod-disk \
  --zone=us-central1-a \
  --snapshot-names=local-backup \
  --storage-location=us-central1

gcloud compute machine-images create offsite-backup \
  --source-instance=prod-vm \
  --source-instance-zone=us-central1-a \
  --storage-location=us  # Multi-regional
```

### 2. Test Recovery Procedures Regularly

```bash
# Monthly DR drill
#!/bin/bash
# 1. Restore random production VM
# 2. Verify functionality
# 3. Document results
# 4. Clean up resources
# 5. Update runbooks

# Automated quarterly DR test
```

### 3. Document Recovery Procedures

Create runbooks for each scenario:

- Single disk recovery
- Complete VM recovery
- Database recovery
- Regional failover
- Contact information
- Escalation procedures

### 4. Monitor Backup Health

```bash
# Alert on backup failures
gcloud alpha monitoring policies create \
  --notification-channels=CHANNEL_ID \
  --display-name="Backup Failure Alert" \
  --condition-threshold-value=1 \
  --condition-filter='
    resource.type="gce_disk"
    AND metric.type="compute.googleapis.com/snapshot/operation/count"
    AND metric.label.state="FAILED"'

# Alert on missing backups
# Create Cloud Function to check last backup timestamp
```

### 5. Automate Everything

```bash
# Backup automation
# Verification automation
# Alert automation
# Recovery testing automation

# Use Cloud Scheduler + Cloud Functions
# Infrastructure as Code (Terraform)
# Configuration Management
```

### 6. Secure Backups

```bash
# Customer-managed encryption keys
gcloud compute disks snapshot my-disk \
  --zone=us-central1-a \
  --snapshot-names=encrypted-backup \
  --kms-key=projects/PROJECT/locations/LOCATION/keyRings/RING/cryptoKeys/KEY

# Immutable backups (prevent deletion)
# Use IAM conditions for time-based protection
# Implement retention locks

# Separate backup admin permissions
# Least privilege access
```

### 7. Calculate Costs

```bash
# Snapshot costs
# Storage: $0.026/GB/month (regional)
# Storage: $0.026/GB/month (multi-regional)

# Example: 1 TB disk, daily snapshots, 30-day retention
# First snapshot: 1024 GB
# Daily changes: ~50 GB (5%)
# Storage after 30 days: ~2500 GB
# Cost: 2500 * $0.026 = $65/month

# Machine images more expensive (full copies)
# Balance cost vs. requirements
```

## Compliance and Governance

### Retention Policies

```bash
# Implement retention based on compliance requirements
gcloud compute resource-policies create snapshot-schedule compliance-backup \
  --region=us-central1 \
  --max-retention-days=2555 \  # 7 years
  --weekly-schedule=sunday \
  --start-time=03:00 \
  --on-source-disk-delete=keep-auto-snapshots
```

### Audit Logging

```bash
# Enable audit logs for backup operations
# Monitor who creates/deletes snapshots
# Track recovery operations

gcloud logging read "
  resource.type=gce_snapshot
  AND (protoPayload.methodName=compute.snapshots.delete
  OR protoPayload.methodName=compute.snapshots.insert)" \
  --format=json
```

### Compliance Requirements

**HIPAA:** 6-year retention, encryption, audit logs
**SOX:** 7-year retention, access controls, change management
**GDPR:** Right to erasure, data portability, encryption
**PCI-DSS:** Encryption, access controls, logging

## Related Resources

- [Compute Engine Overview](compute-engine-overview.md)
- [Virtual Machines](compute-engine-vms.md)
- [Persistent Disks](compute-engine-disks.md)
- [Machine Images](compute-engine-images.md)
- [Snapshots](compute-engine-snapshots.md)
- [gcloud CLI](gcloud-cli.md)
