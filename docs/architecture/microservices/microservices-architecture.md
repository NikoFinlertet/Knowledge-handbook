# Микросервисная архитектура

**Микросервисная архитектура** — это подход к разработке программного обеспечения, при котором приложение структурируется как набор слабо связанных сервисов. Каждый сервис является независимым, развертываемым и масштабируемым компонентом, который взаимодействует с другими сервисами через четко определенные API.

## Основные принципы

### 1. Принцип единственной ответственности
Каждый микросервис должен иметь одну четко определенную бизнес-функцию.

### 2. Автономность
Сервисы должны быть независимы в разработке, развертывании и эксплуатации.

### 3. Децентрализация
Отсутствие центрального управления данными и бизнес-логикой.

### 4. Отказоустойчивость
Система должна продолжать работать даже при отказе отдельных сервисов.

### 5. Эволюционность
Возможность независимой эволюции и изменения сервисов.

## Сравнение с монолитной архитектурой

### Монолитная архитектура
```
┌─────────────────────────────────────────┐
│            Монолитное приложение        │
├─────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐      │
│  │    Users    │  │   Orders    │      │
│  │   Module    │  │   Module    │      │
│  └─────────────┘  └─────────────┘      │
│  ┌─────────────┐  ┌─────────────┐      │
│  │  Products   │  │   Payment   │      │
│  │   Module    │  │   Module    │      │
│  └─────────────┘  └─────────────┘      │
├─────────────────────────────────────────┤
│         Единая база данных              │
└─────────────────────────────────────────┘
```

### Микросервисная архитектура
```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   User      │  │   Order     │  │   Payment   │
│  Service    │  │  Service    │  │  Service    │
├─────────────┤  ├─────────────┤  ├─────────────┤
│   User DB   │  │  Order DB   │  │ Payment DB  │
└─────────────┘  └─────────────┘  └─────────────┘
       │               │               │
       └───────────────┼───────────────┘
                       │
          ┌─────────────────────────┐
          │     API Gateway         │
          └─────────────────────────┘
```

## Компоненты микросервисной архитектуры

### 1. API Gateway

**API Gateway** — это единая точка входа для всех клиентских запросов.

**Функции:**
- Маршрутизация запросов
- Аутентификация и авторизация
- Ограничение скорости (Rate Limiting)
- Мониторинг и логирование
- Трансформация данных

**Применение в проекте:**
```javascript
// services/api-gateway/src/index.js
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');
const { buildSchema } = require('graphql');
const { graphqlHTTP } = require('express-graphql');

const app = express();

// Service Discovery
const serviceDiscovery = {
  async getService(serviceName) {
    const services = {
      'user-service': { url: 'http://user-service:8081' },
      'order-service': { url: 'http://order-service:8082' }
    };
    return services[serviceName];
  }
};

// REST API Proxy
app.use('/api/v1/users', createProxyMiddleware({
  target: 'http://user-service:8081',
  changeOrigin: true,
  pathRewrite: {
    '^/api/v1/users': '/api/v1/users'
  }
}));

app.use('/api/v1/orders', createProxyMiddleware({
  target: 'http://order-service:8082',
  changeOrigin: true,
  pathRewrite: {
    '^/api/v1/orders': '/api/v1/orders'
  }
}));

// GraphQL Gateway
const schema = buildSchema(`
  type User {
    id: ID!
    email: String!
    name: String!
    status: String!
  }
  
  type Order {
    id: ID!
    userId: String!
    items: [OrderItem!]!
    status: String!
    totalAmount: Float!
  }
  
  type OrderItem {
    productId: String!
    quantity: Int!
    price: Float!
  }
  
  type Query {
    user(id: ID!): User
    users: [User!]!
    order(id: ID!): Order
    userOrders(userId: ID!): [Order!]!
  }
  
  type Mutation {
    createUser(email: String!, name: String!): User
    createOrder(userId: ID!): Order
    addItemToOrder(orderId: ID!, productId: String!, quantity: Int!, price: Float!): Order
  }
