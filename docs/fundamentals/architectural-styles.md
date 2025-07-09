# Архитектурные стили

**Архитектурные стили** — это высокоуровневые паттерны организации кода, которые определяют основные принципы разделения ответственности и взаимодействия компонентов в приложении. Каждый стиль решает определенные задачи и имеет свои преимущества и недостатки.

## Основные архитектурные стили

### 1. **MVC (Model-View-Controller)**
### 2. **MVP (Model-View-Presenter)**
### 3. **MVVM (Model-View-ViewModel)**
### 4. **Layered Architecture**
### 5. **Hexagonal Architecture**
### 6. **Component-Based Architecture**

---

## 1. MVC (Model-View-Controller)

### Определение
**MVC** разделяет приложение на три взаимосвязанные части:
- **Model** — данные и бизнес-логика
- **View** — представление данных
- **Controller** — обработка пользовательского ввода

### Применение в веб-приложениях

```go
// Model - данные и бизнес-логика
type User struct {
    ID       string    `json:"id"`
    Email    string    `json:"email"`
    Name     string    `json:"name"`
    IsActive bool      `json:"is_active"`
    CreatedAt time.Time `json:"created_at"`
}

type UserModel struct {
    repo UserRepository
}

func (m *UserModel) GetUser(id string) (*User, error) {
    return m.repo.FindByID(id)
}

func (m *UserModel) CreateUser(email, name string) (*User, error) {
    user := &User{
        ID:        generateID(),
        Email:     email,
        Name:      name,
        IsActive:  true,
        CreatedAt: time.Now(),
    }
    
    return m.repo.Save(user)
}

// View - представление данных
type UserView struct {
    templates *template.Template
}

func (v *UserView) RenderUser(w http.ResponseWriter, user *User) error {
    return v.templates.ExecuteTemplate(w, "user.html", user)
}

func (v *UserView) RenderUsers(w http.ResponseWriter, users []User) error {
    return v.templates.ExecuteTemplate(w, "users.html", map[string]interface{}{
        "Users": users,
    })
}

func (v *UserView) RenderError(w http.ResponseWriter, message string) error {
    http.Error(w, message, http.StatusInternalServerError)
    return nil
}

// Controller - обработка запросов
type UserController struct {
    model *UserModel
    view  *UserView
}

func (c *UserController) GetUser(w http.ResponseWriter, r *http.Request) {
    userID := mux.Vars(r)["id"]
    
    user, err := c.model.GetUser(userID)
    if err != nil {
        c.view.RenderError(w, "User not found")
        return
    }
    
    c.view.RenderUser(w, user)
}

func (c *UserController) CreateUser(w http.ResponseWriter, r *http.Request) {
    var req struct {
        Email string `json:"email"`
        Name  string `json:"name"`
    }
    
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        c.view.RenderError(w, "Invalid request")
        return
    }
    
    user, err := c.model.CreateUser(req.Email, req.Name)
    if err != nil {
        c.view.RenderError(w, "Failed to create user")
        return
    }
    
    c.view.RenderUser(w, user)
}

// Маршрутизация
func SetupRoutes(controller *UserController) *mux.Router {
    router := mux.NewRouter()
    
    router.HandleFunc("/users/{id}", controller.GetUser).Methods("GET")
    router.HandleFunc("/users", controller.CreateUser).Methods("POST")
    
    return router
}
```

### Преимущества MVC
- **Разделение ответственности** — каждый компонент имеет четкую роль
- **Переиспользование** — модели и представления могут использоваться в разных контекстах
- **Тестируемость** — легко тестировать компоненты изолированно

### Недостатки MVC
- **Сложность** — может быть излишне сложным для простых приложений
- **Связанность** — View и Controller часто сильно связаны

---

## 2. MVP (Model-View-Presenter)

### Определение
**MVP** — это эволюция MVC, где:
- **Model** — данные и бизнес-логика
- **View** — пассивный интерфейс
- **Presenter** — посредник между Model и View

### Применение в Go

