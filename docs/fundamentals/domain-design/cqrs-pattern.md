# CQRS (Command Query Responsibility Segregation)

**CQRS** (Command Query Responsibility Segregation) — это архитектурный паттерн, который разделяет операции чтения и записи данных на отдельные модели. Паттерн был популяризован Грегом Янгом и основан на принципе **Command-Query Separation** (CQS), предложенном Бертраном Мейером.

## Основные концепции

### Command-Query Separation (CQS)
**CQS** — это принцип, который гласит, что каждый метод объекта должен быть либо командой, либо запросом, но не обеими сразу:

- **Command** (Команда) — изменяет состояние системы, но не возвращает данные
- **Query** (Запрос) — возвращает данные, но не изменяет состояние системы

### От CQS к CQRS
**CQRS** расширяет принцип CQS на архитектурный уровень, разделяя модели:

```
┌─────────────────────┐
│   Traditional       │
│   Architecture      │
│                     │
│  ┌───────────────┐  │
│  │     Model     │  │
│  │               │  │
│  │ Read & Write  │  │
│  └───────────────┘  │
└─────────────────────┘

           ↓ CQRS

┌─────────────────────┐
│   CQRS              │
│   Architecture      │
│                     │
│  ┌─────────────┐    │
│  │ Write Model │    │
│  │ (Commands)  │    │
│  └─────────────┘    │
│                     │
│  ┌─────────────┐    │
│  │ Read Model  │    │
│  │ (Queries)   │    │
│  └─────────────┘    │
└─────────────────────┘
```

## Компоненты CQRS

### 1. Commands (Команды)

**Команда** — это объект, который представляет намерение изменить состояние системы.

**Характеристики:**
- Имеет императивное название (CreateUser, UpdateOrder)
- Содержит данные, необходимые для выполнения действия
- Может быть отклонена (валидация)
- Не возвращает данные

**Применение в проекте:**
```go
// services/user-service/internal/application/commands.go
type CreateUserCommand struct {
    Email string
    Name  string
}

type UpdateUserCommand struct {
    ID    uuid.UUID
    Email string
    Name  string
}

type BlockUserCommand struct {
    ID     uuid.UUID
    Reason string
}
```

### 2. Command Handlers (Обработчики команд)

**Обработчик команд** — это компонент, который выполняет бизнес-логику команды.

**Принципы:**
- Один обработчик на команду
- Содержит бизнес-логику
- Взаимодействует с агрегатами
- Может генерировать события

**Применение в проекте:**
```go
// services/user-service/internal/application/commands.go
type CommandHandler struct {
    userRepo domain.UserRepository
    eventBus EventBus
}

func (h *CommandHandler) HandleCreateUser(cmd CreateUserCommand) error {
    // Валидация команды
    if cmd.Email == "" {
        return errors.New("email is required")
    }
    
    // Создание агрегата
    user, err := domain.NewUser(cmd.Email, cmd.Name)
    if err != nil {
        return err
    }
    
    // Сохранение в репозиторий
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

func (h *CommandHandler) HandleBlockUser(cmd BlockUserCommand) error {
    // Загрузка агрегата
    user, err := h.userRepo.FindByID(cmd.ID)
    if err != nil {
        return err
    }
    
    if user == nil {
        return errors.New("user not found")
    }
    
    // Выполнение бизнес-логики
    if err := user.Block(); err != nil {
        return err
    }
    
    // Сохранение изменений
    if err := h.userRepo.Save(user); err != nil {
        return err
    }
    
    // Публикация события
    event := domain.UserBlockedEvent{
        UserID:    user.ID(),
        Reason:    cmd.Reason,
        Timestamp: time.Now(),
    }
    
    return h.eventBus.Publish("user.blocked", event)
}
```

### 3. Queries (Запросы)

**Запрос** — это объект, который представляет намерение получить данные из системы.

**Характеристики:**
- Имеет название с вопросительным контекстом (GetUser, FindOrders)
- Содержит параметры для фильтрации
- Не изменяет состояние системы
- Возвращает данные

**Применение в проекте:**
```go
// services/user-service/internal/application/queries.go
type GetUserByIDQuery struct {
    ID uuid.UUID
}

type GetAllUsersQuery struct {
    Limit  int
    Offset int
}

type FindUsersByStatusQuery struct {
    Status domain.UserStatus
    Limit  int
    Offset int
}
```

### 4. Query Handlers (Обработчики запросов)

**Обработчик запросов** — это компонент, который выполняет получение данных.

**Принципы:**
- Один обработчик на запрос
- Оптимизирован для чтения
- Может использовать денормализованные данные
- Не содержит бизнес-логику

