# Cloud Storage Classes

## Core Concepts

Storage classes determine pricing and access characteristics for objects in Cloud Storage. Understanding the economic trade-offs between storage cost, retrieval cost, and minimum storage duration is critical for cost-effective architecture.

**Key Principle**: Lower storage cost = higher retrieval cost + minimum duration penalties. Match storage class to actual access patterns.

## Storage Class Comparison

| Class | Access Frequency | Storage Cost | Retrieval Cost | Min Duration | Availability | Use Case |
|-------|-----------------|--------------|----------------|--------------|--------------|----------|
| **Standard** | >1/month | Highest | None | None | Highest | Hot data |
| **Nearline** | ~1/month | Medium | Low | 30 days | High | Warm data |
| **Coldline** | ~1/quarter | Low | Medium | 90 days | High | Cool data |
| **Archive** | <1/year | Lowest | Highest | 365 days | High | Cold data |
| **Autoclass** | Unknown | Automatic | Varies | None | Varies | Unpredictable |

## Standard Storage Class

### Characteristics

**Pricing** (approximate, varies by location):

- Storage: $0.020 per GB/month (multi-region)
- Retrieval: No charge
- Operations: Class A ~$0.05/10K, Class B ~$0.004/10K
- Minimum duration: None
- Early deletion fee: None

**Availability**:

- Multi-region: 99.95%
- Dual-region: 99.95%
- Region: 99.9%

### When to Use

✅ **Appropriate for**:

**Frequently Accessed Data**:

- Website content and assets
- Streaming media actively used
- Mobile/web application data
- Analytics data in active use
- Content being actively processed

**Short-Term Storage**:

- Temporary processing data
- Build artifacts (recent builds)
- Staging data for pipelines
- Cache storage

**Performance-Critical**:

- Low latency required
- High throughput needed
- Frequent access (>1/month)
- No retrieval delay acceptable

### When NOT to Use

❌ **Inappropriate for**:

- Data accessed less than monthly
- Long-term archives
- Backup data rarely accessed
- Historical data for compliance only
- Aged logs and analytics

**Cost Impact**: Paying premium storage cost for infrequent access wastes money

## Nearline Storage Class

### Characteristics

**Pricing**:

- Storage: $0.010 per GB/month (~50% of Standard)
- Retrieval: $0.01 per GB retrieved
- Minimum duration: 30 days
- Early deletion fee: Charged for remaining days

**Availability**: Same as Standard

### Economics

**Break-Even Analysis**:

```
When is Nearline cheaper than Standard?

Standard cost: $0.020/GB/month
Nearline storage: $0.010/GB/month
Nearline retrieval: $0.01/GB

Nearline is cheaper if:
Access frequency < 1 full retrieval per month
(0.010 storage + 0.01 retrieval) < 0.020
```

**Example**: 1 TB data, accessed 50% monthly

- Standard: 1000 × $0.020 = $20/month
- Nearline: (1000 × $0.010) + (500 × $0.01) = $15/month
- **Savings: $5/month (25%)**

### When to Use

✅ **Appropriate for**:

**Monthly Access Patterns**:

- Monthly reports and analytics
- Backup data accessed for recovery testing
- Aged data referenced occasionally
- Compliance data with periodic review
- Media assets for seasonal campaigns

**Data Retention Requirements**:

- Regulatory data (30+ day retention)
- Application logs (older than 30 days)
- Database backups (recent history)
- Development artifacts (older versions)

### When NOT to Use

❌ **Inappropriate for**:

**Short-Term Storage** (<30 days):

- Temporary processing data
- Short-lived cache
- Data deleted within 30 days
- Rapid turnover data

**Why**: Early deletion fees negate cost savings

**Frequent Access** (>1/month):

- Active application data
- Frequently accessed logs
- Primary datasets
- Live analytics data

**Why**: Retrieval costs exceed Standard storage cost

### Early Deletion Considerations

**Scenario**: Delete object after 20 days

- Charged for 30 days minimum
- Remaining 10 days × storage rate
- No savings over Standard for short-term data

**Architecture Decision**: Only use Nearline if data lives >30 days

## Coldline Storage Class

### Characteristics

**Pricing**:

- Storage: $0.004 per GB/month (~20% of Standard)
- Retrieval: $0.02 per GB retrieved
- Minimum duration: 90 days
- Early deletion fee: Charged for remaining days

**Availability**: Same as Standard

### Economics

**Break-Even Analysis**:

```
Coldline is cheaper than Standard if:
Access frequency < 0.8 full retrievals per month

Coldline is cheaper than Nearline if:
Access frequency < 0.3 full retrievals per month
```

**Example**: 1 TB data, accessed 10% quarterly (3.3% monthly)

- Standard: $20/month
- Nearline: $10 + $3.33 = $13.33/month
- Coldline: $4 + $0.67 = $4.67/month
- **Savings: $15.33/month (77% vs Standard)**

### When to Use

✅ **Appropriate for**:

**Quarterly Access**:

- Quarterly business reports
- Disaster recovery data (tested quarterly)
- Audit data (occasional review)
- Seasonal data archives
- Aged analytics data

**Compliance and Archival**:

- Regulatory data (90+ day retention)
- Legal documents (occasional access)
- Historical records
- Long-term backups (recent years)

**Infrequently Accessed Backups**:

- Database backups (older than 90 days)
- VM snapshots (disaster recovery)
- Application state backups
- Configuration backups

### When NOT to Use

❌ **Inappropriate for**:

**Short-Term Storage** (<90 days):

- Early deletion fees expensive
- No cost benefit

**Frequent Access** (>1/quarter):

- Retrieval costs accumulate
- Nearline or Standard cheaper

**Performance-Critical**:

- Though retrieval is fast, cost implies rare access
- Use Standard for performance needs

## Archive Storage Class

### Characteristics

**Pricing**:

- Storage: $0.0012 per GB/month (~6% of Standard)
- Retrieval: $0.05 per GB retrieved
- Minimum duration: 365 days
- Early deletion fee: Charged for remaining days

**Availability**: Same as Standard (despite name suggesting slower access)

### Economics

**Break-Even Analysis**:

```
Archive is cheaper than Standard if:
Access frequency < 0.38 full retrievals per month

Archive is cheaper than Coldline if:
Access frequency < 0.05 full retrievals per month
```

**Example**: 1 TB data, accessed once per year (8.3% monthly)

- Standard: $20/month × 12 = $240/year
- Coldline: $4/month × 12 + $16.6 = $64.6/year
- Archive: $1.20/month × 12 + $4.2 = $18.6/year
- **Savings: $221.4/year (92% vs Standard)**

### When to Use

✅ **Appropriate for**:

**Annual or Rare Access**:

- Compliance archives (multi-year retention)
- Historical data (rarely accessed)
- Long-term backups (disaster scenarios only)
- Legal archives
- Scientific data preservation

**Regulatory Requirements**:

- 7-year retention (SOX, HIPAA)
- Legal hold data
- Audit trails
- Medical records

**Cold Data**:

- Data rarely if ever accessed
- "Just in case" storage
- Regulatory compliance only
- No business use but cannot delete

### When NOT to Use

❌ **Inappropriate for**:

**Any Regular Access**:

- Monthly, quarterly, even semi-annual access expensive
- Very high retrieval costs
- Use Coldline instead

**Short-Term Storage** (<365 days):

- Massive early deletion fees
- Most expensive option for short-term

**Active Archives**:

- "Archive" doesn't mean slower access
- Use based on access frequency, not retrieval speed
- If accessed regularly, use different class

**Common Misconception**: "Archive is for backups"

- **Reality**: Archive is for RARELY accessed data
- Backups may be accessed frequently (use Nearline/Coldline)
- Archive is for compliance, not operational recovery

## Autoclass

### Characteristics

**How It Works**:

- Automatically transitions objects between classes
- Based on access patterns
- Starts as Standard
- Moves to Nearline after 30 days without access
- Moves to Coldline after 90 days without access
- Moves to Archive after 365 days without access (if enabled)
- Moves back to Standard when accessed

**Pricing**:

- Storage cost based on class object is in
- No retrieval fees for automatic transitions
- Small management fee (~$0.0025 per 1000 objects/month)
- No minimum storage duration penalties

### When to Use

✅ **Appropriate for**:

**Unknown Access Patterns**:

- New data lake
- Uncertain usage patterns
- Variable access across objects
- Mixed workloads

**Simplification**:

- Don't want to manage lifecycle policies
- Automated optimization preferred
- Mixed access patterns in bucket
- Hands-off cost optimization

**Testing and Development**:

- Unpredictable access
- Changing requirements
- Experimental data

### When NOT to Use

❌ **Inappropriate for**:

**Known Access Patterns**:

- Predictable usage (set specific class)
- Consistent access frequency
- Better cost control with manual selection

**Frequent Access**:

- If all data accessed frequently, use Standard
- Autoclass overhead not needed

**Compliance Requirements**:

- Need specific storage class guarantees
- Regulatory requirements for storage type
- Audit requirements

**Cost Optimization**:

- Autoclass convenient but may not be cheapest
- Manual lifecycle policies can be more cost-effective
- Pay for management overhead

### Trade-offs

**Benefits**:

- Automatic optimization
- No lifecycle policy management
- Adapts to changing patterns
- Simple to implement

**Drawbacks**:

- Management fee
- Less control
- May not be optimal for all objects
- Harder to predict costs

## Decision Framework

### Access Pattern Analysis

**Questions to Answer**:

1. **How often will data be accessed?**

   - Daily/weekly → Standard
   - Monthly → Nearline
   - Quarterly → Coldline
   - Yearly/rare → Archive

2. **How long will data be stored?**

   - <30 days → Standard only
   - 30-90 days → Standard or Nearline
   - 90-365 days → Standard, Nearline, or Coldline
   - >365 days → Any class

3. **What percentage retrieved when accessed?**

   - Full dataset → Factor full retrieval cost
   - Partial → Lower effective retrieval cost

4. **Is access pattern predictable?**

   - Yes → Choose specific class
   - No → Consider Autoclass

5. **Are there minimum storage requirements?**

   - Short-term data → Beware early deletion fees
   - Long-term data → All classes viable

### Cost Calculation Formula

**Total Monthly Cost** = Storage Cost + Retrieval Cost + Operation Cost

```
Standard:    (GB × $0.020) + (0 retrieval) + ops
Nearline:    (GB × $0.010) + (GB_retrieved × $0.01) + ops
Coldline:    (GB × $0.004) + (GB_retrieved × $0.02) + ops
Archive:     (GB × $0.0012) + (GB_retrieved × $0.05) + ops
```

**Example**: 10 TB stored, 20% accessed monthly

- Standard: 10,000 × 0.020 = $200
- Nearline: (10,000 × 0.010) + (2,000 × 0.01) = $120
- Coldline: (10,000 × 0.004) + (2,000 × 0.02) = $80
- Archive: (10,000 × 0.0012) + (2,000 × 0.05) = $112

**Best choice: Coldline ($80/month)**

## Multi-Class Bucket Strategy

### Architectural Pattern

**Single Bucket, Multiple Classes**:

- Objects can have different storage classes in same bucket
- Bucket has default class for new objects
- Individual objects can override
- Lifecycle policies transition between classes

**Use Case**: Data aging pattern

```
New objects → Standard (default)
After 30 days → Nearline (lifecycle policy)
After 90 days → Coldline (lifecycle policy)
After 365 days → Archive (lifecycle policy)
After 7 years → Delete (compliance)
```

## Common Mistakes

### Mistake 1: Using Archive for Backups

**Problem**: Backups often accessed for testing and recovery

**Solution**: Use Nearline or Coldline based on access frequency

### Mistake 2: Ignoring Early Deletion Fees

**Problem**: Delete Nearline object after 20 days, pay for 30 days

**Solution**: Use Standard for short-lived data

### Mistake 3: Overusing Standard

**Problem**: Paying premium for data accessed monthly

**Solution**: Analyze access patterns, use Nearline for monthly access

### Mistake 4: Underestimating Retrieval Costs

**Problem**: Archive looks cheap but retrieval costs accumulate

**Solution**: Calculate total cost including retrieval frequency

### Mistake 5: Autoclass for Everything

**Problem**: Management fees and less control

**Solution**: Use Autoclass only when access patterns unknown

## Exam Focus Areas

### Storage Class Selection

- Economic trade-offs (storage vs retrieval vs minimum duration)
- Access pattern matching
- Break-even analysis
- Early deletion fee scenarios

### Cost Optimization

- Lifecycle policies to transition classes
- Appropriate class for use case
- Total cost calculation (not just storage)
- Multi-class bucket strategies

### Architecture Patterns

- Data aging strategies
- Backup and archive distinction
- Compliance requirements
- Performance vs cost trade-offs

### Common Scenarios

- Hot/warm/cold data patterns
- Compliance and retention
- Disaster recovery storage
- Data lake storage classes

### Decision Making

- When to use each class
- When NOT to use each class
- Autoclass vs manual selection
- Multi-class bucket design