`);

app.use('/graphql', graphqlHTTP({
  schema: schema,
  rootValue: createResolvers(serviceDiscovery),
  graphiql: true
}));

app.listen(8080, () => {
  console.log('API Gateway listening on port 8080');
});
```

### 2. Service Discovery

**Service Discovery** — это механизм для автоматического обнаружения и подключения к сервисам.

**Типы:**
- **Client-side discovery** - клиент сам находит сервис
- **Server-side discovery** - прокси/балансировщик находит сервис

**Применение в проекте:**
```javascript
// services/api-gateway/src/index.js
class ServiceDiscovery {
  constructor() {
    this.services = new Map();
    this.healthChecks = new Map();
  }
  
  async registerService(name, url, healthCheckUrl) {
    this.services.set(name, { url, healthCheckUrl });
    this.startHealthCheck(name);
  }
  
  async getService(name) {
    const service = this.services.get(name);
    if (!service) {
      throw new Error(`Service ${name} not found`);
    }
    
    if (!this.isHealthy(name)) {
      throw new Error(`Service ${name} is unhealthy`);
    }
    
    return service;
  }
  
  async startHealthCheck(name) {
    const service = this.services.get(name);
    if (!service) return;
    
    const checkHealth = async () => {
      try {
        const response = await fetch(`${service.url}/health`);
        this.healthChecks.set(name, response.ok);
      } catch (error) {
        this.healthChecks.set(name, false);
      }
    };
    
    checkHealth();
    setInterval(checkHealth, 30000); // Проверка каждые 30 секунд
  }
  
  isHealthy(name) {
    return this.healthChecks.get(name) || false;
  }
}
```

### 3. Circuit Breaker

**Circuit Breaker** — это паттерн для предотвращения каскадных отказов.

**Состояния:**
- **Closed** - нормальная работа
- **Open** - блокировка запросов
- **Half-Open** - пробные запросы

```javascript
// services/api-gateway/src/circuit-breaker.js
class CircuitBreaker {
  constructor(threshold = 5, timeout = 60000) {
    this.threshold = threshold;
    this.timeout = timeout;
    this.failureCount = 0;
    this.lastFailureTime = null;
    this.state = 'CLOSED';
  }
  
  async execute(operation) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.timeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }
    
    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }
  
  onFailure() {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    
    if (this.failureCount >= this.threshold) {
      this.state = 'OPEN';
    }
  }
}
```

### 4. Load Balancer

**Load Balancer** — это компонент для распределения нагрузки между экземплярами сервиса.

**Алгоритмы:**
- **Round Robin** - поочередно
- **Least Connections** - на сервер с наименьшим количеством соединений
- **Weighted Round Robin** - с учетом весов

```javascript
// services/api-gateway/src/load-balancer.js
class LoadBalancer {
  constructor(strategy = 'round-robin') {
    this.strategy = strategy;
    this.servers = new Map();
    this.currentIndex = 0;
  }
  
  addServer(serviceName, serverUrl) {
    if (!this.servers.has(serviceName)) {
      this.servers.set(serviceName, []);
    }
    this.servers.get(serviceName).push({
      url: serverUrl,
      connections: 0,
      weight: 1
    });
  }
  
  getServer(serviceName) {
    const servers = this.servers.get(serviceName);
    if (!servers || servers.length === 0) {
      throw new Error(`No servers available for ${serviceName}`);
    }
    
    switch (this.strategy) {
      case 'round-robin':
        return this.roundRobin(servers);
      case 'least-connections':
        return this.leastConnections(servers);
      default:
        return this.roundRobin(servers);
    }
  }
  
  roundRobin(servers) {
    const server = servers[this.currentIndex % servers.length];
    this.currentIndex++;
    return server;
  }
  
  leastConnections(servers) {
    return servers.reduce((min, server) => 
      server.connections < min.connections ? server : min
    );
  }
}
```

