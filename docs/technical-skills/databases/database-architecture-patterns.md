# Архитектурные паттерны баз данных 🏗️

## 🏠 Навигация
**[[README|🏠 Базы данных]]** → **Архитектурные паттерны**

**Связанные темы:**
- 🐘 [[postgresql|PostgreSQL]]
- 🍃 [[mongodb|MongoDB]]
- 🔴 [[redis|Redis]]  
- 🏭 [[data-warehousing|Data Warehousing]]
- 🏗️ [[database-design|Проектирование БД]]

---

## 🎯 Архитектурные паттерны для современных приложений

В современной разработке критически важно правильно выбрать архитектурный подход к работе с базами данных. Различные паттерны решают разные проблемы масштабирования, производительности и поддержки.

---

## 🔧 Database per Service (Микросервисы)

### 🏗️ Принцип паттерна

**Database per Service** — это архитектурный паттерн, где каждый микросервис имеет свою собственную базу данных. Это обеспечивает слабую связанность и независимость сервисов.

### ✨ Преимущества
- **🔒 Изоляция данных** - каждый сервис управляет своими данными
- **🚀 Независимое развертывание** - изменения в одной БД не влияют на другие
- **⚖️ Технологическое разнообразие** - разные БД для разных задач
- **📈 Независимое масштабирование** - каждая БД масштабируется по потребности

### ⚠️ Недостатки
- **🔄 Сложность транзакций** - невозможны ACID транзакции между сервисами
- **📊 Сложность аналитики** - данные разбросаны по разным БД
- **🔗 Управление консистентностью** - eventual consistency

```python
# services/architecture/database_per_service.py
class ServiceDatabaseManager:
    """Управление базами данных для каждого сервиса"""
    
    def __init__(self):
        self.services = {
            'user_service': {
                'type': 'postgresql',
                'connection': 'postgresql://user:pass@db1:5432/users',
                'purpose': 'User profiles, authentication'
            },
            'order_service': {
                'type': 'postgresql', 
                'connection': 'postgresql://user:pass@db2:5432/orders',
                'purpose': 'Order processing, transactions'
            },
            'product_service': {
                'type': 'mongodb',
                'connection': 'mongodb://db3:27017/products',
                'purpose': 'Product catalog, flexible schema'
            },
            'session_service': {
                'type': 'redis',
                'connection': 'redis://cache1:6379/0',
                'purpose': 'User sessions, temporary data'
            },
            'analytics_service': {
                'type': 'clickhouse',
                'connection': 'clickhouse://analytics:9000/metrics',
                'purpose': 'Real-time analytics, reporting'
            }
        }
    
    def get_service_db(self, service_name: str) -> Dict[str, str]:
        """Получение конфигурации БД для сервиса"""
        return self.services.get(service_name, {})
    
    def cross_service_query_pattern(self):
        """Паттерн для запросов между сервисами"""
        return {
            'approach': 'API calls between services',
            'avoid': 'Direct database joins across services',
            'eventual_consistency': 'Use events for data synchronization',
            'saga_pattern': 'For distributed transactions'
        }

# Пример сервиса пользователей
class UserService:
    def __init__(self):
        self.db = PostgreSQLConnection('postgresql://user:pass@db1:5432/users')
        self.event_publisher = EventPublisher()
    
    def create_user(self, user_data: Dict) -> str:
        """Создание пользователя"""
        user_id = self.db.insert('users', user_data)
        
        # Публикация события для других сервисов
        self.event_publisher.publish('user_created', {
            'user_id': user_id,
            'email': user_data['email'],
            'name': user_data['name']
        })
        
        return user_id
    
    def get_user(self, user_id: str) -> Dict:
        """Получение пользователя"""
        return self.db.find_by_id('users', user_id)

# Пример сервиса заказов
class OrderService:
    def __init__(self):
        self.db = PostgreSQLConnection('postgresql://user:pass@db2:5432/orders')
        self.user_service_api = UserServiceAPI()
        self.event_publisher = EventPublisher()
    
    def create_order(self, order_data: Dict) -> str:
        """Создание заказа"""
        # Проверяем пользователя через API (не через прямой доступ к БД)
        user = self.user_service_api.get_user(order_data['user_id'])
        if not user:
            raise ValueError("User not found")
        
        order_id = self.db.insert('orders', order_data)
        
        # Публикация события
        self.event_publisher.publish('order_created', {
            'order_id': order_id,
            'user_id': order_data['user_id'],
            'total_amount': order_data['total_amount']
        })
        
        return order_id
```

