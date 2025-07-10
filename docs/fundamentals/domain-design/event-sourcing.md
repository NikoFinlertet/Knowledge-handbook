# Event Sourcing

**Event Sourcing** — это архитектурный паттерн, при котором состояние системы сохраняется как последовательность событий, а не как снимок текущего состояния. Вместо обновления записей в базе данных, система сохраняет события, которые описывают изменения состояния.

## Основные концепции

### Традиционный подход vs Event Sourcing

**Традиционный подход:**
```
┌─────────────────────┐
│      Database       │
├─────────────────────┤
│ users               │
│ ┌─────────────────┐ │
│ │ id | name | ... │ │
│ │ 1  | John | ... │ │ ← Текущее состояние
│ │ 2  | Jane | ... │ │
│ └─────────────────┘ │
└─────────────────────┘
```

**Event Sourcing:**
```
┌─────────────────────┐
│   Event Store       │
├─────────────────────┤
│ events              │
│ ┌─────────────────┐ │
│ │ UserCreated     │ │ ← Событие 1
│ │ UserUpdated     │ │ ← Событие 2
│ │ UserDeleted     │ │ ← Событие 3
│ └─────────────────┘ │
└─────────────────────┘
         │
         ▼ (replay)
┌─────────────────────┐
│  Current State      │
│ ┌─────────────────┐ │
│ │ id | name | ... │ │
│ │ 1  | John | ... │ │
│ │ 2  | Jane | ... │ │
│ └─────────────────┘ │
└─────────────────────┘
```

### Событие (Event)

**Событие** — это неизменяемая запись о том, что произошло в системе в определенный момент времени.

**Характеристики события:**
- **Неизменяемость** — событие нельзя изменить после создания
- **Временная метка** — указывает, когда произошло событие
- **Уникальность** — каждое событие имеет уникальный идентификатор
- **Причинно-следственная связь** — события связаны последовательностью

**Применение в проекте:**
```python
# services/order-service/domain/events.py
from dataclasses import dataclass
from datetime import datetime
from typing import List, Dict, Any

@dataclass
class Event:
    event_id: str
    aggregate_id: str
    event_type: str
    event_data: Dict[str, Any]
    timestamp: datetime
    version: int

class OrderCreatedEvent(Event):
    def __init__(self, order_id: str, user_id: str, timestamp: datetime, version: int):
        super().__init__(
            event_id=str(uuid.uuid4()),
            aggregate_id=order_id,
            event_type="OrderCreated",
            event_data={
                "order_id": order_id,
                "user_id": user_id,
                "status": "pending",
                "total_amount": 0.0
            },
            timestamp=timestamp,
            version=version
        )

class ItemAddedToOrderEvent(Event):
    def __init__(self, order_id: str, product_id: str, quantity: int, price: float, timestamp: datetime, version: int):
        super().__init__(
            event_id=str(uuid.uuid4()),
            aggregate_id=order_id,
            event_type="ItemAddedToOrder",
            event_data={
                "order_id": order_id,
                "product_id": product_id,
                "quantity": quantity,
                "price": price
            },
            timestamp=timestamp,
            version=version
        )

class OrderConfirmedEvent(Event):
    def __init__(self, order_id: str, total_amount: float, timestamp: datetime, version: int):
        super().__init__(
            event_id=str(uuid.uuid4()),
            aggregate_id=order_id,
            event_type="OrderConfirmed",
            event_data={
                "order_id": order_id,
                "total_amount": total_amount
            },
            timestamp=timestamp,
            version=version
        )
```

### Агрегат (Aggregate)

**Агрегат** в Event Sourcing — это объект, который генерирует события и может восстанавливать свое состояние из последовательности событий.

