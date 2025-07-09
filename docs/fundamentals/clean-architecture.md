# Clean Architecture - Чистая архитектура

**Clean Architecture** — это архитектурный подход, предложенный Робертом Мартином (Uncle Bob), который создает четкое разделение между бизнес-логикой и внешними зависимостями. Основная цель — создать систему, которая легко тестируется, поддерживается и развивается.

## Основные принципы

### 1. Независимость от фреймворков
Архитектура не зависит от конкретных библиотек или фреймворков. Фреймворки — это инструменты, а не ограничения.

### 2. Тестируемость
Бизнес-логика может быть протестирована без UI, баз данных, веб-серверов и других внешних элементов.

### 3. Независимость от UI
UI может быть легко изменен без изменения бизнес-логики.

### 4. Независимость от базы данных
Бизнес-логика не привязана к конкретной СУБД.

### 5. Независимость от внешних сервисов
Бизнес-логика не знает о внешних сервисах.

---

## Структура слоев

### Слои Clean Architecture (от центра к периферии)

1. **Entities (Сущности)** - Корпоративные бизнес-правила
2. **Use Cases (Варианты использования)** - Специфические бизнес-правила приложения
3. **Interface Adapters (Адаптеры интерфейсов)** - Конвертеры данных
4. **Frameworks & Drivers (Фреймворки и драйверы)** - Внешние инструменты

### Правило зависимостей
**Зависимости должны указывать только внутрь.** Внутренние слои не знают ничего о внешних слоях.

---

## Применение в проекте

### 1. Entities (Доменные сущности)

```go
// services/user-service/internal/domain/entities/user.go
package entities

import (
    "errors"
    "time"
    "github.com/google/uuid"
)

// User - доменная сущность, содержит бизнес-логику
type User struct {
    id        uuid.UUID
    email     string
    name      string
    status    UserStatus
    createdAt time.Time
    updatedAt time.Time
}

type UserStatus int

const (
    UserStatusActive UserStatus = iota
    UserStatusInactive
    UserStatusBlocked
)

// NewUser - фабрика для создания пользователя
func NewUser(email, name string) (*User, error) {
    if email == "" {
        return nil, errors.New("email cannot be empty")
    }
    if name == "" {
        return nil, errors.New("name cannot be empty")
    }
    
    return &User{
        id:        uuid.New(),
        email:     email,
        name:      name,
        status:    UserStatusActive,
        createdAt: time.Now(),
        updatedAt: time.Now(),
    }, nil
}

// Business rules - бизнес-логика внутри сущности
func (u *User) Block(reason string) error {
    if u.status == UserStatusBlocked {
        return errors.New("user is already blocked")
    }
    
    u.status = UserStatusBlocked
    u.updatedAt = time.Now()
    return nil
}

func (u *User) Activate() error {
    if u.status == UserStatusActive {
        return errors.New("user is already active")
    }
    
    u.status = UserStatusActive
    u.updatedAt = time.Now()
    return nil
}

func (u *User) CanLogin() bool {
    return u.status == UserStatusActive
}

func (u *User) IsEmailValid() bool {
    return len(u.email) > 0 && 
           len(u.email) <= 254 && 
           strings.Contains(u.email, "@")
}

// Getters - контролируемый доступ к данным
func (u *User) ID() uuid.UUID     { return u.id }
func (u *User) Email() string     { return u.email }
func (u *User) Name() string      { return u.name }
func (u *User) Status() UserStatus { return u.status }
func (u *User) CreatedAt() time.Time { return u.createdAt }
func (u *User) UpdatedAt() time.Time { return u.updatedAt }
```

### 2. Use Cases (Варианты использования)

