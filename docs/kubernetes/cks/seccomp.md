# Creating Seccomp Audit Profile

## Seccomp

Seccomp (secure computing mode) is a Linux kernel feature that allows restricting the system calls that a process can make. In container environments, seccomp profiles provide an additional layer of security by limiting what actions containers can perform at the system call level.

Audit mode allows you to log all system calls without blocking any of them, which is useful for:
- Understanding what system calls an application requires
- Creating baseline profiles for applications
- Detecting potential malicious activities
- Debugging permission issues

## Creating an Audit Seccomp Profile

Create a JSON file named `audit-seccomp.json` with the following content:

```json
{
  "defaultAction": "SCMP_ACT_ALLOW",
  "architectures": [
    "SCMP_ARCH_X86_64",
    "SCMP_ARCH_X86",
    "SCMP_ARCH_AARCH64"
  ],
  "syscalls": [
    {
      "names": [
        "open",
        "openat",
        "read",
        "write",
        "connect",
        "socket",
        "execve",
        "clone"
      ],
      "action": "SCMP_ACT_LOG"
    }
  ]
}
```

This profile sets the default action to `SCMP_ACT_LOG`, which logs all system calls without blocking them. The `syscalls` array of this profile would only log the specified system calls and silently allow the rest. An empty `syscalls` array means no system calls have special handling - they're all logged.

- **defaultAction**: The action to take by default (in this case, log but allow all syscalls)
- **architectures**: The CPU architectures the profile applies to
- **syscalls**: A list of system calls with specific actions (empty in this audit profile)

## Installing the Profile on a Worker Node

```bash
sudo mkdir -p /var/lib/kubelet/seccomp/profiles
sudo cp audit-seccomp.json /var/lib/kubelet/seccomp/profiles/
sudo chmod 644 /var/lib/kubelet/seccomp/profiles/audit-seccomp.json
```

## Applying the Profile to Containers

#### Method 1: Using annotations (older approach)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: audit-pod
  annotations:
    seccomp.security.alpha.kubernetes.io/pod: "localhost/audit-seccomp.json"
spec:
  containers:
  - name: my-container
    image: nginx
```

#### Method 2: Using securityContext (recommended)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: audit-pod
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: audit-seccomp.json
  containers:
  - name: my-container
    image: nginx
```
