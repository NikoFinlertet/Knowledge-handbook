# Data Warehousing - Хранилища данных 🏭

## 🏠 Навигация
**[[README|🏠 Базы данных]]** → **Data Warehousing**

**Связанные темы:**
- 🐘 [[postgresql|PostgreSQL]]
- 🍃 [[mongodb|MongoDB]]  
- 🔴 [[redis|Redis]]
- 🏗️ [[database-design|Проектирование БД]]
- ⚡ [[sql-optimization|Оптимизация БД]]

---

## 🎯 Что такое Data Warehousing?

**Хранилище данных** (Data Warehouse) — это централизованный репозиторий, который позволяет хранить данные из различных источников для анализа и принятия решений. Это основа для бизнес-аналитики и систем поддержки принятия решений.

### ✨ Ключевые характеристики

1. **📊 Предметно-ориентированное** - данные организованы по бизнес-областям
2. **🔄 Интегрированное** - данные из разных источников в едином формате
3. **📈 Неизменяемое** - исторические данные не изменяются
4. **⏰ Зависящее от времени** - все данные привязаны к временным периодам

---

## 🏗️ OLTP vs OLAP

### 📝 Сравнительная таблица

| Характеристика | OLTP | OLAP |
|---|---|---|
| **Назначение** | Обработка транзакций | Аналитические запросы |
| **Операции** | INSERT, UPDATE, DELETE | SELECT (сложные) |
| **Объем данных** | Небольшие запросы | Большие объемы |
| **Пользователи** | Много одновременных | Аналитики, менеджеры |
| **Структура** | Нормализованная | Денормализованная |
| **Примеры** | PostgreSQL, MySQL | ClickHouse, Snowflake |

```python
# Пример различий в структуре данных

# OLTP - Нормализованная структура
class OLTPStructure:
    """Структура OLTP системы"""
    orders_table = {
        'id': 'PRIMARY KEY',
        'user_id': 'FOREIGN KEY',
        'created_at': 'TIMESTAMP',
        'status': 'VARCHAR'
    }
    
    order_items_table = {
        'id': 'PRIMARY KEY', 
        'order_id': 'FOREIGN KEY',
        'product_id': 'FOREIGN KEY',
        'quantity': 'INTEGER',
        'price': 'DECIMAL'
    }

# OLAP - Денормализованная структура для быстрой аналитики
class OLAPStructure:
    """Структура OLAP системы"""
    sales_fact_table = {
        'order_id': 'BIGINT',
        'user_id': 'BIGINT',
        'user_name': 'VARCHAR',
        'user_city': 'VARCHAR',
        'user_country': 'VARCHAR',
        'product_id': 'BIGINT',
        'product_name': 'VARCHAR',
        'product_category': 'VARCHAR',
        'quantity': 'INTEGER',
        'unit_price': 'DECIMAL',
        'total_amount': 'DECIMAL',
        'order_date': 'DATE',
        'year': 'INTEGER',
        'quarter': 'INTEGER',
        'month': 'INTEGER'
    }
```

---

## ⭐ Схема "Звезда" (Star Schema)

### 🏗️ Архитектура Data Warehouse

