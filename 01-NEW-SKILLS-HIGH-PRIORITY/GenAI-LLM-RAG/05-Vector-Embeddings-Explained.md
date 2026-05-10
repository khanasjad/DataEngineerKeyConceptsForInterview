# Chapter 05: Vector Embeddings Explained

**Understanding the Mathematics Behind Semantic Search**

---

## What are Embeddings?

### Simple Definition

**Embeddings** are numerical representations (vectors) of text that capture meaning and relationships.

### Real-Life Analogy

**GPS Coordinates:**
- New York: [40.7128° N, 74.0060° W]
- Boston: [42.3601° N, 71.0589° W]
- Los Angeles: [34.0522° N, 118.2437° W]

**Observations:**
- New York and Boston are **close** (similar coordinates)
- LA is **far** from both (different coordinates)
- We can calculate distance mathematically

**Similarly, embeddings:**
```
"cat" → [0.2, -0.5, 0.8, ..., 0.1]
"dog" → [0.19, -0.48, 0.79, ..., 0.09]  (similar to "cat")
"car" → [-0.7, 0.3, -0.1, ..., 0.8]     (different from "cat"/"dog")
```

---

## Why Embeddings?

### Traditional Search (Keyword Matching)

```
Query: "python programming"
Documents:
- Doc1: "Learn Python programming"  ✓ Match
- Doc2: "Python tutorial for beginners"  ✓ Match
- Doc3: "Coding in Python language"  ✗ No match (doesn't contain "programming")
```

**Problem:** Misses synonyms, paraphrases, related concepts

---

### Semantic Search (Embeddings)

```
Query: "python programming"
Query embedding: [0.5, -0.2, 0.8, ...]

Documents:
- Doc1: "Learn Python programming"
  Embedding: [0.51, -0.19, 0.81, ...]  → Similarity: 0.98 ✓

- Doc2: "Python tutorial for beginners"
  Embedding: [0.48, -0.22, 0.79, ...]  → Similarity: 0.95 ✓

- Doc3: "Coding in Python language"
  Embedding: [0.49, -0.20, 0.80, ...]  → Similarity: 0.93 ✓ (Found!)
```

**Benefit:** Understands meaning, not just keywords

---

## How Embeddings are Generated

### Step 1: Training an Embedding Model

**Objective:** Learn to represent text as vectors where similar meanings = similar vectors

**Training Data:** Massive text corpus (books, Wikipedia, web pages)

**Training Method (Simplified):**

```
1. Take sentence: "The cat sat on the mat"
2. Predict: Given "cat", what words appear nearby?
   - Likely: "sat", "mat" (actually appear)
   - Unlikely: "car", "computer"
   
3. Adjust model weights to:
   - Put "cat" and "sat" closer in vector space
   - Put "cat" and "car" farther apart
   
4. Repeat billions of times with different texts
```

**Result:** Model learns:
- "cat" and "dog" are similar (both animals)
- "king" and "queen" are similar (both royalty)
- "Paris" and "France" are related (capital-country)

---

### Step 2: Using the Trained Model

**Generate embedding for any text:**

```python
from openai import OpenAI
client = OpenAI()

def get_embedding(text):
    response = client.embeddings.create(
        model="text-embedding-ada-002",
        input=text
    )
    return response.data[0].embedding

# Example
embedding_cat = get_embedding("cat")
# Result: [0.002, -0.015, 0.032, ..., -0.008]  (1536 numbers)

embedding_dog = get_embedding("dog")
# Result: [0.001, -0.014, 0.031, ..., -0.007]  (similar to "cat")

embedding_car = get_embedding("car")
# Result: [-0.025, 0.041, -0.012, ..., 0.033]  (different from "cat"/"dog")
```

---

## Similarity Metrics

### How to Measure "Closeness"?

**Three common metrics:**

### 1. Cosine Similarity

**What it measures:** Angle between vectors (direction, not magnitude)

**Formula:**
```
cosine_similarity = (A · B) / (||A|| × ||B||)

Range: -1 to 1
- 1 = Identical direction
- 0 = Perpendicular (unrelated)
- -1 = Opposite direction
```

**Example:**

```python
import numpy as np

def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

vec_cat = np.array([0.2, -0.5, 0.8])
vec_dog = np.array([0.19, -0.48, 0.79])
vec_car = np.array([-0.7, 0.3, -0.1])

print(cosine_similarity(vec_cat, vec_dog))  # 0.99 (very similar!)
print(cosine_similarity(vec_cat, vec_car))  # -0.15 (different)
```

