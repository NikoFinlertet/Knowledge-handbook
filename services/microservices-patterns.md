# 🎭 Microservices Patterns

## 📝 Описание

Архитектурные паттерны для проектирования и реализации микросервисной архитектуры, покрывающие основные аспекты построения распределенных систем.

---

## 🏗️ Декомпозиция

### 1. Decompose by Business Capability

**Проблема:** Как разделить приложение на сервисы?

**Решение:** Разделение по бизнес-возможностям организации.

```mermaid
graph TB
    subgraph "E-commerce Domain"
        UserMgmt[👤 User Management]
        ProductCatalog[📦 Product Catalog]
        OrderMgmt[🛒 Order Management]
        Payment[💳 Payment Processing]
        Shipping[🚚 Shipping & Delivery]
        Inventory[📊 Inventory Management]
    end
    
    UserMgmt -.-> OrderMgmt
    ProductCatalog -.-> OrderMgmt
    OrderMgmt -.-> Payment
    OrderMgmt -.-> Inventory
    Payment -.-> Shipping
```

**Преимущества:**
- Команды независимы по бизнес-доменам
- Сервисы эволюционируют с разной скоростью
- Четкие границы ответственности

### 2. Decompose by Subdomain (DDD)

**Проблема:** Сложность определения бизнес-возможностей.

**Решение:** Использование Domain-Driven Design для выделения субдоменов.

```
Core Subdomains:
├── Order Management (главный домен)
├── Payment Processing (критичный)
└── User Authentication (основной)

Supporting Subdomains:
├── Product Catalog
├── Inventory Management
└── Shipping

Generic Subdomains:
├── Notifications
├── Logging
└── Monitoring
```

---

## 🚪 Communication Patterns

### 1. API Gateway Pattern

**Проблема:** Клиенты взаимодействуют с множеством сервисов.

**Решение:** Единая точка входа для всех клиентских запросов.

```mermaid
graph TB
    Client[📱 Mobile Client] --> Gateway[🚪 API Gateway]
    WebApp[🌐 Web App] --> Gateway
    ThirdParty[🔗 3rd Party] --> Gateway
    
    Gateway --> Auth[🔐 Auth Service]
    Gateway --> Users[👤 User Service]
    Gateway --> Orders[📦 Order Service]
    Gateway --> Products[🛍️ Product Service]
    
    Gateway --> Cache[(🗄️ Cache)]
    Gateway --> RateLimit[⚡ Rate Limiter]
```

**Responsibilities:**
- Routing и load balancing
- Authentication и authorization
- Rate limiting и throttling
- Request/response transformation
- Monitoring и logging

### 2. Backend for Frontend (BFF)

**Проблема:** Разные клиенты нуждаются в разных данных.

**Решение:** Отдельный backend для каждого типа клиента.

```mermaid
graph TB
    Mobile[📱 Mobile App] --> MobileBFF[📱 Mobile BFF]
    Web[🌐 Web App] --> WebBFF[🌐 Web BFF]
    Admin[⚙️ Admin Panel] --> AdminBFF[⚙️ Admin BFF]
    
    MobileBFF --> Services[🔧 Core Services]
    WebBFF --> Services
    AdminBFF --> Services
    
    subgraph Services[Core Services]
        UserSvc[👤 Users]
        OrderSvc[📦 Orders]
        ProductSvc[🛍️ Products]
    end
```

**Преимущества:**
- Оптимизация под каждый клиент
- Независимая эволюция интерфейсов
- Reduced coupling между UI и backend

---

## 📡 Messaging Patterns

### 1. Asynchronous Messaging

**Проблема:** Синхронная коммуникация создает tight coupling.

**Решение:** Асинхронный обмен сообщениями через message broker.

