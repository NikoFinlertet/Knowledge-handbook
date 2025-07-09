# Функциональное программирование

**Функциональное программирование** — это парадигма программирования, которая рассматривает вычисления как выполнение математических функций и избегает изменения состояния и мутабельных данных. Хотя Go не является чисто функциональным языком, многие принципы ФП могут значительно улучшить качество кода.

## Основные принципы ФП

### 1. **Неизменяемость (Immutability)**
### 2. **Чистые функции (Pure Functions)**
### 3. **Функции высшего порядка (Higher-Order Functions)**
### 4. **Композиция функций (Function Composition)**
### 5. **Избегание побочных эффектов (Side Effects)**
### 6. **Рекурсия вместо циклов**

---

## 1. Неизменяемость (Immutability)

### Определение
**Неизменяемые объекты не могут быть изменены после создания.** Вместо изменения создается новый объект.

### Проблема мутабельности

```go
// ❌ ПЛОХО: мутабельная структура
type User struct {
    ID       string
    Email    string
    Name     string
    IsActive bool
}

func (u *User) Activate() {
    u.IsActive = true // Мутация состояния
}

func (u *User) ChangeEmail(newEmail string) {
    u.Email = newEmail // Мутация состояния
}

// Проблемы с конкурентностью
func ProcessUsers(users []*User) {
    for _, user := range users {
        go func(u *User) {
            u.Activate() // Race condition!
        }(user)
    }
}
```

### Правильное применение неизменяемости

```go
// ✅ ХОРОШО: неизменяемая структура
type User struct {
    id       string
    email    string
    name     string
    isActive bool
}

// Конструктор
func NewUser(id, email, name string) User {
    return User{
        id:       id,
        email:    email,
        name:     name,
        isActive: false,
    }
}

// Геттеры
func (u User) ID() string       { return u.id }
func (u User) Email() string    { return u.email }
func (u User) Name() string     { return u.name }
func (u User) IsActive() bool   { return u.isActive }

// Методы возвращают новые объекты
func (u User) Activate() User {
    return User{
        id:       u.id,
        email:    u.email,
        name:     u.name,
        isActive: true, // Изменяем только нужное поле
    }
}

func (u User) ChangeEmail(newEmail string) User {
    return User{
        id:       u.id,
        email:    newEmail, // Изменяем только email
        name:     u.name,
        isActive: u.isActive,
    }
}

// Безопасность в конкурентной среде
func ProcessUsers(users []User) []User {
    results := make([]User, len(users))
    for i, user := range users {
        results[i] = user.Activate() // Безопасно - создаем новый объект
    }
    return results
}
```

### Применение в проекте

```go
// services/user-service/internal/domain/entities/user.go
type User struct {
    id        uuid.UUID
    email     string
    name      string
    status    UserStatus
    createdAt time.Time
    updatedAt time.Time
}

// Создание новых версий объекта
func (u User) WithStatus(status UserStatus) User {
    return User{
        id:        u.id,
        email:     u.email,
        name:      u.name,
        status:    status,
        createdAt: u.createdAt,
        updatedAt: time.Now(),
    }
}

func (u User) WithEmail(email string) User {
    return User{
        id:        u.id,
        email:     email,
        name:      u.name,
        status:    u.status,
        createdAt: u.createdAt,
        updatedAt: time.Now(),
    }
}
```

---

## 2. Чистые функции (Pure Functions)

### Определение
**Чистая функция** — это функция, которая:
- Всегда возвращает одинаковый результат для одинаковых аргументов
- Не имеет побочных эффектов

### Проблема нечистых функций

```go
// ❌ ПЛОХО: нечистые функции
var globalCounter int

func IncrementAndAdd(a, b int) int {
    globalCounter++ // Побочный эффект
    return a + b + globalCounter // Результат зависит от глобального состояния
}

func GetUserEmail(userID string) string {
    // Побочный эффект - логирование
    log.Printf("Accessing user %s", userID)
    
    // Результат может быть разным каждый раз
    user := database.GetUser(userID)
    return user.Email
}

func ProcessData(data []int) []int {
    // Мутация входных данных
    for i := range data {
        data[i] = data[i] * 2
    }
    return data
}
```

