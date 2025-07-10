# 🎯 Доменно-ориентированное проектирование

## 🎯 Обзор раздела

Этот раздел посвящен продвинутым архитектурным паттернам и подходам к проектированию сложных систем. Здесь изучаются концепции Domain-Driven Design (DDD), CQRS и Event Sourcing.

## 📚 Содержание

### 🏛️ Доменное проектирование
- **[DDD Patterns](ddd-patterns.md)** - Паттерны доменно-ориентированного проектирования
- **[CQRS Pattern](cqrs-pattern.md)** - Разделение команд и запросов
- **[Event Sourcing](event-sourcing.md)** - Архитектура на основе событий

## 🎓 Целевая аудитория

- **Senior разработчики** - изучение сложных архитектурных паттернов
- **Архитекторы** - проектирование enterprise-систем
- **Тимлиды** - принятие архитектурных решений

## 🛠️ Практические навыки

После изучения этого раздела вы сможете:
- Применять принципы Domain-Driven Design
- Проектировать системы с разделением команд и запросов (CQRS)
- Реализовывать Event Sourcing
- Выделять bounded contexts
- Проектировать aggregate roots
- Реализовывать event-driven архитектуру

## 🔗 Связи с другими разделами

### ➡️ Следующие шаги
- **[Technical Foundations](../technical-foundations/README.md)** - Техническая реализация DDD паттернов
- **[Architecture](../../architecture/README.md)** - Применение на уровне системы

### 🔙 Предварительные знания
- **[Principles](../principles/README.md)** - SOLID и другие принципы
- **[Architecture](../architecture/README.md)** - Clean Architecture и паттерны

## 📊 Уровень сложности

| Материал | Сложность | Время изучения |
|----------|-----------|----------------|
| DDD Patterns | ⭐⭐⭐⭐ (4/5) | 4-5 недель |
| CQRS Pattern | ⭐⭐⭐⭐ (4/5) | 2-3 недели |
| Event Sourcing | ⭐⭐⭐⭐⭐ (5/5) | 3-4 недели |

## 🎯 Рекомендации по изучению

### 1. Последовательность изучения
```
DDD Patterns → CQRS Pattern → Event Sourcing
```

### 2. Практические упражнения
- Моделирование домена для сложной системы
- Реализация CQRS в существующем проекте
- Создание Event Store для одного из агрегатов

### 3. Метрики успеха
- Четкое разделение доменов
- Упрощение сложных бизнес-операций
- Улучшение производительности чтения

## 💡 Ключевые концепции

### Domain-Driven Design
```go
// Aggregate Root
type Order struct {
    id       OrderID
    items    []OrderItem
    status   OrderStatus
    total    Money
    events   []DomainEvent
}

func (o *Order) AddItem(item OrderItem) error {
    if o.status != OrderStatusDraft {
        return errors.New("cannot add item to non-draft order")
    }
    o.items = append(o.items, item)
    o.events = append(o.events, ItemAddedEvent{
        OrderID: o.id,
        Item:    item,
    })
    return nil
}

// Value Object
type Money struct {
    amount   int64
    currency string
}

// Domain Service
type OrderService struct {
    repo OrderRepository
}

func (s *OrderService) CalculateDiscount(order Order) Money {
    // Бизнес-логика расчета скидки
}
```

### CQRS (Command Query Responsibility Segregation)
```go
// Command Side
type CreateOrderCommand struct {
    CustomerID string
    Items      []OrderItem
}

type CreateOrderHandler struct {
    repo OrderRepository
}

func (h *CreateOrderHandler) Handle(cmd CreateOrderCommand) error {
    order := NewOrder(cmd.CustomerID, cmd.Items)
    return h.repo.Save(order)
}

// Query Side
type OrderQuery struct {
    ID string
}

type OrderQueryHandler struct {
    readRepo OrderReadRepository
}

func (h *OrderQueryHandler) Handle(query OrderQuery) (OrderView, error) {
    return h.readRepo.FindOrderView(query.ID)
}

// Разные модели для чтения и записи
type Order struct {        // Write model
    id     OrderID
    items  []OrderItem
    status OrderStatus
}

type OrderView struct {    // Read model
    ID           string
    CustomerName string
    ItemCount    int
    Total        string
    StatusText   string
}
```

### Event Sourcing
```go
// Event Store
type EventStore interface {
    SaveEvents(streamID string, events []DomainEvent) error
    LoadEvents(streamID string) ([]DomainEvent, error)
}

// Aggregate восстанавливается из событий
func (o *Order) LoadFromHistory(events []DomainEvent) {
    for _, event := range events {
        o.Apply(event)
    }
}

func (o *Order) Apply(event DomainEvent) {
    switch e := event.(type) {
    case OrderCreatedEvent:
        o.id = e.OrderID
        o.status = OrderStatusDraft
    case ItemAddedEvent:
        o.items = append(o.items, e.Item)
    case OrderCompletedEvent:
        o.status = OrderStatusCompleted
    }
}

// Projection для создания Read Model
type OrderProjection struct {
    store EventStore
}

func (p *OrderProjection) BuildView(orderID string) (OrderView, error) {
    events, err := p.store.LoadEvents(orderID)
    if err != nil {
        return OrderView{}, err
    }
    
    view := OrderView{ID: orderID}
    for _, event := range events {
        p.updateView(&view, event)
    }
    return view, nil
}
```

## 🔄 Практическое применение

### Проектирование DDD системы
1. Определение bounded contexts
2. Выделение aggregates
3. Проектирование domain services
4. Реализация repositories

### Внедрение CQRS
1. Разделение команд и запросов
2. Создание read models
3. Настройка синхронизации
4. Оптимизация производительности

### Реализация Event Sourcing
1. Проектирование событий
2. Создание event store
3. Реализация projections
4. Настройка snapshots 