```mermaid
sequenceDiagram
    participant Order as Order Service
    participant Broker as Message Broker
    participant Payment as Payment Service
    participant Inventory as Inventory Service
    participant Notification as Notification Service
    
    Order->>Broker: OrderCreated Event
    Broker->>Payment: Process Payment
    Broker->>Inventory: Reserve Items
    Broker->>Notification: Send Confirmation
    
    Payment->>Broker: PaymentProcessed Event
    Inventory->>Broker: ItemsReserved Event
    
    Broker->>Order: Update Order Status
```

**Message Types:**
- **Commands** - действие, которое должно быть выполнено
- **Events** - уведомление о произошедшем событии
- **Queries** - запрос данных

### 2. Saga Pattern

**Проблема:** Управление транзакциями через несколько сервисов.

**Решение:** Sequence of local transactions с compensation logic.

```mermaid
graph TB
    subgraph "Order Saga"
        Start([Order Created]) --> Reserve[Reserve Items]
        Reserve --> |Success| ProcessPayment[Process Payment]
        Reserve --> |Failure| CancelOrder[Cancel Order]
        
        ProcessPayment --> |Success| Ship[Ship Order]
        ProcessPayment --> |Failure| ReleaseItems[Release Items]
        
        Ship --> Complete([Order Complete])
        ReleaseItems --> CancelOrder
        CancelOrder --> Failed([Order Failed])
    end
```

**Types:**
- **Choreography** - каждый сервис знает что делать
- **Orchestration** - центральный координатор управляет процессом

---

## 🔒 Data Management Patterns

### 1. Database per Service

**Проблема:** Shared database создает coupling между сервисами.

**Решение:** Каждый сервис имеет свою приватную базу данных.

```mermaid
graph TB
    subgraph "User Service"
        UserService[User Service] --> UserDB[(👥 User DB<br/>PostgreSQL)]
    end
    
    subgraph "Order Service"  
        OrderService[Order Service] --> OrderDB[(📦 Order DB<br/>PostgreSQL)]
    end
    
    subgraph "Product Service"
        ProductService[Product Service] --> ProductDB[(🛍️ Product DB<br/>MongoDB)]
    end
    
    subgraph "Analytics Service"
        AnalyticsService[Analytics Service] --> AnalyticsDB[(📊 Analytics DB<br/>ClickHouse)]
    end
```

**Benefits:**
- Технологическая гетерогенность
- Независимое масштабирование
- Isolation failures
- Team autonomy

### 2. Event Sourcing

**Проблема:** Потеря истории изменений данных.

**Решение:** Хранение последовательности событий как источника истины.

```mermaid
graph LR
    Events[📚 Event Store] --> |Replay| Aggregate[🎯 Current State]
    Commands[⚡ Commands] --> Aggregate
    Aggregate --> |New Events| Events
    
    Events --> Projections[📊 Read Models]
    Projections --> Queries[🔍 Queries]
```

**Advantages:**
- Complete audit trail
- Temporal queries
- Replay events for debugging
- Multiple read models

### 3. CQRS (Command Query Responsibility Segregation)

**Проблема:** Разные требования к чтению и записи данных.

**Решение:** Separate models для команд и запросов.

```mermaid
graph TB
    Commands[⚡ Commands] --> CommandModel[📝 Write Model]
    CommandModel --> EventStore[(📚 Event Store)]
    
    EventStore --> Projections[🔄 Event Handlers]
    Projections --> ReadModel1[(📊 Read Model 1)]
    Projections --> ReadModel2[(📊 Read Model 2)]
    Projections --> ReadModel3[(📊 Read Model 3)]
    
    Queries[🔍 Queries] --> ReadModel1
    Queries --> ReadModel2
    Queries --> ReadModel3
```

---

## 🛡️ Reliability Patterns

### 1. Circuit Breaker

**Проблема:** Cascade failures в распределенной системе.

**Решение:** Automatic failure detection с fallback механизмом.

