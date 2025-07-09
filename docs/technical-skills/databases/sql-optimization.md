# Оптимизация баз данных ⚡

## 🏠 Навигация
**[[README|🏠 Базы данных]]** → **Оптимизация БД**

**Связанные темы:**
- 🐘 [[postgresql|PostgreSQL]]
- 🍃 [[mongodb|MongoDB]]
- 🔴 [[redis|Redis]]
- 🏗️ [[database-design|Проектирование БД]]
- 🏗️ [[database-architecture-patterns|Архитектурные паттерны]]

---

## 🎯 Принципы оптимизации баз данных

Оптимизация базы данных — это процесс настройки структуры, запросов и конфигурации для достижения максимальной производительности при минимальном использовании ресурсов.

### ✨ Ключевые метрики производительности

1. **⚡ Latency** - время отклика на запросы
2. **📊 Throughput** - количество операций в секунду  
3. **💾 Resource Usage** - использование CPU, памяти, дискового I/O
4. **🔄 Concurrency** - количество одновременных операций
5. **🎯 Query Performance** - эффективность отдельных запросов

---

## 📊 Анализ производительности запросов

### 🔍 EXPLAIN ANALYZE в PostgreSQL

```sql
-- Анализ плана выполнения запроса
EXPLAIN (ANALYZE, BUFFERS, VERBOSE) 
SELECT u.name, COUNT(o.id) as order_count, SUM(o.total_amount) as total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at >= '2024-01-01'
GROUP BY u.id, u.name
HAVING COUNT(o.id) > 5
ORDER BY total_spent DESC
LIMIT 10;

-- Результат покажет:
-- - Actual Time: реальное время выполнения каждого шага
-- - Rows: количество обработанных строк
-- - Buffers: статистика использования кэша
-- - Cost: оценочная стоимость операций
```

### 📈 Интерпретация планов выполнения

