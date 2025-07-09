# ☸️ Kubernetes Deployment: Развертывание в K8s

## Описание

Kubernetes deployment обеспечивает контейнерную оркестрацию микросервисов с автоматическим масштабированием, балансировкой нагрузки и управлением жизненным циклом приложений.

## Namespace и базовая структура

### Создание namespace

```yaml
# k8s/00-namespace.yaml
# Создание изолированного пространства имен для микросервисов

apiVersion: v1
kind: Namespace
metadata:
  name: microservices
  labels:
    name: microservices
    environment: production
    project: microservices-app
  annotations:
    description: "Пространство имен для микросервисной архитектуры"
    
---
# Ограничения ресурсов для namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: microservices-quota
  namespace: microservices
spec:
  hard:
    requests.cpu: "4"              # Максимум 4 CPU cores в запросах
    requests.memory: 8Gi           # Максимум 8GB памяти в запросах
    limits.cpu: "8"                # Максимум 8 CPU cores в лимитах
    limits.memory: 16Gi            # Максимум 16GB памяти в лимитах
    persistentvolumeclaims: "10"   # Максимум 10 PVC
    services: "20"                 # Максимум 20 сервисов
    secrets: "10"                  # Максимум 10 секретов
    configmaps: "10"               # Максимум 10 ConfigMap

---
# Сетевые политики для безопасности
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: microservices-network-policy
  namespace: microservices
spec:
  podSelector: {}                  # Применяется ко всем подам в namespace
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: microservices      # Разрешить трафик из того же namespace
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx      # Разрешить трафик от Ingress
  egress:
  - {}                            # Разрешить весь исходящий трафик
```

## ConfigMap и Secrets

### Конфигурационные данные

```yaml
# k8s/01-configmap.yaml
# Конфигурационные параметры приложения

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: microservices
  labels:
    app: microservices
    component: config
data:
  # ==================== НАСТРОЙКИ БАЗЫ ДАННЫХ ====================
  DATABASE_HOST: "postgres-service"
  DATABASE_PORT: "5432"
  DATABASE_NAME: "microservices_db"
  
  # ==================== НАСТРОЙКИ REDIS ====================
  REDIS_HOST: "redis-service"
  REDIS_PORT: "6379"
  
  # ==================== НАСТРОЙКИ ПРИЛОЖЕНИЯ ====================
  LOG_LEVEL: "info"
  ENVIRONMENT: "production"
  API_VERSION: "v1"
  
  # ==================== НАСТРОЙКИ СЕРВИСОВ ====================
  USER_SERVICE_PORT: "8081"
  ORDER_SERVICE_PORT: "8082"
  API_GATEWAY_PORT: "8080"
  
  # ==================== НАСТРОЙКИ МОНИТОРИНГА ====================
  METRICS_ENABLED: "true"
  METRICS_PORT: "9090"
  HEALTH_CHECK_PATH: "/health"
  
  # ==================== НАСТРОЙКИ КЭШИРОВАНИЯ ====================
  CACHE_TTL: "3600"
  CACHE_MAX_SIZE: "1000"

---
# Конфигурация логирования
apiVersion: v1
kind: ConfigMap
metadata:
  name: logging-config
  namespace: microservices
data:
  log4j2.xml: |
    <?xml version="1.0" encoding="UTF-8"?>
    <Configuration status="WARN">
      <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
          <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
        <File name="FileAppender" fileName="/var/log/app/application.log">
          <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </File>
      </Appenders>
      <Loggers>
        <Root level="info">
          <AppenderRef ref="Console"/>
          <AppenderRef ref="FileAppender"/>
        </Root>
      </Loggers>
    </Configuration>
```

### Секретные данные

