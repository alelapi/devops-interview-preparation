# Object Versioning

## Core Concepts

Object versioning maintains multiple variants of an object in the same bucket, protecting against accidental deletion or overwrites. Understanding when versioning provides value versus when backups or other strategies are more appropriate is critical for cost-effective data protection.

**Key Principle**: Versioning is for protecting against accidental changes and deletions, not for backup/archival (use separate buckets or lifecycle policies instead).

## How Versioning Works

### Version Generation

**Without Versioning**:

```
Upload object.txt → Replaces any existing object.txt
Delete object.txt → Permanently deleted
```

**With Versioning**:

```
Upload object.txt v1 → Generation #1234567890 (current)
Upload object.txt v2 → Generation #1234567891 (current), v1 becomes non-current
Delete object.txt   → Generation #1234567892 (delete marker), v2 becomes non-current
```

**Architecture Implications**:

- Objects never truly deleted (delete marker created)
- All versions stored and charged
- Must explicitly delete versions to free space
- Cannot turn off versioning (only suspend)

### Object Generations

**Generation Number**:

- Unique identifier for each version
- Microsecond timestamp
- Automatically assigned
- Immutable

**Current vs Non-Current**:

- **Current**: Latest version, returned by default
- **Non-Current**: Previous versions, must specify generation
- **Delete Marker**: Special version indicating deletion

### Versioning States

**Enabled**:

- All new objects versioned
- Existing objects gain versions
- Cannot disable (only suspend)

**Suspended**:

- New objects get null generation
- Existing versions retained
- New uploads overwrite null generation
- Can re-enable later

**Never Enabled**:

- Default state
- Objects have generation (for consistency) but not versioned
- Cannot recover from deletion/overwrite

**Important**: Once enabled, cannot return to "never enabled" state

## When to Use Versioning

### ✅ Appropriate Uses

**Protection Against Accidental Deletion**:

- User errors
- Application bugs
- Operator mistakes
- Malicious actions

**Protection Against Overwrites**:

- Accidental file replacements
- Concurrent write conflicts
- Application errors
- Data corruption

**Compliance Requirements**:

- Audit trails
- Change tracking
- Document retention
- Regulatory requirements

**Collaboration Scenarios**:

- Multiple users editing
- Concurrent access
- Rollback capability
- Change history

**Critical Data Protection**:

- Configuration files
- Important documents
- Application state
- Database backups

### ❌ Inappropriate Uses

**Long-Term Backup**:

- Use separate backup bucket instead
- Lifecycle policies to Archive class
- Cross-region replication
- Not cost-effective for long-term retention

**Version Control for Code**:

- Use Git, SVN, or other VCS
- Cloud Storage not designed for code versioning
- Better tools available
- Wrong abstraction

**Frequent Changes**:

- Rapidly changing files
- Many small updates
- High-frequency logging
- Append-only data

**Why**: Cost accumulates quickly, version explosion

**Temporary Files**:

- Scratch space
- Intermediate processing results
- Cache data
- Short-lived data

**Why**: Versions of temporary data wasteful

## Versioning vs Alternatives

### Versioning vs Separate Backup Bucket

| Feature | Versioning | Separate Backup Bucket |
|---------|-----------|------------------------|
| **Same Bucket** | Yes | No (different bucket) |
| **Cost** | All versions charged at current class | Can use cheaper classes |
| **Lifecycle** | Can age versions differently | Full lifecycle control |
| **Protection** | Accidental deletion/overwrite | Logical/physical separation |
| **Complexity** | Low | Medium |
| **Best For** | Operational recovery | Disaster recovery |

**Decision**: Use versioning for operational protection, separate buckets for DR/backup

### Versioning vs Snapshots (Compute Engine)

| Feature | Object Versioning | Disk Snapshots |
|---------|------------------|----------------|
| **Granularity** | Per object | Per disk |
| **Incremental** | No (full versions) | Yes |
| **Use Case** | Object storage | Block storage |
| **Cost** | Full object size per version | Incremental only |

**Decision**: Different services, different use cases

### Versioning vs Lifecycle Policies

**Complementary, Not Alternative**:

- Versioning: Creates versions
- Lifecycle: Manages version aging
- Use together for cost optimization
- Lifecycle can delete old versions

**Pattern**: Enable versioning + lifecycle policy for version cleanup

```yaml
# Conceptual
Versioning: Enabled
Lifecycle: Delete versions older than 30 days
```

## Cost Implications

### Storage Costs

**Every Version Charged**:

- Each version is full object size
- All versions in same storage class (unless lifecycle transitions)
- Costs accumulate with versions

