# 🤖 Automation Scripts: Автоматизация развертывания

## Описание

Комплексные скрипты автоматизации для CI/CD pipeline, развертывания микросервисов и управления инфраструктурой с использованием GitLab CI, GitHub Actions и custom bash скриптов.

## CI/CD Pipeline структура

### GitLab CI конфигурация

```yaml
# .gitlab-ci.yml
# Полный CI/CD pipeline для микросервисной архитектуры

# ==================== СТАДИИ PIPELINE ====================
stages:
  - validate        # Валидация кода и конфигураций
  - test           # Тестирование
  - build          # Сборка образов
  - security       # Проверки безопасности
  - deploy-staging # Развертывание на staging
  - integration    # Интеграционные тесты
  - deploy-prod    # Развертывание на production
  - monitoring     # Проверка мониторинга

# ==================== ПЕРЕМЕННЫЕ ====================
variables:
  DOCKER_REGISTRY: "registry.mycompany.com"
  PROJECT_NAME: "microservices"
  STAGING_NAMESPACE: "microservices-staging"
  PROD_NAMESPACE: "microservices"
  KUBECTL_VERSION: "1.28.0"

# ==================== ВАЛИДАЦИЯ КОДА ====================
validate:syntax:
  stage: validate
  image: alpine:latest
  before_script:
    - apk add --no-cache bash yamllint
  script:
    - echo "🔍 Проверка синтаксиса YAML файлов..."
    - find . -name "*.yml" -o -name "*.yaml" | xargs yamllint
    - echo "🔍 Проверка shell скриптов..."
    - find . -name "*.sh" | xargs bash -n
  rules:
    - changes:
        - "**/*.yml"
        - "**/*.yaml"
        - "**/*.sh"

validate:terraform:
  stage: validate
  image: hashicorp/terraform:latest
  script:
    - echo "🔍 Валидация Terraform конфигурации..."
    - cd infrastructure/terraform
    - terraform init -backend=false
    - terraform validate
    - terraform fmt -check
  rules:
    - changes:
        - "infrastructure/terraform/**/*"

# ==================== ТЕСТИРОВАНИЕ ====================
test:unit:
  stage: test
  image: openjdk:17-jdk
  services:
    - postgres:13
    - redis:6-alpine
  variables:
    POSTGRES_DB: test_db
    POSTGRES_USER: test_user
    POSTGRES_PASSWORD: test_password
  before_script:
    - echo "🧪 Настройка тестового окружения..."
  script:
    - echo "🧪 Запуск unit тестов для всех сервисов..."
    - cd services/user-service && ./mvnw test
    - cd ../order-service && ./mvnw test
    - cd ../api-gateway && ./mvnw test
  artifacts:
    reports:
      junit:
        - "**/target/surefire-reports/TEST-*.xml"
    paths:
      - "**/target/jacoco.exec"
    expire_in: 1 week
  coverage: '/Total.*?([0-9]{1,3})%/'

test:integration:
  stage: test
  image: maven:3.8-openjdk-17
  services:
    - docker:dind
  variables:
    DOCKER_HOST: tcp://docker:2376
    DOCKER_TLS_CERTDIR: "/certs"
  script:
    - echo "🔗 Запуск интеграционных тестов..."
    - cd tests/integration
    - mvn test -Dtest.profile=ci

# ==================== СБОРКА ОБРАЗОВ ====================
build:user-service:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  variables:
    IMAGE_NAME: "${DOCKER_REGISTRY}/${PROJECT_NAME}/user-service"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $DOCKER_REGISTRY
  script:
    - echo "🏗️ Сборка образа User Service..."
    - cd services/user-service
    - |
      docker build \
        --build-arg VERSION=$CI_COMMIT_SHA \
        --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
        -t $IMAGE_NAME:$CI_COMMIT_SHA \
        -t $IMAGE_NAME:latest .
    - docker push $IMAGE_NAME:$CI_COMMIT_SHA
    - docker push $IMAGE_NAME:latest
  rules:
    - changes:
        - "services/user-service/**/*"
    - if: '$CI_COMMIT_BRANCH == "main"'

build:order-service:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  variables:
    IMAGE_NAME: "${DOCKER_REGISTRY}/${PROJECT_NAME}/order-service"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $DOCKER_REGISTRY
  script:
    - echo "🏗️ Сборка образа Order Service..."
    - cd services/order-service
    - |
      docker build \
        --build-arg VERSION=$CI_COMMIT_SHA \
        --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
        -t $IMAGE_NAME:$CI_COMMIT_SHA \
        -t $IMAGE_NAME:latest .
    - docker push $IMAGE_NAME:$CI_COMMIT_SHA
    - docker push $IMAGE_NAME:latest
  rules:
    - changes:
        - "services/order-service/**/*"
    - if: '$CI_COMMIT_BRANCH == "main"'

# ==================== ПРОВЕРКИ БЕЗОПАСНОСТИ ====================
security:trivy:
  stage: security
  image: aquasec/trivy:latest
  script:
    - echo "🔒 Сканирование уязвимостей в образах..."
    - trivy image --exit-code 0 --format template --template "@contrib/sarif.tpl" -o trivy-results.sarif $DOCKER_REGISTRY/$PROJECT_NAME/user-service:$CI_COMMIT_SHA
    - trivy image --exit-code 1 --severity HIGH,CRITICAL $DOCKER_REGISTRY/$PROJECT_NAME/user-service:$CI_COMMIT_SHA
  artifacts:
    reports:
      sast: trivy-results.sarif
  allow_failure: false

# ==================== РАЗВЕРТЫВАНИЕ STAGING ====================
deploy:staging:
  stage: deploy-staging
  image: bitnami/kubectl:$KUBECTL_VERSION
  environment:
    name: staging
    url: https://staging-api.mycompany.com
  before_script:
    - echo "🚀 Подготовка к развертыванию на staging..."
    - kubectl config use-context staging-cluster
  script:
    - echo "🚀 Развертывание на staging..."
    - |
      # Обновление образов в манифестах
      sed -i "s|:latest|:$CI_COMMIT_SHA|g" k8s/*.yaml
      
      # Применение манифестов
      kubectl apply -f k8s/ -n $STAGING_NAMESPACE
      
      # Ожидание готовности
      kubectl rollout status deployment/user-service -n $STAGING_NAMESPACE --timeout=300s
      kubectl rollout status deployment/order-service -n $STAGING_NAMESPACE --timeout=300s
      kubectl rollout status deployment/api-gateway -n $STAGING_NAMESPACE --timeout=300s
      
      echo "✅ Развертывание на staging завершено"
  rules:
    - if: '$CI_COMMIT_BRANCH == "develop"'

# ==================== РАЗВЕРТЫВАНИЕ PRODUCTION ====================
deploy:production:
  stage: deploy-prod
  image: bitnami/kubectl:$KUBECTL_VERSION
  environment:
    name: production
    url: https://api.mycompany.com
  before_script:
    - kubectl config use-context production-cluster
  script:
    - echo "🚀 Развертывание на production..."
    - ./scripts/deploy-production.sh $CI_COMMIT_SHA
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
  when: manual          # Требует ручного подтверждения

# ==================== МОНИТОРИНГ ПОСЛЕ РАЗВЕРТЫВАНИЯ ====================
post-deploy:health-check:
  stage: monitoring
  image: curlimages/curl:latest
  script:
    - echo "🏥 Проверка работоспособности сервисов..."
    - ./scripts/health-check.sh
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```

