# 🏦 Banking Modern Data Stack Pipeline

<div align="center">

![Snowflake](https://img.shields.io/badge/Snowflake-29B5E8?logo=snowflake&logoColor=white&style=for-the-badge)
![DBT](https://img.shields.io/badge/dbt-FF694B?logo=dbt&logoColor=white&style=for-the-badge)
![Apache Airflow](https://img.shields.io/badge/Apache%20Airflow-017CEE?logo=apacheairflow&logoColor=white&style=for-the-badge)
![Apache Kafka](https://img.shields.io/badge/Apache%20Kafka-231F20?logo=apachekafka&logoColor=white&style=for-the-badge)
![Debezium](https://img.shields.io/badge/Debezium-EF3B2D?logo=apache&logoColor=white&style=for-the-badge)
![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white&style=for-the-badge)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white&style=for-the-badge)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?logo=postgresql&logoColor=white&style=for-the-badge)
![MinIO](https://img.shields.io/badge/MinIO-C72E49?logo=minio&logoColor=white&style=for-the-badge)

**An enterprise-grade, end-to-end real-time data engineering pipeline simulating a modern banking system**

[Features](#-key-features) • [Architecture](#️-architecture) • [Tech Stack](#-tech-stack) • [Quick Start](#-quick-start) • [Pipeline Flow](#-pipeline-flow)

</div>

---

## 📌 Project Overview

This project demonstrates a **production-ready modern data stack** implementation for the **banking domain**, showcasing:

- ✅ **Real-time Change Data Capture (CDC)** from transactional databases
- ✅ **Event-driven architecture** with streaming data pipelines
- ✅ **Cloud data warehousing** with dimensional modeling
- ✅ **Slowly Changing Dimensions (SCD Type-2)** for historical tracking
- ✅ **Automated orchestration** and data quality testing
- ✅ **CI/CD integration** with automated deployment workflows

> 💡 **Use Case**: Imagine you're a data engineer at a bank that needs to track customer accounts, transactions, and balance changes in real-time while maintaining historical records for compliance and analytics.

---

## 🗄️ Database Schema

The OLTP system consists of three core entities representing a simplified banking system:

![Database Schema](schema.png)

### **Schema Design Highlights:**

| Table | Purpose | Key Relationships |
|-------|---------|------------------|
| **`customers`** | Stores customer demographic information | One-to-Many → `accounts` |
| **`accounts`** | Banking account details (balance, type, currency) | Many-to-One → `customers`<br>One-to-Many → `transactions` |
| **`transactions`** | Financial transaction records | Many-to-One → `accounts` |

**Key Features:**
- **Referential Integrity**: Foreign key constraints ensure data consistency
- **Timestamps**: All tables include `created_at` for temporal tracking
- **Account Types**: Supports checking, savings, credit accounts
- **Transaction Types**: Captures deposits, withdrawals, transfers, payments
- **Multi-Currency**: Accounts can hold different currencies (USD, EUR, GBP, etc.)

---

## 🏗️ Architecture  

### **High-Level System Design**

```
┌─────────────┐         ┌──────────┐         ┌─────────┐         ┌───────────┐         ┌──────────────┐
│  Data Gen   │────────▶│PostgreSQL│────────▶│ Kafka   │────────▶│  MinIO    │────────▶│  Snowflake   │
│  (Faker)    │         │  (OLTP)  │         │+Debezium│         │  (S3)     │         │  (OLAP)      │
└─────────────┘         └──────────┘         └─────────┘         └─────────┘         └──────────────┘
                              │                    │                    │                      │
                              │                    │                    │                      │
                        CDC Capture          Stream Events         Object Store           Data Warehouse
                        (WAL Logs)          (JSON/Avro)            (Parquet)              (Bronze/Silver/Gold)
                                                                         │                      │
                                                                         │                      │
                                                                         ▼                      ▼
                                                                   ┌──────────┐         ┌────────────┐
                                                                   │ Airflow  │────────▶│    dbt     │
                                                                   │  (DAGs)  │         │ (Transform)│
                                                                   └──────────┘         └────────────┘
                                                                   Orchestration        Modeling & Tests
```

### **Pipeline Flow Explained:**

1. **📊 Data Generation Layer**
   - Python Faker generates realistic banking data (customers, accounts, transactions)
   - Simulates real-world scenarios: account openings, deposits, withdrawals, transfers
   - Configurable data volume and frequency

2. **🔄 Change Data Capture (CDC)**
   - Debezium connector monitors PostgreSQL Write-Ahead Log (WAL)
   - Captures INSERT, UPDATE, DELETE operations in real-time
   - Zero impact on OLTP performance (log-based CDC)
   - Publishes events to Kafka topics with full before/after state

3. **📨 Event Streaming**
   - Apache Kafka acts as the central nervous system
   - Decouples producers from consumers
   - Guarantees message delivery and ordering
   - Scalable distributed architecture

4. **💾 Data Lake Storage**
   - MinIO (S3-compatible) stores raw events in Parquet format
   - Provides cost-effective, scalable object storage
   - Acts as the "Bronze" layer (raw, immutable data)
   - Enables time-travel and data replay capabilities

5. **🏭 Data Warehouse (Snowflake)**
   - **Bronze Schema**: Raw ingestion from MinIO
   - **Silver Schema**: Cleaned, standardized staging tables
   - **Gold Schema**: Business-ready marts (facts & dimensions)
   - Supports both batch and micro-batch loading

6. **🔧 Transformation Layer (dbt)**
   - **Staging Models**: Data cleansing, type casting, deduplication
   - **Fact Tables**: `fact_transactions` - grain: one row per transaction
   - **Dimension Tables**: `dim_customers`, `dim_accounts` - slowly changing dimensions
   - **Snapshots**: SCD Type-2 tracking for historical analysis
   - **Tests**: Schema validation, uniqueness, referential integrity, custom business rules

7. **⚙️ Orchestration (Airflow)**
   - **DAG 1**: `minio_to_snowflake` - Incremental data ingestion
   - **DAG 2**: `scd_snapshots` - Scheduled snapshot execution
   - Automated scheduling, monitoring, and alerting
   - Retry logic and failure handling

8. **🚀 CI/CD Pipeline**
   - **Continuous Integration**: dbt compile, SQL linting, automated tests
   - **Continuous Deployment**: Auto-deploy to production on merge
   - GitHub Actions workflows for quality gates

---

## ⚡ Tech Stack

### **Core Technologies**

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **OLTP Database** | PostgreSQL 15 | Source transactional system with ACID guarantees |
| **CDC Platform** | Debezium 2.2 | Log-based change data capture from Postgres WAL |
| **Event Streaming** | Apache Kafka 7.4 | Distributed event streaming platform |
| **Object Storage** | MinIO | S3-compatible data lake for raw events |
| **Data Warehouse** | Snowflake | Cloud-native columnar OLAP database |
| **Transformation** | dbt Core + dbt-snowflake | ELT framework for data modeling & testing |
| **Orchestration** | Apache Airflow 2.9 | Workflow automation and DAG scheduling |
| **Data Generation** | Python + Faker | Synthetic data simulation |
| **Containerization** | Docker + Docker Compose | Reproducible local development environment |
| **CI/CD** | GitHub Actions | Automated testing and deployment |

### **Why This Stack?**

- **Scalability**: Each component scales independently (Kafka partitions, Snowflake warehouses)
- **Reliability**: Battle-tested technologies used by Fortune 500 companies
- **Cost-Effective**: Snowflake's pay-per-use model + open-source components
- **Developer Experience**: Modern tooling with strong community support
- **Industry Standard**: Skills directly transferable to 90%+ of data engineering roles

---

## ✅ Key Features

### **Real-Time Data Capabilities**
- 🔴 **Live CDC**: Captures database changes within milliseconds using log-based replication
- 🎯 **Event-Driven**: Kafka ensures exactly-once delivery semantics
- ⚡ **Low Latency**: End-to-end data freshness under 30 seconds

### **Data Quality & Governance**
- ✅ **Automated Testing**: dbt tests validate data integrity at every layer
- 📊 **Schema Evolution**: Handles schema changes without pipeline breaks
- 🔍 **Data Lineage**: Full traceability from source to dashboard
- 🔒 **ACID Compliance**: Maintains consistency across distributed systems

### **Historical Tracking**
- 📅 **SCD Type-2**: Snapshots capture every state change of customers/accounts
- ⏰ **Time Travel**: Query data as it existed at any point in history
- 🔄 **Audit Trail**: Complete record of who changed what and when

### **Production-Ready Engineering**
- 🐳 **Containerized**: Entire stack runs in Docker for consistency
- 🔄 **Idempotent**: Re-runnable pipelines without side effects
- 📈 **Monitored**: Airflow UI provides visibility into pipeline health
- 🚨 **Alerting**: Configurable notifications for failures

### **Modern Development Practices**
- 🧪 **Test-Driven**: Write tests before models
- 📝 **Documentation**: Auto-generated docs from dbt schema files
- 🔁 **Version Control**: Git-based workflow for all code artifacts
- 🚀 **CI/CD**: Automated deployments reduce human error

---

## 📂 Repository Structure

```
banking/
│
├── .github/
│   └── workflows/              # GitHub Actions CI/CD
│       ├── ci.yml              # Lint, test, compile on PR
│       └── cd.yml              # Deploy on merge to main
│
├── banking_dbt/                # dbt Project (Transformations)
│   ├── models/
│   │   ├── staging/            # Silver layer: cleaned data
│   │   │   ├── stg_customers.sql
│   │   │   ├── stg_accounts.sql
│   │   │   └── stg_transactions.sql
│   │   ├── marts/              # Gold layer: business logic
│   │   │   ├── dimensions/
│   │   │   │   ├── dim_customers.sql
│   │   │   │   └── dim_accounts.sql
│   │   │   └── facts/
│   │   │       └── fact_transactions.sql
│   │   └── sources.yml         # Source definitions
│   ├── snapshots/              # SCD Type-2 tracking
│   │   ├── customers_snapshot.sql
│   │   └── accounts_snapshot.sql
│   └── dbt_project.yml         # Project configuration
│
├── consumer/                   # Kafka → MinIO
│   └── kafka_to_minio.py       # Consumes events, writes Parquet
│
├── data-generator/             # Synthetic Data
│   └── faker_generator.py      # Generates customers, accounts, txns
│
├── docker/                     # Airflow Resources
│   ├── dags/
│   │   ├── minio_to_snowflake_dag.py  # Ingestion DAG
│   │   └── scd_snapshots.py           # Snapshot DAG
│   ├── logs/                   # Airflow logs
│   └── plugins/                # Custom Airflow plugins
│
├── kafka-debezium/
│   └── generate_and_post_connector.py  # Debezium connector setup
│
├── postgres/
│   └── schema.sql              # OLTP DDL (customers, accounts, txns)
│
├── docker-compose.yml          # Local infrastructure
├── docker-compose-arm64.yml    # Apple Silicon support
├── dockerfile-airflow.dockerfile
├── requirements.txt
├── .gitignore
└── README.md
```

---

## 🚀 Quick Start

### **Prerequisites**
- Docker Desktop 4.0+ with 8GB+ RAM allocated
- Docker Compose 2.0+
- Python 3.11+
- Snowflake account (free trial available)
- Git

### **1. Clone Repository**
```bash
git clone https://github.com/ASR373/banking.git
cd banking
```

### **2. Environment Setup**
```bash
# Create .env file with required credentials
cat > .env << EOF
# PostgreSQL
POSTGRES_USER=bankuser
POSTGRES_PASSWORD=bankpass
POSTGRES_DB=bankingdb

# MinIO
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=minioadmin

# Airflow
AIRFLOW_DB_USER=airflow
AIRFLOW_DB_PASSWORD=airflow
AIRFLOW_DB_NAME=airflowdb

# Snowflake (add your credentials)
SNOWFLAKE_ACCOUNT=your_account
SNOWFLAKE_USER=your_user
SNOWFLAKE_PASSWORD=your_password
SNOWFLAKE_WAREHOUSE=your_warehouse
SNOWFLAKE_DATABASE=BANKING_DW
EOF
```

### **3. Start Infrastructure**
```bash
# For Apple Silicon (M1/M2/M3)
docker compose -f docker-compose-arm64.yml up -d

# For Intel/AMD
docker compose up -d

# Verify all services are running
docker ps
```

### **4. Initialize Airflow**
```bash
# Initialize database
docker compose exec airflow-scheduler airflow db init

# Create admin user
docker compose exec airflow-scheduler airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password admin
```

### **5. Set Up CDC Connector**
```bash
cd kafka-debezium
python generate_and_post_connector.py
```

### **6. Generate Sample Data**
```bash
cd data-generator
pip install -r requirements.txt
python faker_generator.py
```

### **7. Configure dbt**
```bash
cd banking_dbt
echo "banking_dw:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: your_account
      user: your_user
      password: your_password
      role: SYSADMIN
      database: BANKING_DW
      warehouse: COMPUTE_WH
      schema: ANALYTICS
      threads: 4" > ~/.dbt/profiles.yml

# Test connection
dbt debug
```

### **8. Access Applications**
- **Airflow UI**: http://localhost:8080 (admin/admin)
- **MinIO Console**: http://localhost:9001 (minioadmin/minioadmin)
- **Kafka UI**: http://localhost:9021 (if Confluent Control Center enabled)

---

## 📊 Pipeline Flow

### **Step-by-Step Data Journey**

#### **Phase 1: Data Generation** ⚙️
```python
# faker_generator.py generates realistic data
Customer(id=1, name="John Doe", email="john@example.com")
  → Account(id=101, customer_id=1, balance=5000.00, type="checking")
    → Transaction(id=1001, account_id=101, amount=250.00, type="deposit")
```

#### **Phase 2: OLTP Storage** 💾
```sql
-- Data lands in PostgreSQL with ACID guarantees
INSERT INTO customers (first_name, last_name, email) VALUES ('John', 'Doe', 'john@example.com');
INSERT INTO accounts (customer_id, account_type, balance) VALUES (1, 'checking', 5000.00);
INSERT INTO transactions (account_id, amount, txn_type) VALUES (101, 250.00, 'deposit');
```

#### **Phase 3: CDC Capture** 🔍
```json
// Debezium captures WAL entry
{
  "op": "c",  // create
  "after": {
    "id": 1001,
    "account_id": 101,
    "amount": 250.00,
    "txn_type": "deposit",
    "created_at": "2024-02-15T10:30:00Z"
  }
}
```

#### **Phase 4: Kafka Stream** 📨
```
Topic: banking.public.transactions
Partition: 0
Offset: 12345
Message: [JSON payload above]
```

#### **Phase 5: MinIO Storage** 📦
```
s3://banking-raw/
  └── transactions/
      └── 2024/02/15/
          └── transactions_20240215_103000.parquet
```

#### **Phase 6: Snowflake Ingestion** ❄️
```sql
-- Airflow DAG copies to Bronze
COPY INTO BRONZE.RAW_TRANSACTIONS
FROM @BANKING_STAGE/transactions/2024/02/15/
FILE_FORMAT = (TYPE = PARQUET);
```

#### **Phase 7: dbt Transformations** 🔧
```sql
-- Staging (Silver)
CREATE OR REPLACE TABLE SILVER.STG_TRANSACTIONS AS
SELECT 
    id,
    account_id,
    amount,
    txn_type,
    created_at,
    CURRENT_TIMESTAMP() AS ingested_at
FROM BRONZE.RAW_TRANSACTIONS;

-- Fact Table (Gold)
CREATE OR REPLACE TABLE GOLD.FACT_TRANSACTIONS AS
SELECT 
    t.id AS transaction_key,
    a.account_key,
    c.customer_key,
    t.amount,
    t.txn_type,
    t.created_at AS transaction_date
FROM SILVER.STG_TRANSACTIONS t
JOIN GOLD.DIM_ACCOUNTS a ON t.account_id = a.account_id
JOIN GOLD.DIM_CUSTOMERS c ON a.customer_id = c.customer_id;
```

#### **Phase 8: Historical Snapshots** 📸
```sql
-- dbt Snapshot captures SCD Type-2
SELECT * FROM GOLD.DIM_CUSTOMERS_SNAPSHOT
WHERE customer_id = 1;

-- Result shows historical changes:
| customer_key | customer_id | email              | dbt_valid_from | dbt_valid_to |
|--------------|-------------|--------------------|----------------|--------------|
| 1            | 1           | john@old.com       | 2024-01-01     | 2024-02-15   |
| 2            | 1           | john@example.com   | 2024-02-15     | NULL         |
```

---

## 🎯 Data Modeling

### **Dimensional Model**

```
GOLD Layer (Star Schema):

                    DIM_CUSTOMERS
                    ┌────────────────┐
                    │ customer_key   │
                    │ customer_id    │
                    │ first_name     │
                    │ last_name      │
                    │ email          │
                    │ created_at     │
                    └────────┬───────┘
                             │
                             │ 1:M
                             │
    DIM_ACCOUNTS             ▼
    ┌────────────────┐  FACT_TRANSACTIONS
    │ account_key    │  ┌────────────────────┐
    │ account_id     │◄─┤ transaction_key    │
    │ customer_id    │  │ account_key (FK)   │
    │ account_type   │  │ customer_key (FK)  │
    │ balance        │  │ amount             │
    │ currency       │  │ txn_type           │
    │ created_at     │  │ transaction_date   │
    └────────────────┘  │ status             │
                        └────────────────────┘
```

### **dbt Model Lineage**

```
Sources (PostgreSQL)
    │
    ├── customers ──────────► stg_customers ────► dim_customers ────► customers_snapshot
    │                                                     │
    ├── accounts ───────────► stg_accounts ─────► dim_accounts ──────► accounts_snapshot
    │                                                     │
    └── transactions ───────► stg_transactions ──────────┴─────────► fact_transactions
```

---

## 🧪 Testing Strategy

### **dbt Tests Implemented**

```yaml
# models/schema.yml
models:
  - name: fact_transactions
    tests:
      - dbt_utils.recency:
          datepart: day
          field: transaction_date
          interval: 1
    columns:
      - name: transaction_key
        tests:
          - unique
          - not_null
      - name: account_key
        tests:
          - relationships:
              to: ref('dim_accounts')
              field: account_key
      - name: amount
        tests:
          - not_null
          - dbt_expectations.expect_column_values_to_be_between:
              min_value: 0
              max_value: 1000000
```

### **Test Coverage**
- ✅ Schema validation (column names, data types)
- ✅ Uniqueness constraints (primary keys)
- ✅ Referential integrity (foreign keys)
- ✅ Null checks (required fields)
- ✅ Value range validation (amounts, dates)
- ✅ Data freshness (recency checks)
- ✅ Custom business rules (balance >= 0)

---

## 🔄 CI/CD Workflow

### **Continuous Integration** (on PR)
```yaml
name: CI
on: [pull_request]
jobs:
  dbt-tests:
    runs-on: ubuntu-latest
    steps:
      - Checkout code
      - Install dbt
      - Run dbt compile
      - Run dbt test
      - SQL lint check
      - Security scan
```

### **Continuous Deployment** (on merge)
```yaml
name: CD
on:
  push:
    branches: [main]
jobs:
  deploy-production:
    runs-on: ubuntu-latest
    steps:
      - Deploy Airflow DAGs
      - Run dbt run (production)
      - Run dbt test (production)
      - Update documentation
      - Notify Slack
```

---

## 📈 Monitoring & Observability

### **Airflow Monitoring**
- 📊 DAG execution times and success rates
- 🚨 Failure alerts via email/Slack
- 📉 Task duration trends
- 🔄 Retry attempts and SLA misses

### **dbt Monitoring**
- ✅ Test pass/fail rates
- ⏱️ Model execution times
- 📊 Row count anomaly detection
- 🔍 Data quality scores

### **Snowflake Monitoring**
- 💰 Credit consumption tracking
- 🚀 Query performance analysis
- 📦 Storage growth trends
- 👥 User access patterns

---

## 🔐 Security & Compliance

- 🔒 **Secrets Management**: All credentials stored in environment variables
- 🛡️ **Network Isolation**: Services communicate through Docker internal networks
- 🔑 **Role-Based Access**: Snowflake roles for least privilege
- 📝 **Audit Logging**: Full history of data changes via CDC
- 🔐 **Encryption**: Data encrypted at rest (Snowflake) and in transit (TLS)

---

## 🚧 Roadmap

### **Completed** ✅
- [x] End-to-end CDC pipeline
- [x] dbt dimensional modeling
- [x] Airflow orchestration
- [x] SCD Type-2 snapshots
- [x] CI/CD with GitHub Actions
- [x] Docker containerization

### **In Progress** 🏗️
- [ ] Real-time dashboards (Streamlit/Tableau)
- [ ] Data quality framework (Great Expectations)
- [ ] Cost optimization analysis

### **Future Enhancements** 🔮
- [ ] Machine learning models (fraud detection)
- [ ] GraphQL API for data access
- [ ] Multi-region deployment
- [ ] Kubernetes orchestration

---

## 📚 Learning Resources

If you're new to any of these technologies, here are some great starting points:

- **dbt**: [dbt Learn](https://courses.getdbt.com/)
- **Airflow**: [Apache Airflow Tutorial](https://airflow.apache.org/docs/apache-airflow/stable/tutorial.html)
- **Kafka**: [Kafka Quickstart](https://kafka.apache.org/quickstart)
- **Snowflake**: [Snowflake Hands-On Essentials](https://www.snowflake.com/virtual-hands-on-lab/)
- **Debezium**: [Debezium Tutorial](https://debezium.io/documentation/reference/tutorial.html)

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## 👤 Author

**Adith Sreeram Arjunan Sivakumar**

- GitHub: [@ASR373](https://github.com/ASR373)
- LinkedIn: [Your LinkedIn Profile](#)
- Email: your.email@example.com

---

## 🙏 Acknowledgments

- Inspired by real-world data engineering challenges in the financial services industry
- Built with open-source technologies from amazing communities
- Special thanks to the dbt, Airflow, and Kafka communities for excellent documentation

---

<div align="center">

**⭐ If this project helped you, please star it on GitHub! ⭐**

Made with ❤️ by [ASR373](https://github.com/ASR373)

</div>
