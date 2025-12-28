# GKE Overview

## Description

Google Kubernetes Engine (GKE) is a managed Kubernetes service that provides a platform for deploying, managing, and scaling containerized applications using Google's infrastructure. GKE abstracts away the complexity of managing Kubernetes control planes and provides deep integration with Google Cloud services.

**Architecture**: Managed Kubernetes clusters consisting of a control plane (managed by Google) and worker nodes (managed by Google or you, depending on cluster type).

## Key Features

### Managed Control Plane

- **Fully Managed**: Google manages the Kubernetes API server, etcd, and other control plane components
- **Automatic Updates**: Control plane automatically updated with latest Kubernetes versions
- **High Availability**: Multi-zone control plane with 99.95% or 99.99% SLA (depending on cluster type)
- **No Control Plane Costs**: Standard clusters don't charge for control plane (Autopilot includes it in per-pod pricing)

### Cluster Management

- **Multiple Cluster Types**: Standard (manual management) and Autopilot (fully managed)
- **Auto-Scaling**: Cluster autoscaling for nodes, Horizontal/Vertical Pod Autoscaling
- **Auto-Repair**: Automatically repairs unhealthy nodes
- **Auto-Upgrade**: Automatically upgrades nodes to match control plane version
- **Node Pools**: Logical grouping of nodes with same configuration

### Integration with Google Cloud

- **Cloud Load Balancing**: Automatic integration for Service type LoadBalancer
- **Cloud Logging & Monitoring**: Native integration with Cloud Operations
- **Workload Identity**: Securely access Google Cloud APIs from pods
- **Binary Authorization**: Ensure only trusted container images are deployed
- **VPC-Native Clusters**: Pods get IP addresses from VPC subnet ranges
- **Private Clusters**: Control plane and nodes isolated from public internet

### Security

- **Workload Identity**: Map Kubernetes service accounts to Google Cloud service accounts
- **Shielded GKE Nodes**: Verifiable node integrity
- **Binary Authorization**: Deploy-time security policy enforcement
- **GKE Sandbox**: Run untrusted workloads using gVisor
- **Security Posture Dashboard**: Centralized security recommendations
- **Network Policies**: Control pod-to-pod communication

### Developer Experience

- **Cloud Code**: IDE integration for development and debugging
- **Config Connector**: Manage GCP resources through Kubernetes
- **kubectl**: Standard Kubernetes CLI
- **Cloud Console UI**: Web-based cluster management
- **Cloud Shell**: Browser-based CLI with pre-installed tools

## Important Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Max nodes per cluster** | 15,000 (Standard), 1,000 node pools | Autopilot scales automatically |
| **Max pods per node** | 110 (default), up to 256 | Configurable with `--max-pods-per-node` |
| **Max pods per cluster** | 200,000 (Standard) | Autopilot manages this automatically |
| **Clusters per project** | 100 per location | Soft limit, can be increased |
| **Node pools per cluster** | 1,000 | Each pool can have different configurations |
| **Max PVs per cluster** | 256 PD per node, 128 local SSDs per node | Persistent disk limits |
| **Services (LoadBalancer)** | 5 per node, 300 per cluster | Network load balancer limits |

## Cluster Types Comparison

| Feature | Standard GKE | Autopilot GKE |
|---------|--------------|---------------|
| **Node Management** | Manual | Fully automated |
| **Pricing Model** | Per node-hour | Per pod resource request |
| **Scaling** | Configure autoscaling | Automatic |
| **Node Configuration** | Full control | Google-managed |
| **SSH Access to Nodes** | Yes | No |
| **Custom Machine Types** | Yes | Predefined pod specs |
| **Node Pools** | Manual creation | Automatically managed |
| **Security Baseline** | Configure manually | Hardened by default |
| **Best For** | Custom requirements, full control | Simplicity, hands-off operations |

## When to Use

### ✅ Use GKE When:

1. **Container Orchestration Needed**

   - Running microservices architectures
   - Need automatic scaling, self-healing, and rolling updates
   - Managing multiple containerized applications

2. **Kubernetes Expertise Available**

   - Team has Kubernetes knowledge
   - Want standard Kubernetes APIs and ecosystem
   - Need portability across clouds

3. **Google Cloud Integration Required**

   - Leveraging Google Cloud services (Cloud SQL, Pub/Sub, BigQuery)
   - Using Workload Identity for secure GCP access
   - Need integration with Cloud Load Balancing

4. **High Availability and Scalability**

   - Applications require 99.95%+ uptime
   - Need to scale from handful to thousands of pods
   - Multi-region or multi-zone deployments

5. **Managed Infrastructure Preferred**

   - Want Google to manage control plane
   - Prefer automatic updates and patches
   - Need security hardening by default (Autopilot)

### ❌ Don't Use GKE When:

1. **Simple Applications**

   - Single container application better suited for Cloud Run
   - Serverless functions (use Cloud Functions)
   - Static websites (use Cloud Storage/Firebase Hosting)

