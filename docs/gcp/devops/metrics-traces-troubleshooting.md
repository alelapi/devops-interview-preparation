# 4.3 – Metrics, Dashboards & Alerts  |  4.4 – Distributed Tracing  |  4.5 – Troubleshooting

---

# 4.3 – Managing Metrics, Dashboards, and Alerts

## Metrics Explorer

- Explore, visualize, and query any metric in Cloud Monitoring
- Supports: **MQL** (Monitoring Query Language), **PromQL** (for Prometheus-style queries)
- Can overlay multiple metrics, apply aggregation, ratio queries

### PromQL in Cloud Monitoring

```promql
# Request rate
rate(http_requests_total[5m])

# Error ratio
sum(rate(http_requests_total{status=~"5.."}[5m]))
  / sum(rate(http_requests_total[5m]))

# P95 latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

### MQL (Monitoring Query Language)

```mql
# CPU utilization for GKE pods
fetch k8s_container
| metric 'kubernetes.io/container/cpu/core_usage_time'
| filter resource.cluster_name = 'prod-cluster'
| align rate(1m)
| every 1m
| group_by [resource.namespace_name], [value: mean(value)]
```

---

## Dashboards

### Creating Dashboards

```bash
# Create dashboard from JSON
gcloud monitoring dashboards create --config-from-file=dashboard.json

# List dashboards
gcloud monitoring dashboards list
```

**Dashboard JSON structure:**
```json
{
  "displayName": "Application Overview",
  "gridLayout": {
    "columns": 2,
    "widgets": [
      {
        "title": "Request Rate",
        "xyChart": {
          "dataSets": [{
            "timeSeriesQuery": {
              "prometheusQuery": "rate(http_requests_total[5m])"
            }
          }]
        }
      }
    ]
  }
}
```

### Dashboard Best Practices
- Include: **golden signals** (latency, traffic, errors, saturation)
- Group by: service, environment, region
- Add **playbooks** as text widgets — guides during incidents
- Use **alert annotations** to show alert events on time series charts
- Share dashboards via URL — accessible to anyone with Cloud Monitoring viewer role

---

## Alerting Policies

### Alert Policy Components
1. **Condition**: what metric/log to evaluate + threshold
2. **Notification channels**: where to send alerts (email, Slack, PagerDuty, webhook, Pub/Sub)
3. **Documentation**: runbook URL, custom message for on-call engineer

```bash
# Create alert via gcloud
gcloud monitoring policies create \
  --display-name="High Error Rate" \
  --condition-display-name="Error rate > 5%" \
  --condition-filter='resource.type="k8s_container" AND metric.type="prometheus.googleapis.com/http_request_errors_total/counter"' \
  --condition-threshold-value=0.05 \
  --condition-comparison=COMPARISON_GT \
  --condition-duration=300s \
  --notification-channels=CHANNEL_ID
```

### Notification Channels

```bash
# Create email notification channel
gcloud monitoring channels create \
  --display-name="OpsTeam Email" \
  --type=email \
  --channel-labels=email_address=ops@company.com

# List channels
gcloud monitoring channels list

# Third-party integrations: webhook (for PagerDuty, Rootly, Opsgenie)
gcloud monitoring channels create \
  --display-name="PagerDuty" \
  --type=webhook_tokenauth \
  --channel-labels=url=https://events.pagerduty.com/integration/KEY/enqueue
```

### SLO-Based Alerting

```bash
# Alert on SLO burn rate
gcloud monitoring policies create \
  --display-name="SLO Fast Burn" \
  --condition-filter='select_slo_burn_rate("projects/PROJECT/services/svc/serviceLevelObjectives/slo")' \
  --condition-threshold-value=14.4 \
  --condition-comparison=COMPARISON_GT \
  --condition-duration=3600s \
  --notification-channels=CHANNEL_ID
```

### Cost Control Alerting

```bash
# Budget alert
gcloud billing budgets create \
  --billing-account=BILLING_ACCOUNT_ID \
  --display-name="Monthly Budget Alert" \
  --budget-amount=1000USD \
  --threshold-rule=percent=80,basis=CURRENT_SPEND \
  --threshold-rule=percent=100,basis=FORECASTED_SPEND