```python
# services/optimization/query_analyzer.py
import re
from typing import Dict, List, Tuple
from dataclasses import dataclass

@dataclass
class QueryPlanNode:
    """Узел плана выполнения запроса"""
    operation: str
    cost: Tuple[float, float]  # (startup_cost, total_cost)
    rows: int
    actual_time: Tuple[float, float]  # (startup_time, total_time)
    actual_rows: int
    buffers_hit: int = 0
    buffers_read: int = 0

class QueryPlanAnalyzer:
    """Анализатор планов выполнения запросов"""
    
    def __init__(self):
        self.performance_thresholds = {
            'slow_query_time': 1000,  # ms
            'high_cost': 10000,
            'many_rows': 100000,
            'buffer_read_ratio': 0.1  # 10% reads from disk
        }
    
    def parse_explain_output(self, explain_text: str) -> List[QueryPlanNode]:
        """Парсинг вывода EXPLAIN ANALYZE"""
        nodes = []
        lines = explain_text.strip().split('\n')
        
        for line in lines:
            if '->' in line or 'Seq Scan' in line or 'Index Scan' in line:
                node = self._parse_line(line)
                if node:
                    nodes.append(node)
        
        return nodes
    
    def _parse_line(self, line: str) -> QueryPlanNode:
        """Парсинг отдельной строки плана"""
        # Извлечение операции
        operation_match = re.search(r'(Seq Scan|Index Scan|Nested Loop|Hash Join|Sort|Aggregate)', line)
        if not operation_match:
            return None
        
        operation = operation_match.group(1)
        
        # Извлечение стоимости: cost=0.00..483.00
        cost_match = re.search(r'cost=([0-9.]+)\.\.([0-9.]+)', line)
        cost = (float(cost_match.group(1)), float(cost_match.group(2))) if cost_match else (0, 0)
        
        # Извлечение строк: rows=1000
        rows_match = re.search(r'rows=([0-9]+)', line)
        rows = int(rows_match.group(1)) if rows_match else 0
        
        # Извлечение времени: actual time=0.123..4.567
        time_match = re.search(r'actual time=([0-9.]+)\.\.([0-9.]+)', line)
        actual_time = (float(time_match.group(1)), float(time_match.group(2))) if time_match else (0, 0)
        
        # Извлечение фактических строк: actual rows=1234
        actual_rows_match = re.search(r'actual rows=([0-9]+)', line)
        actual_rows = int(actual_rows_match.group(1)) if actual_rows_match else 0
        
        return QueryPlanNode(
            operation=operation,
            cost=cost,
            rows=rows,
            actual_time=actual_time,
            actual_rows=actual_rows
        )
    
    def analyze_performance_issues(self, nodes: List[QueryPlanNode]) -> List[str]:
        """Анализ проблем производительности"""
        issues = []
        
        for node in nodes:
            # Медленные операции
            if node.actual_time[1] > self.performance_thresholds['slow_query_time']:
                issues.append(f"Медленная операция {node.operation}: {node.actual_time[1]:.2f}ms")
            
            # Высокая стоимость
            if node.cost[1] > self.performance_thresholds['high_cost']:
                issues.append(f"Высокая стоимость {node.operation}: {node.cost[1]:.0f}")
            
            # Полное сканирование больших таблиц
            if node.operation == 'Seq Scan' and node.actual_rows > self.performance_thresholds['many_rows']:
                issues.append(f"Полное сканирование большой таблицы: {node.actual_rows} строк")
            
            # Неточная оценка оптимизатора
            if node.rows > 0 and abs(node.actual_rows - node.rows) / node.rows > 0.5:
                issues.append(f"Неточная оценка строк в {node.operation}: ожидалось {node.rows}, получено {node.actual_rows}")
        
        return issues
    
    def suggest_optimizations(self, issues: List[str]) -> List[str]:
        """Предложения по оптимизации"""
        suggestions = []
        
        for issue in issues:
            if "Полное сканирование" in issue:
                suggestions.append("Добавьте индекс на колонки в WHERE clause")
            elif "Медленная операция" in issue:
                suggestions.append("Рассмотрите денормализацию или кэширование")
            elif "Высокая стоимость" in issue:
                suggestions.append("Упростите запрос или разбейте на части")
            elif "Неточная оценка" in issue:
                suggestions.append("Обновите статистику: ANALYZE table_name")
        
        return suggestions

# Использование
analyzer = QueryPlanAnalyzer()
explain_text = """
Limit  (cost=15.43..15.45 rows=10 width=64) (actual time=1.234..1.235 rows=10 loops=1)
  ->  Sort  (cost=15.43..15.48 rows=20 width=64) (actual time=1.234..1.234 rows=10 loops=1)
        Sort Key: total_spent DESC
        Sort Method: quicksort  Memory: 25kB
        ->  HashAggregate  (cost=14.95..15.35 rows=20 width=64) (actual time=1.123..1.150 rows=15 loops=1)
"""

nodes = analyzer.parse_explain_output(explain_text)
issues = analyzer.analyze_performance_issues(nodes)
suggestions = analyzer.suggest_optimizations(issues)

for suggestion in suggestions:
    print(f"💡 {suggestion}")
```

---

## 🏗️ Индексы и их оптимизация

### 📚 Типы индексов

#### 🌳 B-Tree индексы (по умолчанию)

```sql
-- Простой индекс
CREATE INDEX idx_users_email ON users(email);

-- Составной индекс (порядок важен!)
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);

-- Частичный индекс (для подмножества данных)
CREATE INDEX idx_orders_pending ON orders(created_at) 
WHERE status = 'pending';

-- Индекс с выражением
CREATE INDEX idx_users_lower_email ON users(LOWER(email));

-- Покрывающий индекс (INCLUDE колонки)
CREATE INDEX idx_orders_covering ON orders(user_id, created_at) 
INCLUDE (total_amount, status);
```

#### 🔗 Hash индексы

```sql
-- Только для точного соответствия (=)
CREATE INDEX idx_users_hash_id ON users USING hash(id);

-- Эффективны для больших таблиц с уникальными значениями
-- НЕ поддерживают: <, >, BETWEEN, ORDER BY
```

#### 🔍 GIN индексы (для массивов, JSONB, полнотекстовый поиск)

```sql
-- Для JSONB данных
CREATE INDEX idx_products_attributes ON products USING gin(attributes);

-- Для массивов
CREATE INDEX idx_users_tags ON users USING gin(tags);

-- Для полнотекстового поиска
CREATE INDEX idx_articles_fts ON articles USING gin(to_tsvector('russian', title || ' ' || content));
```

### ⚡ Стратегии индексации

