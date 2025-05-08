# RuntimeClass

## Introduction

RuntimeClass is a Kubernetes feature that allows you to select the container runtime configuration used to run a Pod's containers. It's particularly valuable in scenarios where you need to provide different levels of isolation, security, or performance for specific workloads. 
The RuntimeClass resource is cluster-scoped (non-namespaced) and maps a runtime class name to the corresponding configuration used by the container runtime to run a container.

## How RuntimeClass Works

RuntimeClass leverages the Container Runtime Interface (CRI) to expose different runtime options to Kubernetes. The workflow is:

1. An administrator configures the CRI implementation on nodes (such as containerd or CRI-O).
2. The administrator creates RuntimeClass objects that reference handlers for these configurations.
3. Users specify a `runtimeClassName` in their Pod specs.
4. Kubelet uses the referenced RuntimeClass to determine which runtime handler to use for the Pod.

## Setting Up RuntimeClass

### Step 1: Configure CRI Implementation on Nodes

The specific configuration depends on your CRI implementation:

#### For containerd (v1.2+)

Edit `/etc/containerd/config.toml` to define runtime handlers:

```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
    runtime_type = "io.containerd.runc.v2"
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
      SystemdCgroup = true

  # Example configuration for Kata Containers
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.kata]
    runtime_type = "io.containerd.kata.v2"
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.kata.options]
      ConfigPath = "/opt/kata/share/defaults/kata-containers/configuration.toml"

  # Example configuration for gVisor
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.gvisor]
    runtime_type = "io.containerd.runsc.v1"
```

After modifying the configuration, restart containerd:

```bash
sudo systemctl restart containerd
```

#### For CRI-O

Edit `/etc/crio/crio.conf` to define runtime handlers:

```toml
[crio.runtime.runtimes.runc]
runtime_path = "/usr/local/bin/runc"

[crio.runtime.runtimes.kata]
runtime_path = "/usr/bin/kata-runtime"

[crio.runtime.runtimes.runsc]
runtime_path = "/usr/local/bin/runsc"
```

After modifying the configuration, restart CRI-O:

```bash
sudo systemctl restart crio
```

### Step 2: Create RuntimeClass Objects

Create RuntimeClass objects to reference your configured handlers:

```yaml
# runc-runtimeclass.yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: runc
handler: runc
```

```yaml
# kata-runtimeclass.yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: kata
handler: kata
```

```yaml
# gvisor-runtimeclass.yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: gvisor
```

Apply the RuntimeClass objects:

```bash
kubectl apply -f runc-runtimeclass.yaml
kubectl apply -f kata-runtimeclass.yaml
kubectl apply -f gvisor-runtimeclass.yaml
```

Verify the RuntimeClass objects:

```bash
kubectl get runtimeclass
```

## Using RuntimeClass in Pods

Once RuntimeClasses are defined, you can use them in your Pod specifications:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-kata
spec:
  runtimeClassName: kata
  containers:
  - name: nginx
    image: nginx:latest
```

When you create this Pod, the kubelet will use the specified RuntimeClass (in this case, "kata") to run the Pod's containers. The kubelet will use the "kata" handler which maps to Kata Containers runtime.

### Scheduling

RuntimeClass also supports scheduling constraints to ensure Pods are scheduled to nodes that support the specified runtime.

```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: kata
handler: kata
scheduling:
  nodeSelector:
    runtime-kata: "true"
  tolerations:
  - key: "runtime-kata"
    operator: "Exists"
    effect: "NoSchedule"
```

With this configuration, Pods using the "kata" RuntimeClass will only be scheduled on nodes with the label `runtime-kata: "true"`, and they will tolerate the NoSchedule taint with the key "runtime-kata".

## Practical Examples

This section provides concrete examples for setting up and using various RuntimeClasses.

### Example 1: Using gVisor for Untrusted Workloads

gVisor is a container runtime that provides an additional layer of isolation by implementing a user-space kernel. Here's how to set it up and use it.

1. Install gVisor on your nodes (commands will vary based on your operating system).

2. Configure containerd:

   ```toml
   [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runsc]
     runtime_type = "io.containerd.runsc.v1"
   ```

3. Create a RuntimeClass:

   ```yaml
   apiVersion: node.k8s.io/v1
   kind: RuntimeClass
   metadata:
     name: gvisor
   handler: runsc
   ```

4. Deploy a Pod that uses gVisor:

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: nginx-gvisor
   spec:
     runtimeClassName: gvisor
     containers:
     - name: nginx
       image: nginx:latest
   ```

### Example 2: Using Kata Containers for Hardware Isolation

Kata Containers provide stronger isolation by running containers in lightweight VMs.

1. Install Kata Containers on your nodes.

2. Configure containerd:

   ```toml
   [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.kata-fc]
     runtime_type = "io.containerd.kata.v2"
     [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.kata-fc.options]
       ConfigPath = "/opt/kata/share/defaults/kata-containers/configuration-fc.toml"
   ```

3. Create a RuntimeClass with scheduling and overhead:

   ```yaml
   apiVersion: node.k8s.io/v1
   kind: RuntimeClass
   metadata:
     name: kata-fc
   handler: kata-fc
   overhead:
     podFixed:
       memory: "130Mi"
       cpu: "250m"
   scheduling:
     nodeSelector:
       katacontainers.io/kata-runtime: "true"
   ```

4. Label nodes that support Kata Containers:

   ```bash
   kubectl label nodes worker1 katacontainers.io/kata-runtime=true
   ```

5. Deploy a Pod that uses Kata Containers:

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: redis-kata
   spec:
     runtimeClassName: kata-fc
     containers:
     - name: redis
       image: redis:latest
   ```
