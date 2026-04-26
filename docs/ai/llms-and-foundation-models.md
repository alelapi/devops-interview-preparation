# Large Language Models (LLMs) & Foundation Models

## 1. What is a Large Language Model (LLM)?

A **Large Language Model** is a type of AI model trained on massive amounts of text data to understand and generate human language.

"Large" refers to two things:
- **Large data:** trained on hundreds of billions of words (books, websites, code, articles)
- **Large model:** billions of internal parameters (weights) that encode knowledge

**Examples:** Google Gemini, GPT-4, Claude, LLaMA, Mistral.

### What can LLMs do?
- Generate text (articles, emails, code, summaries)
- Answer questions
- Translate languages
- Classify or analyze text
- Hold conversations
- Reason through problems step by step

### Key insight
LLMs don't "know" things the way humans do. They are extremely good at predicting **what text should come next** given a context — and this surprisingly general capability leads to intelligent-seeming behavior.

---

## 2. How are LLMs Trained?

Training an LLM happens in multiple phases:

### Phase 1 — Pre-training (Self-Supervised Learning)
The model is trained on a huge corpus of raw text (no human labels needed).

**Task:** Predict the next token (word/subword) given the previous ones.

```
Input:  "The capital of France is ___"
Target: "Paris"
```

The model processes billions of such examples and adjusts its billions of parameters to get better at this task. Over time, it implicitly learns grammar, facts, reasoning patterns, and world knowledge.

- This phase requires **enormous computing power** (thousands of GPUs/TPUs, weeks of training)
- Google uses **TPUs (Tensor Processing Units)** — custom chips optimized for this workload

### Phase 2 — Fine-tuning (Supervised Learning)
After pre-training, the model is further trained on a smaller, curated dataset for a specific task or behavior.

**Example:** Fine-tuning on question-answer pairs so the model responds helpfully to user questions rather than just completing text.

On Google Cloud, this is done via **Vertex AI supervised tuning**.

### Phase 3 — RLHF (Reinforcement Learning from Human Feedback)
Human raters evaluate model outputs and score them. The model is trained to maximize high-quality, safe, and helpful responses.

```
Model generates answer → Human rates it → Model improves toward higher-rated outputs
```

This is what makes LLMs feel "aligned" — they've learned not just to be accurate, but to be helpful, harmless, and honest.

---

## 3. The Transformer Architecture (Simplified)

All modern LLMs are built on the **Transformer** architecture (introduced by Google in 2017 in the paper "Attention Is All You Need").

The key innovation is the **attention mechanism** — the model learns which parts of the input to "pay attention to" when generating each output token.

```
Input tokens → Embeddings → Multiple Transformer Layers → Output tokens
                                  ↑
                          (Attention mechanism
                           learns relationships
                           between all tokens)
```

You don't need to understand the math for GAIL, but you should know:
- Transformers are the architecture behind LLMs
- They handle **long-range dependencies** in text (e.g., a pronoun referring to a noun several sentences back)
- The more layers and parameters, the more capable (and expensive) the model

---

## 4. What are Foundation Models?

A **Foundation Model** is a large model trained on broad, general data that can be **adapted to a wide range of downstream tasks**.

The term was coined by Stanford researchers in 2021. It describes models like Gemini, GPT-4, and Claude.

### Key Characteristics of Foundation Models

| Characteristic | Description |
|---|---|
| **Scale** | Trained on massive datasets with billions of parameters |
| **Generality** | Not trained for one specific task — adaptable to many |
| **Emergent capabilities** | Abilities that weren't explicitly trained (e.g., reasoning, code generation) |
| **Transfer learning** | Knowledge from pre-training transfers to new tasks with minimal additional training |
| **Multimodal potential** | Many can handle text, images, audio, and video |

### The "Foundation" Metaphor
Think of a foundation model like a **pre-built foundation for a house**. Instead of laying every brick yourself (training from scratch), you build on top of this foundation — adapting it for your specific use case through fine-tuning or prompting.

---

## 5. Foundation Models vs Traditional ML Models

This is a common exam topic. The contrast is important.

| | Traditional ML Model | Foundation Model |
|---|---|---|
| **Training data** | Small, task-specific, labeled dataset | Massive, general, mostly unlabeled |
| **Task scope** | One specific task (e.g., classify emails) | General-purpose, adaptable to many tasks |
| **Reusability** | Hard to reuse for other tasks | Designed to be reused and adapted |
| **Training cost** | Low to moderate | Extremely high (millions of $$$) |
| **Adaptation method** | Re-train from scratch | Prompting or fine-tuning |
| **Data requirements** | Needs clean, labeled data | Can learn from raw, unlabeled data |
| **Examples** | Spam classifier, fraud detector | Gemini, GPT-4, Claude |

### Practical implication for GAIL
Organizations don't train foundation models — they **use** them. Google provides foundation models via Vertex AI Model Garden and Gemini APIs. Companies then adapt them with prompting, grounding, or fine-tuning.

---

## 6. Types of Generative AI Models

The GAIL exam requires you to distinguish between three main model families:

### 6.1 Large Language Models (LLMs)
- **Input/Output:** Text → Text (primarily)
- **How they work:** Transformer-based, trained to predict the next token
- **Strengths:** Language understanding, generation, reasoning, Q&A, summarization, code
- **Examples:** Gemini Pro, GPT-4, Claude, LLaMA

