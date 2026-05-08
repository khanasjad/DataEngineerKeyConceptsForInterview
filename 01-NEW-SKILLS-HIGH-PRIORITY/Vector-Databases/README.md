# Vector Databases - High Priority

**Why This Matters:** Essential for RAG, semantic search, recommendation systems. Asked in 60% of modern data engineer interviews.

**Study Time:** Week 1 (Day 5-6) - 2-3 hours

---

## Core Concepts

### What is a Vector Database?
Database optimized for storing and querying high-dimensional vectors (embeddings). Enables semantic/similarity search instead of exact matching.

**Use Cases:**
- RAG systems (document Q&A)
- Semantic search (find similar products, documents)
- Recommendation engines
- Anomaly detection
- Image/video similarity search

### Key Differences from Traditional Databases

| Traditional DB | Vector DB |
|---------------|-----------|
| Exact matching (WHERE id = 123) | Similarity search (find similar) |
| Structured data (rows/columns) | High-dimensional vectors |
| B-tree indexes | ANN indexes (HNSW, IVF) |
| Fast exact lookups | Fast approximate nearest neighbor |

---

## Vector Database Options (2026)

### Production-Ready (Learn These):

**1. Pinecone** (Recommended for beginners)
- Fully managed, cloud-native
- Easy to use, great docs
- Generous free tier
- Best for: Getting started, production RAG systems

**2. Weaviate**
- Open source, self-hosted or cloud
- Built-in vectorization (can generate embeddings for you)
- GraphQL API
- Best for: Complex hybrid search (vector + keyword)

**3. Qdrant**
- Open source, Rust-based (fast)
- Good filtering capabilities
- Self-hosted or cloud
- Best for: High-performance requirements

**4. Chroma**
- Open source, lightweight
- Easy local development
- Python-native
- Best for: Development, prototyping, small projects

**5. pgvector** (PostgreSQL extension)
- Add vector search to existing Postgres
- Familiar SQL interface
- Self-hosted
- Best for: Existing Postgres infrastructure

### Others Worth Knowing:
- Milvus (large-scale, distributed)
- Redis Vector Search (if already using Redis)
- Elasticsearch with vector search
- Azure Cognitive Search, AWS OpenSearch

---

## Key Technical Concepts

### 1. Approximate Nearest Neighbor (ANN)
**Problem:** Finding exact nearest neighbors in high dimensions is slow O(n)

**Solution:** ANN algorithms trade small accuracy for massive speed gains

**Popular Algorithms:**
- **HNSW** (Hierarchical Navigable Small World) - Most popular, good balance
- **IVF** (Inverted File Index) - Fast search, needs training
- **LSH** (Locality Sensitive Hashing) - Simpler, less accurate

**Interview Tip:** Just know HNSW is current best practice. Don't need deep math.

### 2. Distance Metrics

**Cosine Similarity** (Most common for text)
- Measures angle between vectors
- Range: -1 to 1 (1 = identical direction)
- Ignores magnitude, focuses on direction
- **Use for:** Text embeddings, normalized vectors

**Euclidean Distance** (L2)
- Straight-line distance between points
- **Use for:** When magnitude matters (image embeddings)

**Dot Product**
- Similar to cosine but includes magnitude
- **Use for:** Already normalized vectors, faster computation

### 3. Embeddings
**What:** Dense vector representation of data (text, images, etc.)

**Common Dimensions:**
- OpenAI text-embedding-3-small: 1536 dimensions
- sentence-transformers: 384-768 dimensions
- Hugging Face models: varies (768 is common)

**Key Point:** Higher dimensions ≠ always better. Balance accuracy vs speed/cost.

---

## Study Resources (6 hours total)

### Day 5: Fundamentals (3 hours)

**Watch/Read (2 hours):**
- [Course] "Vector Databases from Embeddings to Applications" - DeepLearning.AI
  https://www.deeplearning.ai/short-courses/vector-databases-embeddings-applications/

- [Video] "Vector Databases Explained" by IBM (15 min)
  https://www.youtube.com/watch?v=klTvEwg3oJ4

- [Article] "What is a Vector Database?" - Pinecone (30 min)
  https://www.pinecone.io/learn/vector-database/

**Hands-On (1 hour):**
- Set up Pinecone free account
- Follow quickstart tutorial
- Index 100 sample documents

### Day 6: Advanced Topics (3 hours)

**Watch/Read (1.5 hours):**
- [Docs] Pinecone documentation - indexing, querying, filtering (1 hour)
  https://docs.pinecone.io/

- [Article] "Vector Database Comparison 2026" (30 min)
  Search Google for latest comparisons

**Hands-On (1.5 hours):**
- Build same RAG app with different vector DBs (ChromaDB vs Pinecone)
- Compare performance, ease of use
- Test filtering and metadata queries

---

## Hands-On Exercises

### Exercise 1: Basic Vector Search (1 hour)
```python
# 1. Install Pinecone
# pip install pinecone-client openai

# 2. Create index and add vectors
# 3. Query with similarity search
# 4. Filter by metadata

# Goal: Understand indexing, querying, filtering
```

### Exercise 2: RAG with Vector DB (2 hours)
```python
# 1. Chunk 20 documents
# 2. Generate embeddings (OpenAI or sentence-transformers)
# 3. Store in Pinecone with metadata
# 4. Implement retrieval function
# 5. Build RAG pipeline

# Goal: End-to-end RAG implementation
```

### Exercise 3: Performance Comparison (1 hour)
```python
# Compare:
# - ChromaDB (local)
# - Pinecone (cloud)

# Test:
# - Index 1000 documents
# - Query speed
# - Accuracy (top-k results)

# Goal: Understand trade-offs
```