## Скрипты развертывания

### Основной скрипт развертывания

```bash
#!/bin/bash
# scripts/deploy-production.sh
# Скрипт автоматического развертывания на production

set -euo pipefail

# ==================== ПАРАМЕТРЫ ====================
VERSION=${1:-latest}
NAMESPACE="microservices"
DRY_RUN=${2:-false}
ROLLBACK_ON_FAILURE=${3:-true}

# Цвета для вывода
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

# ==================== ФУНКЦИИ ====================
log() {
    echo -e "${BLUE}[$(date +'%Y-%m-%d %H:%M:%S')]${NC} $1"
}

success() {
    echo -e "${GREEN}[SUCCESS]${NC} $1"
}

warning() {
    echo -e "${YELLOW}[WARNING]${NC} $1"
}

error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# Функция проверки работоспособности
health_check() {
    local service=$1
    local url=$2
    local max_attempts=30
    local attempt=1
    
    log "Проверка работоспособности $service..."
    
    while [ $attempt -le $max_attempts ]; do
        if curl -f -s "$url/health" > /dev/null; then
            success "$service готов к работе"
            return 0
        fi
        
        log "Попытка $attempt/$max_attempts для $service..."
        sleep 10
        ((attempt++))
    done
    
    error "$service не готов к работе после $max_attempts попыток"
    return 1
}

# Функция отката
rollback() {
    local deployment=$1
    warning "Откат развертывания $deployment..."
    kubectl rollout undo deployment/$deployment -n $NAMESPACE
    kubectl rollout status deployment/$deployment -n $NAMESPACE --timeout=300s
}

# ==================== ОСНОВНОЙ ПРОЦЕСС ====================
log "🚀 Начало развертывания версии $VERSION на production"

# Проверка подключения к кластеру
if ! kubectl cluster-info &> /dev/null; then
    error "Нет подключения к Kubernetes кластеру"
    exit 1
fi

# Проверка namespace
if ! kubectl get namespace $NAMESPACE &> /dev/null; then
    error "Namespace $NAMESPACE не существует"
    exit 1
fi

# Сохранение текущих версий для возможного отката
log "📝 Сохранение текущих версий для отката..."
USER_SERVICE_CURRENT=$(kubectl get deployment user-service -n $NAMESPACE -o jsonpath='{.spec.template.spec.containers[0].image}')
ORDER_SERVICE_CURRENT=$(kubectl get deployment order-service -n $NAMESPACE -o jsonpath='{.spec.template.spec.containers[0].image}')
API_GATEWAY_CURRENT=$(kubectl get deployment api-gateway -n $NAMESPACE -o jsonpath='{.spec.template.spec.containers[0].image}')

log "Текущие версии:"
log "  User Service: $USER_SERVICE_CURRENT"
log "  Order Service: $ORDER_SERVICE_CURRENT"
log "  API Gateway: $API_GATEWAY_CURRENT"

# Обновление образов в манифестах
log "📝 Обновление манифестов с версией $VERSION..."
if [ "$DRY_RUN" = "false" ]; then
    sed -i.bak "s|:latest|:$VERSION|g" k8s/*.yaml
    sed -i.bak "s|:v[0-9]\+\.[0-9]\+\.[0-9]\+|:$VERSION|g" k8s/*.yaml
fi

# Массив сервисов для развертывания
SERVICES=("user-service" "order-service" "api-gateway")

# Развертывание каждого сервиса
for service in "${SERVICES[@]}"; do
    log "🔄 Развертывание $service..."
    
    if [ "$DRY_RUN" = "false" ]; then
        # Применение манифеста
        kubectl apply -f "k8s/$(printf '%02d' $(($(echo "$service" | sed 's/[^0-9]*//g') + 3)))-$service.yaml" -n $NAMESPACE
        
        # Ожидание завершения развертывания
        if kubectl rollout status deployment/$service -n $NAMESPACE --timeout=600s; then
            success "Развертывание $service завершено успешно"
        else
            error "Развертывание $service завершилось с ошибкой"
            
            if [ "$ROLLBACK_ON_FAILURE" = "true" ]; then
                rollback $service
            fi
            exit 1
        fi
    else
        log "DRY RUN: kubectl apply -f k8s/*-$service.yaml -n $NAMESPACE"
    fi
done

# Проверка работоспособности всех сервисов
if [ "$DRY_RUN" = "false" ]; then
    log "🏥 Проверка работоспособности всех сервисов..."
    
    # Получение внешнего IP Load Balancer
    EXTERNAL_IP=$(kubectl get service api-gateway -n $NAMESPACE -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    
    if [ -z "$EXTERNAL_IP" ]; then
        warning "Внешний IP не найден, используем port-forward для проверки"
        kubectl port-forward service/api-gateway 8080:8080 -n $NAMESPACE &
        PORT_FORWARD_PID=$!
        sleep 5
        BASE_URL="http://localhost:8080"
    else
        BASE_URL="http://$EXTERNAL_IP:8080"
    fi
    
    # Проверка каждого сервиса
    HEALTH_CHECK_FAILED=false
    
    for service in "${SERVICES[@]}"; do
        service_port=""
        case $service in
            "user-service") service_port="8081" ;;
            "order-service") service_port="8082" ;;
            "api-gateway") service_port="8080" ;;
        esac
        
        if ! health_check $service "$BASE_URL"; then
            HEALTH_CHECK_FAILED=true
        fi
    done
    
    # Очистка port-forward если использовался
    if [ ! -z "${PORT_FORWARD_PID:-}" ]; then
        kill $PORT_FORWARD_PID
    fi
    
    if [ "$HEALTH_CHECK_FAILED" = "true" ]; then
        error "Проверка работоспособности завершилась с ошибками"
        if [ "$ROLLBACK_ON_FAILURE" = "true" ]; then
            for service in "${SERVICES[@]}"; do
                rollback $service
            done
        fi
        exit 1
    fi
fi

# Восстановление оригинальных манифестов
if [ "$DRY_RUN" = "false" ]; then
    log "🔄 Восстановление оригинальных манифестов..."
    for file in k8s/*.yaml.bak; do
        if [ -f "$file" ]; then
            mv "$file" "${file%.bak}"
        fi
    done
fi

# Проверка метрик и алертов
log "📊 Проверка метрик и алертов..."
if [ "$DRY_RUN" = "false" ]; then
    ./scripts/check-metrics.sh
fi

success "🎉 Развертывание версии $VERSION завершено успешно!"

# Вывод информации о развертывании
log "📋 Информация о развертывании:"
if [ "$DRY_RUN" = "false" ]; then
    kubectl get pods -n $NAMESPACE -o wide
    kubectl get services -n $NAMESPACE
    kubectl get ingress -n $NAMESPACE
fi
```

