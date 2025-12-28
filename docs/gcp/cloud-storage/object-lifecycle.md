# Object Lifecycle Management

## Core Concepts

Object lifecycle management automatically transitions objects between storage classes or deletes them based on age, storage class, or other conditions. Understanding lifecycle policy design is critical for cost optimization without sacrificing data availability.

**Key Principle**: Automate data aging to optimize costs; manually managing storage classes for billions of objects is impractical.

## Lifecycle Policy Architecture

### How Lifecycle Works

**Daily Processing**:

- Google scans all objects daily
- Evaluates conditions against each object
- Applies first matching action
- Asynchronous execution (not immediate)
- No operation charges for lifecycle transitions

**Execution Timing**:

- Runs once per day
- Not real-time (up to 24-hour delay)
- Actions applied in batch
- Eventual consistency

**Order of Evaluation**:

- Policies evaluated top to bottom
- First matching condition wins
- No further evaluation after match
- Order matters in configuration

### Lifecycle Actions

**Delete**:

- Permanently delete object
- Cannot be recovered (unless versioning enabled)
- Frees storage space
- Stops storage charges

**SetStorageClass**:

- Transition to different storage class
- Change from Standard → Nearline → Coldline → Archive
- Reduce storage costs
- Increase retrieval costs

**AbortIncompleteMultipartUpload**:

- Delete incomplete multipart upload parts
- Clean up failed uploads
- Reduce storage waste
- Age-based deletion

## Lifecycle Conditions

### Age Condition

**Purpose**: Match objects older than specified days

**Syntax**: `age: <days>`

**Examples**:

```
age: 30    # Objects created >30 days ago
age: 90    # Objects created >90 days ago
age: 365   # Objects created >365 days ago
```

**Use Cases**:

- Data aging policies
- Compliance retention
- Cost optimization
- Automatic cleanup

**Calculation**: Days since object creation

### CreatedBefore Condition

**Purpose**: Match objects created before specific date

**Syntax**: `createdBefore: "YYYY-MM-DD"`

**Example**:

```
createdBefore: "2024-01-01"  # Created before Jan 1, 2024
```

**Use Cases**:

- One-time cleanup of old data
- Compliance deadline
- Project end date
- Historical cutoff

### MatchesStorageClass Condition

**Purpose**: Apply actions to specific storage classes

**Syntax**: `matchesStorageClass: [classes]`

**Examples**:

```
matchesStorageClass: ["STANDARD"]      # Only Standard objects
matchesStorageClass: ["NEARLINE"]      # Only Nearline objects
matchesStorageClass: ["STANDARD", "NEARLINE"]  # Multiple classes
```

**Use Cases**:

- Storage class transitions
- Multi-tier aging
- Selective deletion
- Class-specific policies

### NumberOfNewerVersions Condition

**Purpose**: Match non-current versions (with versioning enabled)

**Syntax**: `numberOfNewerVersions: <count>`

**Example**:

```
numberOfNewerVersions: 3  # Keep 3 most recent versions
```

**Use Cases**:

- Version pruning
- Limit version accumulation
- Control versioning costs
- Compliance with retention limits

### DaysSinceNoncurrentTime Condition

**Purpose**: Age of non-current versions

**Syntax**: `daysSinceNoncurrentTime: <days>`

**Example**:

```
daysSinceNoncurrentTime: 30  # Versions older than 30 days
```

**Use Cases**:

- Version lifecycle
- Age out old versions
- Compliance retention for versions

### MatchesPrefix Condition

**Purpose**: Match objects with specific prefix

**Syntax**: `matchesPrefix: [prefixes]`

**Example**:

```
matchesPrefix: ["logs/"]           # Objects in logs/ prefix
matchesPrefix: ["temp/", "tmp/"]  # Multiple prefixes
```

**Use Cases**:

- Folder-based policies
- Application-specific rules
- Data organization alignment

### MatchesSuffix Condition

**Purpose**: Match objects with specific suffix

**Syntax**: `matchesSuffix: [suffixes]`

**Example**:

```
matchesSuffix: [".tmp", ".log"]   # Temporary and log files
```

**Use Cases**:

- File type specific policies
- Temporary file cleanup
- Extension-based rules

## Common Lifecycle Patterns

### Pattern 1: Multi-Tier Data Aging

**Use Case**: Automatically reduce storage costs as data ages

**Policy**:

```yaml
# Conceptual representation
Rule 1: age 30 days + Standard → Nearline
Rule 2: age 90 days + Nearline → Coldline
Rule 3: age 365 days + Coldline → Archive
Rule 4: age 2555 days (7 years) → Delete
```

