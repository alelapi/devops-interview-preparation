# Kubelet Configuration Exam Exercises

## Overview
This document contains comprehensive exam exercises for modifying and managing Kubelet configuration. Each exercise covers different aspects of Kubelet configuration management, from basic parameter changes to advanced cluster-wide configuration scenarios.

---

## Exercise 1: Basic Kubelet Configuration File Management

### Scenario
You need to modify the Kubelet configuration on a worker node to change basic operational parameters.

### Tasks
1. Locate the current Kubelet configuration file
2. Modify the following parameters:
   - Change the cluster DNS to `10.96.0.10`
   - Set the maximum number of pods per node to 200
   - Enable CPU and Memory manager policies
   - Configure log rotation settings

### Configuration File Example
```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: 0.0.0.0
port: 10250
serializeImagePulls: false
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
maxPods: 200
cpuManagerPolicy: static
memoryManagerPolicy: Static
logRotateMaxSize: 100Mi
logRotateMaxBackups: 5
logRotateMaxAge: 7
```

### Steps to Complete
1. Back up the existing configuration:
```bash
sudo cp /var/lib/kubelet/config.yaml /var/lib/kubelet/config.yaml.backup
```

2. Edit the configuration file:
```bash
sudo vim /var/lib/kubelet/config.yaml
```

3. Restart the Kubelet service:
```bash
sudo systemctl restart kubelet
```

4. Verify the changes:
```bash
sudo systemctl status kubelet
kubectl get nodes -o wide
```

### Expected Outcomes
- Kubelet restarts successfully with new configuration
- Node shows updated capacity and allocatable resources
- DNS resolution works with the new cluster DNS
- Pod limit is enforced according to new maxPods setting

---

## Exercise 2: Kubeadm Upgrade Node Phase Configuration

### Scenario
Use kubeadm to upgrade the kubelet configuration during a cluster upgrade process.

### Tasks
1. Perform kubelet configuration upgrade using kubeadm
2. Verify the upgraded configuration
3. Handle any conflicts between old and new configurations

### Pre-upgrade Preparation
```bash
# Check current kubelet version
kubelet --version

# Check current node configuration
kubectl get node $(hostname) -o yaml | grep kubeletVersion

# Backup current configuration
sudo cp /var/lib/kubelet/config.yaml /var/lib/kubelet/config.yaml.pre-upgrade
```

### Kubeadm Upgrade Process
```bash
# Download the new kubelet configuration from the cluster
sudo kubeadm upgrade node phase kubelet-config

# Restart kubelet to apply the new configuration
sudo systemctl restart kubelet

# Verify the upgrade
sudo systemctl status kubelet
kubectl get nodes
```

### Post-upgrade Verification
```bash
# Check if kubelet is running with new config
ps aux | grep kubelet

# Verify node readiness
kubectl describe node $(hostname)

# Check for any configuration differences
diff /var/lib/kubelet/config.yaml.pre-upgrade /var/lib/kubelet/config.yaml
```

### Expected Outcomes
- Kubelet configuration is updated to match cluster version
- Node remains in Ready state after upgrade
- No configuration conflicts exist
- Kubelet version matches the target cluster version

---

## Exercise 3: Resource Management Configuration

### Scenario
Configure Kubelet for optimal resource management in a high-performance computing environment.

### Tasks
1. Configure CPU Manager with static policy
2. Set up Memory Manager with static policy
3. Configure Topology Manager for NUMA awareness
4. Set up custom resource reservations for system components

### Advanced Resource Configuration
```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
# CPU Management
cpuManagerPolicy: static
cpuManagerPolicyOptions:
  full-pcpus-only: true
cpuManagerReconcilePeriod: 10s

# Memory Management
memoryManagerPolicy: Static

# Topology Management
topologyManagerPolicy: single-numa-node
topologyManagerScope: pod

# Resource Reservations
systemReserved:
  cpu: 500m
  memory: 1Gi
  ephemeral-storage: 2Gi
kubeReserved:
  cpu: 500m
  memory: 1Gi
  ephemeral-storage: 1Gi
evictionHard:
  memory.available: "100Mi"
  nodefs.available: "10%"
  nodefs.inodesFree: "5%"
  imagefs.available: "15%"

# Container Runtime
containerRuntimeEndpoint: unix:///var/run/containerd/containerd.sock
imageServiceEndpoint: unix:///var/run/containerd/containerd.sock

# Feature Gates
featureGates:
  CPUManager: true
  MemoryManager: true
  TopologyManager: true
  KubeletPodResources: true
```

