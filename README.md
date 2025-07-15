# Stock Market Real-Time Kafka Streaming & Analytics Pipeline

# Overview
This project implements a real-time data streaming and analytics pipeline for stock market data. It ingests live stock quotes, streams them via Apache Kafka, stores raw and processed data in AWS S3, transforms data with AWS Glue, and enables analytics in Amazon Redshift. Built as a learning project, it showcases expertise in real-time data processing, cloud architecture, and data warehousing.

# Problem Statement
The pipeline aims to:
- Ingest real-time stock market data from an external API.
- Stream high-volume data with low latency and fault tolerance.
- Store raw and processed data cost-effectively.
- Transform data for analytical use.
- Support complex queries for business intelligence.



# Components
1. Python Scripts:
   - Data Producer: Fetches real-time stock quotes from the Finnhub API (WebSocket for live data).
   - Kafka Producer: Serializes data to JSON and publishes to a Kafka topic.

2. Apache Kafka (Amazon EC2):
   - Hosted on EC2 (t3.medium, Amazon Linux 2) with Kafka 3.6.1 and Zookeeper.
   - Configured for high throughput (3 brokers, 3 partitions, replication factor 2).
   - Security: TLS encryption and VPC security groups (port 9092).

3. Amazon S3 (Data Lake):
   - Raw Data: `s3://kafka.stock.market.project/raw.data/year=YYYY/month=MM/day=DD/`
   - Processed Data: `s3://kafka.stock.market.project/processed.data/year=YYYY/month=MM/day=DD/`
   - Encryption: SSE-S3 for data at rest.

4. AWS Glue (Data Catalog & ETL):
   - Glue Crawler: Infers schemas from S3 raw data and updates the Glue Data Catalog.
   - Glue ETL Jobs: PySpark jobs transform JSON to Parquet, clean data, and write to the processed S3 bucket.

5. Amazon Redshift:
   - Serverless cluster for cost-efficient analytics.
   - Data loaded via `COPY` command from processed S3 data.
   - Optimized for SQL queries and BI tools (e.g., Amazon QuickSight).

# Technical Implementation
# 1. Data Ingestion
- API: Finnhub (WebSocket for real-time quotes, free tier: 60 API calls/minute).
- Libraries: `websocket-client`, `requests`, `kafka-python`.
- Logic:
  - Connect to Finnhub WebSocket for live quotes (e.g., AAPL, GOOGL).
  - Serialize to JSON.
  - Publish to Kafka topic `stock-data`.

# 2. Kafka Setup
- EC2 Setup: 3 t3.medium instances, Amazon Linux 2, Kafka 3.6.1.
- Configuration: 3 brokers, topic `stock-data` (3 partitions, replication factor 2).
- Security: TLS for in-transit encryption, security group restricts port 9092 to trusted IPs.

# 3. Kafka to S3
- Python Consumer:
  - Consumes from `stock-data` topic.
  - Batches messages (100 messages or 10 seconds) and writes JSONL files to S3.
  - Path: `s3://kafka.stock.market.project/raw.data/year=YYYY/month=MM/day=DD/`.
- Alternative: Kafka Connect with Confluent S3 Sink Connector (optional for future).

# 4. AWS Glue
- Crawler: Scans `s3://kafka.stock.market.project/raw.data/` and creates a table in the Glue Data Catalog.
- ETL Job (PySpark):
  - Reads JSON from Data Catalog.
  - Transforms to Parquet (columnar format).
  - Handles missing values and type conversions.
  - Writes to `s3://kafka.stock.market.project/processed.data/year=YYYY/month=MM/day=DD/`.

# 5. Amazon Redshift
- Cluster: Redshift Serverless (base RPU: 8).
- Schema: Tables for stock data (e.g., `stock_prices(symbol, price, timestamp)`).
- Data Loading: `COPY` command from `s3://kafka.stock.market.project/processed.data/`.
- Analytics: Supports SQL queries for trends and BI integration.

# Setup Guide
# Prerequisites
- AWS account with IAM permissions for EC2, S3, Glue, and Redshift.
- Finnhub API key ([finnhub.io](https://finnhub.io)).
- Python 3.8+, `pip install -r requirements.txt` (includes `requests`, `websocket-client`, `kafka-python`, `boto3`).

### Steps
1. Clone Repository:
   ```bash
   git clone <repository-url>
   cd stock-market-pipeline
   ```

2. Configure Environment:
   - Set Finnhub API key in `config.ini`.
   - Configure AWS credentials (`~/.aws/credentials`).

3. EC2 & Kafka:
   - Launch 3 EC2 instances (t3.medium, Amazon Linux 2).
   - Install Kafka 3.6.1 and Zookeeper ([Kafka Quickstart](https://kafka.apache.org/quickstart)).
   - Create topic:
     ```bash
     bin/kafka-topics.sh --create --topic stock-data --bootstrap-server <broker>:9092 --partitions 3 --replication-factor 2
     ```

4. S3 Setup:
   - Create buckets: `s3://kafka.stock.market.project/raw.data/` and `s3://kafka.stock.market.project/processed.data/`.
   - Enable SSE-S3 encryption.

5. Run Producer:
   ```bash
   python producer.py
   ```

6. Run Consumer:
   ```bash
   python consumer_to_s3.py
   ```

7. AWS Glue:
   - Create a crawler for `s3://kafka.stock.market.project/raw.data/` (database: `stock_db`).
   - Create a PySpark ETL job to transform and write to `s3://kafka.stock.market.project/processed.data/`.

8. Redshift:
   - Set up Redshift Serverless (AWS Console).
   - Create table:
     ```sql
     CREATE TABLE stock_prices (
         symbol VARCHAR(10),
         price DECIMAL(10,2),
         timestamp TIMESTAMP
     );
     ```
   - Load data:
     ```sql
     COPY stock_prices FROM 's3://kafka.stock.market.project/processed.data/' 
     IAM_ROLE '<your-role-arn>' FORMAT AS PARQUET;
     ```

9. Query Data:
   - Use Redshift Query Editor or BI tools for analytics.



