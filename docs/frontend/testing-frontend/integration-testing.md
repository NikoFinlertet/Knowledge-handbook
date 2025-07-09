# 🔗 Integration Testing

## 📋 Навигация
- [Mock Service Worker](#mock-service-worker)
- [API Integration Tests](#api-integration-tests)
- [Database Integration](#database-integration)
- [Best Practices](#best-practices)

---

## 🌐 Mock Service Worker

### Настройка MSW

```javascript
// mocks/handlers.js
import { rest } from 'msw';

export const handlers = [
  rest.get('/api/users', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json([
        { id: 1, name: 'John Doe', email: 'john@example.com' },
        { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
      ])
    );
  }),

  rest.post('/api/users', async (req, res, ctx) => {
    const userData = await req.json();
    
    if (!userData.email) {
      return res(
        ctx.status(400),
        ctx.json({ error: 'Email is required' })
      );
    }
    
    return res(
      ctx.status(201),
      ctx.json({ id: 3, ...userData })
    );
  }),

  rest.get('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params;
    
    if (id === '999') {
      return res(ctx.status(404), ctx.json({ error: 'User not found' }));
    }
    
    return res(
      ctx.status(200),
      ctx.json({ id: parseInt(id), name: 'User Name' })
    );
  })
];
```

---

## 🧪 API Integration Tests

### Базовый API Client

```javascript
// apiClient.js
export class ApiClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
  }

  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      headers: { 'Content-Type': 'application/json', ...options.headers },
      ...options
    };

    if (config.body && typeof config.body === 'object') {
      config.body = JSON.stringify(config.body);
    }

    const response = await fetch(url, config);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    return response.json();
  }

  get(endpoint, headers = {}) {
    return this.request(endpoint, { method: 'GET', headers });
  }

  post(endpoint, data, headers = {}) {
    return this.request(endpoint, { method: 'POST', body: data, headers });
  }
}
```

### Интеграционные тесты

```javascript
// apiClient.integration.test.js
import { ApiClient } from './apiClient';
import { setupServer } from 'msw/node';
import { handlers } from '../mocks/handlers';

const server = setupServer(...handlers);

describe('ApiClient Integration Tests', () => {
  let apiClient;

  beforeAll(() => server.listen());
  afterEach(() => server.resetHandlers());
  afterAll(() => server.close());

  beforeEach(() => {
    apiClient = new ApiClient('/api');
  });

  test('should fetch users list', async () => {
    const users = await apiClient.get('/users');
    
    expect(users).toHaveLength(2);
    expect(users[0]).toHaveProperty('id', 1);
    expect(users[0]).toHaveProperty('name', 'John Doe');
  });

  test('should create new user', async () => {
    const userData = {
      name: 'New User',
      email: 'newuser@example.com'
    };

    const createdUser = await apiClient.post('/users', userData);
    
    expect(createdUser).toHaveProperty('id', 3);
    expect(createdUser.name).toBe(userData.name);
  });

  test('should handle validation error', async () => {
    const invalidData = { name: 'User without email' };

    await expect(apiClient.post('/users', invalidData))
      .rejects.toThrow('HTTP 400: Bad Request');
  });

  test('should handle 404 error', async () => {
    await expect(apiClient.get('/users/999'))
      .rejects.toThrow('HTTP 404: Not Found');
  });
});
```

---

## 🗄️ Database Integration

```javascript
// userRepository.test.js
import { UserRepository } from './UserRepository';
import { DatabaseConnection } from './DatabaseConnection';

describe('User Repository Integration', () => {
  let userRepository;
  let db;

  beforeAll(async () => {
    db = new DatabaseConnection({ database: 'test_db' });
    await db.connect();
    userRepository = new UserRepository(db);
  });

  afterAll(async () => {
    await db.disconnect();
  });

  beforeEach(async () => {
    await db.query('TRUNCATE TABLE users RESTART IDENTITY CASCADE');
  });

  test('should create user in database', async () => {
    const userData = {
      name: 'John Doe',
      email: 'john@example.com'
    };

    const createdUser = await userRepository.create(userData);
    
    expect(createdUser).toHaveProperty('id');
    expect(createdUser.name).toBe(userData.name);
    
    const userFromDb = await userRepository.findById(createdUser.id);
    expect(userFromDb).toEqual(createdUser);
  });

  test('should enforce unique email constraint', async () => {
    const userData = { name: 'John', email: 'john@example.com' };

    await userRepository.create(userData);
    
    await expect(userRepository.create(userData))
      .rejects.toThrow('Email already exists');
  });
});
```

---

## 📊 Best Practices

### 🎯 Принципы интеграционного тестирования

1. **Изолированность** - каждый тест независим
2. **Детерминизм** - одинаковые результаты при повторных запусках  
3. **Реалистичность** - имитация реальных сценариев
4. **Производительность** - быстрое выполнение

### 🛠️ Полезные советы

```javascript
// ✅ Хорошо - полная очистка состояния
beforeEach(async () => {
  await db.truncateAllTables();
  server.resetHandlers();
});

// ✅ Хорошо - проверка и данных и побочных эффектов
test('should create user and trigger side effects', async () => {
  const user = await userService.create(userData);
  
  expect(user.id).toBeDefined();
  expect(emailService.send).toHaveBeenCalled();
});
```

---

## 🔗 Связанные разделы

- [[unit-testing|Unit Testing]] - Модульное тестирование
- [[react-testing|React Testing]] - Тестирование React компонентов
- [[testing-patterns|Testing Patterns]] - Паттерны тестирования
- [[e2e-testing|E2E Testing]] - End-to-End тестирование 