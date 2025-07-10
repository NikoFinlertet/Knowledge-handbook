# Docker и контейнеризация микросервисов

**Docker** — это платформа для разработки, доставки и запуска приложений в контейнерах. Контейнеризация позволяет упаковать приложение со всеми его зависимостями в легковесный, портативный контейнер, который может запускаться в любой среде.

## Основы Docker

### Ключевые концепции

- **Image (Образ)** — неизменяемый шаблон для создания контейнеров
- **Container (Контейнер)** — запущенный экземпляр образа
- **Dockerfile** — инструкции для создания образа
- **Registry** — хранилище образов (Docker Hub, AWS ECR, etc.)
- **Volume** — постоянное хранилище данных
- **Network** — сеть для коммуникации между контейнерами

---

## Контейнеризация User Service

### Dockerfile для Go микросервиса

```dockerfile
# services/user-service/Dockerfile
# Многоэтапная сборка Docker образа для Go микросервиса

# ==================== ЭТАП 1: СБОРКА ПРИЛОЖЕНИЯ ====================
# Используем официальный образ Go с Alpine Linux для минимального размера
FROM golang:1.21-alpine AS builder

# Устанавливаем системные пакеты, необходимые для сборки
RUN apk add --no-cache \
    git \           # Для работы с git-зависимостями
    ca-certificates \  # SSL сертификаты для HTTPS запросов
    tzdata          # Данные временных зон

# Создаем непривилегированного пользователя для безопасности
# -D = не создавать пароль, -g = основная группа
RUN adduser -D -g '' appuser

# Устанавливаем рабочую директорию внутри контейнера
WORKDIR /build

# ==================== УПРАВЛЕНИЕ ЗАВИСИМОСТЯМИ ====================
# Копируем только файлы модулей для лучшего кэширования Docker слоев
# Если зависимости не изменились, этот слой будет переиспользован
COPY go.mod go.sum ./

# Загружаем зависимости (кэшируется отдельно от кода)
RUN go mod download

# Проверяем целостность модулей
RUN go mod verify

# ==================== СБОРКА ПРИЛОЖЕНИЯ ====================
# Копируем весь исходный код (изменения здесь не влияют на кэш зависимостей)
COPY . .

# Собираем статический бинарный файл
RUN CGO_ENABLED=0 \      # Отключаем CGO для статической сборки
    GOOS=linux \         # Целевая ОС
    GOARCH=amd64 \       # Целевая архитектура
    go build \
    -ldflags='-w -s -extldflags "-static"' \  # -w=no debug, -s=no symbol table, static linking
    -a \                 # Пересобрать все пакеты
    -installsuffix cgo \ # Суффикс для избежания конфликтов с CGO
    -o app \             # Имя выходного файла
    cmd/main.go          # Точка входа приложения

# ==================== ЭТАП 2: ФИНАЛЬНЫЙ ОБРАЗ ====================
# Используем scratch (пустой образ) для минимального размера
FROM scratch

# ==================== КОПИРОВАНИЕ СИСТЕМНЫХ ФАЙЛОВ ====================
# Копируем SSL сертификаты для HTTPS соединений
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Копируем данные временных зон
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo

# Копируем информацию о пользователях
COPY --from=builder /etc/passwd /etc/passwd

# ==================== КОПИРОВАНИЕ ПРИЛОЖЕНИЯ ====================
# Копируем скомпилированный бинарный файл
COPY --from=builder /build/app /app

# Создаем временную директорию с правильными правами
COPY --from=builder --chown=appuser:appuser /tmp /tmp

# ==================== НАСТРОЙКА БЕЗОПАСНОСТИ ====================
# Переключаемся на непривилегированного пользователя
# Приложение не будет работать от root
USER appuser

# ==================== НАСТРОЙКА СЕТИ ====================
# Документируем порт, который использует приложение
EXPOSE 8080

# ==================== ЗАПУСК ПРИЛОЖЕНИЯ ====================
# Указываем команду для запуска (не может быть переопределена)
ENTRYPOINT ["/app"]
```

### Оптимизированный Dockerfile с кэшированием

