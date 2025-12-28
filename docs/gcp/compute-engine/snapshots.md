# Compute Engine Disk Snapshots

## Description

Snapshots are point-in-time backups of Persistent Disks. They are incremental, storing only the changes since the previous snapshot, making them space-efficient and cost-effective for regular backups. Snapshots are stored in Cloud Storage and can be used to create new disks or restore existing ones.

**Architecture**: Incremental backups stored globally in Cloud Storage, independent of original disk location.

## Key Features

### Incremental Backups

- **First Snapshot**: Full copy of disk data
- **Subsequent Snapshots**: Only changed blocks since last snapshot
- **Space Efficient**: Significant storage savings for frequent backups
- **Fast Creation**: After first snapshot, subsequent ones are faster
- **Automatic Deduplication**: Same data blocks shared across snapshots

### Global Storage

- **Multi-Regional**: Stored across multiple regions
- **High Durability**: 99.9999999999% (12 9's) durability
- **Cross-Region Restore**: Create disks in any region
- **Geographic Redundancy**: Protected against regional failures

### Snapshot Management

- **Scheduled Snapshots**: Automatic backup schedules
- **Retention Policies**: Auto-delete old snapshots
- **Snapshot Chains**: Linked snapshots for efficient storage
- **On-Demand Snapshots**: Manual backup creation

### Use Cases

**Regular Backups**

- Daily/weekly backup schedules
- Before system updates
- Compliance requirements
- Data protection strategy

**Disaster Recovery**

- Point-in-time recovery
- Cross-region DR
- Quick restore capability
- Test restore procedures

**Disk Cloning**

- Create multiple identical disks
- Scale out workloads
- Development/testing environments
- Data distribution

## Important Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Snapshots per disk** | Unlimited | No hard limit |
| **Snapshots per project** | 10,000 | Default quota |
| **Snapshot size** | 64 TB | Maximum size |
| **Snapshot creation rate** | 1 per 10 minutes per disk | For on-demand |
| **Scheduled snapshots** | 1 schedule per disk | Can have multiple disks per schedule |
| **Retention policies** | Max 65,536 hours (~7.5 years) | Per snapshot schedule |
| **Snapshot chains** | No limit | Automatically managed |

## When to Use Snapshots

### ✅ Use Snapshots When:

1. **Regular Backups Needed**

   - Daily/weekly backup schedules
   - Data protection requirements
   - Compliance mandates
   - Version control for data

2. **Incremental Backups Preferred**

   - Frequent backup cycles
   - Cost-effective storage
   - Fast backup windows
   - Large disks (TBs)

3. **Single Disk Backup**

   - Boot disk backups
   - Data disk backups
   - Per-disk granularity needed
   - No full VM configuration required

4. **Cross-Region Restore**

   - Disaster recovery in different region
   - Geographic data replication
   - Multi-region deployments
   - Data migration between regions

5. **Testing and Development**

   - Clone production data to test
   - Create dev environments
   - Quick provisioning
   - Safe experimentation

### ❌ Don't Use Snapshots When:

1. **Full VM Configuration Needed**

   - Use machine images instead
   - Multiple disks + configuration
   - Complete VM cloning
   - Instance template creation

2. **Real-Time Replication Required**

   - Use regional persistent disks
   - Synchronous replication needed
   - Zero data loss requirement (RPO = 0)
   - Immediate failover needed

3. **Application-Level Backup Better**

   - Database exports (mysqldump, pg_dump)
   - Application-aware backups
   - Transactional consistency required
   - Logical backups preferred

4. **Archive Storage Needed**

   - Use Cloud Storage instead
   - Long-term archival (> 1 year)
   - Cost optimization for cold data
   - Object storage requirements

## Snapshot Operations

### Create Snapshots

**On-Demand Snapshot:**

```bash
# Create snapshot from disk
gcloud compute disks snapshot my-disk \
  --zone=us-central1-a \
  --snapshot-names=my-disk-snapshot

# Create with description
gcloud compute disks snapshot my-disk \
  --zone=us-central1-a \
  --snapshot-names=my-disk-backup-$(date +%Y%m%d) \
  --description="Backup before system upgrade"

# Create snapshot in specific location
gcloud compute disks snapshot my-disk \
  --zone=us-central1-a \
  --snapshot-names=my-snapshot \
  --storage-location=us-central1

# Create with labels
gcloud compute disks snapshot my-disk \
  --zone=us-central1-a \
  --snapshot-names=my-snapshot \
  --labels=environment=production,backup-type=manual
```

**Create Snapshot with VSS (Windows):**

```bash
# Windows VSS-consistent snapshot
gcloud compute disks snapshot my-windows-disk \
  --zone=us-central1-a \
  --snapshot-names=vss-snapshot \
  --guest-flush
```

### Scheduled Snapshots

**Create Snapshot Schedule:**

```bash
# Daily snapshot schedule
gcloud compute resource-policies create snapshot-schedule daily-backup \
  --region=us-central1 \
  --max-retention-days=7 \
  --on-source-disk-delete=keep-auto-snapshots \
  --daily-schedule \
  --start-time=02:00

# Hourly snapshot schedule
gcloud compute resource-policies create snapshot-schedule hourly-backup \
  --region=us-central1 \
  --max-retention-days=1 \
  --on-source-disk-delete=keep-auto-snapshots \
  --hourly-schedule=4 \
  --start-time=00:00

# Weekly snapshot schedule (Sunday at 3 AM)
gcloud compute resource-policies create snapshot-schedule weekly-backup \
  --region=us-central1 \
  --max-retention-days=30 \
  --on-source-disk-delete=keep-auto-snapshots \
  --weekly-schedule=sunday \
  --start-time=03:00

# Custom schedule with specific days
gcloud compute resource-policies create snapshot-schedule custom-backup \
  --region=us-central1 \
  --max-retention-days=14 \
  --weekly-schedule=monday,wednesday,friday \
  --start-time=01:00
```

**Attach Schedule to Disk:**

```bash
# Attach policy to disk
gcloud compute disks add-resource-policies my-disk \
  --zone=us-central1-a \
  --resource-policies=daily-backup

# Attach to multiple disks
gcloud compute disks add-resource-policies disk1 disk2 disk3 \
  --zone=us-central1-a \
  --resource-policies=daily-backup

# Remove policy from disk
gcloud compute disks remove-resource-policies my-disk \
  --zone=us-central1-a \
  --resource-policies=daily-backup
```

**Manage Snapshot Schedules:**

```bash
# List snapshot schedules
gcloud compute resource-policies list \
  --filter="snapshotSchedulePolicy:*"

# Describe schedule
gcloud compute resource-policies describe daily-backup \
  --region=us-central1

# Update schedule
gcloud compute resource-policies update snapshot-schedule daily-backup \
  --region=us-central1 \
  --max-retention-days=14

# Delete schedule
gcloud compute resource-policies delete daily-backup \
  --region=us-central1
```

### Restore from Snapshots

**Create New Disk:**

```bash
# Create disk from snapshot
gcloud compute disks create restored-disk \
  --zone=us-central1-a \
  --source-snapshot=my-snapshot \
  --type=pd-balanced

# Create larger disk from snapshot
gcloud compute disks create larger-disk \
  --zone=us-central1-a \
  --source-snapshot=my-snapshot \
  --size=500GB

# Create in different zone
gcloud compute disks create cross-zone-disk \
  --zone=europe-west1-b \
  --source-snapshot=my-snapshot

# Create different disk type
gcloud compute disks create ssd-disk \
  --zone=us-central1-a \
  --source-snapshot=my-snapshot \
  --type=pd-ssd
```

**Replace Boot Disk:**

```bash
# Stop VM
gcloud compute instances stop my-vm --zone=us-central1-a

# Create new boot disk from snapshot
gcloud compute disks create new-boot-disk \
  --zone=us-central1-a \
  --source-snapshot=boot-disk-snapshot

# Detach old boot disk
gcloud compute instances detach-disk my-vm \
  --zone=us-central1-a \
  --disk=old-boot-disk

# Attach new boot disk
gcloud compute instances attach-disk my-vm \
  --zone=us-central1-a \
  --disk=new-boot-disk \
  --boot

# Start VM
gcloud compute instances start my-vm --zone=us-central1-a
```

### Manage Snapshots

**List and Describe:**

```bash
# List all snapshots
gcloud compute snapshots list

# List with filters
gcloud compute snapshots list --filter="sourceDisk:my-disk"

# List by creation time
gcloud compute snapshots list \
  --sort-by=creationTimestamp \
  --limit=10

# Describe snapshot
gcloud compute snapshots describe my-snapshot

# Get snapshot size
gcloud compute snapshots describe my-snapshot \
  --format="value(storageBytes)"
```

**Delete Snapshots:**

```bash
# Delete single snapshot
gcloud compute snapshots delete my-snapshot

# Delete multiple snapshots
gcloud compute snapshots delete snap1 snap2 snap3

# Delete old snapshots (older than 30 days)
for snap in $(gcloud compute snapshots list \
  --filter="creationTimestamp<$(date -d '30 days ago' --iso-8601)" \
  --format="value(name)"); do
  gcloud compute snapshots delete $snap --quiet
done
```

### Copy Snapshots

**Cross-Project Copy:**

```bash
# Share snapshot with another project
gcloud compute snapshots add-iam-policy-binding my-snapshot \
  --member="serviceAccount:SERVICE_ACCOUNT@target-project.iam.gserviceaccount.com" \
  --role="roles/compute.storageAdmin"

# In target project, create disk from shared snapshot
gcloud compute disks create copied-disk \
  --zone=us-central1-a \
  --source-snapshot=projects/source-project/global/snapshots/my-snapshot
```

## Best Practices

### 1. Use Snapshot Schedules

```bash
# Automate backups with schedules
gcloud compute resource-policies create snapshot-schedule prod-daily \
  --region=us-central1 \
  --max-retention-days=7 \
  --daily-schedule \
  --start-time=02:00 \
  --on-source-disk-delete=keep-auto-snapshots

# Apply to all production disks
for disk in $(gcloud compute disks list \
  --filter="labels.environment=production" \
  --format="value(name,zone)"); do
  DISK_NAME=$(echo $disk | awk '{print $1}')
  ZONE=$(echo $disk | awk '{print $2}' | awk -F/ '{print $NF}')
  gcloud compute disks add-resource-policies $DISK_NAME \
    --zone=$ZONE \
    --resource-policies=prod-daily
done
```

### 2. Implement 3-2-1 Backup Rule

```bash
# 3 copies: Original + 2 snapshots
# 2 different storage types: Disk + Snapshots
# 1 off-site: Multi-regional snapshot storage

# Create multi-regional snapshots
gcloud compute disks snapshot my-disk \
  --zone=us-central1-a \
  --snapshot-names=offsite-backup \
  --storage-location=us  # Multi-regional

# Also maintain local snapshots
gcloud compute disks snapshot my-disk \
  --zone=us-central1-a \
  --snapshot-names=local-backup \
  --storage-location=us-central1  # Regional
```

### 3. Use Consistent Naming Convention

```bash
# Good naming pattern: {resource}-{type}-{date}-{sequence}
gcloud compute disks snapshot prod-db-disk \
  --zone=us-central1-a \
  --snapshot-names=prod-db-snapshot-$(date +%Y%m%d-%H%M%S) \
  --labels=environment=production,service=database,type=scheduled
```

### 4. Test Restore Procedures

```bash
#!/bin/bash
# test-restore.sh - Test snapshot restore monthly

# Get latest snapshot
SNAPSHOT=$(gcloud compute snapshots list \
  --filter="sourceDisk:prod-disk" \
  --sort-by=~creationTimestamp \
  --limit=1 \
  --format="value(name)")

# Create test disk from snapshot
gcloud compute disks create test-restore-disk \
  --zone=us-central1-a \
  --source-snapshot=$SNAPSHOT

# Create test VM
gcloud compute instances create test-restore-vm \
  --zone=us-central1-a \
  --disk=name=test-restore-disk,boot=yes \
  --network=test-vpc

# Verify functionality (manual or automated tests)

# Clean up
gcloud compute instances delete test-restore-vm --zone=us-central1-a --quiet
gcloud compute disks delete test-restore-disk --zone=us-central1-a --quiet
```

### 5. Application-Consistent Snapshots

**For Databases:**

```bash
#!/bin/bash
# Flush database to disk before snapshot

# PostgreSQL example
sudo -u postgres psql -c "CHECKPOINT;"

# Create snapshot
gcloud compute disks snapshot db-disk \
  --zone=us-central1-a \
  --snapshot-names=db-consistent-$(date +%Y%m%d-%H%M%S)

# MySQL example
mysql -e "FLUSH TABLES WITH READ LOCK; SYSTEM gcloud compute disks snapshot mysql-disk --zone=us-central1-a --snapshot-names=mysql-$(date +%Y%m%d-%H%M%S); UNLOCK TABLES;"
```

**For File Systems:**

```bash
# Sync filesystem before snapshot
sync

# Freeze filesystem (requires root)
sudo fsfreeze -f /mnt/data

# Create snapshot
gcloud compute disks snapshot data-disk \
  --zone=us-central1-a \
  --snapshot-names=data-frozen-$(date +%Y%m%d-%H%M%S)

# Unfreeze
sudo fsfreeze -u /mnt/data
```

### 6. Monitor Snapshot Status

```bash
# Create monitoring alert for failed snapshots
gcloud alpha monitoring policies create \
  --notification-channels=CHANNEL_ID \
  --display-name="Snapshot Failure Alert" \
  --condition-display-name="Snapshot Operation Failed" \
  --condition-threshold-value=1 \
  --condition-threshold-duration=0s \
  --condition-filter='
    resource.type="gce_disk"
    AND metric.type="compute.googleapis.com/snapshot/operation/count"
    AND metric.label.state="FAILED"'
```

### 7. Optimize Snapshot Costs

```bash
# Delete old snapshots automatically with retention policy
gcloud compute resource-policies create snapshot-schedule cost-optimized \
  --region=us-central1 \
  --max-retention-days=30 \
  --daily-schedule \
  --start-time=02:00 \
  --on-source-disk-delete=keep-auto-snapshots

# Use regional storage for less critical data
gcloud compute disks snapshot my-disk \
  --zone=us-central1-a \
  --snapshot-names=regional-snapshot \
  --storage-location=us-central1  # Cheaper than multi-regional
```

### 8. Label Snapshots for Organization

```bash
# Comprehensive labeling
gcloud compute disks snapshot my-disk \
  --zone=us-central1-a \
  --snapshot-names=labeled-snapshot \
  --labels=\
environment=production,\
service=web,\
backup-type=scheduled,\
retention=7days,\
created-by=automation,\
cost-center=engineering
```

## Snapshot Performance Considerations

### Minimize Performance Impact

**Best Practices:**

- Schedule snapshots during off-peak hours
- First snapshot has most impact (full copy)
- Subsequent snapshots are faster (incremental)
- Snapshots run in background, minimal I/O impact
- Use `--guest-flush` for consistent snapshots

**Performance Tips:**

```bash
# Schedule during low-usage periods
gcloud compute resource-policies create snapshot-schedule off-peak \
  --region=us-central1 \
  --max-retention-days=7 \
  --daily-schedule \
  --start-time=02:00  # 2 AM local time

# For high-write workloads, reduce snapshot frequency
# Weekly instead of daily
gcloud compute resource-policies create snapshot-schedule weekly \
  --region=us-central1 \
  --max-retention-days=28 \
  --weekly-schedule=sunday \
  --start-time=03:00
```

## Troubleshooting

### Snapshot Creation Fails

```bash
# Check disk status
gcloud compute disks describe my-disk --zone=us-central1-a

# Verify no ongoing operations
gcloud compute operations list --filter="targetLink:my-disk"

# Check quota
gcloud compute project-info describe \
  --project=PROJECT_ID \
  --format="value(quotas[?metric=='SNAPSHOTS'])"

# Check for disk errors
gcloud logging read "resource.type=gce_disk AND resource.labels.disk_id=DISK_ID" \
  --limit=50 \
  --format=json
```

### Cannot Restore from Snapshot

```bash
# Verify snapshot exists
gcloud compute snapshots describe my-snapshot

# Check snapshot status
gcloud compute snapshots describe my-snapshot \
  --format="value(status)"

# Verify permissions
gcloud compute snapshots get-iam-policy my-snapshot

# Check target zone compatibility
gcloud compute zones describe us-central1-a
```

### High Snapshot Costs

```bash
# List snapshots by size
gcloud compute snapshots list \
  --format="table(name, diskSizeGb, storageBytes.size(), storageLocations[0])" \
  --sort-by=~storageBytes

# Identify old snapshots
gcloud compute snapshots list \
  --filter="creationTimestamp<$(date -d '90 days ago' --iso-8601)" \
  --format="table(name, creationTimestamp.date(), diskSizeGb)"

# Delete unnecessary snapshots
# Review and delete manually or with script
```

## Related Resources

- [Compute Engine Overview](compute-engine-overview.md)
- [Virtual Machines](compute-engine-vms.md)
- [Persistent Disks](compute-engine-disks.md)
- [Machine Images](compute-engine-images.md)
- [Backups](compute-engine-backups.md)
- [gcloud CLI](gcloud-cli.md)
