# PostgreSQL

> **Мощная объектно-реляционная СУБД**  
> ACID транзакции, расширенные типы данных и enterprise возможности

[[Базы данных]] → **PostgreSQL**

---

## 🎯 Ключевые возможности

### 🏗️ Основные характеристики
- **ACID транзакции** - полная поддержка надежности
- **Расширенные типы данных** - JSON, XML, массивы, пользовательские типы
- **Полнотекстовый поиск** - встроенные возможности поиска
- **Расширения** - PostGIS, pg_stat_statements, и тысячи других
- **Параллельные запросы** - автоматическая параллелизация

### 🔗 Связанные темы
- **[[Базы данных]]** - общий обзор технологий БД
- **[[SQL оптимизация]]** - оптимизация запросов
- **[[Database Design]]** - проектирование схем баз данных
- **[[Репликация баз данных]]** - высокая доступность

---

## 🚀 Базовая настройка

### Создание базовой схемы

```sql
-- services/user-service/database/migrations/001_create_users_table.sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- Создание таблицы пользователей
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    status VARCHAR(50) DEFAULT 'active',
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Индексы для производительности
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_users_created_at ON users(created_at);

-- GIN индекс для JSONB поля
CREATE INDEX idx_users_metadata ON users USING gin (metadata);

-- Функция для автоматического обновления updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Триггер для автоматического обновления updated_at
CREATE TRIGGER update_users_updated_at 
    BEFORE UPDATE ON users 
    FOR EACH ROW 
    EXECUTE FUNCTION update_updated_at_column();
```

### Расширенные типы данных

```sql
-- services/ecommerce/database/advanced_types.sql

-- Пользовательские типы
CREATE TYPE order_status AS ENUM ('pending', 'processing', 'shipped', 'delivered', 'cancelled');
CREATE TYPE address_type AS (
    street VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(100),
    postal_code VARCHAR(20),
    country VARCHAR(100)
);

-- Таблица с расширенными типами
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID REFERENCES users(id),
    status order_status DEFAULT 'pending',
    items JSONB NOT NULL,
    shipping_address address_type,
    tags TEXT[],
    coordinates POINT,
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Работа с массивами
INSERT INTO orders (user_id, items, tags, coordinates)
VALUES (
    'user-uuid',
    '[{"product_id": "123", "quantity": 2, "price": 29.99}]'::jsonb,
    ARRAY['urgent', 'gift', 'priority'],
    POINT(37.7749, -122.4194)  -- Сан-Франциско
);

-- Поиск по массивам
SELECT * FROM orders 
WHERE 'urgent' = ANY(tags);

-- Поиск по JSONB
SELECT * FROM orders 
WHERE items @> '[{"product_id": "123"}]';
```

---

## 🔍 JSONB и продвинутые запросы

### JSONB операции

```sql
-- services/user-service/database/queries/advanced_users.sql

-- Поиск пользователей с определенными метаданными
SELECT id, name, email, metadata
FROM users
WHERE metadata @> '{"subscription": "premium"}';

-- Обновление вложенных JSON полей
UPDATE users 
SET metadata = metadata || '{"last_login": "2024-01-15T10:30:00Z"}'
WHERE id = 'user-uuid';

-- Извлечение данных из JSON
SELECT 
    id,
    name,
    metadata->>'subscription' as subscription_type,
    metadata->'preferences'->>'theme' as theme,
    metadata#>'{preferences,notifications}' as notification_settings
FROM users
WHERE metadata ? 'subscription';

-- Агрегация по JSON полям
SELECT 
    metadata->>'subscription' as subscription_type,
    COUNT(*) as user_count,
    AVG((metadata->>'login_count')::int) as avg_logins
FROM users
WHERE metadata ? 'subscription'
GROUP BY metadata->>'subscription';

-- Индексирование JSON полей
CREATE INDEX idx_users_subscription ON users 
USING gin ((metadata->'subscription'));

-- Функциональный индекс для быстрого поиска
CREATE INDEX idx_users_subscription_type ON users 
((metadata->>'subscription')) 
WHERE metadata ? 'subscription';
```