```dockerfile
# services/user-service/Dockerfile.optimized

FROM golang:1.21-alpine AS builder

# Кэшируем слои с зависимостями
RUN apk add --no-cache git ca-certificates tzdata && \
    adduser -D -g '' appuser

WORKDIR /build

# Отдельно копируем mod файлы для лучшего кэширования
COPY go.mod go.sum ./
RUN go mod download && go mod verify

# Копируем только необходимые файлы
COPY cmd/ cmd/
COPY internal/ internal/
COPY pkg/ pkg/

# Мультистейдж сборка с оптимизациями
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags='-w -s -extldflags "-static"' \
    -a -installsuffix cgo \
    -o app cmd/main.go

# Финальный образ на основе distroless для безопасности
FROM gcr.io/distroless/static:nonroot

COPY --from=builder /build/app /app

EXPOSE 8080

ENTRYPOINT ["/app"]
```

---

## Docker Compose для микросервисов

### Основной docker-compose.yml

```yaml
# docker-compose.yml
# Конфигурация для локальной разработки микросервисной архитектуры
version: '3.8'  # Версия Docker Compose формата

services:
  # ==================== БАЗЫ ДАННЫХ ====================
  postgres:
    # Используем официальный образ PostgreSQL на Alpine Linux
    image: postgres:15-alpine
    # Задаем имя контейнера для удобства обращения
    container_name: postgres_db
    
    # Переменные окружения для настройки PostgreSQL
    environment:
      POSTGRES_DB: microservices_db     # Имя базы данных по умолчанию
      POSTGRES_USER: postgres           # Пользователь-администратор
      POSTGRES_PASSWORD: password       # Пароль (для dev среды)
    
    # Проброс портов: внешний:внутренний
    ports:
      - "5432:5432"  # Стандартный порт PostgreSQL
    
    # Монтирование томов для постоянного хранения данных
    volumes:
      # Том для данных БД (персистентное хранилище)
      - postgres_data:/var/lib/postgresql/data
      # Скрипт инициализации БД (выполняется при первом запуске)
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init-db.sql
    
    # Подключение к пользовательской сети
    networks:
      - microservices_network
    
    # Проверка работоспособности сервиса
    healthcheck:
      # Команда для проверки доступности PostgreSQL
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 30s    # Интервал между проверками
      timeout: 10s     # Таймаут выполнения команды
      retries: 3       # Количество попыток перед признанием сбоя

  redis:
    # Используем официальный образ Redis на Alpine Linux
    image: redis:7-alpine
    # Имя контейнера для идентификации
    container_name: redis_cache
    
    # Проброс портов для подключения к Redis
    ports:
      - "6379:6379"  # Стандартный порт Redis
    
    # Том для персистентного хранения данных Redis
    volumes:
      - redis_data:/data  # Данные Redis сохраняются в именованном томе
    
    # Подключение к сети микросервисов
    networks:
      - microservices_network
    
    # Проверка работоспособности Redis
    healthcheck:
      # Команда ping для проверки доступности Redis
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s    # Проверка каждые 30 секунд
      timeout: 10s     # Таймаут команды
      retries: 3       # Количество попыток

  # ==================== MESSAGE QUEUE ====================
  rabbitmq:
    # Образ RabbitMQ с веб-интерфейсом управления
    image: rabbitmq:3.12-management-alpine
    # Имя контейнера для брокера сообщений
    container_name: rabbitmq_broker
    
    # Настройка учетных данных по умолчанию
    environment:
      RABBITMQ_DEFAULT_USER: admin      # Пользователь администратор
      RABBITMQ_DEFAULT_PASS: password   # Пароль (для dev среды)
    
    # Проброс портов для RabbitMQ
    ports:
      - "5672:5672"   # Порт для AMQP протокола
      - "15672:15672" # Порт для веб-интерфейса управления
    
    # Том для хранения данных RabbitMQ
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq  # Очереди и настройки
    
    # Подключение к сети микросервисов
    networks:
      - microservices_network
    
    # Проверка работоспособности RabbitMQ
    healthcheck:
      # Диагностическая команда RabbitMQ
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 30s    # Интервал проверки
      timeout: 10s     # Таймаут выполнения
      retries: 3       # Количество попыток перед сбоем

  # API Gateway
  api-gateway:
    build: 
      context: ./services/api-gateway
      dockerfile: Dockerfile
    container_name: api_gateway
    ports:
      - "8080:8080"
    environment:
      - PORT=8080
      - USER_SERVICE_URL=http://user-service:8081
      - ORDER_SERVICE_URL=http://order-service:8082
      - REDIS_URL=redis:6379
    depends_on:
      redis:
        condition: service_healthy
      user-service:
        condition: service_healthy
      order-service:
        condition: service_healthy
    networks:
      - microservices_network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # User Service
  user-service:
    build:
      context: ./services/user-service
      dockerfile: Dockerfile
    container_name: user_service
    ports:
      - "8081:8081"
    environment:
      - PORT=8081
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_NAME=microservices_db
      - DB_USER=postgres
      - DB_PASSWORD=password
      - REDIS_URL=redis:6379
      - RABBITMQ_URL=amqp://admin:password@rabbitmq:5672/
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
    networks:
      - microservices_network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8081/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Order Service
  order-service:
    build:
      context: ./services/order-service
      dockerfile: Dockerfile
    container_name: order_service
    ports:
      - "8082:8082"
    environment:
      - PORT=8082
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_NAME=microservices_db
      - DB_USER=postgres
      - DB_PASSWORD=password
      - RABBITMQ_URL=amqp://admin:password@rabbitmq:5672/
      - USER_SERVICE_URL=http://user-service:8081
    depends_on:
      postgres:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
      user-service:
        condition: service_healthy
    networks:
      - microservices_network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8082/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Мониторинг
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
    networks:
      - microservices_network

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana
      - ./monitoring/grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./monitoring/grafana/datasources:/etc/grafana/provisioning/datasources
    depends_on:
      - prometheus
    networks:
      - microservices_network

# Тома для постоянного хранения
volumes:
  postgres_data:
  redis_data:
  rabbitmq_data:
  prometheus_data:
  grafana_data:

# Сеть для коммуникации между сервисами
networks:
  microservices_network:
    driver: bridge
```

