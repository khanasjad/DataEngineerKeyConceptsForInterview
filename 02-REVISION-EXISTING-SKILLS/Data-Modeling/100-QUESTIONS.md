# Data Modeling - 100 Interview Questions & Answers

**Goal:** Master data modeling concepts for data warehouse and analytics

---

## THEORY QUESTIONS (50 Questions)

### Dimensional Modeling (20 Questions)

**Q1: What is dimensional modeling? Why use it?**

**Answer:**
Dimensional modeling is a design technique for data warehouses optimized for query performance and business user understanding.

**Key Concepts:**
- **Facts:** Measurable events (sales, clicks, transactions)
- **Dimensions:** Context for facts (who, what, when, where, why)
- **Star Schema:** Fact table surrounded by dimension tables
- **Designed for:** Analytics, BI tools, fast aggregations

**Why use it:**
✅ **Query Performance:** Denormalized, optimized for reads
✅ **Business-friendly:** Intuitive model (business process oriented)
✅ **BI tool compatible:** Works well with Tableau, Power BI
✅ **Aggregation-friendly:** Pre-joined dimensions speed up queries

**vs Traditional 3NF:**
| Aspect | 3NF (OLTP) | Dimensional (OLAP) |
|--------|------------|-------------------|
| Purpose | Transaction processing | Analytics/reporting |
| Normalization | Highly normalized | Denormalized |
| Joins | Many joins | Few joins |
| Redundancy | Minimal | Acceptable |
| Query speed | Slower for analytics | Fast |

**Example:**
```
3NF (OLTP):
orders → customers → addresses → cities → states

Star Schema (OLAP):
fact_orders ← dim_customers (all customer info flattened)
```

---

**Q2: Explain Star Schema vs Snowflake Schema**

**Answer:**

**Star Schema:**
- Fact table at center
- Dimension tables directly connected to fact
- Dimensions are denormalized (flattened)

**Structure:**
```
       dim_product
            |
dim_customer - fact_sales - dim_time
            |
       dim_store
```

**Pros:**
- ✅ Simpler queries (fewer joins)
- ✅ Better performance (fewer joins)
- ✅ Easier for BI tools

**Cons:**
- ❌ More storage (denormalized)
- ❌ Data redundancy

**Snowflake Schema:**
- Dimension tables are normalized (split into sub-dimensions)

**Structure:**
```
dim_product → dim_category → dim_subcategory
       ↓
  fact_sales
```

**Pros:**
- ✅ Less storage (normalized)
- ✅ Less redundancy

**Cons:**
- ❌ More complex queries (more joins)
- ❌ Slower performance

**Recommendation:**
Use **Star Schema** for data warehouses (performance > storage in analytics)

---

**Q3: What is a Fact Table? Types of facts?**

**Answer:**

**Fact Table:**
- Contains measurable business events
- Has foreign keys to dimensions
- Contains metrics/measures
- Granularity defines detail level

**Structure:**
```sql
CREATE TABLE fact_sales (
    -- Dimension keys (FK)
    date_key INT,
    product_key INT,
    customer_key INT,
    store_key INT,

    -- Measures (facts)
    quantity_sold INT,
    unit_price DECIMAL(10,2),
    total_amount DECIMAL(10,2),
    discount_amount DECIMAL(10,2),

    -- Degenerate dimension (if needed)
    order_number VARCHAR(20)
);
```

**Types of Facts:**

**1. Additive Facts** (can sum across all dimensions)
```sql
-- Can sum across time, product, customer
SUM(quantity_sold)
SUM(total_amount)
```

**2. Semi-Additive Facts** (can sum across some dimensions)
```sql
-- Account balance: Can sum across customers, NOT across time
SUM(account_balance)  -- Across customers: OK
SUM(account_balance)  -- Across dates: WRONG!

-- Use AVG or snapshot for time
AVG(account_balance) OVER time
```

