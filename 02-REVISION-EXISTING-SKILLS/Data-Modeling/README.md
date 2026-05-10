# Data Modeling - Dimensional Modeling & Data Warehouse Design

**Master data modeling techniques for analytics and data engineering interviews**

---

## 📚 What's in This Folder

### **1. 100-QUESTIONS.md**
Comprehensive Q&A covering:
- Dimensional modeling (star schema, snowflake schema)
- Fact and dimension tables
- Slowly Changing Dimensions (SCD Types 0-7)
- Data Vault modeling
- Normalization (1NF, 2NF, 3NF, BCNF)
- Denormalization strategies
- Surrogate vs natural keys
- Data warehouse architecture
- Real-world modeling scenarios

### **2. CHEATSHEET.md**
Quick reference with:
- One-line definitions
- Schema design patterns
- SCD implementation approaches
- Normalization rules
- Best practices checklist
- Interview talking points

---

## 🎯 What You'll Learn

### **Dimensional Modeling**
- **Fact Tables:** Measurements, metrics, transactional data
- **Dimension Tables:** Context, descriptive attributes
- **Star Schema:** Fact table surrounded by dimension tables
- **Snowflake Schema:** Normalized dimension tables
- **Grain:** Level of detail in fact table
- **Conformed Dimensions:** Shared across fact tables
- **Degenerate Dimensions:** Attributes in fact table (order number)

### **Slowly Changing Dimensions**
- **Type 0:** Retain original (never changes)
- **Type 1:** Overwrite old value (no history)
- **Type 2:** Add new row (full history with effective dates)
- **Type 3:** Add new column (limited history)
- **Type 4:** Separate history table
- **Type 6:** Hybrid (1+2+3)

### **Advanced Concepts**
- **Data Vault 2.0:** Hubs, Links, Satellites
- **Anchor Modeling:** Temporal data modeling
- **Kimball vs Inmon:** Bottom-up vs top-down
- **Bridge Tables:** Many-to-many relationships
- **Junk Dimensions:** Low-cardinality flags
- **Role-Playing Dimensions:** Same dimension, multiple roles

---

## 💼 Why Data Modeling Matters

### **Foundation of Analytics**
- Well-designed models → Fast queries
- Poor models → Performance nightmares
- 80% of data warehouse success depends on modeling

### **Career Impact**
- Senior+ roles require modeling expertise
- Architects must design schemas
- Critical for dbt, Looker, Tableau projects
- Bridges engineering and business

### **Interview Importance**
- **70% of DE interviews** include modeling questions
- "Design a schema for X" is common
- Tests both technical and business understanding
- Differentiates senior from junior candidates

---

## 🚀 Quick Start Guide

### **1. Read Cheatsheet (20 minutes)**
Review `CHEATSHEET.md` for core concepts and terminology.

### **2. Study 100 Questions (5-6 hours)**
- **Q1-Q20:** Fundamentals (fact vs dimension, star vs snowflake)
- **Q21-Q40:** SCDs (Types 0-7 with implementations)
- **Q41-Q60:** Advanced (Data Vault, normalization, keys)
- **Q61-Q80:** Patterns (bridges, junk dimensions, role-playing)
- **Q81-Q100:** Real-world scenarios (e-commerce, healthcare, SaaS)

### **3. Practice Design**
Design schemas for common domains:
- E-commerce (orders, customers, products)
- Healthcare (patients, visits, procedures)
- SaaS (subscriptions, usage, billing)
- Financial services (accounts, transactions, customers)

---

## 📖 Topic Coverage

### **Star Schema Example**
```
        ┌─────────────┐
        │ dim_customer│
        └──────┬──────┘
               │
     ┌─────────┴─────────┐
     │                   │
┌────▼─────┐       ┌────▼──────┐
│dim_product│       │ fct_sales │
└──────────┘       └────┬──────┘
                        │
                   ┌────▼─────┐
                   │ dim_date │
                   └──────────┘
```

**Fact Table: fct_sales**
```sql
CREATE TABLE fct_sales (
    sales_key BIGINT PRIMARY KEY,        -- Surrogate key
    customer_key BIGINT,                 -- FK to dim_customer
    product_key BIGINT,                  -- FK to dim_product
    date_key INT,                        -- FK to dim_date
    order_id VARCHAR(50),                -- Degenerate dimension
    quantity INT,                        -- Metric
    unit_price DECIMAL(10,2),            -- Metric
    discount_amount DECIMAL(10,2),       -- Metric
    total_amount DECIMAL(10,2)           -- Metric
);
```

**Dimension Table: dim_customer (SCD Type 2)**
```sql
CREATE TABLE dim_customer (
    customer_key BIGINT PRIMARY KEY,     -- Surrogate key
    customer_id VARCHAR(50),             -- Natural key
    customer_name VARCHAR(200),
    email VARCHAR(200),
    segment VARCHAR(50),
    effective_date DATE,                 -- SCD Type 2
    expiration_date DATE,                -- SCD Type 2
    is_current BOOLEAN                   -- SCD Type 2
);
```

