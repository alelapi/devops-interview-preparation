# Compute Engine Machine Images

## Description

Machine images are Compute Engine resources that store complete VM instance configuration including disks, machine type, metadata, and permissions. They provide a convenient way to create VM instance templates, backup entire VM configurations, and replicate instances across zones and projects.

**Concept**: Complete VM configuration snapshots that can be used to create identical instances or migrate VMs.

## Key Features

### Complete Configuration Capture

- **Instance Configuration**: Machine type, CPU platform, network settings
- **Disk Data**: All attached disks (boot and data disks)
- **Metadata**: Custom metadata, startup scripts, labels
- **Permissions**: Service account and scopes
- **Network Interfaces**: Multiple NICs configuration
- **Scheduling Options**: Preemptibility, automatic restart settings

### Disk Images vs Machine Images

**Disk Images:**

- Single disk snapshot
- Operating system and data from one disk
- Used for boot disks
- Simpler, faster to create
- Cannot capture VM configuration

**Machine Images:**

- Complete VM configuration
- All disks (boot and data)
- Full VM metadata and settings
- Used for complete VM cloning/backup
- Cross-project and cross-region support

### Use Cases

**Backup and Disaster Recovery**

- Complete VM backup including all disks
- Point-in-time recovery
- Cross-region DR copies
- Automated backup schedules

**VM Migration**

- Move VMs between projects
- Clone VMs across regions/zones
- Replicate development environments
- Create test instances from production

**Instance Templates**

- Base configuration for MIGs
- Standardized VM deployments
- Golden images for teams
- Compliance and security baselines

## Important Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Machine images per project** | 1,000 | Soft limit |
| **Image size** | 10 TB | Combined disk sizes |
| **Disks per image** | 128 | Maximum attached disks |
| **Images shared across projects** | Unlimited | Using IAM |
| **Images per image family** | Unlimited | Logical grouping |
| **Concurrent image creation** | 20 per region | Can be increased |

## When to Use Machine Images vs Other Options

### ✅ Use Machine Images When:

1. **Complete VM Backup Needed**

   - Backup entire VM configuration
   - Multiple disks need to be captured
   - Instance settings must be preserved
   - Quick VM restoration required

2. **VM Migration**

   - Moving VMs between projects
   - Replicating VMs across regions
   - Creating identical environments
   - Cloning production to staging

3. **Instance Templates**

   - Standard VM configurations
   - Managed Instance Groups
   - Auto-scaling deployments
   - Team development environments

4. **Disaster Recovery**

   - Cross-region DR copies
   - Complete system restore capability
   - RTO requirements <30 minutes
   - Full configuration preservation

### ❌ Don't Use Machine Images When:

1. **Only Boot Disk Matters**

   - Use disk images instead
   - Faster creation and restore
   - Lower cost
   - No configuration preservation needed

2. **Incremental Backups Needed**

   - Use snapshots instead
   - Incremental, space-efficient
   - Faster backup cycles
   - Better for frequent backups

3. **Single Disk Backup**

   - Use disk snapshots
   - More efficient
   - Faster operations
   - Better for databases with single disk

4. **Container Workloads**

   - Use Container Registry
   - Images for Docker/Kubernetes
   - Not VM-based deployments

## Machine Image Operations

### Create Machine Images

**From Existing VM:**

```bash
# Basic machine image creation
gcloud compute machine-images create my-machine-image \
  --source-instance=my-vm \
  --source-instance-zone=us-central1-a

# With description and labels
gcloud compute machine-images create prod-backup \
  --source-instance=prod-vm \
  --source-instance-zone=us-central1-a \
  --description="Production VM backup - $(date +%Y-%m-%d)" \
  --labels=environment=production,backup=daily

# Create in different storage location
gcloud compute machine-images create my-image \
  --source-instance=my-vm \
  --source-instance-zone=us-central1-a \
  --storage-location=us-west1

# Create from running VM (includes active memory)
gcloud compute machine-images create live-backup \
  --source-instance=my-vm \
  --source-instance-zone=us-central1-a \
  --guest-flush
```