### Сложные JSONB запросы

```sql
-- services/analytics/database/complex_jsonb.sql

-- Работа с массивами в JSONB
SELECT 
    id,
    jsonb_array_elements(metadata->'tags') as tag
FROM products
WHERE jsonb_typeof(metadata->'tags') = 'array';

-- Трансформация JSON структур
SELECT 
    id,
    jsonb_object_agg(
        setting_key, 
        setting_value
    ) as user_settings
FROM (
    SELECT 
        id,
        jsonb_object_keys(metadata->'settings') as setting_key,
        metadata->'settings'->jsonb_object_keys(metadata->'settings') as setting_value
    FROM users
    WHERE metadata ? 'settings'
) t
GROUP BY id;

-- Статистика по вложенным JSON объектам
WITH tag_stats AS (
    SELECT 
        jsonb_array_elements_text(metadata->'tags') as tag,
        COUNT(*) as usage_count
    FROM products
    WHERE metadata ? 'tags'
    GROUP BY jsonb_array_elements_text(metadata->'tags')
)
SELECT 
    tag,
    usage_count,
    RANK() OVER (ORDER BY usage_count DESC) as popularity_rank
FROM tag_stats
ORDER BY usage_count DESC;
```

---

## 🔎 Полнотекстовый поиск

### Настройка полнотекстового поиска

```sql
-- services/search/database/fulltext_search.sql

-- Добавление полнотекстового поиска к пользователям
ALTER TABLE users ADD COLUMN search_vector tsvector;

-- Создание функции для обновления search_vector
CREATE OR REPLACE FUNCTION update_user_search_vector()
RETURNS trigger AS $$
BEGIN
    NEW.search_vector = to_tsvector('english', 
        coalesce(NEW.name, '') || ' ' || 
        coalesce(NEW.email, '') || ' ' || 
        coalesce(NEW.metadata::text, '')
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Триггер для автоматического обновления search_vector
CREATE TRIGGER update_user_search_vector_trigger
    BEFORE INSERT OR UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_user_search_vector();

-- Индекс для полнотекстового поиска
CREATE INDEX idx_users_search_vector ON users USING gin(search_vector);

-- Обновление существующих записей
UPDATE users SET search_vector = to_tsvector('english', 
    coalesce(name, '') || ' ' || 
    coalesce(email, '') || ' ' || 
    coalesce(metadata::text, '')
);
```

### Продвинутый поиск

```sql
-- Базовый полнотекстовый поиск
SELECT id, name, email, 
       ts_rank(search_vector, query) as rank
FROM users, to_tsquery('english', 'john & developer') as query
WHERE search_vector @@ query
ORDER BY rank DESC;

-- Поиск с весами разных полей
CREATE OR REPLACE FUNCTION create_weighted_search_vector(
    name_weight char DEFAULT 'A',
    email_weight char DEFAULT 'B',
    metadata_weight char DEFAULT 'C'
)
RETURNS tsvector AS $$
BEGIN
    RETURN setweight(to_tsvector('english', coalesce(name, '')), name_weight) ||
           setweight(to_tsvector('english', coalesce(email, '')), email_weight) ||
           setweight(to_tsvector('english', coalesce(metadata::text, '')), metadata_weight);
END;
$$ LANGUAGE plpgsql IMMUTABLE;

-- Поиск с автодополнением
SELECT DISTINCT word
FROM ts_stat('SELECT search_vector FROM users')
WHERE word ILIKE $1 || '%'
ORDER BY word
LIMIT 10;

-- Поиск похожих слов (требует расширение pg_trgm)
CREATE EXTENSION IF NOT EXISTS pg_trgm;

SELECT name, email, similarity(name, 'john') as sim
FROM users
WHERE name % 'john'  -- оператор похожести
ORDER BY sim DESC
LIMIT 5;
```

---

## 📊 Партиционирование

### Временное партиционирование