**Применение в проекте:**
```python
# services/order-service/domain/order.py
from datetime import datetime
from typing import List
from .events import Event, OrderCreatedEvent, ItemAddedToOrderEvent, OrderConfirmedEvent

class OrderStatus:
    PENDING = "pending"
    CONFIRMED = "confirmed"
    SHIPPED = "shipped"
    DELIVERED = "delivered"
    CANCELLED = "cancelled"

class OrderItem:
    def __init__(self, product_id: str, quantity: int, price: float):
        if quantity <= 0:
            raise ValueError("Quantity must be positive")
        if price < 0:
            raise ValueError("Price cannot be negative")
        
        self.product_id = product_id
        self.quantity = quantity
        self.price = price
    
    def total_price(self) -> float:
        return self.quantity * self.price

class Order:
    def __init__(self, order_id: str, user_id: str):
        self.order_id = order_id
        self.user_id = user_id
        self.items: List[OrderItem] = []
        self.status = OrderStatus.PENDING
        self.total_amount = 0.0
        self.created_at = datetime.utcnow()
        self.updated_at = datetime.utcnow()
        self.version = 0
        
        # Список неподтвержденных событий для сохранения
        self.uncommitted_events: List[Event] = []
        
        # Генерация события создания заказа
        event = OrderCreatedEvent(
            order_id=self.order_id,
            user_id=self.user_id,
            timestamp=self.created_at,
            version=self.version
        )
        self.uncommitted_events.append(event)
    
    def add_item(self, product_id: str, quantity: int, price: float):
        """Добавить товар в заказ"""
        if self.status != OrderStatus.PENDING:
            raise ValueError("Cannot add items to non-pending order")
        
        item = OrderItem(product_id, quantity, price)
        self.items.append(item)
        self.total_amount += item.total_price()
        self.updated_at = datetime.utcnow()
        self.version += 1
        
        # Генерация события добавления товара
        event = ItemAddedToOrderEvent(
            order_id=self.order_id,
            product_id=product_id,
            quantity=quantity,
            price=price,
            timestamp=self.updated_at,
            version=self.version
        )
        self.uncommitted_events.append(event)
    
    def confirm(self):
        """Подтвердить заказ"""
        if not self.items:
            raise ValueError("Cannot confirm empty order")
        if self.status != OrderStatus.PENDING:
            raise ValueError("Order is not in pending status")
        
        self.status = OrderStatus.CONFIRMED
        self.updated_at = datetime.utcnow()
        self.version += 1
        
        # Генерация события подтверждения заказа
        event = OrderConfirmedEvent(
            order_id=self.order_id,
            total_amount=self.total_amount,
            timestamp=self.updated_at,
            version=self.version
        )
        self.uncommitted_events.append(event)
    
    def get_uncommitted_events(self) -> List[Event]:
        """Получить неподтвержденные события"""
        return self.uncommitted_events
    
    def mark_events_as_committed(self):
        """Отметить события как подтвержденные"""
        self.uncommitted_events.clear()
    
    @staticmethod
    def from_events(events: List[Event]) -> 'Order':
        """Восстановить состояние заказа из событий"""
        if not events:
            raise ValueError("Cannot create order from empty events")
        
        # Сортировка событий по версии
        events.sort(key=lambda e: e.version)
        
        # Создание заказа из первого события
        first_event = events[0]
        if first_event.event_type != "OrderCreated":
            raise ValueError("First event must be OrderCreated")
        
        order = Order.__new__(Order)  # Создаем без вызова __init__
        order.order_id = first_event.event_data["order_id"]
        order.user_id = first_event.event_data["user_id"]
        order.items = []
        order.status = OrderStatus.PENDING
        order.total_amount = 0.0
        order.created_at = first_event.timestamp
        order.updated_at = first_event.timestamp
        order.version = first_event.version
        order.uncommitted_events = []
        
        # Применение остальных событий
        for event in events[1:]:
            order._apply_event(event)
        
        return order
    
    def _apply_event(self, event: Event):
        """Применить событие к текущему состоянию"""
        if event.event_type == "ItemAddedToOrder":
            data = event.event_data
            item = OrderItem(
                product_id=data["product_id"],
                quantity=data["quantity"],
                price=data["price"]
            )
            self.items.append(item)
            self.total_amount += item.total_price()
            self.updated_at = event.timestamp
            self.version = event.version
            
        elif event.event_type == "OrderConfirmed":
            self.status = OrderStatus.CONFIRMED
            self.updated_at = event.timestamp
            self.version = event.version
            
        elif event.event_type == "OrderCancelled":
            self.status = OrderStatus.CANCELLED
            self.updated_at = event.timestamp
            self.version = event.version
        
        # Добавление обработки других типов событий...
```

