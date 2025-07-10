# 🚀 Frontend CI/CD - Автоматизация развертывания

> **Frontend CI/CD** - автоматизация процессов сборки, тестирования и развертывания фронтенд приложений с использованием современных CI/CD платформ.

## 📋 Обзор раздела

CI/CD для фронтенда включает автоматическую сборку, линтинг, тестирование, оптимизацию и развертывание. Современные платформы предоставляют мощные возможности для автоматизации всего жизненного цикла приложения.

## 🎯 GitHub Actions - Полная автоматизация

### Main Workflow Configuration

```yaml
# .github/workflows/ci-cd.yml - Основной CI/CD pipeline
name: 🚀 Frontend CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  release:
    types: [published]

env:
  NODE_VERSION: '18'
  CACHE_KEY_PREFIX: 'frontend-v1'

jobs:
  # =============================================================================
  # ЭТАП ПОДГОТОВКИ И АНАЛИЗА
  # =============================================================================
  analyze:
    name: 📊 Code Analysis
    runs-on: ubuntu-latest
    
    steps:
      - name: 🛎️ Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Нужно для SonarCloud
          
      - name: 🏗️ Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: 📦 Install dependencies
        run: |
          npm ci --prefer-offline --no-audit
          
      - name: 🔍 Run ESLint
        run: |
          npm run lint:report
          
      - name: 🎨 Check code formatting
        run: |
          npm run format:check
          
      - name: 📏 Type checking
        run: |
          npm run type-check
          
      - name: 📊 SonarCloud Scan
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          
      - name: 📈 Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/lcov.info
          flags: frontend
          name: frontend-coverage

  # =============================================================================
  # ЭТАП ТЕСТИРОВАНИЯ
  # =============================================================================
  test:
    name: 🧪 Tests
    runs-on: ubuntu-latest
    needs: [analyze]
    
    strategy:
      matrix:
        test-type: [unit, integration, e2e]
        
    steps:
      - name: 🛎️ Checkout code
        uses: actions/checkout@v4
        
      - name: 🏗️ Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: 📦 Install dependencies
        run: npm ci --prefer-offline --no-audit
        
      - name: 🧪 Run unit tests
        if: matrix.test-type == 'unit'
        run: |
          npm run test:unit -- --coverage --ci --watchAll=false
          
      - name: 🔗 Run integration tests
        if: matrix.test-type == 'integration'
        run: |
          npm run test:integration
          
      - name: 🎭 Run E2E tests
        if: matrix.test-type == 'e2e'
        run: |
          npm run build
          npm run preview &
          npx wait-on http://localhost:4173
          npm run test:e2e:ci
          
      - name: 📊 Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: test-results-${{ matrix.test-type }}
          path: |
            coverage/
            test-results/
            playwright-report/

  # =============================================================================
  # ЭТАП СБОРКИ
  # =============================================================================
  build:
    name: 🏗️ Build Application
    runs-on: ubuntu-latest
    needs: [analyze]
    
    strategy:
      matrix:
        environment: [development, staging, production]
        
    steps:
      - name: 🛎️ Checkout code
        uses: actions/checkout@v4
        
      - name: 🏗️ Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: 📦 Install dependencies
        run: npm ci --prefer-offline --no-audit
        
      - name: 🌍 Setup environment variables
        run: |
          cp .env.${{ matrix.environment }} .env.production
          
      - name: 🏗️ Build application
        run: |
          npm run build
          
      - name: 📊 Analyze bundle size
        run: |
          npm run analyze:bundle
          
      - name: 🗜️ Compress build artifacts
        run: |
          tar -czf build-${{ matrix.environment }}.tar.gz dist/
          
      - name: 📤 Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-${{ matrix.environment }}
          path: |
            build-${{ matrix.environment }}.tar.gz
            dist/
          retention-days: 30

  # =============================================================================
  # ЭТАП БЕЗОПАСНОСТИ
  # =============================================================================
  security:
    name: 🔒 Security Scanning
    runs-on: ubuntu-latest
    needs: [build]
    
    steps:
      - name: 🛎️ Checkout code
        uses: actions/checkout@v4
        
      - name: 🏗️ Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: 📦 Install dependencies
        run: npm ci --prefer-offline --no-audit
        
      - name: 🔍 Audit dependencies
        run: |
          npm audit --audit-level=moderate
          
      - name: 🔒 OWASP ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.7.0
        with:
          target: 'http://localhost:4173'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'
          
      - name: 🛡️ Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

  # =============================================================================
  # ЭТАП РАЗВЕРТЫВАНИЯ
  # =============================================================================
  deploy-preview:
    name: 🚀 Deploy Preview
    runs-on: ubuntu-latest
    needs: [test, build, security]
    if: github.event_name == 'pull_request'
    
    steps:
      - name: 🛎️ Checkout code
        uses: actions/checkout@v4
        
      - name: 📥 Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: build-development
          
      - name: 🚀 Deploy to Vercel Preview
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          scope: ${{ secrets.VERCEL_ORG_ID }}
          
      - name: 💬 Comment PR with preview URL
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '🚀 Preview deployed! Check it out at: https://your-preview-url.vercel.app'
            })

  deploy-staging:
    name: 🎯 Deploy to Staging
    runs-on: ubuntu-latest
    needs: [test, build, security]
    if: github.ref == 'refs/heads/develop'
    environment: staging
    
    steps:
      - name: 📥 Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: build-staging
          
      - name: 🚀 Deploy to staging
        run: |
          # AWS S3 + CloudFront deployment
          aws s3 sync dist/ s3://${{ secrets.S3_STAGING_BUCKET }} --delete
          aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_STAGING_ID }} --paths "/*"
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION: us-east-1

  deploy-production:
    name: 🌟 Deploy to Production
    runs-on: ubuntu-latest
    needs: [test, build, security]
    if: github.ref == 'refs/heads/main' || github.event_name == 'release'
    environment: production
    
    steps:
      - name: 📥 Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: build-production
          
      - name: 🚀 Deploy to production
        run: |
          # Blue-green deployment
          aws s3 sync dist/ s3://${{ secrets.S3_PRODUCTION_BUCKET }} --delete
          aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_PRODUCTION_ID }} --paths "/*"
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION: us-east-1
          
      - name: 🏷️ Create release tag
        if: github.event_name == 'push'
        run: |
          git tag v$(date +'%Y.%m.%d.%H%M')
          git push origin --tags

  # =============================================================================
  # ЭТАП МОНИТОРИНГА
  # =============================================================================
  monitor:
    name: 📊 Post-deployment Monitoring
    runs-on: ubuntu-latest
    needs: [deploy-production]
    if: github.ref == 'refs/heads/main'
    
    steps:
      - name: 🔍 Health check
        run: |
          curl -f https://your-production-url.com/health || exit 1
          
      - name: 📊 Lighthouse CI
        uses: treosh/lighthouse-ci-action@v9
        with:
          urls: |
            https://your-production-url.com
            https://your-production-url.com/products
          configPath: '.lighthouserc.js'
          uploadArtifacts: true
          temporaryPublicStorage: true
          
      - name: 📈 Send metrics to monitoring
        run: |
          # Отправка метрик в DataDog/New Relic
          curl -X POST "https://api.datadoghq.com/api/v1/series" \
            -H "Content-Type: application/json" \
            -H "DD-API-KEY: ${{ secrets.DATADOG_API_KEY }}" \
            -d '{
              "series": [{
                "metric": "frontend.deployment.success",
                "points": [['$(date +%s)', 1]],
                "tags": ["env:production", "version:${{ github.sha }}"]
              }]
            }'
```