```mermaid
stateDiagram-v2
    [*] --> Closed
    Closed --> Open : Failure threshold reached
    Open --> HalfOpen : Timeout elapsed
    HalfOpen --> Closed : Success
    HalfOpen --> Open : Failure
    
    note right of Closed
        Normal operation
        Requests pass through
    end note
    
    note right of Open
        Fail fast
        Return cached data
        or error response
    end note
    
    note right of HalfOpen
        Trial period
        Single request allowed
    end note
```

**Implementation:**
```typescript
class CircuitBreaker {
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED'
  private failures = 0
  private nextAttempt = 0
  
  async call<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'OPEN' && Date.now() < this.nextAttempt) {
      throw new Error('Circuit breaker is OPEN')
    }
    
    try {
      const result = await fn()
      this.onSuccess()
      return result
    } catch (error) {
      this.onFailure()
      throw error
    }
  }
}
```

### 2. Retry Pattern

**Проблема:** Temporary failures в network calls.

**Решение:** Automatic retry с exponential backoff.

```typescript
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  baseDelay: number = 1000
): Promise<T> {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn()
    } catch (error) {
      if (attempt === maxRetries) throw error
      
      const delay = baseDelay * Math.pow(2, attempt)
      const jitter = Math.random() * 0.1 * delay
      
      await sleep(delay + jitter)
    }
  }
}
```

### 3. Bulkhead Pattern

**Проблема:** Resource exhaustion affects всю систему.

**Решение:** Isolation resources в separate pools.

```mermaid
graph TB
    subgraph "Application Server"
        subgraph "Critical Pool"
            CriticalConn[Critical Connections<br/>Pool Size: 20]
        end
        
        subgraph "Standard Pool"
            StandardConn[Standard Connections<br/>Pool Size: 50]
        end
        
        subgraph "Background Pool"
            BackgroundConn[Background Connections<br/>Pool Size: 10]
        end
    end
    
    CriticalAPI[🔥 Critical API] --> CriticalConn
    StandardAPI[📝 Standard API] --> StandardConn
    ReportsAPI[📊 Reports API] --> BackgroundConn
```

---

## 🔍 Observability Patterns

### 1. Distributed Tracing

**Проблема:** Debugging requests через multiple services.

**Решение:** Trace ID propagation через все сервисы.

```mermaid
sequenceDiagram
    participant Client
    participant Gateway as API Gateway
    participant User as User Service
    participant Order as Order Service
    participant Payment as Payment Service
    
    Note over Client: Trace ID: abc123
    
    Client->>Gateway: Request (Trace: abc123)
    Gateway->>User: Get User (Trace: abc123, Span: user-lookup)
    User-->>Gateway: User Data
    
    Gateway->>Order: Create Order (Trace: abc123, Span: order-create)
    Order->>Payment: Process Payment (Trace: abc123, Span: payment-process)
    Payment-->>Order: Payment Result
    Order-->>Gateway: Order Created
    
    Gateway-->>Client: Response (Trace: abc123)
```

### 2. Health Check API

**Проблема:** Мониторинг состояния сервисов.

**Решение:** Standardized health check endpoints.

```json
// GET /health
{
  "status": "healthy",
  "version": "1.2.3",
  "checks": {
    "database": {
      "status": "healthy",
      "responseTime": "2ms"
    },
    "redis": {
      "status": "healthy", 
      "responseTime": "1ms"
    },
    "external_api": {
      "status": "degraded",
      "responseTime": "500ms",
      "message": "Slow response times"
    }
  },
  "dependencies": [
    {
      "name": "user-service",
      "status": "healthy",
      "url": "http://user-service:8080/health"
    }
  ]
}
```

### 3. Centralized Logging

**Проблема:** Logs scattered across множество сервисов.

**Решение:** Structured logging в centralized log aggregation.

