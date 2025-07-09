# SOLID Принципы проектирования

**SOLID** — это аббревиатура пяти основных принципов объектно-ориентированного программирования и проектирования, сформулированных Робертом Мартином (Uncle Bob). Эти принципы помогают создавать более поддерживаемый, расширяемый и тестируемый код.

## Обзор принципов

1. **S** - Single Responsibility Principle (SRP) - Принцип единственной ответственности
2. **O** - Open/Closed Principle (OCP) - Принцип открытости/закрытости
3. **L** - Liskov Substitution Principle (LSP) - Принцип подстановки Лисков
4. **I** - Interface Segregation Principle (ISP) - Принцип разделения интерфейсов
5. **D** - Dependency Inversion Principle (DIP) - Принцип инверсии зависимостей

---

## 1. Single Responsibility Principle (SRP)

### Определение
**Класс должен иметь только одну причину для изменения.** Каждый класс должен быть ответственен только за одну функциональность.

### Проблема нарушения SRP
```go
// ❌ ПЛОХОЙ ПРИМЕР: Нарушение Single Responsibility Principle
// Структура User содержит слишком много ответственностей

type User struct {
    ID    string  // Уникальный идентификатор пользователя
    Email string  // Email адрес пользователя  
    Name  string  // Имя пользователя
}

// ❌ ПРОБЛЕМА: Пользователь отвечает за сохранение в базу данных
// Это ответственность репозитория/DAO слоя
func (u *User) Save() error {
    // Получение подключения к базе данных
    // Нарушает разделение ответственностей - бизнес-логика смешана с персистентностью
    db := getDB()
    // Сохранение пользователя в базу данных
    // При изменении БД придется менять эту структуру
    return db.Save(u)
}

// ❌ ПРОБЛЕМА: Пользователь отвечает за отправку email
// Это ответственность почтового сервиса/уведомлений
func (u *User) SendEmail(message string) error {
    // Получение почтового сервиса
    // Смешивает логику пользователя с логикой уведомлений
    emailService := getEmailService()
    // Отправка email через внешний сервис
    // При изменении способа отправки email придется менять User
    return emailService.Send(u.Email, message)
}

// ❌ ПРОБЛЕМА: Пользователь отвечает за валидацию email
// Это ответственность валидатора
func (u *User) ValidateEmail() error {
    // Простая валидация email адреса
    // При усложнении правил валидации придется менять User
    if !strings.Contains(u.Email, "@") {
        return errors.New("invalid email")
    }
    return nil
}

// ИТОГ: Этот класс нарушает SRP, так как имеет несколько причин для изменения:
// 1. Изменение структуры данных пользователя
// 2. Изменение способа сохранения в БД
// 3. Изменение способа отправки email
// 4. Изменение правил валидации email
```

