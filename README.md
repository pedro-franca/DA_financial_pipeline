# 📈 AAPL Market Intelligence Data Pipeline (Databricks Medallion Architecture, AWS S3, PySpark)
My first databricks pipeline


## 📌 Project Overview  
This project is an automated, production-grade ELT (Extract, Load, Transform) data pipeline. It ingests daily stock market data and AI-powered news sentiment for Apple (AAPL), archives the raw data in an AWS S3 Data Lake, and processes it through a Databricks Medallion Architecture (Bronze, Silver, Gold) using PySpark and Delta Lake. The final output is a clean, business-ready table optimized for financial analysis.
  
## 🚀 Technical Highlights & Best Practices  
I designed this pipeline to reflect senior-level data engineering standards, focusing on cloud architecture, network efficiency, and handling real-world data anomalies:

- **AWS S3 Landing Zone & Serverless Optimization**: Implemented a true Data Lakehouse pattern. Raw API JSON payloads are archived directly to AWS S3 via boto3 to create an immutable historical record. To optimize network I/O and bypass credential restrictions on Databricks Serverless Compute, the Bronze layer builds the Spark DataFrame directly from memory rather than forcing a redundant download from S3, while still appending the S3 file path for strict data lineage.

- **Handling Weekend Data Gaps**: Stock APIs do not return data on weekends, but news continues to break. I implemented a FULL OUTER JOIN and a forward-fill window function (F.last(ignorenulls=True)) to carry Friday's closing price through the weekend, allowing continuous sentiment tracking without breaking moving averages.

- **Relevance-Weighted Sentiment**: Instead of a simple average, the Gold layer calculates a daily sentiment score weighted by the article's relevance to the specific ticker, filtering out noise from broad market news.

- **Idempotent Upserts**: The Silver layer utilizes MERGE operations on Delta Tables. If the pipeline is triggered twice in one day, it updates existing records rather than creating duplicates.

- **Secure Credential Management**: API keys are dynamically loaded from a secrets.json file that is strictly managed via .gitignore, ensuring zero credential leakage in version control.

## 🏗️ Architecture & Data Flow  
  
###📥 1. Extract & Load (The S3 Data Lake)
**Sources**: yfinance API (Stock Prices) & Alpha Vantage API (News Sentiment).

**Process**: Python fetches the data and uses boto3 to drop the raw JSON files into an S3 bucket (s3://portfolio-market-data-raw/...).

### 🥉 Bronze Layer (Raw Data)  

**Process**: Data is ingested in its raw JSON/DataFrame format. Metadata columns (ingestion_timestamp, source_system, ticker_symbol) are appended for auditing and lineage.

**Storage**: Append-only Delta tables.

### 🥈 Silver Layer (Cleansed & Conformed)
**Process**: Timestamps are standardized, data types are cast, and raw JSON arrays are flattened. High-order PySpark functions (F.expr("filter(...)")) extract ticker-specific sentiment from nested arrays.

**Storage**: Upserted (Merged) Delta tables deduplicated by URL and Date.

### 🥇 Gold Layer (Business-Ready Aggregations)
**Process**: Combines stock metrics (5-Day SMA, Daily Return) with aggregated, relevance-weighted daily sentiment. Missing weekend trading volume is explicitly set to 0.

**Storage**: Overwritten daily Delta table optimized for BI dashboards.