## Паттерны межсервисного взаимодействия

### 1. Synchronous Communication

#### REST API
```go
// services/user-service/internal/api/rest_handler.go
type RestHandler struct {
    commandHandler *application.CommandHandler
    queryHandler   *application.QueryHandler
}

func (h *RestHandler) CreateUser(c *gin.Context) {
    var req dto.CreateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    
    cmd := application.CreateUserCommand{
        Email: req.Email,
        Name:  req.Name,
    }
    
    if err := h.commandHandler.HandleCreateUser(cmd); err != nil {
        c.JSON(500, gin.H{"error": err.Error()})
        return
    }
    
    c.JSON(201, gin.H{"message": "User created successfully"})
}
```

#### gRPC
```go
// services/user-service/internal/api/grpc_handler.go
type GrpcHandler struct {
    commandHandler *application.CommandHandler
    queryHandler   *application.QueryHandler
}

func (h *GrpcHandler) CreateUser(ctx context.Context, req *pb.CreateUserRequest) (*pb.CreateUserResponse, error) {
    cmd := application.CreateUserCommand{
        Email: req.Email,
        Name:  req.Name,
    }
    
    if err := h.commandHandler.HandleCreateUser(cmd); err != nil {
        return nil, status.Errorf(codes.Internal, "Failed to create user: %v", err)
    }
    
    return &pb.CreateUserResponse{
        Message: "User created successfully",
    }, nil
}
```

### 2. Asynchronous Communication

#### Message Broker
```go
// services/user-service/internal/infrastructure/event_bus.go
type EventBus interface {
    Publish(topic string, event interface{}) error
    Subscribe(topic string, handler func(event interface{})) error
}

type RabbitMQEventBus struct {
    connection *amqp.Connection
    channel    *amqp.Channel
}

func (b *RabbitMQEventBus) Publish(topic string, event interface{}) error {
    body, err := json.Marshal(event)
    if err != nil {
        return err
    }
    
    return b.channel.Publish(
        "",    // exchange
        topic, // routing key
        false, // mandatory
        false, // immediate
        amqp.Publishing{
            ContentType: "application/json",
            Body:        body,
        },
    )
}

func (b *RabbitMQEventBus) Subscribe(topic string, handler func(event interface{})) error {
    msgs, err := b.channel.Consume(
        topic, // queue
        "",    // consumer
        true,  // auto-ack
        false, // exclusive
        false, // no-local
        false, // no-wait
        nil,   // args
    )
    if err != nil {
        return err
    }
    
    go func() {
        for msg := range msgs {
            var event interface{}
            if err := json.Unmarshal(msg.Body, &event); err == nil {
                handler(event)
            }
        }
    }()
    
    return nil
}
```

### 3. Event-Driven Architecture

**Применение в проекте:**
```go
// services/user-service/internal/application/commands.go
func (h *CommandHandler) HandleCreateUser(cmd CreateUserCommand) error {
    user, err := domain.NewUser(cmd.Email, cmd.Name)
    if err != nil {
        return err
    }
    
    if err := h.userRepo.Save(user); err != nil {
        return err
    }
    
    // Публикация события
    event := domain.UserCreatedEvent{
        UserID:    user.ID(),
        Email:     user.Email(),
        Name:      user.Name(),
        Timestamp: user.CreatedAt(),
    }
    
    return h.eventBus.Publish("user.created", event)
}
```

```python
# services/order-service/main.py
import asyncio
import aio_pika
from .domain.order import Order

class OrderService:
    def __init__(self, connection):
        self.connection = connection
    
    async def handle_user_created(self, event):
        """Обработка события создания пользователя"""
        print(f"User created: {event['user_id']}")
        # Можно создать профиль пользователя в контексте заказов
    
    async def start_consuming(self):
        """Запуск потребления событий"""
        channel = await self.connection.channel()
        
        # Создание очереди для событий пользователей
        queue = await channel.declare_queue("order-service.user-events")
        
        # Подписка на события
        await queue.consume(self.handle_user_created)
```