### Правильное применение чистых функций

```go
// ✅ ХОРОШО: чистые функции

// Чистая функция - всегда одинаковый результат для одинаковых аргументов
func Add(a, b int) int {
    return a + b
}

// Чистая функция - не мутирует входные данные
func DoubleValues(data []int) []int {
    result := make([]int, len(data))
    for i, value := range data {
        result[i] = value * 2
    }
    return result
}

// Чистая функция для валидации
func IsValidEmail(email string) bool {
    return strings.Contains(email, "@") && len(email) > 0
}

// Чистая функция для трансформации данных
func FormatUserName(firstName, lastName string) string {
    return strings.Title(firstName) + " " + strings.Title(lastName)
}

// Чистая функция для фильтрации
func FilterActiveUsers(users []User) []User {
    result := make([]User, 0)
    for _, user := range users {
        if user.IsActive() {
            result = append(result, user)
        }
    }
    return result
}
```

### Применение в проекте

```go
// services/user-service/internal/domain/validators/user_validator.go
// Чистые функции для валидации
func ValidateEmail(email string) error {
    if email == "" {
        return errors.New("email cannot be empty")
    }
    if !strings.Contains(email, "@") {
        return errors.New("invalid email format")
    }
    if len(email) > 254 {
        return errors.New("email too long")
    }
    return nil
}

func ValidateName(name string) error {
    if name == "" {
        return errors.New("name cannot be empty")
    }
    if len(name) < 2 {
        return errors.New("name too short")
    }
    if len(name) > 100 {
        return errors.New("name too long")
    }
    return nil
}

// Чистая функция для трансформации
func NormalizeEmail(email string) string {
    return strings.ToLower(strings.TrimSpace(email))
}
```

---

## 3. Функции высшего порядка (Higher-Order Functions)

### Определение
**Функция высшего порядка** — это функция, которая принимает другие функции в качестве аргументов или возвращает функцию.

### Примеры функций высшего порядка

```go
// Map - применяет функцию к каждому элементу
func Map[T, U any](slice []T, fn func(T) U) []U {
    result := make([]U, len(slice))
    for i, item := range slice {
        result[i] = fn(item)
    }
    return result
}

// Filter - фильтрует элементы по предикату
func Filter[T any](slice []T, predicate func(T) bool) []T {
    result := make([]T, 0)
    for _, item := range slice {
        if predicate(item) {
            result = append(result, item)
        }
    }
    return result
}

// Reduce - сворачивает слайс в одно значение
func Reduce[T, U any](slice []T, initial U, fn func(U, T) U) U {
    result := initial
    for _, item := range slice {
        result = fn(result, item)
    }
    return result
}

// Compose - компонует функции
func Compose[T, U, V any](f func(U) V, g func(T) U) func(T) V {
    return func(x T) V {
        return f(g(x))
    }
}
```

### Применение в обработке данных

```go
// Пример: обработка пользователей
type User struct {
    ID       string
    Email    string
    Name     string
    IsActive bool
    Age      int
}

func ProcessUsers(users []User) []string {
    // Фильтрация активных пользователей
    activeUsers := Filter(users, func(u User) bool {
        return u.IsActive
    })
    
    // Фильтрация совершеннолетних пользователей
    adultUsers := Filter(activeUsers, func(u User) bool {
        return u.Age >= 18
    })
    
    // Извлечение email адресов
    emails := Map(adultUsers, func(u User) string {
        return u.Email
    })
    
    return emails
}

// Или в функциональном стиле:
func ProcessUsersFunctional(users []User) []string {
    return Map(
        Filter(
            Filter(users, func(u User) bool { return u.IsActive }),
            func(u User) bool { return u.Age >= 18 },
        ),
        func(u User) string { return u.Email },
    )
}
```

### Применение в проекте

