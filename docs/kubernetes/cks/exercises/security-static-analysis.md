# Static Security Analysis

## Dockerfile Security Analysis

### Key Security Issues to Look For

#### 1. **Running as Root User**
```dockerfile
# ❌ BAD - Running as root
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y nginx
CMD ["nginx", "-g", "daemon off;"]

# ✅ GOOD - Non-root user
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y nginx
RUN useradd -r -s /bin/false nginx-user
USER nginx-user
CMD ["nginx", "-g", "daemon off;"]
```

#### 2. **Secrets in Image Layers**
```dockerfile
# ❌ BAD - Hardcoded secrets
FROM node:14
ENV API_KEY=sk-1234567890abcdef
ENV DATABASE_PASSWORD=super-secret-password
COPY . /app

# ✅ GOOD - No secrets in Dockerfile
FROM node:14
# Secrets should be passed at runtime via K8s secrets
COPY . /app
```

#### 3. **Unnecessary Packages and Attack Surface**
```dockerfile
# ❌ BAD - Full OS with unnecessary packages
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y \
    curl wget git vim nano ssh openssh-server

# ✅ GOOD - Minimal base image
FROM node:14-alpine
# Only install what's needed
RUN apk add --no-cache dumb-init
```

#### 4. **Improper File Permissions**
```dockerfile
# ❌ BAD - World-writable files
FROM alpine
COPY --chmod=777 app.sh /usr/local/bin/
COPY --chmod=666 config.json /etc/

# ✅ GOOD - Restrictive permissions
FROM alpine
COPY --chmod=755 app.sh /usr/local/bin/
COPY --chmod=644 config.json /etc/
```

#### 5. **Using Latest Tags**
```dockerfile
# ❌ BAD - Unpredictable base image
FROM node:latest
FROM ubuntu:latest

# ✅ GOOD - Specific versions
FROM node:14.21.3-alpine
FROM ubuntu:20.04
```

---

## Kubernetes Manifest Security Analysis

### Key Security Issues to Look For

#### 1. **Missing Security Context**
```yaml
# ❌ BAD - No security context
apiVersion: apps/v1
kind: Deployment
metadata:
  name: insecure-app
spec:
  template:
    spec:
      containers:
      - name: app
        image: nginx:1.20

# ✅ GOOD - Proper security context
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        runAsGroup: 1000
        fsGroup: 1000
      containers:
      - name: app
        image: nginx:1.20
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          capabilities:
            drop:
            - ALL
```

#### 2. **Privileged Containers**
```yaml
# ❌ BAD - Privileged container
containers:
- name: dangerous-app
  image: app:1.0
  securityContext:
    privileged: true

# ✅ GOOD - Non-privileged with specific capabilities
containers:
- name: safe-app
  image: app:1.0
  securityContext:
    privileged: false
    capabilities:
      add:
      - NET_BIND_SERVICE
      drop:
      - ALL
```

#### 3. **Missing Resource Limits**
```yaml
# ❌ BAD - No resource limits
containers:
- name: resource-hog
  image: app:1.0

# ✅ GOOD - Resource limits set
containers:
- name: limited-app
  image: app:1.0
  resources:
    limits:
      memory: "512Mi"
      cpu: "500m"
    requests:
      memory: "256Mi"
      cpu: "250m"
```

#### 4. **Secrets in Environment Variables**
```yaml
# ❌ BAD - Secrets in plain text env vars
containers:
- name: app
  image: myapp:1.0
  env:
  - name: DB_PASSWORD
    value: "super-secret-password"
  - name: API_KEY
    value: "sk-1234567890"

# ✅ GOOD - Using Kubernetes secrets
containers:
- name: app
  image: myapp:1.0
  env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
  - name: API_KEY
    valueFrom:
      secretKeyRef:
        name: api-secret
        key: api-key
```

#### 5. **Network Policies Missing**
```yaml
# ❌ BAD - No network restrictions
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
# No NetworkPolicy defined

# ✅ GOOD - With NetworkPolicy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-app-netpol
spec:
  podSelector:
    matchLabels:
      app: web-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

---

## Tools for Static Analysis

### 1. **Trivy (Recommended for exams)**
```bash
# Scan Dockerfile
trivy config Dockerfile

# Scan Kubernetes manifests
trivy config k8s-manifests/

# Scan specific manifest
trivy config deployment.yaml
```

### 2. **Checkov**
```bash
# Install
pip install checkov

# Scan Dockerfile
checkov -f Dockerfile

# Scan Kubernetes manifests
checkov -d k8s-manifests/
```

### 3. **Kubesec**
```bash
# Online tool
curl -sSX POST --data-binary @deployment.yaml https://v2.kubesec.io/scan

# Local installation
kubesec scan deployment.yaml
```

### 4. **Kube-score**
```bash
# Install
kubectl krew install score