### **SCD Type 2 Implementation**

**Initial Load:**
```sql
INSERT INTO dim_customer VALUES
(1, 'C001', 'Alice Smith', 'alice@email.com', 'Gold', '2024-01-01', '9999-12-31', TRUE);
```

**Customer moves (address change):**
```sql
-- Step 1: Expire old record
UPDATE dim_customer
SET expiration_date = '2024-06-30', is_current = FALSE
WHERE customer_id = 'C001' AND is_current = TRUE;

-- Step 2: Insert new record
INSERT INTO dim_customer VALUES
(2, 'C001', 'Alice Smith', 'alice@newemail.com', 'Gold', '2024-07-01', '9999-12-31', TRUE);
```

**Query for current state:**
```sql
SELECT * FROM dim_customer WHERE is_current = TRUE;
```

**Query for historical state:**
```sql
SELECT * FROM dim_customer
WHERE customer_id = 'C001'
  AND '2024-05-15' BETWEEN effective_date AND expiration_date;
```

### **Normalization Levels**

**1NF (First Normal Form):**
- Atomic values (no arrays/lists)
- Each column has single value
- Unique rows

**2NF (Second Normal Form):**
- 1NF + No partial dependencies
- Non-key columns depend on entire primary key

**3NF (Third Normal Form):**
- 2NF + No transitive dependencies
- Non-key columns depend only on primary key

**Example:**
```sql
-- Unnormalized
CREATE TABLE orders (
    order_id INT,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    product_name VARCHAR(100),
    product_price DECIMAL(10,2)
);

-- 3NF (Normalized)
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    product_id INT REFERENCES products(product_id),
    order_date DATE
);
```

### **Data Vault 2.0**

**Components:**
- **Hubs:** Business keys (Customer, Product, Order)
- **Links:** Relationships between hubs (Customer-Order)
- **Satellites:** Attributes and history (Customer details)

**Example:**
```sql
-- Hub: Customer
CREATE TABLE hub_customer (
    customer_key BIGINT PRIMARY KEY,
    customer_id VARCHAR(50),             -- Business key
    load_date TIMESTAMP,
    record_source VARCHAR(100)
);

-- Satellite: Customer Details
CREATE TABLE sat_customer (
    customer_key BIGINT,
    load_date TIMESTAMP,
    hash_diff VARCHAR(100),              -- For change detection
    customer_name VARCHAR(200),
    email VARCHAR(200),
    segment VARCHAR(50),
    PRIMARY KEY (customer_key, load_date)
);

-- Link: Customer-Order
CREATE TABLE link_customer_order (
    link_key BIGINT PRIMARY KEY,
    customer_key BIGINT,
    order_key BIGINT,
    load_date TIMESTAMP,
    record_source VARCHAR(100)
);
```

---

## 🎓 Interview Preparation

### **Week 1: Dimensional Modeling**
- Study Q1-Q30
- Understand star vs snowflake
- Practice designing fact/dimension tables
- Learn grain concepts

### **Week 2: SCDs**
- Study Q31-Q50
- Implement SCD Type 2 by hand
- Understand when to use each type
- Practice in SQL

### **Week 3: Advanced**
- Study Q51-Q80
- Data Vault basics
- Normalization rules
- Bridge tables and junk dimensions

### **Week 4: Scenarios**
- Study Q81-Q100
- Design schemas for different industries
- Practice whiteboarding
- Mock interviews

---

## 💡 Common Interview Questions

### **Conceptual**

**Q: "Fact vs Dimension table?"**
- **Fact:** Measurements (sales amount, quantity, duration)
  - Numeric, additive
  - Large volume
  - Foreign keys to dimensions
- **Dimension:** Context (customer name, product category, date)
  - Descriptive attributes
  - Smaller volume
  - Used for filtering, grouping

**Q: "Star vs Snowflake schema?"**
- **Star:** Denormalized dimensions, simpler queries, faster
- **Snowflake:** Normalized dimensions, less redundancy, more joins
- Star preferred for OLAP (analytical queries)
- Snowflake for normalized storage

**Q: "Explain SCD Type 2"**
- Tracks full history
- New row for each change
- Uses effective/expiration dates
- is_current flag for latest version
- Enables historical reporting

**Q: "Surrogate vs Natural key?"**
- **Surrogate:** System-generated (auto-increment, sequence)
  - Never changes, consistent size
  - Preferred for dimensions
- **Natural:** Business key (SSN, email, customer_id)
  - May change, variable size
  - Keep as attribute, not primary key

### **Design Questions**

