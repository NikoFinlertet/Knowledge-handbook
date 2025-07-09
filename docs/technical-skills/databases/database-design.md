# Проектирование баз данных 🏗️

## 🏠 Навигация
**[[README|🏠 Базы данных]]** → **Проектирование БД**

**Связанные темы:**
- 🐘 [[postgresql|PostgreSQL]]
- 🍃 [[mongodb|MongoDB]]
- 🔴 [[redis|Redis]]
- 🏭 [[data-warehousing|Data Warehousing]]
- 🏗️ [[database-architecture-patterns|Архитектурные паттерны]]

---

## 🎯 Принципы проектирования баз данных

Правильное проектирование базы данных — это основа эффективной и масштабируемой системы. От качества модели данных зависит производительность, целостность и удобство сопровождения всего приложения.

### ✨ Ключевые цели проектирования

1. **🎯 Корректность** - модель должна точно отражать предметную область
2. **⚡ Производительность** - эффективные запросы и операции
3. **🔒 Целостность** - данные должны быть консистентными
4. **📈 Масштабируемость** - готовность к росту объемов
5. **🛠️ Сопровождаемость** - легкость внесения изменений

---

## 📐 Нормализация баз данных

### 🔍 Первая нормальная форма (1NF)

**Принцип:** Каждая ячейка таблицы должна содержать атомарное значение.

#### ❌ Нарушение 1NF
```sql
-- Плохо: множественные значения в одной ячейке
CREATE TABLE users_bad (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    phones TEXT  -- "123-456-789, 987-654-321, 555-000-111"
);
```

#### ✅ Соответствие 1NF  
```sql
-- Хорошо: отдельная таблица для телефонов
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE user_phones (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    phone VARCHAR(20),
    phone_type VARCHAR(10) -- mobile, home, work
);
```

### 🔍 Вторая нормальная форма (2NF)

**Принцип:** Таблица в 1NF + все неключевые атрибуты полностью зависят от первичного ключа.

#### ❌ Нарушение 2NF
```sql
-- Плохо: частичная зависимость от составного ключа
CREATE TABLE order_items_bad (
    order_id INTEGER,
    product_id INTEGER,
    quantity INTEGER,
    product_name VARCHAR(100),  -- зависит только от product_id
    product_price DECIMAL(10,2), -- зависит только от product_id
    PRIMARY KEY (order_id, product_id)
);
```

#### ✅ Соответствие 2NF
```sql
-- Хорошо: разделение на отдельные таблицы
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2)
);

CREATE TABLE order_items (
    order_id INTEGER,
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER,
    PRIMARY KEY (order_id, product_id)
);
```

### 🔍 Третья нормальная форма (3NF)

**Принцип:** Таблица в 2NF + отсутствие транзитивных зависимостей.

#### ❌ Нарушение 3NF
```sql
-- Плохо: транзитивная зависимость
CREATE TABLE employees_bad (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department_id INTEGER,
    department_name VARCHAR(100), -- зависит от department_id
    department_location VARCHAR(100) -- зависит от department_id
);
```

#### ✅ Соответствие 3NF
```sql
-- Хорошо: вынос зависимых атрибутов
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    location VARCHAR(100)
);

CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department_id INTEGER REFERENCES departments(id)
);
```

---

## 🏗️ Проектирование схемы

### 📊 ER-диаграммы и моделирование