```sql
-- services/order-service/database/partitioning.sql

-- Создание партиционированной таблицы заказов
CREATE TABLE orders (
    id UUID DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    status VARCHAR(50) DEFAULT 'pending',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
) PARTITION BY RANGE (created_at);

-- Создание партиций по месяцам
CREATE TABLE orders_2024_01 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE orders_2024_02 PARTITION OF orders
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

CREATE TABLE orders_2024_03 PARTITION OF orders
    FOR VALUES FROM ('2024-03-01') TO ('2024-04-01');

-- Автоматическое создание партиций
CREATE OR REPLACE FUNCTION create_monthly_partition(table_name text, start_date date)
RETURNS void AS $$
DECLARE
    partition_name text;
    end_date date;
BEGIN
    partition_name := table_name || '_' || to_char(start_date, 'YYYY_MM');
    end_date := start_date + interval '1 month';
    
    EXECUTE format('CREATE TABLE IF NOT EXISTS %I PARTITION OF %I 
                   FOR VALUES FROM (%L) TO (%L)',
                   partition_name, table_name, start_date, end_date);
    
    -- Создаем индексы для новой партиции
    EXECUTE format('CREATE INDEX IF NOT EXISTS %I ON %I (user_id)',
                   partition_name || '_user_id_idx', partition_name);
    EXECUTE format('CREATE INDEX IF NOT EXISTS %I ON %I (status)',
                   partition_name || '_status_idx', partition_name);
END;
$$ LANGUAGE plpgsql;

-- Процедура для автоматического создания будущих партиций
CREATE OR REPLACE FUNCTION maintain_partitions()
RETURNS void AS $$
DECLARE
    start_date date;
BEGIN
    -- Создаем партиции на 3 месяца вперед
    FOR i IN 0..2 LOOP
        start_date := date_trunc('month', CURRENT_DATE) + (i || ' months')::interval;
        PERFORM create_monthly_partition('orders', start_date);
    END LOOP;
END;
$$ LANGUAGE plpgsql;

-- Планировщик для автоматического создания партиций (через pg_cron)
-- SELECT cron.schedule('maintain-partitions', '0 0 1 * *', 'SELECT maintain_partitions();');
```

### Hash партиционирование

```sql
-- Партиционирование по пользователям для равномерного распределения
CREATE TABLE user_activities (
    id UUID DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL,
    activity_type VARCHAR(50) NOT NULL,
    data JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
) PARTITION BY HASH (user_id);

-- Создание 4 партиций для равномерного распределения
CREATE TABLE user_activities_0 PARTITION OF user_activities
    FOR VALUES WITH (modulus 4, remainder 0);

CREATE TABLE user_activities_1 PARTITION OF user_activities
    FOR VALUES WITH (modulus 4, remainder 1);

CREATE TABLE user_activities_2 PARTITION OF user_activities
    FOR VALUES WITH (modulus 4, remainder 2);

CREATE TABLE user_activities_3 PARTITION OF user_activities
    FOR VALUES WITH (modulus 4, remainder 3);
```

---

## 🚀 Оптимизация производительности

### Стратегии индексирования

```sql
-- services/user-service/database/indexes.sql

-- Составной индекс для частых запросов
CREATE INDEX idx_users_status_created_at ON users(status, created_at DESC);

-- Частичный индекс для активных пользователей
CREATE INDEX idx_users_active_email ON users(email) WHERE status = 'active';

-- Функциональный индекс для поиска без учета регистра
CREATE INDEX idx_users_lower_email ON users(lower(email));

-- Покрывающий индекс (включает дополнительные столбцы)
CREATE INDEX idx_users_status_covering ON users(status) INCLUDE (name, email, created_at);

-- Анализ использования индексов
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch,
    pg_size_pretty(pg_relation_size(indexrelid)) as index_size
FROM pg_stat_user_indexes 
WHERE schemaname = 'public'
ORDER BY idx_scan DESC;

-- Поиск неиспользуемых индексов
SELECT 
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) as index_size
FROM pg_stat_user_indexes 
WHERE idx_scan = 0
  AND schemaname = 'public'
ORDER BY pg_relation_size(indexrelid) DESC;
```

