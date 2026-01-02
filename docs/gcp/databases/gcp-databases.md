# GCP Databases

## Core Concepts

GCP offers multiple database services, each optimized for specific use cases. Selecting the right database is critical for performance, cost, and scalability.

**Key Principle**: No one-size-fits-all; match database to workload characteristics.

## Database Decision Tree

```
Need SQL?
├─ Yes → Transactions needed?
│   ├─ Yes → Global scale?
│   │   ├─ Yes → Cloud Spanner
│   │   └─ No → Cloud SQL
│   └─ No (Analytics) → BigQuery
│
└─ No → Access pattern?
    ├─ Key-value, low latency → Memorystore (cache) or Firestore
    ├─ Wide-column, high throughput → Bigtable
    └─ Document, serverless → Firestore
```

## Database Comparison

| Database | Type | Use Case | Max Scale | Latency |
|----------|------|----------|-----------|---------|
| **Cloud SQL** | Relational (SQL) | OLTP, regional | 64 TB | Low |
| **Cloud Spanner** | Relational (SQL) | OLTP, global | Petabytes | Low |
| **BigQuery** | Data warehouse | Analytics (OLAP) | Exabytes | Seconds |
| **Firestore** | Document NoSQL | Mobile, web apps | Terabytes | Low |
| **Bigtable** | Wide-column NoSQL | Time-series, IoT | Petabytes | <10ms |
| **Memorystore** | In-memory cache | Cache layer | 300 GB | <1ms |

## Cloud Spanner

### Overview

Globally distributed, horizontally scalable, strongly consistent relational database with SQL support.

**Key features**: ACID transactions, global replication, automatic sharding

### When to Use

✅ **Appropriate for**:

- Need global scale + SQL + strong consistency
- Multi-region application with low latency globally
- Outgrew Cloud SQL (need horizontal write scaling)
- Financial systems (consistency critical)
- Inventory systems (global consistency)

❌ **Don't use when**:

- Regional only (Cloud SQL cheaper)
- Analytics workload (BigQuery better)
- Don't need ACID transactions (Firestore or Bigtable cheaper)

### Architecture

**Regional**: Data in 3 zones within region

**Multi-region**: Data across multiple regions (dual-region or multi-region)

**Replication**: Synchronous across zones, async across regions

### Scalability

**Horizontal**: Add nodes for more throughput

**Writes**: Scale linearly with nodes

**Reads**: Can add read replicas

**Pattern**: Start with 3 nodes (minimum for production), scale based on CPU

### HA and DR

**HA**: 99.99% (regional), 99.999% (multi-region)

**DR**: Multi-region deployment, automatic failover

**RPO**: Zero (synchronous within region)

**RTO**: Automatic failover (seconds)

### Cost

**Expensive**: $0.90/node/hour (regional), $3/node/hour (multi-region)

**Trade-off**: Global scale + strong consistency = high cost

**Use when**: Features justify cost (global scale needed)

## Firestore

### Overview

Serverless NoSQL document database for mobile and web applications. Real-time synchronization and offline support.

**Key features**: Real-time listeners, offline support, automatic scaling

### When to Use

✅ **Appropriate for**:

- Mobile/web applications
- Real-time data sync across clients
- Serverless backend
- Document-oriented data (JSON-like)
- Need offline support

❌ **Don't use when**:

- Complex queries/analytics (BigQuery)
- High-throughput writes (Bigtable)
- Strong ACID transactions (Spanner)
- Relational data (Cloud SQL)

### Modes

**Native mode** (recommended):

- Real-time, offline, mobile SDKs
- Strong consistency
- Complex queries
- Auto-scaling

**Datastore mode** (legacy):

- Server SDKs only
- Eventual consistency
- Limited query capabilities
- Use Native for new apps

### Scalability

**Automatic**: Scales automatically based on load

**Limits**:

- 1 write/sec per document
- 10,000 writes/sec per database (soft limit)

**Pattern**: Distribute writes across documents (avoid hotspots)

### HA and DR

**Multi-region**: Automatically replicated

**SLA**: 99.999% (multi-region)

**Backups**: Export to Cloud Storage (manual or scheduled)