```python
# services/design/er_modeling.py
from dataclasses import dataclass
from typing import List, Dict, Optional
from enum import Enum

class RelationshipType(Enum):
    ONE_TO_ONE = "1:1"
    ONE_TO_MANY = "1:N"
    MANY_TO_MANY = "N:M"

@dataclass
class Entity:
    """Сущность в ER-модели"""
    name: str
    attributes: List[str]
    primary_key: str
    
class Relationship:
    """Связь между сущностями"""
    def __init__(self, entity1: str, entity2: str, 
                 relationship_type: RelationshipType,
                 description: str = ""):
        self.entity1 = entity1
        self.entity2 = entity2
        self.type = relationship_type
        self.description = description

class ERModel:
    """ER-модель базы данных"""
    
    def __init__(self, name: str):
        self.name = name
        self.entities: Dict[str, Entity] = {}
        self.relationships: List[Relationship] = []
    
    def add_entity(self, entity: Entity):
        """Добавление сущности"""
        self.entities[entity.name] = entity
    
    def add_relationship(self, relationship: Relationship):
        """Добавление связи"""
        self.relationships.append(relationship)
    
    def generate_sql_schema(self) -> str:
        """Генерация SQL схемы"""
        sql_statements = []
        
        # Создание таблиц для сущностей
        for entity in self.entities.values():
            columns = []
            for attr in entity.attributes:
                if attr == entity.primary_key:
                    columns.append(f"{attr} SERIAL PRIMARY KEY")
                else:
                    columns.append(f"{attr} VARCHAR(255)")
            
            sql = f"""
CREATE TABLE {entity.name} (
    {',\n    '.join(columns)}
);"""
            sql_statements.append(sql.strip())
        
        # Добавление внешних ключей для связей
        for rel in self.relationships:
            if rel.type == RelationshipType.ONE_TO_MANY:
                # Добавляем FK в "многие"
                fk_sql = f"""
ALTER TABLE {rel.entity2} 
ADD COLUMN {rel.entity1}_id INTEGER REFERENCES {rel.entity1}(id);"""
                sql_statements.append(fk_sql.strip())
            
            elif rel.type == RelationshipType.MANY_TO_MANY:
                # Создаем промежуточную таблицу
                junction_table = f"{rel.entity1}_{rel.entity2}"
                junction_sql = f"""
CREATE TABLE {junction_table} (
    {rel.entity1}_id INTEGER REFERENCES {rel.entity1}(id),
    {rel.entity2}_id INTEGER REFERENCES {rel.entity2}(id),
    PRIMARY KEY ({rel.entity1}_id, {rel.entity2}_id)
);"""
                sql_statements.append(junction_sql.strip())
        
        return '\n\n'.join(sql_statements)

# Пример: модель интернет-магазина
ecommerce_model = ERModel("E-Commerce System")

# Сущности
ecommerce_model.add_entity(Entity(
    name="users",
    attributes=["id", "email", "password_hash", "name", "created_at"],
    primary_key="id"
))

ecommerce_model.add_entity(Entity(
    name="categories",
    attributes=["id", "name", "description", "parent_id"],
    primary_key="id"
))

ecommerce_model.add_entity(Entity(
    name="products",
    attributes=["id", "name", "description", "price", "stock_quantity"],
    primary_key="id"
))

ecommerce_model.add_entity(Entity(
    name="orders",
    attributes=["id", "total_amount", "status", "created_at"],
    primary_key="id"
))

ecommerce_model.add_entity(Entity(
    name="order_items",
    attributes=["id", "quantity", "price_at_time"],
    primary_key="id"
))

# Связи
ecommerce_model.add_relationship(Relationship(
    "categories", "products", 
    RelationshipType.ONE_TO_MANY,
    "Категория содержит продукты"
))

ecommerce_model.add_relationship(Relationship(
    "users", "orders",
    RelationshipType.ONE_TO_MANY,
    "Пользователь делает заказы"
))

ecommerce_model.add_relationship(Relationship(
    "orders", "order_items",
    RelationshipType.ONE_TO_MANY,
    "Заказ содержит позиции"
))

ecommerce_model.add_relationship(Relationship(
    "products", "order_items",
    RelationshipType.ONE_TO_MANY,
    "Продукт может быть в разных заказах"
))

print(ecommerce_model.generate_sql_schema())
```

---

## 🎨 Паттерны проектирования

### 🔑 Суррогатные и естественные ключи

```sql
-- Суррогатный ключ (рекомендуется)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,  -- Суррогатный ключ
    email VARCHAR(255) UNIQUE NOT NULL,  -- Естественный ключ как constraint
    name VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Преимущества суррогатных ключей:
-- 1. Неизменность
-- 2. Производительность (integer vs string)
-- 3. Простота связей
-- 4. Независимость от бизнес-логики
```

### 🏷️ Lookup Tables (Справочники)

```sql
-- Паттерн справочных таблиц
CREATE TABLE order_statuses (
    id SERIAL PRIMARY KEY,
    code VARCHAR(20) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    is_active BOOLEAN DEFAULT TRUE
);

INSERT INTO order_statuses (code, name) VALUES 
('pending', 'Ожидает обработки'),
('processing', 'В обработке'),
('shipped', 'Отправлен'),
('delivered', 'Доставлен'),
('cancelled', 'Отменен');

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    status_id INTEGER REFERENCES order_statuses(id),
    total_amount DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 📋 Audit Trail (Аудит изменений)

```sql
-- Паттерн аудита изменений
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by INTEGER REFERENCES users(id),
    updated_by INTEGER REFERENCES users(id)
);