```yaml
# k8s/02-secrets.yaml
# Секретные данные (пароли, ключи API, сертификаты)

apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: microservices
  labels:
    app: microservices
    component: secrets
type: Opaque
data:
  # ==================== ПАРОЛИ БАЗ ДАННЫХ ====================
  # Значения должны быть закодированы в base64
  # echo -n 'your-password' | base64
  postgres-password: cG9zdGdyZXNfc3VwZXJfc2VjcmV0X3Bhc3N3b3Jk    # postgres_super_secret_password
  postgres-user: bWljcm9zZXJ2aWNlc191c2Vy                           # microservices_user
  redis-password: cmVkaXNfc3VwZXJfc2VjcmV0X3Bhc3N3b3Jk                # redis_super_secret_password
  
  # ==================== JWT И ШИФРОВАНИЕ ====================
  jwt-secret: and0X3N1cGVyX3NlY3JldF9rZXlfZm9yX2p3dF90b2tlbnM=      # jwt_super_secret_key_for_jwt_tokens
  encryption-key: ZW5jcnlwdGlvbl9zdXBlcl9zZWNyZXRfa2V5XzI1Ng==        # encryption_super_secret_key_256
  
  # ==================== API КЛЮЧИ ====================
  api-key: YXBpX2tleV9mb3JfZXh0ZXJuYWxfc2VydmljZXM=                  # api_key_for_external_services
  
  # ==================== МОНИТОРИНГ ====================
  grafana-admin-password: Z3JhZmFuYV9hZG1pbl9wYXNzd29yZA==           # grafana_admin_password

---
# Секреты для Docker Registry (если используется приватный registry)
apiVersion: v1
kind: Secret
metadata:
  name: registry-secret
  namespace: microservices
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: eyJhdXRocyI6eyJyZWdpc3RyeS5teWNvbXBhbnkuY29tIjp7InVzZXJuYW1lIjoiZGVwbG95IiwicGFzc3dvcmQiOiJyZWdpc3RyeV9wYXNzd29yZCIsImF1dGgiOiJaR1Z3Ykc5NU9uSmxaMmx6ZEhKNVgzQmhjM04zYjNKayJ9fX0=

---
# TLS сертификаты для HTTPS
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
  namespace: microservices
type: kubernetes.io/tls
data:
  tls.crt: |
    # Здесь должен быть base64-encoded SSL сертификат
    LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0t...
  tls.key: |
    # Здесь должен быть base64-encoded приватный ключ
    LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0t...
```

## Deployments микросервисов

### User Service Deployment

