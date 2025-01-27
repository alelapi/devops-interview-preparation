# Elastic Compute Cloud (EC2)

## Introduction

Amazon Elastic Compute Cloud (EC2) is a fundamental web service providing scalable, flexible computing capacity in the AWS cloud. It enables users to launch, manage, and scale virtual server instances with complete control over computing resources, supporting a wide range of workloads from simple web hosting to complex enterprise applications.

## Core Architectural Components

### Instance Types

EC2 offers diverse instance categories optimized for specific use cases:

#### General Purpose Instances
Balanced compute, memory, and networking resources suitable for a broad range of workloads. These instances provide an equilibrium between different computational resources, making them versatile for web servers, small databases, and development environments.

#### Compute Optimized Instances
Designed for compute-intensive applications requiring high-performance processors. Ideal for batch processing, media transcoding, high-performance web servers, machine learning inference, and scientific modeling.

#### Memory Optimized Instances
Engineered to deliver fast performance for workloads processing large datasets in memory. Critical for high-performance databases, distributed web scale cache, in-memory analytics, and real-time big data processing.

#### Storage Optimized Instances
Provide high, sequential read/write access to large datasets. Optimal for distributed file systems, data warehousing, and high-frequency online transaction processing (OLTP) systems.

#### Accelerated Computing Instances
Incorporate hardware accelerators or co-processors for specialized computational tasks. Leveraged extensively in machine learning, graphics rendering, and cryptocurrency mining.

## Launch and Management Strategies

### Instance Purchasing Options

#### On-Demand Instances
Pay for compute capacity by the hour or second with no long-term commitments. Provides maximum flexibility for unpredictable workloads and short-term applications.

#### Reserved Instances
Offer significant cost savings by committing to specific instance configurations for 1-3 year terms. Ideal for predictable, steady-state workloads with consistent computational requirements.

#### Spot Instances
Enable purchasing unused EC2 capacity at steep discounts, potentially up to 90% off on-demand pricing. Suitable for fault-tolerant, flexible workloads like batch processing and scientific computing.

#### Dedicated Hosts
Provide physical servers dedicated exclusively to a single customer, addressing complex licensing and compliance requirements.

## Networking Capabilities

### Network Configuration
EC2 instances operate within Virtual Private Clouds (VPCs), offering granular control over network environments. Users can configure:
- IP address ranges
- Subnet creation
- Route table management
- Network gateway configurations

# Security Groups and Network Access Control Lists (NACLs)

Both Security Groups and Network Access Control Lists (NACLs) are integral components of AWS networking, providing security at different levels within a Virtual Private Cloud (VPC). Here's a detailed explanation of both:

---

## **Security Groups**

### **What are Security Groups?**
- **Stateful** firewalls that operate at the instance level.
- Act as virtual firewalls to control inbound and outbound traffic to and from Amazon EC2 instances.
- Rules can allow or deny traffic based on IP ranges, protocols, and ports.

### **Key Characteristics:**
1. **Instance-Level**: Security Groups are attached directly to EC2 instances.
2. **Stateful**:
   - If you allow inbound traffic, the corresponding outbound traffic is automatically allowed, and vice versa.
3. **Implicit Deny**:
   - By default, all inbound traffic is denied, and all outbound traffic is allowed unless explicitly specified otherwise.
4. **Rule-Based Configuration**:
   - You define rules to allow traffic. Deny rules cannot be explicitly created.
5. **Supports Allow Only**: 
   - Rules define what traffic is permitted, with no option to explicitly block traffic.
6. **Dynamic Updates**:
   - Modifications to a Security Group are automatically applied to all associated resources.

### **Common Use Cases:**
- Control access to EC2 instances based on port (e.g., 22 for SSH, 80/443 for HTTP/HTTPS).
- Restrict traffic to specific IP ranges or other resources within the same VPC.

### **Example Rules:**
| **Type**      | **Protocol** | **Port Range** | **Source**       |
|---------------|--------------|----------------|------------------|
| SSH           | TCP          | 22             | 198.51.100.1/32  |
| HTTP          | TCP          | 80             | 0.0.0.0/0        |
| HTTPS         | TCP          | 443            | 0.0.0.0/0        |

---

## **Network Access Control Lists (NACLs)**

### **What are NACLs?**
- **Stateless** firewalls that operate at the subnet level within a VPC.
- Used to control traffic entering and leaving subnets.

