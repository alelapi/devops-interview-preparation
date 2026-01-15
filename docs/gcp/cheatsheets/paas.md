# PaaS & Serverless Essentials

## 🎯 Heavy Hitters (High Frequency)

### 1. **Serverless Compute Selection (Most Important!)**
The exam LOVES asking "which serverless service should you use?" - memorize this:

| Service | Use Case | Key Features | Scaling |
|---------|----------|--------------|---------|
| **Cloud Run** | Containerized stateless services | Any language, 80-15 min timeout, HTTP/gRPC | 0 to N instances |
| **Cloud Functions** | Event-driven, single-purpose functions | Node.js/Python/Go/Java, 1-60 min timeout | 0 to N instances |
| **App Engine Standard** | Web apps, traditional PaaS | Python/Java/Node.js/PHP/Ruby/Go, fast scaling | 0 to N instances |
| **App Engine Flexible** | Custom runtimes, long-running | Any language via Docker, slower scaling | Min 1 instance |
| **Cloud Run Jobs** | Batch jobs, scheduled tasks | Containers, run to completion | On-demand execution |

**Exam Clue Keywords:**
- "Containerized microservice, stateless" → **Cloud Run**
- "Event-driven, respond to Pub/Sub" → **Cloud Functions** (2nd gen) or **Cloud Run**
- "Web app, traditional PaaS, minimal config" → **App Engine Standard**
- "Need custom runtime, long-running requests" → **App Engine Flexible** or **Cloud Run**
- "Batch job, scheduled task" → **Cloud Run Jobs**

### 2. **Cloud Run vs Cloud Functions**
This comparison comes up frequently!

| Feature | Cloud Run | Cloud Functions |
|---------|-----------|-----------------|
| **Container** | Yes (any container) | No (source code only) |
| **Languages** | Any | Node.js, Python, Go, Java, .NET, Ruby, PHP |
| **Request timeout** | 60 min (max) | 60 min (2nd gen), 9 min (1st gen) |
| **Concurrency** | Up to 1000 requests/instance | 1 request/instance (1st gen), up to 1000 (2nd gen) |
| **Min instances** | 0 or more | 0 or more |
| **Cold start** | Slightly slower | Faster (1st gen) |
| **Use case** | Stateless APIs, microservices | Event-driven, simple functions |
| **Pricing** | Per request + CPU/memory time | Per invocation + CPU/memory time |

**Decision Guide:**
- **Need containers?** → Cloud Run
- **Simple event handler (Pub/Sub, Storage)?** → Cloud Functions (easier)
- **High concurrency needed?** → Cloud Run or Functions 2nd gen
- **Need HTTP API?** → Cloud Run (better routing, custom domains)
- **Want simplest option?** → Cloud Functions (less config)

**Exam Tip:** Cloud Functions 2nd gen is built on Cloud Run, so they're converging. For new projects, Cloud Run is increasingly preferred.

### 3. **App Engine Standard vs Flexible**
Know when to use each!

| Feature | Standard | Flexible |
|---------|----------|----------|
| **Scaling** | Automatic, 0 to N | Automatic, min 1 instance |
| **Startup time** | Milliseconds (fast) | Minutes (slow) |
| **Languages** | Python, Java, Node.js, PHP, Ruby, Go | Any language (via Docker) |
| **Sandbox** | Yes (restricted) | No (full VM access) |
| **SSH access** | No | Yes |
| **Cost** | Cheaper (can scale to 0) | More expensive (min 1 instance) |
| **Use case** | Traditional web apps, fast scaling | Custom runtimes, SSH needed |

**Exam Clue:**
- "Web app, minimal config, fast scaling to 0" → **App Engine Standard**
- "Need SSH access or custom runtime" → **App Engine Flexible**
- "Migrating from Heroku" → **App Engine** (both work, Standard is cheaper)

---

## ☁️ Cloud Run (The Star!)

### **Cloud Run Overview**
- **What**: Fully managed serverless platform for containers
- **Key Features**:
  - Any language (any container)
  - Automatic HTTPS endpoints
  - Scale to zero (pay only when running)
  - Built-in traffic splitting (blue/green deployments)
  - Integrates with Cloud Build, Artifact Registry