```go
// Model - остается тем же
type UserModel struct {
    repo UserRepository
}

func (m *UserModel) GetUser(id string) (*User, error) {
    return m.repo.FindByID(id)
}

// View - пассивный интерфейс
type UserViewInterface interface {
    ShowUser(user *User)
    ShowUsers(users []User)
    ShowError(message string)
    ShowLoading()
    HideLoading()
}

// Конкретная реализация View
type UserHTMLView struct {
    writer    http.ResponseWriter
    templates *template.Template
}

func (v *UserHTMLView) ShowUser(user *User) {
    v.templates.ExecuteTemplate(v.writer, "user.html", user)
}

func (v *UserHTMLView) ShowUsers(users []User) {
    v.templates.ExecuteTemplate(v.writer, "users.html", map[string]interface{}{
        "Users": users,
    })
}

func (v *UserHTMLView) ShowError(message string) {
    http.Error(v.writer, message, http.StatusInternalServerError)
}

func (v *UserHTMLView) ShowLoading() {
    // Показать индикатор загрузки
}

func (v *UserHTMLView) HideLoading() {
    // Скрыть индикатор загрузки
}

// Presenter - содержит всю логику представления
type UserPresenter struct {
    model *UserModel
    view  UserViewInterface
}

func NewUserPresenter(model *UserModel, view UserViewInterface) *UserPresenter {
    return &UserPresenter{
        model: model,
        view:  view,
    }
}

func (p *UserPresenter) GetUser(id string) {
    p.view.ShowLoading()
    
    user, err := p.model.GetUser(id)
    if err != nil {
        p.view.HideLoading()
        p.view.ShowError("User not found")
        return
    }
    
    p.view.HideLoading()
    p.view.ShowUser(user)
}

func (p *UserPresenter) CreateUser(email, name string) {
    p.view.ShowLoading()
    
    // Валидация на уровне presenter
    if email == "" || name == "" {
        p.view.HideLoading()
        p.view.ShowError("Email and name are required")
        return
    }
    
    user, err := p.model.CreateUser(email, name)
    if err != nil {
        p.view.HideLoading()
        p.view.ShowError("Failed to create user")
        return
    }
    
    p.view.HideLoading()
    p.view.ShowUser(user)
}

// HTTP Controller становится тонким слоем
type UserHTTPController struct {
    presenter *UserPresenter
}

func (c *UserHTTPController) GetUser(w http.ResponseWriter, r *http.Request) {
    userID := mux.Vars(r)["id"]
    view := &UserHTMLView{writer: w, templates: templates}
    
    // Создаем presenter для каждого запроса
    presenter := NewUserPresenter(c.presenter.model, view)
    presenter.GetUser(userID)
}
```

### Преимущества MVP
- **Лучшая тестируемость** — View полностью изолирован через интерфейс
- **Слабая связанность** — View не знает о Model
- **Переиспользование** — один Presenter может работать с разными View

### Недостатки MVP
- **Сложность** — больше кода для простых случаев
- **Presenter может стать толстым** — вся логика представления в одном месте

---

## 3. MVVM (Model-View-ViewModel)

### Определение
**MVVM** особенно популярен в приложениях с data binding:
- **Model** — данные и бизнес-логика
- **View** — UI элементы
- **ViewModel** — состояние и команды для View

### Применение в современных веб-приложениях

