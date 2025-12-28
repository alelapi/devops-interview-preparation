# Cloud Storage Overview

## Description

Cloud Storage is Google Cloud's object storage service for storing and retrieving unstructured data at any scale. Understanding when to use object storage versus block storage (Persistent Disks) or file storage (Filestore) is fundamental to cloud architecture.

**Key Principle**: Object storage is for massive-scale, infrequently changing data accessed via HTTP/S; not for databases or file systems requiring POSIX operations.

## Core Concepts

### Object Storage Model

**Architecture**:

- **Buckets**: Top-level containers for objects
- **Objects**: Individual files with metadata
- **Flat namespace**: No directory structure (though prefixes simulate folders)
- **HTTP/S access**: RESTful API, not filesystem mount

**Characteristics**:

- Unlimited capacity
- Strongly consistent (read-after-write)
- Eventually consistent for IAM/ACL changes
- Atomic operations
- No minimum object size
- Maximum object size: 5 TB

### Buckets

**Bucket Properties**:

- Globally unique name
- Geographic location (region, dual-region, multi-region)
- Storage class (default for objects)
- Access control method (IAM, ACL, or both)
- Lifecycle policies
- Versioning settings

**Immutable Properties** (cannot change after creation):

- Bucket name
- Location type (region/dual-region/multi-region)
- Specific location

**Architecture Impact**: Location and name decisions are permanent; plan carefully

### Objects

**Object Components**:

- Data (the file content)
- Metadata (key-value pairs)
- Access control (IAM, ACLs)
- Generation number (versioning)

**Object Metadata**:

- System metadata (size, content-type, timestamps)
- Custom metadata (user-defined key-value pairs)
- Object generation (version identifier)
- Metageneration (metadata version)

## Durability and Availability

### Durability: 99.999999999% (11 9's)

**What This Means**:

- Store 10 million objects
- Expect to lose 1 object every 10,000 years
- Designed for no data loss
- Automatic replication and erasure coding

**Architecture Implication**: Cloud Storage is more durable than any self-managed solution

### Availability Varies by Location

**Multi-Region**: 99.95% SLA

- Data in at least two geographic locations
- >160 km apart
- Automatic failover
- Use for: Global applications, highest availability

**Dual-Region**: 99.95% SLA

- Data in two specific regions
- Choose regions for latency/compliance
- Balance between cost and availability
- Use for: Regional applications with HA requirements

**Region**: 99.9% SLA

- Data in single region (across 3 zones)
- Lower cost than multi/dual-region
- Lower availability than multi/dual-region
- Use for: Regional data, cost optimization

**Architecture Decision**: Balance availability requirements with cost

## Strong Consistency

### Read-After-Write Consistency

**Guarantees**:

- Object immediately available after write
- Listing immediately shows new objects
- Deletes immediately reflected
- No eventual consistency delays

**Architectural Benefits**:

- Simplifies application logic
- No need to handle stale reads
- Reliable for build artifacts and CI/CD
- Safe for concurrent access patterns

**Exceptions**:

- IAM changes: Up to 60 seconds
- Bucket configuration: Up to 60 seconds

## When to Use Cloud Storage

### ✅ Use Cloud Storage When:

**Unstructured Data Storage**:

- Media files (images, videos, audio)
- Document storage
- Backups and archives
- Log files and analytics data
- Build artifacts and binaries

**Static Website Hosting**:

- HTML, CSS, JavaScript files
- Static assets
- Public documentation
- Download repositories

**Data Lake / Analytics**:

- Raw data ingestion
- Data warehouse staging
- BigQuery external tables
- Dataflow input/output

**Backup and Archive**:

- Database backups
- VM images and snapshots (backend storage)
- Long-term archival
- Compliance data retention

**Content Distribution**:

- Cloud CDN origin
- Software distribution
- Global asset delivery
- User-generated content

### ❌ Don't Use Cloud Storage When:

**Database Storage**:

- Use Cloud SQL, Spanner, Firestore
- Object storage not optimized for transactional data
- No query language support
- Not ACID compliant

**File System Requirements**:

- Applications expecting POSIX filesystem
- Random writes within files
- File locking mechanisms
- Use Filestore (NFS) instead

**Block Storage for VMs**:

- VM boot disks
- Database data files
- High-IOPS applications
- Use Persistent Disks instead

**Real-Time Streaming**:

- Message queues
- Real-time event streaming
- Use Pub/Sub instead

**Frequent Small Updates**:

- Collaborative editing
- Append operations
- Partial object updates
- Use database or Firestore

## Cloud Storage vs Alternatives

### Cloud Storage vs Persistent Disks

