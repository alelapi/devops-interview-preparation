# Creating a Pod with Host Path Access and Falco Detection

## Problem Statement

Create a deployment that can read and write to a specific path in the underlying host machine's filesystem using a normal volume added to the container specs. The solution must ensure that access to any other host paths is denied. After creating the deployment and verifying it works, create Falco rules to detect this activity, identifying which container is performing the actions and displaying the output via journalctl.

## Solution

### Part 1: Creating a Deployment with Host Path Access

Let's create a deployment that can access the host filesystem:

```yaml
# host-path-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: host-path-access
  labels:
    app: host-path-access
spec:
  replicas: 1
  selector:
    matchLabels:
      app: host-path-access
  template:
    metadata:
      labels:
        app: host-path-access
    spec:
      securityContext:
        runAsUser: 1000
        runAsGroup: 1000
        fsGroup: 1000
      containers:
      - name: host-path-container
        image: ubuntu:20.04
        command: ["sleep", "infinity"]
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop: ["ALL"]
          readOnlyRootFilesystem: true
        volumeMounts:
        - name: host-volume
          mountPath: /host-data
      volumes:
      - name: host-volume
        hostPath:
          path: /opt/app-data  # The specific host path we want to access
          type: DirectoryOrCreate
```

Apply the deployment:
```bash
kubectl apply -f host-path-deployment.yaml
```

### Part 2: Verify Host Path Access

```bash
# Get the pod name
POD_NAME=$(kubectl get pods -l app=host-path-access -o jsonpath='{.items[0].metadata.name}')

# Exec into the pod
kubectl exec -it $POD_NAME -- bash

# Inside the pod, verify access to the host path
ls -la /host-data
echo "Test file for host path access" > /host-data/test.txt
cat /host-data/test.txt

# Verify we can't write to other locations
echo "This should fail" > /etc/test.txt  # Should fail
touch /bin/test-file  # Should fail

# Exit the pod
exit
```

### Part 3: Creating Falco Rules to Detect Host Path Access

Now, let's create Falco rules to detect when containers access this specific host path:

```yaml
# host-path-detection.yaml
- rule: Container Accessing Host Path
  desc: Detects when a container accesses the specific host path
  condition: >
    container 
    and fd.name startswith "/host-data"
    and (evt.type=open or evt.type=openat)
  output: >
    Container accessing host path 
    (user=%user.name user_uid=%user.uid command=%proc.cmdline 
    file=%fd.name access_type=%evt.is_open_read,%evt.is_open_write
    container_id=%container.id container_name=%container.name
    image=%container.image.repository pod=%k8s.pod.name ns=%k8s.ns.name)
  priority: INFO
  tags: [filesystem, access, allowed]

- rule: Container Writing To Host Path
  desc: Detects when a container writes to the specific host path
  condition: >
    container 
    and fd.name startswith "/host-data"
    and (evt.type=open or evt.type=openat)
    and evt.is_open_write=true
  output: >
    Container WRITING to host path 
    (user=%user.name user_uid=%user.uid command=%proc.cmdline 
    file=%fd.name container_id=%container.id container_name=%container.name
    image=%container.image.repository pod=%k8s.pod.name ns=%k8s.ns.name)
  priority: INFO
  tags: [filesystem, modification, allowed]

- rule: Container Accessing Disallowed Path
  desc: Detects when a container attempts to access any path other than the allowed host path
  condition: >
    container 
    and fd.name startswith "/"
    and not fd.name startswith "/host-data"
    and not fd.name in (/proc, /dev, /sys) 
    and (evt.type=open or evt.type=openat)
    and evt.is_open_write=true
  output: >
    Suspicious: Container attempting to write to DISALLOWED path 
    (user=%user.name user_uid=%user.uid command=%proc.cmdline 
    file=%fd.name container_id=%container.id container_name=%container.name
    image=%container.image.repository pod=%k8s.pod.name ns=%k8s.ns.name)
  priority: WARNING
  tags: [filesystem, modification, suspicious]
```

### Part 4: Deploy the Falco Rules

If you're using Falco as a DaemonSet in Kubernetes:

```bash
# Create a ConfigMap with the rules
kubectl create configmap falco-hostpath-rules --from-file=host-path-detection.yaml

# Update the Falco DaemonSet to use this ConfigMap
kubectl -n falco patch daemonset falco --type=json -p='[
  {
    "op": "add",
    "path": "/spec/template/spec/volumes/-",
    "value": {
      "name": "hostpath-rules",
      "configMap": {
        "name": "falco-hostpath-rules"
      }
    }
  },
  {
    "op": "add",
    "path": "/spec/template/spec/containers/0/volumeMounts/-",
    "value": {
      "mountPath": "/etc/falco/rules.d/",
      "name": "hostpath-rules"
    }
  }
]'
```

If you're running Falco directly on the host:

```bash
# Copy the rules file to the Falco rules directory
sudo cp host-path-detection.yaml /etc/falco/rules.d/

# Restart Falco
sudo systemctl restart falco
```

### Part 5: Monitor Falco Alerts with journalctl

```bash
# Follow Falco logs in journalctl
sudo journalctl -fu falco
```

### Part 6: Test the Falco Detection

```bash
# Trigger allowed access to host path
kubectl exec -it $POD_NAME -- bash -c "echo 'Allowed write to host path' > /host-data/falco-test.txt"

# Attempt disallowed access
kubectl exec -it $POD_NAME -- bash -c "touch /etc/not-allowed.txt || echo 'Access denied as expected'"
```

You should see Falco alerts in the journalctl output that identify which container is accessing the host path.

## Conclusion

This solution demonstrates:

1. How to create a deployment that can access a specific host path using a direct hostPath volume

2. How to restrict file system access to only that specific path

3. How to configure Falco rules to:
   - Detect access to the allowed host path
   - Alert on attempts to access disallowed paths
   - Identify which container is performing these actions

4. How to monitor these activities in real-time using journalctl

The solution fulfills the requirements by providing a deployment with access to a specific host path and proper Falco detection configured to monitor this activity.
