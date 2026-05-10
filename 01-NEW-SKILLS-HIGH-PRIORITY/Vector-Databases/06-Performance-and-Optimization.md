# Chapter 06: Performance and Optimization

**Making Vector Databases Fast and Efficient**

## Performance Metrics

### 1. Latency
**Goal:** <100ms per query (99th percentile)

### 2. Throughput
**Goal:** 1000+ queries per second

### 3. Recall
**Goal:** >95% accuracy (find correct results)

### 4. Index Build Time
**Goal:** Minutes, not hours

---

## Indexing Optimization

### 1. Choose Right Algorithm

**Small dataset (<100K vectors):**
```python
# Use HNSW for best accuracy
index = hnswlib.Index(space='cosine', dim=dim)
index.init_index(
    max_elements=100000,
    ef_construction=200,
    M=16
)
```

**Large dataset (>1M vectors):**
```python
# Use IVF-PQ for compression
import faiss

nlist = 1000  # Clusters
m = 8  # Sub-vectors
nbits = 8

quantizer = faiss.IndexFlatL2(dim)
index = faiss.IndexIVFPQ(quantizer, dim, nlist, m, nbits)
```

### 2. Tune Parameters

**HNSW Parameters:**
```python
# Higher ef_construction = better recall, slower build
index.init_index(
    ef_construction=400,  # Default: 200, try: 200-800
    M=32  # Default: 16, try: 16-64
)

# Search-time parameter
index.set_ef(100)  # Default: 50, try: 50-200
```

**IVF Parameters:**
```python
# More clusters = faster search, lower recall
nlist = int(np.sqrt(num_vectors))  # Rule of thumb

# Search more clusters = higher recall, slower
index.nprobe = int(nlist * 0.1)  # Search 10% of clusters
```

---

## Query Optimization

### 1. Batch Queries

**Bad (Sequential):**
```python
for query in queries:
    results = index.query(query, k=10)
```

**Good (Batch):**
```python
# 10x faster for 100 queries
results = index.query(queries, k=10)  # All at once
```

### 2. Pre-filtering vs Post-filtering

**Pre-filtering (Faster):**
```python
# Filter BEFORE vector search
results = index.query(
    vector=query_vector,
    filter={"category": "electronics"},  # Applied during search
    top_k=10
)
```

**Post-filtering (Slower):**
```python
# Search ALL, then filter
results = index.query(query_vector, top_k=100)
filtered = [r for r in results if r.metadata['category'] == 'electronics'][:10]
```

### 3. Caching

```python
import redis
import hashlib

cache = redis.Redis()

def query_with_cache(query_vector, k=10):
    # Create cache key
    key = hashlib.md5(str(query_vector).encode()).hexdigest()
    
    # Check cache
    cached = cache.get(key)
    if cached:
        return json.loads(cached)
    
    # Query vector DB
    results = index.query(query_vector, k=k)
    
    # Cache for 1 hour
    cache.setex(key, 3600, json.dumps(results))
    
    return results
```

---

## Storage Optimization

### 1. Dimensionality Reduction

**PCA Reduction:**
```python
from sklearn.decomposition import PCA

# Original: 1536 dimensions
embeddings = get_embeddings(texts)

# Reduce to 384 dimensions (75% reduction)
pca = PCA(n_components=384)
reduced = pca.fit_transform(embeddings)

# Trade-off: 95% variance retained, 4x smaller
```

### 2. Quantization

**Convert float32 → int8:**
```python
import numpy as np

# Original (float32): 4 bytes per dimension
embedding_f32 = np.array([0.123, -0.456, ...], dtype=np.float32)

# Quantized (int8): 1 byte per dimension
embedding_i8 = (embedding_f32 * 127).astype(np.int8)

# Result: 4x compression, <5% accuracy loss
```

### 3. Product Quantization (FAISS)

```python
# Compress 128-dim to 16 bytes (8x compression!)
m = 16  # Sub-vectors
nbits = 8  # Bits per sub-vector

index = faiss.IndexPQ(dim, m, nbits)
index.train(embeddings)
index.add(embeddings)

# Search is faster AND uses less memory
```

---

## Scaling Strategies

### 1. Horizontal Sharding

**Split by ID range:**
```
Shard 1: IDs 0-1M       → Server 1
Shard 2: IDs 1M-2M      → Server 2
Shard 3: IDs 2M-3M      → Server 3

Query: Search all shards, merge results
```

**Implementation:**
```python
def distributed_search(query_vector, k=10):
    # Query all shards in parallel
    with ThreadPoolExecutor() as executor:
        futures = [
            executor.submit(shard.query, query_vector, k)
            for shard in shards
        ]
        results = [f.result() for f in futures]
    
    # Merge and re-rank
    merged = merge_results(results, k)
    return merged
```

### 2. Replication

**Read replicas:**
```
┌────────────┐
│   Master   │  ← Writes
└──────┬─────┘
       │
   ┌───┴────┬──────────┐
   ↓        ↓          ↓
Replica1  Replica2  Replica3  ← Reads (load balanced)
```

### 3. Tiered Storage

**Hot/Warm/Cold:**
```
Hot (Redis):    Latest 1M vectors, <10ms latency
Warm (SSD):     Last 10M vectors, <50ms latency
Cold (S3):      Archive >100M vectors, >500ms latency

Query: Check hot → warm → cold
```

