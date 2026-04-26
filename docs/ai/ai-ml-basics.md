# AI & ML Basics

## 1. The Big Picture: AI vs ML vs Deep Learning

These three terms are often used interchangeably, but they have a clear hierarchy:

```
Artificial Intelligence (AI)
└── Machine Learning (ML)
    └── Deep Learning (DL)
```

Think of it like this:
- **AI** is the broad goal: making machines behave intelligently.
- **ML** is one approach to achieving AI: instead of programming rules, you let the machine *learn* from data.
- **Deep Learning** is a specific ML technique inspired by the human brain, using layered neural networks.

---

## 2. Traditional Programming vs Machine Learning

### Traditional Programming
You explicitly write all the rules. The computer follows them exactly.

```
Input + Rules → Output
```

**Example:** A spam filter where you manually write: "if the email contains the word 'lottery', mark as spam."

**Limitations:** Brittle. You can't anticipate every case. Rules become unmanageable at scale.

---

### Machine Learning
You provide data and the expected outputs. The machine *figures out the rules itself*.

```
Input + Output (data) → Rules (model)
```

**Example:** You feed thousands of emails labeled "spam" or "not spam". The model learns the patterns on its own.

**Key insight:** ML is about pattern recognition from experience, not explicit instructions.

---

### Deep Learning
A subset of ML that uses **neural networks** with many layers ("deep" = many layers). Excels at complex, unstructured data like images, audio, and text.

**Example:** Recognizing faces in photos, transcribing speech, generating human-like text (like ChatGPT or Gemini).

**Key insight:** Deep Learning powers most modern AI breakthroughs, including Large Language Models (LLMs).

---

## 3. Types of Machine Learning

### 3.1 Supervised Learning
The model learns from **labeled data** — examples where the correct answer is already known.

- **How it works:** You provide input-output pairs. The model learns to map inputs to outputs.
- **Goal:** Predict the output for new, unseen inputs.

| Input | Label (Output) |
|---|---|
| Email text | Spam / Not Spam |
| House features | Price |
| Image of a cat | "Cat" |

**Common tasks:**
- **Classification** — assign a category (spam or not, dog or cat)
- **Regression** — predict a number (house price, temperature)

**Real-world example:** Google Photos recognizing faces, fraud detection in banking.

---

### 3.2 Unsupervised Learning
The model learns from **unlabeled data** — no correct answers are provided. It discovers hidden patterns or structure on its own.

- **How it works:** The model groups or organizes data based on similarity.
- **Goal:** Find structure where none was explicitly defined.

**Common tasks:**
- **Clustering** — group similar items together (e.g., group customers by behavior)
- **Dimensionality Reduction** — simplify complex data while preserving meaning
- **Anomaly Detection** — identify unusual patterns (e.g., network intrusion detection)

**Real-world example:** Spotify grouping songs into playlists, customer segmentation in marketing.

---

### 3.3 Reinforcement Learning (RL)
The model (called an **agent**) learns by **interacting with an environment** and receiving rewards or penalties.

- **How it works:** The agent takes actions, receives feedback (reward/penalty), and adjusts its strategy over time.
- **Goal:** Maximize cumulative reward.
- **Analogy:** Training a dog with treats — good behavior gets rewarded.

**Real-world examples:**
- Game-playing AI (AlphaGo, chess engines)
- Robotics (teaching a robot to walk)
- Ad bidding optimization
- RLHF (Reinforcement Learning from Human Feedback) — used to align LLMs like Gemini with human preferences

---

### Quick Comparison

| | Supervised | Unsupervised | Reinforcement |
|---|---|---|---|
| **Data** | Labeled | Unlabeled | No dataset needed |
| **Feedback** | Correct answers given | No feedback | Reward/penalty signal |
| **Goal** | Predict | Discover patterns | Maximize reward |
| **Example** | Email spam filter | Customer segmentation | Game-playing AI |

---

## 4. The Machine Learning Lifecycle

Building an ML model is not a one-step process. It follows a lifecycle with distinct stages:

```
1. Problem Definition
        ↓
2. Data Collection
        ↓
3. Data Preparation
        ↓
4. Model Training
        ↓
5. Model Evaluation
        ↓
6. Model Deployment
        ↓
7. Monitoring & Maintenance
        ↓
   (loop back as needed)
```

### Stage 1 — Problem Definition
Clearly define what you want to predict or automate. Is it a classification problem? Regression? What data do you have? What does "success" look like?

### Stage 2 — Data Collection
Gather raw data from relevant sources: databases, APIs, sensors, user interactions, etc. More data generally means better models — but quality matters more than quantity.

### Stage 3 — Data Preparation
This is typically the **most time-consuming** stage (~70-80% of the effort).
- **Cleaning:** Remove duplicates, fix errors, handle missing values
- **Transformation:** Normalize/scale values, encode categorical variables
- **Splitting:** Divide data into training set, validation set, and test set
- **Feature engineering:** Create new meaningful inputs from raw data

### Stage 4 — Model Training
Feed the prepared data into a chosen algorithm. The model adjusts its internal parameters to minimize prediction errors. This is where "learning" actually happens.

Key concepts:
- **Algorithm selection:** Which model type to use (decision tree, neural network, etc.)
- **Hyperparameters:** Settings that control the training process (learning rate, number of layers)
- **Overfitting:** When the model memorizes training data but fails on new data
- **Underfitting:** When the model is too simple to capture patterns

### Stage 5 — Model Evaluation
Test the model on data it has never seen before (the test set). Measure performance using metrics:
- **Accuracy** — % of correct predictions
- **Precision / Recall** — important for imbalanced datasets (e.g., fraud detection)
- **F1 Score** — balance between precision and recall
- **RMSE** — for regression tasks

If performance is insufficient, loop back to earlier stages.

### Stage 6 — Model Deployment
Package the model and expose it via an API or integration so applications can use it in production. On Google Cloud, this typically happens through **Vertex AI**.

### Stage 7 — Monitoring & Maintenance
ML models can degrade over time as real-world data changes — this is called **model drift** or **data drift**. Continuous monitoring ensures the model remains accurate and relevant.

---

## 5. Key Vocabulary Cheat Sheet

| Term | Definition |
|---|---|
| **Feature** | An input variable used to make a prediction (e.g., age, email text) |
| **Label** | The correct output/answer during training (e.g., "spam") |
| **Model** | The trained artifact that makes predictions |
| **Training** | The process of a model learning from data |
| **Inference** | Using a trained model to make predictions on new data |
| **Overfitting** | Model too tailored to training data, fails on new data |
| **Underfitting** | Model too simple, misses patterns |
| **Neural Network** | A layered structure of nodes inspired by the human brain |
| **Parameter** | Internal values the model adjusts during training (e.g., weights) |
| **Hyperparameter** | Settings you choose before training (e.g., learning rate) |
| **Data Drift** | When real-world data changes and the model becomes less accurate |
| **RLHF** | Reinforcement Learning from Human Feedback — used to align LLMs |

---

## 6. How This Connects to Generative AI

Generative AI (like Gemini) is built on top of these foundations:

- It uses **Deep Learning** (specifically Transformer-based neural networks)
- It is trained with a mix of **supervised learning** (next-token prediction) and **reinforcement learning** (RLHF — human raters give feedback to improve quality and safety)
- The ML lifecycle applies to LLMs too: data collection → pre-training → fine-tuning → evaluation → deployment → monitoring

Understanding these basics is what allows you to reason about *why* GenAI models behave the way they do — and how to improve or correct their outputs.

