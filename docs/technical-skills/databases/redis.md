# Redis - Высокопроизводительная In-Memory База Данных 🔴

## 🏠 Навигация
**[[README|🏠 Базы данных]]** → **Redis**

**Связанные темы:**
- 🐘 [[postgresql|PostgreSQL]]
- 🍃 [[mongodb|MongoDB]]
- 🏗️ [[database-design|Проектирование БД]]
- ⚡ [[sql-optimization|Оптимизация БД]]

---

## 🎯 Что такое Redis?

**Redis** (Remote Dictionary Server) — это высокопроизводительная key-value база данных в памяти с поддержкой различных структур данных. Идеально подходит для кеширования, очередей задач, сессий и real-time приложений.

### ✨ Ключевые возможности

1. **⚡ Высокая производительность** - все данные в памяти
2. **🏗️ Богатые структуры данных** - строки, списки, множества, хеши
3. **📡 Публикация/подписка** - real-time messaging
4. **🔄 Транзакции** - атомарные операции
5. **💾 Персистентность** - сохранение на диск
6. **🌐 Кластеризация** - горизонтальное масштабирование

---

## 🏗️ Структуры данных Redis

### 🔤 Строки (Strings)
```redis
# Основные операции
SET user:1001:name "John Doe"
GET user:1001:name
INCR user:1001:login_count
SETEX session:abc123 3600 "user_data"  # TTL 1 час

# Битовые операции
SETBIT user:1001:active_days 100 1  # День 100 активен
GETBIT user:1001:active_days 100
BITCOUNT user:1001:active_days       # Количество активных дней
```

### 🗂️ Хеши (Hashes) - для объектов
```redis
# Операции с хешами
HSET user:1001 name "John Doe" email "john@example.com" age 30
HGET user:1001 name
HMGET user:1001 name email
HGETALL user:1001
HINCRBY user:1001 login_count 1

# Проверка существования поля
HEXISTS user:1001 email
HDEL user:1001 temporary_field
```

### 📋 Списки (Lists) - для очередей
```redis
# Операции со списками
LPUSH notifications:user:1001 "New message"
LPUSH notifications:user:1001 "Friend request"
LRANGE notifications:user:1001 0 -1
RPOP notifications:user:1001

# Блокирующие операции для очередей
BLPOP task_queue 30  # Блокировка на 30 сек
BRPOP task_queue 0   # Бесконечное ожидание
```

### 🎯 Множества (Sets) - для уникальных значений
```redis
# Операции с множествами
SADD user:1001:interests "technology" "music" "sports"
SMEMBERS user:1001:interests
SISMEMBER user:1001:interests "technology"

# Операции между множествами
SINTER user:1001:interests user:1002:interests  # Пересечение
SUNION user:1001:interests user:1002:interests  # Объединение
SDIFF user:1001:interests user:1002:interests   # Разность
```

### 📊 Упорядоченные множества (Sorted Sets) - для рейтингов
```redis
# Операции с отсортированными множествами
ZADD leaderboard 1000 "user:1001" 1500 "user:1002" 800 "user:1003"
ZRANGE leaderboard 0 -1 WITHSCORES
ZREVRANGE leaderboard 0 9 WITHSCORES  # Топ 10
ZSCORE leaderboard "user:1001"        # Получить счет
ZRANK leaderboard "user:1001"         # Получить ранг
```

---

## 🐍 Python интеграция с Redis

### 🔧 Базовый сервис Redis

