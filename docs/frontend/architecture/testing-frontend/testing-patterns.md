# 🎯 Testing Patterns

## 📋 Навигация
- [Test Organization](#test-organization)
- [Page Object Pattern](#page-object-pattern)
- [Factory Pattern](#factory-pattern)
- [Custom Matchers](#custom-matchers)
- [Advanced Patterns](#advanced-patterns)

---

## 🗂️ Test Organization

### Test Utils

```javascript
// testUtils.js
import React from 'react';
import { render } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import { QueryClient, QueryClientProvider } from 'react-query';
import { ThemeProvider } from './ThemeContext';

const createTestQueryClient = () => new QueryClient({
  defaultOptions: { queries: { retry: false } }
});

export const AllTheProviders = ({ children }) => {
  const queryClient = createTestQueryClient();
  
  return (
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>
        <ThemeProvider>
          {children}
        </ThemeProvider>
      </BrowserRouter>
    </QueryClientProvider>
  );
};

export const renderWithProviders = (ui, options) => {
  return render(ui, { wrapper: AllTheProviders, ...options });
};
```

### Data Test IDs

```javascript
// dataTestIds.js
export const DATA_TEST_IDS = {
  // Navigation
  MAIN_NAVIGATION: 'main-navigation',
  USER_MENU: 'user-menu',
  LOGOUT_BUTTON: 'logout-button',
  
  // Forms
  LOGIN_FORM: 'login-form',
  EMAIL_INPUT: 'email-input',
  PASSWORD_INPUT: 'password-input',
  SUBMIT_BUTTON: 'submit-button',
  
  // Lists
  USER_LIST: 'user-list',
  USER_ITEM: 'user-item',
  LOADING_SPINNER: 'loading-spinner',
  
  // Modals
  MODAL_OVERLAY: 'modal-overlay',
  MODAL_CONTENT: 'modal-content',
  MODAL_CLOSE: 'modal-close'
};
```

---

## 📄 Page Object Pattern

### Login Page Object

```javascript
// pageObjects/LoginPage.js
import { screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

export class LoginPage {
  constructor() {
    this.user = userEvent.setup();
  }

  get emailInput() {
    return screen.getByLabelText(/email/i);
  }

  get passwordInput() {
    return screen.getByLabelText(/password/i);
  }

  get submitButton() {
    return screen.getByRole('button', { name: /sign in/i });
  }

  get errorMessage() {
    return screen.queryByRole('alert');
  }

  async fillEmail(email) {
    await this.user.clear(this.emailInput);
    await this.user.type(this.emailInput, email);
  }

  async fillPassword(password) {
    await this.user.clear(this.passwordInput);
    await this.user.type(this.passwordInput, password);
  }

  async submit() {
    await this.user.click(this.submitButton);
  }

  async login(email, password) {
    await this.fillEmail(email);
    await this.fillPassword(password);
    await this.submit();
  }

  shouldShowError(message) {
    expect(this.errorMessage).toBeInTheDocument();
    expect(this.errorMessage).toHaveTextContent(message);
  }

  shouldRedirectTo(path) {
    expect(window.location.pathname).toBe(path);
  }
}
```

### Using Page Objects

```javascript
// login.test.js
import React from 'react';
import { render } from '@testing-library/react';
import { LoginForm } from './LoginForm';
import { LoginPage } from '../pageObjects/LoginPage';

describe('Login Form', () => {
  let loginPage;

  beforeEach(() => {
    render(<LoginForm />);
    loginPage = new LoginPage();
  });

  test('should login with valid credentials', async () => {
    await loginPage.login('user@example.com', 'password123');
    loginPage.shouldRedirectTo('/dashboard');
  });

  test('should show error for invalid credentials', async () => {
    await loginPage.login('invalid@email.com', 'wrongpassword');
    loginPage.shouldShowError('Invalid credentials');
  });
});
```

---

## 🏭 Factory Pattern

### Test Data Factories

```javascript
// factories/UserFactory.js
export class UserFactory {
  static create(overrides = {}) {
    return {
      id: Math.floor(Math.random() * 1000),
      name: 'Test User',
      email: 'test@example.com',
      role: 'user',
      isActive: true,
      createdAt: new Date().toISOString(),
      ...overrides
    };
  }

  static createMany(count, overrides = {}) {
    return Array.from({ length: count }, (_, index) => 
      this.create({ id: index + 1, ...overrides })
    );
  }

  static createAdmin(overrides = {}) {
    return this.create({ 
      role: 'admin', 
      permissions: ['read', 'write', 'delete'],
      ...overrides 
    });
  }

  static createInactive(overrides = {}) {
    return this.create({ isActive: false, ...overrides });
  }
}

// factories/PostFactory.js
export class PostFactory {
  static create(overrides = {}) {
    return {
      id: Math.floor(Math.random() * 1000),
      title: 'Test Post',
      content: 'This is test content',
      authorId: 1,
      tags: ['test'],
      publishedAt: new Date().toISOString(),
      status: 'published',
      ...overrides
    };
  }

  static createDraft(overrides = {}) {
    return this.create({ 
      status: 'draft', 
      publishedAt: null,
      ...overrides 
    });
  }
}
```

### Using Factories

```javascript
// userService.test.js
import { UserFactory } from '../factories/UserFactory';
import { UserService } from './UserService';

describe('UserService', () => {
  test('should process active users', () => {
    const users = UserFactory.createMany(5, { isActive: true });
    const activeUsers = UserService.getActiveUsers(users);
    
    expect(activeUsers).toHaveLength(5);
    expect(activeUsers.every(user => user.isActive)).toBe(true);
  });

  test('should handle admin permissions', () => {
    const admin = UserFactory.createAdmin();
    
    expect(admin.role).toBe('admin');
    expect(admin.permissions).toContain('delete');
  });
});
```

---

## 🎨 Custom Matchers

### Custom Jest Matchers

```javascript
// customMatchers.js
expect.extend({
  toBeInLoadingState(received) {
    const isLoading = received.hasAttribute('aria-busy') && 
                     received.getAttribute('aria-busy') === 'true';
    
    if (isLoading) {
      return {
        message: () => `expected element not to be in loading state`,
        pass: true,
      };
    } else {
      return {
        message: () => `expected element to be in loading state`,
        pass: false,
      };
    }
  },

  toHaveValidationError(received, expectedError) {
    const errorElement = received.querySelector('[role="alert"]');
    const hasError = errorElement && errorElement.textContent.includes(expectedError);
    
    if (hasError) {
      return {
        message: () => `expected element not to have validation error "${expectedError}"`,
        pass: true,
      };
    } else {
      return {
        message: () => `expected element to have validation error "${expectedError}"`,
        pass: false,
      };
    }
  },

  toHaveExactClasses(received, expectedClasses) {
    const actualClasses = received.className.split(' ').filter(Boolean).sort();
    const expected = expectedClasses.sort();
    
    const pass = actualClasses.length === expected.length &&
                 actualClasses.every((cls, i) => cls === expected[i]);
    
    if (pass) {
      return {
        message: () => `expected element not to have exact classes [${expected.join(', ')}]`,
        pass: true,
      };
    } else {
      return {
        message: () => `expected element to have exact classes [${expected.join(', ')}], but got [${actualClasses.join(', ')}]`,
        pass: false,
      };
    }
  }
});
```

### Using Custom Matchers

```javascript
// component.test.js
import './customMatchers';

test('should be in loading state', () => {
  render(<Button loading>Submit</Button>);
  
  const button = screen.getByRole('button');
  expect(button).toBeInLoadingState();
});

test('should show validation error', () => {
  render(<LoginForm />);
  
  const form = screen.getByRole('form');
  expect(form).toHaveValidationError('Email is required');
});
```

---

## 🚀 Advanced Patterns

### Test Hooks

```javascript
// hooks/useTestSetup.js
import { renderHook } from '@testing-library/react';

export const useTestSetup = (hook, options = {}) => {
  const { wrapper, initialProps } = options;
  
  const result = renderHook(hook, {
    wrapper,
    initialProps
  });
  
  return {
    ...result,
    waitForNextUpdate: result.waitForNextUpdate,
    rerender: result.rerender
  };
};
```

### Temporal Testing

```javascript
// temporalTesting.js
export const withTimezone = (timezone, testFn) => {
  return () => {
    const originalTimezone = process.env.TZ;
    process.env.TZ = timezone;
    
    try {
      return testFn();
    } finally {
      process.env.TZ = originalTimezone;
    }
  };
};

// Использование
test('should format date in different timezone', 
  withTimezone('America/New_York', () => {
    const date = new Date('2023-01-01T12:00:00Z');
    expect(formatDate(date)).toBe('Jan 1, 2023, 7:00 AM');
  })
);
```

### Mock Manager

```javascript
// mockManager.js
export class MockManager {
  constructor() {
    this.mocks = new Map();
  }

  create(name, implementation) {
    const mock = jest.fn(implementation);
    this.mocks.set(name, mock);
    return mock;
  }

  get(name) {
    return this.mocks.get(name);
  }

  resetAll() {
    this.mocks.forEach(mock => mock.mockReset());
  }

  restoreAll() {
    this.mocks.forEach(mock => mock.mockRestore?.());
    this.mocks.clear();
  }
}

// Использование
describe('Service Tests', () => {
  const mockManager = new MockManager();

  beforeEach(() => {
    mockManager.create('apiCall', jest.fn().mockResolvedValue({}));
    mockManager.create('logger', jest.fn());
  });

  afterEach(() => {
    mockManager.resetAll();
  });

  afterAll(() => {
    mockManager.restoreAll();
  });
});
```

---

## 📊 Best Practices

### 🎯 Принципы организации тестов

1. **DRY** - Don't Repeat Yourself
2. **Single Responsibility** - один тест проверяет одну вещь
3. **Описательные имена** - понятные названия тестов
4. **Arrange-Act-Assert** - четкая структура

### 🛠️ Полезные советы

```javascript
// ✅ Хорошо - использование beforeEach для setup
describe('UserService', () => {
  let userService;
  let mockApiClient;

  beforeEach(() => {
    mockApiClient = createMockApiClient();
    userService = new UserService(mockApiClient);
  });

  test('should create user', async () => {
    const userData = UserFactory.create();
    const result = await userService.create(userData);
    expect(result).toBeDefined();
  });
});

// ✅ Хорошо - группировка связанных тестов
describe('UserService', () => {
  describe('when creating user', () => {
    test('should validate required fields', () => {});
    test('should hash password', () => {});
    test('should save to database', () => {});
  });

  describe('when user already exists', () => {
    test('should throw error', () => {});
    test('should not modify existing user', () => {});
  });
});
```

---

## 🔗 Связанные разделы

- [[unit-testing|Unit Testing]] - Модульное тестирование
- [[integration-testing|Integration Testing]] - Интеграционное тестирование
- [[react-testing|React Testing]] - Тестирование React компонентов
- [[e2e-testing|E2E Testing]] - End-to-End тестирование
- [[../fundamentals/design-patterns|Design Patterns]] - Паттерны проектирования 