```go
// Model - остается тем же
type UserModel struct {
    repo UserRepository
}

// ViewModel - состояние и команды для View
type UserViewModel struct {
    model *UserModel
    
    // Состояние
    Users     []User `json:"users"`
    IsLoading bool   `json:"is_loading"`
    Error     string `json:"error"`
    
    // Команды (для веб-приложений - HTTP endpoints)
    commands map[string]func(interface{}) error
}

func NewUserViewModel(model *UserModel) *UserViewModel {
    vm := &UserViewModel{
        model:    model,
        commands: make(map[string]func(interface{}) error),
    }
    
    // Регистрация команд
    vm.commands["loadUsers"] = vm.LoadUsers
    vm.commands["createUser"] = vm.CreateUser
    vm.commands["deleteUser"] = vm.DeleteUser
    
    return vm
}

func (vm *UserViewModel) LoadUsers(data interface{}) error {
    vm.IsLoading = true
    vm.Error = ""
    
    users, err := vm.model.GetAllUsers()
    if err != nil {
        vm.IsLoading = false
        vm.Error = "Failed to load users"
        return err
    }
    
    vm.Users = users
    vm.IsLoading = false
    return nil
}

func (vm *UserViewModel) CreateUser(data interface{}) error {
    req := data.(CreateUserRequest)
    
    vm.IsLoading = true
    vm.Error = ""
    
    user, err := vm.model.CreateUser(req.Email, req.Name)
    if err != nil {
        vm.IsLoading = false
        vm.Error = "Failed to create user"
        return err
    }
    
    vm.Users = append(vm.Users, *user)
    vm.IsLoading = false
    return nil
}

func (vm *UserViewModel) DeleteUser(data interface{}) error {
    userID := data.(string)
    
    vm.IsLoading = true
    vm.Error = ""
    
    if err := vm.model.DeleteUser(userID); err != nil {
        vm.IsLoading = false
        vm.Error = "Failed to delete user"
        return err
    }
    
    // Удаляем из состояния
    for i, user := range vm.Users {
        if user.ID == userID {
            vm.Users = append(vm.Users[:i], vm.Users[i+1:]...)
            break
        }
    }
    
    vm.IsLoading = false
    return nil
}

// HTTP Controller для MVVM
type UserMVVMController struct {
    viewModel *UserViewModel
}

func (c *UserMVVMController) GetState(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(c.viewModel)
}

func (c *UserMVVMController) ExecuteCommand(w http.ResponseWriter, r *http.Request) {
    var cmd struct {
        Command string      `json:"command"`
        Data    interface{} `json:"data"`
    }
    
    if err := json.NewDecoder(r.Body).Decode(&cmd); err != nil {
        http.Error(w, "Invalid request", http.StatusBadRequest)
        return
    }
    
    if handler, exists := c.viewModel.commands[cmd.Command]; exists {
        if err := handler(cmd.Data); err != nil {
            http.Error(w, err.Error(), http.StatusInternalServerError)
            return
        }
    } else {
        http.Error(w, "Unknown command", http.StatusBadRequest)
        return
    }
    
    // Возвращаем обновленное состояние
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(c.viewModel)
}
```

### Преимущества MVVM
- **Отличная тестируемость** — ViewModel легко тестировать
- **Data Binding** — автоматическое обновление UI
- **Разделение логики** — View содержит только UI логику

### Недостатки MVVM
- **Сложность** — может быть избыточным для простых приложений
- **Memory leaks** — если не правильно управлять подписками

---

## 4. Layered Architecture

### Определение
**Слоистая архитектура** организует код в горизонтальные слои, каждый из которых может использовать только нижележащие слои.

```go
// Слой 1: Presentation Layer (HTTP handlers)
type UserController struct {
    service UserService
}

func (c *UserController) GetUser(w http.ResponseWriter, r *http.Request) {
    userID := mux.Vars(r)["id"]
    
    user, err := c.service.GetUser(userID)
    if err != nil {
        http.Error(w, err.Error(), http.StatusNotFound)
        return
    }
    
    json.NewEncoder(w).Encode(user)
}

// Слой 2: Business Logic Layer (Services)
type UserService struct {
    repo UserRepository
}

func (s *UserService) GetUser(id string) (*User, error) {
    // Бизнес-логика
    if id == "" {
        return nil, errors.New("user ID cannot be empty")
    }
    
    user, err := s.repo.FindByID(id)
    if err != nil {
        return nil, err
    }
    
    // Дополнительная бизнес-логика
    if !user.IsActive {
        return nil, errors.New("user is not active")
    }
    
    return user, nil
}

func (s *UserService) CreateUser(email, name string) (*User, error) {
    // Валидация
    if email == "" || name == "" {
        return nil, errors.New("email and name are required")
    }
    
    // Проверка существования
    existing, _ := s.repo.FindByEmail(email)
    if existing != nil {
        return nil, errors.New("user with this email already exists")
    }
    
    // Создание пользователя
    user := &User{
        ID:        generateID(),
        Email:     email,
        Name:      name,
        IsActive:  true,
        CreatedAt: time.Now(),
    }
    
    return s.repo.Save(user)
}

// Слой 3: Data Access Layer (Repository)
type UserRepository interface {
    FindByID(id string) (*User, error)
    FindByEmail(email string) (*User, error)
    Save(user *User) (*User, error)
    Delete(id string) error
}

type PostgresUserRepository struct {
    db *sql.DB
}

func (r *PostgresUserRepository) FindByID(id string) (*User, error) {
    query := `SELECT id, email, name, is_active, created_at FROM users WHERE id = $1`
    
    var user User
    err := r.db.QueryRow(query, id).Scan(
        &user.ID, &user.Email, &user.Name, &user.IsActive, &user.CreatedAt,
    )
    
    if err != nil {
        if err == sql.ErrNoRows {
            return nil, errors.New("user not found")
        }
        return nil, err
    }
    
    return &user, nil
}

// Слой 4: Infrastructure Layer (Database, External Services)
type Database struct {
    connection *sql.DB
}

func NewDatabase(connectionString string) (*Database, error) {
    db, err := sql.Open("postgres", connectionString)
    if err != nil {
        return nil, err
    }
    
    if err := db.Ping(); err != nil {
        return nil, err
    }
    
    return &Database{connection: db}, nil
}
```