**From Instance Template:**

```bash
# Create machine image from instance template
gcloud compute machine-images create template-image \
  --source-instance-template=my-template \
  --storage-location=us
```

### Create VMs from Machine Images

**Basic Creation:**

```bash
# Create VM from machine image
gcloud compute instances create restored-vm \
  --zone=us-central1-a \
  --source-machine-image=my-machine-image

# Create in different zone
gcloud compute instances create vm-clone \
  --zone=europe-west1-b \
  --source-machine-image=my-machine-image

# Override machine type
gcloud compute instances create bigger-vm \
  --zone=us-central1-a \
  --source-machine-image=my-machine-image \
  --machine-type=n2-standard-8
```

**With Overrides:**

```bash
# Create VM with custom configuration
gcloud compute instances create custom-vm \
  --zone=us-central1-a \
  --source-machine-image=my-machine-image \
  --machine-type=n2-standard-8 \
  --custom-cpu=8 \
  --custom-memory=32GB \
  --network=my-vpc \
  --subnet=my-subnet \
  --no-address

# Add metadata
gcloud compute instances create vm-with-metadata \
  --zone=us-central1-a \
  --source-machine-image=my-machine-image \
  --metadata=startup-script='#!/bin/bash
    echo "Additional configuration" >> /var/log/startup.log'
```

### List and Describe Machine Images

```bash
# List all machine images
gcloud compute machine-images list

# List with specific format
gcloud compute machine-images list \
  --format="table(
    name,
    sourceInstance.basename(),
    storageLocations[0],
    creationTimestamp.date()
  )"

# Describe machine image
gcloud compute machine-images describe my-machine-image

# Get details in JSON
gcloud compute machine-images describe my-machine-image \
  --format=json
```

### Delete Machine Images

```bash
# Delete machine image
gcloud compute machine-images delete my-machine-image

# Delete multiple images
gcloud compute machine-images delete image1 image2 image3

# Delete with confirmation skip
gcloud compute machine-images delete my-machine-image --quiet
```

## Working with Disk Images

### Create Disk Images

**From Disk:**

```bash
# Create image from disk
gcloud compute images create my-disk-image \
  --source-disk=my-disk \
  --source-disk-zone=us-central1-a

# Create from disk with family
gcloud compute images create ubuntu-2204-custom \
  --source-disk=custom-boot-disk \
  --source-disk-zone=us-central1-a \
  --family=ubuntu-2204-custom \
  --description="Custom Ubuntu 22.04 with company tools"
```

**From Snapshot:**

```bash
# Create image from snapshot
gcloud compute images create my-image \
  --source-snapshot=my-snapshot

# Create in image family
gcloud compute images create app-v1-2-3 \
  --source-snapshot=app-snapshot \
  --family=app-base
```

**From Existing Image:**

```bash
# Clone/copy image to different project
gcloud compute images create my-image-copy \
  --source-image=source-image \
  --source-image-project=source-project
```

**From Raw Disk File:**

```bash
# Upload raw disk to Cloud Storage first
gsutil cp disk.raw gs://my-bucket/

# Create image from raw disk
gcloud compute images create my-image \
  --source-uri=gs://my-bucket/disk.raw
```

### Image Families

**Concept**: Logical grouping of images where the latest image is used automatically.

```bash
# Create image in family
gcloud compute images create web-server-v2 \
  --source-disk=my-disk \
  --source-disk-zone=us-central1-a \
  --family=web-server

# Use latest from family
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --image-family=web-server \
  --image-project=my-project

# List images in family
gcloud compute images list --filter="family:web-server"

# Deprecate old images
gcloud compute images deprecate web-server-v1 \
  --state=DEPRECATED \
  --replacement=web-server-v2
```

### Public Images

