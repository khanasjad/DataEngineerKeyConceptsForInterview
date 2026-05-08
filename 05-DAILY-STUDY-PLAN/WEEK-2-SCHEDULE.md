# Week 2: dbt + MLOps + Interview Prep Ramp-Up

**Focus:** Modern Data Stack (dbt, MLOps) + Start Job Applications

**Daily Time:** 5-6 hours
- 2.5 hours: dbt + MLOps (New skills)
- 2 hours: System Design + Coding Practice
- 1 hour: Job applications + Networking

**Goal:** Build dbt + Feature Store projects, Apply to 60+ jobs

---

## Day 8 (Monday): dbt Fundamentals + System Design

### Morning (2.5 hours): dbt Crash Course

**9:00-11:00** - dbt Fundamentals Course
- https://courses.getdbt.com/courses/fundamentals
- Complete Module 1 & 2:
  - What is dbt
  - Models and sources
  - Tests and documentation

**11:00-11:30** - Install dbt
```bash
pip install dbt-core dbt-postgres  # or dbt-databricks
dbt init my_analytics_project
cd my_analytics_project

# Set up profiles.yml (use DuckDB for local testing)
```

### Afternoon (2 hours): System Design Practice

**2:00-4:00** - Study + Practice
- Read: "Designing Data-Intensive Applications" summaries
- Practice: Design a batch ETL pipeline
  - Draw architecture diagram
  - Choose technologies (Kafka/Spark/Airflow/Delta)
  - Identify bottlenecks and optimizations
  - Estimate capacity (throughput, storage)

### Evening (1 hour): Job Applications

**7:00-8:00** - Start applying
- Search LinkedIn for "Senior Data Engineer" (filter: $140k+)
- Apply to 10 jobs (prioritize Easy Apply for volume)
- Save 20 more for tomorrow
- Update spreadsheet: Company, Role, Date Applied, Status

**Deliverable:** dbt installed, 10 applications sent

---

## Day 9 (Tuesday): dbt Project + Coding Practice

### Morning (2.5 hours): Build dbt Project

**9:00-11:30** - E-commerce dbt project
```sql
Project: Transform raw orders data

models/
├── staging/
│   ├── stg_customers.sql
│   ├── stg_orders.sql
│   └── stg_order_items.sql
├── intermediate/
│   └── int_customer_orders.sql
└── marts/
    ├── fct_orders.sql
    └── dim_customers.sql

Tasks:
1. Create sample data (CSV or use dbt tutorial data)
2. Define sources in sources.yml
3. Build 6 models (staging → intermediate → marts)
4. Add tests (unique, not_null, relationships)
5. Add descriptions for documentation
6. Run: dbt run && dbt test && dbt docs generate
```

### Afternoon (2 hours): SQL + Python Coding

**2:00-3:30** - LeetCode Practice
- SQL: 5 Medium problems
  - Focus on window functions
  - Practice CTEs and subqueries
- Python: 3 Medium problems
  - Data structures (dict, set, list)
  - String manipulation

**3:30-4:00** - Review solutions
- Read editorial for each problem
- Note patterns and techniques

### Evening (1 hour): Job Applications + Networking

**7:00-8:00** - Applications + LinkedIn
- Apply to 15 more jobs
- Connect with 5 data engineers at target companies
- Message template: "Hi [Name], I noticed you work on [X] at [Company]. I'm also working with Kafka/Spark and recently built a RAG system with LLMs. Would love to connect!"

**Deliverable:** dbt project with tests, 15 applications

---

## Day 10 (Wednesday): MLOps Fundamentals + Mock Interview

### Morning (2.5 hours): MLOps + Feature Stores

**9:00-10:30** - Study MLOps
- Read: `/01-NEW-SKILLS-HIGH-PRIORITY/MLOps-FeatureStores/README.md`
- Watch: "MLOps Explained" by Databricks (30 min)
- Understand: Feature stores, training/serving skew, model monitoring

**10:30-11:30** - Install Feast
```bash
pip install feast

# Follow Feast quickstart
feast init feature_repo
cd feature_repo
feast apply
```

### Afternoon (2 hours): Mock Interview Practice

