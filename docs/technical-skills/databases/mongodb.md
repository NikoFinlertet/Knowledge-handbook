# MongoDB - Документоориентированная NoSQL База Данных 🍃

## 🏠 Навигация
**[[README|🏠 Базы данных]]** → **MongoDB**

**Связанные темы:**
- 🐘 [[postgresql|PostgreSQL]]
- 🔴 [[redis|Redis]]
- 🏗️ [[database-design|Проектирование БД]]
- ⚡ [[sql-optimization|Оптимизация БД]]

---

## 🎯 Что такое MongoDB?

**MongoDB** — это документоориентированная NoSQL база данных, хранящая данные в BSON формате (Binary JSON). Разработана для современных приложений, требующих гибкости схемы и горизонтального масштабирования.

### ✨ Ключевые возможности

1. **🔄 Гибкая схема** - документы могут иметь разную структуру
2. **📈 Масштабируемость** - горизонтальное масштабирование через sharding
3. **🔄 Репликация** - автоматическое резервирование данных
4. **🚀 Индексы** - поддержка сложных индексов
5. **⚙️ Агрегация** - мощный pipeline для обработки данных

---

## 📊 Структура данных

MongoDB хранит данные в виде документов (documents) в коллекциях (collections).

```javascript
// Пример документа в MongoDB
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "email": "john.doe@example.com",
  "name": "John Doe",
  "profile": {
    "age": 30,
    "city": "New York",
    "interests": ["technology", "music", "sports"]
  },
  "orders": [
    {
      "orderId": ObjectId("507f1f77bcf86cd799439012"),
      "amount": 100.50,
      "items": ["laptop", "mouse"],
      "date": ISODate("2024-01-15T10:30:00Z")
    }
  ],
  "metadata": {
    "lastLogin": ISODate("2024-01-15T10:30:00Z"),
    "loginCount": 45,
    "subscription": "premium"
  },
  "createdAt": ISODate("2024-01-01T00:00:00Z"),
  "updatedAt": ISODate("2024-01-15T10:30:00Z")
}
```

---

## 🔧 Операции CRUD

### ✅ CREATE - Создание документов

```javascript
// services/order-service/database/mongodb_operations.js

// Создание одного документа
db.users.insertOne({
  email: "alice@example.com",
  name: "Alice Smith",
  profile: {
    age: 25,
    city: "San Francisco"
  },
  createdAt: new Date()
});

// Массовая вставка
db.users.insertMany([
  {
    email: "bob@example.com",
    name: "Bob Johnson",
    profile: { age: 35, city: "Chicago" }
  },
  {
    email: "carol@example.com",
    name: "Carol Williams",
    profile: { age: 28, city: "Boston" }
  }
]);
```

### 📖 READ - Поиск документов

```javascript
// Простой поиск
db.users.find({ "profile.city": "New York" });

// Поиск с условиями
db.users.find({
  "profile.age": { $gte: 25, $lte: 35 },
  "metadata.subscription": "premium"
});

// Поиск с проекцией (выборка полей)
db.users.find(
  { "profile.city": "New York" },
  { name: 1, email: 1, "profile.age": 1 }
);

// Сложные запросы
db.users.find({
  $and: [
    { "profile.age": { $gte: 18 } },
    { $or: [
        { "profile.city": "New York" },
        { "profile.city": "San Francisco" }
      ]
    }
  ]
});
```

### 🔄 UPDATE - Обновление документов

```javascript
// Обновление одного документа
db.users.updateOne(
  { email: "john.doe@example.com" },
  {
    $set: {
      "profile.city": "Los Angeles",
      "metadata.lastLogin": new Date()
    },
    $inc: { "metadata.loginCount": 1 }
  }
);

// Обновление массива
db.users.updateOne(
  { email: "john.doe@example.com" },
  {
    $push: {
      "orders": {
        orderId: ObjectId(),
        amount: 250.00,
        items: ["keyboard", "monitor"],
        date: new Date()
      }
    }
  }
);

// Массовое обновление
db.users.updateMany(
  { "metadata.subscription": "free" },
  { $set: { "metadata.upgradeOffer": true } }
);
```