### Docker Compose для разработки

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  postgres:
    extends:
      file: docker-compose.yml
      service: postgres
    ports:
      - "5432:5432"

  redis:
    extends:
      file: docker-compose.yml
      service: redis
    ports:
      - "6379:6379"

  rabbitmq:
    extends:
      file: docker-compose.yml
      service: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"

  # В режиме разработки запускаем только инфраструктуру
  # Сервисы запускаем локально для отладки
```

### Docker Compose для продакшена

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  api-gateway:
    extends:
      file: docker-compose.yml
      service: api-gateway
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
        reservations:
          memory: 256M
          cpus: '0.25'
      restart_policy:
        condition: on-failure
        max_attempts: 3
    environment:
      - ENV=production
      - LOG_LEVEL=info

  user-service:
    extends:
      file: docker-compose.yml
      service: user-service
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 1G
          cpus: '1.0'
        reservations:
          memory: 512M
          cpus: '0.5'
    environment:
      - ENV=production
      - LOG_LEVEL=info

  order-service:
    extends:
      file: docker-compose.yml
      service: order-service
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 1G
          cpus: '1.0'
        reservations:
          memory: 512M
          cpus: '0.5'
    environment:
      - ENV=production
      - LOG_LEVEL=info
```

---

## Скрипты для управления

### Makefile для автоматизации