**3. Non-Additive Facts** (can't sum at all)
```sql
-- Ratios, percentages
profit_margin = profit / revenue  -- Can't sum margins!

-- Convert to additive
-- Instead, store: profit and revenue (both additive)
-- Calculate ratio at query time
```

**4. Factless Fact Tables** (no measures, just events)
```sql
-- Example: Student attendance
CREATE TABLE fact_attendance (
    student_key INT,
    course_key INT,
    date_key INT,
    -- No measures! Just recording the event
    PRIMARY KEY (student_key, course_key, date_key)
);

-- Use: COUNT(*) to count attendance events
```

---

**Q4: What is a Dimension Table? Types?**

**Answer:**

**Dimension Table:**
- Descriptive attributes providing context
- Typically smaller than facts
- Denormalized (wide tables with many columns)
- Contains business-friendly names

**Structure:**
```sql
CREATE TABLE dim_product (
    product_key INT PRIMARY KEY,  -- Surrogate key
    product_id VARCHAR(50),        -- Natural key
    product_name VARCHAR(200),
    category VARCHAR(100),
    subcategory VARCHAR(100),
    brand VARCHAR(100),
    color VARCHAR(50),
    size VARCHAR(20),
    weight DECIMAL(10,2),
    -- SCD columns (if Type 2)
    effective_date DATE,
    expiration_date DATE,
    is_current BOOLEAN
);
```

**Types of Dimensions:**

**1. Conformed Dimensions**
- Shared across multiple fact tables
- Same dimension used consistently

```sql
-- dim_customer used by multiple facts
fact_sales → dim_customer
fact_returns → dim_customer  -- Same dimension!
fact_support_tickets → dim_customer
```

**2. Role-Playing Dimensions**
- Same dimension used multiple times with different context

```sql
-- dim_date used three times
fact_orders (
    order_date_key → dim_date,
    ship_date_key → dim_date,     -- Same dim, different role
    delivery_date_key → dim_date
)
```

**3. Junk Dimensions**
- Low-cardinality flags grouped together
- Avoids too many FKs in fact table

```sql
-- Instead of many boolean flags in fact table
CREATE TABLE dim_order_flags (
    flags_key INT PRIMARY KEY,
    is_express BOOLEAN,
    is_gift_wrap BOOLEAN,
    is_international BOOLEAN,
    requires_signature BOOLEAN
    -- All combinations: 2^4 = 16 rows
);
```

**4. Degenerate Dimensions**
- Dimension attribute stored in fact table (no separate dim table)
- Usually transaction identifiers

```sql
fact_sales (
    ...facts...,
    order_number VARCHAR(20),  -- Degenerate dimension
    invoice_number VARCHAR(20)
);
```

**5. Slowly Changing Dimensions (SCD)**
- See Q5-Q7 for details

---

**Q5-Q7: Slowly Changing Dimensions (SCD)**

**Q5: What are SCDs? Why needed?**

**Answer:**
Dimension attributes that change over time.

**Example Problem:**
```
Customer "Alice" moves from NYC to LA
- Historical reports should show NYC (for old orders)
- New reports should show LA (for new orders)
```

**Three strategies:** Type 1, Type 2, Type 3

---

**Q6: Explain SCD Type 1, Type 2, Type 3**

**Answer:**

**Type 1: Overwrite (No History)**
```sql
-- Just UPDATE the current row
UPDATE dim_customer
SET city = 'Los Angeles', state = 'CA'
WHERE customer_key = 123;

-- Result: History lost!
```

**When to use:**
- Corrections (fix typos)
- Changes you don't care about historically
- Example: Email address, phone number (maybe)

---

**Type 2: New Row (Full History)**
```sql
-- Before change:
customer_key | customer_id | name  | city     | start_date | end_date   | is_current
1001         | CUST_123    | Alice | NYC      | 2023-01-01 | 2024-05-03 | FALSE

-- After change (new row):
customer_key | customer_id | name  | city         | start_date | end_date   | is_current
1001         | CUST_123    | Alice | NYC          | 2023-01-01 | 2024-05-03 | FALSE
1002         | CUST_123    | Alice | Los Angeles  | 2024-05-04 | 9999-12-31 | TRUE
```

**Implementation:**
```sql
-- Step 1: Expire old record
UPDATE dim_customer
SET end_date = CURRENT_DATE, is_current = FALSE
WHERE customer_id = 'CUST_123' AND is_current = TRUE;

-- Step 2: Insert new record
INSERT INTO dim_customer (customer_id, name, city, state, start_date, is_current)
VALUES ('CUST_123', 'Alice', 'Los Angeles', 'CA', CURRENT_DATE + 1, TRUE);
```

**When to use:**
- Track history for analysis
- Example: Customer address, product price, employee department

**Querying Type 2:**
```sql
-- Current records only
SELECT * FROM dim_customer WHERE is_current = TRUE;

-- Point-in-time query (as of specific date)
SELECT * FROM dim_customer
WHERE customer_id = 'CUST_123'
  AND '2024-01-15' BETWEEN start_date AND end_date;

-- Join with fact (automatic historical accuracy)
SELECT f.*, d.city
FROM fact_sales f
JOIN dim_customer d ON f.customer_key = d.customer_key
-- Customer_key in fact points to correct historical version!
```

---

**Type 3: Add Column (Limited History)**
```sql
-- Add column for previous value
customer_key | name  | current_city | previous_city | city_change_date
123          | Alice | Los Angeles  | NYC           | 2024-05-03

-- Update
UPDATE dim_customer
SET previous_city = current_city,
    current_city = 'Los Angeles',
    city_change_date = CURRENT_DATE
WHERE customer_key = 123;
```

**When to use:**
- Need only last change
- Limited use cases

**Comparison:**
| Type | History | Storage | Query Complexity | Use Case |
|------|---------|---------|------------------|----------|
| Type 1 | None | Low | Simple | Corrections |
| Type 2 | Full | High | Moderate | Historical analysis |
| Type 3 | Last change only | Low | Simple | Rare |

---

**Q8: What is a surrogate key vs natural key?**

**Answer:**

**Natural Key:**
- Business identifier from source system
- Examples: customer_id, product_code, SSN

**Surrogate Key:**
- System-generated identifier
- Usually auto-increment integer
- No business meaning

**Why use surrogate keys in dimensional modeling?**

**Problem with Natural Keys:**
```sql
-- Natural key: Product Code = "ABC123"
-- What if code changes?
-- What if same code reused?
-- What if merging data from multiple sources?
```

**Solution: Surrogate Keys**
```sql
CREATE TABLE dim_product (
    product_key INT PRIMARY KEY,      -- Surrogate (DW generated)
    product_code VARCHAR(50),          -- Natural (from source)
    product_name VARCHAR(200),
    ...
);

-- SCD Type 2 with surrogate:
product_key | product_code | name      | price | is_current
1001        | ABC123       | Widget    | 10.00 | FALSE
1002        | ABC123       | Widget    | 12.00 | TRUE  -- Same natural key, different surrogate!
```

**Benefits:**
✅ **SCD friendly:** New row = new surrogate key
✅ **Performance:** Integer joins faster than string
✅ **Stability:** Doesn't change even if business key changes
✅ **Multi-source:** Handle same natural key from different systems

**Best Practice:**
- Always use surrogate keys in dimension tables
- Keep natural key for reference and joins with source

---

**Q9: What is grain in fact table?**

**Answer:**

**Grain** is the most fundamental design decision for a fact table - it defines what exactly one row represents.

**Definition:**
Grain = the level of detail or specificity of measurements in a fact table

**Examples:**
```
Transaction grain: One row per product per order
Daily grain: One row per product per day
Monthly grain: One row per product per month
```

**Why Critical:**
- Determines query capabilities (can't report at finer grain than stored)
- Affects storage size
- All measures must be compatible with the grain
- Can't mix grains in same fact table!

**Example - E-commerce Orders:**

**Atomic Grain (Most Detailed):**
```sql
-- Grain: One row per product per order line
CREATE TABLE fact_order_lines (
    order_id VARCHAR(50),
    order_line_number INT,
    order_date_key INT,
    product_key INT,
    customer_key INT,

    -- Measures at LINE level
    quantity INT,              -- ✅ Additive
    unit_price DECIMAL(10,2),  -- ❌ Can't sum (use weighted avg)
    line_total DECIMAL(10,2),  -- ✅ Additive

    PRIMARY KEY (order_id, order_line_number)
);

-- Row count: 10M orders × 3 products avg = 30M rows
```

**Daily Aggregate Grain:**
```sql
-- Grain: One row per product per customer per day
CREATE TABLE fact_daily_sales (
    date_key INT,
    product_key INT,
    customer_key INT,

    -- Measures at DAILY level
    order_count INT,              -- ✅ Count of orders
    total_quantity INT,           -- ✅ Sum of quantities
    total_revenue DECIMAL(10,2),  -- ✅ Sum of line totals
    avg_unit_price DECIMAL(10,2), -- ❌ Pre-calculated avg

    PRIMARY KEY (date_key, product_key, customer_key)
);

-- Row count: Much smaller (aggregated)
```

**Optum Healthcare Example:**

**Claims Processing - Atomic Grain:**
```sql
-- Grain: One row per claim line item
CREATE TABLE fact_claims (
    claim_id VARCHAR(50),
    claim_line_number INT,
    service_date_key INT,
    patient_key INT,
    provider_key INT,
    diagnosis_key INT,
    procedure_key INT,

    -- Measures at claim line level
    billed_amount DECIMAL(10,2),
    allowed_amount DECIMAL(10,2),
    paid_amount DECIMAL(10,2),
    patient_responsibility DECIMAL(10,2),
    units_of_service INT,

    PRIMARY KEY (claim_id, claim_line_number)
);

-- Interview talking point:
-- "At Optum, we processed 500GB/day of claims at atomic grain
-- (one row per claim line). This grain allowed us to drill down
-- to individual procedures while also supporting monthly rollups
-- for financial reporting."
```

**Rules for Grain:**
1. **State it explicitly** - "One row per [what] per [time period]"
2. **All measures must fit the grain** - Can't mix order-level and line-level measures
3. **Consider query patterns** - Finer grain = more flexibility but more storage
4. **Document it** - Critical for ETL developers and analysts

**Common Mistake:**
```sql
-- ❌ WRONG: Mixed grain
CREATE TABLE fact_sales (
    order_id VARCHAR(50),
    product_id VARCHAR(50),
    quantity INT,           -- Line level measure
    order_total DECIMAL,    -- Order level measure ❌ WRONG!
    -- Problem: order_total duplicated across all lines!
);
```

**Interview Tip:**
Always ask: "What does one row represent?" before designing any fact table.

---

**Q10: What are conformed dimensions? Why important?**

**Answer:**

**Conformed Dimensions** are dimension tables shared across multiple fact tables with identical structure, meaning, and content.

**Key Principle:**
Same dimension = same keys, same attributes, same values across the entire enterprise

**Example - Non-Conformed (Bad):**
```sql
-- Sales team's customer dimension
dim_customer_sales (
    customer_key, customer_id, name, sales_region
)

-- Support team's customer dimension (different!)
dim_customer_support (
    customer_key, customer_id, full_name, support_tier
)

-- ❌ Problem: Can't join sales and support data!
-- ❌ Problem: Different customer_keys for same customer!
```

**Example - Conformed (Good):**
```sql
-- Single shared customer dimension
CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY,
    customer_id VARCHAR(50),
    name VARCHAR(200),
    email VARCHAR(200),
    sales_region VARCHAR(50),
    support_tier VARCHAR(20),
    -- All attributes needed by all teams
    effective_date DATE,
    is_current BOOLEAN
);

-- Used by multiple fact tables
fact_sales → dim_customer
fact_support_tickets → dim_customer
fact_renewals → dim_customer
fact_surveys → dim_customer
```

**Benefits:**

**1. Consistent Reporting**
```sql
-- Can compare metrics across business processes
SELECT
    c.customer_segment,
    SUM(s.revenue) as sales_revenue,
    COUNT(DISTINCT t.ticket_id) as support_tickets,
    AVG(surv.satisfaction_score) as csat
FROM dim_customer c
LEFT JOIN fact_sales s ON c.customer_key = s.customer_key
LEFT JOIN fact_support_tickets t ON c.customer_key = t.customer_key
LEFT JOIN fact_surveys surv ON c.customer_key = surv.customer_key
GROUP BY c.customer_segment;

-- ✅ Works because all facts use same conformed customer dimension
```

**2. Drill-Across Queries**
- Query multiple fact tables using same dimension filters
- Compare metrics side-by-side

**3. Single Source of Truth**
- One master dimension managed centrally
- Updates propagate to all fact tables

**Optum Healthcare Example:**

```sql
-- Conformed dim_patient used across all healthcare processes
CREATE TABLE dim_patient (
    patient_key INT PRIMARY KEY,
    patient_id VARCHAR(50),     -- Medical Record Number
    date_of_birth DATE,
    gender VARCHAR(10),
    zip_code VARCHAR(10),
    insurance_plan VARCHAR(100),
    risk_score DECIMAL(5,2),
    -- SCD Type 2
    effective_date DATE,
    is_current BOOLEAN
);

-- Used by multiple fact tables:
fact_claims → dim_patient        -- Claims processing
fact_visits → dim_patient         -- Provider visits
fact_prescriptions → dim_patient  -- Pharmacy
fact_lab_results → dim_patient    -- Lab tests
fact_quality_measures → dim_patient -- HEDIS/Stars

-- Query: Patient cost and quality across all touchpoints
SELECT
    p.patient_id,
    p.risk_score,
    SUM(c.paid_amount) as total_cost,
    COUNT(DISTINCT v.visit_id) as visit_count,
    COUNT(DISTINCT rx.prescription_id) as prescription_count,
    AVG(qm.measure_score) as avg_quality_score
FROM dim_patient p
LEFT JOIN fact_claims c ON p.patient_key = c.patient_key
LEFT JOIN fact_visits v ON p.patient_key = v.patient_key
LEFT JOIN fact_prescriptions rx ON p.patient_key = rx.patient_key
LEFT JOIN fact_quality_measures qm ON p.patient_key = qm.patient_key
WHERE p.is_current = TRUE
GROUP BY p.patient_id, p.risk_score;

-- Interview talking point:
-- "At Optum, we maintained conformed dimensions for patients, providers,
-- and diagnosis codes across 15+ fact tables. This enabled enterprise-wide
-- analytics like total cost of care and quality reporting across all lines
-- of business."
```

**Implementation Best Practices:**

1. **Central Dimension Authority** - One team owns each conformed dimension
2. **Governance** - Change control process for dimension changes
3. **Surrogate Keys** - Use same surrogate key assignment across all ETL processes
4. **Synchronization** - Update all fact table loads when dimension changes

**Common Conformed Dimensions:**
- dim_date (universal across all businesses)
- dim_customer
- dim_product
- dim_geography
- dim_employee

---

**Q11: What are bridge tables? When to use them?**

**Answer:**

**Bridge Tables** solve many-to-many relationships in dimensional modeling.

**Problem:**
Star schema assumes many-to-one relationships (many facts → one dimension), but sometimes we have many-to-many.

**Example Scenarios:**
- Products in multiple categories
- Patients with multiple diagnoses
- Accounts with multiple owners
- Orders with multiple shipping addresses

**Without Bridge Table (Wrong):**
```sql
-- ❌ Problem: How to link one patient to multiple diagnoses?
fact_visits (
    visit_key,
    patient_key,
    diagnosis_key,  -- Only one! ❌
    ...
)

-- Can't represent: Patient has diabetes + hypertension + CKD
```

**Solution 1: Bridge Table**
```sql
-- Fact table
CREATE TABLE fact_visits (
    visit_key INT PRIMARY KEY,
    patient_key INT,
    provider_key INT,
    visit_date_key INT,
    -- Measures
    visit_duration_minutes INT,
    cost DECIMAL(10,2)
);

-- Bridge table (many-to-many connector)
CREATE TABLE bridge_visit_diagnoses (
    visit_key INT,
    diagnosis_key INT,
    diagnosis_rank INT,        -- Primary, secondary, tertiary
    weight_factor DECIMAL(5,4), -- For allocation (sums to 1.0)
    PRIMARY KEY (visit_key, diagnosis_key)
);

-- Dimension
CREATE TABLE dim_diagnosis (
    diagnosis_key INT PRIMARY KEY,
    icd10_code VARCHAR(10),
    diagnosis_name VARCHAR(500),
    diagnosis_category VARCHAR(100)
);

-- Query: Visits by diagnosis
SELECT
    d.diagnosis_name,
    COUNT(DISTINCT b.visit_key) as visit_count,
    SUM(f.cost) as total_cost
FROM fact_visits f
JOIN bridge_visit_diagnoses b ON f.visit_key = b.visit_key
JOIN dim_diagnosis d ON b.diagnosis_key = d.diagnosis_key
GROUP BY d.diagnosis_name;
```

**Weight Factor for Allocation:**
```sql
-- Visit cost = $1000, 3 diagnoses
-- Allocate cost proportionally

visit_key | diagnosis | weight_factor | allocated_cost
123       | Diabetes  | 0.50         | $500
123       | Hypertension | 0.30      | $300
123       | CKD       | 0.20         | $200

-- Query with allocation:
SELECT
    d.diagnosis_name,
    SUM(f.cost * b.weight_factor) as allocated_cost
FROM fact_visits f
JOIN bridge_visit_diagnoses b ON f.visit_key = b.visit_key
JOIN dim_diagnosis d ON b.diagnosis_key = d.diagnosis_key
GROUP BY d.diagnosis_name;
```

**Optum Healthcare Example:**

```sql
-- Bridge table for patient comorbidities
CREATE TABLE bridge_patient_conditions (
    patient_key INT,
    diagnosis_key INT,
    condition_rank INT,        -- HCC rank (Hierarchical Condition Category)
    risk_weight DECIMAL(5,3),  -- For RAF score calculation
    onset_date DATE,
    is_chronic BOOLEAN,
    PRIMARY KEY (patient_key, diagnosis_key)
);

-- Query: Patients with diabetes AND hypertension (comorbidity analysis)
SELECT
    p.patient_id,
    COUNT(DISTINCT b.diagnosis_key) as condition_count,
    SUM(b.risk_weight) as total_risk_score
FROM dim_patient p
JOIN bridge_patient_conditions b ON p.patient_key = b.patient_key
JOIN dim_diagnosis d ON b.diagnosis_key = d.diagnosis_key
WHERE d.diagnosis_category IN ('Diabetes', 'Hypertension')
  AND p.is_current = TRUE
  AND b.is_chronic = TRUE
GROUP BY p.patient_id
HAVING COUNT(DISTINCT d.diagnosis_category) >= 2;  -- Both conditions

-- Interview talking point:
-- "At Optum, we used bridge tables to model patient comorbidities.
-- A single patient could have 10+ chronic conditions, and we needed
-- to analyze cost and quality across all condition combinations.
-- The bridge table with risk weights enabled accurate RAF score
-- calculation for Medicare Advantage risk adjustment."
```

**Alternative: Separate Fact Row (if weights not needed)**
```sql
-- Alternative: Create one fact row per diagnosis
CREATE TABLE fact_visit_diagnoses (
    visit_key INT,
    diagnosis_key INT,
    patient_key INT,
    provider_key INT,
    visit_date_key INT,
    -- Measures
    visit_duration_minutes INT,
    cost DECIMAL(10,2),  -- Duplicated across all diagnosis rows
    PRIMARY KEY (visit_key, diagnosis_key)
);

-- Simpler queries but duplicated measures
-- Use DISTINCT for counting visits
SELECT
    d.diagnosis_name,
    COUNT(DISTINCT f.visit_key) as visit_count  -- Must use DISTINCT
FROM fact_visit_diagnoses f
JOIN dim_diagnosis d ON f.diagnosis_key = d.diagnosis_key
GROUP BY d.diagnosis_name;
```

**When to Use Bridge vs Separate Rows:**

| Approach | Use When |
|----------|----------|
| **Bridge Table** | Need weight factors for allocation |
| | Measures at parent level (visit, not diagnosis) |
| | Fewer rows (parent-level grain) |
| **Separate Rows** | No allocation needed |
| | Simpler queries |
| | Measures are per combination |

---

**Q12: What is SCD Type 0? When to use?**

**Answer:**

**SCD Type 0** means the attribute **never changes** once set.

**Definition:**
- Original value is permanent
- No updates allowed
- Historical value is the "truth" forever

**Examples:**
```sql
CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY,
    customer_id VARCHAR(50),

    -- Type 0 attributes (never change)
    date_of_birth DATE,           -- Birth date is permanent
    original_signup_date DATE,     -- First registration
    customer_acquisition_source VARCHAR(50), -- Where they came from initially
    social_security_number VARCHAR(11),  -- Government ID

    -- Type 1 attributes (overwrite)
    email VARCHAR(200),
    phone VARCHAR(20),

    -- Type 2 attributes (track history)
    address VARCHAR(500),
    customer_tier VARCHAR(20),
    effective_date DATE,
    is_current BOOLEAN
);
```

**Why Type 0:**

**1. Immutable by Nature**
- Birth date can't change
- Original signup date is historical fact
- Government IDs rarely change

**2. Business Rules**
- "First touch" attribution must remain constant
- Original acquisition campaign shouldn't change

**3. Regulatory/Audit Requirements**
- Need to preserve original values
- Example: HIPAA patient identifiers

**ETL Handling:**
```sql
-- Initial load
INSERT INTO dim_customer (
    customer_id, date_of_birth, original_signup_date
)
VALUES ('CUST_123', '1985-05-15', '2024-01-01');

-- Later update (Type 0 ignored)
UPDATE dim_customer
SET email = 'newemail@example.com'  -- Type 1: OK to update
WHERE customer_id = 'CUST_123';
-- date_of_birth NOT updated even if source changed (likely data error)

-- Validation: Reject if Type 0 attribute changes
IF source.date_of_birth <> target.date_of_birth THEN
    RAISE ERROR 'Type 0 attribute changed! Possible data quality issue';
END IF;
```

**Optum Healthcare Example:**

```sql
CREATE TABLE dim_patient (
    patient_key INT PRIMARY KEY,
    patient_id VARCHAR(50),

    -- Type 0: Immutable patient identifiers
    date_of_birth DATE,                  -- Never changes
    original_enrollment_date DATE,        -- First enrollment in health plan
    medicare_beneficiary_id VARCHAR(20),  -- Government ID

    -- Type 1: Current contact info (overwrite)
    phone VARCHAR(20),
    email VARCHAR(200),

    -- Type 2: Track history
    address VARCHAR(500),
    insurance_plan VARCHAR(100),
    primary_care_physician_id VARCHAR(50),
    effective_date DATE,
    is_current BOOLEAN
);

-- Query: Patient tenure (based on Type 0 original_enrollment_date)
SELECT
    patient_id,
    original_enrollment_date,
    DATEDIFF(day, original_enrollment_date, CURRENT_DATE) / 365.25 as tenure_years,
    current_insurance_plan
FROM dim_patient
WHERE is_current = TRUE
  AND tenure_years >= 5;  -- Long-term members

-- Interview talking point:
-- "At Optum, we used Type 0 for patient date of birth and original
-- enrollment date. These values never changed even if source systems
-- sent corrections - we'd investigate data quality issues instead.
-- This ensured consistent cohort analysis and tenure calculations
-- for Medicare Stars quality reporting."
```

**Data Quality Monitoring:**
```sql
-- Alert if Type 0 attribute attempts to change
WITH type0_violations AS (
    SELECT
        s.patient_id,
        s.date_of_birth as source_dob,
        t.date_of_birth as target_dob
    FROM source_patients s
    JOIN dim_patient t ON s.patient_id = t.patient_id
    WHERE s.date_of_birth <> t.date_of_birth
      AND t.is_current = TRUE
)
SELECT
    COUNT(*) as violation_count,
    'Type 0 violation: date_of_birth changed' as alert
FROM type0_violations
HAVING COUNT(*) > 0;
```

**Summary - SCD Types:**

| Type | Change Handling | Use Case | Example |
|------|----------------|----------|---------|
| **Type 0** | Never changes | Immutable attributes | Birth date, original signup |
| **Type 1** | Overwrite | Don't care about history | Phone, email |
| **Type 2** | New row | Track full history | Address, price, status |
| **Type 3** | Add column | Track last change only | Previous address |

---

**Q13: Date dimension - what attributes to include? Best practices?**

**Answer:**

The **date dimension** is the most important conformed dimension - used by virtually every fact table.

**Comprehensive Date Dimension:**
```sql
CREATE TABLE dim_date (
    -- Primary key
    date_key INT PRIMARY KEY,        -- YYYYMMDD format: 20240503

    -- Basic date components
    full_date DATE UNIQUE NOT NULL,  -- 2024-05-03

    -- Day attributes
    day_of_month INT,                -- 1-31
    day_of_year INT,                 -- 1-366
    day_of_week INT,                 -- 1-7 (1=Monday, ISO 8601)
    day_name VARCHAR(10),            -- Monday, Tuesday, etc.
    day_abbr VARCHAR(3),             -- Mon, Tue, etc.

    -- Week attributes
    week_of_year INT,                -- 1-53 (ISO week)
    week_of_month INT,               -- 1-5
    first_day_of_week DATE,
    last_day_of_week DATE,

    -- Month attributes
    month INT,                       -- 1-12
    month_name VARCHAR(10),          -- January, February, etc.
    month_abbr VARCHAR(3),           -- Jan, Feb, etc.
    first_day_of_month DATE,
    last_day_of_month DATE,
    days_in_month INT,               -- 28-31

    -- Quarter attributes
    quarter INT,                     -- 1-4
    quarter_name VARCHAR(10),        -- Q1 2024, Q2 2024
    first_day_of_quarter DATE,
    last_day_of_quarter DATE,

    -- Year attributes
    year INT,                        -- 2024
    year_month INT,                  -- 202405
    year_quarter INT,                -- 20242

    -- Fiscal period attributes (if different from calendar)
    fiscal_year INT,                 -- Example: Fiscal year starts July 1
    fiscal_quarter INT,
    fiscal_month INT,
    fiscal_week INT,

    -- Business day flags
    is_weekend BOOLEAN,              -- Saturday or Sunday
    is_weekday BOOLEAN,              -- Monday-Friday
    is_business_day BOOLEAN,         -- Weekday and not a holiday

    -- Holiday flags
    is_holiday BOOLEAN,
    holiday_name VARCHAR(100),       -- New Year's Day, Christmas, etc.
    is_federal_holiday BOOLEAN,
    is_company_holiday BOOLEAN,

    -- Special periods
    is_leap_year BOOLEAN,
    is_month_end BOOLEAN,
    is_quarter_end BOOLEAN,
    is_year_end BOOLEAN,

    -- Relative period offsets (useful for queries)
    days_from_today INT,             -- Negative for past, 0 for today, positive for future
    weeks_from_today INT,
    months_from_today INT,

    -- Seasonal attributes
    season VARCHAR(10)               -- Spring, Summer, Fall, Winter
);
```

**Generate Date Dimension Data:**

**PostgreSQL:**
```sql
INSERT INTO dim_date
SELECT
    TO_CHAR(date, 'YYYYMMDD')::INT as date_key,
    date as full_date,
    EXTRACT(DAY FROM date) as day_of_month,
    EXTRACT(DOY FROM date) as day_of_year,
    EXTRACT(ISODOW FROM date) as day_of_week,  -- 1=Monday
    TO_CHAR(date, 'Day') as day_name,
    TO_CHAR(date, 'Dy') as day_abbr,
    EXTRACT(WEEK FROM date) as week_of_year,
    CEIL(EXTRACT(DAY FROM date) / 7.0) as week_of_month,
    DATE_TRUNC('week', date)::DATE as first_day_of_week,
    (DATE_TRUNC('week', date) + INTERVAL '6 days')::DATE as last_day_of_week,
    EXTRACT(MONTH FROM date) as month,
    TO_CHAR(date, 'Month') as month_name,
    TO_CHAR(date, 'Mon') as month_abbr,
    DATE_TRUNC('month', date)::DATE as first_day_of_month,
    (DATE_TRUNC('month', date) + INTERVAL '1 month - 1 day')::DATE as last_day_of_month,
    EXTRACT(DAY FROM (DATE_TRUNC('month', date) + INTERVAL '1 month - 1 day')) as days_in_month,
    EXTRACT(QUARTER FROM date) as quarter,
    'Q' || EXTRACT(QUARTER FROM date) || ' ' || EXTRACT(YEAR FROM date) as quarter_name,
    DATE_TRUNC('quarter', date)::DATE as first_day_of_quarter,
    (DATE_TRUNC('quarter', date) + INTERVAL '3 months - 1 day')::DATE as last_day_of_quarter,
    EXTRACT(YEAR FROM date) as year,
    TO_CHAR(date, 'YYYYMM')::INT as year_month,
    (EXTRACT(YEAR FROM date)::TEXT || EXTRACT(QUARTER FROM date)::TEXT)::INT as year_quarter,

    -- Fiscal year (example: starts July 1)
    CASE
        WHEN EXTRACT(MONTH FROM date) >= 7 THEN EXTRACT(YEAR FROM date)
        ELSE EXTRACT(YEAR FROM date) - 1
    END as fiscal_year,

    CASE
        WHEN EXTRACT(MONTH FROM date) BETWEEN 7 AND 9 THEN 1
        WHEN EXTRACT(MONTH FROM date) BETWEEN 10 AND 12 THEN 2
        WHEN EXTRACT(MONTH FROM date) BETWEEN 1 AND 3 THEN 3
        ELSE 4
    END as fiscal_quarter,

    -- Business day flags
    EXTRACT(ISODOW FROM date) IN (6, 7) as is_weekend,
    EXTRACT(ISODOW FROM date) BETWEEN 1 AND 5 as is_weekday,

    -- Holidays (simplified - would join with holiday table in production)
    CASE
        WHEN EXTRACT(MONTH FROM date) = 1 AND EXTRACT(DAY FROM date) = 1 THEN TRUE  -- New Year
        WHEN EXTRACT(MONTH FROM date) = 7 AND EXTRACT(DAY FROM date) = 4 THEN TRUE  -- July 4th
        WHEN EXTRACT(MONTH FROM date) = 12 AND EXTRACT(DAY FROM date) = 25 THEN TRUE -- Christmas
        ELSE FALSE
    END as is_holiday,

    -- Leap year
    (EXTRACT(YEAR FROM date) % 4 = 0 AND EXTRACT(YEAR FROM date) % 100 != 0)
    OR (EXTRACT(YEAR FROM date) % 400 = 0) as is_leap_year,

    -- Month/quarter/year end
    date = (DATE_TRUNC('month', date) + INTERVAL '1 month - 1 day')::DATE as is_month_end,
    date = (DATE_TRUNC('quarter', date) + INTERVAL '3 months - 1 day')::DATE as is_quarter_end,
    date = (DATE_TRUNC('year', date) + INTERVAL '1 year - 1 day')::DATE as is_year_end,

    -- Relative offsets
    (date - CURRENT_DATE) as days_from_today,

    -- Season
    CASE
        WHEN EXTRACT(MONTH FROM date) IN (3, 4, 5) THEN 'Spring'
        WHEN EXTRACT(MONTH FROM date) IN (6, 7, 8) THEN 'Summer'
        WHEN EXTRACT(MONTH FROM date) IN (9, 10, 11) THEN 'Fall'
        ELSE 'Winter'
    END as season

FROM generate_series('2020-01-01'::DATE, '2030-12-31'::DATE, '1 day'::INTERVAL) date;
```

**Optum Healthcare Example Queries:**

```sql
-- Query 1: Claims volume by day of week (identify patterns)
SELECT
    d.day_name,
    d.day_of_week,
    COUNT(*) as claim_count,
    SUM(f.paid_amount) as total_paid
FROM fact_claims f
JOIN dim_date d ON f.service_date_key = d.date_key
WHERE d.year = 2024
GROUP BY d.day_name, d.day_of_week
ORDER BY d.day_of_week;

-- Query 2: Month-end processing volume (capacity planning)
SELECT
    d.year_month,
    d.month_name,
    COUNT(CASE WHEN d.is_month_end THEN 1 END) as month_end_claims,
    COUNT(*) as total_claims,
    ROUND(100.0 * COUNT(CASE WHEN d.is_month_end THEN 1 END) / COUNT(*), 2) as pct_month_end
FROM fact_claims f
JOIN dim_date d ON f.received_date_key = d.date_key
WHERE d.year = 2024
GROUP BY d.year_month, d.month_name
ORDER BY d.year_month;

-- Query 3: Fiscal quarter reporting (Medicare Advantage)
SELECT
    d.fiscal_year,
    d.fiscal_quarter,
    COUNT(DISTINCT p.patient_id) as member_count,
    SUM(f.paid_amount) as total_cost,
    SUM(f.paid_amount) / COUNT(DISTINCT p.patient_id) as cost_per_member
FROM fact_claims f
JOIN dim_date d ON f.service_date_key = d.date_key
JOIN dim_patient p ON f.patient_key = p.patient_key
WHERE d.fiscal_year = 2024
  AND p.is_current = TRUE
GROUP BY d.fiscal_year, d.fiscal_quarter
ORDER BY d.fiscal_year, d.fiscal_quarter;

-- Query 4: Business days only (exclude weekends and holidays)
SELECT
    COUNT(DISTINCT d.date_key) as business_days,
    SUM(f.claim_count) / COUNT(DISTINCT d.date_key) as avg_claims_per_business_day
FROM fact_daily_claims f
JOIN dim_date d ON f.date_key = d.date_key
WHERE d.year = 2024
  AND d.is_business_day = TRUE;

-- Interview talking point:
-- "At Optum, the date dimension was critical for Medicare Advantage
-- reporting which operates on fiscal quarters (not calendar). We also
-- used it to identify processing patterns - claims spiked 40% on
-- month-end days, which informed our capacity planning for Kafka
-- partition scaling."
```

**Best Practices:**

1. **Pre-populate** - Generate all dates upfront (10-20 years)
2. **Integer keys** - YYYYMMDD format faster than DATE joins
3. **No gaps** - Every day represented, even if no activity
4. **Time-independent** - Never use "days from today" in dimension itself (calculate in query)
5. **Conformed** - One date dimension for entire enterprise

**Special Variations:**

**Date-Time Dimension (if needed):**
```sql
-- For minute/hour-level analysis
CREATE TABLE dim_datetime (
    datetime_key BIGINT PRIMARY KEY,  -- YYYYMMDDHHmm: 202405031430
    full_datetime TIMESTAMP,
    date_key INT,
    hour INT,  -- 0-23
    minute INT, -- 0-59
    hour_name VARCHAR(10),  -- 2:30 PM
    is_business_hours BOOLEAN  -- 9 AM - 5 PM weekdays
);
```

---

**Q14: Should time be separate from date dimension?**

**Answer:**

**It depends on your grain and query patterns.**

**Option 1: Combine Date + Time (datetime dimension)**
**Option 2: Separate dim_date + dim_time**

**When to Use Separate Time Dimension:**

**Use Case 1: High-Cardinality Time Analysis**
```sql
-- If analyzing patterns by time of day independent of specific date
-- Example: Call center volume by hour/minute

CREATE TABLE dim_time (
    time_key INT PRIMARY KEY,     -- HHmmss format: 143000 (2:30:00 PM)
    full_time TIME,                -- 14:30:00
    hour INT,                      -- 0-23
    hour_12 INT,                   -- 1-12
    minute INT,                    -- 0-59
    second INT,                    -- 0-59
    am_pm VARCHAR(2),              -- AM, PM
    hour_name VARCHAR(10),         -- 2:30 PM
    minute_of_day INT,             -- 0-1439 (60*24)
    second_of_day INT,             -- 0-86399

    -- Business time buckets
    is_business_hours BOOLEAN,     -- 9 AM - 5 PM
    time_bucket VARCHAR(20),       -- Morning, Afternoon, Evening, Night
    shift VARCHAR(20)              -- Day shift, Night shift, etc.
);

-- Fact table uses both dimensions
CREATE TABLE fact_call_center (
    call_date_key INT,             -- References dim_date
    call_time_key INT,             -- References dim_time
    customer_key INT,
    agent_key INT,
    call_duration_seconds INT,
    wait_time_seconds INT
);

-- Query: Call volume by hour of day (across all dates)
SELECT
    t.hour,
    t.hour_name,
    COUNT(*) as call_count,
    AVG(f.wait_time_seconds) as avg_wait_time
FROM fact_call_center f
JOIN dim_time t ON f.call_time_key = t.time_key
GROUP BY t.hour, t.hour_name
ORDER BY t.hour;
-- ✅ Aggregates across all dates to find hourly patterns
```

**Use Case 2: Intraday Trading/Financial Data**
```sql
-- Stock trades at second-level granularity
CREATE TABLE fact_trades (
    trade_date_key INT,
    trade_time_key INT,  -- Separate time dimension
    security_key INT,
    price DECIMAL(10,2),
    volume INT
);

-- dim_time with second-level detail
CREATE TABLE dim_time (
    time_key INT PRIMARY KEY,  -- HHmmss: 093015 (9:30:15 AM)
    hour INT,
    minute INT,
    second INT,
    is_market_hours BOOLEAN,   -- 9:30 AM - 4:00 PM EST
    time_bucket VARCHAR(20)    -- Market open, Mid-day, Market close
);
```

**When to Combine (datetime dimension):**

**Use Case: Specific Point-in-Time Analysis**
```sql
-- When you care about specific datetime combinations
CREATE TABLE dim_datetime (
    datetime_key BIGINT PRIMARY KEY,  -- YYYYMMDDHHmmss
    full_datetime TIMESTAMP,
    date_key INT,  -- Can still reference date separately
    hour INT,
    minute INT,
    second INT,
    day_of_week INT,
    is_weekend BOOLEAN,
    is_business_hours BOOLEAN
);

-- Fact table
CREATE TABLE fact_website_clicks (
    click_datetime_key BIGINT,  -- Single datetime reference
    user_key INT,
    page_key INT,
    session_duration_seconds INT
);
```

**Optum Healthcare Example:**

**Scenario 1: Separate dim_date + dim_time (Patient Visits)**
```sql
-- Patient check-in times (time patterns matter)
CREATE TABLE dim_time (
    time_key INT PRIMARY KEY,  -- HHmm format: 1430
    hour INT,
    minute INT,
    hour_name VARCHAR(10),
    is_business_hours BOOLEAN,  -- 8 AM - 6 PM
    time_slot VARCHAR(20)       -- 8-9 AM, 9-10 AM, etc.
);

CREATE TABLE fact_patient_checkins (
    checkin_date_key INT,  -- dim_date
    checkin_time_key INT,  -- dim_time
    patient_key INT,
    provider_key INT,
    clinic_key INT,
    wait_time_minutes INT
);

-- Query: Average wait time by hour (staffing optimization)
SELECT
    t.hour_name,
    t.time_slot,
    COUNT(*) as checkin_count,
    AVG(f.wait_time_minutes) as avg_wait_time
FROM fact_patient_checkins f
JOIN dim_time t ON f.checkin_time_key = t.time_key
JOIN dim_date d ON f.checkin_date_key = d.date_key
WHERE d.is_weekday = TRUE
  AND d.year = 2024
GROUP BY t.hour_name, t.time_slot
ORDER BY t.time_slot;

-- Interview talking point:
-- "At Optum, we used separate time dimension for clinic operations
-- analysis. This allowed us to identify that wait times spiked 50%
-- between 9-10 AM on Mondays, leading to staffing adjustments."
```

**Scenario 2: Combined datetime (Claims Received Timestamp)**
```sql
-- Claims received timestamp (specific point in time)
CREATE TABLE fact_claims_received (
    received_datetime_key BIGINT,  -- Combined datetime
    claim_id VARCHAR(50),
    patient_key INT,
    processing_time_seconds INT
);

-- Query by specific datetime
SELECT
    DATE_TRUNC('hour', dt.full_datetime) as hour,
    COUNT(*) as claims_received,
    AVG(f.processing_time_seconds) as avg_processing_time
FROM fact_claims_received f
JOIN dim_datetime dt ON f.received_datetime_key = dt.datetime_key
WHERE dt.full_datetime BETWEEN '2024-05-01' AND '2024-05-31'
GROUP BY DATE_TRUNC('hour', dt.full_datetime);
```

**Decision Matrix:**

| Factor | Separate dim_time | Combined datetime |
|--------|-------------------|-------------------|
| **Pattern analysis** (time of day across dates) | ✅ Better | ❌ Harder |
| **Storage** | ✅ Less (dim_time has 86400 rows max) | ❌ More (unique datetimes) |
| **Point-in-time queries** | ❌ Harder (join two dims) | ✅ Easier |
| **Grain** | Minute/second level, many events | Hour level or coarser |
| **Use case** | Call centers, appointments, trading | Web clicks, transactions |

**Recommendation:**
- **Separate** if analyzing time-of-day patterns independent of date
- **Combined** if specific datetime combinations matter
- **Both** possible: Have datetime dimension that references separate date dimension for flexibility

---

**Q15: Fact table partitioning - strategies and benefits?**

**Answer:**

**Partitioning** divides large fact tables into smaller, manageable pieces for performance and maintenance.

**Why Partition:**
- ✅ Query performance (partition pruning)
- ✅ Faster loads (parallel insertion)
- ✅ Easier maintenance (drop old partitions instead of DELETE)
- ✅ Backup efficiency (partition-level backups)

**Most Common: Partition by Date**

**PostgreSQL Example:**
```sql
-- Range partitioning by date_key
CREATE TABLE fact_sales (
    sale_id BIGINT,
    sale_date_key INT,
    customer_key INT,
    product_key INT,
    quantity INT,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (sale_date_key);

-- Create partitions (monthly)
CREATE TABLE fact_sales_202401 PARTITION OF fact_sales
    FOR VALUES FROM (20240101) TO (20240201);

CREATE TABLE fact_sales_202402 PARTITION OF fact_sales
    FOR VALUES FROM (20240201) TO (20240301);

CREATE TABLE fact_sales_202403 PARTITION OF fact_sales
    FOR VALUES FROM (20240301) TO (20240401);

-- Future partitions...
CREATE TABLE fact_sales_202412 PARTITION OF fact_sales
    FOR VALUES FROM (20241201) TO (20250101);

-- Query with partition pruning
SELECT SUM(amount)
FROM fact_sales
WHERE sale_date_key BETWEEN 20240201 AND 20240229;
-- ✅ Only scans fact_sales_202402 partition!

-- EXPLAIN shows partition pruning:
-- Partitions scanned: fact_sales_202402
```

**MySQL Partitioning:**
```sql
CREATE TABLE fact_orders (
    order_id BIGINT,
    order_date_key INT,
    customer_key INT,
    order_amount DECIMAL(10,2)
)
PARTITION BY RANGE (order_date_key) (
    PARTITION p202401 VALUES LESS THAN (20240201),
    PARTITION p202402 VALUES LESS THAN (20240301),
    PARTITION p202403 VALUES LESS THAN (20240401),
    PARTITION p202404 VALUES LESS THAN (20240501),
    PARTITION pmax VALUES LESS THAN MAXVALUE
);

-- Add new partition
ALTER TABLE fact_orders ADD PARTITION (
    PARTITION p202405 VALUES LESS THAN (20240601)
);

-- Drop old partition (much faster than DELETE)
ALTER TABLE fact_orders DROP PARTITION p202401;
```

**Optum Healthcare Example:**

```sql
-- Claims fact table partitioned by service_date_key (monthly)
CREATE TABLE fact_claims (
    claim_id VARCHAR(50),
    claim_line_number INT,
    service_date_key INT,
    received_date_key INT,
    patient_key INT,
    provider_key INT,
    diagnosis_key INT,
    procedure_key INT,
    billed_amount DECIMAL(10,2),
    paid_amount DECIMAL(10,2)
) PARTITION BY RANGE (service_date_key);

-- Monthly partitions
CREATE TABLE fact_claims_202401 PARTITION OF fact_claims
    FOR VALUES FROM (20240101) TO (20240201);

CREATE TABLE fact_claims_202402 PARTITION OF fact_claims
    FOR VALUES FROM (20240201) TO (20240301);

-- ... (create partitions for 24 months rolling)

-- Query: Current month claims (partition pruning)
SELECT
    COUNT(*) as claim_count,
    SUM(paid_amount) as total_paid
FROM fact_claims
WHERE service_date_key BETWEEN 20240501 AND 20240531;
-- ✅ Scans only fact_claims_202405 (1/24th of data!)

-- Query: Retention policy - drop partitions older than 24 months
DROP TABLE fact_claims_202201;  -- Much faster than DELETE

-- Interview talking point:
-- "At Optum, we partitioned our 1.2 billion row claims fact table by
-- service_date_key using monthly partitions. This reduced query time
-- from 45 seconds to 3 seconds for monthly reporting and allowed us
-- to drop old partitions in milliseconds instead of hours-long DELETE
-- operations. We maintained a 24-month rolling window and automated
-- partition creation/deletion via Airflow DAGs."
```

**Other Partitioning Strategies:**

**List Partitioning (by category):**
```sql
-- Partition by region
CREATE TABLE fact_sales (
    sale_id BIGINT,
    region VARCHAR(20),
    amount DECIMAL(10,2)
) PARTITION BY LIST (region);

CREATE TABLE fact_sales_north PARTITION OF fact_sales
    FOR VALUES IN ('North', 'Northeast', 'Northwest');

CREATE TABLE fact_sales_south PARTITION OF fact_sales
    FOR VALUES IN ('South', 'Southeast', 'Southwest');

CREATE TABLE fact_sales_other PARTITION OF fact_sales
    DEFAULT;  -- Catch-all for other regions
```

**Hash Partitioning (distribute evenly):**
```sql
-- Distribute across 8 partitions for parallel processing
CREATE TABLE fact_events (
    event_id BIGINT,
    user_id BIGINT,
    event_data JSONB
) PARTITION BY HASH (user_id);

CREATE TABLE fact_events_p0 PARTITION OF fact_events
    FOR VALUES WITH (MODULUS 8, REMAINDER 0);

CREATE TABLE fact_events_p1 PARTITION OF fact_events
    FOR VALUES WITH (MODULUS 8, REMAINDER 1);

-- ... p2 through p7

-- Benefit: Parallel query processing across partitions
```

**Sub-Partitioning (composite):**
```sql
-- Partition by date, sub-partition by region
CREATE TABLE fact_sales (
    sale_id BIGINT,
    sale_date_key INT,
    region VARCHAR(20),
    amount DECIMAL(10,2)
) PARTITION BY RANGE (sale_date_key);

CREATE TABLE fact_sales_202401 PARTITION OF fact_sales
    FOR VALUES FROM (20240101) TO (20240201)
    PARTITION BY LIST (region);

CREATE TABLE fact_sales_202401_north PARTITION OF fact_sales_202401
    FOR VALUES IN ('North');

CREATE TABLE fact_sales_202401_south PARTITION OF fact_sales_202401
    FOR VALUES IN ('South');
```

**Automated Partition Management:**

```sql
-- Airflow DAG (pseudocode) for monthly partition creation
def create_next_month_partition():
    next_month = (current_month + 1).strftime('%Y%m')
    start_key = f"{next_month}01"
    end_key = f"{next_month_plus_1}01"

    sql = f"""
    CREATE TABLE fact_claims_{next_month}
    PARTITION OF fact_claims
    FOR VALUES FROM ({start_key}) TO ({end_key});
    """
    execute(sql)

-- Scheduled to run on 25th of each month (before month-end)

def drop_old_partitions():
    retention_months = 24
    drop_before = (current_month - retention_months).strftime('%Y%m')

    sql = f"DROP TABLE IF EXISTS fact_claims_{drop_before};"
    execute(sql)

-- Scheduled monthly
```

**Benefits Summary:**

| Benefit | Without Partitioning | With Partitioning |
|---------|---------------------|-------------------|
| **Query time** | Scan all 1B rows | Scan 40M rows (1 month) |
| **Data load** | Lock entire table | Lock one partition |
| **Delete old data** | Hours (DELETE) | Milliseconds (DROP partition) |
| **Backup** | Full table only | Per-partition |
| **Parallel processing** | Limited | Partition-wise parallelism |

---

**Q16: Late-arriving facts - how to handle?**

**Answer:**

**Late-arriving facts** occur when fact records arrive after their related dimension records, creating orphaned facts.

**Problem Scenario:**
```
Day 1: Customer places order (order_date = May 1)
Day 2: Dimension load runs - no customer record yet (not in source)
Day 2: Fact load runs - fact_orders needs customer_key... ❌ FAIL!
Day 3: Customer dimension record finally arrives
```

**Solution Strategies:**

**Strategy 1: Default/Placeholder Dimension Row**

```sql
-- Create default dimension row
INSERT INTO dim_customer (customer_key, customer_id, name)
VALUES (-1, 'UNKNOWN', 'Unknown Customer - Late Arriving');

-- Fact load: Use placeholder if dimension not found
INSERT INTO fact_orders (
    order_date_key,
    customer_key,  -- Use -1 if real customer not found
    product_key,
    amount
)
SELECT
    order_date_key,
    COALESCE(c.customer_key, -1) as customer_key,  -- ✅ Placeholder
    product_key,
    amount
FROM staging_orders s
LEFT JOIN dim_customer c ON s.customer_id = c.customer_id
                         AND c.is_current = TRUE;

-- Later: Backfill when dimension arrives
UPDATE fact_orders f
SET customer_key = c.customer_key
FROM dim_customer c
WHERE f.customer_key = -1
  AND f.order_id IN (
      SELECT order_id FROM staging_orders WHERE customer_id = c.customer_id
  );
```

**Strategy 2: Hold Facts in Staging**

```sql
-- Load facts only if all dimensions exist
CREATE TABLE staging_orders_pending (
    order_id VARCHAR(50),
    customer_id VARCHAR(50),
    product_id VARCHAR(50),
    amount DECIMAL(10,2),
    load_attempt_count INT DEFAULT 1,
    first_seen_date DATE DEFAULT CURRENT_DATE
);

-- Fact load: Only load if dimensions exist
INSERT INTO fact_orders
SELECT
    order_date_key,
    c.customer_key,
    p.product_key,
    amount
FROM staging_orders s
JOIN dim_customer c ON s.customer_id = c.customer_id AND c.is_current = TRUE
JOIN dim_product p ON s.product_id = p.product_id AND p.is_current = TRUE;

-- Move unmatched to pending
INSERT INTO staging_orders_pending (order_id, customer_id, product_id, amount)
SELECT order_id, customer_id, product_id, amount
FROM staging_orders s
WHERE NOT EXISTS (
    SELECT 1 FROM dim_customer c
    WHERE s.customer_id = c.customer_id AND c.is_current = TRUE
);

-- Retry pending (scheduled daily)
-- Try to load pending facts again
INSERT INTO fact_orders
SELECT ...
FROM staging_orders_pending s
JOIN dim_customer c ON s.customer_id = c.customer_id;

-- Remove successfully loaded
DELETE FROM staging_orders_pending
WHERE order_id IN (SELECT order_id FROM fact_orders);

-- Alert if pending too long
SELECT *
FROM staging_orders_pending
WHERE first_seen_date < CURRENT_DATE - 7;  -- ❌ Stuck for 7 days
```

**Strategy 3: Infer Dimension Attributes**

```sql
-- Create minimal dimension row from fact data
INSERT INTO dim_customer (
    customer_id,
    name,
    email,
    is_inferred,  -- Flag as inferred
    effective_date
)
SELECT DISTINCT
    customer_id,
    'Unknown - ' || customer_id as name,  -- Placeholder name
    NULL as email,
    TRUE as is_inferred,
    CURRENT_DATE
FROM staging_orders
WHERE customer_id NOT IN (SELECT customer_id FROM dim_customer);

-- Update when real dimension arrives
UPDATE dim_customer
SET name = s.name,
    email = s.email,
    is_inferred = FALSE
FROM source_customers s
WHERE dim_customer.customer_id = s.customer_id
  AND dim_customer.is_inferred = TRUE;
```

**Optum Healthcare Example:**

```sql
-- Late-arriving patient dimensions (patient not enrolled yet)
-- Patient receives service before enrollment processed

-- Step 1: Create placeholder patient
INSERT INTO dim_patient (
    patient_key,
    patient_id,
    date_of_birth,
    is_inferred
)
VALUES (
    -1,  -- Placeholder key
    'UNKNOWN',
    NULL,
    FALSE  -- Standard placeholder
);

-- Also create inferred patients
CREATE TABLE dim_patient (
    patient_key INT PRIMARY KEY,
    patient_id VARCHAR(50),
    date_of_birth DATE,
    gender VARCHAR(10),
    insurance_plan VARCHAR(100),
    is_inferred BOOLEAN DEFAULT FALSE,  -- Marks inferred records
    effective_date DATE,
    is_current BOOLEAN
);

-- Claim load with late-arriving patient handling
INSERT INTO fact_claims (
    claim_id,
    claim_line_number,
    service_date_key,
    patient_key,  -- Might be placeholder or inferred
    provider_key,
    paid_amount
)
SELECT
    s.claim_id,
    s.claim_line_number,
    s.service_date_key,

    -- Try to find real patient, else create inferred
    COALESCE(
        (SELECT patient_key FROM dim_patient p
         WHERE p.patient_id = s.patient_id AND p.is_current = TRUE),

        -- Create inferred patient on-the-fly
        (INSERT INTO dim_patient (patient_id, date_of_birth, is_inferred, effective_date, is_current)
         VALUES (s.patient_id, s.patient_dob_from_claim, TRUE, CURRENT_DATE, TRUE)
         RETURNING patient_key)
    ) as patient_key,

    s.provider_key,
    s.paid_amount
FROM staging_claims s;

-- Weekly reconciliation: Update inferred patients
UPDATE dim_patient p
SET
    date_of_birth = e.date_of_birth,
    gender = e.gender,
    insurance_plan = e.insurance_plan,
    is_inferred = FALSE
FROM enrollment_data e
WHERE p.patient_id = e.patient_id
  AND p.is_inferred = TRUE;

-- Monitoring: Alert on high percentage of inferred
SELECT
    COUNT(CASE WHEN is_inferred THEN 1 END) as inferred_count,
    COUNT(*) as total_count,
    ROUND(100.0 * COUNT(CASE WHEN is_inferred THEN 1 END) / COUNT(*), 2) as pct_inferred
FROM dim_patient
WHERE is_current = TRUE
HAVING pct_inferred > 5.0;  -- ❌ Alert if > 5%

-- Interview talking point:
-- "At Optum, we handled late-arriving patient dimensions by creating
-- inferred patient records with minimal attributes from the claim itself.
-- We ran weekly reconciliation jobs to backfill complete patient data
-- from enrollment systems. This prevented claim processing delays while
-- maintaining data quality - only 2% of patients remained inferred after
-- 30 days."
```

**Best Practices:**

1. **Always have a placeholder** - Dimension key = -1 for "Unknown"
2. **Flag inferred records** - Track which were created from facts
3. **Retry logic** - Attempt to resolve pending facts periodically
4. **Alerts** - Monitor late-arriving fact counts
5. **Documentation** - Explain business rules for handling unknowns

---

**Q17: Early-arriving facts - how to handle?**

**Answer:**

**Early-arriving facts** occur when the fact record references a dimension that hasn't been loaded yet.

**Problem:**
```
Day 1: Order placed for new product
Day 1: Fact load runs - needs product_key for "Product XYZ"
Day 1: Dimension load hasn't run yet - product not in dim_product
Day 2: Product dimension loads
```

**Same solutions as late-arriving facts!**

The distinction between "late" and "early" is mostly semantic - both result in orphaned facts.

**Solutions** (same as Q16):

1. **Default dimension row** (-1 placeholder)
2. **Hold facts in staging** until dimension arrives
3. **Infer dimension** from fact attributes

**Additional consideration - Load Order:**

```python
# Airflow DAG dependency to prevent early-arriving facts
with DAG('daily_dwh_load'):
    # Dimensions FIRST
    load_dim_customer = PythonOperator(...)
    load_dim_product = PythonOperator(...)
    load_dim_provider = PythonOperator(...)

    # Facts AFTER all dimensions loaded
    load_fact_orders = PythonOperator(...)
    load_fact_claims = PythonOperator(...)

    # Dependencies
    [load_dim_customer, load_dim_product, load_dim_provider] >> load_fact_orders
    [load_dim_customer, load_dim_provider] >> load_fact_claims
```

**Optum Example - Proper Load Sequence:**

```python
# Airflow DAG for claims warehouse
with DAG('claims_dwh_daily', schedule_interval='@daily'):

    # Stage 1: Load all dimensions first
    dim_patient_task = load_dimension('dim_patient')
    dim_provider_task = load_dimension('dim_provider')
    dim_diagnosis_task = load_dimension('dim_diagnosis')
    dim_procedure_task = load_dimension('dim_procedure')

    # Stage 2: Load facts only after ALL dimensions complete
    fact_claims_task = load_fact('fact_claims')

    # Dependencies ensure no early-arriving facts
    [dim_patient_task, dim_provider_task,
     dim_diagnosis_task, dim_procedure_task] >> fact_claims_task

    # Interview talking point:
    # "At Optum, we structured our Airflow DAGs to load all dimensions
    # before facts, preventing early-arriving facts. If a dimension failed,
    # facts wouldn't load, maintaining referential integrity."
```

---

**Q18: What are mini-dimensions? When to use?**

**Answer:**

**Mini-dimensions** split rapidly changing attributes from a main dimension into a separate, smaller dimension.

**Problem: Rapidly Changing Attributes**
```sql
-- dim_customer with Type 2 SCD
-- Problem: Customer buys products daily, triggering new dimension rows!

customer_key | customer_id | name  | total_purchases | loyalty_points | effective_date
1001         | CUST_123    | Alice | 5               | 50             | 2024-01-01
1002         | CUST_123    | Alice | 6               | 65             | 2024-01-02  -- New row!
1003         | CUST_123    | Alice | 7               | 80             | 2024-01-03  -- New row!
... (365 rows per year per customer!) ❌
```

**Solution: Mini-Dimension**
```sql
-- Main dimension (slowly changing attributes only)
CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY,
    customer_id VARCHAR(50),
    name VARCHAR(200),
    email VARCHAR(200),
    registration_date DATE,
    -- Type 2
    effective_date DATE,
    is_current BOOLEAN
);
-- Stays stable (few rows)

-- Mini-dimension (rapidly changing attributes)
CREATE TABLE dim_customer_behavior (
    behavior_key INT PRIMARY KEY,
    purchase_count_range VARCHAR(20),  -- Bands: 0-10, 11-50, 51-100, 100+
    loyalty_tier VARCHAR(20),          -- Bronze, Silver, Gold, Platinum
    avg_order_value_range VARCHAR(20)  -- <$50, $50-$100, $100-$500, $500+
);
-- Small, static (16 combinations = 4 tiers × 4 ranges)

-- Fact table references both
CREATE TABLE fact_orders (
    order_date_key INT,
    customer_key INT,          -- Main dimension
    behavior_key INT,          -- Mini-dimension (snapshot at time of order)
    product_key INT,
    order_amount DECIMAL(10,2)
);

-- Fact load: Determine behavior band at order time
INSERT INTO fact_orders
SELECT
    order_date_key,
    c.customer_key,

    -- Determine behavior_key based on current metrics
    (SELECT behavior_key FROM dim_customer_behavior
     WHERE purchase_count_range = CASE
         WHEN customer_purchase_count BETWEEN 0 AND 10 THEN '0-10'
         WHEN customer_purchase_count BETWEEN 11 AND 50 THEN '11-50'
         WHEN customer_purchase_count BETWEEN 51 AND 100 THEN '51-100'
         ELSE '100+'
     END
     AND loyalty_tier = customer_current_tier
    ) as behavior_key,

    product_key,
    order_amount
FROM staging_orders;
```

**Benefits:**
- ✅ Prevents dimension explosion (365 rows/customer → stable dimension + 16-row mini-dim)
- ✅ Captures customer state at transaction time
- ✅ Enables analysis by behavior segments

**Optum Healthcare Example:**

```sql
-- Problem: Patient risk scores change monthly (HCC-based)
-- Storing in dim_patient creates 12 rows/year per patient × 10M patients = 120M rows!

-- Main patient dimension (stable attributes)
CREATE TABLE dim_patient (
    patient_key INT PRIMARY KEY,
    patient_id VARCHAR(50),
    date_of_birth DATE,
    gender VARCHAR(10),
    -- Type 2 for address changes (infrequent)
    address VARCHAR(500),
    effective_date DATE,
    is_current BOOLEAN
);

-- Mini-dimension: Patient risk profile (changes monthly)
CREATE TABLE dim_patient_risk_profile (
    risk_profile_key INT PRIMARY KEY,
    risk_score_band VARCHAR(20),    -- Low (0-1.0), Medium (1.0-2.0), High (2.0+)
    chronic_condition_count VARCHAR(20), -- 0, 1-2, 3-5, 6+
    age_band VARCHAR(20),           -- 0-17, 18-64, 65-74, 75+
    cost_tier VARCHAR(20)           -- Low, Medium, High, Very High
);
-- Only 4 × 4 × 4 × 4 = 256 rows total!

-- Insert all combinations
INSERT INTO dim_patient_risk_profile
SELECT
    ROW_NUMBER() OVER () as risk_profile_key,
    risk_score_band,
    chronic_condition_count,
    age_band,
    cost_tier
FROM
    (VALUES ('Low'), ('Medium'), ('High')) as r(risk_score_band),
    (VALUES ('0'), ('1-2'), ('3-5'), ('6+')) as c(chronic_condition_count),
    (VALUES ('0-17'), ('18-64'), ('65-74'), ('75+')) as a(age_band),
    (VALUES ('Low'), ('Medium'), ('High'), ('Very High')) as t(cost_tier);

-- Fact table captures risk profile at time of claim
CREATE TABLE fact_claims (
    claim_id VARCHAR(50),
    service_date_key INT,
    patient_key INT,              -- Stable patient dimension
    risk_profile_key INT,         -- Mini-dimension (snapshot)
    provider_key INT,
    paid_amount DECIMAL(10,2)
);

-- Claim load: Determine risk profile band
INSERT INTO fact_claims
SELECT
    claim_id,
    service_date_key,
    p.patient_key,

    -- Lookup risk profile based on current patient metrics
    (SELECT risk_profile_key FROM dim_patient_risk_profile
     WHERE risk_score_band = CASE
         WHEN patient_current_risk_score < 1.0 THEN 'Low'
         WHEN patient_current_risk_score < 2.0 THEN 'Medium'
         ELSE 'High'
     END
     AND chronic_condition_count = CASE
         WHEN patient_condition_count = 0 THEN '0'
         WHEN patient_condition_count <= 2 THEN '1-2'
         WHEN patient_condition_count <= 5 THEN '3-5'
         ELSE '6+'
     END
     AND age_band = CASE
         WHEN patient_age < 18 THEN '0-17'
         WHEN patient_age < 65 THEN '18-64'
         WHEN patient_age < 75 THEN '65-74'
         ELSE '75+'
     END
     AND cost_tier = patient_cost_tier
    ) as risk_profile_key,

    provider_key,
    paid_amount
FROM staging_claims s
JOIN dim_patient p ON s.patient_id = p.patient_id;

-- Query: Cost by risk profile
SELECT
    rp.risk_score_band,
    rp.chronic_condition_count,
    COUNT(DISTINCT f.claim_id) as claim_count,
    SUM(f.paid_amount) as total_cost,
    SUM(f.paid_amount) / COUNT(DISTINCT f.claim_id) as cost_per_claim
FROM fact_claims f
JOIN dim_patient_risk_profile rp ON f.risk_profile_key = rp.risk_profile_key
WHERE f.service_date_key BETWEEN 20240101 AND 20241231
GROUP BY rp.risk_score_band, rp.chronic_condition_count
ORDER BY total_cost DESC;

-- Interview talking point:
-- "At Optum, we used mini-dimensions for patient risk profiles which
-- changed monthly based on HCC scores. Instead of creating 120M SCD Type 2
-- rows, we maintained a 256-row mini-dimension with risk bands. This
-- reduced storage by 99.9% while still capturing patient risk state at
-- time of each claim."
```

**Key Design Principles:**

1. **Use bands/ranges** instead of exact values (reduces cardinality)
2. **Pre-populate** all possible combinations
3. **Small cardinality** (typically < 1000 rows)
4. **Snapshot at fact time** - fact records the mini-dimension state when event occurred

---

**Q19: Aggregate fact tables - when and how?**

**Answer:**

**Aggregate fact tables** store pre-computed summaries at coarser grain for query performance.

**Trade-off:** Storage vs Query Speed

**When to Use:**
- Queries always aggregate to same level (daily, monthly, etc.)
- Atomic fact table too large (billions of rows)
- Dashboard/report performance requirements

**Example:**

**Atomic Fact (Transaction Level):**
```sql
CREATE TABLE fact_sales (
    sale_id BIGINT PRIMARY KEY,
    sale_datetime TIMESTAMP,
    product_key INT,
    customer_key INT,
    store_key INT,
    quantity INT,
    unit_price DECIMAL(10,2),
    sale_amount DECIMAL(10,2)
);
-- 1 billion rows (one per transaction)

-- Query: Monthly sales by product
SELECT
    DATE_TRUNC('month', sale_datetime) as month,
    product_key,
    SUM(sale_amount) as monthly_sales
FROM fact_sales  -- Scans 1B rows! ❌ Slow
WHERE sale_datetime >= '2024-01-01'
GROUP BY DATE_TRUNC('month', sale_datetime), product_key;
-- Query time: 45 seconds
```

**Aggregate Fact (Daily Level):**
```sql
CREATE TABLE fact_sales_daily (
    date_key INT,
    product_key INT,
    customer_key INT,
    store_key INT,
    -- Aggregated measures
    transaction_count INT,
    total_quantity INT,
    total_sales_amount DECIMAL(10,2),
    avg_sale_amount DECIMAL(10,2),
    min_sale_amount DECIMAL(10,2),
    max_sale_amount DECIMAL(10,2),
    PRIMARY KEY (date_key, product_key, customer_key, store_key)
);
-- 10 million rows (1000× fewer)

-- Same query on aggregate
SELECT
    d.year_month,
    f.product_key,
    SUM(f.total_sales_amount) as monthly_sales
FROM fact_sales_daily f
JOIN dim_date d ON f.date_key = d.date_key
WHERE d.year = 2024
GROUP BY d.year_month, f.product_key;
-- Query time: 2 seconds ✅
```

**Aggregate Creation:**
```sql
-- Incremental daily aggregation (run nightly)
INSERT INTO fact_sales_daily
SELECT
    TO_CHAR(sale_datetime, 'YYYYMMDD')::INT as date_key,
    product_key,
    customer_key,
    store_key,
    COUNT(*) as transaction_count,
    SUM(quantity) as total_quantity,
    SUM(sale_amount) as total_sales_amount,
    AVG(sale_amount) as avg_sale_amount,
    MIN(sale_amount) as min_sale_amount,
    MAX(sale_amount) as max_sale_amount
FROM fact_sales
WHERE sale_datetime >= CURRENT_DATE - 1  -- Yesterday only
  AND sale_datetime < CURRENT_DATE
GROUP BY
    TO_CHAR(sale_datetime, 'YYYYMMDD')::INT,
    product_key,
    customer_key,
    store_key;
```

**Optum Healthcare Example:**

**Atomic Fact:**
```sql
CREATE TABLE fact_claims (
    claim_id VARCHAR(50),
    claim_line_number INT,
    service_date_key INT,
    received_date_key INT,
    patient_key INT,
    provider_key INT,
    diagnosis_key INT,
    procedure_key INT,
    billed_amount DECIMAL(10,2),
    allowed_amount DECIMAL(10,2),
    paid_amount DECIMAL(10,2),
    PRIMARY KEY (claim_id, claim_line_number)
);
-- 1.2 billion claim lines

-- Dashboard query (slow on atomic)
SELECT
    d.month_name,
    prov.provider_specialty,
    SUM(paid_amount) as total_paid
FROM fact_claims f  -- Scans 1.2B rows ❌
JOIN dim_date d ON f.service_date_key = d.date_key
JOIN dim_provider prov ON f.provider_key = prov.provider_key
WHERE d.year = 2024
GROUP BY d.month_name, prov.provider_specialty;
-- Query time: 90 seconds ❌
```

**Monthly Aggregate:**
```sql
CREATE TABLE fact_claims_monthly (
    month_key INT,              -- YYYYMM
    provider_key INT,
    provider_specialty VARCHAR(100),  -- Denormalized for performance
    diagnosis_category VARCHAR(100),
    -- Aggregated measures
    claim_count INT,
    claim_line_count INT,
    unique_patient_count INT,
    total_billed DECIMAL(12,2),
    total_paid DECIMAL(12,2),
    avg_paid_per_claim DECIMAL(10,2),
    PRIMARY KEY (month_key, provider_key, diagnosis_category)
);
-- 5 million rows (200× smaller)

-- Monthly aggregation job
INSERT INTO fact_claims_monthly
SELECT
    d.year_month as month_key,
    f.provider_key,
    prov.provider_specialty,
    diag.diagnosis_category,
    COUNT(DISTINCT f.claim_id) as claim_count,
    COUNT(*) as claim_line_count,
    COUNT(DISTINCT f.patient_key) as unique_patient_count,
    SUM(f.billed_amount) as total_billed,
    SUM(f.paid_amount) as total_paid,
    AVG(f.paid_amount) as avg_paid_per_claim
FROM fact_claims f
JOIN dim_date d ON f.service_date_key = d.date_key
JOIN dim_provider prov ON f.provider_key = prov.provider_key
JOIN dim_diagnosis diag ON f.diagnosis_key = diag.diagnosis_key
WHERE d.year_month = 202405  -- Current month
GROUP BY
    d.year_month,
    f.provider_key,
    prov.provider_specialty,
    diag.diagnosis_category;

-- Dashboard query (fast on aggregate)
SELECT
    month_key,
    provider_specialty,
    SUM(total_paid) as total_paid,
    SUM(unique_patient_count) as patient_count
FROM fact_claims_monthly
WHERE month_key BETWEEN 202401 AND 202412
GROUP BY month_key, provider_specialty;
-- Query time: 1 second ✅

-- Interview talking point:
-- "At Optum, we maintained monthly aggregate fact tables for executive
-- dashboards. The atomic fact table had 1.2B rows, but the monthly
-- aggregate had only 5M rows, improving dashboard load time from 90
-- seconds to under 1 second. We ran the aggregation jobs nightly in
-- Airflow, processing only the previous day's claims."
```

**Best Practices:**

1. **Keep atomic fact** - Don't delete, aggregates are supplementary
2. **Document grain clearly** - "One row per product per day"
3. **Include counts** - Distinguish additive (SUM) from non-additive (AVG)
4. **Denormalize when helpful** - Copy dimension attributes into aggregate for fewer joins
5. **Automate refresh** - Schedule incremental aggregation jobs

---

**Q20: Factless fact tables - use cases and examples?**

**Answer:**

**Factless fact tables** contain no numeric measures - only foreign keys to dimensions.

**Purpose:** Track events or relationships, not measurements.

**Use Case 1: Event Tracking**

**Student Attendance:**
```sql
CREATE TABLE fact_attendance (
    student_key INT,
    course_key INT,
    date_key INT,
    instructor_key INT,
    -- No measures! Just the event of attendance
    PRIMARY KEY (student_key, course_key, date_key)
);

-- Query: How many students attended each course?
SELECT
    c.course_name,
    COUNT(DISTINCT f.student_key) as student_count,
    COUNT(*) as total_attendance_days
FROM fact_attendance f
JOIN dim_course c ON f.course_key = c.course_key
JOIN dim_date d ON f.date_key = d.date_key
WHERE d.year_month = 202405
GROUP BY c.course_name;

-- Query: Which students missed class?
SELECT DISTINCT
    s.student_id,
    s.student_name,
    c.course_name,
    d.full_date
FROM dim_student s
CROSS JOIN dim_course c
CROSS JOIN dim_date d
WHERE d.is_business_day = TRUE
  AND d.year_month = 202405
  AND NOT EXISTS (
      SELECT 1 FROM fact_attendance f
      WHERE f.student_key = s.student_key
        AND f.course_key = c.course_key
        AND f.date_key = d.date_key
  );
-- ✅ Tracks what DIDN'T happen!
```

**Use Case 2: Coverage/Eligibility**

**Insurance Coverage:**
```sql
CREATE TABLE fact_insurance_coverage (
    patient_key INT,
    insurance_plan_key INT,
    date_key INT,
    -- No measures - just tracking coverage eligibility
    PRIMARY KEY (patient_key, insurance_plan_key, date_key)
);

-- Query: How many members were covered each day?
SELECT
    d.full_date,
    i.plan_name,
    COUNT(DISTINCT f.patient_key) as covered_members
FROM fact_insurance_coverage f
JOIN dim_date d ON f.date_key = d.date_key
JOIN dim_insurance_plan i ON f.insurance_plan_key = i.insurance_plan_key
WHERE d.year_month = 202405
GROUP BY d.full_date, i.plan_name;

-- Query: Coverage gaps (patients without coverage on specific days)
SELECT
    p.patient_id,
    d.full_date
FROM dim_patient p
CROSS JOIN dim_date d
WHERE d.year = 2024
  AND NOT EXISTS (
      SELECT 1 FROM fact_insurance_coverage f
      WHERE f.patient_key = p.patient_key
        AND f.date_key = d.date_key
  );
```

**Optum Healthcare Example:**

```sql
-- Patient eligibility snapshot (factless)
CREATE TABLE fact_member_eligibility (
    patient_key INT,
    insurance_plan_key INT,
    provider_network_key INT,
    date_key INT,
    -- Factless! Just tracking who was eligible when
    PRIMARY KEY (patient_key, date_key)
);

-- Load: Daily snapshot of all eligible members
INSERT INTO fact_member_eligibility
SELECT
    p.patient_key,
    p.insurance_plan_key,
    p.provider_network_key,
    20240503 as date_key  -- Today
FROM dim_patient p
WHERE p.is_active = TRUE
  AND p.coverage_start_date <= '2024-05-03'
  AND (p.coverage_end_date IS NULL OR p.coverage_end_date >= '2024-05-03');

-- Query: Member-months (enrollment metrics)
SELECT
    i.plan_name,
    d.year_month,
    COUNT(DISTINCT f.patient_key) as member_count,
    COUNT(*) as member_days,
    COUNT(*) / COUNT(DISTINCT d.date_key) as avg_members_per_day
FROM fact_member_eligibility f
JOIN dim_insurance_plan i ON f.insurance_plan_key = i.insurance_plan_key
JOIN dim_date d ON f.date_key = d.date_key
WHERE d.year = 2024
GROUP BY i.plan_name, d.year_month;

-- Query: Identify members who became ineligible
SELECT
    p.patient_id,
    p.patient_name,
    MAX(d.full_date) as last_eligible_date
FROM fact_member_eligibility f
JOIN dim_patient p ON f.patient_key = p.patient_key
JOIN dim_date d ON f.date_key = d.date_key
WHERE d.full_date < CURRENT_DATE - 7  -- Not eligible for 7 days
  AND NOT EXISTS (
      SELECT 1 FROM fact_member_eligibility f2
      JOIN dim_date d2 ON f2.date_key = d2.date_key
      WHERE f2.patient_key = f.patient_key
        AND d2.full_date >= CURRENT_DATE - 7
  )
GROUP BY p.patient_id, p.patient_name;

-- Interview talking point:
-- "At Optum, we maintained a factless eligibility fact table with daily
-- snapshots of all members' enrollment status. This had no numeric measures
-- but enabled critical analytics like member-months calculation for Medicare
-- Advantage, tracking enrollment trends, and identifying disenrollment
-- patterns. The table had 300 million rows (10M members × 30 days) but
-- queries were fast due to proper indexing on date_key and patient_key."
```

**Use Case 3: Promotions/Campaigns**

```sql
-- Which customers were exposed to which promotions
CREATE TABLE fact_promotion_exposure (
    customer_key INT,
    promotion_key INT,
    channel_key INT,
    date_key INT,
    -- No measures! Just tracking who saw what
    PRIMARY KEY (customer_key, promotion_key, date_key)
);

-- Query: Promotion reach
SELECT
    p.promotion_name,
    COUNT(DISTINCT f.customer_key) as customers_reached
FROM fact_promotion_exposure f
JOIN dim_promotion p ON f.promotion_key = p.promotion_key
GROUP BY p.promotion_name;

-- Join with sales fact to analyze conversion
SELECT
    p.promotion_name,
    COUNT(DISTINCT e.customer_key) as exposed_customers,
    COUNT(DISTINCT s.customer_key) as converted_customers,
    ROUND(100.0 * COUNT(DISTINCT s.customer_key) / COUNT(DISTINCT e.customer_key), 2) as conversion_rate
FROM fact_promotion_exposure e
JOIN dim_promotion p ON e.promotion_key = p.promotion_key
LEFT JOIN fact_sales s ON e.customer_key = s.customer_key
                       AND s.sale_date_key BETWEEN e.date_key AND e.date_key + 30
WHERE e.date_key BETWEEN 20240101 AND 20240131
GROUP BY p.promotion_name;
```

**Key Characteristics:**
- No numeric measures (no SUM, AVG)
- Primary key is combination of all dimension keys
- Queries use COUNT(*) or COUNT(DISTINCT)
- Useful for tracking events, coverage, relationships
- Can identify what DIDN'T happen (absences, gaps)

---

### Normalization (15 Questions)

**Q21: What is database normalization? Why do it?**

**Answer:**

**Normalization:** Process of organizing data to reduce redundancy and improve integrity.

**Goals:**
1. Eliminate data redundancy
2. Ensure data dependencies make sense
3. Protect data integrity

**Forms:**
- 1NF: Atomic values
- 2NF: No partial dependencies
- 3NF: No transitive dependencies
- BCNF: Stronger 3NF
- 4NF, 5NF: Rare, specialized

**OLTP vs OLAP:**
| OLTP (Apps) | OLAP (Analytics) |
|-------------|------------------|
| Normalize (3NF) | Denormalize (Star) |
| Avoid redundancy | Redundancy OK |
| Update performance | Query performance |

---

**Q22: Explain 1NF, 2NF, 3NF with examples**

**Answer:**

**0NF (Unnormalized):**
```sql
-- Repeating groups, comma-separated values
customer_id | name  | phone_numbers
1           | Alice | 555-1111, 555-2222, 555-3333  ❌ Not atomic!
```

---

**1NF: Atomic Values**

Rules:
- Each column contains atomic (indivisible) values
- No repeating groups
- Each row is unique

```sql
-- Fixed:
customer_id | name  | phone_number
1           | Alice | 555-1111
1           | Alice | 555-2222      ✅ Atomic!
1           | Alice | 555-3333
```

---

**2NF: No Partial Dependencies**

Rules:
- Must be in 1NF
- No partial dependencies on composite primary key
- All non-key columns fully dependent on entire primary key

**Violation Example:**
```sql
-- Composite key: (order_id, product_id)
order_id | product_id | quantity | product_name | product_price
1        | 101        | 5        | Widget       | 10.00
1        | 102        | 3        | Gadget       | 20.00

-- Problem: product_name depends only on product_id, not full key!
-- If product name changes, must update multiple rows
```

**Fixed (2NF):**
```sql
-- Table 1: Order items
order_id | product_id | quantity
1        | 101        | 5
1        | 102        | 3

-- Table 2: Products (product info separate)
product_id | product_name | product_price
101        | Widget       | 10.00
102        | Gadget       | 20.00
```

---

**3NF: No Transitive Dependencies**

Rules:
- Must be in 2NF
- No transitive dependencies
- Non-key columns depend only on primary key, not on other non-key columns

**Violation Example:**
```sql
-- Primary key: employee_id
employee_id | name  | department_id | department_name | department_location
1           | Alice | 10            | Sales           | Building A

-- Problem: department_name depends on department_id (transitive!)
-- employee_id → department_id → department_name
```

**Fixed (3NF):**
```sql
-- Table 1: Employees
employee_id | name  | department_id
1           | Alice | 10

-- Table 2: Departments
department_id | department_name | department_location
10            | Sales           | Building A
```

**Summary:**
- **1NF:** Atomic values, no repeating groups
- **2NF:** No partial dependencies on composite key
- **3NF:** No transitive dependencies (A → B → C)

---

**Q23: What is BCNF (Boyce-Codd Normal Form)?**

**Answer:**

**BCNF** is a stricter version of 3NF that eliminates certain anomalies that 3NF allows.

**Definition:**
A table is in BCNF if for every functional dependency X → Y, X must be a superkey (candidate key).

**Simpler:** Every determinant must be a candidate key.

**3NF vs BCNF:**

**Example that's in 3NF but NOT BCNF:**
```sql
-- Course scheduling
-- Composite key: (student_id, course_id)
student_id | course_id | instructor | instructor_office
1          | CS101     | Prof A     | Room 201
1          | CS102     | Prof B     | Room 202
2          | CS101     | Prof A     | Room 201

-- Functional dependencies:
-- (student_id, course_id) → instructor        ✅ OK
-- (student_id, course_id) → instructor_office ✅ OK
-- instructor → instructor_office              ❌ PROBLEM!

-- instructor is a determinant but NOT a candidate key!
-- This violates BCNF
```

**Problem:** If Prof A changes office, must update multiple rows.

**Fixed (BCNF):**
```sql
-- Table 1: Student enrollments
student_id | course_id | instructor
1          | CS101     | Prof A
1          | CS102     | Prof B
2          | CS101     | Prof A

-- Table 2: Instructor offices
instructor | instructor_office
Prof A     | Room 201
Prof B     | Room 202

-- Now all determinants are candidate keys ✅
```

**When BCNF Matters:**
- Rare in practice (most 3NF designs are already BCNF)
- Mostly academic/theoretical importance
- Focus on 3NF for interviews

**Optum Example:**

```sql
-- Claims adjudication (violates BCNF)
claim_id | procedure_code | adjudicator | adjudicator_region
C001     | 99213          | ADJ_123     | Northeast
C002     | 99214          | ADJ_123     | Northeast
C003     | 99213          | ADJ_456     | Midwest

-- Problem: adjudicator → adjudicator_region
-- adjudicator is determinant but not candidate key

-- Fixed (BCNF):
-- Table 1
claim_id | procedure_code | adjudicator
C001     | 99213          | ADJ_123
C002     | 99214          | ADJ_123

-- Table 2
adjudicator | adjudicator_region
ADJ_123     | Northeast
ADJ_456     | Midwest

-- Interview talking point:
-- "At Optum, we rarely encountered BCNF violations in practice.
-- Most transactional databases were designed to 3NF which naturally
-- satisfied BCNF. For data warehouses, we intentionally denormalized
-- for query performance, so normalization beyond 3NF wasn't applicable."
```

---

**Q24: Denormalization - when and why?**

**Answer:**

**Denormalization** is intentionally introducing redundancy into a normalized database to improve query performance.

**When to Denormalize:**

**1. Data Warehouses / OLAP Systems**
```sql
-- Normalized (OLTP):
SELECT
    c.customer_name,
    ci.city_name,
    st.state_name,
    r.region_name,
    SUM(o.amount) as total_sales
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN cities ci ON c.city_id = ci.city_id
JOIN states st ON ci.state_id = st.state_id
JOIN regions r ON st.region_id = r.region_id
GROUP BY ...;
-- 4 joins! Slow for analytics

-- Denormalized (OLAP):
CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY,
    customer_id VARCHAR(50),
    customer_name VARCHAR(200),
    city_name VARCHAR(100),         -- Denormalized
    state_name VARCHAR(100),        -- Denormalized
    region_name VARCHAR(100),       -- Denormalized
    full_address VARCHAR(500)       -- Denormalized
);

SELECT
    c.customer_name,
    c.region_name,
    SUM(f.amount) as total_sales
FROM fact_sales f
JOIN dim_customer c ON f.customer_key = c.customer_key
GROUP BY c.customer_name, c.region_name;
-- 1 join! Fast ✅
```

**2. Frequently Accessed Aggregates**
```sql
-- Instead of calculating every time
SELECT customer_id, COUNT(*) as order_count
FROM orders
GROUP BY customer_id;

-- Denormalize: Store in customer table
ALTER TABLE customers ADD COLUMN total_orders INT;

UPDATE customers c
SET total_orders = (SELECT COUNT(*) FROM orders WHERE customer_id = c.customer_id);

-- Query: Just read the column ✅
SELECT customer_id, total_orders FROM customers;
```

**3. Avoid Expensive Joins**
```sql
-- Denormalize product name into order_items
CREATE TABLE order_items (
    order_id BIGINT,
    product_id BIGINT,
    product_name VARCHAR(200),  -- Denormalized from products table
    quantity INT,
    unit_price DECIMAL(10,2)
);

-- Benefit: Display order items without joining products table
SELECT order_id, product_name, quantity
FROM order_items
WHERE order_id = 12345;
-- No join needed! ✅
```

**Denormalization Strategies:**

**1. Add Redundant Columns**
```sql
-- Add customer_name to orders (avoid JOIN)
orders (
    order_id,
    customer_id,
    customer_name  -- Redundant but fast
)
```

**2. Add Derived/Calculated Columns**
```sql
-- Store pre-calculated totals
orders (
    order_id,
    subtotal,
    tax_amount,
    total_amount  -- Derived: subtotal + tax
)
```

**3. Duplicate Tables (Reporting Replicas)**
```sql
-- Keep normalized OLTP database
-- Create denormalized reporting database (replicated)
```

**Optum Healthcare Example:**

```sql
-- Normalized claims structure (OLTP)
claim_lines (
    claim_id,
    line_number,
    procedure_code
) → procedures (procedure_code, procedure_name, category)

-- Problem: Dashboard showing claim details needs to JOIN
-- 1.2 billion claim lines × JOIN = slow!

-- Denormalized (OLAP/Data Warehouse)
CREATE TABLE fact_claims (
    claim_id VARCHAR(50),
    claim_line_number INT,
    procedure_code VARCHAR(10),
    procedure_name VARCHAR(500),        -- Denormalized!
    procedure_category VARCHAR(100),    -- Denormalized!
    diagnosis_code VARCHAR(10),
    diagnosis_name VARCHAR(500),        -- Denormalized!
    patient_key INT,
    provider_key INT,
    paid_amount DECIMAL(10,2)
);

-- Dashboard query (no JOIN needed)
SELECT
    procedure_category,
    COUNT(*) as claim_count,
    SUM(paid_amount) as total_cost
FROM fact_claims
WHERE service_date_key BETWEEN 20240101 AND 20241231
GROUP BY procedure_category;
-- Query time: 3 seconds (vs 45 seconds with JOINs) ✅

-- Interview talking point:
-- "At Optum, we maintained normalized OLTP databases for claims
-- processing to ensure data integrity. But for the data warehouse,
-- we heavily denormalized dimension tables - storing procedure names,
-- diagnosis descriptions, and provider details directly in the fact
-- table where it made sense. This reduced dashboard query times from
-- 45+ seconds to under 5 seconds for 1.2 billion row fact tables."
```

**Trade-offs:**

| Aspect | Normalization | Denormalization |
|--------|---------------|-----------------|
| **Storage** | Less (no redundancy) | More (redundant data) |
| **Updates** | Faster (single location) | Slower (multiple locations) |
| **Reads** | Slower (many JOINs) | Faster (fewer JOINs) |
| **Data integrity** | Easier to maintain | Harder (can go stale) |
| **Complexity** | More tables | Fewer tables |
| **Use case** | OLTP (transactions) | OLAP (analytics) |

**Best Practices:**
1. **Denormalize judiciously** - Only where performance matters
2. **Document** - Make denormalization intentional and documented
3. **ETL sync** - Keep denormalized data synchronized via ETL
4. **Read-heavy workloads** - Best for reporting/analytics
5. **Separate OLTP/OLAP** - Keep transactional DB normalized, denormalize warehouse

---

**Q25: Normalization vs Denormalization - detailed trade-offs?**

**Answer:**

**Normalization vs Denormalization Decision Matrix:**

**When to Normalize (OLTP Systems):**

✅ **Transaction Processing:**
```sql
-- Banking system (must be normalized)
-- Customer updates address - change ONE place
UPDATE customers
SET address = '123 New St'
WHERE customer_id = 'C001';

-- If denormalized, would need to update:
-- - customers table
-- - orders table (customer_name, customer_address)
-- - invoices table
-- - support_tickets table
-- ❌ Risk of inconsistency!
```

✅ **Frequent Updates:**
```sql
-- Product prices change daily
-- Normalized: Update products table once
UPDATE products SET price = 29.99 WHERE product_id = 'P001';

-- Denormalized: Update millions of order_items rows ❌
UPDATE order_items SET product_price = 29.99 WHERE product_id = 'P001';
-- Slow and risky!
```

✅ **Data Integrity Critical:**
```sql
-- Healthcare patient records
-- Must maintain referential integrity
patients → visits → diagnoses → treatments
-- Normalized structure ensures no orphaned records
```

**When to Denormalize (OLAP Systems):**

✅ **Complex Reporting Queries:**
```sql
-- Sales report across 5+ tables
-- Normalized: 5 JOINs (slow)
SELECT ...
FROM sales s
JOIN customers c ON s.customer_id = c.customer_id
JOIN cities ci ON c.city_id = ci.city_id
JOIN states st ON ci.state_id = st.state_id
JOIN regions r ON st.region_id = r.region_id
JOIN products p ON s.product_id = p.product_id;

-- Denormalized: Star schema (fast)
SELECT ...
FROM fact_sales f
JOIN dim_customer c ON f.customer_key = c.customer_key
-- All geography flattened into dim_customer!
```

✅ **Read-Heavy Workloads:**
```sql
-- Dashboard queries (run 1000s of times/day)
-- Optimize for reads, not writes
-- Denormalized structure acceptable
```

✅ **Historical Snapshots:**
```sql
-- Invoice: Customer name at time of purchase
CREATE TABLE invoices (
    invoice_id BIGINT,
    customer_id BIGINT,
    customer_name VARCHAR(200),  -- Denormalized snapshot
    customer_address VARCHAR(500), -- As it was then
    invoice_date DATE,
    amount DECIMAL(10,2)
);

-- Benefit: Even if customer changes name/address,
-- invoice shows historical values ✅
```

**Hybrid Approach (Best Practice):**

```sql
-- OLTP Database (Normalized)
CREATE DATABASE sales_oltp;

-- Normalized tables
customers (customer_id, name, email)
addresses (customer_id, address_id, street, city, state, zip)
orders (order_id, customer_id, order_date)
order_items (order_id, product_id, quantity, price)
products (product_id, name, category, price)

-- OLAP Data Warehouse (Denormalized)
CREATE DATABASE sales_dwh;

-- Denormalized star schema
fact_sales (
    sale_date_key,
    customer_key,
    product_key,
    quantity,
    amount
)

dim_customer (
    customer_key,
    customer_id,
    name,
    email,
    full_address,  -- Denormalized from addresses table
    city,
    state,
    zip
)

dim_product (
    product_key,
    product_id,
    product_name,
    category,
    subcategory,
    current_price
)
```

**Optum Healthcare Example:**

```sql
-- OLTP: Claims Processing System (Normalized)
CREATE TABLE claims (
    claim_id VARCHAR(50) PRIMARY KEY,
    patient_id VARCHAR(50),
    provider_id VARCHAR(50),
    received_date DATE,
    status VARCHAR(20)
);

CREATE TABLE claim_lines (
    claim_id VARCHAR(50),
    line_number INT,
    procedure_code VARCHAR(10),
    diagnosis_code VARCHAR(10),
    billed_amount DECIMAL(10,2),
    PRIMARY KEY (claim_id, line_number)
);

CREATE TABLE procedures (
    procedure_code VARCHAR(10) PRIMARY KEY,
    procedure_name VARCHAR(500),
    procedure_category VARCHAR(100)
);

CREATE TABLE diagnoses (
    diagnosis_code VARCHAR(10) PRIMARY KEY,
    diagnosis_name VARCHAR(500),
    diagnosis_category VARCHAR(100)
);

-- Why normalized here:
-- - Procedure/diagnosis codes updated centrally
-- - Millions of claim lines reference same codes
-- - Data integrity critical (healthcare compliance)

-- OLAP: Claims Analytics Warehouse (Denormalized)
CREATE TABLE fact_claims (
    claim_id VARCHAR(50),
    claim_line_number INT,
    service_date_key INT,
    patient_key INT,
    provider_key INT,

    -- Denormalized procedure attributes
    procedure_code VARCHAR(10),
    procedure_name VARCHAR(500),        -- Denormalized!
    procedure_category VARCHAR(100),    -- Denormalized!

    -- Denormalized diagnosis attributes
    diagnosis_code VARCHAR(10),
    diagnosis_name VARCHAR(500),        -- Denormalized!
    diagnosis_category VARCHAR(100),    -- Denormalized!

    -- Measures
    billed_amount DECIMAL(10,2),
    paid_amount DECIMAL(10,2)
);

-- Why denormalized here:
-- - Read-only analytics (no updates)
-- - Queries need procedure/diagnosis names (avoid JOINs)
-- - Dashboard performance critical (1000s of queries/day)
-- - Historical snapshot (name at time of claim)

-- Performance comparison:
-- Normalized: Dashboard query = 45 seconds (5 JOINs)
-- Denormalized: Dashboard query = 3 seconds (0-1 JOINs) ✅

-- Interview talking point:
-- "At Optum, we followed a hybrid approach - maintaining normalized
-- OLTP databases for claims processing with strict referential integrity,
-- then ETL'd into denormalized data warehouse for analytics. The OLTP
-- system processed 500GB/day of claims with low latency, while the
-- denormalized warehouse powered executive dashboards with sub-3-second
-- response times on billion-row fact tables."
```

**Decision Framework:**

| Question | Normalize | Denormalize |
|----------|-----------|-------------|
| Frequent updates to data? | ✅ Yes | ❌ No |
| Read-heavy workload? | ❌ No | ✅ Yes |
| Data integrity critical? | ✅ Yes | ⚠️ Managed via ETL |
| Complex reporting queries? | ❌ No | ✅ Yes |
| Storage cost sensitive? | ✅ Yes | ❌ No |
| OLTP or OLAP? | OLTP | OLAP |

**Modern Trend: Separate OLTP and OLAP**
- **OLTP:** Fully normalized (3NF/BCNF)
- **ETL Pipeline:** Sync data nightly/real-time
- **OLAP:** Denormalized (star/snowflake schema)

---

**Q26: Foreign keys and referential integrity - explain and best practices?**

**Answer:**

**Foreign Key (FK)** is a column that references the primary key of another table, enforcing referential integrity.

**Referential Integrity:** Ensures relationships between tables remain consistent - no orphaned records.

**Example:**
```sql
-- Parent table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(200),
    email VARCHAR(200)
);

-- Child table with foreign key
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    amount DECIMAL(10,2),

    -- Foreign key constraint
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- ✅ Valid INSERT (customer exists)
INSERT INTO customers VALUES (1, 'Alice', 'alice@example.com');
INSERT INTO orders VALUES (100, 1, '2024-05-01', 50.00);  -- OK

-- ❌ Invalid INSERT (customer doesn't exist)
INSERT INTO orders VALUES (101, 999, '2024-05-02', 75.00);
-- ERROR: foreign key constraint violated
-- customer_id 999 doesn't exist in customers table
```

**Referential Integrity Actions:**

**1. ON DELETE RESTRICT (Default)**
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE RESTRICT  -- Prevent deletion if orders exist
);

-- Try to delete customer with orders
DELETE FROM customers WHERE customer_id = 1;
-- ❌ ERROR: Cannot delete - foreign key constraint
-- Must delete orders first
```

**2. ON DELETE CASCADE**
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE CASCADE  -- Delete orders when customer deleted
);

DELETE FROM customers WHERE customer_id = 1;
-- ✅ SUCCESS: Customer deleted AND all their orders automatically deleted
```

**3. ON DELETE SET NULL**
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NULL,  -- Must be nullable
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE SET NULL  -- Set to NULL when customer deleted
);

