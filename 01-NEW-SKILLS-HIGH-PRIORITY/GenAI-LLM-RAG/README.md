# GenAI / LLM / RAG - HIGHEST PRIORITY

**Why This Matters:** LLM specialists earn $220k-$280k. This is THE skill for $170k+ roles.

**Interview Focus:**
- What is RAG and why use it vs fine-tuning?
- How do vector databases work?
- LLM prompt engineering best practices
- Production deployment challenges (cost, latency, quality)

---

## Core Concepts to Master (Week 1)

### 1. Large Language Models (LLMs)
**What you need to know:**
- GPT, Claude, Llama architecture basics
- Token limits, context windows
- Fine-tuning vs prompt engineering vs RAG
- When to use which approach

**Interview Questions:**
- "Explain the difference between fine-tuning and RAG. When would you use each?"
- "How would you reduce LLM costs in production?"
- "What are the challenges of deploying LLMs at scale?"

### 2. RAG (Retrieval Augmented Generation)
**What you need to know:**
- RAG pipeline architecture: Indexing → Retrieval → Generation
- Chunking strategies for documents
- Embedding models (OpenAI, sentence-transformers)
- Retrieval methods: Similarity search, hybrid search, re-ranking

**Interview Questions:**
- "Design a RAG system for a company's internal documentation"
- "How would you handle documents larger than the context window?"
- "Explain how you'd optimize RAG for accuracy vs latency"

### 3. Embeddings & Vector Search
**What you need to know:**
- Text → vector embeddings (semantic meaning)
- Cosine similarity, dot product
- Approximate nearest neighbor (ANN) search
- Embedding model selection

**Interview Questions:**
- "Explain how embeddings capture semantic meaning"
- "What's the difference between cosine similarity and dot product?"
- "How do you choose an embedding model?"

---

## Study Resources (Week 1: 3 hours/day)

### Day 1-2: LLM Fundamentals (6 hours)
**Priority 1 - Watch/Read:**
- [DeepLearning.AI] "ChatGPT Prompt Engineering for Developers" (1 hour)
  https://www.deeplearning.ai/short-courses/chatgpt-prompt-engineering-for-developers/

- [YouTube] Andrej Karpathy "Intro to Large Language Models" (1 hour)
  https://www.youtube.com/watch?v=zjkBMFhNj_g

- [Article] "What are LLMs and How Do They Work?" (30 min)
  https://www.analyticsvidhya.com/blog/2023/03/an-introduction-to-large-language-models-llms/

**Hands-on:**
- Use OpenAI API to build simple chatbot (2 hours)
- Experiment with different prompts and parameters (1.5 hours)

### Day 3-4: RAG Deep Dive (6 hours)
**Priority 1 - Watch/Read:**
- [DeepLearning.AI] "Building Applications with Vector Databases" (1.5 hours)
  https://www.deeplearning.ai/short-courses/building-applications-vector-databases/

- [Article] "A Beginner's Guide to RAG" by LangChain (45 min)
  https://python.langchain.com/docs/use_cases/question_answering/

- [YouTube] "RAG from Scratch" by LangChain (1 hour)
  https://www.youtube.com/watch?v=sVcwVQRHIc8

**Hands-on:**
- Build simple RAG with LangChain + ChromaDB (2 hours)
- Test with your resume/work documents (45 min)

### Day 5-6: Vector Databases (6 hours)
**Priority 1 - Watch/Read:**
- [Course] "Vector Databases from Embeddings to Applications" (2 hours)
  https://www.deeplearning.ai/short-courses/vector-databases-embeddings-applications/

- [Docs] Pinecone documentation + tutorials (1.5 hours)
  https://docs.pinecone.io/docs/overview

- [Article] "Vector Database Comparison 2026" (30 min)
  Search: "Pinecone vs Weaviate vs Qdrant 2026"

**Hands-on:**
- Set up Pinecone free tier (30 min)
- Build RAG with Pinecone instead of ChromaDB (1.5 hours)

### Day 7: Integration & Production Patterns (3 hours)
**Priority 1 - Read/Watch:**
- [Article] "LLMs in Production" (45 min)
  https://huyenchip.com/2023/04/11/llm-engineering.html

- [Article] "Prompt Engineering Guide" (1 hour)
  https://www.promptingguide.ai/

**Hands-on:**
- Add prompt versioning to your RAG app (30 min)
- Implement basic caching for cost reduction (45 min)

---

## Key Technologies & Tools