# Analyze manifest
kubectl score deployment.yaml
```

---

## Common Exam Questions

### Question Type 1: "Identify Security Issues"
**Prompt**: *"Analyze the following Dockerfile and identify at least 3 security vulnerabilities"*

**Approach**:
1. Look for root user usage
2. Check for hardcoded secrets
3. Verify base image tags
4. Check file permissions
5. Look for unnecessary packages

### Question Type 2: "Fix Security Issues"
**Prompt**: *"Fix the security issues in the following Kubernetes deployment manifest"*

**Approach**:
1. Add securityContext
2. Set resource limits
3. Use secrets instead of plain env vars
4. Ensure non-root execution
5. Add readiness/liveness probes

### Question Type 3: "Use Tool for Analysis"
**Prompt**: *"Use trivy to scan the provided manifest and fix the HIGH severity issues"*

**Approach**:
1. Run trivy scan
2. Identify HIGH severity findings
3. Apply fixes systematically
4. Re-scan to verify

---

## Step-by-Step Solutions

### Scenario 1: Dockerfile Analysis

**Given Dockerfile**:
```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get install -y curl wget git
ENV SECRET_KEY=abc123
COPY app.py /app/
RUN chmod 777 /app/app.py
CMD ["python", "/app/app.py"]
```

**Step 1**: Identify Issues
```bash
# Run trivy scan
trivy config Dockerfile
```

**Step 2**: Issues Found
- Using `latest` tag (unpredictable)
- Running as root user
- Hardcoded secret in ENV
- Overly permissive file permissions (777)
- Unnecessary packages installed

**Step 3**: Fixed Dockerfile
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y python3 && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
RUN useradd -r -s /bin/false appuser
COPY --chown=appuser:appuser --chmod=644 app.py /app/
USER appuser
CMD ["python3", "/app/app.py"]
```

### Scenario 2: Kubernetes Manifest Analysis

**Given Deployment**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web
        image: nginx:latest
        env:
        - name: DB_PASSWORD
          value: "secretpassword"
```

**Step 1**: Scan with Tools
```bash
trivy config deployment.yaml
kubesec scan deployment.yaml
```

**Step 2**: Issues Identified
- No security context
- Using latest tag
- Secret in plain text env var
- No resource limits
- Missing health checks

**Step 3**: Fixed Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
      containers:
      - name: web
        image: nginx:1.21.6-alpine
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          capabilities:
            drop:
            - ALL
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        resources:
          limits:
            memory: "512Mi"
            cpu: "500m"
          requests:
            memory: "256Mi"
            cpu: "250m"
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 30
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
        volumeMounts:
        - name: tmp-volume
          mountPath: /tmp
        - name: var-cache
          mountPath: /var/cache/nginx
      volumes:
      - name: tmp-volume
        emptyDir: {}
      - name: var-cache
        emptyDir: {}
```

---

## Remediation Examples

### Security Context Best Practices
```yaml
# Complete security context example
securityContext:
  # Pod-level
  runAsNonRoot: true
  runAsUser: 1000
  runAsGroup: 1000
  fsGroup: 1000
  seccompProfile:
    type: RuntimeDefault

containers:
- name: app
  securityContext:
    # Container-level
    allowPrivilegeEscalation: false
    readOnlyRootFilesystem: true
    runAsNonRoot: true
    runAsUser: 1000
    capabilities:
      drop:
      - ALL
      add:
      - NET_BIND_SERVICE  # Only if needed
```

### Resource Management
```yaml
resources:
  limits:
    memory: "512Mi"
    cpu: "500m"
    ephemeral-storage: "1Gi"
  requests:
    memory: "256Mi"
    cpu: "250m"
    ephemeral-storage: "500Mi"
```

### Secrets Management
```yaml
# Create secret first
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  api-key: <base64-encoded-value>
  db-password: <base64-encoded-value>

# Reference in deployment
env:
- name: API_KEY
  valueFrom:
    secretKeyRef:
      name: app-secrets
      key: api-key
```

---

## Exam Tips

### Time Management
1. **Quick Scan First**: Use automated tools to identify obvious issues
2. **Prioritize**: Focus on HIGH/CRITICAL severity issues first
3. **Systematic Approach**: Check each category methodically

### Common Commands to Remember
```bash
# Trivy scanning
trivy config .
trivy image nginx:latest

# Kubesec analysis
kubesec scan pod.yaml

# Kubectl dry-run for validation
kubectl apply --dry-run=client -f manifest.yaml

# Check security context
kubectl get pod -o jsonpath='{.spec.securityContext}'
```

### Checklist for Quick Review
- [ ] Non-root user execution
- [ ] No hardcoded secrets
- [ ] Specific image tags (not latest)
- [ ] Resource limits set
- [ ] Security context configured
- [ ] Minimal attack surface
- [ ] Health checks present
- [ ] Network policies defined

---

## Practice Exercises

### Exercise 1: Fix This Dockerfile
```dockerfile
FROM node:latest
RUN apt-get update && apt-get install -y vim curl wget git
ENV JWT_SECRET=mysecretkey123
COPY . /app
WORKDIR /app
RUN npm install
EXPOSE 3000
CMD ["npm", "start"]
```

### Exercise 2: Secure This Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
spec:
  template:
    spec:
      containers:
      - name: api
        image: myapi:latest
        env:
        - name: DATABASE_URL
          value: "postgres://user:password@db:5432/mydb"
        ports:
        - containerPort: 8080
```

*Solutions available upon request or can be worked through using the guidelines above.*