### Правильное применение SRP
```go
// ✅ ПРАВИЛЬНЫЙ ПРИМЕР: Соблюдение Single Responsibility Principle
// Каждый компонент имеет единственную ответственность

// ==================== ДОМЕННАЯ МОДЕЛЬ ====================
// Отвечает ТОЛЬКО за представление данных пользователя и базовую бизнес-логику
type User struct {
    ID    string  // Уникальный идентификатор
    Email string  // Email адрес
    Name  string  // Имя пользователя
}

// Единственная ответственность: валидация бизнес-правил пользователя
func (u *User) IsValid() error {
    // Проверка обязательности email
    if u.Email == "" {
        return errors.New("email cannot be empty")
    }
    // Проверка обязательности имени
    if u.Name == "" {
        return errors.New("name cannot be empty")
    }
    return nil
}

// ==================== СЛОЙ ПЕРСИСТЕНТНОСТИ ====================
// Отвечает ТОЛЬКО за сохранение и извлечение данных из хранилища
type UserRepository interface {
    Save(user *User) error        // Сохранение пользователя
    FindByID(id string) (*User, error)  // Поиск по ID
}

// Конкретная реализация для PostgreSQL
type PostgresUserRepository struct {
    db *sql.DB  // Подключение к базе данных PostgreSQL
}

// Реализация сохранения в PostgreSQL
func (r *PostgresUserRepository) Save(user *User) error {
    // Логика сохранения в PostgreSQL:
    // - Подготовка SQL запроса
    // - Выполнение INSERT/UPDATE
    // - Обработка ошибок БД
    return nil
}

// ==================== СЕРВИС УВЕДОМЛЕНИЙ ====================
// Отвечает ТОЛЬКО за отправку email сообщений
type EmailService interface {
    SendEmail(to, subject, body string) error  // Отправка email
}

// Конкретная реализация через SMTP
type SMTPEmailService struct {
    host string  // SMTP сервер
    port int     // Порт SMTP сервера
}

// Реализация отправки через SMTP протокол
func (s *SMTPEmailService) SendEmail(to, subject, body string) error {
    // Логика отправки email через SMTP:
    // - Установка соединения с SMTP сервером
    // - Аутентификация
    // - Отправка сообщения
    // - Закрытие соединения
    return nil
}

// ==================== СЕРВИС ВАЛИДАЦИИ ====================
// Отвечает ТОЛЬКО за валидацию данных пользователя
type UserValidator struct{}

// Валидация email адреса
func (v *UserValidator) ValidateEmail(email string) error {
    // Проверка формата email (упрощенная)
    // В реальности используется regexp или библиотека валидации
    if !strings.Contains(email, "@") {
        return errors.New("invalid email format")
    }
    return nil
}

// ПРЕИМУЩЕСТВА такого разделения:
// 1. User изменяется только при изменении бизнес-логики пользователя
// 2. UserRepository изменяется только при изменении логики персистентности
// 3. EmailService изменяется только при изменении логики отправки email
// 4. UserValidator изменяется только при изменении правил валидации
// 5. Легко тестировать каждый компонент изолированно
// 6. Легко заменить реализацию любого компонента
```

### Применение в проекте
```go
// services/user-service/internal/domain/user.go
type User struct {
    id        uuid.UUID
    email     string
    name      string
    status    UserStatus
    createdAt time.Time
    updatedAt time.Time
}

// Только бизнес-логика валидации
func (u *User) IsValid() error {
    if u.email == "" {
        return errors.New("email cannot be empty")
    }
    return nil
}

// services/user-service/internal/infrastructure/repository.go
// Только ответственность за персистентность
type PostgresUserRepository struct {
    db *sql.DB
}
```

---

## 2. Open/Closed Principle (OCP)

### Определение
**Программные сущности должны быть открыты для расширения, но закрыты для модификации.** Новая функциональность должна добавляться через расширение, а не изменение существующего кода.

### Проблема нарушения OCP
```go
// ❌ ПЛОХОЙ ПРИМЕР: Нарушение Open/Closed Principle
// Класс закрыт для расширения - нужно модифицировать код для добавления новых типов

type PaymentProcessor struct{}

// ❌ ПРОБЛЕМА: Для добавления нового типа платежа нужно модифицировать этот метод
// Это нарушает принцип "закрыто для модификации"
func (p *PaymentProcessor) ProcessPayment(paymentType string, amount float64) error {
    // Switch statement - основная проблема
    // При добавлении нового типа платежа (например, "bitcoin") нужно:
    // 1. Изменить этот метод
    // 2. Добавить новый case
    // 3. Добавить новый метод обработки
    // 4. Пересобрать и протестировать весь PaymentProcessor
    switch paymentType {
    case "credit_card":
        return p.processCreditCard(amount)  // Обработка кредитной карты
    case "paypal":
        return p.processPayPal(amount)      // Обработка PayPal
    // Для добавления нового типа нужно модифицировать этот метод
    default:
        return errors.New("unsupported payment type")
    }
}

// Приватные методы для обработки каждого типа платежа
func (p *PaymentProcessor) processCreditCard(amount float64) error {
    // Логика обработки кредитной карты
    return nil
}

func (p *PaymentProcessor) processPayPal(amount float64) error {
    // Логика обработки PayPal
    return nil
}

// ПРОБЛЕМЫ такого подхода:
// 1. Нарушение OCP - нужно модифицировать код для новых типов
// 2. Рост сложности - switch statement становится все больше
// 3. Тестирование - нужно перетестировать весь класс
// 4. Ответственность - один класс знает о всех типах платежей
// 5. Coupling - высокая связанность между типами платежей
```

