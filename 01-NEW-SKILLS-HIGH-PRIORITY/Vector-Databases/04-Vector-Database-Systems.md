# Chapter 04: Vector Database Systems

**Comparison of Popular Vector Databases**

## Overview of Vector Databases

| Database | Type | Open-Source | Cloud | Best For |
|----------|------|-------------|-------|----------|
| **Pinecone** | Managed | ❌ | ✅ | Easy setup, production |
| **Weaviate** | Hybrid | ✅ | ✅ | Feature-rich, GraphQL |
| **Milvus** | Self-hosted | ✅ | ❌ | Scalability, billions |
| **Chroma** | Embedded | ✅ | ❌ | Dev/testing, simple |
| **Qdrant** | Self-hosted | ✅ | ❌ | Fast, filtering |
| **FAISS** | Library | ✅ | ❌ | Research, custom builds |

---

## 1. Pinecone (Managed Service)

**Pros:**
- Fully managed (no ops)
- Auto-scaling
- Easy to use
- Good documentation

**Cons:**
- Closed source
- Expensive at scale
- Vendor lock-in

**Setup:**
```python
import pinecone

# Initialize
pinecone.init(api_key="your-api-key", environment="us-west1-gcp")

# Create index
pinecone.create_index(
    name="products",
    dimension=384,
    metric="cosine",
    pod_type="p1.x1"  # Performance tier
)

index = pinecone.Index("products")

# Insert vectors
index.upsert(vectors=[
    ("id1", [0.1, 0.2, ...], {"name": "Product A", "price": 99}),
    ("id2", [0.2, 0.3, ...], {"name": "Product B", "price": 149})
])

# Search
results = index.query(
    vector=[0.15, 0.25, ...],
    top_k=10,
    filter={"price": {"$gte": 100}},  # Metadata filtering
    include_metadata=True
)

# Results
for match in results.matches:
    print(f"ID: {match.id}, Score: {match.score}, Metadata: {match.metadata}")
```

**Pricing:**
- Starter: $70/month (100K vectors)
- Standard: $0.096/hour per pod
- Enterprise: Custom

---

## 2. Weaviate (Hybrid Search)

**Pros:**
- Open-source
- GraphQL API
- Hybrid search (vector + keyword)
- Built-in vectorization

**Cons:**
- More complex setup
- Requires more resources

**Setup (Docker):**
```yaml
# docker-compose.yml
version: '3.4'
services:
  weaviate:
    image: semitechnologies/weaviate:latest
    ports:
      - "8080:8080"
    environment:
      QUERY_DEFAULTS_LIMIT: 25
      AUTHENTICATION_ANONYMOUS_ACCESS_ENABLED: 'true'
      PERSISTENCE_DATA_PATH: '/var/lib/weaviate'
```

**Python Client:**
```python
import weaviate

# Connect
client = weaviate.Client("http://localhost:8080")

# Create schema
schema = {
    "class": "Product",
    "vectorizer": "text2vec-openai",
    "properties": [
        {"name": "name", "dataType": ["string"]},
        {"name": "description", "dataType": ["text"]},
        {"name": "price", "dataType": ["number"]}
    ]
}
client.schema.create_class(schema)

# Insert data
client.data_object.create(
    data_object={
        "name": "Laptop",
        "description": "High-performance laptop for developers",
        "price": 1299
    },
    class_name="Product"
)

# Vector search
results = client.query.get(
    "Product",
    ["name", "description", "price"]
).with_near_text({
    "concepts": ["affordable computer for programming"]
}).with_limit(10).do()

# Hybrid search (vector + keyword)
results = client.query.get(
    "Product",
    ["name", "price"]
).with_hybrid(
    query="laptop",
    alpha=0.5  # 0.5 = balanced, 0 = keyword only, 1 = vector only
).do()
```

---

## 3. Milvus (Scalable Open-Source)

**Pros:**
- Highly scalable (billions of vectors)
- Multiple index types
- GPU support
- Active community

**Cons:**
- Complex deployment
- Steep learning curve

**Setup (Docker):**
```bash
# Download docker-compose
wget https://github.com/milvus-io/milvus/releases/download/v2.3.0/milvus-standalone-docker-compose.yml -O docker-compose.yml

# Start Milvus
docker-compose up -d
```