---

## 📊 CQRS (Command Query Responsibility Segregation)

### 🎯 Принцип паттерна

**CQRS** разделяет операции чтения и записи на разные модели, что позволяет оптимизировать каждую модель под свои задачи.

### ✨ Преимущества
- **⚡ Оптимизация производительности** - разные структуры для чтения и записи
- **📈 Независимое масштабирование** - read и write модели масштабируются отдельно
- **🔍 Сложные запросы** - денормализованные данные для быстрого чтения
- **🛡️ Безопасность** - разделение прав доступа

### ⚠️ Недостатки
- **🔄 Сложность синхронизации** - eventual consistency между моделями
- **📊 Увеличение сложности** - больше кода и инфраструктуры
- **💾 Дублирование данных** - данные хранятся в нескольких местах

```python
# services/architecture/cqrs_pattern.py
import json
from datetime import datetime
from typing import Dict, List, Any

class CQRSOrderService:
    """CQRS паттерн для сервиса заказов"""
    
    def __init__(self):
        # Командная сторона - для записи (нормализованная структура)
        self.command_db = PostgreSQLConnection('postgresql://cmd:5432/orders')
        
        # Сторона запросов - для чтения (денормализованные данные)
        self.read_db = MongoDBConnection('mongodb://read:27017/orders_read')
        
        # Событийный поток для синхронизации
        self.event_stream = RedisStreamService()
    
    # ===== КОМАНДЫ (Write Side) =====
    def create_order(self, order_data: Dict) -> str:
        """Создание заказа (команда)"""
        with self.command_db.transaction():
            # Валидация бизнес-правил
            if not self._validate_order(order_data):
                raise ValueError("Invalid order data")
            
            # Сохранение в нормализованной структуре
            order_id = self.command_db.insert('orders', {
                'user_id': order_data['user_id'],
                'status': 'pending',
                'created_at': datetime.now()
            })
            
            # Сохранение позиций заказа
            for item in order_data['items']:
                self.command_db.insert('order_items', {
                    'order_id': order_id,
                    'product_id': item['product_id'],
                    'quantity': item['quantity'],
                    'price': item['price']
                })
            
            # Публикация события для обновления read-модели
            self.event_stream.add_event('order_events', {
                'event_type': 'order_created',
                'order_id': order_id,
                'user_id': order_data['user_id'],
                'data': json.dumps(order_data),
                'timestamp': datetime.now().isoformat()
            })
            
            return order_id
    
    def update_order_status(self, order_id: str, new_status: str):
        """Обновление статуса заказа"""
        with self.command_db.transaction():
            self.command_db.update(
                'orders', 
                {'id': order_id}, 
                {'status': new_status, 'updated_at': datetime.now()}
            )
            
            # Событие для синхронизации
            self.event_stream.add_event('order_events', {
                'event_type': 'order_status_updated',
                'order_id': order_id,
                'new_status': new_status,
                'timestamp': datetime.now().isoformat()
            })
    
    # ===== ЗАПРОСЫ (Read Side) =====
    def get_order_details(self, order_id: str) -> Dict:
        """Получение деталей заказа (запрос)"""
        # Чтение из денормализованной структуры для быстрого доступа
        return self.read_db.find_one('order_details', {'order_id': order_id})
    
    def get_user_orders(self, user_id: str, limit: int = 10) -> List[Dict]:
        """Получение заказов пользователя"""
        return list(self.read_db.find(
            'user_orders_view',
            {'user_id': user_id},
            limit=limit,
            sort=[('created_at', -1)]
        ))
    
    def get_orders_by_status(self, status: str, limit: int = 50) -> List[Dict]:
        """Получение заказов по статусу"""
        return list(self.read_db.find(
            'order_details',
            {'status': status},
            limit=limit
        ))
    
    def get_revenue_analytics(self, date_from: str, date_to: str) -> Dict:
        """Аналитика доходов"""
        pipeline = [
            {
                '$match': {
                    'created_at': {
                        '$gte': date_from,
                        '$lte': date_to
                    }
                }
            },
            {
                '$group': {
                    '_id': {'$dateToString': {'format': '%Y-%m-%d', 'date': '$created_at'}},
                    'total_revenue': {'$sum': '$total_amount'},
                    'order_count': {'$sum': 1},
                    'avg_order_value': {'$avg': '$total_amount'}
                }
            },
            {
                '$sort': {'_id': 1}
            }
        ]
        
        return list(self.read_db.aggregate('order_details', pipeline))
    
    # ===== ОБРАБОТЧИКИ СОБЫТИЙ =====
    def handle_order_created_event(self, event: Dict):
        """Обработка события создания заказа"""
        order_data = json.loads(event['data'])
        
        # Обогащение данных для read-модели
        user_info = self._get_user_info(order_data['user_id'])
        product_info = self._get_products_info([item['product_id'] for item in order_data['items']])
        
        # Создание денормализованной записи для быстрых запросов
        denormalized_order = {
            'order_id': event['order_id'],
            'user_id': order_data['user_id'],
            'user_name': user_info['name'],
            'user_email': user_info['email'],
            'user_city': user_info.get('city', ''),
            'items': [
                {
                    'product_id': item['product_id'],
                    'product_name': product_info[item['product_id']]['name'],
                    'category': product_info[item['product_id']]['category'],
                    'brand': product_info[item['product_id']].get('brand', ''),
                    'quantity': item['quantity'],
                    'price': item['price'],
                    'line_total': item['quantity'] * item['price']
                }
                for item in order_data['items']
            ],
            'total_amount': sum(item['quantity'] * item['price'] for item in order_data['items']),
            'item_count': sum(item['quantity'] for item in order_data['items']),
            'status': 'pending',
            'created_at': datetime.fromisoformat(event['timestamp']),
            'updated_at': datetime.fromisoformat(event['timestamp'])
        }
        
        # Сохранение в read-модель
        self.read_db.insert('order_details', denormalized_order)
        
        # Обновление пользовательского представления
        self.read_db.insert('user_orders_view', {
            'user_id': order_data['user_id'],
            'order_id': event['order_id'], 
            'total_amount': denormalized_order['total_amount'],
            'item_count': denormalized_order['item_count'],
            'status': 'pending',
            'created_at': datetime.fromisoformat(event['timestamp'])
        })
    
    def handle_order_status_updated_event(self, event: Dict):
        """Обработка обновления статуса заказа"""
        # Обновление в детальном представлении
        self.read_db.update(
            'order_details',
            {'order_id': event['order_id']},
            {
                'status': event['new_status'],
                'updated_at': datetime.fromisoformat(event['timestamp'])
            }
        )
        
        # Обновление в пользовательском представлении
        self.read_db.update(
            'user_orders_view',
            {'order_id': event['order_id']},
            {
                'status': event['new_status'],
                'updated_at': datetime.fromisoformat(event['timestamp'])
            }
        )
    
    # ===== ВСПОМОГАТЕЛЬНЫЕ МЕТОДЫ =====
    def _validate_order(self, order_data: Dict) -> bool:
        """Валидация данных заказа"""
        if not order_data.get('user_id'):
            return False
        if not order_data.get('items'):
            return False
        if any(item['quantity'] <= 0 for item in order_data['items']):
            return False
        return True
    
    def _get_user_info(self, user_id: str) -> Dict:
        """Получение информации о пользователе"""
        # Здесь должен быть вызов User Service API
        return {
            'name': 'John Doe',
            'email': 'john@example.com',
            'city': 'Moscow'
        }
    
    def _get_products_info(self, product_ids: List[str]) -> Dict:
        """Получение информации о продуктах"""
        # Здесь должен быть вызов Product Service API
        return {
            product_id: {
                'name': f'Product {product_id}',
                'category': 'Electronics',
                'brand': 'Brand X'
            }
            for product_id in product_ids
        }

# Использование
cqrs_service = CQRSOrderService()

# Создание заказа (команда)
order_id = cqrs_service.create_order({
    'user_id': 'user123',
    'items': [
        {'product_id': 'prod1', 'quantity': 2, 'price': 100.0},
        {'product_id': 'prod2', 'quantity': 1, 'price': 250.0}
    ]
})

# Получение деталей заказа (запрос)
order_details = cqrs_service.get_order_details(order_id)

# Получение заказов пользователя (запрос)
user_orders = cqrs_service.get_user_orders('user123')

# Аналитика (запрос)
revenue_data = cqrs_service.get_revenue_analytics('2024-01-01', '2024-12-31')
```

