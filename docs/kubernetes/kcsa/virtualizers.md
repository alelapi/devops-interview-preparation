# Virtualization Technologies

## gVisor

gVisor is an application kernel developed by Google that provides an additional security layer for containers by intercepting and handling system calls.

### Key Characteristics:

- Acts as a security sandbox between containers and the host kernel
- Implements a substantial portion of the Linux system call interface in userspace
- Written primarily in Go
- Creates an application kernel that mediates access between the container and host

### How it Works:

- Intercepts system calls from containerized applications
- Implements its own network and filesystem interfaces
- Provides compatibility with standard container runtimes via OCI integration (runsc)
- Significantly reduces the attack surface exposed to containers

### Use Cases:

- Multi-tenant container deployments
- Running untrusted or third-party code
- Enhancing security of web-facing containerized applications

```
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc
---
apiVersion: v1
kind: Pod
metadata:
  name: gvisor-pod
spec:
  runtimeClassName: gvisor
  containers:
  - name: nginx
    image: nginx
```

## Firecracker
Firecracker is a lightweight virtualization technology developed by AWS that powers AWS Lambda and Fargate services.

## Key Characteristics:

- Micro-VM technology combining VM security with container-like performance
- Minimalist VMM (Virtual Machine Monitor) built on KVM
- Written in Rust for memory safety
- Creates lightweight VMs in milliseconds

## How it Works:

- Launches micro-VMs with minimal memory footprint (~5MB per instance)
- Provides a minimal device model (virtio-net, virtio-block, serial console)
- Uses a RESTful API to manage VM lifecycle
- Each workload runs in a separate VM with true hardware-based isolation

## Use Cases:

- Serverless computing platforms
- Container-as-a-service offerings
- Secure isolation of workloads
- High-density computing environments

```
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: kata-fc
handler: kata-fc
---
apiVersion: v1
kind: Pod
metadata:
  name: firecracker-pod
spec:
  runtimeClassName: kata-fc
  containers:
  - name: nginx
    image: nginx
```