```python
# services/optimization/index_advisor.py
from typing import List, Dict, Tuple
import re

class IndexAdvisor:
    """Советчик по индексам"""
    
    def __init__(self):
        self.query_patterns = []
        self.table_stats = {}
    
    def analyze_query_log(self, queries: List[str]) -> Dict[str, List[str]]:
        """Анализ лога запросов для рекомендаций индексов"""
        recommendations = {}
        
        for query in queries:
            # Извлечение таблиц и условий
            tables = self._extract_tables(query)
            where_conditions = self._extract_where_conditions(query)
            join_conditions = self._extract_join_conditions(query)
            order_by_columns = self._extract_order_by(query)
            
            for table in tables:
                if table not in recommendations:
                    recommendations[table] = []
                
                # Рекомендации для WHERE условий
                for condition in where_conditions.get(table, []):
                    index_suggestion = f"CREATE INDEX idx_{table}_{condition} ON {table}({condition});"
                    if index_suggestion not in recommendations[table]:
                        recommendations[table].append(index_suggestion)
                
                # Рекомендации для JOIN условий
                for join_col in join_conditions.get(table, []):
                    index_suggestion = f"CREATE INDEX idx_{table}_{join_col} ON {table}({join_col});"
                    if index_suggestion not in recommendations[table]:
                        recommendations[table].append(index_suggestion)
                
                # Рекомендации для ORDER BY
                if table in order_by_columns:
                    cols = ', '.join(order_by_columns[table])
                    index_suggestion = f"CREATE INDEX idx_{table}_sort ON {table}({cols});"
                    if index_suggestion not in recommendations[table]:
                        recommendations[table].append(index_suggestion)
        
        return recommendations
    
    def _extract_tables(self, query: str) -> List[str]:
        """Извлечение таблиц из запроса"""
        # Упрощенная реализация
        from_match = re.search(r'FROM\s+(\w+)', query, re.IGNORECASE)
        join_matches = re.findall(r'JOIN\s+(\w+)', query, re.IGNORECASE)
        
        tables = []
        if from_match:
            tables.append(from_match.group(1))
        tables.extend(join_matches)
        
        return tables
    
    def _extract_where_conditions(self, query: str) -> Dict[str, List[str]]:
        """Извлечение WHERE условий"""
        where_conditions = {}
        
        # Поиск WHERE секции
        where_match = re.search(r'WHERE\s+(.+?)(?:GROUP BY|ORDER BY|LIMIT|$)', query, re.IGNORECASE | re.DOTALL)
        if not where_match:
            return where_conditions
        
        where_clause = where_match.group(1)
        
        # Поиск условий вида table.column = value
        condition_matches = re.findall(r'(\w+)\.(\w+)\s*[=<>]', where_clause)
        
        for table, column in condition_matches:
            if table not in where_conditions:
                where_conditions[table] = []
            if column not in where_conditions[table]:
                where_conditions[table].append(column)
        
        return where_conditions
    
    def _extract_join_conditions(self, query: str) -> Dict[str, List[str]]:
        """Извлечение JOIN условий"""
        join_conditions = {}
        
        # Поиск условий JOIN
        join_matches = re.findall(r'JOIN\s+\w+.*?ON\s+(\w+)\.(\w+)\s*=\s*(\w+)\.(\w+)', query, re.IGNORECASE)
        
        for match in join_matches:
            table1, col1, table2, col2 = match
            
            if table1 not in join_conditions:
                join_conditions[table1] = []
            if table2 not in join_conditions:
                join_conditions[table2] = []
            
            if col1 not in join_conditions[table1]:
                join_conditions[table1].append(col1)
            if col2 not in join_conditions[table2]:
                join_conditions[table2].append(col2)
        
        return join_conditions
    
    def _extract_order_by(self, query: str) -> Dict[str, List[str]]:
        """Извлечение ORDER BY колонок"""
        order_by_columns = {}
        
        order_match = re.search(r'ORDER BY\s+(.+?)(?:LIMIT|$)', query, re.IGNORECASE)
        if not order_match:
            return order_by_columns
        
        order_clause = order_match.group(1)
        
        # Поиск колонок для сортировки
        column_matches = re.findall(r'(\w+)\.(\w+)', order_clause)
        
        for table, column in column_matches:
            if table not in order_by_columns:
                order_by_columns[table] = []
            if column not in order_by_columns[table]:
                order_by_columns[table].append(column)
        
        return order_by_columns
    
    def calculate_index_selectivity(self, table: str, column: str) -> float:
        """Расчет селективности индекса"""
        # Имитация расчета селективности
        # В реальности нужен доступ к статистике БД
        return 0.1  # 10% селективность
    
    def recommend_composite_indexes(self, table_conditions: Dict[str, List[str]]) -> List[str]:
        """Рекомендации составных индексов"""
        recommendations = []
        
        for table, conditions in table_conditions.items():
            if len(conditions) > 1:
                # Сортируем колонки по селективности (более селективные первыми)
                sorted_conditions = sorted(conditions, 
                    key=lambda col: self.calculate_index_selectivity(table, col))
                
                composite_index = f"CREATE INDEX idx_{table}_composite ON {table}({', '.join(sorted_conditions)});"
                recommendations.append(composite_index)
        
        return recommendations

# Пример использования
advisor = IndexAdvisor()

sample_queries = [
    "SELECT * FROM orders WHERE user_id = 123 AND status = 'pending' ORDER BY created_at DESC",
    "SELECT u.name, COUNT(*) FROM users u JOIN orders o ON u.id = o.user_id WHERE u.created_at > '2024-01-01' GROUP BY u.name",
    "SELECT * FROM products WHERE category_id = 5 AND price BETWEEN 100 AND 500 ORDER BY price"
]

recommendations = advisor.analyze_query_log(sample_queries)

for table, indexes in recommendations.items():
    print(f"\n📋 Рекомендации для таблицы {table}:")
    for index in indexes:
        print(f"  {index}")
```