```python
# services/data-warehouse/architecture.py
from dataclasses import dataclass
from typing import List, Dict, Any
from datetime import datetime
import pandas as pd

@dataclass
class DimensionTable:
    """Таблица измерений"""
    name: str
    primary_key: str
    attributes: List[str]
    slowly_changing_type: int  # SCD Type 1, 2, 3

@dataclass
class FactTable:
    """Таблица фактов"""
    name: str
    dimensions: List[str]  # Внешние ключи
    measures: List[str]    # Метрики
    grain: str            # Уровень детализации

class StarSchemaBuilder:
    """Построение схемы звезда"""
    
    def __init__(self):
        self.dimensions = {}
        self.facts = {}
    
    def add_dimension(self, dimension: DimensionTable):
        """Добавление таблицы измерений"""
        self.dimensions[dimension.name] = dimension
    
    def add_fact(self, fact: FactTable):
        """Добавление таблицы фактов"""
        self.facts[fact.name] = fact
    
    def generate_sql_schema(self) -> str:
        """Генерация SQL схемы"""
        sql_statements = []
        
        # Создание таблиц измерений
        for dim in self.dimensions.values():
            sql = f"""
CREATE TABLE {dim.name} (
    {dim.primary_key} SERIAL PRIMARY KEY,
    {', '.join([f'{attr} VARCHAR(255)' for attr in dim.attributes])},
    valid_from TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    valid_to TIMESTAMP DEFAULT '9999-12-31',
    is_current BOOLEAN DEFAULT TRUE
);
            """
            sql_statements.append(sql.strip())
        
        # Создание таблиц фактов
        for fact in self.facts.values():
            foreign_keys = [f"{dim}_key INTEGER REFERENCES {dim}({self.dimensions[dim].primary_key})" 
                          for dim in fact.dimensions if dim in self.dimensions]
            measures_def = [f"{measure} DECIMAL(15,2)" for measure in fact.measures]
            
            sql = f"""
CREATE TABLE {fact.name} (
    {fact.name}_key SERIAL PRIMARY KEY,
    {', '.join(foreign_keys)},
    {', '.join(measures_def)},
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
            """
            sql_statements.append(sql.strip())
        
        return '\n\n'.join(sql_statements)

# Пример создания схемы для аналитики заказов
schema_builder = StarSchemaBuilder()

# Измерения
schema_builder.add_dimension(DimensionTable(
    name="dim_customer",
    primary_key="customer_key",
    attributes=["customer_id", "name", "email", "city", "country", "segment"],
    slowly_changing_type=2
))

schema_builder.add_dimension(DimensionTable(
    name="dim_product",
    primary_key="product_key", 
    attributes=["product_id", "name", "category", "brand", "price_range"],
    slowly_changing_type=1
))

schema_builder.add_dimension(DimensionTable(
    name="dim_date",
    primary_key="date_key",
    attributes=["date", "year", "quarter", "month", "day_of_week", "is_holiday"],
    slowly_changing_type=1
))

# Факты
schema_builder.add_fact(FactTable(
    name="fact_sales",
    dimensions=["dim_customer", "dim_product", "dim_date"],
    measures=["quantity", "unit_price", "total_amount", "discount"],
    grain="Один заказ на продукт"
))

print(schema_builder.generate_sql_schema())
```

---

## 🔄 ETL/ELT процессы

### 📥 Extract-Transform-Load (ETL)