### Pre-configuration Setup
```bash
# Clear CPU manager state (if changing policy)
sudo systemctl stop kubelet
sudo rm -f /var/lib/kubelet/cpu_manager_state
sudo rm -f /var/lib/kubelet/memory_manager_state

# Apply the new configuration
sudo cp new-kubelet-config.yaml /var/lib/kubelet/config.yaml

# Restart kubelet
sudo systemctl start kubelet
```

### Verification Commands
```bash
# Check CPU manager status
ls -la /var/lib/kubelet/cpu_manager_state
cat /var/lib/kubelet/cpu_manager_state

# Verify resource allocation
kubectl describe nodes

# Check feature gates
kubectl get --raw /api/v1/nodes/$(hostname)/proxy/configz | jq '.kubeletconfig.featureGates'
```

### Expected Outcomes
- CPU Manager allocates exclusive CPU cores to guaranteed pods
- Memory Manager provides NUMA-local memory allocation
- Topology Manager ensures NUMA affinity
- System and Kube reserved resources are properly allocated

---

## Exercise 4: Security and Authentication Configuration

### Scenario
Harden Kubelet security by configuring authentication, authorization, and TLS settings.

### Tasks
1. Enable Webhook authentication and authorization
2. Configure TLS certificate rotation
3. Set up admission controllers
4. Configure security contexts and AppArmor profiles

### Security Configuration
```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
# Authentication
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
    cacheTTL: 30s
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt

# Authorization
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m
    cacheUnauthorizedTTL: 30s

# TLS Configuration
tlsCertFile: /var/lib/kubelet/pki/kubelet.crt
tlsPrivateKeyFile: /var/lib/kubelet/pki/kubelet.key
rotateCertificates: true
serverTLSBootstrap: true

# Security Settings
protectKernelDefaults: true
makeIPTablesUtilChains: true
iptablesMasqueradeBit: 14
iptablesDropBit: 15

# Runtime Security
allowPrivileged: false
hostNetworkSources: []
hostPIDSources: []
hostIPCSources: []

# Admission Controllers
enableAdmissionPlugins:
- NamespaceLifecycle
- LimitRanger
- ServiceAccount
- DefaultStorageClass
- DefaultTolerationSeconds
- MutatingAdmissionWebhook
- ValidatingAdmissionWebhook
- ResourceQuota
- NodeRestriction
```

### Configuration Update Steps
```bash
# Apply the security configuration
sudo cp security-kubelet-config.yaml /var/lib/kubelet/config.yaml

# Restart kubelet to apply changes
sudo systemctl restart kubelet

# Verify security settings are active
sudo systemctl status kubelet
```

### Security Verification
```bash
# Test authentication
curl -k --cert /var/lib/kubelet/pki/kubelet.crt \
     --key /var/lib/kubelet/pki/kubelet.key \
     https://localhost:10250/metrics

# Verify certificate rotation
kubectl get csr | grep kubelet

# Check security settings
kubectl auth can-i --list --as=system:node:$(hostname)
```

### Expected Outcomes
- Anonymous access is disabled
- Webhook authentication and authorization work
- TLS certificates are properly configured and rotating
- Security policies are enforced

---

## Exercise 5: Logging and Monitoring Configuration

### Scenario
Configure comprehensive logging and monitoring for Kubelet operations.

### Tasks
1. Configure structured logging
2. Set up log rotation and retention
3. Enable metrics collection
4. Configure event recording and retention

