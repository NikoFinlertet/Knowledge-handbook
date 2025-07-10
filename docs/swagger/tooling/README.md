# 🔧 Инструменты OpenAPI

## 🎯 Обзор раздела

Этот раздел посвящен инструментам и toolchain для работы с OpenAPI спецификациями. Материалы помогут выбрать правильные инструменты для разработки, валидации и работы с API документацией.

## 📚 Содержание

### 🛠️ Категории инструментов
- **Code Generation** - Генерация клиентов и серверов
- **Documentation Tools** - Создание документации
- **Validation & Testing** - Валидация и тестирование
- **Development Tools** - Редакторы и IDE плагины
- **CI/CD Integration** - Интеграция в процесс разработки

*Примечание: Практические примеры и конфигурации будут добавлены позже*

## 🎓 Целевая аудитория

- **Backend разработчики** - генерация серверного кода
- **Frontend разработчики** - создание клиентских SDK
- **DevOps инженеры** - интеграция в CI/CD pipeline
- **QA инженеры** - автоматизация тестирования API

## 🛠️ Практические навыки

После изучения этого раздела вы сможете:
- Настроить toolchain для OpenAPI разработки
- Генерировать клиентский и серверный код
- Автоматизировать валидацию API спецификаций
- Интегрировать OpenAPI в CI/CD процесс
- Создавать качественную документацию

## 🔗 Связи с другими разделами

### ➡️ Следующие шаги
- **[Examples](../examples/README.md)** - Практические примеры использования инструментов

### 🔙 Предварительные знания
- **[Fundamentals](../fundamentals/README.md)** - Основы OpenAPI
- **[Specification](../specification/README.md)** - Работа со спецификациями

## 📊 Уровень сложности

| Инструмент | Сложность | Время изучения |
|-----------|-----------|----------------|
| Swagger UI | ⭐ (1/5) | 1-2 дня |
| OpenAPI Generator | ⭐⭐ (2/5) | 1-2 недели |
| Spectral Linter | ⭐⭐⭐ (3/5) | 2-3 недели |
| Custom Tools | ⭐⭐⭐⭐ (4/5) | 3-4 недели |

## 🎯 Рекомендации по изучению

### 1. Практические упражнения
- Настройка базовых инструментов
- Генерация кода из спецификаций
- Создание документации
- Настройка CI/CD pipeline

### 2. Метрики успеха
- Понимание возможностей различных инструментов
- Способность настроить полный toolchain
- Знание best practices автоматизации
- Умение интегрировать в процесс разработки

## 💡 Ключевые инструменты

### Генерация кода
```bash
# OpenAPI Generator
openapi-generator-cli generate \
  -i api-spec.yml \
  -g typescript-fetch \
  -o client/

# Swagger Codegen
swagger-codegen generate \
  -i api-spec.yml \
  -l typescript-fetch \
  -o client/
```

### Валидация спецификаций
```bash
# Spectral linting
spectral lint api-spec.yml

# OpenAPI Generator validation
openapi-generator-cli validate -i api-spec.yml

# Swagger Validator
swagger-validator-cli validate api-spec.yml
```

### Создание документации
```bash
# Swagger UI
swagger-ui-serve api-spec.yml

# ReDoc
redoc-cli build api-spec.yml

# Widdershins (markdown)
widdershins api-spec.yml -o api-docs.md
```

## 🔄 Практическое применение

### Настройка toolchain
1. Выбор инструментов под задачи
2. Настройка конфигурации
3. Интеграция в процесс разработки
4. Автоматизация через CI/CD

### Генерация клиентского кода
```javascript
// Generated TypeScript client
import { DefaultApi, Configuration } from './generated-client';

const config = new Configuration({
  basePath: 'https://api.example.com/v1',
  accessToken: 'your-token'
});

const api = new DefaultApi(config);
const users = await api.getUsers();
```

### Валидация в CI/CD
```yaml
# GitHub Actions example
name: API Validation
on: [push, pull_request]
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Validate OpenAPI spec
        run: |
          npm install -g @stoplight/spectral-cli
          spectral lint api-spec.yml
```

## 📈 Развитие навыков

### Базовые навыки
- Использование Swagger UI
- Базовая генерация кода
- Простая валидация спецификаций
- Создание документации

### Продвинутые навыки
- Настройка custom generators
- Создание linting rules
- Интеграция в CI/CD
- Автоматизация тестирования

### Экспертные навыки
- Разработка собственных инструментов
- Создание enterprise toolchain
- Оптимизация процессов
- Архитектурное планирование

## 🎯 Основные категории инструментов

### 📝 Редакторы и IDE
```bash
# Swagger Editor
docker run -p 8080:8080 swaggerapi/swagger-editor

# VS Code extensions
- OpenAPI (Swagger) Editor
- Swagger Viewer
- YAML Language Support
```

### 🏗️ Генераторы кода
```bash
# OpenAPI Generator (рекомендуется)
openapi-generator-cli generate \
  -i spec.yml \
  -g typescript-fetch \
  -o ./generated-client \
  --additional-properties=npmName=my-api-client

# Swagger Codegen (legacy)
swagger-codegen generate \
  -i spec.yml \
  -l typescript-fetch \
  -o ./generated-client
```