## Управление данными

### 1. Database per Service

**Принцип:** Каждый сервис имеет свою собственную базу данных.

**Применение в проекте:**
```yaml
# docker-compose.yml
services:
  user-service:
    image: user-service:latest
    environment:
      - DB_HOST=user-db
      - DB_NAME=users
  
  user-db:
    image: postgres:13
    environment:
      - POSTGRES_DB=users
      - POSTGRES_USER=user_service
      - POSTGRES_PASSWORD=password
  
  order-service:
    image: order-service:latest
    environment:
      - DB_HOST=order-db
      - DB_NAME=orders
  
  order-db:
    image: postgres:13
    environment:
      - POSTGRES_DB=orders
      - POSTGRES_USER=order_service
      - POSTGRES_PASSWORD=password
```

### 2. Saga Pattern

**Saga** — это паттерн для управления распределенными транзакциями.

**Типы:**
- **Choreography** - каждый сервис знает, что делать дальше
- **Orchestration** - центральный координатор управляет процессом

**Choreography Saga:**
```go
// services/user-service/internal/application/commands.go
func (h *CommandHandler) HandleCreateUser(cmd CreateUserCommand) error {
    // Создание пользователя
    user, err := domain.NewUser(cmd.Email, cmd.Name)
    if err != nil {
        return err
    }
    
    if err := h.userRepo.Save(user); err != nil {
        return err
    }
    
    // Публикация события - запуск Saga
    event := domain.UserCreatedEvent{
        UserID:    user.ID(),
        Email:     user.Email(),
        Name:      user.Name(),
        Timestamp: user.CreatedAt(),
    }
    
    return h.eventBus.Publish("user.created", event)
}
```

```python
# services/order-service/main.py
async def handle_user_created(event):
    """Обработка события создания пользователя"""
    try:
        # Создание профиля пользователя в контексте заказов
        profile = UserProfile(
            user_id=event['user_id'],
            email=event['email'],
            name=event['name']
        )
        await profile_repository.save(profile)
        
        # Публикация события об успешном создании профиля
        success_event = UserProfileCreatedEvent(
            user_id=event['user_id'],
            timestamp=datetime.utcnow()
        )
        await event_bus.publish("user.profile.created", success_event)
        
    except Exception as e:
        # Публикация события об ошибке
        error_event = UserProfileCreationFailedEvent(
            user_id=event['user_id'],
            error=str(e),
            timestamp=datetime.utcnow()
        )
        await event_bus.publish("user.profile.creation.failed", error_event)
```

### 3. Event Sourcing

Применение Event Sourcing в микросервисах обеспечивает естественную согласованность данных.

```python
# services/order-service/domain/order.py
class Order:
    def __init__(self, order_id: str, user_id: str):
        self.order_id = order_id
        self.user_id = user_id
        self.items = []
        self.status = OrderStatus.PENDING
        self.total_amount = 0.0
        self.created_at = datetime.utcnow()
        self.updated_at = datetime.utcnow()
        self.version = 0
        self.uncommitted_events = []
        
        # Генерация события создания заказа
        event = OrderCreatedEvent(
            order_id=self.order_id,
            user_id=self.user_id,
            timestamp=self.created_at,
            version=self.version
        )
        self.uncommitted_events.append(event)
        
        # Публикация события для других сервисов
        integration_event = OrderCreatedIntegrationEvent(
            order_id=self.order_id,
            user_id=self.user_id,
            timestamp=self.created_at
        )
        self.uncommitted_events.append(integration_event)
```

## Тестирование микросервисов

