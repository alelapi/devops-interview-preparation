# Kubernetes Auditing Policy

Kubernetes auditing provides a security-relevant chronological set of records documenting the sequence of activities that have affected the system. The audit policy defines what events should be recorded and what data they should include.

## Audit Policy Configuration

Kubernetes audit policies are defined in YAML format. The policy specifies rules that determine what events should be recorded and the level of detail.

### Audit Levels

Kubernetes supports these audit levels, from least to most verbose:

- **None**: Don't log events matching this rule
- **Metadata**: Log request metadata (user, timestamp, resource, verb) but not request or response body
- **Request**: Log event metadata and request body but not response body
- **RequestResponse**: Log event metadata, request and response bodies

### Example Audit Policy

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
  # Log pod changes at RequestResponse level
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["pods"]

  # Log "configmaps" and "secrets" at Metadata level
  - level: Metadata
    resources:
    - group: ""
      resources: ["configmaps", "secrets"]

  # Don't log requests to certain non-resource URL paths
  - level: None
    nonResourceURLs:
    - /api*
    - /version
    - /healthz

  # Log everything else at Metadata level
  - level: Metadata
```

## Implementing Audit Logging

To enable audit logging in Kubernetes, you need to:

1. **Configure the API server** with audit policy file:
   ```
   --audit-policy-file=/etc/kubernetes/audit-policy.yaml
   ```

2. **Specify the log backend**:
   - Log to file:
     ```
     --audit-log-path=/var/log/kubernetes/audit.log
     --audit-log-maxage=30
     --audit-log-maxbackup=10
     ```
   - Or log to webhook:
     ```
     --audit-webhook-config-file=/etc/kubernetes/audit-webhook.yaml
     ```

## Best Practices

1. **Focus on sensitive operations**: Privilege escalation, auth failures, resource deletion
2. **Be selective**: Logging everything at RequestResponse level causes significant overhead
3. **Consider log storage and rotation**: Audit logs can grow very large
4. **Monitor audit logs**: Integrate with security monitoring tools like SIEM systems
5. **Implement a graduated approach**: Log sensitive operations at RequestResponse level, less sensitive at Metadata

Properly configured audit logging is critical for security incident detection, forensic investigations, and compliance in Kubernetes clusters.