```bash
# List public images
gcloud compute images list --project=debian-cloud
gcloud compute images list --project=ubuntu-os-cloud
gcloud compute images list --project=centos-cloud
gcloud compute images list --project=rhel-cloud
gcloud compute images list --project=windows-cloud

# Use public image
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

### Sharing Images

**Share with Specific Project:**

```bash
# Grant image user role to another project
gcloud compute images add-iam-policy-binding my-image \
  --member="serviceAccount:SERVICE_ACCOUNT@other-project.iam.gserviceaccount.com" \
  --role="roles/compute.imageUser"

# Use in other project
gcloud compute instances create vm-from-shared \
  --zone=us-central1-a \
  --image=my-image \
  --image-project=original-project
```

**Make Image Public:**

```bash
# Make image publicly accessible
gcloud compute images add-iam-policy-binding my-image \
  --member="allAuthenticatedUsers" \
  --role="roles/compute.imageUser"

# Remove public access
gcloud compute images remove-iam-policy-binding my-image \
  --member="allAuthenticatedUsers" \
  --role="roles/compute.imageUser"
```

## Best Practices

### 1. Use Image Families for Versioning

```bash
# Create versioned images in family
gcloud compute images create app-base-20240101 \
  --source-disk=my-disk \
  --source-disk-zone=us-central1-a \
  --family=app-base \
  --labels=version=20240101

# Always use family in instance templates
gcloud compute instance-templates create app-template \
  --machine-type=n2-standard-4 \
  --image-family=app-base \
  --image-project=my-project
```

### 2. Regular Backup Schedule

```bash
# Create backup script
#!/bin/bash
DATE=$(date +%Y%m%d-%H%M%S)
gcloud compute machine-images create backup-${DATE} \
  --source-instance=prod-vm \
  --source-instance-zone=us-central1-a \
  --description="Automated backup"

# Schedule with Cloud Scheduler
gcloud scheduler jobs create http machine-image-backup \
  --schedule="0 2 * * *" \
  --uri="https://cloud-function-url" \
  --http-method=POST
```

### 3. Clean Up Old Images

```bash
# List old images
gcloud compute machine-images list \
  --filter="creationTimestamp<$(date -d '30 days ago' --iso-8601)" \
  --format="value(name)"

# Delete images older than 30 days
for image in $(gcloud compute machine-images list \
  --filter="creationTimestamp<$(date -d '30 days ago' --iso-8601)" \
  --format="value(name)"); do
  gcloud compute machine-images delete $image --quiet
done
```

### 4. Use Descriptive Names and Labels

```bash
# Good naming convention
gcloud compute machine-images create prod-web-20240101-0200 \
  --source-instance=prod-web-01 \
  --source-instance-zone=us-central1-a \
  --labels=environment=production,service=web,backup-type=scheduled
```

### 5. Store in Appropriate Location

```bash
# Regional storage for fast restore
gcloud compute machine-images create regional-backup \
  --source-instance=my-vm \
  --source-instance-zone=us-central1-a \
  --storage-location=us-central1

# Multi-regional for DR
gcloud compute machine-images create dr-backup \
  --source-instance=my-vm \
  --source-instance-zone=us-central1-a \
  --storage-location=us

# Specific region for compliance
gcloud compute machine-images create eu-backup \
  --source-instance=my-vm \
  --source-instance-zone=europe-west1-b \
  --storage-location=eu
```

### 6. Test Restore Procedures

```bash
# Regularly test restoring from machine images
# Create test VM in separate network
gcloud compute instances create restore-test \
  --zone=us-central1-a \
  --source-machine-image=latest-backup \
  --network=test-vpc \
  --subnet=test-subnet

# Verify functionality
# Delete test VM after verification
```

### 7. Encrypt Custom Images

```bash
# Create image with customer-managed encryption
gcloud compute images create encrypted-image \
  --source-disk=my-disk \
  --source-disk-zone=us-central1-a \
  --kms-key=projects/PROJECT_ID/locations/us-central1/keyRings/my-keyring/cryptoKeys/my-key