DELETE FROM customers WHERE customer_id = 1;
-- ✅ SUCCESS: Customer deleted, orders.customer_id set to NULL
```

**4. ON UPDATE CASCADE**
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON UPDATE CASCADE  -- Update FK when PK changes
);

-- Change customer PK
UPDATE customers SET customer_id = 999 WHERE customer_id = 1;
-- ✅ All orders.customer_id automatically updated: 1 → 999
```

**When to Use Each:**

| Action | Use Case | Example |
|--------|----------|---------|
| **RESTRICT** | Prevent accidental deletion | Don't delete customers with active orders |
| **CASCADE** | Parent-child lifecycle | Delete user → delete all sessions |
| **SET NULL** | Optional relationships | Delete supplier → products.supplier_id = NULL |
| **SET DEFAULT** | Fallback value | Delete category → product.category_id = 'Uncategorized' |

**Optum Healthcare Example:**

```sql
-- Patient and claims relationship
CREATE TABLE patients (
    patient_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(200),
    date_of_birth DATE,
    status VARCHAR(20)  -- Active, Inactive, Deceased
);

CREATE TABLE claims (
    claim_id VARCHAR(50) PRIMARY KEY,
    patient_id VARCHAR(50),
    claim_date DATE,
    amount DECIMAL(10,2),

    -- RESTRICT: Never delete patient if claims exist (compliance!)
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);

-- Try to delete patient with claims
DELETE FROM patients WHERE patient_id = 'P001';
-- ❌ ERROR: Foreign key constraint
-- Healthcare compliance: Cannot delete patient records with claims!

-- Update patient ID (rare, but supported)
UPDATE patients SET patient_id = 'P001_NEW' WHERE patient_id = 'P001';
-- ✅ All claims.patient_id automatically updated to 'P001_NEW'

-- Proper way to "delete" patient (compliance)
UPDATE patients SET status = 'Deceased' WHERE patient_id = 'P001';
-- Patient marked inactive but claims preserved ✅

-- Interview talking point:
-- "At Optum, we used RESTRICT foreign keys between patients and claims
-- to prevent accidental deletion of patient records that had associated
-- claims. This was critical for HIPAA compliance and audit trails.
-- Patient records could only be soft-deleted (status = 'Inactive')
-- while preserving all historical claims data."
```