```python
# services/order-service/infrastructure/redis_service.py
import redis
import json
import time
from typing import Dict, List, Any, Optional
from datetime import datetime, timedelta

class RedisService:
    def __init__(self, host: str = "localhost", port: int = 6379, db: int = 0):
        self.redis_client = redis.Redis(
            host=host, 
            port=port, 
            db=db,
            decode_responses=True,
            socket_connect_timeout=5,
            socket_timeout=5,
            retry_on_timeout=True
        )
        
        # Проверка соединения
        try:
            self.redis_client.ping()
        except redis.ConnectionError:
            raise Exception("Failed to connect to Redis")
    
    # Кэширование данных
    def cache_user(self, user_id: str, user_data: Dict[str, Any], ttl: int = 3600):
        """Кэширование данных пользователя"""
        key = f"user:{user_id}"
        self.redis_client.setex(key, ttl, json.dumps(user_data))
    
    def get_cached_user(self, user_id: str) -> Optional[Dict[str, Any]]:
        """Получение кэшированных данных пользователя"""
        key = f"user:{user_id}"
        cached_data = self.redis_client.get(key)
        
        if cached_data:
            return json.loads(cached_data)
        return None
    
    def invalidate_user_cache(self, user_id: str):
        """Инвалидация кэша пользователя"""
        key = f"user:{user_id}"
        self.redis_client.delete(key)
    
    # Сессии
    def create_session(self, session_id: str, user_data: Dict[str, Any], ttl: int = 3600):
        """Создание сессии"""
        key = f"session:{session_id}"
        self.redis_client.setex(key, ttl, json.dumps(user_data))
    
    def get_session(self, session_id: str) -> Optional[Dict[str, Any]]:
        """Получение данных сессии"""
        key = f"session:{session_id}"
        session_data = self.redis_client.get(key)
        
        if session_data:
            return json.loads(session_data)
        return None
    
    def extend_session(self, session_id: str, ttl: int = 3600):
        """Продление сессии"""
        key = f"session:{session_id}"
        self.redis_client.expire(key, ttl)
    
    def delete_session(self, session_id: str):
        """Удаление сессии"""
        key = f"session:{session_id}"
        self.redis_client.delete(key)
    
    # Очереди задач
    def add_task_to_queue(self, queue_name: str, task_data: Dict[str, Any]):
        """Добавление задачи в очередь"""
        task_json = json.dumps(task_data)
        self.redis_client.lpush(f"queue:{queue_name}", task_json)
    
    def get_task_from_queue(self, queue_name: str, timeout: int = 0) -> Optional[Dict[str, Any]]:
        """Получение задачи из очереди"""
        result = self.redis_client.brpop(f"queue:{queue_name}", timeout=timeout)
        
        if result:
            queue_key, task_json = result
            return json.loads(task_json)
        return None
    
    def get_queue_length(self, queue_name: str) -> int:
        """Получение длины очереди"""
        return self.redis_client.llen(f"queue:{queue_name}")
    
    # Счетчики и лимиты
    def increment_counter(self, key: str, ttl: int = None) -> int:
        """Увеличение счетчика"""
        count = self.redis_client.incr(key)
        
        if ttl and count == 1:  # Устанавливаем TTL только при первом увеличении
            self.redis_client.expire(key, ttl)
        
        return count
    
    def check_rate_limit(self, user_id: str, action: str, limit: int, window: int) -> bool:
        """Проверка лимита действий"""
        key = f"rate_limit:{user_id}:{action}"
        current_count = self.increment_counter(key, window)
        
        return current_count <= limit
    
    def get_rate_limit_info(self, user_id: str, action: str) -> Dict[str, Any]:
        """Информация о лимите"""
        key = f"rate_limit:{user_id}:{action}"
        count = self.redis_client.get(key) or 0
        ttl = self.redis_client.ttl(key)
        
        return {
            "current_count": int(count),
            "reset_in": ttl if ttl > 0 else 0
        }
    
    # Публикация/подписка
    def publish_event(self, channel: str, event_data: Dict[str, Any]):
        """Публикация события"""
        event_json = json.dumps(event_data)
        self.redis_client.publish(channel, event_json)
    
    def subscribe_to_channel(self, channel: str):
        """Подписка на канал"""
        pubsub = self.redis_client.pubsub()
        pubsub.subscribe(channel)
        return pubsub
    
    # Рейтинги и лидерборды
    def add_to_leaderboard(self, leaderboard_name: str, user_id: str, score: float):
        """Добавление в лидерборд"""
        key = f"leaderboard:{leaderboard_name}"
        self.redis_client.zadd(key, {user_id: score})
    
    def get_top_players(self, leaderboard_name: str, limit: int = 10) -> List[Dict[str, Any]]:
        """Получение топ игроков"""
        key = f"leaderboard:{leaderboard_name}"
        players = self.redis_client.zrevrange(key, 0, limit - 1, withscores=True)
        
        return [
            {"user_id": player[0], "score": player[1], "rank": i + 1}
            for i, player in enumerate(players)
        ]
    
    def get_user_rank(self, leaderboard_name: str, user_id: str) -> Optional[int]:
        """Получение ранга пользователя"""
        key = f"leaderboard:{leaderboard_name}"
        rank = self.redis_client.zrevrank(key, user_id)
        
        return rank + 1 if rank is not None else None
    
    # Распределенные блокировки
    def acquire_lock(self, lock_name: str, ttl: int = 30) -> bool:
        """Получение распределенной блокировки"""
        key = f"lock:{lock_name}"
        lock_id = f"{time.time()}:{id(self)}"
        
        # Пытаемся установить блокировку
        if self.redis_client.set(key, lock_id, nx=True, ex=ttl):
            return True
        
        return False
    
    def release_lock(self, lock_name: str):
        """Освобождение блокировки"""
        key = f"lock:{lock_name}"
        
        # Lua скрипт для атомарного освобождения
        lua_script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
        """
        
        lock_id = f"{time.time()}:{id(self)}"
        self.redis_client.eval(lua_script, 1, key, lock_id)
    
    # Аналитика в реальном времени
    def track_page_view(self, page: str, user_id: str = None):
        """Отслеживание просмотров страниц"""
        # Общий счетчик
        self.redis_client.incr(f"page_views:{page}")
        
        # Счетчик за сегодня
        today = datetime.now().strftime("%Y-%m-%d")
        self.redis_client.incr(f"page_views:{page}:{today}")
        
        # Уникальные посетители
        if user_id:
            self.redis_client.sadd(f"unique_visitors:{page}:{today}", user_id)
    
    def get_page_analytics(self, page: str) -> Dict[str, Any]:
        """Получение аналитики страницы"""
        today = datetime.now().strftime("%Y-%m-%d")
        
        return {
            "total_views": int(self.redis_client.get(f"page_views:{page}") or 0),
            "today_views": int(self.redis_client.get(f"page_views:{page}:{today}") or 0),
            "unique_visitors_today": self.redis_client.scard(f"unique_visitors:{page}:{today}")
        }
    
    def close(self):
        """Закрытие соединения"""
        self.redis_client.close()
```