### 1. Unit Tests
```go
// services/user-service/internal/application/commands_test.go
func TestCreateUserCommand(t *testing.T) {
    // Arrange
    mockRepo := &MockUserRepository{}
    mockEventBus := &MockEventBus{}
    handler := &CommandHandler{
        userRepo: mockRepo,
        eventBus: mockEventBus,
    }
    
    cmd := CreateUserCommand{
        Email: "test@example.com",
        Name:  "Test User",
    }
    
    // Act
    err := handler.HandleCreateUser(cmd)
    
    // Assert
    assert.NoError(t, err)
    assert.Equal(t, 1, mockRepo.SaveCallCount)
    assert.Equal(t, 1, mockEventBus.PublishCallCount)
}
```

### 2. Integration Tests
```go
// services/user-service/tests/integration_test.go
func TestCreateUserIntegration(t *testing.T) {
    // Arrange
    db := setupTestDatabase()
    eventBus := setupTestEventBus()
    
    repo := infrastructure.NewPostgresUserRepository(db)
    handler := application.NewCommandHandler(repo, eventBus)
    
    cmd := application.CreateUserCommand{
        Email: "integration@example.com",
        Name:  "Integration Test",
    }
    
    // Act
    err := handler.HandleCreateUser(cmd)
    
    // Assert
    assert.NoError(t, err)
    
    // Verify database state
    users, err := repo.FindAll()
    assert.NoError(t, err)
    assert.Len(t, users, 1)
    assert.Equal(t, "integration@example.com", users[0].Email())
}
```

### 3. Contract Tests
```go
// services/user-service/tests/contract_test.go
func TestUserServiceContract(t *testing.T) {
    // Arrange
    server := setupTestServer()
    defer server.Close()
    
    // Act
    resp, err := http.Post(
        server.URL+"/api/v1/users",
        "application/json",
        strings.NewReader(`{"email":"contract@example.com","name":"Contract Test"}`),
    )
    
    // Assert
    assert.NoError(t, err)
    assert.Equal(t, http.StatusCreated, resp.StatusCode)
    
    var result map[string]interface{}
    json.NewDecoder(resp.Body).Decode(&result)
    assert.Equal(t, "User created successfully", result["message"])
}
```

### 4. End-to-End Tests
```javascript
// tests/e2e/user-order-flow.test.js
describe('User Order Flow', () => {
  test('should create user and place order', async () => {
    // Create user
    const userResponse = await request(apiGateway)
      .post('/api/v1/users')
      .send({
        email: 'e2e@example.com',
        name: 'E2E Test User'
      });
    
    expect(userResponse.status).toBe(201);
    
    // Wait for user creation to propagate
    await new Promise(resolve => setTimeout(resolve, 1000));
    
    // Create order
    const orderResponse = await request(apiGateway)
      .post('/api/v1/orders')
      .send({
        userId: userResponse.body.id
      });
    
    expect(orderResponse.status).toBe(201);
    
    // Verify order was created
    const getOrderResponse = await request(apiGateway)
      .get(`/api/v1/orders/${orderResponse.body.id}`);
    
    expect(getOrderResponse.status).toBe(200);
    expect(getOrderResponse.body.userId).toBe(userResponse.body.id);
  });
});
```

## Мониторинг и наблюдаемость

### 1. Health Checks
```go
// services/user-service/internal/api/health_handler.go
func (h *HealthHandler) HealthCheck(c *gin.Context) {
    // Проверка подключения к базе данных
    if err := h.db.Ping(); err != nil {
        c.JSON(503, gin.H{
            "status": "unhealthy",
            "error": "database connection failed",
        })
        return
    }
    
    // Проверка подключения к message broker
    if err := h.eventBus.Ping(); err != nil {
        c.JSON(503, gin.H{
            "status": "unhealthy",
            "error": "message broker connection failed",
        })
        return
    }
    
    c.JSON(200, gin.H{
        "status": "healthy",
        "timestamp": time.Now().UTC(),
        "uptime": time.Since(h.startTime).String(),
    })
}
```