```python
# services/etl/etl_pipeline.py
import pandas as pd
import psycopg2
from sqlalchemy import create_engine
from typing import Dict, List, Any
import logging
from datetime import datetime, timedelta

class ETLPipeline:
    def __init__(self, source_configs: Dict, target_config: Dict):
        self.source_configs = source_configs
        self.target_engine = create_engine(target_config['connection_string'])
        self.logger = logging.getLogger(__name__)
    
    def extract_from_postgresql(self, config: Dict) -> pd.DataFrame:
        """Извлечение данных из PostgreSQL"""
        source_engine = create_engine(config['connection_string'])
        
        query = config.get('query', f"SELECT * FROM {config['table']}")
        if 'incremental_column' in config:
            # Инкрементальная загрузка
            last_run = self.get_last_run_timestamp(config['table'])
            query += f" WHERE {config['incremental_column']} > '{last_run}'"
        
        return pd.read_sql(query, source_engine)
    
    def extract_from_mongodb(self, config: Dict) -> pd.DataFrame:
        """Извлечение данных из MongoDB"""
        from pymongo import MongoClient
        
        client = MongoClient(config['connection_string'])
        db = client[config['database']]
        collection = db[config['collection']]
        
        # Инкрементальная загрузка
        query = {}
        if 'incremental_field' in config:
            last_run = self.get_last_run_timestamp(config['collection'])
            query[config['incremental_field']] = {'$gt': last_run}
        
        cursor = collection.find(query)
        data = list(cursor)
        
        return pd.DataFrame(data)
    
    def transform_sales_data(self, df: pd.DataFrame) -> pd.DataFrame:
        """Трансформация данных о продажах"""
        # Очистка данных
        df = df.dropna(subset=['order_id', 'user_id', 'total_amount'])
        df = df[df['total_amount'] > 0]
        
        # Создание новых колонок
        df['order_date'] = pd.to_datetime(df['created_at']).dt.date
        df['year'] = pd.to_datetime(df['created_at']).dt.year
        df['month'] = pd.to_datetime(df['created_at']).dt.month
        df['quarter'] = pd.to_datetime(df['created_at']).dt.quarter
        
        # Категоризация сумм заказов
        df['amount_category'] = pd.cut(
            df['total_amount'],
            bins=[0, 50, 200, 500, float('inf')],
            labels=['Small', 'Medium', 'Large', 'Enterprise']
        )
        
        # Агрегация по дням
        daily_sales = df.groupby(['order_date', 'user_id']).agg({
            'total_amount': 'sum',
            'order_id': 'count'
        }).reset_index()
        
        daily_sales.rename(columns={'order_id': 'order_count'}, inplace=True)
        
        return daily_sales
    
    def load_to_warehouse(self, df: pd.DataFrame, table_name: str, if_exists: str = 'append'):
        """Загрузка данных в хранилище"""
        try:
            df.to_sql(
                table_name,
                self.target_engine,
                if_exists=if_exists,
                index=False,
                method='multi',
                chunksize=1000
            )
            
            self.logger.info(f"Loaded {len(df)} rows to {table_name}")
            self.update_last_run_timestamp(table_name)
            
        except Exception as e:
            self.logger.error(f"Error loading data to {table_name}: {e}")
            raise
    
    def run_sales_etl(self):
        """Запуск ETL для данных о продажах"""
        try:
            # Extract
            sales_df = self.extract_from_postgresql(self.source_configs['sales'])
            users_df = self.extract_from_postgresql(self.source_configs['users'])
            products_df = self.extract_from_mongodb(self.source_configs['products'])
            
            # Transform
            sales_transformed = self.transform_sales_data(sales_df)
            
            # Обогащение данными пользователей
            enriched_sales = sales_transformed.merge(
                users_df[['id', 'name', 'email', 'city', 'country']],
                left_on='user_id',
                right_on='id',
                how='left'
            )
            
            # Load
            self.load_to_warehouse(enriched_sales, 'fact_daily_sales')
            
            self.logger.info("Sales ETL completed successfully")
            
        except Exception as e:
            self.logger.error(f"Sales ETL failed: {e}")
            raise
    
    def get_last_run_timestamp(self, table_name: str) -> datetime:
        """Получение времени последнего запуска"""
        try:
            query = f"SELECT MAX(etl_timestamp) FROM etl_log WHERE table_name = '{table_name}'"
            result = pd.read_sql(query, self.target_engine)
            last_run = result.iloc[0, 0]
            return last_run if last_run else datetime(1900, 1, 1)
        except:
            return datetime(1900, 1, 1)
    
    def update_last_run_timestamp(self, table_name: str):
        """Обновление времени последнего запуска"""
        with self.target_engine.connect() as conn:
            conn.execute(f"""
                INSERT INTO etl_log (table_name, etl_timestamp) 
                VALUES ('{table_name}', '{datetime.now()}')
            """)
```

---

## ✅ Data Quality - Контроль качества данных

