# 4.1 – Instrumenting & Collecting Telemetry  |  4.2 – Managing & Analyzing Logs

---

# 4.1 – Instrumenting and Collecting Telemetry

## The Three Pillars of Observability

| Pillar | GCP Service | Data type |
|---|---|---|
| **Metrics** | Cloud Monitoring + Google Cloud Managed Service for Prometheus | Time-series numbers |
| **Logs** | Cloud Logging | Structured/unstructured text events |
| **Traces** | Cloud Trace | Request path + timing across services |

---

## Collecting Logs

### Ops Agent (Compute Engine)

- **Primary agent** for Compute Engine VMs
- Replaces legacy Stackdriver Logging agent + Monitoring agent
- Single process: uses **Fluent Bit** (logs) + **OpenTelemetry Collector** (metrics/traces)
- Supports: OTLP metrics and traces from instrumented apps

```bash
# Install Ops Agent on VM
curl -sSO https://dl.google.com/cloudagents/add-google-cloud-ops-agent-repo.sh
sudo bash add-google-cloud-ops-agent-repo.sh --also-install

# Check status
sudo systemctl status google-cloud-ops-agent
```

**Custom log collection (config.yaml):**
```yaml
logging:
  receivers:
    my_app_logs:
      type: files
      include_paths:
        - /var/log/myapp/*.log
  service:
    pipelines:
      my_pipeline:
        receivers: [my_app_logs]
```

### GKE Logging

- **System logs**: automatically collected (kubelet, container runtime, k8s control plane)
- **Application logs**: `stdout`/`stderr` from containers automatically ingested
- **Cloud Logging agent** (Fluentd/Fluent Bit DaemonSet) runs by default in GKE

```bash
# Enable logging on GKE cluster
gcloud container clusters update CLUSTER \
  --enable-cloud-logging \
  --logging=SYSTEM,WORKLOAD  # WORKLOAD = app logs
```

### Cloud Audit Logs

| Type | Contains | Default |
|---|---|---|
| **Admin Activity** | Who created/modified resources | Always on, free |
| **Data Access** | Who read/wrote data | Off by default, billable |
| **System Event** | GCP-automated resource changes | Always on, free |
| **Policy Denied** | IAM policy denials | Always on, free |

```bash
# Enable Data Access logs for a service
gcloud projects get-iam-policy PROJECT > policy.yaml
# Add auditConfigs section for service
# ...
gcloud projects set-iam-policy PROJECT policy.yaml
```

### VPC Flow Logs
- Record **network flow samples** for VPC subnets
- Useful for: network troubleshooting, security monitoring, traffic analysis
- Enable per subnet; configurable sampling rate (default 1/10 packets)

```bash
gcloud compute networks subnets update my-subnet \
  --region=us-central1 \
  --enable-flow-logs \
  --logging-flow-sampling=0.5 \
  --logging-aggregation-interval=interval-5-sec
```

### Cloud Service Mesh Logs
- Access logs, audit logs from Envoy sidecars
- Trace context propagated via headers (X-B3-TraceId, etc.)
- Configure via `Telemetry` CRD in Istio/Cloud Service Mesh

---

## Collecting Metrics

### GCP Platform Metrics (auto-collected)
- Every GCP service emits metrics to Cloud Monitoring automatically
- Prefix pattern: `compute.googleapis.com/`, `container.googleapis.com/`, `run.googleapis.com/`
- No configuration needed — always available

### Google Cloud Managed Service for Prometheus (GMP)
- Drop-in **managed Prometheus** for GKE and other workloads
- No Prometheus server to manage — GCP runs it
- Uses standard Prometheus scraping + PromQL
- Data stored in **Cloud Monitoring** (not separate Prometheus storage)

```yaml
# PodMonitoring CRD — scrape app metrics
apiVersion: monitoring.googleapis.com/v1
kind: PodMonitoring
metadata:
  name: app-metrics
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
  - port: metrics
    interval: 30s
```

```bash
# Enable GMP on GKE cluster
gcloud container clusters update CLUSTER \
  --enable-managed-prometheus
```