```go
// services/user-service/internal/usecases/user_usecase.go
package usecases

import (
    "context"
    "errors"
    "github.com/google/uuid"
    "myproject/internal/domain/entities"
    "myproject/internal/domain/repositories"
)

// UserUseCase - интерфейс для вариантов использования
type UserUseCase interface {
    CreateUser(ctx context.Context, req CreateUserRequest) (*CreateUserResponse, error)
    GetUser(ctx context.Context, id uuid.UUID) (*GetUserResponse, error)
    BlockUser(ctx context.Context, id uuid.UUID, reason string) error
    ActivateUser(ctx context.Context, id uuid.UUID) error
}

// CreateUserRequest - входящий DTO
type CreateUserRequest struct {
    Email string `json:"email" validate:"required,email"`
    Name  string `json:"name" validate:"required,min=2,max=100"`
}

// CreateUserResponse - исходящий DTO
type CreateUserResponse struct {
    ID        uuid.UUID `json:"id"`
    Email     string    `json:"email"`
    Name      string    `json:"name"`
    Status    string    `json:"status"`
    CreatedAt time.Time `json:"created_at"`
}

// GetUserResponse - исходящий DTO для получения пользователя
type GetUserResponse struct {
    ID        uuid.UUID `json:"id"`
    Email     string    `json:"email"`
    Name      string    `json:"name"`
    Status    string    `json:"status"`
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
}

// userUseCase - реализация вариантов использования
type userUseCase struct {
    userRepo repositories.UserRepository
    eventBus EventBus
    logger   Logger
}

func NewUserUseCase(
    userRepo repositories.UserRepository,
    eventBus EventBus,
    logger Logger,
) UserUseCase {
    return &userUseCase{
        userRepo: userRepo,
        eventBus: eventBus,
        logger:   logger,
    }
}

func (uc *userUseCase) CreateUser(ctx context.Context, req CreateUserRequest) (*CreateUserResponse, error) {
    uc.logger.Info("Creating user", "email", req.Email)
    
    // Проверка существования пользователя
    existingUser, err := uc.userRepo.FindByEmail(ctx, req.Email)
    if err != nil && !errors.Is(err, repositories.ErrUserNotFound) {
        return nil, err
    }
    if existingUser != nil {
        return nil, errors.New("user with this email already exists")
    }
    
    // Создание доменной сущности
    user, err := entities.NewUser(req.Email, req.Name)
    if err != nil {
        return nil, err
    }
    
    // Сохранение
    if err := uc.userRepo.Save(ctx, user); err != nil {
        return nil, err
    }
    
    // Публикация события
    event := UserCreatedEvent{
        UserID:    user.ID(),
        Email:     user.Email(),
        Name:      user.Name(),
        CreatedAt: user.CreatedAt(),
    }
    
    if err := uc.eventBus.Publish(ctx, "user.created", event); err != nil {
        uc.logger.Error("Failed to publish user created event", "error", err)
    }
    
    return &CreateUserResponse{
        ID:        user.ID(),
        Email:     user.Email(),
        Name:      user.Name(),
        Status:    statusToString(user.Status()),
        CreatedAt: user.CreatedAt(),
    }, nil
}

func (uc *userUseCase) GetUser(ctx context.Context, id uuid.UUID) (*GetUserResponse, error) {
    user, err := uc.userRepo.FindByID(ctx, id)
    if err != nil {
        if errors.Is(err, repositories.ErrUserNotFound) {
            return nil, errors.New("user not found")
        }
        return nil, err
    }
    
    return &GetUserResponse{
        ID:        user.ID(),
        Email:     user.Email(),
        Name:      user.Name(),
        Status:    statusToString(user.Status()),
        CreatedAt: user.CreatedAt(),
        UpdatedAt: user.UpdatedAt(),
    }, nil
}

func (uc *userUseCase) BlockUser(ctx context.Context, id uuid.UUID, reason string) error {
    user, err := uc.userRepo.FindByID(ctx, id)
    if err != nil {
        return err
    }
    
    if err := user.Block(reason); err != nil {
        return err
    }
    
    if err := uc.userRepo.Save(ctx, user); err != nil {
        return err
    }
    
    // Публикация события
    event := UserBlockedEvent{
        UserID:    user.ID(),
        Reason:    reason,
        BlockedAt: user.UpdatedAt(),
    }
    
    return uc.eventBus.Publish(ctx, "user.blocked", event)
}

func statusToString(status entities.UserStatus) string {
    switch status {
    case entities.UserStatusActive:
        return "active"
    case entities.UserStatusInactive:
        return "inactive"
    case entities.UserStatusBlocked:
        return "blocked"
    default:
        return "unknown"
    }
}
```

