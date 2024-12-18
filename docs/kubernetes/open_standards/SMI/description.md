# Service Mesh Interface (SMI)

The **Service Mesh Interface (SMI)** is an open standard for managing service-to-service communication in Kubernetes. It provides a consistent and portable way to integrate service meshes like Linkerd, Istio, and Consul Connect into Kubernetes clusters. SMI standardizes APIs for traffic management, observability, and security.

Below is an example demonstrating **SMI Traffic Split**, one of the core SMI capabilities.

---

## **Traffic Split Example**

### **Use Case**:

A gradual rollout (canary release) of a new version of a microservice while splitting traffic between two versions.

---

### **1. Prerequisites**:

- A Kubernetes cluster with a service mesh like **Linkerd** or **Istio** installed.
- Two deployments of the same service:
  - `v1` for the current stable version.
  - `v2` for the new version being tested.

---

### **2. Deployments and Services**

#### **Deployment for v1**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
      version: v1
  template:
    metadata:
      labels:
        app: backend
        version: v1
    spec:
      containers:
        - name: backend
          image: my-backend:v1
          ports:
            - containerPort: 80
```

#### **Deployment for v2**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
      version: v2
  template:
    metadata:
      labels:
        app: backend
        version: v2
    spec:
      containers:
        - name: backend
          image: my-backend:v2
          ports:
            - containerPort: 80
```

#### **Service Definition**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 80
```

---

### **3. SMI Traffic Split**

The following Traffic Split definition directs 90% of the traffic to version `v1` of the backend and 10% of the traffic to version `v2`. As confidence in `v2` grows, the weights can be adjusted gradually.

```yaml
apiVersion: split.smi-spec.io/v1alpha2
kind: TrafficSplit
metadata:
  name: backend-traffic-split
spec:
  service: backend-service
  backends:
    - service: backend-v1
      weight: 90
    - service: backend-v2
      weight: 10
```

### **Explanation**:

- **service**: Refers to the Kubernetes service (`backend-service`) that acts as the main entry point.
- **backends**: Defines the traffic distribution between the two versions (`backend-v1` and `backend-v2`) using **weights**.

---

## **4. Observability**

With **SMI** observability tools integrated into your service mesh (like Linkerd), you can monitor:

- Traffic metrics between `v1` and `v2`.
- Response times and error rates.

Example using Linkerd CLI:

```bash
linkerd stat traffic-split backend-traffic-split
```

---

## **Conclusion**

The **SMI Traffic Split** API provides a standardized way to manage traffic between different versions of a service. It simplifies gradual rollouts, canary releases, and A/B testing in Kubernetes clusters, ensuring smooth service mesh interoperability regardless of the underlying implementation (Linkerd, Istio, Consul Connect).