**Cost Impact**:

- 0-30 days: $0.020/GB/month (Standard)
- 30-90 days: $0.010/GB/month (Nearline)
- 90-365 days: $0.004/GB/month (Coldline)
- 365-2555 days: $0.0012/GB/month (Archive)
- After 2555 days: Deleted (zero cost)

**Use Cases**:

- Log storage and aging
- Data lake with predictable access patterns
- Compliance with cost optimization
- Analytics data lifecycle

### Pattern 2: Temporary Data Cleanup

**Use Case**: Automatically delete temporary or staging data

**Policy**:

```yaml
# Conceptual representation
Rule: age 7 days + matchesPrefix "temp/" → Delete
```

**Benefits**:

- Prevent storage waste
- Automatic cleanup
- No manual intervention
- Predictable costs

**Use Cases**:

- Temporary processing data
- Build artifacts
- Cache storage
- Staging areas

### Pattern 3: Compliance Retention

**Use Case**: Retain for compliance period, then delete

**Policy**:

```yaml
# Conceptual representation
Rule 1: age 30 days + Standard → Coldline (reduce cost immediately)
Rule 2: age 2555 days (7 years) → Delete (HIPAA, SOX)
```

**Compliance Requirements**:

- HIPAA: 6 years
- SOX: 7 years
- GDPR: As per agreement
- Industry-specific regulations

### Pattern 4: Version Pruning

**Use Case**: Limit number of versions per object

**Policy**:

```yaml
# Conceptual representation
Rule: numberOfNewerVersions 5 → Delete
```

**Benefits**:

- Control versioning costs
- Prevent unbounded growth
- Maintain recent history
- Balance protection and cost

**Consideration**: Keep enough versions for recovery needs

### Pattern 5: Graduated Versioning

**Use Case**: Age versions differently than current objects

**Policy**:

```yaml
# Conceptual representation
# Current object lifecycle
Rule 1: age 30 days + Standard → Nearline

# Version lifecycle
Rule 2: daysSinceNoncurrentTime 7 days → Nearline
Rule 3: daysSinceNoncurrentTime 30 days → Delete
```

**Benefits**:

- Aggressive version cleanup
- Protect current objects longer
- Reduce versioning costs
- Maintain recent version history

## Cost Optimization Strategies

### Early Transition Strategy

**Principle**: Transition to lower class as soon as minimum duration met

**Pattern**:

```
Standard → Nearline at 30 days
Nearline → Coldline at 90 days
Coldline → Archive at 365 days
```

**When Appropriate**:

- Predictable access decay
- Known access patterns
- Cost optimization priority
- Rare access after initial period

**When Inappropriate**:

- Unpredictable access
- Frequent retrieval needed
- Performance critical
- Unknown usage patterns

### Delayed Transition Strategy

**Principle**: Keep in higher class longer for flexibility

**Pattern**:

```
Standard for 90 days
Standard → Coldline at 90 days
Coldline → Archive at 365 days
Skip Nearline tier
```

**When Appropriate**:

- Occasional access within 90 days
- Avoid multiple transition costs
- Simpler policy
- Uncertain 30-90 day access

### Archive Quickly for Compliance

**Principle**: Move to Archive immediately for known cold data

**Pattern**:

```
Standard → Archive at 30 days (or immediately via object class)
```

**When Appropriate**:

- Known rare access
- Compliance-only data
- Cost critical
- Retrieval cost acceptable

**Consideration**: 365-day minimum retention in Archive

### Selective Lifecycle by Prefix

**Principle**: Different rules for different data types

**Pattern**:

```
logs/ → Aggressive aging (Archive at 30 days, Delete at 365)
backups/ → Moderate aging (Nearline at 30, Coldline at 90, keep long-term)
data/ → Conservative (Standard for 365 days, then Archive)
```

**Benefits**:

- Optimize each data type
- Granular control
- Balance competing requirements
- Application-specific policies

## Lifecycle and Early Deletion Fees

### Understanding Early Deletion

**Storage Class Minimums**:

- Standard: None
- Nearline: 30 days
- Coldline: 90 days
- Archive: 365 days

**Fee Calculation**:

```
Early deletion fee = (Remaining days / Days in month) × Monthly storage cost
```

**Example**: Delete Nearline object after 20 days

```
Minimum: 30 days
Used: 20 days
Remaining: 10 days

Fee: (10/30) × $0.010/GB = $0.0033/GB
```

### Lifecycle Transition Impact

**Transition Timing**:

- Lifecycle transition counts as delete + create
- Early deletion fees apply
- Must account for minimum durations