---

## 🛠️ Оптимизация запросов

### 🔄 Переписывание запросов

#### ❌ Плохие практики и ✅ их исправления

```sql
-- ❌ ПЛОХО: SELECT *
SELECT * FROM orders WHERE created_at > '2024-01-01';

-- ✅ ХОРОШО: Указывайте нужные колонки
SELECT id, user_id, total_amount, status 
FROM orders 
WHERE created_at > '2024-01-01';

-- ❌ ПЛОХО: Функции в WHERE (не использует индексы)
SELECT * FROM users WHERE YEAR(created_at) = 2024;

-- ✅ ХОРОШО: Диапазон дат
SELECT * FROM users 
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';

-- ❌ ПЛОХО: OR условия
SELECT * FROM products WHERE category_id = 1 OR category_id = 2 OR category_id = 3;

-- ✅ ХОРОШО: IN список
SELECT * FROM products WHERE category_id IN (1, 2, 3);

-- ❌ ПЛОХО: Коррелированный подзапрос
SELECT * FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o 
    WHERE o.user_id = u.id AND o.created_at > '2024-01-01'
);

-- ✅ ХОРОШО: JOIN
SELECT DISTINCT u.*
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE o.created_at > '2024-01-01';

-- ❌ ПЛОХО: COUNT(*) для проверки существования
SELECT COUNT(*) FROM orders WHERE user_id = 123;

-- ✅ ХОРОШО: EXISTS для булевой проверки
SELECT EXISTS(SELECT 1 FROM orders WHERE user_id = 123);
```

### 🔧 Оптимизация сложных запросов

```sql
-- Оптимизация запроса с агрегацией и сортировкой
-- ❌ МЕДЛЕННЫЙ вариант
SELECT 
    u.name,
    COUNT(o.id) as order_count,
    SUM(o.total_amount) as total_spent,
    AVG(o.total_amount) as avg_order
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at >= '2024-01-01'
GROUP BY u.id, u.name
HAVING COUNT(o.id) > 5
ORDER BY total_spent DESC
LIMIT 100;

-- ✅ ОПТИМИЗИРОВАННЫЙ вариант
-- 1. Сначала отфильтруем пользователей
WITH active_users AS (
    SELECT id, name 
    FROM users 
    WHERE created_at >= '2024-01-01'
),
-- 2. Затем агрегируем заказы
user_stats AS (
    SELECT 
        o.user_id,
        COUNT(*) as order_count,
        SUM(o.total_amount) as total_spent,
        AVG(o.total_amount) as avg_order
    FROM orders o
    WHERE o.user_id IN (SELECT id FROM active_users)
    GROUP BY o.user_id
    HAVING COUNT(*) > 5
)
-- 3. Соединяем результаты
SELECT 
    au.name,
    us.order_count,
    us.total_spent,
    us.avg_order
FROM active_users au
INNER JOIN user_stats us ON au.id = us.user_id
ORDER BY us.total_spent DESC
LIMIT 100;

-- Нужные индексы:
-- CREATE INDEX idx_users_created_at ON users(created_at);
-- CREATE INDEX idx_orders_user_id ON orders(user_id);
-- CREATE INDEX idx_orders_user_amount ON orders(user_id, total_amount);
```

