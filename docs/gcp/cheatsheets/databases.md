# Database Essentials

## 🎯 Heavy Hitters (High Frequency)

### 1. **Database Selection (Most Important!)**
The exam LOVES asking "which database should you use?" - memorize this:

| Database | Use Case | Key Features |
|----------|----------|--------------|
| **Cloud SQL** | Traditional RDBMS, < 64 TB | PostgreSQL, MySQL, SQL Server; ACID; Vertical scaling |
| **AlloyDB** | High-performance PostgreSQL | 4x faster than Cloud SQL; 100% PostgreSQL compatible; AI/ML features |
| **Cloud Spanner** | Global RDBMS, > 1 TB | Horizontally scalable; 99.999% SLA; Global consistency |
| **Firestore** | Document database, mobile/web | Real-time sync; Offline support; NoSQL; Serverless |
| **Bigtable** | Wide-column NoSQL, IoT/analytics | Petabyte-scale; Low latency (<10ms); Time-series data |
| **BigQuery** | Data warehouse | Serverless; SQL analytics; Petabyte-scale; Separate storage/compute |

### 2. **Cloud SQL vs AlloyDB vs Cloud Spanner**

- **Cloud SQL**: Traditional workloads, < 64 TB, single region (or cross-region read replicas)

  - **Exam Clue**: "MySQL/PostgreSQL, moderate scale" → Cloud SQL
  
- **AlloyDB**: PostgreSQL only, need high performance, < 100 TB

  - **Exam Clue**: "Migrate PostgreSQL, need better performance" → AlloyDB
  - **4x faster** than Cloud SQL for transactional workloads
  - **100x faster** for analytical queries (columnar engine)
  
- **Cloud Spanner**: Need global distribution, > 1 TB, 99.999% SLA

  - **Exam Clue**: "Global application, strong consistency, horizontal scaling" → Spanner

### 3. **Firestore vs Bigtable**

- **Firestore**: Document-based, mobile/web apps, real-time sync

  - **Exam Clue**: "Mobile app, real-time updates, offline support" → Firestore
  - **Max 1 MB per document**
  - **1 write/sec per document** limit
  
- **Bigtable**: Wide-column, IoT/time-series, high throughput

  - **Exam Clue**: "IoT sensors, time-series, petabyte-scale, low latency" → Bigtable
  - **Not for**: < 1 TB data, transactional workloads
  - **Linear scaling**: Add nodes for more throughput

---

## 🗄️ Relational Databases (SQL)

### **Cloud SQL**

- **Engines**: PostgreSQL, MySQL, SQL Server
- **Max Size**: 64 TB
- **Availability**: 

  - **Regional**: Single zone or HA (automatic failover)
  - **Read Replicas**: Cross-region for read scaling
- **Backup**: Automated + on-demand, point-in-time recovery
- **Migration**: Database Migration Service (DMS)
- **Connections**: 

  - Public IP + Cloud SQL Auth Proxy
  - Private IP (VPC peering)
  - Private Service Connect (PSC)
- **Exam Tip**: Use Private IP or PSC for production workloads

**Key Limits:**

- **Max storage**: 64 TB (PostgreSQL/MySQL), 10 TB (SQL Server)
- **Max connections**: Depends on instance size (~4000 for large instances)

### **AlloyDB for PostgreSQL**
- **What**: Fully managed PostgreSQL-compatible database
- **Performance**: 
  - 4x faster transactional workloads vs Cloud SQL
  - 100x faster analytical queries (columnar engine)
- **Max Size**: 64 TB (can request more)
- **Availability**: 99.99% SLA, regional HA
- **Key Features**:
  - **Columnar engine**: Run analytics on transactional data (HTAP)
  - **AI/ML integration**: pgvector extension for vector embeddings
  - **Query insights**: Real-time performance monitoring
- **Migration**: Use DMS from Cloud SQL or on-prem PostgreSQL
- **Exam Clue**: "PostgreSQL + better performance + analytics" → AlloyDB

**When AlloyDB over Cloud SQL:**
- Need high performance (OLTP + OLAP)
- Running analytics on transactional data
- Using vector embeddings (pgvector)
- Large PostgreSQL databases

