# Data Modeling Cheatsheet - Quick Reference

## Dimensional Modeling (Kimball)
- **Fact table**: Measures/metrics (quantitative data like sales amount, quantity)
- **Dimension table**: Descriptive attributes (who, what, when, where, why)
- **Star schema**: Fact table surrounded by denormalized dimensions
- **Snowflake schema**: Normalized dimensions (dimension tables reference other dimensions)
- **Grain**: Level of detail in fact table (e.g., daily sales per product per store)

## Fact Table Types
- **Transaction fact**: One row per event (sales transaction, web click)
- **Periodic snapshot**: State at regular intervals (daily inventory, monthly balance)
- **Accumulating snapshot**: Track process milestones (order placed → shipped → delivered)
- **Factless fact**: Events with no measures (student attendance, product promotion)

## Dimension Types
- **Conformed dimension**: Shared across multiple facts (same Customer dim for Sales & Service)
- **Role-playing dimension**: Same dimension used multiple times (Date as Order Date, Ship Date)
- **Junk dimension**: Combine low-cardinality flags/indicators
- **Degenerate dimension**: Dimension data in fact table (order number, invoice ID)
- **Mini-dimension**: Rapidly changing attributes separated out

## Slowly Changing Dimensions (SCD)
- **Type 0**: Never changes (original value retained)
- **Type 1**: Overwrite old value (no history)
- **Type 2**: Add new row with effective dates (full history)
- **Type 3**: Add new column (limited history, e.g., current + previous)
- **Type 4**: Separate history table
- **Type 6**: Hybrid (1+2+3)

## Keys
- **Surrogate key**: System-generated unique ID (auto-increment, UUID)
- **Natural key**: Business identifier (SSN, email, product SKU)
- **Composite key**: Multiple columns form key
- **Foreign key**: Reference to another table

## Normalization
- **1NF**: Atomic values, no repeating groups
- **2NF**: 1NF + no partial dependencies
- **3NF**: 2NF + no transitive dependencies
- **BCNF**: 3NF + every determinant is candidate key
- **Denormalization**: Intentionally add redundancy for performance

## Data Vault
- **Hub**: Business keys (Customer, Product)
- **Link**: Relationships between hubs (Customer-Product purchase)
- **Satellite**: Descriptive attributes with history
- **Benefits**: Flexible, auditable, parallel loading

## Data Warehouse Layers
- **Bronze/Raw**: Unprocessed source data (landing zone)
- **Silver/Staging**: Cleaned, validated, conformed
- **Gold/Presentation**: Aggregated, business-ready (star schema)

## Modeling Patterns
- **Bridge table**: Many-to-many relationships
- **Outrigger**: Dimension references another dimension (avoid, prefer denormalization)
- **Hierarchies**: Natural drill-paths (Product → Category → Department)
- **Shrunken dimension**: Subset of larger dimension (Month vs Day)

## Data Lakehouse
- **Combines**: Data lake (flexibility, raw data) + data warehouse (structure, performance)
- **Key technologies**: Delta Lake, Iceberg, Hudi
- **Features**: ACID transactions, schema enforcement, time travel, unified batch/streaming

## Design Decisions
- **Grain selection**: Most granular level needed (lower = more flexibility)
- **Dimension vs fact**: If mostly descriptive → dimension; if mostly numeric → fact
- **SCD type**: Balance between history needs and complexity
- **Partitioning**: By date, region, or high-cardinality dimension
- **Indexing**: On primary keys, foreign keys, and frequent WHERE columns

## Best Practices
✅ Define grain clearly | ✅ Use surrogate keys | ✅ Denormalize dimensions | ✅ Implement SCD Type 2 for history | ✅ Conformed dimensions across marts | ✅ Document business rules | ✅ Partition large tables | ✅ Add indexes strategically | ✅ Test with real queries