### Window Functions и CTE

```sql
-- services/analytics-service/database/analytics_queries.sql

-- Ранжирование пользователей по активности
SELECT 
    user_id,
    login_count,
    ROW_NUMBER() OVER (ORDER BY login_count DESC) as rank,
    RANK() OVER (ORDER BY login_count DESC) as dense_rank,
    PERCENT_RANK() OVER (ORDER BY login_count DESC) as percentile,
    NTILE(10) OVER (ORDER BY login_count DESC) as decile
FROM user_analytics
WHERE last_login >= CURRENT_DATE - INTERVAL '30 days';

-- Скользящее среднее заказов
SELECT 
    order_date,
    daily_orders,
    AVG(daily_orders) OVER (
        ORDER BY order_date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg_7days,
    SUM(daily_orders) OVER (
        ORDER BY order_date 
        ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
    ) as rolling_sum_30days
FROM daily_order_stats
ORDER BY order_date;

-- CTE для иерархических данных
WITH RECURSIVE category_hierarchy AS (
    -- Базовый случай: корневые категории
    SELECT id, name, parent_id, 1 as level, name as path
    FROM categories 
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- Рекурсивный случай: дочерние категории
    SELECT c.id, c.name, c.parent_id, ch.level + 1, 
           ch.path || ' > ' || c.name
    FROM categories c
    JOIN category_hierarchy ch ON c.parent_id = ch.id
    WHERE ch.level < 10  -- Предотвращение бесконечной рекурсии
)
SELECT * FROM category_hierarchy ORDER BY path;

-- Сложные аналитические запросы
WITH monthly_stats AS (
    SELECT 
        DATE_TRUNC('month', created_at) as month,
        COUNT(*) as total_orders,
        SUM(total_amount) as revenue,
        COUNT(DISTINCT user_id) as unique_customers,
        AVG(total_amount) as avg_order_value
    FROM orders
    WHERE created_at >= CURRENT_DATE - INTERVAL '12 months'
    GROUP BY DATE_TRUNC('month', created_at)
),
growth_stats AS (
    SELECT 
        month,
        total_orders,
        revenue,
        unique_customers,
        avg_order_value,
        LAG(revenue) OVER (ORDER BY month) as prev_month_revenue,
        LAG(total_orders) OVER (ORDER BY month) as prev_month_orders,
        (revenue - LAG(revenue) OVER (ORDER BY month)) / 
        NULLIF(LAG(revenue) OVER (ORDER BY month), 0) * 100 as revenue_growth,
        (total_orders - LAG(total_orders) OVER (ORDER BY month)) /
        NULLIF(LAG(total_orders) OVER (ORDER BY month), 0) * 100 as order_growth
    FROM monthly_stats
)
SELECT 
    month,
    total_orders,
    revenue,
    unique_customers,
    avg_order_value,
    COALESCE(revenue_growth, 0) as revenue_growth_percent,
    COALESCE(order_growth, 0) as order_growth_percent
FROM growth_stats
ORDER BY month;
```

---

## 🔄 Репликация и высокая доступность

### Streaming репликация

```sql
-- Настройка на мастере
-- postgresql.conf
wal_level = replica
max_wal_senders = 10
max_replication_slots = 10
synchronous_commit = on

-- pg_hba.conf
# Разрешить подключение реплики
host replication replicator 10.0.0.0/24 md5

-- Создание пользователя для репликации
CREATE USER replicator REPLICATION LOGIN CONNECTION LIMIT 5 ENCRYPTED PASSWORD 'secure_password';
```

```bash
# На slave сервере - создание базовой копии
pg_basebackup -h master_host -D /var/lib/postgresql/data -U replicator -P -v -R -X stream -C -S replica_slot

# recovery.conf (PostgreSQL < 12) или postgresql.conf (PostgreSQL >= 12)
# standby_mode = 'on'
# primary_conninfo = 'host=master_host port=5432 user=replicator password=secure_password'
# primary_slot_name = 'replica_slot'
```

