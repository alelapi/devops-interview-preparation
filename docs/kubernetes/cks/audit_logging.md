# Audit Logging

Audit logging in Kubernetes is crucial for security monitoring and compliance. This guide explains how to configure the API server to enable comprehensive audit logging.

## Create an Audit Policy File

First, create an audit policy file that defines what events should be recorded:

```bash
mkdir -p /etc/kubernetes/audit
```

Create the audit policy file at `/etc/kubernetes/audit/policy.yaml`:

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
  # Log pod changes at RequestResponse level
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["pods"]
      
  # Log persistent volume changes
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["persistentvolumes"]
      verbs: ["create", "delete", "update"]
      
  # Log auth at Metadata level
  - level: Metadata
    resources:
    - group: "authentication.k8s.io"
      resources: ["*"]
      
  # Log all other resources at the Metadata level
  - level: Metadata
    resources:
    - group: ""
      resources: ["*"]
    - group: "apps"
      resources: ["*"]
    - group: "rbac.authorization.k8s.io"
      resources: ["*"]
      
  # A catch-all rule to log all other events at the Metadata level
  - level: Metadata
    omitStages:
      - "RequestReceived"
```

## Configure the API Server

```bash
nano /etc/kubernetes/manifests/kube-apiserver.yaml
```

Add the audit policy and log path parameters:

```yaml
spec:
  containers:
  - command:
    - kube-apiserver
    - --audit-policy-file=/etc/kubernetes/audit/policy.yaml
    - --audit-log-path=/var/log/kubernetes/audit/audit.log
    - --audit-log-maxage=30
    - --audit-log-maxbackup=10
    - --audit-log-maxsize=100
    # ... other existing parameters
    volumeMounts:
    - mountPath: /etc/kubernetes/audit
      name: audit-config
      readOnly: true
    - mountPath: /var/log/kubernetes/audit
      name: audit-log
  volumes:
  - hostPath:
      path: /etc/kubernetes/audit
      type: DirectoryOrCreate
    name: audit-config
  - hostPath:
      path: /var/log/kubernetes/audit
      type: DirectoryOrCreate
    name: audit-log
```

## Audit Log Levels

The policy file uses these audit levels:

- **None**: Don't log events matching this rule
- **Metadata**: Log request metadata but not request or response body
- **Request**: Log event metadata and request body
- **RequestResponse**: Log event metadata, request and response bodies

## Advanced Configuration

### Configure Log Backend Format

For JSON format (better for processing):

```
--audit-log-format=json
```

### Webhook Backend

To send audit logs to an external webhook:

```
--audit-webhook-config-file=/etc/kubernetes/audit/webhook-config.yaml
--audit-webhook-batch-max-size=10000
--audit-webhook-batch-max-wait=5s
```

Example webhook configuration:

```yaml
apiVersion: v1
kind: Config
clusters:
- name: audit-webhook
  cluster:
    server: https://audit.example.com/webhook
contexts:
- context:
    cluster: audit-webhook
    user: ""
  name: default-context
current-context: default-context
preferences: {}
users: []
```

