# Дизайн API: REST, GraphQL, gRPC

**API (Application Programming Interface)** — это интерфейс для взаимодействия между различными программными компонентами. В веб-разработке существует несколько основных подходов к проектированию API: REST, GraphQL и gRPC.

## REST API

### Основные принципы REST

**REST (Representational State Transfer)** — это архитектурный стиль для проектирования распределенных систем.

**Принципы:**
1. **Stateless** - без состояния
2. **Client-Server** - разделение клиента и сервера
3. **Cacheable** - кэшируемость
4. **Uniform Interface** - единообразный интерфейс
5. **Layered System** - слоистая система
6. **Code on Demand** - код по запросу (опционально)

### HTTP Methods

```
GET     - Получение ресурса
POST    - Создание ресурса
PUT     - Полное обновление ресурса
PATCH   - Частичное обновление ресурса
DELETE  - Удаление ресурса
```

### Применение в проекте

```go
// services/user-service/internal/api/rest_handler.go
func (h *RestHandler) SetupRoutes(r *gin.Engine) {
    v1 := r.Group("/api/v1")
    {
        users := v1.Group("/users")
        {
            users.GET("", h.GetUsers)          // GET /api/v1/users
            users.POST("", h.CreateUser)       // POST /api/v1/users
            users.GET("/:id", h.GetUser)       // GET /api/v1/users/:id
            users.PUT("/:id", h.UpdateUser)    // PUT /api/v1/users/:id
            users.DELETE("/:id", h.DeleteUser) // DELETE /api/v1/users/:id
        }
    }
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

### Преимущества REST
- Простота и понятность
- Хорошая поддержка HTTP
- Кэшируемость
- Stateless

### Недостатки REST
- Over-fetching/Under-fetching данных
- Множественные запросы
- Версионирование API

## GraphQL

### Основные концепции

**GraphQL** — это язык запросов для API и среда выполнения для выполнения этих запросов.

**Компоненты:**
- **Schema** - описание типов данных
- **Queries** - запросы данных
- **Mutations** - изменение данных
- **Subscriptions** - подписки на изменения

### Применение в проекте

```javascript
// services/api-gateway/src/graphql/schema.js
const { buildSchema } = require('graphql');