### **Cloud Run Services vs Jobs**
- **Services**: 
  - Handle HTTP/gRPC requests
  - Long-running, always available
  - Auto-scale based on traffic
  - **Use**: APIs, web services, webhooks
  
- **Jobs**: 
  - Run to completion (batch processing)
  - Execute tasks, exit
  - Can be scheduled (Cloud Scheduler)
  - **Use**: Data processing, ETL, scheduled tasks

### **Key Configuration**
- **CPU allocation**:
  - **During request only** (cheaper): CPU only during request
  - **Always allocated**: CPU always available (better for background work)
- **Concurrency**: Max requests per instance (default 80, max 1000)
- **Min/max instances**: Control scaling (0 for scale-to-zero)
- **Memory**: 128 MiB - 32 GiB
- **CPU**: 0.08 - 8 vCPU
- **Request timeout**: 5 min (default), up to 60 min

### **Cloud Run Networking**
- **Ingress**:
  - **All** (default): Accept traffic from internet
  - **Internal**: Only from VPC or Cloud Run services
  - **Internal and Cloud Load Balancing**: VPC + load balancer
- **Egress**:
  - **All**: Can reach internet + VPC
  - **Private ranges only**: Can reach VPC (via Serverless VPC Access)

### **Serverless VPC Access**
- **What**: Connector to access VPC resources (databases, VMs)
- **Use**: Cloud Run → VPC private resources
- **Cost**: Pay per connector + data processed
- **Setup**: Create connector → Configure in Cloud Run service
- **Exam Clue**: "Cloud Run needs to access Cloud SQL private IP" → Serverless VPC Access

### **Cloud Run Pricing**
- **Billed on**:
  - Request count
  - CPU time (allocated)
  - Memory time (allocated)
  - Network egress
- **Free tier**: 2M requests/month, 360,000 GB-seconds, 180,000 vCPU-seconds
- **Cost optimization**: 
  - Use CPU allocation "during request only"
  - Optimize container image size (faster cold starts)
  - Set appropriate min instances (avoid cold starts vs cost)

### **Exam Scenarios for Cloud Run**
```
Containerized microservice, stateless          → Cloud Run Services
Batch job, scheduled task                      → Cloud Run Jobs
Need to access Cloud SQL private IP            → Cloud Run + Serverless VPC Access
Deploy with zero downtime                      → Cloud Run (gradual rollout)
Need blue/green deployment                     → Cloud Run (traffic splitting)
Scale to zero to minimize costs                → Cloud Run (min instances = 0)
Need custom domain                             → Cloud Run (Cloud Load Balancing)
```

---

## ⚡ Cloud Functions

### **Cloud Functions Overview**
- **What**: Event-driven serverless functions (FaaS)
- **Generations**:
  - **1st gen**: Legacy, simpler, limited features
  - **2nd gen**: Built on Cloud Run, better features (recommended)