**Example**: 1 GB file, 10 versions

```
Without versioning: 1 GB × $0.020 = $0.020/month
With versioning: 10 GB × $0.020 = $0.200/month
Cost multiplier: 10x
```

**Architecture Impact**: Monitor version count to control costs

### Early Deletion Fees and Versions

**Version Age Calculation**:

- Each version has its own creation time
- Minimum duration per version
- Lifecycle transitions affect each version

**Example**: Nearline object with 5 versions

```
Current version: Created today (day 0)
Version 2: Created 10 days ago
Version 3: Created 20 days ago
Version 4: Created 25 days ago
Version 5: Created 35 days ago

Lifecycle: Delete versions older than 30 days

Version 5: Deleted at day 35 (no early deletion fee)
Versions 1-4: Retained
```

### Cost Optimization Strategies

**Version Lifecycle Policies**:

```yaml
# Conceptual
Rule 1: numberOfNewerVersions 3 → Delete  # Keep only 3 recent
Rule 2: daysSinceNoncurrentTime 30 → Delete  # Delete after 30 days non-current
```

**Benefits**:

- Control version accumulation
- Automatic cleanup
- Predictable costs
- Balance protection and cost

**Transition Non-Current Versions**:

```yaml
# Conceptual
Current objects: Keep in Standard
Non-current versions: Move to Nearline after 7 days
```

**Benefits**:

- Reduce storage costs for old versions
- Maintain current object performance
- Cost-effective version retention

## Versioning Patterns

### Pattern 1: Short-Term Version Retention

**Use Case**: Protect against recent mistakes, not long-term history

**Configuration**:

- Versioning: Enabled
- Lifecycle: Delete non-current versions after 30 days
- Keep only 5 most recent versions

**Appropriate For**:

- Application data
- Configuration files
- Working documents
- Development assets

**Cost Profile**: Moderate (limited versions)

### Pattern 2: Compliance Version Retention

**Use Case**: Regulatory requirements for change history

**Configuration**:

- Versioning: Enabled
- Lifecycle: Transition non-current to Archive after 30 days
- Retention lock: 7 years
- Keep all versions

**Appropriate For**:

- Financial records
- Medical documents
- Legal documents
- Audit trails

**Cost Profile**: High (all versions, long retention)

### Pattern 3: Recent Version Accessibility

**Use Case**: Frequent rollback needs, recent changes only

**Configuration**:

- Versioning: Enabled
- Lifecycle: Keep 10 recent versions
- Non-current to Nearline after 7 days
- Delete after 90 days non-current

**Appropriate For**:

- Website content
- Application assets
- Content management
- Collaboration scenarios

**Cost Profile**: Low-Moderate (limited versions, lower storage class)

### Pattern 4: Minimal Versioning

**Use Case**: Protection only, minimize costs

**Configuration**:

- Versioning: Enabled
- Lifecycle: Keep only 1-2 previous versions
- Delete non-current after 7 days

**Appropriate For**:

- Large files
- Binary assets
- Media files
- Cost-sensitive workloads

**Cost Profile**: Low (minimal versions)

## Versioning and Object Lifecycle

### Lifecycle Rules for Versions

**Conditions for Versions**:

- `numberOfNewerVersions`: Limit version count
- `daysSinceNoncurrentTime`: Age since becoming non-current
- `isLive: false`: Target only non-current versions

**Common Patterns**:

```yaml
# Conceptual
# Keep 5 recent versions
Rule 1: numberOfNewerVersions 5 + isLive false → Delete

# Age out old versions
Rule 2: daysSinceNoncurrentTime 90 + isLive false → Delete

# Cheap storage for versions
Rule 3: daysSinceNoncurrentTime 7 + isLive false + Nearline → Coldline
```

### Version-Specific Transitions

**Pattern**: Different lifecycle for current vs non-current

```yaml
# Conceptual
Current object lifecycle:
  age 30 days + Standard → Nearline

Non-current version lifecycle:
  daysSinceNoncurrentTime 7 → Nearline
  daysSinceNoncurrentTime 30 → Delete
```

**Benefits**:

- Aggressive version cleanup
- Protect current object longer
- Cost optimization
- Flexibility

## Deletion Behavior with Versioning

### Soft Delete (Delete Marker)

**Default Behavior**:

```
Delete object → Create delete marker (special version)
List bucket → Object not shown (appears deleted)
Get object → 404 Not Found
```

**Recovery**:

```
Delete the delete marker → Object reappears (previous version becomes current)
```

**Use Case**: Accidental deletion recovery

### Permanent Delete

**Deleting Specific Version**:

