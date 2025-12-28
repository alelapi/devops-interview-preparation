# Compute Engine Virtual Machines (VMs)

## Description

A Compute Engine Virtual Machine (VM) instance is a virtualized server running on Google's infrastructure. Each VM runs a guest operating system and can be configured with varying amounts of CPU, memory, storage, and networking resources to match your workload requirements.

**Concept**: Virtualized compute resources with customizable configurations, providing full control over the operating system and applications.

## Key Features

### Machine Types

**Predefined Machine Types**

- **E2**: Cost-optimized (2-32 vCPUs, 0.5-128 GB RAM)
- **N2**: Balanced performance (2-128 vCPUs, 0.5-864 GB RAM)
- **N2D**: AMD-based (2-224 vCPUs, 0.5-896 GB RAM)
- **N1**: Previous generation (1-96 vCPUs, 0.9-624 GB RAM)
- **C2**: Compute-optimized (4-60 vCPUs, 16-240 GB RAM)
- **C2D**: AMD compute-optimized (2-112 vCPUs, 4-448 GB RAM)
- **M1**: Memory-optimized (40-160 vCPUs, 961-3844 GB RAM)
- **M2**: Ultra-memory (208-416 vCPUs, 5888-11776 GB RAM)
- **A2**: GPU-accelerated (12-96 vCPUs with NVIDIA A100)

**Custom Machine Types**

- Create VMs with custom CPU and memory
- 1 vCPU to 96 vCPUs (N2) or 224 vCPUs (N2D)
- 0.9 GB to 8 GB RAM per vCPU
- Extended memory up to 624 GB total

### Operating Systems

**Linux Distributions:**

- Debian (10, 11, 12)
- Ubuntu (18.04, 20.04, 22.04, 24.04)
- CentOS Stream 8, 9
- Rocky Linux 8, 9
- RHEL 7, 8, 9
- SLES 12, 15
- Fedora CoreOS
- Container-Optimized OS

**Windows:**

- Windows Server 2016, 2019, 2022
- SQL Server on Windows (various versions)
- Bring Your Own License (BYOL) support

### VM Lifecycle States

**PROVISIONING**: Resources being allocated
**STAGING**: Resources acquired, preparing to boot
**RUNNING**: VM is running
**STOPPING**: Shutting down
**TERMINATED**: VM is stopped
**SUSPENDING**: Being suspended (beta)
**SUSPENDED**: VM suspended to disk (beta)
**REPAIRING**: Under repair

### Instance Options

**Spot VMs (Preemptible)**

- Up to 91% discount
- Can be terminated any time with 30-second warning
- Maximum 24-hour runtime
- No live migration
- Best for fault-tolerant workloads

**Shielded VMs**

- Secure Boot: Verify bootloader integrity
- vTPM: Virtual Trusted Platform Module
- Integrity Monitoring: Alert on boot-time changes
- Protection against rootkits and bootkits

**Confidential VMs**

- Memory encryption in use
- Hardware-based isolation
- No Google access to data in memory
- Based on AMD SEV technology

**Sole-Tenant Nodes**

- Dedicated physical servers
- Workload separation for compliance
- BYOL licensing for per-core software
- Node groups for resource pooling

## Important Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Instances per project** | 24 per region (default) | Can be increased |
| **vCPUs per project** | 24 per region (default) | Varies by family |
| **Custom machine vCPUs** | 1-96 (N2), 1-224 (N2D) | Based on family |
| **Memory per vCPU** | 0.9-8 GB | Extended up to 624 GB total |
| **Network interfaces** | 8 maximum | Per VM |
| **Metadata** | 512 KB | Key-value pairs |
| **Attached disks** | 128 | Including boot disk |
| **GPUs per VM** | Varies by GPU type | A2: 1-16 A100 GPUs |

## When to Use Different Machine Types

### E2 (Cost-Optimized)

✅ **Use For:**

- Development and testing environments
- Low-traffic web servers
- Small databases
- Microservices
- Batch processing jobs

❌ **Don't Use For:**

- High-performance computing
- Memory-intensive applications
- Sustained high CPU usage
- GPU workloads