### Logging and Monitoring Configuration
```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
# Logging Configuration
logging:
  format: json
  flushFrequency: 5s
  verbosity: 2
  options:
    json:
      infoBufferSize: "0"

# Log Rotation
logRotateMaxSize: 100Mi
logRotateMaxBackups: 5
logRotateMaxAge: 7

# Container Log Management
containerLogMaxSize: 50Mi
containerLogMaxFiles: 5

# Event Configuration
eventRecordQPS: 50
eventBurst: 100
eventTTL: 1h

# Metrics
enableProfilingHandler: true
enableDebuggingHandlers: true
metricsBindAddress: 0.0.0.0:10255

# Health Checks
healthzBindAddress: 0.0.0.0:10248
healthzPort: 10248

# Node Status
nodeStatusMaxImages: 50
nodeStatusUpdateFrequency: 10s
nodeStatusReportFrequency: 5m

# Runtime Monitoring
runtimeRequestTimeout: 10m
imageMinimumGCAge: 2m
imageGCHighThresholdPercent: 85
imageGCLowThresholdPercent: 80
```

### Apply Configuration
```bash
# Update kubelet configuration
sudo cp logging-kubelet-config.yaml /var/lib/kubelet/config.yaml

# Restart kubelet
sudo systemctl restart kubelet
```

### Monitoring Commands
```bash
# Check Kubelet metrics
curl http://localhost:10255/metrics

# Monitor health endpoint
curl http://localhost:10248/healthz

# View structured logs
journalctl -u kubelet -o json-pretty

# Check event logs
kubectl get events --field-selector involvedObject.kind=Node

# Monitor resource usage
kubectl top nodes
```

### Expected Outcomes
- Structured JSON logging is enabled
- Log rotation works correctly
- Metrics are accessible and properly formatted
- Events are recorded and retained appropriately

---

## Exercise 6: Container Runtime Configuration

### Scenario
Configure Kubelet to work with different container runtimes and optimize container operations.

### Tasks
1. Configure containerd runtime settings
2. Set up image pull policies and parallel pulls
3. Configure registry authentication
4. Optimize container lifecycle management

### Container Runtime Configuration
```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
# Container Runtime
containerRuntimeEndpoint: unix:///var/run/containerd/containerd.sock
imageServiceEndpoint: unix:///var/run/containerd/containerd.sock

# Image Management
imageMinimumGCAge: 2m
imageGCHighThresholdPercent: 85
imageGCLowThresholdPercent: 80
serializeImagePulls: false
maxParallelImagePulls: 5

# Registry Configuration
registryPullQPS: 10
registryBurst: 20

# Container Lifecycle
streamingConnectionIdleTimeout: 4h
dockerDisableSharedPID: false
podPidsLimit: 4096

# Runtime Class Support
runtimeRequestTimeout: 10m

# Volume Configuration
volumePluginDir: /usr/libexec/kubernetes/kubelet-plugins/volume/exec/
```

### Apply Configuration
```bash
# Update configuration
sudo cp runtime-kubelet-config.yaml /var/lib/kubelet/config.yaml

# Restart kubelet
sudo systemctl restart kubelet
```

### Verification Commands
```bash
# Test runtime connection
sudo crictl version

# Check image operations
sudo crictl images
sudo crictl pull nginx:latest

# Monitor container runtime
sudo crictl stats
sudo crictl ps
```

### Expected Outcomes
- Containerd runtime is properly configured
- Image pulls work efficiently with parallel downloads
- Container lifecycle operations are optimized

---

## Exercise 7: Node Taints, Labels, and Scheduling Configuration

### Scenario
Configure Kubelet to properly handle node scheduling, taints, and labels for workload placement.

### Tasks
1. Configure node labels and annotations
2. Set up node taints for specialized workloads
3. Configure scheduling policies
4. Implement node cordoning and draining procedures

### Node Configuration
```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
# Node Labels
nodeLabels:
  node-type: compute
  hardware: gpu
  zone: us-west-1a
  instance-type: m5.xlarge

# Provider Configuration
providerID: aws:///us-west-1a/i-1234567890abcdef0
cloudProvider: aws

# Scheduling Configuration
maxPods: 110
podsPerCore: 0

# Node Status
nodeStatusUpdateFrequency: 10s
nodeStatusReportFrequency: 5m
```