```python
# services/etl/data_quality.py
import pandas as pd
from typing import List, Dict, Any, Callable
import numpy as np

class DataQualityChecker:
    def __init__(self):
        self.rules = {}
        self.results = {}
    
    def add_rule(self, name: str, rule_func: Callable, severity: str = 'ERROR'):
        """Добавление правила качества данных"""
        self.rules[name] = {
            'function': rule_func,
            'severity': severity
        }
    
    def check_completeness(self, df: pd.DataFrame, required_columns: List[str]) -> Dict:
        """Проверка полноты данных"""
        results = {}
        for col in required_columns:
            if col in df.columns:
                null_count = df[col].isnull().sum()
                null_percentage = (null_count / len(df)) * 100
                results[col] = {
                    'null_count': null_count,
                    'null_percentage': null_percentage,
                    'status': 'PASS' if null_percentage < 5 else 'FAIL'
                }
            else:
                results[col] = {'status': 'MISSING_COLUMN'}
        
        return results
    
    def check_uniqueness(self, df: pd.DataFrame, unique_columns: List[str]) -> Dict:
        """Проверка уникальности"""
        results = {}
        for col in unique_columns:
            if col in df.columns:
                duplicate_count = df[col].duplicated().sum()
                results[col] = {
                    'duplicate_count': duplicate_count,
                    'status': 'PASS' if duplicate_count == 0 else 'FAIL'
                }
        
        return results
    
    def check_data_ranges(self, df: pd.DataFrame, range_rules: Dict) -> Dict:
        """Проверка диапазонов данных"""
        results = {}
        for col, (min_val, max_val) in range_rules.items():
            if col in df.columns:
                out_of_range = df[(df[col] < min_val) | (df[col] > max_val)]
                results[col] = {
                    'out_of_range_count': len(out_of_range),
                    'status': 'PASS' if len(out_of_range) == 0 else 'FAIL'
                }
        
        return results
    
    def check_referential_integrity(self, df: pd.DataFrame, parent_df: pd.DataFrame, 
                                  foreign_key: str, parent_key: str) -> Dict:
        """Проверка ссылочной целостности"""
        orphaned_records = df[~df[foreign_key].isin(parent_df[parent_key])]
        
        return {
            'orphaned_count': len(orphaned_records),
            'orphaned_percentage': (len(orphaned_records) / len(df)) * 100,
            'status': 'PASS' if len(orphaned_records) == 0 else 'FAIL'
        }
    
    def generate_quality_report(self, df: pd.DataFrame) -> Dict:
        """Генерация отчета о качестве данных"""
        report = {
            'timestamp': datetime.now().isoformat(),
            'row_count': len(df),
            'column_count': len(df.columns),
            'checks': {}
        }
        
        # Базовые проверки
        report['checks']['completeness'] = self.check_completeness(
            df, ['user_id', 'order_id', 'total_amount']
        )
        
        report['checks']['uniqueness'] = self.check_uniqueness(
            df, ['order_id']
        )
        
        report['checks']['ranges'] = self.check_data_ranges(
            df, {'total_amount': (0, 10000)}
        )
        
        # Общий статус
        all_checks = []
        for check_type, checks in report['checks'].items():
            for check_name, result in checks.items():
                if isinstance(result, dict) and 'status' in result:
                    all_checks.append(result['status'])
        
        report['overall_status'] = 'PASS' if all(s == 'PASS' for s in all_checks) else 'FAIL'
        
        return report

# Использование
quality_checker = DataQualityChecker()

# Пример проверки данных заказов
sales_data = pd.read_sql("SELECT * FROM orders", engine)
quality_report = quality_checker.generate_quality_report(sales_data)

print(f"Data quality status: {quality_report['overall_status']}")
```

---

## 📊 OLAP кубы и многомерный анализ

### 🧮 Построение OLAP куба

