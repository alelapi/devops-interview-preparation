# Pod Security Standards (PSS): Overview and Implementation Guide

## Introduction to Pod Security Standards

Pod Security Standards (PSS) is a feature in Kubernetes that defines different levels of security for pods. It replaced the deprecated PodSecurityPolicy (PSP) in Kubernetes 1.25. The PSS provides a standardized way to enforce security controls for pods, helping cluster administrators ensure workloads meet specific security requirements without needing to create and manage custom policies.

## PSS Profiles

PSS defines three security profiles with increasing levels of restriction:

1. **Privileged**: Unrestricted policy, providing the widest possible level of permissions. This profile has essentially no restrictions on pod configuration.

2. **Baseline**: Minimally restrictive policy which prevents known privilege escalations. Allows the default (minimally specified) pod configuration.

3. **Restricted**: Heavily restricted policy, following current pod hardening best practices. This is the most secure profile, designed to enforce security best practices.

## PSS Implementation Methods

There are three ways to implement Pod Security Standards:

1. **Pod Security Admission Controller**: Built into Kubernetes since v1.23, it can enforce the standards at the namespace level.

2. **Namespace Labels**: Used to configure the Pod Security Admission Controller with different modes (enforce, audit, warn).

3. **3rd Party Solutions**: Various tools like OPA/Gatekeeper, Kyverno, etc. can enforce PSS.

## PSS Modes

Each profile can be applied in one of three modes:

1. **enforce**: Policy violations will cause the pod to be rejected
2. **audit**: Policy violations trigger audit annotations, but are allowed
3. **warn**: Policy violations trigger user-facing warnings, but are allowed

## Exam Task Example: Making a Deployment Compatible with PSS-Restricted

### Task Description

Create a deployment that is compatible with the Pod Security Standard "restricted" profile in enforce mode. The deployment should run a container that:
1. Reads and writes to a persistent volume
2. Exposes port 8080
3. Runs as a non-root user
4. Meets all the requirements of the restricted profile

### Solution

#### Step 1: Configure Namespace with PSS

First, let's create a namespace with the restricted profile in enforce mode:

```bash
kubectl create namespace pss-restricted
kubectl label --overwrite ns pss-restricted \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/audit=restricted
```

#### Step 2: Create a Compliant Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pss-restricted-app
  namespace: pss-restricted
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pss-restricted-app
  template:
    metadata:
      labels:
        app: pss-restricted-app
    spec:
      # Security Context at Pod level
      securityContext:
        runAsNonRoot: true
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: app
        image: nginx:1.21
        # Container Security Context
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
          runAsUser: 1000
          runAsGroup: 3000
          seccompProfile:
            type: RuntimeDefault
          readOnlyRootFilesystem: true
        ports:
        - containerPort: 8080
        resources:
          limits:
            cpu: "500m"
            memory: "512Mi"
          requests:
            cpu: "100m"
            memory: "128Mi"
        volumeMounts:
        - name: data-volume
          mountPath: /data
          readOnly: false
        - name: tmp-volume
          mountPath: /tmp
      volumes:
      - name: data-volume
        persistentVolumeClaim:
          claimName: pss-restricted-pvc
      - name: tmp-volume
        emptyDir: {}
```

#### Step 3: Create the Persistent Volume Claim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pss-restricted-pvc
  namespace: pss-restricted
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard
```

#### Step 4: Apply the Configuration

```bash
kubectl apply -f pss-pvc.yaml
kubectl apply -f pss-deployment.yaml
```

#### Step 5: Verify Deployment Status

```bash
kubectl get deployment -n pss-restricted
kubectl describe deployment pss-restricted-app -n pss-restricted
```

### Key PSS Restricted Requirements Explained

When the Pod Security Standard enforces the restricted profile, the following security requirements must be met:

1. **Pod-level Requirements**:
   - `securityContext.runAsNonRoot: true`: Ensures pods run as non-root users
   - `securityContext.seccompProfile.type: RuntimeDefault`: Enables default seccomp profile

2. **Container-level Requirements**:
   - `securityContext.allowPrivilegeEscalation: false`: Prevents privilege escalation
   - `securityContext.capabilities.drop: ["ALL"]`: Drops all Linux capabilities
   - `securityContext.runAsUser` and `runAsGroup`: Explicit non-zero user/group IDs
   - `securityContext.seccompProfile.type: RuntimeDefault`: Enables seccomp at container level
   - `readOnlyRootFilesystem: true`: Makes root filesystem read-only

3. **Volume-related Considerations**:
   - When a read-only root filesystem is used, you must provide writable volumes for paths like `/tmp`
   - PersistentVolumeClaims are allowed and need appropriate mount paths

4. **Prohibited configurations**:
   - `hostPath` volumes
   - `hostNetwork`, `hostIPC`, and `hostPID` set to true
   - `privileged` containers
   - Adding capabilities beyond a minimal set
   - HostPort usage

## Common Issues and Troubleshooting

When working with PSS in restricted mode, you might encounter the following issues:

1. **Pod Rejection Errors**: If a pod is rejected, check the error message from kubectl, which will specify which PSS policy was violated.

2. **Runtime Failures**: Even if a pod starts, it might fail if it needs to write to locations that are now read-only.

3. **Legacy Applications**: Older applications often assume root access or specific capabilities, requiring refactoring.

### Troubleshooting Commands

```bash
# Check if pod was rejected due to PSS violations
kubectl get events -n pss-restricted

# Check audit annotations for violations
kubectl get pod <pod-name> -n pss-restricted -o yaml | grep "audit"

# Check warning events
kubectl get events -n pss-restricted | grep Warning
```

## Additional Exam Task Examples

### Task 1: Configure a Namespace for PSS Audit Mode

Create a namespace that audits according to the restricted profile but only enforces the baseline profile.

```bash
kubectl create namespace mixed-pss
kubectl label --overwrite ns mixed-pss \
  pod-security.kubernetes.io/enforce=baseline \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted
```

### Task 2: Troubleshoot a Pod Failing Due to PSS Restrictions

Given a pod that is failing to deploy in a namespace with PSS restricted enforcement, identify and fix the issues.

Original pod manifest:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privileged-pod
  namespace: pss-restricted
spec:
  containers:
  - name: web
    image: nginx
    ports:
    - containerPort: 80
    securityContext:
      privileged: true  # Violation! 
```

Fixed pod manifest:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: compliant-pod
  namespace: pss-restricted
spec:
  securityContext:
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: web
    image: nginx
    ports:
    - containerPort: 80
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
      runAsUser: 101  # nginx user
      runAsGroup: 101
      seccompProfile:
        type: RuntimeDefault
```

## Conclusion

Pod Security Standards provide a robust framework for securing Kubernetes workloads. Understanding the different profiles and their requirements is essential for creating secure deployments that can run in restricted environments. When working with the restricted profile:

1. Always specify non-root users
2. Drop ALL capabilities at the container level
3. Prevent privilege escalation
4. Use appropriate seccomp profiles
5. Make root filesystems read-only when possible
6. Use emptyDir volumes for temporary writable storage

By following these practices, you can ensure your workloads are compatible with Kubernetes security best practices and can run in environments with strict security requirements.