```go
// services/user-service/internal/usecases/user_usecase.go
type UserUseCase struct {
    repo UserRepository
}

// Функция высшего порядка для обработки с логированием
func (uc *UserUseCase) WithLogging(operation string, fn func() error) error {
    uc.logger.Info("Starting operation", "operation", operation)
    
    err := fn()
    
    if err != nil {
        uc.logger.Error("Operation failed", "operation", operation, "error", err)
    } else {
        uc.logger.Info("Operation completed", "operation", operation)
    }
    
    return err
}

// Использование
func (uc *UserUseCase) CreateUser(req CreateUserRequest) (*User, error) {
    var user *User
    
    err := uc.WithLogging("create_user", func() error {
        var err error
        user, err = uc.createUserInternal(req)
        return err
    })
    
    return user, err
}

// Функция высшего порядка для обработки с таймаутом
func WithTimeout[T any](timeout time.Duration, fn func() (T, error)) (T, error) {
    ctx, cancel := context.WithTimeout(context.Background(), timeout)
    defer cancel()
    
    result := make(chan T, 1)
    errChan := make(chan error, 1)
    
    go func() {
        value, err := fn()
        if err != nil {
            errChan <- err
        } else {
            result <- value
        }
    }()
    
    select {
    case value := <-result:
        var zero T
        return value, nil
    case err := <-errChan:
        var zero T
        return zero, err
    case <-ctx.Done():
        var zero T
        return zero, ctx.Err()
    }
}
```

---

## 4. Композиция функций (Function Composition)

### Определение
**Композиция функций** — это процесс объединения двух или более функций для создания новой функции.

### Примеры композиции

```go
// Базовые функции
func ValidateEmail(email string) (string, error) {
    if email == "" {
        return "", errors.New("email cannot be empty")
    }
    if !strings.Contains(email, "@") {
        return "", errors.New("invalid email format")
    }
    return email, nil
}

func NormalizeEmail(email string) (string, error) {
    return strings.ToLower(strings.TrimSpace(email)), nil
}

func CheckEmailExists(repo UserRepository) func(string) (string, error) {
    return func(email string) (string, error) {
        user, err := repo.FindByEmail(email)
        if err != nil && !errors.Is(err, ErrUserNotFound) {
            return "", err
        }
        if user != nil {
            return "", errors.New("email already exists")
        }
        return email, nil
    }
}

// Композиция функций
func ProcessEmail(repo UserRepository, email string) (string, error) {
    // Цепочка функций
    result, err := ValidateEmail(email)
    if err != nil {
        return "", err
    }
    
    result, err = NormalizeEmail(result)
    if err != nil {
        return "", err
    }
    
    result, err = CheckEmailExists(repo)(result)
    if err != nil {
        return "", err
    }
    
    return result, nil
}

// Более элегантная композиция
type EmailProcessor func(string) (string, error)

func Chain(processors ...EmailProcessor) EmailProcessor {
    return func(email string) (string, error) {
        result := email
        for _, processor := range processors {
            var err error
            result, err = processor(result)
            if err != nil {
                return "", err
            }
        }
        return result, nil
    }
}

// Использование
func ProcessEmailChain(repo UserRepository, email string) (string, error) {
    processor := Chain(
        ValidateEmail,
        NormalizeEmail,
        CheckEmailExists(repo),
    )
    
    return processor(email)
}
```

### Применение в проекте