```makefile
# Makefile
# Автоматизация управления Docker Compose окружением

# ==================== КОНФИГУРАЦИОННЫЕ ПЕРЕМЕННЫЕ ====================
# Путь к основному файлу Docker Compose
DOCKER_COMPOSE_FILE = docker-compose.yml
# Файл для среды разработки
DOCKER_COMPOSE_DEV = docker-compose.dev.yml
# Файл для производственной среды
DOCKER_COMPOSE_PROD = docker-compose.prod.yml

# ==================== ОБЪЯВЛЕНИЕ PHONY ЦЕЛЕЙ ====================
# Указываем, что эти цели не создают файлы
.PHONY: help build start stop restart logs clean test

# ==================== СПРАВОЧНАЯ ИНФОРМАЦИЯ ====================
# Цель help - показывает доступные команды
help:
	@echo "Available commands:"
	@echo "  build      - Build all services"        # Сборка всех сервисов
	@echo "  start      - Start all services"        # Запуск всех сервисов
	@echo "  start-dev  - Start development environment"  # Запуск dev среды
	@echo "  start-prod - Start production environment"   # Запуск prod среды
	@echo "  stop       - Stop all services"         # Остановка всех сервисов
	@echo "  restart    - Restart all services"      # Перезапуск всех сервисов
	@echo "  logs       - Show logs"                 # Просмотр логов
	@echo "  clean      - Remove all containers and volumes"  # Очистка системы
	@echo "  test       - Run tests"                 # Запуск тестов

# ==================== СБОРКА ОБРАЗОВ ====================
# Сборка всех сервисов из docker-compose.yml
build:
	docker-compose -f $(DOCKER_COMPOSE_FILE) build

# Сборка конкретного сервиса (использование: make build-service SERVICE=user-service)
build-service:
	docker-compose -f $(DOCKER_COMPOSE_FILE) build $(SERVICE)

# ==================== УПРАВЛЕНИЕ ЖИЗНЕННЫМ ЦИКЛОМ ====================
# Запуск всех сервисов в фоновом режиме (-d = detached)
start:
	docker-compose -f $(DOCKER_COMPOSE_FILE) up -d

# Запуск в режиме разработки (только инфраструктура)
start-dev:
	docker-compose -f $(DOCKER_COMPOSE_DEV) up -d

# Запуск в продакшене (основной + prod конфигурация)
start-prod:
	docker-compose -f $(DOCKER_COMPOSE_FILE) -f $(DOCKER_COMPOSE_PROD) up -d

# Остановка всех сервисов и удаление контейнеров
stop:
	docker-compose -f $(DOCKER_COMPOSE_FILE) down

# Перезапуск всех сервисов (зависит от целей stop и start)
restart: stop start

# ==================== МОНИТОРИНГ И ОТЛАДКА ====================
# Просмотр логов всех сервисов в реальном времени (-f = follow)
logs:
	docker-compose -f $(DOCKER_COMPOSE_FILE) logs -f

# Просмотр логов конкретного сервиса
logs-service:
	docker-compose -f $(DOCKER_COMPOSE_FILE) logs -f $(SERVICE)

# ==================== ОЧИСТКА СИСТЕМЫ ====================
# Полная очистка контейнеров, томов и сетей
clean:
	# Остановка и удаление контейнеров, сетей и томов
	docker-compose -f $(DOCKER_COMPOSE_FILE) down -v --remove-orphans
	# Удаление неиспользуемых образов, контейнеров и сетей
	docker system prune -f
	# Удаление неиспользуемых томов
	docker volume prune -f

# ==================== ТЕСТИРОВАНИЕ ====================
# Запуск тестов в контейнерах
test:
	# Запуск Go тестов в user-service
	docker-compose -f $(DOCKER_COMPOSE_FILE) exec user-service go test ./...
	# Запуск Python тестов в order-service
	docker-compose -f $(DOCKER_COMPOSE_FILE) exec order-service python -m pytest

# ==================== УТИЛИТЫ ====================
# Проверка состояния всех сервисов
status:
	docker-compose -f $(DOCKER_COMPOSE_FILE) ps

# Получение shell доступа к контейнеру
shell:
	docker-compose -f $(DOCKER_COMPOSE_FILE) exec $(SERVICE) /bin/sh

# Масштабирование сервиса (использование: make scale SERVICE=user-service REPLICAS=3)
scale:
	docker-compose -f $(DOCKER_COMPOSE_FILE) up -d --scale $(SERVICE)=$(REPLICAS)
```

### Скрипт для локальной разработки

```bash
#!/bin/bash
# scripts/dev-setup.sh

set -e

echo "🚀 Setting up development environment..."

# Проверка наличия Docker
if ! command -v docker &> /dev/null; then
    echo "❌ Docker is not installed. Please install Docker first."
    exit 1
fi

# Проверка наличия Docker Compose
if ! command -v docker-compose &> /dev/null; then
    echo "❌ Docker Compose is not installed. Please install Docker Compose first."
    exit 1
fi

# Создание директорий для данных
echo "📁 Creating data directories..."
mkdir -p data/postgres
mkdir -p data/redis
mkdir -p data/rabbitmq
mkdir -p logs

# Запуск инфраструктуры
echo "🔧 Starting infrastructure services..."
docker-compose -f docker-compose.dev.yml up -d

# Ожидание готовности БД
echo "⏳ Waiting for database to be ready..."
while ! docker-compose -f docker-compose.dev.yml exec -T postgres pg_isready -U postgres &> /dev/null; do
    sleep 1
done

echo "📊 Running database migrations..."
# Здесь можно добавить миграции
# docker-compose -f docker-compose.dev.yml exec postgres psql -U postgres -d microservices_db -f /migrations/init.sql

echo "✅ Development environment is ready!"
echo ""
echo "Available services:"
echo "  PostgreSQL: localhost:5432"
echo "  Redis: localhost:6379"
echo "  RabbitMQ Management: http://localhost:15672 (admin/password)"
echo ""
echo "To start your services locally:"
echo "  cd services/user-service && go run cmd/main.go"
echo "  cd services/order-service && python main.py"
echo "  cd services/api-gateway && npm start"
```

