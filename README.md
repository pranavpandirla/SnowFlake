# Snowflake Retail Analytics Data Warehouse

## Project Description
A production-ready data warehousing solution built on Snowflake that demonstrates modern data architecture patterns, ELT workflows, and advanced analytics capabilities for a retail business. This project showcases dimensional modeling, data transformation pipelines, CDC (Change Data Capture), time travel, and performance optimization techniques.

## Architecture Overview

```
External Data Sources (AWS S3/Azure Blob)
    ↓
Snowflake Stages (External/Internal)
    ↓
RAW Database (Landing Zone)
    ↓
Snowpipe (Continuous Ingestion) + COPY INTO Commands
    ↓
STAGING Database (Transformation Layer)
    ↓
Stored Procedures + Tasks + Streams (CDC)
    ↓
ANALYTICS Database (Consumption Layer)
    ↓
BI Tools (Tableau, Power BI, Looker)
```

## Tech Stack
- **Data Warehouse**: Snowflake
- **Storage**: AWS S3 (external stage)
- **Orchestration**: Snowflake Tasks & Streams
- **CDC**: Snowflake Streams
- **Languages**: SQL, Python, JavaScript (UDFs)
- **BI Tools**: Power BI, Tableau (optional)
- **Version Control**: Git

## Features
✅ Multi-layer architecture (RAW → STAGING → ANALYTICS)  
✅ Dimensional modeling with star schema  
✅ Slowly Changing Dimensions (SCD Type 2)  
✅ Change Data Capture using Streams  
✅ Automated ELT pipelines with Tasks  
✅ Time Travel & Zero-Copy Cloning  
✅ Role-Based Access Control (RBAC)  
✅ Query optimization & performance tuning  
✅ Data quality checks & validation  
✅ Incremental data loading  
✅ Cost optimization strategies  

## Project Structure
```
snowflake-retail-analytics/
├── data/                          # Sample data files
│   ├── customers.csv
│   ├── products.csv
│   ├── stores.csv
│   ├── transactions.csv
│   └── inventory.csv
├── sql/                           # SQL scripts
│   ├── 01_setup/                  # Initial setup
│   │   ├── 01_create_databases.sql
│   │   ├── 02_create_schemas.sql
│   │   ├── 03_create_roles.sql
│   │   ├── 04_create_warehouses.sql
│   │   └── 05_create_stages.sql
│   ├── 02_raw/                    # Raw layer
│   │   ├── create_raw_tables.sql
│   │   └── load_raw_data.sql
│   ├── 03_staging/                # Staging layer
│   │   ├── create_staging_tables.sql
│   │   ├── staging_transformations.sql
│   │   └── create_streams.sql
│   ├── 04_analytics/              # Analytics layer
│   │   ├── create_dimensions.sql
│   │   ├── create_facts.sql
│   │   ├── create_aggregates.sql
│   │   └── create_views.sql
│   ├── 05_automation/             # Automation
│   │   ├── create_stored_procedures.sql
│   │   ├── create_tasks.sql
│   │   └── create_snowpipe.sql
│   └── 06_optimization/           # Performance
│       ├── clustering_keys.sql
│       ├── materialized_views.sql
│       └── query_optimization.sql
├── scripts/                       # Python scripts
│   ├── generate_sample_data.py
│   ├── upload_to_s3.py
│   └── data_quality_checks.py
├── docs/                          # Documentation
│   ├── setup_guide.md
│   ├── architecture.md
│   ├── data_model.md
│   └── resume_guide.md
├── tests/                         # Testing
│   ├── data_quality_tests.sql
│   └── validation_queries.sql
├── requirements.txt
└── README.md
```

## Prerequisites
- Snowflake account (30-day free trial available)
- AWS account (for S3 storage) or use Snowflake internal stages
- Python 3.8+
- Snowflake CLI (SnowSQL) - optional but recommended

## Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/yourusername/snowflake-retail-analytics.git
cd snowflake-retail-analytics
```

### 2. Install Dependencies
```bash
pip install -r requirements.txt
```

### 3. Generate Sample Data
```bash
python scripts/generate_sample_data.py
```

### 4. Configure Snowflake Connection
Create a `config.json` file:
```json
{
  "account": "your-account.snowflakecomputing.com",
  "user": "your-username",
  "password": "your-password",
  "warehouse": "COMPUTE_WH",
  "database": "RETAIL_DW",
  "schema": "PUBLIC"
}
```

### 5. Run Setup Scripts
Execute SQL scripts in order:
```bash
# Using SnowSQL
snowsql -f sql/01_setup/01_create_databases.sql
snowsql -f sql/01_setup/02_create_schemas.sql
# ... continue with remaining scripts