```go
// services/user-service/internal/domain/processors/user_processor.go
type UserProcessor func(User) (User, error)

// Процессоры
func ValidateUserProcessor(user User) (User, error) {
    if user.Email() == "" {
        return user, errors.New("email is required")
    }
    return user, nil
}

func NormalizeUserProcessor(user User) (User, error) {
    normalizedEmail := strings.ToLower(strings.TrimSpace(user.Email()))
    return user.WithEmail(normalizedEmail), nil
}

func EnrichUserProcessor(user User) (User, error) {
    // Обогащение данных
    return user.WithCreatedAt(time.Now()), nil
}

// Композиция процессоров
func ProcessUser(processors ...UserProcessor) UserProcessor {
    return func(user User) (User, error) {
        result := user
        for _, processor := range processors {
            var err error
            result, err = processor(result)
            if err != nil {
                return result, err
            }
        }
        return result, nil
    }
}

// Использование в use case
func (uc *UserUseCase) CreateUser(req CreateUserRequest) (*User, error) {
    user := NewUser(req.Email, req.Name)
    
    processor := ProcessUser(
        ValidateUserProcessor,
        NormalizeUserProcessor,
        EnrichUserProcessor,
    )
    
    processedUser, err := processor(user)
    if err != nil {
        return nil, err
    }
    
    return uc.repo.Save(processedUser)
}
```

---

## 5. Избегание побочных эффектов (Side Effects)

### Определение
**Побочные эффекты** — это изменения в состоянии программы, которые видны за пределами функции.

### Проблема побочных эффектов

```go
// ❌ ПЛОХО: много побочных эффектов
var requestCount int

func ProcessRequest(req Request) Response {
    // Побочный эффект - изменение глобального состояния
    requestCount++
    
    // Побочный эффект - логирование
    log.Printf("Processing request %d", requestCount)
    
    // Побочный эффект - запись в файл
    file, _ := os.OpenFile("requests.log", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
    file.WriteString(fmt.Sprintf("Request: %v\n", req))
    file.Close()
    
    // Побочный эффект - отправка email
    sendEmailNotification(req)
    
    // Основная логика
    return Response{Data: process(req.Data)}
}
```

### Правильное разделение эффектов

```go
// ✅ ХОРОШО: разделение чистой логики и побочных эффектов

// Чистая функция - основная логика
func ProcessRequestData(data RequestData) ResponseData {
    // Только преобразование данных
    return ResponseData{
        ProcessedData: transformData(data),
        Timestamp:     time.Now(),
    }
}

// Функция с побочными эффектами
func ProcessRequestWithEffects(
    req Request,
    logger Logger,
    fileWriter FileWriter,
    emailSender EmailSender,
    counter *RequestCounter,
) Response {
    // Побочные эффекты выделены и контролируются
    counter.Increment()
    logger.Log("Processing request", "count", counter.Value())
    fileWriter.Write(req.String())
    
    // Чистая логика
    responseData := ProcessRequestData(req.Data)
    
    // Побочные эффекты после основной логики
    if req.NotifyEmail {
        emailSender.Send(req.Email, "Request processed")
    }
    
    return Response{Data: responseData}
}
```

### Применение в проекте

```go
// services/user-service/internal/usecases/user_usecase.go
type UserUseCase struct {
    repo         UserRepository
    logger       Logger
    eventBus     EventBus
    emailService EmailService
}

// Чистая функция для основной логики
func (uc *UserUseCase) buildUser(req CreateUserRequest) (*User, error) {
    // Только бизнес-логика без побочных эффектов
    user, err := NewUser(req.Email, req.Name)
    if err != nil {
        return nil, err
    }
    
    return user.WithCreatedAt(time.Now()), nil
}

// Функция с контролируемыми побочными эффектами
func (uc *UserUseCase) CreateUser(ctx context.Context, req CreateUserRequest) (*User, error) {
    // Чистая логика
    user, err := uc.buildUser(req)
    if err != nil {
        return nil, err
    }
    
    // Побочные эффекты сгруппированы и контролируются
    return uc.executeWithEffects(ctx, user)
}

func (uc *UserUseCase) executeWithEffects(ctx context.Context, user *User) (*User, error) {
    // Эффект: сохранение в базу
    if err := uc.repo.Save(ctx, user); err != nil {
        uc.logger.Error("Failed to save user", "error", err)
        return nil, err
    }
    
    // Эффект: логирование
    uc.logger.Info("User created", "user_id", user.ID())
    
    // Эффект: публикация события
    event := UserCreatedEvent{
        UserID:    user.ID(),
        Email:     user.Email(),
        CreatedAt: user.CreatedAt(),
    }
    
    if err := uc.eventBus.Publish(ctx, "user.created", event); err != nil {
        uc.logger.Error("Failed to publish event", "error", err)
        // Не возвращаем ошибку - пользователь создан успешно
    }
    
    // Эффект: отправка welcome email (асинхронно)
    go func() {
        if err := uc.emailService.SendWelcomeEmail(user.Email()); err != nil {
            uc.logger.Error("Failed to send welcome email", "error", err)
        }
    }()
    
    return user, nil
}
```

