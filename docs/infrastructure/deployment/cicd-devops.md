# CI/CD и DevOps

**CI/CD** (Continuous Integration/Continuous Deployment) — это практики автоматизации сборки, тестирования и развертывания приложений.

**DevOps** — это методология, объединяющая разработку (Development) и операции (Operations) для ускорения доставки программного обеспечения.

## Основные компоненты CI/CD

```
Код → CI → Тесты → Сборка → CD → Развертывание → Мониторинг
 ↓      ↓      ↓       ↓       ↓         ↓            ↓
Git → GitHub → Tests → Docker → K8s → Production → Metrics
```

## Continuous Integration (CI)

**CI** автоматически собирает и тестирует код при каждом коммите.

### GitHub Actions

**GitHub Actions** — это платформа для автоматизации CI/CD прямо в GitHub.

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Линтинг и статический анализ
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run ESLint
        run: npm run lint
      
      - name: Run Prettier
        run: npm run format:check

  # Тестирование Go сервиса
  test-user-service:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Go
        uses: actions/setup-go@v4
        with:
          go-version: '1.21'
      
      - name: Cache Go modules
        uses: actions/cache@v3
        with:
          path: ~/go/pkg/mod
          key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
          restore-keys: |
            ${{ runner.os }}-go-
      
      - name: Download dependencies
        run: go mod download
        working-directory: ./services/user-service
      
      - name: Run tests
        run: go test -v -race -coverprofile=coverage.out ./...
        working-directory: ./services/user-service
        env:
          DATABASE_URL: postgres://postgres:postgres@localhost:5432/testdb?sslmode=disable
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./services/user-service/coverage.out
          flags: user-service

  # Тестирование Python сервиса
  test-order-service:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Cache pip
        uses: actions/cache@v3
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest pytest-cov pytest-asyncio
        working-directory: ./services/order-service
      
      - name: Run tests
        run: pytest --cov=./ --cov-report=xml
        working-directory: ./services/order-service
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./services/order-service/coverage.xml
          flags: order-service

  # E2E тесты
  e2e-tests:
    runs-on: ubuntu-latest
    needs: [test-user-service, test-order-service]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Start services
        run: |
          docker-compose -f docker-compose.test.yml up -d
          sleep 30  # Ждем запуска сервисов
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
      
      - name: Install Playwright
        run: |
          npm install @playwright/test
          npx playwright install
      
      - name: Run E2E tests
        run: npx playwright test
        env:
          BASE_URL: http://localhost:3000
      
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
      
      - name: Stop services
        run: docker-compose -f docker-compose.test.yml down

  # Сборка Docker образов
  build:
    runs-on: ubuntu-latest
    needs: [lint, test-user-service, test-order-service, e2e-tests]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    strategy:
      matrix:
        service: [user-service, order-service, api-gateway, frontend]
    
    steps:
      - uses: actions/checkout@v4
      
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
            type=raw,value=latest,enable={{is_default_branch}}
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: ./services/${{ matrix.service }}
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # Деплой в staging
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build]
    environment: staging
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to staging
        run: |
          # Деплой в staging среду
          echo "Deploying to staging..."
          # Здесь может быть скрипт для деплоя
```

### Многоэтапный pipeline

```yaml
# .github/workflows/advanced-ci.yml
name: Advanced CI/CD Pipeline

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