### Преимущества слоистой архитектуры
- **Простота понимания** — четкая иерархия слоев
- **Разделение ответственности** — каждый слой решает свои задачи
- **Переиспользование** — нижние слои могут использоваться разными верхними

### Недостатки слоистой архитектуры
- **Производительность** — данные проходят через все слои
- **Жесткость** — сложно изменить структуру слоев

---

## 5. Hexagonal Architecture (Ports and Adapters)

### Определение
**Гексагональная архитектура** помещает бизнес-логику в центр, а внешние зависимости подключаются через порты и адаптеры.

```go
// Центр: Domain/Business Logic
type User struct {
    ID        string
    Email     string
    Name      string
    IsActive  bool
    CreatedAt time.Time
}

func (u *User) Deactivate() error {
    if !u.IsActive {
        return errors.New("user is already deactivated")
    }
    u.IsActive = false
    return nil
}

func (u *User) Activate() error {
    if u.IsActive {
        return errors.New("user is already active")
    }
    u.IsActive = true
    return nil
}

// Порты (интерфейсы)
type UserRepository interface {
    Save(user *User) error
    FindByID(id string) (*User, error)
    FindByEmail(email string) (*User, error)
}

type EmailService interface {
    SendWelcomeEmail(email string) error
    SendDeactivationEmail(email string) error
}

type UserService interface {
    CreateUser(email, name string) (*User, error)
    GetUser(id string) (*User, error)
    DeactivateUser(id string) error
}

// Центральный сервис (Application Layer)
type UserApplicationService struct {
    userRepo     UserRepository
    emailService EmailService
}

func NewUserApplicationService(repo UserRepository, emailService EmailService) UserService {
    return &UserApplicationService{
        userRepo:     repo,
        emailService: emailService,
    }
}

func (s *UserApplicationService) CreateUser(email, name string) (*User, error) {
    // Проверка существования
    existing, _ := s.userRepo.FindByEmail(email)
    if existing != nil {
        return nil, errors.New("user already exists")
    }
    
    // Создание пользователя
    user := &User{
        ID:        generateID(),
        Email:     email,
        Name:      name,
        IsActive:  true,
        CreatedAt: time.Now(),
    }
    
    // Сохранение
    if err := s.userRepo.Save(user); err != nil {
        return nil, err
    }
    
    // Отправка welcome email
    if err := s.emailService.SendWelcomeEmail(email); err != nil {
        // Логируем ошибку, но не возвращаем - пользователь создан
        log.Printf("Failed to send welcome email: %v", err)
    }
    
    return user, nil
}

func (s *UserApplicationService) DeactivateUser(id string) error {
    user, err := s.userRepo.FindByID(id)
    if err != nil {
        return err
    }
    
    if err := user.Deactivate(); err != nil {
        return err
    }
    
    if err := s.userRepo.Save(user); err != nil {
        return err
    }
    
    // Отправка уведомления о деактивации
    if err := s.emailService.SendDeactivationEmail(user.Email); err != nil {
        log.Printf("Failed to send deactivation email: %v", err)
    }
    
    return nil
}

// Адаптеры (реализации портов)

// HTTP Adapter
type UserHTTPAdapter struct {
    userService UserService
}

func NewUserHTTPAdapter(service UserService) *UserHTTPAdapter {
    return &UserHTTPAdapter{userService: service}
}

func (a *UserHTTPAdapter) CreateUser(w http.ResponseWriter, r *http.Request) {
    var req struct {
        Email string `json:"email"`
        Name  string `json:"name"`
    }
    
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, "Invalid request", http.StatusBadRequest)
        return
    }
    
    user, err := a.userService.CreateUser(req.Email, req.Name)
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    
    json.NewEncoder(w).Encode(user)
}

// Database Adapter
type PostgresUserAdapter struct {
    db *sql.DB
}

func NewPostgresUserAdapter(db *sql.DB) UserRepository {
    return &PostgresUserAdapter{db: db}
}

func (a *PostgresUserAdapter) Save(user *User) error {
    query := `INSERT INTO users (id, email, name, is_active, created_at) 
              VALUES ($1, $2, $3, $4, $5)
              ON CONFLICT (id) DO UPDATE SET
              email = EXCLUDED.email,
              name = EXCLUDED.name,
              is_active = EXCLUDED.is_active`
    
    _, err := a.db.Exec(query, user.ID, user.Email, user.Name, user.IsActive, user.CreatedAt)
    return err
}

// Email Adapter
type SMTPEmailAdapter struct {
    host string
    port int
}

func NewSMTPEmailAdapter(host string, port int) EmailService {
    return &SMTPEmailAdapter{host: host, port: port}
}

func (a *SMTPEmailAdapter) SendWelcomeEmail(email string) error {
    // Реализация отправки email через SMTP
    return nil
}

func (a *SMTPEmailAdapter) SendDeactivationEmail(email string) error {
    // Реализация отправки email через SMTP
    return nil
}

// Композиция приложения
func SetupApplication() *UserHTTPAdapter {
    // Инфраструктура
    db := setupDatabase()
    
    // Адаптеры
    userRepo := NewPostgresUserAdapter(db)
    emailService := NewSMTPEmailAdapter("smtp.example.com", 587)
    
    // Сервис
    userService := NewUserApplicationService(userRepo, emailService)
    
    // HTTP адаптер
    return NewUserHTTPAdapter(userService)
}
```