### 3. Interface Adapters (Адаптеры интерфейсов)

#### Repository Interface (Порт)
```go
// services/user-service/internal/domain/repositories/user_repository.go
package repositories

import (
    "context"
    "errors"
    "github.com/google/uuid"
    "myproject/internal/domain/entities"
)

var (
    ErrUserNotFound = errors.New("user not found")
    ErrUserExists   = errors.New("user already exists")
)

// UserRepository - порт (интерфейс) для работы с пользователями
type UserRepository interface {
    Save(ctx context.Context, user *entities.User) error
    FindByID(ctx context.Context, id uuid.UUID) (*entities.User, error)
    FindByEmail(ctx context.Context, email string) (*entities.User, error)
    Delete(ctx context.Context, id uuid.UUID) error
    List(ctx context.Context, limit, offset int) ([]*entities.User, error)
}
```

#### HTTP Controller (Адаптер)
```go
// services/user-service/internal/adapters/http/user_controller.go
package http

import (
    "encoding/json"
    "net/http"
    "strconv"
    
    "github.com/gorilla/mux"
    "github.com/google/uuid"
    "myproject/internal/usecases"
)

type UserController struct {
    userUseCase usecases.UserUseCase
    logger      Logger
}

func NewUserController(userUseCase usecases.UserUseCase, logger Logger) *UserController {
    return &UserController{
        userUseCase: userUseCase,
        logger:      logger,
    }
}

func (c *UserController) CreateUser(w http.ResponseWriter, r *http.Request) {
    var req usecases.CreateUserRequest
    
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        c.writeErrorResponse(w, http.StatusBadRequest, "Invalid request body")
        return
    }
    
    // Валидация можно вынести в отдельный слой
    if err := c.validateCreateUserRequest(req); err != nil {
        c.writeErrorResponse(w, http.StatusBadRequest, err.Error())
        return
    }
    
    resp, err := c.userUseCase.CreateUser(r.Context(), req)
    if err != nil {
        c.logger.Error("Failed to create user", "error", err)
        c.writeErrorResponse(w, http.StatusInternalServerError, "Failed to create user")
        return
    }
    
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(resp)
}

func (c *UserController) GetUser(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    userID, err := uuid.Parse(vars["id"])
    if err != nil {
        c.writeErrorResponse(w, http.StatusBadRequest, "Invalid user ID")
        return
    }
    
    user, err := c.userUseCase.GetUser(r.Context(), userID)
    if err != nil {
        if err.Error() == "user not found" {
            c.writeErrorResponse(w, http.StatusNotFound, "User not found")
            return
        }
        c.logger.Error("Failed to get user", "error", err)
        c.writeErrorResponse(w, http.StatusInternalServerError, "Failed to get user")
        return
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}

func (c *UserController) BlockUser(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    userID, err := uuid.Parse(vars["id"])
    if err != nil {
        c.writeErrorResponse(w, http.StatusBadRequest, "Invalid user ID")
        return
    }
    
    var req struct {
        Reason string `json:"reason"`
    }
    
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        c.writeErrorResponse(w, http.StatusBadRequest, "Invalid request body")
        return
    }
    
    if err := c.userUseCase.BlockUser(r.Context(), userID, req.Reason); err != nil {
        c.logger.Error("Failed to block user", "error", err)
        c.writeErrorResponse(w, http.StatusInternalServerError, "Failed to block user")
        return
    }
    
    w.WriteHeader(http.StatusNoContent)
}

func (c *UserController) writeErrorResponse(w http.ResponseWriter, code int, message string) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(code)
    json.NewEncoder(w).Encode(map[string]string{"error": message})
}

func (c *UserController) validateCreateUserRequest(req usecases.CreateUserRequest) error {
    if req.Email == "" {
        return errors.New("email is required")
    }
    if req.Name == "" {
        return errors.New("name is required")
    }
    if len(req.Name) < 2 {
        return errors.New("name must be at least 2 characters")
    }
    return nil
}
```