---

## 🌊 Redis Streams - Event Streaming

### 📡 Сервис Redis Streams

```python
# services/event-streaming/redis_streams.py
import redis
import json
import time
from typing import Dict, List, Any, Optional

class RedisStreamService:
    def __init__(self, host: str = "localhost", port: int = 6379, db: int = 0):
        self.redis_client = redis.Redis(host=host, port=port, db=db, decode_responses=True)
    
    def add_event(self, stream_name: str, event_data: Dict[str, Any]) -> str:
        """Добавление события в стрим"""
        event_id = self.redis_client.xadd(
            stream_name,
            event_data,
            maxlen=10000  # Ограничение размера стрима
        )
        return event_id
    
    def read_events(self, stream_name: str, consumer_group: str, 
                   consumer_name: str, count: int = 10) -> List[Dict]:
        """Чтение событий из стрима"""
        try:
            # Создание группы потребителей если не существует
            self.redis_client.xgroup_create(stream_name, consumer_group, '0', mkstream=True)
        except redis.ResponseError:
            pass  # Группа уже существует
        
        # Чтение новых событий
        events = self.redis_client.xreadgroup(
            consumer_group,
            consumer_name,
            {stream_name: '>'},
            count=count,
            block=1000  # Блокировка на 1 секунду
        )
        
        processed_events = []
        for stream, messages in events:
            for event_id, fields in messages:
                processed_events.append({
                    'id': event_id,
                    'stream': stream,
                    'data': fields
                })
        
        return processed_events
    
    def acknowledge_event(self, stream_name: str, consumer_group: str, event_id: str):
        """Подтверждение обработки события"""
        self.redis_client.xack(stream_name, consumer_group, event_id)
    
    def get_pending_events(self, stream_name: str, consumer_group: str) -> List[Dict]:
        """Получение необработанных событий"""
        pending = self.redis_client.xpending_range(
            stream_name, consumer_group, '-', '+', count=100
        )
        
        return [
            {
                'id': event[0],
                'consumer': event[1],
                'idle_time': event[2],
                'delivery_count': event[3]
            }
            for event in pending
        ]

# Использование для обработки заказов
class OrderEventProcessor:
    def __init__(self):
        self.stream_service = RedisStreamService()
        self.stream_name = "order_events"
        self.consumer_group = "order_processors"
        self.consumer_name = f"processor_{time.time()}"
    
    def publish_order_created(self, order_data: Dict[str, Any]):
        """Публикация события создания заказа"""
        event_data = {
            'event_type': 'order_created',
            'order_id': order_data['id'],
            'user_id': order_data['user_id'],
            'amount': str(order_data['amount']),
            'timestamp': str(time.time()),
            'data': json.dumps(order_data)
        }
        
        event_id = self.stream_service.add_event(self.stream_name, event_data)
        print(f"Order event published: {event_id}")
    
    def process_order_events(self):
        """Обработка событий заказов"""
        while True:
            events = self.stream_service.read_events(
                self.stream_name,
                self.consumer_group,
                self.consumer_name
            )
            
            for event in events:
                try:
                    event_type = event['data']['event_type']
                    
                    if event_type == 'order_created':
                        self._handle_order_created(event)
                    elif event_type == 'order_cancelled':
                        self._handle_order_cancelled(event)
                    
                    # Подтверждение обработки
                    self.stream_service.acknowledge_event(
                        self.stream_name,
                        self.consumer_group,
                        event['id']
                    )
                    
                except Exception as e:
                    print(f"Error processing event {event['id']}: {e}")
    
    def _handle_order_created(self, event: Dict[str, Any]):
        """Обработка создания заказа"""
        order_data = json.loads(event['data']['data'])
        print(f"Processing order creation: {order_data['id']}")
        
        # Отправка email
        # Обновление инвентаря
        # Создание записи для доставки
```