### N2/N2D (General Purpose)

✅ **Use For:**

- Web and application servers
- Medium to large databases
- Cache servers
- Enterprise applications
- Most production workloads

❌ **Don't Use For:**

- Highest single-thread performance (use C2)
- Very large memory requirements (use M-series)
- GPU workloads (use A2)

### C2/C2D (Compute-Optimized)

✅ **Use For:**

- High-performance computing (HPC)
- Gaming servers
- Ad serving
- High-traffic web serving
- Media transcoding
- CPU-intensive simulations

❌ **Don't Use For:**

- Memory-intensive workloads
- GPU-accelerated workloads
- Cost-sensitive development

### M1/M2/M3 (Memory-Optimized)

✅ **Use For:**

- Large in-memory databases (SAP HANA, Redis)
- In-memory analytics
- Microsoft SQL Server
- Real-time big data processing
- High-performance relational databases

❌ **Don't Use For:**

- Compute-intensive tasks
- Cost-sensitive workloads
- Small to medium applications

### A2/A3 (GPU-Accelerated)

✅ **Use For:**

- Machine learning training
- Deep learning inference
- High-performance computing
- Rendering and visualization
- Scientific simulations

❌ **Don't Use For:**

- General-purpose computing
- CPU-only workloads
- Cost-sensitive applications

## VM Configuration Options

### CPU Platform

Specify minimum CPU platform:

```bash
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --min-cpu-platform="Intel Ice Lake"
```

**Available Platforms:**

- Intel Cascade Lake
- Intel Ice Lake
- Intel Sapphire Rapids
- AMD Milan
- AMD Genoa

### Boot Disk Options

**Disk Types:**

- **pd-standard**: Standard HDD (cheaper, slower)
- **pd-balanced**: Balanced SSD (recommended)
- **pd-ssd**: High-performance SSD
- **pd-extreme**: Highest performance (custom IOPS)
- **hyperdisk-balanced**: Next-gen balanced performance
- **hyperdisk-extreme**: Next-gen extreme performance

**Size Considerations:**

- Minimum: 10 GB
- Recommended: 20+ GB for OS and updates
- Performance scales with size (for pd-standard, pd-balanced, pd-ssd)

```bash
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --boot-disk-size=50GB \
  --boot-disk-type=pd-balanced
```

### Network Configuration

**Network Interfaces:**

```bash
# Single network interface
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --network=my-vpc \
  --subnet=my-subnet \
  --private-network-ip=10.0.1.10

# No external IP (private only)
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --network=my-vpc \
  --subnet=my-subnet \
  --no-address

# Multiple network interfaces
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --network-interface=network=vpc1,subnet=subnet1 \
  --network-interface=network=vpc2,subnet=subnet2
```

### Metadata and Startup Scripts

**Metadata:**

```bash
# Single metadata item
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --metadata=environment=production,team=platform

# Metadata from file
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --metadata-from-file=startup-script=startup.sh
```

**Startup Script Example:**

```bash
#!/bin/bash

# Update packages
apt-get update
apt-get upgrade -y

# Install web server
apt-get install -y nginx

# Configure application
cat > /etc/nginx/sites-available/default <<'EOF'
server {
    listen 80 default_server;
    location / {
        return 200 'Hello from GCE!\n';
        add_header Content-Type text/plain;
    }
}
EOF

# Start service
systemctl restart nginx
systemctl enable nginx
```

### Service Accounts

**Default Service Account:**

```bash
# Use default compute service account
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --scopes=cloud-platform
```

**Custom Service Account:**

```bash
# Create custom service account
gcloud iam service-accounts create vm-sa \
  --display-name="VM Service Account"

# Grant permissions
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:vm-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/compute.instanceAdmin.v1"

# Create VM with custom SA
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --service-account=vm-sa@PROJECT_ID.iam.gserviceaccount.com \
  --scopes=cloud-platform
```

## VM Operations

### Create VM Instances

**Basic Creation:**

```bash
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=e2-medium
```

**Production VM:**

