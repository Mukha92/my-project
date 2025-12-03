# 🏦 Banking Modern Data Stack

![ClickHouse](https://img.shields.io/badge/ClickHouse-FFCC01?logo=clickhouse&logoColor=black)
![DBT](https://img.shields.io/badge/dbt-FF694B?logo=dbt&logoColor=white)
![Apache Airflow](https://img.shields.io/badge/Apache%20Airflow-017CEE?logo=apacheairflow&logoColor=white)
![Apache Kafka](https://img.shields.io/badge/Apache%20Kafka-231F20?logo=apachekafka&logoColor=white)
![Debezium](https://img.shields.io/badge/Debezium-EF3B2D?logo=apache&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![Git](https://img.shields.io/badge/Git-F05032?logo=git&logoColor=white)
![CI/CD](https://img.shields.io/badge/CI%2FCD-000000?logo=githubactions&logoColor=white)

---

## 📌 Project Overview

This project demonstrates an **end-to-end modern data stack pipeline** for a **Banking domain**.  
We simulate **customer, account, and transaction data**, stream changes in real time, transform them into analytics-ready models, and visualize insights — following **best practices of CI/CD and data warehousing**.

👉 Think of it as a **real-world banking data ecosystem** built on modern data tools with **ClickHouse** as the analytical database.

---

## 🏗️ Architecture  

**Pipeline Flow:**
1. **Data Generator** → Simulates banking transactions, accounts & customers (via Faker).  
2. **Kafka + Debezium** → Streams change data (CDC) from PostgreSQL into MinIO (S3-compatible storage).  
3. **Airflow** → Orchestrates data ingestion from MinIO to ClickHouse.  
4. **ClickHouse** → OLAP Data Warehouse (Bronze → Silver → Gold).  
5. **DBT** → Applies transformations, builds marts & snapshots (SCD Type-2).  
6. **CI/CD with GitHub Actions** → Automated tests, build & deployment.  

---

## ⚡ Tech Stack

- **ClickHouse** → Fast OLAP Database for Analytics  
- **DBT** → Transformations, testing, snapshots (SCD Type-2)  
- **Apache Airflow** → Orchestration & DAG scheduling  
- **Apache Kafka + Debezium** → Real-time streaming & CDC  
- **MinIO** → S3-compatible object storage  
- **Postgres** → Source OLTP system  
- **Python (Faker)** → Data simulation  
- **Docker & docker-compose** → Containerized setup  
- **Git & GitHub Actions** → CI/CD workflows  

---

## ✅ Key Features

- **PostgreSQL OLTP**: Source relational database with ACID guarantees (customers, accounts, transactions)  
- **Simulated banking system**: customers, accounts, and transactions  
- **Change Data Capture (CDC)** via Kafka + Debezium (capturing Postgres WAL)  
- **Raw → Staging → Fact/Dimension** models in DBT  
- **Snapshots for history tracking** (slowly changing dimensions)  
- **Automated pipeline orchestration** using Airflow  
- **CI/CD pipeline** with dbt tests + GitHub Actions  
- **ClickHouse as analytical database** with MergeTree engine for high performance

---

## 📂 Repository Structure

```text
banking-modern-datastack/
├── .github/workflows/         # CI/CD pipelines (ci.yml, cd.yml)
├── banking_dbt/              # DBT project
│   ├── models/
│   │   ├── staging/           # Staging models
│   │   ├── marts/             # Facts & dimensions
│   │   └── sources.yml
│   ├── snapshots/             # SCD2 snapshots
│   ├── profiles/              # DBT profiles (ClickHouse connection)
│   └── dbt_project.yml
├── consumer/
│   └── kafka_to_minio.py      # Kafka consumer → MinIO writer
├── data-generator/            # Faker-based data simulator
│   └── faker_generator.py
├── docker/                    # Airflow DAGs, ClickHouse init scripts
│   ├── dags/                  # DAGs (minio_to_clickhouse, scd_snapshots)
│   └── clickhouse/
│       └── init/              # ClickHouse initialization SQL
├── kafka-debezium/            # Kafka connectors & CDC logic
│   └── generate_and_post_connector.py
├── postgres/                  # Postgres schema (OLTP DDL & seeds)
│   └── schema.sql
├── .env.example               # Example environment variables
├── .gitignore
├── docker-compose.yml         # Containerized infrastructure
├── dockerfile-airflow.dockerfile
├── requirements.txt
├── INSTALLATION_RU.md         # 🇷🇺 Detailed installation guide (Russian)
├── USAGE_RU.md                # 🇷🇺 Usage guide (Russian)
└── README.md
```

---

## 🚀 Quick Start

### Prerequisites
- Docker Desktop (20.10+)
- Docker Compose (1.29+)
- Python 3.8+ (for data generator)
- 8 GB RAM minimum
- 10 GB free disk space

### Installation

**📖 Detailed Russian guide:** [INSTALLATION_RU.md](INSTALLATION_RU.md)

1. **Clone the repository:**
```bash
git clone <repository_url>
cd banking-modern-datastack
```

2. **Set up environment variables:**
```bash
cp .env.example .env
# Edit .env if needed
```

3. **Start all services:**
```bash
docker-compose up -d
```

4. **Initialize PostgreSQL schema:**
```bash
docker exec -i <postgres_container> psql -U postgres -d banking < postgres/schema.sql
```

5. **Install Python dependencies:**
```bash
pip install -r requirements.txt
```

6. **Create Debezium connector:**
```bash
python kafka-debezium/generate_and_post_connector.py
```

7. **Generate test data:**
```bash
python data-generator/faker_generator.py --once
```

8. **Start Kafka consumer:**
```bash
python consumer/kafka_to_minio.py
```

9. **Initialize Airflow:**
```bash
docker exec -it airflow-webserver airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password admin
```

10. **Access Airflow UI and trigger DAGs:**
- URL: http://localhost:8080
- Login: admin / admin
- Enable and run: `minio_to_clickhouse_banking`

11. **Run DBT models:**
```bash
docker exec -it airflow-scheduler bash
cd /opt/airflow/banking_dbt
cp profiles/profiles.yml /home/airflow/.dbt/profiles.yml
dbt run --profiles-dir /home/airflow/.dbt
dbt snapshot --profiles-dir /home/airflow/.dbt
```

---

## 🌐 Access Points

| Service | URL | Credentials |
|---------|-----|-------------|
| Airflow UI | http://localhost:8080 | admin / admin |
| MinIO Console | http://localhost:9001 | minioadmin / minioadmin |
| ClickHouse Play | http://localhost:8123/play | - |
| PostgreSQL | localhost:5432 | postgres / postgres123 |
| ClickHouse Native | localhost:9002 | default / clickhouse123 |

---

## 📊 Data Flow

1. **PostgreSQL** ← Faker generates banking data
2. **Debezium** ← Captures changes from PostgreSQL (CDC)
3. **Kafka** ← Streams change events
4. **Consumer** ← Reads from Kafka, writes Parquet to MinIO
5. **Airflow DAG** ← Loads data from MinIO to ClickHouse (raw tables)
6. **DBT** ← Transforms: raw → staging → marts (dimensions/facts)
7. **Snapshots** ← Creates historical snapshots (SCD Type-2)

### ClickHouse Layers

- **Bronze (Raw)**: `banking.raw_*` - Raw data from MinIO
- **Silver (Staging)**: `banking.silver.stg_*` - Cleaned views
- **Gold (Marts)**: `banking.silver.dim_*`, `banking.silver.fact_*`, `banking.gold.*_snapshot` - Analytics-ready

---

## 📖 Documentation

- **🇷🇺 [Подробная инструкция по установке (Russian)](INSTALLATION_RU.md)**
- **🇷🇺 [Руководство по использованию (Russian)](USAGE_RU.md)**

---

## 🔍 Sample Queries

```sql
-- Top 10 customers by transaction volume
SELECT 
    c.first_name,
    c.last_name,
    COUNT(t.transaction_id) as total_transactions,
    ROUND(SUM(t.amount), 2) as total_amount
FROM banking.silver.fact_transactions t
JOIN banking.silver.dim_customers c ON t.customer_id = c.customer_id
WHERE c.is_current = 1
GROUP BY c.first_name, c.last_name
ORDER BY total_amount DESC
LIMIT 10;

-- Account type statistics
SELECT 
    account_type,
    COUNT(*) as total_accounts,
    ROUND(AVG(balance), 2) as avg_balance,
    ROUND(SUM(balance), 2) as total_balance
FROM banking.silver.dim_accounts
WHERE is_current = 1
GROUP BY account_type;
```

---

## 🛠️ Development

### Adding New Tables
1. Create table in PostgreSQL (`postgres/schema.sql`)
2. Update Debezium connector
3. Add to consumer (`consumer/kafka_to_minio.py`)
4. Create raw table in ClickHouse (`docker/clickhouse/init/01_init.sql`)
5. Add to Airflow DAG
6. Create DBT staging model
7. Create marts as needed

### Running Tests
```bash
cd banking_dbt
dbt test --profiles-dir /home/airflow/.dbt
```

---

## 📊 CI/CD

The project includes GitHub Actions workflows:
- **ci.yml** → Lint, dbt compile, run tests
- **cd.yml** → Deploy DAGs & dbt models on merge

---

## 🐛 Troubleshooting

Common issues and solutions:

1. **Container won't start**: Check logs with `docker-compose logs -f <service>`
2. **Port conflicts**: Modify ports in `docker-compose.yml`
3. **DBT connection issues**: Verify `profiles.yml` and environment variables
4. **No data in ClickHouse**: Check Airflow DAG logs and MinIO contents

For detailed troubleshooting, see [USAGE_RU.md](USAGE_RU.md).

---

## 📈 Performance

ClickHouse provides:
- **Fast analytical queries** with columnar storage
- **Efficient compression** ~10x compression ratio
- **Real-time ingestion** thousands of rows per second
- **Horizontal scaling** for large datasets

---

## 🤝 Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

---

## 📞 Support

For questions or issues:
- Create an Issue in this repository
- Contact the maintainer

---

## 📜 License

This project is open-source and available under the MIT License.

---

**Author**: *Banking Data Engineering Team*  
**Powered by**: ClickHouse, DBT, Airflow, Kafka & Modern Data Stack

**Удачи в изучении Modern Data Stack! 🚀**
