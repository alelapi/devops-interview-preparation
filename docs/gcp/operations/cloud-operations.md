# Cloud Operations

## Core Concepts

Cloud Operations (formerly Stackdriver) is a suite of tools for monitoring, logging, debugging, and managing applications and infrastructure. Provides unified observability across GCP and other clouds.

**Key Principle**: Observability through metrics, logs, and traces; proactive monitoring, not reactive troubleshooting.

## Cloud Operations Suite

| Service | Purpose | Key Feature |
|---------|---------|-------------|
| **Cloud Monitoring** | Metrics, dashboards, alerts | Time-series data, SLOs |
| **Cloud Logging** | Log collection and analysis | Centralized logs, filters |
| **Cloud Trace** | Distributed tracing | Request latency analysis |
| **Cloud Profiler** | CPU/memory profiling | Production profiling |
| **Cloud Debugger** | Live debugging | No code changes, snapshots |
| **Error Reporting** | Error aggregation | Smart grouping, notifications |

## Cloud Monitoring

### Purpose

Collect, visualize, and alert on metrics from GCP resources, applications, and external sources.

### Key Features

**Metrics Collection**:

- Automatic (GCP resources)
- Custom (application metrics)
- Agent-based (VM detailed metrics)

**Dashboards**:

- Predefined (GCE, GKE, etc.)
- Custom (MQL, PromQL)
- Charts, tables, heatmaps

**Alerting**:

- Metric-based (CPU > 80%)
- Log-based (error rate spikes)
- Uptime checks (availability)
- Multi-condition policies

**SLIs and SLOs**:

- Service-level indicators (latency, availability)
- Service-level objectives (99.9% uptime)
- Error budget tracking

### Common Patterns

**Infrastructure Monitoring**:

```
Alert: CPU > 80% for 5 minutes → Page on-call
Alert: Disk usage > 90% → Auto-expand or alert
Uptime check: HTTP endpoint unavailable → Alert
```

**Application Performance**:

```
SLO: 99.9% requests < 500ms
Error Budget: Calculate remaining tolerance
Alert: Error budget burn rate too high
```

### Monitoring Agent

**Purpose**: Collect system and application metrics from VMs

**Installation**: Optional but recommended for detailed metrics

**Benefits**: Memory, disk I/O, process metrics

## Cloud Logging

### Purpose

Centralized log management: collection, storage, analysis, and alerting.

### Log Types

**Platform Logs** (automatic):

- Admin Activity (who did what)
- Data Access (who accessed data, must enable)
- System Events (GCP actions)
- Access Transparency (Google admin access)

**Application Logs**:

- Stdout/stderr (automatic for many services)
- Structured logging (recommended)
- Custom log writes

### Log Routing

**Default behavior**: Logs stored in Cloud Logging for 30 days

**Sinks**: Export logs to destinations

- Cloud Storage (long-term archival)
- BigQuery (analysis, SQL queries)
- Pub/Sub (streaming to external systems)
- Other projects (centralized logging)

**Filters**: Route specific logs (e.g., only errors)

### Log-Based Metrics

**Purpose**: Create metrics from log entries

**Use cases**:

- Count error occurrences
- Track specific events
- Custom business metrics
- Alert on log patterns

**Example**: Alert when 5XX errors > 10/minute

### Best Practices

**Structured Logging**:

```json
{
  "severity": "ERROR",
  "message": "Payment failed",
  "userId": "123",
  "amount": 99.99
}
```

**Benefits**: Searchable fields, better analysis

**Log Sampling**: Reduce volume for high-traffic apps (sample 10%)

**Retention**: Default 30 days, export to Storage for longer

## Cloud Trace

### Purpose

Distributed tracing for understanding request latency across services.

### How It Works

```
Request → Service A → Service B → Service C
         └─── Trace spans collected ────┘
```

**Trace**: End-to-end request journey
**Span**: Single operation within trace

### Use Cases

- Identify slow services in request path
- Understand service dependencies
- Optimize critical paths
- Troubleshoot latency issues

### Automatic Instrumentation

**Supported**:

- App Engine (automatic)
- Cloud Run (automatic)
- GKE (with service mesh)

**Manual**:

- Compute Engine (use client libraries)
- Other services (OpenTelemetry)

### Analysis

**Features**:

- Latency distribution
- Request waterfall view
- Service dependency graph
- Comparison across traces

**Example**: "95% of requests to Service A take 200ms, but 5% take 5s due to Service B dependency"

## Cloud Profiler

### Purpose

Continuous CPU and memory profiling in production with minimal overhead.

### Key Features

- No performance impact (<1% overhead)
- Always-on profiling
- Multiple languages (Java, Python, Go, Node.js)
- Compare time periods

### Use Cases

- Identify performance bottlenecks
- Optimize resource usage
- Reduce compute costs
- Memory leak detection

### Best Practice

**Enable in production**: Designed for production use, safe

## Cloud Debugger

### Purpose

Debug production applications without stopping or restarting.

### How It Works

**Snapshots**: Capture variable state at specific line

**Logpoints**: Inject log statement without code changes

**No downtime**: Debug live production apps

### Limitations

- Snapshots expire after capture
- Not all languages supported
- Cannot modify state (read-only)

### Use Case

Troubleshooting production issues without redeployment

