# CI/CD Pipeline: От разработки до продакшена

**CI/CD (Continuous Integration/Continuous Deployment)** — это методология DevOps, которая автоматизирует процесс сборки, тестирования и развертывания приложений. Позволяет быстро и безопасно доставлять изменения в продакшен.

## Основы CI/CD

### Ключевые принципы

- **Continuous Integration** — автоматическая сборка и тестирование при каждом коммите
- **Continuous Deployment** — автоматическое развертывание в продакшен после успешных тестов
- **Continuous Delivery** — автоматическая подготовка к развертыванию с ручным подтверждением
- **Infrastructure as Code** — инфраструктура описывается в коде
- **Automated Testing** — автоматизированное тестирование на всех этапах

### Этапы CI/CD Pipeline

```mermaid
graph LR
    A[Code Commit] --> B[Build]
    B --> C[Unit Tests]
    C --> D[Integration Tests]
    D --> E[Security Scan]
    E --> F[Package]
    F --> G[Deploy to Staging]
    G --> H[E2E Tests]
    H --> I[Deploy to Production]
    I --> J[Monitoring]
```

---

## GitHub Actions для микросервисов

### Основной workflow

```yaml
# .github/workflows/ci-cd.yml
# Комплексный CI/CD pipeline для микросервисной архитектуры
name: CI/CD Pipeline

# ==================== ТРИГГЕРЫ ЗАПУСКА ====================
on:
  # Запуск при пуше в основные ветки
  push:
    branches: [main, develop]  # Основные ветки разработки
  # Запуск при создании Pull Request в main
  pull_request:
    branches: [main]

# ==================== ГЛОБАЛЬНЫЕ ПЕРЕМЕННЫЕ ====================
env:
  REGISTRY: ghcr.io                    # GitHub Container Registry
  IMAGE_NAME: ${{ github.repository }} # Имя репозитория как базовое имя образа

jobs:
  # ==================== ЭТАП 1: СТАТИЧЕСКИЙ АНАЛИЗ КОДА ====================
  code-quality:
    # Запуск на последней версии Ubuntu
    runs-on: ubuntu-latest
    
    steps:
    # Скачивание исходного кода из репозитория
    - uses: actions/checkout@v4
    
    # ==================== НАСТРОЙКА ОКРУЖЕНИЙ ====================
    # Установка Go для анализа user-service
    - name: Setup Go
      uses: actions/setup-go@v4
      with:
        go-version: '1.21'  # Версия Go для проекта
    
    # Установка Node.js для анализа api-gateway
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'  # LTS версия Node.js
    
    # Установка Python для анализа order-service
    - name: Setup Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'  # Версия Python для проекта
    
    # ==================== GO СТАТИЧЕСКИЙ АНАЛИЗ ====================
    # Линтер для проверки стиля кода Go
    - name: Go Lint
      run: |
        # Установка инструмента линтинга
        go install golang.org/x/lint/golint@latest
        # Проверка всех пакетов в user-service с выходом при ошибках
        golint -set_exit_status ./services/user-service/...
    
    # Встроенная проверка Go на корректность кода
    - name: Go Vet
      run: |
        cd services/user-service
        # Статический анализ на потенциальные ошибки
        go vet ./...
    
    # Проверка безопасности Go кода
    - name: Go Security Check
      run: |
        # Установка инструмента анализа безопасности
        go install github.com/securecodewarrior/gosec/v2/cmd/gosec@latest
        # Сканирование на уязвимости безопасности
        gosec ./services/user-service/...
    
    # ==================== PYTHON СТАТИЧЕСКИЙ АНАЛИЗ ====================
    # Комплексная проверка Python кода
    - name: Python Lint
      run: |
        cd services/order-service
        # Установка инструментов анализа Python
        pip install flake8 black mypy
        # Проверка на критические ошибки синтаксиса
        flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
        # Проверка форматирования кода
        black --check .
        # Статическая проверка типов
        mypy . --ignore-missing-imports
    
    # ==================== JAVASCRIPT/TYPESCRIPT АНАЛИЗ ====================
    # Проверка качества JS/TS кода в API Gateway
    - name: JS/TS Lint
      run: |
        cd services/api-gateway
        # Установка зависимостей
        npm install
        # Проверка стиля кода ESLint
        npm run lint
        # Проверка типов TypeScript
        npm run type-check

  # Этап 2: Юнит тестирование
  unit-tests:
    runs-on: ubuntu-latest
    needs: code-quality
    strategy:
      matrix:
        service: [user-service, order-service, api-gateway]
        include:
          - service: user-service
            language: go
          - service: order-service
            language: python
          - service: api-gateway
            language: node
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup language environment
      uses: actions/setup-go@v4
      if: matrix.language == 'go'
      with:
        go-version: '1.21'
    
    - name: Setup Python
      uses: actions/setup-python@v4
      if: matrix.language == 'python'
      with:
        python-version: '3.11'
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      if: matrix.language == 'node'
      with:
        node-version: '18'
    
    - name: Install dependencies
      run: |
        cd services/${{ matrix.service }}
        if [ "${{ matrix.language }}" = "go" ]; then
          go mod download
        elif [ "${{ matrix.language }}" = "python" ]; then
          pip install -r requirements.txt
          pip install pytest pytest-cov
        elif [ "${{ matrix.language }}" = "node" ]; then
          npm install
        fi
    
    - name: Run unit tests
      run: |
        cd services/${{ matrix.service }}
        if [ "${{ matrix.language }}" = "go" ]; then
          go test -v -race -coverprofile=coverage.out ./...
          go tool cover -html=coverage.out -o coverage.html
        elif [ "${{ matrix.language }}" = "python" ]; then
          pytest --cov=. --cov-report=xml --cov-report=html
        elif [ "${{ matrix.language }}" = "node" ]; then
          npm test -- --coverage
        fi
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./services/${{ matrix.service }}/coverage.xml
        flags: ${{ matrix.service }}
        name: ${{ matrix.service }}-coverage

  # Этап 3: Сборка Docker образов
  build-images:
    runs-on: ubuntu-latest
    needs: unit-tests
    strategy:
      matrix:
        service: [user-service, order-service, api-gateway]
    
    outputs:
      image-digest: ${{ steps.build.outputs.digest }}
    
    steps:
    - name: Checkout
      uses: actions/checkout@v4
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: Login to GitHub Container Registry
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
    
    - name: Build and push
      id: build
      uses: docker/build-push-action@v5
      with:
        context: ./services/${{ matrix.service }}
        platforms: linux/amd64,linux/arm64
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
        provenance: false
    
    - name: Generate SBOM
      uses: anchore/sbom-action@v0
      with:
        image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${{ matrix.service }}:${{ github.sha }}
        format: spdx-json
        output-file: ${{ matrix.service }}-sbom.spdx.json
    
    - name: Upload SBOM
      uses: actions/upload-artifact@v3
      with:
        name: ${{ matrix.service }}-sbom
        path: ${{ matrix.service }}-sbom.spdx.json

  # Этап 4: Сканирование безопасности
  security-scan:
    runs-on: ubuntu-latest
    needs: build-images
    strategy:
      matrix:
        service: [user-service, order-service, api-gateway]
    
    steps:
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${{ matrix.service }}:${{ github.sha }}
        format: sarif
        output: trivy-results-${{ matrix.service }}.sarif
    
    - name: Upload Trivy scan results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: trivy-results-${{ matrix.service }}.sarif

  # Этап 5: Интеграционные тесты
  integration-tests:
    runs-on: ubuntu-latest
    needs: build-images
    
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
      
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Create test environment
      run: |
        cat > .env.test << EOF
        DATABASE_URL=postgres://postgres:postgres@localhost:5432/testdb
        REDIS_URL=redis://localhost:6379
        JWT_SECRET=test_secret
        EOF
    
    - name: Run integration tests
      run: |
        docker-compose -f docker-compose.test.yml up -d
        sleep 30
        
        # Проверка здоровья сервисов
        curl -f http://localhost:8080/health || exit 1
        curl -f http://localhost:8081/health || exit 1
        curl -f http://localhost:8082/health || exit 1
        
        # Запуск интеграционных тестов
        npm run test:integration
    
    - name: Cleanup
      if: always()
      run: docker-compose -f docker-compose.test.yml down -v

  # Этап 6: Развертывание в staging
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [security-scan, integration-tests]
    if: github.ref == 'refs/heads/develop'
    environment: staging
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Deploy to staging
      run: |
        echo "Deploying to staging environment..."
        # Здесь будет реальная логика развертывания
        
    - name: Run E2E tests
      run: |
        echo "Running E2E tests on staging..."
        # Здесь будут E2E тесты
    
    - name: Notify Slack
      uses: 8398a7/action-slack@v3
      if: always()
      with:
        status: ${{ job.status }}
        text: 'Staging deployment completed'
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

  # Этап 7: Развертывание в продакшен
  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Deploy to production
      run: |
        echo "Deploying to production environment..."
        # Blue-Green deployment или Rolling update
        
    - name: Verify deployment
      run: |
        echo "Verifying production deployment..."
        # Проверка работоспособности
        
    - name: Create release
      uses: actions/create-release@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: v${{ github.run_number }}
        release_name: Release v${{ github.run_number }}
        body: |
          Changes in this Release:
          - Auto-generated release from commit ${{ github.sha }}
        draft: false
        prerelease: false
```

