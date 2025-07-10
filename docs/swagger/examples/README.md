# 🎯 Практические примеры

## 🎯 Обзор раздела

Этот раздел содержит практические примеры использования OpenAPI в реальных проектах. Материалы демонстрируют различные сценарии применения, от простых API до сложных enterprise-решений.

## 📚 Содержание

### 🏗️ Практические сценарии
- **CRUD API Examples** - Примеры стандартных CRUD операций
- **Authentication Patterns** - Различные методы аутентификации
- **Microservices Integration** - Интеграция в микросервисную архитектуру
- **Enterprise Patterns** - Сложные enterprise-решения
- **CI/CD Integration** - Интеграция в процесс разработки

*Примечание: Практические примеры и код будут добавлены позже*

## 🎓 Целевая аудитория

- **Backend разработчики** - реализация API по спецификации
- **Frontend разработчики** - интеграция с API
- **DevOps инженеры** - автоматизация процессов
- **Архитекторы** - проектирование API стратегии

## 🛠️ Практические навыки

После изучения этого раздела вы сможете:
- Применять OpenAPI в реальных проектах
- Интегрировать API документацию в процесс разработки
- Создавать качественные API следуя best practices
- Автоматизировать генерацию кода и тестирование
- Работать с различными архитектурными паттернами

## 🔗 Связи с другими разделами

### 🔙 Предварительные знания
- **[Fundamentals](../fundamentals/README.md)** - Основы OpenAPI
- **[Specification](../specification/README.md)** - Работа со спецификациями
- **[Tooling](../tooling/README.md)** - Инструменты разработки

### ➡️ Дальнейшее изучение
- **[Backend APIs](../../backend/apis/README.md)** - Проектирование API
- **[Frontend](../../frontend/README.md)** - Интеграция на клиенте
- **[Infrastructure](../../infrastructure/README.md)** - Развертывание API

## 📊 Уровень сложности

| Пример | Сложность | Время изучения |
|--------|-----------|----------------|
| Simple CRUD | ⭐⭐ (2/5) | 1-2 недели |
| Authentication | ⭐⭐⭐ (3/5) | 2-3 недели |
| Microservices | ⭐⭐⭐⭐ (4/5) | 3-4 недели |
| Enterprise | ⭐⭐⭐⭐⭐ (5/5) | 4-6 недель |

## 🎯 Рекомендации по изучению

### 1. Практические упражнения
- Реализация примеров в своем проекте
- Модификация под свои требования
- Создание собственных паттернов
- Интеграция в существующие системы

### 2. Метрики успеха
- Понимание практических паттернов
- Способность адаптировать примеры
- Знание лучших практик на основе опыта
- Умение создавать enterprise-решения

## 💡 Практические сценарии

### Простой CRUD API
```yaml
# User Management API
openapi: 3.0.0
info:
  title: User Management API
  version: 1.0.0
paths:
  /users:
    get:
      summary: Get all users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 10
      responses:
        '200':
          description: List of users
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  pagination:
                    $ref: '#/components/schemas/Pagination'
```

### Аутентификация с JWT
```yaml
# Authentication endpoints
paths:
  /auth/login:
    post:
      summary: User login
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                email:
                  type: string
                  format: email
                password:
                  type: string
                  minLength: 8
      responses:
        '200':
          description: Login successful
          content:
            application/json:
              schema:
                type: object
                properties:
                  token:
                    type: string
                  user:
                    $ref: '#/components/schemas/User'
```

### Микросервисная архитектура
```yaml
# Service-to-service communication
info:
  title: Order Service API
  version: 1.0.0
paths:
  /orders:
    post:
      summary: Create order
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateOrderRequest'
      responses:
        '201':
          description: Order created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
```

## 🔄 Практическое применение

### Разработка API-first
1. Создание спецификации перед кодом
2. Совместная работа над API дизайном
3. Генерация серверных заглушек
4. Параллельная разработка клиента и сервера

### Интеграция в CI/CD
```yaml
# GitHub Actions workflow
name: API Development
on: [push, pull_request]
jobs:
  api-workflow:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Validate API spec
        run: spectral lint api-spec.yml
      
      - name: Generate client code
        run: |
          openapi-generator-cli generate \
            -i api-spec.yml \
            -g typescript-fetch \
            -o client/
      
      - name: Run contract tests
        run: |
          prism mock api-spec.yml &
          npm test -- --testPathPattern=contract
```

### Автоматизация тестирования
```javascript
// Contract testing example
import { DefaultApi } from './generated-client';

describe('User API Contract Tests', () => {
  const api = new DefaultApi({ basePath: 'http://localhost:4010' });
  
  it('should create user with valid data', async () => {
    const user = await api.createUser({
      email: 'test@example.com',
      name: 'Test User'
    });
    
    expect(user.id).toBeDefined();
    expect(user.email).toBe('test@example.com');
  });
});
```

## 📈 Развитие навыков

### Базовые сценарии
- Создание простых CRUD API
- Базовая аутентификация
- Генерация клиентского кода
- Простые тесты