### **Cloud Spanner**
- **What**: Globally distributed, horizontally scalable RDBMS
- **CAP Theorem**: CP (Consistency + Partition tolerance)
- **Consistency**: Strong consistency (external consistency)
- **Scaling**: Horizontal (add nodes/processing units)
- **Max Size**: Unlimited (petabyte-scale)
- **Availability**: 
  - **Multi-region**: 99.999% SLA (5 nines!)
  - **Regional**: 99.99% SLA
- **Use Cases**:
  - Global applications (gaming, financial)
  - Need > 1 TB with strong consistency
  - Require horizontal scaling
- **Cost**: Most expensive SQL option
- **Exam Clue**: "Global consistency, 99.999% SLA, > 1 TB" → Spanner

**Key Concepts:**
- **Nodes**: 2 TB storage + 10,000 QPS per node (approximate)
- **Processing Units**: 1000 PU = 1 node (granular scaling)
- **Interleaved tables**: Parent-child rows co-located for performance
- **Primary key design**: Avoid hotspots (don't use sequential IDs)

**When Spanner over Cloud SQL:**
- Need > 1 TB AND horizontal scaling
- Global application requiring consistency
- Need 99.999% availability
- Can't tolerate replication lag

---

## 📦 NoSQL Databases

### **Firestore (Native Mode)**
- **Type**: Document database (NoSQL)
- **Use Cases**:
  - Mobile/web applications
  - Real-time synchronization
  - Offline support
  - Serverless backends
- **Structure**: Collections → Documents → Fields
- **Queries**: SQL-like queries on documents
- **Limits**:
  - **Document size**: 1 MB max
  - **Write rate**: 1 write/sec per document
  - **Query depth**: 100 levels
- **Pricing**: Pay per read/write/delete operation
- **Exam Clue**: "Mobile app, real-time, offline" → Firestore

**vs Datastore (Legacy):**
- Firestore is the successor (don't choose Datastore on exam)
- Firestore has better querying, real-time updates
- Can migrate Datastore → Firestore

### **Bigtable**
- **Type**: Wide-column NoSQL (like HBase/Cassandra)
- **Use Cases**:
  - Time-series data (IoT sensors, stock prices)
  - High-throughput analytics
  - Large-scale data (> 1 TB)
  - Low-latency reads/writes (< 10ms)
- **Not For**:
  - < 1 TB data (not cost-effective)
  - Transactional workloads (no ACID)
  - Complex queries (no joins, limited filtering)
- **Structure**: Tables → Row keys → Column families → Columns
- **Scaling**: Linear (add nodes for more throughput)
- **Replication**: Up to 8 clusters (sync or async)
- **Storage**: HDD (cheaper, higher latency) or SSD (faster, more expensive)
- **Exam Clue**: "IoT, time-series, petabyte-scale, low latency" → Bigtable

**Key Design Principles:**
- **Row key design**: Critical for performance (avoid hotspots)
- **Tall and narrow**: Many rows, few columns
- **Column families**: Group related columns
- **Time-series**: Reverse timestamp in row key for latest data first

**Performance:**
- **Throughput**: ~10,000 QPS per node (depends on workload)
- **Latency**: < 10ms for reads/writes (SSD)
- **Scaling**: Add nodes (linear scaling)

---

## 📊 Analytics & Data Warehouse

### **BigQuery**
- **Type**: Serverless data warehouse (analytics)
- **Use Cases**:
  - Data analytics and BI
  - Large-scale data analysis (petabyte-scale)
  - Machine learning (BQML)
  - Log analysis
- **Key Features**:
  - **Serverless**: No infrastructure management
  - **Separation**: Storage and compute decoupled
  - **SQL**: Standard SQL (ANSI SQL 2011)
  - **Partitioning**: Date/timestamp/integer partitioning
  - **Clustering**: Sort data within partitions
- **Not For**: 
  - Transactional workloads (OLTP)
  - Low-latency point queries
  - Frequent small updates
- **Pricing**: 
  - **On-demand**: $5/TB queried
  - **Flat-rate**: Reserved slots (predictable cost)
  - **Storage**: $0.02/GB/month (active), $0.01/GB/month (long-term)

**Performance Optimization:**
- **Partitioning**: Reduce data scanned
- **Clustering**: Improve query performance (up to 4 columns)
- **Materialized views**: Pre-compute aggregations
- **BI Engine**: In-memory analysis (sub-second queries)

**Exam Tips:**
- **Cost optimization**: Partition tables, limit query scope, use clustering
- **BigQuery ML**: Train ML models with SQL
- **BigQuery Omni**: Query data in AWS S3, Azure Blob Storage

---

## 🔄 Migration Strategies

### **Database Migration Service (DMS)**
- **What**: Managed service to migrate databases to Cloud SQL, AlloyDB
- **Supported Sources**: 
  - PostgreSQL → Cloud SQL PostgreSQL, AlloyDB
  - MySQL → Cloud SQL MySQL
  - SQL Server → Cloud SQL SQL Server
  - Oracle → Cloud SQL PostgreSQL (via ora2pg)
- **Migration Types**:
  - **One-time**: Single migration with downtime
  - **Continuous (CDC)**: Minimal downtime, replication
- **Process**: 
  1. Create migration job
  2. Continuous replication (CDC)
  3. Cutover when ready
- **Exam Clue**: "Migrate database with minimal downtime" → DMS

### **Migration Patterns**
```
MySQL/PostgreSQL on-prem → Cloud SQL
  ├── Small DB (< 100 GB): Export/import or DMS
  ├── Large DB (> 100 GB): DMS (continuous replication)
  └── Need better performance: → AlloyDB (via DMS)

SQL Server on-prem → Cloud SQL SQL Server
  └── Use DMS or native backup/restore

Oracle → Cloud SQL PostgreSQL
  └── ora2pg or partner tools (complex)

Need horizontal scaling → Cloud Spanner
  └── Schema redesign + application changes required

NoSQL migrations:
  ├── MongoDB → Firestore (document model)
  ├── Cassandra/HBase → Bigtable (wide-column)
  └── Redis → Memorystore (managed Redis)
```

---

## 🛡️ High Availability & Disaster Recovery

### **Cloud SQL HA**
- **Regional HA**: Primary + standby in different zones
  - **Failover**: < 60 seconds (automatic)
  - **Synchronous replication**: No data loss
  - **SLA**: 99.95% (HA configuration)
- **Read Replicas**: Cross-region for read scaling
  - **Async replication**: Potential lag
  - **Use for**: Read scaling, DR, geo-distribution
- **Backup**: 
  - **Automated**: Daily, 7-365 day retention
  - **On-demand**: Manual backups
  - **PITR**: Point-in-time recovery (up to 7 days)

### **AlloyDB HA**
- **Regional HA**: Primary + standby (automatic)
- **SLA**: 99.99%
- **Backup**: Automated + on-demand, continuous PITR
- **Read pools**: Scale reads independently

### **Cloud Spanner HA**
- **Multi-region**: 99.999% SLA (3+ regions)
- **Regional**: 99.99% SLA
- **Replication**: Synchronous (zero RPO)
- **Failover**: Automatic (no data loss)
- **Backup**: Scheduled backups, PITR (up to 7 days)

### **Firestore HA**
- **Multi-region**: Automatic replication
- **SLA**: 99.999% (multi-region)
- **No manual HA setup needed**

### **Bigtable HA**
- **Replication**: Up to 8 clusters
- **SLA**: 99.9% (single cluster), 99.99% (multi-cluster)
- **Failover**: Automatic (with app-profile routing)
- **Backup**: On-demand backups, scheduled backups

---

## 🔐 Database Security

### **Authentication & Authorization**
- **Cloud SQL**:
  - Database users (MySQL/PostgreSQL users)
  - Cloud SQL IAM authentication (PostgreSQL, MySQL 8.0+)
  - Cloud SQL Auth Proxy (secure connection)
- **AlloyDB**: IAM authentication, database users
- **Cloud Spanner**: IAM roles (Fine-grained IAM)
- **Firestore**: Firebase Auth or Cloud Identity
- **Bigtable**: IAM roles only
- **BigQuery**: IAM roles, column-level security, row-level security

### **Network Security**
- **Private IP**: VPC peering (Cloud SQL) or Private Service Connect
- **Private Service Connect**: Recommended for new deployments
- **Authorized networks**: IP allowlisting (public IP)
- **Cloud SQL Auth Proxy**: Encrypted connection, IAM auth

### **Encryption**
- **At rest**: Encrypted by default (Google-managed keys)
- **CMEK**: Customer-managed encryption keys (Cloud KMS)
- **In transit**: TLS/SSL (enforced or optional)

### **Audit Logging**
- **Admin Activity**: Who did what (enabled by default)
- **Data Access**: Who accessed what data (enable explicitly)
- **Query logs**: BigQuery, Cloud SQL (performance)

---

## 🎓 Exam Decision Trees

### **Which Database?**
```
Need SQL + < 64 TB + single region              → Cloud SQL
Need SQL + PostgreSQL + high performance        → AlloyDB
Need SQL + > 1 TB + global + horizontal scale   → Cloud Spanner
Need document database + real-time + mobile     → Firestore
Need time-series + IoT + > 1 TB + low latency   → Bigtable
Need analytics + petabyte-scale + serverless    → BigQuery
```

### **Cloud SQL vs AlloyDB vs Spanner**
```
PostgreSQL + need better performance            → AlloyDB
PostgreSQL + analytics on transactional data    → AlloyDB (columnar engine)
Need > 64 TB + horizontal scaling               → Cloud Spanner
Need 99.999% SLA + global consistency           → Cloud Spanner
Traditional MySQL/PostgreSQL + < 64 TB          → Cloud SQL
Budget-conscious + moderate performance         → Cloud SQL
```

### **Firestore vs Bigtable**
```
Document model + real-time sync                 → Firestore
Mobile/web app + offline support                → Firestore
< 1 TB data + serverless                        → Firestore
> 1 TB + time-series + IoT                      → Bigtable
Need lowest latency (< 10ms)                    → Bigtable
High-throughput writes (millions/sec)           → Bigtable
```

### **High Availability**
```
Need 99.999% SLA                                → Cloud Spanner (multi-region)
Need automatic failover + SQL                   → Cloud SQL HA or AlloyDB
Need cross-region read scaling                  → Cloud SQL read replicas
Need zero RPO (no data loss)                    → Cloud Spanner or Cloud SQL HA
Need multi-region replication + NoSQL           → Firestore or Bigtable replication
```

### **Migration**
```
MySQL/PostgreSQL + minimal downtime             → Database Migration Service
PostgreSQL + need better performance            → Migrate to AlloyDB via DMS
Oracle → PostgreSQL                             → ora2pg + Cloud SQL/AlloyDB
Need horizontal scaling after migration         → Cloud Spanner (redesign required)
SQL Server → GCP                                → Cloud SQL SQL Server (DMS or backup/restore)
```

---

## ⚡ Quick Reminders

### **Size Limits**
- **Cloud SQL**: 64 TB (PostgreSQL/MySQL), 10 TB (SQL Server)
- **AlloyDB**: 64 TB (can request more)
- **Cloud Spanner**: Unlimited (petabyte-scale)
- **Firestore**: 1 MB per document
- **Bigtable**: 10 GB per cell (but don't use cells that large)
- **BigQuery**: No storage limit

### **Performance**
- **Cloud SQL**: Vertical scaling (up to 624 GB RAM)
- **AlloyDB**: 4x faster than Cloud SQL (transactional), 100x faster (analytical)
- **Cloud Spanner**: Linear horizontal scaling
- **Firestore**: 1 write/sec per document limit
- **Bigtable**: ~10,000 QPS per node, < 10ms latency
- **BigQuery**: Serverless, auto-scales

### **Cost Considerations**
- **Cheapest**: Cloud SQL (for small databases)
- **Most expensive**: Cloud Spanner (but necessary for global scale)
- **BigQuery**: Pay per TB scanned (optimize with partitioning!)
- **Bigtable**: Not cost-effective for < 1 TB
- **Firestore**: Pay per operation (can get expensive with many writes)

### **Common Gotchas**
- **Cloud SQL**: Limited to 64 TB, can't scale horizontally
- **AlloyDB**: PostgreSQL ONLY (no MySQL, SQL Server)
- **Cloud Spanner**: Expensive, requires schema redesign (avoid hotspots)
- **Firestore**: Document size limit (1 MB), write rate limit (1/sec)
- **Bigtable**: No ACID transactions, no complex queries
- **BigQuery**: NOT for OLTP, costs scale with data scanned

---

## 🔍 Troubleshooting Quick Checks

**Cloud SQL connection issues:**
- ✅ Check authorized networks (if using public IP)
- ✅ Verify Cloud SQL Auth Proxy configured correctly
- ✅ Check private IP connectivity (VPC peering or PSC)
- ✅ Verify IAM permissions (cloudsql.client role)
- ✅ Check firewall rules allow connection

**Cloud SQL high replication lag:**
- ✅ Check read replica is sized appropriately
- ✅ Verify network connectivity between regions
- ✅ Check for long-running transactions on primary
- ✅ Consider upgrading instance size

**Bigtable hotspotting:**
- ✅ Review row key design (avoid sequential keys)
- ✅ Use reverse timestamp for time-series
- ✅ Distribute writes across key space
- ✅ Add nodes if underprovisioned

**BigQuery slow queries:**
- ✅ Check if table is partitioned
- ✅ Add clustering on commonly filtered columns
- ✅ Use WHERE clauses to limit data scanned
- ✅ Check query execution plan (EXPLAIN)
- ✅ Consider materialized views for repeated queries

**Firestore high costs:**
- ✅ Check for unnecessary reads (optimize client-side caching)
- ✅ Review document listeners (don't use on large collections)
- ✅ Batch writes when possible
- ✅ Use proper indexes (avoid composite index explosion)

---

## 📚 Pro Tips for Exam

1. **Read carefully**: Look for keywords like "global", "real-time", "time-series", "analytics"
2. **Scale requirements**: < 64 TB → Cloud SQL/AlloyDB; > 1 TB + horizontal → Spanner
3. **Performance**: "PostgreSQL + faster" → AlloyDB; "globally consistent" → Spanner
4. **Real-time**: "Mobile + real-time sync" → Firestore
5. **Analytics**: "Petabyte-scale analytics" → BigQuery; "OLTP + analytics" → AlloyDB
6. **IoT/Time-series**: "Low latency + high throughput + time-series" → Bigtable
7. **Migration**: "Minimal downtime" → Database Migration Service (DMS)
8. **HA/DR**: "99.999% SLA" → Spanner multi-region; "automatic failover" → Cloud SQL HA
9. **Cost**: Spanner is expensive, Cloud SQL is cheap, BigQuery costs scale with data scanned
10. **Don't overthink**: If scenario matches common pattern, choose obvious answer

---

## 🎯 Memorization Shortcuts

**Traditional SQL workload < 64 TB:**
Cloud SQL (MySQL/PostgreSQL/SQL Server)

**PostgreSQL + need better performance:**
AlloyDB (4x faster)

**Global app + strong consistency + > 1 TB:**
Cloud Spanner (99.999% SLA)

**Mobile app + real-time + offline:**
Firestore (document database)

**IoT sensors + time-series + low latency:**
Bigtable (< 10ms)

**Analytics + petabyte-scale:**
BigQuery (serverless)

**Migrate database with minimal downtime:**
Database Migration Service (DMS)

**PostgreSQL + analytics on same data:**
AlloyDB (columnar engine)

**Need 99.999% SLA:**
Cloud Spanner multi-region

**Vector embeddings + semantic search:**
AlloyDB (pgvector) or Cloud SQL PostgreSQL

---

## 🧪 Scenario-Based Examples

**Scenario 1**: "Global gaming company needs database for player profiles, must handle millions of concurrent users across regions with strong consistency."
- **Answer**: Cloud Spanner (global, horizontally scalable, strong consistency)

**Scenario 2**: "IoT company collects sensor data from millions of devices, needs to store time-series data and query recent readings with < 10ms latency."
- **Answer**: Bigtable (time-series, low latency, high throughput)

**Scenario 3**: "E-commerce company wants to migrate PostgreSQL database from on-prem to GCP, needs better performance for analytics queries on transactional data."
- **Answer**: AlloyDB (PostgreSQL compatible, columnar engine for analytics)

**Scenario 4**: "Startup building mobile app needs real-time chat, offline support, and wants serverless database."
- **Answer**: Firestore (real-time sync, offline, serverless)

**Scenario 5**: "Finance company needs to analyze petabytes of transaction logs for fraud detection, budget is a concern."
- **Answer**: BigQuery (petabyte-scale analytics, pay per query, use partitioning to optimize costs)

**Scenario 6**: "Company migrating MySQL database with 20 TB of data, needs minimal downtime during migration."
- **Answer**: Database Migration Service with continuous replication (CDC)

**Scenario 7**: "Healthcare app needs HIPAA-compliant PostgreSQL database with automatic failover and CMEK encryption."
- **Answer**: Cloud SQL PostgreSQL HA + CMEK + Private IP

**Scenario 8**: "Retail company needs to run both OLTP and OLAP queries on same dataset without ETL pipeline."
- **Answer**: AlloyDB (HTAP with columnar engine)