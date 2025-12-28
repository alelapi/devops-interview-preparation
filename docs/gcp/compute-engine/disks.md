# Compute Engine Persistent Disks

## Description

Persistent Disks are durable block storage devices that function like physical hard drives or SSDs. They are independent of VM instances, can be attached to multiple VMs (in read-only mode), and persist beyond the life of a VM. Persistent disks provide reliable, high-performance storage for Compute Engine workloads.

**Architecture**: Network-attached block storage that is automatically replicated for durability and can be dynamically resized.

## Key Features

### Disk Types

**Standard Persistent Disks (pd-standard)**

- HDD-backed storage
- Sequential I/O performance
- Cost-effective for large data
- 0.75 IOPS/GB read, 1.5 IOPS/GB write

**Balanced Persistent Disks (pd-balanced)**

- SSD-backed storage
- Balance of performance and cost
- Recommended for most workloads
- 6 IOPS/GB read, 6 IOPS/GB write
- 28 MB/s throughput per GB

**SSD Persistent Disks (pd-ssd)**

- High-performance SSD storage
- Low-latency, high IOPS
- Database and high-performance apps
- 30 IOPS/GB read, 30 IOPS/GB write
- 48 MB/s throughput per GB

**Extreme Persistent Disks (pd-extreme)**

- Highest performance, lowest latency
- Custom IOPS configuration
- Up to 120,000 IOPS per disk
- Premium pricing
- Requires specific machine types (N2, C2, M1/M2)

**Hyperdisk Balanced (hyperdisk-balanced)**

- Next-generation balanced performance
- Dynamic IOPS and throughput configuration
- Better price/performance than pd-balanced
- 4000-160000 IOPS per disk

**Hyperdisk Extreme (hyperdisk-extreme)**

- Next-generation extreme performance
- Up to 350,000 IOPS per disk
- Lowest latency
- Premium pricing

### Storage Features

**Durability and Availability**

- Automatic replication within zone (zonal disks)
- Replication across zones (regional disks)
- 99.9999% durability (regional)
- 99.999% durability (zonal)

**Flexibility**

- Attach/detach without VM restart (data disks)
- Resize without downtime
- Change disk type after creation
- Attach to multiple VMs in read-only mode

**Performance**

- Performance scales with disk size
- Baseline performance guaranteed
- Burst performance for short periods
- Multi-queue SCSI support

**Regional Persistent Disks**

- Synchronous replication across two zones
- Higher availability
- Automatic failover
- Higher cost (2x zonal disk)

### Local SSDs

**Characteristics:**

- Physically attached to server
- Ultra-high performance (up to 2,400,000 IOPS)
- Very low latency
- Ephemeral (data lost on VM stop/delete)
- 375 GB per device, up to 24 devices (9 TB)

**Use Cases:**

- Temporary cache
- Scratch space
- High-performance databases with replication
- Data processing pipelines

## Important Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Disks per VM** | 128 | Including boot disk |
| **Disk size (zonal)** | 64 TB | pd-standard, pd-balanced, pd-ssd |
| **Disk size (regional)** | 64 TB | Same as zonal |
| **Disk size (pd-extreme)** | 64 TB | Minimum 500 GB |
| **Disk size (hyperdisk)** | 64 TB | Configurable IOPS |
| **Local SSD per VM** | 24 devices (9 TB) | Varies by machine type |
| **Total PD size per VM** | 257 TB | Across all attached disks |
| **Persistent disks per project** | 500 per zone | Can be increased |
| **Snapshots per disk** | Unlimited | Subject to project quota |

### Performance Limits

**pd-standard (HDD):**

- Read IOPS: 0.75 per GB (max 7,500)
- Write IOPS: 1.5 per GB (max 15,000)
- Read throughput: 0.12 MB/s per GB (max 1,200 MB/s)
- Write throughput: 0.12 MB/s per GB (max 400 MB/s)

**pd-balanced (SSD):**

- Read/Write IOPS: 6 per GB (max 80,000)
- Throughput: 28 MB/s per GB (max 1,200 MB/s read, 1,000 MB/s write)

**pd-ssd:**

- Read/Write IOPS: 30 per GB (max 100,000)
- Throughput: 48 MB/s per GB (max 1,200 MB/s read, 1,000 MB/s write)

**pd-extreme:**

- IOPS: Configurable up to 120,000
- Throughput: Up to 2,400 MB/s