```python
# services/olap/cube_builder.py
import pandas as pd
import numpy as np
from typing import List, Dict, Any

class OLAPCube:
    def __init__(self, data: pd.DataFrame, dimensions: List[str], measures: List[str]):
        self.data = data
        self.dimensions = dimensions
        self.measures = measures
        self.cube = self._build_cube()
    
    def _build_cube(self) -> Dict:
        """Построение многомерного куба"""
        cube = {}
        
        # Создание всех возможных комбинаций измерений
        from itertools import combinations
        
        for r in range(len(self.dimensions) + 1):
            for dim_combo in combinations(self.dimensions, r):
                if not dim_combo:
                    # Общие итоги
                    key = 'TOTAL'
                    cube[key] = self.data[self.measures].sum().to_dict()
                else:
                    # Группировка по комбинации измерений
                    grouped = self.data.groupby(list(dim_combo))[self.measures].sum()
                    cube[dim_combo] = grouped.to_dict('index')
        
        return cube
    
    def slice(self, dimension: str, value: Any) -> Dict:
        """Срез по измерению"""
        filtered_data = self.data[self.data[dimension] == value]
        return OLAPCube(
            filtered_data, 
            [d for d in self.dimensions if d != dimension],
            self.measures
        )
    
    def dice(self, filters: Dict[str, Any]) -> Dict:
        """Кубик (фильтрация по нескольким измерениям)"""
        filtered_data = self.data.copy()
        for dim, value in filters.items():
            if isinstance(value, list):
                filtered_data = filtered_data[filtered_data[dim].isin(value)]
            else:
                filtered_data = filtered_data[filtered_data[dim] == value]
        
        return OLAPCube(filtered_data, self.dimensions, self.measures)
    
    def drill_down(self, dimension: str, parent_value: Any, child_dimension: str) -> Dict:
        """Детализация (drill-down)"""
        filtered_data = self.data[self.data[dimension] == parent_value]
        return filtered_data.groupby(child_dimension)[self.measures].sum().to_dict()
    
    def roll_up(self, from_dimension: str, to_dimension: str) -> Dict:
        """Агрегация (roll-up)"""
        # Пример: от дня к месяцу, от продукта к категории
        return self.data.groupby(to_dimension)[self.measures].sum().to_dict()
    
    def pivot_analysis(self, row_dims: List[str], col_dims: List[str], 
                      measure: str, aggfunc: str = 'sum') -> pd.DataFrame:
        """Сводная таблица"""
        return pd.pivot_table(
            self.data,
            values=measure,
            index=row_dims,
            columns=col_dims,
            aggfunc=aggfunc,
            fill_value=0
        )

# Пример использования для анализа продаж
sales_data = pd.read_sql("""
    SELECT 
        o.created_at::date as order_date,
        EXTRACT(year FROM o.created_at) as year,
        EXTRACT(month FROM o.created_at) as month,
        u.city,
        u.country,
        p.category as product_category,
        oi.quantity,
        oi.price,
        oi.quantity * oi.price as revenue
    FROM orders o
    JOIN users u ON o.user_id = u.id
    JOIN order_items oi ON o.id = oi.order_id
    JOIN products p ON oi.product_id = p.id
    WHERE o.created_at >= '2024-01-01'
""", engine)

# Создание OLAP куба
cube = OLAPCube(
    sales_data,
    dimensions=['year', 'month', 'city', 'country', 'product_category'],
    measures=['quantity', 'revenue']
)

# Анализ продаж по городам
city_analysis = cube.pivot_analysis(['city'], ['month'], 'revenue')
print("Revenue by City and Month:")
print(city_analysis)

# Детализация по категориям продуктов в Москве
moscow_products = cube.dice({'city': 'Moscow'}).pivot_analysis(
    ['product_category'], ['month'], 'revenue'
)
print("\nMoscow Revenue by Product Category:")
print(moscow_products)
```

---

## 📈 ClickHouse для Аналитики

### ⚡ Columnstore база данных