### Must Learn (Week 1):
- **LangChain** - RAG framework (Python)
- **OpenAI API** - GPT models
- **ChromaDB** - Local vector database
- **Pinecone** - Production vector database
- **Hugging Face** - Open source models & embeddings

### Good to Know (Week 2):
- **LlamaIndex** - Alternative to LangChain
- **Weaviate** - Vector database alternative
- **Anthropic Claude** - GPT alternative
- **sentence-transformers** - Open source embeddings

---

## Hands-On Project: Build This Week

### Project: "Tech Documentation Q&A System"
**Build a RAG system that answers questions about technical documentation**

**Requirements:**
1. Ingest 50+ documentation pages (use your company docs or open source)
2. Chunk documents intelligently (500-1000 tokens)
3. Generate embeddings and store in Pinecone
4. Implement RAG pipeline with LangChain
5. Add prompt template for technical answers
6. Track: query latency, cost per query, answer quality

**Tech Stack:**
- Python + LangChain
- OpenAI GPT-4 or GPT-3.5-turbo
- Pinecone vector database
- Streamlit for simple UI (optional)

**Time:** 8-10 hours spread over Week 1

**Outcome:**
- Working demo on GitHub
- Add to resume: "Built production-grade RAG system processing 50+ documents"
- Interview talking point about handling embeddings, chunking strategy, cost optimization

---

## Interview Preparation - Common Questions

### Conceptual Questions:
1. **What is RAG and when would you use it?**
   - Answer: Retrieval Augmented Generation combines retrieval from knowledge base with LLM generation. Use when you need factual, up-to-date information not in LLM training data. Better than fine-tuning for frequently changing data.

2. **How do vector databases differ from traditional databases?**
   - Answer: Vector DBs store high-dimensional embeddings and perform similarity search using ANN algorithms. Traditional DBs use exact matching. Vector DBs enable semantic search - finding similar meaning, not just keywords.

3. **What are the main challenges in production LLM systems?**
   - Answer: Cost (API calls add up), latency (user experience), quality/hallucinations (need validation), context limits (chunking strategy), and data privacy (sensitive data to LLM).

### System Design Questions:
1. **Design a customer support chatbot using RAG**
   - Cover: Document ingestion, embedding generation, vector storage, retrieval strategy, prompt design, response validation, monitoring

2. **How would you optimize a RAG system for low latency?**
   - Answer: Caching (repeated queries), faster embedding models, vector DB optimization (indexes), parallel retrieval, streaming responses, prompt compression

3. **Design an LLM-powered data quality assistant**
   - Cover: Schema ingestion, data profiling integration, anomaly detection with LLM reasoning, RAG for historical context, alert generation

---

## Metrics for Week 1 Success

**Knowledge Check:**
- [ ] Can explain RAG pipeline end-to-end
- [ ] Understand embeddings and vector search
- [ ] Know when to use fine-tuning vs RAG vs prompting
- [ ] Familiar with LangChain basics
- [ ] Can discuss production challenges (cost, latency, quality)

**Hands-On:**
- [ ] Built working RAG demo
- [ ] Used OpenAI API
- [ ] Set up Pinecone or ChromaDB
- [ ] Implemented chunking strategy
- [ ] Added basic prompt engineering

**Interview Ready:**
- [ ] Can design RAG system on whiteboard
- [ ] Have concrete project to discuss
- [ ] Know cost optimization strategies
- [ ] Understand trade-offs (accuracy vs latency vs cost)

---

## Quick Reference Cheat Sheet

**RAG Pipeline:**
```
1. INDEXING (Offline)
   Documents → Chunking → Embeddings → Vector DB

2. RETRIEVAL (Runtime)
   User Query → Embedding → Similarity Search → Top K chunks

3. GENERATION (Runtime)
   Prompt Template + Retrieved Chunks + Query → LLM → Answer
```

**Key Terminology:**
- **Embedding:** Vector representation of text (e.g., 1536 dimensions for OpenAI)
- **Chunking:** Breaking documents into smaller pieces (500-1000 tokens typical)
- **Similarity Search:** Finding vectors closest to query vector
- **Context Window:** Max tokens LLM can process (e.g., 128k for GPT-4)
- **Hallucination:** LLM generating false information
- **Prompt Template:** Structured prompt with placeholders

**Cost Optimization:**
- Use cheaper models for simple tasks (GPT-3.5 vs GPT-4)
- Cache common queries
- Optimize chunk retrieval (retrieve less if possible)
- Use open source models for non-critical paths

---

**Next:** Move to `Vector-Databases/` folder for deeper dive on vector DBs
