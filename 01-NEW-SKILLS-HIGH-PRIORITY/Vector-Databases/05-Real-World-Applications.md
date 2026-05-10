# Chapter 05: Real-World Applications

**Practical Use Cases for Vector Databases**

## 1. Semantic Search

### Problem
Traditional keyword search fails to understand meaning.

### Solution with Vector DB

```python
# Build semantic search system
from sentence_transformers import SentenceTransformer
import chromadb

# Initialize
model = SentenceTransformer('all-MiniLM-L6-v2')
client = chromadb.Client()
collection = client.create_collection("documents")

# Index documents
documents = [
    "Python programming tutorial for beginners",
    "Advanced machine learning with TensorFlow",
    "Data structures and algorithms in Java"
]

# Add to vector DB (auto-embedding)
collection.add(
    documents=documents,
    ids=[f"doc{i}" for i in range(len(documents))]
)

# Search by meaning
query = "learn coding for newbies"
results = collection.query(
    query_texts=[query],
    n_results=3
)

# Returns: "Python programming tutorial for beginners"
# Even though keywords don't match exactly!
```

---

## 2. Recommendation Systems

### Product Recommendations

```python
import pinecone

# Initialize
pinecone.init(api_key="key", environment="us-west1-gcp")
index = pinecone.Index("products")

# User viewed product
viewed_product_id = "product_123"
viewed_product_vector = get_product_embedding(viewed_product_id)

# Find similar products
similar = index.query(
    vector=viewed_product_vector,
    top_k=10,
    filter={"category": "electronics", "price": {"$lt": 500}}
)

# Show recommendations
for match in similar.matches:
    print(f"Recommend: {match.metadata['name']} (Score: {match.score:.2f})")
```

### User-to-User Recommendations

```python
# Create user profiles from purchase history
user_profile = aggregate_purchase_embeddings(user_id)

# Find similar users
similar_users = index.query(
    vector=user_profile,
    top_k=20,
    filter={"user_id": {"$ne": user_id}}
)

# Recommend products liked by similar users
recommendations = get_products_from_similar_users(similar_users)
```

---

## 3. RAG (Retrieval Augmented Generation)

### Document Q&A System

```python
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Pinecone
from langchain.llms import OpenAI
from langchain.chains import RetrievalQA

# Setup
embeddings = OpenAIEmbeddings()
vectorstore = Pinecone.from_documents(
    documents=docs,
    embedding=embeddings,
    index_name="company-docs"
)

# Create QA chain
qa = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    chain_type="stuff",
    retriever=vectorstore.as_retriever(search_kwargs={"k": 3})
)

# Ask question
question = "What is our return policy?"
answer = qa.run(question)

# Process:
# 1. Convert question to embedding
# 2. Search vector DB for relevant docs
# 3. Feed docs + question to LLM
# 4. Generate answer with sources
```

---

## 4. Image Search

### Reverse Image Search

```python
import clip
import torch
from PIL import Image

# Load CLIP model
model, preprocess = clip.load("ViT-B/32")

# Index images
image_paths = ["img1.jpg", "img2.jpg", "img3.jpg"]
image_embeddings = []

for path in image_paths:
    image = preprocess(Image.open(path)).unsqueeze(0)
    with torch.no_grad():
        embedding = model.encode_image(image)
    image_embeddings.append(embedding)

# Store in vector DB
index.upsert(vectors=[
    (path, emb.tolist(), {"filename": path})
    for path, emb in zip(image_paths, image_embeddings)
])

# Search with query image
query_image = preprocess(Image.open("query.jpg")).unsqueeze(0)
with torch.no_grad():
    query_embedding = model.encode_image(query_image)

similar_images = index.query(
    vector=query_embedding.tolist(),
    top_k=5
)
```

### Text-to-Image Search

```python
# Search images using text description
text_query = "a red sports car"
text = clip.tokenize([text_query])

with torch.no_grad():
    text_embedding = model.encode_text(text)

similar_images = index.query(
    vector=text_embedding.tolist(),
    top_k=10
)

# Returns images matching the description
```

---

## 5. Anomaly Detection

### Fraud Detection

```python
# Build normal transaction profiles
normal_transactions = get_transaction_embeddings(user_id, is_fraud=False)

# Calculate "normal" centroid
normal_centroid = np.mean(normal_transactions, axis=0)

# Store in vector DB
index.upsert([("normal_profile", normal_centroid.tolist())])

# Check new transaction
new_transaction_embedding = embed_transaction(new_transaction)
similarity = index.query(
    vector=new_transaction_embedding,
    top_k=1
)

# If similarity is low, flag as potential fraud
if similarity.matches[0].score < 0.7:  # Threshold
    flag_as_fraud(new_transaction)
```