### 4. Frameworks & Drivers (Инфраструктура)

#### Database Repository Implementation
```go
// services/user-service/internal/adapters/database/postgres_user_repository.go
package database

import (
    "context"
    "database/sql"
    "errors"
    "time"
    
    "github.com/google/uuid"
    "github.com/lib/pq"
    "myproject/internal/domain/entities"
    "myproject/internal/domain/repositories"
)

type PostgresUserRepository struct {
    db *sql.DB
}

func NewPostgresUserRepository(db *sql.DB) repositories.UserRepository {
    return &PostgresUserRepository{db: db}
}

func (r *PostgresUserRepository) Save(ctx context.Context, user *entities.User) error {
    query := `
        INSERT INTO users (id, email, name, status, created_at, updated_at)
        VALUES ($1, $2, $3, $4, $5, $6)
        ON CONFLICT (id) DO UPDATE SET
            email = EXCLUDED.email,
            name = EXCLUDED.name,
            status = EXCLUDED.status,
            updated_at = EXCLUDED.updated_at
    `
    
    _, err := r.db.ExecContext(ctx, query,
        user.ID(),
        user.Email(),
        user.Name(),
        int(user.Status()),
        user.CreatedAt(),
        user.UpdatedAt(),
    )
    
    if err != nil {
        if pqErr, ok := err.(*pq.Error); ok {
            if pqErr.Code == "23505" { // unique violation
                return repositories.ErrUserExists
            }
        }
        return err
    }
    
    return nil
}

func (r *PostgresUserRepository) FindByID(ctx context.Context, id uuid.UUID) (*entities.User, error) {
    query := `
        SELECT id, email, name, status, created_at, updated_at
        FROM users
        WHERE id = $1
    `
    
    var (
        userID    uuid.UUID
        email     string
        name      string
        status    int
        createdAt time.Time
        updatedAt time.Time
    )
    
    err := r.db.QueryRowContext(ctx, query, id).Scan(
        &userID, &email, &name, &status, &createdAt, &updatedAt,
    )
    
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, repositories.ErrUserNotFound
        }
        return nil, err
    }
    
    return r.mapToEntity(userID, email, name, status, createdAt, updatedAt)
}

func (r *PostgresUserRepository) FindByEmail(ctx context.Context, email string) (*entities.User, error) {
    query := `
        SELECT id, email, name, status, created_at, updated_at
        FROM users
        WHERE email = $1
    `
    
    var (
        userID    uuid.UUID
        userName  string
        status    int
        createdAt time.Time
        updatedAt time.Time
    )
    
    err := r.db.QueryRowContext(ctx, query, email).Scan(
        &userID, &email, &userName, &status, &createdAt, &updatedAt,
    )
    
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, repositories.ErrUserNotFound
        }
        return nil, err
    }
    
    return r.mapToEntity(userID, email, userName, status, createdAt, updatedAt)
}

func (r *PostgresUserRepository) Delete(ctx context.Context, id uuid.UUID) error {
    query := `DELETE FROM users WHERE id = $1`
    
    result, err := r.db.ExecContext(ctx, query, id)
    if err != nil {
        return err
    }
    
    rowsAffected, err := result.RowsAffected()
    if err != nil {
        return err
    }
    
    if rowsAffected == 0 {
        return repositories.ErrUserNotFound
    }
    
    return nil
}

func (r *PostgresUserRepository) List(ctx context.Context, limit, offset int) ([]*entities.User, error) {
    query := `
        SELECT id, email, name, status, created_at, updated_at
        FROM users
        ORDER BY created_at DESC
        LIMIT $1 OFFSET $2
    `
    
    rows, err := r.db.QueryContext(ctx, query, limit, offset)
    if err != nil {
        return nil, err
    }
    defer rows.Close()
    
    var users []*entities.User
    
    for rows.Next() {
        var (
            userID    uuid.UUID
            email     string
            name      string
            status    int
            createdAt time.Time
            updatedAt time.Time
        )
        
        if err := rows.Scan(&userID, &email, &name, &status, &createdAt, &updatedAt); err != nil {
            return nil, err
        }
        
        user, err := r.mapToEntity(userID, email, name, status, createdAt, updatedAt)
        if err != nil {
            return nil, err
        }
        
        users = append(users, user)
    }
    
    return users, nil
}

// mapToEntity - преобразование из DB модели в доменную сущность
func (r *PostgresUserRepository) mapToEntity(
    id uuid.UUID, 
    email, name string, 
    status int, 
    createdAt, updatedAt time.Time,
) (*entities.User, error) {
    // Используем рефлексию или builder для создания сущности с приватными полями
    // В реальном проекте можно использовать пакет типа gopkg.in/yaml.v2 для unmarshaling
    // или создать специальный конструктор в доменном слое
    
    user, err := entities.NewUser(email, name)
    if err != nil {
        return nil, err
    }
    
    // Здесь нужно установить остальные поля
    // В реальном проекте это решается через:
    // 1. Публичные сеттеры (не рекомендуется)
    // 2. Фабричный метод в доменном слое
    // 3. Reflection (осторожно)
    // 4. Отдельный DTO для восстановления состояния
    
    return user, nil
}
```

