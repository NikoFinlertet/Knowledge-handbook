# Архитектура для Senior Developer

## Содержание
1. [Системный дизайн](#системный-дизайн)
2. [Архитектурные паттерны](#архитектурные-паттерны)
3. [Масштабирование](#масштабирование)
4. [Принятие архитектурных решений](#принятие-архитектурных-решений)
5. [Оценка сложности](#оценка-сложности)
6. [Техническое планирование](#техническое-планирование)
7. [Архитектурные trade-offs](#архитектурные-trade-offs)
8. [Практические примеры](#практические-примеры)

---

## Системный дизайн

### Принципы системного дизайна

#### CAP Theorem
```
Consistency + Availability + Partition Tolerance
Можно выбрать только 2 из 3:

CP (Consistency + Partition Tolerance):
- ACID базы данных
- Блокировки при разделении сети
- Пример: PostgreSQL, MongoDB

AP (Availability + Partition Tolerance):
- Eventual consistency
- Система работает при разделении
- Пример: Cassandra, DynamoDB

CA (Consistency + Availability):
- Невозможно в распределенных системах
- Только в single-node системах
```

#### BASE vs ACID
```
ACID (Традиционные БД):
- Atomicity: Все или ничего
- Consistency: Валидные данные
- Isolation: Изоляция транзакций
- Durability: Постоянство данных

BASE (Distributed Systems):
- Basically Available: Система доступна
- Soft state: Состояние может меняться
- Eventually consistent: Консистентность в итоге
```

### Масштабирование систем

#### Horizontal vs Vertical Scaling
```
Vertical Scaling (Scale Up):
+ Простота реализации
+ Consistency гарантии
- Лимит мощности
- Single point of failure
- Дороговизна

Horizontal Scaling (Scale Out):
+ Теоретически бесконечное масштабирование
+ Отказоустойчивость
+ Экономическая эффективность
- Сложность архитектуры
- Consistency challenges
- Network latency
```

#### Стратегии масштабирования базы данных

##### Database Sharding
```python
# Пример стратегии шардинга
class ShardingStrategy:
    def __init__(self, shards_count=4):
        self.shards_count = shards_count
    
    def get_shard_key(self, user_id):
        return user_id % self.shards_count
    
    def get_database_config(self, shard_key):
        return {
            'host': f'db-shard-{shard_key}.example.com',
            'database': f'app_shard_{shard_key}'
        }

# Challenges:
# - Rebalancing при добавлении шардов
# - Cross-shard queries
# - Hotspots
```

##### Database Replication
```sql
-- Master-Slave setup
-- Master: Все writes
-- Slaves: Reads, backups

-- Read-write splitting
SELECT * FROM users WHERE id = 1;  -- Slave
INSERT INTO users (name) VALUES ('John');  -- Master
UPDATE users SET name = 'Jane' WHERE id = 1;  -- Master
```

#### Кэширование стратегии

##### Cache Patterns
```python
# 1. Cache-Aside (Lazy Loading)
def get_user_cache_aside(user_id):
    # Сначала проверяем кэш
    user = cache.get(f"user:{user_id}")
    if user is None:
        # Если нет в кэше, загружаем из БД
        user = db.get_user(user_id)
        # Сохраняем в кэш
        cache.set(f"user:{user_id}", user, ttl=3600)
    return user

# 2. Write-Through
def update_user_write_through(user_id, data):
    # Обновляем в БД
    db.update_user(user_id, data)
    # Обновляем в кэше
    cache.set(f"user:{user_id}", data, ttl=3600)

# 3. Write-Behind (Write-Back)
def update_user_write_behind(user_id, data):
    # Сначала обновляем кэш
    cache.set(f"user:{user_id}", data, ttl=3600)
    # Асинхронно обновляем БД
    async_queue.put(('update_user', user_id, data))
```

##### Cache Invalidation
```python
# TTL-based invalidation
cache.set("user:1", user_data, ttl=3600)  # 1 час

# Event-based invalidation
class UserService:
    def update_user(self, user_id, data):
        db.update_user(user_id, data)
        # Инвалидируем кэш
        cache.delete(f"user:{user_id}")
        # Или обновляем
        cache.set(f"user:{user_id}", data, ttl=3600)

# Cache warming
def warm_cache():
    popular_users = db.get_popular_users()
    for user in popular_users:
        cache.set(f"user:{user.id}", user, ttl=7200)
```

---

## Архитектурные паттерны

### Distributed Systems Patterns

#### Circuit Breaker Pattern
```python
import time
from enum import Enum

class CircuitState(Enum):
    CLOSED = "closed"      # Нормальная работа
    OPEN = "open"         # Обрыв цепи
    HALF_OPEN = "half_open"  # Проверка восстановления

class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.failure_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED
    
    def call(self, func, *args, **kwargs):
        if self.state == CircuitState.OPEN:
            if time.time() - self.last_failure_time > self.timeout:
                self.state = CircuitState.HALF_OPEN
            else:
                raise Exception("Circuit breaker is OPEN")
        
        try:
            result = func(*args, **kwargs)
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise e
    
    def on_success(self):
        self.failure_count = 0
        self.state = CircuitState.CLOSED
    
    def on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN

# Использование
circuit_breaker = CircuitBreaker()

def call_external_service():
    return circuit_breaker.call(external_api.get_data)
```

#### Bulkhead Pattern
```python
# Изоляция ресурсов для разных типов операций
class ResourcePool:
    def __init__(self, pool_size=10):
        self.pool = Queue(maxsize=pool_size)
        for _ in range(pool_size):
            self.pool.put(create_connection())
    
    def get_connection(self):
        return self.pool.get(timeout=5)
    
    def return_connection(self, conn):
        self.pool.put(conn)

# Отдельные пулы для разных операций
class DatabaseService:
    def __init__(self):
        self.read_pool = ResourcePool(pool_size=20)
        self.write_pool = ResourcePool(pool_size=5)
        self.analytics_pool = ResourcePool(pool_size=3)
    
    def read_operation(self, query):
        conn = self.read_pool.get_connection()
        try:
            return conn.execute(query)
        finally:
            self.read_pool.return_connection(conn)
    
    def write_operation(self, query):
        conn = self.write_pool.get_connection()
        try:
            return conn.execute(query)
        finally:
            self.write_pool.return_connection(conn)
```

#### Saga Pattern
```python
# Distributed Transaction Management
class SagaOrchestrator:
    def __init__(self):
        self.steps = []
        self.compensations = []
    
    def add_step(self, action, compensation):
        self.steps.append(action)
        self.compensations.append(compensation)
    
    def execute(self):
        executed_steps = []
        
        try:
            for i, step in enumerate(self.steps):
                result = step()
                executed_steps.append(i)
                
        except Exception as e:
            # Rollback в обратном порядке
            for step_index in reversed(executed_steps):
                try:
                    self.compensations[step_index]()
                except Exception as comp_error:
                    # Логируем ошибку компенсации
                    logger.error(f"Compensation failed: {comp_error}")
            
            raise e

# Пример использования
def create_order_saga():
    saga = SagaOrchestrator()
    
    # Резервирование товара
    saga.add_step(
        action=lambda: inventory_service.reserve_item(item_id, quantity),
        compensation=lambda: inventory_service.release_item(item_id, quantity)
    )
    
    # Списание средств
    saga.add_step(
        action=lambda: payment_service.charge(user_id, amount),
        compensation=lambda: payment_service.refund(user_id, amount)
    )
    
    # Создание заказа
    saga.add_step(
        action=lambda: order_service.create_order(order_data),
        compensation=lambda: order_service.cancel_order(order_id)
    )
    
    return saga.execute()
```

### Event-Driven Architecture

#### Event Sourcing Advanced
```python
class EventStore:
    def __init__(self):
        self.events = []
        self.snapshots = {}
    
    def append_event(self, aggregate_id, event):
        event_record = {
            'aggregate_id': aggregate_id,
            'event_type': event.__class__.__name__,
            'event_data': event.to_dict(),
            'timestamp': time.time(),
            'version': self.get_next_version(aggregate_id)
        }
        self.events.append(event_record)
    
    def get_events(self, aggregate_id, from_version=0):
        return [e for e in self.events 
                if e['aggregate_id'] == aggregate_id 
                and e['version'] > from_version]
    
    def create_snapshot(self, aggregate_id, aggregate_state, version):
        self.snapshots[aggregate_id] = {
            'state': aggregate_state,
            'version': version,
            'timestamp': time.time()
        }
    
    def get_snapshot(self, aggregate_id):
        return self.snapshots.get(aggregate_id)

class EventSourcingRepository:
    def __init__(self, event_store):
        self.event_store = event_store
    
    def save(self, aggregate):
        for event in aggregate.uncommitted_events:
            self.event_store.append_event(aggregate.id, event)
        
        # Создаем снимок каждые 100 событий
        if len(aggregate.uncommitted_events) % 100 == 0:
            self.event_store.create_snapshot(
                aggregate.id, 
                aggregate.get_state(), 
                aggregate.version
            )
        
        aggregate.mark_events_as_committed()
    
    def load(self, aggregate_id):
        # Пытаемся загрузить снимок
        snapshot = self.event_store.get_snapshot(aggregate_id)
        
        if snapshot:
            aggregate = Aggregate.from_snapshot(snapshot)
            from_version = snapshot['version']
        else:
            aggregate = Aggregate(aggregate_id)
            from_version = 0
        
        # Загружаем события после снимка
        events = self.event_store.get_events(aggregate_id, from_version)
        
        for event_record in events:
            event = self.deserialize_event(event_record)
            aggregate.apply_event(event)
        
        return aggregate
```

---

## Принятие архитектурных решений

### Architecture Decision Records (ADRs)

#### Структура ADR
```markdown
# ADR-001: Выбор базы данных для микросервиса пользователей

## Статус
Принято

## Контекст
Нам нужно выбрать базу данных для микросервиса пользователей.
Требования:
- Высокая доступность
- Консистентность данных
- Поддержка транзакций
- Масштабируемость до 1M пользователей

## Рассмотренные варианты
1. PostgreSQL
2. MongoDB
3. DynamoDB

## Решение
Выбран PostgreSQL

## Обоснование
- Соответствует требованиям ACID
- Отличная экосистема
- Команда имеет экспертизу
- Горизонтальное масштабирование через шардинг

## Последствия
Положительные:
- Надежность и консистентность данных
- Богатые возможности SQL
- Хорошая производительность

Отрицательные:
- Сложность настройки шардинга
- Необходимость в DBA экспертизе
```

### Архитектурные характеристики (качества)

#### Performance
```python
# Метрики производительности
class PerformanceMetrics:
    def __init__(self):
        self.response_times = []
        self.throughput_counter = 0
        self.error_counter = 0
    
    def record_response_time(self, response_time):
        self.response_times.append(response_time)
        
        # Sliding window для последних 1000 запросов
        if len(self.response_times) > 1000:
            self.response_times.pop(0)
    
    def get_percentiles(self):
        sorted_times = sorted(self.response_times)
        length = len(sorted_times)
        
        return {
            'p50': sorted_times[int(length * 0.5)],
            'p95': sorted_times[int(length * 0.95)],
            'p99': sorted_times[int(length * 0.99)]
        }
    
    def get_throughput(self):
        return self.throughput_counter / 60  # RPS
```

#### Availability
```python
# Расчет доступности
class AvailabilityCalculator:
    def __init__(self):
        self.total_time = 0
        self.downtime = 0
    
    def calculate_availability(self):
        uptime = self.total_time - self.downtime
        return (uptime / self.total_time) * 100
    
    def calculate_sla_compliance(self, target_availability=99.9):
        current_availability = self.calculate_availability()
        return current_availability >= target_availability
    
    def calculate_allowed_downtime(self, period_hours=24*30):
        # Для 99.9% доступности
        allowed_downtime = period_hours * 3.6 * 0.1  # 0.1% от времени
        return allowed_downtime  # в минутах
```

#### Scalability
```python
# Анализ точек масштабирования
class ScalabilityAnalyzer:
    def __init__(self):
        self.bottlenecks = []
        self.capacity_limits = {}
    
    def analyze_database_capacity(self, current_load, growth_rate):
        # Прогнозирование нагрузки
        projected_load = current_load * (1 + growth_rate) ** 12  # на год
        
        if projected_load > self.capacity_limits.get('database', float('inf')):
            self.bottlenecks.append({
                'component': 'database',
                'type': 'capacity',
                'timeline': 'within 12 months',
                'solution': 'horizontal scaling or optimization'
            })
    
    def analyze_cpu_usage(self, cpu_percentiles):
        if cpu_percentiles['p95'] > 80:
            self.bottlenecks.append({
                'component': 'cpu',
                'type': 'utilization',
                'timeline': 'immediate',
                'solution': 'vertical scaling or load balancing'
            })
```

---

## Техническое планирование

### Оценка сложности

#### Story Points и Planning Poker
```python
class StoryPointEstimator:
    def __init__(self):
        self.fibonacci = [1, 2, 3, 5, 8, 13, 21, 34, 55, 89]
        self.complexity_factors = {
            'technical_complexity': 0.3,
            'business_complexity': 0.2,
            'risk_factor': 0.2,
            'unknowns': 0.3
        }
    
    def estimate_story(self, task):
        score = 0
        
        # Техническая сложность
        if task.has_new_technology:
            score += 5
        if task.has_integrations:
            score += 3
        if task.has_performance_requirements:
            score += 2
        
        # Бизнес-сложность
        if task.affects_multiple_domains:
            score += 3
        if task.has_compliance_requirements:
            score += 5
        
        # Риски
        if task.has_external_dependencies:
            score += 2
        if task.has_tight_deadline:
            score += 1
        
        # Неизвестные факторы
        if task.research_required:
            score += 8
        if task.proof_of_concept_needed:
            score += 5
        
        return self.map_to_fibonacci(score)
    
    def map_to_fibonacci(self, score):
        for fib in self.fibonacci:
            if score <= fib:
                return fib
        return 89  # Максимальная сложность
```

### Технический долг

#### Debt Quadrant
```python
class TechnicalDebtAnalyzer:
    def __init__(self):
        self.debt_items = []
    
    def categorize_debt(self, debt_item):
        # Quadrant classification
        if debt_item.deliberate and debt_item.prudent:
            return "Deliberate & Prudent"  # Запланированный долг
        elif debt_item.deliberate and not debt_item.prudent:
            return "Deliberate & Reckless"  # Халатный долг
        elif not debt_item.deliberate and debt_item.prudent:
            return "Inadvertent & Prudent"  # Учебный долг
        else:
            return "Inadvertent & Reckless"  # Случайный долг
    
    def calculate_debt_impact(self, debt_item):
        impact_score = 0
        
        # Влияние на производительность команды
        if debt_item.slows_development:
            impact_score += debt_item.development_delay * 10
        
        # Влияние на качество
        if debt_item.increases_bugs:
            impact_score += debt_item.bug_probability * 20
        
        # Влияние на производительность системы
        if debt_item.affects_performance:
            impact_score += debt_item.performance_impact * 15
        
        return impact_score
    
    def prioritize_debt_payment(self):
        # Сортировка по отношению impact/effort
        return sorted(self.debt_items, 
                     key=lambda x: x.impact / x.effort, 
                     reverse=True)
```

---

## Архитектурные trade-offs

### Consistency vs Performance
```python
# Пример выбора между консистентностью и производительностью
class ConsistencyStrategy:
    def __init__(self, strategy_type):
        self.strategy_type = strategy_type
    
    def handle_write(self, data):
        if self.strategy_type == "strong_consistency":
            # Synchronous replication
            return self.synchronous_write(data)
        elif self.strategy_type == "eventual_consistency":
            # Asynchronous replication
            return self.asynchronous_write(data)
        elif self.strategy_type == "session_consistency":
            # Read your own writes
            return self.session_consistent_write(data)
    
    def synchronous_write(self, data):
        # Медленнее, но гарантированная консистентность
        primary_result = self.primary_db.write(data)
        
        # Ждем подтверждения от всех реплик
        for replica in self.replicas:
            replica.write(data)
        
        return primary_result
    
    def asynchronous_write(self, data):
        # Быстрее, но eventual consistency
        primary_result = self.primary_db.write(data)
        
        # Асинхронно реплицируем
        for replica in self.replicas:
            self.async_queue.put(('write', replica, data))
        
        return primary_result
```

### Latency vs Throughput
```python
class LoadBalancingStrategy:
    def __init__(self, strategy="round_robin"):
        self.strategy = strategy
        self.servers = []
        self.current_index = 0
    
    def select_server(self, request):
        if self.strategy == "round_robin":
            # Высокий throughput, но может быть higher latency
            return self.round_robin_select()
        elif self.strategy == "least_connections":
            # Лучший latency, но lower throughput
            return self.least_connections_select()
        elif self.strategy == "weighted_response_time":
            # Баланс между latency и throughput
            return self.weighted_response_time_select()
    
    def round_robin_select(self):
        server = self.servers[self.current_index]
        self.current_index = (self.current_index + 1) % len(self.servers)
        return server
    
    def least_connections_select(self):
        return min(self.servers, key=lambda s: s.active_connections)
    
    def weighted_response_time_select(self):
        # Выбираем сервер с учетом времени отклика
        weights = [1.0 / s.avg_response_time for s in self.servers]
        return self.weighted_random_choice(self.servers, weights)
```

---

## Практические примеры

### Микросервисная архитектура: E-commerce

#### Service Decomposition
```python
# Декомпозиция по бизнес-доменам
class EcommerceArchitecture:
    def __init__(self):
        self.services = {
            'user_service': {
                'responsibilities': [
                    'user_registration', 'authentication', 
                    'user_profile', 'preferences'
                ],
                'data': ['users', 'profiles', 'auth_tokens'],
                'apis': ['REST', 'GraphQL']
            },
            'product_service': {
                'responsibilities': [
                    'product_catalog', 'inventory', 
                    'search', 'recommendations'
                ],
                'data': ['products', 'categories', 'inventory'],
                'apis': ['REST', 'GraphQL', 'Search API']
            },
            'order_service': {
                'responsibilities': [
                    'order_creation', 'order_tracking', 
                    'order_history'
                ],
                'data': ['orders', 'order_items', 'order_status'],
                'apis': ['REST', 'Events']
            },
            'payment_service': {
                'responsibilities': [
                    'payment_processing', 'refunds', 
                    'payment_methods'
                ],
                'data': ['payments', 'transactions', 'payment_methods'],
                'apis': ['REST', 'Webhooks']
            }
        }
    
    def analyze_service_interactions(self):
        interactions = {
            'user_to_product': 'sync_api_call',
            'product_to_inventory': 'sync_api_call',
            'order_to_payment': 'async_event',
            'order_to_user': 'async_notification',
            'payment_to_order': 'webhook'
        }
        return interactions
```

#### Data Consistency Strategy
```python
class DataConsistencyManager:
    def __init__(self):
        self.saga_orchestrator = SagaOrchestrator()
        self.event_store = EventStore()
    
    def process_order(self, order_data):
        # Используем Saga для распределенной транзакции
        saga = self.create_order_saga(order_data)
        
        try:
            result = saga.execute()
            # Публикуем событие успешного заказа
            self.event_store.publish_event(
                OrderCompletedEvent(order_id=result.order_id)
            )
            return result
        except Exception as e:
            # Saga автоматически откатывает изменения
            self.event_store.publish_event(
                OrderFailedEvent(order_data=order_data, error=str(e))
            )
            raise
    
    def create_order_saga(self, order_data):
        saga = SagaOrchestrator()
        
        # Шаг 1: Резервирование товара
        saga.add_step(
            action=lambda: self.inventory_service.reserve_items(
                order_data['items']
            ),
            compensation=lambda: self.inventory_service.release_items(
                order_data['items']
            )
        )
        
        # Шаг 2: Создание заказа
        saga.add_step(
            action=lambda: self.order_service.create_order(order_data),
            compensation=lambda: self.order_service.cancel_order(
                order_data['order_id']
            )
        )
        
        # Шаг 3: Обработка платежа
        saga.add_step(
            action=lambda: self.payment_service.process_payment(
                order_data['payment_info']
            ),
            compensation=lambda: self.payment_service.refund_payment(
                order_data['payment_info']
            )
        )
        
        return saga
```

### Performance Optimization

#### Database Optimization
```python
class DatabaseOptimizer:
    def __init__(self, db_connection):
        self.db = db_connection
        self.query_analyzer = QueryAnalyzer()
    
    def optimize_query(self, query):
        # Анализируем план выполнения
        execution_plan = self.db.explain(query)
        
        optimizations = []
        
        # Проверяем использование индексов
        if execution_plan.has_full_table_scan():
            optimizations.append({
                'type': 'index_recommendation',
                'suggestion': self.suggest_index(query)
            })
        
        # Проверяем JOIN операции
        if execution_plan.has_nested_loop_join():
            optimizations.append({
                'type': 'join_optimization',
                'suggestion': 'Consider hash join or merge join'
            })
        
        return optimizations
    
    def suggest_index(self, query):
        # Анализируем WHERE условия
        where_conditions = self.query_analyzer.extract_where_conditions(query)
        
        # Предлагаем составной индекс
        index_columns = []
        for condition in where_conditions:
            if condition.is_equality():
                index_columns.append(condition.column)
        
        return f"CREATE INDEX idx_optimized ON {query.table} ({', '.join(index_columns)})"
```

---

## Заключение

Для перехода на Senior уровень в архитектуре необходимо:

1. **Системное мышление**: Понимание trade-offs и архитектурных решений
2. **Масштабирование**: Знание паттернов и стратегий масштабирования
3. **Принятие решений**: Способность обосновать архитектурные выборы
4. **Техническое планирование**: Оценка сложности и управление техническим долгом
5. **Практический опыт**: Реализация сложных архитектурных решений

### Дальнейшее изучение
- Книги: "Designing Data-Intensive Applications", "Building Microservices"
- Практика: Участие в архитектурных обзорах
- Сообщества: Архитектурные группы и конференции 