| Feature | Cloud Storage | Persistent Disk |
|---------|--------------|-----------------|
| **Access Method** | HTTP/S API | Block device (mounted) |
| **Use Case** | Unstructured objects | VM storage, databases |
| **Performance** | High throughput | High IOPS |
| **Capacity** | Unlimited | Up to 64 TB per disk |
| **Cost** | Based on storage class | Based on disk type |
| **Attachment** | Any number of clients | Limited per VM |
| **Snapshot** | Object versioning | Incremental snapshots |

**Decision**: Use Cloud Storage for objects, Persistent Disks for databases and VMs

### Cloud Storage vs Filestore

| Feature | Cloud Storage | Filestore |
|---------|--------------|-----------|
| **Protocol** | HTTP/S | NFS |
| **Consistency** | Object-level | POSIX filesystem |
| **Use Case** | Object storage | Shared file storage |
| **Scale** | Unlimited | Up to 100 TB |
| **Performance** | Throughput optimized | IOPS optimized |
| **Mounting** | API/SDK | NFS mount |

**Decision**: Use Cloud Storage for objects, Filestore for POSIX filesystem requirements

### Cloud Storage vs Cloud SQL

| Feature | Cloud Storage | Cloud SQL |
|---------|--------------|-----------|
| **Data Model** | Objects | Relational tables |
| **Query** | List/Get by name | SQL queries |
| **Transactions** | None | ACID compliant |
| **Use Case** | Unstructured data | Structured data |
| **Consistency** | Strong per object | Transactional |

**Decision**: Use Cloud Storage for files, Cloud SQL for relational data

## Location Types

### Multi-Region

**Characteristics**:

- At least two geographic areas (>160 km apart)
- Highest availability (99.95%)
- Geo-redundant
- Higher cost

**Available Multi-Regions**:

- US (multiple US locations)
- EU (multiple Europe locations)
- ASIA (multiple Asia locations)

**Use Cases**:

- Global applications
- Highest availability requirements
- Content distribution
- Disaster recovery

**Cost**: Highest storage cost, no egress within multi-region

### Dual-Region

**Characteristics**:

- Two specific regions
- 99.95% availability
- Geo-redundant
- Choose regions for compliance/latency
- Turbo replication option (async replication <15 min)

**Examples**:

- NAM4 (Iowa + South Carolina)
- EUR4 (Netherlands + Finland)

**Use Cases**:

- Regional applications with HA
- Data residency requirements
- Balance cost and availability
- Specific latency requirements

**Cost**: Between multi-region and region

### Region

**Characteristics**:

- Single region (3 zones)
- 99.9% availability
- Zone-redundant within region
- Lowest cost

**Use Cases**:

- Regional applications
- Cost optimization
- Data locality requirements
- Compute in same region (lower latency, no egress)

**Cost**: Lowest storage cost

## Access Control

### IAM (Recommended)

**Characteristics**:

- Bucket-level and project-level permissions
- Fine-grained roles
- Condition-based access
- Integration with organization policies
- Audit logging

**Roles**:

- `roles/storage.objectViewer`: Read objects
- `roles/storage.objectCreator`: Create objects
- `roles/storage.objectAdmin`: Full object control
- `roles/storage.admin`: Full bucket control

**Use Cases**:

- Modern applications
- Service-to-service access
- Centralized access control
- Conditional access policies

### ACLs (Legacy)

**Characteristics**:

- Object-level and bucket-level
- Simpler but less flexible
- Compatible with S3 ACLs
- Being phased out

**Use Cases**:

- Legacy applications
- S3 compatibility requirements
- Specific object-level permissions
- Not recommended for new applications

### Uniform Bucket-Level Access

**Recommendation**: Enable uniform bucket-level access (IAM only, no ACLs)

**Benefits**:

- Simplified permission model
- Better security
- Easier auditing
- Organization policy enforcement

**Architecture Decision**: Use IAM with uniform bucket-level access for all new buckets

## Signed URLs

### Use Cases

**Temporary Access**:

- Time-limited access to private objects
- No authentication required
- Share with external users
- Download links with expiration

**Architecture Pattern**:

```
Application (with credentials) → Generate signed URL → Share URL → User downloads directly
```

**Benefits**:

- No proxy through application server
- Direct download from Cloud Storage
- Reduced infrastructure costs
- Better performance

**Time Limits**:

- Maximum 7 days (service account key)
- Maximum 12 hours (user credentials)
- Set appropriate expiration for use case

## Performance Characteristics

### Throughput

**Capabilities**:

- Multi-gigabit per second throughput
- Scales with parallel requests
- No bottleneck for large files
- Optimized for bandwidth

**Optimization**:

- Parallel uploads for large files
- Multiple threads/workers
- Composite uploads for >32 MB files

### Request Rate Limits

**Bucket Limits** (sustained):

- 5,000 writes per second
- 50,000 reads per second

**Object Limits**:

- 1,000 operations per second per object

**Architecture Impact**:

- Design to avoid hotspots (single object)
- Use object prefixes for distribution
- Consider request rate in architecture

### Latency

**Typical Latency**:

- Single-digit milliseconds (same region)
- Tens of milliseconds (cross-region)
- Sub-second for first byte

**Optimizations**:

- Collocate compute and storage
- Use Cloud CDN for global access
- Minimize request overhead

## Cost Model

### Storage Costs

**Pricing by Class** (per GB/month):

- Standard: ~$0.020
- Nearline: ~$0.010
- Coldline: ~$0.004
- Archive: ~$0.0012

**Variations**:

- Multi-region: Higher than region
- Region: Lowest
- Dual-region: Between multi and region

### Operation Costs

**Class A Operations** (writes, lists): ~$0.05 per 10,000

- Insert, update, list
- Lifecycle transitions
- Composition operations

**Class B Operations** (reads): ~$0.004 per 10,000

- Get object
- Get metadata

**Free Operations**:

- Delete
- Get bucket metadata (non-object)

### Network Costs

**Egress** (data out):

- Same location (region/multi-region): Free
- Cross-region: ~$0.01-$0.12 per GB
- To internet: ~$0.12 per GB (first GB free)
- GCP services same region: Free
- GCP services cross-region: Charged

**Ingress** (data in): Free

### Retrieval Costs (Non-Standard Classes)

**Nearline**: ~$0.01 per GB retrieved
**Coldline**: ~$0.02 per GB retrieved
**Archive**: ~$0.05 per GB retrieved

**Architecture Impact**: Factor retrieval costs into storage class selection

## Integration Patterns

### With Compute Services

**Compute Engine**: Upload/download via gsutil or API
**GKE**: Mount via Cloud Storage FUSE or S3 API
**Cloud Functions**: Triggered by object changes, direct access
**Cloud Run**: Access via client libraries

### With Data Services

**BigQuery**: External tables, import/export
**Dataflow**: Source/sink for pipelines
**Dataproc**: HDFS replacement, job I/O
**Composer**: DAG storage, data staging

### With ML/AI

**Vertex AI**: Training data, model storage
**AutoML**: Dataset storage
**AI Platform**: Job input/output

### With Migration Services

**Transfer Service**: Online data transfer
**Transfer Appliance**: Offline data transfer
**Database Migration Service**: Backup storage

## Security Considerations

### Encryption

**At Rest** (default):

- Google-managed encryption keys
- Automatic for all data
- No configuration needed

**Customer-Managed Keys (CMEK)**:

- Cloud KMS integration
- Customer controls key lifecycle
- Compliance requirements
- Audit key usage

**Customer-Supplied Keys (CSEK)**:

- Customer provides keys
- Google doesn't store keys
- More operational overhead
- Enhanced security

### In Transit

**HTTPS Only**:

- TLS encryption automatic
- No unencrypted option for API
- Best practice for all access

### Bucket Policies

**Organization Policies**:

- Enforce public access prevention
- Require uniform access
- Location restrictions
- Domain restrictions

**VPC Service Controls**:

- Perimeter-based access
- Prevent data exfiltration
- Additional security layer

## Compliance and Governance

### Data Residency

**Control via Location**:

- Choose specific region
- Dual-region for specific pairs
- Multi-region for broad geography
- Data stays in chosen location

### Retention Policies

**Bucket-Level Retention**:

- Minimum retention period
- Objects cannot be deleted before period
- Locked policies cannot be reduced
- Compliance use cases (regulatory requirements)

### Object Holds

**Types**:

- Event-based hold: Until event clears
- Temporary hold: Manual hold/release

**Use Cases**:

- Legal hold
- Investigation period
- Compliance requirements

## Exam Focus Areas

### Design Decisions

- When to use Cloud Storage vs alternatives
- Storage class selection criteria
- Location type selection (region/dual-region/multi-region)
- Access control method (IAM vs ACL)

### Cost Optimization

- Storage class economics
- Network egress patterns
- Operation cost implications
- Lifecycle management

### Performance

- Request rate limits
- Throughput optimization
- Latency considerations
- Parallel operations

### Security

- Encryption options
- Access control patterns
- Signed URLs use cases
- VPC Service Controls

### Integration

- BigQuery external tables
- Dataflow pipelines
- Cloud Functions triggers
- Content delivery (CDN)

### Compliance

- Data residency controls
- Retention policies
- Object holds
- Audit logging
