# Тестирование: Unit, Integration, E2E

**Тестирование** — это процесс проверки соответствия программного обеспечения требованиям и выявления дефектов.

## Пирамида тестирования

```
        /\
       /  \
      / E2E \        Мало тестов, медленные, дорогие
     /______\
    /        \
   /Integration\     Умеренное количество, средняя скорость
  /____________\
 /              \
/   Unit Tests   \   Много тестов, быстрые, дешевые
/________________\
```

## Unit Tests (Модульные тесты)

**Unit тесты** проверяют отдельные компоненты (функции, классы) в изоляции.

### Характеристики:
- **Быстрые** (~миллисекунды)
- **Изолированные** (без внешних зависимостей)
- **Детерминированные** (стабильные результаты)
- **Фокусированные** (один аспект кода)

### Go Unit Tests

```go
// services/user-service/domain/user_test.go
package domain

import (
    "testing"
    "time"
    
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

func TestUser_IsValid(t *testing.T) {
    tests := []struct {
        name    string
        user    User
        wantErr bool
    }{
        {
            name: "valid user",
            user: User{
                ID:           "user-123",
                Email:        "test@example.com",
                Name:         "John Doe",
                Status:       "active",
                PasswordHash: "hashed_password",
            },
            wantErr: false,
        },
        {
            name: "invalid email",
            user: User{
                ID:           "user-123",
                Email:        "invalid-email",
                Name:         "John Doe",
                Status:       "active",
                PasswordHash: "hashed_password",
            },
            wantErr: true,
        },
        {
            name: "empty name",
            user: User{
                ID:           "user-123",
                Email:        "test@example.com",
                Name:         "",
                Status:       "active",
                PasswordHash: "hashed_password",
            },
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := tt.user.IsValid()
            
            if tt.wantErr {
                assert.Error(t, err)
            } else {
                assert.NoError(t, err)
            }
        })
    }
}

func TestUser_Block(t *testing.T) {
    // Given
    user := User{
        ID:     "user-123",
        Email:  "test@example.com",
        Name:   "John Doe",
        Status: "active",
    }
    
    // When
    err := user.Block()
    
    // Then
    require.NoError(t, err)
    assert.Equal(t, "blocked", user.Status)
    assert.WithinDuration(t, time.Now(), user.UpdatedAt, time.Second)
}
```

### Тесты с моками

```go
// services/user-service/application/user_service_test.go
package application

import (
    "context"
    "testing"
    "errors"
    
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
    "github.com/stretchr/testify/require"
)

// Mock для репозитория
type MockUserRepository struct {
    mock.Mock
}

func (m *MockUserRepository) Create(ctx context.Context, user *User) error {
    args := m.Called(ctx, user)
    return args.Error(0)
}

func (m *MockUserRepository) GetByID(ctx context.Context, id string) (*User, error) {
    args := m.Called(ctx, id)
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).(*User), args.Error(1)
}

func TestUserService_CreateUser(t *testing.T) {
    // Given
    mockRepo := new(MockUserRepository)
    service := NewUserService(mockRepo)
    
    cmd := CreateUserCommand{
        Email:    "test@example.com",
        Name:     "John Doe",
        Password: "password123",
    }
    
    // Setup mock expectations
    mockRepo.On("Create", mock.Anything, mock.AnythingOfType("*User")).Return(nil)
    
    // When
    result, err := service.CreateUser(context.Background(), cmd)
    
    // Then
    require.NoError(t, err)
    assert.NotEmpty(t, result.ID)
    assert.Equal(t, cmd.Email, result.Email)
    assert.Equal(t, cmd.Name, result.Name)
    
    // Verify mock was called
    mockRepo.AssertExpectations(t)
}

func TestUserService_CreateUser_RepositoryError(t *testing.T) {
    // Given
    mockRepo := new(MockUserRepository)
    service := NewUserService(mockRepo)
    
    cmd := CreateUserCommand{
        Email:    "test@example.com",
        Name:     "John Doe",
        Password: "password123",
    }
    
    expectedErr := errors.New("database error")
    mockRepo.On("Create", mock.Anything, mock.AnythingOfType("*User")).Return(expectedErr)
    
    // When
    result, err := service.CreateUser(context.Background(), cmd)
    
    // Then
    assert.Error(t, err)
    assert.Nil(t, result)
    assert.Contains(t, err.Error(), "database error")
    
    mockRepo.AssertExpectations(t)
}
```

