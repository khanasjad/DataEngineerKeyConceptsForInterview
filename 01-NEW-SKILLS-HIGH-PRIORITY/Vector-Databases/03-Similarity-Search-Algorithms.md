# Chapter 03: Similarity Search Algorithms

**How Vector Databases Find Similar Items Fast**

## The Challenge

**Problem:** Given 1 million vectors, find the 10 most similar to a query vector

**Naive Approach (Brute Force):**
```python
def naive_search(query, database):
    """Compare query to every vector"""
    similarities = []
    for vector in database:
        sim = cosine_similarity(query, vector)
        similarities.append((vector, sim))
    
    # Return top 10
    return sorted(similarities, key=lambda x: x[1], reverse=True)[:10]

# Time complexity: O(n * d)
# n = 1 million vectors, d = 768 dimensions
# Too slow for production!
```

**Solution:** Approximate Nearest Neighbors (ANN)

## Approximate vs Exact Search

| Approach | Accuracy | Speed | Use Case |
|----------|----------|-------|----------|
| **Exact** | 100% | Slow (O(n)) | Small datasets (<10K) |
| **ANN** | 95-99% | Fast (O(log n)) | Large datasets (>100K) |

**Trade-off:** Accept 95% accuracy for 100x speed improvement

## Popular ANN Algorithms

### 1. HNSW (Hierarchical Navigable Small World)

**Concept:** Multi-layer graph where each node connects to nearest neighbors

**How it works:**
```
Layer 2: [Entry point] → [Node A]
           ↓
Layer 1: [Node B] → [Node C] → [Node D]
           ↓
Layer 0: [All vectors] (full resolution)

Search:
1. Start at entry point (top layer)
2. Navigate to closest neighbor
3. Drop to lower layer
4. Repeat until bottom layer
5. Refine search at bottom
```

**Implementation (Hnswlib):**
```python
import hnswlib
import numpy as np

# Initialize index
dim = 128  # Dimension
num_elements = 10000

index = hnswlib.Index(space='cosine', dim=dim)
index.init_index(
    max_elements=num_elements,
    ef_construction=200,  # Higher = better accuracy, slower build
    M=16  # Number of connections per node
)

# Add vectors
data = np.random.rand(num_elements, dim).astype('float32')
ids = np.arange(num_elements)
index.add_items(data, ids)

# Search
query = np.random.rand(dim).astype('float32')
labels, distances = index.knn_query(query, k=10)

print(f"Top 10 similar: {labels[0]}")
print(f"Distances: {distances[0]}")
```

**Pros:**
- Very fast queries
- Good recall (95-99%)
- Memory efficient

**Cons:**
- No dynamic updates (must rebuild)
- Memory-intensive during build

---

### 2. IVF (Inverted File Index)

**Concept:** Cluster vectors, search only relevant clusters

**How it works:**
```
1. Training Phase:
   - Cluster all vectors into N clusters (e.g., 100)
   - Each vector assigned to nearest cluster

2. Index Phase:
   - Store vectors in cluster buckets

3. Search Phase:
   - Find nearest K clusters to query (e.g., 5)
   - Search only within those clusters
   - Much faster than searching all vectors!
```

**Implementation (FAISS):**
```python
import faiss
import numpy as np

# Data
d = 128  # dimension
nb = 100000  # database size
nq = 10  # query size

xb = np.random.random((nb, d)).astype('float32')
xq = np.random.random((nq, d)).astype('float32')

# Create IVF index
nlist = 100  # Number of clusters
quantizer = faiss.IndexFlatL2(d)
index = faiss.IndexIVFFlat(quantizer, d, nlist)

# Train
index.train(xb)

# Add vectors
index.add(xb)

# Search (probe 5 clusters)
index.nprobe = 5
distances, indices = index.search(xq, k=10)

print(f"Nearest neighbors: {indices}")
```

**Pros:**
- Scalable to billions of vectors
- Good balance of speed/accuracy
- Supports GPU acceleration

**Cons:**
- Requires training phase
- Accuracy depends on cluster quality

---

### 3. LSH (Locality Sensitive Hashing)

**Concept:** Hash similar vectors to same buckets

**How it works:**
```
1. Create multiple hash functions
2. Hash each vector multiple times
3. Store in hash tables
4. Query: Hash query vector, look up buckets

Example:
Vector A: [0.1, 0.9] → Hash: "bucket_7"
Vector B: [0.2, 0.8] → Hash: "bucket_7"  ← Same bucket (similar!)
Vector C: [0.9, 0.1] → Hash: "bucket_23"  ← Different bucket
```

**Implementation:**
```python
from sklearn.neighbors import LSHForest

# Create LSH index
lsh = LSHForest(n_estimators=20, n_candidates=200)

# Fit
lsh.fit(vectors)

# Query
query_vector = np.array([[0.5, 0.5]])
distances, indices = lsh.kneighbors(query_vector, n_neighbors=10)
```

