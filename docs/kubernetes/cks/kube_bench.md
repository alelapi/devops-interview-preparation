# kube-bench

## Introduction to kube-bench

kube-bench is an open-source tool that checks whether Kubernetes is deployed securely by running the checks documented in the CIS Kubernetes Benchmark. The tool helps cluster administrators ensure their Kubernetes deployments meet industry security standards.

kube-bench automates security checks against the Center for Internet Security (CIS) Kubernetes Benchmark, which provides guidelines for configuring Kubernetes securely. These benchmarks are widely recognized as security standards for configuring various systems.

Key features:
- Ability to run checks for multiple Kubernetes components
- Support for different Kubernetes versions
- Support for various deployment environments
- Integration with CI/CD pipelines
- Customizable test configurations

## Deployment Options

### Running as a Kubernetes Job

A Kubernetes job is a good option for one-time assessments:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: kube-bench
spec:
  template:
    spec:
      hostPID: true
      containers:
        - name: kube-bench
          image: aquasec/kube-bench:latest
          securityContext:
            privileged: true
          volumeMounts:
          - name: var-lib-kubelet
            mountPath: /var/lib/kubelet
            readOnly: true
          - name: etc-systemd
            mountPath: /etc/systemd
            readOnly: true
          - name: etc-kubernetes
            mountPath: /etc/kubernetes
            readOnly: true
      restartPolicy: Never
      volumes:
      - name: var-lib-kubelet
        hostPath:
          path: "/var/lib/kubelet"
      - name: etc-systemd
        hostPath:
          path: "/etc/systemd"
      - name: etc-kubernetes
        hostPath:
          path: "/etc/kubernetes"
```

### Running as a DaemonSet

To run kube-bench on every node in your cluster, use a DaemonSet:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-bench
  namespace: security
spec:
  selector:
    matchLabels:
      app: kube-bench
  template:
    metadata:
      labels:
        app: kube-bench
    spec:
      hostPID: true
      containers:
      - name: kube-bench
        image: aquasec/kube-bench:latest
        command: ["kube-bench", "--json", "--logtostderr=true", "node"]
        securityContext:
          privileged: true
        volumeMounts:
        - name: var-lib-kubelet
          mountPath: /var/lib/kubelet
          readOnly: true
        - name: etc-systemd
          mountPath: /etc/systemd
          readOnly: true
        - name: etc-kubernetes
          mountPath: /etc/kubernetes
          readOnly: true
        - name: output
          mountPath: /output
      volumes:
      - name: var-lib-kubelet
        hostPath:
          path: "/var/lib/kubelet"
      - name: etc-systemd
        hostPath:
          path: "/etc/systemd"
      - name: etc-kubernetes
        hostPath:
          path: "/etc/kubernetes"
      - name: output
        hostPath:
          path: "/tmp/kube-bench"
          type: DirectoryOrCreate
```

### Running Locally on Nodes

For direct execution on a node:

```bash
# If installed as a package
kube-bench

# Using the binary
./kube-bench
```

## Basic Usage

### Running Default Checks

To run all checks against your cluster:

```bash
kube-bench
```

This will automatically detect your Kubernetes version and run the appropriate CIS benchmark checks.

### Targeting Specific Components

You can target specific Kubernetes components:

```bash
# Master node checks
kube-bench run --targets master

# Worker node checks
kube-bench run --targets node

# etcd node checks
kube-bench run --targets etcd

# Multiple targets
kube-bench run --targets master,node

# Control plane components
kube-bench run --targets control-plane

# Policies
kube-bench run --targets policies
```

## Advanced Configuration

### Custom Configuration Files

You can customize the checks using your own configuration files:

```bash
kube-bench --config-dir /path/to/custom/configs
```

The structure should match the default config structure:

```
/path/to/custom/configs/
├── config.yaml
├── controlplane.yaml
├── etcd.yaml
├── master.yaml
├── node.yaml
└── policies.yaml
```

### Excluding Specific Tests

To exclude certain tests:

```bash
kube-bench run --targets master --exclude 1.1.2,1.2.1
```

### Running Specific Test Groups

To run only specific test groups or checks:

```bash
# Run only section 1 tests on master
kube-bench run --targets master --check 1

# Run specific checks
kube-bench run --targets master --check 1.1.1,1.1.2
```

## Output Formats

### Default Output

By default, kube-bench outputs results in a human-readable format:

```bash
kube-bench
```

Example output:
```
[INFO] 1 Master Node Security Configuration
[INFO] 1.1 Master Node Configuration Files
[PASS] 1.1.1 Ensure that the API server pod specification file permissions are set to 644 or more restrictive (Automated)
[FAIL] 1.1.2 Ensure that the API server pod specification file ownership is set to root:root (Automated)
```

### JSON Output

For programmatic processing, use JSON output:

```bash
kube-bench --json > kube-bench-results.json
```

Example structure:
```json
{
  "controls": [
    {
      "id": "1",
      "text": "Master Node Security Configuration",
      "tests": [
        {
          "section": "1.1",
          "type": "manual",
          "pass": true,
          "text": "Ensure that the API server pod specification file permissions are set to 644 or more restrictive"
        }
      ]
    }
  ]
}
```

### JUnit XML for CI/CD

For CI/CD integration, use JUnit XML format:

```bash
kube-bench --junit > kube-bench-results.xml
```

## Interpreting Results

### Understanding Severity Levels

Results are categorized with these severities:

- **PASS**: The check was successful
- **FAIL**: The check failed and needs remediation
- **WARN**: The check found something that might need attention
- **INFO**: Informational only, no action needed
- **NOTE**: Additional information about a test

### Reading Test Output

For each check, kube-bench provides:

1. The CIS benchmark ID (e.g., "1.1.1")
2. Description of the check
3. Result (PASS/FAIL/WARN/INFO)
4. Remediation suggestions for failed checks

### Remediation Steps

For each failed check, kube-bench provides remediation instructions:

```
[FAIL] 1.1.20 Ensure that the Kubernetes PKI directory and file ownership is set to root:root (Automated)
        [remediation]
        Run the below command (based on the file location on your system) on the master node.
        For example,
        chown -R root:root /etc/kubernetes/pki/
```