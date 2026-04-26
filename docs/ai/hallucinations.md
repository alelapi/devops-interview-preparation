# Hallucinations

## 1. What is a Hallucination?

A **hallucination** is when an AI model generates output that is **confidently stated but factually incorrect, fabricated, or unsupported** by any real source.

The term comes from psychology — just like a person hallucinating perceives something that isn't there, an LLM "perceives" information that doesn't exist.

### Examples of hallucinations
- Citing a scientific paper that was never published
- Stating a historical date that is wrong
- Inventing a law or regulation
- Generating a plausible-sounding but fake product specification
- Attributing a quote to a person who never said it

### The dangerous part
The model doesn't signal uncertainty. It states fabricated information with the same tone and confidence as correct information. This is what makes hallucinations genuinely risky in production systems.

---

## 2. Why Do Hallucinations Happen?

Understanding the root causes helps you choose the right mitigation strategy.

### 2.1 LLMs predict tokens, not truth
LLMs are trained to predict the most statistically likely next token — not to verify facts. If a plausible-sounding but incorrect continuation is likely given the context, the model will generate it.

```
"The first person to walk on the moon was Neil Armstrong in ___"
→ The model predicts "1969" (correct)

"The CEO of ExampleCorp in 2023 was ___"
→ The model invents a plausible name if it has no data
```

### 2.2 Knowledge gaps
The model's training data has a **cutoff date**. Events, people, or facts after that cutoff are unknown — but the model may still generate something rather than admit it doesn't know.

### 2.3 Rare or sparse training data
Topics with little representation in training data are more prone to hallucination. The model "fills gaps" with statistically likely but incorrect information.

### 2.4 Ambiguous or leading prompts
Poorly written prompts can steer the model toward a hallucinated answer:
```
❌ "Tell me about the studies that prove X causes Y"
   → Model may invent studies to satisfy the premise
✅ "Are there studies linking X to Y? If so, describe them."
```

### 2.5 Overconfidence in reasoning
On complex multi-step logic tasks, errors compound. An early incorrect assumption leads to a confident but wrong conclusion.

---

## 3. Types of Hallucinations

| Type | Description | Example |
|---|---|---|
| **Factual hallucination** | Incorrect real-world facts | Wrong date, wrong person |
| **Source hallucination** | Fabricated citations or references | Fake paper titles |
| **Reasoning hallucination** | Logical errors stated confidently | Wrong math answer |
| **Contextual hallucination** | Contradicts information in the prompt | Ignores provided context |
| **Temporal hallucination** | Outdated or post-cutoff information | Events after training date |

---

## 4. Mitigation Strategies

The GAIL exam focuses on *knowing which strategy to apply* in a given scenario.

### 4.1 Grounding
Anchor the model's response to **verified, external data** rather than relying on its training knowledge.

```
Without grounding: "What is our refund policy?"
→ Model may invent a plausible policy

With grounding: [Inject actual policy document into the prompt]
                "Based on the provided policy document, what is our refund policy?"
→ Model answers from the document, not from memory
```

**Google Cloud implementation:**
- **Vertex AI Grounding with Google Search** — real-time web retrieval
- **Vertex AI Search** — grounding against your own enterprise documents
- **RAG APIs** — inject retrieved chunks into the prompt automatically

> Grounding is the single most effective hallucination mitigation technique for factual tasks.

---

### 4.2 Retrieval-Augmented Generation (RAG)
A specific grounding architecture where a retrieval system fetches relevant documents and injects them into the prompt before generation.

```
User query → Retrieve relevant docs → Inject into prompt → Model generates grounded answer
```

The model is instructed to answer **only from the provided context**, reducing its ability to fabricate.

---

### 4.3 Human-in-the-Loop (HITL)
Introduce **human review** at critical decision points in the AI pipeline. Humans validate or correct model outputs before they are acted upon or shown to end users.

**When to use:** High-stakes decisions — medical advice, legal documents, financial recommendations, content moderation.

**Trade-off:** Slower and more expensive, but essential where errors have real consequences.

**Google Cloud implementation:** Vertex AI supports HITL workflows natively via labeling and review pipelines.

---

### 4.4 Output Validation
Programmatically check the model's output against rules, schemas, or external data sources before using it.

**Examples:**
- Validate that a generated JSON is well-formed
- Check that a cited document actually exists
- Cross-reference a stated fact against a database
- Use a second LLM call to verify the first ("LLM-as-judge")

```
Model output → Validation layer → ✅ Pass / ❌ Reject & retry
```

---

### 4.5 Prompt Engineering
Well-structured prompts reduce hallucination risk:

- **Be explicit:** "If you don't know the answer, say 'I don't know'."
- **Constrain the source:** "Answer only based on the provided document."
- **Avoid leading questions:** Don't assume facts in your prompt.
- **Use chain-of-thought:** Forces the model to show reasoning, making errors easier to catch.
- **Lower temperature:** Reduces randomness for factual tasks.

---

### 4.6 Fine-tuning
Training the model on domain-specific, accurate data reduces hallucinations within that domain by giving the model stronger, more reliable knowledge to draw from.

> Fine-tuning is covered in detail in the Fine-tuning study notes.

---

### 4.7 Model Selection
Some models hallucinate less than others. Larger, more recent models (e.g., Gemini 1.5/2.0 Pro) generally have fewer hallucinations than smaller, older ones. For factual tasks, choose the most capable model available.

---

## 5. Mitigation Strategy Selector

| Scenario | Best strategy |
|---|---|
| Model doesn't have access to your company's data | Grounding / RAG |
| High-stakes output (medical, legal, financial) | HITL |
| Model output needs to match a specific format | Output validation |
| Model gives wrong answers on domain-specific topics | Fine-tuning |
| Model invents facts when it should admit uncertainty | Prompt engineering |
| General factual Q&A needing up-to-date information | Grounding with Google Search |

---

## 6. Hallucinations in the Enterprise Context

For GAIL, hallucinations are not just a technical problem — they are a **business risk**:

- **Legal risk:** A model citing fake regulations or inventing contract terms
- **Reputational risk:** A customer-facing chatbot confidently giving wrong product information
- **Safety risk:** A medical assistant hallucinating drug dosages
- **Compliance risk:** Fabricated data in a regulated industry report

This is why enterprise GenAI deployments always combine grounding + output validation + HITL for high-stakes workflows.

---

## 7. Key Vocabulary Cheat Sheet

| Term | Definition |
|---|---|
| **Hallucination** | Confident, incorrect, or fabricated model output |
| **Grounding** | Anchoring model output to verified external data |
| **RAG** | Retrieval-Augmented Generation — fetch and inject relevant docs before generation |
| **HITL** | Human-in-the-Loop — human review at key pipeline steps |
| **Output validation** | Programmatic checking of model output against rules or data |
| **Confabulation** | Alternative term for hallucination (used in research contexts) |
| **Knowledge cutoff** | Date after which the model has no training data |
| **LLM-as-judge** | Using a second LLM to evaluate or verify the output of a first |
| **Temperature** | Low temperature reduces hallucination risk for factual tasks |

