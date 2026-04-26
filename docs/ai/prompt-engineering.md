# Prompt Engineering

## 1. What is Prompt Engineering?

A **prompt** is the input you give to an LLM — a question, instruction, or piece of text that tells the model what to do.

**Prompt Engineering** is the practice of designing and refining prompts to get the most accurate, relevant, and useful outputs from a model.

### Why does it matter?
The same model can produce drastically different results depending on how you phrase your input:

| Prompt | Output Quality |
|---|---|
| `"Summarize this"` | Vague, possibly unhelpful |
| `"Summarize the following customer complaint in 2 sentences, focusing on the core issue and the customer's emotional tone."` | Precise, useful, actionable |

Prompt engineering is a skill — and for GAIL, it's tested both as a concept and as a practical technique for improving GenAI applications.

---

## 2. Anatomy of a Good Prompt

A well-structured prompt typically contains some or all of these elements:

```
┌─────────────────────────────────────────────┐
│  SYSTEM CONTEXT (optional)                  │
│  "You are a helpful customer support agent" │
├─────────────────────────────────────────────┤
│  INSTRUCTION                                │
│  "Classify the sentiment of this review"   │
├─────────────────────────────────────────────┤
│  EXAMPLES (optional)                        │
│  "Positive: 'Great product!' → positive"   │
├─────────────────────────────────────────────┤
│  INPUT DATA                                 │
│  "Review: 'Arrived late and broken.'"      │
├─────────────────────────────────────────────┤
│  OUTPUT FORMAT (optional)                   │
│  "Respond with: positive / negative /      │
│   neutral only."                           │
└─────────────────────────────────────────────┘
```

Not every prompt needs all sections — but including more structure generally improves results.

---

## 3. Prompting Techniques

### 3.1 Zero-Shot Prompting
You give the model a task with **no examples**. You rely entirely on the model's pre-trained knowledge.

```
Prompt:
"Translate the following sentence to Italian:
'The meeting starts at 9am.'"

Output:
"La riunione inizia alle 9."
```

**When to use:** Simple, well-defined tasks where the model is already capable.

**Limitation:** For complex or domain-specific tasks, the model may misinterpret what you want without examples.

---

### 3.2 Few-Shot Prompting
You provide **a small number of examples** (typically 2–5) inside the prompt, showing the model the pattern you expect.

```
Prompt:
"Classify the sentiment of each review.

Review: 'Absolutely love this product!' → positive
Review: 'Waste of money, broke after a week.' → negative
Review: 'It works fine, nothing special.' → neutral

Review: 'Shipping was fast but the packaging was damaged.' → "

Output:
"negative"
```

**When to use:** When the task has a specific format, style, or domain the model needs to match. Very effective for classification, extraction, and transformation tasks.

**Key insight:** The examples in few-shot prompting act as in-context training — the model adapts its behavior without any actual retraining.

---

### 3.3 Chain-of-Thought (CoT) Prompting
You instruct the model to **think step by step** before giving a final answer. This dramatically improves performance on reasoning, math, and multi-step logic tasks.

**Without CoT:**
```
Prompt: "Roger has 5 tennis balls. He buys 2 more cans of 3 balls each. How many does he have?"
Output: "11"  ← correct but arrived at by chance
```

**With CoT:**
```
Prompt: "Roger has 5 tennis balls. He buys 2 more cans of 3 balls each.
How many does he have? Think step by step."

Output:
"Roger starts with 5 balls.
He buys 2 cans × 3 balls = 6 balls.
5 + 6 = 11 balls total."
```

**Why it works:** Forcing the model to externalize reasoning reduces errors by making each step explicit and checkable.

**Variants:**
- **Zero-shot CoT:** Just add "Think step by step" or "Let's reason through this"
- **Few-shot CoT:** Provide full worked examples with step-by-step reasoning shown

**When to use:** Math problems, logic puzzles, multi-step planning, complex decision-making questions.

---

### 3.4 System Prompting
A **system prompt** is a set of instructions given to the model *before* the user interaction begins, typically by the developer. It sets the persona, tone, rules, and constraints.

```
System: "You are a concise technical assistant for platform engineers.
         Always respond in bullet points. Never discuss pricing.
         If asked something outside your scope, say 'I can't help with that'."

User: "What's the best way to monitor a Kubernetes cluster?"

Output: Bullet-pointed, technical response, on-topic.
```

**When to use:** In applications and products where you need consistent model behavior across all user interactions. Critical for enterprise GenAI deployments.

---

### 3.5 Role Prompting
Assign the model a specific **persona or role** to influence its style and depth of response.

```
"You are an experienced cardiologist explaining heart disease to a patient
with no medical background. Use simple language and avoid jargon."
```

**Why it works:** The model has seen vast amounts of text associated with different roles and adjusts its style accordingly.

---

### 3.6 Retrieval-Augmented Generation (RAG) Prompting
Inject **external, retrieved information** directly into the prompt so the model grounds its answer in specific documents rather than relying on training knowledge alone.

```
Context: [Retrieved document: Company Q3 earnings report]
Question: "What was the revenue growth in Q3?"
Instruction: "Answer based only on the provided context."
```

> RAG is covered in depth in the Grounding study notes.

---

## 4. Output Controlling Parameters

These are settings you can adjust when calling a model via API (e.g., in Vertex AI or Google AI Studio) to control the *style* and *randomness* of the output.

### 4.1 Temperature
Controls the **randomness / creativity** of the output.