**When to use:** Most common for text embeddings (OpenAI, Sentence Transformers)

---

### 2. Euclidean Distance

**What it measures:** Straight-line distance between points

**Formula:**
```
euclidean_distance = sqrt((a1-b1)² + (a2-b2)² + ... + (an-bn)²)

Range: 0 to ∞
- 0 = Identical
- Larger = More different
```

**Example:**

```python
def euclidean_distance(a, b):
    return np.linalg.norm(a - b)

print(euclidean_distance(vec_cat, vec_dog))  # 0.05 (close)
print(euclidean_distance(vec_cat, vec_car))  # 1.52 (far)
```

**When to use:** Image embeddings, when magnitude matters

---

### 3. Dot Product

**What it measures:** Combined magnitude and direction

**Formula:**
```
dot_product = a1×b1 + a2×b2 + ... + an×bn

Range: -∞ to ∞
- Positive = Similar direction
- Negative = Opposite direction
- Magnitude matters (longer vectors score higher)
```

**When to use:** Fast approximation, when vectors are normalized

---

## Properties of Embeddings

### Property 1: Semantic Similarity

**Similar meaning → Similar vectors**

```
get_embedding("happy") ≈ get_embedding("joyful")
get_embedding("sad") ≈ get_embedding("unhappy")
```

---

### Property 2: Arithmetic Operations

**Vector algebra reveals relationships!**

```
king - man + woman ≈ queen

Paris - France + Italy ≈ Rome

walking - walk + swim ≈ swimming
```

**How it works:**

```python
vec_king = get_embedding("king")
vec_man = get_embedding("man")
vec_woman = get_embedding("woman")

result_vec = vec_king - vec_man + vec_woman

# Find most similar word to result_vec
# Answer: "queen"
```

---

### Property 3: Clustering

**Group similar concepts together**

```
Animals cluster:
- [dog, cat, lion, tiger] → Close to each other
- [car, truck, bus] → Close to each other
- Animals and Vehicles → Far apart
```

---

## Embedding Models

### 1. OpenAI Embeddings

**Model:** `text-embedding-ada-002`

```python
from openai import OpenAI
client = OpenAI()

response = client.embeddings.create(
    model="text-embedding-ada-002",
    input="Hello world"
)
embedding = response.data[0].embedding  # 1536 dimensions
```

**Pros:** High quality, easy to use
**Cons:** Paid API, requires internet
**Cost:** $0.0001 per 1K tokens

---

### 2. Sentence Transformers (Open-Source)

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')
embedding = model.encode("Hello world")  # 384 dimensions
```

**Popular models:**
- `all-MiniLM-L6-v2`: Fast, small (384 dim)
- `all-mpnet-base-v2`: Better quality (768 dim)
- `multi-qa-mpnet-base-dot-v1`: Optimized for Q&A

**Pros:** Free, runs locally, fast
**Cons:** Slightly lower quality than OpenAI

---

### 3. Cohere Embeddings

```python
import cohere
co = cohere.Client('your-api-key')

response = co.embed(
    texts=["Hello world"],
    model='embed-english-v3.0'
)
embedding = response.embeddings[0]  # 1024 dimensions
```

**Pros:** Multilingual, good quality
**Cons:** Paid API

---

## Use Cases for Embeddings

### 1. Semantic Search

**Find documents similar to query**

```python
# 1. Embed all documents
doc_embeddings = [get_embedding(doc) for doc in documents]

# 2. Embed query
query_embedding = get_embedding("machine learning")

# 3. Find most similar
similarities = [
    cosine_similarity(query_embedding, doc_emb)
    for doc_emb in doc_embeddings
]

# 4. Return top K
top_docs = sorted(zip(documents, similarities), key=lambda x: x[1], reverse=True)[:5]
```

---

### 2. Recommendation Systems

**Recommend similar items**

```python
# User liked "Python for Data Science"
liked_embedding = get_embedding("Python for Data Science")

# Find similar courses
course_embeddings = {course: get_embedding(course) for course in all_courses}

recommendations = sorted(
    course_embeddings.items(),
    key=lambda x: cosine_similarity(liked_embedding, x[1]),
    reverse=True
)[:10]
```

---

### 3. Clustering/Categorization

**Group similar items automatically**

```python
from sklearn.cluster import KMeans

# Embed all articles
article_embeddings = [get_embedding(article) for article in articles]