**Performance Considerations:**

**Foreign Keys Impact Performance:**
```sql
-- FK check on every INSERT/UPDATE/DELETE

-- INSERT into child table: Check if parent exists
INSERT INTO orders (customer_id) VALUES (1);
-- Database checks: SELECT customer_id FROM customers WHERE customer_id = 1

-- DELETE from parent: Check if any children exist (if RESTRICT)
DELETE FROM customers WHERE customer_id = 1;
-- Database checks: SELECT COUNT(*) FROM orders WHERE customer_id = 1

-- Can slow down bulk loads!
```

**Bulk Load Strategy:**
```sql
-- Disable FK constraints during bulk load
ALTER TABLE orders DISABLE TRIGGER ALL;  -- PostgreSQL
-- Or: SET FOREIGN_KEY_CHECKS=0;          -- MySQL

-- Bulk load millions of rows
COPY orders FROM 'orders.csv';

-- Re-enable and validate
ALTER TABLE orders ENABLE TRIGGER ALL;
-- Database validates all FKs at once
```

**Data Warehouse Consideration:**
```sql
-- OLTP: Use foreign keys (enforce integrity)
CREATE TABLE orders (
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- OLAP/Data Warehouse: Often NO foreign keys
CREATE TABLE fact_sales (
    customer_key INT,  -- References dim_customer
    product_key INT    -- References dim_product
    -- NO FK constraints! (performance)
);

-- Why no FKs in data warehouse:
-- - Data already validated in OLTP
-- - ETL process ensures integrity
-- - FK checks slow down bulk loads
-- - Read-only analytics (no updates)

-- Interview talking point:
-- "At Optum, we enforced foreign keys in OLTP claims processing databases
-- to ensure referential integrity. But in the data warehouse, we omitted
-- FK constraints for performance - the ETL process already validated data,
-- and avoiding FK checks improved bulk load speed from 4 hours to 45 minutes
-- for nightly dimension updates."
```