### Скрипт для продакшена

```bash
#!/bin/bash
# scripts/deploy.sh

set -e

ENVIRONMENT=${1:-production}
VERSION=${2:-latest}

echo "🚀 Deploying to $ENVIRONMENT environment..."

# Проверка переменных окружения
if [[ "$ENVIRONMENT" == "production" ]]; then
    if [[ -z "$DB_PASSWORD" || -z "$REDIS_PASSWORD" ]]; then
        echo "❌ Missing required environment variables for production"
        exit 1
    fi
fi

# Сборка образов с тегами
echo "🔨 Building images..."
docker-compose build --parallel

# Тегирование образов
echo "🏷️ Tagging images..."
docker tag learn_user-service:latest user-service:$VERSION
docker tag learn_order-service:latest order-service:$VERSION
docker tag learn_api-gateway:latest api-gateway:$VERSION

# Пуш в registry (если нужно)
if [[ "$PUSH_TO_REGISTRY" == "true" ]]; then
    echo "📤 Pushing to registry..."
    docker push user-service:$VERSION
    docker push order-service:$VERSION
    docker push api-gateway:$VERSION
fi

# Остановка старых контейнеров
echo "🛑 Stopping old containers..."
docker-compose down

# Запуск новых контейнеров
echo "▶️ Starting new containers..."
if [[ "$ENVIRONMENT" == "production" ]]; then
    docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
else
    docker-compose up -d
fi

# Проверка здоровья сервисов
echo "🔍 Checking service health..."
sleep 30

services=("api-gateway" "user-service" "order-service")
for service in "${services[@]}"; do
    if docker-compose ps | grep -q "$service.*healthy"; then
        echo "✅ $service is healthy"
    else
        echo "❌ $service is not healthy"
        docker-compose logs $service
        exit 1
    fi
done

echo "✅ Deployment completed successfully!"
```

---

## Multi-stage сборки и оптимизация

### Оптимизированный Dockerfile для Python

```dockerfile
# services/order-service/Dockerfile

FROM python:3.11-slim as builder

# Устанавливаем зависимости для сборки
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Копируем requirements и устанавливаем зависимости
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Производственный образ
FROM python:3.11-slim

# Создаем пользователя
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Копируем установленные пакеты
COPY --from=builder /root/.local /home/appuser/.local

# Обновляем PATH
ENV PATH=/home/appuser/.local/bin:$PATH

WORKDIR /app

# Копируем код приложения
COPY --chown=appuser:appuser . .

# Переключаемся на непривилегированного пользователя
USER appuser

EXPOSE 8082

CMD ["python", "main.py"]
```

### .dockerignore для оптимизации

```dockerignore
# .dockerignore

# Git
.git
.gitignore

# Documentation
README.md
docs/

# Development files
.vscode/
.idea/
*.swp
*.swo
*~

# Logs
logs/
*.log

# OS generated files
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# Language specific
node_modules/
npm-debug.log
__pycache__/
*.pyc
vendor/
target/

# Test files
*_test.go
test/
tests/
*_test.py

# Build artifacts
build/
dist/
```

---

## Docker в CI/CD

### GitHub Actions с Docker