- Range: typically `0.0` to `2.0`
- **Low temperature (0.0–0.3):** Deterministic, predictable, always picks the most likely next token. Good for factual tasks.
- **High temperature (0.8–2.0):** More random, creative, diverse. Good for brainstorming or creative writing.

```
Temperature = 0.1 → "The capital of France is Paris."
Temperature = 1.5 → "Ah, France! A land of croissants, where Paris dreams in cobblestones..."
```

**Analogy:** Temperature is like a dial that goes from "always safe choice" to "roll the dice."

---

### 4.2 Top-P (Nucleus Sampling)
Controls output diversity by limiting the model to choosing from the **smallest set of tokens** whose cumulative probability reaches `P`.

- Range: `0.0` to `1.0`
- **Top-P = 0.1:** Only considers the top 10% most likely tokens → very focused
- **Top-P = 0.9:** Considers a broader set → more varied output

**How it differs from temperature:**
- Temperature scales probabilities up/down before sampling
- Top-P cuts off the *long tail* of unlikely tokens entirely

**In practice:** Temperature and Top-P are often used together. A common default is `temperature=0.7, top_p=0.9`.

---

### 4.3 Top-K
Limits sampling to the **K most likely next tokens**.

- `Top-K = 1` → always picks the single most likely token (greedy, deterministic)
- `Top-K = 40` → randomly selects from the 40 most probable next tokens

Less commonly tuned than temperature or top-p, but available in Vertex AI.

---

### 4.4 Max Output Tokens
Sets a hard limit on the **length of the response**.

- Prevents unexpectedly long (and costly) outputs
- Too low and the model may cut off mid-sentence
- Set based on your use case: summaries need fewer tokens than full reports

---

### 4.5 Stop Sequences
A string or list of strings that tell the model to **stop generating** when encountered.

```
stop_sequences = ["\n\n", "END"]
```

Useful when you want structured outputs and need to prevent the model from generating extra content beyond a delimiter.

---

### Parameter Quick Reference

| Parameter | Controls | Low value | High value |
|---|---|---|---|
| **Temperature** | Randomness | Deterministic, focused | Creative, varied |
| **Top-P** | Token pool breadth | Very focused | More diverse |
| **Top-K** | Number of token candidates | Narrow, predictable | Broader options |
| **Max tokens** | Response length | Short | Long |
| **Stop sequences** | Where to stop | N/A | N/A |

---

## 5. Prompt Engineering Best Practices

### Be specific and detailed
Vague prompts produce vague answers.
```
❌ "Write about dogs"
✅ "Write a 150-word paragraph for a pet adoption website about the benefits
    of adopting adult dogs, targeting first-time pet owners."
```

### Specify the output format
```
✅ "Respond as a JSON object with keys: 'sentiment', 'confidence', 'reason'."
✅ "Use bullet points. Maximum 5 items."
✅ "Answer in one sentence."
```

### Use positive instructions over negative ones
```
❌ "Don't be verbose"
✅ "Be concise. Maximum 3 sentences."
```

### Iterate and test
Prompt engineering is empirical — try, measure, refine. There's no perfect formula.

### Separate instructions from data clearly
```
✅ "Summarize the following article. Article: [article text here]"
```
Using labels like `Article:`, `Question:`, `Context:` reduces ambiguity.

---

## 6. Common Prompting Pitfalls

| Pitfall | Problem | Fix |
|---|---|---|
| **Ambiguous task** | Model guesses what you want | Be explicit about task and format |
| **No context** | Generic, surface-level response | Provide relevant background |
| **Prompt injection** | Malicious input overrides your instructions | Use system prompts and input validation |
| **Overloading one prompt** | Too many tasks at once | Break complex tasks into steps |
| **Ignoring temperature** | Factual task with high temp → hallucinations | Lower temperature for factual outputs |

---

## 7. Prompt Engineering in Google Cloud

In the Google ecosystem, you can experiment with prompts using:

| Tool | Purpose |
|---|---|
| **Google AI Studio** | Free, browser-based prompt playground for Gemini models |
| **Vertex AI Prompt Management** | Store, version, and deploy prompts in production |
| **Vertex AI Studio** | Enterprise-grade prompt testing and model tuning |
| **NotebookLM** | Document-grounded AI — prompting against your own documents |

### Prompt types in Vertex AI / AI Studio
- **Freeform prompt:** Open-ended, conversational
- **Structured prompt:** Includes examples (few-shot), context, and explicit instructions
- **Chat prompt:** Multi-turn conversation with system instructions

---

## 8. Key Vocabulary Cheat Sheet

| Term | Definition |
|---|---|
| **Prompt** | The input you provide to an LLM |
| **Zero-shot** | Task given with no examples |
| **Few-shot** | Task given with a small number of examples |
| **Chain-of-thought** | Prompting the model to reason step by step |
| **System prompt** | Pre-conversation instructions that set model behavior |
| **Temperature** | Controls randomness of output (0 = deterministic, high = creative) |
| **Top-P** | Limits token selection to a cumulative probability threshold |
| **Top-K** | Limits token selection to the K most probable tokens |
| **Max tokens** | Maximum length of the model's response |
| **Stop sequence** | String that signals the model to stop generating |
| **In-context learning** | Model adapts behavior based on examples in the prompt |
| **Prompt injection** | Malicious input designed to override system instructions |
| **Hallucination** | Confident but incorrect model output |
| **Grounding** | Anchoring outputs to verified external data |

