# Chapter 03: Prompt Engineering

**Mastering the Art of Communicating with LLMs**

---

## Table of Contents
1. [What is Prompt Engineering?](#what-is-prompt-engineering)
2. [Prompt Components](#prompt-components)
3. [Basic Prompting Techniques](#basic-prompting-techniques)
4. [Advanced Techniques](#advanced-techniques)
5. [Best Practices](#best-practices)
6. [Common Mistakes](#common-mistakes)

---

## What is Prompt Engineering?

### Simple Definition

**Prompt Engineering** is the art and science of crafting inputs (prompts) to get the best outputs from LLMs.

### Real-Life Analogy

**Bad Prompt** = Vague Question to Expert
```
You: "Tell me about food"
Expert: "Um... what specifically? History? Recipes? Nutrition?"
Result: Unfocused answer
```

**Good Prompt** = Specific Question
```
You: "Explain the Maillard reaction in cooking and why it makes food taste better, using simple terms"
Expert: [Gives focused, relevant answer]
Result: Exactly what you need
```

---

## Prompt Components

### Basic Structure

```
┌─────────────────────────────────────────┐
│  INSTRUCTION (What to do)              │
│  "Summarize the following article"     │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  CONTEXT (Background information)       │
│  "This is a news article about AI"     │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  INPUT DATA (What to process)           │
│  [Article text goes here...]           │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  OUTPUT INDICATOR (Format specification)│
│  "Provide 3 bullet points"             │
└─────────────────────────────────────────┘
```

---

### Example: Complete Prompt

```
ROLE:
You are an expert data engineer with 10 years of experience.

TASK:
Review the following SQL query for performance issues.

CONTEXT:
This query runs on a PostgreSQL database with 50 million rows.
It's currently taking 45 seconds to execute.

INPUT:
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE YEAR(o.order_date) = 2024

OUTPUT FORMAT:
Provide your response as:
1. Issues found (numbered list)
2. Optimized query
3. Expected performance improvement
```

---

## Basic Prompting Techniques

### 1. Zero-Shot Prompting

**Definition:** Ask without examples

**Example:**

```
Prompt:
"Classify the sentiment of this review: 'This product is amazing!'"

Response:
"Positive"
```

**When to use:**
- Simple tasks
- Model already knows the task
- Quick queries

---

### 2. One-Shot Prompting

**Definition:** Give ONE example

**Example:**

```
Prompt:
"Extract entities from text.

Example:
Input: 'John Smith works at Microsoft in Seattle'
Output: {'name': 'John Smith', 'company': 'Microsoft', 'location': 'Seattle'}

Now extract from:
Input: 'Sarah Johnson is a data engineer at Google in New York'
Output:"

Response:
"{'name': 'Sarah Johnson', 'company': 'Google', 'location': 'New York'}"
```

**When to use:**
- Clarify format
- Show desired output style
- Ambiguous tasks

---

### 3. Few-Shot Prompting

**Definition:** Give MULTIPLE examples

**Example:**

```
Prompt:
"Classify customer support tickets by urgency.

Examples:
'System is down, can't process orders' → CRITICAL
'How do I reset my password?' → LOW
'Payment failed for 100 customers' → HIGH
'Website is slow' → MEDIUM

Now classify:
'Customer data was accidentally exposed'"

Response:
"CRITICAL"
```

**Benefits:**
- More accurate
- Learns patterns
- Handles edge cases better

**Trade-off:** Uses more tokens (costs more)

---

### 4. Chain-of-Thought (CoT) Prompting

**Definition:** Instruct model to "think step by step"

**Example:**

**Without CoT:**
```
Prompt: "What is 23 * 47?"
Response: "1081" (often wrong!)
```

**With CoT:**
```
Prompt:
"What is 23 * 47? Let's think step by step."

Response:
"Let's break this down:
23 * 47
= 23 * (40 + 7)
= (23 * 40) + (23 * 7)
= 920 + 161
= 1081"
```

**Why it works:** Forces model to show reasoning, reduces errors

---

**Real-World Example: Data Validation**

```
Prompt:
"A customer's age is listed as 250 years old. Is this valid?
Let's think step by step."

Response:
"Let me analyze this:
1. Maximum human lifespan is around 120 years
2. 250 years exceeds this by more than double
3. This is clearly an error (typo, system glitch, or invalid input)
4. Most likely causes: Entered birth year instead of age, or data corruption

Conclusion: No, this is NOT valid. Flag for data quality review."
```

---

### 5. Self-Consistency

**Definition:** Generate multiple responses, pick most common answer

**Example:**

```
Question: "How many Rs in 'strawberry'?"

Generate 5 responses:
Response 1: "3" ✓
Response 2: "2"
Response 3: "3" ✓
Response 4: "3" ✓
Response 5: "2"

Majority: "3" (appears 3 times)
Final Answer: "3"
```

**When to use:**
- Critical decisions
- Math problems
- Ambiguous questions

---

## Advanced Techniques

### 1. Role Prompting

**Give model a persona**

**Examples:**

```
BASIC:
"Explain database indexing"

WITH ROLE:
"You are a senior database administrator teaching a junior developer.
Explain database indexing using simple analogies."

Result: More appropriate tone and examples
```

**Roles for data engineering:**
```
- "You are an expert data architect..."
- "You are a Python developer specializing in ETL..."
- "You are a SQL performance tuning consultant..."
- "You are a technical interviewer..."
```

---

### 2. Instruction Following

**Be explicit about requirements**

```
BAD:
"Write a function to process data"

GOOD:
"Write a Python function that:
1. Takes a list of dictionaries as input
2. Filters records where 'status' == 'active'
3. Sorts by 'created_date' descending
4. Returns top 10 records
5. Includes error handling for missing keys
6. Has type hints and docstring"
```

---

### 3. Delimiters for Clarity

**Use markers to separate sections**

```
Prompt:
\"\"\"
Summarize the article between the === markers.

===
[Long article text here...]
===

Requirements:
- 3 sentences maximum
- Focus on key findings
- Use bullet points
\"\"\"
```

**Why useful:** Prevents confusion about what's instruction vs content

---

### 4. Output Structuring

**Specify exact format**

**Example: JSON Output**

```
Prompt:
"Extract information from this email and return as JSON.

Email: 'Hi, this is John Doe (john@example.com). I'm interested in the
Data Engineer position. I have 5 years of experience with Python and SQL.'

JSON format:
{
  \"name\": \"\",
  \"email\": \"\",
  \"position\": \"\",
  \"experience_years\": 0,
  \"skills\": []
}"

Response:
{
  \"name\": \"John Doe\",
  \"email\": \"john@example.com\",
  \"position\": \"Data Engineer\",
  \"experience_years\": 5,
  \"skills\": [\"Python\", \"SQL\"]
}
```

---

### 5. Constraining Output

**Set boundaries**

```
Prompt:
"Explain MapReduce in Hadoop.

Constraints:
- Maximum 100 words
- Use simple language (no jargon)
- Include one real-world analogy
- Do NOT include code examples"
```

**Benefits:**
- Shorter responses (lower cost)
- Focused content
- Prevents hallucination

---

### 6. Iterative Refinement

**Use model output as input for refinement**

```
ITERATION 1:
"Write a SQL query to find top customers"
Result: Basic SELECT with ORDER BY

ITERATION 2:
"Improve this query to handle ties and include customer details:
[Previous query]"
Result: Better query with RANK()

ITERATION 3:
"Optimize for performance on 100M rows:
[Previous query]"
Result: Query with proper indexes suggested
```

---

## Best Practices

### 1. Be Specific

```
❌ BAD:
"Tell me about Spark"

✅ GOOD:
"Explain Spark's shuffle operation, why it's expensive, and 3 ways to minimize shuffles in a production pipeline"
```

---

### 2. Provide Context

```
❌ BAD:
"Is this query fast?"
[SQL query]

✅ GOOD:
"I have a PostgreSQL table with 50M rows, no indexes except primary key.
This query is taking 2 minutes. Is this expected, and how can I optimize it?"
[SQL query]
```

---

### 3. Break Down Complex Tasks

```
❌ BAD:
"Build a complete data pipeline"

✅ GOOD:
Task 1: "Design the architecture for ingesting customer data from REST API"
Task 2: "Write the ETL logic for data transformation"
Task 3: "Create data quality checks"
Task 4: "Design the orchestration workflow"
```

---

### 4. Use Examples (Few-Shot)

```
❌ BAD:
"Convert this to camelCase"

✅ GOOD:
"Convert snake_case to camelCase:
user_name → userName
order_id → orderId
created_at → createdAt

Now convert: customer_email"
```

---

### 5. Specify Output Format

```
❌ BAD:
"List Spark optimization techniques"

✅ GOOD:
"List 5 Spark optimization techniques in this format:
- Technique: [name]
- Problem it solves: [description]
- Example: [code snippet]
- When to use: [scenario]"
```

---

### 6. Handle Ambiguity

```
❌ BAD:
"How do I partition data?"

✅ GOOD:
"I'm working with a 500GB dataset in Parquet format on S3.
Should I partition by date, region, or both?
Consider:
- Queries mostly filter by date (last 30 days)
- Occasionally filter by region (10 regions total)
- Need to balance partition size vs query performance"
```

---

## Common Mistakes

### 1. Being Too Vague

```
❌ "Explain joins"

Why bad: Too broad, unclear what level of detail

✅ "Explain the difference between INNER JOIN and LEFT JOIN using
a customer-orders example. Show the result set for each type."
```

---

### 2. Asking Multiple Questions at Once

```
❌ "What's the difference between batch and stream processing, which is better,
when should I use each, and what tools support them?"

Why bad: Model might only answer part, get confused

✅ Split into separate prompts:
1. "Compare batch vs stream processing (definition, use cases, trade-offs)"
2. "Which tools support batch processing? (Spark, Hadoop, etc.)"
3. "Which tools support stream processing? (Kafka, Flink, etc.)"
```

---

### 3. Not Providing Examples

```
❌ "Parse this log format"
[Log lines]

Why bad: Model doesn't know desired output format

✅ "Parse Apache logs into this JSON format:
Example:
Input: '192.168.1.1 - - [01/Jan/2024:00:00:01] \"GET /api/users\"'
Output: {\"ip\": \"192.168.1.1\", \"timestamp\": \"2024-01-01T00:00:01\", \"method\": \"GET\", \"path\": \"/api/users\"}

Now parse: [Your log lines]"
```

---

### 4. Ignoring Token Limits

```
❌ Prompt with 20,000 words of context

Why bad: Exceeds token limit, gets cut off

✅ Summarize context or split into multiple queries:
"First, summarize this 100-page document"
Then: "Based on that summary, answer: [question]"
```

---

### 5. Not Setting Constraints

```
❌ "Generate test data"

Why bad: Might generate 10,000 rows (expensive, slow)

✅ "Generate 10 sample rows of customer test data with these columns:
customer_id, name, email, signup_date"
```

---

## Data Engineering Prompt Examples

### Example 1: SQL Generation

```
Prompt:
"I have two tables:
- orders (order_id, customer_id, order_date, total_amount)
- customers (customer_id, name, email, country)

Write a SQL query to find:
- Top 10 customers by total purchase amount
- For the last quarter only
- Include customer name and country
- Handle NULL amounts (treat as 0)
- Use PostgreSQL syntax"

Response:
SELECT
    c.customer_id,
    c.name,
    c.country,
    COALESCE(SUM(o.total_amount), 0) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= DATE_TRUNC('quarter', CURRENT_DATE) - INTERVAL '3 months'
   OR o.order_date IS NULL
GROUP BY c.customer_id, c.name, c.country
ORDER BY total_spent DESC
LIMIT 10;
```

---

### Example 2: Data Quality Checks

```
Prompt:
"I have a customer table with columns: customer_id, email, phone, age, country.

Generate Python code using pandas to check for:
1. Duplicate customer_ids
2. Invalid email formats
3. Age outside range 0-120
4. NULL values in required fields
5. Country codes not in ISO list

Return a report showing count of issues by type."

Response:
[Python code with comprehensive data quality checks]
```

---

### Example 3: Architecture Design

```
Prompt:
"Design a data pipeline architecture for this scenario:

Requirements:
- Source: REST API returning JSON (1M records/day)
- Process: Clean, deduplicate, enrich with external data
- Destination: Data warehouse for analytics
- Latency: Hourly updates acceptable
- Scale: Expected to grow 10x in 2 years

Constraints:
- Use AWS services
- Cost-efficient
- Fault-tolerant

Provide:
1. Architecture diagram (text format)
2. Services used and why
3. Data flow explanation
4. Potential bottlenecks"

Response:
[Detailed architecture with S3, Lambda, Glue, Redshift...]
```

---

### Example 4: Debugging Help

```
Prompt:
"My Spark job is failing with this error:
'org.apache.spark.shuffle.FetchFailedException'

Context:
- Processing 500 GB of data
- 100 executors, 4 cores each, 8GB memory per executor
- Job does multiple groupBy operations
- Fails after running for 3 hours

Questions:
1. What causes this error?
2. How do I fix it?
3. How do I prevent it in future?"

Response:
"This is a shuffle-related memory issue...
[Detailed explanation and solutions]"
```

---

## Prompt Templates for Data Engineering

### Template 1: SQL Query Generation

```
"Generate a SQL query for [database type].

Tables and columns:
- [table1]: [columns]
- [table2]: [columns]

Requirements:
- [What you need to query]
- [Filters/conditions]
- [Aggregations]
- [Sorting/limits]

Additional constraints:
- [Performance considerations]
- [Handle edge cases]"
```

---

### Template 2: Code Review

```
"Review this [language] code for a data engineering pipeline.

Code:
```[language]
[Your code here]
```

Review for:
1. Correctness
2. Performance issues
3. Error handling
4. Best practices
5. Security concerns

Provide:
- Issues found (with severity: Critical/High/Medium/Low)
- Specific line numbers
- Recommended fixes
- Improved version of code"
```

---

### Template 3: Troubleshooting

```
"I'm experiencing an issue with [tool/technology].

Problem:
[Describe the issue]

Error message:
[Error text]

Context:
- Environment: [dev/prod/staging]
- Data volume: [size]
- Infrastructure: [specs]
- What I've tried: [previous attempts]

Please provide:
1. Root cause analysis
2. Step-by-step solution
3. How to verify fix
4. How to prevent recurrence"
```

---

## Summary

### Key Takeaways:

✅ **Prompting is a skill** - Practice makes perfect
✅ **Be specific** - More details = better output
✅ **Use examples** - Few-shot prompting improves accuracy
✅ **Structure prompts** - Clear sections (role, task, context, output)
✅ **Iterate** - Refine prompts based on results
✅ **Set constraints** - Control length, format, scope

### Prompt Engineering Checklist:

- [ ] Clear instruction
- [ ] Sufficient context
- [ ] Examples (if needed)
- [ ] Output format specified
- [ ] Constraints defined
- [ ] Ambiguity removed

---

**Continue to Chapter 04 to learn about RAG (Retrieval Augmented Generation)! 🚀**