### Преимущества гексагональной архитектуры
- **Независимость от внешних зависимостей** — легко менять адаптеры
- **Отличная тестируемость** — можно легко создать mock адаптеры
- **Гибкость** — можно добавлять новые адаптеры без изменения ядра

### Недостатки гексагональной архитектуры
- **Сложность** — больше кода и абстракций
- **Overengineering** — может быть избыточным для простых приложений

---

## 6. Component-Based Architecture

### Определение
**Компонентная архитектура** организует код в независимые, переиспользуемые компоненты.

```go
// Компонент для управления пользователями
type UserComponent struct {
    repository UserRepository
    validator  UserValidator
    events     EventBus
}

func NewUserComponent(repo UserRepository, validator UserValidator, events EventBus) *UserComponent {
    return &UserComponent{
        repository: repo,
        validator:  validator,
        events:     events,
    }
}

func (c *UserComponent) CreateUser(req CreateUserRequest) (*User, error) {
    // Валидация
    if err := c.validator.Validate(req); err != nil {
        return nil, err
    }
    
    // Создание пользователя
    user := &User{
        ID:        generateID(),
        Email:     req.Email,
        Name:      req.Name,
        IsActive:  true,
        CreatedAt: time.Now(),
    }
    
    // Сохранение
    if err := c.repository.Save(user); err != nil {
        return nil, err
    }
    
    // Публикация события
    c.events.Publish(UserCreatedEvent{
        UserID: user.ID,
        Email:  user.Email,
        Name:   user.Name,
    })
    
    return user, nil
}

// Компонент для управления заказами
type OrderComponent struct {
    repository    OrderRepository
    userComponent *UserComponent
    events        EventBus
}

func NewOrderComponent(repo OrderRepository, userComp *UserComponent, events EventBus) *OrderComponent {
    return &OrderComponent{
        repository:    repo,
        userComponent: userComp,
        events:        events,
    }
}

func (c *OrderComponent) CreateOrder(req CreateOrderRequest) (*Order, error) {
    // Проверка пользователя через компонент
    user, err := c.userComponent.GetUser(req.UserID)
    if err != nil {
        return nil, err
    }
    
    if !user.IsActive {
        return nil, errors.New("user is not active")
    }
    
    // Создание заказа
    order := &Order{
        ID:        generateID(),
        UserID:    req.UserID,
        Items:     req.Items,
        Status:    OrderStatusPending,
        CreatedAt: time.Now(),
    }
    
    // Сохранение
    if err := c.repository.Save(order); err != nil {
        return nil, err
    }
    
    // Публикация события
    c.events.Publish(OrderCreatedEvent{
        OrderID: order.ID,
        UserID:  order.UserID,
        Items:   order.Items,
    })
    
    return order, nil
}

// Компонент для уведомлений
type NotificationComponent struct {
    emailService EmailService
    smsService   SMSService
    events       EventBus
}

func NewNotificationComponent(email EmailService, sms SMSService, events EventBus) *NotificationComponent {
    comp := &NotificationComponent{
        emailService: email,
        smsService:   sms,
        events:       events,
    }
    
    // Подписка на события
    events.Subscribe("user.created", comp.handleUserCreated)
    events.Subscribe("order.created", comp.handleOrderCreated)
    
    return comp
}

func (c *NotificationComponent) handleUserCreated(event interface{}) {
    userEvent := event.(UserCreatedEvent)
    
    // Отправка welcome email
    c.emailService.SendWelcomeEmail(userEvent.Email)
}

func (c *NotificationComponent) handleOrderCreated(event interface{}) {
    orderEvent := event.(OrderCreatedEvent)
    
    // Отправка уведомления о заказе
    c.emailService.SendOrderConfirmation(orderEvent.UserID, orderEvent.OrderID)
}

// Композиция компонентов
type Application struct {
    userComponent         *UserComponent
    orderComponent        *OrderComponent
    notificationComponent *NotificationComponent
}

func NewApplication() *Application {
    // Инфраструктура
    db := setupDatabase()
    events := setupEventBus()
    
    // Сервисы
    emailService := setupEmailService()
    smsService := setupSMSService()
    
    // Компоненты
    userComp := NewUserComponent(
        NewUserRepository(db),
        NewUserValidator(),
        events,
    )
    
    orderComp := NewOrderComponent(
        NewOrderRepository(db),
        userComp,
        events,
    )
    
    notificationComp := NewNotificationComponent(
        emailService,
        smsService,
        events,
    )
    
    return &Application{
        userComponent:         userComp,
        orderComponent:        orderComp,
        notificationComponent: notificationComp,
    }
}
```