## 🦊 GitLab CI Configuration

### Complete GitLab Pipeline

```yaml
# .gitlab-ci.yml - GitLab CI/CD конфигурация
variables:
  NODE_VERSION: "18"
  DOCKER_DRIVER: overlay2
  CACHE_KEY: "$CI_COMMIT_REF_SLUG-$CI_PROJECT_ID"

stages:
  - validate
  - test
  - build
  - security
  - deploy
  - monitor

# =============================================================================
# КЭШИРОВАНИЕ И ШАБЛОНЫ
# =============================================================================
.node_template: &node_template
  image: node:${NODE_VERSION}-alpine
  cache:
    key: ${CACHE_KEY}
    paths:
      - node_modules/
      - .npm/
  before_script:
    - npm ci --cache .npm --prefer-offline --no-audit

.aws_template: &aws_template
  image: amazon/aws-cli:latest
  before_script:
    - aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
    - aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
    - aws configure set default.region us-east-1

# =============================================================================
# ЭТАП ВАЛИДАЦИИ
# =============================================================================
lint:
  <<: *node_template
  stage: validate
  script:
    - npm run lint:report
    - npm run format:check
    - npm run type-check
  artifacts:
    reports:
      junit: reports/eslint-report.xml
    paths:
      - reports/
    expire_in: 1 week
  only:
    - merge_requests
    - main
    - develop

# =============================================================================
# ЭТАП ТЕСТИРОВАНИЯ
# =============================================================================
test:unit:
  <<: *node_template
  stage: test
  script:
    - npm run test:unit -- --coverage --ci --watchAll=false
  artifacts:
    reports:
      junit: reports/jest-junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    paths:
      - coverage/
    expire_in: 1 week
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'

test:e2e:
  <<: *node_template
  stage: test
  services:
    - name: selenium/standalone-chrome:latest
      alias: selenium
  script:
    - npm run build
    - npm run preview &
    - npx wait-on http://localhost:4173
    - npm run test:e2e:ci
  artifacts:
    when: always
    paths:
      - test-results/
      - playwright-report/
    expire_in: 1 week
  only:
    - merge_requests
    - main
    - develop

# =============================================================================
# ЭТАП СБОРКИ
# =============================================================================
build:development:
  <<: *node_template
  stage: build
  script:
    - cp .env.development .env.production
    - npm run build
    - npm run analyze:bundle
  artifacts:
    paths:
      - dist/
      - bundle-analyzer-report.html
    expire_in: 1 week
  only:
    - merge_requests
    - develop

build:production:
  <<: *node_template
  stage: build
  script:
    - cp .env.production .env.production
    - npm run build
    - npm run analyze:bundle
  artifacts:
    paths:
      - dist/
      - bundle-analyzer-report.html
    expire_in: 1 month
  only:
    - main
    - tags

# =============================================================================
# ЭТАП БЕЗОПАСНОСТИ
# =============================================================================
security:audit:
  <<: *node_template
  stage: security
  script:
    - npm audit --audit-level=moderate
    - npx snyk test --severity-threshold=high
  allow_failure: true
  only:
    - merge_requests
    - main
    - develop

security:sast:
  stage: security
  include:
    - template: Security/SAST.gitlab-ci.yml
  only:
    - merge_requests
    - main

# =============================================================================
# ЭТАП РАЗВЕРТЫВАНИЯ
# =============================================================================
deploy:review:
  <<: *aws_template
  stage: deploy
  script:
    - aws s3 sync dist/ s3://$S3_REVIEW_BUCKET/$CI_MERGE_REQUEST_IID --delete
    - echo "Review app deployed to https://$CI_MERGE_REQUEST_IID.review.example.com"
  environment:
    name: review/$CI_MERGE_REQUEST_IID
    url: https://$CI_MERGE_REQUEST_IID.review.example.com
    on_stop: stop:review
  dependencies:
    - build:development
  only:
    - merge_requests

stop:review:
  <<: *aws_template
  stage: deploy
  script:
    - aws s3 rm s3://$S3_REVIEW_BUCKET/$CI_MERGE_REQUEST_IID --recursive
  environment:
    name: review/$CI_MERGE_REQUEST_IID
    action: stop
  when: manual
  only:
    - merge_requests

deploy:staging:
  <<: *aws_template
  stage: deploy
  script:
    - aws s3 sync dist/ s3://$S3_STAGING_BUCKET --delete
    - aws cloudfront create-invalidation --distribution-id $CLOUDFRONT_STAGING_ID --paths "/*"
  environment:
    name: staging
    url: https://staging.example.com
  dependencies:
    - build:development
  only:
    - develop

deploy:production:
  <<: *aws_template
  stage: deploy
  script:
    - aws s3 sync dist/ s3://$S3_PRODUCTION_BUCKET --delete
    - aws cloudfront create-invalidation --distribution-id $CLOUDFRONT_PRODUCTION_ID --paths "/*"
  environment:
    name: production
    url: https://example.com
  dependencies:
    - build:production
  when: manual
  only:
    - main

# =============================================================================
# ЭТАП МОНИТОРИНГА
# =============================================================================
lighthouse:
  image: cypress/browsers:node16.16.0-chrome105-ff104
  stage: monitor
  script:
    - npm install -g @lhci/cli
    - lhci autorun
  artifacts:
    reports:
      performance: lhci_reports/manifest.json
  only:
    - main
```

