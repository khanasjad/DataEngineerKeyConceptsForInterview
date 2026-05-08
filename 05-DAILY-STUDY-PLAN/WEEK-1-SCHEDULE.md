# Week 1: GenAI/LLM + Core Skills Revision

**Focus:** Learn GenAI/RAG (NEW) + Revise Kafka/Spark (EXISTING)

**Daily Time:** 5 hours
- 3 hours: GenAI/LLM/RAG (New skills)
- 2 hours: Kafka/Spark revision + Interview prep

**Goal:** Build 1 RAG project + Sharpen existing skills

---

## Day 1 (Monday): LLM Fundamentals + Kafka Revision

### Morning (3 hours): LLM Basics

**9:00-10:00** - Watch "ChatGPT Prompt Engineering for Developers" (DeepLearning.AI)
- https://www.deeplearning.ai/short-courses/chatgpt-prompt-engineering-for-developers/

**10:00-11:00** - Watch Andrej Karpathy "Intro to Large Language Models"
- https://www.youtube.com/watch?v=zjkBMFhNj_g
- Take notes on: tokens, context windows, temperature, top-p

**11:00-12:00** - Hands-on: OpenAI API
```python
# Tasks:
1. Sign up for OpenAI API (get free credits)
2. Install openai library: pip install openai
3. Build simple chatbot (5-10 prompts)
4. Experiment with parameters (temperature, max_tokens)
5. Test different system prompts
```

### Evening (2 hours): Kafka Interview Prep

**7:00-8:00** - Review Kafka architecture
- Read: `/02-REVISION-EXISTING-SKILLS/Kafka/INTERVIEW-PREP.md` (Questions 1-6)
- Draw architecture diagram from memory
- Practice explaining: partitions, consumer groups, offsets

**8:00-9:00** - Practice interview questions
- Answer Questions 1-6 out loud (as if in interview)
- Write down your Optum Kafka experience (specific examples)
- Prepare metrics: throughput, latency, lag numbers

**Deliverable:** OpenAI API chatbot running, Kafka notes

---

## Day 2 (Tuesday): RAG Fundamentals + Spark Revision

### Morning (3 hours): RAG Deep Dive

**9:00-10:30** - Watch "Building Applications with Vector Databases" (DeepLearning.AI)
- https://www.deeplearning.ai/short-courses/building-applications-vector-databases/

**10:30-11:00** - Read "Beginner's Guide to RAG" (LangChain)
- https://python.langchain.com/docs/use_cases/question_answering/
- Understand RAG pipeline: Indexing → Retrieval → Generation

**11:00-12:00** - Install dependencies
```bash
pip install langchain openai chromadb tiktoken

# Test installation
python -c "import langchain; import chromadb; print('Success!')"
```

### Evening (2 hours): Spark Interview Prep

**7:00-8:00** - Review Spark architecture and optimization
- Read: `/02-REVISION-EXISTING-SKILLS/Spark-Databricks/INTERVIEW-PREP.md` (Q1-8)
- Practice explaining: lazy evaluation, DAG, shuffles

**8:00-9:00** - Review Delta Lake
- Read Delta Lake questions (Q10-12)
- Write out Delta merge syntax from memory
- Prepare Databricks talking points

**Deliverable:** LangChain installed, Spark notes

---

## Day 3 (Wednesday): Build RAG Demo + System Design Prep

### Morning (3 hours): RAG Hands-On Project

**9:00-12:00** - Build simple RAG system
```python
# Project: Document Q&A with your resume

Tasks:
1. Load your resume PDF (use PyPDF2)
2. Chunk into 500-token pieces
3. Generate embeddings (OpenAI)
4. Store in ChromaDB
5. Implement query function:
   - User question → embedding
   - Retrieve top 3 chunks
   - Send to GPT with prompt template
6. Test with 5 questions about your experience
```

**Resources:**
- LangChain quickstart docs
- `/01-NEW-SKILLS-HIGH-PRIORITY/GenAI-LLM-RAG/README.md`

### Evening (2 hours): Data System Design

**7:00-9:00** - Study system design patterns
- Read: "Designing Data-Intensive Applications" Chapter 1 (online summary)
- Practice: Design Kafka-based real-time pipeline
- Draw architecture diagram for Optum RQNS project

**Deliverable:** Working RAG demo with resume

---

## Day 4 (Thursday): Vector Databases + Coding Practice

### Morning (3 hours): Vector Database Deep Dive

**9:00-10:00** - Read Vector Database concepts
- `/01-NEW-SKILLS-HIGH-PRIORITY/Vector-Databases/README.md`
- Focus on: ANN algorithms, distance metrics, use cases

**10:00-11:00** - Set up Pinecone
```python
Tasks:
1. Sign up for Pinecone free tier
2. Create index
3. Follow quickstart tutorial
4. Index 100 sample documents
5. Query with similarity search
```

**11:00-12:00** - Upgrade RAG demo
```python
Replace ChromaDB with Pinecone:
1. Migrate vectors to Pinecone
2. Update query logic
3. Add metadata filtering
4. Compare performance
```

### Evening (2 hours): SQL Practice

**7:00-9:00** - LeetCode SQL
- Solve 5 Medium SQL problems
- Focus on: JOINs, window functions, CTEs
- Examples:
  - "Nth Highest Salary"
  - "Department Top Three Salaries"
  - "Consecutive Numbers"

