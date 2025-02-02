# Elastic File System (EFS)

## Introduction

Amazon Elastic File System (EFS) is a managed Network File System (NFS) designed to provide scalable, shared storage for EC2 instances. The service enables multiple EC2 instances to simultaneously access a common file system across multiple Availability Zones (AZs).

## Core Functionality

EFS operates as a fully managed NFS service that supports concurrent mounting from multiple EC2 instances. The system delivers high availability by working across multiple AZs within a region. While it comes at a higher cost point (approximately three times that of gp2), its scalability and pay-per-use pricing model make it suitable for various enterprise applications.

## Use Cases and Compatibility

The service excels in several key applications, including content management systems, web serving, data sharing, and WordPress hosting. EFS implements the NFSv4.1 protocol and is specifically designed for Linux-based AMIs, making it incompatible with Windows environments. The system provides POSIX-compliant file system access with standard file APIs, enabling seamless integration with existing Linux-based applications.

Security is managed through AWS security groups, which control access to EFS resources. Data security is enhanced through KMS-based encryption at rest. The file system's automatic scaling capability eliminates the need for capacity planning, allowing users to pay only for the storage they consume.

## Performance Characteristics

### Scaling Capabilities

EFS demonstrates impressive scaling capabilities, supporting thousands of concurrent NFS clients with throughput exceeding 10 GB/s. The system can automatically expand to petabyte-scale, requiring no manual intervention for growth management.

### Performance Modes

EFS offers two performance modes, which must be selected during creation:

The General Purpose mode serves as the default option, optimized for latency-sensitive applications such as web servers and content management systems. The Max I/O mode caters to high-throughput requirements and parallel processing scenarios, commonly used in big data analytics and media processing, though it may introduce higher latency.

### Throughput Options

EFS provides three throughput modes to match different workload requirements:

The Bursting mode provides baseline throughput of 50MiB/s per TB of storage, with burst capability up to 100MiB/s. Provisioned mode allows setting specific throughput requirements independent of storage size, supporting configurations up to 1 GiB/s per TB of storage. The Elastic mode automatically adjusts throughput based on workload demands, supporting up to 3GiB/s for reads and 1GiB/s for writes, making it ideal for unpredictable workload patterns.

## Storage Classes and Management

### Tiered Storage

EFS implements a lifecycle management feature that enables automatic file movement between storage tiers based on access patterns:

The Standard tier serves frequently accessed files with optimal performance. The Infrequent Access tier (EFS-IA) offers lower storage costs but includes retrieval fees. The Archive tier provides the most cost-effective storage for rarely accessed data, offering up to 50% cost savings for files accessed only a few times annually.

### Availability Options

EFS offers two availability configurations:

The Standard configuration provides Multi-AZ redundancy, making it suitable for production environments. The One Zone configuration limits storage to a single AZ, offering a more cost-effective solution for development environments while maintaining backup capabilities by default. One Zone storage is compatible with IA storage (EFS One Zone-IA) and can provide over 90% cost savings compared to standard configurations.

## EBS vs EFS Comparison

### Storage Characteristics

EFS differs significantly from EBS in several key aspects. While EBS volumes are typically limited to single instance attachment (except for multi-attach io1/io2) and confined to specific AZs, EFS provides simultaneous access to hundreds of instances across multiple AZs. This makes EFS particularly suitable for shared file system requirements, such as WordPress installations.

### Cost and Performance Considerations

Though EFS commands a higher price point than EBS, it offers cost optimization through storage tiers. The decision between EFS, EBS, and Instance Store should consider factors such as access patterns, performance requirements, and budget constraints. EFS's ability to serve as a centralized storage solution can often justify its higher cost through reduced operational complexity and improved data consistency across instances.

The choice between these storage solutions should be guided by specific application requirements, considering factors such as access patterns, performance needs, sharing requirements, and cost constraints.