### Правильное применение OCP
```go
// ✅ ПРАВИЛЬНЫЙ ПРИМЕР: Соблюдение Open/Closed Principle
// Открыто для расширения (новые реализации), закрыто для модификации (интерфейс неизменен)

// ==================== ИНТЕРФЕЙС - ЗАКРЫТ ДЛЯ МОДИФИКАЦИИ ====================
// Интерфейс определяет контракт, который не изменяется при добавлении новых типов
type PaymentProcessor interface {
    ProcessPayment(amount float64) error  // Единственный метод для обработки платежа
}

// ==================== РЕАЛИЗАЦИИ - РАСШИРЕНИЯ ФУНКЦИОНАЛЬНОСТИ ====================

// ==================== КРЕДИТНАЯ КАРТА ====================
// Конкретная реализация для обработки кредитных карт
type CreditCardProcessor struct {
    cardNumber string  // Номер карты
    expiryDate string  // Срок действия
}

// Реализация интерфейса PaymentProcessor для кредитных карт
func (c *CreditCardProcessor) ProcessPayment(amount float64) error {
    // Логика обработки кредитной карты:
    // 1. Валидация номера карты
    // 2. Проверка срока действия
    // 3. Связь с банком
    // 4. Авторизация транзакции
    // 5. Подтверждение списания
    return nil
}

// ==================== PAYPAL ====================
// Конкретная реализация для обработки PayPal платежей
type PayPalProcessor struct {
    accountEmail string  // Email аккаунта PayPal
}

// Реализация интерфейса PaymentProcessor для PayPal
func (p *PayPalProcessor) ProcessPayment(amount float64) error {
    // Логика обработки PayPal:
    // 1. Аутентификация через PayPal API
    // 2. Проверка баланса
    // 3. Создание транзакции
    // 4. Подтверждение платежа
    return nil
}

// ==================== КРИПТОВАЛЮТА ====================
// Новый тип можно добавить БЕЗ ИЗМЕНЕНИЯ существующего кода
// Это демонстрирует принцип "открыто для расширения"
type CryptoProcessor struct {
    walletAddress string  // Адрес криптовалютного кошелька
}

// Реализация интерфейса PaymentProcessor для криптовалют
func (c *CryptoProcessor) ProcessPayment(amount float64) error {
    // Логика обработки криптовалют:
    // 1. Валидация адреса кошелька
    // 2. Проверка баланса
    // 3. Создание транзакции в блокчейне
    // 4. Подтверждение транзакции
    return nil
}

// ==================== КЛИЕНТСКИЙ КОД ====================
// Сервис использует интерфейс и НЕ ИЗМЕНЯЕТСЯ при добавлении новых процессоров
type PaymentService struct {
    processor PaymentProcessor  // Зависимость от интерфейса, а не от конкретной реализации
}

// Метод для выполнения платежа - работает с любой реализацией PaymentProcessor
func (s *PaymentService) MakePayment(amount float64) error {
    // Полиморфизм - вызов метода определяется во время выполнения
    return s.processor.ProcessPayment(amount)
}

// ==================== ПРЕИМУЩЕСТВА ТАКОГО ПОДХОДА ====================
// 1. Новые типы платежей добавляются без изменения существующего кода
// 2. Каждый процессор инкапсулирует свою логику
// 3. Легко тестировать - можно создать mock реализации
// 4. Низкая связанность - процессоры не знают друг о друге
// 5. Высокая когезия - каждый процессор отвечает только за свой тип платежа

// ==================== ПРИМЕР ИСПОЛЬЗОВАНИЯ ====================
// func main() {
//     // Можно легко менять реализацию
//     var processor PaymentProcessor
//     
//     processor = &CreditCardProcessor{cardNumber: "1234", expiryDate: "12/25"}
//     service := &PaymentService{processor: processor}
//     service.MakePayment(100.0)
//     
//     // Переключение на другой тип без изменения кода
//     processor = &PayPalProcessor{accountEmail: "user@example.com"}
//     service.processor = processor
//     service.MakePayment(200.0)
// }
```