### Event Store

**Event Store** — это специализированная база данных для хранения событий.

**Применение в проекте:**
```python
# services/order-service/infrastructure/event_store.py
import asyncpg
from typing import List, Optional
from datetime import datetime
from ..domain.events import Event
from ..domain.order import Order

class EventStore:
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
        self.pool = None
    
    async def initialize(self):
        """Инициализация пула соединений"""
        self.pool = await asyncpg.create_pool(self.connection_string)
        
        # Создание таблицы для событий
        async with self.pool.acquire() as conn:
            await conn.execute("""
                CREATE TABLE IF NOT EXISTS events (
                    event_id VARCHAR PRIMARY KEY,
                    aggregate_id VARCHAR NOT NULL,
                    event_type VARCHAR NOT NULL,
                    event_data JSONB NOT NULL,
                    timestamp TIMESTAMP NOT NULL,
                    version INTEGER NOT NULL,
                    UNIQUE(aggregate_id, version)
                )
            """)
            
            await conn.execute("""
                CREATE INDEX IF NOT EXISTS idx_events_aggregate_id 
                ON events(aggregate_id)
            """)
            
            await conn.execute("""
                CREATE INDEX IF NOT EXISTS idx_events_timestamp 
                ON events(timestamp)
            """)
    
    async def save_events(self, aggregate_id: str, events: List[Event], expected_version: int):
        """Сохранить события в store"""
        if not events:
            return
        
        async with self.pool.acquire() as conn:
            async with conn.transaction():
                # Проверка версии для optimistic locking
                current_version = await conn.fetchval(
                    "SELECT COALESCE(MAX(version), -1) FROM events WHERE aggregate_id = $1",
                    aggregate_id
                )
                
                if current_version != expected_version:
                    raise ConcurrencyException(
                        f"Concurrency conflict. Expected version {expected_version}, "
                        f"but current version is {current_version}"
                    )
                
                # Сохранение событий
                for event in events:
                    await conn.execute("""
                        INSERT INTO events (event_id, aggregate_id, event_type, event_data, timestamp, version)
                        VALUES ($1, $2, $3, $4, $5, $6)
                    """, 
                    event.event_id,
                    event.aggregate_id,
                    event.event_type,
                    event.event_data,
                    event.timestamp,
                    event.version
                    )
    
    async def get_events(self, aggregate_id: str, from_version: int = 0) -> List[Event]:
        """Получить события для агрегата"""
        async with self.pool.acquire() as conn:
            rows = await conn.fetch("""
                SELECT event_id, aggregate_id, event_type, event_data, timestamp, version
                FROM events 
                WHERE aggregate_id = $1 AND version >= $2
                ORDER BY version ASC
            """, aggregate_id, from_version)
            
            return [
                Event(
                    event_id=row["event_id"],
                    aggregate_id=row["aggregate_id"],
                    event_type=row["event_type"],
                    event_data=row["event_data"],
                    timestamp=row["timestamp"],
                    version=row["version"]
                ) for row in rows
            ]
    
    async def get_events_by_type(self, event_type: str, from_timestamp: datetime = None) -> List[Event]:
        """Получить события по типу"""
        async with self.pool.acquire() as conn:
            if from_timestamp:
                rows = await conn.fetch("""
                    SELECT event_id, aggregate_id, event_type, event_data, timestamp, version
                    FROM events 
                    WHERE event_type = $1 AND timestamp >= $2
                    ORDER BY timestamp ASC
                """, event_type, from_timestamp)
            else:
                rows = await conn.fetch("""
                    SELECT event_id, aggregate_id, event_type, event_data, timestamp, version
                    FROM events 
                    WHERE event_type = $1
                    ORDER BY timestamp ASC
                """, event_type)
            
            return [
                Event(
                    event_id=row["event_id"],
                    aggregate_id=row["aggregate_id"],
                    event_type=row["event_type"],
                    event_data=row["event_data"],
                    timestamp=row["timestamp"],
                    version=row["version"]
                ) for row in rows
            ]
    
    async def close(self):
        """Закрыть пул соединений"""
        if self.pool:
            await self.pool.close()

class ConcurrencyException(Exception):
    """Исключение при конфликте версий"""
    pass
```