### **1st Gen vs 2nd Gen**
| Feature | 1st Gen | 2nd Gen |
|---------|---------|---------|
| **Timeout** | 9 min | 60 min |
| **Concurrency** | 1 request/instance | Up to 1000 requests/instance |
| **Instance size** | Up to 8 GB RAM | Up to 16 GB RAM, 4 vCPU |
| **Traffic splitting** | No | Yes |
| **Min instances** | 0 (can't set) | 0+ (configurable) |
| **Recommendations** | Legacy | **Use this for new projects** |

**Exam Tip:** Always choose 2nd gen for new projects unless specifically mentioned 1st gen.

### **Event Sources**
Cloud Functions can be triggered by:
- **HTTP/HTTPS**: Direct invocation via HTTP
- **Pub/Sub**: Messages published to topic
- **Cloud Storage**: Object create/delete/archive/metadata update
- **Firestore**: Document create/update/delete
- **Firebase**: Auth events, Remote Config, Test Lab
- **Cloud Logging**: Log entries (via Pub/Sub)
- **Cloud Scheduler**: Time-based triggers

### **Function Types**
- **HTTP Functions**: Respond to HTTP requests
  - Like a REST API endpoint
  - Can call directly via URL
  - **Use**: Webhooks, API endpoints
  
- **Event-driven Functions**: Respond to events (Pub/Sub, Storage, etc.)
  - Triggered automatically by event
  - Background processing
  - **Use**: Data processing pipelines, event handlers

### **Cold Starts**
- **What**: Delay when function instance starts from zero
- **Duration**: 
  - 1st gen: < 1 second (small functions)
  - 2nd gen: 1-3 seconds (more powerful)
- **Mitigation**:
  - Use min instances (keep warm)
  - Optimize dependencies (smaller = faster)
  - Use 1st gen for latency-critical (if < 9 min timeout OK)

### **Best Practices**
- **Keep functions small**: Single purpose, lightweight
- **Minimize dependencies**: Faster cold starts
- **Use min instances**: For production (avoid cold starts)
- **Handle idempotency**: Events can be delivered multiple times
- **Timeout appropriately**: Don't use max if not needed (saves cost)

### **Exam Scenarios for Cloud Functions**
```
Process images uploaded to Cloud Storage       → Cloud Functions (Storage trigger)
Respond to Pub/Sub message                     → Cloud Functions (Pub/Sub trigger)
Simple webhook, no container needed            → Cloud Functions (HTTP trigger)
Firestore document change handler              → Cloud Functions (Firestore trigger)
Scheduled task (every hour)                    → Cloud Functions + Cloud Scheduler
Need > 9 min timeout                           → Cloud Functions 2nd gen or Cloud Run
```

---

## 🚀 App Engine

### **App Engine Overview**
- **What**: Original GCP PaaS (Platform as a Service)
- **Key Features**:
  - Automatic scaling
  - Traffic splitting (A/B testing, blue/green)
  - Versions management
  - Integrated services (Task Queues, Memcache, Search)
  - **One app per project** (important limitation!)

### **Standard vs Flexible (Detailed)**

**App Engine Standard:**
- **Sandbox environment**: Restricted file system, network access
- **Languages**: Python 2/3, Java 8/11/17, Node.js, PHP 7/8, Ruby, Go
- **Scaling**: 
  - Automatic: 0 to N (can scale to zero)
  - Basic: Manual scaling, single instance
  - Manual: Fixed number of instances
- **Startup**: Milliseconds (fast cold starts)
- **Cost**: Cheaper (can scale to zero)
- **Instance classes**: F1, F2, F4, F4_1G (fixed sizes)
- **Pricing**: Instance hours + network

**App Engine Flexible:**
- **VM-based**: Full Compute Engine VMs (more flexible)
- **Languages**: Any language (via Dockerfile)
- **Scaling**: Automatic only, **min 1 instance always running**
- **Startup**: Minutes (VM boot time)
- **Cost**: More expensive (always running >= 1 instance)
- **SSH access**: Yes (can debug on VM)
- **Instance sizes**: Customizable (CPU, memory)
- **Pricing**: vCPU hours + memory hours + network

### **App Engine Components**
- **Application**: Top-level container (one per project)
- **Services**: Microservices within app (formerly "modules")
- **Versions**: Different versions of same service
- **Instances**: Running version of service

**Structure:**
```
Project
  └── Application (one)
        ├── Service 1 (default)
        │     ├── Version 1 (traffic: 90%)
        │     └── Version 2 (traffic: 10%)
        ├── Service 2 (api)
        │     └── Version 1 (traffic: 100%)
        └── Service 3 (admin)
              └── Version 1 (traffic: 100%)
```

### **Traffic Splitting**
- **Methods**:
  - **IP address**: Sticky sessions by IP
  - **Cookie**: Sticky sessions by cookie (GOOGAPPUID)
  - **Random**: Truly random distribution
- **Use cases**:
  - A/B testing
  - Canary deployments
  - Blue/green deployments
- **Gradual rollout**: Shift traffic slowly (e.g., 10% → 50% → 100%)

### **App Engine Scaling Types**
- **Automatic** (Standard & Flexible):
  - Scale based on load
  - Can scale to 0 (Standard only)
  - Configure min/max instances
  
- **Basic** (Standard only):
  - On-demand instances
  - Scale to 0 when no requests
  - No load balancing (requests queued)
  - **Use**: Dev/test environments
  
- **Manual** (Standard only):
  - Fixed number of instances
  - No auto-scaling
  - **Use**: Background workers, specific needs

### **App Engine vs Cloud Run**
| Feature | App Engine | Cloud Run |
|---------|------------|-----------|
| **Containers** | No (Standard), Yes (Flexible) | Yes (always) |
| **Multi-project** | No (one app/project) | Yes |
| **Traffic splitting** | Yes | Yes |
| **Cold start** | Fast (Standard), Slow (Flexible) | Medium |
| **Scale to zero** | Yes (Standard only) | Yes |
| **Versions** | Built-in versioning | Manual via revision |
| **Legacy** | Mature, lots of legacy features | Modern, simpler |
| **Recommendation** | Legacy apps, existing App Engine | **New apps** |

**Exam Tip:** For new applications, Cloud Run is usually preferred. App Engine is good for legacy migrations or if you need built-in traffic splitting/versioning.

### **Exam Scenarios for App Engine**
```
Traditional web app, minimal config             → App Engine Standard
Need SSH access for debugging                   → App Engine Flexible
Gradual rollout with traffic splitting          → App Engine (or Cloud Run)
Already using App Engine, need to continue      → App Engine
Microservices in single app                     → App Engine (multiple services)
Need custom runtime with auto-scaling           → App Engine Flexible
```

---

## 🔄 Event-Driven Architecture

### **Cloud Pub/Sub**
- **What**: Asynchronous messaging service (event bus)
- **Use cases**:
  - Decouple microservices
  - Event distribution
  - Streaming analytics pipelines
  - Parallel processing
- **Key concepts**:
  - **Topic**: Named resource for messages
  - **Subscription**: Named resource receiving messages
  - **Publisher**: Sends messages to topic
  - **Subscriber**: Receives messages from subscription

**Subscription Types:**
- **Pull**: Subscriber requests messages (polling)
  - More control over rate
  - Batch processing
  
- **Push**: Pub/Sub sends messages to HTTPS endpoint
  - Serverless (Cloud Functions, Cloud Run)
  - Lower latency
  - **Requires**: Public HTTPS endpoint

**Message Guarantees:**
- **At-least-once delivery**: May receive duplicate messages
- **Best-effort ordering**: Messages may arrive out of order
- **Ordering keys**: Enable ordering within same key

**Exam Scenarios:**
```
Decouple microservices                         → Pub/Sub
Fan-out event to multiple consumers            → Pub/Sub (multiple subscriptions)
Trigger Cloud Function on event                → Pub/Sub + Cloud Functions
Async processing of web requests               → Pub/Sub
Stream data to BigQuery                        → Pub/Sub + Dataflow
```

### **Cloud Scheduler**
- **What**: Fully managed cron job service
- **Targets**:
  - HTTP/HTTPS endpoints
  - Pub/Sub topics
  - App Engine HTTP targets
- **Schedule**: Cron syntax or App Engine cron.yaml
- **Use cases**:
  - Run Cloud Functions on schedule
  - Trigger Cloud Run Jobs
  - Send periodic Pub/Sub messages
  - Scheduled data exports/backups

**Exam Scenarios:**
```
Run task every hour                            → Cloud Scheduler + Cloud Functions/Run
Scheduled ETL job                              → Cloud Scheduler + Cloud Run Jobs
Daily backup                                   → Cloud Scheduler + Cloud Functions
Periodic report generation                     → Cloud Scheduler → Pub/Sub → Cloud Run
```

### **Eventarc**
- **What**: Event-driven orchestration (unified eventing)
- **Purpose**: Route events from 90+ Google sources to serverless
- **Sources**:
  - Cloud Storage
  - Cloud Audit Logs
  - Pub/Sub
  - Workflows
  - Custom sources (via Pub/Sub)
- **Targets**:
  - Cloud Run
  - Cloud Functions 2nd gen
  - Workflows
- **Benefits**: 
  - Unified event routing
  - Filter events (don't process everything)
  - CloudEvents standard

**Eventarc vs Pub/Sub:**
- **Eventarc**: High-level, easy filtering, many sources
- **Pub/Sub**: Lower-level, more control, custom routing

**Exam Clue:** "Route events from Cloud Storage/Audit Logs to Cloud Run" → Eventarc

### **Cloud Tasks**
- **What**: Managed task queue service (guaranteed execution)
- **Use cases**:
  - Guaranteed task execution
  - Rate-limited API calls
  - Async processing with retries
- **vs Pub/Sub**:
  - **Cloud Tasks**: Task queues, explicit task execution, rate limiting
  - **Pub/Sub**: Event bus, at-least-once delivery, fan-out
- **Target**: HTTP endpoints (App Engine, Cloud Functions, Cloud Run)

**When to use Cloud Tasks:**
- Need guaranteed execution (explicit deduplication)
- Rate limiting required (don't overwhelm target)
- Task scheduling (delay until specific time)
- Explicit task acknowledgment

**Exam Scenarios:**
```
Rate-limit API calls to external service       → Cloud Tasks
Guaranteed task execution with retries         → Cloud Tasks
Fan-out events to multiple consumers           → Pub/Sub (not Tasks)
Schedule task for future execution             → Cloud Tasks
```

---

## 🔗 API Management

### **Cloud Endpoints**
- **What**: API management for REST/gRPC APIs
- **Features**:
  - API key validation
  - Monitoring & logging
  - Rate limiting
  - Authentication (JWT, Auth0, Firebase)
- **Supported**: App Engine, Cloud Run, GKE, Compute Engine
- **Configuration**: OpenAPI spec or gRPC service definition
- **Free**: No additional cost (just infrastructure)

**Use cases:**
- Secure your APIs
- Monitor API usage
- Rate limit clients
- Validate API keys

### **Apigee**
- **What**: Enterprise API management platform
- **Features**:
  - Everything in Cloud Endpoints +
  - API analytics & monetization
  - Developer portal
  - API versioning & lifecycle
  - Complex policies & transformations
  - Hybrid/multi-cloud deployment
- **Cost**: Expensive (enterprise pricing)

**Cloud Endpoints vs Apigee:**
- **Cloud Endpoints**: Simple, free, basic API management
- **Apigee**: Enterprise, expensive, full API lifecycle management

**Exam Clue:**
- "Simple API management, rate limiting" → **Cloud Endpoints**
- "Enterprise API platform, monetization, developer portal" → **Apigee**

### **API Gateway**
- **What**: Fully managed API gateway (serverless)
- **Purpose**: Secure & manage APIs for serverless backends
- **Backends**: Cloud Functions, Cloud Run
- **Features**: 
  - Authentication (API keys, JWT, Service accounts)
  - Rate limiting & quotas
  - OpenAPI spec
  - Cloud Logging & Monitoring
- **vs Cloud Endpoints**: API Gateway is newer, serverless-focused

**Exam Tip:** API Gateway is increasingly replacing Cloud Endpoints for serverless APIs.

---

## 🗂️ Workflow Orchestration

### **Cloud Workflows**
- **What**: Orchestrate services (serverless, APIs, GCP services)
- **Use cases**:
  - Multi-step workflows
  - Service orchestration
  - Error handling & retries
  - Human approval steps
- **Definition**: YAML or JSON
- **Integration**: Cloud Functions, Cloud Run, HTTP APIs, GCP services
- **Features**:
  - Built-in retries & error handling
  - Parallel execution
  - Conditional logic
  - Wait states (sleep)

**Example Workflow:**
```yaml
- init:
    assign:
      - projectId: ${sys.get_env("GOOGLE_CLOUD_PROJECT_ID")}
- callCloudRun:
    call: http.get
    args:
      url: https://myservice.run.app
    result: apiResponse
- returnOutput:
    return: ${apiResponse.body}
```

**When to use Workflows:**
- Orchestrate multiple Cloud Functions/Cloud Run services
- Complex multi-step processes
- Need human approval in pipeline
- Error handling & retries required

**Exam Scenarios:**
```
Orchestrate multiple microservices             → Cloud Workflows
Multi-step data processing pipeline            → Cloud Workflows (or Dataflow)
Need human approval in workflow                → Cloud Workflows (callbacks)
Chain Cloud Functions together                 → Cloud Workflows
```

---

## 🎓 Exam Decision Trees

### **Serverless Compute Selection**
```
Containerized stateless service                → Cloud Run Services
Event-driven function (Pub/Sub, Storage)       → Cloud Functions 2nd gen
Batch job, scheduled task                      → Cloud Run Jobs
Traditional web app, minimal config            → App Engine Standard
Need SSH access, custom runtime                → App Engine Flexible
Need any language, containerized               → Cloud Run
Want simplest event handler                    → Cloud Functions
```

### **Cloud Run vs Cloud Functions**
```
Need containers                                → Cloud Run
Want simplest setup                            → Cloud Functions
Need HTTP API with routing                     → Cloud Run
Simple event handler (Storage, Pub/Sub)        → Cloud Functions (easier)
Need high concurrency (1000+ req/instance)     → Cloud Run or Functions 2nd gen
Need < 1 sec cold start                        → Cloud Functions 1st gen
Need > 9 min timeout                           → Cloud Functions 2nd gen or Cloud Run
```

### **App Engine Standard vs Flexible**
```
Fast scaling, scale to 0                       → App Engine Standard
Need SSH access for debugging                  → App Engine Flexible
Custom runtime (not standard languages)        → App Engine Flexible
Cost-effective, fast cold starts               → App Engine Standard
Need to install system packages                → App Engine Flexible
```

### **Event Routing**
```
Decouple microservices                         → Pub/Sub
Route events from Cloud Storage/Audit Logs     → Eventarc
Scheduled tasks                                → Cloud Scheduler
Guaranteed task execution with rate limiting   → Cloud Tasks
Fan-out to multiple consumers                  → Pub/Sub
Orchestrate multiple services                  → Cloud Workflows
```

### **API Management**
```
Simple API management, free                    → Cloud Endpoints
Enterprise API platform, monetization          → Apigee
Serverless API gateway                         → API Gateway
Just need authentication on API                → Cloud Endpoints or API Gateway
```

---

## ⚡ Quick Reminders

### **Request Timeouts**
- **Cloud Run**: Up to 60 min
- **Cloud Functions 1st gen**: Up to 9 min
- **Cloud Functions 2nd gen**: Up to 60 min
- **App Engine Standard**: 10 min (automatic), 24h (manual/basic)
- **App Engine Flexible**: 60 min

### **Concurrency**
- **Cloud Run**: Up to 1000 requests/instance
- **Cloud Functions 1st gen**: 1 request/instance
- **Cloud Functions 2nd gen**: Up to 1000 requests/instance
- **App Engine**: Multiple requests/instance

### **Cold Start Performance**
- **Fastest**: Cloud Functions 1st gen (< 1s)
- **Fast**: App Engine Standard (< 1s)
- **Medium**: Cloud Run (1-3s), Cloud Functions 2nd gen
- **Slow**: App Engine Flexible (minutes)

### **Scale to Zero**
- **Yes**: Cloud Run, Cloud Functions, App Engine Standard
- **No**: App Engine Flexible (min 1 instance)

### **Common Gotchas**
- **App Engine**: One app per project (major limitation!)
- **Cloud Functions 1st gen**: Only 1 concurrent request per instance
- **App Engine Flexible**: Always min 1 instance (can't scale to zero)
- **Cloud Run**: CPU allocated only during request (by default)
- **Pub/Sub**: At-least-once delivery (handle duplicates!)
- **Cloud Functions**: Not for long-running jobs (use Cloud Run Jobs)

---

## 🔍 Troubleshooting Quick Checks

**Cloud Run service not accessible:**
- ✅ Check ingress settings (all/internal/internal+LB)
- ✅ Verify IAM permissions (Cloud Run Invoker role)
- ✅ Check if service is deployed (healthy revisions)
- ✅ Review service logs (Cloud Logging)
- ✅ Verify domain mapping (if using custom domain)

**Cloud Function not triggering:**
- ✅ Check trigger configuration (Pub/Sub topic, Storage bucket)
- ✅ Verify IAM permissions on trigger source
- ✅ Review function logs (errors during deployment?)
- ✅ Check event format matches function signature
- ✅ Verify quota limits not exceeded

**App Engine high latency:**
- ✅ Check instance scaling (might need more instances)
- ✅ Review application logs (slow queries?)
- ✅ Check instance class (F1 vs F2 vs F4)
- ✅ Consider warmup requests (reduce cold starts)
- ✅ Review trace data (Cloud Trace)

**Pub/Sub messages not being processed:**
- ✅ Check subscription exists and is active
- ✅ Verify subscriber is pulling/receiving messages
- ✅ Review ack deadline (too short?)
- ✅ Check for unacknowledged messages building up
- ✅ Verify subscriber has processing capacity

**Cloud Run can't access Cloud SQL:**
- ✅ Check Cloud SQL instance has private IP
- ✅ Verify Serverless VPC Access connector configured
- ✅ Check VPC connector is in same region
- ✅ Review IAM permissions (Cloud SQL Client role)
- ✅ Verify connection string is correct

---

## 📚 Pro Tips for Exam

1. **Default choice for new apps**: Cloud Run (most flexible, modern)
2. **Cloud Functions**: Choose when you need simple event handling (Pub/Sub, Storage)
3. **App Engine**: Legacy, but good for traditional web apps with minimal config
4. **Scale to zero**: Cloud Run, Cloud Functions, App Engine Standard (not Flexible!)
5. **Concurrency**: Cloud Run and Functions 2nd gen support high concurrency (1000+)
6. **Cold starts**: Functions 1st gen is fastest, but limited to 9 min timeout
7. **One app per project**: App Engine limitation (Cloud Run doesn't have this!)
8. **Event routing**: Pub/Sub for fan-out, Eventarc for Cloud Storage/Audit Logs
9. **Orchestration**: Cloud Workflows for multi-step processes
10. **API management**: Cloud Endpoints (simple/free) vs Apigee (enterprise/expensive)

---

## 🎯 Memorization Shortcuts

**Containerized microservice:**
Cloud Run Services

**Event-driven function (Pub/Sub, Storage):**
Cloud Functions 2nd gen

**Batch job, scheduled task:**
Cloud Run Jobs

**Traditional web app, minimal config:**
App Engine Standard

**Need SSH access:**
App Engine Flexible

**Decouple microservices:**
Pub/Sub

**Scheduled tasks:**
Cloud Scheduler

**Orchestrate multiple services:**
Cloud Workflows

**Guaranteed task execution:**
Cloud Tasks

**Simple API management:**
Cloud Endpoints

**Enterprise API platform:**
Apigee

**Scale to zero:**
Cloud Run, Cloud Functions, App Engine Standard

---

## 🧪 Scenario-Based Examples

**Scenario 1**: "Startup needs to deploy containerized microservices that scale automatically, including scale to zero."
- **Answer**: Cloud Run Services (containers, auto-scaling, scale-to-zero)

**Scenario 2**: "Process images uploaded to Cloud Storage, resize them, and save back to Storage."
- **Answer**: Cloud Functions 2nd gen with Cloud Storage trigger

**Scenario 3**: "Existing Python web app needs to be deployed with minimal configuration changes."
- **Answer**: App Engine Standard (traditional PaaS, Python supported)

**Scenario 4**: "Need to run ETL job every night at 2 AM, job takes 20 minutes to complete."
- **Answer**: Cloud Scheduler + Cloud Run Jobs (scheduled batch processing)

**Scenario 5**: "Microservices need to communicate asynchronously, fan-out events to multiple consumers."
- **Answer**: Pub/Sub (event bus, multiple subscriptions)

**Scenario 6**: "Multi-step data processing pipeline: validate → transform → load, with error handling."
- **Answer**: Cloud Workflows (orchestrate steps, built-in error handling)

**Scenario 7**: "API needs rate limiting to prevent abuse, basic authentication with API keys."
- **Answer**: Cloud Run + Cloud Endpoints (or API Gateway)

**Scenario 8**: "Cloud Run service needs to connect to Cloud SQL instance using private IP."
- **Answer**: Cloud Run + Serverless VPC Access connector

**Scenario 9**: "Need to process Pub/Sub messages with guaranteed execution and retry logic."
- **Answer**: Cloud Functions 2nd gen with Pub/Sub trigger (automatic retries)

**Scenario 10**: "Deploy containerized app with gradual rollout (10% → 50% → 100% traffic)."
- **Answer**: Cloud Run with gradual rollout (traffic splitting between revisions)

**Scenario 11**: "Call external API with rate limiting (max 100 requests/min), guarantee all tasks execute."
- **Answer**: Cloud Tasks (rate limiting + guaranteed execution)

**Scenario 12**: "React to Cloud Storage object creation events and trigger Cloud Run service."
- **Answer**: Eventarc (route Storage events to Cloud Run)

---

## 💡 Advanced Patterns

### **Serverless Data Pipeline**
```
Cloud Storage (data arrives)
  → Eventarc (detect new file)
  → Cloud Run (validate & transform)
  → Pub/Sub (async processing)
  → Cloud Functions (load to BigQuery)
  → BigQuery (analytics)
```

### **Microservices Architecture**
```
Client → Cloud Load Balancer
  → Cloud Run (API Gateway service)
  → Pub/Sub (event bus)
  → Cloud Run Services (microservices)
  → Cloud SQL / Firestore (data)
```

### **Event-Driven Processing**
```
Source Event (Storage, Pub/Sub, Firestore)
  → Cloud Functions 2nd gen (handler)
  → Process data
  → Pub/Sub (fan-out)
  → Multiple Cloud Functions (parallel processing)
```

### **Scheduled Batch Processing**
```
Cloud Scheduler (cron)
  → Pub/Sub (trigger)
  → Cloud Run Jobs (batch processing)
  → Cloud Storage (results)
  → Cloud Functions (notification)
```

### **Multi-Step Workflow**
```
Cloud Workflows (orchestrator)
  ├── Step 1: Cloud Run (validate)
  ├── Step 2: Cloud Functions (transform)
  ├── Step 3: External API call
  └── Step 4: Cloud Run (load)
```

---

## 🔒 Security Best Practices

### **Authentication & Authorization**
- **Cloud Run/Functions**: 
  - Use IAM for service-to-service auth
  - Cloud Run Invoker role for invoking
  - Don't make public unless necessary
  
- **App Engine**: 
  - Use IAM, Firebase Auth, or Identity Platform
  - Configure `app.yaml` for auth requirements
  
- **API Gateway/Endpoints**: 
  - API keys for external clients
  - JWT for user authentication
  - Service accounts for service-to-service

### **Network Security**
- **Private services**: Set ingress to "internal only"
- **VPC Access**: Use Serverless VPC Access for private resources
- **Load Balancer**: Use Cloud Load Balancing for DDoS protection
- **Egress**: Restrict egress to specific VPC ranges if needed

### **Secrets Management**
- **Secret Manager**: Store API keys, passwords, certificates
- **Never**: Store secrets in environment variables (visible in console!)
- **Access**: Mount secrets as environment variables at runtime
- **IAM**: Grant Secret Manager Secret Accessor role

### **Exam Tip**: Always use Secret Manager for secrets, not environment variables!

---

This comprehensive guide covers all the PaaS and serverless services you need to know for the GCP Professional Cloud Architect exam. Focus on understanding when to use each service (Cloud Run vs Functions vs App Engine), event-driven patterns (Pub/Sub, Eventarc, Cloud Scheduler), and orchestration (Workflows). Good luck!