### Продвинутые паттерны
- Сложные схемы валидации
- Множественная аутентификация
- Версионирование API
- Интеграция с базами данных

### Enterprise-решения
- Микросервисная архитектура
- API Gateway интеграция
- Мониторинг и логирование
- Безопасность и compliance

## 🎯 Архитектурные паттерны

### API Gateway Pattern
```yaml
# API Gateway configuration
openapi: 3.0.0
info:
  title: API Gateway
  version: 1.0.0
paths:
  /api/v1/users/{proxy+}:
    x-amazon-apigateway-any-method:
      x-amazon-apigateway-integration:
        uri: http://user-service:3000/{proxy}
        httpMethod: ANY
        type: http_proxy
```

### Backend for Frontend (BFF)
```yaml
# Mobile-specific API
paths:
  /mobile/dashboard:
    get:
      summary: Get mobile dashboard data
      responses:
        '200':
          description: Mobile-optimized dashboard
          content:
            application/json:
              schema:
                type: object
                properties:
                  user:
                    $ref: '#/components/schemas/UserSummary'
                  notifications:
                    type: array
                    items:
                      $ref: '#/components/schemas/Notification'
```

### Event-Driven Architecture
```yaml
# Event publishing API
paths:
  /events:
    post:
      summary: Publish event
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                eventType:
                  type: string
                  enum: ['user.created', 'order.placed']
                payload:
                  type: object
```

## 🏆 Best Practices примеры

### Консистентность API
```yaml
# Consistent response structure
components:
  schemas:
    ApiResponse:
      type: object
      properties:
        success:
          type: boolean
        data:
          type: object
        error:
          type: object
          properties:
            code:
              type: string
            message:
              type: string
```

### Обработка ошибок
```yaml
# Comprehensive error handling
responses:
  '400':
    description: Bad Request
    content:
      application/json:
        schema:
          type: object
          properties:
            error:
              type: object
              properties:
                type:
                  type: string
                  example: 'VALIDATION_ERROR'
                message:
                  type: string
                  example: 'Invalid input data'
                details:
                  type: array
                  items:
                    type: object
                    properties:
                      field:
                        type: string
                      message:
                        type: string
```

### Версионирование
```yaml
# API versioning strategy
info:
  title: My API
  version: 2.0.0
paths:
  /v2/users:
    get:
      summary: Get users (v2)
      deprecated: false
  /v1/users:
    get:
      summary: Get users (v1)
      deprecated: true
      description: "This endpoint is deprecated. Use /v2/users instead."
```

## 🔧 Инструменты и шаблоны

### Проектные шаблоны
```bash
# Project structure
api-project/
├── specs/
│   ├── user-service.yml
│   ├── order-service.yml
│   └── gateway.yml
├── generated/
│   ├── clients/
│   └── servers/
├── tests/
│   ├── contract/
│   └── integration/
└── docs/
    ├── api.html
    └── postman.json
```

### Автоматизация
```bash
# Makefile example
.PHONY: validate generate test deploy

validate:
	spectral lint specs/*.yml

generate:
	openapi-generator-cli generate -i specs/user-service.yml -g spring -o server/
	openapi-generator-cli generate -i specs/user-service.yml -g typescript-fetch -o client/

test:
	prism mock specs/user-service.yml &
	npm test
	kill %1

deploy:
	docker build -t api-server .
	docker push api-server:latest
```

### Мониторинг и метрики
```yaml
# Health check endpoints
paths:
  /health:
    get:
      summary: Health check
      responses:
        '200':
          description: Service is healthy
          content:
            application/json:
              schema:
                type: object
                properties:
                  status:
                    type: string
                    enum: ['healthy', 'degraded', 'unhealthy']
                  timestamp:
                    type: string
                    format: date-time
                  dependencies:
                    type: array
                    items:
                      type: object
                      properties:
                        name:
                          type: string
                        status:
                          type: string
```

## 💻 Готовые решения

### Стартовый проект
```yaml
# Basic starter template
openapi: 3.0.0
info:
  title: Starter API
  version: 1.0.0
  description: Ready-to-use API template
servers:
  - url: http://localhost:3000
    description: Development server
paths:
  /health:
    get:
      summary: Health check
      responses:
        '200':
          description: OK
  /users:
    get:
      summary: List users
      responses:
        '200':
          description: Users list
    post:
      summary: Create user
      responses:
        '201':
          description: User created
components:
  schemas:
    User:
      type: object
      required: [email]
      properties:
        id:
          type: integer
        email:
          type: string
          format: email
        name:
          type: string
```

### Enterprise template
```yaml
# Enterprise-ready template
openapi: 3.0.0
info:
  title: Enterprise API
  version: 1.0.0
  description: Enterprise-grade API template
  contact:
    name: API Support Team
    email: api-support@company.com
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT
servers:
  - url: https://api.company.com/v1
    description: Production
  - url: https://api-staging.company.com/v1
    description: Staging
security:
  - bearerAuth: []
paths:
  # Standard endpoints with full documentation
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
  schemas:
    # Complete schema definitions
``` 