### 5. Dependency Injection и Composition Root

```go
// services/user-service/cmd/main.go
package main

import (
    "context"
    "database/sql"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
    
    "github.com/gorilla/mux"
    _ "github.com/lib/pq"
    
    "myproject/internal/adapters/database"
    httpAdapter "myproject/internal/adapters/http"
    "myproject/internal/usecases"
)

func main() {
    // Инициализация инфраструктуры
    db, err := setupDatabase()
    if err != nil {
        log.Fatal("Failed to setup database:", err)
    }
    defer db.Close()
    
    logger := setupLogger()
    eventBus := setupEventBus()
    
    // Инициализация репозиториев (адаптеры)
    userRepo := database.NewPostgresUserRepository(db)
    
    // Инициализация use cases
    userUseCase := usecases.NewUserUseCase(userRepo, eventBus, logger)
    
    // Инициализация HTTP контроллеров
    userController := httpAdapter.NewUserController(userUseCase, logger)
    
    // Настройка маршрутов
    router := mux.NewRouter()
    setupRoutes(router, userController)
    
    // Запуск сервера
    server := &http.Server{
        Addr:         ":8080",
        Handler:      router,
        ReadTimeout:  15 * time.Second,
        WriteTimeout: 15 * time.Second,
        IdleTimeout:  60 * time.Second,
    }
    
    // Graceful shutdown
    go func() {
        if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatal("Failed to start server:", err)
        }
    }()
    
    log.Println("Server started on :8080")
    
    // Ожидание сигнала для graceful shutdown
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    
    log.Println("Shutting down server...")
    
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    if err := server.Shutdown(ctx); err != nil {
        log.Fatal("Server forced to shutdown:", err)
    }
    
    log.Println("Server exited")
}

func setupDatabase() (*sql.DB, error) {
    dsn := os.Getenv("DATABASE_URL")
    if dsn == "" {
        dsn = "postgres://user:password@localhost:5432/mydb?sslmode=disable"
    }
    
    db, err := sql.Open("postgres", dsn)
    if err != nil {
        return nil, err
    }
    
    if err := db.Ping(); err != nil {
        return nil, err
    }
    
    return db, nil
}

func setupLogger() Logger {
    // Инициализация логгера
    return &ConsoleLogger{}
}

func setupEventBus() EventBus {
    // Инициализация event bus
    return &InMemoryEventBus{}
}

func setupRoutes(router *mux.Router, userController *httpAdapter.UserController) {
    api := router.PathPrefix("/api/v1").Subrouter()
    
    // User routes
    api.HandleFunc("/users", userController.CreateUser).Methods("POST")
    api.HandleFunc("/users/{id}", userController.GetUser).Methods("GET")
    api.HandleFunc("/users/{id}/block", userController.BlockUser).Methods("POST")
    api.HandleFunc("/users/{id}/activate", userController.ActivateUser).Methods("POST")
}
```