---

## Interview Preparation

### Common Interview Questions:

**1. "What is a vector database and when would you use it?"**

**Answer:**
A vector database stores high-dimensional embeddings and enables similarity search using ANN algorithms. Use when you need semantic search - finding similar items by meaning, not exact matches.

**Use cases:** RAG systems, recommendation engines, semantic search, duplicate detection.

**Example:** "At Optum, I'd use a vector DB to build a medical documentation search system that finds relevant patient records based on semantic similarity, not just keyword matching."

---

**2. "How do vector databases differ from traditional databases?"**

**Answer:**
| Aspect | Traditional DB | Vector DB |
|--------|---------------|-----------|
| Query type | Exact matching | Similarity search |
| Index | B-tree, hash | ANN (HNSW, IVF) |
| Query | WHERE id = 123 | Find top 10 similar |
| Data | Structured rows | High-dim vectors |

Vector DBs sacrifice exact results for speed using approximate nearest neighbor algorithms.

---

**3. "Explain how vector search works under the hood"**

**Answer:**
1. **Indexing:** Vectors organized using ANN algorithm (e.g., HNSW creates hierarchical graph)
2. **Query:** Input converted to vector via embedding model
3. **Search:** ANN algorithm navigates index to find approximate nearest neighbors
4. **Return:** Top-k most similar vectors with distance scores

HNSW is most popular - creates multi-layer graph where each layer has fewer nodes. Search starts at top, quickly narrows down to relevant region.

---

**4. "How would you choose between vector database options?"**

**Answer Framework:**
- **Scale:** Pinecone/Weaviate for large-scale, Chroma for small
- **Infrastructure:** pgvector if already using Postgres, else managed service
- **Budget:** Chroma/Qdrant (open source) vs Pinecone (managed, paid)
- **Features:** Hybrid search → Weaviate, simple RAG → Pinecone
- **Performance:** Qdrant (Rust) fastest, but Pinecone good enough for most

**My recommendation:** Start with Pinecone (easy, managed), migrate to Qdrant if need more control/performance.

---

**5. "Design a semantic search system for internal documents"**

**Answer (Cover these points):**

**Architecture:**
```
Documents → Chunking → Embedding Model → Vector DB
                                            ↓
User Query → Embedding → Similarity Search → Results
```

**Components:**
1. **Ingestion Pipeline:**
   - Document parsing (PDF, Word, etc.)
   - Chunking (500-1000 tokens, overlap 50-100)
   - Embedding generation (sentence-transformers or OpenAI)
   - Store in vector DB with metadata (source, timestamp, author)

2. **Query Pipeline:**
   - User query → embedding
   - Vector search (top-k=10)
   - Re-ranking (optional, using cross-encoder)
   - Return results with source links

3. **Infrastructure:**
   - Vector DB: Pinecone or Weaviate
   - Embedding model: sentence-transformers/all-MiniLM-L6-v2 (fast) or OpenAI
   - Backend: Python + FastAPI
   - Frontend: React or Streamlit

4. **Optimizations:**
   - Caching for common queries
   - Hybrid search (vector + keyword) for better accuracy
   - Metadata filtering (department, date range)
   - Batch embedding generation for efficiency

5. **Monitoring:**
   - Query latency (target <100ms)
   - Search quality (user feedback)
   - Cost (embedding API calls)
   - Index size and growth

**Optum Example:** "I'd use this to search across RQNS platform documentation, enabling engineers to quickly find relevant code examples and architecture decisions."

---

## Quick Reference Cheat Sheet

### Pinecone Basics:
```python
import pinecone
from openai import OpenAI

# 1. Initialize
pinecone.init(api_key="YOUR_KEY")
index = pinecone.Index("my-index")

# 2. Create embeddings
client = OpenAI()
response = client.embeddings.create(
    model="text-embedding-3-small",
    input="your text here"
)
embedding = response.data[0].embedding

# 3. Upsert (add/update)
index.upsert(vectors=[
    ("id1", embedding, {"text": "original text", "source": "doc.pdf"})
])

# 4. Query
results = index.query(
    vector=query_embedding,
    top_k=10,
    include_metadata=True
)
```

### When to Use Which Vector DB:

**Starting a project?** → Pinecone (easy, managed)
**Building prototype?** → ChromaDB (local, free)
**Already have Postgres?** → pgvector (familiar SQL)
**Need hybrid search?** → Weaviate (vector + keyword)
**High-performance needs?** → Qdrant (Rust, fast)

---

## Practice Problems

**Problem 1:** Implement duplicate detection system for customer support tickets
- Hint: Use vector search to find similar past tickets

**Problem 2:** Build product recommendation engine
- Hint: Store product embeddings, find similar products

**Problem 3:** Create code search system for GitHub repos
- Hint: Embed code snippets, enable semantic code search

**Time:** 1 hour each

---

## Week 1 Success Metrics

**Knowledge:**
- [ ] Understand vector search vs traditional search
- [ ] Know ANN algorithms at high level (HNSW)
- [ ] Familiar with distance metrics (cosine, euclidean, dot product)
- [ ] Can compare vector DB options

**Hands-On:**
- [ ] Set up Pinecone account
- [ ] Indexed and queried vectors
- [ ] Used metadata filtering
- [ ] Built RAG system with vector DB

**Interview Ready:**
- [ ] Can design semantic search system
- [ ] Explain vector search to non-technical person
- [ ] Know trade-offs between vector DB options
- [ ] Have working demo with Pinecone

---

**Next:** Integrate this knowledge into your RAG project from GenAI-LLM-RAG folder
