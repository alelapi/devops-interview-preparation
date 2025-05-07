# OPA and Gatekeeper

## Introduction

Open Policy Agent (OPA) is a general-purpose policy engine that enables unified policy enforcement across the entire stack. When combined with Gatekeeper, OPA becomes a powerful tool for enforcing policies in Kubernetes environments. This guide will take you through practical implementations of OPA and Gatekeeper, from basic concepts to advanced use cases, preparing you for real-world scenarios and practical exams.

## Understanding OPA and Gatekeeper

OPA is a general-purpose policy engine that enables unified policy enforcement across various systems. Gatekeeper is the Kubernetes-native implementation of OPA, providing a way to enforce policies on Kubernetes resources during creation and update operations.

### Key Concepts

- **Policy as Code**: Define policies in a declarative language (Rego) rather than hardcoding them in applications.
- **Admission Control**: Gatekeeper works as a validating admission controller in Kubernetes.
- **Constraint Framework**: A system for defining, applying, and monitoring policy compliance.

### The Relationship Between OPA and Gatekeeper

Gatekeeper extends OPA's capabilities specifically for Kubernetes:

- It provides Kubernetes custom resources for defining policies
- It integrates with the Kubernetes admission controller
- It enables audit functionality for existing resources

## Installing Gatekeeper

Let's begin with installing Gatekeeper on a Kubernetes cluster:

### Using Helm

```bash
# Add the Gatekeeper Helm repository
helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts

# Update the Helm repositories
helm repo update

# Install Gatekeeper
helm install gatekeeper gatekeeper/gatekeeper --namespace gatekeeper-system --create-namespace
```

### Using kubectl

```bash
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.11/deploy/gatekeeper.yaml
```

### Verify the Installation

```bash
kubectl get pods -n gatekeeper-system
```

You should see pods like `gatekeeper-audit-*`, `gatekeeper-controller-manager-*`, etc., all in a Running state.

## ConstraintTemplates and Constraints

Gatekeeper uses two custom resources to define and enforce policies:

### ConstraintTemplates

A ConstraintTemplate defines the logic of the policy using Rego and the schema for the Constraint that will implement the policy. Think of it as a reusable policy definition that can be applied in different contexts.

#### Anatomy of a ConstraintTemplate

```yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels  # Name of the template
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels  # Kind of the Constraint this template creates
      validation:
        # Schema for the `parameters` field in the Constraint
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels

        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("you must provide labels: %v", [missing])
        }
```

Let's break this down:

1. **CRD Specification**: Defines the kind name and validation schema for the Constraint.
2. **Target**: Specifies where the policy will be enforced. For Kubernetes, this is usually `admission.k8s.gatekeeper.sh`.
3. **Rego Policy**: The policy logic written in Rego language.

### Constraints

A Constraint is an instance of a ConstraintTemplate that enforces the policy defined in the template. It allows you to specify which resources the policy should apply to and provide parameters to customize the policy.

#### Example Constraint

```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels  # This must match the kind defined in the ConstraintTemplate
metadata:
  name: require-team-label
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]  # This policy applies to Namespaces
  parameters:
    labels: ["team"]  # Require 'team' label on all Namespaces
```

Let's break this down:

1. **Kind**: Must match the kind defined in the ConstraintTemplate.
2. **Match**: Defines which resources the Constraint applies to.
3. **Parameters**: Values passed to the Rego policy in the ConstraintTemplate.

## Practical Examples

Let's walk through some practical examples of ConstraintTemplates and Constraints that solve real-world problems:

### Example 1: Require Specific Labels

This example ensures that all Namespaces have required labels:

```yaml
# First, create the ConstraintTemplate
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels

        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("you must provide labels: %v", [missing])
        }
```

```yaml
# Then, create the Constraint
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: ns-must-have-env
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels: ["environment"]
```

Now, if you try to create a Namespace without the `environment` label:

```bash
kubectl create namespace test
```

It will fail with an error like:

```
Error from server ([ns-must-have-env] you must provide labels: {"environment"}): admission webhook "validation.gatekeeper.sh" denied the request: [ns-must-have-env] you must provide labels: {"environment"}
```

### Example 2: Block Latest Image Tags

This example blocks the use of the `:latest` tag for container images:

```yaml
# ConstraintTemplate
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8sblocklatestimages
spec:
  crd:
    spec:
      names:
        kind: K8sBlockLatestImages
      validation:
        openAPIV3Schema:
          type: object
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sblocklatestimages

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          endswith(container.image, ":latest")
          msg := sprintf("container <%v> uses the latest tag", [container.name])
        }

        violation[{"msg": msg}] {
          container := input.review.object.spec.initContainers[_]
          endswith(container.image, ":latest")
          msg := sprintf("initContainer <%v> uses the latest tag", [container.name])
        }
```

```yaml
# Constraint
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sBlockLatestImages
metadata:
  name: block-latest-images
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
      - apiGroups: ["apps"]
        kinds: ["Deployment", "StatefulSet", "DaemonSet"]
```

### Example 3: Require Resource Limits

This example ensures all containers have CPU and memory limits:

```yaml
# ConstraintTemplate
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequirelimits
spec:
  crd:
    spec:
      names:
        kind: K8sRequireLimits
      validation:
        openAPIV3Schema:
          type: object
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequirelimits

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits.cpu
          msg := sprintf("container <%v> has no CPU limit", [container.name])
        }

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits.memory
          msg := sprintf("container <%v> has no memory limit", [container.name])
        }
```

```yaml
# Constraint
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequireLimits
metadata:
  name: require-resource-limits
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
      - apiGroups: ["apps"]
        kinds: ["Deployment", "StatefulSet"]
```

## Advanced Use Cases

### Mutating Admission Control

While traditionally Gatekeeper has been used for validation, recent versions support mutation as well. Here's an example of a MutatingWebhook that adds default resource limits:

```yaml
apiVersion: mutations.gatekeeper.sh/v1
kind: AssignMetadata
metadata:
  name: add-team-label
spec:
  match:
    scope: Namespaced
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  location: "metadata.labels.team"
  parameters:
    assign:
      value: "default-team"
```

### External Data

Gatekeeper can use data from external sources through the `data.inventory` object:

```yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8suniqueservices
spec:
  crd:
    spec:
      names:
        kind: K8sUniqueServices
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8suniqueservices

        violation[{"msg": msg}] {
          input.review.kind.kind == "Service"
          input.review.object.spec.type == "LoadBalancer"
          port := input.review.object.spec.ports[_]
          existing := data.inventory.namespace[namespace].Service[name]
          existing.spec.type == "LoadBalancer"
          existing_port := existing.spec.ports[_]
          port.port == existing_port.port
          not (input.review.object.metadata.name == existing.metadata.name)
          msg := sprintf("Service port %v already in use by service %v", [port.port, existing.metadata.name])
        }
```

### Exemptions

You can create exemptions for your policies using the `excludedNamespaces` field in the Constraint:

```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequireLimits
metadata:
  name: require-resource-limits
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    excludedNamespaces: ["kube-system", "gatekeeper-system"]
```


