# 📈 AAPL Market Intelligence Data Pipeline (Databricks Medallion Architecture)
My first databricks pipeline


## 📌 Project Overview  
This project is an automated, production-grade data pipeline built in Databricks. It ingests daily stock market data and AI-powered news sentiment for Apple (AAPL), processes it through a Medallion Architecture (Bronze, Silver, Gold) using PySpark and Delta Lake, and outputs a clean, business-ready table for financial analysis.

## 🚀 Technical Highlights & Best Practices  
I designed this pipeline to reflect enterprise-level data engineering standards, focusing on data quality, security, and cost-efficiency:

Handling Weekend Data Gaps: Stock APIs do not return data on weekends, but news continues to break. I implemented a FULL OUTER JOIN and a forward-fill window function (F.last(ignorenulls=True)) to carry Friday's closing price through the weekend, allowing continuous sentiment tracking without breaking moving averages.

Relevance-Weighted Sentiment: Instead of a simple average, the Gold layer calculates a daily sentiment score weighted by the article's relevance to the specific ticker, filtering out noise from broad market news.

Idempotent Upserts: The Silver layer utilizes MERGE operations on Delta Tables. If the pipeline is triggered twice in one day, it updates existing records rather than creating duplicates.

Cold-Start Protection: The Gold layer includes defensive programming to handle "day one" calculations, preventing null errors or divide-by-zero exceptions when calculating daily percentage returns.

Cost Optimization: Orchestrated via Databricks Workflows, the pipeline runs on an ephemeral Job Cluster that spins up only for the execution duration and terminates immediately after, minimizing cloud compute costs.

Secure Credential Management: API keys are dynamically loaded from a secrets.json file that is strictly managed via .gitignore, ensuring zero credential leakage in version control.

## 🏗️ Architecture & Data Flow  
### 🥉 Bronze Layer (Raw Data)  
Sources: yfinance API (Stock Prices) & Alpha Vantage API (News Sentiment).

Process: Data is ingested in its raw JSON/DataFrame format. Metadata columns (ingestion_timestamp, source_system, ticker_symbol) are appended for auditing and lineage.

Storage: Append-only Delta tables.

### 🥈 Silver Layer (Cleansed & Conformed)
Process: Timestamps are standardized, data types are cast, and raw JSON arrays are flattened. High-order PySpark functions (F.expr("filter(...)")) extract ticker-specific sentiment from nested arrays.

Storage: Upserted (Merged) Delta tables deduplicated by URL and Date.

### 🥇 Gold Layer (Business-Ready Aggregations)
Process: Combines stock metrics (5-Day SMA, Daily Return) with aggregated, relevance-weighted daily sentiment. Missing weekend trading volume is explicitly set to 0.

Storage: Overwritten daily Delta table optimized for BI dashboards.
