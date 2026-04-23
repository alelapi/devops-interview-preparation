# Grounding in AI

**Grounding** refers to the practice of connecting an AI model's outputs to **reliable, external, real-world information** — so its responses are anchored in verified facts rather than relying solely on what it learned during training.

## Why it matters

LLMs are trained on a static snapshot of data. Without grounding, they can:
- Produce **hallucinations** (confident but wrong facts)
- Give **stale information** (outdated knowledge)
- Lack **context-specific knowledge** (your internal docs, your database, etc.)

## Common grounding techniques

### RAG (Retrieval-Augmented Generation)
The most popular approach. Before generating a response, the system retrieves relevant documents (from a vector DB, search engine, etc.) and injects them into the prompt as context. The model then answers *based on* those documents.

### Tool use / function calling
The model is given tools (web search, SQL queries, API calls) it can invoke to fetch real-time data before answering.

### Fine-tuning on domain data
Training or fine-tuning the model on specific, curated datasets so the knowledge is baked in — though this is less flexible and more expensive than RAG.

### Prompt injection of facts
Simply including relevant facts or documents directly in the system prompt or user message.

## A practical analogy

Think of an ungrounded LLM as someone answering from memory alone. A grounded LLM is like that same person, but allowed to **look things up** before answering — much more reliable.

## Grounding with Google (Vertex AI)

Google Cloud offers native grounding capabilities directly within **Vertex AI**.

### Grounding with Google Search
Vertex AI models (e.g., Gemini) can be configured to ground responses using **Google Search** in real time. When enabled, the model automatically retrieves up-to-date web content before generating an answer, reducing hallucinations and improving factual accuracy.

### Grounding with Vertex AI Search
You can also ground responses against **your own data** by connecting a Vertex AI Search data store (backed by documents, websites, or structured data). This is the enterprise RAG equivalent within Google's ecosystem.

### Key concepts for the certification

| Concept | Description |
|---|---|
| **Grounding sources** | Google Search, Vertex AI Search, or custom data stores |
| **Dynamic grounding** | The model decides when to trigger retrieval based on query confidence |
| **Grounding metadata** | Responses include source citations and support scores |
| **Support score** | A confidence metric (0–1) indicating how well the source supports the claim |
| **Grounding chunks** | Snippets of retrieved content attached to the grounded response |

### How it works (Vertex AI flow)
1. User sends a prompt to a Gemini model via Vertex AI
2. The grounding tool retrieves relevant content (Search or data store)
3. Retrieved chunks are injected into the model's context
4. The model generates a response **anchored to** the retrieved content
5. The response includes **citations and grounding metadata**

### Why it matters for GAIL
Grounding is a core pillar of building **reliable, production-grade GenAI applications** on Google Cloud. The certification tests your understanding of when and how to apply grounding to minimize hallucinations, ensure data freshness, and comply with enterprise accuracy requirements.

## In practice

Grounding is already used implicitly when injecting Kubernetes manifests or runbooks into a prompt and asking the model to reason about them. That's RAG-style grounding in practice. Tools like **Open WebUI + Ollama** also support document grounding natively via their RAG pipelines.