### Применение в проекте
```go
// services/user-service/internal/application/commands.go
// Интерфейс закрыт для модификации
type Command interface {
    Execute() error
}

// Новые команды добавляются через расширение
type CreateUserCommand struct {
    Email string
    Name  string
}

func (c *CreateUserCommand) Execute() error {
    // Логика создания пользователя
    return nil
}

type BlockUserCommand struct {
    UserID string
    Reason string
}

func (c *BlockUserCommand) Execute() error {
    // Логика блокировки пользователя
    return nil
}
```

---

## 3. Liskov Substitution Principle (LSP)

### Определение
**Объекты суперкласса должны заменяться объектами подклассов без нарушения корректности программы.** Подклассы должны быть взаимозаменяемы со своими базовыми классами.

### Проблема нарушения LSP
```go
// ❌ ПЛОХО: нарушение LSP
type Rectangle struct {
    width  float64
    height float64
}

func (r *Rectangle) SetWidth(width float64) {
    r.width = width
}

func (r *Rectangle) SetHeight(height float64) {
    r.height = height
}

func (r *Rectangle) Area() float64 {
    return r.width * r.height
}

type Square struct {
    Rectangle
}

// Нарушение LSP: изменение поведения базового класса
func (s *Square) SetWidth(width float64) {
    s.width = width
    s.height = width // Неожиданное поведение!
}

func (s *Square) SetHeight(height float64) {
    s.width = height  // Неожиданное поведение!
    s.height = height
}

// Тест, который покажет нарушение LSP
func TestArea(shape *Rectangle) bool {
    shape.SetWidth(5)
    shape.SetHeight(4)
    return shape.Area() == 20 // Для Square вернет false!
}
```

### Правильное применение LSP
```go
// ✅ ХОРОШО: соблюдение LSP

// Абстрактный интерфейс
type Shape interface {
    Area() float64
    Perimeter() float64
}

// Реализации, которые можно подставлять друг вместо друга
type Rectangle struct {
    width  float64
    height float64
}

func (r *Rectangle) Area() float64 {
    return r.width * r.height
}

func (r *Rectangle) Perimeter() float64 {
    return 2 * (r.width + r.height)
}

type Square struct {
    side float64
}

func (s *Square) Area() float64 {
    return s.side * s.side
}

func (s *Square) Perimeter() float64 {
    return 4 * s.side
}

// Функция, которая работает с любой реализацией Shape
func CalculateTotalArea(shapes []Shape) float64 {
    total := 0.0
    for _, shape := range shapes {
        total += shape.Area()
    }
    return total
}
```

### Применение в проекте
```go
// services/user-service/internal/domain/repository.go
type UserRepository interface {
    Save(user *User) error
    FindByID(id uuid.UUID) (*User, error)
    FindByEmail(email string) (*User, error)
}

// Все реализации должны соблюдать контракт интерфейса
type PostgresUserRepository struct {
    db *sql.DB
}

type MongoUserRepository struct {
    collection *mongo.Collection
}

type InMemoryUserRepository struct {
    users map[uuid.UUID]*User
}

// Любую реализацию можно подставить без изменения логики
func CreateUser(repo UserRepository, email, name string) error {
    user := NewUser(email, name)
    return repo.Save(user)
}
```

---

## 4. Interface Segregation Principle (ISP)

