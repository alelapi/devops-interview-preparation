# Fine-tuning

## 1. What is Fine-tuning?

**Fine-tuning** is the process of taking a pre-trained foundation model and continuing to train it on a **smaller, task-specific dataset** to specialize its behavior for a particular domain, style, or task.

Think of it like this:
- **Pre-training** = a person completing a university degree (broad, general knowledge)
- **Fine-tuning** = that same person doing a 3-month internship in a specific company/role (specialized, practical skills)

The model doesn't forget its general knowledge — it *builds on top of it* with domain-specific expertise.

---

## 2. Why Fine-tune? Prompting vs Fine-tuning

Before choosing fine-tuning, it's important to know when it's actually needed — because it's expensive and time-consuming compared to prompting.

### Prompting (default approach)
You guide the model's behavior entirely through the input text. No training required.

```
Prompt: "You are a legal assistant. Respond formally and cite relevant laws."
```

**Pros:** Fast, flexible, no cost beyond API calls, easy to change.
**Cons:** Prompt takes up context window space; behavior can be inconsistent; may not capture highly specialized knowledge.

---

### Fine-tuning (when prompting isn't enough)
You actually modify the model's weights using new training data.

**Pros:** More consistent behavior, better performance on specialized tasks, no need to repeat instructions in every prompt, can compress domain knowledge into the model.
**Cons:** Requires labeled training data, time, compute cost, and maintenance.

---

### Decision guide: Prompt or Fine-tune?

| Situation | Use |
|---|---|
| General task the model already does well | Prompting |
| Need consistent tone/style across all responses | Fine-tuning |
| Domain-specific vocabulary the model doesn't know | Fine-tuning |
| Task requires proprietary knowledge | Fine-tuning + Grounding |
| Prototype or early-stage product | Prompting |
| High-volume production system needing efficiency | Fine-tuning |
| Model needs to follow a very specific output format | Fine-tuning |

---

## 3. How Fine-tuning Works

### Step 1 — Start with a foundation model
You begin with a pre-trained model (e.g., Gemini) that already understands language, reasoning, and general knowledge.

### Step 2 — Prepare your dataset
Collect and clean a dataset of **input-output pairs** relevant to your task.

```
Input:  "What is the coverage limit for flood damage?"
Output: "According to Policy Section 4.2, flood damage is covered up to $250,000..."

Input:  "Is earthquake damage included?"
Output: "Earthquake damage is not covered under standard policies. Refer to Section 7..."
```

The quality and diversity of this dataset directly determines how good the fine-tuned model will be.

**Rule of thumb:** You typically need hundreds to thousands of high-quality examples for effective fine-tuning.

### Step 3 — Train
The model is trained on your dataset. Rather than training from scratch, the model makes small adjustments to its weights to better handle your specific examples. This is much faster and cheaper than pre-training.

### Step 4 — Evaluate
Measure the fine-tuned model's performance on a held-out test set. Compare against the base model and against prompt-only approaches.

### Step 5 — Deploy
Deploy the fine-tuned model via the API. On Vertex AI, fine-tuned models are hosted as dedicated endpoints.

---

## 4. Types of Fine-tuning

### 4.1 Supervised Fine-tuning (SFT)
The most common type. You provide labeled input-output pairs and the model learns to produce the desired output given an input.

**Use cases:** Customer support bots, domain-specific Q&A, classification, structured output generation.

---

### 4.2 RLHF (Reinforcement Learning from Human Feedback)
Human raters score model outputs. The model is trained to maximize scores. This is how Google aligns Gemini to be helpful, safe, and accurate.

**Use cases:** Improving general response quality, reducing harmful outputs, aligning to brand voice.

> RLHF is more commonly used by model providers (like Google) than by enterprises deploying models.

---

### 4.3 PEFT — Parameter-Efficient Fine-tuning
Instead of updating all of the model's billions of parameters (which is expensive), PEFT techniques update only a **small subset** of parameters.

The most popular PEFT method is **LoRA (Low-Rank Adaptation):**
- Adds small, trainable matrices alongside frozen original weights
- Dramatically reduces memory and compute requirements
- Quality is close to full fine-tuning at a fraction of the cost