**Применение в проекте:**
```go
// services/user-service/internal/application/queries.go
type QueryHandler struct {
    userRepo domain.UserRepository
}

func (h *QueryHandler) HandleGetUserByID(query GetUserByIDQuery) (*dto.UserResponse, error) {
    user, err := h.userRepo.FindByID(query.ID)
    if err != nil {
        return nil, err
    }
    
    if user == nil {
        return nil, errors.New("user not found")
    }
    
    return &dto.UserResponse{
        ID:        user.ID(),
        Email:     user.Email(),
        Name:      user.Name(),
        Status:    user.Status().String(),
        CreatedAt: user.CreatedAt(),
        UpdatedAt: user.UpdatedAt(),
    }, nil
}

func (h *QueryHandler) HandleGetAllUsers(query GetAllUsersQuery) (*dto.UsersListResponse, error) {
    users, err := h.userRepo.FindAll()
    if err != nil {
        return nil, err
    }
    
    var userResponses []dto.UserResponse
    for _, user := range users {
        userResponses = append(userResponses, dto.UserResponse{
            ID:        user.ID(),
            Email:     user.Email(),
            Name:      user.Name(),
            Status:    user.Status().String(),
            CreatedAt: user.CreatedAt(),
            UpdatedAt: user.UpdatedAt(),
        })
    }
    
    return &dto.UsersListResponse{
        Users: userResponses,
        Total: len(userResponses),
    }, nil
}
```

### 5. DTOs (Data Transfer Objects)

**DTO** — это объекты для передачи данных между слоями приложения.

**Применение в проекте:**
```go
// services/user-service/internal/application/dto.go
type UserResponse struct {
    ID        uuid.UUID `json:"id"`
    Email     string    `json:"email"`
    Name      string    `json:"name"`
    Status    string    `json:"status"`
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
}

type UsersListResponse struct {
    Users []UserResponse `json:"users"`
    Total int            `json:"total"`
}

type CreateUserRequest struct {
    Email string `json:"email" binding:"required,email"`
    Name  string `json:"name" binding:"required"`
}

type UpdateUserRequest struct {
    Email string `json:"email" binding:"required,email"`
    Name  string `json:"name" binding:"required"`
}
```

## Архитектура CQRS

### Простая CQRS архитектура
```
┌─────────────────────┐
│      Client         │
└─────────────────────┘
           │
           ▼
┌─────────────────────┐
│   API Gateway       │
└─────────────────────┘
           │
    ┌──────┴──────┐
    ▼             ▼
┌─────────┐   ┌─────────┐
│Commands │   │ Queries │
│  Side   │   │  Side   │
└─────────┘   └─────────┘
    │             │
    ▼             ▼
┌─────────┐   ┌─────────┐
│Write DB │   │ Read DB │
└─────────┘   └─────────┘
```

### CQRS с Event Sourcing
```
┌─────────────────────┐
│      Client         │
└─────────────────────┘
           │
           ▼
┌─────────────────────┐
│   API Gateway       │
└─────────────────────┘
           │
    ┌──────┴──────┐
    ▼             ▼
┌─────────┐   ┌─────────┐
│Commands │   │ Queries │
│  Side   │   │  Side   │
└─────────┘   └─────────┘
    │             │
    ▼             ▼
┌─────────┐   ┌─────────┐
│ Event   │   │ Read    │
│ Store   │   │ Models  │
└─────────┘   └─────────┘
    │             ▲
    └─────────────┘
    Events flow
```

## Применение в REST API

**Применение в проекте:**
```go
// services/user-service/internal/api/rest_handler.go
type RestHandler struct {
    commandHandler *application.CommandHandler
    queryHandler   *application.QueryHandler
}

// Command endpoint
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

// Query endpoint
func (h *RestHandler) GetUser(c *gin.Context) {
    idStr := c.Param("id")
    id, err := uuid.Parse(idStr)
    if err != nil {
        c.JSON(400, gin.H{"error": "Invalid user ID"})
        return
    }
    
    query := application.GetUserByIDQuery{ID: id}
    
    user, err := h.queryHandler.HandleGetUserByID(query)
    if err != nil {
        c.JSON(404, gin.H{"error": err.Error()})
        return
    }
    
    c.JSON(200, user)
}
```

## Преимущества CQRS

### 1. Независимое масштабирование
- Чтение и запись можно масштабировать независимо
- Разные технологии для разных задач
- Оптимизация под конкретные потребности

### 2. Оптимизация производительности
- Модели чтения оптимизированы для запросов
- Денормализация данных для быстрого доступа
- Кэширование на уровне запросов

### 3. Безопасность
- Раздельные разрешения для чтения и записи
- Аудит команд
- Контроль доступа к данным

### 4. Сложные бизнес-логики
- Команды инкапсулируют бизнес-операции
- Валидация на уровне команд
- Четкое разделение ответственности

## Недостатки CQRS

### 1. Сложность
- Дополнительные слои абстракции
- Больше кода для простых операций
- Кривая обучения

### 2. Согласованность данных
- Eventual consistency между моделями
- Сложность синхронизации
- Обработка конфликтов

### 3. Дублирование данных
- Хранение одних данных в разных форматах
- Дополнительные затраты на хранение
- Сложность поддержания актуальности

## Когда использовать CQRS

### Подходящие случаи:
1. **Сложная бизнес-логика** - множество бизнес-правил
2. **Разные требования к чтению/записи** - оптимизация под разные сценарии
3. **Высокая нагрузка** - необходимость независимого масштабирования
4. **Аудит и отслеживание** - важность истории изменений
5. **Collaborative domains** - много пользователей изменяют одни данные

