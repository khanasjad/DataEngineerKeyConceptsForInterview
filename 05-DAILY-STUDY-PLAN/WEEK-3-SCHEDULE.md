# Week 3: Interview Intensive + System Design Mastery

**Focus:** Mock Interviews + Live Interviews Begin + System Design Mastery

**Daily Time:** 6 hours
- 2 hours: System Design Practice
- 2 hours: Coding Practice (SQL + Python)
- 1 hour: Behavioral + Company Prep
- 1 hour: Mock/Live Interviews

**Goal:** Complete 10+ mock interviews, Start live interviews, Master system design

---

## Day 15 (Monday): System Design Patterns + Mock Interview

### Morning (2 hours): System Design - Real-Time Pipeline

**9:00-11:00** - Master real-time architecture
```
Problem: "Design a real-time analytics dashboard for e-commerce"

Requirements:
- Track user events (page views, clicks, purchases)
- Show metrics updated every 30 seconds
- Handle 10K events/sec peak
- 7-day data retention

Your Solution Should Cover:
1. Data ingestion (Kafka)
2. Stream processing (Spark Streaming / Flink)
3. Storage (Delta Lake / Time-series DB)
4. Serving layer (Redis cache)
5. Visualization (BI tool)

Discuss:
- Partitioning strategy
- Exactly-once semantics
- Failure handling
- Scaling approach
- Cost optimization

Practice:
- Draw architecture on whiteboard/paper
- Explain each component
- Estimate capacity (throughput, storage)
- Identify bottlenecks
- Time yourself: 35-40 minutes
```

### Afternoon (2 hours): Coding Practice - SQL Intensive

**2:00-4:00** - LeetCode SQL Marathon
```
Solve 10 SQL problems:

Easy (warm-up):
- Employees Earning More Than Managers
- Combine Two Tables
- Duplicate Emails

Medium (focus here):
- Nth Highest Salary
- Department Highest Salary
- Consecutive Numbers
- Exchange Seats
- Rank Scores

Hard (stretch):
- Trips and Users
- Department Top Three Salaries

Technique: For each problem
1. Understand requirements (5 min)
2. Write solution (10-15 min)
3. Test with examples (5 min)
4. Review optimal solution (5 min)
```

### Evening (2 hours): Mock Interview + Behavioral

**7:00-8:00** - Schedule mock interview on Pramp / Interviewing.io
- Request: System design focus
- Scenario: "Design a batch ETL pipeline"
- Practice explaining trade-offs
- Take notes on feedback

**8:00-9:00** - Behavioral question practice
```
Common questions + Your STAR answers:

1. "Tell me about a challenging project"
   → Optum RQNS platform architecture

2. "Describe a time you optimized performance"
   → Kafka pipeline optimization (40% improvement)

3. "How do you handle disagreements with team members?"
   → [Prepare specific example]

4. "Tell me about a failure and what you learned"
   → [Prepare specific example]

Practice: Record yourself, review, improve clarity
```

**Deliverable:** 10 SQL problems solved, 1 mock interview done

---

## Day 16 (Tuesday): Batch ETL Design + Python Coding

### Morning (2 hours): System Design - Data Warehouse

**9:00-11:00** - Master batch ETL architecture
```
Problem: "Design a data warehouse for a healthcare company"

Requirements:
- Ingest data from 10+ sources (databases, APIs, files)
- Process 5TB daily
- Support analytics queries (<5 sec)
- Historical data (5 years)
- HIPAA compliance

Your Solution Should Cover:
1. Ingestion layer (Airflow DAGs, Kafka Connect)
2. Storage (Data Lake: S3/ADLS + Delta Lake)
3. Processing (Spark for transformations)
4. Modeling (Star schema: facts + dimensions)
5. Serving (Databricks SQL / Snowflake)
6. Governance (Unity Catalog, data lineage)

Discuss:
- ETL vs ELT
- Partitioning strategy (by date)
- Slowly Changing Dimensions (Type 2)
- Data quality checks
- Security (encryption, access control)

Healthcare-specific:
- PHI data handling
- Audit logging
- De-identification

Practice on whiteboard, time: 40 minutes
```