### Apply Configuration and Node Management
```bash
# Apply kubelet configuration
sudo cp node-kubelet-config.yaml /var/lib/kubelet/config.yaml
sudo systemctl restart kubelet

# Add node labels
kubectl label nodes worker-node-1 hardware=gpu
kubectl label nodes worker-node-1 workload-type=ml

# Add node taints
kubectl taint nodes worker-node-1 dedicated=gpu:NoSchedule
kubectl taint nodes worker-node-1 gpu=true:NoExecute

# Configure node annotations
kubectl annotate nodes worker-node-1 cluster-autoscaler.kubernetes.io/scale-down-disabled=true
```

### Node Maintenance Commands
```bash
# Cordon node (prevent new pods)
kubectl cordon worker-node-1

# Drain node (remove existing pods)
kubectl drain worker-node-1 --ignore-daemonsets --delete-emptydir-data

# Uncordon node (allow scheduling)
kubectl uncordon worker-node-1

# Check node scheduling status
kubectl get nodes -o wide
kubectl describe node worker-node-1
```

### Expected Outcomes
- Node labels and taints are properly applied
- Specialized workloads schedule correctly
- Node maintenance operations work smoothly
- Scheduling policies are enforced

---

## Exercise 8: Performance Tuning and Optimization

### Scenario
Optimize Kubelet configuration for high-performance workloads and large-scale deployments.

### Tasks
1. Configure for high pod density
2. Optimize garbage collection settings
3. Tune networking and DNS performance
4. Configure for minimal latency

### High-Performance Configuration
```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
# High Density Configuration
maxPods: 500
podsPerCore: 10

# Optimized Frequencies
nodeStatusUpdateFrequency: 4s
nodeStatusReportFrequency: 1m
syncFrequency: 10s
fileCheckFrequency: 10s
httpCheckFrequency: 10s

# Garbage Collection Optimization
imageGCHighThresholdPercent: 90
imageGCLowThresholdPercent: 85
imageMinimumGCAge: 30s

# Container GC
minimumGCAge: 30s
maxPerPodContainerCount: 2
maxContainerCount: 500

# Event Optimization
eventRecordQPS: 100
eventBurst: 200

# DNS Optimization
clusterDNS:
- 10.96.0.10
resolverConfig: /etc/resolv.conf
dnsPolicy: ClusterFirst

# Runtime Optimization
serializeImagePulls: false
maxParallelImagePulls: 10
registryPullQPS: 20
registryBurst: 40

# Resource Management
systemReserved:
  cpu: 1000m
  memory: 2Gi
  ephemeral-storage: 5Gi
kubeReserved:
  cpu: 1000m
  memory: 2Gi
  ephemeral-storage: 2Gi

# Security Optimizations
authentication:
  webhook:
    cacheTTL: 2m
authorization:
  webhook:
    cacheAuthorizedTTL: 10m
    cacheUnauthorizedTTL: 1m
```

### Apply Optimization
```bash
# Apply performance configuration
sudo cp performance-kubelet-config.yaml /var/lib/kubelet/config.yaml
sudo systemctl restart kubelet
```

### Performance Monitoring
```bash
# Monitor Kubelet performance
kubectl top nodes
kubectl get nodes -o custom-columns=NAME:.metadata.name,PODS:.status.capacity.pods,ALLOCATABLE:.status.allocatable.pods

# Check resource usage
curl -s localhost:10255/metrics | grep kubelet_

# Monitor garbage collection
journalctl -u kubelet | grep -i "garbage collect"
```

### Expected Outcomes
- Increased pod density without performance degradation
- Optimized resource utilization
- Reduced latency for pod operations
- Efficient garbage collection

---

## Exercise 9: Kubeadm Integration and Cluster-wide Updates

### Scenario
Use kubeadm to manage kubelet configuration updates across the entire cluster during upgrades.

