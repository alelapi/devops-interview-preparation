# BigQuery

## Core Concepts

BigQuery is a serverless, fully managed data warehouse for analytics at petabyte scale. SQL-based querying with automatic scaling and high performance.

**Key Principle**: Designed for analytics (OLAP), not transactions (OLTP); pay for storage and queries, not infrastructure.

## When to Use BigQuery

### ✅ Use When

- Analytics and reporting (data warehouse)
- Large-scale data analysis (terabytes to petabytes)
- Ad-hoc SQL queries
- Business intelligence and dashboards
- Log analysis at scale
- Machine learning on structured data

### ❌ Don't Use When

- Transactional workload (OLTP) → Cloud SQL, Spanner
- Low-latency reads/writes → Firestore, Bigtable
- Blob storage → Cloud Storage
- Small datasets (<100 GB) → Cloud SQL may be cheaper
- Real-time streaming with millisecond latency → Bigtable

## Architecture

### Serverless

**No infrastructure management**:

- No servers to provision
- Automatic scaling
- No capacity planning
- Pay per query

**Separation of storage and compute**:

- Storage: Pay for data stored
- Compute: Pay for queries executed
- Scale independently

### Data Organization

**Structure**:

```
Project
└── Dataset (location, access control)
    └── Tables (schema, partitioning, clustering)
        └── Rows and columns
```

**Dataset**: Logical container, regional or multi-regional

**Table types**:

- Native tables (managed by BigQuery)
- External tables (data in Cloud Storage/Drive/Bigtable)
- Views (virtual tables from queries)
- Materialized views (precomputed results)

## Performance Optimization

### Partitioning

**Purpose**: Divide large tables into smaller segments

**Types**:

- Time-based (daily, hourly, monthly, yearly)
- Integer range
- Ingestion time

**Benefits**:

- Faster queries (scan less data)
- Lower costs (process less data)
- Easier data management (delete old partitions)

**Example**: Partition logs table by date, query only last 7 days

### Clustering

**Purpose**: Organize data within partitions by column values

**Benefits**:

- Faster queries on clustered columns
- Lower costs (better pruning)
- Automatic (BigQuery manages)

**Use with**: Up to 4 columns commonly queried together

**Pattern**: Partition by date, cluster by user_id and region

### Query Optimization

**Best practices**:

- Select only needed columns (avoid `SELECT *`)
- Filter early (WHERE clause reduces data scanned)
- Use partitioned tables
- Avoid self-joins (use window functions)
- Use approximate aggregation (HyperLogLog for COUNT DISTINCT)

**Explanation tabs**: Shows query execution plan, bytes processed

## Storage Options

### Active Storage

**Default**: Standard storage

**Price**: ~$0.020/GB/month (first 10 GB free)

**Use for**: Frequently queried data

### Long-Term Storage

**Automatic**: Tables not edited for 90 days

**Price**: ~$0.010/GB/month (50% discount)

**Benefit**: Cost savings for archival data

**Important**: Query pricing same regardless of storage type

## Data Loading

### Batch Loading

**Methods**:

- Load jobs (Cloud Storage, local files)
- SQL INSERT statements
- Dataflow pipelines
- Third-party ETL tools

**Free**: Batch loading is free

**Use for**: Large bulk loads, scheduled ETL

### Streaming Inserts

**Purpose**: Real-time data ingestion

**Latency**: Available for query immediately

**Cost**: $0.010 per 200 MB (higher than batch)

**Limits**: 100,000 rows/sec per table

**Use for**: Real-time dashboards, log streaming

**Alternative**: Storage Write API (cheaper, better performance)

## Data Export

**Destinations**:

- Cloud Storage (CSV, JSON, Avro, Parquet)
- Other BigQuery tables
- Google Sheets (small datasets)

**Limits**: 1 GB per file (use wildcards for larger)

**Cost**: Export is free, storage charges apply

## Access Control

### Dataset-Level

**IAM Roles**:

- `bigquery.dataViewer`: Read data and metadata
- `bigquery.dataEditor`: Create, update, delete tables
- `bigquery.dataOwner`: Full control including permissions

**Scope**: All tables in dataset

### Table-Level

**Authorized views**: Allow access to specific views, not underlying tables

**Use case**: Row-level security, column masking

### Row-Level Security

**Purpose**: Filter rows based on user identity

**Implementation**: Create row-level policies

**Example**: Users only see their own organization's data

## Data Loss Prevention (DLP) API

### Purpose

Discover, classify, and protect sensitive data in BigQuery (PII, PHI, financial data).

### Capabilities

**Discovery**:

- Scan datasets for sensitive data
- Identify PII (SSN, credit cards, emails, phone numbers)
- Custom info types (regex patterns)

**Classification**:

- Likelihood scores (very likely, likely, possible, unlikely)
- Info types (PERSON_NAME, CREDIT_CARD_NUMBER, etc.)
- Statistical analysis

**De-identification**:

- Masking (replace with *****)
- Tokenization (consistent hash)
- Date shifting (preserve relative dates)
- Generalization (age → age range)

### Common Patterns

**PII Discovery**:

```
DLP API scans BigQuery table → Identifies sensitive columns → Generate report
```

**De-identification Pipeline**:

```
Source table → DLP API (de-identify) → Destination table (masked data)
```

**Continuous Monitoring**:

```
Cloud Functions + DLP API → Scan new data → Alert if PII found
```

### Use Cases

- GDPR compliance (identify PII)
- HIPAA compliance (protect PHI)
- PCI-DSS (find credit card data)
- Data lake governance
- Safe analytics on sensitive data

### Integration

**Automatic**: Scan tables directly via API

**Patterns**:

- Scheduled scans (Cloud Scheduler + Cloud Functions)
- Pre-export scanning (ensure no PII exported)
- CI/CD validation (check for accidental PII)

### Cost

**DLP API pricing**: Per GB scanned

**Optimization**:

- Scan samples, not entire tables
- Use column-level scans
- Cache results, don't rescan unchanged data

## BigQuery ML

**Purpose**: Build and deploy ML models using SQL

**Supported models**:

- Linear regression
- Logistic regression
- K-means clustering
- Time series forecasting
- Deep neural networks
- Import TensorFlow models

**Benefits**: No data export, SQL-based, integrated

**Use case**: Predictions on BigQuery data without separate ML platform

## Federated Queries

### External Data Sources

**Supported**:

- Cloud Storage (CSV, JSON, Avro, Parquet, ORC)
- Cloud Bigtable
- Cloud SQL (read-only)
- Google Sheets

**Use cases**:

- Query data without loading
- Join BigQuery with external data
- One-time analysis
- Data lake pattern (data in Storage, query in BigQuery)

**Limitations**:

- Slower than native tables
- No caching
- Subject to external source performance

## BI Engine

**Purpose**: In-memory analysis for fast dashboards

**Benefits**:

- Sub-second query response
- Interactive dashboards
- No query charges for cached data
- Automatic caching

**Capacity**: Pay for memory reservation (per GB/hour)

**Use for**: Frequently accessed dashboards, real-time analytics

## High Availability

### Automatic

**Built-in**: Multi-zone replication within region

**No configuration needed**: Highly available by default

**SLA**: 99.99% monthly uptime

### Multi-Region

**Datasets**: US, EU (multi-region) or specific regions

**Benefits**:

- Geographic redundancy
- Lower latency for global users
- Higher availability

**Limitation**: Cannot change location after creation

## Disaster Recovery

### Backup Strategy

**Table snapshots**: Point-in-time copies

**Table clones**: Lightweight copies (copy-on-write)

**Cross-region copy**: Export and re-import in another region

**Time travel**: Query data from up to 7 days ago

**Pattern**: Periodic exports to Cloud Storage in multi-region

### Time Travel

**Purpose**: Query historical data without explicit backups

