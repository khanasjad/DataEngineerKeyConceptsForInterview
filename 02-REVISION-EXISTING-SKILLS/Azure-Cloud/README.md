# Azure Cloud - Data Engineering Services

**Master Microsoft Azure data services for cloud data engineering interviews**

---

## 📚 What's in This Folder

### **1. 100-QUESTIONS.md**
Comprehensive Q&A covering:
- Azure data services (ADF, Synapse, Databricks, ADLS)
- Storage accounts and data lake architecture
- Data Factory pipelines and activities
- Synapse Analytics (SQL pools, Spark pools)
- Security and access control (RBAC, SAS, ACLs)
- Networking (VNets, private endpoints)
- Monitoring and cost optimization
- Real-world architecture patterns
- Optum Azure implementations

### **2. CHEATSHEET.md**
Quick reference with:
- One-line service definitions
- Common use cases
- Key configurations
- Security best practices
- Cost optimization tips
- Interview talking points

---

## 🎯 What You'll Learn

### **Core Azure Data Services**
- **Azure Data Factory (ADF):** ETL/ELT orchestration (cloud Airflow)
- **Azure Synapse Analytics:** Unified analytics (DW + Spark + pipelines)
- **Azure Databricks:** Managed Spark platform
- **Azure Data Lake Storage (ADLS Gen2):** Scalable data lake
- **Azure SQL Database:** Managed relational database
- **Azure Cosmos DB:** NoSQL database (multi-model)
- **Azure Event Hubs:** Event streaming (like Kafka)
- **Azure Stream Analytics:** Real-time stream processing

### **Advanced Concepts**
- **Data Lake Zones:** Raw → Bronze → Silver → Gold
- **Managed Identity:** Passwordless authentication
- **Private Endpoints:** Secure connectivity via VNet
- **PolyBase:** Query external data from SQL
- **Partition Strategy:** Optimize ADLS for performance
- **Delta Lake on Databricks:** ACID transactions
- **Synapse Serverless:** Query files without provisioning

### **Security & Governance**
- **RBAC:** Role-Based Access Control
- **ACLs:** File/folder-level permissions (ADLS)
- **SAS Tokens:** Shared Access Signatures
- **Key Vault:** Secrets management
- **Purview:** Data catalog and governance
- **Network Security:** VNets, firewalls, NSGs
- **Encryption:** At-rest and in-transit

---

## 💼 Why Azure Matters

### **Industry Adoption**
- **#2 cloud provider** (after AWS)
- **Enterprise focus:** Strong in Fortune 500
- **Hybrid cloud:** Azure Arc for on-prem integration
- **Microsoft ecosystem:** Tight integration with Office, Teams, Power BI

### **Key Use Cases**
- **Data Warehousing:** Synapse dedicated SQL pools
- **Data Lake:** ADLS Gen2 for centralized storage
- **ETL/ELT:** ADF for data movement and transformation
- **Big Data Processing:** Databricks for Spark workloads
- **Real-time Analytics:** Event Hubs + Stream Analytics
- **Machine Learning:** Azure ML, Databricks ML

### **Interview Focus**
1. Data Factory pipeline design
2. ADLS Gen2 architecture and partitioning
3. Synapse vs Databricks (when to use each)
4. Security best practices (RBAC, ACLs, managed identity)
5. Cost optimization strategies

---

## 🚀 Quick Start Guide

### **1. Read Cheatsheet (15 minutes)**
Review `CHEATSHEET.md` for service overview and terminology.

### **2. Study 100 Questions (4-5 hours)**
- **Q1-Q25:** Azure fundamentals (services, storage, networking)
- **Q26-Q50:** Data Factory (pipelines, activities, triggers)
- **Q51-Q75:** Synapse, Databricks, SQL Database
- **Q76-Q100:** Security, monitoring, cost optimization

### **3. Hands-On Practice**

**Free Azure Account:**
- Sign up: https://azure.microsoft.com/free/
- $200 credit for 30 days
- Free tier for many services