```python
# services/analytics/clickhouse_service.py
from clickhouse_driver import Client
import pandas as pd
from typing import Dict, List, Any
import datetime

class ClickHouseAnalytics:
    def __init__(self, host: str = 'localhost', port: int = 9000):
        self.client = Client(host=host, port=port)
    
    def create_analytics_tables(self):
        """Создание таблиц для аналитики"""
        
        # Таблица событий пользователей
        self.client.execute("""
            CREATE TABLE IF NOT EXISTS user_events (
                event_id UUID,
                user_id UInt64,
                event_type LowCardinality(String),
                event_timestamp DateTime,
                page_url String,
                user_agent String,
                ip_address IPv4,
                session_id String,
                properties Map(String, String),
                created_date Date MATERIALIZED toDate(event_timestamp)
            ) ENGINE = MergeTree()
            PARTITION BY toYYYYMM(event_timestamp)
            ORDER BY (user_id, event_timestamp)
            TTL event_timestamp + INTERVAL 2 YEAR
        """)
        
        # Материализованное представление для дневной аналитики
        self.client.execute("""
            CREATE MATERIALIZED VIEW IF NOT EXISTS daily_user_stats
            ENGINE = SummingMergeTree()
            PARTITION BY toYYYYMM(event_date)
            ORDER BY (event_date, user_id)
            AS SELECT
                toDate(event_timestamp) as event_date,
                user_id,
                count() as event_count,
                uniq(session_id) as session_count,
                countIf(event_type = 'page_view') as page_views,
                countIf(event_type = 'purchase') as purchases
            FROM user_events
            GROUP BY event_date, user_id
        """)
        
        # Таблица продаж с денормализованными данными
        self.client.execute("""
            CREATE TABLE IF NOT EXISTS sales_fact (
                order_id UInt64,
                user_id UInt64,
                product_id UInt64,
                order_date Date,
                order_timestamp DateTime,
                quantity UInt32,
                unit_price Decimal(10,2),
                total_amount Decimal(10,2),
                discount_amount Decimal(10,2),
                user_city LowCardinality(String),
                user_country LowCardinality(String),
                product_name String,
                product_category LowCardinality(String),
                product_brand LowCardinality(String),
                year UInt16 MATERIALIZED toYear(order_date),
                month UInt8 MATERIALIZED toMonth(order_date),
                day_of_week UInt8 MATERIALIZED toDayOfWeek(order_date)
            ) ENGINE = MergeTree()
            PARTITION BY toYYYYMM(order_date)
            ORDER BY (order_date, user_id, product_id)
        """)
    
    def insert_user_event(self, event_data: Dict[str, Any]):
        """Вставка события пользователя"""
        self.client.execute(
            """
            INSERT INTO user_events 
            (event_id, user_id, event_type, event_timestamp, page_url, 
             user_agent, ip_address, session_id, properties)
            VALUES
            """,
            [event_data]
        )
    
    def get_user_behavior_funnel(self, start_date: str, end_date: str) -> pd.DataFrame:
        """Анализ воронки пользователей"""
        query = f"""
        SELECT 
            formatDateTime(event_timestamp, '%Y-%m-%d') as date,
            countIf(event_type = 'page_view') as page_views,
            uniq(user_id) as unique_visitors,
            countIf(event_type = 'add_to_cart') as cart_additions,
            countIf(event_type = 'purchase') as purchases,
            countIf(event_type = 'add_to_cart') / uniq(user_id) * 100 as cart_conversion_rate,
            countIf(event_type = 'purchase') / countIf(event_type = 'add_to_cart') * 100 as purchase_conversion_rate
        FROM user_events
        WHERE event_timestamp >= '{start_date}' AND event_timestamp <= '{end_date}'
        GROUP BY date
        ORDER BY date
        """
        
        result = self.client.execute(query)
        columns = ['date', 'page_views', 'unique_visitors', 'cart_additions', 
                  'purchases', 'cart_conversion_rate', 'purchase_conversion_rate']
        
        return pd.DataFrame(result, columns=columns)
    
    def get_cohort_analysis(self, months_back: int = 12) -> pd.DataFrame:
        """Когортный анализ пользователей"""
        query = f"""
        WITH first_orders AS (
            SELECT 
                user_id,
                min(order_date) as first_order_date,
                toStartOfMonth(min(order_date)) as cohort_month
            FROM sales_fact
            WHERE order_date >= today() - INTERVAL {months_back} MONTH
            GROUP BY user_id
        ),
        cohort_data AS (
            SELECT 
                f.cohort_month,
                dateDiff('month', f.cohort_month, toStartOfMonth(s.order_date)) as period_number,
                count(DISTINCT s.user_id) as users
            FROM first_orders f
            JOIN sales_fact s ON f.user_id = s.user_id
            WHERE s.order_date >= f.first_order_date
            GROUP BY f.cohort_month, period_number
        ),
        cohort_sizes AS (
            SELECT 
                cohort_month,
                count(DISTINCT user_id) as cohort_size
            FROM first_orders
            GROUP BY cohort_month
        )
        SELECT 
            c.cohort_month,
            c.period_number,
            c.users,
            cs.cohort_size,
            c.users / cs.cohort_size * 100 as retention_rate
        FROM cohort_data c
        JOIN cohort_sizes cs ON c.cohort_month = cs.cohort_month
        ORDER BY c.cohort_month, c.period_number
        """
        
        result = self.client.execute(query)
        columns = ['cohort_month', 'period_number', 'users', 'cohort_size', 'retention_rate']
        
        return pd.DataFrame(result, columns=columns)
    
    def get_product_performance(self, days_back: int = 30) -> pd.DataFrame:
        """Анализ производительности продуктов"""
        query = f"""
        SELECT 
            product_category,
            product_name,
            count() as order_count,
            sum(quantity) as total_quantity,
            sum(total_amount) as total_revenue,
            avg(total_amount) as avg_order_value,
            uniq(user_id) as unique_customers,
            sum(total_amount) / uniq(user_id) as revenue_per_customer
        FROM sales_fact
        WHERE order_date >= today() - {days_back}
        GROUP BY product_category, product_name
        HAVING total_revenue > 1000
        ORDER BY total_revenue DESC
        LIMIT 50
        """
        
        result = self.client.execute(query)
        columns = ['product_category', 'product_name', 'order_count', 'total_quantity',
                  'total_revenue', 'avg_order_value', 'unique_customers', 'revenue_per_customer']
        
        return pd.DataFrame(result, columns=columns)

# Использование
analytics = ClickHouseAnalytics()
analytics.create_analytics_tables()

# Анализ воронки за последние 30 дней
funnel_data = analytics.get_user_behavior_funnel(
    (datetime.datetime.now() - datetime.timedelta(days=30)).strftime('%Y-%m-%d %H:%M:%S'),
    datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
)
print("Funnel Analysis:")
print(funnel_data)

# Когортный анализ
cohort_data = analytics.get_cohort_analysis(months_back=6)
print("\nCohort Analysis:")
print(cohort_data.pivot(index='cohort_month', columns='period_number', values='retention_rate'))
```