jobs:
  # Параллельная сборка сервисов
  build-services:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: 
          - name: user-service
            dockerfile: Dockerfile
            context: ./services/user-service
          - name: order-service
            dockerfile: Dockerfile
            context: ./services/order-service
          - name: api-gateway
            dockerfile: Dockerfile
            context: ./services/api-gateway
          - name: frontend
            dockerfile: Dockerfile
            context: ./frontend
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build ${{ matrix.service.name }}
        run: |
          docker build -t ${{ matrix.service.name }}:${{ github.sha }} \
            -f ${{ matrix.service.context }}/${{ matrix.service.dockerfile }} \
            ${{ matrix.service.context }}
      
      - name: Run security scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: '${{ matrix.service.name }}:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload security scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

  # Тестирование производительности
  performance-tests:
    runs-on: ubuntu-latest
    needs: [build-services]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Start services
        run: docker-compose -f docker-compose.test.yml up -d
      
      - name: Wait for services
        run: |
          timeout 300 bash -c 'until curl -f http://localhost:3000/health; do sleep 5; done'
      
      - name: Run load tests
        run: |
          docker run --rm -i --network host \
            grafana/k6:latest run --vus 10 --duration 30s - < tests/load/user-flow.js
      
      - name: Stop services
        run: docker-compose -f docker-compose.test.yml down

  # Условный деплой
  deploy:
    runs-on: ubuntu-latest
    needs: [build-services, performance-tests]
    environment: ${{ github.event.inputs.environment }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to ${{ github.event.inputs.environment }}
        run: |
          echo "Deploying to ${{ github.event.inputs.environment }}"
          # Скрипт деплоя
```

## Docker

**Docker** — это платформа для контейнеризации приложений.

### Многоэтапная сборка

```dockerfile
# services/user-service/Dockerfile
# Этап сборки
FROM golang:1.21-alpine AS builder

WORKDIR /app

# Копируем модули для кэширования зависимостей
COPY go.mod go.sum ./
RUN go mod download

# Копируем исходный код
COPY . .

# Сборка приложения
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main ./cmd/main.go

# Этап выполнения
FROM alpine:latest

# Установка необходимых пакетов
RUN apk --no-cache add ca-certificates tzdata

WORKDIR /root/

# Копируем бинарник из этапа сборки
COPY --from=builder /app/main .

# Создаем непривилегированного пользователя
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup

# Переключаемся на непривилегированного пользователя
USER appuser

# Указываем порт
EXPOSE 8080

# Команда запуска
CMD ["./main"]
```

```dockerfile
# services/order-service/Dockerfile
FROM python:3.11-slim

# Устанавливаем системные зависимости
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        gcc \
        libc6-dev \
        libpq-dev && \
    rm -rf /var/lib/apt/lists/*

# Создаем рабочую директорию
WORKDIR /app

# Копируем requirements и устанавливаем зависимости
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Копируем исходный код
COPY . .

# Создаем непривилегированного пользователя
RUN useradd --create-home --shell /bin/bash appuser && \
    chown -R appuser:appuser /app

USER appuser

# Указываем порт
EXPOSE 8000

# Команда запуска
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Docker Compose для разработки

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  # Базы данных
  postgres-user:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: userdb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_user_data:/var/lib/postgresql/data
      - ./services/user-service/database/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d userdb"]
      interval: 5s
      timeout: 5s
      retries: 5

  postgres-order:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: orderdb
      POSTGRES_USER: order
      POSTGRES_PASSWORD: password
    ports:
      - "5433:5432"
    volumes:
      - postgres_order_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U order -d orderdb"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5

  # Сервисы
  user-service:
    build:
      context: ./services/user-service
      dockerfile: Dockerfile.dev
    ports:
      - "8081:8080"
    environment:
      - DATABASE_URL=postgres://user:password@postgres-user:5432/userdb?sslmode=disable
      - REDIS_URL=redis://redis:6379
    depends_on:
      postgres-user:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./services/user-service:/app
    develop:
      watch:
        - action: rebuild
          path: ./services/user-service/go.mod
        - action: rebuild
          path: ./services/user-service/go.sum
        - action: sync
          path: ./services/user-service
          target: /app

  order-service:
    build:
      context: ./services/order-service
      dockerfile: Dockerfile.dev
    ports:
      - "8082:8000"
    environment:
      - DATABASE_URL=postgresql://order:password@postgres-order:5432/orderdb
      - USER_SERVICE_URL=http://user-service:8080
    depends_on:
      postgres-order:
        condition: service_healthy
      user-service:
        condition: service_started
    volumes:
      - ./services/order-service:/app
    develop:
      watch:
        - action: rebuild
          path: ./services/order-service/requirements.txt
        - action: sync
          path: ./services/order-service
          target: /app

  api-gateway:
    build:
      context: ./services/api-gateway
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    environment:
      - USER_SERVICE_URL=http://user-service:8080
      - ORDER_SERVICE_URL=http://order-service:8000
      - REDIS_URL=redis://redis:6379
    depends_on:
      - user-service
      - order-service
    volumes:
      - ./services/api-gateway:/app
    develop:
      watch:
        - action: rebuild
          path: ./services/api-gateway/package.json
        - action: sync
          path: ./services/api-gateway
          target: /app

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    ports:
      - "3001:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:3000
    depends_on:
      - api-gateway
    volumes:
      - ./frontend:/app
    develop:
      watch:
        - action: rebuild
          path: ./frontend/package.json
        - action: sync
          path: ./frontend
          target: /app

volumes:
  postgres_user_data:
  postgres_order_data:
  redis_data:
```

## Kubernetes

**Kubernetes** — это система оркестрации контейнеров для автоматизации развертывания, масштабирования и управления.

### Основные объекты Kubernetes

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: microservices-app
  labels:
    app: microservices
    environment: production

---
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: microservices-app
data:
  DATABASE_HOST: "postgres-service"
  DATABASE_PORT: "5432"
  REDIS_HOST: "redis-service"
  REDIS_PORT: "6379"
  LOG_LEVEL: "info"

---
# k8s/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: microservices-app
type: Opaque
data:
  DATABASE_PASSWORD: cGFzc3dvcmQ=  # base64 encoded "password"
  JWT_SECRET: c2VjcmV0LWtleQ==      # base64 encoded "secret-key"
```

### Deployment для User Service

```yaml
# k8s/user-service.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: microservices-app
  labels:
    app: user-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
      - name: user-service
        image: ghcr.io/username/microservices-example/user-service:latest
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: DATABASE_URL
        - name: REDIS_URL
          value: "redis://$(REDIS_HOST):$(REDIS_PORT)"
        envFrom:
        - configMapRef:
            name: app-config
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        securityContext:
          runAsNonRoot: true
          runAsUser: 1001
          readOnlyRootFilesystem: true
          allowPrivilegeEscalation: false

---
apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: microservices-app
spec:
  selector:
    app: user-service
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

### Horizontal Pod Autoscaler

```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: user-service-hpa
  namespace: microservices-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: user-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### Ingress для внешнего доступа

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: microservices-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - api.example.com
    secretName: api-tls
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /api/users
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
      - path: /api/orders
        pathType: Prefix
        backend:
          service:
            name: order-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-gateway
            port:
              number: 80
```

## Helm Charts

**Helm** — это менеджер пакетов для Kubernetes.

```yaml
# helm/microservices-app/Chart.yaml
apiVersion: v2
name: microservices-app
description: A Helm chart for microservices application
type: application
version: 0.1.0
appVersion: "1.0.0"

dependencies:
- name: postgresql
  version: "12.1.9"
  repository: https://charts.bitnami.com/bitnami
  condition: postgresql.enabled
- name: redis
  version: "17.3.7"
  repository: https://charts.bitnami.com/bitnami
  condition: redis.enabled
```

```yaml
# helm/microservices-app/values.yaml
# Global settings
global:
  imageRegistry: ghcr.io/username/microservices-example
  imagePullSecrets: []

# Application settings
userService:
  enabled: true
  replicaCount: 3
  image:
    repository: user-service
    tag: latest
    pullPolicy: IfNotPresent
  service:
    type: ClusterIP
    port: 80
  resources:
    requests:
      memory: "64Mi"
      cpu: "250m"
    limits:
      memory: "128Mi"
      cpu: "500m"
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 10
    targetCPUUtilizationPercentage: 70

orderService:
  enabled: true
  replicaCount: 2
  image:
    repository: order-service
    tag: latest
    pullPolicy: IfNotPresent
  service:
    type: ClusterIP
    port: 80
  resources:
    requests:
      memory: "128Mi"
      cpu: "250m"
    limits:
      memory: "256Mi"
      cpu: "500m"

# Database settings
postgresql:
  enabled: true
  auth:
    postgresPassword: "password"
    database: "microservices"
  primary:
    persistence:
      enabled: true
      size: 8Gi

redis:
  enabled: true
  auth:
    enabled: false
  master:
    persistence:
      enabled: true
      size: 2Gi

# Ingress settings
ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  hosts:
    - host: api.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: api-tls
      hosts:
        - api.example.com
```

## Continuous Deployment

### ArgoCD для GitOps

```yaml
# argocd/application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: microservices-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/username/microservices-example
    targetRevision: HEAD
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: microservices-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

### Environments и Promotion

```yaml
# .github/workflows/cd.yml
name: CD Pipeline

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        required: true
        type: choice
        options: [staging, production]

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to staging
        run: |
          kubectl config use-context staging
          helm upgrade --install microservices-app ./helm/microservices-app \
            --namespace microservices-staging \
            --create-namespace \
            --values ./helm/microservices-app/values-staging.yaml \
            --set global.imageTag=${{ github.sha }}
  
  run-smoke-tests:
    runs-on: ubuntu-latest
    needs: [deploy-staging]
    steps:
      - uses: actions/checkout@v4
      
      - name: Run smoke tests
        run: |
          # Smoke tests против staging
          curl -f https://staging.api.example.com/health || exit 1
          # Более сложные тесты...
  
  deploy-production:
    runs-on: ubuntu-latest
    needs: [run-smoke-tests]
    environment: production
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to production
        run: |
          kubectl config use-context production
          helm upgrade --install microservices-app ./helm/microservices-app \
            --namespace microservices-production \
            --create-namespace \
            --values ./helm/microservices-app/values-production.yaml \
            --set global.imageTag=${{ github.sha }}
```

## Выбор инструментов

### GitHub Actions vs Jenkins vs GitLab CI

| Критерий | GitHub Actions | Jenkins | GitLab CI |
|----------|----------------|---------|-----------|
| **Простота** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| **Гибкость** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Экосистема** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Стоимость** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |

### Docker vs Podman

| Критерий | Docker | Podman |
|----------|---------|---------|
| **Простота** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Безопасность** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Экосистема** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Производительность** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

### Kubernetes vs Docker Swarm

| Критерий | Kubernetes | Docker Swarm |
|----------|-------------|--------------|
| **Масштабируемость** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Простота** | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Функциональность** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Сообщество** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |

## Лучшие практики

### CI/CD Pipeline
1. **Быстрая обратная связь** - минимизируйте время выполнения
2. **Fail fast** - останавливайтесь при первой ошибке
3. **Параллелизация** - выполняйте независимые задачи параллельно
4. **Кэширование** - используйте кэш для ускорения сборки
5. **Безопасность** - сканируйте уязвимости и секреты

### Docker
1. **Многоэтапная сборка** - уменьшайте размер образов
2. **Непривилегированные пользователи** - не запускайте как root
3. **Минимальные базовые образы** - используйте alpine или scratch
4. **Сканирование безопасности** - проверяйте уязвимости
5. **Метки и теги** - используйте семантическое версионирование

### Kubernetes
1. **Resource limits** - устанавливайте ограничения ресурсов
2. **Health checks** - настраивайте liveness/readiness пробы
3. **Secrets management** - не храните секреты в коде
4. **Namespace isolation** - разделяйте окружения
5. **Monitoring** - следите за состоянием кластера

## Заключение

Правильно настроенный CI/CD pipeline обеспечивает:
- **Быструю доставку** новых функций
- **Высокое качество** кода
- **Надежные развертывания**
- **Эффективную команду разработки**

Выбор инструментов зависит от размера команды, требований к безопасности и бюджета.

## См. также
- [Тестирование](./testing.md)
- [Мониторинг](./deployment-monitoring.md)
- [Безопасность](./security.md) 