### Скрипт проверки работоспособности

```bash
#!/bin/bash
# scripts/health-check.sh
# Комплексная проверка работоспособности микросервисов

set -euo pipefail

# ==================== КОНФИГУРАЦИЯ ====================
NAMESPACE="microservices"
TIMEOUT=300
MAX_RETRIES=5

# Цвета для вывода
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

# ==================== ФУНКЦИИ ПРОВЕРКИ ====================
log() {
    echo -e "${BLUE}[$(date +'%Y-%m-%d %H:%M:%S')]${NC} $1"
}

success() {
    echo -e "${GREEN}✅${NC} $1"
}

error() {
    echo -e "${RED}❌${NC} $1"
}

# Проверка готовности подов
check_pods() {
    log "Проверка состояния подов..."
    
    local failed_pods=()
    
    while IFS= read -r line; do
        pod_name=$(echo "$line" | awk '{print $1}')
        ready=$(echo "$line" | awk '{print $2}')
        status=$(echo "$line" | awk '{print $3}')
        
        if [[ "$ready" != *"/"* ]] || [[ "$status" != "Running" ]]; then
            failed_pods+=("$pod_name")
        fi
    done < <(kubectl get pods -n $NAMESPACE --no-headers)
    
    if [ ${#failed_pods[@]} -eq 0 ]; then
        success "Все поды готовы к работе"
        return 0
    else
        error "Проблемные поды: ${failed_pods[*]}"
        return 1
    fi
}

# Проверка endpoints сервисов
check_endpoints() {
    log "Проверка endpoints сервисов..."
    
    local services=("user-service" "order-service" "api-gateway")
    local failed_services=()
    
    for service in "${services[@]}"; do
        local endpoints=$(kubectl get endpoints $service -n $NAMESPACE -o jsonpath='{.subsets[*].addresses[*].ip}' 2>/dev/null || echo "")
        
        if [ -z "$endpoints" ]; then
            failed_services+=("$service")
        else
            success "$service имеет активные endpoints: $endpoints"
        fi
    done
    
    if [ ${#failed_services[@]} -eq 0 ]; then
        return 0
    else
        error "Сервисы без endpoints: ${failed_services[*]}"
        return 1
    fi
}

# HTTP проверка сервисов
check_http_health() {
    log "HTTP проверка работоспособности сервисов..."
    
    local base_url=""
    
    # Попытка получить внешний IP
    local external_ip=$(kubectl get service api-gateway -n $NAMESPACE -o jsonpath='{.status.loadBalancer.ingress[0].ip}' 2>/dev/null || echo "")
    
    if [ -n "$external_ip" ]; then
        base_url="http://$external_ip"
    else
        # Использование port-forward
        log "Внешний IP не найден, используем port-forward..."
        kubectl port-forward service/api-gateway 8080:8080 -n $NAMESPACE >/dev/null 2>&1 &
        local port_forward_pid=$!
        sleep 5
        base_url="http://localhost:8080"
    fi
    
    # Проверка API Gateway
    local retry=0
    local max_retries=5
    
    while [ $retry -lt $max_retries ]; do
        if curl -f -s "$base_url/health" >/dev/null 2>&1; then
            success "API Gateway отвечает на health check"
            break
        else
            ((retry++))
            if [ $retry -eq $max_retries ]; then
                error "API Gateway не отвечает на health check после $max_retries попыток"
                return 1
            fi
            log "Попытка $retry/$max_retries для API Gateway..."
            sleep 10
        fi
    done
    
    # Очистка port-forward
    if [ -n "${port_forward_pid:-}" ]; then
        kill $port_forward_pid 2>/dev/null || true
    fi
    
    return 0
}

# Проверка метрик
check_metrics() {
    log "Проверка доступности метрик..."
    
    local prometheus_url="http://prometheus:9090"
    
    # Проверка базовых метрик
    local metrics=("up" "http_requests_total" "node_cpu_seconds_total")
    
    for metric in "${metrics[@]}"; do
        if curl -s "$prometheus_url/api/v1/query?query=$metric" | grep -q "success"; then
            success "Метрика $metric доступна"
        else
            error "Метрика $metric недоступна"
        fi
    done
}

# Проверка алертов
check_alerts() {
    log "Проверка активных алертов..."
    
    local alertmanager_url="http://alertmanager:9093"
    
    local active_alerts=$(curl -s "$alertmanager_url/api/v1/alerts" | jq '.data | length' 2>/dev/null || echo "0")
    
    if [ "$active_alerts" -eq 0 ]; then
        success "Нет активных алертов"
    else
        error "Обнаружено $active_alerts активных алертов"
        # Вывод деталей алертов
        curl -s "$alertmanager_url/api/v1/alerts" | jq '.data[] | {alertname: .labels.alertname, status: .status.state}'
    fi
}

# ==================== ОСНОВНАЯ ПРОВЕРКА ====================
main() {
    log "🏥 Начало комплексной проверки работоспособности"
    
    local checks_passed=0
    local total_checks=5
    
    # Проверка подов
    if check_pods; then
        ((checks_passed++))
    fi
    
    # Проверка endpoints
    if check_endpoints; then
        ((checks_passed++))
    fi
    
    # HTTP проверка
    if check_http_health; then
        ((checks_passed++))
    fi
    
    # Проверка метрик
    if check_metrics; then
        ((checks_passed++))
    fi
    
    # Проверка алертов
    if check_alerts; then
        ((checks_passed++))
    fi
    
    # Итоговый результат
    log "📊 Результаты проверки: $checks_passed/$total_checks проверок пройдено"
    
    if [ $checks_passed -eq $total_checks ]; then
        success "🎉 Все проверки пройдены успешно!"
        return 0
    else
        error "⚠️ Некоторые проверки завершились с ошибками"
        return 1
    fi
}

# Запуск основной функции
main "$@"
```

## GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
# GitHub Actions workflow для автоматического развертывания

name: Deploy Microservices

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ==================== ТЕСТИРОВАНИЕ ====================
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      redis:
        image: redis
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
    
    - name: Cache Maven packages
      uses: actions/cache@v3
      with:
        path: ~/.m2
        key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
        restore-keys: ${{ runner.os }}-m2
    
    - name: Run tests
      run: |
        cd services/user-service && mvn test
        cd ../order-service && mvn test
        cd ../api-gateway && mvn test

  # ==================== СБОРКА И ПУБЛИКАЦИЯ ====================
  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    
    strategy:
      matrix:
        service: [user-service, order-service, api-gateway]
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v3
    
    - name: Log in to Container Registry
      uses: docker/login-action@v2
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${{ matrix.service }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: ./services/${{ matrix.service }}
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}

  # ==================== РАЗВЕРТЫВАНИЕ ====================
  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v3
    
    - name: Set up kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: 'v1.28.0'
    
    - name: Deploy to Kubernetes
      env:
        KUBE_CONFIG: ${{ secrets.KUBE_CONFIG }}
      run: |
        echo "$KUBE_CONFIG" | base64 -d > kubeconfig
        export KUBECONFIG=kubeconfig
        ./scripts/deploy-production.sh ${{ github.sha }}