**Best Practices:**

1. **OLTP:** Always use foreign keys for data integrity
2. **OLAP/DWH:** Consider omitting for performance (validate in ETL)
3. **Cascade carefully:** CASCADE can delete massive amounts of data
4. **Index foreign keys:** Create indexes on FK columns for join performance
5. **Document:** If omitting FKs, document why and how integrity is maintained

**Index Foreign Keys:**
```sql
-- Without index: FK check requires full table scan ❌
CREATE TABLE orders (
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- With index: FK check uses index ✅
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

-- JOIN performance
SELECT c.name, COUNT(o.order_id)
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.name;
-- Uses index on orders.customer_id ✅
```

---

**Q27: Composite keys vs Surrogate keys - when to use each?**

**Answer:**

**Composite Key:** Primary key made up of multiple columns.

**Surrogate Key:** Single-column, system-generated primary key (usually auto-increment integer).

**Composite Key Example:**
```sql
CREATE TABLE order_items (
    order_id BIGINT,
    product_id BIGINT,
    quantity INT,
    unit_price DECIMAL(10,2),

    PRIMARY KEY (order_id, product_id)  -- Composite key
);

-- Grain: One row per product per order
-- Natural composite key makes sense here
```

**Surrogate Key Example:**
```sql
CREATE TABLE order_items (
    order_item_id BIGINT PRIMARY KEY AUTO_INCREMENT,  -- Surrogate key
    order_id BIGINT,
    product_id BIGINT,
    quantity INT,
    unit_price DECIMAL(10,2),

    UNIQUE (order_id, product_id)  -- Business constraint
);

-- Simpler primary key, easier to reference
```

**When to Use Composite Keys:**

**1. Natural Grain is Composite**
```sql
-- Many-to-many relationship
CREATE TABLE student_courses (
    student_id BIGINT,
    course_id BIGINT,
    enrollment_date DATE,
    grade VARCHAR(2),

    PRIMARY KEY (student_id, course_id)  -- Natural composite
);

-- Makes sense: One enrollment per student per course
```

**2. Fact Tables (Data Warehouse)**
```sql
-- Daily sales fact
CREATE TABLE fact_daily_sales (
    date_key INT,
    product_key INT,
    store_key INT,
    sales_amount DECIMAL(10,2),

    PRIMARY KEY (date_key, product_key, store_key)  -- Composite grain
);

-- Grain: One row per product per store per day
```

**When to Use Surrogate Keys:**

**1. Long/Complex Natural Keys**
```sql
-- Natural key: Too long!
CREATE TABLE customers (
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    date_of_birth DATE,
    ssn VARCHAR(11),
    PRIMARY KEY (first_name, last_name, date_of_birth, ssn)  -- ❌ Bulky!
);

-- Surrogate key: Clean ✅
CREATE TABLE customers (
    customer_id BIGINT PRIMARY KEY AUTO_INCREMENT,  -- Surrogate
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    date_of_birth DATE,
    ssn VARCHAR(11),
    UNIQUE (ssn)  -- Natural key as constraint
);
```

**2. Dimension Tables (SCD Type 2)**
```sql
-- Without surrogate (can't track history)
CREATE TABLE dim_customer (
    customer_id VARCHAR(50) PRIMARY KEY,  -- Natural key
    name VARCHAR(200),
    address VARCHAR(500)
);

-- With surrogate (enables SCD Type 2)
CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY AUTO_INCREMENT,  -- Surrogate
    customer_id VARCHAR(50),  -- Natural key
    name VARCHAR(200),
    address VARCHAR(500),
    effective_date DATE,
    expiration_date DATE,
    is_current BOOLEAN
);

-- Multiple rows for same customer_id (history!)
customer_key | customer_id | address      | is_current
1001         | CUST_123    | Old Address  | FALSE
1002         | CUST_123    | New Address  | TRUE
```

**3. Foreign Key References Easier**
```sql
-- Composite FK (messy)
CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    customer_first_name VARCHAR(100),
    customer_last_name VARCHAR(100),
    customer_dob DATE,
    FOREIGN KEY (customer_first_name, customer_last_name, customer_dob)
        REFERENCES customers(first_name, last_name, date_of_birth)  -- ❌ Bulky!
);

-- Surrogate FK (clean)
CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    customer_id BIGINT,  -- Simple FK ✅
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

**4. Natural Key Changes**
```sql
-- Natural key: Email (can change!)
CREATE TABLE users (
    email VARCHAR(200) PRIMARY KEY,
    name VARCHAR(200)
);

-- If email changes, must update all child tables ❌

-- Surrogate key: Stable even if email changes ✅
CREATE TABLE users (
    user_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(200) UNIQUE,
    name VARCHAR(200)
);

UPDATE users SET email = 'newemail@example.com' WHERE user_id = 123;
-- Child tables unaffected (still reference user_id) ✅
```

**Optum Healthcare Example:**

```sql
-- Composite key: Claim lines (natural)
CREATE TABLE claim_lines (
    claim_id VARCHAR(50),
    line_number INT,
    procedure_code VARCHAR(10),
    billed_amount DECIMAL(10,2),

    PRIMARY KEY (claim_id, line_number)  -- Composite: Natural grain
);

-- Why composite here:
-- - Natural grain: One line per claim
-- - Stable (never changes)
-- - Short (2 columns only)

-- Surrogate key: Patient dimension (SCD Type 2)
CREATE TABLE dim_patient (
    patient_key BIGINT PRIMARY KEY AUTO_INCREMENT,  -- Surrogate
    patient_id VARCHAR(50),  -- Natural key (Medical Record Number)
    name VARCHAR(200),
    date_of_birth DATE,
    insurance_plan VARCHAR(100),
    -- SCD Type 2
    effective_date DATE,
    expiration_date DATE,
    is_current BOOLEAN
);

-- Multiple versions for same patient_id
patient_key | patient_id | insurance_plan     | is_current
1001        | P12345     | Medicare Advantage | FALSE
1002        | P12345     | Medicare Original  | TRUE

-- Facts reference patient_key (not patient_id)
CREATE TABLE fact_claims (
    claim_id VARCHAR(50),
    patient_key BIGINT,  -- References dim_patient.patient_key
    -- Correct historical version automatically!
);

-- Interview talking point:
-- "At Optum, we used composite keys for fact tables where the natural
-- grain was multi-column (like claim_id + line_number). But for dimension
-- tables, we always used surrogate keys to support SCD Type 2 and handle
-- patient ID changes. This allowed us to track full history - for example,
-- when a patient switched insurance plans, we created a new patient_key
-- while preserving the same patient_id."
```

**Performance Comparison:**

| Aspect | Composite Key | Surrogate Key |
|--------|---------------|---------------|
| **Storage** | Varies (depends on columns) | Compact (INT/BIGINT) |
| **Join performance** | Slower (multi-column) | Faster (single INT) |
| **Index size** | Larger | Smaller |
| **History tracking** | Difficult | Easy (SCD Type 2) |
| **Uniqueness** | Business enforced | System enforced |
| **Readability** | More semantic | Less semantic |

**Best Practices:**

1. **Fact tables:** Composite keys (natural grain)
2. **Dimension tables:** Surrogate keys (SCD Type 2 support)
3. **Junction tables:** Composite keys (many-to-many)
4. **Transactional tables:** Surrogate if natural key is complex/unstable
5. **Always keep natural key:** Even with surrogate, keep natural key as UNIQUE constraint

**Hybrid Approach:**
```sql
CREATE TABLE dim_product (
    product_key INT PRIMARY KEY AUTO_INCREMENT,  -- Surrogate (for SCD)
    product_code VARCHAR(50) UNIQUE,              -- Natural key (business)
    product_name VARCHAR(200),
    price DECIMAL(10,2),
    effective_date DATE,
    is_current BOOLEAN
);

-- Benefits:
-- - product_key: Simple FKs, supports history
-- - product_code: Business meaning, lookups
```

---

**Q28: Indexing strategies - B-tree, bitmap, covering indexes?**

**Answer:**

**Indexes** speed up data retrieval by creating a lookup structure, trading storage and write performance for read performance.

**1. B-Tree Index (Default, Most Common)**

**How it works:**
- Balanced tree structure
- Stores sorted key-value pairs
- Efficient for range queries and equality searches

**When to use:**
- High-cardinality columns (many unique values)
- Range queries (>, <, BETWEEN)
- ORDER BY, GROUP BY
- Equality searches (=)

**Example:**
```sql
-- Create B-tree index (default)
CREATE INDEX idx_customers_email ON customers(email);

-- Benefits:
-- 1. Equality search
SELECT * FROM customers WHERE email = 'alice@example.com';
-- Uses index ✅

-- 2. Range search
SELECT * FROM orders WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
CREATE INDEX idx_orders_date ON orders(order_date);
-- Uses index for range scan ✅

-- 3. Sorting
SELECT * FROM customers ORDER BY last_name;
CREATE INDEX idx_customers_lastname ON customers(last_name);
-- Index already sorted, no additional sort needed ✅
```

**Cardinality Matters:**
```sql
-- High cardinality: Good for B-tree ✅
CREATE INDEX idx_customers_email ON customers(email);
-- 1 million customers, ~1 million unique emails

-- Low cardinality: Inefficient B-tree ❌
CREATE INDEX idx_orders_status ON orders(status);
-- 10 million orders, only 3 statuses (Pending, Shipped, Delivered)
-- Index doesn't help much (scans 1/3 of table anyway)
```

---

**2. Bitmap Index**

**How it works:**
- Stores bitmaps (bit arrays) for each distinct value
- Efficient for low-cardinality columns
- Great for complex WHERE clauses with multiple conditions

**When to use:**
- Low-cardinality columns (few unique values)
- Data warehouse/OLAP (read-heavy)
- Complex queries with multiple filters

**Example:**
```sql
-- Bitmap index (Oracle, PostgreSQL with extension)
CREATE BITMAP INDEX idx_orders_status ON orders(status);
CREATE BITMAP INDEX idx_orders_priority ON orders(priority);
CREATE BITMAP INDEX idx_orders_region ON orders(region);

-- Query with multiple filters (bitmap magic!)
SELECT * FROM orders
WHERE status = 'Shipped'
  AND priority = 'High'
  AND region = 'Northeast';

-- Database performs bitmap AND operation:
-- status_bitmap     [1,0,1,1,0,1,0,1...]  (Shipped)
-- priority_bitmap   [1,1,0,0,1,0,1,0...]  (High)
-- region_bitmap     [1,0,0,1,1,0,0,1...]  (Northeast)
-- Result            [1,0,0,0,0,0,0,0...]  (AND all three)
-- ✅ Very fast bitmap operations!
```

**B-tree vs Bitmap:**

| Aspect | B-tree | Bitmap |
|--------|--------|--------|
| **Cardinality** | High (email, SSN) | Low (status, gender, region) |
| **Updates** | Fast | Slow (entire bitmap) |
| **Storage** | More for low-cardinality | Less for low-cardinality |
| **Complex WHERE** | Multiple index scans | Single bitmap AND/OR |
| **Use case** | OLTP | OLAP/Data Warehouse |

---

**3. Covering Index (Include Columns)**

**How it works:**
- Index contains all columns needed for query
- Database can satisfy query entirely from index (no table access!)

**When to use:**
- Frequently queried column combinations
- Avoid expensive table lookups

**Example:**
```sql
-- Query: Get customer name and email
SELECT first_name, last_name, email
FROM customers
WHERE last_name = 'Smith';

-- Without covering index:
CREATE INDEX idx_customers_lastname ON customers(last_name);
-- 1. Use index to find matching rows
-- 2. Access table to get first_name, last_name, email ❌ Expensive!

-- With covering index (PostgreSQL):
CREATE INDEX idx_customers_lastname_covering
ON customers(last_name)
INCLUDE (first_name, email);

-- Or (MySQL):
CREATE INDEX idx_customers_lastname_covering
ON customers(last_name, first_name, email);

-- Now:
-- 1. Use index to find matching rows
-- 2. Get ALL needed columns from index ✅ No table access!
```

**Covering Index Benefits:**
```sql
-- Before: Index Scan + Table Lookup
EXPLAIN SELECT first_name, email FROM customers WHERE last_name = 'Smith';
-- Index Scan on idx_customers_lastname
-- -> Heap Fetch on customers (expensive!)

-- After: Index-Only Scan
EXPLAIN SELECT first_name, email FROM customers WHERE last_name = 'Smith';
-- Index Only Scan on idx_customers_lastname_covering ✅
-- No heap fetch!
```

---

**4. Composite/Multi-Column Index**

**When to use:**
- Queries filter on multiple columns
- Column order matters!

**Example:**
```sql
-- Composite index
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- Query 1: Uses index ✅
SELECT * FROM orders
WHERE customer_id = 123 AND order_date = '2024-05-01';

-- Query 2: Uses index (leftmost prefix) ✅
SELECT * FROM orders WHERE customer_id = 123;

-- Query 3: Does NOT use index ❌
SELECT * FROM orders WHERE order_date = '2024-05-01';
-- order_date is not leftmost column!

-- Fix: Create separate index
CREATE INDEX idx_orders_date ON orders(order_date);
```

**Column Order Matters:**
```sql
-- Rule: Most selective column first

-- Bad: Low selectivity first
CREATE INDEX idx_orders_bad ON orders(status, customer_id);
-- status has 3 values (not selective)
-- customer_id has 100K values (very selective)

-- Good: High selectivity first ✅
CREATE INDEX idx_orders_good ON orders(customer_id, status);

-- Query:
WHERE customer_id = 123 AND status = 'Shipped'
-- Filters to 1 customer first (selective)
-- Then filters by status (small remaining set)
```

---

**Optum Healthcare Example:**

```sql
-- Claims fact table (1.2 billion rows)
CREATE TABLE fact_claims (
    claim_id VARCHAR(50),
    claim_line_number INT,
    service_date_key INT,
    received_date_key INT,
    patient_key INT,
    provider_key INT,
    diagnosis_key INT,
    procedure_code VARCHAR(10),
    claim_status VARCHAR(20),  -- Pending, Approved, Denied (low cardinality)
    claim_type VARCHAR(20),    -- Inpatient, Outpatient, Pharmacy (low cardinality)
    billed_amount DECIMAL(10,2),
    paid_amount DECIMAL(10,2),
    PRIMARY KEY (claim_id, claim_line_number)
);

-- B-tree indexes (high cardinality)
CREATE INDEX idx_claims_patient ON fact_claims(patient_key);
-- 10M patients, high cardinality ✅

CREATE INDEX idx_claims_provider ON fact_claims(provider_key);
-- 500K providers, high cardinality ✅

CREATE INDEX idx_claims_service_date ON fact_claims(service_date_key);
-- 365+ dates, range queries common ✅

-- Bitmap indexes (low cardinality - data warehouse)
CREATE BITMAP INDEX idx_claims_status ON fact_claims(claim_status);
-- Only 3 values: Pending, Approved, Denied

CREATE BITMAP INDEX idx_claims_type ON fact_claims(claim_type);
-- Only 3 values: Inpatient, Outpatient, Pharmacy

-- Covering index for common dashboard query
CREATE INDEX idx_claims_patient_covering
ON fact_claims(patient_key, service_date_key)
INCLUDE (paid_amount, procedure_code);

-- Query: Patient cost summary (uses covering index)
SELECT
    patient_key,
    service_date_key,
    SUM(paid_amount) as total_cost,
    COUNT(DISTINCT procedure_code) as procedure_count
FROM fact_claims
WHERE patient_key = 12345
  AND service_date_key BETWEEN 20240101 AND 20241231
GROUP BY patient_key, service_date_key;
-- Index-only scan! No table access ✅

-- Composite index for filtered reporting
CREATE INDEX idx_claims_date_status_type
ON fact_claims(service_date_key, claim_status, claim_type);

-- Complex query with multiple filters
SELECT
    service_date_key,
    claim_status,
    claim_type,
    COUNT(*) as claim_count,
    SUM(paid_amount) as total_paid
FROM fact_claims
WHERE service_date_key BETWEEN 20240101 AND 20241231
  AND claim_status = 'Approved'
  AND claim_type = 'Inpatient'
GROUP BY service_date_key, claim_status, claim_type;

-- With bitmap indexes, database does:
-- date_bitmap AND status_bitmap AND type_bitmap ✅
-- Very fast for data warehouse queries!

-- Interview talking point:
-- "At Optum, we used B-tree indexes on high-cardinality columns like
-- patient_key and provider_key in our 1.2 billion row claims fact table.
-- For data warehouse queries with low-cardinality filters (claim_status,
-- claim_type), we used bitmap indexes which reduced complex query times
-- from 45 seconds to 3 seconds. We also created covering indexes for
-- common dashboard queries, eliminating table lookups and improving
-- response times by 80%."
```

**Indexing Best Practices:**

1. **Index columns in WHERE, JOIN, ORDER BY**
2. **Don't over-index** - Slows down INSERT/UPDATE/DELETE
3. **Monitor index usage** - Drop unused indexes
4. **Consider cardinality** - B-tree for high, bitmap for low
5. **Composite index column order** - Most selective first
6. **Covering indexes** - For frequently queried column sets
7. **Partial indexes** - Index only subset of rows
8. **Expression indexes** - Index computed values

**Monitoring Index Usage:**
```sql
-- PostgreSQL: Check index usage
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan as index_scans,
    idx_tup_read as tuples_read,
    idx_tup_fetch as tuples_fetched
FROM pg_stat_user_indexes
WHERE idx_scan = 0  -- Never used!
ORDER BY schemaname, tablename;

