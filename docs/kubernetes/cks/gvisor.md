# gVisor

## Introduction to gVisor

gVisor is an application kernel, written in Go, that implements a substantial portion of the Linux system call interface. It provides an additional layer of isolation between running applications and the host operating system, acting as a user-space kernel for containers.

Unlike traditional container runtimes that share the host kernel, gVisor intercepts application system calls and acts as the guest kernel, providing better security isolation without the overhead of a full virtual machine.

Key benefits include:
- Enhanced security isolation
- Reduced attack surface
- Compatible with existing container workflows
- Lightweight compared to VMs

## Basic Configuration

### Platform Configuration

gVisor supports multiple platforms with different isolation characteristics. Edit `/etc/containerd/config.toml` or configure Docker to use your preferred platform.

**Available platforms:**

1. **ptrace**: Uses `ptrace` system call for isolation
   - Compatible with most systems
   - Slower than other platforms

2. **KVM**: Uses hardware virtualization
   - Requires KVM support in the kernel
   - Better performance than ptrace

To configure the platform:

```bash
# Create configuration with specific platform
sudo runsc install --runtime=runsc-ptrace --platform=ptrace
sudo runsc install --runtime=runsc-kvm --platform=kvm
```

### Network Configuration

gVisor provides multiple networking options that can be configured in the runtime spec:

```bash
# Default networking
sudo runsc install --network=sandbox

# Host networking
sudo runsc install --network=host
```

### Storage Configuration

Configure the overlay filesystem for better performance:

```bash
sudo runsc install --overlay
```

For Docker, create a config file at `/etc/docker/daemon.json`:

```json
{
  "runtimes": {
    "runsc": {
      "path": "/usr/local/bin/runsc",
      "runtimeArgs": [
        "--overlay",
        "--network=sandbox"
      ]
    },
    "runsc-kvm": {
      "path": "/usr/local/bin/runsc",
      "runtimeArgs": [
        "--platform=kvm",
        "--overlay",
        "--network=sandbox"
      ]
    }
  }
}
```

Restart Docker after configuration changes:

```bash
sudo systemctl restart docker
```

## Using gVisor

1. For a Kubernetes cluster with containerd, ensure containerd is configured to use gVisor as shown above.

2. Create a RuntimeClass:

```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc
```

3. Apply the RuntimeClass:

```bash
kubectl apply -f runtimeclass.yaml
```

4. Create a pod using gVisor:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-gvisor
spec:
  runtimeClassName: gvisor
  containers:
  - name: nginx
    image: nginx
```

5. Verify the pod is running with gVisor:

```bash
kubectl exec -it nginx-gvisor -- dmesg | grep gVisor
```

## Monitoring and Debugging

gVisor provides several debugging tools:

1. **View debug logs**:

```bash
# Enable debug logs
sudo runsc install --debug --debug-log=/var/log/runsc/
tail -f /var/log/runsc/*.log
```

2. **Get stacks** of a running container:

```bash
runsc debug --stacks <container-id>
```

3. **Create a CPU profile**:

```bash
runsc debug --cpu-profile=/tmp/profile.out <container-id>
```

4. **Examine container logs**:

```bash
runsc debug --logs <container-id>
```

5. **Trace system calls**:

```bash
runsc debug --trace <container-id>
```