---

## 🌐 Redis Cluster

```python
# services/redis-cluster/cluster_service.py
from rediscluster import RedisCluster
from typing import List, Dict, Any, Optional

class RedisClusterService:
    def __init__(self, startup_nodes: List[Dict[str, Any]]):
        """
        startup_nodes = [
            {"host": "127.0.0.1", "port": "7000"},
            {"host": "127.0.0.1", "port": "7001"},
            {"host": "127.0.0.1", "port": "7002"}
        ]
        """
        self.cluster = RedisCluster(
            startup_nodes=startup_nodes,
            decode_responses=True,
            skip_full_coverage_check=True
        )
    
    def distributed_set(self, key: str, value: str, ttl: int = None):
        """Распределенная запись"""
        if ttl:
            self.cluster.setex(key, ttl, value)
        else:
            self.cluster.set(key, value)
    
    def distributed_get(self, key: str) -> Optional[str]:
        """Распределенное чтение"""
        return self.cluster.get(key)
    
    def get_cluster_info(self) -> Dict[str, Any]:
        """Информация о кластере"""
        nodes = self.cluster.cluster_nodes()
        return {
            'nodes': len(nodes),
            'masters': len([n for n in nodes.values() if 'master' in n['flags']]),
            'slaves': len([n for n in nodes.values() if 'slave' in n['flags']])
        }
```

---

## 🔧 Lua скриптинг в Redis

```python
# services/redis-scripts/lua_scripts.py
class RedisLuaScripts:
    def __init__(self, redis_client):
        self.redis_client = redis_client
        
        # Атомарное увеличение с лимитом
        self.increment_with_limit = self.redis_client.register_script("""
            local key = KEYS[1]
            local limit = tonumber(ARGV[1])
            local current = redis.call('GET', key)
            
            if current == false then
                current = 0
            else
                current = tonumber(current)
            end
            
            if current < limit then
                redis.call('INCR', key)
                return current + 1
            else
                return -1
            end
        """)
        
        # Распределенная блокировка с обновлением TTL
        self.extend_lock = self.redis_client.register_script("""
            local key = KEYS[1]
            local identifier = ARGV[1]
            local ttl = tonumber(ARGV[2])
            
            if redis.call('GET', key) == identifier then
                redis.call('EXPIRE', key, ttl)
                return 1
            else
                return 0
            end
        """)
        
        # Атомарное обновление счетчиков
        self.update_counters = self.redis_client.register_script("""
            local counters = cjson.decode(ARGV[1])
            local results = {}
            
            for key, increment in pairs(counters) do
                local new_value = redis.call('INCRBY', key, increment)
                results[key] = new_value
            end
            
            return cjson.encode(results)
        """)
    
    def rate_limit(self, user_id: str, limit: int = 100) -> bool:
        """Ограничение частоты запросов"""
        key = f"rate_limit:{user_id}"
        result = self.increment_with_limit([key], [limit])
        
        if result == -1:
            return False  # Лимит превышен
        
        # Установка TTL на ключ если это первый запрос
        if result == 1:
            self.redis_client.expire(key, 3600)  # 1 час
        
        return True
    
    def batch_update_counters(self, counter_updates: Dict[str, int]) -> Dict[str, int]:
        """Пакетное обновление счетчиков"""
        import json
        result = self.update_counters([], [json.dumps(counter_updates)])
        return json.loads(result)

# Использование
lua_service = RedisLuaScripts(redis_client)

# Проверка лимита запросов
if lua_service.rate_limit("user123", 1000):
    print("Request allowed")
else:
    print("Rate limit exceeded")

# Пакетное обновление
updates = {
    "page_views:home": 1,
    "user_actions:user123": 1,
    "daily_stats:2024-01-15": 1
}
results = lua_service.batch_update_counters(updates)
print(f"Updated counters: {results}")
```