### Tasks
1. Prepare cluster for kubelet configuration upgrade
2. Update control plane kubelet configuration
3. Update worker node kubelet configuration
4. Verify cluster-wide configuration consistency

### Control Plane Configuration Update
```bash
# On control plane node
# Update kubeadm configuration first
sudo kubeadm upgrade plan

# Upgrade control plane components
sudo kubeadm upgrade apply v1.29.0

# Update kubelet configuration
sudo kubeadm upgrade node phase kubelet-config

# Restart kubelet
sudo systemctl restart kubelet

# Verify control plane
kubectl get nodes
kubectl cluster-info
```

### Worker Node Configuration Update
```bash
# On each worker node
# Download new kubelet configuration
sudo kubeadm upgrade node phase kubelet-config

# Restart kubelet with new configuration
sudo systemctl restart kubelet

# Verify node status
kubectl get nodes
```

### Custom Configuration with Kubeadm
Create a kubeadm configuration file for custom kubelet settings:

```yaml
# kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.29.0
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
maxPods: 200
cpuManagerPolicy: static
systemReserved:
  cpu: 500m
  memory: 1Gi
kubeReserved:
  cpu: 500m
  memory: 1Gi
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs
```

### Apply Custom Configuration
```bash
# Apply the configuration during upgrade
sudo kubeadm upgrade apply --config=kubeadm-config.yaml v1.29.0

# Update nodes with new configuration
sudo kubeadm upgrade node phase kubelet-config

# Restart kubelet
sudo systemctl restart kubelet
```

### Verification Commands
```bash
# Check kubelet version across cluster
kubectl get nodes -o custom-columns=NAME:.metadata.name,VERSION:.status.nodeInfo.kubeletVersion

# Verify configuration consistency
for node in $(kubectl get nodes -o name); do
  echo "=== $node ==="
  kubectl get --raw /api/v1/nodes/${node#node/}/proxy/configz | jq '.kubeletconfig.maxPods'
done

# Check cluster health
kubectl get componentstatuses
kubectl get nodes -o wide
```

### Expected Outcomes
- All nodes run the same kubelet version
- Configuration is consistent across the cluster
- Cluster remains healthy during updates
- Custom configurations are properly applied

---

## Exercise 10: Troubleshooting Configuration Issues

### Scenario
Diagnose and fix various Kubelet configuration problems that may occur in production environments.

### Common Issues and Solutions

#### Issue 1: Kubelet Fails to Start After Configuration Change
```bash
# Check systemd status
sudo systemctl status kubelet

# Check logs for errors
journalctl -u kubelet -n 50

# Validate configuration syntax
sudo kubelet --config=/var/lib/kubelet/config.yaml --dry-run

# Restore backup configuration if needed
sudo cp /var/lib/kubelet/config.yaml.backup /var/lib/kubelet/config.yaml
sudo systemctl restart kubelet
```

#### Issue 2: Node Not Ready After Configuration
```bash
# Check node status
kubectl get nodes
kubectl describe node $(hostname)

# Check Kubelet logs
journalctl -u kubelet -f

# Verify container runtime
sudo systemctl status containerd
sudo crictl version

# Check certificates
openssl x509 -in /var/lib/kubelet/pki/kubelet.crt -text -noout
```

#### Issue 3: Resource Manager Policy Conflicts
```bash
# Reset CPU manager state
sudo systemctl stop kubelet
sudo rm -f /var/lib/kubelet/cpu_manager_state
sudo rm -f /var/lib/kubelet/memory_manager_state

# Apply corrected configuration
sudo cp fixed-kubelet-config.yaml /var/lib/kubelet/config.yaml
sudo systemctl start kubelet

# Verify policy is active
cat /var/lib/kubelet/cpu_manager_state
```

### Emergency Recovery Procedure
```bash
# Complete kubelet reset and reconfiguration
sudo systemctl stop kubelet

# Reset kubelet configuration to defaults
sudo kubeadm reset phase cleanup-node

# Rejoin the cluster
sudo kubeadm join <control-plane-endpoint> --token <token> --discovery-token-ca-cert-hash <hash>

# Verify cluster rejoin
kubectl get nodes
kubectl describe node $(hostname)
```