---

## 6. Рекурсия вместо циклов

### Определение
**Рекурсия** — это техника, где функция вызывает саму себя для решения подзадач.

### Примеры рекурсивных функций

```go
// Рекурсивная обработка вложенных структур
type Category struct {
    ID       string
    Name     string
    Children []Category
}

// Рекурсивный поиск категории
func FindCategoryByID(categories []Category, id string) *Category {
    for _, category := range categories {
        if category.ID == id {
            return &category
        }
        
        // Рекурсивный поиск в дочерних категориях
        if found := FindCategoryByID(category.Children, id); found != nil {
            return found
        }
    }
    return nil
}

// Рекурсивное преобразование структуры
func TransformCategories(categories []Category, transformer func(Category) Category) []Category {
    result := make([]Category, len(categories))
    
    for i, category := range categories {
        // Преобразование текущей категории
        transformed := transformer(category)
        
        // Рекурсивное преобразование дочерних категорий
        transformed.Children = TransformCategories(category.Children, transformer)
        
        result[i] = transformed
    }
    
    return result
}

// Рекурсивная валидация
func ValidateCategory(category Category) error {
    // Валидация текущей категории
    if category.Name == "" {
        return errors.New("category name cannot be empty")
    }
    
    // Рекурсивная валидация дочерних категорий
    for _, child := range category.Children {
        if err := ValidateCategory(child); err != nil {
            return fmt.Errorf("child category validation failed: %w", err)
        }
    }
    
    return nil
}
```

### Применение в проекте

```go
// services/order-service/internal/domain/entities/order.go
type OrderItem struct {
    ID       string
    Name     string
    Price    float64
    Quantity int
    SubItems []OrderItem // Вложенные товары (например, комплекты)
}

// Рекурсивный подсчет общей стоимости
func (item OrderItem) TotalPrice() float64 {
    // Базовая стоимость
    total := item.Price * float64(item.Quantity)
    
    // Рекурсивное добавление стоимости подтоваров
    for _, subItem := range item.SubItems {
        total += subItem.TotalPrice()
    }
    
    return total
}

// Рекурсивная валидация товаров
func (item OrderItem) Validate() error {
    if item.Name == "" {
        return errors.New("item name cannot be empty")
    }
    
    if item.Price < 0 {
        return errors.New("item price cannot be negative")
    }
    
    if item.Quantity <= 0 {
        return errors.New("item quantity must be positive")
    }
    
    // Рекурсивная валидация подтоваров
    for _, subItem := range item.SubItems {
        if err := subItem.Validate(); err != nil {
            return fmt.Errorf("sub-item validation failed: %w", err)
        }
    }
    
    return nil
}
```

---

## Функциональные паттерны в Go

### 1. Option Pattern (Функциональные опции)

```go
// Функциональные опции для конфигурации
type UserServiceOption func(*UserService)

func WithLogger(logger Logger) UserServiceOption {
    return func(s *UserService) {
        s.logger = logger
    }
}

func WithCache(cache Cache) UserServiceOption {
    return func(s *UserService) {
        s.cache = cache
    }
}

func WithMetrics(metrics Metrics) UserServiceOption {
    return func(s *UserService) {
        s.metrics = metrics
    }
}

func NewUserService(repo UserRepository, options ...UserServiceOption) *UserService {
    service := &UserService{
        repo: repo,
    }
    
    // Применение функциональных опций
    for _, option := range options {
        option(service)
    }
    
    return service
}

// Использование
userService := NewUserService(
    postgresRepo,
    WithLogger(logger),
    WithCache(redisCache),
    WithMetrics(prometheusMetrics),
)
```