## Ключевые концепции

### 1. Снимки (Snapshots)

**Снимок** — это сохранение текущего состояния агрегата для оптимизации восстановления.

```python
# services/order-service/infrastructure/event_store.py
class SnapshotStore:
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
        self.pool = None
    
    async def initialize(self):
        self.pool = await asyncpg.create_pool(self.connection_string)
        
        async with self.pool.acquire() as conn:
            await conn.execute("""
                CREATE TABLE IF NOT EXISTS snapshots (
                    aggregate_id VARCHAR PRIMARY KEY,
                    snapshot_data JSONB NOT NULL,
                    version INTEGER NOT NULL,
                    timestamp TIMESTAMP NOT NULL
                )
            """)
    
    async def save_snapshot(self, aggregate_id: str, snapshot_data: dict, version: int):
        """Сохранить снимок"""
        async with self.pool.acquire() as conn:
            await conn.execute("""
                INSERT INTO snapshots (aggregate_id, snapshot_data, version, timestamp)
                VALUES ($1, $2, $3, $4)
                ON CONFLICT (aggregate_id) DO UPDATE SET
                    snapshot_data = $2,
                    version = $3,
                    timestamp = $4
            """, aggregate_id, snapshot_data, version, datetime.utcnow())
    
    async def get_snapshot(self, aggregate_id: str) -> Optional[dict]:
        """Получить снимок"""
        async with self.pool.acquire() as conn:
            row = await conn.fetchrow("""
                SELECT snapshot_data, version, timestamp
                FROM snapshots 
                WHERE aggregate_id = $1
            """, aggregate_id)
            
            return dict(row) if row else None

# Оптимизированное восстановление с использованием снимков
class OptimizedOrderRepository:
    def __init__(self, event_store: EventStore, snapshot_store: SnapshotStore):
        self.event_store = event_store
        self.snapshot_store = snapshot_store
    
    async def get_order(self, order_id: str) -> Optional[Order]:
        """Получить заказ с использованием снимка"""
        # Попытка получить снимок
        snapshot = await self.snapshot_store.get_snapshot(order_id)
        
        if snapshot:
            # Восстановление из снимка
            order = Order.from_snapshot(snapshot["snapshot_data"])
            # Получение событий после снимка
            events = await self.event_store.get_events(order_id, snapshot["version"] + 1)
            # Применение событий
            for event in events:
                order._apply_event(event)
            return order
        else:
            # Полное восстановление из событий
            events = await self.event_store.get_events(order_id)
            return Order.from_events(events) if events else None
    
    async def save_order(self, order: Order):
        """Сохранить заказ"""
        events = order.get_uncommitted_events()
        expected_version = order.version - len(events)
        
        await self.event_store.save_events(order.order_id, events, expected_version)
        order.mark_events_as_committed()
        
        # Сохранение снимка каждые 10 событий
        if order.version % 10 == 0:
            snapshot_data = order.to_snapshot()
            await self.snapshot_store.save_snapshot(order.order_id, snapshot_data, order.version)
```

### 2. Проекции (Projections)

**Проекция** — это материализованное представление данных, построенное на основе событий.