**Key Labs:**
1. Create storage account and ADLS Gen2
2. Build ADF pipeline (copy data)
3. Create Synapse workspace
4. Query ADLS files with Synapse serverless
5. Set up RBAC and managed identity

---

## 📖 Topic Coverage

### **Azure Data Lake Storage (ADLS Gen2)**

**Hierarchy:**
```
Storage Account
 └── Container
      └── Folder/
           └── Subfolder/
                └── file.parquet
```

**Medallion Architecture:**
```
ADLS Gen2
 ├── raw/              # Landing zone (immutable)
 ├── bronze/           # Raw data (minimal processing)
 ├── silver/           # Cleaned, conformed
 └── gold/             # Business-level aggregates
```

**Partitioning Strategy:**
```
gold/
 └── sales/
      └── year=2024/
           └── month=01/
                └── day=15/
                     └── data.parquet
```

**Key Features:**
- Hierarchical namespace (file system semantics)
- ACLs for fine-grained security
- Low cost ($0.018/GB/month)
- Optimized for analytics workloads

### **Azure Data Factory**

**Pipeline Components:**
- **Activities:** Units of work (Copy, Databricks, SQL)
- **Datasets:** Data references (source, sink)
- **Linked Services:** Connection info
- **Triggers:** Schedule, tumbling window, event-based
- **Integration Runtime:** Compute for execution

**Example Pipeline:**
```json
{
  "name": "CopyPipeline",
  "activities": [
    {
      "name": "CopyFromBlobToADLS",
      "type": "Copy",
      "inputs": [{"referenceName": "SourceDataset"}],
      "outputs": [{"referenceName": "SinkDataset"}],
      "source": {"type": "BlobSource"},
      "sink": {"type": "ParquetSink"}
    }
  ],
  "triggers": [
    {
      "name": "DailyTrigger",
      "type": "ScheduleTrigger",
      "recurrence": {
        "frequency": "Day",
        "interval": 1,
        "startTime": "2024-01-01T00:00:00Z"
      }
    }
  ]
}
```

**Common Activities:**
- Copy Data (data movement)
- Databricks Notebook (Spark processing)
- Stored Procedure (SQL execution)
- Web Activity (API calls)
- ForEach (iteration)
- If Condition (branching)

### **Azure Synapse Analytics**

**Components:**
1. **Dedicated SQL Pool:** Provisioned data warehouse (MPP)
2. **Serverless SQL Pool:** Query files on-demand (no infra)
3. **Spark Pool:** Managed Spark clusters
4. **Pipelines:** Same as ADF (integrated)

**Dedicated SQL Pool:**
```sql
-- Create external table (PolyBase)
CREATE EXTERNAL TABLE sales_external
WITH (
    LOCATION = '/gold/sales/',
    DATA_SOURCE = adls_datasource,
    FILE_FORMAT = parquet_format
)
AS SELECT * FROM sales;

-- Load data
COPY INTO sales
FROM 'https://storage.blob.core.windows.net/data/sales.csv'
WITH (FILE_TYPE = 'CSV', CREDENTIAL = (IDENTITY = 'Managed Identity'));
```

**Serverless SQL Pool:**
```sql
-- Query Parquet files directly
SELECT *
FROM OPENROWSET(
    BULK 'https://storage.dfs.core.windows.net/container/gold/sales/*.parquet',
    FORMAT = 'PARQUET'
) AS sales
WHERE year = 2024;
```

### **Azure Databricks**

**Key Features:**
- Managed Spark platform
- Delta Lake built-in
- Collaborative notebooks
- MLflow integration
- Unity Catalog (data governance)

**Example Notebook:**
```python
# Read from ADLS
df = spark.read.format("delta").load("/mnt/gold/sales")

# Transform
result = df.filter(df['amount'] > 1000) \
           .groupBy('region').sum('amount')

# Write as Delta
result.write.format("delta").mode("overwrite").save("/mnt/gold/sales_summary")

# Optimize
spark.sql("OPTIMIZE delta.`/mnt/gold/sales_summary`")
```