**Duration**: 7 days default (configurable 2-7 days)

**Use cases**:

- Recover accidentally deleted/modified data
- Compare data over time
- Audit changes

**Query syntax**: `FOR SYSTEM_TIME AS OF timestamp`

### Deleted Table Recovery

**Duration**: Restore deleted tables within 7 days

**Limitation**: Dataset deletion is permanent (cannot recover)

## Scalability

### Automatic Scaling

**Query processing**: Thousands of concurrent queries

**Storage**: Exabyte scale

**No tuning**: Automatic resource allocation

**Slots**: Units of computational capacity (automatic or reserved)

### Reservation and Slots

**On-demand** (default): Pay per query, automatic capacity

**Flat-rate pricing**: Reserve slots, predictable cost

**Flex slots**: Commit for 60 seconds minimum

**Use reserved when**: Predictable workload, >$10K/month queries

## Cost Optimization

### Query Costs

**Pricing**: $5 per TB processed (first 1 TB/month free)

**Optimization**:

- Partition and cluster tables
- Select specific columns (not *)
- Use table preview (free)
- Cache query results (24 hours free)
- Approximate aggregation functions
- Streaming buffer charges

### Storage Costs

**Active**: $0.020/GB/month
**Long-term** (90+ days): $0.010/GB/month (automatic)

**Optimization**:

- Delete unused tables
- Use table expiration
- Export to Cloud Storage (cheaper long-term)
- Partition pruning

### Monitoring

**Metrics**:

- Bytes scanned per query
- Slot utilization (reserved pricing)
- Storage trends

**Set budgets**: Alert when costs exceed threshold

## Security

### Encryption

**At rest**: Automatic (Google-managed or CMEK)

**In transit**: TLS by default

**Column-level encryption**: Application-level before load

### Access Logging

**Audit logs**:

- Admin Activity (dataset/table changes)
- Data Access (must enable, query logs)

**Use for**: Compliance, security investigations

### VPC Service Controls

**Purpose**: Prevent data exfiltration

**Pattern**: BigQuery inside perimeter, cannot export outside

## Best Practices

### Schema Design

- Denormalize for performance (avoid JOINs)
- Use nested/repeated fields (STRUCT, ARRAY)
- Partition by date/timestamp
- Cluster by high-cardinality columns

### Query Patterns

- Filter early in WHERE clause
- Avoid SELECT * in production
- Use materialized views for common aggregations
- Sample large datasets for exploration (TABLESAMPLE)

### Data Lifecycle

- Set table expiration for temporary data
- Archive to Cloud Storage for long-term
- Use partitioned tables for time-series
- Enable time travel for 7 days

### Cost Management

- Estimate before running (dry run)
- Use query validator
- Set maximum bytes billed
- Monitor spending with billing exports

## Exam Focus

### Core Concepts

- Serverless data warehouse (no infrastructure)
- Separation of storage and compute
- Analytics (OLAP), not transactions (OLTP)
- Pay per query (on-demand) or slots (reserved)

### Use Cases

- Data warehouse and BI
- Log analysis at scale
- Ad-hoc analytics
- BigQuery ML for predictions
- NOT for: OLTP, low-latency K/V, small databases

### Performance

- Partitioning (reduce data scanned)
- Clustering (faster queries)
- Materialized views (precompute)
- Query optimization patterns

### DLP API

- Discover sensitive data (PII, PHI)
- De-identification techniques
- Compliance (GDPR, HIPAA, PCI-DSS)
- Automated scanning patterns

### Cost Optimization

- Partition and cluster
- Avoid SELECT *
- Use caching
- Long-term storage (automatic after 90 days)
- Reserved slots for predictable workload

### Integration

- Federated queries (Cloud Storage, Bigtable, Cloud SQL)
- Streaming inserts vs batch loading
- Export to Cloud Storage
- BigQuery ML

### Disaster Recovery

- Time travel (7 days)
- Table snapshots
- Cross-region exports
- Deleted table recovery (7 days)