```
Delete object with generation number → Version permanently deleted
Delete current version → Previous version becomes current
```

**Cannot Be Recovered**: Permanent deletion

### Delete Marker Cleanup

**Problem**: Delete markers accumulate

**Solution**: Lifecycle policy

```yaml
# Conceptual
Rule: Delete marker with no other versions → Delete marker
```

**Benefit**: Clean up orphaned delete markers

## Recovery Scenarios

### Scenario 1: Accidental File Overwrite

**Problem**: Uploaded wrong version

**Recovery**:

1. List object versions
2. Identify correct version (by timestamp)
3. Copy correct version to new object or restore as current

**Timeline**: Immediate (minutes)

### Scenario 2: Accidental Deletion

**Problem**: Deleted important file

**Recovery**:

1. Identify delete marker
2. Delete the delete marker
3. Previous version becomes current

**Timeline**: Immediate (seconds)

### Scenario 3: Ransomware/Malware

**Problem**: Files encrypted/corrupted

**Recovery**:

1. Identify last good version (before infection)
2. Restore all objects to that generation
3. Delete infected versions

**Timeline**: Hours (depending on scale)

**Architecture Pattern**: Versioning + retention lock prevents malicious deletion of versions

## Retention Lock and Versioning

### Bucket-Level Retention

**How It Works**:

- Minimum retention period
- Objects/versions cannot be deleted before period
- Applies to all objects in bucket
- Lock can be permanent or temporary

**With Versioning**:

- Applies to each version independently
- Version age starts when version created (not object)
- Protects against accidental and malicious deletion

**Use Case**: Compliance (WORM - Write Once Read Many)

### Object Hold

**Event-Based Hold**:

- Hold until event occurs
- Manual release required
- Applies to object (all versions)

**Temporary Hold**:

- Manual hold/release
- Investigation or legal purposes
- Can be removed any time

**With Versioning**: Holds apply per version

## Performance Considerations

### List Operations

**Impact of Versioning**:

- Listing includes all versions (if requested)
- More versions = more data to return
- Can impact list performance
- Use prefix/delimiter to limit scope

**Optimization**:

- List current versions only (default)
- Request versions only when needed
- Use pagination for large result sets

### Storage Operations

**No Performance Impact**:

- Read/write performance unchanged
- Versioning handled transparently
- No latency increase
- Scales to any number of versions

## Monitoring and Management

### Metrics to Track

**Version Count**:

- Total versions per object
- Non-current version count
- Delete markers

**Storage Usage**:

- Versioned object storage
- Version storage breakdown
- Cost attribution

**Lifecycle Actions**:

- Versions deleted per day
- Versions transitioned
- Delete markers cleaned

### Alerting

**Cost Alerts**:

- Unexpected version accumulation
- Storage cost increase
- Budget exceeded

**Operational Alerts**:

- High version count per object
- Rapid version creation
- Lifecycle failure

## Versioning Best Practices

### 1. Enable with Lifecycle

**Pattern**: Always pair versioning with lifecycle policies

```yaml
# Conceptual
Versioning: Enabled
Lifecycle: 

  - Keep 10 recent versions
  - Delete versions > 90 days non-current
```

**Benefit**: Automatic cost control

### 2. Monitor Version Count

**Practice**: Regular audits of version counts

**Tools**: Cloud Monitoring, custom scripts

**Action**: Adjust lifecycle if accumulation excessive

### 3. Test Recovery Procedures

**Practice**: Quarterly recovery drills

**Steps**:

1. Simulate deletion
2. Recover from version
3. Verify data integrity
4. Document procedure

### 4. Use for Appropriate Data

**Enable For**:

- Configuration files
- Important documents
- Critical application data
- Collaboration documents

**Don't Enable For**:

- Large media files (high cost)
- Temporary data
- Log files (use lifecycle)
- Frequently changing data

### 5. Document Version Policy

**Include**:

- Retention period
- Version count limits
- Recovery procedures
- Responsible parties

## Exam Focus Areas

### When to Use Versioning

- Accidental deletion protection
- Overwrite protection
- Compliance requirements
- Versioning vs other backup methods

### Cost Management

- Version storage cost calculation
- Lifecycle policies for versions
- Version count control
- Cost optimization strategies

### Lifecycle Integration

- Version-specific lifecycle rules
- Current vs non-current policies
- Delete marker cleanup
- Storage class transitions for versions

### Recovery Scenarios

- Deletion recovery process
- Overwrite recovery
- Ransomware recovery
- Version selection and restoration

### Architecture Patterns

- Version retention strategies
- Compliance patterns
- Cost-optimized versioning
- Versioning vs separate backups