### Определение
**Клиенты не должны зависеть от интерфейсов, которые они не используют.** Лучше иметь много специализированных интерфейсов, чем один большой.

### Проблема нарушения ISP
```go
// ❌ ПЛОХО: толстый интерфейс
type Worker interface {
    Work() error
    Eat() error
    Sleep() error
    Program() error
    Design() error
    Manage() error
}

// Робот не может есть и спать
type Robot struct{}

func (r *Robot) Work() error { return nil }
func (r *Robot) Program() error { return nil }

// Нарушение ISP: вынужден реализовывать ненужные методы
func (r *Robot) Eat() error {
    return errors.New("robots don't eat")
}

func (r *Robot) Sleep() error {
    return errors.New("robots don't sleep")
}

func (r *Robot) Design() error {
    return errors.New("this robot doesn't design")
}

func (r *Robot) Manage() error {
    return errors.New("this robot doesn't manage")
}
```

### Правильное применение ISP
```go
// ✅ ХОРОШО: разделенные интерфейсы

// Базовые интерфейсы
type Worker interface {
    Work() error
}

type Biological interface {
    Eat() error
    Sleep() error
}

type Programmer interface {
    Program() error
}

type Designer interface {
    Design() error
}

type Manager interface {
    Manage() error
}

// Реализации используют только нужные интерфейсы
type Robot struct{}

func (r *Robot) Work() error    { return nil }
func (r *Robot) Program() error { return nil }

type Human struct {
    name string
}

func (h *Human) Work() error    { return nil }
func (h *Human) Eat() error     { return nil }
func (h *Human) Sleep() error   { return nil }
func (h *Human) Program() error { return nil }

type Designer struct {
    Human
}

func (d *Designer) Design() error { return nil }

// Функции принимают только нужные интерфейсы
func AssignWork(worker Worker) error {
    return worker.Work()
}

func FeedCreature(bio Biological) error {
    return bio.Eat()
}

func RequestDesign(designer Designer) error {
    return designer.Design()
}
```

### Применение в проекте
```go
// services/user-service/internal/domain/user.go

// Разделенные интерфейсы вместо одного большого
type UserReader interface {
    FindByID(id uuid.UUID) (*User, error)
    FindByEmail(email string) (*User, error)
}

type UserWriter interface {
    Save(user *User) error
    Delete(id uuid.UUID) error
}

type UserSearcher interface {
    Search(query string) ([]*User, error)
}

// Сервисы используют только нужные интерфейсы
type UserQueryService struct {
    reader UserReader
}

type UserCommandService struct {
    writer UserWriter
}

type UserSearchService struct {
    searcher UserSearcher
}
```

---

## 5. Dependency Inversion Principle (DIP)

### Определение
**Модули высокого уровня не должны зависеть от модулей низкого уровня. Оба должны зависеть от абстракций.** Абстракции не должны зависеть от деталей, детали должны зависеть от абстракций.

### Проблема нарушения DIP
```go
// ❌ ПЛОХО: высокоуровневый модуль зависит от низкоуровневого
type PostgreSQLDatabase struct {
    connectionString string
}

func (db *PostgreSQLDatabase) Save(data string) error {
    // Прямое обращение к PostgreSQL
    return nil
}

// Высокоуровневый сервис напрямую зависит от конкретной реализации БД
type UserService struct {
    database *PostgreSQLDatabase // Жесткая зависимость!
}

func (s *UserService) CreateUser(name, email string) error {
    userData := fmt.Sprintf(`{"name": "%s", "email": "%s"}`, name, email)
    return s.database.Save(userData)
}

func NewUserService() *UserService {
    return &UserService{
        database: &PostgreSQLDatabase{
            connectionString: "postgresql://...",
        },
    }
}
```