### Логическая репликация

```sql
-- На publisher (master)
CREATE PUBLICATION user_data_pub FOR TABLE users, user_profiles;

-- На subscriber (replica)
CREATE SUBSCRIPTION user_data_sub 
CONNECTION 'host=master_host port=5432 user=replicator password=secure_password dbname=myapp' 
PUBLICATION user_data_pub;

-- Мониторинг репликации
SELECT 
    slot_name,
    slot_type,
    database,
    active,
    restart_lsn,
    confirmed_flush_lsn
FROM pg_replication_slots;

SELECT 
    client_addr,
    state,
    sent_lsn,
    write_lsn,
    flush_lsn,
    replay_lsn,
    write_lag,
    flush_lag,
    replay_lag
FROM pg_stat_replication;
```

---

## 📈 Мониторинг и диагностика

### Статистика и метрики

```sql
-- services/monitoring/database/stats_queries.sql

-- Статистика по таблицам
SELECT 
    schemaname,
    tablename,
    n_tup_ins as inserts,
    n_tup_upd as updates,
    n_tup_del as deletes,
    n_live_tup as live_tuples,
    n_dead_tup as dead_tuples,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
ORDER BY n_tup_ins + n_tup_upd + n_tup_del DESC;

-- Размеры таблиц и индексов
SELECT 
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
    pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) as table_size,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) - 
                   pg_relation_size(schemaname||'.'||tablename)) as index_size
FROM pg_tables 
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Медленные запросы (требует pg_stat_statements)
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

SELECT 
    query,
    calls,
    total_time,
    mean_time,
    rows,
    100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
FROM pg_stat_statements 
ORDER BY mean_time DESC 
LIMIT 10;

-- Блокировки
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement,
    blocking_activity.query AS current_statement_in_blocking_process
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity 
    ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity 
    ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

### EXPLAIN и оптимизация запросов

```sql
-- Анализ плана выполнения
EXPLAIN (ANALYZE, BUFFERS, VERBOSE) 
SELECT u.name, u.email, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.status = 'active'
  AND u.created_at >= CURRENT_DATE - INTERVAL '1 year'
GROUP BY u.id, u.name, u.email
HAVING COUNT(o.id) > 5
ORDER BY order_count DESC;

-- Автоматическое объяснение медленных запросов
-- postgresql.conf
-- auto_explain.log_min_duration = 1000ms
-- auto_explain.log_analyze = on
-- auto_explain.log_buffers = on
-- auto_explain.log_timing = on
-- auto_explain.log_verbose = on

-- Поиск missing индексов
SELECT 
    schemaname,
    tablename,
    seq_scan,
    seq_tup_read,
    seq_tup_read / seq_scan as avg_seq_tup_read
FROM pg_stat_user_tables
WHERE seq_scan > 0
ORDER BY seq_tup_read DESC;
```

---

## 🔧 Расширения и продвинутые возможности

### Полезные расширения

```sql
-- Часто используемые расширения
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";     -- UUID генерация
CREATE EXTENSION IF NOT EXISTS "pgcrypto";      -- Криптографические функции
CREATE EXTENSION IF NOT EXISTS "hstore";        -- Key-value хранение
CREATE EXTENSION IF NOT EXISTS "ltree";         -- Иерархические данные
CREATE EXTENSION IF NOT EXISTS "pg_trgm";       -- Триграммы для поиска
CREATE EXTENSION IF NOT EXISTS "btree_gin";     -- GIN индексы для B-tree типов
CREATE EXTENSION IF NOT EXISTS "pg_stat_statements"; -- Статистика запросов

-- PostGIS для геоданных
CREATE EXTENSION IF NOT EXISTS "postgis";

-- Создание таблицы с геоданными
CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    location GEOMETRY(POINT, 4326),
    area GEOMETRY(POLYGON, 4326)
);

-- Пространственные индексы
CREATE INDEX idx_locations_location ON locations USING GIST (location);
CREATE INDEX idx_locations_area ON locations USING GIST (area);