const schema = buildSchema(`
  type User {
    id: ID!
    email: String!
    name: String!
    status: String!
    createdAt: String!
  }
  
  type Order {
    id: ID!
    userId: String!
    items: [OrderItem!]!
    status: String!
    totalAmount: Float!
    createdAt: String!
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

const createUserResolvers = (serviceDiscovery) => ({
  Query: {
    user: async (_, { id }) => {
      const service = await serviceDiscovery.getService('user-service');
      const response = await axios.get(`${service.url}/api/v1/users/${id}`);
      return response.data;
    },
    users: async (_) => {
      const service = await serviceDiscovery.getService('user-service');
      const response = await axios.get(`${service.url}/api/v1/users`);
      return response.data.users;
    }
  },
  Mutation: {
    createUser: async (_, { email, name }) => {
      const service = await serviceDiscovery.getService('user-service');
      const response = await axios.post(`${service.url}/api/v1/users`, {
        email,
        name
      });
      return response.data;
    }
  }
});
```

### Преимущества GraphQL
- Точный запрос данных
- Единая точка входа
- Строгая типизация
- Интроспекция API

### Недостатки GraphQL
- Сложность кэширования
- Риск N+1 запросов
- Кривая обучения

## gRPC

### Основные принципы

**gRPC** — это высокопроизводительная RPC-система, использующая Protocol Buffers.

**Особенности:**
- Бинарный протокол
- HTTP/2
- Streaming
- Кодогенерация

### Protocol Buffers

```protobuf
// services/user-service/proto/user.proto
syntax = "proto3";

package user;

service UserService {
  rpc CreateUser(CreateUserRequest) returns (CreateUserResponse);
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
  rpc ListUsers(ListUsersRequest) returns (ListUsersResponse);
  rpc UpdateUser(UpdateUserRequest) returns (UpdateUserResponse);
  rpc DeleteUser(DeleteUserRequest) returns (DeleteUserResponse);
}

message CreateUserRequest {
  string email = 1;
  string name = 2;
}

message CreateUserResponse {
  string id = 1;
  string message = 2;
}

message GetUserRequest {
  string id = 1;
}

message GetUserResponse {
  string id = 1;
  string email = 2;
  string name = 3;
  string status = 4;
  string created_at = 5;
  string updated_at = 6;
}
```

### Применение в проекте

```go
// services/user-service/internal/api/grpc_handler.go
type GrpcHandler struct {
    pb.UnimplementedUserServiceServer
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

func (h *GrpcHandler) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.GetUserResponse, error) {
    id, err := uuid.Parse(req.Id)
    if err != nil {
        return nil, status.Errorf(codes.InvalidArgument, "Invalid user ID: %v", err)
    }
    
    query := application.GetUserByIDQuery{ID: id}
    
    user, err := h.queryHandler.HandleGetUserByID(query)
    if err != nil {
        return nil, status.Errorf(codes.NotFound, "User not found: %v", err)
    }
    
    return &pb.GetUserResponse{
        Id:        user.ID.String(),
        Email:     user.Email,
        Name:      user.Name,
        Status:    user.Status,
        CreatedAt: user.CreatedAt.Format(time.RFC3339),
        UpdatedAt: user.UpdatedAt.Format(time.RFC3339),
    }, nil
}
```

### Преимущества gRPC
- Высокая производительность
- Строгая типизация
- Streaming поддержка
- Кодогенерация

### Недостатки gRPC
- Сложность отладки
- Ограниченная поддержка браузеров
- Бинарный формат

## Сравнение подходов

### REST vs GraphQL vs gRPC

| Характеристика | REST | GraphQL | gRPC |
|---|---|---|---|
| Производительность | Средняя | Средняя | Высокая |
| Простота | Высокая | Средняя | Низкая |
| Гибкость запросов | Низкая | Высокая | Средняя |
| Кэширование | Простое | Сложное | Сложное |
| Поддержка браузеров | Отличная | Хорошая | Ограниченная |
| Типизация | Слабая | Сильная | Сильная |

### Когда использовать каждый подход

**REST:**
- Простые CRUD операции
- Общедоступные API
- Кэширование важно
- Простота разработки

**GraphQL:**
- Сложные запросы данных
- Мобильные приложения
- Агрегация данных
- Быстрая итерация

**gRPC:**
- Межсервисная коммуникация
- Высокие требования к производительности
- Строгая типизация важна
- Streaming данных

## Лучшие практики

### REST API Design

```go
// Правильная структура URL
GET    /api/v1/users              // Получить всех пользователей
POST   /api/v1/users              // Создать пользователя
GET    /api/v1/users/{id}         // Получить конкретного пользователя
PUT    /api/v1/users/{id}         // Обновить пользователя
DELETE /api/v1/users/{id}         // Удалить пользователя

// Вложенные ресурсы
GET    /api/v1/users/{id}/orders  // Получить заказы пользователя
POST   /api/v1/users/{id}/orders  // Создать заказ для пользователя
```

### GraphQL Schema Design

```graphql
# Используйте понятные имена типов
type User {
  id: ID!
  email: String!
  name: String!
  orders: [Order!]!  # Связанные данные
}

# Используйте Input типы для мутаций
input CreateUserInput {
  email: String!
  name: String!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
}
```

### gRPC Service Design

```protobuf
// Используйте версионирование
syntax = "proto3";

package user.v1;

// Группируйте связанные методы
service UserService {
  rpc CreateUser(CreateUserRequest) returns (CreateUserResponse);
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
  rpc UpdateUser(UpdateUserRequest) returns (UpdateUserResponse);
  rpc DeleteUser(DeleteUserRequest) returns (DeleteUserResponse);
  rpc ListUsers(ListUsersRequest) returns (stream User);
}

// Используйте понятные имена сообщений
message CreateUserRequest {
  string email = 1 [(validate.rules).string.email = true];
  string name = 2 [(validate.rules).string.min_len = 1];
}
```

## Документация API

### OpenAPI/Swagger

```yaml
# docs/swagger/api-spec.yml
openapi: 3.0.0
info:
  title: User Service API
  version: 1.0.0
  description: API для управления пользователями

paths:
  /api/v1/users:
    get:
      summary: Получить список пользователей
      responses:
        '200':
          description: Список пользователей
          content:
            application/json:
              schema:
                type: object
                properties:
                  users:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
    post:
      summary: Создать нового пользователя
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: Пользователь создан
        '400':
          description: Некорректные данные

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
        email:
          type: string
          format: email
        name:
          type: string
        status:
          type: string
          enum: [active, blocked, deleted]
```

## Заключение

Выбор подхода к дизайну API зависит от конкретных требований:

- **REST** - для простых публичных API
- **GraphQL** - для сложных клиентских приложений
- **gRPC** - для высокопроизводительной межсервисной коммуникации

Современные системы часто используют комбинацию подходов, как в нашем проекте.

## См. также
- [Микросервисная архитектура](./microservices-architecture.md)
- [CQRS Pattern](./cqrs-pattern.md)
- [GoF Design Patterns](./gof-patterns.md) 