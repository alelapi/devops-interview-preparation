# Anthos

## Core Concepts

Anthos is a hybrid and multi-cloud application platform built on Kubernetes. Enables consistent development and operations across on-premises, GCP, AWS, and Azure.

**Key Principle**: Write once, run anywhere; consistent K8s experience across environments.

## Anthos Components

| Component | Purpose | Key Feature |
|-----------|---------|-------------|
| **GKE/Anthos clusters** | Kubernetes clusters | Anywhere (on-prem, clouds) |
| **Anthos Config Management** | Policy and config | GitOps, multi-cluster sync |
| **Anthos Service Mesh** | Service-to-service communication | Traffic management, security |
| **Cloud Run for Anthos** | Serverless on K8s | Knative-based |
| **Anthos on VMware/Bare Metal** | On-premises K8s | Run on existing infrastructure |

## When to Use Anthos

### ✅ Use Anthos When

- Hybrid cloud strategy (on-prem + cloud)
- Multi-cloud deployment needed
- Existing on-premises infrastructure
- Kubernetes standardization across environments
- Modernizing VMs to containers gradually
- Need consistent policy enforcement
- Service mesh requirements

### ❌ Don't Use Anthos When

- GCP-only deployment → GKE sufficient
- Simple applications → Cloud Run
- No Kubernetes needed → App Engine, Cloud Functions
- Not ready for containerization → Compute Engine
- Budget-constrained (Anthos has licensing cost)

## Deployment Options

### GKE (Google Cloud)

**Benefits**: Fully managed, latest features, easiest

**Use case**: Cloud-native applications on GCP

### Anthos on VMware

**Architecture**: Run GKE on VMware vSphere

**Benefits**:

- Use existing VMware investment
- Keep data on-premises (compliance)
- Hybrid cloud connectivity

**Requirements**: vSphere 6.7+, sufficient resources

### Anthos on Bare Metal

**Architecture**: Run GKE directly on physical servers

**Benefits**:

- No hypervisor overhead
- Edge computing scenarios
- Cost savings (no VMware licenses)

**Requirements**: Qualified hardware, network configuration

### Anthos on AWS/Azure

**Purpose**: Consistent K8s across clouds

**Benefits**: Multi-cloud strategy, avoid lock-in

**Limitations**: Additional complexity, cost

## Anthos Config Management (ACM)

### Purpose

Centralized configuration and policy management for multiple clusters using GitOps.

### Key Features

**Policy Controller**:

- Enforce policies across clusters
- Based on Open Policy Agent (OPA)
- Examples: Require labels, restrict registries, enforce resource limits

**Config Sync**:

- Sync configs from Git to clusters
- Single source of truth
- Automatic reconciliation
- Namespace and cluster-scoped configs

### GitOps Workflow

```
Git Repository (configs) → Config Sync → Multiple clusters apply configs
```

**Benefits**: Version control, audit trail, declarative, automated

### Common Policies

- Require specific labels
- Enforce namespace quotas
- Restrict container registries
- Require pod security policies
- Enforce naming conventions

## Anthos Service Mesh (ASM)

### Purpose

Managed service mesh for observability, security, and traffic management.

### Architecture

Based on Istio, fully managed by Google

**Components**:

- **Control Plane**: Managed by Google
- **Data Plane**: Envoy sidecars in pods

### Features

**Traffic Management**:

- Load balancing
- Circuit breaking
- Retries and timeouts
- Canary deployments
- Traffic splitting

**Security**:

- mTLS between services (automatic)
- Authorization policies
- Service-to-service auth
- Certificate management

**Observability**:

- Service topology visualization
- Distributed tracing
- Metrics and logs
- Service-level objectives (SLOs)

### Use Cases

- Microservices communication
- Zero-trust security
- Canary deployments
- Service observability
- Multi-cluster service mesh

## Cloud Run for Anthos

**Purpose**: Serverless containers on your GKE clusters

**Benefits**:

- Knative-based
- Auto-scaling (including to zero)
- Simplified deployment
- Use existing GKE clusters

**vs Cloud Run**: Same developer experience, runs on your infrastructure

**Use case**: Serverless on-premises or in specific clusters

## Architecture Patterns

### Hybrid Application

```
On-prem (Anthos on VMware): Legacy systems + databases
GCP (GKE): Modern microservices
Connected via: VPN/Interconnect + Anthos Service Mesh
```

**Benefits**: Gradual migration, keep sensitive data on-prem

### Multi-Region HA

```
GKE Cluster (us-central1)
GKE Cluster (europe-west1)
GKE Cluster (asia-east1)
Managed by: Anthos Config Management
Service Mesh: Cross-cluster communication
```

### Edge Computing

```
Anthos on Bare Metal (retail stores/factories)
Central GKE (cloud)
Sync: Anthos Config Management
```

**Use case**: Low latency, local data processing, offline capability

### Multi-Cloud

```
GKE (GCP) + EKS via Anthos (AWS) + AKS via Anthos (Azure)
Unified: Config Management + Service Mesh
```

**Benefits**: Avoid vendor lock-in, geographic coverage, redundancy

## Migrate to Containers (M4C)

**Purpose**: Migrate VMs to containers running on GKE/Anthos

**Process**:

1. Discover and assess VMs
2. Generate migration plan
3. Convert VM to container
4. Deploy to GKE/Anthos
5. Optimize containerized workload

**Use case**: Modernize legacy applications, VM to container migration

## Security Features

### Binary Authorization

**Purpose**: Enforce only trusted container images deployed

**How**: Cryptographic signatures on images, attestation checks

**Use case**: Compliance, supply chain security

### Policy Controller

**Purpose**: Enforce organizational policies on clusters

**Examples**:

- Only approved container registries
- Required labels on resources
- Prohibited capabilities
- Resource quotas

### Service Mesh Security

**mTLS**: Automatic encryption between services
**Authorization**: Fine-grained access control
**Certificate Management**: Automated, no manual cert handling

### Workload Identity

**Purpose**: Kubernetes service accounts → Google service accounts

**Benefits**: Secure access to GCP services, no keys in pods

## Cost Considerations

**Anthos Licensing**:

- Per-vCPU pricing for on-premises
- Included with GKE on GCP
- Additional cost for multi-cloud

**Infrastructure Costs**:

- GKE: Standard GCP compute pricing
- On-premises: Your hardware + VMware licenses (if applicable)
- Network: Interconnect, VPN, egress

**Optimization**:

- Right-size clusters
- Use Autopilot GKE (managed)
- Optimize workload placement
- Monitor with recommendations

## Monitoring and Operations

### Cloud Monitoring

- Cluster metrics
- Application metrics
- Service mesh metrics
- Custom metrics

### Cloud Logging

- Cluster logs
- Application logs
- Audit logs
- Centralized logging

### Service Mesh Observability

- Service topology
- Request rates
- Latencies
- Error rates
- SLI/SLO tracking

## Exam Focus

### Core Concepts

- What is Anthos (hybrid/multi-cloud K8s platform)
- Components (GKE, Config Mgmt, Service Mesh)
- When to use vs GKE alone
- Deployment options (VMware, bare metal, clouds)

### Use Cases

- Hybrid cloud (on-prem + cloud)
- Multi-cloud strategy
- Edge computing
- Gradual VM-to-container migration

### Architecture

- Hybrid application design
- Multi-cluster management
- Service mesh benefits
- GitOps with Config Management

### Config Management

- Policy enforcement
- Config sync
- GitOps workflow
- Multi-cluster consistency

### Service Mesh

- mTLS between services
- Traffic management
- Observability
- Security policies

### Security

- Binary Authorization
- Policy Controller
- Workload Identity
- Zero-trust networking

### Migration

- Migrate to Containers (M4C)
- VM to container modernization
- Gradual migration strategy