-- Таблица истории изменений
CREATE TABLE product_audit (
    id SERIAL PRIMARY KEY,
    product_id INTEGER REFERENCES products(id),
    field_name VARCHAR(100),
    old_value TEXT,
    new_value TEXT,
    changed_by INTEGER REFERENCES users(id),
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    operation VARCHAR(10) -- INSERT, UPDATE, DELETE
);

-- Триггер для автоматического аудита
CREATE OR REPLACE FUNCTION audit_product_changes()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'UPDATE' THEN
        -- Проверяем каждое поле на изменения
        IF OLD.name != NEW.name THEN
            INSERT INTO product_audit (product_id, field_name, old_value, new_value, changed_by, operation)
            VALUES (NEW.id, 'name', OLD.name, NEW.name, NEW.updated_by, 'UPDATE');
        END IF;
        
        IF OLD.price != NEW.price THEN
            INSERT INTO product_audit (product_id, field_name, old_value, new_value, changed_by, operation)
            VALUES (NEW.id, 'price', OLD.price::TEXT, NEW.price::TEXT, NEW.updated_by, 'UPDATE');
        END IF;
        
        NEW.updated_at = CURRENT_TIMESTAMP;
        RETURN NEW;
    END IF;
    
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER product_audit_trigger
    AFTER UPDATE ON products
    FOR EACH ROW
    EXECUTE FUNCTION audit_product_changes();
```

### 🔄 Soft Delete (Мягкое удаление)

```sql
-- Паттерн мягкого удаления
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    is_deleted BOOLEAN DEFAULT FALSE,
    deleted_at TIMESTAMP NULL,
    deleted_by INTEGER REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Индекс для активных записей
CREATE INDEX idx_products_active ON products (id) WHERE is_deleted = FALSE;

-- Представление только активных продуктов
CREATE VIEW active_products AS
SELECT id, name, price, created_at
FROM products 
WHERE is_deleted = FALSE;

-- Функция мягкого удаления
CREATE OR REPLACE FUNCTION soft_delete_product(product_id INTEGER, deleted_by_user INTEGER)
RETURNS BOOLEAN AS $$
BEGIN
    UPDATE products 
    SET is_deleted = TRUE,
        deleted_at = CURRENT_TIMESTAMP,
        deleted_by = deleted_by_user
    WHERE id = product_id AND is_deleted = FALSE;
    
    RETURN FOUND;
END;
$$ LANGUAGE plpgsql;
```

---

## 🔒 Ограничения целостности

### ✅ Check Constraints

```sql
-- Проверочные ограничения
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL CHECK (price > 0),
    stock_quantity INTEGER CHECK (stock_quantity >= 0),
    rating DECIMAL(3,2) CHECK (rating >= 0 AND rating <= 5),
    category_id INTEGER REFERENCES categories(id),
    
    -- Сложные ограничения
    CONSTRAINT valid_price_range CHECK (
        CASE 
            WHEN category_id = 1 THEN price BETWEEN 10 AND 1000  -- Электроника
            WHEN category_id = 2 THEN price BETWEEN 5 AND 500    -- Книги
            ELSE price > 0
        END
    )
);

-- Ограничения на уровне таблицы
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    discount_amount DECIMAL(10,2) DEFAULT 0,
    total_amount DECIMAL(10,2) NOT NULL,
    
    -- Скидка не может быть больше суммы заказа
    CONSTRAINT valid_discount CHECK (discount_amount <= total_amount),
    
    -- Минимальная сумма заказа
    CONSTRAINT min_order_amount CHECK (total_amount >= 1.00)
);
```

### 🔗 Уникальные ограничения

```sql
-- Простые уникальные ограничения
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20) UNIQUE,
    username VARCHAR(50) UNIQUE NOT NULL
);

-- Составные уникальные ограничения
CREATE TABLE user_addresses (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    address_type VARCHAR(20), -- home, work, billing
    street VARCHAR(255),
    city VARCHAR(100),
    
    -- Пользователь может иметь только один адрес каждого типа
    UNIQUE (user_id, address_type)
);