**2:00-4:00** - Pramp or Interviewing.io
- Book 1-2 mock interviews
- Focus areas:
  - System design: Real-time data pipeline
  - Coding: SQL + Python
  - Behavioral: STAR method for Optum projects

**Alternative (if no slots):**
- Self-mock using common questions
- Record yourself
- Review and improve

### Evening (1 hour): Company Research

**7:00-8:00** - Target company deep dive
- Pick 5 companies you applied to
- Research:
  - Engineering blog (tech stack)
  - Recent news
  - Interview experiences (Glassdoor, Blind)
- Prepare company-specific questions

**Deliverable:** Feast installed, mock interview done

---

## Day 11 (Thursday): Feature Store Project + System Design

### Morning (2.5 hours): Build Feature Pipeline

**9:00-11:30** - Customer features project
```python
Project: Customer churn prediction features

Tasks:
1. Create sample customer/orders data (CSV)
2. Define features in Feast:
   - total_purchases_30d
   - avg_order_amount
   - days_since_last_order
3. Create feature view
4. Materialize to offline store
5. Retrieve for training:
   features = store.get_historical_features(...)
6. Simulate online serving:
   features = store.get_online_features(...)
```

### Afternoon (2 hours): System Design Deep Dive

**2:00-4:00** - Practice common designs
- Design 1: "Build a real-time recommendation system"
  - Cover: Kafka, Feature Store, Model Serving, Caching
- Design 2: "Design a data warehouse for analytics"
  - Cover: ETL, Dimensional Modeling, Partitioning, Query optimization

**Draw diagrams, estimate capacity, discuss trade-offs**

### Evening (1 hour): Applications + Behavioral Prep

**7:00-7:30** - Apply to 10 more jobs

**7:30-8:00** - Write STAR stories
- S: Situation
- T: Task
- A: Action
- R: Result

**Your Optum stories:**
1. "Optimized Kafka pipeline (40% improvement)"
2. "Scaled real-time streaming to millions of records"
3. "Implemented Azure infrastructure with Terraform"
4. "Led AI integration for pipeline optimization"

**Deliverable:** Feature store demo, STAR stories written

---

## Day 12 (Friday): Integration + Portfolio Polish

### Morning (2.5 hours): Integrate Projects

**9:00-11:30** - Connect dbt + Feast
```python
Project: Unified data pipeline

Architecture:
Raw Data → dbt (transformations) → Feature Store → ML Model

Tasks:
1. Create dbt models that output features
2. Use dbt output as Feast batch source
3. Document the integration
4. Create architecture diagram
5. Write README explaining end-to-end flow
```

### Afternoon (2 hours): Portfolio Polish

**2:00-4:00** - GitHub cleanup
```
Tasks:
1. Create portfolio README
2. Polish all project READMEs
3. Add badges (Python, Apache Spark, etc.)
4. Ensure code is well-commented
5. Add architecture diagrams
6. Write mini-blog posts for each project

Projects:
- RAG Q&A System
- dbt Analytics Pipeline
- Feast Feature Store
```

### Evening (1 hour): Resume Update

**7:00-8:00** - Add all projects to resume
```markdown
PROJECTS SECTION:

AI-Powered Data Quality Assistant
• Built production-grade RAG system using LangChain, Pinecone, GPT-4
• Processed 100+ documents with <500ms latency and 95% retrieval accuracy
• Implemented cost optimization reducing API costs by 40%

Modern Analytics Pipeline with dbt
• Designed dimensional model with fact/dimension tables using dbt
• Implemented automated testing and documentation for 10+ data models
• Integrated with Databricks for scalable transformations

ML Feature Pipeline
• Built feature store using Feast for low-latency model serving
• Implemented both batch and real-time feature computation
• Designed for <100ms feature retrieval supporting ML inference
```

**Deliverable:** Portfolio polished, resume updated

---

## Day 13 (Saturday): Apache Flink (Optional) + Interview Prep

### Morning (2 hours): Apache Flink Basics (Optional)

**9:00-11:00** - Quick Flink overview
- Read: "Apache Flink Introduction"
- Watch: "Flink vs Spark Streaming" (YouTube)
- Understand: Event time, watermarks, state management
- **Note:** Only if time permits, not critical for 1-month goal