-- Drop unused index
DROP INDEX idx_never_used;
```

---

**Q29: Partitioning vs Sharding - differences and when to use?**

**Answer:**

**Partitioning** and **Sharding** both divide data into smaller chunks, but differ in scope and purpose.

**Partitioning: Divide Within Single Database**

**Definition:**
- Split large table into smaller physical pieces (partitions)
- Logical table remains one
- All partitions on same database server (or distributed in same cluster)

**Types:**

**1. Horizontal Partitioning (by rows)**
```sql
-- Partition by date range
CREATE TABLE orders (
    order_id BIGINT,
    order_date DATE,
    customer_id BIGINT,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (order_date);

CREATE TABLE orders_2024_q1 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');

CREATE TABLE orders_2024_q2 PARTITION OF orders
    FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');

-- Partition by list (category)
CREATE TABLE customers PARTITION BY LIST (country);

CREATE TABLE customers_us PARTITION OF customers
    FOR VALUES IN ('US');

CREATE TABLE customers_eu PARTITION OF customers
    FOR VALUES IN ('UK', 'FR', 'DE');

-- Partition by hash (even distribution)
CREATE TABLE events PARTITION BY HASH (user_id);

CREATE TABLE events_p0 PARTITION OF events
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE events_p1 PARTITION OF events
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);
```

**2. Vertical Partitioning (by columns)**
```sql
-- Split frequently accessed vs rarely accessed columns

-- Hot data (frequently accessed)
CREATE TABLE customers_hot (
    customer_id BIGINT PRIMARY KEY,
    email VARCHAR(200),
    first_name VARCHAR(100),
    last_name VARCHAR(100)
);

-- Cold data (rarely accessed)
CREATE TABLE customers_cold (
    customer_id BIGINT PRIMARY KEY,
    address VARCHAR(500),
    notes TEXT,
    preferences JSONB,
    FOREIGN KEY (customer_id) REFERENCES customers_hot(customer_id)
);

-- Query hot data only (fast!)
SELECT email, first_name FROM customers_hot WHERE customer_id = 123;
```

**Benefits of Partitioning:**
- ✅ Query performance (partition pruning)
- ✅ Easier maintenance (drop old partitions)
- ✅ Parallel query execution
- ✅ Improved backup/restore
- ✅ All on same server (simpler)

---

**Sharding: Divide Across Multiple Databases**

**Definition:**
- Split data across multiple separate database servers
- Each shard is independent database
- Horizontal scaling (add more servers)

**Sharding Strategies:**

**1. Range-Based Sharding**
```sql
-- Shard by customer ID ranges

-- Shard 1 (Server 1): customers 1-1000000
customers_shard1:
  customer_id 1 - 1000000

-- Shard 2 (Server 2): customers 1000001-2000000
customers_shard2:
  customer_id 1000001 - 2000000

-- Shard 3 (Server 3): customers 2000001-3000000
customers_shard3:
  customer_id 2000001 - 3000000

-- Application routing logic:
def get_shard(customer_id):
    if customer_id <= 1000000:
        return shard1_connection
    elif customer_id <= 2000000:
        return shard2_connection
    else:
        return shard3_connection
```

**2. Hash-Based Sharding**
```sql
-- Shard by hash of customer ID

-- Application logic:
def get_shard(customer_id):
    shard_num = hash(customer_id) % num_shards
    return shard_connections[shard_num]

-- Even distribution across shards
-- customer_id 123 → hash → shard 2
-- customer_id 456 → hash → shard 0
-- customer_id 789 → hash → shard 1
```

**3. Geographic Sharding**
```sql
-- Shard by region

-- Shard 1 (US East): US customers
shard_us_east:
  customers WHERE region = 'US'

-- Shard 2 (EU): European customers
shard_eu:
  customers WHERE region IN ('UK', 'FR', 'DE')

-- Shard 3 (APAC): Asia-Pacific customers
shard_apac:
  customers WHERE region IN ('JP', 'SG', 'AU')

-- Benefits: Data locality, compliance (GDPR)
```

**Benefits of Sharding:**
- ✅ Horizontal scalability (add more servers)
- ✅ Handle massive datasets (petabyte scale)
- ✅ Distribute write load across servers
- ✅ Geographic distribution

**Challenges of Sharding:**
- ❌ Complex application logic (routing)
- ❌ Cross-shard queries expensive
- ❌ Rebalancing shards difficult
- ❌ Transactions across shards complex
- ❌ No native database support (application-level)

---

**Partitioning vs Sharding Comparison:**

| Aspect | Partitioning | Sharding |
|--------|--------------|----------|
| **Scope** | Single database server | Multiple database servers |
| **Scalability** | Vertical (bigger server) | Horizontal (more servers) |
| **Complexity** | Low (database handles it) | High (application handles it) |
| **Cross-partition queries** | Easy (same database) | Hard (multiple databases) |
| **Maintenance** | Easier | Harder |
| **Cost** | Lower (fewer servers) | Higher (many servers) |
| **Use case** | Large tables, read performance | Massive scale, write scaling |

---

**When to Use Each:**

**Use Partitioning When:**
- Single database can handle total data volume
- Need better query performance on large tables
- Want simpler maintenance (drop old partitions)
- Time-series data (partition by date)
- Read-heavy workload

**Use Sharding When:**
- Data exceeds single database capacity
- Need to scale writes horizontally
- Want geographic distribution
- Multi-tenant SaaS (shard by tenant)
- Willing to handle application complexity

---

**Optum Healthcare Example:**

**Partitioning (Used for Claims Data Warehouse):**
```sql
-- 1.2 billion row claims fact table
-- Partitioned by service_date_key (monthly)

CREATE TABLE fact_claims (
    claim_id VARCHAR(50),
    claim_line_number INT,
    service_date_key INT,
    patient_key INT,
    provider_key INT,
    paid_amount DECIMAL(10,2),
    PRIMARY KEY (claim_id, claim_line_number, service_date_key)
) PARTITION BY RANGE (service_date_key);

-- Monthly partitions
CREATE TABLE fact_claims_202401 PARTITION OF fact_claims
    FOR VALUES FROM (20240101) TO (20240201);

CREATE TABLE fact_claims_202402 PARTITION OF fact_claims
    FOR VALUES FROM (20240201) TO (20240301);

-- ...24 months of partitions

-- Query with partition pruning
SELECT SUM(paid_amount)
FROM fact_claims
WHERE service_date_key BETWEEN 20240501 AND 20240531;
-- Scans only fact_claims_202405 partition ✅
-- Query time: 3 seconds (vs 45 seconds scanning all partitions)

-- Maintenance: Drop old partitions
DROP TABLE fact_claims_202201;  -- Milliseconds vs hours for DELETE

-- Interview talking point:
-- "At Optum, we partitioned our 1.2 billion row claims fact table by
-- service_date_key using monthly partitions. This reduced query times
-- from 45 seconds to 3 seconds through partition pruning, and allowed
-- us to drop old partitions in milliseconds for retention policies."
```

**Sharding (Could Use for Multi-Tenant SaaS):**
```sql
-- If Optum had multi-tenant SaaS product
-- Shard by health plan (tenant)

-- Shard 1: Health Plan A (5M members)
shard_plan_a:
  patients, claims, providers for Plan A

-- Shard 2: Health Plan B (3M members)
shard_plan_b:
  patients, claims, providers for Plan B

-- Shard 3: Health Plan C (7M members)
shard_plan_c:
  patients, claims, providers for Plan C

-- Application routing:
def get_database_connection(plan_id):
    shard_map = {
        'PLAN_A': shard_a_connection,
        'PLAN_B': shard_b_connection,
        'PLAN_C': shard_c_connection
    }
    return shard_map[plan_id]

# Query within single tenant (fast)
conn = get_database_connection('PLAN_A')
claims = conn.execute("SELECT * FROM claims WHERE patient_id = ?")

# Cross-tenant reporting (complex!)
# Must query all shards and aggregate
results = []
for shard in all_shards:
    results.extend(shard.execute("SELECT * FROM claims WHERE status = 'Pending'"))
aggregate(results)
```

**Hybrid Approach:**
```sql
-- Partition within each shard

-- Shard 1 (Health Plan A)
-- Partitioned by date within shard
fact_claims_plan_a_202401
fact_claims_plan_a_202402
...

-- Shard 2 (Health Plan B)
-- Partitioned by date within shard
fact_claims_plan_b_202401
fact_claims_plan_b_202402
...

-- Benefits:
-- - Sharding: Distribute load across servers
-- - Partitioning: Fast queries within each shard
```

---

**Q30: Vertical vs Horizontal Partitioning - detailed explanation?**

**Answer:**

**Horizontal Partitioning (by rows)**

**Definition:**
- Split table into multiple tables with same columns but different rows
- Each partition contains subset of rows

**Example:**
```sql
-- Original table: 100M orders
orders (
    order_id,
    order_date,
    customer_id,
    amount
) -- 100M rows

-- Horizontally partitioned by year
orders_2022 (order_id, order_date, customer_id, amount)  -- 30M rows
orders_2023 (order_id, order_date, customer_id, amount)  -- 35M rows
orders_2024 (order_id, order_date, customer_id, amount)  -- 35M rows

-- Same schema, different row subsets
```

**When to Use:**
- Large tables (billions of rows)
- Time-series data (partition by date)
- Queries filter on partition key
- Archival/retention policies

**Benefits:**
- ✅ Faster queries (smaller partitions to scan)
- ✅ Easier maintenance (drop old partitions)
- ✅ Parallel query execution
- ✅ Better backup/restore

---

**Vertical Partitioning (by columns)**

**Definition:**
- Split table into multiple tables with different columns
- Each partition contains subset of columns
- All partitions have same number of rows

**Example:**
```sql
-- Original table: Wide with many columns
customers (
    customer_id,          -- PK
    email,                -- Frequently accessed
    first_name,           -- Frequently accessed
    last_name,            -- Frequently accessed
    phone,                -- Frequently accessed
    address,              -- Rarely accessed
    city,                 -- Rarely accessed
    state,                -- Rarely accessed
    zip,                  -- Rarely accessed
    preferences,          -- Rarely accessed (large JSONB)
    notes,                -- Rarely accessed (large TEXT)
    created_at,           -- Rarely accessed
    updated_at            -- Rarely accessed
) -- 10M rows

-- Vertically partitioned
customers_hot (
    customer_id PRIMARY KEY,
    email,
    first_name,
    last_name,
    phone
) -- 10M rows, small row size

customers_cold (
    customer_id PRIMARY KEY,
    address,
    city,
    state,
    zip,
    preferences,
    notes,
    created_at,
    updated_at,
    FOREIGN KEY (customer_id) REFERENCES customers_hot(customer_id)
) -- 10M rows, large row size

-- Same rows, different columns
```

**When to Use:**
- Wide tables with many columns
- Some columns accessed frequently, others rarely
- Large columns (TEXT, JSONB, BLOB)
- Want to improve cache hit rate

**Benefits:**
- ✅ Smaller row size for hot table (better caching)
- ✅ Faster scans on frequently accessed columns
- ✅ Separate storage tiers (hot SSD, cold HDD)
- ✅ Reduced I/O for common queries

---

**Comparison:**

| Aspect | Horizontal | Vertical |
|--------|------------|----------|
| **Splits by** | Rows | Columns |
| **Schema** | Same columns, different rows | Different columns, same rows |
| **Use case** | Large tables, time-series | Wide tables, hot/cold data |
| **Query pattern** | Filter on partition key | Access subset of columns |
| **Complexity** | Low (database native) | Medium (manual split) |
| **JOIN impact** | None (within partition) | More JOINs (across partitions) |

---

**Optum Healthcare Example:**

**Horizontal Partitioning:**
```sql
-- Claims fact table partitioned by service_date (horizontal)
CREATE TABLE fact_claims (
    claim_id VARCHAR(50),
    service_date_key INT,
    patient_key INT,
    paid_amount DECIMAL(10,2),
    PRIMARY KEY (claim_id, service_date_key)
) PARTITION BY RANGE (service_date_key);

-- Partitions (same schema, different rows)
fact_claims_202401  -- Jan 2024 claims
fact_claims_202402  -- Feb 2024 claims
fact_claims_202403  -- Mar 2024 claims

-- Query: Only scans relevant partition
SELECT SUM(paid_amount)
FROM fact_claims
WHERE service_date_key BETWEEN 20240201 AND 20240229;
-- Scans only fact_claims_202402 ✅
```

**Vertical Partitioning:**
```sql
-- Patient dimension with hot/cold split (vertical)

-- Hot: Frequently accessed for claims processing
CREATE TABLE dim_patient_hot (
    patient_key BIGINT PRIMARY KEY,
    patient_id VARCHAR(50),
    date_of_birth DATE,
    gender VARCHAR(10),
    insurance_plan VARCHAR(100),
    risk_score DECIMAL(5,2),
    is_current BOOLEAN
) -- 10M rows × 100 bytes = 1GB

-- Cold: Rarely accessed demographics
CREATE TABLE dim_patient_cold (
    patient_key BIGINT PRIMARY KEY,
    full_address VARCHAR(500),
    phone VARCHAR(20),
    email VARCHAR(200),
    emergency_contact VARCHAR(200),
    medical_history TEXT,       -- Large
    preferences JSONB,          -- Large
    notes TEXT,                 -- Large
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    FOREIGN KEY (patient_key) REFERENCES dim_patient_hot(patient_key)
) -- 10M rows × 2000 bytes = 20GB

-- Claims processing query (90% of queries)
SELECT
    p.patient_id,
    p.insurance_plan,
    p.risk_score,
    SUM(c.paid_amount) as total_cost
FROM fact_claims c
JOIN dim_patient_hot p ON c.patient_key = p.patient_key
WHERE c.service_date_key = 20240501
GROUP BY p.patient_id, p.insurance_plan, p.risk_score;
-- Joins small hot table (1GB) ✅
-- No need to access cold table (20GB)
-- Better memory caching!

-- Rare query needing cold data
SELECT
    h.patient_id,
    h.insurance_plan,
    c.full_address,
    c.phone,
    c.email
FROM dim_patient_hot h
JOIN dim_patient_cold c ON h.patient_key = c.patient_key
WHERE h.patient_id = 'P12345';
-- JOIN required but rare ✅

-- Interview talking point:
-- "At Optum, we used horizontal partitioning for our 1.2 billion row
-- claims fact table, partitioning by service_date_key. We also used
-- vertical partitioning for the patient dimension, splitting frequently
-- accessed attributes (demographics, insurance) into a hot table and
-- rarely accessed attributes (addresses, notes) into a cold table.
-- This reduced the hot patient dimension from 20GB to 1GB, improving
-- cache hit rates and speeding up 90% of queries by 50%."
```

**Hybrid: Both Horizontal and Vertical:**
```sql
-- Patient dimension:
-- - Vertically partitioned (hot/cold)
-- - Hot table horizontally partitioned (by state for compliance)

-- Hot table partitioned by state
CREATE TABLE dim_patient_hot PARTITION BY LIST (state);

CREATE TABLE dim_patient_hot_ca PARTITION OF dim_patient_hot
    FOR VALUES IN ('CA');  -- California patients

CREATE TABLE dim_patient_hot_ny PARTITION OF dim_patient_hot
    FOR VALUES IN ('NY');  -- New York patients

-- Benefits:
-- - Vertical: Separate hot/cold data
-- - Horizontal: State-specific compliance, data residency
```

---

**Q31: Database constraints - types and usage?**

**Answer:**

**Constraints** enforce data integrity rules at the database level.

**1. PRIMARY KEY**

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,  -- Unique, NOT NULL
    name VARCHAR(200)
);

-- Composite primary key
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);
```

**Rules:**
- Must be unique
- Cannot be NULL
- Only one per table
- Automatically creates index

---

**2. FOREIGN KEY**

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);
```

**Rules:**
- References PRIMARY KEY or UNIQUE column in another table
- Enforces referential integrity
- Prevents orphaned records

---

**3. UNIQUE**

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    email VARCHAR(200) UNIQUE,  -- No duplicates allowed
    username VARCHAR(50) UNIQUE
);

-- Composite unique constraint
CREATE TABLE enrollments (
    enrollment_id INT PRIMARY KEY,
    student_id INT,
    course_id INT,
    UNIQUE (student_id, course_id)  -- Student can enroll once per course
);
```

**Rules:**
- No duplicate values allowed
- Can have multiple UNIQUE constraints per table
- NULL values allowed (usually one NULL per column)

---

**4. CHECK**

```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    price DECIMAL(10,2) CHECK (price >= 0),  -- Non-negative price
    stock_quantity INT CHECK (stock_quantity >= 0),
    category VARCHAR(50) CHECK (category IN ('Electronics', 'Clothing', 'Food'))
);

-- Named constraint
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    order_date DATE,
    ship_date DATE,
    CONSTRAINT valid_dates CHECK (ship_date >= order_date)
);
```

**Rules:**
- Validates data based on condition
- Can reference multiple columns
- Evaluated on INSERT/UPDATE

---

**5. NOT NULL**

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,  -- Required
    last_name VARCHAR(100) NOT NULL,   -- Required
    email VARCHAR(200) NOT NULL,
    phone VARCHAR(20)  -- Optional (can be NULL)
);
```

**Rules:**
- Column must have a value
- Cannot INSERT/UPDATE to NULL

---

**6. DEFAULT**

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    order_date DATE DEFAULT CURRENT_DATE,
    status VARCHAR(20) DEFAULT 'Pending',
    priority INT DEFAULT 1
);

-- INSERT without specifying default columns
INSERT INTO orders (order_id) VALUES (1);
-- order_date = today, status = 'Pending', priority = 1
```

**Rules:**
- Provides default value if not specified
- Applied on INSERT

---

**Optum Healthcare Example:**

```sql
CREATE TABLE claims (
    -- Primary key
    claim_id VARCHAR(50) PRIMARY KEY,

    -- Foreign keys with referential integrity
    patient_id VARCHAR(50) NOT NULL,
    provider_id VARCHAR(50) NOT NULL,
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id)
        ON DELETE RESTRICT,  -- Can't delete patient with claims
    FOREIGN KEY (provider_id) REFERENCES providers(provider_id)
        ON DELETE RESTRICT,

    -- Unique constraint
    external_claim_id VARCHAR(100) UNIQUE,  -- External system ID

    -- Check constraints for data validation
    service_date DATE NOT NULL CHECK (service_date <= CURRENT_DATE),  -- Can't be future
    received_date DATE NOT NULL CHECK (received_date >= service_date),  -- Received after service
    billed_amount DECIMAL(10,2) NOT NULL CHECK (billed_amount >= 0),  -- Non-negative
    paid_amount DECIMAL(10,2) CHECK (paid_amount >= 0 AND paid_amount <= billed_amount),  -- Paid ≤ Billed

    -- Status with valid values
    claim_status VARCHAR(20) NOT NULL DEFAULT 'Received'
        CHECK (claim_status IN ('Received', 'Processing', 'Approved', 'Denied', 'Paid')),

    -- Required fields
    diagnosis_code VARCHAR(10) NOT NULL,
    procedure_code VARCHAR(10) NOT NULL,

    -- Audit fields with defaults
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Benefits:
-- ✅ Data integrity enforced at database level
-- ✅ Prevents invalid data (negative amounts, invalid statuses)
-- ✅ Business rules encoded in schema
-- ✅ Referential integrity (no orphaned claims)

-- Interview talking point:
-- "At Optum, we used comprehensive database constraints on our claims
-- tables to ensure HIPAA compliance and data integrity. CHECK constraints
-- validated that paid_amount never exceeded billed_amount, service dates
-- weren't in the future, and claim statuses were valid. FOREIGN KEY
-- constraints with ON DELETE RESTRICT prevented accidental deletion of
-- patients or providers who had associated claims, which was critical
-- for regulatory compliance and audit trails."
```

---

**Q32: Cascading deletes/updates - how do they work?**

**Answer:**

**Cascading** automatically propagates changes from parent to child tables based on foreign key constraints.

**ON DELETE CASCADE**

Automatically delete child rows when parent is deleted.

```sql
-- Setup
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(200)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    amount DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE CASCADE  -- Delete orders when customer deleted
);

-- Data
INSERT INTO customers VALUES (1, 'Alice'), (2, 'Bob');
INSERT INTO orders VALUES (100, 1, 50.00), (101, 1, 75.00), (102, 2, 100.00);

-- Delete customer
DELETE FROM customers WHERE customer_id = 1;

-- Result: Customer AND both orders deleted automatically ✅
-- orders 100 and 101 are gone (cascaded)
-- order 102 remains (different customer)
```

---

**ON DELETE SET NULL**

Set child foreign key to NULL when parent is deleted.

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,  -- Must be nullable
    amount DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE SET NULL  -- Set to NULL when customer deleted
);

DELETE FROM customers WHERE customer_id = 1;

-- Result: Customer deleted, orders remain with customer_id = NULL
-- order 100: customer_id = NULL
-- order 101: customer_id = NULL
```

---

**ON DELETE RESTRICT (Default)**

Prevent parent deletion if children exist.

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE RESTRICT  -- Prevent deletion if orders exist
);

DELETE FROM customers WHERE customer_id = 1;
-- ❌ ERROR: Cannot delete customer - foreign key constraint violation
-- Must delete orders first
```

---

**ON DELETE SET DEFAULT**

Set child foreign key to default value when parent deleted.

```sql
-- Create default customer
INSERT INTO customers VALUES (0, 'Unknown Customer');

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT DEFAULT 0,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE SET DEFAULT  -- Set to 0 when customer deleted
);

DELETE FROM customers WHERE customer_id = 1;

-- Result: orders.customer_id set to 0 (Unknown Customer)
```

---

**ON UPDATE CASCADE**

Automatically update child foreign keys when parent primary key changes.

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON UPDATE CASCADE  -- Update FK when PK changes
);

-- Change customer PK
UPDATE customers SET customer_id = 999 WHERE customer_id = 1;

-- Result: All orders.customer_id updated from 1 to 999 ✅
```

---

**Cascade Chains (Multi-Level)**

```sql
-- Three-level cascade
customers → orders → order_items

CREATE TABLE customers (
    customer_id INT PRIMARY KEY
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE CASCADE
);

CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
        ON DELETE CASCADE
);

-- Delete customer
DELETE FROM customers WHERE customer_id = 1;

-- Result:
-- 1. Customer deleted
-- 2. All customer's orders deleted (cascade)
-- 3. All order_items for those orders deleted (cascade chain)
-- ✅ Everything cleaned up!
```

---

**Optum Healthcare Example:**

```sql
-- Patient session and temporary data (CASCADE)
CREATE TABLE patient_sessions (
    session_id VARCHAR(50) PRIMARY KEY,
    patient_id VARCHAR(50),
    login_time TIMESTAMP,
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id)
        ON DELETE CASCADE  -- OK: Sessions are temporary
);

CREATE TABLE session_activity (
    activity_id BIGINT PRIMARY KEY,
    session_id VARCHAR(50),
    activity_type VARCHAR(50),
    FOREIGN KEY (session_id) REFERENCES patient_sessions(session_id)
        ON DELETE CASCADE  -- OK: Activity tied to session
);

-- Claims data (RESTRICT - never cascade delete!)
CREATE TABLE claims (
    claim_id VARCHAR(50) PRIMARY KEY,
    patient_id VARCHAR(50),
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id)
        ON DELETE RESTRICT  -- NEVER delete claims! (compliance)
);

-- Correct approach for "deleting" patient
UPDATE patients
SET status = 'Inactive',
    soft_deleted_at = CURRENT_TIMESTAMP
WHERE patient_id = 'P001';

-- Sessions can be deleted (temporary data)
DELETE FROM patient_sessions WHERE patient_id = 'P001';
-- ✅ session_activity rows cascade deleted automatically

-- Claims are preserved (regulatory requirement)
SELECT COUNT(*) FROM claims WHERE patient_id = 'P001';
-- ✅ All claims still exist

-- Interview talking point:
-- "At Optum, we used ON DELETE CASCADE for temporary data like user
-- sessions and activity logs, which could safely be deleted. However,
-- for critical healthcare data like claims and patient medical records,
-- we used ON DELETE RESTRICT to prevent accidental deletion. This was
-- a HIPAA compliance requirement - we could only soft-delete patients
-- (set status='Inactive') while preserving all historical claims data.
-- Cascading deletes would have been catastrophic for audit trails."
```

**When to Use Each:**

| Action | Use Case | Example |
|--------|----------|---------|
| **CASCADE** | Parent-child lifecycle tied together | User → Sessions, Post → Comments |
| **RESTRICT** | Prevent accidental deletion | Customer → Orders (check first) |
| **SET NULL** | Optional relationship | Order → Salesperson (can be unassigned) |
| **SET DEFAULT** | Fallback to default | Order → Unknown Customer |

**Dangers of CASCADE:**

```sql
-- ⚠️ CASCADE can delete massive amounts of data