```python
# services/order-service/infrastructure/projections.py
class OrderProjection:
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
        self.pool = None
    
    async def initialize(self):
        self.pool = await asyncpg.create_pool(self.connection_string)
        
        async with self.pool.acquire() as conn:
            await conn.execute("""
                CREATE TABLE IF NOT EXISTS order_projections (
                    order_id VARCHAR PRIMARY KEY,
                    user_id VARCHAR NOT NULL,
                    status VARCHAR NOT NULL,
                    total_amount DECIMAL NOT NULL,
                    item_count INTEGER NOT NULL,
                    created_at TIMESTAMP NOT NULL,
                    updated_at TIMESTAMP NOT NULL,
                    version INTEGER NOT NULL
                )
            """)
    
    async def handle_event(self, event: Event):
        """Обработка события для обновления проекции"""
        async with self.pool.acquire() as conn:
            if event.event_type == "OrderCreated":
                await conn.execute("""
                    INSERT INTO order_projections 
                    (order_id, user_id, status, total_amount, item_count, created_at, updated_at, version)
                    VALUES ($1, $2, $3, $4, $5, $6, $7, $8)
                """, 
                event.event_data["order_id"],
                event.event_data["user_id"],
                event.event_data["status"],
                event.event_data["total_amount"],
                0,
                event.timestamp,
                event.timestamp,
                event.version
                )
                
            elif event.event_type == "ItemAddedToOrder":
                await conn.execute("""
                    UPDATE order_projections SET
                        total_amount = total_amount + ($1 * $2),
                        item_count = item_count + 1,
                        updated_at = $3,
                        version = $4
                    WHERE order_id = $5
                """,
                event.event_data["price"],
                event.event_data["quantity"],
                event.timestamp,
                event.version,
                event.event_data["order_id"]
                )
                
            elif event.event_type == "OrderConfirmed":
                await conn.execute("""
                    UPDATE order_projections SET
                        status = 'confirmed',
                        updated_at = $1,
                        version = $2
                    WHERE order_id = $3
                """,
                event.timestamp,
                event.version,
                event.event_data["order_id"]
                )
    
    async def get_order_summary(self, order_id: str) -> Optional[dict]:
        """Получить сводку по заказу"""
        async with self.pool.acquire() as conn:
            row = await conn.fetchrow("""
                SELECT * FROM order_projections WHERE order_id = $1
            """, order_id)
            
            return dict(row) if row else None
    
    async def get_user_orders(self, user_id: str) -> List[dict]:
        """Получить заказы пользователя"""
        async with self.pool.acquire() as conn:
            rows = await conn.fetch("""
                SELECT * FROM order_projections 
                WHERE user_id = $1 
                ORDER BY created_at DESC
            """, user_id)
            
            return [dict(row) for row in rows]
```

### 3. Обработчики событий (Event Handlers)

**Обработчик событий** — это компонент, который реагирует на события и выполняет побочные эффекты.