### Python Unit Tests

```python
# services/order-service/tests/test_order_domain.py
import pytest
from datetime import datetime
from decimal import Decimal

from domain.order import Order, OrderItem, OrderStatus
from domain.events import OrderCreatedEvent, OrderConfirmedEvent

class TestOrder:
    def test_create_order_success(self):
        """Тест успешного создания заказа"""
        # Given
        user_id = "user-123"
        items = [
            OrderItem(product_id="prod-1", quantity=2, price=Decimal("10.50")),
            OrderItem(product_id="prod-2", quantity=1, price=Decimal("5.00"))
        ]
        
        # When
        order = Order.create(user_id, items)
        
        # Then
        assert order.user_id == user_id
        assert len(order.items) == 2
        assert order.total_amount == Decimal("26.00")
        assert order.status == OrderStatus.PENDING
        assert len(order.events) == 1
        assert isinstance(order.events[0], OrderCreatedEvent)
    
    def test_create_order_empty_items(self):
        """Тест создания заказа с пустым списком товаров"""
        # Given
        user_id = "user-123"
        items = []
        
        # When & Then
        with pytest.raises(ValueError, match="Order must have at least one item"):
            Order.create(user_id, items)
    
    def test_confirm_order_success(self):
        """Тест успешного подтверждения заказа"""
        # Given
        order = Order.create("user-123", [
            OrderItem(product_id="prod-1", quantity=1, price=Decimal("10.00"))
        ])
        
        # When
        order.confirm()
        
        # Then
        assert order.status == OrderStatus.CONFIRMED
        assert len(order.events) == 2
        assert isinstance(order.events[1], OrderConfirmedEvent)
    
    def test_confirm_order_already_confirmed(self):
        """Тест подтверждения уже подтвержденного заказа"""
        # Given
        order = Order.create("user-123", [
            OrderItem(product_id="prod-1", quantity=1, price=Decimal("10.00"))
        ])
        order.confirm()
        
        # When & Then
        with pytest.raises(ValueError, match="Order is already confirmed"):
            order.confirm()
    
    def test_calculate_total_amount(self):
        """Тест расчета общей суммы заказа"""
        # Given
        items = [
            OrderItem(product_id="prod-1", quantity=2, price=Decimal("15.75")),
            OrderItem(product_id="prod-2", quantity=3, price=Decimal("8.33")),
            OrderItem(product_id="prod-3", quantity=1, price=Decimal("100.00"))
        ]
        
        # When
        order = Order.create("user-123", items)
        
        # Then
        expected_total = Decimal("15.75") * 2 + Decimal("8.33") * 3 + Decimal("100.00")
        assert order.total_amount == expected_total

@pytest.fixture
def sample_order():
    """Фикстура для создания тестового заказа"""
    items = [
        OrderItem(product_id="prod-1", quantity=2, price=Decimal("10.00")),
        OrderItem(product_id="prod-2", quantity=1, price=Decimal("15.00"))
    ]
    return Order.create("user-123", items)

class TestOrderWithFixtures:
    def test_order_has_correct_items(self, sample_order):
        """Тест с использованием фикстуры"""
        assert len(sample_order.items) == 2
        assert sample_order.total_amount == Decimal("35.00")
```

### Тесты с моками в Python