### 🗑️ DELETE - Удаление документов

```javascript
// Удаление одного документа
db.users.deleteOne({ email: "inactive@example.com" });

// Массовое удаление
db.users.deleteMany({ 
  "metadata.lastLogin": { $lt: new Date("2023-01-01") } 
});
```

---

## 🚀 Индексы MongoDB

Индексы критически важны для производительности запросов в MongoDB.

```javascript
// services/order-service/database/mongodb_indexes.js

// Простой индекс
db.users.createIndex({ email: 1 });

// Составной индекс
db.users.createIndex({ "profile.city": 1, "profile.age": -1 });

// Текстовый индекс для поиска
db.users.createIndex({
  name: "text",
  email: "text",
  "profile.city": "text"
});

// Частичный индекс
db.users.createIndex(
  { "metadata.subscription": 1 },
  { partialFilterExpression: { "metadata.subscription": { $exists: true } } }
);

// TTL индекс для автоматического удаления
db.sessions.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 3600 } // Удаляем через 1 час
);

// Анализ производительности индексов
db.users.explain("executionStats").find({ "profile.city": "New York" });
```

---

## ⚙️ Агрегация Pipeline

Агрегация - мощный инструмент для обработки и анализа данных.

### 📊 Базовая агрегация

```javascript
// services/order-service/database/mongodb_aggregation.js

// Группировка пользователей по городам
db.users.aggregate([
  {
    $group: {
      _id: "$profile.city",
      totalUsers: { $sum: 1 },
      averageAge: { $avg: "$profile.age" },
      premiumUsers: {
        $sum: {
          $cond: [
            { $eq: ["$metadata.subscription", "premium"] },
            1,
            0
          ]
        }
      }
    }
  },
  {
    $sort: { totalUsers: -1 }
  }
]);
```

### 🔗 Объединение коллекций с $lookup

```javascript
// Lookup для объединения коллекций
db.orders.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "user"
    }
  },
  {
    $unwind: "$user"
  },
  {
    $group: {
      _id: "$user.profile.city",
      totalOrders: { $sum: 1 },
      totalRevenue: { $sum: "$totalAmount" }
    }
  }
]);
```

### 📈 Сложная аналитика

```javascript
// Анализ заказов по пользователям
db.users.aggregate([
  {
    $unwind: "$orders"
  },
  {
    $group: {
      _id: "$_id",
      name: { $first: "$name" },
      email: { $first: "$email" },
      totalOrders: { $sum: 1 },
      totalAmount: { $sum: "$orders.amount" },
      averageOrderValue: { $avg: "$orders.amount" }
    }
  },
  {
    $match: {
      totalOrders: { $gte: 5 }
    }
  },
  {
    $sort: { totalAmount: -1 }
  },
  {
    $limit: 10
  }
]);
```

---

## 🌐 Шардирование и Реплика сеты

### 🔄 Конфигурация реплика сета

```javascript
// Инициализация реплика сета
rs.initiate({
  _id: "myReplicaSet",
  members: [
    { _id: 0, host: "mongo1:27017", priority: 2 },
    { _id: 1, host: "mongo2:27017", priority: 1 },
    { _id: 2, host: "mongo3:27017", priority: 1, arbiterOnly: true }
  ]
});

// Проверка статуса
rs.status();

// Настройка чтения с реплик
db.setReadPref("secondaryPreferred");
```

### 📊 Шардирование

```javascript
// Включение шардирования
sh.enableSharding("ecommerce");

// Создание шардированной коллекции
sh.shardCollection("ecommerce.orders", { "user_id": 1, "created_at": 1 });

// Проверка распределения
sh.status();

// Балансировка шардов
sh.setBalancerState(true);
```

---

## 💡 Транзакции в MongoDB