---

## 📊 Redis Modules - TimeSeries

```python
# services/redis-modules/timeseries_service.py
import redis
import time
from typing import List, Tuple, Dict, Any

class RedisTimeSeriesService:
    def __init__(self, host: str = "localhost", port: int = 6379):
        """Требует Redis с модулем RedisTimeSeries"""
        self.redis_client = redis.Redis(host=host, port=port, decode_responses=True)
    
    def create_timeseries(self, key: str, labels: Dict[str, str] = None):
        """Создание временного ряда"""
        args = [key]
        if labels:
            for label, value in labels.items():
                args.extend(['LABELS', label, value])
        
        self.redis_client.execute_command('TS.CREATE', *args)
    
    def add_sample(self, key: str, timestamp: int, value: float):
        """Добавление значения"""
        self.redis_client.execute_command('TS.ADD', key, timestamp, value)
    
    def get_range(self, key: str, from_time: int, to_time: int) -> List[Tuple[int, float]]:
        """Получение диапазона значений"""
        result = self.redis_client.execute_command('TS.RANGE', key, from_time, to_time)
        return [(int(ts), float(val)) for ts, val in result]
    
    def get_aggregated_range(self, key: str, from_time: int, to_time: int, 
                           aggregation: str, bucket_duration: int) -> List[Tuple[int, float]]:
        """Получение агрегированных данных"""
        result = self.redis_client.execute_command(
            'TS.RANGE', key, from_time, to_time,
            'AGGREGATION', aggregation, bucket_duration
        )
        return [(int(ts), float(val)) for ts, val in result]

# Использование для метрик
class MetricsCollector:
    def __init__(self):
        self.ts_service = RedisTimeSeriesService()
        
        # Создание временных рядов для метрик
        try:
            self.ts_service.create_timeseries(
                'metrics:cpu_usage',
                {'host': 'web01', 'metric': 'cpu'}
            )
            self.ts_service.create_timeseries(
                'metrics:memory_usage',
                {'host': 'web01', 'metric': 'memory'}
            )
            self.ts_service.create_timeseries(
                'metrics:request_count',
                {'service': 'api', 'metric': 'requests'}
            )
        except:
            pass  # Уже существуют
    
    def record_cpu_usage(self, usage_percent: float):
        """Запись использования CPU"""
        timestamp = int(time.time() * 1000)
        self.ts_service.add_sample('metrics:cpu_usage', timestamp, usage_percent)
    
    def record_request(self):
        """Запись HTTP запроса"""
        timestamp = int(time.time() * 1000)
        self.ts_service.add_sample('metrics:request_count', timestamp, 1)
    
    def get_cpu_stats(self, hours: int = 1) -> List[Tuple[int, float]]:
        """Получение статистики CPU за последний час"""
        now = int(time.time() * 1000)
        from_time = now - (hours * 3600 * 1000)
        
        return self.ts_service.get_aggregated_range(
            'metrics:cpu_usage',
            from_time,
            now,
            'avg',
            60000  # 1 минута
        )
```

---

## 🔄 Паттерны использования Redis

### 💾 Кэширование с fallback
```python
# services/order-service/patterns/cache_fallback.py
async def get_user_with_cache(user_id: str) -> Dict[str, Any]:
    """Получение пользователя с кэшированием"""
    # Попытка получить из кэша
    cached_user = redis_service.get_cached_user(user_id)
    if cached_user:
        return cached_user
    
    # Если нет в кэше, получаем из БД
    user = await database.get_user(user_id)
    if user:
        # Кэшируем на 1 час
        redis_service.cache_user(user_id, user, ttl=3600)
    
    return user
```