---

## 🎯 Практические задания

### 🔰 Начальный уровень
1. Создайте простую схему "звезда" для анализа продаж
2. Реализуйте базовый ETL процесс из одного источника
3. Создайте отчет по продажам с группировкой по месяцам

### 🚀 Средний уровень
1. Постройте комплексную схему с медленно изменяющимися измерениями (SCD Type 2)
2. Реализуйте ETL с обработкой ошибок и логированием
3. Создайте OLAP куб с возможностью drill-down и roll-up операций

### 💎 Продвинутый уровень
1. Создайте реал-тайм ETL с использованием потоковой обработки
2. Реализуйте систему мониторинга качества данных
3. Постройте систему аналитики с машинным обучением для прогнозирования

---

## 🌟 Лучшие практики

### ✅ Рекомендации

#### 🏗️ Проектирование
- **Начинайте с бизнес-требований** - схема должна отвечать на бизнес-вопросы
- **Используйте правильную гранулярность** - найдите баланс между детализацией и производительностью
- **Планируйте для роста** - учитывайте будущие объемы данных
- **Документируйте все** - метаданные критически важны

#### 🔄 ETL процессы
- **Инкрементальная загрузка** - загружайте только новые/измененные данные
- **Идемпотентность** - повторный запуск должен давать тот же результат
- **Обработка ошибок** - логирование и уведомления о проблемах
- **Мониторинг производительности** - отслеживание времени выполнения

#### 📊 Производительность
- **Правильные индексы** - на часто используемые колонки
- **Партиционирование** - по времени или другим ключам
- **Компрессия данных** - экономия места и улучшение I/O
- **Материализованные представления** - для частых запросов

### ❌ Чего избегать

- **Overengineering** - не создавайте слишком сложную архитектуру с самого начала
- **Пренебрежение качеством данных** - плохие данные = плохие решения
- **Игнорирование безопасности** - защищайте конфиденциальные данные
- **Отсутствие версионирования** - отслеживайте изменения в схеме

---

## 🔗 Связанные темы

- 🐘 **[[postgresql|PostgreSQL]]** - для источников OLTP данных
- 🍃 **[[mongodb|MongoDB]]** - для неструктурированных источников
- 🔴 **[[redis|Redis]]** - для кеширования аналитики
- 🏗️ **[[database-design|Проектирование БД]]** - принципы моделирования
- ⚡ **[[sql-optimization|Оптимизация БД]]** - производительность запросов

**[[README|🏠 Вернуться к обзору баз данных]]** 