**PITR**: Not native (use exports)

### Cost

**Pay per**: Reads, writes, deletes, storage

**Free tier**: 1 GB storage, 50K reads, 20K writes/day

**Cheaper than**: Cloud SQL for small, variable workloads

## Bigtable

### Overview

Wide-column NoSQL database for high-throughput, low-latency workloads at massive scale. HBase-compatible.

**Key features**: Petabyte scale, <10ms latency, high throughput

### When to Use

✅ **Appropriate for**:

- Time-series data (IoT, metrics, logs)
- High throughput (>100K QPS)
- Large analytical workloads (MapReduce, Dataflow)
- IoT data ingestion
- Financial data (tick data)
- Ad tech (real-time bidding)

❌ **Don't use when**:

- Small dataset (<1 TB) or low throughput
- Need transactions (Spanner)
- Need SQL (Cloud SQL, BigQuery)
- Document model better (Firestore)

### Architecture

**Row key**: Single key, determines data distribution

**Column families**: Group related columns

**Sparse**: Not all rows have all columns

**Important**: Row key design critical for performance (avoid hotspots)

### Scalability

**Horizontal**: Add nodes for more throughput

**Linear scaling**: Throughput increases with nodes

**Minimum**: 3 nodes production (1 node dev)

**Pattern**: ~10K QPS per node

### HA and DR

**Replication**: Optional (cross-region or multi-cluster)

**Backups**: Scheduled or on-demand

**Consistency**: Eventual consistency (replication)

**Use case**: Replication for HA and low-latency global access

### Performance

**Row key design**: Critical for performance

