# Принципы разработки программного обеспечения

Основные принципы разработки — это фундаментальные правила, которые помогают создавать качественный, поддерживаемый и эффективный код. Эти принципы работают на всех уровнях: от отдельных функций до архитектуры всего приложения.

## Основные принципы

### 1. DRY (Don't Repeat Yourself)
### 2. KISS (Keep It Simple, Stupid)
### 3. YAGNI (You Aren't Gonna Need It)
### 4. SoC (Separation of Concerns)
### 5. LoD (Law of Demeter)
### 6. Fail Fast
### 7. Composition over Inheritance

---

## 1. DRY - Don't Repeat Yourself

### Определение
**Каждая часть знания должна иметь единственное, недвусмысленное, авторитетное представление в системе.**

### Проблема дублирования кода

```go
// ❌ ПЛОХО: дублирование логики валидации
func CreateUser(email, name string) error {
    // Валидация email
    if email == "" {
        return errors.New("email cannot be empty")
    }
    if !strings.Contains(email, "@") {
        return errors.New("invalid email format")
    }
    if len(email) > 254 {
        return errors.New("email too long")
    }
    
    // Валидация name
    if name == "" {
        return errors.New("name cannot be empty")
    }
    if len(name) < 2 {
        return errors.New("name too short")
    }
    
    // Создание пользователя
    user := &User{Email: email, Name: name}
    return userRepo.Save(user)
}

func UpdateUser(id string, email, name string) error {
    // Та же валидация email - ДУБЛИРОВАНИЕ!
    if email == "" {
        return errors.New("email cannot be empty")
    }
    if !strings.Contains(email, "@") {
        return errors.New("invalid email format")
    }
    if len(email) > 254 {
        return errors.New("email too long")
    }
    
    // Та же валидация name - ДУБЛИРОВАНИЕ!
    if name == "" {
        return errors.New("name cannot be empty")
    }
    if len(name) < 2 {
        return errors.New("name too short")
    }
    
    // Обновление пользователя
    user := &User{ID: id, Email: email, Name: name}
    return userRepo.Update(user)
}
```

### Правильное применение DRY

```go
// ✅ ХОРОШО: извлечение общей логики

// Валидатор с единственной ответственностью
type UserValidator struct{}

func (v *UserValidator) ValidateEmail(email string) error {
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

func (v *UserValidator) ValidateName(name string) error {
    if name == "" {
        return errors.New("name cannot be empty")
    }
    if len(name) < 2 {
        return errors.New("name too short")
    }
    return nil
}

func (v *UserValidator) ValidateUser(email, name string) error {
    if err := v.ValidateEmail(email); err != nil {
        return err
    }
    if err := v.ValidateName(name); err != nil {
        return err
    }
    return nil
}

// Использование единого валидатора
func CreateUser(email, name string) error {
    validator := &UserValidator{}
    if err := validator.ValidateUser(email, name); err != nil {
        return err
    }
    
    user := &User{Email: email, Name: name}
    return userRepo.Save(user)
}

func UpdateUser(id string, email, name string) error {
    validator := &UserValidator{}
    if err := validator.ValidateUser(email, name); err != nil {
        return err
    }
    
    user := &User{ID: id, Email: email, Name: name}
    return userRepo.Update(user)
}
```

### Применение в проекте

```go
// services/user-service/internal/domain/validators/user_validator.go
type UserValidator struct {
    emailRegex *regexp.Regexp
}

func NewUserValidator() *UserValidator {
    return &UserValidator{
        emailRegex: regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`),
    }
}

func (v *UserValidator) ValidateCreateUserRequest(req CreateUserRequest) error {
    if err := v.ValidateEmail(req.Email); err != nil {
        return fmt.Errorf("email validation failed: %w", err)
    }
    if err := v.ValidateName(req.Name); err != nil {
        return fmt.Errorf("name validation failed: %w", err)
    }
    return nil
}

// Общие HTTP утилиты
// services/user-service/internal/adapters/http/utils.go
func WriteJSONResponse(w http.ResponseWriter, code int, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(code)
    json.NewEncoder(w).Encode(data)
}

func WriteErrorResponse(w http.ResponseWriter, code int, message string) {
    WriteJSONResponse(w, code, map[string]string{"error": message})
}
```

### Когда НЕ применять DRY

```go
// ❌ ПЛОХО: преждевременная абстракция
func ProcessData(data interface{}) interface{} {
    // Слишком общая функция, которая пытается обработать все
    switch v := data.(type) {
    case User:
        return processUser(v)
    case Order:
        return processOrder(v)
    case Product:
        return processProduct(v)
    default:
        return nil
    }
}