-- One DELETE could cascade to millions of rows:
DELETE FROM customers WHERE customer_id = 1;
-- → Deletes 1,000 orders
-- → Deletes 5,000 order_items
-- → Deletes 5,000 shipments
-- → Deletes 10,000 tracking_events
-- = 21,001 rows deleted with one statement!

-- Be very careful with CASCADE in production!
```

---

**Q33: Data type selection - INT vs BIGINT, VARCHAR vs TEXT?**

**Answer:**

Choosing correct data types impacts storage, performance, and data integrity.

**Integer Types:**

```sql
-- SMALLINT: -32,768 to 32,767 (2 bytes)
CREATE TABLE settings (
    setting_id SMALLINT,  -- Good: Few settings
    priority SMALLINT     -- Good: Priority 1-10
);

-- INT: -2.1B to 2.1B (4 bytes)
CREATE TABLE customers (
    customer_id INT  -- Good: Up to 2 billion customers
);

-- BIGINT: -9.2 quintillion to 9.2 quintillion (8 bytes)
CREATE TABLE events (
    event_id BIGINT  -- Good: Billions+ events
);

-- AUTO_INCREMENT considerations
CREATE TABLE orders (
    order_id INT AUTO_INCREMENT  -- Will run out at 2.1B
    -- Use BIGINT for high-volume tables!
);

CREATE TABLE clicks (
    click_id BIGINT AUTO_INCREMENT  -- Safe for billions
);
```

**When to use:**
- **SMALLINT**: Small ranges (status codes, flags, priorities)
- **INT**: Most general use cases (customers, products)
- **BIGINT**: High-volume fact tables, event logging

---

**String Types:**

**CHAR vs VARCHAR:**

```sql
-- CHAR: Fixed length, padded with spaces
state CHAR(2)          -- 'CA' → 'CA' (always 2 bytes)
country_code CHAR(3)   -- 'USA' → 'USA' (always 3 bytes)

-- VARCHAR: Variable length
name VARCHAR(200)      -- 'Alice' → 5 bytes + overhead
email VARCHAR(200)     -- 'a@b.com' → 7 bytes + overhead
```

**When to use CHAR:**
- Fixed-length data (state codes, country codes, SSN)
- Slight performance benefit for fixed searches
- Storage waste if actual length varies

**When to use VARCHAR:**
- Variable-length data (names, emails, descriptions)
- Most common choice

---

**VARCHAR vs TEXT:**

```sql
-- VARCHAR(n): Max length specified
description VARCHAR(500)    -- Max 500 characters
-- Enforces length limit
-- Faster indexing (can index full column)

-- TEXT: Unlimited length (up to DB limit)
notes TEXT                  -- No length limit
-- Use for large content
-- Indexing requires prefix: CREATE INDEX idx_notes ON table(notes(100))
```

**Performance:**
```sql
-- VARCHAR: Stored inline with row (if small enough)
customers (
    customer_id INT,
    name VARCHAR(200)  -- Stored with row ✅ Fast
);

-- TEXT: Stored separately (out-of-line)
customers (
    customer_id INT,
    biography TEXT  -- Stored separately, pointer in row
);
-- SELECT * fetches extra data ❌ Slower
-- SELECT customer_id, name (no TEXT) ✅ Fast
```

**When to use:**
- **VARCHAR**: Known max length, needs indexing, <1000 chars
- **TEXT**: Large content, no length limit needed

---

**Decimal Types:**

```sql
-- DECIMAL/NUMERIC: Exact precision (for money!)
price DECIMAL(10,2)  -- 10 total digits, 2 after decimal
-- 12345678.99 ✅
-- Exact representation, no rounding errors

-- FLOAT/DOUBLE: Approximate, faster
weight FLOAT  -- Scientific calculations
-- May have rounding errors ❌
-- Never use for money!

-- Money example:
CREATE TABLE orders (
    order_id INT,
    subtotal DECIMAL(10,2),  -- Exact
    tax DECIMAL(10,2),        -- Exact
    total DECIMAL(10,2)       -- Exact
);
```

---

**Date/Time Types:**

```sql
-- DATE: Date only (YYYY-MM-DD)
birth_date DATE  -- '1990-05-15'

-- TIME: Time only (HH:MM:SS)
appointment_time TIME  -- '14:30:00'

-- DATETIME: Date + Time
created_at DATETIME  -- '2024-05-03 14:30:00'

-- TIMESTAMP: Date + Time + Timezone
logged_at TIMESTAMP  -- '2024-05-03 14:30:00 UTC'
-- Auto-updates with CURRENT_TIMESTAMP

