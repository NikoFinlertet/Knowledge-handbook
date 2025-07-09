# 🚀 E2E Testing с Cypress

## 📋 Навигация
- [Cypress Setup](#cypress-setup)
- [Basic E2E Tests](#basic-e2e-tests)
- [Page Objects](#page-objects)
- [Advanced Scenarios](#advanced-scenarios)
- [Performance Testing](#performance-testing)
- [Accessibility Testing](#accessibility-testing)

---

## ⚙️ Cypress Setup

### Installation

```bash
npm install --save-dev cypress
```

### Custom Commands

```javascript
// cypress/support/commands.js
Cypress.Commands.add('login', (email, password) => {
  cy.visit('/login');
  cy.get('[data-testid="email-input"]').type(email);
  cy.get('[data-testid="password-input"]').type(password);
  cy.get('[data-testid="submit-button"]').click();
});

Cypress.Commands.add('createUser', (userData) => {
  cy.request({
    method: 'POST',
    url: '/api/users',
    body: userData,
    headers: {
      'Authorization': `Bearer ${Cypress.env('authToken')}`
    }
  });
});

Cypress.Commands.add('resetDatabase', () => {
  cy.request('POST', '/api/test/reset-database');
});

Cypress.Commands.add('seedDatabase', (data) => {
  cy.request('POST', '/api/test/seed-database', data);
});
```

---

## 🧪 Basic E2E Tests

### Authentication Flow

```javascript
// cypress/e2e/auth.cy.js
describe('Authentication Flow', () => {
  beforeEach(() => {
    cy.resetDatabase();
  });

  it('should login with valid credentials', () => {
    cy.createUser({
      email: 'test@example.com',
      password: 'password123',
      name: 'Test User'
    });

    cy.visit('/login');
    cy.get('[data-testid="email-input"]').type('test@example.com');
    cy.get('[data-testid="password-input"]').type('password123');
    cy.get('[data-testid="submit-button"]').click();

    cy.url().should('include', '/dashboard');
    cy.get('[data-testid="user-menu"]').should('be.visible');
    cy.get('[data-testid="user-menu"]').should('contain', 'Test User');
  });

  it('should show error for invalid credentials', () => {
    cy.visit('/login');
    cy.get('[data-testid="email-input"]').type('invalid@example.com');
    cy.get('[data-testid="password-input"]').type('wrongpassword');
    cy.get('[data-testid="submit-button"]').click();

    cy.get('[role="alert"]').should('contain', 'Invalid email or password');
    cy.url().should('include', '/login');
  });

  it('should logout user', () => {
    cy.login('test@example.com', 'password123');
    cy.url().should('include', '/dashboard');
    
    cy.get('[data-testid="user-menu"]').click();
    cy.get('[data-testid="logout-button"]').click();
    
    cy.url().should('include', '/login');
  });

  it('should persist authentication across page reloads', () => {
    cy.login('test@example.com', 'password123');
    cy.reload();
    cy.get('[data-testid="user-menu"]').should('be.visible');
  });
});
```

---

## 📄 Page Objects

### Login Page Object

```javascript
// cypress/support/pageObjects/LoginPage.js
class LoginPage {
  visit() {
    cy.visit('/login');
    return this;
  }

  fillEmail(email) {
    cy.get('[data-testid="email-input"]').type(email);
    return this;
  }

  fillPassword(password) {
    cy.get('[data-testid="password-input"]').type(password);
    return this;
  }

  submit() {
    cy.get('[data-testid="submit-button"]').click();
    return this;
  }

  shouldShowError(message) {
    cy.get('[role="alert"]').should('contain', message);
    return this;
  }

  shouldRedirectToDashboard() {
    cy.url().should('include', '/dashboard');
    return this;
  }
}

export const loginPage = new LoginPage();
```

### Using Page Objects

```javascript
// cypress/e2e/loginWithPageObject.cy.js
import { loginPage } from '../support/pageObjects/LoginPage';

describe('Login with Page Object', () => {
  beforeEach(() => {
    cy.resetDatabase();
  });

  it('should login successfully', () => {
    cy.createUser({
      email: 'test@example.com',
      password: 'password123',
      name: 'Test User'
    });

    loginPage
      .visit()
      .fillEmail('test@example.com')
      .fillPassword('password123')
      .submit()
      .shouldRedirectToDashboard();
  });

  it('should show error for invalid credentials', () => {
    loginPage
      .visit()
      .fillEmail('invalid@example.com')
      .fillPassword('wrongpassword')
      .submit()
      .shouldShowError('Invalid email or password');
  });
});
```

---

## 🎯 Advanced Scenarios

### User Management

```javascript
// cypress/e2e/userManagement.cy.js
describe('User Management', () => {
  beforeEach(() => {
    cy.resetDatabase();
    cy.login('admin@example.com', 'admin123');
  });

  it('should create new user', () => {
    cy.visit('/users');
    cy.get('[data-testid="add-user-button"]').click();
    
    cy.get('[data-testid="name-input"]').type('New User');
    cy.get('[data-testid="email-input"]').type('newuser@example.com');
    cy.get('[data-testid="password-input"]').type('password123');
    cy.get('[data-testid="role-select"]').select('user');
    
    cy.get('[data-testid="save-button"]').click();
    
    cy.get('[data-testid="success-message"]').should('contain', 'User created successfully');
    cy.get('[data-testid="user-list"]').should('contain', 'New User');
  });

  it('should validate form fields', () => {
    cy.visit('/users');
    cy.get('[data-testid="add-user-button"]').click();
    cy.get('[data-testid="save-button"]').click();
    
    cy.get('[data-testid="name-input"]').should('have.attr', 'aria-invalid', 'true');
    cy.get('[data-testid="email-input"]').should('have.attr', 'aria-invalid', 'true');
  });

  it('should edit existing user', () => {
    cy.seedDatabase({
      users: [{ name: 'John Doe', email: 'john@example.com', role: 'user' }]
    });

    cy.visit('/users');
    cy.get('[data-testid="user-item"]').first().find('[data-testid="edit-button"]').click();
    
    cy.get('[data-testid="name-input"]').clear().type('John Updated');
    cy.get('[data-testid="save-button"]').click();
    
    cy.get('[data-testid="user-list"]').should('contain', 'John Updated');
  });
});
```

### Shopping Cart Flow

```javascript
// cypress/e2e/shopping.cy.js
describe('Shopping Cart Flow', () => {
  beforeEach(() => {
    cy.resetDatabase();
    cy.seedDatabase({
      products: [
        { id: 1, name: 'Product 1', price: 29.99, stock: 10 },
        { id: 2, name: 'Product 2', price: 39.99, stock: 5 }
      ]
    });
  });

  it('should complete purchase flow', () => {
    cy.visit('/products');
    
    // Добавляем товары в корзину
    cy.get('[data-product-id="1"] [data-testid="add-to-cart"]').click();
    cy.get('[data-product-id="2"] [data-testid="add-to-cart"]').click();
    
    // Проверяем счетчик корзины
    cy.get('[data-testid="cart-counter"]').should('contain', '2');
    
    // Переходим в корзину
    cy.get('[data-testid="cart-link"]').click();
    cy.url().should('include', '/cart');
    
    // Проверяем содержимое корзины
    cy.get('[data-testid="cart-item"]').should('have.length', 2);
    cy.get('[data-testid="total-amount"]').should('contain', '$69.98');
    
    // Оформляем заказ
    cy.get('[data-testid="checkout-button"]').click();
    
    // Заполняем форму доставки
    cy.get('[data-testid="shipping-name"]').type('John Doe');
    cy.get('[data-testid="shipping-address"]').type('123 Main St');
    cy.get('[data-testid="shipping-city"]').type('New York');
    cy.get('[data-testid="shipping-zip"]').type('10001');
    
    // Выбираем способ оплаты
    cy.get('[data-testid="payment-method"]').select('credit-card');
    cy.get('[data-testid="card-number"]').type('4242424242424242');
    cy.get('[data-testid="card-expiry"]').type('12/25');
    cy.get('[data-testid="card-cvc"]').type('123');
    
    // Подтверждаем заказ
    cy.get('[data-testid="place-order-button"]').click();
    
    // Проверяем успешное оформление
    cy.get('[data-testid="order-success"]').should('be.visible');
    cy.url().should('include', '/order-confirmation');
  });
});
```

---

## ⚡ Performance Testing

### Load Time Tests

```javascript
// cypress/e2e/performance.cy.js
describe('Performance Tests', () => {
  it('should load dashboard within acceptable time', () => {
    cy.login('test@example.com', 'password123');
    
    cy.window().then((win) => {
      const start = win.performance.now();
      
      cy.visit('/dashboard').then(() => {
        const end = win.performance.now();
        const loadTime = end - start;
        
        expect(loadTime).to.be.lessThan(2000); // менее 2 секунд
      });
    });
  });

  it('should measure Core Web Vitals', () => {
    cy.visit('/');
    
    cy.window().then((win) => {
      // Largest Contentful Paint
      new win.PerformanceObserver((list) => {
        const entries = list.getEntries();
        const lcp = entries[entries.length - 1];
        expect(lcp.startTime).to.be.lessThan(2500); // LCP < 2.5s
      }).observe({ entryTypes: ['largest-contentful-paint'] });
      
      // First Input Delay
      new win.PerformanceObserver((list) => {
        list.getEntries().forEach((entry) => {
          expect(entry.processingStart - entry.startTime).to.be.lessThan(100); // FID < 100ms
        });
      }).observe({ entryTypes: ['first-input'] });
    });
  });
});
```

---

## ♿ Accessibility Testing

### A11y Tests

```javascript
// cypress/e2e/accessibility.cy.js
describe('Accessibility Tests', () => {
  beforeEach(() => {
    cy.visit('/');
    cy.injectAxe();
  });

  it('should have no accessibility violations', () => {
    cy.checkA11y();
  });

  it('should support keyboard navigation', () => {
    cy.visit('/login');
    
    // Навигация по Tab
    cy.get('body').tab();
    cy.focused().should('have.attr', 'data-testid', 'email-input');
    
    cy.focused().tab();
    cy.focused().should('have.attr', 'data-testid', 'password-input');
    
    cy.focused().tab();
    cy.focused().should('have.attr', 'data-testid', 'submit-button');
  });

  it('should have proper ARIA attributes', () => {
    cy.visit('/dashboard');
    
    // Проверяем роли
    cy.get('[role="navigation"]').should('exist');
    cy.get('[role="main"]').should('exist');
    
    // Проверяем aria-labels
    cy.get('[data-testid="user-menu"]').should('have.attr', 'aria-label');
    
    // Проверяем aria-expanded для выпадающих меню
    cy.get('[data-testid="dropdown-toggle"]').should('have.attr', 'aria-expanded', 'false');
    cy.get('[data-testid="dropdown-toggle"]').click();
    cy.get('[data-testid="dropdown-toggle"]').should('have.attr', 'aria-expanded', 'true');
  });

  it('should have proper heading hierarchy', () => {
    cy.visit('/dashboard');
    
    // Проверяем иерархию заголовков
    cy.get('h1').should('have.length', 1);
    cy.get('h1').should('contain', 'Dashboard');
    
    // H2 должны идти после H1
    cy.get('h2').each(($h2) => {
      cy.get('h1').should('exist');
    });
  });

  it('should have sufficient color contrast', () => {
    cy.visit('/');
    cy.checkA11y(null, {
      rules: {
        'color-contrast': { enabled: true }
      }
    });
  });
});
```

---

## 📊 Best Practices

### 🎯 Принципы E2E тестирования

1. **Тестируйте критические пути пользователя**
2. **Минимизируйте количество E2E тестов**
3. **Используйте стабильные селекторы**
4. **Изолируйте тесты друг от друга**
5. **Подготавливайте данные через API**

### 🛠️ Полезные советы

```javascript
// ✅ Хорошо - использование data-testid
cy.get('[data-testid="submit-button"]').click();

// ❌ Плохо - нестабильные селекторы
cy.get('.btn.btn-primary.submit').click();

// ✅ Хорошо - подготовка данных через API
beforeEach(() => {
  cy.resetDatabase();
  cy.createUser({ email: 'test@example.com' });
});

// ❌ Плохо - подготовка данных через UI
beforeEach(() => {
  cy.visit('/register');
  cy.fillRegistrationForm();
  cy.submit();
});

// ✅ Хорошо - проверка состояния
cy.get('[data-testid="loading"]').should('not.exist');
cy.get('[data-testid="user-list"]').should('contain', 'John Doe');

// ✅ Хорошо - ожидание элементов
cy.get('[data-testid="modal"]').should('be.visible');
cy.get('[data-testid="form-submit"]').should('be.enabled');
```

---

## 🔗 Связанные разделы

- [[unit-testing|Unit Testing]] - Модульное тестирование
- [[integration-testing|Integration Testing]] - Интеграционное тестирование
- [[react-testing|React Testing]] - Тестирование React компонентов
- [[testing-patterns|Testing Patterns]] - Паттерны тестирования
- [[../fundamentals/accessibility|Accessibility]] - Доступность веб-приложений 