```python
# services/order-service/infrastructure/mongodb_transactions.py
from pymongo import MongoClient
from pymongo.errors import PyMongoError
import logging
import datetime

class MongoDBTransactionService:
    def __init__(self, connection_string: str):
        self.client = MongoClient(connection_string)
        self.db = self.client.ecommerce
    
    def create_order_with_inventory_update(self, order_data: dict, inventory_updates: list):
        """Создание заказа с обновлением инвентаря в транзакции"""
        with self.client.start_session() as session:
            try:
                with session.start_transaction():
                    # Создание заказа
                    order_result = self.db.orders.insert_one(
                        order_data, session=session
                    )
                    
                    # Обновление инвентаря
                    for update in inventory_updates:
                        result = self.db.inventory.update_one(
                            {"product_id": update["product_id"]},
                            {"$inc": {"quantity": -update["quantity"]}},
                            session=session
                        )
                        
                        # Проверка что товар есть в наличии
                        product = self.db.inventory.find_one(
                            {"product_id": update["product_id"]},
                            session=session
                        )
                        
                        if product["quantity"] < 0:
                            raise ValueError(f"Insufficient inventory for product {update['product_id']}")
                    
                    # Логирование операции
                    self.db.order_logs.insert_one({
                        "order_id": order_result.inserted_id,
                        "action": "order_created",
                        "timestamp": datetime.datetime.utcnow(),
                        "inventory_updates": inventory_updates
                    }, session=session)
                    
                    session.commit_transaction()
                    return order_result.inserted_id
                    
            except PyMongoError as e:
                session.abort_transaction()
                logging.error(f"Transaction failed: {e}")
                raise
```

---

## 📁 GridFS для файлов

```python
# services/file-service/infrastructure/gridfs_service.py
from pymongo import MongoClient
import gridfs
from typing import BinaryIO, Optional

class GridFSService:
    def __init__(self, connection_string: str, database_name: str):
        self.client = MongoClient(connection_string)
        self.db = self.client[database_name]
        self.fs = gridfs.GridFS(self.db)
    
    def store_file(self, file_data: BinaryIO, filename: str, metadata: dict = None) -> str:
        """Сохранение файла в GridFS"""
        file_id = self.fs.put(
            file_data,
            filename=filename,
            metadata=metadata or {}
        )
        return str(file_id)
    
    def get_file(self, file_id: str) -> Optional[gridfs.GridOut]:
        """Получение файла из GridFS"""
        try:
            return self.fs.get(file_id)
        except gridfs.NoFile:
            return None
    
    def delete_file(self, file_id: str) -> bool:
        """Удаление файла из GridFS"""
        try:
            self.fs.delete(file_id)
            return True
        except gridfs.NoFile:
            return False

# Использование
fs_service = GridFSService("mongodb://localhost:27017", "file_storage")

# Сохранение изображения
with open("avatar.jpg", "rb") as f:
    file_id = fs_service.store_file(
        f, 
        "avatar.jpg",
        {"user_id": "user123", "type": "avatar", "size": "large"}
    )
```

---

## 🐍 Python интеграция с PyMongo

