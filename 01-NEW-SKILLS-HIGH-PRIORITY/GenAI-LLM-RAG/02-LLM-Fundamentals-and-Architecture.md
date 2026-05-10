# Chapter 02: LLM Fundamentals and Architecture

**Understanding How Large Language Models Actually Work**

---

## Table of Contents
1. [Neural Networks Basics](#neural-networks-basics)
2. [The Transformer Architecture](#the-transformer-architecture)
3. [Attention Mechanism Explained](#attention-mechanism-explained)
4. [How LLMs are Trained](#how-llms-are-trained)
5. [Model Architectures (GPT, BERT, T5)](#model-architectures)
6. [Scaling Laws](#scaling-laws)

---

## Neural Networks Basics

Before understanding LLMs, let's understand the building blocks.

### What is a Neural Network?

**Simple Definition:** A mathematical system inspired by the human brain that learns patterns from data.

### Real-Life Analogy

**Neural Network = Learning to Recognize Faces**

```
Step 1: See many faces (training data)
Step 2: Brain adjusts connections (learning)
Step 3: Can recognize new faces (inference)
```

Similarly, a neural network:
```
Step 1: Sees training data
Step 2: Adjusts internal parameters (weights)
Step 3: Makes predictions on new data
```

---

### Basic Structure

```
Input Layer → Hidden Layers → Output Layer
    ↓             ↓              ↓
  Data      Transformations    Result
```

**Example: Spam Detection**

```
Input: Email text
   ↓
Hidden Layer 1: Detect word patterns
Hidden Layer 2: Identify suspicious combinations
Hidden Layer 3: Context understanding
   ↓
Output: Spam probability (0.85 = 85% spam)
```

---

### Key Concepts

#### 1. Neurons (Nodes)

**What they are:** Basic processing units

**What they do:**
```
1. Receive inputs (numbers)
2. Multiply by weights (importance)
3. Add bias (adjustment)
4. Apply activation function (non-linearity)
5. Output result
```

**Simple Example:**
```
Inputs: [0.5, 0.8, 0.3]
Weights: [0.2, 0.5, 0.1]

Calculation:
= (0.5 × 0.2) + (0.8 × 0.5) + (0.3 × 0.1)
= 0.1 + 0.4 + 0.03
= 0.53
```

---

#### 2. Weights

**What they are:** Numbers representing connection strength between neurons

**Real-Life Analogy:**
When deciding if an email is spam:
- Word "free" → High weight (strong spam indicator)
- Word "meeting" → Low weight (neutral)
- Word "prize" → High weight (strong spam indicator)

---

#### 3. Activation Functions

**What they do:** Introduce non-linearity (allow network to learn complex patterns)

**Common Functions:**

**ReLU (Rectified Linear Unit):**
```
f(x) = max(0, x)

Examples:
f(-5) = 0
f(0) = 0
f(5) = 5
```

**Sigmoid:**
```
f(x) = 1 / (1 + e^(-x))

Output: Always between 0 and 1
Use case: Binary classification (spam/not spam)
```

**Softmax:**
```
Converts numbers to probabilities (sum = 1)

Input: [2.0, 1.0, 0.5]
Output: [0.66, 0.24, 0.10]

Use case: Multi-class classification (news category)
```

---

### Deep Learning

**Definition:** Neural networks with many layers (deep = many layers)

```
Shallow Network:
Input → Hidden Layer → Output (2-3 layers)

Deep Network:
Input → Hidden → Hidden → Hidden → ... → Output (10-100+ layers)
```

**Why deeper is better:**
- Early layers: Learn simple patterns (edges, colors)
- Middle layers: Learn combinations (shapes, textures)
- Late layers: Learn complex concepts (objects, faces)

---

## The Transformer Architecture

### Why Transformers?

**Before Transformers (RNNs/LSTMs):**

**Problem 1: Sequential Processing**
```
Text: "The cat sat on the mat"

RNN processes:
Step 1: "The"
Step 2: "cat" (while remembering "The")
Step 3: "sat" (while remembering "The cat")
...

Issues:
- Slow (can't parallelize)
- Forgets early words in long sentences
- Hard to connect distant words
```

---

**Problem 2: Long-Range Dependencies**

```
Text: "The trophy doesn't fit in the suitcase because IT is too big"

Question: What does "IT" refer to?
- Trophy? (IT is too big → trophy is too big ✓)
- Suitcase? (doesn't make sense)

RNN struggles: "IT" is far from "trophy" and "suitcase"
```

---

### Transformer Solution

**Key Innovation: Self-Attention Mechanism**

**Allows model to:**
1. Process all words simultaneously (parallel)
2. Connect any word to any other word (regardless of distance)
3. Decide which words are important for context

---

### Transformer Architecture Overview

```
┌─────────────────────────────────────┐
│         INPUT TEXT                  │
│   "The cat sat on the mat"         │
└─────────────┬───────────────────────┘
              ↓
┌─────────────────────────────────────┐
│      TOKENIZATION                   │
│   ["The", "cat", "sat", "on", ...] │
└─────────────┬───────────────────────┘
              ↓
┌─────────────────────────────────────┐
│      TOKEN EMBEDDINGS               │
│   Convert words to vectors          │
└─────────────┬───────────────────────┘
              ↓
┌─────────────────────────────────────┐
│   POSITIONAL ENCODING               │
│   Add position information          │
└─────────────┬───────────────────────┘
              ↓
┌─────────────────────────────────────┐
│   TRANSFORMER BLOCKS (repeat N×)    │
│                                     │
│   ┌──────────────────────────┐    │
│   │  Multi-Head Attention    │    │
│   └──────────┬───────────────┘    │
│              ↓                      │
│   ┌──────────────────────────┐    │
│   │  Feed Forward Network    │    │
│   └──────────────────────────┘    │
└─────────────┬───────────────────────┘
              ↓
┌─────────────────────────────────────┐
│      OUTPUT LAYER                   │
│   Predict next token probabilities  │
└─────────────────────────────────────┘
```

---

### Components Explained

#### 1. Tokenization

**What it does:** Breaks text into smaller units (tokens)

**Example:**
```
Text: "ChatGPT is amazing!"

Tokens:
["Chat", "G", "PT", " is", " amazing", "!"]

Token IDs:
[5158, 38, 11571, 318, 4998, 0]
```

**Why needed:** Neural networks work with numbers, not text.

---

#### 2. Token Embeddings

**What it does:** Converts tokens to vectors (dense numerical representations)

**Example:**
```
Token: "cat"
Embedding: [0.2, -0.5, 0.8, 0.1, ..., -0.3]  (768 dimensions)

Token: "dog"
Embedding: [0.19, -0.48, 0.79, 0.09, ..., -0.29]  (similar to "cat")

Token: "car"
Embedding: [-0.7, 0.3, -0.1, 0.5, ..., 0.8]  (different from "cat")
```

**Why important:**
- Similar words have similar vectors
- Model can understand relationships
- Example: king - man + woman ≈ queen

---

#### 3. Positional Encoding

**Problem:** Transformers process all words simultaneously, losing word order.

```
"Dog bites man" vs "Man bites dog" → Same words, different meaning!
```

**Solution:** Add position information to embeddings.

**How it works:**
```
Token: "cat" at position 2
Original embedding: [0.2, -0.5, 0.8, ...]
Positional encoding: [0.01, 0.02, -0.01, ...]
Final: [0.21, -0.48, 0.79, ...]  (embedding + position)
```

**Formula (sine/cosine):**
```python
pos = 2  # Word position
d = 768  # Embedding dimension

for i in range(d):
    if i % 2 == 0:
        PE[pos][i] = sin(pos / 10000^(i/d))
    else:
        PE[pos][i] = cos(pos / 10000^((i-1)/d))
```

---

## Attention Mechanism Explained

### What is Attention?

**Definition:** Mechanism that allows model to "focus" on relevant parts of input when processing each word.

### Real-Life Analogy

**Reading a Book:**

When you read: "The trophy doesn't fit in the suitcase because **it** is too big"

Your brain automatically:
1. Sees "it"
2. Looks back to find what "it" refers to
3. Pays attention to "trophy" and "suitcase"
4. Determines "it" = "trophy" (makes sense in context)

**Attention does the same thing automatically!**

---

### How Attention Works (Simplified)

**Goal:** For each word, figure out which other words are important.

**Process:**

```
Text: "The cat sat on the mat"

For word "sat", attention computes:
- How relevant is "The" to "sat"? → 0.05
- How relevant is "cat" to "sat"? → 0.8 (subject of action!)
- How relevant is "sat" to "sat"? → 0.1 (itself)
- How relevant is "on" to "sat"? → 0.05
- How relevant is "the" to "sat"? → 0.0
- How relevant is "mat" to "sat"? → 0.0

Result: When processing "sat", pay most attention to "cat"
```

---

### Attention Formula (Mathematics)

Don't worry if this looks complex - we'll break it down!

**Formula:**
```
Attention(Q, K, V) = softmax(Q × K^T / √d_k) × V

Where:
Q = Query (what I'm looking for)
K = Key (what information is available)
V = Value (the actual information)
d_k = dimension of key (for scaling)
```

**Real-World Analogy: Library Search**

```
Q (Query): "Books about Python programming"
K (Keys): Book titles on shelves
V (Values): Actual book contents

Process:
1. Compare query with all keys (titles)
2. Find best matches (attention scores)
3. Retrieve corresponding values (book contents)
```

---

### Step-by-Step Example

**Text:** "The cat sat"

**Step 1: Create Q, K, V matrices**

```
For each word, create three vectors:
- Query (Q): What am I looking for?
- Key (K): What do I represent?
- Value (V): What information do I contain?

Word "cat":
Q_cat = [0.5, 0.3, 0.8]  (learned during training)
K_cat = [0.4, 0.6, 0.2]
V_cat = [0.9, 0.1, 0.7]
```

---

**Step 2: Compute attention scores**

For word "sat" looking at all words:

```
Score("sat" → "The") = Q_sat · K_The = 0.2
Score("sat" → "cat") = Q_sat · K_cat = 0.8
Score("sat" → "sat") = Q_sat · K_sat = 0.5

(Dot product measures similarity)
```

---

**Step 3: Apply softmax (convert to probabilities)**

```
Raw scores: [0.2, 0.8, 0.5]
After softmax: [0.15, 0.55, 0.30]  (sum = 1.0)

Interpretation:
- 15% attention to "The"
- 55% attention to "cat" (most relevant!)
- 30% attention to "sat" (itself)
```

---

**Step 4: Compute weighted sum of values**

```
Output_sat = 0.15 × V_The + 0.55 × V_cat + 0.30 × V_sat

This output now contains context from all relevant words!
```

---

### Multi-Head Attention

**Problem:** Single attention might miss different types of relationships.

**Solution:** Use multiple attention "heads" (8, 12, or 16 typically)

**Example:**

```
Text: "The bank is near the river"

Head 1: Focuses on syntax
- "bank" attends to "is" (verb agreement)

Head 2: Focuses on semantics
- "bank" attends to "river" (river bank, not financial bank!)

Head 3: Focuses on position
- "bank" attends to nearby words

Final output: Combines all heads → Rich understanding
```

---

### Self-Attention vs Cross-Attention

#### Self-Attention (GPT)
**What it does:** Each word attends to other words in the SAME sequence

```
Input: "The cat sat on the mat"
Each word looks at: All other words in same sentence
```

**Use case:** Language models (GPT, BERT)

---

#### Cross-Attention (Seq2Seq, T5)
**What it does:** Words in output attend to words in INPUT

```
Translation task:
Input (English): "Hello, how are you?"
Output (French): "Bonjour, comment allez-vous?"

When generating "Bonjour", attend to "Hello"
When generating "comment", attend to "how"
```

**Use case:** Translation, summarization

---

## How LLMs are Trained

### Training Phases

```
Phase 1: Pre-training (Unsupervised)
├─ Train on massive text corpus
├─ Learn language patterns
├─ Predict next word (self-supervised)
└─ Cost: Millions of dollars

Phase 2: Fine-tuning (Supervised)
├─ Train on specific tasks
├─ Instruction following
├─ RLHF (human feedback)
└─ Cost: Thousands of dollars

Phase 3: Deployment
├─ Inference optimization
├─ API serving
└─ Monitoring
```

---

### Phase 1: Pre-Training

**Objective:** Learn general language understanding

**Method:** Next Token Prediction (Causal Language Modeling)

**How it works:**

```
Training Text: "The cat sat on the mat"

Model learns to predict:
Input: "The" → Predict: "cat"
Input: "The cat" → Predict: "sat"
Input: "The cat sat" → Predict: "on"
...

Repeats billions of times with different texts
```

---

**Training Data Sources:**

```
1. Web Pages (Common Crawl)
   - News articles
   - Blogs
   - Wikipedia
   - Forums

2. Books
   - Fiction and non-fiction
   - Technical manuals
   - Academic texts

3. Code Repositories (GitHub)
   - For code-capable models like Copilot

4. Academic Papers
   - ArXiv, PubMed
   - Research articles

Total: ~1 trillion tokens (750 GB+ of text)
```

---

**Training Process:**

```python
# Simplified training loop

for epoch in range(epochs):
    for batch in training_data:
        # Forward pass
        predictions = model(batch['input'])

        # Calculate loss (how wrong was prediction?)
        loss = calculate_loss(predictions, batch['target'])

        # Backward pass (update weights)
        loss.backward()
        optimizer.step()

        # Repeat millions of times
```

**Real Training Stats (GPT-3):**
- Training tokens: 300 billion
- Training time: Several months
- GPUs: 10,000+
- Cost: ~$4-12 million
- Compute: 314 zetaflops (10^21 operations)

---

### Phase 2: Fine-Tuning

#### A. Instruction Tuning

**Goal:** Teach model to follow instructions

**Training Data:**

```
Example 1:
Instruction: "Summarize this article in 3 sentences"
Input: [Long article]
Output: [3-sentence summary]

Example 2:
Instruction: "Translate to French"
Input: "Hello, how are you?"
Output: "Bonjour, comment allez-vous?"

Example 3:
Instruction: "Extract names from this text"
Input: "John and Mary went to Paris"
Output: ["John", "Mary", "Paris"]
```

**Dataset Size:** 10K - 100K examples (much smaller than pre-training)

---

#### B. RLHF (Reinforcement Learning from Human Feedback)

**Why needed:** Model might generate harmful/incorrect/unhelpful content

**Process:**

```
Step 1: Generate Multiple Responses
User: "How to make a cake?"

Model generates 4 responses:
A) Detailed recipe with steps
B) "I don't know"
C) Nonsense text
D) Harmful instructions (poison)
```

```
Step 2: Human Ranking
Humans rank: A > B > C > D

(A is most helpful, D is harmful)
```

```
Step 3: Train Reward Model
Learn to predict human preferences
Input: Response
Output: Score (0-1)

A → 0.95
B → 0.40
C → 0.10
D → 0.01
```

```
Step 4: Optimize Policy
Train model to maximize reward
- Generate responses
- Get reward scores
- Update to generate higher-scoring responses
- Repeat
```

**Result:** Model becomes more helpful, harmless, and honest

---

### Training Loss Functions

#### Cross-Entropy Loss (Pre-Training)

**What it measures:** How different model's prediction is from actual next token

**Example:**

```
Actual next token: "cat" (token ID: 2034)

Model predictions:
"cat" → 0.7 probability
"dog" → 0.2 probability
"car" → 0.1 probability

Cross-entropy loss:
Loss = -log(0.7) = 0.36  (lower is better)

If model predicted:
"cat" → 0.1 probability
Loss = -log(0.1) = 2.3  (higher = worse)
```

**Goal:** Minimize loss → Better predictions

---

### Optimization Algorithms

#### Adam Optimizer (Most Common)

**What it does:** Adjusts learning rate dynamically for each parameter

**Key features:**
- Momentum: Uses past gradients (smoother updates)
- Adaptive learning: Different learning rates for different parameters
- Efficient: Converges faster than basic SGD

```python
# Simplified Adam update
m = beta1 * m + (1 - beta1) * gradient  # Momentum
v = beta2 * v + (1 - beta2) * gradient^2  # Variance
weight = weight - learning_rate * m / sqrt(v)
```

---

## Model Architectures

### 1. GPT (Generative Pre-trained Transformer)

**Architecture:** Decoder-only transformer

**How it works:**
```
Input: "The cat sat on"
Process: Each token sees only PREVIOUS tokens (causal attention)
Output: "the" (next token prediction)
```

**Key Feature:** Causal (left-to-right) attention
```
"The" can't see "cat", "sat", "on" (future tokens masked)
"cat" can see "The" but not "sat", "on"
"sat" can see "The", "cat" but not "on"
```

**Use cases:**
- Text generation
- Chatbots (ChatGPT)
- Code completion

**Variants:**
- GPT-1: 117M parameters (2018)
- GPT-2: 1.5B parameters (2019)
- GPT-3: 175B parameters (2020)
- GPT-4: 1T+ parameters (2023) [estimated]

---

### 2. BERT (Bidirectional Encoder Representations from Transformers)

**Architecture:** Encoder-only transformer

**How it works:**
```
Input: "The cat sat on the mat"
Process: Each token sees ALL tokens (bidirectional attention)
Output: Contextual embeddings for each token
```

**Key Feature:** Bidirectional attention
```
"cat" can see:
- Left context: "The"
- Right context: "sat on the mat"

Result: Richer understanding
```

**Training Method:** Masked Language Modeling (MLM)

```
Original: "The cat sat on the mat"
Masked: "The [MASK] sat on the mat"
Task: Predict "cat"

Also: "The cat [MASK] on the mat"
Task: Predict "sat"
```

**Use cases:**
- Text classification
- Named entity recognition
- Question answering
- Sentence embeddings

**Not good for:** Text generation (not trained for it)

---

### 3. T5 (Text-to-Text Transfer Transformer)

**Architecture:** Encoder-Decoder transformer

**Key Idea:** Convert ALL tasks to text-to-text format

**Examples:**

```
Translation:
Input: "translate English to French: Hello"
Output: "Bonjour"

Summarization:
Input: "summarize: [long article]"
Output: "[summary]"

Classification:
Input: "sentiment: I love this product"
Output: "positive"

Question Answering:
Input: "question: What is the capital? context: France's capital is Paris"
Output: "Paris"
```

**Benefits:**
- Unified architecture for all tasks
- Transfer learning across tasks
- Flexible for any text task

---

### Architecture Comparison

| Model | Type | Attention | Best For |
|-------|------|-----------|----------|
| **GPT** | Decoder-only | Causal (left-to-right) | Text generation, chatbots |
| **BERT** | Encoder-only | Bidirectional | Classification, NER, embeddings |
| **T5** | Encoder-Decoder | Both | Versatile (all text tasks) |

---

### Popular Models by Architecture

**GPT Family:**
- GPT-3, GPT-4 (OpenAI)
- Claude (Anthropic)
- PaLM, Gemini (Google)
- LLaMA (Meta)

**BERT Family:**
- BERT, RoBERTa
- DistilBERT (smaller, faster)
- ALBERT (parameter sharing)

**T5 Family:**
- T5, Flan-T5
- BART (Facebook)
- MT5 (multilingual T5)

---

## Scaling Laws

### What are Scaling Laws?

**Discovery:** Model performance improves predictably with scale

**Key Factors:**
1. **Model Size** (parameters)
2. **Dataset Size** (training tokens)
3. **Compute** (training time × GPU power)

---

### The Power Law Relationship

**Formula:**
```
Loss ∝ 1 / (N^α)

Where:
N = Number of parameters
α ≈ 0.076 (empirically determined)
```

**Translation:**
- 10× more parameters → Consistent improvement in performance
- 100× more parameters → Even better (but diminishing returns)

---

### Scaling Observations

**Model Size:**
```
100M params → Can form sentences
1B params → Basic reasoning
10B params → Good at many tasks
100B params → Strong performance (GPT-3)
1T+ params → State-of-the-art (GPT-4)
```

**Training Data:**
```
1M tokens → Learns basic patterns
100M tokens → Learns grammar
10B tokens → Learns facts
100B tokens → Learns reasoning
1T+ tokens → Emergent abilities
```

---

### Emergent Abilities

**Definition:** Capabilities that suddenly appear at certain scales

**Examples:**

**Small Models (1B params):**
- Basic Q&A
- Simple classification

**Large Models (100B+ params):**
- Chain-of-thought reasoning
- Few-shot learning
- Code generation
- Multi-step problem solving
- Arithmetic

**Example - Math Reasoning:**

```
Problem: "John has 5 apples. He gives 2 to Mary. How many does he have?"

Small model: "3" (memorized pattern)

Large model:
"Let me think step by step:
1. John starts with 5 apples
2. He gives away 2 apples
3. 5 - 2 = 3
Therefore, John has 3 apples left."
```

---

### Cost vs Performance Trade-off

**Training Cost:**

| Model | Parameters | Training Cost | Training Time |
|-------|------------|---------------|---------------|
| GPT-2 | 1.5B | $50K | 1 week |
| GPT-3 | 175B | $4-12M | 1-3 months |
| GPT-4 | 1T+ (est.) | $50-100M | Several months |

**Inference Cost (per 1M tokens):**

| Model | Cost |
|-------|------|
| GPT-3.5 | $0.50 |
| GPT-4 (8K) | $30.00 |
| GPT-4 (32K) | $60.00 |
| Claude 2 (100K) | $8.00 |

**Lesson:** Bigger isn't always better for production (cost, latency)

---

## Summary

### Key Takeaways:

✅ **Neural Networks:** Learn patterns through layers of neurons
✅ **Transformers:** Use attention mechanism to process text in parallel
✅ **Attention:** Allows model to focus on relevant parts of input
✅ **Training:** Pre-training (general) + Fine-tuning (specific)
✅ **Architectures:**
  - GPT: Text generation
  - BERT: Understanding/classification
  - T5: Versatile text-to-text
✅ **Scaling Laws:** Bigger models → Better performance (but costly)

### What's Next?

**Chapter 03:** Prompt Engineering
- Writing effective prompts
- Few-shot learning
- Chain-of-thought prompting
- Prompt optimization techniques
- Real-world examples

---

**Continue to Chapter 03 to learn how to get the best results from LLMs! 🚀**