# Or execute in Snowflake Web UI
```

### 6. Load Data
```bash
# Upload data to S3 (if using external stage)
python scripts/upload_to_s3.py

# Or load from local stage
snowsql -f sql/02_raw/load_raw_data.sql
```

## Architecture Layers

### 1. RAW Database
- **Purpose**: Preserve source data as-is
- **Format**: Semi-structured (JSON/Parquet) or CSV
- **Retention**: Long-term storage
- **Loading**: COPY INTO or Snowpipe

### 2. STAGING Database
- **Purpose**: Data cleansing and standardization
- **Format**: Normalized tables
- **Retention**: 30-90 days
- **Processing**: SQL transformations

### 3. ANALYTICS Database
- **Purpose**: Business-ready data models
- **Format**: Star schema (dimensions + facts)
- **Retention**: As per business needs
- **Access**: BI tools and analysts

## Data Model

### Fact Tables
- **FACT_SALES**: Transaction-level sales data
- **FACT_INVENTORY**: Daily inventory snapshots
- **FACT_CUSTOMER_ACTIVITY**: Customer interaction events

### Dimension Tables
- **DIM_CUSTOMER**: Customer master data (SCD Type 2)
- **DIM_PRODUCT**: Product catalog (SCD Type 2)
- **DIM_STORE**: Store locations
- **DIM_DATE**: Date dimension (pre-populated)
- **DIM_TIME**: Time of day dimension

### Aggregate Tables
- **AGG_DAILY_SALES**: Pre-aggregated daily metrics
- **AGG_PRODUCT_PERFORMANCE**: Product KPIs
- **AGG_CUSTOMER_METRICS**: Customer lifetime value

## Key Features

### Change Data Capture (CDC)
```sql
-- Create stream on staging table
CREATE STREAM STG_CUSTOMER_STREAM ON TABLE STAGING.CUSTOMERS;

-- Process changes
MERGE INTO ANALYTICS.DIM_CUSTOMER tgt
USING STG_CUSTOMER_STREAM src
ON tgt.CUSTOMER_ID = src.CUSTOMER_ID
WHEN MATCHED AND src.METADATA$ACTION = 'DELETE' 
  THEN UPDATE SET IS_CURRENT = FALSE, END_DATE = CURRENT_TIMESTAMP()
WHEN MATCHED AND src.METADATA$ACTION = 'INSERT'
  THEN UPDATE SET ... (handle updates)
WHEN NOT MATCHED 
  THEN INSERT (...) VALUES (...);
```

### Automated Pipelines
```sql
-- Create task for daily processing
CREATE TASK DAILY_SALES_AGGREGATION
  WAREHOUSE = TRANSFORM_WH
  SCHEDULE = 'USING CRON 0 2 * * * UTC'
AS
  CALL SP_PROCESS_DAILY_SALES();
```

### Time Travel
```sql
-- Query data from 1 hour ago
SELECT * FROM FACT_SALES AT(OFFSET => -3600);

-- Restore accidentally deleted data
CREATE TABLE FACT_SALES_RESTORED 
  CLONE FACT_SALES BEFORE(STATEMENT => '<query_id>');
```

### Performance Optimization
```sql
-- Add clustering key for better performance
ALTER TABLE FACT_SALES 
  CLUSTER BY (TRANSACTION_DATE, STORE_ID);

-- Create materialized view
CREATE MATERIALIZED VIEW MV_MONTHLY_SALES AS
  SELECT DATE_TRUNC('MONTH', TRANSACTION_DATE) AS MONTH,
         STORE_ID,
         SUM(TOTAL_AMOUNT) AS TOTAL_SALES
  FROM FACT_SALES
  GROUP BY 1, 2;