## 📊 Quality Gates & Monitoring

### SonarQube Configuration

```properties
# sonar-project.properties
sonar.projectKey=frontend-app
sonar.projectName=Frontend Application
sonar.projectVersion=1.0

# Источники и тесты
sonar.sources=src
sonar.tests=src
sonar.test.inclusions=**/*.test.ts,**/*.test.tsx,**/*.spec.ts,**/*.spec.tsx
sonar.exclusions=**/*.stories.*,**/node_modules/**,**/dist/**,**/build/**

# Покрытие кода
sonar.typescript.lcov.reportPaths=coverage/lcov.info
sonar.javascript.lcov.reportPaths=coverage/lcov.info

# ESLint отчеты
sonar.eslint.reportPaths=reports/eslint-report.json

# Quality Gates
sonar.qualitygate.wait=true

# Настройки для TypeScript
sonar.typescript.node=node
```

### Lighthouse CI Configuration

```javascript
// .lighthouserc.js
module.exports = {
  ci: {
    collect: {
      url: [
        'http://localhost:4173',
        'http://localhost:4173/products',
        'http://localhost:4173/about'
      ],
      startServerCommand: 'npm run preview',
      startServerReadyPattern: 'ready in',
      numberOfRuns: 3
    },
    
    assert: {
      assertions: {
        'categories:performance': ['warn', { minScore: 0.9 }],
        'categories:accessibility': ['error', { minScore: 0.9 }],
        'categories:best-practices': ['warn', { minScore: 0.9 }],
        'categories:seo': ['warn', { minScore: 0.9 }],
        'categories:pwa': ['warn', { minScore: 0.5 }]
      }
    },
    
    upload: {
      target: 'temporary-public-storage'
    },
    
    server: {
      port: 9001,
      storage: './lighthouse-reports'
    }
  }
};
```