### **Security**

**RBAC (Role-Based Access Control):**
```
Subscription
 └── Resource Group
      └── Storage Account
           └── Container
```
- **Roles:** Owner, Contributor, Reader, custom roles
- **Scope:** Subscription, resource group, or resource
- **Principal:** User, group, service principal, managed identity

**ACLs (Access Control Lists):**
```bash
# Grant read/execute on folder
az storage fs access set \
  --account-name mystorageaccount \
  --file-system mycontainer \
  --path /gold/sales/ \
  --permissions r-x \
  --acl-user user@company.com
```

**Managed Identity:**
- System-assigned (tied to resource lifecycle)
- User-assigned (standalone, reusable)
- No credentials to manage
- Use for ADF, Databricks, Synapse

**Example (ADF using Managed Identity):**
```json
{
  "name": "ADLS_LinkedService",
  "type": "AzureBlobFS",
  "typeProperties": {
    "url": "https://mystorageaccount.dfs.core.windows.net"
  },
  "authentication": "ManagedIdentity"
}
```

---

## 🎓 Interview Preparation

### **Week 1: Fundamentals**
- Study Q1-Q30
- Understand core services (ADF, Synapse, ADLS)
- Practice creating storage accounts
- Learn RBAC basics

### **Week 2: Data Engineering**
- Study Q31-Q60
- Deep dive on ADF pipelines
- Practice Synapse SQL
- Understand Delta Lake on Databricks

### **Week 3: Security & Operations**
- Study Q61-Q85
- Master RBAC, ACLs, managed identity
- Learn monitoring (Log Analytics, metrics)
- Cost optimization strategies

### **Week 4: Real-world Scenarios**
- Study Q86-Q100
- Design end-to-end architectures
- Practice explaining trade-offs
- Review Optum use cases

---

## 💡 Common Interview Questions

### **Conceptual**

**Q: "ADLS Gen1 vs Gen2?"**
- **Gen2:** Hierarchical namespace, better performance, lower cost
- **Gen2:** POSIX ACLs (file/folder permissions)
- **Gen2:** Preferred for new projects
- **Gen1:** Legacy, being retired

**Q: "Synapse vs Databricks?"**
- **Synapse:** All-in-one (SQL DW + Spark + Pipelines)
  - Use for: SQL-heavy workloads, integrated pipelines
- **Databricks:** Best-in-class Spark, ML focus
  - Use for: Complex Spark jobs, ML workflows, Delta Lake

**Q: "Dedicated vs Serverless SQL Pool?"**
- **Dedicated:** Provisioned, predictable performance, higher cost
  - Use for: Production DW, consistent workloads
- **Serverless:** Pay-per-query, no provisioning, variable latency
  - Use for: Ad-hoc queries, exploration, cost-sensitive workloads

**Q: "Managed Identity vs Service Principal?"**
- **Managed Identity:** No credentials, Azure manages lifecycle
  - Preferred for Azure resources
- **Service Principal:** Manual credential management
  - Use for: Non-Azure apps, legacy systems

### **Scenario Questions**

**Q: "Design data lake architecture for retail company"**
```
Architecture:
1. Landing (raw/)
   - CSV, JSON from source systems
   - Retention: 7 days

2. Bronze (bronze/)
   - Raw data in Parquet
   - No transformations
   - Retention: 90 days

3. Silver (silver/)
   - Cleaned, validated data
   - Delta Lake format
   - Retention: 2 years

4. Gold (gold/)
   - Business aggregates
   - Partitioned by date
   - Retention: Indefinite

Data Factory:
- Copy from on-prem to raw/
- Databricks job: raw → bronze → silver → gold
- Schedule: Daily at 2 AM
```

**Q: "Secure access to ADLS from ADF?"**
```
Best Practice: Managed Identity
1. Enable system-assigned identity on ADF
2. Grant "Storage Blob Data Contributor" role to ADF identity
3. Create linked service using Managed Identity auth
4. No credentials stored, Azure manages access

Alternative: Service Principal (if managed identity not possible)
- Create service principal in Azure AD
- Store credentials in Key Vault
- Reference Key Vault secret in linked service
```