```

## Key Metrics & KPIs

### Sales Analytics
- Daily/Weekly/Monthly revenue trends
- Sales by product category
- Store performance comparison
- Year-over-year growth

### Customer Analytics
- Customer lifetime value (CLV)
- Customer segmentation (RFM analysis)
- Churn prediction metrics
- Customer acquisition trends

### Inventory Analytics
- Stock levels by location
- Inventory turnover ratio
- Out-of-stock incidents
- Overstock identification

### Product Analytics
- Best/worst performing products
- Product affinity analysis
- Price elasticity
- Seasonal trends

## Cost Optimization

### Strategies Implemented
1. **Virtual Warehouses**: Right-sized for workload
2. **Auto-suspend**: 5 minutes of inactivity
3. **Auto-resume**: Enabled for convenience
4. **Clustering**: On high-cardinality columns
5. **Result Caching**: Enabled by default
6. **Zero-copy Cloning**: For dev/test environments

### Estimated Costs
- **Storage**: ~$23/TB/month
- **Compute**: ~$2-4/credit (varies by edition)
- **Data Transfer**: Minimal (same region)

## Security & Governance

### Role-Based Access Control
```sql
-- Analyst role: Read-only access to analytics
GRANT USAGE ON DATABASE ANALYTICS TO ROLE ANALYST_ROLE;
GRANT SELECT ON ALL TABLES IN SCHEMA ANALYTICS.CORE TO ROLE ANALYST_ROLE;

-- Data Engineer role: Full access to staging
GRANT ALL ON DATABASE STAGING TO ROLE DATA_ENGINEER_ROLE;
```

### Data Masking
```sql
-- Mask sensitive customer data
CREATE MASKING POLICY EMAIL_MASK AS (VAL STRING) RETURNS STRING ->
  CASE 
    WHEN CURRENT_ROLE() IN ('ADMIN_ROLE') THEN VAL
    ELSE REGEXP_REPLACE(VAL, '.+@', '***@')
  END;

ALTER TABLE DIM_CUSTOMER MODIFY COLUMN EMAIL 
  SET MASKING POLICY EMAIL_MASK;
```

## Monitoring & Logging

### Query History Analysis
```sql
SELECT QUERY_TEXT,
       EXECUTION_TIME / 1000 AS EXECUTION_SECONDS,
       WAREHOUSE_SIZE,
       BYTES_SCANNED / 1024 / 1024 / 1024 AS GB_SCANNED
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE START_TIME >= DATEADD(day, -7, CURRENT_TIMESTAMP())
  AND EXECUTION_TIME > 10000  -- Queries > 10 seconds
ORDER BY EXECUTION_TIME DESC
LIMIT 20;
```

### Credit Usage Tracking
```sql
SELECT WAREHOUSE_NAME,
       SUM(CREDITS_USED) AS TOTAL_CREDITS,
       SUM(CREDITS_USED) * 2.5 AS ESTIMATED_COST_USD
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY
WHERE START_TIME >= DATEADD(month, -1, CURRENT_TIMESTAMP())
GROUP BY WAREHOUSE_NAME
ORDER BY TOTAL_CREDITS DESC;
```

## Testing & Validation

### Data Quality Checks
- Row count validation
- Null value detection
- Referential integrity checks
- Duplicate detection
- Data type validation
- Business rule validation

### Example Tests
```sql
-- Check for duplicate customers
SELECT CUSTOMER_ID, COUNT(*) 
FROM DIM_CUSTOMER 
WHERE IS_CURRENT = TRUE
GROUP BY CUSTOMER_ID 
HAVING COUNT(*) > 1;

-- Validate referential integrity
SELECT COUNT(*) AS ORPHAN_RECORDS
FROM FACT_SALES f
LEFT JOIN DIM_CUSTOMER c ON f.CUSTOMER_KEY = c.CUSTOMER_KEY
WHERE c.CUSTOMER_KEY IS NULL;
```

## Future Enhancements
- [ ] Integration with dbt (data build tool)
- [ ] Real-time streaming with Snowpipe
- [ ] Machine learning with Snowpark
- [ ] Data sharing with external partners
- [ ] Integration with Airflow for orchestration
- [ ] Advanced analytics with Python UDFs
- [ ] Data cataloging and lineage tracking

## Contributing
Contributions are welcome! Please feel free to submit a Pull Request.

## License
MIT License

## Contact
[Your Name] - [Your Email]  
Project Link: https://github.com/yourusername/snowflake-retail-analytics

## Acknowledgments
- Snowflake Documentation
- Data Warehouse Toolkit by Ralph Kimball
- Community best practices

---

**⭐ If you find this project helpful, please give it a star!**
