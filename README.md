# 🏦 Инструкция по запуску Banking Modern Data Stack с ClickHouse

Этот проект демонстрирует **end-to-end modern data stack pipeline** для банковского домена с использованием ClickHouse вместо Snowflake.

---

## 📋 Требования к системе

### Необходимое ПО
- **Docker Desktop** (версия 20.10 или выше)
- **Docker Compose** (версия 1.29 или выше)
- **Python** 3.8+ (для локального запуска генератора данных)
- **Git** (для клонирования репозитория)
- Минимум **8 GB RAM** для комфортной работы всех контейнеров
- Минимум **10 GB** свободного места на диске

### Порты (должны быть свободны)
- `5432` - PostgreSQL (OLTP база)
- `5433` - Airflow PostgreSQL
- `2181` - Zookeeper
- `9092`, `29092` - Kafka
- `8083` - Kafka Connect (Debezium)
- `9000` - MinIO API
- `9001` - MinIO Console
- `8123` - ClickHouse HTTP
- `9002` - ClickHouse Native Protocol
- `8080` - Airflow Web UI

---

## 🚀 Шаг 1: Клонирование репозитория

```bash
git clone <URL вашего репозитория>
cd banking-modern-datastack
```

---

## ⚙️ Шаг 2: Настройка переменных окружения

1. Скопируйте пример файла с переменными окружения:

```bash
# Windows PowerShell
Copy-Item .env.example .env

# или вручную создайте файл .env
```

2. Откройте файл `.env` и при необходимости измените параметры (по умолчанию все настроено):

