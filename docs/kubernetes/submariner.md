# Submariner

**Submariner** is a tool that facilitates **network connectivity across multiple Kubernetes clusters**. It provides secure and seamless communication between Pods and Services across different clusters, even if they are in separate networks.

## Key Features of Submariner

1. **Cross-Cluster Networking**:

   - Establishes network connectivity between Kubernetes clusters without requiring them to share the same network.

2. **Service Discovery**:

   - Enables Pods in one cluster to discover and communicate with Services in another cluster.

3. **Secure Communication**:

   - Uses IPsec or WireGuard for secure communication between clusters.

4. **Load Balancing**:
   - Provides efficient load balancing for cross-cluster traffic.

## Use Cases

- Multi-cluster deployments where clusters are in different networks or regions.
- Enabling cross-cluster communication for hybrid or multi-cloud setups.
- Building multi-cluster service meshes for advanced traffic control.

## How It Works

- **Gateway Nodes**: Submariner designates a gateway node in each cluster to handle inter-cluster traffic.
- **Tunnel Creation**: Secure tunnels are established between the gateway nodes of participating clusters.
- **Routing**: Submariner ensures that Pods and Services can route traffic seamlessly across clusters.

## Example: Service Connectivity

A Pod in Cluster A can directly access a Service in Cluster B using the standard Service DNS name, such as `my-service.my-namespace.svc.cluster.local`.