---

## Memory Optimization

### 1. Memory Mapping

```python
import faiss

# Memory-mapped index (doesn't load into RAM)
index = faiss.read_index("large_index.faiss", faiss.IO_FLAG_MMAP)

# Only accessed pages are loaded
# Good for indexes > RAM size
```

### 2. Lazy Loading

```python
class LazyIndex:
    def __init__(self, index_path):
        self.index_path = index_path
        self._index = None
    
    @property
    def index(self):
        if self._index is None:
            self._index = load_index(self.index_path)
        return self._index
    
    def query(self, vector, k=10):
        return self.index.query(vector, k)
```

---

## GPU Acceleration

### Using FAISS with GPU

```python
import faiss

# CPU index
index_cpu = faiss.IndexFlatL2(dim)
index_cpu.add(embeddings)

# Move to GPU
res = faiss.StandardGpuResources()
index_gpu = faiss.index_cpu_to_gpu(res, 0, index_cpu)

# 10-100x faster search!
D, I = index_gpu.search(queries, k=10)
```

### Multi-GPU

```python
# Use multiple GPUs
ngpus = 4
index = faiss.index_cpu_to_all_gpus(index_cpu, ngpu=ngpus)
```

---

## Monitoring

### Key Metrics to Track

```python
import time
from prometheus_client import Histogram, Counter

# Latency
query_latency = Histogram(
    'vector_search_duration_seconds',
    'Time to complete vector search'
)

# Throughput
query_counter = Counter(
    'vector_search_total',
    'Total vector searches'
)

@query_latency.time()
def query_vector_db(vector, k=10):
    query_counter.inc()
    return index.query(vector, k)
```

### Dashboard Metrics

```
Latency:
- p50: 15ms
- p95: 45ms
- p99: 120ms

Throughput:
- QPS: 1,500

Recall:
- @10: 97%
- @100: 99%

Resource Usage:
- CPU: 45%
- Memory: 12 GB / 32 GB
- Disk I/O: 150 MB/s
```

---

## Benchmarking

### Load Testing

```python
import time
import numpy as np
from concurrent.futures import ThreadPoolExecutor

def benchmark_search(index, num_queries=1000, k=10):
    # Generate random queries
    queries = np.random.rand(num_queries, dim).astype('float32')
    
    # Warm up
    for _ in range(10):
        index.query(queries[0], k)
    
    # Benchmark
    start = time.time()
    for query in queries:
        index.query(query, k)
    duration = time.time() - start
    
    qps = num_queries / duration
    latency_avg = (duration / num_queries) * 1000  # ms
    
    print(f"QPS: {qps:.0f}")
    print(f"Avg Latency: {latency_avg:.2f}ms")
```

### Parallel Load Test

```python
def parallel_benchmark(index, num_threads=10, queries_per_thread=100):
    def worker():
        for _ in range(queries_per_thread):
            query = np.random.rand(dim).astype('float32')
            index.query(query, k=10)
    
    start = time.time()
    with ThreadPoolExecutor(max_workers=num_threads) as executor:
        futures = [executor.submit(worker) for _ in range(num_threads)]
        [f.result() for f in futures]
    duration = time.time() - start
    
    total_queries = num_threads * queries_per_thread
    qps = total_queries / duration
    print(f"Concurrent QPS: {qps:.0f}")
```

---

## Best Practices Checklist

### Indexing
- [ ] Choose algorithm based on dataset size
- [ ] Tune parameters (ef_construction, M, nlist)
- [ ] Use GPU for large-scale indexing
- [ ] Monitor index build time

### Querying
- [ ] Batch queries when possible
- [ ] Use pre-filtering over post-filtering
- [ ] Cache frequent queries
- [ ] Set appropriate k value (not too high)

### Storage
- [ ] Apply dimensionality reduction if needed
- [ ] Use quantization for compression
- [ ] Implement tiered storage (hot/warm/cold)

### Scaling
- [ ] Shard large datasets
- [ ] Use read replicas for high throughput
- [ ] Monitor resource usage
- [ ] Plan capacity based on growth

### Monitoring
- [ ] Track latency (p50, p95, p99)
- [ ] Monitor throughput (QPS)
- [ ] Measure recall accuracy
- [ ] Set up alerts for degradation

---

## Optimization Checklist

**Start Here:**
1. Profile current performance (latency, QPS, recall)
2. Identify bottleneck (query slow? build slow? memory?)
3. Apply relevant optimizations
4. Measure improvement
5. Repeat

**Quick Wins:**
- ✅ Enable batching (10x speedup)
- ✅ Add caching (2-5x speedup)
- ✅ Use GPU (10-100x speedup for large datasets)
- ✅ Pre-filter instead of post-filter (2-3x speedup)

---

## Summary

✅ **Indexing:** Choose right algorithm, tune parameters
✅ **Query:** Batch, cache, pre-filter
✅ **Storage:** Dimensionality reduction, quantization
✅ **Scaling:** Sharding, replication, tiered storage
✅ **Acceleration:** GPU, parallel processing
✅ **Monitoring:** Track latency, throughput, recall

**You've completed the Vector-Databases learning path! 🎉**

**All 3 topics (18 chapters) are now ready for learning! 🚀**