---

## 6. Chatbot with Memory

### Conversational AI

```python
from langchain.memory import VectorStoreRetrieverMemory
from langchain.chains import ConversationChain

# Store conversation history in vector DB
memory = VectorStoreRetrieverMemory(
    retriever=vectorstore.as_retriever(search_kwargs={"k": 5})
)

# Create conversational chain
conversation = ConversationChain(
    llm=OpenAI(),
    memory=memory
)

# Chat
conversation.predict(input="My name is John")
# Bot: "Nice to meet you, John!"

# Later in conversation (after many messages)
conversation.predict(input="What's my name?")
# Bot: "Your name is John" (retrieved from vector DB!)
```

---

## 7. Duplicate Detection

### Product Deduplication

```python
# New product listing
new_product = {
    "title": "Apple iPhone 14 Pro - 256GB - Space Black",
    "description": "Latest iPhone with A16 chip..."
}

# Convert to embedding
new_embedding = embed_product(new_product)

# Search for similar products
similar = index.query(
    vector=new_embedding,
    top_k=5
)

# Check if duplicate (high similarity)
for match in similar.matches:
    if match.score > 0.95:  # 95% similar
        print(f"Potential duplicate: {match.metadata['title']}")
        merge_or_reject(new_product, match.id)
```

---

## 8. Code Search

### Semantic Code Search

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('code-search-net')

# Index code snippets
code_snippets = [
    "def bubble_sort(arr): ...",
    "def binary_search(arr, target): ...",
    "class LinkedList: ..."
]

embeddings = model.encode(code_snippets)
index.upsert([
    (f"code{i}", emb.tolist(), {"code": snippet})
    for i, (snippet, emb) in enumerate(zip(code_snippets, embeddings))
])

# Search by natural language
query = "find algorithm to sort numbers"
query_emb = model.encode(query)

results = index.query(vector=query_emb.tolist(), top_k=3)
# Returns: bubble_sort function
```

---

## 9. Personalization

### Personalized Content Feed

```python
# User profile from interactions
user_profile = aggregate_user_interactions(user_id)

# Find relevant content
content = index.query(
    vector=user_profile,
    top_k=50,
    filter={
        "published_date": {"$gte": "2024-01-01"},
        "language": "en"
    }
)

# Rank by recency + relevance
ranked_content = rank_content(content, boost_recent=True)
```

---

## 10. Customer Support

### Auto-Ticket Routing

```python
# New support ticket
ticket = {
    "subject": "Payment failed",
    "description": "I tried to pay but got error 500"
}

ticket_embedding = embed_ticket(ticket)

# Find similar past tickets
similar_tickets = index.query(
    vector=ticket_embedding,
    top_k=10
)

# Route to appropriate team based on past resolutions
team = determine_team(similar_tickets)
suggested_solutions = extract_solutions(similar_tickets)

route_ticket(ticket, team=team, suggestions=suggested_solutions)
```

---

## Architecture Patterns

### Pattern 1: Offline Indexing + Online Serving

```
┌────────────────────┐
│  Batch Process     │  ← Index all data (nightly)
│  (Spark/Airflow)   │
└─────────┬──────────┘
          ↓
┌────────────────────┐
│   Vector DB        │
└─────────┬──────────┘
          ↓
┌────────────────────┐
│   API Service      │  ← Real-time queries
│   (FastAPI)        │
└────────────────────┘
```

### Pattern 2: Streaming Indexing

```
┌────────────────────┐
│   Kafka Stream     │  ← Real-time events
└─────────┬──────────┘
          ↓
┌────────────────────┐
│   Processor        │  ← Embed + Index
│   (Flink/Spark)    │
└─────────┬──────────┘
          ↓
┌────────────────────┐
│   Vector DB        │  ← Updated continuously
└────────────────────┘
```

---

## Summary

✅ **Semantic Search:** Understand meaning, not just keywords
✅ **Recommendations:** Find similar items/users
✅ **RAG:** Ground LLM responses in real data
✅ **Image Search:** Text-to-image, reverse image search
✅ **Anomaly Detection:** Find outliers
✅ **Deduplication:** Detect similar/duplicate items
✅ **Personalization:** Tailor content to users

**Continue to Chapter 06 for Performance Optimization! 🚀**