# Cluster into 5 groups
kmeans = KMeans(n_clusters=5)
clusters = kmeans.fit_predict(article_embeddings)

# Result: Articles grouped by topic
```

---

### 4. Duplicate Detection

**Find near-duplicate text**

```python
threshold = 0.95  # 95% similar

for i, doc1 in enumerate(documents):
    for j, doc2 in enumerate(documents[i+1:]):
        similarity = cosine_similarity(
            get_embedding(doc1),
            get_embedding(doc2)
        )
        
        if similarity > threshold:
            print(f"Potential duplicate: {doc1} <-> {doc2}")
```

---

## Data Engineering Applications

### 1. SQL Query Similarity

**Find similar past queries**

```python
# User writes new query
new_query = "SELECT * FROM orders WHERE date > '2024-01-01'"
new_embedding = get_embedding(new_query)

# Search query history
similar_queries = vector_db.search(new_embedding, top_k=5)

# Show similar past queries with their execution plans
for query, similarity in similar_queries:
    print(f"Similar query ({similarity:.2f}): {query}")
    print(f"Execution time: {query.execution_time}")
    print(f"Optimization tips: {query.tips}")
```

---

### 2. Table/Column Discovery

**Find relevant tables for a task**

```python
# User asks: "I need customer purchase history"
query_embedding = get_embedding("customer purchase history")

# Search metadata
table_descriptions = {
    "customers": "Customer demographic information",
    "orders": "Purchase orders and transactions",
    "order_items": "Individual items in orders"
}

# Find most relevant tables
for table, description in table_descriptions.items():
    desc_embedding = get_embedding(description)
    score = cosine_similarity(query_embedding, desc_embedding)
    print(f"{table}: {score:.2f}")

# Result:
# orders: 0.92 ← Most relevant!
# order_items: 0.88
# customers: 0.75
```

---

### 3. Documentation Search

**Semantic search in technical docs**

```python
# Embed all documentation
docs = [
    "How to set up Airflow DAG scheduling",
    "Debugging failed Spark jobs",
    "Best practices for data partitioning"
]

doc_embeddings = [get_embedding(doc) for doc in docs]

# User query
query = "My pipeline isn't running on schedule"
query_embedding = get_embedding(query)

# Find most relevant docs
scores = [cosine_similarity(query_embedding, emb) for emb in doc_embeddings]

# Result: "How to set up Airflow DAG scheduling" (highest score)
```

---

## Best Practices

### 1. Choose Right Model

**Factors to consider:**
- Quality needed
- Speed requirements
- Cost constraints
- Online vs offline

**Recommendations:**
- **Production search:** OpenAI ada-002 (high quality)
- **Local/fast:** Sentence Transformers MiniLM
- **Multilingual:** Cohere or multilingual BERT

---

### 2. Batch Processing

**Don't embed one-by-one!**

```python
# BAD (slow)
for doc in documents:
    embedding = get_embedding(doc)

# GOOD (fast)
batch_size = 100
for i in range(0, len(documents), batch_size):
    batch = documents[i:i+batch_size]
    embeddings = embed_batch(batch)  # Single API call
```

---

### 3. Caching

**Cache embeddings to avoid re-computation**

```python
import hashlib
import pickle

embedding_cache = {}

def get_embedding_cached(text):
    # Create hash of text
    text_hash = hashlib.md5(text.encode()).hexdigest()
    
    # Check cache
    if text_hash in embedding_cache:
        return embedding_cache[text_hash]
    
    # Generate and cache
    embedding = get_embedding(text)
    embedding_cache[text_hash] = embedding
    return embedding
```

---

### 4. Dimensionality Reduction (Optional)

**Reduce vector size for faster search**

```python
from sklearn.decomposition import PCA

# Original: 1536 dimensions
embeddings = [get_embedding(text) for text in texts]

# Reduce to 256 dimensions
pca = PCA(n_components=256)
reduced_embeddings = pca.fit_transform(embeddings)

# Trade-off: Faster search, slightly lower accuracy
```

---

## Summary

### Key Takeaways:

✅ **Embeddings:** Vectors that capture semantic meaning
✅ **Similarity Metrics:** Cosine (most common), Euclidean, Dot product
✅ **Models:** OpenAI (best quality), Sentence Transformers (free/local), Cohere (multilingual)
✅ **Use Cases:** Semantic search, recommendations, clustering, duplicate detection
✅ **Best Practices:** Batch processing, caching, right model selection

---

**Continue to Chapter 06 for Advanced GenAI Topics! 🚀**
