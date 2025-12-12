# Employee Data Pipeline

![Status](https://img.shields.io/badge/status-active-success.svg)
![Python](https://img.shields.io/badge/python-3.12-blue.svg)
![Coverage](https://img.shields.io/badge/coverage-90%25-brightgreen.svg)

## Описание
Простой, но полноценный ETL-пайплайн для генерации, обработки и анализа синтетических данных о сотрудниках. Проект демонстрирует современные практики разработки на Python (SOLID, Type Hinting, Testing).

## Структура Проекта (Architecture)
Проект построен по модульному принципу:
- **Generate**: Генерация данных (Faker).
- **Transform**: Загрузка (Pandas) и Анализ данных.
- **Load**: Сохранение отчетов (Excel).

```mermaid
sequenceDiagram
    participant Main as Main Pipeline
    participant Gen as Generator (Faker)
    participant Loader as Loader (Pandas)
    participant Analyzer as Analyzer (Pandas)
    participant Reporter as Reporter (Excel)

    Main->>Gen: generate(count=1000)
    Gen-->>Main: CSV Path (Data/Date)
    Main->>Loader: load(CSV Path)
    Loader-->>Main: DataFrame
    Main->>Analyzer: analyze(DataFrame)
    Analyzer-->>Main: Metrics Dict
    Main->>Reporter: save_report(Metrics, Path)
    Reporter-->>Main: Done (Reports/Date)
```

## Установка

1. Убедитесь, что установлен Python 3.12+ и Poetry.
2. Клонируйте репозиторий.
3. Установите зависимости:
```bash
make install
# или
poetry install
```

## Запуск
```bash
make run
# или
poetry run python -m employee_pipeline.main
```
Результаты будут сохранены в:
- `output/data/employees_YYYY-MM-DD.csv`
- `output/reports/report_YYYY-MM-DD.xlsx`

## Тестирование и QA
### Общий запуск
Все тесты с проверкой покрытия:
```bash
make test
# или
poetry run pytest --cov=employee_pipeline
```

### Запуск по модулям
Можно запускать тесты для каждого модуля отдельно:

**Domain (Models & Interfaces):**
```bash
make test-domain
# или
poetry run pytest tests/test_domain.py
```

**Generation:**
```bash
make test-generate
# или
poetry run pytest tests/test_generate.py
```

**Transformation (Load & Analyze):**
```bash
make test-transform
# или
poetry run pytest tests/test_transform.py
```

**Reporting:**
```bash
make test-load
# или
poetry run pytest tests/test_load.py
```

### Линтинг и Типы
```bash
make lint
# или
poetry run ruff check . && poetry run mypy .
```

## Документация
Сборка документации MkDocs:
```bash
make docs
```

## CI/CD
Настроен GitHub Actions workflow (`.github/workflows/ci.yml`), который автоматически запускает линтинг и тесты при пуше в `main`.

## Технологии
- **Python 3.12**
- **Poetry**
- **Pandas**
- **Faker**
- **Mypy** (Strict)
- **Ruff**
- **MkDocs / Sphinx**