```python
# services/order-service/tests/test_order_service.py
import pytest
from unittest.mock import Mock, patch, AsyncMock
from decimal import Decimal

from application.order_service import OrderService
from domain.order import Order, OrderItem
from infrastructure.event_store import EventStore

class TestOrderService:
    @pytest.fixture
    def mock_event_store(self):
        """Мок для event store"""
        return Mock(spec=EventStore)
    
    @pytest.fixture
    def mock_user_service(self):
        """Мок для user service"""
        mock = Mock()
        mock.get_user = AsyncMock(return_value={
            "id": "user-123",
            "name": "John Doe",
            "email": "john@example.com"
        })
        return mock
    
    @pytest.fixture
    def order_service(self, mock_event_store, mock_user_service):
        """Фикстура для OrderService"""
        return OrderService(mock_event_store, mock_user_service)
    
    @pytest.mark.asyncio
    async def test_create_order_success(self, order_service, mock_event_store, mock_user_service):
        """Тест успешного создания заказа"""
        # Given
        order_data = {
            "user_id": "user-123",
            "items": [
                {"product_id": "prod-1", "quantity": 2, "price": "10.00"},
                {"product_id": "prod-2", "quantity": 1, "price": "15.00"}
            ]
        }
        
        mock_event_store.save_events.return_value = None
        
        # When
        result = await order_service.create_order(order_data)
        
        # Then
        assert result["user_id"] == "user-123"
        assert result["total_amount"] == "35.00"
        assert result["status"] == "pending"
        
        # Verify user service was called
        mock_user_service.get_user.assert_called_once_with("user-123")
        
        # Verify events were saved
        mock_event_store.save_events.assert_called_once()
    
    @pytest.mark.asyncio
    async def test_create_order_user_not_found(self, order_service, mock_user_service):
        """Тест создания заказа для несуществующего пользователя"""
        # Given
        mock_user_service.get_user.side_effect = Exception("User not found")
        
        order_data = {
            "user_id": "nonexistent-user",
            "items": [{"product_id": "prod-1", "quantity": 1, "price": "10.00"}]
        }
        
        # When & Then
        with pytest.raises(Exception, match="User not found"):
            await order_service.create_order(order_data)
    
    @patch('application.order_service.datetime')
    def test_create_order_with_mocked_time(self, mock_datetime, order_service):
        """Тест с мокированием времени"""
        # Given
        fixed_time = datetime(2024, 1, 15, 10, 30, 0)
        mock_datetime.utcnow.return_value = fixed_time
        
        # When
        order_data = {
            "user_id": "user-123",
            "items": [{"product_id": "prod-1", "quantity": 1, "price": "10.00"}]
        }
        
        # Then time should be mocked
        # (логика теста зависит от конкретной реализации)
```

## Integration Tests (Интеграционные тесты)

**Integration тесты** проверяют взаимодействие между компонентами.

### Go Integration Tests

```go
// services/user-service/tests/integration/user_repository_test.go
package integration

import (
    "context"
    "testing"
    "database/sql"
    
    "github.com/testcontainers/testcontainers-go"
    "github.com/testcontainers/testcontainers-go/modules/postgres"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

func TestUserRepository_Integration(t *testing.T) {
    // Запуск PostgreSQL контейнера
    ctx := context.Background()
    
    postgresContainer, err := postgres.RunContainer(ctx,
        testcontainers.WithImage("postgres:15-alpine"),
        postgres.WithDatabase("testdb"),
        postgres.WithUsername("test"),
        postgres.WithPassword("test"),
    )
    require.NoError(t, err)
    defer postgresContainer.Terminate(ctx)
    
    // Получение connection string
    connStr, err := postgresContainer.ConnectionString(ctx)
    require.NoError(t, err)
    
    // Создание репозитория
    repo, err := NewUserRepository(connStr)
    require.NoError(t, err)
    defer repo.Close()
    
    // Выполнение миграций
    err = repo.RunMigrations()
    require.NoError(t, err)
    
    t.Run("Create and Get User", func(t *testing.T) {
        // Given
        user := &User{
            ID:           "user-123",
            Email:        "test@example.com",
            Name:         "John Doe",
            Status:       "active",
            PasswordHash: "hashed_password",
        }
        
        // When - Create
        err := repo.Create(ctx, user)
        require.NoError(t, err)
        
        // Then - Get
        retrievedUser, err := repo.GetByID(ctx, user.ID)
        require.NoError(t, err)
        
        assert.Equal(t, user.ID, retrievedUser.ID)
        assert.Equal(t, user.Email, retrievedUser.Email)
        assert.Equal(t, user.Name, retrievedUser.Name)
        assert.Equal(t, user.Status, retrievedUser.Status)
    })
    
    t.Run("Update User Status", func(t *testing.T) {
        // Given
        user := &User{
            ID:           "user-456",
            Email:        "test2@example.com",
            Name:         "Jane Doe",
            Status:       "active",
            PasswordHash: "hashed_password",
        }
        
        err := repo.Create(ctx, user)
        require.NoError(t, err)
        
        // When
        err = repo.UpdateStatus(ctx, user.ID, "blocked")
        require.NoError(t, err)
        
        // Then
        updatedUser, err := repo.GetByID(ctx, user.ID)
        require.NoError(t, err)
        
        assert.Equal(t, "blocked", updatedUser.Status)
    })
}
```