**Use cases:** Chatbots, document summarization, code assistants, translation, content generation.

---

### 6.2 Diffusion Models
- **Input/Output:** Noise → Image (or Audio/Video)
- **How they work:** They learn to progressively **remove noise** from a random signal to reconstruct a meaningful output (like an image)

```
Random noise → [Denoising steps] → Final image
```

The training process works in reverse: take a real image, gradually add noise until it's pure static, then teach the model to reverse this process.

- **Strengths:** High-quality image, audio, and video generation
- **Examples:** Stable Diffusion, DALL-E, Google Imagen, Sora (video)

**Use cases:** Image generation, image editing, video synthesis, product design, art creation.

**Key difference from LLMs:** Diffusion models generate *visual/audio media*, not text. They operate in pixel/latent space, not token space.

---

### 6.3 Multimodal Models
- **Input/Output:** Multiple modalities (text + image + audio + video)
- **How they work:** Combine different model architectures (e.g., vision encoder + language model) so the model can reason across modalities

```
Input:  [Image of a chart] + "Summarize this data"
Output: "The chart shows revenue grew 40% in Q3..."
```

- **Strengths:** Cross-modal understanding and generation
- **Examples:** Gemini 1.5/2.0 (Google's flagship multimodal model), GPT-4o, Claude 3

**Use cases:**
- Describing or analyzing images
- Answering questions about documents with charts
- Generating images from text descriptions
- Video understanding and Q&A
- Audio transcription and analysis

**Key insight for GAIL:** Gemini is Google's primary multimodal foundation model. Knowing that Gemini can handle text, images, audio, video, and code in a single model is a core exam point.

---

## 7. Side-by-Side Comparison

| | LLM | Diffusion Model | Multimodal Model |
|---|---|---|---|
| **Primary input** | Text | Text prompt (for image gen) | Text + Image + Audio + Video |
| **Primary output** | Text | Image / Video / Audio | Any combination |
| **Architecture** | Transformer | U-Net / Diffusion process | Hybrid (Transformer + Vision encoder) |
| **Google example** | Gemini (text mode) | Imagen | Gemini 1.5/2.0 Pro |
| **Typical use** | Chat, summarization | Image generation | Document Q&A, visual reasoning |

---

## 8. Token: The Fundamental Unit of LLMs

Everything in an LLM revolves around **tokens** — the basic units of text the model processes.

- A token is roughly **3/4 of a word** on average
- "ChatGPT is great" ≈ 4–5 tokens
- LLMs have a **context window** — the maximum number of tokens they can process at once

| Model | Approximate Context Window |
|---|---|
| Gemini 1.0 Pro | 32,000 tokens |
| Gemini 1.5 Pro | 1,000,000 tokens (~750,000 words) |
| Gemini 2.0 Flash | 1,000,000 tokens |

**Why context window matters:** A larger context window means the model can "remember" longer conversations, process full documents, and reason over more information at once.

---

## 9. Emergent Capabilities

One of the most surprising properties of large foundation models is **emergence** — capabilities that appear only at scale and were never explicitly trained.

Examples of emergent capabilities in LLMs:
- **Multi-step reasoning** — solving math problems step by step
- **In-context learning** — learning from examples given in the prompt (few-shot)
- **Code generation** — writing functional code without being explicitly trained for it
- **Language translation** — without being a dedicated translation model

**Key insight:** Nobody programmed these capabilities directly. They *emerged* from scale. This is both what makes foundation models powerful and what makes them unpredictable.

---

## 10. Key Vocabulary Cheat Sheet

| Term | Definition |
|---|---|
| **Token** | Basic unit of text an LLM processes (~¾ of a word) |
| **Context window** | Max tokens a model can process in one request |
| **Parameter** | Internal value the model learns during training (billions in LLMs) |
| **Embedding** | A numerical vector representation of text/image meaning |
| **Transformer** | The neural network architecture powering modern LLMs |
| **Attention mechanism** | How a Transformer weighs the importance of each token relative to others |
| **Pre-training** | Initial training on massive unlabeled data |
| **Fine-tuning** | Further training on smaller, task-specific data |
| **RLHF** | Reinforcement Learning from Human Feedback — aligns models to human preferences |
| **Foundation model** | Large, general-purpose model adaptable to many tasks |
| **Multimodal** | A model that handles multiple input/output types (text, image, audio, video) |
| **Diffusion model** | Generates images/video by learning to reverse a noise process |
| **Emergent capability** | Ability that appears at scale but wasn't explicitly trained |
| **Latency** | Time to receive the first response from the model |
| **Throughput** | Number of tokens generated per second |

---

## 11. Google's LLM / Foundation Model Ecosystem (for GAIL)

| Model / Product | Type | Where to access |
|---|---|---|
| **Gemini 2.0 Flash** | Multimodal LLM (fast, efficient) | Vertex AI, AI Studio |
| **Gemini 1.5 Pro** | Multimodal LLM (long context) | Vertex AI, AI Studio |
| **Imagen 3** | Diffusion model (image generation) | Vertex AI |
| **Chirp** | Audio / Speech model | Vertex AI |
| **Codey** | Code-specialized LLM | Vertex AI |
| **Vertex AI Model Garden** | Catalog of Google + third-party models | Vertex AI |