**Local SSD:**

- Read IOPS: Up to 2,400,000 (combined)
- Write IOPS: Up to 1,200,000 (combined)
- Read throughput: Up to 9,360 MB/s
- Write throughput: Up to 4,680 MB/s

## When to Use Different Disk Types

### pd-standard (HDD)

✅ **Use For:**

- Sequential data access
- Backup and archival storage
- Large data volumes (logs, media)
- Development/testing
- Cost-sensitive workloads

❌ **Don't Use For:**

- Database storage
- High IOPS requirements
- Low-latency applications
- Random I/O workloads

### pd-balanced (SSD)

✅ **Use For:**

- General-purpose workloads (recommended default)
- Web servers and applications
- Medium-sized databases
- Enterprise applications
- Balance of price and performance

❌ **Don't Use For:**

- Highest performance databases
- Ultra-low latency requirements
- Very large sequential I/O
- Applications needing >80,000 IOPS

### pd-ssd

✅ **Use For:**

- High-performance databases
- Transaction processing
- Analytics workloads
- I/O intensive applications
- Low-latency requirements

❌ **Don't Use For:**

- Cost-sensitive workloads
- Sequential-only access
- Applications with <6 IOPS/GB needs
- Backup storage

### pd-extreme

✅ **Use For:**

- Highest performance databases (SAP HANA, Oracle)
- Real-time analytics
- High-frequency trading
- Large-scale enterprise apps requiring >100,000 IOPS

❌ **Don't Use For:**

- General workloads
- Cost-sensitive applications
- Small databases
- Non-performance-critical apps

### Regional Persistent Disks

✅ **Use For:**

- High-availability applications
- Databases requiring regional redundancy
- Production workloads needing failover
- Critical data requiring zone-level redundancy

❌ **Don't Use For:**

- Cost-sensitive workloads (2x cost)
- Single-zone deployments
- Non-critical applications
- Temporary data

### Local SSDs

✅ **Use For:**

- Temporary cache or scratch space
- High-performance databases with external replication
- Data processing pipelines
- Applications needing ultra-low latency
- Temporary data that can be rebuilt

❌ **Don't Use For:**

- Persistent application data
- Critical data without backups
- Applications that can't handle data loss
- Data that must survive VM stops

## Disk Operations

### Create Persistent Disks

**Create Empty Disk:**

```bash
# Create balanced PD (default)
gcloud compute disks create my-disk \
  --zone=us-central1-a \
  --size=100GB \
  --type=pd-balanced

# Create SSD disk
gcloud compute disks create my-ssd-disk \
  --zone=us-central1-a \
  --size=500GB \
  --type=pd-ssd

# Create regional disk
gcloud compute disks create my-regional-disk \
  --region=us-central1 \
  --replica-zones=us-central1-a,us-central1-b \
  --size=200GB \
  --type=pd-balanced

# Create extreme disk with custom IOPS
gcloud compute disks create my-extreme-disk \
  --zone=us-central1-a \
  --size=1000GB \
  --type=pd-extreme \
  --provisioned-iops=100000
```

**Create from Snapshot:**

```bash
gcloud compute disks create my-disk \
  --zone=us-central1-a \
  --source-snapshot=my-snapshot \
  --type=pd-balanced
```

**Create from Image:**

```bash
gcloud compute disks create boot-disk \
  --zone=us-central1-a \
  --size=50GB \
  --type=pd-balanced \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud
```

### Attach and Detach Disks

**Attach Disk to VM:**

```bash
# Attach in read-write mode
gcloud compute instances attach-disk my-vm \
  --zone=us-central1-a \
  --disk=my-disk

# Attach in read-only mode
gcloud compute instances attach-disk my-vm \
  --zone=us-central1-a \
  --disk=my-disk \
  --mode=ro

# Attach with custom device name
gcloud compute instances attach-disk my-vm \
  --zone=us-central1-a \
  --disk=my-disk \
  --device-name=custom-disk-name
```

**Format and Mount (Inside VM):**

```bash
# List disks
sudo lsblk

# Format disk (WARNING: destroys data)
sudo mkfs.ext4 -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/sdb

# Create mount point
sudo mkdir -p /mnt/disks/data

# Mount disk
sudo mount -o discard,defaults /dev/sdb /mnt/disks/data

# Persist mount in /etc/fstab
echo UUID=$(sudo blkid -s UUID -o value /dev/sdb) /mnt/disks/data ext4 discard,defaults,nofail 0 2 | sudo tee -a /etc/fstab

# Verify
df -h /mnt/disks/data
```