**Why it matters for GAIL:** Vertex AI supports PEFT/LoRA-based fine-tuning, making it accessible to enterprises without massive GPU budgets.

---

### 4.4 Distillation
A large, expensive model (the "teacher") is used to train a smaller, faster model (the "student") to mimic its behavior.

**Use case:** You want a cost-efficient model for high-volume inference that behaves like a larger model.

---

## 5. Fine-tuning vs Other Adaptation Techniques

| Technique | Changes model weights? | Requires training data? | Cost | Best for |
|---|---|---|---|---|
| **Prompting** | ❌ No | ❌ No | Low | General tasks |
| **Few-shot prompting** | ❌ No | ✅ Examples in prompt | Low | Quick task adaptation |
| **RAG / Grounding** | ❌ No | ✅ Documents | Low-Medium | Dynamic, factual knowledge |
| **Fine-tuning (SFT)** | ✅ Yes | ✅ Labeled pairs | Medium-High | Consistent specialized behavior |
| **PEFT / LoRA** | ✅ Partially | ✅ Labeled pairs | Medium | Efficient specialization |
| **Pre-training from scratch** | ✅ Yes (all) | ✅ Massive corpus | Very High | Building a new foundation model |

---

## 6. Fine-tuning on Google Cloud (Vertex AI)

Google Cloud exposes fine-tuning capabilities through **Vertex AI**:

### Supervised Tuning (Vertex AI)
- Available for Gemini models
- Provide a JSONL dataset of prompt-response pairs
- Google manages the training infrastructure
- The result is a **tuned model version** you can deploy as an endpoint

### Reinforcement Tuning
- Available for Gemini models via Vertex AI
- Uses reward signals to align the model to specific goals

### Model Garden
- **Vertex AI Model Garden** offers access to open-source models (Llama, Mistral, etc.) that can be fine-tuned on custom data with more control

### Key GAIL exam points:
- Fine-tuning in Vertex AI does **not** require you to manage infrastructure
- Your training data stays within your Google Cloud project (data privacy)
- Fine-tuned models are billed differently from base models (dedicated endpoints)

---

## 7. Data Requirements for Fine-tuning

The dataset is the most critical factor in fine-tuning success.

### Quality over quantity
A few hundred high-quality, diverse examples often outperform thousands of noisy ones.

### Dataset format (Vertex AI)
```json
{"input_text": "Classify this review: 'Great product!'", "output_text": "positive"}
{"input_text": "Classify this review: 'Terrible, broke on day 1'", "output_text": "negative"}
```

### Common data pitfalls
- **Biased data** → biased model (see Responsible AI notes)
- **Too narrow** → model overfits, fails on edge cases
- **Inconsistent labels** → model learns contradictions
- **Too little data** → minimal improvement over base model

---

## 8. When Fine-tuning is NOT the Right Answer

Fine-tuning is often over-used. Know when to avoid it:

| Situation | Better alternative |
|---|---|
| You just need a different tone | System prompt / role prompting |
| You need up-to-date or dynamic knowledge | RAG / Grounding |
| You're still in prototyping | Few-shot prompting |
| Your task changes frequently | Prompting (fine-tuning is static) |
| You don't have labeled training data | Prompting or RAG |

---

## 9. Key Vocabulary Cheat Sheet

| Term | Definition |
|---|---|
| **Fine-tuning** | Further training a pre-trained model on a task-specific dataset |
| **SFT** | Supervised Fine-Tuning — training on labeled input-output pairs |
| **RLHF** | Reinforcement Learning from Human Feedback — trains on human preference scores |
| **PEFT** | Parameter-Efficient Fine-Tuning — updates only a small subset of weights |
| **LoRA** | Low-Rank Adaptation — popular PEFT method, cheap and effective |
| **Distillation** | Training a small model to mimic a larger one |
| **Overfitting** | Model memorizes training data, fails on new inputs |
| **Tuned model** | A fine-tuned version of a base model deployed as its own endpoint |
| **Base model** | The original, unmodified foundation model |
| **JSONL** | JSON Lines format — standard for Vertex AI fine-tuning datasets |