-- Best practice for audit fields:
CREATE TABLE claims (
    claim_id VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

**Boolean:**

```sql
-- BOOLEAN (PostgreSQL)
is_active BOOLEAN  -- TRUE, FALSE, NULL

-- TINYINT (MySQL)
is_active TINYINT(1)  -- 0 = FALSE, 1 = TRUE

-- BIT (SQL Server)
is_active BIT  -- 0 or 1
```

---

**JSON:**

```sql
-- JSON (unindexed)
metadata JSON  -- Store as text, parse on read

-- JSONB (PostgreSQL, indexed)
preferences JSONB  -- Binary format, can index
CREATE INDEX idx_prefs ON users USING GIN (preferences);

-- Query:
SELECT * FROM users WHERE preferences->>'theme' = 'dark';
```

---

**Optum Healthcare Example:**

```sql
CREATE TABLE claims (
    -- Primary key: BIGINT (billions of claims)
    claim_key BIGINT PRIMARY KEY AUTO_INCREMENT,

    -- Business IDs: VARCHAR (external systems)
    claim_id VARCHAR(50) NOT NULL,
    external_claim_id VARCHAR(100),

    -- Foreign keys: INT or BIGINT depending on volume
    patient_key INT,      -- 10M patients (INT sufficient)
    provider_key INT,     -- 500K providers (INT sufficient)

    -- Codes: CHAR (fixed length)
    diagnosis_code CHAR(7),    -- ICD-10: Always 7 chars (A12.345)
    procedure_code CHAR(5),    -- CPT: Always 5 chars (99213)
    member_id CHAR(11),        -- SSN: Always 11 chars

    -- Amounts: DECIMAL (exact, for money!)
    billed_amount DECIMAL(12,2),   -- Up to $999,999,999.99
    paid_amount DECIMAL(12,2),
    copay_amount DECIMAL(8,2),     -- Smaller max (up to $999,999.99)

    -- Dates: DATE (no time needed)
    service_date DATE,
    received_date DATE,
    processed_date DATE,

    -- Status: VARCHAR (small set of values)
    claim_status VARCHAR(20),  -- 'Received', 'Approved', 'Denied', etc.

    -- Descriptions: VARCHAR (known limits)
    procedure_description VARCHAR(500),
    denial_reason VARCHAR(1000),

    -- Clinical notes: TEXT (unlimited)
    clinical_notes TEXT,       -- Can be very long

    -- Flags: BOOLEAN/TINYINT
    is_urgent BOOLEAN,
    is_resubmission BOOLEAN,

    -- Metadata: JSONB (flexible structure)
    additional_data JSONB,     -- Extra fields from various sources

    -- Audit timestamps: TIMESTAMP
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Storage calculation:
-- BIGINT (8) + VARCHAR(50) (50) + INT (4) + ...
-- = ~200 bytes per row
-- 1 billion claims × 200 bytes = 200 GB
-- vs using BIGINT everywhere: 400 GB ✅ Saved 200 GB!

-- Interview talking point:
-- "At Optum, we carefully selected data types for our billion-row claims
-- table. We used CHAR for fixed-length medical codes (ICD-10, CPT) which
-- saved storage and improved indexing performance. We used DECIMAL for all
-- monetary amounts to avoid floating-point rounding errors - critical for
-- financial accuracy in healthcare. Patient_key was INT (10M patients fit
-- in 2.1B limit) but claim_key was BIGINT (billions of claims). This
-- optimization reduced our table size from 400GB to 200GB while maintaining
-- data integrity."
```

**Decision Matrix:**

| Data | Type | Reason |
|------|------|--------|
| **Money** | DECIMAL(10,2) | Exact precision, no rounding |
| **IDs (high volume)** | BIGINT | Billions of records |
| **IDs (low volume)** | INT | Under 2.1 billion |
| **Status codes** | CHAR(2) | Fixed length |
| **Names, emails** | VARCHAR(n) | Variable, known max |
| **Long text** | TEXT | Unlimited length |
| **Flags** | BOOLEAN | TRUE/FALSE |
| **Dates** | DATE | No time component |
| **Audit timestamps** | TIMESTAMP | Auto-update, timezone |
| **Flexible data** | JSONB | Unstructured/semi-structured |

---

**Q34: Temporal tables - versioning and audit trails?**

**Answer:**

**Temporal tables** (also called **system-versioned tables**) automatically track the full history of data changes with timestamps.

**How They Work:**

```sql
-- SQL Server / PostgreSQL temporal table
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(200),
    department VARCHAR(100),
    salary DECIMAL(10,2),

    -- System versioning columns
    valid_from TIMESTAMP GENERATED ALWAYS AS ROW START,
    valid_to TIMESTAMP GENERATED ALWAYS AS ROW END,

    PERIOD FOR SYSTEM_TIME (valid_from, valid_to)
) WITH SYSTEM VERSIONING;

-- Automatically creates history table: employees_history
```

**Automatic History Tracking:**

```sql
-- Insert
INSERT INTO employees VALUES (1, 'Alice', 'Engineering', 100000);
-- Current table: Alice record
-- History table: empty

-- Update
UPDATE employees SET salary = 110000 WHERE employee_id = 1;
-- Current table: Alice with salary 110000
-- History table: Alice with salary 100000, valid_from -> valid_to timestamps

-- Update again
UPDATE employees SET department = 'Management' WHERE employee_id = 1;
-- Current table: Alice in Management, salary 110000
-- History table:
--   - Alice, Engineering, 100000 (original)
--   - Alice, Engineering, 110000 (after first update)

-- Delete
DELETE FROM employees WHERE employee_id = 1;
-- Current table: empty
-- History table:
--   - All previous versions preserved ✅
```

**Querying History:**

```sql
-- Current data (default)
SELECT * FROM employees WHERE employee_id = 1;
-- Returns: Current version only

-- Point-in-time query (as of specific date)
SELECT * FROM employees
FOR SYSTEM_TIME AS OF '2024-01-15 10:00:00'
WHERE employee_id = 1;
-- Returns: Version as it was on Jan 15, 2024

-- All history
SELECT * FROM employees
FOR SYSTEM_TIME ALL
WHERE employee_id = 1;
-- Returns: All versions with valid_from/valid_to timestamps

-- Changes between dates
SELECT * FROM employees
FOR SYSTEM_TIME BETWEEN '2024-01-01' AND '2024-03-01'
WHERE employee_id = 1;
-- Returns: All versions that were valid in Q1 2024
```

---

**Manual Audit Trail Approach:**

```sql
-- Audit table (manual approach)
CREATE TABLE employees_audit (
    audit_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    employee_id INT,
    name VARCHAR(200),
    department VARCHAR(100),
    salary DECIMAL(10,2),

    -- Audit metadata
    operation VARCHAR(10),  -- INSERT, UPDATE, DELETE
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    changed_by VARCHAR(100)
);

-- Trigger to populate audit table on UPDATE
CREATE TRIGGER employees_audit_trigger
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    INSERT INTO employees_audit (
        employee_id, name, department, salary,
        operation, changed_by
    )
    VALUES (
        OLD.employee_id, OLD.name, OLD.department, OLD.salary,
        'UPDATE', CURRENT_USER
    );
END;

-- Query audit trail
SELECT
    employee_id,
    department,
    salary,
    changed_at,
    changed_by
FROM employees_audit
WHERE employee_id = 1
ORDER BY changed_at DESC;
```

---

**Optum Healthcare Example:**

```sql
-- Patient dimension with system versioning (temporal table)
CREATE TABLE dim_patient (
    patient_key BIGINT PRIMARY KEY,
    patient_id VARCHAR(50),
    name VARCHAR(200),
    date_of_birth DATE,
    insurance_plan VARCHAR(100),
    address VARCHAR(500),

    -- System versioning
    valid_from TIMESTAMP GENERATED ALWAYS AS ROW START,
    valid_to TIMESTAMP GENERATED ALWAYS AS ROW END,
    PERIOD FOR SYSTEM_TIME (valid_from, valid_to)
) WITH SYSTEM VERSIONING;

-- Scenario: Patient changes insurance
-- Jan 1: Patient enrolled in Medicare Advantage
INSERT INTO dim_patient VALUES (
    1, 'P12345', 'John Smith', '1950-01-01', 'Medicare Advantage', '123 Main St'
);

-- June 1: Patient switches to Medicare Original
UPDATE dim_patient
SET insurance_plan = 'Medicare Original'
WHERE patient_id = 'P12345';

-- Dec 1: Patient moves
UPDATE dim_patient
SET address = '456 Oak Ave'
WHERE patient_id = 'P12345';

-- Query: What was patient's insurance on July 15?
SELECT insurance_plan
FROM dim_patient
FOR SYSTEM_TIME AS OF '2024-07-15'
WHERE patient_id = 'P12345';
-- Result: Medicare Original ✅

-- Query: Full patient history
SELECT
    patient_id,
    insurance_plan,
    address,
    valid_from,
    valid_to
FROM dim_patient
FOR SYSTEM_TIME ALL
WHERE patient_id = 'P12345'
ORDER BY valid_from;

-- Result:
-- P12345, Medicare Advantage, 123 Main St, 2024-01-01, 2024-06-01
-- P12345, Medicare Original, 123 Main St, 2024-06-01, 2024-12-01
-- P12345, Medicare Original, 456 Oak Ave, 2024-12-01, 9999-12-31 (current)

-- Claims with correct historical patient data
SELECT
    c.claim_id,
    c.service_date,
    p.insurance_plan,  -- Insurance plan at time of service!
    c.paid_amount
FROM fact_claims c
JOIN dim_patient FOR SYSTEM_TIME AS OF c.service_date p
    ON c.patient_id = p.patient_id
WHERE c.claim_id = 'CLM_123';

-- ✅ Always gets correct insurance plan for the claim service date!

-- Interview talking point:
-- "At Optum, we used temporal tables for patient and provider dimensions
-- to maintain full audit history for HIPAA compliance. When analyzing
-- claims, we could join to the patient dimension AS OF the service date
-- to get the correct insurance plan that was active when the service
-- occurred - critical for accurate financial reporting. The temporal
-- tables automatically tracked all changes with zero application code,
-- and we could reconstruct patient demographics for any point in time
-- for regulatory audits."
```

---

**Benefits:**

| Approach | Pros | Cons |
|----------|------|------|
| **Temporal Tables** | ✅ Automatic history<br>✅ Point-in-time queries<br>✅ Zero code | ❌ Database-specific<br>❌ Storage overhead |
| **Manual Audit** | ✅ Full control<br>✅ Portable | ❌ Trigger complexity<br>❌ Maintenance |
| **SCD Type 2** | ✅ Data warehouse standard<br>✅ Well understood | ❌ Manual ETL logic<br>❌ Complex queries |

**Use Cases:**
- Regulatory compliance (HIPAA, SOX, GDPR)
- Audit trails for financial data
- Data warehouse dimensions (SCD Type 2 alternative)
- Fraud detection (analyze historical patterns)
- Time-travel debugging

---

**Q35: Master Data Management (MDM) - what and why?**

**Answer:**

**Master Data Management (MDM)** is the practice of creating and maintaining a single, authoritative source of truth for critical business entities (customers, products, suppliers, etc.).

**Problem MDM Solves:**

**Without MDM:**
```
Sales System:       Customer "John Smith", ID: S12345, john@email.com
Support System:     Customer "J. Smith", ID: SUP-789, jsmith@email.com
Finance System:     Customer "Smith, John", ID: FIN_999, john.smith@email.com

❌ Same person, 3 different IDs, inconsistent data!
❌ Can't get unified customer view
❌ Duplicate marketing emails
❌ Can't calculate customer lifetime value
```

**With MDM:**
```
MDM System: Master Customer ID: MDM_001
  ├── Name: John Smith (golden record)
  ├── Email: john@email.com (primary)
  ├── Linked to Sales ID: S12345
  ├── Linked to Support ID: SUP-789
  ├── Linked to Finance ID: FIN_999
  └── Single source of truth ✅

All systems reference MDM_001 for customer data
```

---

**MDM Architecture:**

**1. Registry Style (Lightweight)**
```sql
-- MDM registry: Just maps local IDs to master ID
CREATE TABLE mdm_customer_registry (
    mdm_customer_id VARCHAR(50) PRIMARY KEY,  -- Master ID
    source_system VARCHAR(50),                 -- Which system
    source_system_id VARCHAR(50),              -- ID in that system
    UNIQUE (source_system, source_system_id)
);

-- Data
mdm_customer_id | source_system | source_system_id
MDM_001         | SALES         | S12345
MDM_001         | SUPPORT       | SUP-789
MDM_001         | FINANCE       | FIN_999

-- Query: Get all IDs for customer
SELECT source_system, source_system_id
FROM mdm_customer_registry
WHERE mdm_customer_id = 'MDM_001';
```

**2. Consolidation Style (Heavy)**
```sql
-- MDM hub: Stores golden record
CREATE TABLE mdm_customer (
    mdm_customer_id VARCHAR(50) PRIMARY KEY,
    -- Golden record (best/merged data from all sources)
    name VARCHAR(200),
    email VARCHAR(200),
    phone VARCHAR(20),
    address VARCHAR(500),
    -- Metadata
    source_of_truth VARCHAR(50),  -- Which system is authoritative
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Linkage to source systems
CREATE TABLE mdm_customer_linkage (
    mdm_customer_id VARCHAR(50),
    source_system VARCHAR(50),
    source_system_id VARCHAR(50),
    confidence_score DECIMAL(3,2),  -- Match confidence
    FOREIGN KEY (mdm_customer_id) REFERENCES mdm_customer(mdm_customer_id)
);

-- Golden record
mdm_customer_id | name        | email           | source_of_truth
MDM_001         | John Smith  | john@email.com  | SALES

-- Linkages
mdm_customer_id | source_system | source_system_id | confidence
MDM_001         | SALES         | S12345           | 1.00
MDM_001         | SUPPORT       | SUP-789          | 0.95
MDM_001         | FINANCE       | FIN_999          | 0.90
```

---

**Data Matching/Deduplication:**

```sql
-- Fuzzy matching to identify duplicates
SELECT
    c1.customer_id as id1,
    c2.customer_id as id2,
    c1.name as name1,
    c2.name as name2,
    c1.email as email1,
    c2.email as email2,
    -- Match score (simple example)
    CASE
        WHEN c1.email = c2.email THEN 100
        WHEN LEVENSHTEIN(c1.name, c2.name) < 3 THEN 80
        WHEN c1.phone = c2.phone THEN 90
        ELSE 0
    END as match_score
FROM customers c1
JOIN customers c2 ON c1.customer_id < c2.customer_id
WHERE c1.email = c2.email  -- Exact email match
   OR LEVENSHTEIN(c1.name, c2.name) < 3  -- Similar names
   OR c1.phone = c2.phone  -- Same phone
HAVING match_score > 70;

-- Likely duplicates for manual review
```

---

**Optum Healthcare Example:**

```sql
-- MDM for providers (doctors, hospitals)
-- Problem: Same provider in multiple systems with different IDs

-- MDM Provider Hub
CREATE TABLE mdm_provider (
    mdm_provider_id VARCHAR(50) PRIMARY KEY,  -- Master provider ID

    -- Golden record
    provider_name VARCHAR(200),
    npi VARCHAR(10),  -- National Provider Identifier (authoritative)
    specialty VARCHAR(100),
    address VARCHAR(500),
    phone VARCHAR(20),

    -- Metadata
    source_of_truth VARCHAR(50),
    confidence_level VARCHAR(20),  -- High, Medium, Low
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Provider linkage to source systems
CREATE TABLE mdm_provider_linkage (
    mdm_provider_id VARCHAR(50),
    source_system VARCHAR(50),  -- Claims, Credentialing, Network
    source_provider_id VARCHAR(50),
    match_method VARCHAR(50),  -- NPI_exact, Name_fuzzy, Manual
    confidence_score DECIMAL(3,2),
    linked_at TIMESTAMP,
    linked_by VARCHAR(100),
    FOREIGN KEY (mdm_provider_id) REFERENCES mdm_provider(mdm_provider_id)
);

-- Example data
-- MDM Provider Hub
mdm_provider_id | provider_name       | npi        | specialty   | source_of_truth
MDM_PROV_001    | Dr. Jane Smith MD   | 1234567890 | Cardiology  | Credentialing

-- Linkages across systems
mdm_provider_id | source_system    | source_provider_id | match_method | confidence
MDM_PROV_001    | Claims           | CLM_PROV_456      | NPI_exact    | 1.00
MDM_PROV_001    | Credentialing    | CRED_123          | NPI_exact    | 1.00
MDM_PROV_001    | Network          | NET_789           | NPI_exact    | 1.00
MDM_PROV_001    | Legacy_System    | LEG_999           | Name_fuzzy   | 0.85

-- Query: Unified provider view
SELECT
    m.mdm_provider_id,
    m.provider_name,
    m.npi,
    m.specialty,
    COUNT(l.source_system) as linked_systems,
    STRING_AGG(l.source_system, ', ') as systems
FROM mdm_provider m
JOIN mdm_provider_linkage l ON m.mdm_provider_id = l.mdm_provider_id
WHERE m.npi = '1234567890'
GROUP BY m.mdm_provider_id, m.provider_name, m.npi, m.specialty;

-- Claims query using MDM
SELECT
    c.claim_id,
    p.provider_name,  -- From MDM golden record
    p.npi,
    c.paid_amount
FROM claims c
JOIN mdm_provider_linkage l
    ON c.provider_id = l.source_provider_id
    AND l.source_system = 'Claims'
JOIN mdm_provider p
    ON l.mdm_provider_id = p.mdm_provider_id
WHERE c.service_date BETWEEN '2024-01-01' AND '2024-12-31';

-- ✅ Always use golden record provider name from MDM

-- Provider network analysis
SELECT
    p.specialty,
    COUNT(DISTINCT p.mdm_provider_id) as unique_providers,
    SUM(c.paid_amount) as total_paid
FROM mdm_provider p
JOIN mdm_provider_linkage l ON p.mdm_provider_id = l.mdm_provider_id
JOIN claims c ON l.source_provider_id = c.provider_id
WHERE l.source_system = 'Claims'
  AND c.service_date >= '2024-01-01'
GROUP BY p.specialty
ORDER BY total_paid DESC;

-- Interview talking point:
-- "At Optum, we implemented MDM for providers across 15+ source systems.
-- The same doctor could have different IDs in claims, credentialing,
-- network management, and legacy systems. We used NPI (National Provider
-- Identifier) as the golden key for matching, achieving 95% automated
-- matching confidence. This enabled unified provider analytics - we could
-- track a provider's total claims volume, quality metrics, and network
-- status across all systems. MDM reduced provider data errors from 12%
-- to under 2% and enabled accurate Star Ratings reporting for Medicare
-- Advantage."
```

---

**MDM Benefits:**

✅ **Single Source of Truth** - One golden record for each entity
✅ **Data Quality** - Deduplication, standardization
✅ **360° View** - Complete customer/provider/product view
✅ **Compliance** - Consistent data for regulatory reporting
✅ **Analytics** - Accurate aggregation across systems

**MDM Challenges:**

❌ **Complexity** - Matching, deduplication logic
❌ **Governance** - Who owns the golden record?
❌ **Data Quality** - Source systems still contain duplicates
❌ **Performance** - Extra JOIN to MDM hub
❌ **Cost** - MDM software licenses expensive

**When to Use MDM:**
- Multiple source systems with overlapping data
- Need unified customer/product view
- Data quality issues (duplicates, inconsistencies)
- Regulatory compliance requirements
- Enterprise-wide analytics

---

### Modeling Techniques (15 Questions)

**Q36: What is Data Vault modeling?**

**Answer:**
Data Vault is a modeling methodology for enterprise data warehouses, designed for agility and audit.

**Components:**

**1. Hubs:** Business keys
```sql
CREATE TABLE hub_customer (
    customer_hash_key CHAR(32) PRIMARY KEY,  -- Hash of business key
    customer_id VARCHAR(50),                  -- Business key
    load_date TIMESTAMP,
    record_source VARCHAR(50)
);
```

**2. Links:** Relationships
```sql
CREATE TABLE link_order (
    order_hash_key CHAR(32) PRIMARY KEY,
    customer_hash_key CHAR(32),
    product_hash_key CHAR(32),
    order_id VARCHAR(50),
    load_date TIMESTAMP,
    record_source VARCHAR(50)
);
```

**3. Satellites:** Descriptive attributes
```sql
CREATE TABLE sat_customer (
    customer_hash_key CHAR(32),
    load_date TIMESTAMP,
    name VARCHAR(200),
    email VARCHAR(100),
    address VARCHAR(500),
    PRIMARY KEY (customer_hash_key, load_date)
);
```

**Benefits:**
- ✅ Audit trail (all history preserved)
- ✅ Agile (add sources easily)
- ✅ Parallel loading

**Drawbacks:**
- ❌ Complex queries
- ❌ More tables
- ❌ Learning curve

**When to use:**
- Large enterprises with multiple source systems
- Need full audit trail
- Frequently changing requirements

---

**Q37: What is Anchor modeling?**

**Answer:**
Similar to Data Vault but more granular. Each attribute can have its own history.

**Not commonly used** - mostly academic interest.

---

**Q38: Explain Kimball vs Inmon approach**

**Answer:**

**Kimball (Bottom-Up, Dimensional Modeling):**
- Start with business processes
- Build dimensional models (star schemas)
- Data marts first, then consolidate
- Conformed dimensions for consistency

**Structure:**
```
Source Systems → Staging → Dimensional Models (Data Marts) → Reports
```

**Pros:**
- ✅ Faster ROI (quick wins)
- ✅ Business-friendly
- ✅ Good for BI

**Cons:**
- ❌ Can lead to silos
- ❌ Harder to integrate later

---

**Inmon (Top-Down, Normalized DW):**
- Build enterprise data warehouse first (3NF)
- Then create data marts from DW

**Structure:**
```
Source Systems → Staging → Normalized DW (3NF) → Data Marts → Reports
```

**Pros:**
- ✅ Enterprise view
- ✅ Single source of truth
- ✅ Easier integration

**Cons:**
- ❌ Slower ROI
- ❌ Complex
- ❌ More upfront work

**Modern Approach:**
Hybrid - normalized core (data lake) + dimensional marts

---

**Q39-Q50: Advanced Modeling (condensed)**

**Q39: What is a Data Lake vs Data Warehouse?**
| Data Lake | Data Warehouse |
|-----------|----------------|
| Raw data, schema-on-read | Processed, schema-on-write |
| All formats | Structured |
| Cheap storage | Expensive |
| For data scientists | For business users |

**Q40: Lake house architecture?**
- Combine lake (storage) + warehouse (processing)
- Delta Lake, Iceberg, Hudi

**Q41: Medallion architecture (Bronze/Silver/Gold)?**
- Bronze: Raw data (lake)
- Silver: Cleaned, validated
- Gold: Business-ready (dimensions/facts)

**Q42: Type 4 SCD (Mini-dimensions)?**
- Split rapidly changing attributes into separate dimension
- Preserve history without bloating main dimension

**Q43: Type 6 SCD (Hybrid)?**
- Combines Type 1 + Type 2 + Type 3
- Track current + previous + full history

**Q44-Q50: Quick answers**
- **Q44:** Real-time vs batch dimensional models
- **Q45:** Snapshot fact tables (periodic snapshots)
- **Q46:** Accumulating snapshot fact tables (workflow tracking)
- **Q47:** Transaction fact tables (one row per event)
- **Q48:** Clickstream modeling
- **Q49:** Event sourcing vs dimensional modeling
- **Q50:** Graph databases for relationships

---

## PRACTICAL QUESTIONS (50 Questions)

### Design Problems (25 Questions)

**Q51: Design a data model for an e-commerce system**

**Answer:**

**Star Schema Design:**

**Fact Table: fact_orders**
```sql
CREATE TABLE fact_orders (
    -- Dimension keys
    order_date_key INT,
    customer_key INT,
    product_key INT,
    shipping_address_key INT,
    payment_method_key INT,

    -- Degenerate dimensions
    order_id VARCHAR(50),
    order_line_number INT,

    -- Measures
    quantity INT,
    unit_price DECIMAL(10,2),
    discount_amount DECIMAL(10,2),
    tax_amount DECIMAL(10,2),
    shipping_cost DECIMAL(10,2),
    total_amount DECIMAL(10,2),

    PRIMARY KEY (order_id, order_line_number)
);
```

**Dimension: dim_customer**
```sql
CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY,
    customer_id VARCHAR(50),
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    email VARCHAR(200),
    phone VARCHAR(20),
    registration_date DATE,
    customer_segment VARCHAR(50),  -- VIP, Regular, New
    loyalty_tier VARCHAR(20),
    -- SCD Type 2 columns
    effective_date DATE,
    expiration_date DATE,
    is_current BOOLEAN
);
```

**Dimension: dim_product**
```sql
CREATE TABLE dim_product (
    product_key INT PRIMARY KEY,
    product_id VARCHAR(50),
    product_name VARCHAR(200),
    sku VARCHAR(100),
    category VARCHAR(100),
    subcategory VARCHAR(100),
    brand VARCHAR(100),
    size VARCHAR(50),
    color VARCHAR(50),
    weight DECIMAL(10,2),
    cost_price DECIMAL(10,2),
    list_price DECIMAL(10,2),
    -- SCD Type 2
    effective_date DATE,
    expiration_date DATE,
    is_current BOOLEAN
);
```

**Dimension: dim_date**
```sql
CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,  -- YYYYMMDD
    full_date DATE,
    day_of_week INT,
    day_name VARCHAR(10),
    day_of_month INT,
    day_of_year INT,
    week_of_year INT,
    month INT,
    month_name VARCHAR(10),
    quarter INT,
    year INT,
    is_weekend BOOLEAN,
    is_holiday BOOLEAN,
    holiday_name VARCHAR(100),
    fiscal_year INT,
    fiscal_quarter INT
);
```

**Dimension: dim_shipping_address**
```sql
CREATE TABLE dim_shipping_address (
    address_key INT PRIMARY KEY,
    address_line1 VARCHAR(200),
    address_line2 VARCHAR(200),
    city VARCHAR(100),
    state VARCHAR(50),
    postal_code VARCHAR(20),
    country VARCHAR(100),
    latitude DECIMAL(10,6),
    longitude DECIMAL(10,6)
);
```

**Junk Dimension: dim_payment_method**
```sql
CREATE TABLE dim_payment_method (
    payment_key INT PRIMARY KEY,
    payment_type VARCHAR(50),  -- Credit, Debit, PayPal, etc.
    is_saved_card BOOLEAN,
    is_installment BOOLEAN
);
```

**Typical Queries:**
```sql
-- Sales by product category and month
SELECT
    d.month_name,
    d.year,
    p.category,
    SUM(f.total_amount) as revenue,
    SUM(f.quantity) as units_sold
FROM fact_orders f
JOIN dim_date d ON f.order_date_key = d.date_key
JOIN dim_product p ON f.product_key = p.product_key
WHERE d.year = 2024 AND p.is_current = TRUE
GROUP BY d.month_name, d.year, p.category;

-- Customer lifetime value
SELECT
    c.customer_id,
    c.customer_segment,
    COUNT(DISTINCT f.order_id) as order_count,
    SUM(f.total_amount) as lifetime_value
FROM fact_orders f
JOIN dim_customer c ON f.customer_key = c.customer_key
WHERE c.is_current = TRUE
GROUP BY c.customer_id, c.customer_segment;
```

---

**Q52-Q75: More Design Problems (condensed)**

Due to length, I'll provide condensed answers for remaining design questions:

**Q52: Design model for SaaS subscription business**
- Facts: fact_subscriptions (recurring), fact_usage (consumption)
- Dimensions: dim_customer, dim_plan, dim_date
- SCD Type 2 for plan changes

**Q53: Design model for healthcare/patient data**
- Facts: fact_visits, fact_procedures, fact_prescriptions
- Dimensions: dim_patient, dim_provider, dim_diagnosis, dim_medication
- HIPAA considerations: de-identification, audit logs

**Q54: Design model for social media analytics**
- Facts: fact_posts, fact_interactions (likes, comments, shares)
- Dimensions: dim_user, dim_content_type, dim_time (hourly grain)
- Bridge tables: post_hashtags, post_mentions

**Q55: Design model for financial transactions**
- Facts: fact_transactions (high volume, partitioned by date)
- Dimensions: dim_account, dim_transaction_type, dim_merchant
- Aggregate fact: fact_daily_balances

**Q56-Q75: Quick design scenarios**
- **Q56:** IoT sensor data model
- **Q57:** Video streaming analytics
- **Q58:** Supply chain / inventory
- **Q59:** Customer support tickets
- **Q60:** Marketing campaigns
- **Q61:** HR / employee data
- **Q62:** Education / student performance
- **Q63:** Ride-sharing (Uber-like)
- **Q64:** Gaming analytics
- **Q65:** Retail point-of-sale
- **Q66:** Insurance claims
- **Q67:** Real estate listings
- **Q68:** Hotel/booking system
- **Q69:** Telecommunications CDR
- **Q70:** Energy/utility usage
- **Q71:** Fraud detection model
- **Q72:** Recommendation system data
- **Q73:** A/B testing results model
- **Q74:** Ad tech (impressions, clicks, conversions)
- **Q75:** Multi-tenant SaaS model

---

### Implementation Questions (25 Questions)

**Q76: How do you load a dimension table with SCD Type 2?**

**Answer:**
Step-by-step ETL process:

**Step 1: Extract source data**
```sql
-- Source table
SELECT customer_id, name, email, city, state
FROM source_system.customers
WHERE updated_date >= last_load_date;
```

**Step 2: Identify changes**
```sql
-- Compare with current dimension
WITH source AS (
    SELECT customer_id, name, email, city, state
    FROM source_system.customers
),
current AS (
    SELECT customer_id, name, email, city, state
    FROM dim_customer
    WHERE is_current = TRUE
),
changed AS (
    SELECT s.*
    FROM source s
    LEFT JOIN current c USING (customer_id)
    WHERE c.customer_id IS NULL  -- New record
       OR s.city <> c.city       -- City changed
       OR s.state <> c.state     -- State changed
    -- Don't track name, email changes (Type 1 attributes)
)
SELECT * FROM changed;
```

**Step 3: Expire old records**
```sql
UPDATE dim_customer
SET end_date = CURRENT_DATE - 1,
    is_current = FALSE
WHERE customer_id IN (SELECT customer_id FROM changed)
  AND is_current = TRUE;
```

**Step 4: Insert new records**
```sql
INSERT INTO dim_customer (
    customer_id, name, email, city, state,
    start_date, end_date, is_current
)
SELECT
    customer_id, name, email, city, state,
    CURRENT_DATE as start_date,
    '9999-12-31' as end_date,
    TRUE as is_current
FROM changed;
```

**Step 5: Update Type 1 attributes** (if any)
```sql
UPDATE dim_customer d
SET name = s.name, email = s.email
FROM source_system.customers s
WHERE d.customer_id = s.customer_id
  AND d.is_current = TRUE;
```

---

**Q77-Q100: Implementation & Best Practices (condensed)**

**Q77: Surrogate key generation strategies?**
- Auto-increment (simple, but gaps possible)
- Sequence (database-level)
- Hash of natural key (deterministic, good for parallel loads)
- GUID (distributed systems)

**Q78: Handling late-arriving dimensions?**
```sql
-- Use default/placeholder dimension
-- Load fact with placeholder key
-- Update fact when dimension arrives (backfill)
```

**Q79: Fact table partitioning strategy?**
```sql
-- Partition by date (most common)
CREATE TABLE fact_sales PARTITION BY RANGE (order_date_key) (
    PARTITION p_2024_01 VALUES LESS THAN (20240201),
    PARTITION p_2024_02 VALUES LESS THAN (20240301),
    ...
);
```

**Q80: Slowly changing dimension performance optimization?**
- Bitmap indexes on is_current
- Partition by effective_date
- Materialized view for current records only

**Q81-Q100: Quick answers**
- **Q81:** Bulk vs incremental loads
- **Q82:** CDC (Change Data Capture) implementation
- **Q83:** Error handling in ETL
- **Q84:** Data quality checks
- **Q85:** Slowly changing fact (rare, but possible)
- **Q86:** Handling NULL foreign keys
- **Q87:** Aggregate tables maintenance
- **Q88:** Indexing strategies for facts vs dimensions
- **Q89:** Bitmap vs B-tree indexes
- **Q90:** Columnar storage for facts
- **Q91:** Compression techniques
- **Q92:** Handling currency/multi-currency
- **Q93:** Time zone handling
- **Q94:** Internationalization (i18n) considerations
- **Q95:** Data retention policies
- **Q96:** Archive vs delete
- **Q97:** Disaster recovery for data warehouse
- **Q98:** Backup strategies
- **Q99:** Testing dimensional models
- **Q100:** Documentation best practices

---

## Summary & Study Guide

**Core Concepts (Must Know):**
1. ✅ Star vs Snowflake schema
2. ✅ Fact vs Dimension tables
3. ✅ SCD Types (esp. Type 1 and 2)
4. ✅ Surrogate vs Natural keys
5. ✅ Normalization (1NF, 2NF, 3NF)
6. ✅ Kimball vs Inmon
7. ✅ Conformed dimensions
8. ✅ Fact table types (transaction, snapshot, accumulating)

**Practice:**
- Design 5 different business domains
- Implement SCD Type 2 ETL
- Write analytical queries on star schema

**Interview Tips:**
- Start with requirements gathering
- Define grain first
- Identify facts (measurable) vs dimensions (context)
- Discuss trade-offs (normalize vs denormalize)
- Consider SCD needs
- Think about query patterns

**Common Interview Question:**
"Design a data warehouse for [business domain]"

**Your Answer Structure:**
1. Understand business process
2. Identify facts (what we're measuring)
3. Identify dimensions (context)
4. Define grain
5. Design star schema
6. Discuss SCD strategy
7. Mention partitioning, indexing
8. Sample queries

Good luck! 🚀

---

**Q60: What is a bridge table?**

Resolves many-to-many relationships.

```sql
-- Students and Courses (many-to-many)
CREATE TABLE student_course_bridge (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    grade CHAR(2),
    PRIMARY KEY (student_id, course_id)
);
```

---

**Q61: Explain fact constellation schema (galaxy schema).**

Multiple fact tables sharing dimension tables.

Example: Sales fact + Inventory fact both use Product, Store, Time dimensions.

---

**Q62: What is a degenerate dimension?**

Dimension data stored in fact table (no separate dimension table).

Example: Order number, invoice number stored directly in fact.

---

**Q63: Explain snapshot fact tables.**

Capture state at regular intervals (daily balance, inventory levels).

```sql
CREATE TABLE account_balance_snapshot (
    account_id INT,
    date_key INT,
    balance DECIMAL(15,2),
    account_status VARCHAR(20),
    PRIMARY KEY (account_id, date_key)
);
```

---

**Q64: What is a role-playing dimension?**

Same dimension used multiple times in different roles.

Example: Date dimension used as Order Date, Ship Date, Delivery Date.

---

**Q65: Explain conformed dimensions.**

Shared dimensions across multiple fact tables/data marts. Ensures consistent reporting.

Example: Single Customer dimension used by Sales, Service, Marketing marts.

---

**Q66: What is data vault modeling?**

Flexible, audit-able model with Hubs, Links, Satellites.

- **Hubs:** Business keys
- **Links:** Relationships
- **Satellites:** Descriptive attributes with history

---

**Q67: Explain anchor modeling.**

Similar to data vault. Highly normalized, temporal.

Uses Anchors (entities), Attributes, Ties (relationships).

---

**Q68: What is a helper table (junk dimension)?**

Combines low-cardinality flags/indicators.

```sql
CREATE TABLE transaction_flags_dim (
    flag_key INT PRIMARY KEY,
    is_online BOOLEAN,
    is_refund BOOLEAN,
    payment_method VARCHAR(20)
);
```

---

**Q69: Explain surrogate keys vs natural keys.**

- **Surrogate:** System-generated (auto-increment, UUID)
- **Natural:** Business identifier (SSN, email)

**Recommendation:** Use surrogate keys in dimensions, natural keys for business logic.

---

**Q70: What is a mini-dimension?**

Rapidly changing attributes separated into smaller dimension.

Example: Customer age band (changes frequently) → separate mini-dimension.

---

**Q71: Explain type 0 SCD (retain original).**

Never changes. Original value preserved forever.

---

**Q72: What is a heterogeneous slowly changing dimension?**

Different attributes use different SCD types.

Example: Customer name (Type 2), Email (Type 1), Loyalty tier (Type 3).

---

**Q73: Explain late-arriving facts.**

Fact arrives after dimension snapshot.

**Solution:** Look up dimension based on fact's timestamp, not current date.

---

**Q74: What is a factless fact table?**

Records events with no measures (only dimensions).

Example: Student attendance (Student, Course, Date, Present=Yes).

---

**Q75: Explain dimension outriggers.**

Dimension references another dimension (snowflaking).

Example: City dimension → State dimension → Country dimension.

Generally avoid (prefer denormalization).

---

**Q76: What is the bus matrix?**

Documents which dimensions are used by which facts. Ensures conformed dimensions.

|  | Time | Product | Customer | Store |
|--|------|---------|----------|-------|
| Sales Fact | ✓ | ✓ | ✓ | ✓ |
| Inventory Fact | ✓ | ✓ | | ✓ |

---

**Q77: Explain temporal data modeling.**

Track valid time vs transaction time.

```sql
CREATE TABLE product_prices (
    product_id INT,
    price DECIMAL(10,2),
    valid_from DATE,
    valid_to DATE,
    transaction_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

**Q78: What is a shrunken dimension?**

Subset of a larger dimension (aggregated).

Example: Month dimension (subset of Day dimension).

---

**Q79: Explain transaction fact tables.**

Record individual transactions (most granular).

Example: Each sale, each click, each claim.

---

**Q80: What is a periodic snapshot fact table?**

Captures state at regular intervals.

Example: Daily inventory levels, monthly account balances.

---

**Q81: Explain accumulating snapshot fact tables.**

Track process with multiple milestones.

Example: Order lifecycle (Order Date, Pack Date, Ship Date, Deliver Date).

---

**Q82: What is a coverage table?**

Tracks time periods when conditions were true.

Example: Insurance coverage periods.

---

**Q83: Explain supertype and subtype entities.**

**Supertype:** General entity (Vehicle).
**Subtypes:** Specialized entities (Car, Truck, Motorcycle).

Use when entities share some attributes but have unique ones.

---

**Q84: What is a dependent dimension?**

Depends on another dimension for meaning.

Example: Product color only meaningful in context of Product.

---

**Q85: Explain data warehouse granularity trade-offs.**

**Fine grain (transaction-level):**
- ✅ Maximum flexibility
- ❌ Large storage, slow queries

**Coarse grain (aggregated):**
- ✅ Fast queries, less storage
- ❌ Less flexibility, can't drill down

**Recommendation:** Store transaction-level, create aggregate tables for performance.

---

**Q86: What is a multi-valued dimension?**

Dimension with multiple values for single fact.

Example: Claim with multiple diagnosis codes.

**Solution:** Bridge table or JSON array in modern databases.

---

**Q87: Explain dimension hierarchies.**

Natural drill-down paths.

Example: Product → Category → Department → Division.

---

**Q88: What is a step dimension?**

Tracks sequential steps in a process.

Example: Application status (Submitted → Reviewed → Approved → Funded).

---

**Q89: Explain schema-on-read vs schema-on-write.**

- **Schema-on-write:** Define schema before loading (traditional DW)
- **Schema-on-read:** Define schema when querying (data lake)

Data lake allows flexibility, DW ensures data quality.

---

**Q90: What is a composite key in dimensional modeling?**

Multiple columns form primary key (especially in bridge tables).

```sql
PRIMARY KEY (customer_id, product_id, transaction_date)
```

---

**Q91: Explain slowly changing dimensions in data lakes.**

Use Delta Lake, merge on business key:

```python
deltaTable.merge(updates, "target.customer_id = source.customer_id") \
    .whenMatchedUpdate(set = {...}) \
    .whenNotMatchedInsert(values = {...}) \
    .execute()
```

---

**Q92: What is a reference dimension?**

Small, static lookup dimension (status codes, country codes).

---

**Q93: Explain dimension conformity across environments.**

Same dimension definitions in dev, staging, prod.

**Achieved via:** Shared metadata repository, CI/CD deployment, dbt models.

---

**Q94: What is a multi-source dimension?**

Dimension populated from multiple source systems.

**Challenge:** Resolve conflicts, merge duplicates.

**Solution:** Master data management (MDM).

---

**Q95: Explain type 6 SCD (hybrid).**

Combination of Type 1 + Type 2 + Type 3.

```sql
CREATE TABLE customer_dim (
    customer_key INT,  -- Surrogate key
    customer_id INT,  -- Natural key
    name VARCHAR(100),  -- Type 1
    current_address VARCHAR(200),  -- Type 1
    original_address VARCHAR(200),  -- Type 3
    address_history TEXT,  -- Type 2 stored as JSON
    effective_date DATE,
    expiry_date DATE,
    is_current BOOLEAN
);
```

---

**Q96: What is a multi-tenant data model?**

Single model serves multiple customers/organizations.

```sql
CREATE TABLE claims (
    tenant_id INT,  -- Isolates data by tenant
    claim_id INT,
    amount DECIMAL(15,2),
    PRIMARY KEY (tenant_id, claim_id)
);

-- Row-level security
CREATE POLICY tenant_isolation ON claims
    USING (tenant_id = current_setting('app.current_tenant')::INT);
```

---

**Q97: Explain data modeling for real-time analytics.**

Use streaming-friendly models:
- Immutable append-only facts
- Precomputed aggregates
- Materialized views
- Change data capture (CDC)

---

**Q98: What is a semantic layer?**

Business-friendly abstraction over physical data model.

Tools: dbt metrics, Looker LookML, Tableau Semantic Layer.

---

**Q99: Explain data modeling for machine learning.**

Feature stores centralize ML features:
- **Entities:** Customer, Product
- **Features:** customer_lifetime_value, product_avg_rating
- **Timestamps:** Point-in-time correctness

Tools: Feast, Tecton, Databricks Feature Store.

---

**Q100: Tell me about your data modeling approach at Optum.**

**Healthcare claims data warehouse:**
- **Star schema** with 200+ dimension tables, 50+ fact tables
- **SCD Type 2** for Member, Provider, Benefit Plan dimensions
- **Transaction facts:** Claims, Prescriptions, Authorizations
- **Accumulating snapshot:** Claim lifecycle (Submitted → Adjudicated → Paid)
- **Conformed dimensions:** Shared Time, Geography, Product across all marts
- **Data vault** for staging layer (audit trail, flexibility)
- **Delta Lake** for data lake layer (ACID, time travel)

**Key decisions:**
- Grain: Individual claim line item (most granular)
- Partitioning: By claim year-month (query performance)
- Surrogate keys: Auto-increment integers
- Late-arriving facts: Backdate using claim received date

**Results:**
- 5TB data warehouse
- Query performance: 95th percentile < 10 seconds
- 500+ business users
- 99.9% data quality score

---


**Q94:** Multi-valued attributes? Use bridge table or JSON. **Q95:** Temporal modeling? Track valid_from/valid_to + transaction_time. **Q96:** Data vault vs Kimball? Data vault: flexible, auditable (Hubs/Links/Satellites). Kimball: dimensional, query-optimized (facts/dimensions). **Q97:** NoSQL data modeling? Denormalize, optimize for access patterns (e.g., Cosmos DB partition key). **Q98:** Graph data modeling? Nodes (entities) + Edges (relationships). Use Neo4j, Cosmos DB Gremlin. **Q99:** Data modeling for streaming? Append-only, immutable events. Use Kafka + ksqlDB or Flink. **Q100:** Data modeling anti-patterns? Over-normalization, no surrogate keys, missing indexes, ignoring SCD, poor grain definition.


---

**Q101: Data modeling review checklist?**
Verify: correct grain, surrogate keys, SCD type, indexes defined, partitioning strategy, naming conventions, documentation, test queries, performance validated.