### Afternoon (2 hours): Python Coding Marathon

**2:00-4:00** - LeetCode Python
```
Solve 8 problems:

Easy (warm-up):
- Two Sum
- Valid Parentheses
- Merge Two Sorted Lists

Medium (focus):
- Group Anagrams
- 3Sum
- Longest Substring Without Repeating Characters
- Container With Most Water
- Product of Array Except Self

Focus on:
- Hash maps / dictionaries
- Two pointers technique
- Sliding window
- Time/space complexity

For each: Code + explain approach + complexity analysis
```

### Evening (2 hours): Company-Specific Prep + Applications

**7:00-8:00** - Research 5 companies you're interviewing with
```
For each company:
1. Engineering blog → recent projects, tech stack
2. Glassdoor → interview experiences
3. LinkedIn → find data engineers, their backgrounds
4. Prepare 3 company-specific questions:
   - About their data platform
   - About team structure
   - About growth opportunities
```

**8:00-9:00** - Apply to 20 more jobs
- Focus on companies with recent funding / growth
- Tailor resume for each (highlight relevant skills)
- Custom cover letter for top 5 targets

**Deliverable:** 8 Python problems solved, company research done

---

## Day 17 (Wednesday): ML Platform Design + Live Interview

### Morning (2 hours): System Design - ML Platform

**9:00-11:00** - Master ML infrastructure design
```
Problem: "Design an ML platform for fraud detection"

Requirements:
- Real-time predictions (<100ms latency)
- 50K transactions/sec peak
- 99.9% availability
- Model retraining weekly
- A/B testing support

Your Solution Should Cover:
1. Feature Pipeline:
   - Batch features (Spark → Feature Store)
   - Real-time features (Kafka → Flink → Redis)

2. Model Training:
   - Feature retrieval (Feast offline store)
   - Experiment tracking (MLflow)
   - Model registry

3. Model Serving:
   - REST API (FastAPI on Kubernetes)
   - Feature lookup (Redis)
   - Prediction + logging

4. Monitoring:
   - Data drift (feature distributions)
   - Model performance (precision/recall)
   - System metrics (latency, throughput)

5. Feedback Loop:
   - Labeled data → retraining
   - Model updates (canary deployment)

Discuss:
- Training/serving skew prevention
- Feature consistency (batch vs real-time)
- Model versioning
- Rollback strategy
- Cost optimization

This is advanced - combines multiple skills!
Time: 45 minutes
```

### Afternoon (2 hours): Live Interview (if scheduled)

**2:00-4:00** - First live interview!
```
Preparation (1 hour before):
- Review company research
- Review your projects (be ready to explain)
- Review STAR stories
- Prepare questions to ask

During interview:
- Think out loud
- Ask clarifying questions
- Draw diagrams
- Discuss trade-offs
- Be enthusiastic!

After interview:
- Send thank-you email within 24 hours
- Note questions asked (for future prep)
- Reflect on what went well / improve
```

**If no live interview yet:**
- Do 2 mock interviews instead
- Practice system design + coding
- Review common questions

### Evening (2 hours): Coding Practice + Reflection

**7:00-8:30** - Mixed coding practice
- 3 SQL problems (medium/hard)
- 2 Python problems (medium)
- Focus on areas where you struggled in mock interviews

**8:30-9:00** - Reflect and adjust
- Review mock interview feedback
- Identify weak areas
- Adjust study plan for rest of week

**Deliverable:** 1 live/mock interview, 5 problems solved

---

## Day 18 (Thursday): Streaming Design + Interview Prep

### Morning (2 hours): System Design - Streaming Analytics