```

## Image Deprecation

### Deprecation States

**ACTIVE**: Image is available for use (default)
**DEPRECATED**: Image is available but not recommended
**OBSOLETE**: Image is available but strongly discouraged
**DELETED**: Image is no longer available

### Manage Image Lifecycle

```bash
# Deprecate old image
gcloud compute images deprecate old-image \
  --state=DEPRECATED \
  --replacement=new-image

# Make image obsolete
gcloud compute images deprecate very-old-image \
  --state=OBSOLETE

# Delete deprecated images (manual)
gcloud compute images delete old-image

# Un-deprecate image
gcloud compute images deprecate image-name \
  --state=ACTIVE
```

## Automation and Scheduling

### Automated Backup Script

```bash
#!/bin/bash
# backup-vm.sh

PROJECT_ID="my-project"
INSTANCE_NAME="prod-vm"
ZONE="us-central1-a"
RETENTION_DAYS=30

# Create machine image with timestamp
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
IMAGE_NAME="${INSTANCE_NAME}-backup-${TIMESTAMP}"

echo "Creating machine image: ${IMAGE_NAME}"
gcloud compute machine-images create ${IMAGE_NAME} \
  --project=${PROJECT_ID} \
  --source-instance=${INSTANCE_NAME} \
  --source-instance-zone=${ZONE} \
  --description="Automated backup created on ${TIMESTAMP}"

# Delete images older than retention period
CUTOFF_DATE=$(date -d "${RETENTION_DAYS} days ago" --iso-8601)
OLD_IMAGES=$(gcloud compute machine-images list \
  --project=${PROJECT_ID} \
  --filter="name~'${INSTANCE_NAME}-backup' AND creationTimestamp<${CUTOFF_DATE}" \
  --format="value(name)")

for IMAGE in ${OLD_IMAGES}; do
  echo "Deleting old image: ${IMAGE}"
  gcloud compute machine-images delete ${IMAGE} \
    --project=${PROJECT_ID} \
    --quiet
done
```

### Schedule with Cloud Scheduler

```bash
# Create Cloud Function to handle backup
# Then schedule it
gcloud scheduler jobs create http daily-vm-backup \
  --schedule="0 2 * * *" \
  --time-zone="America/New_York" \
  --uri="https://REGION-PROJECT_ID.cloudfunctions.net/backup-vm" \
  --http-method=POST \
  --oidc-service-account-email="scheduler@PROJECT_ID.iam.gserviceaccount.com"
```

## Troubleshooting

### Image Creation Fails

```bash
# Check instance status
gcloud compute instances describe my-vm --zone=us-central1-a

# Verify disk status
gcloud compute disks list --filter="users:my-vm"

# Check for active operations
gcloud compute operations list --filter="targetLink:my-vm"

# Ensure sufficient quota
gcloud compute project-info describe --project=PROJECT_ID
```

### Cannot Create VM from Image

```bash
# Verify image exists
gcloud compute machine-images describe my-image

# Check IAM permissions
gcloud compute machine-images get-iam-policy my-image

# Verify target zone supports image features
gcloud compute zones describe us-central1-a
```

### Slow Image Operations

```bash
# Images with many/large disks take longer
# Check total disk size
gcloud compute machine-images describe my-image \
  --format="value(sourceInstance.disks[].diskSizeGb)"

# Use regional storage for faster access
# Consider using snapshots for individual disks instead
```

## Related Resources

- [Compute Engine Overview](compute-engine-overview.md)
- [Virtual Machines](compute-engine-vms.md)
- [Persistent Disks](compute-engine-disks.md)
- [Snapshots](compute-engine-snapshots.md)
- [Backups](compute-engine-backups.md)
- [gcloud CLI](gcloud-cli.md)