-- Геопространственные запросы
SELECT name, ST_Distance(location, ST_MakePoint(-122.4194, 37.7749)) as distance
FROM locations
WHERE ST_DWithin(location, ST_MakePoint(-122.4194, 37.7749), 1000)  -- 1км
ORDER BY distance;
```

### Пользовательские функции

```sql
-- services/utils/database/custom_functions.sql

-- Функция для валидации email
CREATE OR REPLACE FUNCTION is_valid_email(email TEXT)
RETURNS BOOLEAN AS $$
BEGIN
    RETURN email ~* '^[A-Za-z0-9._%-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,4}$';
END;
$$ LANGUAGE plpgsql IMMUTABLE;

-- Функция для генерации slug
CREATE OR REPLACE FUNCTION generate_slug(input_text TEXT)
RETURNS TEXT AS $$
BEGIN
    RETURN lower(
        regexp_replace(
            regexp_replace(
                trim(input_text), 
                '[^a-zA-Z0-9\s]', '', 'g'
            ), 
            '\s+', '-', 'g'
        )
    );
END;
$$ LANGUAGE plpgsql IMMUTABLE;

-- Функция для расчета возраста
CREATE OR REPLACE FUNCTION calculate_age(birth_date DATE)
RETURNS INTEGER AS $$
BEGIN
    RETURN EXTRACT(YEAR FROM AGE(CURRENT_DATE, birth_date));
END;
$$ LANGUAGE plpgsql IMMUTABLE;

-- Агрегатная функция для JSON массивов
CREATE OR REPLACE FUNCTION jsonb_array_concat(jsonb, jsonb)
RETURNS jsonb AS $$
BEGIN
    RETURN CASE 
        WHEN $1 IS NULL THEN $2
        WHEN $2 IS NULL THEN $1
        ELSE $1 || $2
    END;
END;
$$ LANGUAGE plpgsql IMMUTABLE;

CREATE AGGREGATE jsonb_array_agg(jsonb) (
    SFUNC = jsonb_array_concat,
    STYPE = jsonb,
    INITCOND = '[]'
);
```

---

## 🔗 Связанные темы

### 📚 База данных знаний
- **[[SQL оптимизация]]** - продвинутые техники оптимизации
- **[[Database Design]]** - принципы проектирования схем
- **[[Транзакции и ACID]]** - теория транзакций
- **[[Репликация баз данных]]** - высокая доступность

### 🛠️ Практические инструменты
- **[[pgAdmin]]** - графический интерфейс
- **[[pg_dump и pg_restore]]** - резервное копирование
- **[[pg_stat_statements]]** - мониторинг запросов
- **[[PostGIS]]** - геопространственные данные

### 🏗️ Архитектурные паттерны
- **[[Database per Service]]** - микросервисная архитектура
- **[[CQRS]]** - разделение команд и запросов
- **[[Event Sourcing]]** - событийное хранение

---

## 🎓 Практические задания

### 📝 Начальный уровень
1. **Основы PostgreSQL**
   - [ ] Создать схему с пользователями и заказами
   - [ ] Настроить индексы для частых запросов
   - [ ] Реализовать триггеры для аудита

2. **JSONB и поиск**
   - [ ] Работа с JSONB полями
   - [ ] Настройка полнотекстового поиска
   - [ ] Оптимизация JSON запросов

### 🔧 Средний уровень
3. **Продвинутые возможности**
   - [ ] Партиционирование больших таблиц
   - [ ] Window functions для аналитики
   - [ ] Пользовательские функции и типы

4. **Высокая доступность**
   - [ ] Настройка streaming репликации
   - [ ] Мониторинг и алерты
   - [ ] Процедуры восстановления

### 🚀 Продвинутый уровень
5. **Enterprise решения**
   - [ ] Автоматическое масштабирование
   - [ ] Оптимизация производительности
   - [ ] Интеграция с Kubernetes

---

*"PostgreSQL - это не просто база данных, это платформа для построения надежных, масштабируемых и высокопроизводительных систем"* 🐘💾 