```yaml
# k8s/03-user-service.yaml
# Deployment для User Service

apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: microservices
  labels:
    app: user-service
    component: microservice
    tier: backend
spec:
  replicas: 3                      # Количество экземпляров
  strategy:
    type: RollingUpdate           # Стратегия обновления
    rollingUpdate:
      maxSurge: 1                 # Максимум +1 pod при обновлении
      maxUnavailable: 0           # Не допускать недоступности подов
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
        component: microservice
        tier: backend
      annotations:
        prometheus.io/scrape: "true"    # Разрешить Prometheus собирать метрики
        prometheus.io/port: "9090"      # Порт для метрик
        prometheus.io/path: "/metrics"  # Путь для метрик
    spec:
      # ==================== БЕЗОПАСНОСТЬ ====================
      serviceAccountName: microservices-sa
      securityContext:
        runAsNonRoot: true         # Запуск от непривилегированного пользователя
        runAsUser: 1000           # UID пользователя
        fsGroup: 1000             # GID группы для volumes
      
      # ==================== КОНТЕЙНЕРЫ ====================
      containers:
      - name: user-service
        image: registry.mycompany.com/user-service:v1.2.3
        imagePullPolicy: Always
        
        # ==================== ПОРТЫ ====================
        ports:
        - name: http
          containerPort: 8081
          protocol: TCP
        - name: metrics
          containerPort: 9090
          protocol: TCP
        
        # ==================== ПЕРЕМЕННЫЕ ОКРУЖЕНИЯ ====================
        env:
        - name: DATABASE_URL
          value: "postgres://$(POSTGRES_USER):$(POSTGRES_PASSWORD)@$(DATABASE_HOST):$(DATABASE_PORT)/$(DATABASE_NAME)"
        - name: REDIS_URL
          value: "redis://:$(REDIS_PASSWORD)@$(REDIS_HOST):$(REDIS_PORT)"
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: jwt-secret
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: postgres-user
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: postgres-password
        - name: REDIS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: redis-password
        
        # ==================== КОНФИГУРАЦИЯ ИЗ CONFIGMAP ====================
        envFrom:
        - configMapRef:
            name: app-config
        
        # ==================== РЕСУРСЫ ====================
        resources:
          requests:
            memory: "256Mi"        # Запрос памяти
            cpu: "200m"           # Запрос 0.2 CPU
          limits:
            memory: "512Mi"        # Лимит памяти
            cpu: "500m"           # Лимит 0.5 CPU
        
        # ==================== ПРОВЕРКИ РАБОТОСПОСОБНОСТИ ====================
        livenessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 30  # Начальная задержка
          periodSeconds: 10        # Интервал проверки
          timeoutSeconds: 5        # Таймаут запроса
          failureThreshold: 3      # Количество неудач для перезапуска
          
        readinessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 5   # Начальная задержка
          periodSeconds: 5         # Интервал проверки
          timeoutSeconds: 3        # Таймаут запроса
          failureThreshold: 3      # Количество неудач для исключения из балансировки
        
        # ==================== STARTUP PROBE ====================
        startupProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 10     # Дать больше времени на старт
        
        # ==================== VOLUMES ====================
        volumeMounts:
        - name: logs
          mountPath: /var/log/app
        - name: config
          mountPath: /app/config
          readOnly: true
        - name: temp
          mountPath: /tmp
      
      # ==================== VOLUMES ====================
      volumes:
      - name: logs
        emptyDir: {}              # Временное хранилище для логов
      - name: config
        configMap:
          name: logging-config    # Конфигурация логирования
      - name: temp
        emptyDir:
          sizeLimit: 100Mi        # Ограничение размера временного хранилища
      
      # ==================== ДОПОЛНИТЕЛЬНЫЕ НАСТРОЙКИ ====================
      imagePullSecrets:
      - name: registry-secret     # Секрет для доступа к приватному registry
      
      # Настройки планировщика
      affinity:
        podAntiAffinity:         # Предотвращение размещения подов на одном узле
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - user-service
              topologyKey: "kubernetes.io/hostname"
      
      # Tolerations для специальных узлов
      tolerations:
      - key: "microservices"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"

---
# Service для User Service
apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: microservices
  labels:
    app: user-service
    component: microservice
spec:
  type: ClusterIP               # Внутренний сервис
  ports:
  - name: http
    port: 8081                  # Порт сервиса
    targetPort: http            # Порт контейнера
    protocol: TCP
  - name: metrics
    port: 9090
    targetPort: metrics
    protocol: TCP
  selector:
    app: user-service           # Селектор для подов
  sessionAffinity: None         # Без привязки сессий
```

### Order Service Deployment

```yaml
# k8s/04-order-service.yaml
# Deployment для Order Service

apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  namespace: microservices
  labels:
    app: order-service
    component: microservice
    tier: backend
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
        component: microservice
        tier: backend
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
        prometheus.io/path: "/metrics"
    spec:
      serviceAccountName: microservices-sa
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
      
      containers:
      - name: order-service
        image: registry.mycompany.com/order-service:v1.2.3
        imagePullPolicy: Always
        
        ports:
        - name: http
          containerPort: 8082
          protocol: TCP
        - name: metrics
          containerPort: 9090
          protocol: TCP
        
        env:
        - name: DATABASE_URL
          value: "postgres://$(POSTGRES_USER):$(POSTGRES_PASSWORD)@$(DATABASE_HOST):$(DATABASE_PORT)/$(DATABASE_NAME)"
        - name: USER_SERVICE_URL
          value: "http://user-service:8081"  # Обращение к другому сервису через DNS
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: postgres-user
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: postgres-password
        
        envFrom:
        - configMapRef:
            name: app-config
        
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        
        livenessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
          
        readinessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        
        startupProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 10
        
        volumeMounts:
        - name: logs
          mountPath: /var/log/app
        - name: config
          mountPath: /app/config
          readOnly: true
      
      volumes:
      - name: logs
        emptyDir: {}
      - name: config
        configMap:
          name: logging-config
      
      imagePullSecrets:
      - name: registry-secret
      
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - order-service
              topologyKey: "kubernetes.io/hostname"

---
apiVersion: v1
kind: Service
metadata:
  name: order-service
  namespace: microservices
  labels:
    app: order-service
    component: microservice
spec:
  type: ClusterIP
  ports:
  - name: http
    port: 8082
    targetPort: http
    protocol: TCP
  - name: metrics
    port: 9090
    targetPort: metrics
    protocol: TCP
  selector:
    app: order-service
```

