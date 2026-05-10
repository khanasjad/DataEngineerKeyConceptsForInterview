# Chapter 01: Introduction to Vector Databases

**Specialized Databases for Similarity Search**

## What is a Vector Database?

### Simple Definition

A **Vector Database** is a specialized database optimized for storing and querying vector embeddings (numerical representations of data).

### Real-Life Analogy

**Traditional Database (SQL):**
- Like a filing cabinet organized by exact labels
- Find document by ID: "Get file #12345"
- Fast for exact matches

**Vector Database:**
- Like a library organized by similarity
- Find books similar to one you liked
- Fast for "find similar items"

## Why Vector Databases?

### Problem with Traditional Databases

**Scenario:** Find similar products

**SQL Approach (doesn't work well):**
```sql
SELECT * FROM products
WHERE description LIKE '%lightweight%'
  AND description LIKE '%comfortable%'
  AND description LIKE '%running%';
```

**Problems:**
- Misses synonyms ("featherweight" instead of "lightweight")
- Misses semantic similarity
- Keyword matching only

**Vector Database Approach:**
```python
# Convert query to vector
query_vector = embed("lightweight comfortable running shoe")

# Find similar products
similar_products = vector_db.search(
    query_vector,
    limit=10
)
```

**Benefits:**
- Finds semantically similar items
- Understands synonyms and context
- Works with images, text, audio

## Traditional vs Vector Database

| Aspect | Traditional DB | Vector DB |
|--------|----------------|-----------|
| **Data type** | Structured (rows/columns) | Vectors (arrays of numbers) |
| **Query** | Exact match (WHERE id = 123) | Similarity search (find nearest) |
| **Index** | B-Tree, Hash | HNSW, IVF, LSH |
| **Use case** | CRUD operations | Semantic search, recommendations |
| **Example** | PostgreSQL, MySQL | Pinecone, Weaviate, Milvus |

## Key Concepts

### 1. Vector Embeddings

**Definition:** Numerical representation of data

**Example:**
```python
# Text to vector
text = "running shoe"
vector = [0.2, -0.5, 0.8, ..., 0.1]  # 768 dimensions

# Image to vector
image = load_image("shoe.jpg")
vector = [0.3, 0.1, -0.4, ..., 0.7]  # 2048 dimensions
```

### 2. Similarity Search

**Goal:** Find vectors closest to query vector

**Methods:**
- **Cosine Similarity:** Angle between vectors
- **Euclidean Distance:** Straight-line distance
- **Dot Product:** Combined magnitude and direction

### 3. Approximate Nearest Neighbors (ANN)

**Problem:** Exact search slow for millions of vectors

**Solution:** Trade accuracy for speed
- 95% accurate, 100x faster

### 4. Indexing

**Purpose:** Speed up search

**Algorithms:**
- **HNSW:** Hierarchical Navigable Small World
- **IVF:** Inverted File Index
- **LSH:** Locality Sensitive Hashing

## Use Cases

### 1. Semantic Search
```python
# Search documents by meaning
query = "machine learning tutorials"
query_vector = embed(query)
results = vector_db.search(query_vector)

# Returns:
# - "Introduction to ML"
# - "Deep Learning Guide"
# - "Neural Networks Explained"
# (Even though query words don't match exactly)
```

### 2. Recommendation Systems
```python
# User liked product A
product_a_vector = get_product_embedding("product_123")

# Find similar products
similar = vector_db.search(product_a_vector, limit=10)
```

### 3. Image Search
```python
# Find similar images
query_image_vector = embed_image("query.jpg")
similar_images = vector_db.search(query_image_vector)
```

### 4. RAG (Retrieval Augmented Generation)
```python
# Step 1: User asks question
question = "What is our return policy?"
question_vector = embed(question)

# Step 2: Find relevant documents
docs = vector_db.search(question_vector)

# Step 3: Feed to LLM
answer = llm.generate(question, context=docs)
```

### 5. Duplicate Detection
```python
# Find near-duplicate products
for product in new_products:
    vector = embed(product)
    duplicates = vector_db.search(vector, threshold=0.95)
    if duplicates:
        flag_as_duplicate(product)
```

## Popular Vector Databases

### 1. Pinecone (Managed)
```python
import pinecone

pinecone.init(api_key="your-key")
index = pinecone.Index("products")

# Insert vectors
index.upsert(vectors=[
    ("id1", [0.1, 0.2, 0.3], {"name": "Product A"}),
    ("id2", [0.2, 0.3, 0.4], {"name": "Product B"})
])

# Search
results = index.query(
    vector=[0.15, 0.25, 0.35],
    top_k=5
)
```

### 2. Weaviate (Open-Source)
```python
import weaviate

client = weaviate.Client("http://localhost:8080")

# Insert data
client.data_object.create({
    "name": "Product A",
    "description": "Lightweight running shoe"
}, "Product")

# Search
results = client.query.get(
    "Product",
    ["name", "description"]
).with_near_text({
    "concepts": ["comfortable athletic footwear"]
}).do()
```

### 3. Milvus (Scalable)
```python
from pymilvus import connections, Collection

# Connect
connections.connect(host="localhost", port="19530")

# Create collection
collection = Collection("products")

# Insert
collection.insert([
    [1, 2, 3],  # IDs
    [[0.1, 0.2], [0.3, 0.4], [0.5, 0.6]]  # Vectors
])

# Search
results = collection.search(
    data=[[0.15, 0.25]],
    anns_field="embedding",
    param={"metric_type": "L2", "params": {"nprobe": 10}},
    limit=5
)
```

### 4. Chroma (Embeddable)
```python
import chromadb

client = chromadb.Client()
collection = client.create_collection("documents")

# Add documents (auto-embedding)
collection.add(
    documents=["Document 1 text", "Document 2 text"],
    ids=["id1", "id2"]
)

# Query
results = collection.query(
    query_texts=["search query"],
    n_results=5
)
```

## Architecture Components

```
Application Layer
    ↓
┌─────────────────────────────┐
│    Vector Database          │
│                             │
│  ┌────────────────────────┐ │
│  │   Embedding Storage    │ │
│  │   (Vectors + Metadata) │ │
│  └────────────────────────┘ │
│  ┌────────────────────────┐ │
│  │   Index (HNSW/IVF)    │ │
│  │   (For fast search)    │ │
│  └────────────────────────┘ │
│  ┌────────────────────────┐ │
│  │   Query Engine         │ │
│  │   (Similarity search)  │ │
│  └────────────────────────┘ │
└─────────────────────────────┘
```

## Summary

✅ **Vector DB:** Specialized for similarity search
✅ **Use Cases:** Semantic search, recommendations, RAG, image search
✅ **Key Concepts:** Embeddings, ANN, similarity metrics
✅ **Popular Options:** Pinecone (managed), Weaviate (open-source), Milvus (scalable)
✅ **vs Traditional:** Optimized for "find similar" not "find exact"

**Continue to Chapter 02 for deep-dive on Vector Embeddings! 🚀**