### 2. Pipeline Pattern

```go
// Пайплайн для обработки данных
type Pipeline[T any] struct {
    steps []func(T) (T, error)
}

func NewPipeline[T any]() *Pipeline[T] {
    return &Pipeline[T]{
        steps: make([]func(T) (T, error), 0),
    }
}

func (p *Pipeline[T]) AddStep(step func(T) (T, error)) *Pipeline[T] {
    p.steps = append(p.steps, step)
    return p
}

func (p *Pipeline[T]) Execute(input T) (T, error) {
    result := input
    for _, step := range p.steps {
        var err error
        result, err = step(result)
        if err != nil {
            return result, err
        }
    }
    return result, nil
}

// Использование
func ProcessUserData(userData UserData) (UserData, error) {
    pipeline := NewPipeline[UserData]().
        AddStep(ValidateUserData).
        AddStep(NormalizeUserData).
        AddStep(EnrichUserData).
        AddStep(SanitizeUserData)
    
    return pipeline.Execute(userData)
}
```

### 3. Either Pattern (Result/Error)

```go
// Either для обработки ошибок в функциональном стиле
type Either[T any] struct {
    value T
    err   error
}

func Success[T any](value T) Either[T] {
    return Either[T]{value: value}
}

func Failure[T any](err error) Either[T] {
    var zero T
    return Either[T]{value: zero, err: err}
}

func (e Either[T]) IsSuccess() bool {
    return e.err == nil
}

func (e Either[T]) IsFailure() bool {
    return e.err != nil
}

func (e Either[T]) Value() T {
    return e.value
}

func (e Either[T]) Error() error {
    return e.err
}

// Map для трансформации значения
func Map[T, U any](e Either[T], fn func(T) U) Either[U] {
    if e.IsFailure() {
        return Failure[U](e.err)
    }
    return Success(fn(e.value))
}

// FlatMap для цепочки операций
func FlatMap[T, U any](e Either[T], fn func(T) Either[U]) Either[U] {
    if e.IsFailure() {
        return Failure[U](e.err)
    }
    return fn(e.value)
}

// Использование
func ProcessUserChain(userID string) Either[ProcessedUser] {
    return FlatMap(
        GetUser(userID),
        func(user User) Either[ProcessedUser] {
            return Map(
                ValidateUser(user),
                func(validUser User) ProcessedUser {
                    return ProcessUser(validUser)
                },
            )
        },
    )
}
```

---

## Практические применения в микросервисах

### 1. Обработка событий

```go
// Функциональный подход к обработке событий
type EventHandler[T any] func(T) error

type EventProcessor[T any] struct {
    handlers []EventHandler[T]
}

func NewEventProcessor[T any]() *EventProcessor[T] {
    return &EventProcessor[T]{
        handlers: make([]EventHandler[T], 0),
    }
}

func (p *EventProcessor[T]) AddHandler(handler EventHandler[T]) *EventProcessor[T] {
    p.handlers = append(p.handlers, handler)
    return p
}

func (p *EventProcessor[T]) Process(event T) error {
    for _, handler := range p.handlers {
        if err := handler(event); err != nil {
            return err
        }
    }
    return nil
}

// Использование
func SetupUserEventProcessor() *EventProcessor[UserCreatedEvent] {
    return NewEventProcessor[UserCreatedEvent]().
        AddHandler(SendWelcomeEmail).
        AddHandler(CreateUserProfile).
        AddHandler(UpdateAnalytics).
        AddHandler(SendToDataWarehouse)
}
```

### 2. Middleware как функции высшего порядка