**Detach Disk:**

```bash
# Unmount first (inside VM)
sudo umount /mnt/disks/data

# Detach disk
gcloud compute instances detach-disk my-vm \
  --zone=us-central1-a \
  --disk=my-disk
```

### Resize Disks

```bash
# Resize disk (increase only)
gcloud compute disks resize my-disk \
  --zone=us-central1-a \
  --size=200GB

# Resize disk in place (no detach needed)
gcloud compute disks resize my-disk \
  --zone=us-central1-a \
  --size=300GB

# Resize filesystem (inside VM after disk resize)
# For ext4
sudo resize2fs /dev/sdb

# For XFS
sudo xfs_growfs /mnt/disks/data

# Verify
df -h
```

### Change Disk Type

```bash
# Stop VM first
gcloud compute instances stop my-vm --zone=us-central1-a

# Create snapshot of current disk
gcloud compute disks snapshot my-disk \
  --zone=us-central1-a \
  --snapshot-names=my-disk-snapshot

# Create new disk from snapshot with different type
gcloud compute disks create my-new-disk \
  --zone=us-central1-a \
  --source-snapshot=my-disk-snapshot \
  --type=pd-ssd

# Detach old disk
gcloud compute instances detach-disk my-vm \
  --zone=us-central1-a \
  --disk=my-disk

# Attach new disk
gcloud compute instances attach-disk my-vm \
  --zone=us-central1-a \
  --disk=my-new-disk \
  --device-name=persistent-disk-0

# Start VM
gcloud compute instances start my-vm --zone=us-central1-a
```

### Disk Management

**List Disks:**

```bash
# List all disks
gcloud compute disks list

# List disks in specific zone
gcloud compute disks list --zones=us-central1-a

# List with details
gcloud compute disks list --format="table(
  name,
  zone,
  type,
  sizeGb,
  status,
  users[0]:label=IN_USE_BY
)"
```

**Describe Disk:**

```bash
gcloud compute disks describe my-disk \
  --zone=us-central1-a
```

**Delete Disk:**

```bash
# Delete disk (must be detached first)
gcloud compute disks delete my-disk \
  --zone=us-central1-a

# Delete multiple disks
gcloud compute disks delete disk1 disk2 disk3 \
  --zone=us-central1-a
```

### Working with Local SSDs

**Create VM with Local SSDs:**

```bash
# Create VM with local SSDs
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-8 \
  --local-ssd=interface=NVME \
  --local-ssd=interface=NVME \
  --local-ssd=interface=NVME

# Create with SCSI interface
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-8 \
  --local-ssd=interface=SCSI \
  --local-ssd=interface=SCSI
```

**Format and Mount Local SSDs (Inside VM):**

```bash
# Install mdadm for RAID
sudo apt-get update
sudo apt-get install -y mdadm

# Find local SSDs
ls -l /dev/disk/by-id/google-local-*

# Create RAID 0 array (for performance)
sudo mdadm --create /dev/md0 \
  --level=0 \
  --raid-devices=3 \
  /dev/disk/by-id/google-local-ssd-0 \
  /dev/disk/by-id/google-local-ssd-1 \
  /dev/disk/by-id/google-local-ssd-2

# Format RAID array
sudo mkfs.ext4 -F /dev/md0

# Mount
sudo mkdir -p /mnt/disks/local-ssd
sudo mount /dev/md0 /mnt/disks/local-ssd

# Set permissions
sudo chmod a+w /mnt/disks/local-ssd
```

## Best Practices

### 1. Choose Appropriate Disk Type

```bash
# Default recommendation: pd-balanced
# Database/high-performance: pd-ssd or pd-extreme
# Large sequential data: pd-standard
# Ultra-high performance: Local SSD

# Start with pd-balanced and upgrade if needed
gcloud compute disks create my-disk \
  --zone=us-central1-a \
  --size=100GB \
  --type=pd-balanced
```

### 2. Size Disks for Performance