---

## 💾 Кэширование и буферизация

### 🏪 Уровни кэширования

```python
# services/optimization/caching_strategy.py
import redis
import json
from typing import Any, Optional, Dict
from datetime import datetime, timedelta

class MultiLevelCache:
    """Многоуровневое кэширование"""
    
    def __init__(self):
        # Уровень 1: In-memory кэш (самый быстрый)
        self.memory_cache: Dict[str, Dict] = {}
        self.memory_cache_ttl: Dict[str, datetime] = {}
        
        # Уровень 2: Redis (быстрый, но сетевой)
        self.redis_client = redis.Redis(host='localhost', port=6379, db=0)
        
        # Уровень 3: Database query cache
        self.query_cache_enabled = True
    
    def get(self, key: str) -> Optional[Any]:
        """Получение из кэша с проверкой всех уровней"""
        
        # Уровень 1: Memory cache
        if key in self.memory_cache:
            if key in self.memory_cache_ttl and datetime.now() < self.memory_cache_ttl[key]:
                return self.memory_cache[key]['data']
            else:
                # Expired
                del self.memory_cache[key]
                if key in self.memory_cache_ttl:
                    del self.memory_cache_ttl[key]
        
        # Уровень 2: Redis cache
        try:
            redis_data = self.redis_client.get(key)
            if redis_data:
                data = json.loads(redis_data)
                # Populate memory cache
                self.memory_cache[key] = {'data': data}
                self.memory_cache_ttl[key] = datetime.now() + timedelta(minutes=5)
                return data
        except:
            pass
        
        return None
    
    def set(self, key: str, value: Any, ttl_minutes: int = 60):
        """Сохранение в кэш на всех уровнях"""
        
        # Уровень 1: Memory cache (TTL 5 минут)
        self.memory_cache[key] = {'data': value}
        self.memory_cache_ttl[key] = datetime.now() + timedelta(minutes=min(5, ttl_minutes))
        
        # Уровень 2: Redis cache
        try:
            self.redis_client.setex(key, timedelta(minutes=ttl_minutes), json.dumps(value))
        except:
            pass
    
    def invalidate(self, pattern: str):
        """Инвалидация кэша по паттерну"""
        
        # Memory cache
        keys_to_delete = [k for k in self.memory_cache.keys() if pattern in k]
        for key in keys_to_delete:
            del self.memory_cache[key]
            if key in self.memory_cache_ttl:
                del self.memory_cache_ttl[key]
        
        # Redis cache
        try:
            keys = self.redis_client.keys(f"*{pattern}*")
            if keys:
                self.redis_client.delete(*keys)
        except:
            pass

class QueryResultCache:
    """Кэширование результатов запросов"""
    
    def __init__(self, cache: MultiLevelCache):
        self.cache = cache
    
    def cached_query(self, query_hash: str, query_func, ttl_minutes: int = 30):
        """Декоратор для кэширования результатов запросов"""
        
        # Проверяем кэш
        cached_result = self.cache.get(f"query:{query_hash}")
        if cached_result is not None:
            return cached_result
        
        # Выполняем запрос
        result = query_func()
        
        # Сохраняем в кэш
        self.cache.set(f"query:{query_hash}", result, ttl_minutes)
        
        return result
    
    def invalidate_queries_for_table(self, table_name: str):
        """Инвалидация кэша при изменении таблицы"""
        self.cache.invalidate(f"query:table:{table_name}")

# Пример использования
cache = MultiLevelCache()
query_cache = QueryResultCache(cache)

def get_user_orders(user_id: int):
    """Получение заказов пользователя с кэшированием"""
    
    query_hash = f"user_orders:{user_id}"
    
    def fetch_from_db():
        # Здесь был бы реальный запрос к БД
        return [
            {'id': 1, 'total': 100.0, 'status': 'completed'},
            {'id': 2, 'total': 250.0, 'status': 'pending'}
        ]
    
    return query_cache.cached_query(query_hash, fetch_from_db, ttl_minutes=15)

# Использование
orders = get_user_orders(123)  # Первый раз - запрос к БД
orders = get_user_orders(123)  # Второй раз - из кэша
```