### 2. Distributed Tracing
```go
// services/user-service/internal/middleware/tracing.go
func TracingMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // Извлечение trace ID из заголовков
        traceID := c.GetHeader("X-Trace-ID")
        if traceID == "" {
            traceID = uuid.New().String()
        }
        
        // Установка trace ID в контекст
        c.Set("trace_id", traceID)
        c.Header("X-Trace-ID", traceID)
        
        // Логирование начала запроса
        log.WithField("trace_id", traceID).
            WithField("method", c.Request.Method).
            WithField("path", c.Request.URL.Path).
            Info("Request started")
        
        start := time.Now()
        
        c.Next()
        
        // Логирование завершения запроса
        duration := time.Since(start)
        log.WithField("trace_id", traceID).
            WithField("status", c.Writer.Status()).
            WithField("duration", duration).
            Info("Request completed")
    }
}
```

### 3. Centralized Logging
```go
// services/user-service/internal/infrastructure/logger.go
type StructuredLogger struct {
    logger *logrus.Logger
}

func NewStructuredLogger() *StructuredLogger {
    logger := logrus.New()
    logger.SetFormatter(&logrus.JSONFormatter{})
    
    return &StructuredLogger{logger: logger}
}

func (l *StructuredLogger) LogCommand(cmd interface{}, err error) {
    entry := l.logger.WithFields(logrus.Fields{
        "type": "command",
        "command": reflect.TypeOf(cmd).Name(),
        "timestamp": time.Now().UTC(),
    })
    
    if err != nil {
        entry.WithError(err).Error("Command execution failed")
    } else {
        entry.Info("Command executed successfully")
    }
}

func (l *StructuredLogger) LogQuery(query interface{}, result interface{}, err error) {
    entry := l.logger.WithFields(logrus.Fields{
        "type": "query",
        "query": reflect.TypeOf(query).Name(),
        "timestamp": time.Now().UTC(),
    })
    
    if err != nil {
        entry.WithError(err).Error("Query execution failed")
    } else {
        entry.Info("Query executed successfully")
    }
}
```

## Развертывание и оркестрация

### 1. Docker Containerization
```dockerfile
# services/user-service/Dockerfile
FROM golang:1.19-alpine AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN go build -o user-service ./cmd/main.go

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/

COPY --from=builder /app/user-service .

EXPOSE 8081
CMD ["./user-service"]
```

### 2. Kubernetes Deployment
```yaml
# k8s/user-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
      - name: user-service
        image: user-service:latest
        ports:
        - containerPort: 8081
        env:
        - name: DB_HOST
          value: "user-db"
        - name: DB_NAME
          value: "users"
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: user-db-secret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: user-db-secret
              key: password
        readinessProbe:
          httpGet:
            path: /health
            port: 8081
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 8081
          initialDelaySeconds: 30
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  selector:
    app: user-service
  ports:
  - port: 8081
    targetPort: 8081
  type: ClusterIP
```

