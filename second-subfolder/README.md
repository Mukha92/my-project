# Employee Data Pipeline

## Описание Проекта
Простой, но мощный ETL-пайплайн для генерации, обработки и анализа синтетических данных о сотрудниках. Этот проект служит эталонной реализацией production-grade приложения на Python, следующего современным лучшим практикам и принципам SOLID.

## Функциональность
- **Генерация Данных (Generate)**: масштабируемая генерация синтетических данных с использованием `Faker`.
- **ETL Процесс**: Валидация, загрузка, проверка качества и трансформация данных.
- **Анализ (Analyze)**: Расчет ключевых HR-метрик (статистика по зарплатам, удержание, возрастное распределение).
- **Отчетность (Report)**: Экспорт консолидированных отчетов в Excel.
- **Высокие Стандарты Качества**:
    - **100% Типобезопасность** (`mypy --strict`).
    - **Тестирование** (`pytest` с покрытием 90%+).
    - **Линтинг** (`ruff`, `codespell`).
    - **Документирование** (Google-style docstrings, Sphinx/MkDocs).

## Этапы Пайплайна
1. **Generate**: Создание `employees.csv` с синтетическими данными.
2. **Check & Load**: Проверка наличия файла и схемы, загрузка в `pandas`.
3. **Analyze**: Расчет метрик (средняя зарплата, отделы и т.д.).
4. **Report**: Сохранение результатов в `report.xlsx`.

## Архитектура (SOLID)
- **SRP**: отдельные модули для Генерации, Загрузки, Анализа, Отчетности.
- **OCP**: Интерфейсы для Источников Данных и Форматов Отчетов позволяют расширение без модификации.
- **LSP**: Подклассы `DataSource` или `ReportGenerator` взаимозаменяемы.
- **ISP**: Маленькие, специфические интерфейсы (например, `IDataLoader`, `IMetricsCalculator`).
- **DIP**: Высокоуровневый пайплайн зависит от абстракций, а не от конкретных реализаций.

## Поток Пайплайна (Sequence Diagram)

```mermaid
sequenceDiagram
    participant Main
    participant Pipeline
    participant Generator
    participant Loader
    participant Analyzer
    participant Reporter

    Main->>Pipeline: run(count)
    Pipeline->>Generator: generate(csv_path, count)
    Generator-->>Pipeline: (file created)
    Pipeline->>Loader: load(csv_path)
    Loader-->>Pipeline: DataFrame
    Pipeline->>Analyzer: analyze(DataFrame)
    Analyzer-->>Pipeline: AnalysisResult
    Pipeline->>Reporter: save_report(result, report.xlsx)
    Reporter-->>Pipeline: (report saved)
    Pipeline-->>Main: Success
```

## Архитектура (Class Diagram)

```mermaid
classDiagram
    class IDataGenerator {
        <<interface>>
        +generate(path, count)
    }
    class IDataLoader {
        <<interface>>
        +load(path)
    }
    class IAnalyzer {
        <<interface>>
        +analyze(df)
    }
    class IReporter {
        <<interface>>
        +save_report(result, path)
    }

    class FakerGenerator {
        +generate(path, count)
    }
    class CSVLoader {
        +load(path)
    }
    class PandasAnalyzer {
        +analyze(df)
    }
    class ExcelReporter {
        +save_report(result, path)
    }

    class ETLPipeline {
        +run(csv, report, count)
    }

    ETLPipeline --> IDataGenerator
    ETLPipeline --> IDataLoader
    ETLPipeline --> IAnalyzer
    ETLPipeline --> IReporter

    FakerGenerator ..|> IDataGenerator
    CSVLoader ..|> IDataLoader
    PandasAnalyzer ..|> IAnalyzer
    ExcelReporter ..|> IReporter
```

## Инфраструктура
- **Управление Зависимостями**: Poetry.
- **CI/CD**: GitHub Actions (Lint, Test, Type Check).
- **Автоматизация**: Makefile для типовых задач.