### Правильное применение DIP
```go
// ✅ ХОРОШО: зависимость от абстракции

// Абстракция (интерфейс)
type Database interface {
    Save(data string) error
    Find(id string) (string, error)
}

// Низкоуровневые модули зависят от абстракции
type PostgreSQLDatabase struct {
    connectionString string
}

func (db *PostgreSQLDatabase) Save(data string) error {
    // Реализация для PostgreSQL
    return nil
}

func (db *PostgreSQLDatabase) Find(id string) (string, error) {
    // Реализация поиска для PostgreSQL
    return "", nil
}

type MongoDatabase struct {
    connectionString string
}

func (db *MongoDatabase) Save(data string) error {
    // Реализация для MongoDB
    return nil
}

func (db *MongoDatabase) Find(id string) (string, error) {
    // Реализация поиска для MongoDB
    return "", nil
}

// Высокоуровневый модуль зависит от абстракции
type UserService struct {
    database Database // Зависимость от интерфейса!
}

func (s *UserService) CreateUser(name, email string) error {
    userData := fmt.Sprintf(`{"name": "%s", "email": "%s"}`, name, email)
    return s.database.Save(userData)
}

// Dependency Injection через конструктор
func NewUserService(db Database) *UserService {
    return &UserService{
        database: db,
    }
}

// Использование
func main() {
    // Можно легко переключать реализации
    postgresDB := &PostgreSQLDatabase{connectionString: "postgresql://..."}
    mongoDB := &MongoDatabase{connectionString: "mongodb://..."}
    
    userService1 := NewUserService(postgresDB)
    userService2 := NewUserService(mongoDB)
    
    // Сервисы работают одинаково с разными БД
    userService1.CreateUser("John", "john@example.com")
    userService2.CreateUser("Jane", "jane@example.com")
}
```

### Применение в проекте
```go
// services/user-service/internal/application/services.go

// Высокоуровневый сервис зависит от абстракций
type UserApplicationService struct {
    userRepo   domain.UserRepository // Интерфейс
    eventBus   EventBus              // Интерфейс
    validator  UserValidator         // Интерфейс
}

func NewUserApplicationService(
    userRepo domain.UserRepository,
    eventBus EventBus,
    validator UserValidator,
) *UserApplicationService {
    return &UserApplicationService{
        userRepo:  userRepo,
        eventBus:  eventBus,
        validator: validator,
    }
}

// services/user-service/cmd/main.go
func main() {
    // Dependency Injection
    db := setupDatabase()
    userRepo := infrastructure.NewPostgresUserRepository(db)
    eventBus := infrastructure.NewRedisEventBus()
    validator := application.NewUserValidator()
    
    userService := application.NewUserApplicationService(
        userRepo,
        eventBus,
        validator,
    )
    
    // Сервис не знает о конкретных реализациях
}
```

---

## Преимущества применения SOLID

### 1. Поддерживаемость
```go
// Легко модифицировать и расширять
type PaymentService struct {
    processor PaymentProcessor // Можно заменить любой реализацией
    logger    Logger          // Можно заменить любой реализацией
}
```

### 2. Тестируемость
```go
// Легко тестировать с mock-объектами
func TestUserService_CreateUser(t *testing.T) {
    mockRepo := &MockUserRepository{}
    mockEventBus := &MockEventBus{}
    
    service := NewUserService(mockRepo, mockEventBus)
    
    err := service.CreateUser("test@example.com", "Test User")
    assert.NoError(t, err)
}
```

### 3. Переиспользуемость
```go
// Компоненты можно переиспользовать в разных контекстах
func NewUserService(repo UserRepository) *UserService {
    return &UserService{repo: repo}
}

// Тот же сервис с разными репозиториями
postgresService := NewUserService(postgresRepo)
mongoService := NewUserService(mongoRepo)
inMemoryService := NewUserService(inMemoryRepo)
```

### 4. Слабая связанность
```go
// Компоненты не зависят друг от друга напрямую
type OrderService struct {
    userService UserServiceInterface    // Интерфейс, а не конкретный тип
    paymentService PaymentServiceInterface // Интерфейс, а не конкретный тип
}
```

---

## Анти-паттерны и нарушения SOLID