**9:00-11:00** - Design streaming system
```
Problem: "Design a real-time recommendation system"

Requirements:
- Recommend products based on user behavior
- Update recommendations in real-time
- Personalized for 10M users
- Handle 100K events/sec

Your Solution Should Cover:
1. Event Collection:
   - User events → Kafka (clicks, views, purchases)

2. Feature Engineering:
   - Real-time: User session features (Flink)
   - Batch: User history features (Spark)
   - Feature Store (Feast): Combine both

3. Model Serving:
   - Pre-computed recommendations (batch job)
   - Real-time ranking (online model)
   - Caching (Redis)

4. Serving API:
   - GET /recommendations/user/{id}
   - <50ms p99 latency
   - Load balancing

5. Feedback Loop:
   - Click events → retrain models
   - A/B testing framework

Discuss:
- Cold start problem (new users)
- Real-time vs batch recommendations
- Caching strategy
- Personalization vs performance trade-off

Practice explaining to non-technical stakeholder too!
```

### Afternoon (2 hours): Mock Interview

**2:00-4:00** - Two mock interviews (1 hour each)
```
Interview 1: System Design
- "Design a metrics collection system like Datadog"
- Practice explaining monitoring, time-series data, aggregations

Interview 2: Coding
- SQL + Python mixed
- Practice explaining thought process
- Focus on communication, not just correctness
```

### Evening (2 hours): Behavioral Deep Dive

**7:00-9:00** - Master behavioral questions
```
Prepare answers for 20 common questions:

Leadership:
1. Tell me about a time you led a project
2. How do you handle underperforming team members?
3. Describe a time you had to influence without authority

Problem-Solving:
4. Tell me about a difficult technical problem you solved
5. Describe a time you failed
6. How do you approach debugging production issues?

Collaboration:
7. Describe a conflict with a colleague
8. Tell me about a time you disagreed with a decision
9. How do you handle tight deadlines?

Technical:
10. What's the most complex system you've built?
11. How do you stay updated with new technologies?
12. Tell me about a trade-off you made

Your Optum Examples:
- RQNS platform architecture
- Kafka optimization (40% improvement)
- Azure infrastructure automation
- AI integration for data pipelines

Practice: Answer each in 2-3 minutes using STAR format
Record yourself, ensure concise and clear
```

**Deliverable:** 2 mock interviews, behavioral answers polished

---

## Day 19 (Friday): Interview Blitz + Portfolio Final Polish

### Morning (2 hours): Rapid-Fire Practice

**9:00-11:00** - Speed drill
```
Set timer: 30 minutes per design

Design 4 systems back-to-back:
1. "Design Uber's surge pricing system"
   - Real-time demand/supply tracking
   - Dynamic pricing algorithm
   - Kafka, Flink, Redis

2. "Design Airbnb's search system"
   - Full-text + filters
   - Elasticsearch, ranking
   - Caching strategy

3. "Design Netflix recommendation pipeline"
   - Collaborative filtering
   - Batch + real-time features
   - A/B testing

4. "Design a log aggregation system"
   - Kafka, Elasticsearch, Kibana
   - Log parsing, indexing
   - Alerting

Goal: Get comfortable designing quickly
Don't deep dive, cover breadth
```

### Afternoon (2 hours): Final Portfolio Polish

**2:00-4:00** - Make portfolio shine
```
Tasks:
1. Create portfolio website (GitHub Pages) or:
   - Awesome README with project links

2. Each project should have:
   - Clear description
   - Architecture diagram
   - Key technologies
   - Results/metrics
   - Code snippets
   - How to run

3. LinkedIn update:
   - Post about RAG project
   - Post about feature store project
   - Include visuals (diagrams, screenshots)

4. Resume final polish:
   - Quantify everything
   - Action verbs
   - Tailored for each application

5. Prepare portfolio walkthrough:
   - 5-minute presentation per project
   - Practice explaining technical decisions
   - Be ready for deep-dive questions
```