**Example**: Standard → Nearline at 25 days → Coldline at 60 days

```
Nearline duration: 35 days (60 - 25)
Coldline minimum: 90 days

Problem: Transitioned to Coldline before Nearline minimum
Solution: Wait until day 115 (25 + 90) for Coldline transition
```

**Architecture Decision**: Plan transitions to avoid early deletion fees

### Optimal Transition Timing

**Formula**:

```
Transition to Nearline: Day 30 minimum
Transition to Coldline: Day 120 minimum (30 + 90)
Transition to Archive: Day 485 minimum (30 + 90 + 365)
```

**Policy Design**:

```yaml
# Conceptual - Avoid early deletion fees
Rule 1: age 30 days + Standard → Nearline
Rule 2: age 120 days + Nearline → Coldline  # Not 90!
Rule 3: age 485 days + Coldline → Archive   # Not 365!
```

## Lifecycle Policy Limitations

### Constraints

**Policy Limits**:

- 100 rules per bucket maximum
- Conditions combined with AND logic
- Actions mutually exclusive per rule
- Daily execution only (not real-time)

**Transition Restrictions**:

- Can only transition to "colder" classes
- Cannot transition Archive → Coldline → Nearline → Standard
- One-way transitions only
- No circular transitions

**Architecture Implication**: Design forward-only aging policies

### Performance Considerations

**Asynchronous Execution**:

- Up to 24-hour delay
- Not for time-critical operations
- Eventual consistency
- Best-effort timing

**Large Bucket Impact**:

- Billions of objects processed daily
- No performance impact on bucket operations
- Lifecycle runs independently
- Scalable to any size

## Lifecycle and Versioning Interaction

### Version-Aware Policies

**Non-Current Versions**:

- Lifecycle can target versions separately
- Different rules for current vs non-current
- Version-specific conditions
- Control version accumulation

**Pattern**: Aggressive version cleanup

```yaml
# Conceptual
Current objects:
  age 30 days → Nearline

Non-current versions:
  numberOfNewerVersions 3 → Delete
  daysSinceNoncurrentTime 30 → Delete
```

**Benefits**:

- Keep current objects longer
- Limit version storage costs
- Maintain recent version history
- Automatic version pruning

## Monitoring and Validation

### Metrics to Track

**Lifecycle Operations**:

- Objects transitioned per day
- Objects deleted per day
- Storage class distribution
- Cost trends

**Cost Impact**:

- Storage cost reduction
- Early deletion fees incurred
- Net cost savings
- Return on investment

### Testing Lifecycle Policies

**Best Practices**:

1. **Test on subset first**: Create test bucket with sample data
2. **Verify timing**: Confirm transitions occur as expected
3. **Monitor costs**: Check for early deletion fees
4. **Review regularly**: Access patterns change over time
5. **Adjust as needed**: Refine based on actual usage

**Test Bucket Pattern**:

- Create separate bucket
- Copy representative sample
- Apply lifecycle policy
- Monitor for 30-90 days
- Validate behavior
- Apply to production

## Lifecycle vs Autoclass

### Comparison

| Feature | Lifecycle Policies | Autoclass |
|---------|-------------------|-----------|
| **Control** | Explicit rules | Automatic |
| **Flexibility** | High (custom rules) | Limited (fixed algorithm) |
| **Complexity** | Medium-High | Low |
| **Cost** | Free | Small management fee |
| **Optimization** | Manual tuning | Automatic |
| **Predictability** | High | Variable |

### When to Use Each

**Lifecycle Policies**:

- Known access patterns
- Specific compliance rules
- Complex multi-tier aging
- Cost optimization priority
- Need full control

**Autoclass**:

- Unknown access patterns
- Simple automation preferred
- Variable object access
- Hands-off management
- Willing to pay management fee

**Hybrid Approach**:

- Autoclass for main data (unknown patterns)
- Lifecycle for cleanup (delete old data)
- Lifecycle for compliance (retention rules)

## Exam Focus Areas

### Policy Design

- Multi-tier aging strategies
- Transition timing to avoid early deletion fees
- Condition selection and combination
- Order of rule evaluation

### Cost Optimization

- Storage class transition economics
- Early deletion fee calculation
- Break-even analysis
- Total cost of ownership

### Use Case Matching

- Compliance retention policies
- Temporary data cleanup
- Data lake aging
- Version management

### Architecture Patterns

- Log lifecycle management
- Backup retention
- Data archival strategies
- Cost-optimized storage

### Lifecycle vs Alternatives

- When to use lifecycle vs Autoclass
- Manual class selection vs automatic
- Versioning interaction
- Performance implications