### Преимущества компонентной архитектуры
- **Модульность** — компоненты можно разрабатывать независимо
- **Переиспользование** — компоненты можно использовать в разных контекстах
- **Масштабируемость** — легко добавлять новые компоненты

### Недостатки компонентной архитектуры
- **Сложность интеграции** — нужно управлять зависимостями между компонентами
- **Производительность** — может быть накладные расходы на коммуникацию

---

## Выбор архитектурного стиля

### Критерии выбора

```go
// Для простых CRUD приложений
type SimpleCRUDApp struct {
    // Layered Architecture достаточно
    controller *UserController
    service    *UserService
    repository *UserRepository
}

// Для сложных бизнес-процессов
type ComplexBusinessApp struct {
    // Hexagonal Architecture + DDD
    userService   *UserDomainService
    orderService  *OrderDomainService
    paymentService *PaymentDomainService
}

// Для интерактивных приложений
type InteractiveApp struct {
    // MVVM + Component-Based
    userViewModel    *UserViewModel
    orderViewModel   *OrderViewModel
    uiComponents     []UIComponent
}
```

### Рекомендации по выбору

| Сценарий | Рекомендуемый стиль | Причина |
|----------|---------------------|---------|
| Простые CRUD | Layered Architecture | Минимальная сложность |
| Сложная бизнес-логика | Hexagonal + DDD | Изоляция бизнес-логики |
| Интерактивные UI | MVVM | Data binding, состояние |
| Микросервисы | Component-Based | Модульность |
| Legacy системы | MVP | Постепенная миграция |

---

## 🔗 Связанные темы

### 🏗️ Основы
- **[Clean Architecture](clean-architecture.md)** - Современная интерпретация архитектурных принципов
- **[SOLID Principles](solid-principles.md)** - Принципы, применимые во всех стилях
- **[Design Patterns](gof-patterns.md)** - Паттерны, используемые в архитектурах

### 🏛️ Архитектура
- **[Microservices Architecture](../architecture/microservices-architecture.md)** - Архитектурные стили в микросервисах
- **[API Design](../architecture/api-design.md)** - Проектирование API для разных стилей

### ⚙️ Технические навыки
- **[Frontend Advanced](../frontend/frontend-advanced.md)** - MVVM и MVP в frontend
- **[Testing](../technical-skills/testing.md)** - Тестирование разных архитектурных стилей

---

**Путь обучения**: [Clean Architecture](clean-architecture.md) → Architectural Styles → [Microservices Architecture](../architecture/microservices-architecture.md)  
**Сложность**: ⭐⭐⭐ (3/5)  
**Время изучения**: 2-3 недели  
**Практика**: Реализация разных стилей в демо-проектах 