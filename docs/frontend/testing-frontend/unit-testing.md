# 🔧 Unit Testing с Jest

## 📋 Навигация
- [Основы Jest](#основы-jest)
- [Мокирование](#мокирование)
- [Асинхронное тестирование](#асинхронное-тестирование)
- [Продвинутые возможности](#продвинутые-возможности)
- [Конфигурация Jest](#конфигурация-jest)

---

## 🎯 Основы Jest

### Базовые функции для тестирования

```javascript
// math.js
export const add = (a, b) => a + b;
export const multiply = (a, b) => a * b;
export const divide = (a, b) => {
  if (b === 0) {
    throw new Error('Division by zero');
  }
  return a / b;
};

export const calculateDiscount = (price, discountPercent) => {
  if (price < 0 || discountPercent < 0 || discountPercent > 100) {
    throw new Error('Invalid parameters');
  }
  return price * (1 - discountPercent / 100);
};
```

### Базовые тесты

```javascript
// math.test.js
import { add, multiply, divide, calculateDiscount } from './math';

describe('Math Functions', () => {
  describe('add function', () => {
    test('should add two positive numbers', () => {
      expect(add(2, 3)).toBe(5);
    });

    test('should add negative numbers', () => {
      expect(add(-2, -3)).toBe(-5);
    });

    test('should handle zero', () => {
      expect(add(0, 5)).toBe(5);
      expect(add(5, 0)).toBe(5);
    });
  });

  describe('divide function', () => {
    test('should divide two numbers', () => {
      expect(divide(10, 2)).toBe(5);
    });

    test('should throw error when dividing by zero', () => {
      expect(() => divide(10, 0)).toThrow('Division by zero');
    });

    test('should handle decimal results', () => {
      expect(divide(10, 3)).toBeCloseTo(3.333, 3);
    });
  });

  describe('calculateDiscount', () => {
    test('should calculate discount correctly', () => {
      expect(calculateDiscount(100, 20)).toBe(80);
      expect(calculateDiscount(50, 10)).toBe(45);
    });

    test('should throw error for invalid parameters', () => {
      expect(() => calculateDiscount(-100, 20)).toThrow('Invalid parameters');
      expect(() => calculateDiscount(100, -10)).toThrow('Invalid parameters');
      expect(() => calculateDiscount(100, 150)).toThrow('Invalid parameters');
    });
  });
});
```

---

## 🎭 Мокирование

### Мокирование модулей

```javascript
// userService.js
import { apiClient } from './apiClient';
import { logger } from './logger';

export class UserService {
  async getUser(id) {
    try {
      logger.info(`Fetching user ${id}`);
      const user = await apiClient.get(`/users/${id}`);
      logger.info(`User ${id} fetched successfully`);
      return user;
    } catch (error) {
      logger.error(`Failed to fetch user ${id}:`, error);
      throw error;
    }
  }

  async createUser(userData) {
    if (!userData.email) {
      throw new Error('Email is required');
    }

    const user = await apiClient.post('/users', userData);
    logger.info(`User created: ${user.id}`);
    return user;
  }
}
```

### Тесты с моками

```javascript
// userService.test.js
import { UserService } from './userService';
import { apiClient } from './apiClient';
import { logger } from './logger';

// Мокаем внешние зависимости
jest.mock('./apiClient');
jest.mock('./logger');

describe('UserService', () => {
  let userService;
  const mockApiClient = apiClient;
  const mockLogger = logger;

  beforeEach(() => {
    userService = new UserService();
    jest.clearAllMocks();
  });

  describe('getUser', () => {
    test('should fetch user successfully', async () => {
      const userId = 1;
      const mockUser = { id: 1, name: 'John Doe', email: 'john@example.com' };
      
      mockApiClient.get.mockResolvedValue(mockUser);

      const result = await userService.getUser(userId);

      expect(result).toEqual(mockUser);
      expect(mockApiClient.get).toHaveBeenCalledWith('/users/1');
      expect(mockLogger.info).toHaveBeenCalledWith('Fetching user 1');
      expect(mockLogger.info).toHaveBeenCalledWith('User 1 fetched successfully');
    });

    test('should handle API errors', async () => {
      const userId = 1;
      const error = new Error('User not found');
      
      mockApiClient.get.mockRejectedValue(error);

      await expect(userService.getUser(userId)).rejects.toThrow('User not found');
      expect(mockLogger.error).toHaveBeenCalledWith('Failed to fetch user 1:', error);
    });
  });

  describe('createUser', () => {
    test('should create user successfully', async () => {
      const userData = { name: 'Jane Doe', email: 'jane@example.com' };
      const createdUser = { id: 2, ...userData };
      
      mockApiClient.post.mockResolvedValue(createdUser);

      const result = await userService.createUser(userData);

      expect(result).toEqual(createdUser);
      expect(mockApiClient.post).toHaveBeenCalledWith('/users', userData);
      expect(mockLogger.info).toHaveBeenCalledWith('User created: 2');
    });

    test('should throw error when email is missing', async () => {
      const userData = { name: 'Jane Doe' };

      await expect(userService.createUser(userData)).rejects.toThrow('Email is required');
      expect(mockApiClient.post).not.toHaveBeenCalled();
    });
  });
});
```

### Spies и частичные моки

```javascript
// fileProcessor.test.js
import fs from 'fs';
import { FileProcessor } from './fileProcessor';

describe('FileProcessor', () => {
  test('should read file and process content', () => {
    const fileContent = 'test content';
    const readFileSpy = jest.spyOn(fs, 'readFileSync').mockReturnValue(fileContent);
    
    const processor = new FileProcessor();
    const result = processor.processFile('test.txt');

    expect(result).toBe('TEST CONTENT');
    expect(readFileSpy).toHaveBeenCalledWith('test.txt', 'utf8');
    
    readFileSpy.mockRestore();
  });

  test('should use real implementation for some methods', () => {
    const processor = new FileProcessor();
    const validateSpy = jest.spyOn(processor, 'validateContent');
    
    processor.processFile('test.txt');
    
    expect(validateSpy).toHaveBeenCalled();
  });
});
```

---

## ⏰ Асинхронное тестирование

### Тестирование Promises

```javascript
// asyncService.js
export class AsyncService {
  async fetchData(url) {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return response.json();
  }

  async processDataWithDelay(data, delay = 1000) {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve(data.map(item => item.toUpperCase()));
      }, delay);
    });
  }
}
```

### Async/await тесты

```javascript
// asyncService.test.js
import { AsyncService } from './asyncService';

// Мокаем fetch
global.fetch = jest.fn();

describe('AsyncService', () => {
  let asyncService;

  beforeEach(() => {
    asyncService = new AsyncService();
    fetch.mockClear();
  });

  describe('fetchData', () => {
    test('should fetch data successfully', async () => {
      const mockData = { id: 1, name: 'Test' };
      fetch.mockResolvedValue({
        ok: true,
        json: async () => mockData,
      });

      const result = await asyncService.fetchData('/api/data');

      expect(result).toEqual(mockData);
      expect(fetch).toHaveBeenCalledWith('/api/data');
    });

    test('should handle fetch errors', async () => {
      fetch.mockResolvedValue({
        ok: false,
        status: 404,
      });

      await expect(asyncService.fetchData('/api/data'))
        .rejects.toThrow('HTTP 404');
    });

    test('should handle network errors', async () => {
      fetch.mockRejectedValue(new Error('Network error'));

      await expect(asyncService.fetchData('/api/data'))
        .rejects.toThrow('Network error');
    });
  });

  describe('processDataWithDelay', () => {
    test('should process data after delay', async () => {
      jest.useFakeTimers();
      
      const testData = ['hello', 'world'];
      const processPromise = asyncService.processDataWithDelay(testData);
      
      // Прокручиваем время
      jest.advanceTimersByTime(1000);
      
      const result = await processPromise;
      expect(result).toEqual(['HELLO', 'WORLD']);
      
      jest.useRealTimers();
    });
  });
});
```

### Тестирование с setTimeout/setInterval

```javascript
// timer.test.js
import { Timer } from './timer';

describe('Timer', () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  test('should call callback after delay', () => {
    const callback = jest.fn();
    const timer = new Timer();
    
    timer.schedule(callback, 1000);
    
    expect(callback).not.toHaveBeenCalled();
    
    jest.advanceTimersByTime(1000);
    
    expect(callback).toHaveBeenCalledTimes(1);
  });

  test('should call callback multiple times for interval', () => {
    const callback = jest.fn();
    const timer = new Timer();
    
    timer.repeat(callback, 500);
    
    jest.advanceTimersByTime(1500);
    
    expect(callback).toHaveBeenCalledTimes(3);
  });
});
```

---

## 🚀 Продвинутые возможности

### Setup и Teardown

```javascript
describe('Database Tests', () => {
  let database;

  // Выполняется один раз перед всеми тестами
  beforeAll(async () => {
    database = new Database();
    await database.connect();
  });

  // Выполняется перед каждым тестом
  beforeEach(async () => {
    await database.clear();
    await database.seed(defaultData);
  });

  // Выполняется после каждого теста
  afterEach(async () => {
    await database.clear();
  });

  // Выполняется один раз после всех тестов
  afterAll(async () => {
    await database.disconnect();
  });

  test('should create user', async () => {
    const user = await database.createUser({ name: 'Test' });
    expect(user.id).toBeDefined();
  });
});
```

### Параметризованные тесты

```javascript
// validation.test.js
describe('Email Validation', () => {
  const validEmails = [
    'test@example.com',
    'user.name@domain.co.uk',
    'user+tag@example.org'
  ];

  const invalidEmails = [
    'invalid-email',
    '@example.com',
    'user@',
    'user..name@example.com'
  ];

  test.each(validEmails)('should validate "%s" as valid email', (email) => {
    expect(isValidEmail(email)).toBe(true);
  });

  test.each(invalidEmails)('should validate "%s" as invalid email', (email) => {
    expect(isValidEmail(email)).toBe(false);
  });

  // Табличный формат
  test.each([
    ['basic', 'test@example.com', true],
    ['subdomain', 'user@sub.example.com', true],
    ['missing @', 'userexample.com', false],
    ['missing domain', 'user@', false],
  ])('%s email: %s should be %s', (description, email, expected) => {
    expect(isValidEmail(email)).toBe(expected);
  });
});
```

### Custom Matchers

```javascript
// customMatchers.js
expect.extend({
  toBeValidEmail(received) {
    const pass = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(received);
    
    if (pass) {
      return {
        message: () => `expected ${received} not to be a valid email`,
        pass: true,
      };
    } else {
      return {
        message: () => `expected ${received} to be a valid email`,
        pass: false,
      };
    }
  },

  toHaveProperty(received, property, value) {
    const hasProperty = Object.prototype.hasOwnProperty.call(received, property);
    const hasCorrectValue = received[property] === value;
    
    const pass = hasProperty && (value !== undefined ? hasCorrectValue : true);
    
    return {
      message: () => pass 
        ? `expected object not to have property ${property}${value !== undefined ? ` with value ${value}` : ''}`
        : `expected object to have property ${property}${value !== undefined ? ` with value ${value}` : ''}`,
      pass,
    };
  }
});

// Использование
test('should validate custom matchers', () => {
  expect('test@example.com').toBeValidEmail();
  expect({ name: 'John', age: 30 }).toHaveProperty('name', 'John');
});
```

---

## ⚙️ Конфигурация Jest

### jest.config.js

```javascript
module.exports = {
  // Корневая директория для тестов
  testMatch: [
    '<rootDir>/src/**/__tests__/**/*.{js,jsx,ts,tsx}',
    '<rootDir>/src/**/*.{test,spec}.{js,jsx,ts,tsx}'
  ],

  // Настройка среды выполнения
  testEnvironment: 'jsdom',

  // Setup файлы
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.js'],

  // Покрытие кода
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.js',
    '!src/serviceWorker.js',
  ],

  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },

  // Трансформация файлов
  transform: {
    '^.+\\.(js|jsx|ts|tsx)$': 'babel-jest',
    '^.+\\.css$': 'jest-transform-css',
  },

  // Мокирование модулей
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
  },

  // Игнорируемые модули
  transformIgnorePatterns: [
    'node_modules/(?!(some-es6-module)/)',
  ],
};
```

### setupTests.js

```javascript
// src/setupTests.js
import '@testing-library/jest-dom';
import './customMatchers';

// Мокируем глобальные объекты
global.ResizeObserver = jest.fn().mockImplementation(() => ({
  observe: jest.fn(),
  unobserve: jest.fn(),
  disconnect: jest.fn(),
}));

// Мокируем localStorage
const localStorageMock = {
  getItem: jest.fn(),
  setItem: jest.fn(),
  removeItem: jest.fn(),
  clear: jest.fn(),
};
global.localStorage = localStorageMock;

// Настройка тестовой среды
beforeEach(() => {
  jest.clearAllMocks();
  localStorage.clear();
});
```

---

## 📊 Best Practices

### 🎯 Принципы unit тестирования

1. **Fast** - тесты должны выполняться быстро
2. **Independent** - тесты не зависят друг от друга
3. **Repeatable** - одинаковые результаты при повторных запусках
4. **Self-validating** - четкий результат pass/fail
5. **Timely** - тесты пишутся вместе с кодом

### 🛠️ Полезные советы

```javascript
// ✅ Хорошо - описательные названия тестов
test('should throw error when email is missing in user data', () => {});

// ❌ Плохо - неясные названия
test('user test', () => {});

// ✅ Хорошо - группировка связанных тестов
describe('UserValidator', () => {
  describe('when validating email', () => {
    test('should accept valid email formats', () => {});
    test('should reject invalid email formats', () => {});
  });
});

// ✅ Хорошо - использование beforeEach для setup
beforeEach(() => {
  mockDatabase.clear();
  mockLogger.reset();
});
```

---

## 🔗 Связанные разделы

- [[integration-testing|Integration Testing]] - Интеграционное тестирование
- [[react-testing|React Testing]] - Тестирование React компонентов
- [[testing-patterns|Testing Patterns]] - Паттерны тестирования
- [[e2e-testing|E2E Testing]] - End-to-End тестирование
- [[../fundamentals/testing-theory|Testing Theory]] - Теория тестирования 