```python
# services/order-service/infrastructure/mongodb.py
from pymongo import MongoClient, ASCENDING, DESCENDING
from pymongo.collection import Collection
from pymongo.errors import DuplicateKeyError, ConnectionFailure
from typing import List, Dict, Any, Optional
from bson import ObjectId
import datetime

class MongoDBRepository:
    def __init__(self, connection_string: str, database_name: str):
        self.client = MongoClient(connection_string)
        self.db = self.client[database_name]
        self.users: Collection = self.db.users
        self.orders: Collection = self.db.orders
        self.sessions: Collection = self.db.sessions
        
        # Создание индексов
        self._create_indexes()
    
    def _create_indexes(self):
        """Создание необходимых индексов"""
        # Индексы для users
        self.users.create_index([("email", ASCENDING)], unique=True)
        self.users.create_index([("profile.city", ASCENDING), ("profile.age", DESCENDING)])
        self.users.create_index([("metadata.subscription", ASCENDING)])
        
        # Текстовый индекс для поиска
        self.users.create_index([
            ("name", "text"),
            ("email", "text"),
            ("profile.city", "text")
        ])
        
        # Индексы для orders
        self.orders.create_index([("userId", ASCENDING)])
        self.orders.create_index([("createdAt", DESCENDING)])
        self.orders.create_index([("status", ASCENDING)])
        
        # TTL индекс для sessions
        self.sessions.create_index([("createdAt", ASCENDING)], expireAfterSeconds=3600)
    
    def create_user(self, user_data: Dict[str, Any]) -> str:
        """Создание пользователя"""
        user_data["createdAt"] = datetime.datetime.utcnow()
        user_data["updatedAt"] = datetime.datetime.utcnow()
        
        try:
            result = self.users.insert_one(user_data)
            return str(result.inserted_id)
        except DuplicateKeyError:
            raise ValueError(f"User with email {user_data['email']} already exists")
    
    def find_users_by_city(self, city: str, limit: int = 10) -> List[Dict[str, Any]]:
        """Поиск пользователей по городу"""
        cursor = self.users.find(
            {"profile.city": city},
            {"name": 1, "email": 1, "profile": 1}
        ).limit(limit)
        
        return list(cursor)
    
    def search_users(self, query: str) -> List[Dict[str, Any]]:
        """Полнотекстовый поиск пользователей"""
        cursor = self.users.find(
            {"$text": {"$search": query}},
            {"score": {"$meta": "textScore"}}
        ).sort([("score", {"$meta": "textScore"})])
        
        return list(cursor)
    
    def update_user_login(self, user_id: str) -> bool:
        """Обновление данных входа пользователя"""
        result = self.users.update_one(
            {"_id": ObjectId(user_id)},
            {
                "$set": {
                    "metadata.lastLogin": datetime.datetime.utcnow(),
                    "updatedAt": datetime.datetime.utcnow()
                },
                "$inc": {"metadata.loginCount": 1}
            }
        )
        return result.modified_count > 0
    
    def get_user_statistics(self) -> List[Dict[str, Any]]:
        """Получение статистики пользователей"""
        pipeline = [
            {
                "$group": {
                    "_id": "$profile.city",
                    "totalUsers": {"$sum": 1},
                    "averageAge": {"$avg": "$profile.age"},
                    "premiumUsers": {
                        "$sum": {
                            "$cond": [
                                {"$eq": ["$metadata.subscription", "premium"]},
                                1,
                                0
                            ]
                        }
                    }
                }
            },
            {
                "$sort": {"totalUsers": -1}
            }
        ]
        
        return list(self.users.aggregate(pipeline))
    
    def close(self):
        """Закрытие соединения"""
        self.client.close()
```

---

## 📚 Расширенная агрегация для аналитики