---

## Преимущества Clean Architecture

### 1. **Независимость от деталей**
```go
// Use case не знает о том, как данные сохраняются
func (uc *userUseCase) CreateUser(ctx context.Context, req CreateUserRequest) (*CreateUserResponse, error) {
    user, err := entities.NewUser(req.Email, req.Name)
    if err != nil {
        return nil, err
    }
    
    // Репозиторий может быть любым: PostgreSQL, MongoDB, In-Memory
    if err := uc.userRepo.Save(ctx, user); err != nil {
        return nil, err
    }
    
    return &CreateUserResponse{...}, nil
}
```

### 2. **Легкость тестирования**
```go
// Тестирование use case с mock-репозиторием
func TestUserUseCase_CreateUser(t *testing.T) {
    mockRepo := &MockUserRepository{}
    mockEventBus := &MockEventBus{}
    mockLogger := &MockLogger{}
    
    useCase := usecases.NewUserUseCase(mockRepo, mockEventBus, mockLogger)
    
    req := usecases.CreateUserRequest{
        Email: "test@example.com",
        Name:  "Test User",
    }
    
    resp, err := useCase.CreateUser(context.Background(), req)
    
    assert.NoError(t, err)
    assert.NotEmpty(t, resp.ID)
    assert.Equal(t, req.Email, resp.Email)
}
```

### 3. **Гибкость в выборе технологий**
```go
// Можно легко переключиться с PostgreSQL на MongoDB
func main() {
    // Было
    // userRepo := database.NewPostgresUserRepository(db)
    
    // Стало
    userRepo := database.NewMongoUserRepository(mongoClient)
    
    // Use case остается тем же
    userUseCase := usecases.NewUserUseCase(userRepo, eventBus, logger)
}
```

### 4. **Разделение ответственности**
```go
// Каждый слой имеет свою ответственность:
// - Entity: бизнес-логика
// - Use Case: оркестрация
// - Repository: персистентность  
// - Controller: HTTP интерфейс
```

---

## Пример: Order Service с Clean Architecture

```go
// services/order-service/internal/domain/entities/order.go
package entities

import (
    "errors"
    "time"
    "github.com/google/uuid"
)

type Order struct {
    id         uuid.UUID
    userID     uuid.UUID
    items      []OrderItem
    status     OrderStatus
    totalAmount float64
    createdAt  time.Time
    updatedAt  time.Time
}

type OrderItem struct {
    productID uuid.UUID
    quantity  int
    price     float64
}

type OrderStatus int

const (
    OrderStatusPending OrderStatus = iota
    OrderStatusConfirmed
    OrderStatusShipped
    OrderStatusDelivered
    OrderStatusCancelled
)

func NewOrder(userID uuid.UUID, items []OrderItem) (*Order, error) {
    if len(items) == 0 {
        return nil, errors.New("order must have at least one item")
    }
    
    totalAmount := calculateTotalAmount(items)
    
    return &Order{
        id:          uuid.New(),
        userID:      userID,
        items:       items,
        status:      OrderStatusPending,
        totalAmount: totalAmount,
        createdAt:   time.Now(),
        updatedAt:   time.Now(),
    }, nil
}

func (o *Order) Confirm() error {
    if o.status != OrderStatusPending {
        return errors.New("only pending orders can be confirmed")
    }
    
    o.status = OrderStatusConfirmed
    o.updatedAt = time.Now()
    return nil
}

func (o *Order) Cancel() error {
    if o.status == OrderStatusShipped || o.status == OrderStatusDelivered {
        return errors.New("cannot cancel shipped or delivered orders")
    }
    
    o.status = OrderStatusCancelled
    o.updatedAt = time.Now()
    return nil
}

func calculateTotalAmount(items []OrderItem) float64 {
    total := 0.0
    for _, item := range items {
        total += item.price * float64(item.quantity)
    }
    return total
}
```