```python
# services/order-service/main.py
from fastapi import FastAPI, HTTPException
from .infrastructure.event_store import EventStore
from .infrastructure.projections import OrderProjection
from .domain.order import Order

app = FastAPI()

# Инициализация компонентов
event_store = EventStore("postgresql://user:pass@localhost/events")
order_projection = OrderProjection("postgresql://user:pass@localhost/projections")

@app.on_event("startup")
async def startup():
    await event_store.initialize()
    await order_projection.initialize()

class OrderService:
    def __init__(self, event_store: EventStore):
        self.event_store = event_store
    
    async def create_order(self, user_id: str) -> str:
        order = Order(str(uuid.uuid4()), user_id)
        
        events = order.get_uncommitted_events()
        await self.event_store.save_events(order.order_id, events, -1)
        order.mark_events_as_committed()
        
        # Обработка событий для проекций
        for event in events:
            await order_projection.handle_event(event)
        
        return order.order_id
    
    async def add_item_to_order(self, order_id: str, product_id: str, quantity: int, price: float):
        # Восстановление заказа из событий
        events = await self.event_store.get_events(order_id)
        if not events:
            raise HTTPException(status_code=404, detail="Order not found")
        
        order = Order.from_events(events)
        
        # Выполнение бизнес-операции
        order.add_item(product_id, quantity, price)
        
        # Сохранение новых событий
        new_events = order.get_uncommitted_events()
        await self.event_store.save_events(order_id, new_events, order.version - len(new_events))
        order.mark_events_as_committed()
        
        # Обновление проекций
        for event in new_events:
            await order_projection.handle_event(event)
    
    async def confirm_order(self, order_id: str):
        events = await self.event_store.get_events(order_id)
        if not events:
            raise HTTPException(status_code=404, detail="Order not found")
        
        order = Order.from_events(events)
        order.confirm()
        
        new_events = order.get_uncommitted_events()
        await self.event_store.save_events(order_id, new_events, order.version - len(new_events))
        order.mark_events_as_committed()
        
        for event in new_events:
            await order_projection.handle_event(event)

order_service = OrderService(event_store)

# API endpoints
@app.post("/orders")
async def create_order(user_id: str):
    order_id = await order_service.create_order(user_id)
    return {"order_id": order_id}

@app.post("/orders/{order_id}/items")
async def add_item(order_id: str, product_id: str, quantity: int, price: float):
    await order_service.add_item_to_order(order_id, product_id, quantity, price)
    return {"message": "Item added successfully"}

@app.post("/orders/{order_id}/confirm")
async def confirm_order(order_id: str):
    await order_service.confirm_order(order_id)
    return {"message": "Order confirmed"}

@app.get("/orders/{order_id}")
async def get_order(order_id: str):
    order_data = await order_projection.get_order_summary(order_id)
    if not order_data:
        raise HTTPException(status_code=404, detail="Order not found")
    return order_data

@app.get("/users/{user_id}/orders")
async def get_user_orders(user_id: str):
    orders = await order_projection.get_user_orders(user_id)
    return {"orders": orders}
```

## Преимущества Event Sourcing

### 1. Полная история изменений
- Все изменения сохраняются навсегда
- Возможность воспроизведения любого момента времени
- Аудит всех операций

### 2. Временные запросы
- Возможность узнать состояние в любой момент времени
- Анализ трендов и паттернов
- Отладка и диагностика

### 3. Высокая производительность записи
- Только добавление новых записей (append-only)
- Отсутствие блокировок на чтение
- Масштабируемость записи

### 4. Естественная совместимость с CQRS
- Команды генерируют события
- Запросы используют проекции
- Оптимизация для разных сценариев

### 5. Устойчивость к сбоям
- События неизменяемы
- Возможность восстановления из событий
- Репликация через воспроизведение

## Недостатки Event Sourcing

### 1. Сложность
- Высокая кривая обучения
- Сложность отладки
- Необходимость понимания событийной модели

### 2. Размер данных
- Рост объема данных со временем
- Необходимость архивации старых событий
- Сложность оптимизации хранения

### 3. Eventual Consistency
- Проекции могут отставать от событий
- Сложность обеспечения согласованности
- Необходимость обработки конфликтов

### 4. Миграция схемы
- Сложность изменения структуры событий
- Необходимость поддержки версий
- Миграция существующих данных

## Паттерны и практики

### 1. Версионирование событий
```python
class EventV1:
    def __init__(self, user_id: str, name: str):
        self.version = 1
        self.user_id = user_id
        self.name = name

class EventV2:
    def __init__(self, user_id: str, first_name: str, last_name: str):
        self.version = 2
        self.user_id = user_id
        self.first_name = first_name
        self.last_name = last_name
    
    @classmethod
    def from_v1(cls, event_v1: EventV1):
        # Миграция из версии 1 в версию 2
        names = event_v1.name.split(" ", 1)
        return cls(
            user_id=event_v1.user_id,
            first_name=names[0],
            last_name=names[1] if len(names) > 1 else ""
        )
```