```yaml
# .github/workflows/docker-build.yml
name: Build and Push Docker Images

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: [user-service, order-service, api-gateway]
    
    permissions:
      contents: read
      packages: write

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${{ matrix.service }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-

    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: ./services/${{ matrix.service }}
        platforms: linux/amd64,linux/arm64
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  test:
    runs-on: ubuntu-latest
    needs: build-and-push
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Start services
      run: |
        docker-compose -f docker-compose.yml -f docker-compose.test.yml up -d
        sleep 30

    - name: Run integration tests
      run: |
        docker-compose exec -T api-gateway curl -f http://localhost:8080/health
        docker-compose exec -T user-service curl -f http://localhost:8081/health
        docker-compose exec -T order-service curl -f http://localhost:8082/health

    - name: Run API tests
      run: |
        docker-compose exec -T api-gateway npm test

    - name: Clean up
      if: always()
      run: docker-compose down -v
```

---

## Monitoring и логирование

### Health checks в Docker

```go
// services/user-service/internal/handlers/health.go
package handlers

import (
    "net/http"
    "encoding/json"
    "time"
)

type HealthHandler struct {
    db    Database
    redis Redis
}

type HealthResponse struct {
    Status    string            `json:"status"`
    Timestamp time.Time         `json:"timestamp"`
    Services  map[string]string `json:"services"`
}

func (h *HealthHandler) Health(w http.ResponseWriter, r *http.Request) {
    response := HealthResponse{
        Status:    "ok",
        Timestamp: time.Now(),
        Services:  make(map[string]string),
    }

    // Проверка БД
    if err := h.db.Ping(); err != nil {
        response.Status = "error"
        response.Services["database"] = "unhealthy: " + err.Error()
    } else {
        response.Services["database"] = "healthy"
    }

    // Проверка Redis
    if err := h.redis.Ping(); err != nil {
        response.Status = "error"
        response.Services["redis"] = "unhealthy: " + err.Error()
    } else {
        response.Services["redis"] = "healthy"
    }

    statusCode := http.StatusOK
    if response.Status == "error" {
        statusCode = http.StatusServiceUnavailable
    }

    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(statusCode)
    json.NewEncoder(w).Encode(response)
}
```

### Централизованное логирование

```yaml
# logging/docker-compose.logging.yml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.10.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    networks:
      - microservices_network

  logstash:
    image: docker.elastic.co/logstash/logstash:8.10.0
    container_name: logstash
    volumes:
      - ./logging/logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5000:5000"
    depends_on:
      - elasticsearch
    networks:
      - microservices_network

  kibana:
    image: docker.elastic.co/kibana/kibana:8.10.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch
    networks:
      - microservices_network

volumes:
  elasticsearch_data:
```

---

## Лучшие практики Docker

### 1. Безопасность

```dockerfile
# Используйте конкретные версии
FROM golang:1.21.3-alpine3.18

# Не запускайте от root
RUN adduser -D -g '' appuser
USER appuser

# Минимизируйте attack surface
FROM scratch
COPY --from=builder /app /app

# Используйте multi-stage builds
FROM builder AS final
```

### 2. Производительность

```dockerfile
# Оптимизация слоев
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
    && rm -rf /var/lib/apt/lists/*

# Кэширование зависимостей
COPY go.mod go.sum ./
RUN go mod download
COPY . .
```

### 3. Размер образов

```bash
# Анализ размера слоев
docker history image_name

# Очистка
docker system prune -a
docker image prune -a
```

---

## 🔗 Связанные темы

### 🚀 DevOps и инфраструктура
- **[CI/CD Pipeline](cicd-devops.md)** - Непрерывная интеграция и развертывание
- **[Linux Deployment](linux-deployment.md)** - Развертывание на Linux серверах
- **[Deployment & Monitoring](deployment-monitoring.md)** - Мониторинг контейнеризованных приложений

### ⚙️ Технические навыки
- **[Databases](../technical-skills/databases.md)** - Контейнеризация баз данных
- **[Security](../technical-skills/security.md)** - Безопасность контейнеров
- **[Testing](../technical-skills/testing.md)** - Тестирование в контейнерах

### 🏛️ Архитектура
- **[Microservices Architecture](../architecture/microservices-architecture.md)** - Контейнеризация микросервисов

---

**Путь обучения**: Docker Containerization → [CI/CD Pipeline](cicd-devops.md) → [Linux Deployment](linux-deployment.md)  
**Сложность**: ⭐⭐⭐ (3/5)  
**Время изучения**: 2-3 недели  
**Практика**: Контейнеризация всех сервисов проекта, настройка локальной и продакшн среды 