// ✅ ХОРОШО: специализированные функции
func ProcessUser(user User) ProcessedUser {
    // Специфичная логика для пользователя
    return ProcessedUser{...}
}

func ProcessOrder(order Order) ProcessedOrder {
    // Специфичная логика для заказа
    return ProcessedOrder{...}
}
```

---

## 2. KISS - Keep It Simple, Stupid

### Определение
**Простота должна быть ключевой целью в дизайне, и излишняя сложность должна быть избегнута.**

### Проблема излишней сложности

```go
// ❌ ПЛОХО: излишне сложная реализация
type UserManager struct {
    users                map[string]*User
    cache                map[string]*CachedUser
    indexByEmail         map[string]string
    indexByName          map[string][]string
    eventListeners       []EventListener
    validationStrategies map[string]ValidationStrategy
    transformPipeline    []TransformFunc
}

func (um *UserManager) CreateUser(data map[string]interface{}) (*User, error) {
    // Сложная логика с множеством условий
    if strategy, exists := um.validationStrategies[data["type"].(string)]; exists {
        if err := strategy.Validate(data); err != nil {
            return nil, err
        }
    }
    
    // Применение цепочки трансформаций
    transformedData := data
    for _, transform := range um.transformPipeline {
        transformedData = transform(transformedData)
    }
    
    // Создание пользователя с множеством проверок
    user := &User{}
    if err := um.populateUserFromData(user, transformedData); err != nil {
        return nil, err
    }
    
    // Множественное индексирование
    um.users[user.ID] = user
    um.indexByEmail[user.Email] = user.ID
    um.indexByName[user.Name] = append(um.indexByName[user.Name], user.ID)
    
    // Уведомления
    for _, listener := range um.eventListeners {
        listener.OnUserCreated(user)
    }
    
    return user, nil
}
```

### Правильное применение KISS

```go
// ✅ ХОРОШО: простая и понятная реализация
type UserService struct {
    repo      UserRepository
    validator UserValidator
}

func NewUserService(repo UserRepository, validator UserValidator) *UserService {
    return &UserService{
        repo:      repo,
        validator: validator,
    }
}

func (s *UserService) CreateUser(email, name string) (*User, error) {
    // Простая валидация
    if err := s.validator.ValidateUser(email, name); err != nil {
        return nil, err
    }
    
    // Простое создание
    user := &User{
        ID:        generateID(),
        Email:     email,
        Name:      name,
        CreatedAt: time.Now(),
    }
    
    // Простое сохранение
    if err := s.repo.Save(user); err != nil {
        return nil, err
    }
    
    return user, nil
}

func (s *UserService) GetUser(id string) (*User, error) {
    return s.repo.FindByID(id)
}

func (s *UserService) GetUserByEmail(email string) (*User, error) {
    return s.repo.FindByEmail(email)
}
```

### Применение в проекте

```go
// services/user-service/internal/usecases/user_usecase.go
// Простая структура use case
type UserUseCase struct {
    userRepo UserRepository
    logger   Logger
}

func (uc *UserUseCase) CreateUser(ctx context.Context, req CreateUserRequest) (*User, error) {
    // Простая последовательность действий
    user, err := NewUser(req.Email, req.Name)
    if err != nil {
        return nil, err
    }
    
    if err := uc.userRepo.Save(ctx, user); err != nil {
        uc.logger.Error("Failed to save user", "error", err)
        return nil, err
    }
    
    uc.logger.Info("User created successfully", "user_id", user.ID)
    return user, nil
}
```

---

## 3. YAGNI - You Aren't Gonna Need It

### Определение
**Не добавляйте функциональность, пока она действительно не понадобится.**

### Проблема преждевременной оптимизации

```go
// ❌ ПЛОХО: избыточная функциональность "на будущее"
type UserService struct {
    primaryRepo   UserRepository
    secondaryRepo UserRepository  // "Может понадобится для failover"
    cacheRepo     CacheRepository // "Может понадобится для кэширования"
    eventBus      EventBus        // "Может понадобится для events"
    metrics       MetricsCollector // "Может понадобится для мониторинга"
    circuitBreaker CircuitBreaker // "Может понадобится для resilience"
}