**Q: "Optimize costs for Synapse Analytics?"**
```
Strategies:
1. Use serverless SQL for ad-hoc queries
2. Pause dedicated SQL pool when not in use
3. Right-size DWU (start small, scale up)
4. Use Parquet/Delta (columnar, compressed)
5. Partition data (skip unnecessary scans)
6. Set up auto-pause for dev environments
7. Monitor query performance, optimize slow queries
8. Use result set caching for repeated queries
```

---

## 🔗 Resources

### **Official Docs**
- [Azure Data Factory](https://docs.microsoft.com/azure/data-factory/)
- [Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics/)
- [Azure Databricks](https://docs.microsoft.com/azure/databricks/)
- [ADLS Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-introduction)

### **Learning**
- [Microsoft Learn](https://docs.microsoft.com/learn/) (free courses)
- [DP-203 Exam](https://docs.microsoft.com/certifications/exams/dp-203) (Data Engineering on Azure)
- [Azure Architecture Center](https://docs.microsoft.com/azure/architecture/)

### **Practice**
- Azure Free Account ($200 credit)
- Hands-on labs on Microsoft Learn
- Sandbox environments

---

## 🎯 Key Takeaways

### **Must Know**
✅ Core services (ADF, Synapse, ADLS, Databricks)
✅ Data lake architecture (raw/bronze/silver/gold)
✅ RBAC vs ACLs
✅ Managed identity for authentication
✅ ADF pipeline design

### **Should Know**
✅ Synapse dedicated vs serverless
✅ Partitioning strategies
✅ Private endpoints and VNets
✅ Cost optimization techniques
✅ Monitoring and alerting

### **Nice to Know**
✅ Azure Purview for governance
✅ Event Hubs for streaming
✅ Cosmos DB for NoSQL
✅ Azure ML integration
✅ Hybrid cloud with Azure Arc

### **Red Flags**
❌ Storing credentials in code (use Key Vault)
❌ No partitioning strategy (impacts performance and cost)
❌ Public access to storage (security risk)
❌ Not using managed identity (credential management burden)
❌ Over-provisioning resources (wasted cost)

---

## 📝 Architecture Patterns

### **Modern Data Warehouse**
```
Sources → ADF → ADLS (staging) → Databricks (transform) →
Synapse Dedicated SQL (DW) → Power BI (reporting)
```

### **Lambda Architecture (Batch + Streaming)**
```
Batch:
  Source → ADF → ADLS → Databricks → Delta Lake → Synapse

Real-time:
  Event Hubs → Stream Analytics → ADLS → Delta Lake → Synapse
```

### **Lakehouse Architecture**
```
Sources → ADF → ADLS (Delta Lake) → Databricks/Synapse Serverless → Power BI
```

---

## 💼 Interview Practice Scenarios

### **Scenario 1: Daily ETL Pipeline**
Design ADF pipeline:
- Source: On-prem SQL Server
- Process: Incremental load (watermark)
- Transform: Databricks (clean, aggregate)
- Sink: Synapse SQL Pool
- Schedule: Daily at 2 AM
- Monitoring: Email on failure

### **Scenario 2: Security Setup**
Secure data lake:
- ADLS Gen2 with private endpoint
- RBAC for coarse-grained access
- ACLs for file-level permissions
- Managed identity for ADF, Databricks
- Key Vault for secrets
- Network isolation via VNet

### **Scenario 3: Cost Optimization**
Reduce monthly bill:
- Pause Synapse pool nights/weekends
- Use serverless for exploration
- Compress data (Parquet/Delta)
- Delete old data (lifecycle management)
- Right-size Databricks clusters
- Monitor with Cost Management

---

**Master Azure and build cloud-native data platforms! ☁️**

*For detailed answers and examples, see 100-QUESTIONS.md*
*For quick review before interviews, see CHEATSHEET.md*