```

---

## Third-Party Alerting Integrations

| Tool | Integration method |
|---|---|
| **PagerDuty** | Webhook notification channel → PagerDuty Events API |
| **Rootly** | Webhook notification channel |
| **Opsgenie** | Webhook notification channel |
| **Slack** | Slack notification channel (built-in) |
| **Pub/Sub** | Route alerts to Pub/Sub → custom handler |

---

## Gemini Cloud Assist for Metrics

- In Metrics Explorer: "Show me CPU usage for my production GKE cluster"
- Interpret anomalies: "Why is my error rate spiking?"
- Generate alert conditions from natural language

---

# 4.4 – Distributed Tracing

## Core Concepts

- A **trace** represents an end-to-end request across services
- A **span** represents a single operation within a trace (one service call, one DB query)
- **Trace ID**: unique ID propagated across all services for a request
- **Parent span ID**: links child spans to parent, forming the trace tree

```
Trace (single request):
  span: API Gateway (100ms total)
    span: Auth Service (10ms)
    span: User Service (50ms)
      span: Database Query (30ms)
    span: Cache Lookup (5ms)
```

---

## OpenTelemetry (OTel)

GCP's preferred instrumentation framework — vendor-neutral, widely supported.

### Instrumentation (Python example)

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter

# Setup
provider = TracerProvider()
otlp_exporter = OTLPSpanExporter(endpoint="http://localhost:4317")  # Ops Agent OTLP receiver
provider.add_span_processor(BatchSpanProcessor(otlp_exporter))
trace.set_tracer_provider(provider)

tracer = trace.get_tracer("my-service")

# Instrument a function
with tracer.start_as_current_span("process-order") as span:
    span.set_attribute("order.id", order_id)
    span.set_attribute("user.id", user_id)
    result = process_order(order_id)
```

### Auto-instrumentation (no code changes)
```bash
# Python auto-instrumentation
pip install opentelemetry-distro opentelemetry-exporter-otlp
opentelemetry-bootstrap --action=install
opentelemetry-instrument python app.py
```

---

## Cloud Trace

- **Managed distributed tracing** service
- Collects spans from: Ops Agent OTLP receiver, direct API, auto-instrumented GCP services
- GCP services that auto-generate traces: Cloud Run, App Engine, GKE (with mesh), Cloud Functions

### Sending Traces to Cloud Trace

Option 1: **Ops Agent OTLP receiver** (recommended for VMs)
```yaml
# ops-agent config.yaml
traces:
  receivers:
    otlp:
      type: otlp
      grpc_endpoint: 0.0.0.0:4317
  exporters:
    google:
      type: google_cloud_trace
  service:
    pipelines:
      trace_pipeline:
        receivers: [otlp]
        exporters: [google]
```

Option 2: **Direct Cloud Trace exporter**
```python
from opentelemetry.exporter.cloud_trace import CloudTraceSpanExporter
exporter = CloudTraceSpanExporter(project_id="my-project")
```

---

## Analyzing Traces in Cloud Trace

### Trace Waterfall
- Visualize span hierarchy and timing
- Identify **bottlenecks**: longest spans in the critical path
- Look for: unexpected fan-out, sequential calls that could be parallel, slow DB queries

```
API Gateway [100ms] ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Auth Service [10ms] ━━━
  User Service [50ms]     ━━━━━━━━━━━━━━━
    DB Query [30ms]           ━━━━━━━━━
  Cache [5ms]                              ━━
```

### What to look for:
- **Long tail latency**: p99 >> p50 — investigate outlier spans
- **Missing spans**: broken trace propagation between services
- **High span count**: excessive calls (N+1 problem, chatty microservices)
- **Errors in spans**: `span.status = ERROR`

---

## Correlating Traces with Logs

### Structured Logging with Trace Context

```python
import logging
from opentelemetry import trace

# Get current trace context
span = trace.get_current_span()
ctx = span.get_span_context()

# Add trace fields to structured log
logging.info({
    "message": "Processing order",
    "trace": f"projects/PROJECT/traces/{format(ctx.trace_id, '032x')}",
    "spanId": format(ctx.span_id, '016x'),
    "traceSampled": True
})
```

**In Logs Explorer**: when a log has `trace` and `spanId` fields in the correct format, the UI shows a link to the trace → click through directly.

### Special log fields for trace correlation

```json
{
  "message": "Order processed",
  "logging.googleapis.com/trace": "projects/PROJECT/traces/TRACE_ID",
  "logging.googleapis.com/spanId": "SPAN_ID",
  "logging.googleapis.com/traceSampled": true
}
```