func (s *UserService) CreateUser(req CreateUserRequest) (*User, error) {
    // Сложная логика "на будущее"
    s.metrics.Increment("user.create.attempts")
    
    // Проверка circuit breaker
    if !s.circuitBreaker.AllowRequest() {
        s.metrics.Increment("user.create.circuit_breaker_open")
        return nil, errors.New("service temporarily unavailable")
    }
    
    user, err := NewUser(req.Email, req.Name)
    if err != nil {
        s.metrics.Increment("user.create.validation_errors")
        return nil, err
    }
    
    // Попытка сохранения в primary
    if err := s.primaryRepo.Save(user); err != nil {
        s.metrics.Increment("user.create.primary_repo_errors")
        
        // Fallback на secondary (который может никогда не понадобиться)
        if err := s.secondaryRepo.Save(user); err != nil {
            s.metrics.Increment("user.create.secondary_repo_errors")
            s.circuitBreaker.RecordFailure()
            return nil, err
        }
    }
    
    // Сохранение в кэш (который может не понадобиться)
    s.cacheRepo.Set(user.ID, user)
    
    // Публикация события (которое может не понадобиться)
    s.eventBus.Publish("user.created", user)
    
    s.metrics.Increment("user.create.success")
    s.circuitBreaker.RecordSuccess()
    
    return user, nil
}
```

### Правильное применение YAGNI

```go
// ✅ ХОРОШО: только необходимая функциональность
type UserService struct {
    repo UserRepository
}

func NewUserService(repo UserRepository) *UserService {
    return &UserService{
        repo: repo,
    }
}

func (s *UserService) CreateUser(req CreateUserRequest) (*User, error) {
    user, err := NewUser(req.Email, req.Name)
    if err != nil {
        return nil, err
    }
    
    if err := s.repo.Save(user); err != nil {
        return nil, err
    }
    
    return user, nil
}

// Дополнительная функциональность добавляется по мере необходимости
func (s *UserService) GetUser(id string) (*User, error) {
    return s.repo.FindByID(id)
}

// Позже, когда действительно понадобится:
// func (s *UserService) CacheUser(user *User) { ... }
// func (s *UserService) PublishUserCreated(user *User) { ... }
```

---

## 4. SoC - Separation of Concerns

### Определение
**Разделение программы на различные части, каждая из которых решает отдельную задачу.**

### Проблема смешивания ответственностей

```go
// ❌ ПЛОХО: смешивание HTTP, бизнес-логики и данных
func CreateUserHandler(w http.ResponseWriter, r *http.Request) {
    // HTTP логика
    body, err := io.ReadAll(r.Body)
    if err != nil {
        http.Error(w, "Bad request", http.StatusBadRequest)
        return
    }
    
    var req map[string]string
    if err := json.Unmarshal(body, &req); err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }
    
    // Бизнес-логика валидации
    email := req["email"]
    name := req["name"]
    
    if email == "" {
        http.Error(w, "Email is required", http.StatusBadRequest)
        return
    }
    if !strings.Contains(email, "@") {
        http.Error(w, "Invalid email", http.StatusBadRequest)
        return
    }
    if name == "" {
        http.Error(w, "Name is required", http.StatusBadRequest)
        return
    }
    
    // Логика работы с данными
    db, err := sql.Open("postgres", "connection_string")
    if err != nil {
        http.Error(w, "Database error", http.StatusInternalServerError)
        return
    }
    defer db.Close()
    
    _, err = db.Exec("INSERT INTO users (email, name) VALUES ($1, $2)", email, name)
    if err != nil {
        http.Error(w, "Failed to create user", http.StatusInternalServerError)
        return
    }
    
    // HTTP ответ
    response := map[string]string{
        "message": "User created successfully",
        "email":   email,
        "name":    name,
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}
```

### Правильное применение SoC

```go
// ✅ ХОРОШО: разделение ответственностей

// 1. HTTP слой - только HTTP логика
type UserController struct {
    userService UserService
}

func (c *UserController) CreateUser(w http.ResponseWriter, r *http.Request) {
    var req CreateUserRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        c.writeError(w, http.StatusBadRequest, "Invalid request")
        return
    }
    
    user, err := c.userService.CreateUser(req)
    if err != nil {
        c.writeError(w, http.StatusBadRequest, err.Error())
        return
    }
    
    c.writeJSON(w, http.StatusCreated, user)
}

// 2. Бизнес-логика - только бизнес правила
type UserService struct {
    repo      UserRepository
    validator UserValidator
}

func (s *UserService) CreateUser(req CreateUserRequest) (*User, error) {
    if err := s.validator.Validate(req); err != nil {
        return nil, err
    }
    
    user, err := NewUser(req.Email, req.Name)
    if err != nil {
        return nil, err
    }
    
    return s.repo.Save(user)
}