## API Gateway и Ingress

### API Gateway Deployment

```yaml
# k8s/05-api-gateway.yaml
# Deployment для API Gateway

apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  namespace: microservices
  labels:
    app: api-gateway
    component: gateway
    tier: frontend
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: api-gateway
  template:
    metadata:
      labels:
        app: api-gateway
        component: gateway
        tier: frontend
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
        prometheus.io/path: "/metrics"
    spec:
      serviceAccountName: microservices-sa
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
      
      containers:
      - name: api-gateway
        image: registry.mycompany.com/api-gateway:v1.2.3
        imagePullPolicy: Always
        
        ports:
        - name: http
          containerPort: 8080
          protocol: TCP
        - name: metrics
          containerPort: 9090
          protocol: TCP
        
        env:
        - name: USER_SERVICE_URL
          value: "http://user-service:8081"
        - name: ORDER_SERVICE_URL
          value: "http://order-service:8082"
        - name: REDIS_URL
          value: "redis://:$(REDIS_PASSWORD)@$(REDIS_HOST):$(REDIS_PORT)"
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: jwt-secret
        - name: REDIS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: redis-password
        
        envFrom:
        - configMapRef:
            name: app-config
        
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        
        livenessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
          
        readinessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        
        startupProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 10
        
        volumeMounts:
        - name: logs
          mountPath: /var/log/app
        - name: config
          mountPath: /app/config
          readOnly: true
      
      volumes:
      - name: logs
        emptyDir: {}
      - name: config
        configMap:
          name: logging-config
      
      imagePullSecrets:
      - name: registry-secret
      
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - api-gateway
              topologyKey: "kubernetes.io/hostname"

---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway
  namespace: microservices
  labels:
    app: api-gateway
    component: gateway
spec:
  type: ClusterIP
  ports:
  - name: http
    port: 8080
    targetPort: http
    protocol: TCP
  - name: metrics
    port: 9090
    targetPort: metrics
    protocol: TCP
  selector:
    app: api-gateway
```

### Ingress конфигурация

```yaml
# k8s/06-ingress.yaml
# Ingress для внешнего доступа к API Gateway

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: microservices-ingress
  namespace: microservices
  labels:
    app: microservices
    component: ingress
  annotations:
    # ==================== NGINX INGRESS НАСТРОЙКИ ====================
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /
    
    # ==================== SSL И HTTPS ====================
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    
    # ==================== RATE LIMITING ====================
    nginx.ingress.kubernetes.io/rate-limit: "100"                # 100 RPS
    nginx.ingress.kubernetes.io/rate-limit-window: "1s"
    nginx.ingress.kubernetes.io/rate-limit-burst: "50"
    
    # ==================== CORS ====================
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "https://app.mycompany.com"
    nginx.ingress.kubernetes.io/cors-allow-methods: "GET, POST, PUT, DELETE, OPTIONS"
    nginx.ingress.kubernetes.io/cors-allow-headers: "DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range,Authorization"
    
    # ==================== ДОПОЛНИТЕЛЬНЫЕ ЗАГОЛОВКИ ====================
    nginx.ingress.kubernetes.io/configuration-snippet: |
      add_header X-Frame-Options "SAMEORIGIN" always;
      add_header X-Content-Type-Options "nosniff" always;
      add_header X-XSS-Protection "1; mode=block" always;
      add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    
    # ==================== LOAD BALANCING ====================
    nginx.ingress.kubernetes.io/upstream-vhost: "api.mycompany.com"
    nginx.ingress.kubernetes.io/load-balance: "round_robin"
    
    # ==================== ТАЙМАУТЫ ====================
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "5"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
    
    # ==================== РАЗМЕРЫ ====================
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    nginx.ingress.kubernetes.io/client-max-body-size: "10m"

spec:
  # ==================== TLS КОНФИГУРАЦИЯ ====================
  tls:
  - hosts:
    - api.mycompany.com
    secretName: microservices-tls-secret    # Автоматически создается cert-manager
  
  # ==================== ПРАВИЛА МАРШРУТИЗАЦИИ ====================
  rules:
  - host: api.mycompany.com
    http:
      paths:
      # Основной API маршрут
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-gateway
            port:
              number: 8080
      
      # Health check endpoint
      - path: /health
        pathType: Exact
        backend:
          service:
            name: api-gateway
            port:
              number: 8080
      
      # Метрики (только для внутреннего мониторинга)
      - path: /metrics
        pathType: Exact
        backend:
          service:
            name: api-gateway
            port:
              number: 9090

---
# ClusterIssuer для автоматического получения SSL сертификатов
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@mycompany.com
    privateKeySecretRef:
      name: letsencrypt-prod-private-key
    solvers:
    - http01:
        ingress:
          class: nginx
```