---

## 📊 Сравнительная таблица баз данных

### 🔍 Выбор подходящей базы данных

| Характеристика | PostgreSQL | MongoDB | Redis | ClickHouse |
|---|---|---|---|---|
| **🏗️ Тип** | Реляционная | Документная | Key-Value | Колоночная |
| **🔒 ACID** | Полная поддержка | Поддержка (с 4.0) | Ограниченная | Ограниченная |
| **📈 Масштабирование** | Вертикальное + репликация | Горизонтальное (шардинг) | Горизонтальное (кластер) | Горизонтальное |
| **📋 Схема** | Фиксированная | Гибкая | Нет схемы | Фиксированная |
| **🗣️ Язык запросов** | SQL | MongoDB Query API | Commands/Lua | SQL |
| **⚡ OLTP производительность** | Высокая | Высокая | Очень высокая | Средняя |
| **📊 OLAP производительность** | Средняя | Низкая | Низкая | Очень высокая |
| **💾 Использование памяти** | Умеренное | Высокое | Очень высокое | Умеренное |
| **💿 Персистентность** | Диск | Диск + память | Память + диск | Диск |
| **🎯 Лучше всего для** | Транзакции, отчеты | Быстрое развитие, CMS | Кэш, сессии, очереди | Аналитика, DWH |
| **💰 Стоимость владения** | Низкая | Средняя | Низкая | Средняя |
| **👥 Экосистема** | Очень богатая | Богатая | Специализированная | Развивающаяся |