---

## Структура проекта

```
services/user-service/
├── cmd/
│   └── main.go                          # Composition Root
├── internal/
│   ├── domain/                          # Entities + Repository Interfaces
│   │   ├── entities/
│   │   │   └── user.go
│   │   └── repositories/
│   │       └── user_repository.go
│   ├── usecases/                        # Use Cases
│   │   └── user_usecase.go
│   └── adapters/                        # Interface Adapters
│       ├── http/
│       │   └── user_controller.go
│       ├── database/
│       │   └── postgres_user_repository.go
│       └── events/
│           └── event_bus.go
└── migrations/                          # Database migrations
    └── 001_create_users_table.sql
```

---

## Антипаттерны и ошибки

### 1. **Анемичная доменная модель**
```go
// ❌ ПЛОХО: сущность без логики
type User struct {
    ID     uuid.UUID
    Email  string
    Name   string
    Status int
}

// Вся логика в сервисе
func (s *UserService) BlockUser(user *User) {
    user.Status = 2 // Magic number
}
```

```go
// ✅ ХОРОШО: богатая доменная модель
type User struct {
    id     uuid.UUID
    email  string
    name   string
    status UserStatus
}

func (u *User) Block() error {
    if u.status == UserStatusBlocked {
        return errors.New("user is already blocked")
    }
    u.status = UserStatusBlocked
    return nil
}
```

### 2. **Нарушение правила зависимостей**
```go
// ❌ ПЛОХО: use case зависит от HTTP
func (uc *UserUseCase) CreateUser(w http.ResponseWriter, r *http.Request) {
    // Use case не должен знать о HTTP
}
```

```go
// ✅ ХОРОШО: use case независим от транспорта
func (uc *UserUseCase) CreateUser(ctx context.Context, req CreateUserRequest) (*CreateUserResponse, error) {
    // Работает с доменными типами
}
```

### 3. **Слишком много слоев**
```go
// ❌ ПЛОХО: избыточные слои
type UserEntity struct{}
type UserModel struct{}
type UserDTO struct{}
type UserRequest struct{}
type UserResponse struct{}
type UserViewModel struct{}
```

---

## Практические рекомендации

### 1. **Начинайте с простого**
```go
// Не нужно сразу создавать все слои
// Начните с основных: Entity, Use Case, Repository
```

### 2. **Используйте интерфейсы**
```go
// Всегда определяйте интерфейсы в том слое, который их использует
type UserRepository interface {
    Save(ctx context.Context, user *User) error
}
```

### 3. **Тестируйте бизнес-логику**
```go
// Большинство тестов должно быть на уровне use cases
func TestCreateUser_Success(t *testing.T) {
    // Тестируем бизнес-логику изолированно
}
```

### 4. **Избегайте over-engineering**
```go
// Для простых CRUD операций можно использовать более простую архитектуру
// Clean Architecture лучше подходит для сложных доменов
```

---

## 🔗 Связанные темы

### 🏗️ Основы
- **[SOLID Principles](solid-principles.md)** - Принципы, лежащие в основе Clean Architecture
- **[DDD Patterns](ddd-patterns.md)** - Доменно-ориентированное проектирование
- **[Design Patterns](gof-patterns.md)** - Паттерны проектирования для реализации

### 🏛️ Архитектура
- **[Microservices Architecture](../architecture/microservices-architecture.md)** - Применение Clean Architecture в микросервисах
- **[API Design](../architecture/api-design.md)** - Проектирование API слоев

### ⚙️ Технические навыки
- **[Testing](../technical-skills/testing.md)** - Тестирование Clean Architecture
- **[Databases](../technical-skills/databases.md)** - Работа с данными в Clean Architecture

---

**Путь обучения**: [SOLID Principles](solid-principles.md) → Clean Architecture → [DDD Patterns](ddd-patterns.md)  
**Сложность**: ⭐⭐⭐⭐ (4/5)  
**Время изучения**: 3-4 недели  
**Практика**: Рефакторинг `user-service/` и `order-service/` под Clean Architecture 