**Python Client:**
```python
from pymilvus import connections, Collection, FieldSchema, CollectionSchema, DataType

# Connect
connections.connect(host="localhost", port="19530")

# Define schema
fields = [
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True, auto_id=True),
    FieldSchema(name="embedding", dtype=DataType.FLOAT_VECTOR, dim=384),
    FieldSchema(name="name", dtype=DataType.VARCHAR, max_length=200),
    FieldSchema(name="price", dtype=DataType.FLOAT)
]
schema = CollectionSchema(fields, description="Product embeddings")

# Create collection
collection = Collection(name="products", schema=schema)

# Insert data
entities = [
    [[0.1, 0.2, ...], [0.2, 0.3, ...]],  # Embeddings
    ["Product A", "Product B"],           # Names
    [99.0, 149.0]                         # Prices
]
collection.insert(entities)

# Create index
index_params = {
    "metric_type": "IP",  # Inner product
    "index_type": "IVF_FLAT",
    "params": {"nlist": 128}
}
collection.create_index(field_name="embedding", index_params=index_params)

# Load to memory
collection.load()

# Search
search_params = {"metric_type": "IP", "params": {"nprobe": 10}}
results = collection.search(
    data=[[0.15, 0.25, ...]],
    anns_field="embedding",
    param=search_params,
    limit=10,
    expr="price > 100"  # Filter
)

for hits in results:
    for hit in hits:
        print(f"ID: {hit.id}, Distance: {hit.distance}")
```

---

## 4. Chroma (Embedded)

**Pros:**
- Simplest setup
- Embedded in app
- Auto-embedding
- Great for prototyping

**Cons:**
- Not for production scale
- Limited features

**Setup:**
```bash
pip install chromadb
```

**Usage:**
```python
import chromadb

# Initialize
client = chromadb.Client()

# Create collection
collection = client.create_collection("products")

# Add documents (auto-embedding!)
collection.add(
    documents=[
        "High-performance laptop for developers",
        "Affordable notebook for students"
    ],
    metadatas=[
        {"name": "Laptop Pro", "price": 1299},
        {"name": "Student Laptop", "price": 599}
    ],
    ids=["id1", "id2"]
)

# Query
results = collection.query(
    query_texts=["cheap computer for coding"],
    n_results=5,
    where={"price": {"$lt": 1000}}  # Filter
)

print(results['documents'])
print(results['metadatas'])
```

---

## 5. Qdrant

**Pros:**
- Fast Rust implementation
- Powerful filtering
- Good balance of features/performance

**Cons:**
- Smaller community
- Less mature than Milvus

**Setup (Docker):**
```bash
docker run -p 6333:6333 qdrant/qdrant
```

**Python Client:**
```python
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct

# Connect
client = QdrantClient(host="localhost", port=6333)

# Create collection
client.create_collection(
    collection_name="products",
    vectors_config=VectorParams(size=384, distance=Distance.COSINE)
)

# Insert vectors
client.upsert(
    collection_name="products",
    points=[
        PointStruct(
            id=1,
            vector=[0.1, 0.2, ...],
            payload={"name": "Product A", "price": 99}
        ),
        PointStruct(
            id=2,
            vector=[0.2, 0.3, ...],
            payload={"name": "Product B", "price": 149}
        )
    ]
)

# Search with filtering
results = client.search(
    collection_name="products",
    query_vector=[0.15, 0.25, ...],
    limit=10,
    query_filter={
        "must": [
            {"key": "price", "range": {"gte": 100}}
        ]
    }
)

for result in results:
    print(f"ID: {result.id}, Score: {result.score}, Payload: {result.payload}")
```

---

## Feature Comparison

| Feature | Pinecone | Weaviate | Milvus | Chroma | Qdrant |
|---------|----------|----------|--------|--------|--------|
| **Ease of Use** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Scalability** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| **Performance** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Filtering** | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Cost** | 💰💰💰 | 💰 | 💰 | Free | 💰 |
| **Updates** | Fast | Medium | Medium | Fast | Fast |
| **Community** | Medium | Large | Large | Medium | Growing |

---

## Choosing the Right Database

**Decision Tree:**

```
Production or Prototype?
├─ Prototype → Chroma
└─ Production →

    Need managed service?
    ├─ Yes → Pinecone
    └─ No →

        Billions of vectors?
        ├─ Yes → Milvus
        └─ No →

            Need hybrid search?
            ├─ Yes → Weaviate
            └─ No → Qdrant
```

---

## Migration Example

**From Pinecone to Weaviate:**

```python
# 1. Export from Pinecone
pinecone_index = pinecone.Index("old-index")
results = pinecone_index.fetch(ids=all_ids)

# 2. Transform data
weaviate_data = []
for id, item in results.vectors.items():
    weaviate_data.append({
        "id": id,
        "vector": item.values,
        "metadata": item.metadata
    })

# 3. Import to Weaviate
client = weaviate.Client("http://localhost:8080")
for item in weaviate_data:
    client.data_object.create(
        data_object=item["metadata"],
        class_name="Products",
        vector=item["vector"]
    )
```

---

## Summary

✅ **Pinecone:** Easiest, managed, expensive
✅ **Weaviate:** Feature-rich, hybrid search
✅ **Milvus:** Most scalable, billions of vectors
✅ **Chroma:** Simplest, embedded, prototyping
✅ **Qdrant:** Fast, powerful filtering

**Continue to Chapter 05 for Real-World Applications! 🚀**