### 3. Service Mesh (Istio)
```yaml
# k8s/user-service-virtualservice.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: user-service
spec:
  http:
  - match:
    - uri:
        prefix: /api/v1/users
    route:
    - destination:
        host: user-service
        subset: v1
      weight: 90
    - destination:
        host: user-service
        subset: v2
      weight: 10
  - fault:
      delay:
        percentage:
          value: 0.1
        fixedDelay: 5s
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: user-service
spec:
  host: user-service
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

## Преимущества микросервисной архитектуры

### 1. Масштабируемость
- Независимое масштабирование сервисов
- Оптимизация ресурсов под конкретные нагрузки
- Горизонтальное масштабирование

### 2. Гибкость разработки
- Независимые команды разработки
- Разные технологии для разных сервисов
- Быстрая доставка функций

### 3. Отказоустойчивость
- Изоляция отказов
- Отказ одного сервиса не влияет на всю систему
- Возможность graceful degradation

### 4. Простота развертывания
- Независимые развертывания
- Canary deployments
- Blue-green deployments

## Недостатки микросервисной архитектуры

### 1. Сложность
- Распределенные системы сложны в отладке
- Много moving parts
- Сложность тестирования

### 2. Сетевая задержка
- Межсервисные вызовы по сети
- Latency и throughput
- Необходимость оптимизации коммуникации

### 3. Согласованность данных
- Eventual consistency
- Распределенные транзакции
- Сложность поддержания целостности

### 4. Операционная сложность
- Мониторинг множества сервисов
- Distributed tracing
- Centralized logging

## Когда использовать микросервисы

### Подходящие случаи:
1. **Большие команды** - несколько команд разработки
2. **Сложные бизнес-процессы** - четкое разделение доменов
3. **Высокие требования к масштабируемости** - разные части системы требуют разного масштабирования
4. **Разные технологические требования** - разные сервисы требуют разных технологий
5. **Быстрое развитие** - необходимость быстрой доставки функций

### Неподходящие случаи:
1. **Малые команды** - overhead превышает пользу
2. **Простые приложения** - монолит проще и эффективнее
3. **Низкие требования к масштабируемости** - монолит справляется
4. **Тесно связанные данные** - сложность разделения
5. **Неопределенные границы** - границы сервисов не ясны

## Заключение

Микросервисная архитектура — это мощный подход для построения сложных, масштабируемых систем. Основные принципы:

1. **Разделение по бизнес-функциям** - каждый сервис решает конкретную задачу
2. **Автономность** - независимая разработка, развертывание и масштабирование
3. **Децентрализация** - отсутствие единых точек отказа
4. **Устойчивость к сбоям** - изоляция отказов и graceful degradation

Микросервисы особенно эффективны в сочетании с современными практиками: DevOps, CI/CD, контейнеризация, оркестрация, мониторинг и наблюдаемость.

---

## 🔗 Связанные темы

### 🏗️ Основы
- **[DDD Patterns](../fundamentals/ddd-patterns.md)** - Bounded Context для микросервисов
- **[Event Sourcing](../fundamentals/event-sourcing.md)** - Событийная архитектура между сервисами
- **[CQRS Pattern](../fundamentals/cqrs-pattern.md)** - Разделение ответственности в сервисах

### 🏛️ Архитектурные темы
- **[API Design](api-design.md)** - Проектирование межсервисного взаимодействия
- **[Senior Architecture](senior-architecture.md)** - Архитектурные решения для микросервисов

### ⚙️ Технические навыки
- **[Databases](../technical-skills/databases.md)** - Data per service паттерн
- **[Concurrency & Async](../technical-skills/concurrency-async.md)** - Асинхронная коммуникация
- **[Testing](../technical-skills/testing.md)** - Тестирование распределенных систем
- **[Security](../technical-skills/security.md)** - Безопасность микросервисов

### 🚀 DevOps и инфраструктура
- **[CI/CD & DevOps](../infrastructure/cicd-devops.md)** - Автоматизация для микросервисов
- **[Deployment & Monitoring](../infrastructure/deployment-monitoring.md)** - Мониторинг распределенных систем

### 🎨 Frontend интеграция
- **[Frontend Advanced](../frontend/frontend-advanced.md)** - Micro-frontends архитектура

### 👥 Лидерство
- **[Team Management](../leadership/team-management.md)** - Управление командами микросервисов
- **[Senior Leadership](../leadership/senior-leadership.md)** - Техническое лидерство в микросервисах
- **[Senior Business](../leadership/senior-business.md)** - Бизнес-ценность микросервисов

---

**Путь обучения**: [DDD Patterns](../fundamentals/ddd-patterns.md) → Microservices → [API Design](api-design.md)  
**Сложность**: ⭐⭐⭐⭐⭐ (5/5)  
**Время изучения**: 4-6 недель  
**Практика**: Весь проект (`user-service/`, `order-service/`, `api-gateway/`)