## Horizontal Pod Autoscaler

### HPA конфигурация

```yaml
# k8s/07-hpa.yaml
# Horizontal Pod Autoscaler для автоматического масштабирования

# HPA для User Service
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: user-service-hpa
  namespace: microservices
  labels:
    app: user-service
    component: autoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: user-service
  minReplicas: 2                    # Минимальное количество подов
  maxReplicas: 10                   # Максимальное количество подов
  
  # ==================== МЕТРИКИ ДЛЯ МАСШТАБИРОВАНИЯ ====================
  metrics:
  # Масштабирование по CPU
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70      # Целевая утилизация CPU 70%
  
  # Масштабирование по памяти
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80      # Целевая утилизация памяти 80%
  
  # Кастомная метрика: запросов в секунду
  - type: Pods
    pods:
      metric:
        name: requests_per_second
      target:
        type: AverageValue
        averageValue: "50"          # 50 запросов в секунду на под
  
  # ==================== ПОВЕДЕНИЕ МАСШТАБИРОВАНИЯ ====================
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300    # 5 минут стабилизации перед уменьшением
      policies:
      - type: Percent
        value: 50                        # Уменьшать не более чем на 50%
        periodSeconds: 60                # За период 1 минута
      - type: Pods
        value: 2                         # Или не более 2 подов за раз
        periodSeconds: 60
      selectPolicy: Min                  # Выбирать более консервативный подход
    
    scaleUp:
      stabilizationWindowSeconds: 0      # Немедленное увеличение при необходимости
      policies:
      - type: Percent
        value: 100                       # Удваивать количество подов
        periodSeconds: 15                # За 15 секунд
      - type: Pods
        value: 4                         # Или добавлять максимум 4 пода
        periodSeconds: 15
      selectPolicy: Max                  # Выбирать более агрессивный подход

---
# HPA для Order Service
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
  namespace: microservices
  labels:
    app: order-service
    component: autoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
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
  
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
      selectPolicy: Min
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 4
        periodSeconds: 15
      selectPolicy: Max

---
# HPA для API Gateway
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-gateway-hpa
  namespace: microservices
  labels:
    app: api-gateway
    component: autoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-gateway
  minReplicas: 3                        # API Gateway должен иметь больше экземпляров
  maxReplicas: 15
  
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60          # Более низкий порог для точки входа
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 75
  
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 33                       # Более осторожное уменьшение для gateway
        periodSeconds: 60
      selectPolicy: Min
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 5                        # Больше подов для gateway
        periodSeconds: 15
      selectPolicy: Max
```

## ServiceAccount и RBAC

### Права доступа

```yaml
# k8s/08-rbac.yaml
# Role-Based Access Control для микросервисов

# ServiceAccount для подов микросервисов
apiVersion: v1
kind: ServiceAccount
metadata:
  name: microservices-sa
  namespace: microservices
  labels:
    app: microservices
    component: rbac

---
# Role с минимальными необходимыми правами
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: microservices-role
  namespace: microservices
rules:
# Чтение информации о сервисах (для service discovery)
- apiGroups: [""]
  resources: ["services", "endpoints"]
  verbs: ["get", "list", "watch"]

# Чтение ConfigMaps (для динамической конфигурации)
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list", "watch"]

# Чтение Secrets (если приложению нужен доступ к секретам)
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
  resourceNames: ["app-secrets"]  # Ограничение только определенными секретами

# Чтение информации о подах (для health checks)
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]

---
# Привязка роли к ServiceAccount
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: microservices-rolebinding
  namespace: microservices
subjects:
- kind: ServiceAccount
  name: microservices-sa
  namespace: microservices
roleRef:
  kind: Role
  name: microservices-role
  apiGroup: rbac.authorization.k8s.io
```