2. **No Container Experience**

   - Team lacks container and Kubernetes knowledge
   - Learning curve not justified by requirements
   - Simpler solutions available (App Engine, Cloud Run)

3. **Windows-Heavy Workloads**

   - Primarily Windows containers (GKE supports Windows but consider GCE)
   - Legacy Windows applications not containerized

4. **Extremely Cost-Sensitive Small Workloads**

   - Single small VM might be cheaper than minimum cluster
   - Very low traffic applications
   - Development environments (unless Autopilot)

5. **Complete Infrastructure Control Needed**

   - Need to modify control plane configuration
   - Require kernel-level modifications
   - Custom network overlays incompatible with GKE

## Common Use Cases

### Microservices Architecture

```
GKE Cluster
├── Frontend Service (Deployment)
│   └── Pods: 3 replicas
├── API Service (Deployment)
│   └── Pods: 5 replicas
├── Auth Service (Deployment)
│   └── Pods: 2 replicas
└── Database (StatefulSet)
    └── Pods: 3 replicas (with persistent volumes)

Ingress: HTTPS load balancer
Service Mesh: Istio for service-to-service communication
```

### CI/CD Pipeline

```
GitHub → Cloud Build → Artifact Registry → GKE
                                            ├── Dev Cluster
                                            ├── Staging Cluster
                                            └── Production Cluster
```

### Batch Processing

```
GKE Autopilot
└── Jobs/CronJobs
    ├── Data processing jobs (scales to zero when complete)
    ├── ML training jobs
    └── ETL pipelines
```

## GKE vs Other Google Services

**GKE vs Cloud Run**

- GKE: Full Kubernetes, more control, stateful workloads
- Cloud Run: Serverless containers, simpler, stateless HTTP services

**GKE vs Compute Engine**

- GKE: Container orchestration, automatic scaling/healing
- Compute Engine: Full VM control, traditional applications

**GKE vs App Engine**

- GKE: More flexibility, any language/runtime, complex apps
- App Engine: Simpler PaaS, limited languages, quick deployment

## Pricing Considerations

**Standard GKE**

- **Control Plane**: Free for zonal clusters, $0.10/hour for regional
- **Nodes**: Standard Compute Engine pricing for VMs
- **Network**: Egress charges apply
- **Cost Optimization**: Use Spot VMs, committed use discounts, right-size nodes

**Autopilot GKE**

- **No Node Charges**: Pay only for pod resource requests
- **Control Plane**: Included in pod pricing
- **vCPU**: $0.0445/hour per vCPU requested
- **Memory**: $0.00488/hour per GB requested
- **Cost Optimization**: Right-size pod requests, use vertical pod autoscaler

**General Tips**

- Use Autopilot for unpredictable workloads (pay only for used resources)
- Use Standard with Spot VMs for batch/fault-tolerant workloads (up to 91% discount)
- Enable cluster autoscaling to scale down during off-hours
- Use resource quotas to prevent cost overruns

## Getting Started

### Create a GKE Cluster (Standard)

```bash
# Create zonal cluster
gcloud container clusters create my-cluster \
  --zone=us-central1-a \
  --num-nodes=3 \
  --machine-type=e2-medium \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10

# Get credentials
gcloud container clusters get-credentials my-cluster --zone=us-central1-a

# Verify connection
kubectl get nodes
```

### Create a GKE Cluster (Autopilot)

```bash
# Create Autopilot cluster (regional by default)
gcloud container clusters create-auto my-autopilot-cluster \
  --region=us-central1

# Get credentials
gcloud container clusters get-credentials my-autopilot-cluster --region=us-central1

# Deploy application - nodes provisioned automatically
kubectl apply -f deployment.yaml
```

## Best Practices

### 1. Cluster Configuration

- Use regional clusters for production (99.95% SLA)
- Enable Workload Identity for secure GCP access
- Use VPC-native clusters (alias IP ranges)
- Enable Binary Authorization for production
- Configure maintenance windows for upgrades

### 2. Security

- Enable Workload Identity (not metadata server)
- Use least-privilege IAM roles
- Implement Network Policies
- Use Private clusters for sensitive workloads
- Enable GKE Dataplane V2 for improved networking

### 3. Resource Management

- Set resource requests and limits on all pods
- Use resource quotas and limit ranges per namespace
- Enable Horizontal Pod Autoscaler for variable workloads
- Use Vertical Pod Autoscaler to right-size requests

### 4. Monitoring & Logging

- Enable GKE monitoring and logging (now default)
- Use Workload metrics for application-level monitoring
- Set up alerts for cluster and pod health
- Use Cloud Trace for distributed tracing

### 5. Cost Optimization

- Use Spot VMs for fault-tolerant workloads
- Enable cluster autoscaling
- Right-size node pools and pod resources
- Use Autopilot for variable or unpredictable workloads
- Clean up unused resources (PVs, LoadBalancers)