---

## Gemini Cloud Assist for Traces

- Analyze trace anomalies: "Why is this trace 10x slower than average?"
- Suggest which service is the bottleneck
- Explain unusual patterns in trace waterfall

---

# 4.5 – Troubleshooting Issues

## Troubleshooting Framework

1. **Define the symptom** — user impact, not internal metric
2. **Check recent changes** — deployments, config changes, infra changes
3. **Examine the golden signals** — latency, traffic, errors, saturation
4. **Drill down** — logs → traces → metrics for the affected service
5. **Isolate the component** — narrow to service → pod → node → dependency
6. **Mitigate** — rollback, add capacity, redirect traffic (fix user impact first)
7. **Root cause** — after user impact resolved

---

## Infrastructure Issues

```bash
# Node issues in GKE
kubectl get nodes
kubectl describe node NODE_NAME  # Events section

# Pod scheduling failures
kubectl get events --field-selector reason=FailedScheduling
kubectl describe pod POD_NAME

# Resource exhaustion
kubectl top nodes
kubectl top pods --all-namespaces

# GCE VM issues
gcloud compute instances describe INSTANCE --zone=ZONE
gcloud compute ssh INSTANCE --zone=ZONE  # Check /var/log/syslog
```

## CI/CD Pipeline Issues

```bash
# Cloud Build failure
gcloud builds list --filter="status=FAILURE" --limit=10
gcloud builds log BUILD_ID

# Cloud Deploy rollout failure
gcloud deploy rollouts describe ROLLOUT_NAME --delivery-pipeline=PIPELINE --region=REGION
gcloud logging read 'resource.type="clouddeploy.googleapis.com/DeliveryPipeline"' --limit=50
```

## Application Issues

```bash
# Find errors in GKE workloads
gcloud logging read \
  'resource.type="k8s_container" AND severity>=ERROR AND resource.labels.cluster_name="prod"' \
  --limit=100 --project=PROJECT

# Trace slow requests
# Cloud Trace → Trace List → Sort by latency → Examine waterfalls

# Check pod logs directly
kubectl logs DEPLOYMENT_NAME --tail=100 -f
kubectl logs POD_NAME -c CONTAINER_NAME --previous  # Previous crash logs
```

## Observability Issues

| Issue | Check |
|---|---|
| Metrics not appearing | SA permissions (`roles/monitoring.metricWriter`), Ops Agent running |
| Logs not ingested | SA permissions (`roles/logging.logWriter`), check Ops Agent status |
| Traces not showing | OTel exporter config, SA permissions (`roles/cloudtrace.agent`) |
| Alerts not firing | Alert condition, notification channel, alert policy status |
| Uptime check failing | Check endpoint accessibility from global probers, firewall rules |

## Performance and Latency Issues

```bash
# Check for resource throttling in GKE
kubectl top pods -n production
kubectl describe pod POD --namespace=production | grep -A5 "Limits:"

# Check Cloud Run cold starts
gcloud logging read \
  'resource.type="cloud_run_revision" AND textPayload:"Cold Start"'

# Check for GCE network performance
gcloud compute instances get-serial-port-output INSTANCE --zone=ZONE
```

### Common Performance Root Causes

| Symptom | Likely cause | Fix |
|---|---|---|
| p99 latency spike | GC pauses, cold starts | JVM tuning, `--min-instances` |
| Gradual latency increase | Memory leak, connection pool exhaustion | Rolling restart, connection pool tuning |
| Sudden error rate spike | New deployment, dependency failure | Rollback, circuit breaker |
| Periodic latency spikes | Cron jobs, batch processing | Schedule off-peak, resource limits |
| High CPU throttling (K8s) | CPU limits too low | Increase limits or remove (VPA) |

---

## Exam Tips

- **Golden signals**: Latency, Traffic, Errors, Saturation (L-T-E-S)
- PromQL supported natively in Cloud Monitoring (GMP) — use for Kubernetes metrics
- Trace correlation in logs requires: `logging.googleapis.com/trace` and `logging.googleapis.com/spanId` fields
- Cloud Trace auto-instruments: Cloud Run, App Engine, some GKE configs
- Ops Agent OTLP receiver = unified way to collect app traces/metrics on VMs
- `kubectl logs --previous` = logs from the **previous container instance** (after crash)
- Gemini Cloud Assist available in: Logs Explorer, Metrics Explorer, Cloud Trace UI