```python
# services/analytics-service/mongodb_aggregation.py
def get_user_behavior_analytics(self, date_from: datetime, date_to: datetime):
    """Комплексная аналитика поведения пользователей"""
    pipeline = [
        {
            "$match": {
                "created_at": {"$gte": date_from, "$lte": date_to}
            }
        },
        {
            "$lookup": {
                "from": "users",
                "localField": "user_id",
                "foreignField": "_id",
                "as": "user"
            }
        },
        {"$unwind": "$user"},
        {
            "$addFields": {
                "hour": {"$hour": "$created_at"},
                "dayOfWeek": {"$dayOfWeek": "$created_at"},
                "orderValue": {"$sum": "$items.price"}
            }
        },
        {
            "$group": {
                "_id": {
                    "user_segment": {
                        "$switch": {
                            "branches": [
                                {"case": {"$gte": ["$orderValue", 1000]}, "then": "premium"},
                                {"case": {"$gte": ["$orderValue", 100]}, "then": "regular"},
                                {"case": {"$lt": ["$orderValue", 100]}, "then": "budget"}
                            ]
                        }
                    },
                    "hour": "$hour",
                    "dayOfWeek": "$dayOfWeek"
                },
                "order_count": {"$sum": 1},
                "avg_order_value": {"$avg": "$orderValue"},
                "total_revenue": {"$sum": "$orderValue"},
                "unique_users": {"$addToSet": "$user_id"}
            }
        },
        {
            "$addFields": {
                "unique_user_count": {"$size": "$unique_users"}
            }
        },
        {
            "$sort": {"_id.user_segment": 1, "_id.dayOfWeek": 1, "_id.hour": 1}
        }
    ]
    
    return list(self.db.orders.aggregate(pipeline))

def get_product_recommendation_data(self, user_id: str, limit: int = 10):
    """Данные для рекомендации товаров"""
    pipeline = [
        {"$match": {"user_id": user_id}},
        {"$unwind": "$items"},
        {
            "$lookup": {
                "from": "products",
                "localField": "items.product_id",
                "foreignField": "_id",
                "as": "product"
            }
        },
        {"$unwind": "$product"},
        {
            "$group": {
                "_id": "$product.category",
                "purchased_products": {"$addToSet": "$items.product_id"},
                "total_spent": {"$sum": "$items.price"},
                "purchase_count": {"$sum": 1}
            }
        },
        {
            "$lookup": {
                "from": "orders",
                "let": {"category": "$_id", "user_products": "$purchased_products"},
                "pipeline": [
                    {"$unwind": "$items"},
                    {
                        "$lookup": {
                            "from": "products",
                            "localField": "items.product_id",
                            "foreignField": "_id",
                            "as": "item_product"
                        }
                    },
                    {"$unwind": "$item_product"},
                    {
                        "$match": {
                            "$expr": {
                                "$and": [
                                    {"$eq": ["$item_product.category", "$$category"]},
                                    {"$not": {"$in": ["$items.product_id", "$$user_products"]}}
                                ]
                            }
                        }
                    },
                    {
                        "$group": {
                            "_id": "$items.product_id",
                            "purchase_frequency": {"$sum": 1},
                            "avg_rating": {"$avg": "$items.rating"}
                        }
                    },
                    {"$sort": {"purchase_frequency": -1}},
                    {"$limit": 5}
                ],
                "as": "recommended_products"
            }
        },
        {"$sort": {"total_spent": -1}},
        {"$limit": limit}
    ]
    
    return list(self.db.orders.aggregate(pipeline))
```

---

## 🎯 Практические задания

### 🔰 Начальный уровень
1. Создайте коллекцию блогов с статьями и комментариями
2. Реализуйте систему тегов для статей
3. Создайте простую систему поиска по названию

### 🚀 Средний уровень
1. Реализуйте систему ролей пользователей с вложенными разрешениями
2. Создайте агрегационный pipeline для подсчета статистики активности
3. Реализуйте полнотекстовый поиск с ранжированием

### 💎 Продвинутый уровень
1. Создайте распределенную систему с шардированием
2. Реализуйте систему real-time уведомлений с change streams
3. Создайте систему аналитики с комплексными агрегациями

---

## 🌟 Лучшие практики

### ✅ Рекомендации
- **Проектируйте схему под запросы** - структура документов должна соответствовать паттернам доступа
- **Используйте встраивание для связанных данных** - если данные читаются вместе, храните их вместе
- **Создавайте составные индексы** - для запросов по нескольким полям
- **Ограничивайте размер документов** - максимум 16MB

### ❌ Чего избегать
- **Избыточной нормализации** - MongoDB не реляционная БД
- **JOIN-подобных операций** - минимизируйте использование $lookup
- **Больших массивов в документах** - влияют на производительность
- **Отсутствия индексов** - критично для производительности

---

## 🔗 Связанные темы

- 🐘 **[[postgresql|PostgreSQL]]** - для сравнения с реляционными БД
- 🔴 **[[redis|Redis]]** - для кеширования и быстрого доступа
- 🏗️ **[[database-design|Проектирование БД]]** - принципы проектирования
- ⚡ **[[sql-optimization|Оптимизация БД]]** - общие принципы оптимизации

**[[README|🏠 Вернуться к обзору баз данных]]** 