### 🎯 Рекомендации по выбору

#### 🐘 Используйте PostgreSQL когда:
- Нужны ACID транзакции
- Сложные реляционные запросы
- Строгая консистентность данных
- Финансовые приложения
- Отчетность и аналитика (средние объемы)

#### 🍃 Используйте MongoDB когда:
- Быстрое прототипирование
- Гибкая схема данных
- Контент-менеджмент системы
- Каталоги продуктов
- JSON-документы как основной формат

#### 🔴 Используйте Redis когда:
- Кэширование данных
- Сессии пользователей
- Очереди задач (простые)
- Real-time лидерборды
- Pub/Sub системы

#### 📊 Используйте ClickHouse когда:
- Big Data аналитика
- Data warehousing
- Логирование и метрики
- Временные ряды
- OLAP кубы

---

## 🏗️ Паттерны комбинирования баз данных

### 🔄 Polyglot Persistence

```python
# services/architecture/polyglot_persistence.py
class ECommerceDataArchitecture:
    """Пример полиглот персистенции для eCommerce"""
    
    def __init__(self):
        # PostgreSQL для транзакционных данных
        self.transactional_db = PostgreSQL('postgresql://tx:5432/ecommerce')
        
        # MongoDB для каталога продуктов
        self.catalog_db = MongoDB('mongodb://catalog:27017/products')
        
        # Redis для кэширования и сессий
        self.cache_db = Redis('redis://cache:6379/0')
        
        # ClickHouse для аналитики
        self.analytics_db = ClickHouse('clickhouse://analytics:9000/metrics')
        
        # Elasticsearch для поиска
        self.search_db = Elasticsearch('http://search:9200')
    
    def create_user(self, user_data: Dict) -> str:
        """Создание пользователя - PostgreSQL"""
        return self.transactional_db.insert('users', user_data)
    
    def add_product(self, product_data: Dict) -> str:
        """Добавление продукта - MongoDB"""
        product_id = self.catalog_db.insert('products', product_data)
        
        # Индексация для поиска
        self.search_db.index('products', product_id, product_data)
        
        return product_id
    
    def create_order(self, order_data: Dict) -> str:
        """Создание заказа - PostgreSQL"""
        order_id = self.transactional_db.insert('orders', order_data)
        
        # Аналитическое событие - ClickHouse
        self.analytics_db.insert('order_events', {
            'order_id': order_id,
            'user_id': order_data['user_id'],
            'total_amount': order_data['total_amount'],
            'timestamp': datetime.now()
        })
        
        return order_id
    
    def cache_user_session(self, session_id: str, user_data: Dict):
        """Кэширование сессии - Redis"""
        self.cache_db.setex(
            f"session:{session_id}",
            3600,  # 1 час
            json.dumps(user_data)
        )
    
    def search_products(self, query: str) -> List[Dict]:
        """Поиск продуктов - Elasticsearch"""
        return self.search_db.search('products', {
            'query': {
                'multi_match': {
                    'query': query,
                    'fields': ['name', 'description', 'category']
                }
            }
        })
    
    def get_sales_analytics(self, date_from: str, date_to: str) -> Dict:
        """Аналитика продаж - ClickHouse"""
        return self.analytics_db.execute(f"""
            SELECT 
                toDate(timestamp) as date,
                count() as order_count,
                sum(total_amount) as revenue,
                avg(total_amount) as avg_order_value
            FROM order_events
            WHERE date >= '{date_from}' AND date <= '{date_to}'
            GROUP BY date
            ORDER BY date
        """)
```

