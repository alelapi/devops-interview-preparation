# Admission Controller

An **Admission Controller** is a component in Kubernetes that intercepts API requests to the Kubernetes API server before the objects are persisted in etcd. Admission controllers can modify, validate, or reject these requests based on custom logic or policies.

## Purpose

- To enforce policies and best practices for resources created or updated in the Kubernetes cluster.
- To validate and mutate incoming API requests.

## How It Works

1. A user sends a request to the Kubernetes API server (e.g., create a Pod or Deployment).
2. The request goes through authentication and authorization checks.
3. The request is processed by **admission controllers**, which:
   - Mutate the request (e.g., add default values or labels).
   - Validate the request against policies.
   - Approve or reject the request based on the outcome.

## Types of Admission Controllers

1. **Mutating Admission Controllers**:

   - Modify the incoming request before it is persisted.
   - Example: Adding default resource limits to Pods.

2. **Validating Admission Controllers**:

   - Validate the request and either approve or reject it.
   - Example: Ensuring that Pods do not use privileged containers.

## Built-in Admission Controllers

Some common admission controllers in Kubernetes include:

- **PodSecurity**: Implements Pod Security Admission (PSA).
- **NamespaceLifecycle**: Ensures that objects are created only in active namespaces.
- **LimitRanger**: Enforces resource limits on Pods and containers.
- **ResourceQuota**: Ensures that resource quotas are not exceeded in a namespace.

## Custom Admission Controllers

- Kubernetes allows you to define **Dynamic Admission Controllers** using **Admission Webhooks**.
- **MutatingAdmissionWebhook** and **ValidatingAdmissionWebhook** allow you to create custom logic to process API requests.