// 3. Слой данных - только работа с данными
type PostgresUserRepository struct {
    db *sql.DB
}

func (r *PostgresUserRepository) Save(user *User) (*User, error) {
    query := `INSERT INTO users (id, email, name) VALUES ($1, $2, $3) RETURNING id`
    
    var id string
    err := r.db.QueryRow(query, user.ID, user.Email, user.Name).Scan(&id)
    if err != nil {
        return nil, err
    }
    
    user.ID = id
    return user, nil
}

// 4. Валидация - только валидация
type UserValidator struct{}

func (v *UserValidator) Validate(req CreateUserRequest) error {
    if req.Email == "" {
        return errors.New("email is required")
    }
    if !v.isValidEmail(req.Email) {
        return errors.New("invalid email format")
    }
    if req.Name == "" {
        return errors.New("name is required")
    }
    return nil
}
```

---

## 5. LoD - Law of Demeter (Принцип минимального знания)

### Определение
**Объект должен иметь ограниченное знание о других объектах: только о тех, которые тесно связаны с ним.**

### Проблема нарушения LoD

```go
// ❌ ПЛОХО: нарушение Law of Demeter
type OrderService struct {
    userService UserService
}

func (s *OrderService) CreateOrder(userID string, items []OrderItem) (*Order, error) {
    // Длинная цепочка вызовов - нарушение LoD
    user := s.userService.GetUser(userID)
    if user.GetProfile().GetSettings().GetNotifications().IsEnabled() {
        // Отправка уведомления
        user.GetProfile().GetSettings().GetNotifications().Send("Order created")
    }
    
    // Еще одна длинная цепочка
    paymentMethod := user.GetProfile().GetPaymentSettings().GetDefaultMethod()
    if paymentMethod.GetCard().IsExpired() {
        return nil, errors.New("default payment method expired")
    }
    
    // Создание заказа
    order := &Order{
        UserID: userID,
        Items:  items,
    }
    
    return order, nil
}
```

### Правильное применение LoD

```go
// ✅ ХОРОШО: соблюдение Law of Demeter

// Инкапсуляция логики в соответствующие сервисы
type OrderService struct {
    userService         UserService
    notificationService NotificationService
    paymentService      PaymentService
}

func (s *OrderService) CreateOrder(userID string, items []OrderItem) (*Order, error) {
    // Каждый сервис предоставляет простой интерфейс
    if s.notificationService.IsNotificationEnabled(userID) {
        s.notificationService.SendOrderNotification(userID, "Order created")
    }
    
    if !s.paymentService.HasValidPaymentMethod(userID) {
        return nil, errors.New("no valid payment method")
    }
    
    order := &Order{
        UserID: userID,
        Items:  items,
    }
    
    return order, nil
}

// Каждый сервис инкапсулирует свою логику
type NotificationService struct {
    userRepo UserRepository
}

func (s *NotificationService) IsNotificationEnabled(userID string) bool {
    user, err := s.userRepo.FindByID(userID)
    if err != nil {
        return false
    }
    
    // Инкапсулированная логика
    return user.IsNotificationEnabled()
}

type PaymentService struct {
    userRepo UserRepository
}

func (s *PaymentService) HasValidPaymentMethod(userID string) bool {
    user, err := s.userRepo.FindByID(userID)
    if err != nil {
        return false
    }
    
    // Инкапсулированная логика
    return user.HasValidPaymentMethod()
}
```

---

## 6. Fail Fast

### Определение
**Система должна прекращать работу как можно раньше при обнаружении ошибки.**

### Проблема позднего обнаружения ошибок

```go
// ❌ ПЛОХО: ошибка обнаруживается поздно
func ProcessUserData(userData map[string]interface{}) error {
    // Много работы без валидации
    processedData := make(map[string]interface{})
    
    for key, value := range userData {
        // Сложная обработка
        processed := complexProcessing(value)
        processedData[key] = processed
    }
    
    // Валидация в конце - поздно!
    if userData["email"] == nil {
        return errors.New("email is required")
    }
    
    if userData["name"] == nil {
        return errors.New("name is required")
    }
    
    // Еще больше работы...
    return saveToDatabase(processedData)
}
```

### Правильное применение Fail Fast

```go
// ✅ ХОРОШО: быстрая проверка ошибок
func ProcessUserData(userData map[string]interface{}) error {
    // Fail Fast - валидация в начале
    if userData["email"] == nil {
        return errors.New("email is required")
    }
    
    if userData["name"] == nil {
        return errors.New("name is required")
    }
    
    email, ok := userData["email"].(string)
    if !ok {
        return errors.New("email must be string")
    }
    
    name, ok := userData["name"].(string)
    if !ok {
        return errors.New("name must be string")
    }
    
    // Только после валидации - основная работа
    processedData := map[string]interface{}{
        "email": strings.ToLower(email),
        "name":  strings.TrimSpace(name),
    }
    
    return saveToDatabase(processedData)
}

