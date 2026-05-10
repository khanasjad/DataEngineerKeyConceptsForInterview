# Chapter 06: Advanced GenAI Topics

**Fine-Tuning, Agents, Function Calling, and Production Deployment**

---

## Table of Contents
1. [Fine-Tuning LLMs](#fine-tuning-llms)
2. [LLM Agents](#llm-agents)
3. [Function Calling](#function-calling)
4. [Multimodal Models](#multimodal-models)
5. [Production Deployment](#production-deployment)
6. [Cost Optimization](#cost-optimization)

---

## Fine-Tuning LLMs

### What is Fine-Tuning?

**Definition:** Further training a pre-trained model on specific data to specialize it.

### When to Fine-Tune

✅ **Fine-tune when:**
- Need specific behavior/style
- Domain-specific language (medical, legal)
- Consistent output format
- Have sufficient training data (1000+ examples)

❌ **Don't fine-tune when:**
- Need latest information → Use RAG
- Small dataset (<100 examples) → Use few-shot prompting
- General tasks → Use base model

---

### Fine-Tuning Process

```
Step 1: Prepare Training Data
[
  {"prompt": "Translate to SQL: Show top 10 customers", "completion": "SELECT ..."},
  {"prompt": "Optimize this query: SELECT *...", "completion": "Use LIMIT..."},
  ...
]

Step 2: Upload to Platform
↓

Step 3: Start Training
- Base model: GPT-3.5
- Training data: 1000 examples
- Epochs: 3
- Learning rate: Auto
↓

Step 4: Evaluate
- Test on holdout set
- Compare to base model
↓

Step 5: Deploy
- Use fine-tuned model ID
- Monitor performance
```

---

### Fine-Tuning Example (OpenAI)

```python
from openai import OpenAI
client = OpenAI()

# 1. Prepare training file (JSONL format)
training_data = [
    {
        "messages": [
            {"role": "system", "content": "You are a SQL expert"},
            {"role": "user", "content": "Write query for top 10 customers"},
            {"role": "assistant", "content": "SELECT customer_id, SUM(amount) as total FROM orders GROUP BY customer_id ORDER BY total DESC LIMIT 10"}
        ]
    },
    # ... more examples
]

# 2. Upload file
file = client.files.create(
    file=open("training_data.jsonl", "rb"),
    purpose="fine-tune"
)

# 3. Create fine-tuning job
job = client.fine_tuning.jobs.create(
    training_file=file.id,
    model="gpt-3.5-turbo"
)

# 4. Wait for completion
# Check status: client.fine_tuning.jobs.retrieve(job.id)

# 5. Use fine-tuned model
response = client.chat.completions.create(
    model=job.fine_tuned_model,  # e.g., "ft:gpt-3.5-turbo:org:id"
    messages=[{"role": "user", "content": "Write query for..."}]
)
```

---

## LLM Agents

### What is an Agent?

**Definition:** An LLM that can use tools and make decisions autonomously to accomplish tasks.

### Simple vs Agent

**Simple LLM:**
```
User: "What's the weather in Paris?"
LLM: "I don't have real-time information"
```

**LLM Agent:**
```
User: "What's the weather in Paris?"

Agent thinking:
1. I need current weather data
2. I have access to weather API tool
3. Let me call weather_api(city="Paris")
4. [API returns: 18°C, Sunny]
5. Generate response with this data

Agent: "The weather in Paris is currently 18°C and sunny"
```

---

### Agent Components

```
┌──────────────────────────────────┐
│  1. LLM (Brain)                  │
│  - Reasoning                     │
│  - Decision making               │
└────────────┬─────────────────────┘
             ↓
┌──────────────────────────────────┐
│  2. Tools (Hands)                │
│  - Search engine                 │
│  - Calculator                    │
│  - Database query                │
│  - APIs                          │
└────────────┬─────────────────────┘
             ↓
┌──────────────────────────────────┐
│  3. Memory (Notes)               │
│  - Conversation history          │
│  - Intermediate results          │
└──────────────────────────────────┘
```

---

### Agent Example (LangChain)

```python
from langchain.agents import initialize_agent, Tool
from langchain.llms import OpenAI
from langchain.sql_database import SQLDatabase

# Define tools
tools = [
    Tool(
        name="SQL_Query",
        func=lambda q: sql_database.run(q),
        description="Execute SQL query. Input should be valid SQL."
    ),
    Tool(
        name="Calculator",
        func=lambda expr: eval(expr),
        description="Calculate math expressions"
    )
]

# Initialize agent
llm = OpenAI(temperature=0)
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent="zero-shot-react-description",
    verbose=True
)

# Use agent
result = agent.run("How many customers made purchases above $1000 last month?")
```

**What happens:**
```
Agent reasoning:
1. I need to query the database
2. Let me use SQL_Query tool
3. SQL: SELECT COUNT(*) FROM orders WHERE amount > 1000 AND date > '2024-01-01'
4. Result: 342
5. Format answer: "342 customers made purchases above $1000 last month"
```

---

### ReAct Pattern (Reasoning + Acting)

```
Question: "What's the average salary in our company?"

Thought: I need to query the employee database
Action: SQL_Query
Action Input: SELECT AVG(salary) FROM employees
Observation: 75000

Thought: Now I can answer
Final Answer: The average salary is $75,000
```

---

## Function Calling

### What is Function Calling?

**Definition:** LLM decides when and how to call external functions based on user input.

### How It Works

```
User: "Send email to john@example.com about today's meeting"

LLM analyzes request:
- Action: Send email
- Parameters:
  * to: "john@example.com"
  * subject: "Today's meeting"
  * body: [generate appropriate text]

Calls: send_email(to="john@example.com", subject="Today's meeting", body="...")
```

---

### Function Calling Example (OpenAI)

```python
from openai import OpenAI
import json

client = OpenAI()

# Define functions
functions = [
    {
        "name": "get_customer_data",
        "description": "Retrieve customer information from database",
        "parameters": {
            "type": "object",
            "properties": {
                "customer_id": {
                    "type": "string",
                    "description": "The customer ID"
                }
            },
            "required": ["customer_id"]
        }
    },
    {
        "name": "execute_sql",
        "description": "Execute SQL query on database",
        "parameters": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "The SQL query to execute"
                }
            },
            "required": ["query"]
        }
    }
]

# User request
messages = [
    {"role": "user", "content": "Get information for customer C12345"}
]

# Call LLM
response = client.chat.completions.create(
    model="gpt-4",
    messages=messages,
    functions=functions,
    function_call="auto"  # Let model decide
)

# Check if function called
message = response.choices[0].message

if message.function_call:
    function_name = message.function_call.name
    function_args = json.loads(message.function_call.arguments)
    
    print(f"Calling: {function_name}")
    print(f"Arguments: {function_args}")
    
    # Execute function
    if function_name == "get_customer_data":
        result = get_customer_data(**function_args)
    
    # Send result back to LLM
    messages.append(message)
    messages.append({
        "role": "function",
        "name": function_name,
        "content": json.dumps(result)
    })
    
    # Get final response
    final_response = client.chat.completions.create(
        model="gpt-4",
        messages=messages
    )
    
    print(final_response.choices[0].message.content)
```

---

### Data Engineering Use Case

```python
# Define data engineering tools
functions = [
    {
        "name": "check_pipeline_status",
        "description": "Check status of data pipeline",
        "parameters": {
            "type": "object",
            "properties": {
                "pipeline_name": {"type": "string"}
            }
        }
    },
    {
        "name": "get_table_schema",
        "description": "Get schema of database table",
        "parameters": {
            "type": "object",
            "properties": {
                "table_name": {"type": "string"}
            }
        }
    },
    {
        "name": "run_data_quality_check",
        "description": "Run data quality checks on table",
        "parameters": {
            "type": "object",
            "properties": {
                "table_name": {"type": "string"},
                "checks": {
                    "type": "array",
                    "items": {"type": "string"}
                }
            }
        }
    }
]

# User: "Check if the customer table has any NULL emails"
# LLM will call: run_data_quality_check(table_name="customers", checks=["null_emails"])
```

---

## Multimodal Models

### What are Multimodal Models?

**Definition:** Models that can process multiple types of input (text, images, audio).

### Examples

**GPT-4 Vision:**
```python
response = client.chat.completions.create(
    model="gpt-4-vision-preview",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "What's in this image?"},
                {
                    "type": "image_url",
                    "image_url": "https://example.com/image.jpg"
                }
            ]
        }
    ]
)
```

**Use Cases:**
- ER diagram to SQL schema
- Dashboard screenshots to requirements
- Chart/graph interpretation
- Document OCR and understanding

---

## Production Deployment

### Architecture Patterns

#### 1. Direct API Call

```
User → Your App → OpenAI API → Response
```

**Pros:** Simple
**Cons:** Every request costs money, slow

---

#### 2. Caching Layer

```
User → Your App → Cache Check → If miss: OpenAI API
                       ↓
                   If hit: Return cached
```

```python
import hashlib
import redis

cache = redis.Redis()

def get_llm_response_cached(prompt):
    # Create cache key
    key = hashlib.md5(prompt.encode()).hexdigest()
    
    # Check cache
    cached = cache.get(key)
    if cached:
        return json.loads(cached)
    
    # Call LLM
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}]
    )
    
    result = response.choices[0].message.content
    
    # Cache for 1 hour
    cache.setex(key, 3600, json.dumps(result))
    
    return result
```

---

#### 3. Queue-Based (Async)

```
User → Add to Queue → Background Worker → OpenAI API
   ↓                        ↓
Receive job ID        Process and store result
   ↓
Poll for result
```

**Benefits:** Handle spikes, batch processing

---

### Monitoring

**Key Metrics:**

```python
import time
from prometheus_client import Counter, Histogram

# Request counter
llm_requests = Counter('llm_requests_total', 'Total LLM requests')

# Latency
llm_latency = Histogram('llm_request_duration_seconds', 'LLM request latency')

# Cost
llm_cost = Counter('llm_cost_dollars', 'Total LLM cost')

def llm_call_with_monitoring(prompt):
    llm_requests.inc()
    
    start = time.time()
    response = client.chat.completions.create(model="gpt-4", messages=[...])
    duration = time.time() - start
    
    llm_latency.observe(duration)
    
    # Calculate cost
    tokens = response.usage.total_tokens
    cost = tokens * 0.00003  # $0.03 per 1K tokens
    llm_cost.inc(cost)
    
    return response
```

---

### Error Handling

```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10)
)
def llm_call_with_retry(prompt):
    try:
        response = client.chat.completions.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}],
            timeout=30
        )
        return response.choices[0].message.content
    
    except openai.RateLimitError:
        print("Rate limit hit, retrying...")
        raise  # Retry
    
    except openai.APIError as e:
        print(f"API error: {e}")
        raise  # Retry
    
    except Exception as e:
        print(f"Unexpected error: {e}")
        return "Sorry, I encountered an error. Please try again."
```

---

## Cost Optimization

### Strategies

**1. Use Appropriate Model**
```python
# Expensive
response = client.chat.completions.create(model="gpt-4", ...)  # $0.03/1K tokens

# Cheaper (if quality acceptable)
response = client.chat.completions.create(model="gpt-3.5-turbo", ...)  # $0.0005/1K tokens
```

**Savings: 60x cheaper!**

---

**2. Compress Prompts**

```python
# Bad (verbose)
prompt = """
Please analyze the following data and provide insights about the trends.
The data shows sales figures from last quarter.
Here is the data:
[data]
Please format your response as a bulleted list with each insight on a new line.
"""

# Good (concise)
prompt = """
Analyze trends in Q1 sales data. Return as bulleted insights.
Data: [data]
"""
```

**Savings: 50% fewer tokens**

---

**3. Cache Aggressively**

```python
# Cache embeddings
embedding_cache = {}

# Cache common queries
response_cache = {}

# Cache for 24 hours for static content
```

---

**4. Batch Requests**

```python
# Bad: 100 separate API calls
for text in texts:
    embedding = get_embedding(text)

# Good: 1 API call
embeddings = get_embeddings_batch(texts)  # Process 100 at once
```

---

**5. Use Streaming for UX**

```python
# Non-streaming: Wait 30s for full response
response = client.chat.completions.create(...)

# Streaming: Show partial response immediately
stream = client.chat.completions.create(stream=True, ...)
for chunk in stream:
    print(chunk.choices[0].delta.content, end="")
```

**Benefit:** Better user experience, same cost

---

## Summary

### Key Takeaways:

✅ **Fine-Tuning:** Specialize model for specific tasks/domains
✅ **Agents:** LLMs that use tools autonomously (ReAct pattern)
✅ **Function Calling:** LLM decides when to call functions
✅ **Multimodal:** Process text + images together
✅ **Production:** Caching, monitoring, error handling, cost optimization

### Data Engineering Applications:

- **SQL Assistant Agent:** Uses tools to query DB, explain results
- **Data Quality Agent:** Automatically checks data and suggests fixes
- **Documentation Generator:** Analyzes code/schemas, generates docs
- **Pipeline Monitor:** Watches Airflow/dbt, alerts on issues

---

**You've completed all 6 chapters! Build projects to apply your learning! 🚀**