## Скрипты развертывания

### Скрипт автоматического развертывания

```bash
#!/bin/bash
# scripts/k8s-deploy.sh
# Скрипт для развертывания микросервисов в Kubernetes

set -e

# ==================== ПАРАМЕТРЫ ====================
NAMESPACE=${1:-microservices}
VERSION=${2:-latest}
ENVIRONMENT=${3:-production}
DRY_RUN=${4:-false}

# Цвета для вывода
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

echo -e "${BLUE}🚀 Развертывание микросервисов в Kubernetes${NC}"
echo "=================================="
echo "Namespace: $NAMESPACE"
echo "Version: $VERSION"
echo "Environment: $ENVIRONMENT"
echo "Dry Run: $DRY_RUN"
echo ""

# ==================== ПРОВЕРКА ЗАВИСИМОСТЕЙ ====================
echo -e "${YELLOW}🔍 Проверка зависимостей...${NC}"

# Проверка kubectl
if ! command -v kubectl &> /dev/null; then
    echo -e "${RED}❌ kubectl не установлен${NC}"
    exit 1
fi

# Проверка подключения к кластеру
if ! kubectl cluster-info &> /dev/null; then
    echo -e "${RED}❌ Нет подключения к Kubernetes кластеру${NC}"
    exit 1
fi

echo -e "${GREEN}✅ Все зависимости проверены${NC}"

# ==================== СОЗДАНИЕ NAMESPACE ====================
echo -e "\n${YELLOW}📁 Создание namespace...${NC}"

if [ "$DRY_RUN" = "true" ]; then
    echo "DRY RUN: kubectl apply -f k8s/00-namespace.yaml"
else
    kubectl apply -f k8s/00-namespace.yaml
fi

# ==================== ПРИМЕНЕНИЕ CONFIGMAPS И SECRETS ====================
echo -e "\n${YELLOW}⚙️ Применение конфигураций...${NC}"

config_files=(
    "k8s/01-configmap.yaml"
    "k8s/02-secrets.yaml"
    "k8s/08-rbac.yaml"
)

for file in "${config_files[@]}"; do
    if [ -f "$file" ]; then
        echo "Применение $file..."
        if [ "$DRY_RUN" = "true" ]; then
            echo "DRY RUN: kubectl apply -f $file"
        else
            kubectl apply -f "$file"
        fi
    else
        echo -e "${RED}❌ Файл $file не найден${NC}"
        exit 1
    fi
done

# ==================== РАЗВЕРТЫВАНИЕ СЕРВИСОВ ====================
echo -e "\n${YELLOW}🔄 Развертывание микросервисов...${NC}"

# Замена версии в манифестах
if [ "$VERSION" != "latest" ]; then
    echo "Обновление версии образов до $VERSION..."
    for file in k8s/0[3-5]-*.yaml; do
        if [ "$DRY_RUN" = "true" ]; then
            echo "DRY RUN: Обновление версии в $file"
        else
            sed -i.bak "s|:v[0-9]\+\.[0-9]\+\.[0-9]\+|:$VERSION|g" "$file"
        fi
    done
fi

service_files=(
    "k8s/03-user-service.yaml"
    "k8s/04-order-service.yaml" 
    "k8s/05-api-gateway.yaml"
)

for file in "${service_files[@]}"; do
    echo "Развертывание $file..."
    if [ "$DRY_RUN" = "true" ]; then
        echo "DRY RUN: kubectl apply -f $file"
    else
        kubectl apply -f "$file"
        
        # Ожидание готовности deployment
        service_name=$(basename "$file" .yaml | cut -d'-' -f2-)
        echo "Ожидание готовности $service_name..."
        kubectl rollout status deployment/"$service_name" -n "$NAMESPACE" --timeout=300s
    fi
done

# ==================== ПРИМЕНЕНИЕ INGRESS И HPA ====================
echo -e "\n${YELLOW}🌐 Настройка Ingress и автомасштабирования...${NC}"

additional_files=(
    "k8s/06-ingress.yaml"
    "k8s/07-hpa.yaml"
)

for file in "${additional_files[@]}"; do
    if [ -f "$file" ]; then
        echo "Применение $file..."
        if [ "$DRY_RUN" = "true" ]; then
            echo "DRY RUN: kubectl apply -f $file"
        else
            kubectl apply -f "$file"
        fi
    fi
done

# ==================== ВОССТАНОВЛЕНИЕ BACKUP ФАЙЛОВ ====================
if [ "$VERSION" != "latest" ] && [ "$DRY_RUN" = "false" ]; then
    echo -e "\n${YELLOW}🔄 Восстановление оригинальных файлов...${NC}"
    for file in k8s/0[3-5]-*.yaml.bak; do
        if [ -f "$file" ]; then
            mv "$file" "${file%.bak}"
        fi
    done
fi

# ==================== ПРОВЕРКА РАЗВЕРТЫВАНИЯ ====================
echo -e "\n${YELLOW}🔍 Проверка состояния развертывания...${NC}"

if [ "$DRY_RUN" = "false" ]; then
    echo "Статус подов:"
    kubectl get pods -n "$NAMESPACE" -o wide
    
    echo -e "\nСтатус сервисов:"
    kubectl get services -n "$NAMESPACE"
    
    echo -e "\nСтатус Ingress:"
    kubectl get ingress -n "$NAMESPACE"
    
    echo -e "\nСтатус HPA:"
    kubectl get hpa -n "$NAMESPACE"
    
    # ==================== HEALTH CHECK ====================
    echo -e "\n${YELLOW}🏥 Проверка работоспособности сервисов...${NC}"
    
    # Ожидание готовности всех подов
    echo "Ожидание готовности всех подов..."
    kubectl wait --for=condition=ready pod -l component=microservice -n "$NAMESPACE" --timeout=300s
    
    # Проверка endpoints
    services=("user-service" "order-service" "api-gateway")
    for service in "${services[@]}"; do
        echo "Проверка $service..."
        if kubectl get endpoints "$service" -n "$NAMESPACE" | grep -q "$(kubectl get endpoints "$service" -n "$NAMESPACE" -o jsonpath='{.subsets[0].addresses[0].ip}')"; then
            echo -e "${GREEN}✅ $service готов${NC}"
        else
            echo -e "${RED}❌ $service не готов${NC}"
        fi
    done
    
    echo -e "\n${GREEN}🎉 Развертывание завершено успешно!${NC}"
    echo -e "\n📋 Полезные команды:"
    echo "   kubectl get all -n $NAMESPACE"
    echo "   kubectl logs -f deployment/api-gateway -n $NAMESPACE"
    echo "   kubectl port-forward service/api-gateway 8080:8080 -n $NAMESPACE"
else
    echo -e "\n${BLUE}ℹ️ Dry run завершен. Для реального развертывания запустите без параметра dry-run${NC}"
fi
```

## Проверочные вопросы

1. **Что такое Kubernetes namespace и зачем он нужен?**
2. **В чем разница между ConfigMap и Secret?**
3. **Как работает HPA и по каким метрикам он масштабирует?**
4. **Что такое liveness, readiness и startup probes?**
5. **Зачем нужен ServiceAccount и RBAC в Kubernetes?**

## Практические задания

### Задание 1: Базовое развертывание
- Создайте namespace для тестового приложения
- Разверните простой микросервис с ConfigMap
- Настройте Service и проверьте доступность

### Задание 2: Автомасштабирование
- Настройте HPA для микросервиса
- Проведите нагрузочное тестирование
- Проследите работу автомасштабирования

### Задание 3: Полное развертывание
- Разверните полную микросервисную архитектуру
- Настройте Ingress с SSL
- Проверьте мониторинг и логирование

## Связанные темы

- [[server-setup]] - Подготовка серверов для K8s
- [[nginx-load-balancer]] - Альтернатива Ingress
- [[monitoring-logging]] - Мониторинг в K8s
- [[automation-scripts]] - CI/CD для K8s 