### 1. God Object (нарушение SRP)
```go
// ❌ Класс делает слишком много
type UserManager struct {
    // Управление пользователями
    // Отправка email
    // Валидация
    // Аутентификация
    // Авторизация
    // Логирование
    // Кэширование
}
```

### 2. Rigid Design (нарушение OCP)
```go
// ❌ Сложно расширять без модификации существующего кода
func ProcessPayment(paymentType string) {
    switch paymentType {
    case "card": // Hardcoded типы
    case "paypal":
    // Добавление нового типа требует изменения этой функции
    }
}
```

### 3. Fragile Design (нарушение LSP)
```go
// ❌ Подклассы ломают поведение базового класса
type Bird interface {
    Fly() error
}

type Penguin struct{} 

func (p *Penguin) Fly() error {
    return errors.New("penguins can't fly") // Нарушение контракта
}
```

### 4. Interface Pollution (нарушение ISP)
```go
// ❌ Слишком большие интерфейсы
type DatabaseInterface interface {
    // SQL методы
    Query(sql string) error
    Execute(sql string) error
    // NoSQL методы  
    FindDocument(collection, id string) error
    InsertDocument(collection string, doc interface{}) error
    // Cache методы
    Get(key string) interface{}
    Set(key string, value interface{}) error
    // File методы
    ReadFile(path string) ([]byte, error)
    WriteFile(path string, data []byte) error
}
```

### 5. Concrete Dependencies (нарушение DIP)
```go
// ❌ Зависимость от конкретных реализаций
type UserService struct {
    postgresRepo *PostgresUserRepository // Конкретный тип!
    redisCache   *RedisCache             // Конкретный тип!
    smtpMailer   *SMTPMailer             // Конкретный тип!
}
```

---

## Практические рекомендации

### 1. Применяйте постепенно
```go
// Начните с простых принципов
// 1. Выделите интерфейсы (DIP)
// 2. Разделите ответственности (SRP)
// 3. Сделайте расширяемым (OCP)
```

### 2. Не переусложняйте
```go
// Для простых случаев SOLID может быть избыточным
type Calculator struct{}

func (c *Calculator) Add(a, b int) int {
    return a + b // Простая функция не требует интерфейса
}
```

### 3. Используйте инструменты анализа
```bash
# Анализ кода на соответствие принципам
golangci-lint run
go vet ./...
staticcheck ./...
```

### 4. Тестируйте принципы
```go
// Хорошие тесты показывают соблюдение SOLID
func TestUserService_DependencyInjection(t *testing.T) {
    // Если легко создать mock - DIP соблюден
    mockRepo := &MockUserRepository{}
    service := NewUserService(mockRepo)
    
    // Если легко тестировать - SRP соблюден
    err := service.CreateUser("test@example.com", "Test")
    assert.NoError(t, err)
}
```

---

## 🔗 Связанные темы

### ➡️ Следующие шаги
- **[Clean Architecture](clean-architecture.md)** - Применение SOLID в архитектуре
- **[Design Patterns](gof-patterns.md)** - Паттерны, основанные на SOLID
- **[DDD Patterns](ddd-patterns.md)** - Доменное проектирование с SOLID

### 🏛️ Архитектурное применение
- **[Microservices Architecture](../architecture/microservices-architecture.md)** - SOLID в микросервисах
- **[API Design](../architecture/api-design.md)** - Проектирование API с SOLID

### ⚙️ Технические навыки
- **[Testing](../technical-skills/testing.md)** - Тестирование SOLID кода
- **[Senior Technical Mastery](../technical-skills/senior-technical-mastery.md)** - Мастерство SOLID

---

**Путь обучения**: [Fundamentals Overview](README.md) → SOLID Principles → [Clean Architecture](clean-architecture.md)  
**Сложность**: ⭐⭐⭐ (3/5)  
**Время изучения**: 2-3 недели  
**Практика**: Рефакторинг кода в `user-service/`, `order-service/` 