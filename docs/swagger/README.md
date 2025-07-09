# 📋 API Documentation

## 🎯 Описание

Этот раздел содержит OpenAPI спецификации для всех API сервисов проекта. Спецификации используются для автоматической генерации документации и клиентских SDK.

---

## 📚 Файлы

### [api-spec.yml](api-spec.yml)
**Основная OpenAPI спецификация для всех микросервисов**

#### 🔧 Включает:
- **User Service API** - Управление пользователями
- **Order Service API** - Управление заказами
- **API Gateway** - Единая точка входа
- **Authentication** - JWT токены и авторизация

#### 📊 Покрытие:
- ✅ **REST endpoints** - Все CRUD операции
- ✅ **GraphQL schema** - Unified API
- ✅ **Authentication** - JWT и OAuth2
- ✅ **Error handling** - Стандартные коды ошибок
- ✅ **Validation** - Схемы валидации данных

---

## 🚀 Использование

### Просмотр документации

1. **Swagger UI** (локально):
   ```bash
   # Запустите все сервисы
   docker-compose up -d
   
   # Откройте в браузере
   http://localhost:4000/docs
   ```

2. **Swagger UI** (онлайн):
   ```bash
   # Скопируйте содержимое api-spec.yml
   # Вставьте в https://editor.swagger.io/
   ```

### Генерация клиентов

```bash
# Генерация JavaScript клиента
swagger-codegen generate -i api-spec.yml -l javascript -o ./clients/js

# Генерация Python клиента
swagger-codegen generate -i api-spec.yml -l python -o ./clients/python

# Генерация Go клиента
swagger-codegen generate -i api-spec.yml -l go -o ./clients/go
```

---

## 🔄 Обновление спецификации

### Автоматическое обновление

API спецификация автоматически обновляется при изменении кода:

- **User Service**: `go run cmd/main.go` генерирует OpenAPI из аннотаций
- **Order Service**: `python main.py` генерирует OpenAPI из FastAPI
- **API Gateway**: GraphQL schema экспортируется в OpenAPI формат

### Ручное обновление

```bash
# Объединение спецификаций всех сервисов
swagger-codegen-cli generate \
  -i services/user-service/api/swagger.json \
  -i services/order-service/api/swagger.json \
  -i services/api-gateway/api/swagger.json \
  -o docs/swagger/api-spec.yml
```

---

## 🎯 Стандарты API

### REST API Guidelines

- **HTTP методы**: GET, POST, PUT, DELETE, PATCH
- **Status codes**: 200, 201, 400, 401, 403, 404, 500
- **Content-Type**: application/json
- **Authentication**: Bearer JWT tokens

### GraphQL Guidelines

- **Schema**: Type-first approach
- **Queries**: Для чтения данных
- **Mutations**: Для изменения данных
- **Subscriptions**: Для real-time обновлений

### Error Handling

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  }
}
```

---

## 🔧 Полезные инструменты

### Валидация

```bash
# Валидация OpenAPI спецификации
swagger-codegen-cli validate -i api-spec.yml

# Линтинг API спецификации
spectral lint api-spec.yml
```

### Тестирование

```bash
# Тестирование API с помощью Postman
newman run postman_collection.json

# Тестирование с помощью curl
curl -X GET "http://localhost:4000/api/users" \
  -H "Authorization: Bearer $JWT_TOKEN"
```

---

## 📖 Дополнительные ресурсы

### 📚 Документация
- [OpenAPI Specification](https://swagger.io/specification/)
- [Swagger Tools](https://swagger.io/tools/)
- [GraphQL Best Practices](https://graphql.org/learn/best-practices/)

### 🛠️ Инструменты
- [Swagger Editor](https://editor.swagger.io/)
- [Postman](https://www.postman.com/)
- [Insomnia](https://insomnia.rest/)
- [GraphQL Playground](https://github.com/graphql/graphql-playground)

---

## 🔗 Связанные материалы

- [API Design](../architecture/api-design.md) - Принципы проектирования API
- [Testing](../technical-skills/testing.md) - Тестирование API
- [Security](../technical-skills/security.md) - Безопасность API
- [Microservices](../architecture/microservices-architecture.md) - API в микросервисах 