---

## GitLab CI/CD для микросервисов

### .gitlab-ci.yml

```yaml
# .gitlab-ci.yml
stages:
  - validate
  - test
  - build
  - security
  - deploy-staging
  - deploy-production

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  REGISTRY: $CI_REGISTRY
  IMAGE_NAME: $CI_PROJECT_PATH

# Шаблон для сборки сервисов
.build-service:
  stage: build
  image: docker:20.10.16
  services:
    - docker:20.10.16-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $REGISTRY/$IMAGE_NAME/$SERVICE_NAME:$CI_COMMIT_SHA ./services/$SERVICE_NAME
    - docker push $REGISTRY/$IMAGE_NAME/$SERVICE_NAME:$CI_COMMIT_SHA
    - docker tag $REGISTRY/$IMAGE_NAME/$SERVICE_NAME:$CI_COMMIT_SHA $REGISTRY/$IMAGE_NAME/$SERVICE_NAME:latest
    - docker push $REGISTRY/$IMAGE_NAME/$SERVICE_NAME:latest

# Шаблон для тестирования
.test-service:
  stage: test
  before_script:
    - cd services/$SERVICE_NAME
  artifacts:
    reports:
      junit: services/$SERVICE_NAME/test-results.xml
      coverage: services/$SERVICE_NAME/coverage.xml
    paths:
      - services/$SERVICE_NAME/coverage/

# Валидация кода
code-quality:
  stage: validate
  image: golangci/golangci-lint:latest
  script:
    - golangci-lint run ./services/user-service/...
  only:
    - merge_requests
    - develop
    - main

# Сборка сервисов
build-user-service:
  extends: .build-service
  variables:
    SERVICE_NAME: user-service

build-order-service:
  extends: .build-service
  variables:
    SERVICE_NAME: order-service

build-api-gateway:
  extends: .build-service
  variables:
    SERVICE_NAME: api-gateway

# Тестирование
test-user-service:
  extends: .test-service
  image: golang:1.21
  variables:
    SERVICE_NAME: user-service
  script:
    - go test -v -race -coverprofile=coverage.out ./...
    - go tool cover -func=coverage.out

test-order-service:
  extends: .test-service
  image: python:3.11
  variables:
    SERVICE_NAME: order-service
  script:
    - pip install -r requirements.txt
    - pytest --cov=. --cov-report=xml --cov-report=html

test-api-gateway:
  extends: .test-service
  image: node:18
  variables:
    SERVICE_NAME: api-gateway
  script:
    - npm install
    - npm test -- --coverage

# Интеграционные тесты
integration-tests:
  stage: test
  image: docker/compose:latest
  services:
    - docker:20.10.16-dind
  script:
    - docker-compose -f docker-compose.test.yml up -d
    - sleep 30
    - docker-compose -f docker-compose.test.yml exec -T api-gateway npm run test:integration
  after_script:
    - docker-compose -f docker-compose.test.yml down -v

# Сканирование безопасности
container-security:
  stage: security
  image: aquasec/trivy:latest
  script:
    - trivy image --exit-code 0 --no-progress --format template --template "@contrib/sarif.tpl" -o trivy-results.sarif $REGISTRY/$IMAGE_NAME/user-service:$CI_COMMIT_SHA
    - trivy image --exit-code 0 --no-progress --format template --template "@contrib/sarif.tpl" -o trivy-results.sarif $REGISTRY/$IMAGE_NAME/order-service:$CI_COMMIT_SHA
    - trivy image --exit-code 0 --no-progress --format template --template "@contrib/sarif.tpl" -o trivy-results.sarif $REGISTRY/$IMAGE_NAME/api-gateway:$CI_COMMIT_SHA
  artifacts:
    reports:
      sast: trivy-results.sarif

# Развертывание в staging
deploy-staging:
  stage: deploy-staging
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - echo "Deploying to staging..."
    - curl -X POST -H "Content-Type: application/json" -d '{"environment":"staging","version":"'$CI_COMMIT_SHA'"}' $STAGING_WEBHOOK_URL
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

# Развертывание в продакшен
deploy-production:
  stage: deploy-production
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - echo "Deploying to production..."
    - curl -X POST -H "Content-Type: application/json" -d '{"environment":"production","version":"'$CI_COMMIT_SHA'"}' $PRODUCTION_WEBHOOK_URL
  environment:
    name: production
    url: https://api.example.com
  when: manual
  only:
    - main
```