```go
// Middleware для HTTP обработчиков
type HandlerFunc func(http.ResponseWriter, *http.Request)
type Middleware func(HandlerFunc) HandlerFunc

func LoggingMiddleware(logger Logger) Middleware {
    return func(next HandlerFunc) HandlerFunc {
        return func(w http.ResponseWriter, r *http.Request) {
            logger.Info("Request received", "method", r.Method, "url", r.URL.String())
            next(w, r)
            logger.Info("Request processed", "method", r.Method, "url", r.URL.String())
        }
    }
}

func AuthMiddleware(authService AuthService) Middleware {
    return func(next HandlerFunc) HandlerFunc {
        return func(w http.ResponseWriter, r *http.Request) {
            token := r.Header.Get("Authorization")
            if !authService.IsValid(token) {
                http.Error(w, "Unauthorized", http.StatusUnauthorized)
                return
            }
            next(w, r)
        }
    }
}

// Композиция middleware
func Chain(middlewares ...Middleware) Middleware {
    return func(handler HandlerFunc) HandlerFunc {
        for i := len(middlewares) - 1; i >= 0; i-- {
            handler = middlewares[i](handler)
        }
        return handler
    }
}

// Использование
func SetupHandlers() {
    middleware := Chain(
        LoggingMiddleware(logger),
        AuthMiddleware(authService),
        RateLimitMiddleware(rateLimiter),
    )
    
    http.HandleFunc("/users", middleware(CreateUserHandler))
}
```

---

## Преимущества функционального подхода

### 1. **Предсказуемость**
```go
// Чистые функции всегда дают одинаковый результат
func CalculateTotal(items []Item) float64 {
    total := 0.0
    for _, item := range items {
        total += item.Price
    }
    return total
}
```

### 2. **Легкость тестирования**
```go
func TestCalculateTotal(t *testing.T) {
    items := []Item{
        {Price: 10.0},
        {Price: 20.0},
    }
    
    result := CalculateTotal(items)
    assert.Equal(t, 30.0, result)
}
```

### 3. **Композируемость**
```go
// Функции легко комбинируются
pipeline := Compose(
    ValidateData,
    NormalizeData,
    ProcessData,
)
```

### 4. **Параллелизм**
```go
// Неизменяемые данные безопасны для параллельной обработки
func ProcessItemsInParallel(items []Item) []ProcessedItem {
    results := make([]ProcessedItem, len(items))
    
    var wg sync.WaitGroup
    for i, item := range items {
        wg.Add(1)
        go func(i int, item Item) {
            defer wg.Done()
            results[i] = ProcessItem(item) // Безопасно - нет мутаций
        }(i, item)
    }
    
    wg.Wait()
    return results
}
```

---

## Ограничения и компромиссы

### 1. **Производительность**
```go
// Создание новых объектов может быть дорогим
func (u User) WithEmail(email string) User {
    return User{
        id:       u.id,
        email:    email,
        name:     u.name,
        // ... копирование всех полей
    }
}

// Решение: используйте для важных данных, оптимизируйте критические пути
```

### 2. **Память**
```go
// Функциональный подход может потреблять больше памяти
func ProcessLargeDataset(data []LargeObject) []ProcessedObject {
    // Создается новый слайс
    return Map(data, ProcessObject)
}

// Решение: используйте streaming или chunking для больших данных
```

---

## 🔗 Связанные темы

### 🏗️ Основы
- **[Development Principles](development-principles.md)** - Принципы разработки
- **[SOLID Principles](solid-principles.md)** - Принципы ООП
- **[Design Patterns](gof-patterns.md)** - Паттерны проектирования

### 🏛️ Архитектура
- **[Clean Architecture](clean-architecture.md)** - Чистая архитектура
- **[Microservices Architecture](../architecture/microservices-architecture.md)** - ФП в микросервисах

### ⚙️ Технические навыки
- **[Concurrency & Async](../technical-skills/concurrency-async.md)** - Параллелизм и ФП
- **[Testing](../technical-skills/testing.md)** - Тестирование функционального кода

---

**Путь обучения**: [Development Principles](development-principles.md) → Functional Programming → [Clean Architecture](clean-architecture.md)  
**Сложность**: ⭐⭐⭐ (3/5)  
**Время изучения**: 2-3 недели  
**Практика**: Рефакторинг существующего кода с применением ФП принципов 