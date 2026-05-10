# Chapter 04: RAG (Retrieval Augmented Generation)

**Building Intelligent Systems that Ground LLMs in Real Data**

---

## Table of Contents
1. [What is RAG?](#what-is-rag)
2. [Why RAG is Needed](#why-rag-is-needed)
3. [RAG Architecture](#rag-architecture)
4. [Building a RAG System](#building-a-rag-system)
5. [Chunking Strategies](#chunking-strategies)
6. [Retrieval Methods](#retrieval-methods)
7. [Evaluation](#evaluation)

---

## What is RAG?

### Simple Definition

**RAG (Retrieval Augmented Generation)** = Search + LLM

Instead of relying solely on LLM's training data, **RAG retrieves relevant information first**, then uses it to generate accurate answers.

---

### Real-Life Analogy

**WITHOUT RAG:**
```
You: Ask a friend (LLM) about company's vacation policy
Friend: Answers from memory (might be outdated/wrong)
Problem: Hallucination risk!
```

**WITH RAG:**
```
You: Ask a friend about vacation policy
Friend: First checks the employee handbook (retrieval)
        Then answers based on what handbook says (generation)
Result: Accurate, up-to-date answer with source!
```

---

## Why RAG is Needed

### Problem 1: Knowledge Cutoff

**LLM Training:**
```
GPT-4 training cutoff: April 2023
User in Jan 2024: "What happened in the 2024 Olympics?"
LLM: "I don't have information after April 2023"
```

**RAG Solution:**
```
1. Search latest news about 2024 Olympics
2. Feed results to LLM
3. LLM generates answer using current information
```

---

### Problem 2: Hallucinations

**Without RAG:**
```
User: "What is our company's return policy?"
LLM: Makes up plausible-sounding policy (completely wrong!)
```

**With RAG:**
```
1. Search company knowledge base
2. Retrieve actual return policy document
3. LLM answers based on retrieved document
4. Includes citation to source
```

---

### Problem 3: Domain-Specific Knowledge

**Without RAG:**
```
User: "What's in our internal database schema documentation?"
LLM: Doesn't know (wasn't in training data)
```

**With RAG:**
```
1. Search internal documentation
2. Find schema docs
3. Generate answer with specific table/column details
```

---

### RAG vs Fine-Tuning

| Aspect | RAG | Fine-Tuning |
|--------|-----|-------------|
| **Update knowledge** | Real-time (add new docs) | Requires retraining |
| **Cost** | Low (just storage + embeddings) | High ($$$) |
| **Time to deploy** | Hours | Weeks |
| **Sources/citations** | Yes ✓ | No |
| **When to use** | Frequently changing data | Specific behavior/style |

**Example:**
- RAG: Company documentation Q&A
- Fine-tuning: Medical diagnosis assistant

---

## RAG Architecture

### High-Level Flow

```
┌──────────────────────────────────────────┐
│  1. INDEXING PHASE (One-time setup)     │
└──────────────┬───────────────────────────┘
               ↓
    ┌─────────────────────┐
    │   Knowledge Base    │
    │  (docs, databases)  │
    └──────┬──────────────┘
           ↓
    ┌─────────────────────┐
    │  Chunk Documents    │
    │  (split into pieces)│
    └──────┬──────────────┘
           ↓
    ┌─────────────────────┐
    │ Generate Embeddings │
    │  (convert to vectors)│
    └──────┬──────────────┘
           ↓
    ┌─────────────────────┐
    │  Store in Vector DB │
    │  (Pinecone, Weaviate)│
    └─────────────────────┘

┌──────────────────────────────────────────┐
│  2. QUERY PHASE (Runtime)               │
└──────────────┬───────────────────────────┘
               ↓
    ┌─────────────────────┐
    │   User Question     │
    └──────┬──────────────┘
           ↓
    ┌─────────────────────┐
    │ Convert to Embedding│
    └──────┬──────────────┘
           ↓
    ┌─────────────────────┐
    │  Search Vector DB   │
    │ (find similar chunks)│
    └──────┬──────────────┘
           ↓
    ┌─────────────────────┐
    │ Retrieve Top K Docs │
    │     (e.g. top 5)    │
    └──────┬──────────────┘
           ↓
    ┌─────────────────────┐
    │ Create Prompt with: │
    │ - User question     │
    │ - Retrieved context │
    └──────┬──────────────┘
           ↓
    ┌─────────────────────┐
    │   Send to LLM       │
    └──────┬──────────────┘
           ↓
    ┌─────────────────────┐
    │  Generated Answer   │
    │  (with sources)     │
    └─────────────────────┘
```

---

## Building a RAG System

### Step 1: Prepare Knowledge Base

**Example: Company Documentation**

```python
documents = [
    {
        "id": "doc1",
        "title": "Data Pipeline Guidelines",
        "content": "All data pipelines must run hourly...",
        "metadata": {"department": "engineering", "updated": "2024-01-15"}
    },
    {
        "id": "doc2",
        "title": "SQL Best Practices",
        "content": "Always use LIMIT when testing queries...",
        "metadata": {"department": "engineering", "updated": "2024-02-01"}
    }
]
```

---

### Step 2: Chunk Documents

**Why chunk?**
- Documents too long for context window
- Better retrieval precision
- More focused answers

**Example:**

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,  # characters
    chunk_overlap=50  # overlap between chunks
)

chunks = text_splitter.split_documents(documents)

# Result:
# [
#   "All data pipelines must run hourly and include error handling...",
#   "When writing SQL queries, always use LIMIT for testing...",
#   ...
# ]
```

---

### Step 3: Generate Embeddings

**Convert text to vectors:**

```python
from openai import OpenAI
client = OpenAI()

def get_embedding(text):
    response = client.embeddings.create(
        model="text-embedding-ada-002",
        input=text
    )
    return response.data[0].embedding

# Generate embeddings for all chunks
for chunk in chunks:
    chunk['embedding'] = get_embedding(chunk['content'])

# Example output:
# chunk['embedding'] = [0.002, -0.015, 0.032, ..., -0.008]  # 1536 dimensions
```

---

### Step 4: Store in Vector Database

**Using Pinecone:**

```python
import pinecone

# Initialize
pinecone.init(api_key="your-api-key")
index = pinecone.Index("company-docs")

# Upsert (insert/update) vectors
index.upsert(vectors=[
    {
        "id": "chunk1",
        "values": embedding1,  # Vector
        "metadata": {"text": "...", "source": "doc1"}
    },
    {
        "id": "chunk2",
        "values": embedding2,
        "metadata": {"text": "...", "source": "doc2"}
    }
])
```

---

### Step 5: Query-Time Retrieval

**User asks question:**

```python
user_question = "How often should data pipelines run?"

# 1. Convert question to embedding
question_embedding = get_embedding(user_question)

# 2. Search vector database
results = index.query(
    vector=question_embedding,
    top_k=3,  # Return top 3 most similar
    include_metadata=True
)

# 3. Extract relevant text
context_chunks = [r['metadata']['text'] for r in results['matches']]
```

---

### Step 6: Generate Answer with LLM

**Create prompt with context:**

```python
context = "\n\n".join(context_chunks)

prompt = f"""
Answer the question based on the context below. If the answer is not
in the context, say "I don't have that information."

Context:
{context}

Question: {user_question}

Answer:
"""

# Call LLM
response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": prompt}]
)

answer = response.choices[0].message.content
```

---

### Complete RAG Example (Python)

```python
from openai import OpenAI
import pinecone

client = OpenAI()

def rag_query(question):
    # 1. Embed question
    q_embedding = client.embeddings.create(
        model="text-embedding-ada-002",
        input=question
    ).data[0].embedding
    
    # 2. Search vector DB
    index = pinecone.Index("company-docs")
    results = index.query(
        vector=q_embedding,
        top_k=3,
        include_metadata=True
    )
    
    # 3. Build context
    context = "\n\n".join([r['metadata']['text'] for r in results['matches']])
    
    # 4. Generate answer
    prompt = f"""
    Use the context below to answer the question.
    
    Context:
    {context}
    
    Question: {question}
    
    Answer:
    """
    
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}]
    )
    
    return {
        "answer": response.choices[0].message.content,
        "sources": [r['metadata']['source'] for r in results['matches']]
    }

# Usage
result = rag_query("How often should pipelines run?")
print(result['answer'])
print("Sources:", result['sources'])
```

---

## Chunking Strategies

### Fixed-Size Chunking

**Method:** Split by character count

```python
chunk_size = 500
chunks = [text[i:i+chunk_size] for i in range(0, len(text), chunk_size)]
```

**Pros:** Simple, predictable size
**Cons:** May split mid-sentence

---

### Semantic Chunking

**Method:** Split by meaning (paragraphs, sections)

```python
chunks = text.split('\n\n')  # Split by paragraph
```

**Pros:** Preserves context
**Cons:** Variable chunk sizes

---

### Recursive Chunking

**Method:** Try multiple delimiters in order

```python
# Try: \n\n → \n → . → space
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
    separators=["\n\n", "\n", ". ", " ", ""]
)
```

**Pros:** Balance between semantic and size
**Cons:** More complex

---

### Best Practices

✅ **Chunk size:** 300-1000 characters (balance precision vs context)
✅ **Overlap:** 10-20% (ensure continuity)
✅ **Preserve structure:** Don't split mid-sentence
✅ **Include metadata:** Title, section, page number

---

## Retrieval Methods

### 1. Dense Retrieval (Most Common)

**How it works:**
- Convert text to dense vectors (embeddings)
- Find nearest neighbors using cosine similarity

**Pros:** Understands semantics
**Cons:** Computationally expensive

---

### 2. Sparse Retrieval (BM25, TF-IDF)

**How it works:**
- Keyword matching
- Statistical relevance

**Pros:** Fast, good for exact matches
**Cons:** Misses synonyms/paraphrases

---

### 3. Hybrid Retrieval

**Combine both:**

```python
# Dense retrieval score
dense_score = cosine_similarity(q_embedding, doc_embedding)

# Sparse retrieval score
sparse_score = bm25(query, document)

# Final score
final_score = 0.7 * dense_score + 0.3 * sparse_score
```

**Best of both worlds!**

---

## Evaluation

### Metrics

**1. Retrieval Accuracy:**
- Are relevant documents retrieved?
- Metrics: Precision@K, Recall@K, MRR

**2. Answer Quality:**
- Is answer correct?
- Metrics: Human evaluation, answer relevance

**3. Latency:**
- How fast?
- Target: < 2 seconds end-to-end

---

### Testing RAG Systems

```python
# Test cases
test_cases = [
    {
        "question": "What is the return policy?",
        "expected_source": "policy-doc-123",
        "expected_answer_contains": "30 days"
    },
    # More test cases...
]

for test in test_cases:
    result = rag_query(test['question'])
    
    # Check source retrieval
    assert test['expected_source'] in result['sources']
    
    # Check answer quality
    assert test['expected_answer_contains'] in result['answer'].lower()
```

---

## Data Engineering RAG Use Cases

### 1. SQL Query Assistant

```
User: "Show me customers who haven't ordered in 90 days"

RAG:
1. Retrieves schema documentation
2. Finds relevant tables (customers, orders)
3. Generates SQL query with proper joins
```

---

### 2. Data Pipeline Documentation

```
User: "How does the daily sales ETL work?"

RAG:
1. Retrieves Airflow DAG code
2. Finds related documentation
3. Explains pipeline steps, dependencies, schedule
```

---

### 3. Incident Troubleshooting

```
User: "Pipeline failed with error 'Connection timeout'"

RAG:
1. Searches past incidents
2. Finds similar errors and resolutions
3. Suggests fixes based on historical data
```

---

## Summary

### Key Takeaways:

✅ **RAG** = Retrieval + Generation (search before answering)
✅ **Why RAG:** Reduces hallucinations, adds current knowledge, provides sources
✅ **Components:** Documents → Chunks → Embeddings → Vector DB → Retrieval → LLM
✅ **Chunking:** 300-1000 chars, overlap 10-20%
✅ **Evaluation:** Test retrieval accuracy and answer quality

---

**Continue to Chapter 05 to learn about Vector Embeddings in detail! 🚀**