## Error Reporting

### Purpose

Aggregate and display errors from applications, with smart grouping and notifications.

### Features

**Smart Grouping**: Similar errors grouped together

**Stack Trace**: Full stack traces for debugging

**Notifications**: Email, mobile alerts on new errors

**Integration**: Works with Cloud Logging automatically

### Supported Services

- App Engine, Cloud Functions, Cloud Run (automatic)
- Compute Engine, GKE (via Logging agent)
- External applications (via API)

## Unified Observability

### The Three Pillars

**Metrics** (Cloud Monitoring):

- What is happening (CPU, memory, requests)
- When did it happen
- Historical trends

**Logs** (Cloud Logging):

- Why it happened
- Detailed context
- Debugging information

**Traces** (Cloud Trace):

- How requests flow through system
- Where latency occurs
- Service dependencies

**Together**: Complete picture of system health

## SLI, SLO, and SLA

### Definitions

**SLI** (Service Level Indicator):

- Quantitative measure of service level
- Examples: Latency, availability, error rate

**SLO** (Service Level Objective):

- Target value for SLI
- Examples: 99.9% availability, 95th percentile latency < 200ms

**SLA** (Service Level Agreement):

- Contractual commitment
- Penalties if SLO not met
- Example: 99.95% uptime or refund

### Relationship

```
SLI (measurement) → SLO (internal target) → SLA (customer contract)
```

### Error Budget

**Concept**: Allowed downtime based on SLO

**Example**: 99.9% availability SLO = 43.2 minutes downtime/month allowed

**Use**: Prioritize features vs reliability

## Monitoring Strategy

### What to Monitor

**Golden Signals** (Google SRE):

- **Latency**: Request duration
- **Traffic**: Request rate
- **Errors**: Failed requests
- **Saturation**: Resource utilization

**Infrastructure**:

- CPU, memory, disk, network
- Service health
- Resource quotas

**Application**:

- Business metrics (orders, payments)
- User experience (page load time)
- Custom KPIs

### Alert Design

**Good Alerts**:

- Actionable (can fix)
- Represent real problems
- Rarely false positives

**Bad Alerts**:

- Noisy (too many)
- Not actionable
- Alert fatigue

**Best Practice**: Alert on SLO burn rate, not arbitrary thresholds

## Cost Optimization

**Logging**:

- Default 30-day retention (free)
- Export to Storage for cheaper long-term
- Use log exclusion filters (reduce volume)
- Sample high-volume logs

**Monitoring**:

- Free tier: 150 MB logs ingestion/month
- Custom metrics charged beyond free tier
- Use sampling for high-cardinality metrics

**Trace/Profiler/Debugger**: Free (no additional charge)

## Integration Patterns

### Multi-Cloud Monitoring

**Ops Agent**: Monitor GCP, AWS, on-premises from single dashboard

**Use case**: Unified monitoring across hybrid/multi-cloud

### Centralized Logging

**Pattern**: All projects route logs to central project

```
Project A logs → Sink → Central Logging Project
Project B logs → Sink → Central Logging Project
```

**Benefits**: Single pane of glass, better analysis

### Alert Routing

**Integration**:

- PagerDuty, Slack (notifications)
- Cloud Functions (automated remediation)
- Cloud Tasks (workflow orchestration)

## Compliance and Audit

### Audit Logs

**Types**:

- Admin Activity: Always enabled, 400-day retention
- Data Access: Must enable, 30-day default retention
- System Events: Automatic
- Access Transparency: Google employee access

**Use for**: Compliance evidence, security investigations

### Log Retention

**Compliance requirements**:

- HIPAA: 6 years
- SOX: 7 years
- PCI-DSS: 1 year

**Implementation**: Export logs to Cloud Storage, set retention policy

## Best Practices

### Monitoring

- Define SLOs for critical services
- Alert on SLO burn rate
- Use dashboards for visibility
- Regular review of alert policies

### Logging

- Use structured logging
- Sample high-volume logs
- Export for long-term retention
- Enable data access logs for sensitive resources

### Tracing

- Enable for all services
- Use for performance optimization
- Trace critical request paths
- Set sampling rate appropriately

### Observability

- Implement all three pillars (metrics, logs, traces)
- Correlate across pillars (same request ID)
- Monitor business metrics, not just infrastructure
- Proactive monitoring, not reactive

## Exam Focus

### Core Concepts

- Observability pillars (metrics, logs, traces)
- SLI vs SLO vs SLA
- Golden signals (latency, traffic, errors, saturation)
- Error budgets

### Service Purpose

- Cloud Monitoring: Metrics, alerts, dashboards
- Cloud Logging: Centralized logs, sinks
- Cloud Trace: Distributed tracing, latency
- Cloud Profiler: Production profiling
- Error Reporting: Error aggregation

### Architecture

- Log routing (sinks to Storage, BigQuery, Pub/Sub)
- Centralized logging pattern
- Multi-cloud monitoring
- Alert routing and automation

### Best Practices

- SLO-based alerting
- Structured logging
- Log retention for compliance
- Enable data access logs for sensitive resources
- Sample high-volume logs

### Integration

- Automatic (App Engine, Cloud Run, GKE)
- Agent-based (Compute Engine)
- API/SDK (custom applications)
- Multi-cloud (Ops Agent)