### Python Integration Tests

```python
# services/order-service/tests/integration/test_event_store.py
import pytest
import asyncio
from testcontainers.postgres import PostgresContainer
from decimal import Decimal

from infrastructure.event_store import EventStore
from domain.order import Order, OrderItem
from domain.events import OrderCreatedEvent, OrderConfirmedEvent

class TestEventStoreIntegration:
    @pytest.fixture(scope="session")
    def postgres_container(self):
        """Фикстура для PostgreSQL контейнера"""
        with PostgresContainer("postgres:15-alpine") as postgres:
            yield postgres
    
    @pytest.fixture
    async def event_store(self, postgres_container):
        """Фикстура для Event Store"""
        connection_string = postgres_container.get_connection_url()
        store = EventStore(connection_string)
        
        # Создание таблиц
        await store.create_tables()
        
        yield store
        
        # Очистка
        await store.close()
    
    @pytest.mark.asyncio
    async def test_save_and_load_events(self, event_store):
        """Тест сохранения и загрузки событий"""
        # Given
        order = Order.create("user-123", [
            OrderItem(product_id="prod-1", quantity=2, price=Decimal("10.00"))
        ])
        order.confirm()
        
        aggregate_id = order.id
        events = order.events
        
        # When - Save events
        await event_store.save_events(aggregate_id, events, expected_version=0)
        
        # Then - Load events
        loaded_events = await event_store.load_events(aggregate_id)
        
        assert len(loaded_events) == 2
        assert isinstance(loaded_events[0], OrderCreatedEvent)
        assert isinstance(loaded_events[1], OrderConfirmedEvent)
    
    @pytest.mark.asyncio
    async def test_optimistic_locking(self, event_store):
        """Тест оптимистической блокировки"""
        # Given
        order = Order.create("user-123", [
            OrderItem(product_id="prod-1", quantity=1, price=Decimal("10.00"))
        ])
        
        aggregate_id = order.id
        events = order.events
        
        # When - Save events first time
        await event_store.save_events(aggregate_id, events, expected_version=0)
        
        # Then - Second save with same version should fail
        with pytest.raises(Exception, match="Concurrency conflict"):
            await event_store.save_events(aggregate_id, events, expected_version=0)
    
    @pytest.mark.asyncio
    async def test_get_events_by_type(self, event_store):
        """Тест получения событий по типу"""
        # Given
        order1 = Order.create("user-123", [
            OrderItem(product_id="prod-1", quantity=1, price=Decimal("10.00"))
        ])
        order1.confirm()
        
        order2 = Order.create("user-456", [
            OrderItem(product_id="prod-2", quantity=2, price=Decimal("15.00"))
        ])
        
        # When
        await event_store.save_events(order1.id, order1.events, expected_version=0)
        await event_store.save_events(order2.id, order2.events, expected_version=0)
        
        # Then
        created_events = await event_store.get_events_by_type("OrderCreatedEvent")
        confirmed_events = await event_store.get_events_by_type("OrderConfirmedEvent")
        
        assert len(created_events) == 2
        assert len(confirmed_events) == 1
```

## End-to-End Tests (E2E тесты)

**E2E тесты** проверяют работу всей системы от начала до конца.

### API E2E Tests