## 🔧 Deployment Strategies

### AWS S3 + CloudFront Deployment

```bash
#!/bin/bash
# scripts/deploy-aws.sh

set -e

ENVIRONMENT=${1:-staging}
BUILD_DIR="dist"
DATE=$(date +%Y%m%d_%H%M%S)

echo "🚀 Deploying to $ENVIRONMENT..."

# Настройка переменных окружения
case $ENVIRONMENT in
  "staging")
    S3_BUCKET=$S3_STAGING_BUCKET
    CLOUDFRONT_ID=$CLOUDFRONT_STAGING_ID
    ;;
  "production")
    S3_BUCKET=$S3_PRODUCTION_BUCKET
    CLOUDFRONT_ID=$CLOUDFRONT_PRODUCTION_ID
    ;;
  *)
    echo "❌ Unknown environment: $ENVIRONMENT"
    exit 1
    ;;
esac

# Проверка переменных
if [[ -z "$S3_BUCKET" || -z "$CLOUDFRONT_ID" ]]; then
  echo "❌ Missing required environment variables"
  exit 1
fi

# Backup текущей версии
echo "📦 Creating backup..."
aws s3 sync s3://$S3_BUCKET s3://$S3_BUCKET-backup-$DATE --delete

# Синхронизация файлов
echo "📤 Uploading files to S3..."
aws s3 sync $BUILD_DIR/ s3://$S3_BUCKET \
  --delete \
  --cache-control "public, max-age=31536000, immutable" \
  --exclude "*.html" \
  --exclude "service-worker.js"

# HTML файлы с коротким кэшем
aws s3 sync $BUILD_DIR/ s3://$S3_BUCKET \
  --cache-control "public, max-age=0, must-revalidate" \
  --include "*.html" \
  --include "service-worker.js"

# Инвалидация CloudFront
echo "🔄 Invalidating CloudFront cache..."
INVALIDATION_ID=$(aws cloudfront create-invalidation \
  --distribution-id $CLOUDFRONT_ID \
  --paths "/*" \
  --query 'Invalidation.Id' \
  --output text)

echo "⏳ Waiting for invalidation to complete..."
aws cloudfront wait invalidation-completed \
  --distribution-id $CLOUDFRONT_ID \
  --id $INVALIDATION_ID

echo "✅ Deployment completed successfully!"
echo "🌐 URL: https://$(aws cloudfront get-distribution --id $CLOUDFRONT_ID --query 'Distribution.DomainName' --output text)"
```

