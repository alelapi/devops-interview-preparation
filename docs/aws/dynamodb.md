# Amazon DynamoDB

Amazon DynamoDB is a fully managed NoSQL database service provided by AWS, designed for applications requiring consistent, single-digit millisecond latency at any scale. DynamoDB is widely used for building scalable, high-performance, and serverless applications.

## Key Characteristics of DynamoDB

### **1. Fully Managed Service**

- AWS manages all administrative tasks like provisioning, scaling, backups, and updates.
- No server management required.

### **2. NoSQL Database**

- DynamoDB is a NoSQL database, meaning it stores data in a key-value and document-based format.
- Ideal for use cases like session management, gaming leaderboards, or IoT applications.

### **3. High Performance**

- Provides consistent, single-digit millisecond latency at any scale.
- Supports applications with high read and write throughput demands.

### **4. Scalability**

- Automatically scales up or down based on traffic patterns.
- Supports both provisioned capacity and on-demand capacity modes.

### **5. Global Tables**

- Allows you to replicate tables across multiple AWS regions for low-latency access and disaster recovery.

### **6. Flexible Schema**

- Does not enforce a fixed schema, making it adaptable for applications with evolving data models.
- Each item can have different attributes.

### **7. Strong and Eventual Consistency**

- Offers two read consistency models:
  - **Strongly Consistent Reads**: Ensure the most recent write is read.
  - **Eventually Consistent Reads**: May return stale data but with lower latency.

### **8. Partition Key and Sort Key**

- **Partition Key**: A unique identifier for each item. Determines the partition where the data is stored.
- **Sort Key**: Optional secondary key that allows for range-based queries within a partition.
- Together, the partition key and sort key form a composite primary key, enabling efficient organization and retrieval of data.

### **9. Secondary Indexes**

- DynamoDB supports indexes for flexible querying:
  - **Global Secondary Indexes (GSI)**: Can be queried across partitions and have a different partition and sort key than the base table.
  - **Local Secondary Indexes (LSI)**: Share the same partition key as the base table but allow for a different sort key.
- Secondary indexes enable efficient queries on attributes other than the primary key.

### **10. Data Types**

- Supports three primary data types:
  - **Scalar Types**: String, Number, Binary, Boolean, Null.
  - **Document Types**: List, Map.
  - **Set Types**: String Set, Number Set, Binary Set.

### **11. Integration with AWS Services**

- Seamlessly integrates with other AWS services like Lambda, S3, IAM, CloudWatch, and Kinesis.

### **12. Security**

- Encryption at rest using AWS Key Management Service (KMS).
- Fine-grained access control with IAM policies.
- Secure communication with SSL/TLS.

### **13. Stream Processing**

- DynamoDB Streams capture real-time data changes for downstream processing.
- Can integrate with AWS Lambda for event-driven architectures.

### **14. Backup and Restore**

- Offers point-in-time recovery (PITR) for accidental data deletion or corruption.
- Supports on-demand and continuous backups.

### **15. Pricing Model**

- Pay-as-you-go pricing based on:
  - Read and write capacity units (RCUs/WCUs).
  - Storage used.
  - Optional features like backups and streams.

### **16. Serverless**

- Fully serverless with no infrastructure management.
- Automatically scales based on usage.

---

DynamoDB is highly suitable for applications requiring high availability, scalability, and low-latency data access. With its rich set of features, it supports a wide range of use cases, from real-time analytics to mobile backends and gaming applications.