**Websites:**
- LeetCode SQL 50 questions
- HackerRank SQL (Medium level)

**Deliverable:** RAG with Pinecone, 5 SQL problems solved

---

## Day 5 (Friday): Production RAG + Python Practice

### Morning (3 hours): Enhance RAG Project

**9:00-12:00** - Add production features
```python
Enhancements:
1. Prompt engineering:
   - Create structured prompt template
   - Add "explain your sources" instruction
   - Handle "I don't know" responses

2. Cost tracking:
   - Log token usage per query
   - Calculate cost
   - Add caching for repeated queries

3. Evaluation:
   - Create 10 test questions
   - Measure answer quality
   - Track retrieval accuracy

4. Documentation:
   - Write README.md
   - Add architecture diagram
   - Document design decisions
```

### Evening (2 hours): Python Coding Practice

**7:00-9:00** - LeetCode Python
- Solve 3 Medium problems
- Focus on: data structures, algorithms
- Examples:
  - "Two Sum" variations
  - "Valid Parentheses"
  - "Merge Intervals"

**Deliverable:** Enhanced RAG project, Python practice

---

## Day 6 (Saturday): Project Finalization + Mock Interview

### Morning (3 hours): Finalize RAG Project

**9:00-10:30** - Polish and deploy
```python
Tasks:
1. Add simple Streamlit UI (optional)
2. Push to GitHub with detailed README
3. Add requirements.txt
4. Create demo video/screenshots
5. Write blog post draft (LinkedIn)
```

**10:30-12:00** - Prepare interview talking points
```
Practice explaining:
- Why RAG over fine-tuning?
- How does vector search work?
- What are the cost/latency trade-offs?
- How would you scale this to production?

Prepare metrics:
- # of documents indexed
- Average query latency
- Cost per query
- Retrieval accuracy
```

### Afternoon (2 hours): Mock Interview Practice

**2:00-4:00** - Self mock interview
- Record yourself answering:
  - "Tell me about your Kafka experience"
  - "Design a real-time analytics pipeline"
  - "Explain a complex Spark optimization you did"
  - "What is RAG and when would you use it?"
- Watch recording, note improvements

**Deliverable:** GitHub project, mock interview practice

---

## Day 7 (Sunday): Review + Weekly Assessment

### Morning (2 hours): Weekly Review

**9:00-10:00** - Test yourself (closed book)
```
GenAI/LLM:
- What is RAG?
- Explain vector embeddings
- When to use fine-tuning vs RAG?
- How does Pinecone differ from Postgres?

Kafka:
- Explain consumer groups
- What causes consumer lag?
- How does exactly-once semantics work?

Spark:
- What is a shuffle?
- How to optimize broadcast joins?
- Explain Delta Lake benefits
```

**10:00-11:00** - Review mistakes
- Go back to docs for anything unclear
- Update your notes
- Create flashcards for key concepts

### Afternoon (3 hours): Prepare for Week 2

**2:00-3:00** - Plan Week 2 projects
- Sketch feature store architecture
- List dbt models to build
- Identify datasets to use

**3:00-4:00** - Job applications setup
- Update LinkedIn headline: "Data Engineer | Kafka, Spark, Azure, GenAI/LLM"
- Set up job alerts (LinkedIn, Indeed)
- Prepare list of 50 target companies

**4:00-5:00** - Resume update
```markdown
Add to resume:
• Built RAG-powered Q&A system using LangChain, Pinecone, and GPT-4,
  processing 100+ documents with sub-500ms latency
• Implemented vector database for semantic search with 95%+ retrieval accuracy
```

**Deliverable:** Week 1 assessment complete, Week 2 plan ready

---

## Week 1 Success Metrics

**Knowledge:**
- [ ] Can explain LLMs, RAG, vector databases
- [ ] Understand Kafka architecture and optimization
- [ ] Know Spark performance tuning

**Projects:**
- [ ] RAG demo on GitHub
- [ ] Working Pinecone integration
- [ ] Documentation and README

**Practice:**
- [ ] 5+ SQL problems solved
- [ ] 3+ Python problems solved
- [ ] Mock interview completed

**Job Search:**
- [ ] LinkedIn updated
- [ ] Job alerts set up
- [ ] Resume updated with RAG project

---

## Daily Checklist Template

**Morning (3 hours):**
- [ ] Study new skill (GenAI/LLM/Vector DB)
- [ ] Hands-on coding/project work
- [ ] Document learnings

**Evening (2 hours):**
- [ ] Review existing skills (Kafka/Spark/Azure)
- [ ] Practice coding (SQL/Python)
- [ ] Interview question practice

**Before Bed:**
- [ ] Update learning journal
- [ ] Review tomorrow's plan
- [ ] 15-min review of flashcards

---

## Adjustment Tips

**If ahead of schedule:**
- Start Week 2 materials (dbt, MLOps)
- Do extra coding practice
- Apply to 5-10 jobs

**If behind schedule:**
- Focus on RAG project (highest priority)
- Skip optional readings
- Reduce coding practice to 3 problems

**Stay motivated:**
- GenAI/LLM skills = $220k+ salary potential
- RAG project = huge interview differentiator
- You have strong fundamentals, just adding premium skills

---

**Next:** Week 2 focuses on MLOps, dbt, and intensive interview prep!