**Good**: Distributed (field#timestamp)

**Bad**: Sequential (timestamp#field) → hotspots

**Latency**: Single-digit milliseconds (with good row key)

### Cost

**Expensive**: $0.65/node/hour + storage

**Storage**: SSD ($0.17/GB) or HDD ($0.026/GB)

**Minimum**: 3 nodes production = ~$1,400/month minimum

**Use when**: Volume justifies cost (>1 TB, high throughput)

## Memorystore

### Overview

Fully managed in-memory data store (Redis or Memcached). Ultra-low latency caching.

**Key features**: <1ms latency, Redis/Memcached compatible

### When to Use

✅ **Appropriate for**:

- Cache layer (reduce database load)
- Session storage
- Real-time analytics
- Leaderboards, counters
- Pub/Sub messaging (Redis)

❌ **Don't use when**:

- Persistent storage needed (data loss on restart)
- Primary database (use actual database)
- Large datasets (max 300 GB)

### Redis vs Memcached

| Feature | Redis | Memcached |
|---------|-------|-----------|
| **Data structures** | Rich (lists, sets, sorted sets) | Simple (key-value) |
| **Persistence** | Optional snapshots | None |
| **Replication** | Yes (HA) | No (single node) |
| **Pub/Sub** | Yes | No |
| **Use case** | Complex caching, sessions | Simple cache |

**Recommendation**: Redis for most use cases (more features)

### Tiers

**Basic** (Redis only):

- Single node
- No replication
- Cheaper
- Dev/test

**Standard** (Redis only):

- HA with replication
- Automatic failover
- Production

### HA and DR

**Standard tier**: Automatic failover (zone redundant)

**Backups**: Export RDB snapshots (Redis)

**Persistence**: Optional (reduces performance)

**Recommendation**: Use as cache (can rebuild), not primary store

### Scalability

**Vertical**: Up to 300 GB per instance

**Horizontal**: Application-level sharding

**Read replicas**: Standard tier supports read replicas

### Cost

**Pricing**: Per GB RAM per hour

**Basic**: ~$0.036/GB/hour (~$26/GB/month)

**Standard**: ~$0.054/GB/hour (~$39/GB/month)

**Expensive**: For memory size, use for cache only (high value)

## Selection Criteria

### By Workload Type

**OLTP** (Transactional):

- Regional: Cloud SQL
- Global: Cloud Spanner
- Serverless: Firestore

**OLAP** (Analytics):

- BigQuery (always)

**Key-Value**:

- Cache: Memorystore
- Persistent: Firestore

**Time-Series**:

- Bigtable (high-throughput)
- BigQuery (analytics)

**Mobile/Web**:

- Firestore (first choice)
- Cloud SQL (if relational needed)

### By Scale Requirements

**<100 GB**: Cloud SQL, Firestore

**100 GB - 10 TB**: Cloud SQL, Firestore, BigQuery

**>10 TB**: BigQuery, Bigtable, Spanner

**Petabyte**: BigQuery, Bigtable

### By Consistency Requirements

**Strong consistency**:

- Cloud SQL
- Cloud Spanner
- Firestore (Native mode)

**Eventual consistency acceptable**:

- Bigtable
- Memorystore
- Firestore (Datastore mode)

### By Latency Requirements

**<1ms**: Memorystore

**<10ms**: Bigtable, Firestore, Cloud SQL (local)

**<100ms**: Cloud SQL, Spanner

**Seconds**: BigQuery

### By Cost Sensitivity

**Cheapest to most expensive**:

1. Cloud SQL (small workloads)
2. Firestore (serverless, pay per use)
3. BigQuery (analytics, pay per query)
4. Memorystore (cache)
5. Bigtable (high minimum)
6. Cloud Spanner (most expensive)

## Multi-Database Patterns

### Lambda Architecture

```
Real-time: Bigtable (hot data, recent)
Batch: BigQuery (cold data, historical)
Serving: Cached in Memorystore
```

### CQRS (Command Query Responsibility Segregation)

```
Writes: Cloud SQL or Spanner
Reads: Firestore or Memorystore (materialized views)
Analytics: BigQuery (separate data warehouse)
```

### Cache-Aside Pattern

```
Application → Check Memorystore
  ├─ Hit → Return cached data
  └─ Miss → Query Cloud SQL → Cache result → Return
```

### Polyglot Persistence

**Pattern**: Use best database for each use case

**Example**:

- User profiles: Firestore (flexible schema)
- Transactions: Cloud SQL (ACID)
- Analytics: BigQuery (data warehouse)
- Session data: Memorystore (cache)
- Time-series: Bigtable (IoT metrics)

## Migration Paths

### On-Premises → Cloud SQL

**Tools**: Database Migration Service

**Use when**: Lift-and-shift, minimal changes

### MySQL/PostgreSQL → Cloud Spanner

**Requires**: Schema changes (avoid hotspots)

**Use when**: Need global scale

### MongoDB → Firestore

**Use when**: Document model, need serverless

### HBase → Bigtable

**Tools**: HBase replication

**Use when**: High-throughput time-series

### Traditional DW → BigQuery

**Tools**: Dataflow, Transfer Service

**Use when**: Analytics, data warehouse consolidation

## Exam Focus

### Decision Criteria

- Workload type (OLTP vs OLAP)
- Scale requirements (GB to PB)
- Consistency needs (strong vs eventual)
- Latency requirements
- Cost sensitivity

### Database Characteristics

**Cloud SQL**: Regional SQL, OLTP, <64 TB
**Spanner**: Global SQL, strong consistency, expensive
**BigQuery**: Analytics, petabyte scale, serverless
**Firestore**: Document, serverless, mobile/web
**Bigtable**: Wide-column, high-throughput, time-series
**Memorystore**: Cache, <1ms latency, not persistent

### Use Cases

- Global application → Spanner
- Analytics → BigQuery
- Mobile app → Firestore
- IoT time-series → Bigtable
- Cache layer → Memorystore
- Regional OLTP → Cloud SQL

### HA and DR

- Cloud SQL: Regional HA, cross-region replicas
- Spanner: Multi-region, automatic failover
- BigQuery: Multi-region, time travel
- Firestore: Multi-region, exports
- Bigtable: Replication, backups
- Memorystore: Standard tier HA

### Cost

- Most expensive: Spanner, Bigtable (high minimum)
- Cheapest: Cloud SQL (small), Firestore (serverless)
- BigQuery: Pay per query (can be cheap or expensive)
- Memorystore: Expensive per GB (use for cache)

### Common Patterns

- Cache-aside (Memorystore + database)
- CQRS (write/read separation)
- Polyglot persistence (best tool per use case)
- Lambda architecture (real-time + batch)