---

## Jenkins Pipeline

### Jenkinsfile

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_REPO = 'your-username'
        KUBECONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Code Quality') {
            parallel {
                stage('Go Lint') {
                    steps {
                        dir('services/user-service') {
                            sh 'golangci-lint run .'
                        }
                    }
                }
                stage('Python Lint') {
                    steps {
                        dir('services/order-service') {
                            sh 'flake8 .'
                            sh 'black --check .'
                        }
                    }
                }
                stage('JS Lint') {
                    steps {
                        dir('services/api-gateway') {
                            sh 'npm install'
                            sh 'npm run lint'
                        }
                    }
                }
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests - Go') {
                    steps {
                        dir('services/user-service') {
                            sh 'go test -v -race -coverprofile=coverage.out ./...'
                            sh 'go tool cover -html=coverage.out -o coverage.html'
                        }
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'services/user-service',
                                reportFiles: 'coverage.html',
                                reportName: 'Go Coverage Report'
                            ])
                        }
                    }
                }
                stage('Unit Tests - Python') {
                    steps {
                        dir('services/order-service') {
                            sh 'pip install -r requirements.txt'
                            sh 'pytest --cov=. --cov-report=xml --cov-report=html'
                        }
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'services/order-service/htmlcov',
                                reportFiles: 'index.html',
                                reportName: 'Python Coverage Report'
                            ])
                        }
                    }
                }
                stage('Unit Tests - Node') {
                    steps {
                        dir('services/api-gateway') {
                            sh 'npm install'
                            sh 'npm test -- --coverage'
                        }
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'services/api-gateway/coverage/lcov-report',
                                reportFiles: 'index.html',
                                reportName: 'Node Coverage Report'
                            ])
                        }
                    }
                }
            }
        }
        
        stage('Build Images') {
            parallel {
                stage('Build User Service') {
                    steps {
                        script {
                            def image = docker.build("${DOCKER_REPO}/user-service:${env.BUILD_NUMBER}", "./services/user-service")
                            docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-hub-credentials') {
                                image.push()
                                image.push('latest')
                            }
                        }
                    }
                }
                stage('Build Order Service') {
                    steps {
                        script {
                            def image = docker.build("${DOCKER_REPO}/order-service:${env.BUILD_NUMBER}", "./services/order-service")
                            docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-hub-credentials') {
                                image.push()
                                image.push('latest')
                            }
                        }
                    }
                }
                stage('Build API Gateway') {
                    steps {
                        script {
                            def image = docker.build("${DOCKER_REPO}/api-gateway:${env.BUILD_NUMBER}", "./services/api-gateway")
                            docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-hub-credentials') {
                                image.push()
                                image.push('latest')
                            }
                        }
                    }
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                script {
                    sh '''
                        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                            -v $(pwd):/tmp aquasec/trivy:latest image \
                            --exit-code 0 --no-progress --format template --template "@contrib/sarif.tpl" \
                            -o /tmp/trivy-results.sarif ${DOCKER_REPO}/user-service:${BUILD_NUMBER}
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-results.sarif', allowEmptyArchive: true
                }
            }
        }
        
        stage('Integration Tests') {
            steps {
                sh 'docker-compose -f docker-compose.test.yml up -d'
                sh 'sleep 30'
                sh 'docker-compose -f docker-compose.test.yml exec -T api-gateway npm run test:integration'
            }
            post {
                always {
                    sh 'docker-compose -f docker-compose.test.yml down -v'
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                script {
                    kubernetesDeploy(
                        configs: 'k8s/staging/*.yaml',
                        kubeconfigId: 'kubeconfig'
                    )
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                script {
                    kubernetesDeploy(
                        configs: 'k8s/production/*.yaml',
                        kubeconfigId: 'kubeconfig'
                    )
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            slackSend(
                channel: '#deployments',
                color: 'good',
                message: "✅ Pipeline succeeded: ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend(
                channel: '#deployments',
                color: 'danger',
                message: "❌ Pipeline failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
            )
        }
    }
}
```

---

## Стратегии развертывания

### 1. Blue-Green Deployment

```yaml
# k8s/blue-green-deployment.yaml
apiVersion: v1
kind: Service
metadata:
  name: user-service
  labels:
    app: user-service
spec:
  selector:
    app: user-service
    version: blue  # Переключение между blue и green
  ports:
  - port: 80
    targetPort: 8080

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service-blue
  labels:
    app: user-service
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
      version: blue
  template:
    metadata:
      labels:
        app: user-service
        version: blue
    spec:
      containers:
      - name: user-service
        image: user-service:v1.0.0
        ports:
        - containerPort: 8080

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service-green
  labels:
    app: user-service
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
      version: green
  template:
    metadata:
      labels:
        app: user-service
        version: green
    spec:
      containers:
      - name: user-service
        image: user-service:v1.1.0
        ports:
        - containerPort: 8080
```

### 2. Rolling Update

```yaml
# k8s/rolling-update.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2        # Максимум 2 дополнительных пода
      maxUnavailable: 1  # Максимум 1 под может быть недоступен
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
        image: user-service:latest
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 10
```

### 3. Canary Deployment

```yaml
# k8s/canary-deployment.yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: user-service
spec:
  replicas: 10
  strategy:
    canary:
      steps:
      - setWeight: 10  # 10% трафика на новую версию
      - pause:
          duration: 2m
      - setWeight: 25  # 25% трафика
      - pause:
          duration: 5m
      - setWeight: 50  # 50% трафика
      - pause:
          duration: 10m
      - setWeight: 100 # 100% трафика
      canaryService: user-service-canary
      stableService: user-service-stable
      trafficRouting:
        nginx:
          stableIngress: user-service-stable
          annotationPrefix: nginx.ingress.kubernetes.io
      analysis:
        templates:
        - templateName: success-rate
        startingStep: 2
        args:
        - name: service-name
          value: user-service
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
        image: user-service:v1.1.0
        ports:
        - containerPort: 8080
```

---

## Мониторинг CI/CD

### Prometheus метрики для CI/CD

```yaml
# monitoring/cicd-metrics.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    
    scrape_configs:
    - job_name: 'cicd-metrics'
      static_configs:
      - targets: ['jenkins:8080']
      metrics_path: '/prometheus'
      
    - job_name: 'deployment-metrics'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
        
    rule_files:
    - "cicd-rules.yml"

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: cicd-rules
data:
  cicd-rules.yml: |
    groups:
    - name: cicd
      rules:
      - alert: DeploymentFailed
        expr: increase(deployment_failures_total[5m]) > 0
        for: 0m
        labels:
          severity: critical
        annotations:
          summary: "Deployment failed"
          description: "Deployment {{ $labels.environment }} failed"
          
      - alert: HighTestFailureRate
        expr: (test_failures_total / test_runs_total) > 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High test failure rate"
          description: "Test failure rate is {{ $value }}%"
          
      - alert: LongPipelineRuntime
        expr: pipeline_runtime_seconds > 3600
        for: 0m
        labels:
          severity: warning
        annotations:
          summary: "Pipeline running too long"
          description: "Pipeline {{ $labels.pipeline }} running for {{ $value }} seconds"
```

---

## Безопасность в CI/CD

### Управление секретами

```yaml
# k8s/secrets-management.yaml
apiVersion: v1
kind: Secret
metadata:
  name: registry-credentials
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-docker-config>

---
apiVersion: v1
kind: Secret
metadata:
  name: database-credentials
type: Opaque
data:
  username: <base64-encoded-username>
  password: <base64-encoded-password>

---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: vault-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: app-secrets
    creationPolicy: Owner
  data:
  - secretKey: database-password
    remoteRef:
      key: secret/database
      property: password
```

### Подписывание образов

```yaml
# .github/workflows/sign-images.yml
name: Sign Container Images

on:
  push:
    branches: [main]

jobs:
  sign-images:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write
    
    steps:
    - name: Install Cosign
      uses: sigstore/cosign-installer@v3
      with:
        cosign-release: 'v2.2.0'
    
    - name: Log in to GitHub Container Registry
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Sign container image
      run: |
        cosign sign --yes ghcr.io/${{ github.repository }}/user-service:${{ github.sha }}
        cosign sign --yes ghcr.io/${{ github.repository }}/order-service:${{ github.sha }}
        cosign sign --yes ghcr.io/${{ github.repository }}/api-gateway:${{ github.sha }}
```

---

## Лучшие практики CI/CD

### 1. Конфигурация Pipeline

```yaml
# Пример хорошей конфигурации
stages:
  - validate    # Быстрая валидация (< 2 мин)
  - test        # Параллельные тесты (< 10 мин)
  - build       # Сборка артефактов (< 5 мин)
  - security    # Сканирование безопасности (< 5 мин)
  - deploy      # Развертывание (< 3 мин)

# Принципы:
# - Fail Fast: быстрое обнаружение ошибок
# - Parallel Execution: параллельное выполнение
# - Caching: кэширование зависимостей
# - Artifacts: сохранение результатов между этапами
```

### 2. Тестирование в Pipeline

```bash
# Пирамида тестирования в CI/CD
Unit Tests (70%)     # Быстрые, изолированные
Integration Tests (20%) # Тестирование взаимодействия
E2E Tests (10%)      # Полные сценарии

# Стратегия:
# - Unit tests на каждый commit
# - Integration tests на merge в develop
# - E2E tests на deploy в staging
# - Smoke tests на deploy в production
```

### 3. Управление окружениями

```yaml
# Стратегия окружений
environments:
  development:
    auto_deploy: true
    protection_rules: []
  
  staging:
    auto_deploy: true
    protection_rules:
      - required_reviewers: 1
  
  production:
    auto_deploy: false
    protection_rules:
      - required_reviewers: 2
      - required_status_checks: [security-scan, integration-tests]
```

---

## 🔗 Связанные темы

### 🚀 DevOps и инфраструктура
- **[Docker Containerization](docker-containerization.md)** - Контейнеризация для CI/CD
- **[Linux Deployment](linux-deployment.md)** - Развертывание на Linux серверах
- **[Deployment & Monitoring](deployment-monitoring.md)** - Мониторинг CI/CD процессов

### ⚙️ Технические навыки
- **[Testing](../technical-skills/testing.md)** - Автоматизированное тестирование
- **[Security](../technical-skills/security.md)** - Безопасность в CI/CD

### 🏛️ Архитектура
- **[Microservices Architecture](../architecture/microservices-architecture.md)** - CI/CD для микросервисов

---

**Путь обучения**: [Docker Containerization](docker-containerization.md) → CI/CD Pipeline → [Linux Deployment](linux-deployment.md)  
**Сложность**: ⭐⭐⭐⭐ (4/5)  
**Время изучения**: 3-4 недели  
**Практика**: Настройка полного CI/CD pipeline для микросервисов с автоматическим тестированием и развертыванием 