### Expected Outcomes
- Ability to diagnose configuration issues quickly
- Successful rollback procedures when needed
- Proper validation of configuration changes
- Recovery from various failure scenarios

---

## Comprehensive Final Exercise: Multi-Node Configuration Management

### Scenario
Manage kubelet configuration across a heterogeneous cluster with different node types.

### Node-Specific Configurations

#### GPU Node Configuration
```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
maxPods: 100
cpuManagerPolicy: static
memoryManagerPolicy: Static
topologyManagerPolicy: single-numa-node
systemReserved:
  cpu: 1000m
  memory: 4Gi
  nvidia.com/gpu: 0
kubeReserved:
  cpu: 1000m
  memory: 2Gi
nodeLabels:
  accelerator: nvidia-tesla-v100
  workload-type: ml
featureGates:
  DevicePlugins: true
```

#### Edge Node Configuration
```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
maxPods: 30
cpuManagerPolicy: none
memoryManagerPolicy: None
imageGCHighThresholdPercent: 95
imageGCLowThresholdPercent: 90
systemReserved:
  cpu: 200m
  memory: 512Mi
kubeReserved:
  cpu: 200m
  memory: 256Mi
nodeLabels:
  node-type: edge
  location: remote
evictionHard:
  memory.available: "50Mi"
  nodefs.available: "5%"
```

### Configuration Deployment Process
```bash
# For GPU nodes
sudo cp gpu-kubelet-config.yaml /var/lib/kubelet/config.yaml
sudo systemctl restart kubelet
kubectl label nodes gpu-node-1 accelerator=nvidia-tesla-v100
kubectl taint nodes gpu-node-1 dedicated=gpu:NoSchedule

# For edge nodes  
sudo cp edge-kubelet-config.yaml /var/lib/kubelet/config.yaml
sudo systemctl restart kubelet
kubectl label nodes edge-node-1 node-type=edge
kubectl taint nodes edge-node-1 edge=true:NoSchedule

# Verify configurations
kubectl get nodes --show-labels
kubectl describe nodes
```

### Cluster-wide Validation
```bash
# Check all node configurations
kubectl get nodes -o custom-columns=NAME:.metadata.name,LABELS:.metadata.labels,TAINTS:.spec.taints

# Verify kubelet versions
kubectl get nodes -o custom-columns=NAME:.metadata.name,VERSION:.status.nodeInfo.kubeletVersion

# Test workload scheduling
kubectl apply -f test-workloads.yaml
kubectl get pods -o wide

# Monitor cluster health
kubectl get componentstatuses
kubectl top nodes
```

### Expected Outcomes
- Different node types have appropriate configurations
- Workloads schedule to correct node types
- Cluster maintains overall health and stability
- Configuration changes are applied consistently

---

## Answer Key and Validation

### General Validation Commands
```bash
# Check kubelet service status
sudo systemctl status kubelet

# Verify configuration syntax
sudo kubelet --config=/var/lib/kubelet/config.yaml --dry-run

# Check node readiness
kubectl get nodes
kubectl describe node $(hostname)

# Monitor kubelet logs
journalctl -u kubelet -f

# Verify resource allocation
kubectl describe node $(hostname) | grep -A 10 "Allocatable"

# Check current configuration
kubectl get --raw /api/v1/nodes/$(hostname)/proxy/configz | jq
```

### Common Troubleshooting Steps
1. Always backup configuration before changes
2. Validate YAML syntax before applying
3. Check systemd service status after restart
4. Monitor logs for error messages
5. Verify node remains in Ready state
6. Test pod scheduling functionality

### Configuration Best Practices
- Use incremental changes rather than large configuration overhauls
- Test configurations in development before production
- Keep backup copies of working configurations
- Document all configuration changes
- Use kubeadm for cluster-wide consistency
- Monitor cluster health after configuration changes