// Применение в конструкторах
func NewUserService(repo UserRepository, validator UserValidator) (*UserService, error) {
    // Fail Fast - проверка зависимостей
    if repo == nil {
        return nil, errors.New("repository cannot be nil")
    }
    if validator == nil {
        return nil, errors.New("validator cannot be nil")
    }
    
    return &UserService{
        repo:      repo,
        validator: validator,
    }, nil
}
```

---

## 7. Composition over Inheritance

### Определение
**Предпочитайте композицию наследованию для достижения полиморфизма и повторного использования кода.**

### Проблема наследования

```go
// ❌ ПЛОХО: глубокая иерархия наследования (псевдокод)
type Animal struct {
    Name string
}

func (a *Animal) Eat() { fmt.Println("Eating") }

type Mammal struct {
    Animal
    FurColor string
}

func (m *Mammal) GiveBirth() { fmt.Println("Giving birth") }

type Dog struct {
    Mammal
    Breed string
}

func (d *Dog) Bark() { fmt.Println("Woof!") }

type WorkingDog struct {
    Dog
    Job string
}

func (wd *WorkingDog) Work() { fmt.Println("Working") }

// Проблемы:
// 1. Жесткая связь
// 2. Сложно тестировать
// 3. Сложно изменять
// 4. Может нарушать LSP
```

### Правильное применение Composition

```go
// ✅ ХОРОШО: композиция через интерфейсы

// Определение поведений через интерфейсы
type Eater interface {
    Eat()
}

type Barker interface {
    Bark()
}

type Worker interface {
    Work()
}

// Реализации поведений
type BasicEater struct{}
func (e *BasicEater) Eat() { fmt.Println("Eating") }

type DogBarker struct{}
func (b *DogBarker) Bark() { fmt.Println("Woof!") }

type SecurityWorker struct{}
func (w *SecurityWorker) Work() { fmt.Println("Guarding") }

// Композиция через встраивание
type Dog struct {
    Name   string
    Breed  string
    eater  Eater
    barker Barker
}

func NewDog(name, breed string) *Dog {
    return &Dog{
        Name:   name,
        Breed:  breed,
        eater:  &BasicEater{},
        barker: &DogBarker{},
    }
}

func (d *Dog) Eat() { d.eater.Eat() }
func (d *Dog) Bark() { d.barker.Bark() }

type WorkingDog struct {
    Dog
    worker Worker
}

func NewWorkingDog(name, breed string, worker Worker) *WorkingDog {
    return &WorkingDog{
        Dog:    *NewDog(name, breed),
        worker: worker,
    }
}

func (wd *WorkingDog) Work() { wd.worker.Work() }
```

### Применение в проекте

```go
// services/user-service/internal/usecases/user_usecase.go
// Композиция сервисов
type UserUseCase struct {
    repo       UserRepository
    validator  UserValidator
    notifier   NotificationService
    auditor    AuditService
}

func NewUserUseCase(
    repo UserRepository,
    validator UserValidator,
    notifier NotificationService,
    auditor AuditService,
) *UserUseCase {
    return &UserUseCase{
        repo:      repo,
        validator: validator,
        notifier:  notifier,
        auditor:   auditor,
    }
}

func (uc *UserUseCase) CreateUser(req CreateUserRequest) (*User, error) {
    // Композиция поведений
    if err := uc.validator.Validate(req); err != nil {
        return nil, err
    }
    
    user, err := NewUser(req.Email, req.Name)
    if err != nil {
        return nil, err
    }
    
    if err := uc.repo.Save(user); err != nil {
        return nil, err
    }
    
    // Независимые сервисы
    uc.notifier.SendWelcomeEmail(user)
    uc.auditor.LogUserCreation(user)
    
    return user, nil
}
```

---

## Дополнительные принципы

### 8. Single Source of Truth

```go
// ✅ Одна точка истины для конфигурации
type Config struct {
    Database struct {
        Host     string
        Port     int
        Name     string
        User     string
        Password string
    }
    Server struct {
        Host string
        Port int
    }
}