```bash
gcloud compute instances create prod-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=50GB \
  --boot-disk-type=pd-balanced \
  --network=my-vpc \
  --subnet=my-subnet \
  --no-address \
  --tags=http-server,https-server \
  --labels=env=production,team=platform \
  --service-account=vm-sa@PROJECT_ID.iam.gserviceaccount.com \
  --scopes=cloud-platform \
  --metadata-from-file=startup-script=startup.sh \
  --shielded-secure-boot \
  --shielded-vtpm \
  --shielded-integrity-monitoring
```

**Spot VM:**

```bash
gcloud compute instances create spot-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --preemptible \
  --instance-termination-action=DELETE
```

### Manage VM Instances

**List and Describe:**

```bash
# List all instances
gcloud compute instances list

# List in specific zone
gcloud compute instances list --zones=us-central1-a

# Describe instance
gcloud compute instances describe my-vm --zone=us-central1-a

# Get instance details in JSON
gcloud compute instances describe my-vm \
  --zone=us-central1-a \
  --format=json
```

**Start, Stop, Reset:**

```bash
# Stop VM (saves disk state)
gcloud compute instances stop my-vm --zone=us-central1-a

# Start VM
gcloud compute instances start my-vm --zone=us-central1-a

# Reset VM (hard reboot)
gcloud compute instances reset my-vm --zone=us-central1-a

# Suspend VM (beta)
gcloud compute instances suspend my-vm --zone=us-central1-a

# Resume VM (beta)
gcloud compute instances resume my-vm --zone=us-central1-a
```

**Update VM Configuration:**

```bash
# Change machine type (VM must be stopped)
gcloud compute instances stop my-vm --zone=us-central1-a
gcloud compute instances set-machine-type my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-8
gcloud compute instances start my-vm --zone=us-central1-a

# Add metadata
gcloud compute instances add-metadata my-vm \
  --zone=us-central1-a \
  --metadata=new-key=new-value

# Update labels
gcloud compute instances update my-vm \
  --zone=us-central1-a \
  --update-labels=environment=staging

# Add network tags
gcloud compute instances add-tags my-vm \
  --zone=us-central1-a \
  --tags=web-server,backend
```

**Delete VM:**

```bash
# Delete instance
gcloud compute instances delete my-vm --zone=us-central1-a

# Delete instance but keep boot disk
gcloud compute instances delete my-vm \
  --zone=us-central1-a \
  --keep-disks=boot
```

### Access VM Instances

**SSH Access:**

```bash
# SSH with gcloud (recommended)
gcloud compute ssh my-vm --zone=us-central1-a

# SSH with specific user
gcloud compute ssh username@my-vm --zone=us-central1-a

# SSH with custom SSH key
gcloud compute ssh my-vm \
  --zone=us-central1-a \
  --ssh-key-file=~/.ssh/my-key

# Run command via SSH
gcloud compute ssh my-vm \
  --zone=us-central1-a \
  --command="sudo systemctl status nginx"
```

**Serial Console:**

```bash
# Connect to serial console
gcloud compute connect-to-serial-port my-vm \
  --zone=us-central1-a

# View serial port output
gcloud compute instances get-serial-port-output my-vm \
  --zone=us-central1-a
```

**RDP (Windows VMs):**

```bash
# Get Windows password
gcloud compute reset-windows-password my-windows-vm \
  --zone=us-central1-a \
  --user=admin
```

## Best Practices

### 1. Right-Sizing VMs

```bash
# Use Cloud Monitoring recommender
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --location=us-central1-a \
  --recommender=google.compute.instance.MachineTypeRecommender

# Start small and scale up
# Monitor: CPU, memory, disk I/O, network
# Adjust machine type based on actual usage
```

### 2. Use Managed Instance Groups

```bash
# Create instance template
gcloud compute instance-templates create web-template \
  --machine-type=n2-standard-4 \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=20GB \
  --tags=http-server

# Create MIG
gcloud compute instance-groups managed create web-mig \
  --base-instance-name=web \
  --template=web-template \
  --size=3 \
  --zone=us-central1-a

# Enable autoscaling
gcloud compute instance-groups managed set-autoscaling web-mig \
  --zone=us-central1-a \
  --max-num-replicas=10 \
  --min-num-replicas=2 \
  --target-cpu-utilization=0.6
```