-- Частичные уникальные ограничения (с условием)
CREATE TABLE user_settings (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    setting_key VARCHAR(100),
    setting_value TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    
    -- Уникальность только для активных настроек
    CONSTRAINT unique_active_setting 
    EXCLUDE USING btree (user_id WITH =, setting_key WITH =) 
    WHERE (is_active = TRUE)
);
```

---

## 📊 Денормализация для производительности

### ⚡ Когда денормализовать

```sql
-- Пример: частые запросы со сложными JOIN'ами
-- Нормализованная структура (медленно)
SELECT 
    o.id,
    u.name as user_name,
    u.email,
    COUNT(oi.id) as item_count,
    SUM(oi.quantity * oi.price) as total_amount
FROM orders o
JOIN users u ON o.user_id = u.id
JOIN order_items oi ON o.id = oi.order_id
GROUP BY o.id, u.name, u.email;

-- Денормализованная структура (быстро)
CREATE TABLE order_summary (
    order_id INTEGER PRIMARY KEY REFERENCES orders(id),
    user_name VARCHAR(255),
    user_email VARCHAR(255),
    item_count INTEGER,
    total_amount DECIMAL(10,2),
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Автоматическое обновление через триггеры
CREATE OR REPLACE FUNCTION update_order_summary()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO order_summary (order_id, user_name, user_email, item_count, total_amount)
    SELECT 
        o.id,
        u.name,
        u.email,
        COUNT(oi.id),
        SUM(oi.quantity * oi.price)
    FROM orders o
    JOIN users u ON o.user_id = u.id
    LEFT JOIN order_items oi ON o.id = oi.order_id
    WHERE o.id = NEW.order_id
    GROUP BY o.id, u.name, u.email
    ON CONFLICT (order_id) DO UPDATE SET
        item_count = EXCLUDED.item_count,
        total_amount = EXCLUDED.total_amount,
        last_updated = CURRENT_TIMESTAMP;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

---

## 🎯 Практические задания

### 🔰 Начальный уровень
1. Спроектируйте схему для библиотеки (книги, авторы, читатели, выдачи)
2. Приведите таблицу к 3NF, исправив нарушения нормализации
3. Добавьте ограничения целостности для обеспечения корректности данных

### 🚀 Средний уровень
1. Создайте ER-модель для социальной сети с друзьями, постами и комментариями
2. Реализуйте систему аудита изменений для критических таблиц
3. Спроектируйте многоуровневую иерархию категорий с эффективными запросами

### 💎 Продвинутый уровень
1. Создайте схему для многоарендной SaaS системы с изоляцией данных
2. Реализуйте temporal tables для хранения истории изменений
3. Спроектируйте схему для системы с миллиардами записей, учитывая шардинг

---

## 🌟 Лучшие практики

### ✅ Рекомендации

#### 🏗️ Общие принципы
- **Понимайте предметную область** - модель должна отражать бизнес-требования
- **Начинайте с нормализации** - потом денормализуйте по необходимости
- **Используйте суррогатные ключи** - для стабильности и производительности
- **Добавляйте метаданные** - created_at, updated_at, version

#### 🔒 Целостность данных
- **Ограничения на уровне БД** - не полагайтесь только на приложение
- **Внешние ключи** - для ссылочной целостности
- **Check constraints** - для бизнес-правил
- **Null constraints** - четко определяйте обязательные поля

#### 📊 Производительность
- **Индексы на часто используемые поля** - особенно для WHERE, JOIN, ORDER BY
- **Избегайте избыточной нормализации** - баланс между нормализацией и производительностью
- **Партиционирование больших таблиц** - для управляемости
- **Архивация старых данных** - для поддержания производительности

### ❌ Чего избегать

- **God tables** - избегайте таблиц с десятками колонок
- **Избыточная денормализация** - не денормализуйте без измерений
- **Отсутствие документации** - схема должна быть понятна
- **Игнорирование производительности** - тестируйте на реальных объемах

---

## 🔗 Связанные темы

- 🐘 **[[postgresql|PostgreSQL]]** - реализация принципов в PostgreSQL
- 🍃 **[[mongodb|MongoDB]]** - документно-ориентированное моделирование
- 🏭 **[[data-warehousing|Data Warehousing]]** - принципы OLAP моделирования
- 🏗️ **[[database-architecture-patterns|Архитектурные паттерны]]** - паттерны на уровне архитектуры
- ⚡ **[[sql-optimization|Оптимизация БД]]** - оптимизация производительности

**[[README|🏠 Вернуться к обзору баз данных]]** 