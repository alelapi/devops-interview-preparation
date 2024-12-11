# Pod Security Admission (PSA)

**Pod Security Admission (PSA)** is a Kubernetes feature that enforces security policies at the namespace level to control how Pods are created and managed based on predefined security standards. It is the successor to the deprecated **Pod Security Policies (PSPs)** and provides a simpler way to apply security controls.

## Purpose

- To enforce security best practices for Kubernetes Pods.
- To prevent potentially unsafe Pod configurations (e.g., privilege escalation, use of host namespaces).

## How It Works

- PSA evaluates Pod specifications during the admission phase (before the Pod is created) to ensure compliance with the security standards.
- Security policies are defined by labeling namespaces with one of three predefined security levels:
  - **Privileged**: Minimal restrictions, suitable for trusted environments.
  - **Baseline**: Basic restrictions to enforce reasonable security defaults.
  - **Restricted**: Strong restrictions for high-security environments.

## Key Features

1. **Namespace-Level Control**:
   - PSA applies policies based on namespace labels, making it simple to manage security across the cluster.
2. **Three Modes**:
   - **Enforce**: Rejects Pods that violate the policy.
   - **Audit**: Logs violations but does not block Pod creation.
   - **Warn**: Issues warnings to users creating non-compliant Pods.

## Example Namespace Labels

```bash
kubectl label namespace dev pod-security.kubernetes.io/enforce=restricted
kubectl label namespace dev pod-security.kubernetes.io/audit=baseline
kubectl label namespace dev pod-security.kubernetes.io/warn=privileged
```