**Alternative (if skipping Flink):**
- Extra system design practice
- More coding problems
- Additional mock interview

### Afternoon (2 hours): Comprehensive Review

**2:00-4:00** - Test knowledge
```
Quiz yourself (closed book):

GenAI/LLM:
- Design a RAG system for documentation search
- Explain vector database trade-offs
- How to optimize LLM costs in production?

dbt:
- What's the difference between table and incremental?
- How to test referential integrity?
- Explain dbt project structure

MLOps:
- What is training/serving skew?
- How does a feature store work?
- Design a real-time feature pipeline

Spark:
- How to handle data skew?
- Explain broadcast joins
- When to use cache vs persist?

Kafka:
- How does exactly-once work?
- What causes consumer lag?
- Explain replication and ISR
```

### Evening (2 hours): Behavioral + Leadership Prep

**6:00-8:00** - Prepare leadership answers
- Review your resume "LEADERSHIP" section
- Expand each bullet with STAR format
- Practice answering:
  - "Tell me about a time you led a technical initiative"
  - "Describe a conflict with a team member"
  - "How do you handle tight deadlines?"

**Deliverable:** Self-assessment complete, behavioral prep done

---

## Day 14 (Sunday): Week 2 Review + Week 3 Planning

### Morning (2 hours): Weekly Review & Assessment

**9:00-10:00** - Review progress
- Portfolio check: All projects on GitHub?
- Job applications: 60+ sent?
- Mock interviews: At least 1 completed?
- Knowledge gaps identified?

**10:00-11:00** - Update tracking
```
Week 2 Metrics:
- Projects completed: [RAG, dbt, Feast]
- Applications sent: [Target: 60+]
- Interview requests: [Track callbacks]
- Technical skills: [Rate 1-5]
  - GenAI/RAG: __
  - dbt: __
  - MLOps: __
  - System Design: __
  - Coding: __
```

### Afternoon (3 hours): Week 3 Preparation

**2:00-3:00** - Review interview calendar
- Schedule more mock interviews (target: 5 for Week 3)
- Book time for company-specific prep
- Set up coding practice routine

**3:00-4:00** - System design patterns
- Create template for common designs:
  - Real-time pipeline
  - Batch ETL
  - Data warehouse
  - ML platform
  - Streaming analytics

**4:00-5:00** - Coding practice plan
- Identify weak areas (SQL vs Python)
- Create study plan for Week 3
- Bookmark 30 problems to solve

### Evening (1 hour): Network + Learn from Others

**7:00-8:00** - Join communities
- Join dbt Slack
- Join MLOps community Slack
- Join r/dataengineering Discord
- Engage: Ask 1 question, answer 1 question

**Deliverable:** Week 2 complete, Week 3 plan ready

---

## Week 2 Success Metrics

**New Skills:**
- [ ] dbt project with 6+ models, tests, docs
- [ ] Feature store demo with Feast
- [ ] Understand MLOps concepts (feature engineering, monitoring)
- [ ] dbt integrated with feature pipeline

**Interview Prep:**
- [ ] System design practice (2+ scenarios)
- [ ] Mock interview completed
- [ ] STAR stories written for 5+ situations
- [ ] Company research for 10+ companies

**Coding Practice:**
- [ ] 15+ SQL problems solved
- [ ] 10+ Python problems solved
- [ ] Comfortable with window functions, CTEs

**Job Applications:**
- [ ] 60+ applications sent
- [ ] LinkedIn connections made
- [ ] Resume updated with all projects

**Portfolio:**
- [ ] 3 projects on GitHub
- [ ] All projects documented
- [ ] Architecture diagrams created

---

## Week 2 Adjustment Tips

**If you're crushing it:**
- Start Week 3 materials early
- Apply to 80+ jobs (more volume)
- Do extra mock interviews
- Write LinkedIn posts about projects

**If behind:**
- Prioritize: dbt + Feature store projects
- Skip Flink (not critical)
- Apply to 40+ jobs minimum
- Focus on quality over quantity in projects

**Energy management:**
- Week 2 is intense - take breaks
- Weekend: lighter study, more review
- Stay hydrated, sleep 7+ hours
- Celebrate small wins

---

**Next:** Week 3 = Interview intensive + Live interviews start!
