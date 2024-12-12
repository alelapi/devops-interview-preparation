# Components of Kubernetes

Kubernetes is composed of several key components that work together to orchestrate containerized applications. Below is a list of the core components, grouped into **control plane components** and **node components**, along with their descriptions.

---

## **Control Plane Components**

The control plane manages the Kubernetes cluster and makes global decisions about scheduling, scaling, and maintaining the cluster's state.

### 1. **API Server (`kube-apiserver`)**

- **Description**: Acts as the central control plane component, exposing the Kubernetes API. It serves as the primary point of communication for users, administrators, and all cluster components.
- **Key Features**:
  - Processes API requests (e.g., `kubectl` commands).
  - Provides authentication, authorization, and admission control.
  - Persists the cluster's state to etcd.
- **Example**: Handles commands like `kubectl apply` to create resources.

### 2. **etcd**

- **Description**: A distributed key-value store used as the primary database for storing all cluster data.
- **Key Features**:
  - Stores information about nodes, Pods, ConfigMaps, Secrets, and more.
  - Ensures data consistency across the cluster.
- **Example**: Stores the desired state of a Deployment and its associated Pods.

### 3. **Scheduler (`kube-scheduler`)**

- **Description**: Determines on which node a Pod should run based on resource requirements, constraints, and available capacity.
- **Key Features**:
  - Uses policies and priorities to select the optimal node.
  - Factors in taints, tolerations, node affinity, and Pod affinity/anti-affinity.
- **Example**: Assigns a Pod to a node with sufficient memory and CPU.

### 4. **Controller Manager (`kube-controller-manager`)**

- **Description**: Runs multiple controllers that regulate the cluster's state by watching the API server and taking action to meet the desired state.
- **Key Controllers**:
  - **Node Controller**: Manages node health and lifecycle.
  - **Replication Controller**: Ensures the correct number of Pod replicas are running.
  - **Service Controller**: Maintains network load balancers for services.
  - **Endpoint Controller**: Updates Endpoints for services.
- **Example**: Ensures a Deployment with three replicas always has three Pods running.

### 5. **Cloud Controller Manager**

- **Description**: Integrates Kubernetes with cloud provider-specific APIs to manage resources like load balancers, storage, and networking.
- **Key Controllers**:
  - **Node Controller**: Manages cloud-based node operations.
  - **Route Controller**: Configures routes in the cloud for cluster networking.
  - **Service Controller**: Manages external load balancers.
- **Example**: Creates a cloud load balancer for a Kubernetes Service of type `LoadBalancer`.

---

## **Node Components**

Node components run on each worker node and manage workloads, ensuring that containers operate as specified.

### 1. **Kubelet**

- **Description**: An agent running on each node that communicates with the API server to ensure containers are running as instructed.
- **Key Features**:
  - Manages Pods and their containers.
  - Monitors Pod health and resource usage.
  - Interacts with the container runtime (e.g., Docker, containerd).
- **Example**: Starts and stops containers as defined in a Pod spec.

### 2. **Kube-Proxy**

- **Description**: A network proxy running on each node to manage networking for services.
- **Key Features**:
  - Implements Kubernetes Services by forwarding traffic to the correct Pods.
  - Supports protocols like TCP, UDP, and SCTP.
- **Example**: Routes external requests to the appropriate backend Pod in a Service.

### 3. **Container Runtime**

- **Description**: The software responsible for running containers on a node.
- **Supported Runtimes**:
  - Docker (deprecated as of Kubernetes 1.20+).
  - containerd.
  - CRI-O.
  - Any runtime that implements the Kubernetes Container Runtime Interface (CRI).
- **Example**: Pulls a container image from a registry and starts it.

---

## **Add-Ons**

Add-ons provide additional functionality that is not part of the core Kubernetes components but is essential for a fully functional cluster.

### 1. **CoreDNS**

- **Description**: Provides DNS for Kubernetes services and Pods.
- **Example**: Resolves service names to IP addresses (e.g., `my-service.default.svc.cluster.local`).

### 2. **Dashboard**

- **Description**: A web-based user interface for managing and monitoring the cluster.
- **Example**: Provides a visual representation of workloads, resources, and cluster status.

### 3. **Metrics Server**

- **Description**: Collects resource usage data (e.g., CPU, memory) for Pods and nodes.
- **Example**: Enables horizontal Pod autoscaling based on CPU usage.

### 4. **Ingress Controller**

- **Description**: Manages HTTP and HTTPS routing to services within the cluster.
- **Example**: Routes external traffic to a service using a custom domain (e.g., `example.com`).

### 5. **Logging and Monitoring Tools**

- **Examples**:
  - **Prometheus/Grafana**: Collect and visualize metrics.
  - **ELK Stack (Elasticsearch, Logstash, Kibana)**: Aggregate and analyze logs.

---

## **Interaction of Components**

1. A user submits a request to the **API server** (e.g., `kubectl apply`).
2. The API server validates the request and persists the desired state in **etcd**.
3. The **Scheduler** assigns the Pod to an appropriate node.
4. The **Controller Manager** ensures the desired state matches the actual state (e.g., launching Pods, scaling Deployments).
5. The **Kubelet** on the assigned node pulls the container image, starts the container, and reports status to the API server.
6. **Kube-Proxy** manages networking so traffic can reach the Pods.

---

## **Conclusion**

These components work together to manage the lifecycle of applications, maintain the desired state, and ensure scalability and high availability in Kubernetes clusters. Understanding these components is essential for effectively deploying and managing containerized workloads.