### Docker Deployment

```dockerfile
# Dockerfile.production
FROM node:18-alpine AS builder

WORKDIR /app

# Копирование файлов зависимостей
COPY package*.json ./
RUN npm ci --only=production --prefer-offline --no-audit

# Копирование исходного кода
COPY . .

# Сборка приложения
ARG NODE_ENV=production
ARG REACT_APP_API_URL
ARG REACT_APP_VERSION
RUN npm run build

# Production образ
FROM nginx:alpine AS production

# Копирование конфигурации nginx
COPY nginx.conf /etc/nginx/nginx.conf

# Копирование статических файлов
COPY --from=builder /app/dist /usr/share/nginx/html

# Добавление health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/health || exit 1

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# nginx.conf
events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    
    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
    
    server {
        listen 80;
        server_name localhost;
        root /usr/share/nginx/html;
        index index.html;
        
        # Caching strategy
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
        
        location / {
            try_files $uri $uri/ /index.html;
            add_header Cache-Control "no-cache";
        }
        
        # Health check endpoint
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }
}
```

## 🎯 Практические задания

### 🟢 Базовый уровень
1. **GitHub Actions** - Настроить базовый CI/CD pipeline для React приложения
2. **Automated Testing** - Добавить автоматическое тестирование в pipeline
3. **Environment Variables** - Настроить переменные окружения для разных сред

### 🟡 Средний уровень
4. **Multi-environment** - Создать pipeline для staging и production
5. **Security Scanning** - Добавить проверки безопасности в CI/CD
6. **Performance Monitoring** - Интегрировать Lighthouse CI

### 🔴 Продвинутый уровень
7. **Blue-Green Deployment** - Реализовать стратегию blue-green развертывания
8. **Custom Actions** - Создать собственные GitHub Actions
9. **Complete DevOps** - Настроить полную DevOps экосистему

## 🔗 Связанные разделы

- **[[linting-formatting|Linting & Formatting]]** - Автоматизация качества кода в CI/CD
- **[[../testing-frontend|Frontend Testing]]** - Стратегии тестирования в pipeline
- **[[../../infrastructure/linux-deployment|Linux Deployment]]** - Инфраструктура для развертывания
- **[[development-environment|Development Environment]]** - Локальная настройка среды

---

⏱️ **Время изучения**: 15-25 часов  
🎯 **Уровень**: Средний - Продвинутый  
📋 **Предварительные требования**: Git, GitHub/GitLab, AWS/Docker основы 