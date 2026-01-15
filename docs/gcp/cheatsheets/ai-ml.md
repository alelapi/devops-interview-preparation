# AI/ML Essentials

## 🎯 Heavy Hitters (High Frequency)

### 1. **Vertex AI Agent Builder**

- **What**: Low-code platform to build GenAI chatbots/conversational AI
- **When**: Customer support bots, RAG applications, enterprise chatbots
- **Key Feature**: Connects to data sources (websites, docs) automatically
- **Exam Clue**: "Build a chatbot quickly without coding" → Agent Builder

### 2. **Vector Search**

- **What**: Find semantically similar content (not just keyword matching)
- **Use Cases**: 
  - Semantic search ("find similar products")
  - Image similarity ("find visually similar images")
  - Recommendation engines
- **Two Options**:
  - **Vertex AI Vector Search**: Standalone, managed service
  - **AlloyDB + pgvector**: If data already in AlloyDB/PostgreSQL
- **Exam Clue**: "semantic", "similar", "embeddings" → Vector Search

### 3. **Securing AI**

- **Model Armor**: Filters toxic/harmful AI outputs before reaching users
- **VPC Service Controls**: Creates security perimeter around Vertex AI to prevent data exfiltration
- **Exam Clue**: "Prevent training data leakage" → VPC-SC

---

## 🧠 Core Vertex AI Concepts

### **Model Garden**

- **What**: Marketplace/"App Store" for AI models
- **Options**: Google's Gemini, OSS (Llama, Claude), third-party models
- **When**: Client needs to compare/choose between different model providers
- **Exam Clue**: "Evaluate multiple models" → Model Garden

### **Gemini Cloud Assist**

- **What**: AI-powered operations assistant
- **Use Cases**:
  - GKE cost optimization recommendations
  - Network troubleshooting
  - Quick infrastructure insights
- **Exam Clue**: "Quickly optimize/troubleshoot infrastructure" → Cloud Assist

---

## 📊 Data-to-AI Workflow

### **BigQuery ML**

- **When**: Data already in BigQuery + simple ML (regression/classification)
- **Benefit**: No data movement, SQL-based ML
- **Exam Clue**: "Data in BQ, simple prediction" → BQML
- **Not For**: Complex deep learning, image/video models

### **Vertex AI Pipelines**

- **What**: MLOps orchestration (automated training/retraining workflows)
- **When**: Need repeatable, production ML pipelines with CI/CD
- **Components**: Kubeflow Pipelines or TFX
- **Exam Clue**: "Automate model retraining", "MLOps" → Pipelines

---

## 🔒 AI Security (Critical for PCA)

### **VPC Service Controls (AI Context)**

- **What**: Security perimeter preventing data from leaving your environment
- **Use With**: Vertex AI, BigQuery, Cloud Storage
- **Exam Clue**: "Prevent data exfiltration during training" → VPC-SC
- **Setup**: Create perimeter → Add projects → Restrict egress

### **Sensitive Data Protection (DLP)**

- **What**: Identify and redact PII/sensitive data
- **Use Cases**:
  - Redact names/SSNs before model training
  - De-identify healthcare data (HIPAA)
  - Scan datasets for PII
- **Methods**: Masking, tokenization, redaction
- **Exam Clue**: "Remove PII before training" → DLP API

---

## 🎓 Exam Decision Tree

```
Question mentions "chatbot" → Agent Builder
Question mentions "semantic/similar" → Vector Search
Question mentions "data leakage prevention" → VPC Service Controls
Question mentions "PII removal" → DLP
Data in BigQuery + simple ML → BigQuery ML
Need automated retraining → Vertex AI Pipelines
Compare multiple models → Model Garden
Quick infra optimization → Gemini Cloud Assist
```

---

## ⚡ Quick Reminders

- **Vertex AI** = Unified ML platform (training, deployment, monitoring)
- **Embeddings** = Vector representations → Use Vector Search
- **RAG** = Retrieval Augmented Generation → Agent Builder + Vector Search
- **MLOps** = Pipelines + Monitoring + Continuous training
- **Security Layers**: VPC-SC (network) + DLP (data) + Model Armor (output)