**Pros:**
- Fast for high-dimensional data
- Memory efficient
- Supports dynamic updates

**Cons:**
- Lower accuracy than HNSW
- Requires tuning hash functions

---

### 4. Product Quantization (PQ)

**Concept:** Compress vectors by quantizing sub-vectors

**How it works:**
```
1. Split vector into sub-vectors
   [0.1, 0.2, 0.3, 0.4, 0.5, 0.6] →
   [0.1, 0.2] | [0.3, 0.4] | [0.5, 0.6]

2. Quantize each sub-vector to nearest centroid
   [0.1, 0.2] → Centroid #7
   [0.3, 0.4] → Centroid #12
   [0.5, 0.6] → Centroid #3

3. Store compressed: [7, 12, 3] (much smaller!)

4. Search: Compute distances using codebook
```

**Implementation (FAISS):**
```python
import faiss

d = 128
nb = 100000

# Create PQ index
m = 16  # Number of sub-vectors
nbits = 8  # Bits per sub-vector

index = faiss.IndexPQ(d, m, nbits)
index.train(xb)
index.add(xb)

# Search
D, I = index.search(xq, k=10)
```

**Pros:**
- Massive compression (10-100x)
- Scales to billions
- Fast search

**Cons:**
- Requires training
- Lower accuracy than HNSW

---

## Algorithm Comparison

| Algorithm | Speed | Accuracy | Memory | Updates | Best For |
|-----------|-------|----------|--------|---------|----------|
| **HNSW** | ⚡⚡⚡ | 99% | High | Slow | Small-medium datasets |
| **IVF** | ⚡⚡ | 95% | Medium | Slow | Large datasets |
| **LSH** | ⚡⚡ | 90% | Low | Fast | High-dimensional |
| **PQ** | ⚡⚡⚡ | 90% | Very Low | Slow | Billions of vectors |

---

## Hybrid Approaches

### IVF + PQ (FAISS)

```python
# Combine IVF clustering with PQ compression
nlist = 100  # IVF clusters
m = 8  # PQ sub-vectors
nbits = 8

quantizer = faiss.IndexFlatL2(d)
index = faiss.IndexIVFPQ(quantizer, d, nlist, m, nbits)

index.train(xb)
index.add(xb)

# Best of both: Fast + Scalable
```

### HNSW + PQ

```python
# HNSW for speed, PQ for compression
index = faiss.IndexHNSWPQ(d, m, M)
```

---

## Tuning Parameters

### HNSW

```python
# ef_construction: Build-time accuracy
# Higher → Better graph, slower build
index.init_index(ef_construction=400)  # Default: 200

# ef: Search-time accuracy
# Higher → Better recall, slower search
index.set_ef(100)  # Default: 50

# M: Connections per node
# Higher → Better recall, more memory
```

### IVF

```python
# nlist: Number of clusters
# More → Faster search, lower recall
index = faiss.IndexIVFFlat(quantizer, d, nlist=1000)

# nprobe: Clusters to search
# More → Better recall, slower search
index.nprobe = 10  # Search top 10 clusters
```

---

## Performance Benchmarks

**Dataset:** 1 million 768-dim vectors

| Algorithm | Build Time | Query Time (ms) | Recall@10 | Memory (GB) |
|-----------|-----------|-----------------|-----------|-------------|
| Brute Force | - | 500 | 100% | 3.0 |
| HNSW | 10 min | 0.5 | 99% | 4.5 |
| IVF | 5 min | 2.0 | 95% | 3.2 |
| LSH | 2 min | 3.0 | 90% | 2.5 |
| IVF-PQ | 6 min | 1.5 | 93% | 0.5 |

---

## Choosing the Right Algorithm

**Decision Tree:**

```
How many vectors?
├─ < 10K → Brute force or HNSW
├─ 10K - 1M → HNSW
├─ 1M - 100M → IVF or IVF-PQ
└─ > 100M → IVF-PQ or distributed

Need updates?
├─ Frequent → LSH
└─ Rare → HNSW/IVF

Memory constrained?
├─ Yes → IVF-PQ or LSH
└─ No → HNSW

Need highest accuracy?
├─ Yes → HNSW (ef=200)
└─ No → IVF-PQ
```

---

## Summary

✅ **ANN:** Trade accuracy for speed (95% accurate, 100x faster)
✅ **HNSW:** Best accuracy, fast queries
✅ **IVF:** Scalable, good balance
✅ **LSH:** Memory efficient, supports updates
✅ **PQ:** Massive compression
✅ **Hybrid:** Combine for best results

**Continue to Chapter 04 for Vector Database Systems! 🚀**