### 2. Upcasting (Повышение версии)
```python
def upcast_event(event: dict) -> dict:
    """Повышение версии события до актуальной"""
    version = event.get("version", 1)
    
    if version == 1:
        # Повышение с версии 1 до 2
        event["version"] = 2
        names = event["name"].split(" ", 1)
        event["first_name"] = names[0]
        event["last_name"] = names[1] if len(names) > 1 else ""
        del event["name"]
    
    return event
```

### 3. Saga Pattern для долгосрочных процессов
```python
class OrderSaga:
    def __init__(self, order_id: str):
        self.order_id = order_id
        self.state = "started"
        self.events = []
    
    def handle_order_confirmed(self, event: Event):
        if self.state == "started":
            # Запуск следующего шага
            self.state = "payment_processing"
            self.events.append(PaymentRequestedEvent(
                order_id=self.order_id,
                amount=event.event_data["total_amount"]
            ))
    
    def handle_payment_completed(self, event: Event):
        if self.state == "payment_processing":
            self.state = "shipping"
            self.events.append(ShippingRequestedEvent(
                order_id=self.order_id
            ))
    
    def handle_shipping_completed(self, event: Event):
        if self.state == "shipping":
            self.state = "completed"
            self.events.append(OrderCompletedEvent(
                order_id=self.order_id
            ))
```

## Интеграция с другими паттернами

### 1. CQRS + Event Sourcing
```python
# Command Side - генерирует события
class CreateOrderCommand:
    def __init__(self, user_id: str):
        self.user_id = user_id

class CreateOrderHandler:
    def __init__(self, event_store: EventStore):
        self.event_store = event_store
    
    async def handle(self, command: CreateOrderCommand):
        order = Order(str(uuid.uuid4()), command.user_id)
        events = order.get_uncommitted_events()
        await self.event_store.save_events(order.order_id, events, -1)
        return order.order_id

# Query Side - использует проекции
class GetOrderQuery:
    def __init__(self, order_id: str):
        self.order_id = order_id

class GetOrderHandler:
    def __init__(self, projection: OrderProjection):
        self.projection = projection
    
    async def handle(self, query: GetOrderQuery):
        return await self.projection.get_order_summary(query.order_id)
```

### 2. Domain Events в DDD
```python
class User:
    def __init__(self, user_id: str, email: str):
        self.user_id = user_id
        self.email = email
        self.domain_events = []
    
    def change_email(self, new_email: str):
        old_email = self.email
        self.email = new_email
        
        # Генерация доменного события
        event = EmailChangedEvent(
            user_id=self.user_id,
            old_email=old_email,
            new_email=new_email,
            timestamp=datetime.utcnow()
        )
        self.domain_events.append(event)
```

## Когда использовать Event Sourcing

### Подходящие случаи:
1. **Требуется аудит** - все изменения должны быть записаны
2. **Сложная бизнес-логика** - множество состояний и переходов
3. **Временные запросы** - нужно знать состояние в прошлом
4. **High-write системы** - много операций записи
5. **Отладка и диагностика** - важность воспроизведения

### Неподходящие случаи:
1. **Простые CRUD операции** - избыточная сложность
2. **Ограниченные ресурсы** - требует больше места
3. **Низкие требования к аудиту** - нет необходимости в истории
4. **Сильная связанность данных** - сложно разбить на события

## Заключение

Event Sourcing — это мощный паттерн для построения систем с высокими требованиями к аудиту, отладке и масштабируемости. Основные принципы:

1. **События как источник истины** - все изменения сохраняются как события
2. **Неизменяемость событий** - события никогда не удаляются или изменяются
3. **Восстановление состояния** - текущее состояние восстанавливается из событий
4. **Проекции для запросов** - материализованные представления для оптимизации чтения

Event Sourcing особенно эффективен в сочетании с CQRS, DDD и микросервисной архитектурой, обеспечивая полную историю изменений и высокую производительность.

## См. также
- [CQRS Pattern](./cqrs-pattern.md)
- [Domain-Driven Design (DDD)](./ddd-patterns.md)
- [Микросервисная архитектура](./microservices-architecture.md)
- [GoF Design Patterns](./gof-patterns.md) 