### Application Metrics (Custom)
- Use **OpenTelemetry SDK** in your app → export to Cloud Monitoring via Ops Agent OTLP receiver
- Or use **Cloud Monitoring client libraries** directly
- Custom metrics billed per sample after free tier

```python
from opentelemetry import metrics
from opentelemetry.exporter.otlp.proto.grpc.metric_exporter import OTLPMetricExporter

meter = metrics.get_meter("my-app")
request_counter = meter.create_counter("http.requests")
request_counter.add(1, {"method": "GET", "status": "200"})
```

### Hybrid/Multi-Cloud Metrics
- **Ops Agent** on non-GCP VMs (AWS EC2, on-prem) → sends to Cloud Monitoring
- **OpenTelemetry Collector** can collect from any source → forward to GCP
- Use **metric labels** to identify source environment

---

## Synthetic Monitoring

- **Uptime checks**: proactively test HTTP/HTTPS/TCP endpoints from multiple global locations
- **Synthetic monitors**: run custom scripts (Node.js) to test complex user journeys

```bash
# Create uptime check
gcloud monitoring uptime create my-check \
  --display-name="App Health" \
  --resource-type=uptime-url \
  --hostname=app.example.com \
  --port=443 \
  --request-path=/healthz \
  --check-interval=60s
```

**Synthetic monitor (Cloud Monitoring):**
- Runs a Cloud Function that executes test scripts
- Can test multi-step workflows (login → checkout → confirm)
- Results appear as metrics → alerting on failure

---

## Custom Metrics and Log-Based Metrics

### Custom Metrics

```bash
# Write a custom metric via API
gcloud monitoring time-series create \
  --project=PROJECT \
  custom.googleapis.com/myapp/queue_depth \
  --metric-kind=GAUGE \
  --value-type=INT64 \
  --points="[{'interval': {'endTime': '2024-01-01T00:00:00Z'}, 'value': {'int64Value': 42}}]"
```

### Log-Based Metrics
- Extract metrics from log entries — count errors, extract latency values from logs
- **Counter metric**: count log entries matching a filter
- **Distribution metric**: extract numeric values (e.g., latency) from structured logs

```bash
# Create counter metric for 5xx errors
gcloud logging metrics create http_5xx_errors \
  --description="Count of 5xx HTTP errors" \
  --log-filter='resource.type="k8s_container" AND httpRequest.status>=500'

# Distribution metric for latency
gcloud logging metrics create request_latency \
  --description="Request latency distribution" \
  --log-filter='resource.type="gce_instance" AND jsonPayload.latency!=""' \
  --value-extractor='EXTRACT(jsonPayload.latency)'
```

---

# 4.2 – Managing and Analyzing Logs

## Cloud Logging Architecture

```
Log Sources → Cloud Logging API → _Default bucket (30d retention)
                                → _Required bucket (400d, admin activity)
                                → Custom buckets (your retention)
                                → Log Router → Sinks → BigQuery / Pub/Sub / GCS
```

## Logs Explorer & Logging Query Language (LQL)

### Query Syntax

```bash
# Filter by resource type
resource.type="k8s_container"

# Filter by severity
severity>=ERROR

# Filter by label
resource.labels.cluster_name="prod-cluster"

# Filter by log name
logName="projects/PROJECT/logs/cloudaudit.googleapis.com%2Factivity"

# Full text search
"NullPointerException"

# Time range (combined with UI picker)
timestamp >= "2024-01-01T00:00:00Z" AND timestamp <= "2024-01-01T23:59:59Z"

# Structured field access
jsonPayload.status_code=500

# Boolean logic
(severity=ERROR OR severity=CRITICAL) AND resource.labels.namespace_name="production"

# Exclusion (negate)
NOT httpRequest.status=200
```

### Useful Filters for DevOps