### **Key Characteristics:**
1. **Subnet-Level**: NACLs apply to all resources within the associated subnet(s).
2. **Stateless**:
   - Both inbound and outbound rules must be defined explicitly. Return traffic is not automatically allowed.
3. **Explicit Rules for Allow and Deny**:
   - Unlike Security Groups, NACLs support both `allow` and `deny` rules.
4. **Rules are Evaluated in Order**:
   - Each rule is evaluated in numerical order, starting from the lowest rule number.
   - Traffic is allowed or denied based on the first matching rule.
5. **Default Behavior**:
   - Default NACL allows all inbound and outbound traffic.
   - Custom NACLs deny all traffic unless rules are explicitly added.

### **Common Use Cases:**
- Apply coarse-grained security controls at the subnet level.
- Use as an additional layer of security alongside Security Groups.
- Block specific IP addresses or ranges.

### **Example Rules:**
| **Rule #** | **Type** | **Protocol** | **Port Range** | **Source**        | **Allow/Deny** |
|------------|----------|--------------|----------------|-------------------|----------------|
| 100        | HTTP     | TCP          | 80             | 0.0.0.0/0         | Allow          |
| 200        | SSH      | TCP          | 22             | 203.0.113.0/24    | Allow          |
| 300        | All Traffic | All       | All            | 0.0.0.0/0         | Deny           |

---

## **Comparison Between Security Groups and NACLs**

| **Aspect**                  | **Security Groups**            | **NACLs**                       |
|-----------------------------|--------------------------------|----------------------------------|
| **Level of Operation**       | Instance Level                | Subnet Level                    |
| **Statefulness**             | Stateful                      | Stateless                       |
| **Allow/Deny Rules**         | Allow only                    | Allow and Deny                  |
| **Evaluation of Rules**      | All rules evaluated equally   | Rules evaluated in order        |
| **Direction of Traffic**     | Separate inbound and outbound | Separate inbound and outbound   |
| **Default Behavior**         | Inbound: Deny, Outbound: Allow | Inbound & Outbound: Allow all (default NACL) |

---

## **Best Practices**

1. **Use Both**: Combine Security Groups and NACLs for a layered defense approach.
2. **Least Privilege**: Only allow traffic that is necessary for your application.
3. **Organize Rule Sets**:
   - Use Security Groups for instance-level control.
   - Use NACLs to block/allow traffic at the subnet level.
4. **Monitor and Audit**:
   - Regularly review and update rules to ensure compliance and avoid unnecessary exposure.
5. **Default Rules**:
   - Ensure custom NACLs deny all traffic by default until rules are explicitly added.

By leveraging Security Groups and NACLs together, you can implement a robust and secure network architecture in AWS.


## Storage Options

### Amazon EBS (Elastic Block Store)
Persistent block-level storage volumes attachable to EC2 instances. Supports various volume types:
- General Purpose SSD
- Provisioned IOPS SSD
- Throughput Optimized HDD
- Cold HDD

### Instance Store
Temporary block-level storage directly attached to the host computer, providing high I/O performance for temporary data.

## Monitoring and Management

### Amazon CloudWatch
Provides comprehensive monitoring capabilities:
- Performance metrics
- Resource utilization tracking
- Automated scaling
- Custom metric creation

### AWS Systems Manager
Enables centralized operational management across AWS resources, facilitating:
- Configuration management
- Patch management
- Automated tasks
- Compliance tracking

## Pricing Model

### Factors Influencing Cost
- Instance type
- Region
- Operating system
- Purchasing option
- Additional services and data transfer

## Use Cases

### Enterprise Applications
- Web hosting
- Enterprise application servers
- Development and testing environments

### Scientific Computing
- High-performance computing
- Genomic research
- Climate modeling

### Media Processing
- Video rendering
- Transcoding
- Content delivery

## Best Practices

- Right-size instances based on workload
- Implement auto-scaling
- Utilize multiple availability zones
- Leverage appropriate purchasing models
- Implement robust security configurations
- Continuously monitor performance metrics

## Limitations and Considerations

- Maximum of 20 On-Demand instances per region
- Specific service quotas and limits
- Regional availability variations
- Potential data transfer costs

## Conclusion

AWS EC2 represents a powerful, flexible computing platform enabling organizations to scale computational resources dynamically, efficiently, and cost-effectively. By understanding its comprehensive capabilities, users can design robust, scalable cloud architectures tailored to diverse computational requirements.
