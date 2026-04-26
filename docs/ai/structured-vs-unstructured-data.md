# Structured vs Unstructured Data

## 1. Why Data Types Matter in AI

Every AI model — whether traditional ML or a generative model — **learns from data**. The type of data you have determines which models you can use, how you prepare the data, and which Google Cloud tools are most appropriate.

The GAIL exam tests your ability to recognize data types and match them to the right AI approach and tooling.

---

## 2. Structured Data

**Structured data** is data that is **organized in a predefined format**, typically rows and columns, with a clear schema. Every entry follows the same structure.

### Characteristics
- Stored in relational databases or spreadsheets
- Has defined data types (integer, float, string, date)
- Easily searchable and queryable with SQL
- Each field has a clear, consistent meaning

### Examples

| Customer ID | Age | City | Purchase Amount | Churned |
|---|---|---|---|---|
| 1001 | 34 | Milan | €120.50 | No |
| 1002 | 28 | Rome | €45.00 | Yes |
| 1003 | 52 | Turin | €310.00 | No |

Other examples:
- Financial transactions (amount, date, merchant, account ID)
- Sensor readings (temperature, pressure, timestamp)
- E-commerce orders (product ID, quantity, price, shipping address)
- Survey responses with predefined choices

### Use in AI
Structured data is the traditional home of **classical machine learning**:
- Predict customer churn (classification)
- Forecast sales revenue (regression)
- Detect fraudulent transactions (anomaly detection)
- Segment customers by purchase behavior (clustering)

**Google Cloud tools:**
- **BigQuery ML** — run ML models directly on structured data using SQL
- **Vertex AI AutoML Tables** — automatically build ML models from tabular data
- **Vertex AI Tabular** — custom training on structured datasets

---

## 3. Unstructured Data

**Unstructured data** is data that **does not follow a predefined format or schema**. It cannot be easily organized into rows and columns.

### Characteristics
- No fixed schema or format
- Requires preprocessing to extract meaning
- Much more abundant than structured data (~80-90% of all data generated)
- Rich in information but harder to query directly

### Examples by type

| Type | Examples |
|---|---|
| **Text** | Emails, documents, articles, chat logs, social media posts, contracts |
| **Images** | Photos, medical scans, satellite imagery, product images |
| **Audio** | Call recordings, podcasts, voice messages |
| **Video** | Security footage, training videos, customer demos |
| **Code** | Source files, scripts, notebooks |

### Use in AI
Unstructured data is where **deep learning and generative AI** excel:
- Summarize customer emails (text)
- Detect tumors in X-rays (images)
- Transcribe call center recordings (audio)
- Generate product descriptions from images (multimodal)
- Classify support tickets by topic (text)

**Google Cloud tools:**
- **Vertex AI** — training and deploying models on unstructured data
- **Document AI** — extract structured information from unstructured documents
- **Vision AI** — image classification, object detection, OCR
- **Speech-to-Text API** — transcribe audio to text
- **Natural Language API** — sentiment analysis, entity extraction from text
- **Video Intelligence API** — analyze and index video content
- **Gemini** — natively handles text, images, audio, video in one model

---

## 4. Semi-Structured Data

A third category worth knowing — data that has **some organizational structure but doesn't fit neatly into rows and columns**.

### Examples
- JSON files
- XML documents
- Log files
- HTML pages
- Emails with metadata headers + free-text body

### Use in AI
Often requires parsing/preprocessing before use. For example:
- Extracting fields from JSON API responses
- Parsing log files to identify anomalies
- Extracting structured data from HTML web pages (scraping)

---

## 5. Side-by-Side Comparison

| | Structured | Semi-Structured | Unstructured |
|---|---|---|---|
| **Format** | Fixed schema, rows/columns | Flexible schema, key-value | No schema |
| **Storage** | Relational databases, CSV | JSON, XML, NoSQL | Files, object storage |
| **Query method** | SQL | JSON queries, NoSQL queries | AI/ML models, search |
| **Volume** | ~10% of enterprise data | ~10% | ~80% |
| **ML approach** | Classical ML, AutoML | Varies | Deep learning, GenAI |
| **Example** | Sales table | Log file | Customer email |
| **GCP tool** | BigQuery, AutoML Tables | Firestore, BigQuery | Vertex AI, Gemini, Document AI |

---

## 6. Data in the GenAI Pipeline

Understanding how both data types flow through a GenAI system is key for GAIL.

### RAG Pipeline (most common GenAI architecture)

```
Unstructured data (PDFs, docs, emails)
        ↓
  Preprocessing & chunking
        ↓
  Embedding model (converts text → vectors)
        ↓
  Vector database (stores embeddings)
        ↓
  User query → Retrieve relevant chunks → Inject into prompt → LLM response
```

### Training / Fine-tuning Pipeline

```
Raw data (structured + unstructured)
        ↓
  Data cleaning & labeling
        ↓
  Feature engineering (structured) / Tokenization (unstructured)
        ↓
  Model training
        ↓
  Evaluation → Deployment
```

### Key insight for GAIL
LLMs primarily work with **unstructured data** — they read and generate text. However, many enterprise AI applications combine both:
- A LLM generates a summary (unstructured output)
- The result is stored and tagged in a database (structured)
- Structured metadata (user ID, date, category) is used to filter which documents to retrieve in a RAG pipeline

---

## 7. Data Quality: Why It Matters

"Garbage in, garbage out" — the quality of your data directly determines model quality.

### Key data quality dimensions

| Dimension | Description | AI impact |
|---|---|---|
| **Completeness** | Are there missing values? | Missing data causes skewed predictions |
| **Accuracy** | Is the data correct? | Incorrect labels → wrong model behavior |
| **Consistency** | Is the same concept represented the same way? | Inconsistent labels confuse the model |
| **Relevance** | Is the data relevant to the task? | Irrelevant data adds noise |
| **Timeliness** | Is the data up to date? | Stale data causes model drift |
| **Representativeness** | Does data cover all groups fairly? | Unrepresentative data causes bias |

---

## 8. Data in Google Cloud Storage

Knowing where different data types live in GCP is useful for the exam:

| Data type | Typical GCP storage |
|---|---|
| Structured tabular data | BigQuery, Cloud SQL, Cloud Spanner |
| Semi-structured (JSON/logs) | Firestore, BigQuery, Cloud Logging |
| Unstructured files (text, images, video) | Cloud Storage (GCS) |
| Embeddings / vectors | Vertex AI Vector Search, AlloyDB |
| Streaming data | Pub/Sub + Dataflow |

---

## 9. Key Vocabulary Cheat Sheet

| Term | Definition |
|---|---|
| **Structured data** | Data organized in rows and columns with a fixed schema |
| **Unstructured data** | Data without a predefined format (text, images, audio, video) |
| **Semi-structured data** | Flexible schema data (JSON, XML, logs) |
| **Schema** | The predefined structure defining what fields exist and their types |
| **Feature** | An input variable used by an ML model (typically from structured data) |
| **Embedding** | A numerical vector representation of unstructured data (text, image) |
| **Vector database** | Database optimized for storing and searching embeddings |
| **Tokenization** | Splitting text into tokens for processing by an LLM |
| **OCR** | Optical Character Recognition — converting image text to machine-readable text |
| **Data preprocessing** | Cleaning and transforming raw data before model training or inference |
| **Data drift** | When production data diverges from training data over time |
| **BigQuery ML** | SQL-based ML directly on structured data in BigQuery |
| **Document AI** | Google Cloud service for extracting structure from unstructured documents |