### 🔒 Распределенная блокировка
```python
# services/order-service/patterns/distributed_lock.py
def process_order_with_lock(order_id: str):
    """Обработка заказа с блокировкой"""
    lock_name = f"order_processing:{order_id}"
    
    if redis_service.acquire_lock(lock_name, ttl=300):  # 5 минут
        try:
            # Обработка заказа
            process_order_logic(order_id)
        finally:
            redis_service.release_lock(lock_name)
    else:
        raise Exception("Order is already being processed")
```

### 📡 Pub/Sub для событий
```python
# services/order-service/patterns/pubsub_events.py
def handle_order_created(order_data: Dict[str, Any]):
    """Обработка создания заказа"""
    # Публикуем событие
    redis_service.publish_event("order_created", {
        "order_id": order_data["id"],
        "user_id": order_data["user_id"],
        "amount": order_data["total_amount"],
        "timestamp": datetime.now().isoformat()
    })

def subscribe_to_order_events():
    """Подписка на события заказов"""
    pubsub = redis_service.subscribe_to_channel("order_created")
    
    for message in pubsub.listen():
        if message["type"] == "message":
            event_data = json.loads(message["data"])
            handle_order_event(event_data)
```

### 🚦 Rate Limiting
```python
# services/api/middleware/rate_limiting.py
def check_api_rate_limit(user_id: str, endpoint: str) -> bool:
    """Проверка лимитов API"""
    # Лимит на минуту
    minute_key = f"rate_limit:{user_id}:{endpoint}:minute"
    minute_count = redis_service.increment_counter(minute_key, ttl=60)
    
    # Лимит на час
    hour_key = f"rate_limit:{user_id}:{endpoint}:hour"
    hour_count = redis_service.increment_counter(hour_key, ttl=3600)
    
    return minute_count <= 100 and hour_count <= 1000
```

---

## 🎯 Практические задания

### 🔰 Начальный уровень
1. Реализуйте систему кеширования пользователей
2. Создайте простую очередь задач с Redis Lists
3. Реализуйте счетчик посещений сайта

### 🚀 Средний уровень
1. Создайте систему сессий с автоматическим продлением
2. Реализуйте лидерборд для игры
3. Создайте систему rate limiting для API

### 💎 Продвинутый уровень
1. Реализуйте распределенную блокировку с Lua скриптами
2. Создайте систему real-time аналитики с Redis Streams
3. Реализуйте систему уведомлений с Pub/Sub

---

## 🌟 Лучшие практики

### ✅ Рекомендации
- **Используйте правильные структуры данных** - выбирайте оптимальный тип для задачи
- **Устанавливайте TTL для временных данных** - предотвращает переполнение памяти
- **Группируйте связанные ключи** - используйте префиксы для организации
- **Мониторьте использование памяти** - Redis хранит все в RAM

### ❌ Чего избегать
- **Больших значений в памяти** - Redis не для больших файлов
- **Блокирующих операций в production** - используйте таймауты
- **Отсутствия персистентности** - настройте RDB/AOF для важных данных
- **Игнорирования безопасности** - используйте пароли и изоляцию сети

---

## 📈 Мониторинг и производительность

### 📊 Ключевые метрики
```python
# Мониторинг Redis
def get_redis_metrics(redis_client):
    info = redis_client.info()
    
    return {
        'memory_usage': info['used_memory_human'],
        'memory_peak': info['used_memory_peak_human'],
        'connected_clients': info['connected_clients'],
        'operations_per_sec': info['instantaneous_ops_per_sec'],
        'keyspace_hits': info['keyspace_hits'],
        'keyspace_misses': info['keyspace_misses'],
        'hit_rate': info['keyspace_hits'] / (info['keyspace_hits'] + info['keyspace_misses'])
    }
```

### 🔍 Анализ производительности
```redis
# Медленные запросы
SLOWLOG GET 10

# Информация о команде
INFO commandstats

# Анализ памяти
MEMORY USAGE key_name
MEMORY STATS
```

---

## 🔗 Связанные темы

- 🐘 **[[postgresql|PostgreSQL]]** - для структурированных данных
- 🍃 **[[mongodb|MongoDB]]** - для документов и гибких схем
- 🏗️ **[[database-design|Проектирование БД]]** - принципы выбора БД
- ⚡ **[[sql-optimization|Оптимизация БД]]** - общие принципы производительности

**[[README|🏠 Вернуться к обзору баз данных]]** 