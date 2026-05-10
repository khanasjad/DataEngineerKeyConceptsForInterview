# Azure Data Factory & Synapse Analytics - Complete Guide

**Master ADF and Synapse for Azure Data Engineering**

---

## 📋 TABLE OF CONTENTS

### PART 1: AZURE DATA FACTORY
1. [ADF Overview & Architecture](#adf-overview--architecture)
2. [ADF Core Components](#adf-core-components)
3. [ADF Activities Deep Dive](#adf-activities-deep-dive)
4. [ADF Triggers & Scheduling](#adf-triggers--scheduling)
5. [ADF Integration Runtime](#adf-integration-runtime)
6. [ADF Pipelines & Control Flow](#adf-pipelines--control-flow)
7. [ADF Hands-On Examples](#adf-hands-on-examples)

### PART 2: AZURE SYNAPSE ANALYTICS
8. [Synapse Overview & Architecture](#synapse-overview--architecture)
9. [Synapse Dedicated SQL Pools](#synapse-dedicated-sql-pools)
10. [Synapse Serverless SQL Pools](#synapse-serverless-sql-pools)
11. [Synapse Spark Pools](#synapse-spark-pools)
12. [Synapse Pipelines](#synapse-pipelines)
13. [Synapse Integration](#synapse-integration)
14. [Synapse Hands-On Examples](#synapse-hands-on-examples)

### PART 3: COMPARISON & SCENARIOS
15. [ADF vs Synapse - When to Use Which](#adf-vs-synapse---when-to-use-which)
16. [Real-World Architecture Patterns](#real-world-architecture-patterns)
17. [Interview Questions](#interview-questions)
18. [Best Practices](#best-practices)

---

# PART 1: AZURE DATA FACTORY

## ADF Overview & Architecture

### **What is Azure Data Factory?**

**Azure Data Factory (ADF)** is a cloud-based ETL (Extract, Transform, Load) and data integration service.

**Think of it as:**
- **Cloud Airflow:** Orchestrates data workflows
- **ETL Tool:** Moves and transforms data between systems
- **Data Hub:** Connects 90+ data sources

**Key Capabilities:**
- Data movement (copy data between sources)
- Data transformation (via Databricks, Spark, SQL)
- Workflow orchestration (schedule and monitor pipelines)
- Hybrid integration (on-prem and cloud)

---

### **ADF Architecture**

```
┌─────────────────────────────────────────────────────────────────┐
│                     Azure Data Factory                          │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                   AUTHOR & MONITOR                        │  │
│  │  (ADF Studio - Web UI for building pipelines)            │  │
│  └──────────────────────────────────────────────────────────┘  │
│                            │                                     │
│  ┌─────────────────────────┼─────────────────────────────────┐ │
│  │         CONTROL PLANE (Azure Region)                      │ │
│  │  ┌───────────────┐  ┌──────────────┐  ┌───────────────┐ │ │
│  │  │   Triggers    │  │  Orchestrator│  │   Metadata    │ │ │
│  │  │   (Schedule)  │  │  (Execute    │  │   Store       │ │ │
│  │  │               │  │   Pipelines) │  │               │ │ │
│  │  └───────────────┘  └──────────────┘  └───────────────┘ │ │
│  └─────────────────────────────────────────────────────────┘ │
│                            │                                     │
│  ┌─────────────────────────┼─────────────────────────────────┐ │
│  │         DATA PLANE (Integration Runtime)                  │ │
│  │                                                            │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │ │
│  │  │   Azure IR   │  │  Self-hosted │  │  Azure-SSIS  │   │ │
│  │  │  (Cloud-to-  │  │     IR       │  │     IR       │   │ │
│  │  │   Cloud)     │  │  (On-prem    │  │  (SSIS       │   │ │
│  │  │              │  │   to Cloud)  │  │   Packages)  │   │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │ │
│  └─────────────────────────────────────────────────────────┘ │
│                            │                                     │
│  ┌─────────────────────────▼─────────────────────────────────┐ │
│  │              DATA SOURCES & DESTINATIONS                   │ │
│  │  [Azure Blob] [SQL DB] [Snowflake] [S3] [On-Prem SQL]    │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

**Components:**
1. **Control Plane (Management):**
   - Pipeline definitions
   - Triggers and schedules
   - Metadata and monitoring

2. **Data Plane (Execution):**
   - Integration Runtime (where code runs)
   - Actual data movement and transformation

3. **Authoring:**
   - ADF Studio (web UI)
   - ARM templates (JSON)
   - Git integration (CI/CD)

---

## ADF Core Components

### **1. Pipelines**

**Definition:** A logical grouping of activities that perform a task together.

**Think of it as:**
- Workflow definition
- Like a DAG in Airflow
- Container for activities

**Example Pipeline Structure:**
```json
{
  "name": "DataIngestionPipeline",
  "properties": {
    "activities": [
      {
        "name": "CopyFromBlobToSQL",
        "type": "Copy"
      },
      {
        "name": "ExecuteStoredProcedure",
        "type": "SqlServerStoredProcedure",
        "dependsOn": [
          {
            "activity": "CopyFromBlobToSQL",
            "dependencyConditions": ["Succeeded"]
          }
        ]
      }
    ]
  }
}
```

**Pipeline Properties:**
- **Activities:** Tasks to execute
- **Parameters:** Runtime inputs
- **Variables:** Internal state
- **Annotations:** Tags and metadata

---

### **2. Activities**

**Definition:** Processing steps within a pipeline.

**Three Types:**

#### **A. Data Movement Activities**
- **Copy Activity:** Move data between sources
- Most commonly used activity

**Example:**
```json
{
  "name": "CopyBlobToSQL",
  "type": "Copy",
  "inputs": [{"referenceName": "BlobDataset"}],
  "outputs": [{"referenceName": "SQLDataset"}],
  "typeProperties": {
    "source": {
      "type": "BlobSource"
    },
    "sink": {
      "type": "SqlSink",
      "writeBatchSize": 10000
    },
    "enableStaging": false,
    "parallelCopies": 4
  }
}
```

#### **B. Data Transformation Activities**
- **Databricks Notebook:** Run Spark jobs
- **HDInsight Spark:** Run Spark on HDInsight
- **Stored Procedure:** Execute SQL stored procedures
- **Data Flow:** Visual data transformation (low-code)

**Example:**
```json
{
  "name": "TransformDataWithDatabricks",
  "type": "DatabricksNotebook",
  "linkedServiceName": {
    "referenceName": "DatabricksLinkedService",
    "type": "LinkedServiceReference"
  },
  "typeProperties": {
    "notebookPath": "/Shared/ETL/transform_data",
    "baseParameters": {
      "input_path": "/mnt/raw/data",
      "output_path": "/mnt/processed/data"
    }
  }
}
```

#### **C. Control Flow Activities**
- **If Condition:** Branching logic
- **ForEach:** Iteration over items
- **Until:** Loop until condition met
- **Wait:** Delay execution
- **Execute Pipeline:** Call another pipeline
- **Web Activity:** Call REST APIs
- **Get Metadata:** Retrieve file/dataset info

**Example - ForEach:**
```json
{
  "name": "ForEachFile",
  "type": "ForEach",
  "typeProperties": {
    "items": {
      "value": "@pipeline().parameters.FileList",
      "type": "Expression"
    },
    "isSequential": false,
    "batchCount": 10,
    "activities": [
      {
        "name": "CopyFile",
        "type": "Copy"
      }
    ]
  }
}
```

---

### **3. Datasets**

**Definition:** Named reference to data you want to use in activities.

**Think of it as:**
- Pointer to data location
- Schema definition
- Like a table reference

**Example - CSV Dataset:**
```json
{
  "name": "CsvDataset",
  "properties": {
    "linkedServiceName": {
      "referenceName": "AzureBlobStorage",
      "type": "LinkedServiceReference"
    },
    "type": "DelimitedText",
    "typeProperties": {
      "location": {
        "type": "AzureBlobStorageLocation",
        "container": "data",
        "folderPath": "input",
        "fileName": "sales.csv"
      },
      "columnDelimiter": ",",
      "firstRowAsHeader": true
    },
    "schema": [
      {"name": "OrderID", "type": "String"},
      {"name": "Amount", "type": "Decimal"}
    ]
  }
}
```

**Parameterized Dataset:**
```json
{
  "name": "ParameterizedCsvDataset",
  "properties": {
    "parameters": {
      "FileName": {"type": "String"}
    },
    "typeProperties": {
      "location": {
        "fileName": {
          "value": "@dataset().FileName",
          "type": "Expression"
        }
      }
    }
  }
}
```

---

### **4. Linked Services**

**Definition:** Connection strings that define how to connect to data sources.

**Think of it as:**
- Database connection
- Like JDBC URL
- Credentials and endpoints

**Example - Azure SQL Database:**
```json
{
  "name": "AzureSqlDatabase",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:myserver.database.windows.net,1433;Database=mydb;",
      "authenticationType": "ManagedIdentity"
    }
  }
}
```

**Example - Azure Blob Storage:**
```json
{
  "name": "AzureBlobStorage",
  "properties": {
    "type": "AzureBlobStorage",
    "typeProperties": {
      "serviceEndpoint": "https://mystorageaccount.blob.core.windows.net/",
      "authenticationType": "ManagedIdentity"
    }
  }
}
```

**Common Linked Services:**
- Azure SQL Database
- Azure Blob Storage / ADLS Gen2
- Azure Databricks
- Snowflake
- AWS S3
- On-premises SQL Server (via Self-hosted IR)
- REST API (via Web linked service)

---

### **5. Triggers**

**Definition:** Determines when pipeline execution should be kicked off.

**Three Types:**

#### **A. Schedule Trigger**
Runs on a time schedule (cron-like).

```json
{
  "name": "DailyTrigger",
  "properties": {
    "type": "ScheduleTrigger",
    "typeProperties": {
      "recurrence": {
        "frequency": "Day",
        "interval": 1,
        "startTime": "2024-01-01T00:00:00Z",
        "timeZone": "UTC",
        "schedule": {
          "hours": [2],
          "minutes": [0]
        }
      }
    },
    "pipelines": [
      {
        "pipelineReference": {
          "referenceName": "MyPipeline",
          "type": "PipelineReference"
        }
      }
    ]
  }
}
```

**Recurrence Patterns:**
- Minute, Hour, Day, Week, Month
- Cron-like expressions
- Time zone aware

#### **B. Tumbling Window Trigger**
Runs on fixed-size, non-overlapping time intervals.

```json
{
  "name": "HourlyTumblingWindow",
  "properties": {
    "type": "TumblingWindowTrigger",
    "typeProperties": {
      "frequency": "Hour",
      "interval": 1,
      "startTime": "2024-01-01T00:00:00Z",
      "delay": "00:15:00",
      "maxConcurrency": 5,
      "retryPolicy": {
        "count": 3,
        "intervalInSeconds": 300
      }
    }
  }
}
```

**Use Cases:**
- Processing hourly data files
- Backfilling historical data
- Each window processes specific time range

**Key Features:**
- Dependency on previous windows
- Automatic backfill
- Self-healing (retry failed windows)

#### **C. Event-Based Trigger**
Triggered by events (e.g., blob created).

```json
{
  "name": "BlobEventTrigger",
  "properties": {
    "type": "BlobEventsTrigger",
    "typeProperties": {
      "blobPathBeginsWith": "/data/input/",
      "blobPathEndsWith": ".csv",
      "ignoreEmptyBlobs": true,
      "events": ["Microsoft.Storage.BlobCreated"]
    },
    "pipelines": [
      {
        "pipelineReference": {
          "referenceName": "ProcessNewFile",
          "type": "PipelineReference"
        },
        "parameters": {
          "FileName": "@trigger().outputs.body.fileName"
        }
      }
    ]
  }
}
```

**Use Cases:**
- Process files as they arrive
- Event-driven architectures
- Real-time data ingestion

---

### **6. Integration Runtime (IR)**

**Definition:** The compute infrastructure used by ADF to execute activities.

**Think of it as:**
- Where your code runs
- Like Spark executors
- Bridge between cloud and on-prem

**Three Types:**

#### **A. Azure Integration Runtime (Default)**
- Cloud-to-cloud data movement
- Serverless, managed by Azure
- Auto-scales based on workload

**Use When:**
- Source and destination are both in Azure
- Cloud-to-cloud scenarios

**Regions:**
- Auto-resolve (ADF picks closest region)
- Specific region (for data residency)

#### **B. Self-Hosted Integration Runtime**
- On-premises or private network data access
- You install and manage on your VM/server
- Can connect to private endpoints

**Use When:**
- Source or destination is on-premises
- Private network connectivity required
- Data behind firewall

**Setup:**
```powershell
# Download and install IR on your server
# Register with ADF using authentication key

# Install
.\InstallGatewayOnLocalMachine.ps1

# Connect to ADF
.\RegisterIntegrationRuntime.ps1 -gatewayKey "<your-key>"
```

#### **C. Azure-SSIS Integration Runtime**
- Run SSIS (SQL Server Integration Services) packages in cloud
- Managed SSIS cluster

**Use When:**
- Migrating SSIS workloads to cloud
- Running existing SSIS packages

---

## ADF Activities Deep Dive

### **Copy Activity - Deep Dive**

**Most Important Activity in ADF**

**Capabilities:**
- Move data between 90+ connectors
- Schema mapping
- Data type conversion
- Performance optimization (parallel copies, staging)

**Full Example:**
```json
{
  "name": "CopyActivity",
  "type": "Copy",
  "inputs": [{"referenceName": "SourceDataset"}],
  "outputs": [{"referenceName": "SinkDataset"}],
  "typeProperties": {
    "source": {
      "type": "BlobSource",
      "recursive": true,
      "wildcardFileName": "*.csv"
    },
    "sink": {
      "type": "SqlSink",
      "writeBatchSize": 10000,
      "writeBatchTimeout": "00:30:00",
      "preCopyScript": "TRUNCATE TABLE staging.sales",
      "sqlWriterStoredProcedureName": "usp_load_sales",
      "tableOption": "autoCreate"
    },
    "enableStaging": true,
    "stagingSettings": {
      "linkedServiceName": {
        "referenceName": "StagingBlobStorage"
      },
      "path": "staging"
    },
    "parallelCopies": 4,
    "dataIntegrationUnits": 4,
    "translator": {
      "type": "TabularTranslator",
      "mappings": [
        {"source": {"name": "OrderID"}, "sink": {"name": "order_id"}},
        {"source": {"name": "Amount"}, "sink": {"name": "amount"}}
      ]
    }
  }
}
```

**Performance Tuning:**

1. **Parallel Copies:**
   - Split large dataset into chunks
   - Process multiple chunks simultaneously
   - `parallelCopies`: 4-32 (depending on data size)

2. **Data Integration Units (DIU):**
   - Compute power for copy operation
   - 1 DIU = 1 vCore + 4 GB memory
   - Auto-scale: 4-256 DIUs
   - `dataIntegrationUnits`: 4 (default), up to 256

3. **Staging:**
   - Use intermediate Blob storage for large copies
   - Compress data during transfer
   - Improves performance for cross-region copies

4. **Partitioning:**
   - Physical partitions (for partitioned sources)
   - Dynamic range partitioning
   - `partitionOption`: "PhysicalPartitionsOfTable" or "DynamicRange"

**Copy Methods:**

| Source | Sink | Method |
|--------|------|--------|
| Azure Blob | Azure SQL | Direct copy |
| On-prem SQL | Azure SQL | Via Self-hosted IR |
| S3 | ADLS Gen2 | Via Azure IR |
| Snowflake | Synapse | Via staging (PolyBase) |

---

### **Data Flow Activity**

**Visual, low-code data transformation.**

**Components:**
1. **Source:** Input data
2. **Transformations:** Filter, Join, Aggregate, Pivot, etc.
3. **Sink:** Output destination

**Example - Data Flow Definition:**
```json
{
  "name": "CustomerDataFlow",
  "properties": {
    "type": "MappingDataFlow",
    "typeProperties": {
      "sources": [
        {
          "name": "Customers",
          "dataset": {"referenceName": "CustomerDataset"}
        },
        {
          "name": "Orders",
          "dataset": {"referenceName": "OrderDataset"}
        }
      ],
      "transformations": [
        {
          "name": "JoinCustomersOrders",
          "type": "Join",
          "inputs": ["Customers", "Orders"],
          "joinType": "inner",
          "leftColumns": ["CustomerID"],
          "rightColumns": ["CustomerID"]
        },
        {
          "name": "AggregateByCustomer",
          "type": "Aggregate",
          "input": "JoinCustomersOrders",
          "groupBy": ["CustomerID", "CustomerName"],
          "aggregates": [
            {"column": "OrderCount", "expression": "count()"},
            {"column": "TotalAmount", "expression": "sum(Amount)"}
          ]
        }
      ],
      "sinks": [
        {
          "name": "OutputSQL",
          "dataset": {"referenceName": "SQLDataset"}
        }
      ]
    }
  }
}
```

**Common Transformations:**
- **Filter:** WHERE clause
- **Select:** Column selection, rename
- **Join:** Inner, left, right, full
- **Aggregate:** GROUP BY operations
- **Pivot/Unpivot:** Reshape data
- **Derived Column:** Add calculated columns
- **Lookup:** Enrich data from reference table
- **Conditional Split:** Split stream based on conditions

**When to Use:**
- Simple transformations (SQL-like logic)
- No coding required
- Visual debugging

**When NOT to Use:**
- Complex transformations (use Databricks)
- Custom business logic (use Python/Scala)

---

### **Web Activity**

**Call external REST APIs.**

**Example:**
```json
{
  "name": "CallExternalAPI",
  "type": "WebActivity",
  "typeProperties": {
    "url": "https://api.example.com/data",
    "method": "POST",
    "headers": {
      "Content-Type": "application/json",
      "Authorization": "@concat('Bearer ', variables('AccessToken'))"
    },
    "body": {
      "date": "@pipeline().parameters.ProcessDate",
      "type": "incremental"
    },
    "authentication": {
      "type": "ManagedIdentity",
      "resource": "https://api.example.com"
    }
  },
  "outputs": [
    {
      "referenceName": "OutputDataset"
    }
  ]
}
```

**Use Cases:**
- Call custom APIs
- Trigger external workflows
- Get access tokens
- Send notifications (Slack, Teams)

---

## ADF Pipelines & Control Flow

### **Control Flow Patterns**

#### **1. If Condition**

**Branching logic based on condition.**

```json
{
  "name": "IfConditionActivity",
  "type": "IfCondition",
  "typeProperties": {
    "expression": {
      "value": "@greater(pipeline().parameters.FileCount, 0)",
      "type": "Expression"
    },
    "ifTrueActivities": [
      {
        "name": "ProcessFiles",
        "type": "Copy"
      }
    ],
    "ifFalseActivities": [
      {
        "name": "SendNoFilesAlert",
        "type": "WebActivity"
      }
    ]
  }
}
```

#### **2. ForEach Loop**

**Iterate over collection.**

```json
{
  "name": "ForEachFile",
  "type": "ForEach",
  "typeProperties": {
    "items": {
      "value": "@pipeline().parameters.FileList",
      "type": "Expression"
    },
    "isSequential": false,
    "batchCount": 20,
    "activities": [
      {
        "name": "CopyFile",
        "type": "Copy",
        "inputs": [
          {
            "referenceName": "SourceDataset",
            "parameters": {
              "FileName": "@item()"
            }
          }
        ]
      }
    ]
  }
}
```

**Parallel vs Sequential:**
- `isSequential: false` - Run in parallel (default 20 concurrent)
- `isSequential: true` - Run one at a time
- `batchCount` - Max parallel executions

#### **3. Until Loop**

**Repeat until condition is true.**

```json
{
  "name": "UntilActivity",
  "type": "Until",
  "typeProperties": {
    "expression": {
      "value": "@equals(variables('Status'), 'Completed')",
      "type": "Expression"
    },
    "timeout": "00:15:00",
    "activities": [
      {
        "name": "CheckStatus",
        "type": "WebActivity"
      },
      {
        "name": "Wait",
        "type": "Wait",
        "typeProperties": {
          "waitTimeInSeconds": 30
        }
      }
    ]
  }
}
```

**Use Cases:**
- Polling external system status
- Waiting for file availability
- Retry with backoff

#### **4. Execute Pipeline**

**Call another pipeline (modular design).**

```json
{
  "name": "ExecuteChildPipeline",
  "type": "ExecutePipeline",
  "typeProperties": {
    "pipeline": {
      "referenceName": "ChildPipeline",
      "type": "PipelineReference"
    },
    "parameters": {
      "InputPath": "@pipeline().parameters.DataPath",
      "ProcessDate": "@formatDateTime(utcnow(), 'yyyy-MM-dd')"
    },
    "waitOnCompletion": true
  }
}
```

**Patterns:**
- Parent-child pipelines
- Reusable modules
- Separation of concerns

---

### **Variables and Parameters**

**Parameters (Input):**
```json
{
  "name": "MyPipeline",
  "properties": {
    "parameters": {
      "SourcePath": {"type": "String"},
      "TargetPath": {"type": "String"},
      "ProcessDate": {"type": "String", "defaultValue": "@utcnow()"}
    }
  }
}
```

**Variables (Internal State):**
```json
{
  "name": "MyPipeline",
  "properties": {
    "variables": {
      "FileCount": {"type": "Integer", "defaultValue": 0},
      "Status": {"type": "String", "defaultValue": "NotStarted"}
    },
    "activities": [
      {
        "name": "SetVariable",
        "type": "SetVariable",
        "typeProperties": {
          "variableName": "FileCount",
          "value": {
            "value": "@activity('GetMetadata').output.childItems.length",
            "type": "Expression"
          }
        }
      }
    ]
  }
}
```

---

### **Expressions and Functions**

**Common Functions:**

```javascript
// String functions
@concat('prefix_', pipeline().parameters.Name)
@substring(pipeline().parameters.FileName, 0, 10)
@replace(pipeline().parameters.Path, 'old', 'new')

// Date functions
@utcnow()
@formatDateTime(utcnow(), 'yyyy-MM-dd')
@addDays(utcnow(), -7)

// Logical functions
@if(greater(pipeline().parameters.Count, 0), 'Yes', 'No')
@equals(variables('Status'), 'Completed')
@and(condition1, condition2)

// Array functions
@length(pipeline().parameters.FileList)
@first(pipeline().parameters.FileList)
@item()  // In ForEach loop

// Pipeline functions
@pipeline().RunId
@pipeline().TriggerTime
@pipeline().parameters.ParamName

// Activity outputs
@activity('CopyActivity').output.rowsCopied
@activity('LookupActivity').output.firstRow.column_name
```

---

## ADF Triggers & Scheduling

### **Schedule Trigger - Advanced**

**Daily at 2 AM:**
```json
{
  "recurrence": {
    "frequency": "Day",
    "interval": 1,
    "schedule": {
      "hours": [2],
      "minutes": [0]
    },
    "timeZone": "Eastern Standard Time"
  }
}
```

**Every Monday at 9 AM:**
```json
{
  "recurrence": {
    "frequency": "Week",
    "interval": 1,
    "schedule": {
      "weekDays": ["Monday"],
      "hours": [9],
      "minutes": [0]
    }
  }
}
```

**First day of month:**
```json
{
  "recurrence": {
    "frequency": "Month",
    "interval": 1,
    "schedule": {
      "monthDays": [1],
      "hours": [0],
      "minutes": [0]
    }
  }
}
```

---

### **Tumbling Window - Advanced**

**Hourly windows with dependency:**
```json
{
  "name": "HourlyProcessing",
  "properties": {
    "type": "TumblingWindowTrigger",
    "typeProperties": {
      "frequency": "Hour",
      "interval": 1,
      "startTime": "2024-01-01T00:00:00Z",
      "endTime": "2024-12-31T23:59:59Z",
      "delay": "00:15:00",
      "maxConcurrency": 5,
      "retryPolicy": {
        "count": 3,
        "intervalInSeconds": 300
      },
      "dependsOn": [
        {
          "type": "TumblingWindowTriggerDependencyReference",
          "referenceName": "PreviousWindowTrigger",
          "offset": "-01:00:00"
        }
      ]
    },
    "pipeline": {
      "pipelineReference": {
        "referenceName": "ProcessData",
        "type": "PipelineReference"
      },
      "parameters": {
        "WindowStart": "@trigger().outputs.windowStartTime",
        "WindowEnd": "@trigger().outputs.windowEndTime"
      }
    }
  }
}
```

**Window Variables:**
- `@trigger().outputs.windowStartTime` - Start of current window
- `@trigger().outputs.windowEndTime` - End of current window

**Use Cases:**
- Process hourly data files
- Incremental data loads
- Time-series processing

---

## ADF Hands-On Examples

### **Example 1: Simple Copy Pipeline**

**Scenario:** Copy CSV from Blob to SQL Database

**Step 1: Create Linked Services**

```json
// Blob Storage Linked Service
{
  "name": "AzureBlobStorage",
  "type": "AzureBlobStorage",
  "typeProperties": {
    "connectionString": "DefaultEndpointsProtocol=https;AccountName=mystorageaccount;...",
    "authenticationType": "AccountKey"
  }
}

// SQL Database Linked Service
{
  "name": "AzureSqlDatabase",
  "type": "AzureSqlDatabase",
  "typeProperties": {
    "connectionString": "Server=tcp:myserver.database.windows.net,1433;Database=mydb;",
    "authenticationType": "ManagedIdentity"
  }
}
```

**Step 2: Create Datasets**

```json
// Source Dataset (CSV in Blob)
{
  "name": "BlobCsvDataset",
  "properties": {
    "linkedServiceName": {"referenceName": "AzureBlobStorage"},
    "type": "DelimitedText",
    "typeProperties": {
      "location": {
        "type": "AzureBlobStorageLocation",
        "container": "input",
        "fileName": "sales.csv"
      },
      "columnDelimiter": ",",
      "firstRowAsHeader": true
    }
  }
}

// Sink Dataset (SQL Table)
{
  "name": "SqlTableDataset",
  "properties": {
    "linkedServiceName": {"referenceName": "AzureSqlDatabase"},
    "type": "AzureSqlTable",
    "typeProperties": {
      "schema": "dbo",
      "table": "Sales"
    }
  }
}
```

**Step 3: Create Pipeline**

```json
{
  "name": "CopyBlobToSQL",
  "properties": {
    "activities": [
      {
        "name": "CopyData",
        "type": "Copy",
        "inputs": [{"referenceName": "BlobCsvDataset"}],
        "outputs": [{"referenceName": "SqlTableDataset"}],
        "typeProperties": {
          "source": {"type": "DelimitedTextSource"},
          "sink": {
            "type": "SqlSink",
            "writeBatchSize": 10000,
            "preCopyScript": "TRUNCATE TABLE dbo.Sales"
          }
        }
      }
    ]
  }
}
```

**Step 4: Create Trigger**

```json
{
  "name": "DailyTrigger",
  "properties": {
    "type": "ScheduleTrigger",
    "typeProperties": {
      "recurrence": {
        "frequency": "Day",
        "interval": 1,
        "startTime": "2024-01-01T02:00:00Z"
      }
    },
    "pipelines": [
      {"pipelineReference": {"referenceName": "CopyBlobToSQL"}}
    ]
  }
}
```

---

### **Example 2: Incremental Load Pattern**

**Scenario:** Load only new/changed data from SQL to ADLS

**Step 1: Lookup Last Watermark**

```json
{
  "name": "GetLastWatermark",
  "type": "Lookup",
  "typeProperties": {
    "source": {
      "type": "SqlSource",
      "sqlReaderQuery": "SELECT MAX(ModifiedDate) as LastWatermark FROM dbo.Watermark"
    },
    "dataset": {"referenceName": "WatermarkDataset"}
  }
}
```

**Step 2: Copy Incremental Data**

```json
{
  "name": "CopyIncrementalData",
  "type": "Copy",
  "dependsOn": [{"activity": "GetLastWatermark", "dependencyConditions": ["Succeeded"]}],
  "typeProperties": {
    "source": {
      "type": "SqlSource",
      "sqlReaderQuery": {
        "value": "@concat('SELECT * FROM dbo.Orders WHERE ModifiedDate > ''', activity('GetLastWatermark').output.firstRow.LastWatermark, '''')",
        "type": "Expression"
      }
    },
    "sink": {"type": "ParquetSink"}
  }
}
```

**Step 3: Update Watermark**

```json
{
  "name": "UpdateWatermark",
  "type": "SqlServerStoredProcedure",
  "dependsOn": [{"activity": "CopyIncrementalData", "dependencyConditions": ["Succeeded"]}],
  "typeProperties": {
    "storedProcedureName": "usp_UpdateWatermark",
    "storedProcedureParameters": {
      "NewWatermark": {
        "value": "@activity('CopyIncrementalData').output.executionDetails[0].source.rowsRead",
        "type": "DateTime"
      }
    }
  }
}
```

---

### **Example 3: Dynamic Pipeline (Process Multiple Files)**

**Scenario:** Process all CSV files in a folder

**Step 1: Get File List**

```json
{
  "name": "GetFileList",
  "type": "GetMetadata",
  "typeProperties": {
    "dataset": {"referenceName": "BlobFolderDataset"},
    "fieldList": ["childItems"]
  }
}
```

**Step 2: ForEach Loop**

```json
{
  "name": "ForEachFile",
  "type": "ForEach",
  "dependsOn": [{"activity": "GetFileList", "dependencyConditions": ["Succeeded"]}],
  "typeProperties": {
    "items": {
      "value": "@activity('GetFileList').output.childItems",
      "type": "Expression"
    },
    "isSequential": false,
    "batchCount": 10,
    "activities": [
      {
        "name": "CopyFile",
        "type": "Copy",
        "inputs": [
          {
            "referenceName": "DynamicCsvDataset",
            "parameters": {
              "FileName": "@item().name"
            }
          }
        ],
        "outputs": [{"referenceName": "SqlTableDataset"}]
      }
    ]
  }
}
```

---

# PART 2: AZURE SYNAPSE ANALYTICS

## Synapse Overview & Architecture

### **What is Azure Synapse Analytics?**

**Azure Synapse Analytics** is a unified analytics service that brings together:
- Enterprise data warehousing (SQL)
- Big data analytics (Spark)
- Data integration (Pipelines, like ADF)
- Unified workspace (single pane of glass)

**Think of it as:**
- **All-in-one analytics platform**
- **Snowflake + Databricks + ADF** combined
- **Lakehouse architecture** (data lake + data warehouse)

**Key Components:**
1. **Dedicated SQL Pool:** Provisioned data warehouse (MPP)
2. **Serverless SQL Pool:** Query files on-demand (no provisioning)
3. **Spark Pool:** Managed Spark clusters
4. **Pipelines:** Same as ADF (ETL/orchestration)
5. **Data Explorer:** Time-series and log analytics

---

### **Synapse Architecture**

```
┌───────────────────────────────────────────────────────────────────┐
│                    Azure Synapse Workspace                        │
│                                                                    │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │              SYNAPSE STUDIO (Web UI)                        │ │
│  │  [Data] [Develop] [Integrate] [Monitor] [Manage]           │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                    │
│  ┌──────────────┬──────────────┬──────────────┬───────────────┐ │
│  │              │              │              │               │ │
│  │  Dedicated   │  Serverless  │   Spark      │   Pipelines   │ │
│  │  SQL Pool    │  SQL Pool    │   Pool       │   (ADF-like)  │ │
│  │              │              │              │               │ │
│  │  Provisioned │  On-demand   │  Managed     │  ETL          │ │
│  │  DW (MPP)    │  queries     │  Spark       │  Orchestrate  │ │
│  │              │              │              │               │ │
│  └──────┬───────┴──────┬───────┴──────┬───────┴───────┬───────┘ │
│         │              │              │               │          │
└─────────┼──────────────┼──────────────┼───────────────┼──────────┘
          │              │              │               │
          ▼              ▼              ▼               ▼
┌─────────────────────────────────────────────────────────────────┐
│                 ADLS Gen2 (Primary Storage)                     │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌────────────┐ │
│  │   Raw     │  │  Bronze   │  │  Silver   │  │    Gold    │ │
│  │ (Landing) │  │ (Parquet) │  │  (Delta)  │  │ (Curated)  │ │
│  └───────────┘  └───────────┘  └───────────┘  └────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

**Integration:**
- Primary storage: ADLS Gen2
- Can query data without moving it
- Unified security and governance
- Single billing and management

---

## Synapse Dedicated SQL Pools

### **What is Dedicated SQL Pool?**

**Dedicated SQL Pool** = Traditional data warehouse (MPP - Massively Parallel Processing)

**Think of it as:**
- SQL Server on steroids
- Snowflake-like DW
- Provisioned compute (pay even when idle)

**Architecture:**

```
┌─────────────────────────────────────────────────────┐
│           Dedicated SQL Pool (MPP)                  │
│                                                      │
│  ┌────────────┐                                     │
│  │  Control   │  ← User queries come here           │
│  │   Node     │                                     │
│  └──────┬─────┘                                     │
│         │                                            │
│    ┌────┴────┬────────┬────────┬────────┐          │
│    ▼         ▼        ▼        ▼        ▼          │
│  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐     │
│  │Comp │  │Comp │  │Comp │  │Comp │  │Comp │     │
│  │Node │  │Node │  │Node │  │Node │  │Node │     │
│  │  1  │  │  2  │  │  3  │  │  4  │  │  5  │     │
│  └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘     │
│     │        │        │        │        │          │
│  ┌──▼────────▼────────▼────────▼────────▼──────┐  │
│  │           Azure Storage (Data)              │  │
│  │      (60 Distributions per Compute Node)    │  │
│  └─────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘

Total Distributions = Compute Nodes × 60
e.g., DW100c = 1 compute node = 60 distributions
      DW1000c = 10 compute nodes = 600 distributions
```

**Key Concepts:**

1. **Distributions:**
   - Data split into 60 units
   - Distributed across compute nodes
   - Unit of parallelism

2. **Data Warehouse Units (DWU):**
   - Measure of compute power
   - DW100c (small) to DW30000c (large)
   - Can scale up/down dynamically
   - Pricing: ~$1.20/hour (DW100c) to ~$360/hour (DW30000c)

3. **Distribution Types:**

**a. Hash Distribution:**
```sql
CREATE TABLE Sales (
    SalesOrderID INT,
    CustomerID INT,
    OrderDate DATE,
    Amount DECIMAL(10,2)
)
WITH (
    DISTRIBUTION = HASH(CustomerID),
    CLUSTERED COLUMNSTORE INDEX
);
```
- Best for large fact tables
- Distributes rows based on hash of column
- Good for joins on distribution key

**b. Round Robin:**
```sql
CREATE TABLE Staging_Sales (
    SalesOrderID INT,
    CustomerID INT,
    OrderDate DATE,
    Amount DECIMAL(10,2)
)
WITH (
    DISTRIBUTION = ROUND_ROBIN,
    HEAP
);
```
- Even distribution across all distributions
- Fast loading
- Use for staging tables

**c. Replicated:**
```sql
CREATE TABLE DimDate (
    DateKey INT,
    FullDate DATE,
    Year INT,
    Month INT
)
WITH (
    DISTRIBUTION = REPLICATE,
    CLUSTERED COLUMNSTORE INDEX
);
```
- Full copy on each compute node
- Best for small dimension tables (< 2 GB)
- Eliminates data movement for joins

---

### **Dedicated SQL Pool - Common Operations**

**Create Table:**
```sql
-- Large fact table (hash distributed)
CREATE TABLE FactSales (
    SalesKey BIGINT NOT NULL,
    DateKey INT NOT NULL,
    CustomerKey INT NOT NULL,
    ProductKey INT NOT NULL,
    Quantity INT,
    Amount DECIMAL(18,2)
)
WITH (
    DISTRIBUTION = HASH(CustomerKey),
    CLUSTERED COLUMNSTORE INDEX,
    PARTITION (
        DateKey RANGE RIGHT FOR VALUES (
            20240101, 20240201, 20240301  -- Monthly partitions
        )
    )
);

-- Small dimension table (replicated)
CREATE TABLE DimCustomer (
    CustomerKey INT NOT NULL,
    CustomerName NVARCHAR(100),
    Country NVARCHAR(50)
)
WITH (
    DISTRIBUTION = REPLICATE,
    CLUSTERED COLUMNSTORE INDEX
);
```

**Load Data (COPY command - fastest):**
```sql
COPY INTO FactSales
FROM 'https://mystorageaccount.blob.core.windows.net/data/sales/*.parquet'
WITH (
    FILE_TYPE = 'PARQUET',
    CREDENTIAL = (IDENTITY = 'Managed Identity'),
    MAXERRORS = 10,
    COMPRESSION = 'SNAPPY'
);
```

**Query:**
```sql
-- Leverages hash distribution on CustomerKey
SELECT
    c.Country,
    SUM(f.Amount) AS TotalSales
FROM FactSales f
INNER JOIN DimCustomer c
    ON f.CustomerKey = c.CustomerKey
WHERE f.DateKey >= 20240101
GROUP BY c.Country;
```

**Performance Tuning:**
```sql
-- Update statistics (critical!)
CREATE STATISTICS stat_customer ON FactSales(CustomerKey);
CREATE STATISTICS stat_date ON FactSales(DateKey);

-- Rebuild indexes
ALTER INDEX ALL ON FactSales REBUILD;

-- Check query plan
EXPLAIN
SELECT * FROM FactSales WHERE CustomerKey = 123;
```

---

### **Dedicated SQL Pool - PolyBase (External Tables)**

**Query data in ADLS without loading:**

```sql
-- Step 1: Create master key
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'StrongPassword123!';

-- Step 2: Create database scoped credential
CREATE DATABASE SCOPED CREDENTIAL ADLSCredential
WITH IDENTITY = 'Managed Identity';

-- Step 3: Create external data source
CREATE EXTERNAL DATA SOURCE ADLS_DataSource
WITH (
    TYPE = HADOOP,
    LOCATION = 'abfss://container@mystorageaccount.dfs.core.windows.net/',
    CREDENTIAL = ADLSCredential
);

-- Step 4: Create external file format
CREATE EXTERNAL FILE FORMAT ParquetFormat
WITH (
    FORMAT_TYPE = PARQUET,
    DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'
);

-- Step 5: Create external table
CREATE EXTERNAL TABLE ext_Sales (
    SalesOrderID INT,
    CustomerID INT,
    Amount DECIMAL(18,2)
)
WITH (
    LOCATION = '/gold/sales/',
    DATA_SOURCE = ADLS_DataSource,
    FILE_FORMAT = ParquetFormat
);

-- Step 6: Query external table
SELECT * FROM ext_Sales WHERE Amount > 1000;

-- Step 7: Load into internal table (CTAS)
CREATE TABLE FactSales
WITH (
    DISTRIBUTION = HASH(CustomerID),
    CLUSTERED COLUMNSTORE INDEX
)
AS
SELECT * FROM ext_Sales;
```

---

## Synapse Serverless SQL Pool

### **What is Serverless SQL Pool?**

**Serverless SQL Pool** = Query data in ADLS without provisioning any infrastructure.

**Think of it as:**
- Athena (AWS) or BigQuery (GCP) equivalent
- Pay-per-query (not pay-per-hour)
- No setup, no servers to manage

**Use Cases:**
- Ad-hoc data exploration
- Data lake queries
- Logical data warehouse (query files as tables)
- Cost-effective for infrequent queries

**Pricing:**
- $5 per TB of data scanned
- No cost when idle

---

### **Serverless SQL Pool - Querying Files**

**Query CSV:**
```sql
SELECT *
FROM OPENROWSET(
    BULK 'https://mystorageaccount.dfs.core.windows.net/data/sales.csv',
    FORMAT = 'CSV',
    PARSER_VERSION = '2.0',
    HEADER_ROW = TRUE
) AS sales;
```

**Query Parquet:**
```sql
SELECT *
FROM OPENROWSET(
    BULK 'https://mystorageaccount.dfs.core.windows.net/gold/sales/*.parquet',
    FORMAT = 'PARQUET'
) AS sales
WHERE year = 2024;
```

**Query Delta Lake:**
```sql
SELECT *
FROM OPENROWSET(
    BULK 'https://mystorageaccount.dfs.core.windows.net/delta/sales/',
    FORMAT = 'DELTA'
) AS sales;
```

**Query Partitioned Data:**
```sql
-- Partition pruning (only scans 2024/01 folder)
SELECT *
FROM OPENROWSET(
    BULK 'https://mystorageaccount.dfs.core.windows.net/sales/year=*/month=*/*.parquet',
    FORMAT = 'PARQUET'
) AS sales
WHERE
    sales.filepath(1) = '2024'  -- year partition
    AND sales.filepath(2) = '01';  -- month partition
```

---

### **Serverless SQL Pool - External Tables**

**Create logical views over data lake:**

```sql
-- Create database
CREATE DATABASE SalesDB;
GO

USE SalesDB;
GO

-- Create external data source
CREATE EXTERNAL DATA SOURCE ADLS_Sales
WITH (
    LOCATION = 'https://mystorageaccount.dfs.core.windows.net/gold/sales/'
);

-- Create external file format
CREATE EXTERNAL FILE FORMAT ParquetFormat
WITH (FORMAT_TYPE = PARQUET);

-- Create external table
CREATE EXTERNAL TABLE Sales (
    SalesOrderID INT,
    CustomerID INT,
    OrderDate DATE,
    Amount DECIMAL(18,2)
)
WITH (
    LOCATION = '/',
    DATA_SOURCE = ADLS_Sales,
    FILE_FORMAT = ParquetFormat
);

-- Query like a normal table
SELECT
    YEAR(OrderDate) AS Year,
    SUM(Amount) AS TotalSales
FROM Sales
GROUP BY YEAR(OrderDate);
```

**Create view (for Power BI):**
```sql
CREATE VIEW vw_SalesSummary
AS
SELECT
    YEAR(OrderDate) AS Year,
    MONTH(OrderDate) AS Month,
    COUNT(*) AS OrderCount,
    SUM(Amount) AS TotalSales
FROM Sales
GROUP BY YEAR(OrderDate), MONTH(OrderDate);
```

---

## Synapse Spark Pools

### **What is Spark Pool?**

**Spark Pool** = Managed Apache Spark cluster in Synapse.

**Think of it as:**
- Databricks alternative
- Same Spark, managed by Azure
- Integrated with Synapse workspace

**Key Features:**
- Auto-pause (save costs)
- Auto-scale (dynamic sizing)
- Delta Lake built-in
- Integrated with pipelines
- Notebooks (collaborative)

**Pricing:**
- Pay per minute (active time)
- ~$0.22/vCore/hour (small nodes)
- Auto-pause after inactivity (5-240 minutes)

---

### **Spark Pool - Configuration**

**Create Spark Pool (Azure Portal):**
```
Node Size: Small (4 vCores, 32 GB)
Autoscale: Enabled
  Min nodes: 3
  Max nodes: 10
Auto-pause: 15 minutes
Apache Spark version: 3.4
```

**Create Spark Pool (Terraform):**
```hcl
resource "azurerm_synapse_spark_pool" "example" {
  name                 = "sparkpool"
  synapse_workspace_id = azurerm_synapse_workspace.example.id
  node_size_family     = "MemoryOptimized"
  node_size            = "Small"

  auto_scale {
    max_node_count = 10
    min_node_count = 3
  }

  auto_pause {
    delay_in_minutes = 15
  }

  spark_version = "3.4"
}
```

---

### **Spark Pool - PySpark Notebook Examples**

**Read from ADLS:**
```python
# Read Parquet
df = spark.read.parquet("abfss://container@storage.dfs.core.windows.net/gold/sales/")

# Display
df.show()

# Schema
df.printSchema()
```

**Transform:**
```python
from pyspark.sql.functions import col, sum, year

# Filter and aggregate
result = df.filter(col("amount") > 100) \
           .groupBy(year("order_date").alias("year")) \
           .agg(sum("amount").alias("total_sales"))

result.show()
```

**Write to Delta Lake:**
```python
# Write as Delta
df.write.format("delta") \
    .mode("overwrite") \
    .partitionBy("year") \
    .save("abfss://container@storage.dfs.core.windows.net/delta/sales/")
```

**Delta Lake Operations:**
```python
from delta.tables import DeltaTable

# Read Delta table
delta_df = spark.read.format("delta").load("/delta/sales/")

# Update records
delta_table = DeltaTable.forPath(spark, "/delta/sales/")

delta_table.update(
    condition = "status = 'pending'",
    set = {"status": "'completed'"}
)

# Merge (upsert)
delta_table.alias("target").merge(
    updates_df.alias("source"),
    "target.id = source.id"
).whenMatchedUpdate(set = {
    "amount": "source.amount",
    "updated_at": "source.timestamp"
}).whenNotMatchedInsert(values = {
    "id": "source.id",
    "amount": "source.amount",
    "created_at": "source.timestamp"
}).execute()

# Time travel
old_df = spark.read.format("delta").option("versionAsOf", 5).load("/delta/sales/")
```

**SQL in Spark:**
```python
# Register as temp view
df.createOrReplaceTempView("sales")

# Run SQL
result = spark.sql("""
    SELECT
        year(order_date) AS year,
        sum(amount) AS total_sales
    FROM sales
    WHERE status = 'completed'
    GROUP BY year(order_date)
    ORDER BY year
""")

result.show()
```

---

## Synapse Pipelines

**Same as ADF** (100% feature parity)

All ADF concepts apply:
- Pipelines, Activities, Datasets, Linked Services, Triggers
- Same JSON definitions
- Same UI (Synapse Studio)

**Key Differences from ADF:**
1. **Integrated in Synapse Workspace:**
   - Single UI for pipelines + SQL + Spark
   - Easier to orchestrate across all components

2. **Native Synapse Activities:**
   - Synapse Notebook activity (run Spark notebooks)
   - Synapse SQL Script activity

3. **Same Integration Runtime:**
   - Azure IR, Self-hosted IR, Azure-SSIS IR

**Example - Synapse Notebook Activity:**
```json
{
  "name": "RunSparkNotebook",
  "type": "SynapseNotebook",
  "typeProperties": {
    "notebook": {
      "referenceName": "TransformData",
      "type": "NotebookReference"
    },
    "sparkPool": {
      "referenceName": "sparkpool",
      "type": "BigDataPoolReference"
    },
    "parameters": {
      "input_path": {
        "value": "@pipeline().parameters.InputPath",
        "type": "Expression"
      }
    }
  }
}
```

---

## Synapse Integration

### **Unity Catalog (Data Governance)**

**Features:**
- Centralized metadata
- Access control (row/column level)
- Data lineage
- Auditing

**Example:**
```sql
-- Create database in Unity Catalog
CREATE DATABASE IF NOT EXISTS sales_db;

-- Grant permissions
GRANT SELECT ON DATABASE sales_db TO user@company.com;

-- Row-level security
CREATE ROW ACCESS POLICY region_policy
AS (region STRING)
RETURNS BOOLEAN
RETURN region = current_user().region;

ALTER TABLE sales_db.sales_data
SET ROW FILTER region_policy ON (region);
```

---

### **Power BI Integration**

**Direct connection from Synapse to Power BI:**

```python
# In Synapse notebook
df = spark.sql("SELECT * FROM sales_summary")

# Publish to Power BI
df.write.format("com.microsoft.kusto.spark.synapse.connector") \
    .option("kustoCluster", "https://powerbi.kusto.windows.net") \
    .option("kustoDatabase", "MyWorkspace") \
    .option("kustoTable", "SalesData") \
    .mode("append") \
    .save()
```

**Serverless SQL endpoint:**
- Connect Power BI directly to Serverless SQL Pool
- Query data lake through SQL interface
- No data movement required

---

# PART 3: COMPARISON & SCENARIOS

## ADF vs Synapse - When to Use Which

### **Comparison Table**

| Feature | Azure Data Factory | Azure Synapse Analytics |
|---------|-------------------|------------------------|
| **Primary Purpose** | ETL/orchestration | All-in-one analytics |
| **Data Warehouse** | ❌ No | ✅ Yes (Dedicated SQL Pool) |
| **Spark Processing** | ❌ No (use Databricks) | ✅ Yes (Spark Pools) |
| **Pipelines** | ✅ Yes | ✅ Yes (same as ADF) |
| **Serverless SQL** | ❌ No | ✅ Yes |
| **Workspace UI** | ADF Studio | Synapse Studio (all-in-one) |
| **Git Integration** | ✅ Yes | ✅ Yes |
| **Pricing** | Pay per activity run | Pay per compute (DWU, Spark) |
| **Use Case** | Data movement, orchestration | Analytics platform, DW + Spark + Pipelines |

---

### **Decision Tree**

```
Do you need a data warehouse?
├── Yes
│   ├── Also need Spark? → Synapse (DW + Spark + Pipelines)
│   └── Only need DW? → Synapse Dedicated SQL Pool
│
└── No (just ETL/orchestration)
    ├── Need Spark? → ADF + Databricks
    └── Just data movement? → ADF alone
```

---

### **When to Use ADF**

✅ **Pure ETL/orchestration needs**
- Moving data between systems
- No analytics workload
- Simple transformations (Copy, Data Flow)

✅ **Already have separate analytics tools**
- Using Databricks for Spark
- Using Snowflake for DW
- ADF just for orchestration

✅ **Cost-sensitive for small workloads**
- Pay only for pipeline runs
- No idle compute costs

**Example Architecture:**
```
Sources → ADF → ADLS Gen2 → Databricks → Snowflake → Power BI
         (orchestration)      (transform)    (DW)
```

---

### **When to Use Synapse**

✅ **All-in-one analytics platform**
- Need DW + Spark + Pipelines in one place
- Unified workspace for data engineers and analysts

✅ **Data warehouse workloads**
- Traditional DW (star schema)
- Provisioned compute (Dedicated SQL Pool)
- BI reporting on structured data

✅ **Lakehouse architecture**
- Combine data lake (Spark) and DW (SQL)
- Query data lake with SQL (Serverless SQL Pool)
- Delta Lake as storage format

✅ **Integrated development**
- Data engineers (Spark notebooks)
- Data analysts (SQL queries)
- Data scientists (ML notebooks)
- All in one workspace

**Example Architecture:**
```
Sources → Synapse Pipelines → ADLS Gen2 (Delta Lake)
                                    ↓
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
             Spark Pools   Serverless SQL   Dedicated SQL
                  ↓               ↓               ↓
                Power BI      Dashboards      BI Reports
```

---

## Real-World Architecture Patterns

### **Pattern 1: Modern Data Warehouse (Synapse)**

```
┌─────────────────────────────────────────────────────────┐
│                  Data Sources                           │
│  [On-prem SQL] [SaaS Apps] [APIs] [Files]              │
└─────────────┬───────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────┐
│              Synapse Pipelines                          │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                │
│  │ Ingest  │→ │Transform│→ │  Load   │                │
│  └─────────┘  └─────────┘  └─────────┘                │
└─────────────┬───────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────┐
│                  ADLS Gen2                              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌──────────┐ │
│  │  Raw    │→ │ Bronze  │→ │ Silver  │→ │  Gold    │ │
│  │(Landing)│  │(Parquet)│  │ (Delta) │  │(Curated) │ │
│  └─────────┘  └─────────┘  └─────────┘  └──────────┘ │
└─────────────┬──────────────────────┬────────────────────┘
              │                      │
              ▼                      ▼
┌──────────────────────┐  ┌──────────────────────┐
│ Synapse Dedicated    │  │ Synapse Serverless   │
│    SQL Pool          │  │    SQL Pool          │
│  (Star Schema DW)    │  │  (Ad-hoc queries)    │
└──────────┬───────────┘  └──────────┬───────────┘
           │                         │
           └────────┬────────────────┘
                    ▼
           ┌─────────────────┐
           │    Power BI     │
           │   (Reporting)   │
           └─────────────────┘
```

**Components:**
1. **Synapse Pipelines:** Ingest from sources
2. **ADLS Gen2:** Medallion architecture (Bronze → Silver → Gold)
3. **Spark Pools:** Transform raw to curated
4. **Dedicated SQL Pool:** Star schema data warehouse
5. **Serverless SQL Pool:** Exploration and ad-hoc queries
6. **Power BI:** Dashboards and reports

---

### **Pattern 2: Hybrid ETL (ADF + Databricks + Synapse)**

```
┌─────────────────────────────────────────────────────────┐
│                  Data Sources                           │
│  [On-prem Oracle] [SAP] [Salesforce] [REST APIs]       │
└─────────────┬───────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────┐
│          Azure Data Factory                             │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Copy Activities (move data to ADLS)             │  │
│  │  Databricks Notebook (complex transformations)   │  │
│  │  Synapse SQL Script (load to DW)                 │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────┬───────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────┐
│                  ADLS Gen2                              │
│  [Raw] → [Processed] → [Curated]                       │
└─────────────┬──────────────────────┬────────────────────┘
              │                      │
              ▼                      ▼
┌──────────────────────┐  ┌──────────────────────┐
│   Databricks         │  │  Synapse Dedicated   │
│  (Spark Transform)   │  │    SQL Pool (DW)     │
└──────────────────────┘  └──────────┬───────────┘
                                     │
                                     ▼
                            ┌─────────────────┐
                            │    Tableau      │
                            └─────────────────┘
```

**Why this pattern:**
- ADF for orchestration (best in class)
- Databricks for complex Spark workloads (better than Synapse Spark)
- Synapse for DW (cost-effective, integrated)

---

### **Pattern 3: Lakehouse Architecture (Delta Lake)**

```
┌─────────────────────────────────────────────────────────┐
│                  Data Sources                           │
│  [IoT Devices] [Clickstream] [Transactions]            │
└─────────────┬───────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────┐
│              Event Hubs / Kafka                         │
│           (Real-time Streaming)                         │
└─────────────┬───────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────┐
│          Synapse Spark Pools                            │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Streaming jobs write to Delta Lake              │  │
│  │  Batch jobs for historical processing            │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────┬───────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────┐
│           ADLS Gen2 (Delta Lake Format)                 │
│  ┌────────────────────────────────────────────────┐    │
│  │  Bronze (raw events)                           │    │
│  │  Silver (cleaned, validated)                   │    │
│  │  Gold (aggregated, business logic)             │    │
│  └────────────────────────────────────────────────┘    │
└─────────────┬──────────────────────┬────────────────────┘
              │                      │
              ▼                      ▼
┌──────────────────────┐  ┌──────────────────────┐
│ Synapse Serverless   │  │  Synapse Spark       │
│    SQL Pool          │  │    (ML, Analytics)   │
│  (SQL Analytics)     │  │                      │
└──────────┬───────────┘  └──────────┬───────────┘
           │                         │
           └────────┬────────────────┘
                    ▼
           ┌─────────────────┐
           │   Power BI      │
           │   Azure ML      │
           └─────────────────┘
```

**Benefits:**
- Single source of truth (Delta Lake)
- ACID transactions
- Time travel
- Schema evolution
- Both batch and streaming

---

## Interview Questions

### **Q1: What is Azure Data Factory and when would you use it?**

**Answer:**
Azure Data Factory is a cloud-based ETL/data integration service for orchestrating data movement and transformation workflows.

**Use it for:**
- Moving data between 90+ sources (cloud and on-prem)
- Orchestrating data pipelines (like Airflow in Azure)
- Simple transformations via Data Flows
- Scheduled or event-driven data processing

**Architecture:** Control plane (metadata, scheduling) + Data plane (Integration Runtime for execution)

**Key components:** Pipelines, Activities, Datasets, Linked Services, Triggers, Integration Runtime

---

### **Q2: Explain Azure Synapse Analytics. How is it different from ADF?**

**Answer:**

**Synapse** = Unified analytics platform (DW + Spark + Pipelines in one workspace)

**Components:**
1. Dedicated SQL Pool (provisioned DW)
2. Serverless SQL Pool (query data lake on-demand)
3. Spark Pools (managed Spark)
4. Pipelines (same as ADF)

**ADF vs Synapse:**
- **ADF:** Just orchestration/ETL (no compute for analytics)
- **Synapse:** All-in-one (ETL + DW + Spark + SQL)

**Use ADF** when: You just need ETL/orchestration
**Use Synapse** when: You need analytics platform (DW + Spark + ETL together)

---

### **Q3: What is Integration Runtime in ADF?**

**Answer:**

**Integration Runtime (IR)** = Compute infrastructure that executes ADF activities.

**Three types:**

1. **Azure IR (Cloud-to-cloud):**
   - Managed by Azure
   - Auto-scale
   - Use for: Azure Blob → Azure SQL

2. **Self-hosted IR (Hybrid):**
   - Install on your VM/server
   - Access on-prem or private networks
   - Use for: On-prem SQL Server → Azure

3. **Azure-SSIS IR (SSIS packages):**
   - Managed SSIS cluster
   - Use for: Migrate SSIS workloads to cloud

**Example:** To copy from on-prem SQL Server to Azure, you need Self-hosted IR installed on a server that can access your on-prem database.

---

### **Q4: What is the difference between Synapse Dedicated SQL Pool and Serverless SQL Pool?**

**Answer:**

| Feature | Dedicated SQL Pool | Serverless SQL Pool |
|---------|-------------------|---------------------|
| **Compute** | Provisioned (pay even idle) | On-demand (pay per query) |
| **Use Case** | Production DW, consistent workloads | Ad-hoc queries, exploration |
| **Performance** | Predictable, high | Variable based on data |
| **Data Storage** | Internal tables (managed) | External files (ADLS) |
| **Cost** | ~$1.20/hour (DW100c) and up | $5 per TB scanned |
| **Scaling** | Manual (scale up/down DWU) | Automatic |
| **When to Use** | Production DW with SLA | Cost-effective exploration |

**Example:**
- Dedicated: Core business reports (daily revenue dashboard)
- Serverless: Data scientists exploring new datasets

---

### **Q5: Design an end-to-end data pipeline in Azure.**

**Answer:**

**Scenario:** Ingest sales data from on-prem SQL Server, process with Spark, load to data warehouse, visualize in Power BI.

**Architecture:**

1. **Data Sources:** On-prem SQL Server

2. **Ingestion (ADF):**
   - Copy Activity with Self-hosted IR
   - Extract to ADLS Gen2 (raw zone)
   - Trigger: Daily at 2 AM

3. **Processing (Synapse Spark):**
   - Read from raw zone (Parquet)
   - Transform: Clean, validate, aggregate
   - Write to silver/gold zones (Delta Lake)

4. **Data Warehouse (Synapse Dedicated SQL Pool):**
   - Load curated data from gold zone
   - Star schema (fact + dimensions)
   - Optimize with hash distribution, partitioning

5. **Consumption (Power BI):**
   - Connect to Synapse Dedicated SQL Pool
   - Create dashboards and reports

**Pipeline Flow:**
```
On-prem SQL → (ADF Self-hosted IR) → ADLS Raw →
(Synapse Spark) → ADLS Silver/Gold (Delta) →
(Synapse SQL Pool) → Power BI
```

**Monitoring:** ADF Monitor for pipeline runs, Synapse Studio for query performance

---

## Best Practices

### **ADF Best Practices**

1. **Use Managed Identity for authentication** (no credentials)
2. **Parameterize pipelines** (reusable, dynamic)
3. **Enable Git integration** (version control, CI/CD)
4. **Use Self-hosted IR for on-prem** (secure connectivity)
5. **Optimize Copy Activity:**
   - Set parallelCopies (4-32)
   - Use staging for large copies
   - Enable compression
6. **Error handling:**
   - Configure retries on activities
   - Add failure callbacks
   - Monitor with alerts
7. **Cost optimization:**
   - Use tumbling window for backfills
   - Right-size Integration Runtime
   - Delete old logs/runs

---

### **Synapse Best Practices**

1. **Dedicated SQL Pool:**
   - Hash distribute large fact tables
   - Replicate small dimension tables (< 2 GB)
   - Partition large tables by date
   - Update statistics regularly
   - Use columnstore indexes

2. **Serverless SQL Pool:**
   - Partition data in ADLS (year/month/day)
   - Use Parquet/Delta (columnar, compressed)
   - Leverage partition pruning
   - Create views for reusable queries

3. **Spark Pools:**
   - Enable auto-pause (save costs)
   - Use Delta Lake for ACID transactions
   - Partition large datasets
   - Cache frequently used DataFrames
   - Use broadcast joins for small tables

4. **Security:**
   - Use managed identities
   - Enable firewall rules
   - Set up row-level security
   - Use private endpoints for VNet

5. **Cost optimization:**
   - Pause Dedicated SQL Pool when not in use
   - Use Serverless for exploration (cheaper)
   - Right-size Spark pools (start small)
   - Monitor query costs

---

## Summary

### **Key Takeaways**

**Azure Data Factory:**
- ✅ ETL/orchestration service
- ✅ 90+ connectors
- ✅ Hybrid (cloud + on-prem)
- ✅ Pay per activity run
- ✅ Use for: Data movement, pipeline orchestration

**Azure Synapse Analytics:**
- ✅ All-in-one analytics platform
- ✅ DW (Dedicated SQL Pool) + Spark + Pipelines + Serverless SQL
- ✅ Unified workspace
- ✅ Pay per compute (DWU, vCore)
- ✅ Use for: Analytics platform, lakehouse, integrated workflows

**When to Use:**
- **ADF alone:** Just need ETL/orchestration
- **Synapse:** Need DW + Spark + Pipelines in one platform
- **ADF + Databricks + Synapse:** Best-of-breed (ADF orchestration, Databricks Spark, Synapse DW)

---

**You now have complete knowledge of ADF and Synapse! Master these for Azure data engineering interviews! 🚀**

*This comprehensive guide covers everything from fundamentals to production patterns.*