```bash
# GKE container logs for a specific pod
resource.type="k8s_container"
resource.labels.pod_name:"myapp-"
severity>=WARNING

# Cloud Build failures
resource.type="build"
jsonPayload.status="FAILURE"

# Cloud Deploy rollout events
resource.type="clouddeploy.googleapis.com/DeliveryPipeline"

# K8s audit logs
logName="projects/PROJECT/logs/cloudaudit.googleapis.com%2Factivity"
protoPayload.resourceName:"namespaces/production"

# VPC flow logs - rejected traffic
resource.type="gce_subnetwork"
jsonPayload.disposition="DENIED"
```

---

## Log Sinks (Export and Routing)

```bash
# Export logs to BigQuery
gcloud logging sinks create my-bq-sink \
  bigquery.googleapis.com/projects/PROJECT/datasets/logs_dataset \
  --log-filter='severity>=WARNING' \
  --description="Warning+ logs to BigQuery"

# Export to GCS (archival)
gcloud logging sinks create archive-sink \
  storage.googleapis.com/my-log-archive-bucket \
  --log-filter='logName:"cloudaudit"' \
  --billing-project=PROJECT

# Export to Pub/Sub (streaming)
gcloud logging sinks create stream-sink \
  pubsub.googleapis.com/projects/PROJECT/topics/log-events \
  --log-filter='resource.type="k8s_container"'
```

### Grant sink SA write access
```bash
# After creating sink, grant SA permission to write to destination
SINK_SA=$(gcloud logging sinks describe my-bq-sink --format="value(writerIdentity)")
gcloud projects add-iam-policy-binding PROJECT \
  --member="$SINK_SA" \
  --role="roles/bigquery.dataEditor"
```

---

## Log Retention

| Bucket | Default retention | Notes |
|---|---|---|
| `_Default` | 30 days | Configurable (1-3650 days) |
| `_Required` | 400 days | Fixed — admin activity, system events; cannot reduce |
| Custom | You define | Create for long-term specific needs |

```bash
# Update retention on _Default bucket
gcloud logging buckets update _Default \
  --location=global \
  --retention-days=90
```

---

## Handling Sensitive Data (PII/PHI)

### Log Redaction Options
1. **Data Access Audit Log**: don't enable for services with PII in request/response body
2. **Cloud DLP + Dataflow**: real-time de-identification of log streams before storage
3. **Cloud Logging log processors** (beta): filter/redact fields at ingestion time
4. **Structured logging**: emit logs without PII fields in the application layer (best)

```bash
# Exclusion filter: drop logs with credit card numbers
gcloud logging sinks update _Default \
  --exclusion-filter='jsonPayload.message=~"[0-9]{16}"' \
  --exclusion-description="Drop potential PAN data"
```

---

## Gemini Cloud Assist for Logs

- Available in **Logs Explorer** UI
- Can: explain a log entry, generate a query from natural language, suggest next steps
- Example prompts: "Show me all errors in the payment service in the last hour", "Explain this stack trace"

---

## Log Costs Optimization

| Technique | Description |
|---|---|
| **Exclusion filters** | Drop high-volume low-value logs (e.g., health checks, 200s) before ingestion |
| **Log sampling** | VPC Flow Logs: reduce sampling rate (0.1 = 10%) |
| **Log-based metrics** | Extract signal from logs → use metrics instead of querying raw logs |
| **Routing to cheaper storage** | Route cold logs to GCS (cheaper than Logging storage) |
| **Retention tuning** | Reduce _Default retention to minimum needed |

```bash
# Exclude health check logs (save $$$)
gcloud logging sinks update _Default \
  --exclusion-filter='httpRequest.requestUrl="/healthz" AND httpRequest.status=200'
```

---

## Exam Tips

- Ops Agent = Fluent Bit (logs) + OTel Collector (metrics/traces) — single agent
- VPC Flow Logs = network-level telemetry; not app logs
- `_Required` bucket retention = 400 days, **cannot be changed**
- Log-based metrics = extract metrics from logs → use in alerting and dashboards
- Sinks route to: BigQuery (analysis), Pub/Sub (streaming), GCS (archival)
- **Exclusion filters** applied at ingestion → don't pay for unwanted logs
- Gemini Cloud Assist can explain log entries and generate LQL queries
- Data Access audit logs = **OFF by default** — must explicitly enable per service