### Evening (2 hours): Interview Scenarios

**7:00-9:00** - Practice full interview loop
```
Simulate on-site interview (4 rounds):

Round 1: Coding (30 min)
- 1 Medium SQL + 1 Medium Python
- Solve under time pressure

Round 2: System Design (45 min)
- Design data pipeline for your domain
- Full whiteboard walkthrough

Round 3: Behavioral (30 min)
- Answer 10 behavioral questions
- Practice asking good questions back

Round 4: Technical Deep Dive (30 min)
- Deep dive on Optum projects
- Kafka, Spark, Azure questions
- Be ready to explain architecture decisions

Review: How did you do?
- Time management OK?
- Communication clear?
- Showed enthusiasm?
```

**Deliverable:** Portfolio published, full mock interview done

---

## Day 20-21 (Weekend): Review + Live Interviews

### Saturday (4 hours): Comprehensive Review

**9:00-11:00** - Knowledge check
```
Test yourself on everything:

GenAI/LLM (15 min):
- Explain RAG end-to-end
- Vector database trade-offs
- LLM cost optimization

dbt (10 min):
- Project structure
- Incremental models
- Testing strategy

MLOps (15 min):
- Feature store architecture
- Training/serving skew
- Model monitoring

Kafka (20 min):
- Exactly-once semantics
- Consumer group rebalancing
- Performance optimization

Spark (20 min):
- Shuffle optimization
- Delta Lake merge
- Broadcast joins

System Design (30 min):
- Design 2 systems from scratch
- Focus on communication clarity

Coding (30 min):
- 2 SQL + 2 Python problems
- No looking at solutions!
```

**2:00-4:00** - Gap filling
- Review anything unclear
- Re-do problems you struggled with
- Update cheat sheets

### Sunday (4 hours): Final Preparation

**9:00-11:00** - Company-specific prep
```
For each company you're interviewing with:
1. Research engineering team (LinkedIn)
2. Read engineering blog thoroughly
3. Understand their tech stack
4. Prepare 5 thoughtful questions
5. Understand their product/business
```

**2:00-4:00** - Relax and light review
- Review flashcards
- Watch a tech talk (inspiration)
- Light coding practice (stay sharp)
- Prepare interview outfits/setup
- Get good sleep!

**Deliverable:** Ready for Week 4 interview blitz!

---

## Week 3 Success Metrics

**Interview Practice:**
- [ ] 10+ mock interviews completed
- [ ] 2-5 live interviews completed
- [ ] Comfortable with system design (can design in 35-40 min)
- [ ] Behavioral stories polished and confident

**Coding:**
- [ ] 30+ SQL problems solved (total)
- [ ] 25+ Python problems solved (total)
- [ ] Can solve medium problems in 20-30 minutes

**System Design Mastery:**
- [ ] Can design: Real-time pipeline, Batch ETL, ML platform, Streaming analytics
- [ ] Understand trade-offs and can explain clearly
- [ ] Practice estimating capacity

**Portfolio:**
- [ ] 3 projects polished on GitHub
- [ ] LinkedIn updated with projects
- [ ] Portfolio ready to present

**Applications:**
- [ ] 100+ total applications sent
- [ ] Active interview pipeline (5-10 companies)
- [ ] Tracking all interviews in spreadsheet

---

## Week 3 Tips

**Interview Energy:**
- This is intense week - pace yourself
- Sleep 7-8 hours daily
- Take breaks between practice sessions
- Stay positive - you're well-prepared!

**If interviews not yet scheduled:**
- Keep applying aggressively
- More mock interviews (10+ total)
- Reach out to recruiters directly
- Leverage network for referrals

**Communication is key:**
- In interviews, think out loud
- Ask clarifying questions
- Explain trade-offs
- Show enthusiasm and curiosity

---

**Next:** Week 4 = Interview blitz + Offer negotiations!