### 3. Use Startup and Shutdown Scripts

**Startup Script:**

```bash
#!/bin/bash
set -e

# Wait for network
while ! ping -c 1 google.com &> /dev/null; do
    echo "Waiting for network..."
    sleep 1
done

# Application setup
apt-get update
apt-get install -y nginx
systemctl start nginx
```

**Shutdown Script:**

```bash
#!/bin/bash
# Save state before shutdown
pg_dump mydb > /var/backups/db_backup.sql
gsutil cp /var/backups/db_backup.sql gs://my-bucket/backups/
```

### 4. Implement Health Checks

```bash
# Create health check
gcloud compute health-checks create http http-health-check \
  --port=80 \
  --check-interval=10s \
  --timeout=5s \
  --healthy-threshold=2 \
  --unhealthy-threshold=3

# Create firewall rule for health checks
gcloud compute firewall-rules create allow-health-check \
  --network=my-vpc \
  --action=allow \
  --direction=ingress \
  --source-ranges=35.191.0.0/16,130.211.0.0/22 \
  --rules=tcp:80
```

### 5. Use Labels and Tags

```bash
# Labels for organization and billing
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --labels=env=production,team=platform,cost-center=engineering

# Tags for firewall rules
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --tags=web-server,backend-service
```

### 6. Enable Monitoring and Logging

```bash
# Enable Cloud Monitoring
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --scopes=cloud-platform

# Install monitoring agent (on VM)
curl -sSO https://dl.google.com/cloudagents/add-google-cloud-ops-agent-repo.sh
sudo bash add-google-cloud-ops-agent-repo.sh --also-install

# Configure ops agent
sudo tee /etc/google-cloud-ops-agent/config.yaml > /dev/null <<EOF
logging:
  receivers:
    syslog:
      type: files
      include_paths:

      - /var/log/syslog
  service:
    pipelines:
      default_pipeline:
        receivers: [syslog]
metrics:
  receivers:
    hostmetrics:
      type: hostmetrics
  service:
    pipelines:
      default_pipeline:
        receivers: [hostmetrics]
EOF

sudo systemctl restart google-cloud-ops-agent
```

### 7. Security Hardening

```bash
# Use OS Login instead of SSH keys
gcloud compute instances add-metadata my-vm \
  --zone=us-central1-a \
  --metadata=enable-oslogin=TRUE

# Enable Shielded VM features
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --shielded-secure-boot \
  --shielded-vtpm \
  --shielded-integrity-monitoring

# No external IP
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --no-address

# Minimal IAM scopes
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --scopes=logging-write,monitoring-write
```

## Troubleshooting

### Instance Won't Start

```bash
# Check serial console output
gcloud compute instances get-serial-port-output my-vm \
  --zone=us-central1-a

# Check operations logs
gcloud compute operations list \
  --filter="targetLink:my-vm"

# Verify quotas
gcloud compute project-info describe \
  --project=PROJECT_ID
```

### High CPU or Memory Usage

```bash
# Check metrics in Cloud Monitoring
gcloud monitoring time-series list \
  --filter='metric.type="compute.googleapis.com/instance/cpu/utilization"
    AND resource.labels.instance_id="INSTANCE_ID"' \
  --format=json

# SSH and investigate
gcloud compute ssh my-vm --zone=us-central1-a
top
free -m
df -h
```

### Network Connectivity Issues

```bash
# Check firewall rules
gcloud compute firewall-rules list \
  --filter="network:my-vpc"

# Test connectivity from VM
gcloud compute ssh my-vm --zone=us-central1-a
ping 8.8.8.8
curl https://www.google.com

# Check routes
gcloud compute routes list \
  --filter="network:my-vpc"
```

## Related Resources

- [Compute Engine Overview](compute-engine-overview.md)
- [Persistent Disks](compute-engine-disks.md)
- [Machine Images](compute-engine-images.md)
- [Snapshots](compute-engine-snapshots.md)
- [Backups](compute-engine-backups.md)
- [gcloud CLI](gcloud-cli.md)
