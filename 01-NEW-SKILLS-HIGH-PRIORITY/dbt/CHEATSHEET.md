# dbt (Data Build Tool) Cheatsheet - Quick Reference

## dbt Fundamentals
- **dbt**: Transform data in warehouse using SQL (T in ELT)
- **Model**: SQL SELECT statement defining a table/view
- **Source**: Raw data tables (define in sources.yml)
- **Seed**: CSV files loaded into warehouse
- **Snapshot**: Type 2 SCD (track changes over time)
- **Test**: Data quality checks (schema tests, data tests)

## Materialization Types
- **Table**: Full table refresh (create or replace table)
- **View**: Virtual table (query runs on read)
- **Incremental**: Append/merge new data only (efficient for large tables)
- **Ephemeral**: CTE, not materialized (used as building block)
- **Materialized view**: Precomputed view (not all warehouses support)

## dbt Commands
```bash
dbt run                    # Run all models
dbt run --select model     # Run specific model
dbt run --select tag:daily # Run models with tag
dbt test                   # Run all tests
dbt test --select model    # Test specific model
dbt build                  # Run + test in dependency order
dbt docs generate          # Generate documentation
dbt docs serve             # Serve docs locally
```

## Model Configuration
```sql
{{ config(
    materialized='incremental',
    unique_key='id',
    on_schema_change='append_new_columns',
    tags=['daily', 'finance']
) }}
```

## Jinja & Macros
- **Jinja**: Templating language `{{ }}` and `{% %}`
- **Variables**: `{{ var('start_date') }}`
- **Macros**: Reusable SQL functions
- **Loops**: `{% for item in list %} ... {% endfor %}`
- **Conditions**: `{% if condition %} ... {% endif %}`

## Ref & Source
- **{{ ref('model_name') }}**: Reference another model (dependency)
- **{{ source('source_name', 'table_name') }}**: Reference raw source table
- **Benefits**: Dependency graph, lineage tracking, error handling

## Testing
- **Schema tests**: unique, not_null, relationships, accepted_values
- **Data tests**: Custom SQL queries (must return 0 rows if pass)
- **dbt_utils tests**: recency, expression_is_true, cardinality_equality
- **Custom tests**: Write your own in tests/ folder

## Incremental Models
```sql
{{ config(materialized='incremental', unique_key='id') }}

select * from {{ source('raw', 'events') }}
{% if is_incremental() %}
  where event_time > (select max(event_time) from {{ this }})
{% endif %}
```

## Packages
- **dbt_utils**: Utilities (date_spine, surrogate_key, union_relations)
- **dbt_expectations**: Great Expectations-style tests
- **Codegen**: Generate dbt code (sources, models, tests)
- **Install**: Define in packages.yml, run `dbt deps`

## Project Structure
```
my_dbt_project/
├── dbt_project.yml        # Project config
├── models/
│   ├── staging/           # Raw data cleaned
│   ├── intermediate/      # Business logic
│   └── marts/            # Final analytics tables
├── tests/                 # Custom tests
├── macros/               # Custom macros
├── seeds/                # CSV files
├── snapshots/            # SCD Type 2
└── analyses/             # Ad-hoc queries
```

## Model Layers
- **Staging**: 1:1 with source, clean column names, basic typing
- **Intermediate**: Business logic, joins, calculations
- **Marts**: Final analytics tables, aggregated, optimized for BI

## Documentation
- **schema.yml**: Describe models, columns, tests
- **Descriptions**: Show in generated docs
- **dbt docs generate**: Create static site
- **Lineage graph**: Visual DAG of dependencies

## Orchestration
- **Airflow**: Use BashOperator or dbt provider
- **Dagster**: Native dbt integration
- **dbt Cloud**: Managed scheduling (SaaS)
- **Prefect**: Python-based orchestration

## Best Practices
✅ One model per file | ✅ Use staging layer | ✅ Modular models | ✅ Test everything | ✅ Document models | ✅ Use macros for DRY | ✅ Incremental for large tables | ✅ Version control (Git) | ✅ CI/CD with slim CI | ✅ Monitor run times

## Common Patterns
- **Surrogate key**: `{{ dbt_utils.surrogate_key(['col1', 'col2']) }}`
- **Date spine**: Generate date series `{{ dbt_utils.date_spine(...) }}`
- **Union tables**: `{{ dbt_utils.union_relations(relations=[...]) }}`
- **Pivot**: Use dbt_utils.pivot macro

## Configuration Hierarchy
1. dbt_project.yml (project-level)
2. Folder config in dbt_project.yml
3. Model file config block
4. Inline config in SQL

## Incremental Strategies
- **Append**: Add new rows (fast, but no updates)
- **Merge**: Upsert based on unique_key (Snowflake, BigQuery, Databricks)
- **Delete+insert**: Delete matching, insert new (all warehouses)

## Environment Management
- **Profiles.yml**: Define targets (dev, prod)
- **Target**: Switch with `--target prod`
- **Separate schemas**: dev_username, prod

## Cost Optimization
- **Incremental models**: Process only new data
- **Clustering**: Optimize query performance (Snowflake, BigQuery)
- **Partitioning**: Reduce data scanned
- **Slim CI**: Only test changed models in PRs

---

**Quick Win:** `dbt run --select model+ --exclude model` (run model and all downstream, except model itself)