```

## Terraform Infrastructure

```hcl
# infrastructure/terraform/main.tf
# Terraform конфигурация для автоматизации инфраструктуры

terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.20"
    }
  }
}

# ==================== ПРОВАЙДЕРЫ ====================
provider "aws" {
  region = var.aws_region
}

provider "kubernetes" {
  host                   = module.eks.cluster_endpoint
  cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)
  token                  = data.aws_eks_cluster_auth.cluster.token
}

# ==================== EKS КЛАСТЕР ====================
module "eks" {
  source = "terraform-aws-modules/eks/aws"
  
  cluster_name    = var.cluster_name
  cluster_version = "1.28"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
  
  # Node groups
  eks_managed_node_groups = {
    microservices = {
      instance_types = ["t3.medium"]
      min_size       = 2
      max_size       = 10
      desired_size   = 3
      
      labels = {
        role = "microservices"
      }
      
      taints = {
        microservices = {
          key    = "microservices"
          value  = "true"
          effect = "NO_SCHEDULE"
        }
      }
    }
  }
  
  tags = var.common_tags
}

# ==================== МОНИТОРИНГ ====================
resource "kubernetes_namespace" "monitoring" {
  metadata {
    name = "monitoring"
  }
}

resource "helm_release" "prometheus" {
  name       = "prometheus"
  repository = "https://prometheus-community.github.io/helm-charts"
  chart      = "kube-prometheus-stack"
  namespace  = kubernetes_namespace.monitoring.metadata[0].name
  
  values = [
    file("${path.module}/helm-values/prometheus.yaml")
  ]
}
```

## Проверочные вопросы

1. **Какие стадии включает типичный CI/CD pipeline?**
2. **В чем разница между blue-green и rolling deployment?**
3. **Как обеспечить безопасность в CI/CD pipeline?**
4. **Зачем нужен health check после развертывания?**

## Практические задания

### Задание 1: Настройка CI/CD
- Создайте GitLab CI pipeline
- Добавьте автоматические тесты
- Настройте автоматическое развертывание

### Задание 2: Скрипты автоматизации
- Напишите скрипт развертывания
- Добавьте функции отката
- Реализуйте health check

### Задание 3: Infrastructure as Code
- Создайте Terraform конфигурацию
- Автоматизируйте создание кластера
- Добавьте мониторинг через код

## Связанные темы

- [[kubernetes-deployment]] - K8s развертывание
- [[monitoring-logging]] - Мониторинг в CI/CD
- [[ansible-automation]] - Ansible автоматизация 