\`\`\`env
# PostgreSQL (OLTP Source)
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=banking
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres123

# ClickHouse
CLICKHOUSE_HOST=clickhouse
CLICKHOUSE_PORT=9002
CLICKHOUSE_USER=default
CLICKHOUSE_PASSWORD=clickhouse123
CLICKHOUSE_DB=banking

# MinIO
MINIO_ENDPOINT=http://host.docker.internal:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET=banking-raw

# Kafka
KAFKA_BOOTSTRAP=host.docker.internal:29092
KAFKA_GROUP=banking-cdc-consumer
\`\`\`

---

## 🐳 Шаг 3: Запуск инфраструктуры через Docker Compose

1. **Запустите все сервисы:**

\`\`\`bash
docker-compose up -d
\`\`\`

2. **Проверьте статус контейнеров:**

\`\`\`bash
docker-compose ps
\`\`\`

Все сервисы должны быть в статусе `Up`:
- `zookeeper`
- `kafka`
- `connect` (Debezium)
- `postgres`
- `clickhouse`
- `minio`
- `airflow-webserver`
- `airflow-scheduler`
- `airflow-postgres`

3. **Просмотр логов (если что-то не работает):**

\`\`\`bash
# Все сервисы
docker-compose logs -f

# Конкретный сервис
docker-compose logs -f clickhouse
docker-compose logs -f kafka
\`\`\`

---

## 🗄️ Шаг 4: Инициализация PostgreSQL (OLTP база)

1. **Подключитесь к PostgreSQL контейнеру:**

\`\`\`bash
docker exec -it <POSTGRES_CONTAINER_ID> psql -U postgres -d banking
\`\`\`

Или используйте GUI клиент (DBeaver, pgAdmin):
- Host: `localhost`
- Port: `5432`
- Database: `banking`
- User: `postgres`
- Password: `postgres123`

2. **Выполните SQL скрипт для создания схемы:**

\`\`\`bash
docker exec -i <POSTGRES_CONTAINER_ID> psql -U postgres -d banking < postgres/schema.sql
\`\`\`

Или скопируйте содержимое файла `postgres/schema.sql` и выполните его вручную.

---

## 🔌 Шаг 5: Настройка Debezium Connector (CDC)

1. **Дождитесь, пока Kafka Connect будет готов** (обычно 30-60 секунд после запуска):

\`\`\`bash
# Проверка статуса Kafka Connect
curl http://localhost:8083/connectors
\`\`\`

Должен вернуть `[]` (пустой массив).

2. **Установите зависимости Python:**

\`\`\`bash
pip install -r requirements.txt
\`\`\`

3. **Запустите скрипт для создания Debezium connector:**

\`\`\`bash
python kafka-debezium/generate_and_post_connector.py
\`\`\`

4. **Проверьте, что connector создан:**

\`\`\`bash
curl http://localhost:8083/connectors
\`\`\`

Должен вернуть список с вашим connector.

---

## 📊 Шаг 6: Генерация тестовых данных

1. **Запустите генератор данных один раз:**

\`\`\`bash
python data-generator/faker_generator.py --once
\`\`\`

Эта команда сгенерирует:
- 10 клиентов
- 20 счетов (по 2 на клиента)
- 50 транзакций

2. **Для непрерывной генерации данных** (запускается в фоне):

\`\`\`bash
python data-generator/faker_generator.py
\`\`\`

Программа будет генерировать данные каждые 2 секунды. Нажмите `Ctrl+C` для остановки.

---

## 📦 Шаг 7: Запуск Kafka Consumer (MinIO)

**Consumer читает данные из Kafka и сохраняет в MinIO в формате Parquet.**

1. **Запустите consumer:**

\`\`\`bash
python consumer/kafka_to_minio.py
\`\`\`

2. **Проверьте MinIO Console:**

Откройте в браузере: http://localhost:9001

Логин: `minioadmin`  
Пароль: `minioadmin`

В bucket `banking-raw` должны появиться папки:
- `customers/`
- `accounts/`
- `transactions/`

С файлами в формате Parquet.

---

## ✈️ Шаг 8: Инициализация Airflow

1. **Создайте администратора Airflow (только при первом запуске):**

\`\`\`bash
docker exec -it airflow-webserver airflow users create \\
    --username admin \\
    --firstname Admin \\
    --lastname User \\
    --role Admin \\
    --email admin@example.com \\
    --password admin
\`\`\`

2. **Откройте Airflow UI:**

URL: http://localhost:8080

Логин: `admin`  
Пароль: `admin`

3. **Включите DAGs:**

Найдите и включите (toggle ON) следующие DAGs:
- `minio_to_clickhouse_banking` - загрузка данных из MinIO в ClickHouse (каждые 5 минут)
- `SCD2_snapshots` - создание снапшотов для SCD Type 2 (ежедневно)

4. **Запустите DAG вручную (для тестирования):**

Нажмите кнопку "▶" рядом с DAG `minio_to_clickhouse_banking`.

---

## 🎯 Шаг 9: Настройка DBT

1. **Войдите в контейнер Airflow:**

\`\`\`bash
docker exec -it airflow-scheduler bash
\`\`\`

2. **Скопируйте профиль DBT:**

\`\`\`bash
mkdir -p /home/airflow/.dbt
cp /opt/airflow/banking_dbt/profiles/profiles.yml /home/airflow/.dbt/profiles.yml
\`\`\`

3. **Проверьте подключение DBT к ClickHouse:**

\`\`\`bash
cd /opt/airflow/banking_dbt
dbt debug --profiles-dir /home/airflow/.dbt
\`\`\`

Должно вывести: `All checks passed!`

4. **Установите зависимости DBT:**

\`\`\`bash
dbt deps --profiles-dir /home/airflow/.dbt
\`\`\`

---

## 🔄 Шаг 10: Запуск DBT трансформаций

1. **Выполните DBT models (staging + marts):**

\`\`\`bash
cd /opt/airflow/banking_dbt
dbt run --profiles-dir /home/airflow/.dbt
\`\`\`

Это создаст:
- **Staging views**: `stg_customers`, `stg_accounts`, `stg_transactions`
- **Marts tables**: `dim_customers`, `dim_accounts`, `fact_transactions`

2. **Выполните snapshots (SCD Type 2):**

\`\`\`bash
dbt snapshot --profiles-dir /home/airflow/.dbt
\`\`\`

Это создаст:
- `customers_snapshot`
- `accounts_snapshot`

3. **Запустите тесты DBT:**

\`\`\`bash
dbt test --profiles-dir /home/airflow/.dbt
\`\`\`

---

## ✅ Шаг 11: Проверка данных в ClickHouse

1. **Откройте ClickHouse Play UI:**

URL: http://localhost:8123/play

2. **Выполните запросы для проверки данных:**

\`\`\`sql
-- Проверка raw таблиц
SELECT count() FROM banking.raw_customers;
SELECT count() FROM banking.raw_accounts;
SELECT count() FROM banking.raw_transactions;

-- Проверка staging views
SELECT count() FROM banking.silver.stg_customers;
SELECT count() FROM banking.silver.stg_accounts;
SELECT count() FROM banking.silver.stg_transactions;

-- Проверка marts
SELECT * FROM banking.silver.dim_customers LIMIT 10;
SELECT * FROM banking.silver.dim_accounts LIMIT 10;
SELECT * FROM banking.silver.fact_transactions LIMIT 10;

-- Проверка snapshots
SELECT count() FROM banking.gold.customers_snapshot;
SELECT count() FROM banking.gold.accounts_snapshot;
\`\`\`

3. **Аналитический запрос (пример):**

\`\`\`sql
SELECT 
    c.first_name,
    c.last_name,
    COUNT(t.transaction_id) as total_transactions,
    SUM(t.amount) as total_amount
FROM banking.silver.fact_transactions t
JOIN banking.silver.dim_customers c ON t.customer_id = c.customer_id
WHERE c.is_current = 1
GROUP BY c.first_name, c.last_name
ORDER BY total_amount DESC
LIMIT 10;
\`\`\`

---

## 🔄 Полный цикл работы системы

### Поток данных:

1. **Data Generator** → генерирует данные в PostgreSQL
2. **Debezium** → захватывает изменения (CDC) из PostgreSQL
3. **Kafka** → стримит события изменений
4. **Consumer** → читает из Kafka и пишет Parquet файлы в MinIO
5. **Airflow DAG** → периодически загружает данные из MinIO в ClickHouse (raw таблицы)
6. **DBT** → трансформирует raw → staging → marts/dimensions/facts
7. **Snapshots** → создают исторические снимки для SCD Type 2

### Архитектурные слои в ClickHouse:

- **Bronze (Raw)**: `banking.raw_*` - исходные данные из MinIO
- **Silver (Staging)**: `banking.silver.stg_*` - очищенные view
- **Gold (Marts)**: `banking.silver.dim_*`, `banking.silver.fact_*`, `banking.gold.*_snapshot` - готовые для аналитики

---

## 🛑 Остановка системы

\`\`\`bash
# Остановить все контейнеры
docker-compose down

# Остановить и удалить все данные (volumes)
docker-compose down -v
\`\`\`

---

## 🔧 Troubleshooting

### Проблема: Контейнер не запускается

\`\`\`bash
# Просмотр логов
docker-compose logs -f <service_name>

# Перезапуск контейнера
docker-compose restart <service_name>
\`\`\`

### Проблема: Порт уже занят

Измените порты в `docker-compose.yml` на свободные.

### Проблема: DBT не может подключиться к ClickHouse

1. Проверьте, что ClickHouse запущен:
\`\`\`bash
docker ps | grep clickhouse
\`\`\`

2. Проверьте переменные окружения в `.env`

3. Проверьте `profiles.yml`

### Проблема: Нет данных в MinIO

1. Проверьте, что Kafka работает
2. Проверьте, что Debezium connector создан
3. Проверьте логи consumer

### Проблема: Данные не попадают в ClickHouse

1. Проверьте логи Airflow DAG
2. Проверьте, что файлы есть в MinIO
3. Проверьте подключение к ClickHouse из Airflow:
\`\`\`bash
docker exec -it airflow-scheduler bash
python -c "from clickhouse_driver import Client; client = Client(host='clickhouse', port=9002); print(client.execute('SELECT version()'))"
\`\`\`

---

## 📞 Поддержка

При возникновении вопросов создайте Issue в репозитории или свяжитесь с maintainer.

**Удачи в изучении Modern Data Stack! 🚀**