### Неподходящие случаи:
1. **Простые CRUD операции** - избыточная сложность
2. **Малые проекты** - overhead превышает пользу
3. **Сильная связанность данных** - сложность разделения моделей

## CQRS в микросервисной архитектуре

### Применение в проекте:

**User Service (Go):**
```go
// Commands
type CreateUserCommand struct { ... }
type BlockUserCommand struct { ... }

// Queries  
type GetUserByIDQuery struct { ... }
type GetAllUsersQuery struct { ... }
```

**Order Service (Python):**
```python
# services/order-service/main.py
# Commands
@app.post("/orders")
async def create_order(order_data: OrderCreate):
    # Command handling
    pass

# Queries
@app.get("/orders/{order_id}")
async def get_order(order_id: str):
    # Query handling
    pass
```

### Межсервисная коммуникация:
```go
// Event publishing в Command Handler
event := domain.UserCreatedEvent{
    UserID:    user.ID(),
    Email:     user.Email(),
    Name:      user.Name(),
    Timestamp: user.CreatedAt(),
}

eventBus.Publish("user.created", event)
```

## Паттерны и практики

### 1. Command Bus
Централизованная обработка команд:
```go
type CommandBus interface {
    Execute(cmd Command) error
}

type InMemoryCommandBus struct {
    handlers map[string]CommandHandler
}

func (b *InMemoryCommandBus) Execute(cmd Command) error {
    handler, exists := b.handlers[cmd.Type()]
    if !exists {
        return errors.New("handler not found")
    }
    return handler.Handle(cmd)
}
```

### 2. Query Bus
Централизованная обработка запросов:
```go
type QueryBus interface {
    Execute(query Query) (interface{}, error)
}
```

### 3. Result Pattern
Обработка успешных и неуспешных результатов:
```go
type Result struct {
    Success bool
    Data    interface{}
    Error   error
}

func (h *CommandHandler) Handle(cmd Command) Result {
    if err := h.validate(cmd); err != nil {
        return Result{Success: false, Error: err}
    }
    
    data, err := h.process(cmd)
    if err != nil {
        return Result{Success: false, Error: err}
    }
    
    return Result{Success: true, Data: data}
}
```

## Интеграция с другими паттернами

### 1. Event Sourcing
```python
# services/order-service/domain/order.py
class Order:
    def confirm(self):
        if self.status != OrderStatus.PENDING:
            raise ValueError("Order is not in pending status")
        
        self.status = OrderStatus.CONFIRMED
        self.updated_at = datetime.utcnow()
        
        # Генерация события для Event Store
        event = OrderConfirmedEvent(
            order_id=self.order_id,
            user_id=self.user_id,
            total_amount=self.total_amount,
            timestamp=self.updated_at
        )
        
        self.uncommitted_events.append(event)
```

### 2. Domain-Driven Design
```go
// Domain Events в DDD
type UserCreatedEvent struct {
    UserID    uuid.UUID
    Email     string
    Name      string
    Timestamp time.Time
}

// Aggregate Root генерирует события
func (u *User) Block() error {
    u.status = UserStatusBlocked
    u.updatedAt = time.Now()
    
    // Добавляем событие в список
    u.domainEvents = append(u.domainEvents, UserBlockedEvent{
        UserID:    u.id,
        Timestamp: u.updatedAt,
    })
    
    return nil
}
```

## Тестирование CQRS

### Unit тесты для Command Handlers:
```go
func TestCreateUserCommandHandler(t *testing.T) {
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

### Integration тесты для Query Handlers:
```go
func TestGetUserByIDQueryHandler(t *testing.T) {
    // Arrange
    db := setupTestDatabase()
    repo := NewPostgresUserRepository(db)
    handler := &QueryHandler{userRepo: repo}
    
    // Setup test data
    user := createTestUser()
    repo.Save(user)
    
    query := GetUserByIDQuery{ID: user.ID()}
    
    // Act
    result, err := handler.HandleGetUserByID(query)
    
    // Assert
    assert.NoError(t, err)
    assert.Equal(t, user.Email(), result.Email)
    assert.Equal(t, user.Name(), result.Name)
}
```

## Заключение

CQRS — это мощный архитектурный паттерн, который особенно эффективен в сложных системах с различными требованиями к чтению и записи. Основные принципы CQRS:

1. **Разделение ответственности** - команды и запросы решают разные задачи
2. **Оптимизация под сценарии** - разные модели для разных операций
3. **Независимое масштабирование** - чтение и запись масштабируются отдельно
4. **Четкая структура** - ясное разделение слоев и компонентов

CQRS хорошо сочетается с DDD, Event Sourcing и микросервисной архитектурой, обеспечивая гибкость и производительность в сложных бизнес-приложениях.

## См. также
- [Event Sourcing](./event-sourcing.md)
- [Domain-Driven Design (DDD)](./ddd-patterns.md)
- [GoF Design Patterns](./gof-patterns.md)
- [Микросервисная архитектура](./microservices-architecture.md) 