```mermaid
graph TB
    subgraph "Services"
        UserSvc[👤 User Service] --> Logstash[📋 Logstash]
        OrderSvc[📦 Order Service] --> Logstash
        PaymentSvc[💳 Payment Service] --> Logstash
    end
    
    Logstash --> Elasticsearch[(🔍 Elasticsearch)]
    Elasticsearch --> Kibana[📊 Kibana Dashboard]
    
    subgraph "Log Structure"
        Structure["{<br/>  timestamp,<br/>  level,<br/>  service,<br/>  traceId,<br/>  message,<br/>  metadata<br/>}"]
    end
```

---

## 🔧 Testing Patterns

### 1. Consumer-Driven Contract Testing

**Проблема:** Breaking changes между producer и consumer.

**Решение:** Contracts defined by consumers, verified by producers.

```mermaid
graph LR
    Consumer[👤 User Service<br/>Consumer] --> Contract[📄 Contract<br/>user-profile.json]
    Contract --> Producer[📦 Profile Service<br/>Producer]
    
    Contract --> Tests[🧪 Contract Tests]
    Tests --> |Verify| Producer
```

### 2. Test Pyramid для Microservices

```mermaid
pyramid
    title Microservice Test Pyramid
    
    "E2E Tests" : 5
    "Contract Tests" : 15
    "Integration Tests" : 25
    "Unit Tests" : 55
```

**Levels:**
- **Unit Tests** - business logic, domain models
- **Integration Tests** - database, external APIs
- **Contract Tests** - service interfaces
- **E2E Tests** - complete user journeys

---

## 🚀 Deployment Patterns

### 1. Blue-Green Deployment

**Проблема:** Zero-downtime deployments.

**Решение:** Two identical production environments.

```mermaid
graph TB
    LoadBalancer[⚖️ Load Balancer] --> |100% Traffic| Blue[🔵 Blue Environment<br/>Version 1.0]
    LoadBalancer -.-> |0% Traffic| Green[🟢 Green Environment<br/>Version 1.1]
    
    Blue --> BlueDB[(💾 Shared Database)]
    Green --> BlueDB
```

**Process:**
1. Deploy new version to Green
2. Test Green environment
3. Switch traffic to Green
4. Blue становится staging для следующего release

### 2. Canary Deployment

**Проблема:** Risk mitigation при новых releases.

**Решение:** Gradual rollout к subset пользователей.

```mermaid
graph TB
    LoadBalancer[⚖️ Load Balancer] --> |95% Traffic| Stable[✅ Stable Version<br/>v1.0]
    LoadBalancer --> |5% Traffic| Canary[🐦 Canary Version<br/>v1.1]
    
    Canary --> Monitoring[📊 Monitoring<br/>Error rates, Performance]
    Monitoring --> |Success| RolloutMore[📈 Increase Traffic]
    Monitoring --> |Failure| Rollback[⏪ Rollback]
```

---

## 📚 Связанные темы

- [[../fundamentals/ddd-patterns|DDD Patterns]]
- [[../fundamentals/cqrs-pattern|CQRS Pattern]]
- [[../fundamentals/event-sourcing|Event Sourcing]]
- [[../architecture/microservices-architecture|Microservices Architecture]]
- [[../technical-skills/testing|Testing Strategies]]
- [[user-service|User Service]] - DDD + CQRS example
- [[order-service|Order Service]] - Event Sourcing example
- [[api-gateway|API Gateway]] - Gateway pattern example

---

## 🎯 Best Practices

### Design Principles
- **Single Responsibility** - один сервис, одна задача
- **Loose Coupling** - минимальные зависимости
- **High Cohesion** - связанная функциональность вместе
- **Autonomous Teams** - team ownership сервисов

### Communication Guidelines
- **Async by default** - prefer events over direct calls
- **Idempotent operations** - safe retry mechanisms
- **Backward compatibility** - версионирование APIs
- **Circuit breakers** - fail fast, graceful degradation

### Data Management
- **Database per service** - data isolation
- **Eventual consistency** - accept временную inconsistency
- **Saga patterns** - distributed transaction management
- **Event sourcing** - audit trail и temporal queries 