```bash
# Performance scales with size
# For pd-balanced: 6 IOPS/GB, need 13,334 GB for max 80,000 IOPS
# For pd-ssd: 30 IOPS/GB, need 3,334 GB for max 100,000 IOPS

# If you need 50,000 IOPS with pd-balanced:
# Minimum size: 50,000 / 6 = 8,334 GB
gcloud compute disks create high-iops-disk \
  --zone=us-central1-a \
  --size=8400GB \
  --type=pd-balanced
```

### 3. Use Regional Disks for HA

```bash
# For critical databases and applications
gcloud compute disks create ha-disk \
  --region=us-central1 \
  --replica-zones=us-central1-a,us-central1-b \
  --size=500GB \
  --type=pd-ssd

# Create regional VM with regional disk
gcloud compute instances create ha-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --disk=name=ha-disk,boot=yes
```

### 4. Regular Snapshots for Backup

```bash
# Create snapshot schedule
gcloud compute resource-policies create snapshot-schedule daily-backup \
  --region=us-central1 \
  --max-retention-days=7 \
  --on-source-disk-delete=keep-auto-snapshots \
  --daily-schedule \
  --start-time=02:00

# Attach schedule to disk
gcloud compute disks add-resource-policies my-disk \
  --zone=us-central1-a \
  --resource-policies=daily-backup
```

### 5. Monitor Disk Performance

```bash
# View disk metrics
gcloud monitoring time-series list \
  --filter='metric.type="compute.googleapis.com/instance/disk/read_bytes_count"
    AND resource.labels.instance_id="INSTANCE_ID"' \
  --format=json

# Inside VM: Monitor IOPS and throughput
iostat -x 1

# Monitor disk usage
df -h
du -sh /mnt/disks/data/*
```

### 6. Optimize Disk Configuration

```bash
# Use discard option for SSD trim
sudo mount -o discard,defaults /dev/sdb /mnt/disks/data

# Enable multi-queue SCSI (in /etc/default/grub)
GRUB_CMDLINE_LINUX="scsi_mod.use_blk_mq=1"

# Optimize for databases
# Set readahead to 0 or low value
sudo blockdev --setra 0 /dev/sdb
```

### 7. Data Protection

```bash
# Create snapshots before major changes
gcloud compute disks snapshot my-disk \
  --zone=us-central1-a \
  --snapshot-names=pre-upgrade-snapshot

# Use regional disks for critical data
# Implement backup strategy
# Test restore procedures regularly
```

## Disk Encryption

### Customer-Managed Encryption Keys (CMEK)

```bash
# Create KMS key
gcloud kms keyrings create my-keyring \
  --location=us-central1

gcloud kms keys create my-key \
  --location=us-central1 \
  --keyring=my-keyring \
  --purpose=encryption

# Create disk with CMEK
gcloud compute disks create encrypted-disk \
  --zone=us-central1-a \
  --size=100GB \
  --type=pd-balanced \
  --kms-key=projects/PROJECT_ID/locations/us-central1/keyRings/my-keyring/cryptoKeys/my-key
```

### Default Encryption

All persistent disks are encrypted at rest by default using Google-managed keys. No configuration required.

## Troubleshooting

### Disk Won't Attach

```bash
# Check if disk is already attached
gcloud compute disks describe my-disk --zone=us-central1-a

# Check disk status
gcloud compute disks list --filter="name=my-disk"

# Verify disk and VM are in same zone
gcloud compute instances describe my-vm --zone=us-central1-a
```

### Poor Performance

```bash
# Check disk type and size
gcloud compute disks describe my-disk --zone=us-central1-a

# Calculate expected IOPS
# pd-balanced: 6 IOPS/GB
# Example: 100 GB disk = 600 IOPS baseline

# Monitor actual IOPS (inside VM)
iostat -x 1 10

# Check if hitting limits
# Cloud Monitoring → Compute Engine → Disk
```

### Disk Full

```bash
# Check disk usage
df -h

# Find large files/directories
sudo du -sh /* | sort -h
sudo du -h /var | sort -h | tail -20

# Clean up if possible or resize disk
gcloud compute disks resize my-disk \
  --zone=us-central1-a \
  --size=200GB

# Then resize filesystem
sudo resize2fs /dev/sdb
```

## Related Resources

- [Compute Engine Overview](compute-engine-overview.md)
- [Virtual Machines](compute-engine-vms.md)
- [Machine Images](compute-engine-images.md)
- [Snapshots](compute-engine-snapshots.md)
- [Backups](compute-engine-backups.md)
- [gcloud CLI](gcloud-cli.md)
