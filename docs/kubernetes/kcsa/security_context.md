# Security Context

## Examples
```
apiVersion: v1
kind: Pod
metadata:
  name: mixed-security-context
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000  # file system group ownership for volumes mounted in a Pod
    supplementalGroups: [1001, 1002]  # adds secondary group IDs to the processes running in all containers of a Pod
    seLinuxOptions:  # SELinux (Security-Enhanced Linux) parameters for containers and pods. SELinux provides mandatory access controls by enforcing security policies that restrict what processes can do
      level: "s0:c123,c456"
    seccompProfile: # apply seccomp (secure computing mode) profiles to restrict the system calls that containers can make to the Linux kernel
      type: RuntimeDefault
  containers:
  - name: first-container
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: ["ALL"]
  - name: second-container
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      runAsUser: 2000  # Overrides the Pod-level setting
      capabilities:
        add: ["NET_ADMIN"]
```