```python
# tests/e2e/test_user_flow.py
import pytest
import httpx
import asyncio
from testcontainers.compose import DockerCompose
import time

class TestUserFlowE2E:
    @pytest.fixture(scope="session")
    def docker_compose(self):
        """Фикстура для запуска всех сервисов"""
        with DockerCompose(".", compose_file_name="docker-compose.test.yml") as compose:
            # Ждем готовности сервисов
            time.sleep(30)
            yield compose
    
    @pytest.fixture
    def api_client(self, docker_compose):
        """HTTP клиент для API Gateway"""
        return httpx.AsyncClient(
            base_url="http://localhost:3000",
            timeout=30.0
        )
    
    @pytest.mark.asyncio
    async def test_complete_user_order_flow(self, api_client):
        """Тест полного флоу: создание пользователя -> создание заказа -> подтверждение"""
        
        # Step 1: Создание пользователя
        user_data = {
            "email": "test@example.com",
            "name": "Test User",
            "password": "password123"
        }
        
        response = await api_client.post("/api/users", json=user_data)
        assert response.status_code == 201
        
        user = response.json()
        user_id = user["id"]
        
        # Step 2: Получение пользователя
        response = await api_client.get(f"/api/users/{user_id}")
        assert response.status_code == 200
        
        retrieved_user = response.json()
        assert retrieved_user["email"] == user_data["email"]
        assert retrieved_user["name"] == user_data["name"]
        
        # Step 3: Создание заказа
        order_data = {
            "user_id": user_id,
            "items": [
                {"product_id": "prod-1", "quantity": 2, "price": "10.50"},
                {"product_id": "prod-2", "quantity": 1, "price": "5.00"}
            ]
        }
        
        response = await api_client.post("/api/orders", json=order_data)
        assert response.status_code == 201
        
        order = response.json()
        order_id = order["id"]
        assert order["user_id"] == user_id
        assert order["total_amount"] == "26.00"
        assert order["status"] == "pending"
        
        # Step 4: Подтверждение заказа
        response = await api_client.post(f"/api/orders/{order_id}/confirm")
        assert response.status_code == 200
        
        confirmed_order = response.json()
        assert confirmed_order["status"] == "confirmed"
        
        # Step 5: Проверка через GraphQL
        graphql_query = """
        query GetUserWithOrders($userId: ID!) {
            user(id: $userId) {
                id
                name
                email
                orders {
                    id
                    status
                    totalAmount
                    items {
                        productId
                        quantity
                        price
                    }
                }
            }
        }
        """
        
        response = await api_client.post(
            "/graphql",
            json={
                "query": graphql_query,
                "variables": {"userId": user_id}
            }
        )
        
        assert response.status_code == 200
        graphql_data = response.json()
        
        assert "errors" not in graphql_data
        user_data = graphql_data["data"]["user"]
        assert user_data["id"] == user_id
        assert len(user_data["orders"]) == 1
        assert user_data["orders"][0]["status"] == "confirmed"
    
    @pytest.mark.asyncio
    async def test_error_handling_flow(self, api_client):
        """Тест обработки ошибок"""
        
        # Попытка создать заказ для несуществующего пользователя
        order_data = {
            "user_id": "nonexistent-user",
            "items": [
                {"product_id": "prod-1", "quantity": 1, "price": "10.00"}
            ]
        }
        
        response = await api_client.post("/api/orders", json=order_data)
        assert response.status_code == 400
        
        error_data = response.json()
        assert "error" in error_data
        assert "User not found" in error_data["error"]
    
    @pytest.mark.asyncio
    async def test_concurrent_orders(self, api_client):
        """Тест конкурентных заказов"""
        
        # Создаем пользователя
        user_data = {
            "email": "concurrent@example.com",
            "name": "Concurrent User",
            "password": "password123"
        }
        
        response = await api_client.post("/api/users", json=user_data)
        user_id = response.json()["id"]
        
        # Создаем множество заказов одновременно
        order_tasks = []
        for i in range(5):
            order_data = {
                "user_id": user_id,
                "items": [
                    {"product_id": f"prod-{i}", "quantity": 1, "price": "10.00"}
                ]
            }
            task = api_client.post("/api/orders", json=order_data)
            order_tasks.append(task)
        
        # Ждем выполнения всех заказов
        responses = await asyncio.gather(*order_tasks)
        
        # Проверяем, что все заказы созданы успешно
        for response in responses:
            assert response.status_code == 201
        
        # Проверяем, что все заказы есть в системе
        response = await api_client.get(f"/api/users/{user_id}/orders")
        assert response.status_code == 200
        
        orders = response.json()
        assert len(orders) == 5
```

### Frontend E2E Tests (Playwright)