func LoadConfig() (*Config, error) {
    // Загрузка из единого источника
    return loadFromFile("config.yaml")
}
```

### 9. Explicit is Better than Implicit

```go
// ✅ Явные зависимости
func NewUserService(
    repo UserRepository,        // Явно
    validator UserValidator,    // Явно
    logger Logger,             // Явно
) *UserService {
    return &UserService{
        repo:      repo,
        validator: validator,
        logger:    logger,
    }
}

// ❌ Неявные зависимости
func NewUserService() *UserService {
    return &UserService{
        repo:      getGlobalRepo(),      // Неявно
        validator: getGlobalValidator(), // Неявно
        logger:    getGlobalLogger(),    // Неявно
    }
}
```

### 10. Principle of Least Surprise

```go
// ✅ Предсказуемое поведение
func (s *UserService) GetUser(id string) (*User, error) {
    // Делает именно то, что ожидается
    return s.repo.FindByID(id)
}

// ❌ Неожиданное поведение
func (s *UserService) GetUser(id string) (*User, error) {
    user, err := s.repo.FindByID(id)
    if err != nil {
        return nil, err
    }
    
    // Неожиданные побочные эффекты
    s.logUserAccess(user)
    s.updateLastSeen(user)
    s.sendAnalyticsEvent(user)
    
    return user, nil
}
```

---

## Баланс принципов

### Когда принципы конфликтуют

```go
// DRY vs KISS - иногда дублирование проще
// ✅ Простое дублирование может быть лучше сложной абстракции
func ValidateEmail(email string) error {
    if email == "" {
        return errors.New("email is required")
    }
    // Простая проверка
    if !strings.Contains(email, "@") {
        return errors.New("invalid email")
    }
    return nil
}

func ValidateUsername(username string) error {
    if username == "" {
        return errors.New("username is required")
    }
    // Другая логика валидации
    if len(username) < 3 {
        return errors.New("username too short")
    }
    return nil
}
```

### Прагматичный подход

```go
// Начните просто, усложняйте по мере необходимости
type UserService struct {
    repo UserRepository
}

// Позже, когда понадобится:
type UserService struct {
    repo      UserRepository
    validator UserValidator  // Добавлено по мере необходимости
}

// Еще позже:
type UserService struct {
    repo      UserRepository
    validator UserValidator
    cache     CacheService   // Добавлено когда стало нужно
}
```

---

## Практические рекомендации

### 1. Начинайте с простого
```go
// Не усложняйте без необходимости
func CreateUser(email, name string) (*User, error) {
    // Простое решение
    return &User{Email: email, Name: name}, nil
}
```

### 2. Рефакторинг по мере роста
```go
// Когда появляются новые требования - рефакторинг
func CreateUser(req CreateUserRequest) (*User, error) {
    // Добавили валидацию
    if err := validateUser(req); err != nil {
        return nil, err
    }
    
    return &User{Email: req.Email, Name: req.Name}, nil
}
```

### 3. Тестирование принципов
```go
// Хорошие тесты показывают соблюдение принципов
func TestUserService_CreateUser(t *testing.T) {
    // Легко тестировать = принципы соблюдены
    mockRepo := &MockUserRepository{}
    service := NewUserService(mockRepo)
    
    user, err := service.CreateUser("test@example.com", "Test User")
    
    assert.NoError(t, err)
    assert.Equal(t, "test@example.com", user.Email)
}
```

---

## 🔗 Связанные темы

### 🏗️ Основы
- **[SOLID Principles](solid-principles.md)** - Более специфичные принципы ООП
- **[Clean Architecture](clean-architecture.md)** - Архитектурное применение принципов
- **[Design Patterns](gof-patterns.md)** - Паттерны, следующие принципам

### 🏛️ Архитектура
- **[Microservices Architecture](../architecture/microservices-architecture.md)** - Принципы в микросервисах
- **[API Design](../architecture/api-design.md)** - Принципы проектирования API

### ⚙️ Технические навыки
- **[Testing](../technical-skills/testing.md)** - Тестирование принципов
- **[Senior Technical Mastery](../technical-skills/senior-technical-mastery.md)** - Мастерство применения принципов

---

**Путь обучения**: Development Principles → [SOLID Principles](solid-principles.md) → [Clean Architecture](clean-architecture.md)  
**Сложность**: ⭐⭐ (2/5)  
**Время изучения**: 1-2 недели  
**Практика**: Анализ и рефакторинг существующего кода в проекте 