**Q: "Design schema for e-commerce order system"**
```
Fact Table: fct_orders
- order_key (PK)
- customer_key (FK)
- product_key (FK)
- date_key (FK)
- quantity
- unit_price
- discount
- total_amount

Dimensions:
- dim_customer (SCD Type 2: track address changes)
- dim_product (SCD Type 1: price updates overwrite)
- dim_date (static)
- dim_promotion (if applicable)
```

**Q: "Model SaaS subscription business"**
```
Fact Tables:
- fct_subscriptions (subscription events)
- fct_usage (daily/monthly usage metrics)
- fct_billing (invoices, payments)

Dimensions:
- dim_customer (SCD Type 2)
- dim_plan (SCD Type 2: plan features change)
- dim_date (date dimension)

Metrics:
- MRR (Monthly Recurring Revenue)
- Churn rate
- Customer lifetime value
```

**Q: "Handle many-to-many relationship?"**
Use bridge table:
```sql
-- Example: Product can have multiple categories
CREATE TABLE dim_product (
    product_key BIGINT PRIMARY KEY,
    product_id VARCHAR(50),
    product_name VARCHAR(200)
);

CREATE TABLE dim_category (
    category_key BIGINT PRIMARY KEY,
    category_name VARCHAR(100)
);

CREATE TABLE bridge_product_category (
    product_key BIGINT,
    category_key BIGINT,
    PRIMARY KEY (product_key, category_key)
);
```

---

## 🔗 Resources

### **Books**
- **"The Data Warehouse Toolkit"** by Ralph Kimball (Bible of dimensional modeling)
- **"Building a Scalable Data Warehouse"** by Dan Linstedt (Data Vault)
- **"Star Schema: The Complete Reference"** by Christopher Adamson

### **Online**
- [Kimball Group](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/)
- [Data Vault Alliance](https://datavaultalliance.com/)
- [dbt Docs - Dimensional Modeling](https://docs.getdbt.com/guides/best-practices/how-we-structure/3-intermediate)

### **Practice**
- Design schemas for real business scenarios
- Reverse engineer existing data warehouses
- Practice drawing ERDs by hand

---

## 🎯 Key Takeaways

### **Must Know**
✅ Fact vs dimension tables
✅ Star schema design
✅ SCD Type 2 implementation
✅ Grain definition
✅ Surrogate keys

### **Should Know**
✅ Snowflake schema
✅ All SCD types (0-7)
✅ Data Vault basics
✅ Normalization (1NF, 2NF, 3NF)
✅ Bridge tables

### **Nice to Know**
✅ Junk dimensions
✅ Role-playing dimensions
✅ Conformed dimensions
✅ Anchor modeling
✅ Kimball vs Inmon approaches

### **Red Flags**
❌ Using natural keys as primary keys
❌ Not tracking history (no SCDs)
❌ Over-normalizing OLAP schemas
❌ Not defining grain clearly
❌ Missing date dimensions

---

## 📝 Design Checklist

When designing a data warehouse:

**1. Understand Requirements**
- [ ] Identify business processes
- [ ] Define metrics/KPIs
- [ ] Understand data sources
- [ ] Clarify reporting needs

**2. Identify Facts**
- [ ] What are we measuring?
- [ ] Define grain (one row = ?)
- [ ] List all metrics
- [ ] Identify degenerate dimensions

**3. Identify Dimensions**
- [ ] What provides context?
- [ ] Which need history (SCD Type)?
- [ ] Conformed dimensions?
- [ ] Role-playing dimensions?

**4. Design Schema**
- [ ] Draw ERD
- [ ] Define primary/foreign keys
- [ ] Choose surrogate keys
- [ ] Plan for scalability

**5. Validate**
- [ ] Can answer all business questions?
- [ ] Grain consistent?
- [ ] Historical tracking adequate?
- [ ] Performance acceptable?

---

## 💼 Interview Practice Scenarios

### **Scenario 1: Retail Sales**
Design a data warehouse for retail chain:
- Multiple stores, regions
- Products with categories
- Promotions and discounts
- Customer loyalty program
- Need to track: sales, inventory, returns

### **Scenario 2: Healthcare**
Design for hospital patient management:
- Patients with demographics
- Visits and admissions
- Procedures and diagnoses
- Providers (doctors, nurses)
- Need to track: costs, outcomes, wait times

### **Scenario 3: Streaming Service**
Design for video streaming platform:
- Users with subscriptions
- Content (movies, shows, episodes)
- Viewing sessions
- Devices
- Need to track: engagement, churn, recommendations

**Practice answering:**
1. What are the fact tables?
2. What are the dimensions?
3. Which dimensions need SCD Type 2?
4. What is the grain of each fact table?
5. Draw the star schema

---

**Master data modeling and design robust data warehouses! 🏗️**

*For detailed answers and examples, see 100-QUESTIONS.md*
*For quick review, see CHEATSHEET.md*