### 🔄 Материализованные представления

```sql
-- Создание материализованного представления для часто используемых агрегаций
CREATE MATERIALIZED VIEW user_order_stats AS
SELECT 
    u.id as user_id,
    u.name,
    u.email,
    COUNT(o.id) as total_orders,
    SUM(o.total_amount) as total_spent,
    AVG(o.total_amount) as avg_order_value,
    MAX(o.created_at) as last_order_date,
    MIN(o.created_at) as first_order_date
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name, u.email;

-- Создание индекса на материализованном представлении
CREATE INDEX idx_user_stats_total_spent ON user_order_stats(total_spent DESC);
CREATE INDEX idx_user_stats_user_id ON user_order_stats(user_id);

-- Автоматическое обновление через триггеры
CREATE OR REPLACE FUNCTION refresh_user_stats()
RETURNS TRIGGER AS $$
BEGIN
    -- Обновляем только затронутые записи
    REFRESH MATERIALIZED VIEW CONCURRENTLY user_order_stats;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Триггеры на изменение данных
CREATE TRIGGER update_user_stats_on_order
    AFTER INSERT OR UPDATE OR DELETE ON orders
    FOR EACH STATEMENT
    EXECUTE FUNCTION refresh_user_stats();

-- Использование материализованного представления
SELECT * FROM user_order_stats 
WHERE total_spent > 1000 
ORDER BY total_spent DESC 
LIMIT 10;
```

---

## 🎯 Практические задания

### 🔰 Начальный уровень
1. Проанализируйте план выполнения медленного запроса и предложите оптимизации
2. Создайте индексы для ускорения конкретных запросов
3. Переепишите запрос с подзапросами, используя JOIN

### 🚀 Средний уровень
1. Реализуйте систему кэширования результатов запросов с автоматической инвалидацией
2. Создайте материализованные представления для аналитических запросов
3. Оптимизируйте запросы с оконными функциями

### 💎 Продвинутый уровень
1. Создайте систему автоматических рекомендаций индексов на основе логов запросов
2. Реализуйте партиционирование больших таблиц с автоматическим управлением
3. Создайте систему мониторинга производительности с алертами

---

## 🌟 Лучшие практики

### ✅ Рекомендации

#### 🔍 Анализ запросов
- **Регулярно анализируйте планы** - используйте EXPLAIN ANALYZE
- **Мониторьте медленные запросы** - настройте логирование
- **Обновляйте статистику** - регулярно выполняйте ANALYZE
- **Тестируйте на реальных данных** - dev окружение должно быть похоже на prod

#### 🏗️ Индексирование
- **Индексы для WHERE условий** - особенно для часто используемых фильтров
- **Составные индексы** - учитывайте порядок колонок
- **Покрывающие индексы** - для ускорения SELECT
- **Удаляйте неиспользуемые индексы** - они замедляют INSERT/UPDATE

#### 💾 Кэширование
- **Кэшируйте тяжелые запросы** - особенно аналитические
- **Стратегия инвалидации** - планируйте обновление кэша
- **Мониторьте hit rate** - отслеживайте эффективность кэша
- **Не кэшируйте быстро меняющиеся данные** - баланс между свежестью и производительностью

### ❌ Чего избегать

- **Преждевременная оптимизация** - сначала измерьте, потом оптимизируйте
- **Избыточные индексы** - каждый индекс имеет свою стоимость
- **Игнорирование планов выполнения** - не оптимизируйте вслепую
- **Кэширование всего подряд** - кэш тоже требует ресурсов

---

## 🔗 Связанные темы

- 🐘 **[[postgresql|PostgreSQL]]** - специфические оптимизации PostgreSQL
- 🍃 **[[mongodb|MongoDB]]** - оптимизация NoSQL запросов
- 🔴 **[[redis|Redis]]** - кэширование и производительность
- 🏗️ **[[database-design|Проектирование БД]]** - правильная структура как основа производительности
- 🏗️ **[[database-architecture-patterns|Архитектурные паттерны]]** - паттерны для масштабируемости

**[[README|🏠 Вернуться к обзору баз данных]]** 