```typescript
// tests/e2e/user-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('User Flow E2E', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('http://localhost:3001');
  });

  test('should create user and order successfully', async ({ page }) => {
    // Step 1: Создание пользователя
    await page.click('[data-testid="create-user-button"]');
    
    await page.fill('[data-testid="email-input"]', 'test@example.com');
    await page.fill('[data-testid="name-input"]', 'Test User');
    await page.fill('[data-testid="password-input"]', 'password123');
    
    await page.click('[data-testid="submit-button"]');
    
    // Ждем редиректа или подтверждения
    await expect(page.locator('[data-testid="user-created-message"]')).toBeVisible();
    
    // Step 2: Создание заказа
    await page.click('[data-testid="create-order-button"]');
    
    // Добавляем товары
    await page.click('[data-testid="add-product-button"]');
    await page.selectOption('[data-testid="product-select"]', 'prod-1');
    await page.fill('[data-testid="quantity-input"]', '2');
    await page.fill('[data-testid="price-input"]', '10.50');
    
    await page.click('[data-testid="add-another-product"]');
    await page.selectOption('[data-testid="product-select-1"]', 'prod-2');
    await page.fill('[data-testid="quantity-input-1"]', '1');
    await page.fill('[data-testid="price-input-1"]', '5.00');
    
    await page.click('[data-testid="create-order-submit"]');
    
    // Step 3: Проверка создания заказа
    await expect(page.locator('[data-testid="order-created-message"]')).toBeVisible();
    
    // Проверяем отображение общей суммы
    await expect(page.locator('[data-testid="total-amount"]')).toHaveText('$26.00');
    
    // Step 4: Подтверждение заказа
    await page.click('[data-testid="confirm-order-button"]');
    
    // Проверяем изменение статуса
    await expect(page.locator('[data-testid="order-status"]')).toHaveText('confirmed');
    
    // Step 5: Проверка в списке заказов
    await page.click('[data-testid="view-orders-button"]');
    
    const orderRows = page.locator('[data-testid="order-row"]');
    await expect(orderRows).toHaveCount(1);
    
    const firstOrder = orderRows.first();
    await expect(firstOrder.locator('[data-testid="order-status"]')).toHaveText('confirmed');
    await expect(firstOrder.locator('[data-testid="order-total"]')).toHaveText('$26.00');
  });

  test('should handle validation errors', async ({ page }) => {
    // Попытка создать пользователя с невалидными данными
    await page.click('[data-testid="create-user-button"]');
    
    await page.fill('[data-testid="email-input"]', 'invalid-email');
    await page.fill('[data-testid="name-input"]', '');
    await page.fill('[data-testid="password-input"]', '123');
    
    await page.click('[data-testid="submit-button"]');
    
    // Проверяем отображение ошибок валидации
    await expect(page.locator('[data-testid="email-error"]')).toBeVisible();
    await expect(page.locator('[data-testid="name-error"]')).toBeVisible();
    await expect(page.locator('[data-testid="password-error"]')).toBeVisible();
  });
});
```

## Лучшие практики

### Unit Tests
1. **Тестируйте поведение, а не реализацию**
2. **Используйте описательные имена тестов**
3. **Следуйте паттерну AAA** (Arrange-Act-Assert)
4. **Каждый тест должен быть независимым**
5. **Используйте моки для внешних зависимостей**

### Integration Tests
1. **Тестируйте реальные взаимодействия**
2. **Используйте тестовые базы данных**
3. **Очищайте данные между тестами**
4. **Группируйте связанные тесты**

### E2E Tests
1. **Тестируйте критичные пользовательские сценарии**
2. **Делайте тесты стабильными**
3. **Используйте явные ожидания**
4. **Параллелизируйте выполнение**

### Общие принципы
1. **Быстрая обратная связь**
2. **Детерминированность**
3. **Изоляция**
4. **Читаемость**
5. **Поддерживаемость**

## Заключение

Правильная стратегия тестирования включает:
- **Много unit тестов** для быстрой обратной связи
- **Умеренное количество integration тестов** для проверки взаимодействий
- **Несколько E2E тестов** для критичных сценариев

Это обеспечивает баланс между скоростью выполнения и покрытием тестами.

## См. также
- [CI/CD и DevOps](./cicd-devops.md)
- [Микросервисная архитектура](./microservices-architecture.md)
- [Безопасность](./security.md) 