---

## 🎯 Практические задания

### 🔰 Начальный уровень
1. Создайте простую микросервисную архитектуру с отдельными БД для пользователей и заказов
2. Реализуйте базовый CQRS паттерн для одной сущности
3. Сравните производительность запросов в разных БД

### 🚀 Средний уровень
1. Реализуйте систему событий для синхронизации данных между сервисами
2. Создайте комплексную CQRS архитектуру с обработкой событий
3. Реализуйте polyglot persistence для интернет-магазина

### 💎 Продвинутый уровень
1. Создайте распределенную транзакционную систему с использованием Saga паттерна
2. Реализуйте Event Sourcing с снэпшотами
3. Создайте систему миграции данных между разными архитектурными паттернами

---

## 🌟 Лучшие практики

### ✅ Рекомендации

#### 🏗️ Архитектурный дизайн
- **Начинайте просто** - не усложняйте архитектуру без необходимости
- **Выбирайте БД под задачу** - каждая база данных решает определенные проблемы
- **Планируйте консистентность** - определите требования к консистентности заранее
- **Документируйте решения** - архитектурные решения должны быть понятны команде

#### 🔄 Управление данными
- **Владение данными** - каждый сервис владеет своими данными
- **API для доступа** - никогда не обращайтесь к чужой БД напрямую
- **Eventual consistency** - принимайте асинхронность в распределенных системах
- **Идемпотентность** - операции должны быть безопасными для повтора

#### 📊 Мониторинг и производительность
- **Метрики производительности** - отслеживайте латентность и throughput
- **Distributed tracing** - трассировка запросов между сервисами
- **Алертинг** - уведомления о проблемах в real-time
- **Capacity planning** - планирование ресурсов заранее

### ❌ Чего избегать

- **Distributed transactions** - избегайте 2PC, используйте Saga
- **Shared databases** - не делите БД между сервисами
- **Chatty interfaces** - минимизируйте количество сетевых вызовов
- **Tight coupling** - сервисы не должны зависеть от внутренней структуры других сервисов

---

## 🔗 Связанные темы

- 🐘 **[[postgresql|PostgreSQL]]** - для транзакционных данных
- 🍃 **[[mongodb|MongoDB]]** - для гибких схем и каталогов
- 🔴 **[[redis|Redis]]** - для кэширования и real-time функций
- 🏭 **[[data-warehousing|Data Warehousing]]** - для аналитических систем
- 🏗️ **[[database-design|Проектирование БД]]** - принципы проектирования

**[[README|🏠 Вернуться к обзору баз данных]]** 