### ✅ Валидаторы и линтеры
```bash
# Spectral (мощный линтер)
spectral lint api-spec.yml --ruleset spectral:oas

# OpenAPI Generator validator
openapi-generator-cli validate -i api-spec.yml

# Swagger Parser
swagger-parser validate api-spec.yml
```

### 📖 Документация
```bash
# Swagger UI
swagger-ui-serve api-spec.yml -p 8080

# ReDoc
redoc-cli serve api-spec.yml -p 8080
redoc-cli build api-spec.yml --output docs.html

# Widdershins (Markdown)
widdershins api-spec.yml -o api-docs.md
```

### 🧪 Тестирование и моки
```bash
# Prism (mock server)
prism mock api-spec.yml -p 4010

# Dredd (API testing)
dredd api-spec.yml http://localhost:3000

# Postman Newman
newman run postman-collection.json
```

## 🔧 Конфигурация инструментов

### OpenAPI Generator
```yaml
# openapitools.json
{
  "generator-cli": {
    "version": "6.0.0",
    "generators": {
      "typescript-fetch": {
        "generatorName": "typescript-fetch",
        "output": "./src/generated",
        "additionalProperties": {
          "npmName": "my-api-client",
          "npmVersion": "1.0.0"
        }
      }
    }
  }
}
```

### Spectral Rules
```yaml
# .spectral.yml
extends: ["spectral:oas"]
rules:
  operation-description: error
  operation-summary: error
  operation-tags: error
  paths-kebab-case: error
  no-$ref-siblings: error
```

### CI/CD Pipeline
```yaml
# .github/workflows/api.yml
name: API Workflow
on: [push, pull_request]
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      - name: Install dependencies
        run: npm install -g @stoplight/spectral-cli
      - name: Lint API spec
        run: spectral lint api-spec.yml
      - name: Generate client
        run: |
          npm install -g @openapitools/openapi-generator-cli
          openapi-generator-cli generate -i api-spec.yml -g typescript-fetch -o client/
```

## 🏆 Best Practices

### Выбор инструментов
- **OpenAPI Generator** вместо Swagger Codegen
- **Spectral** для комплексного linting
- **Prism** для mock серверов
- **ReDoc** для красивой документации

### Автоматизация
- Валидация в pre-commit hooks
- Генерация кода в CI/CD
- Автоматическое обновление документации
- Версионирование generated artifacts

### Организация проекта
```
project/
├── api/
│   ├── spec.yml              # OpenAPI спецификация
│   ├── .spectral.yml        # Linting rules
│   └── openapitools.json    # Generator config
├── generated/
│   ├── client/              # Generated client code
│   └── server/              # Generated server stubs
└── docs/
    ├── api.html             # Generated docs
    └── postman.json         # Generated collection
```

## 🔍 Сравнение инструментов

### Генераторы кода
| Инструмент | Языки | Поддержка | Активность |
|-----------|-------|-----------|------------|
| OpenAPI Generator | 50+ | OpenAPI 3.0+ | Активная |
| Swagger Codegen | 40+ | OpenAPI 2.0/3.0 | Поддержка |
| Custom generators | По необходимости | Любая | Зависит от команды |

### Валидаторы
| Инструмент | Возможности | Кастомизация | Производительность |
|-----------|-------------|--------------|-------------------|
| Spectral | Мощные правила | Высокая | Высокая |
| Swagger Validator | Базовая | Низкая | Средняя |
| OpenAPI Generator | Встроенная | Средняя | Высокая |

### Документация
| Инструмент | Внешний вид | Интерактивность | Кастомизация |
|-----------|-------------|----------------|--------------|
| Swagger UI | Стандартный | Высокая | Средняя |
| ReDoc | Красивый | Средняя | Высокая |
| Widdershins | Markdown | Низкая | Высокая |

## 💻 Практические команды

### Установка инструментов
```bash
# Node.js инструменты
npm install -g @openapitools/openapi-generator-cli
npm install -g @stoplight/spectral-cli
npm install -g redoc-cli
npm install -g @apidevtools/swagger-parser

# Docker альтернативы
docker pull swaggerapi/swagger-ui
docker pull swaggerapi/swagger-editor
docker pull stoplight/prism
```

### Быстрая настройка
```bash
# Создание базовой структуры
mkdir my-api && cd my-api
openapi-generator-cli author template -g typescript-fetch

# Валидация спецификации
spectral lint api-spec.yml

# Генерация клиента
openapi-generator-cli generate -i api-spec.yml -g typescript-fetch -o client/

# Запуск документации
swagger-ui-serve api-spec.yml
```

### Интеграция с проектом
```bash
# package.json scripts
{
  "scripts": {
    "api:lint": "spectral lint api-spec.yml",
    "api:generate": "openapi-generator-cli generate -i api-spec.yml -g typescript-fetch -o src/generated",
    "api:docs": "redoc-cli build api-spec.yml --output docs/api.html",
    "api:mock": "prism mock api-spec.yml -p 4010"
  }
}
``` 