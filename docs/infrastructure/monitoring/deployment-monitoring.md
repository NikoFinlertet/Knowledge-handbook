# Развертывание и мониторинг

## Prometheus

**Prometheus** — система мониторинга с временными рядами.

### Конфигурация
```yaml
# prometheus.yml
# Основная конфигурация Prometheus для мониторинга микросервисов

# ==================== ГЛОБАЛЬНЫЕ НАСТРОЙКИ ====================
global:
  # Интервал сбора метрик по умолчанию (каждые 15 секунд)
  scrape_interval: 15s
  
  # Интервал выполнения правил алертов (каждые 15 секунд)
  evaluation_interval: 15s

# ==================== ПРАВИЛА АЛЕРТОВ ====================
# Файлы с правилами для генерации алертов
rule_files:
  - "alerts.yml"  # Файл с определениями алертов

# ==================== КОНФИГУРАЦИЯ СБОРА МЕТРИК ====================
scrape_configs:
  # ==================== USER SERVICE ====================
  - job_name: 'user-service'        # Имя задачи для User Service
    static_configs:
      - targets: ['user-service:8080']  # Адрес и порт сервиса
    metrics_path: /metrics            # Эндпоинт для сбора метрик
    scrape_interval: 10s             # Более частый сбор (каждые 10 секунд)

  # ==================== ORDER SERVICE ====================
  - job_name: 'order-service'       # Имя задачи для Order Service
    static_configs:
      - targets: ['order-service:8000']  # Адрес и порт сервиса (Python обычно на 8000)
    metrics_path: /metrics            # Эндпоинт для сбора метрик
    # Используется глобальный scrape_interval (15s)

  # ==================== API GATEWAY ====================
  - job_name: 'api-gateway'         # Имя задачи для API Gateway
    static_configs:
      - targets: ['api-gateway:3000']  # Адрес и порт сервиса (Node.js обычно на 3000)
    # Используется стандартный metrics_path (/metrics)
    # Используется глобальный scrape_interval (15s)

# ==================== НАСТРОЙКИ АЛЕРТОВ ====================
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093         # Адрес Alertmanager для отправки алертов
```

### Метрики в Go
```go
// services/user-service/internal/metrics/prometheus.go
// Модуль для сбора и экспорта метрик Prometheus в Go приложении

package metrics

import (
    "github.com/prometheus/client_golang/prometheus"      // Основная библиотека Prometheus
    "github.com/prometheus/client_golang/prometheus/promauto" // Автоматическая регистрация метрик
)

// ==================== ОПРЕДЕЛЕНИЕ МЕТРИК ====================
// Глобальные переменные с метриками, автоматически регистрируемые в Prometheus

var (
    // ==================== COUNTER VEC: Счетчик HTTP запросов ====================
    // Counter - монотонно возрастающий счетчик (никогда не уменьшается)
    // Vec означает, что метрика имеет labels (ярлыки) для группировки
    RequestsTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",              // Имя метрики в Prometheus
            Help: "Total number of HTTP requests",   // Описание метрики
        },
        []string{"method", "endpoint", "status"},    // Labels для группировки по методу, эндпоинту и статусу
    )

    // ==================== HISTOGRAM VEC: Распределение времени ответа ====================
    // Histogram - для измерения распределения значений (время выполнения, размеры и т.д.)
    RequestDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",  // Имя метрики
            Help:    "HTTP request duration in seconds", // Описание
            Buckets: prometheus.DefBuckets,            // Стандартные бакеты: 0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10
        },
        []string{"method", "endpoint"},               // Labels для группировки
    )

    // ==================== GAUGE: Текущее значение соединений с БД ====================
    // Gauge - метрика, которая может увеличиваться и уменьшаться (состояние системы)
    DatabaseConnections = promauto.NewGauge(
        prometheus.GaugeOpts{
            Name: "database_connections_active",       // Имя метрики
            Help: "Number of active database connections", // Описание
        },
    )

    // ==================== COUNTER: Общее количество регистраций ====================
    // Простой счетчик без labels
    UserRegistrations = promauto.NewCounter(
        prometheus.CounterOpts{
            Name: "user_registrations_total",          // Имя метрики
            Help: "Total number of user registrations", // Описание
        },
    )
)

// ==================== HTTP MIDDLEWARE ДЛЯ СБОРА МЕТРИК ====================
// Middleware автоматически собирает метрики для всех HTTP запросов
func PrometheusMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Записываем время начала обработки запроса
        start := time.Now()
        
        // ==================== ЗАХВАТ СТАТУС КОДА ====================
        // Создаем wrapper для ResponseWriter, чтобы захватить статус код ответа
        // По умолчанию Go не предоставляет доступ к статус коду в middleware
        ww := &responseWriter{ResponseWriter: w, statusCode: 200}
        
        // ==================== ОБРАБОТКА ЗАПРОСА ====================
        // Вызываем следующий handler в цепочке
        next.ServeHTTP(ww, r)
        
        // ==================== ВЫЧИСЛЕНИЕ ВРЕМЕНИ ВЫПОЛНЕНИЯ ====================
        // Вычисляем время выполнения запроса в секундах
        duration := time.Since(start).Seconds()
        
        // ==================== ОБНОВЛЕНИЕ МЕТРИК ====================
        // Увеличиваем счетчик запросов с соответствующими labels
        RequestsTotal.WithLabelValues(
            r.Method,                    // HTTP метод (GET, POST, etc.)
            r.URL.Path,                  // Путь запроса (/users, /orders, etc.)
            strconv.Itoa(ww.statusCode), // Статус код ответа (200, 404, 500, etc.)
        ).Inc()                          // Увеличиваем счетчик на 1
        
        // Записываем время выполнения в histogram
        RequestDuration.WithLabelValues(
            r.Method,     // HTTP метод
            r.URL.Path,   // Путь запроса
        ).Observe(duration) // Добавляем наблюдение (время выполнения)
    })
}

// ==================== HELPER СТРУКТУРА ====================
// responseWriter - wrapper для захвата статус кода ответа
type responseWriter struct {
    http.ResponseWriter
    statusCode int
}

// Переопределяем метод WriteHeader для захвата статус кода
func (rw *responseWriter) WriteHeader(code int) {
    rw.statusCode = code
    rw.ResponseWriter.WriteHeader(code)
}
```

### Метрики в Python
```python
# services/order-service/metrics.py
# Модуль для сбора и экспорта метрик Prometheus в Python приложении

from prometheus_client import Counter, Histogram, Gauge, start_http_server  # Основные типы метрик Prometheus
from functools import wraps  # Для создания декораторов
import time                  # Для измерения времени

# ==================== ОПРЕДЕЛЕНИЕ МЕТРИК ====================
# Глобальные переменные с метриками Prometheus

# ==================== COUNTER: Счетчик HTTP запросов ====================
# Counter - монотонно возрастающий счетчик (никогда не уменьшается)
REQUEST_COUNT = Counter(
    'http_requests_total',              # Имя метрики в Prometheus
    'Total HTTP requests',              # Описание метрики
    ['method', 'endpoint', 'status']    # Labels для группировки по методу, эндпоинту и статусу
)

# ==================== HISTOGRAM: Распределение времени ответа ====================
# Histogram - для измерения распределения значений (время выполнения, размеры и т.д.)
REQUEST_LATENCY = Histogram(
    'http_request_duration_seconds',    # Имя метрики
    'HTTP request latency',             # Описание
    ['method', 'endpoint']              # Labels для группировки
)

# ==================== COUNTER: Счетчик созданных заказов ====================
# Бизнес-метрика для отслеживания количества созданных заказов
ORDERS_CREATED = Counter(
    'orders_created_total',             # Имя метрики
    'Total orders created'              # Описание
)

# ==================== GAUGE: Текущее количество активных заказов ====================
# Gauge - метрика, которая может увеличиваться и уменьшаться (состояние системы)
ACTIVE_ORDERS = Gauge(
    'orders_active',                    # Имя метрики
    'Number of active orders'           # Описание
)

# ==================== ДЕКОРАТОР ДЛЯ ОТСЛЕЖИВАНИЯ МЕТРИК ====================
# Декоратор для автоматического сбора метрик HTTP запросов
def track_requests(f):
    """
    Декоратор для отслеживания HTTP запросов.
    Автоматически собирает метрики времени выполнения и количества запросов.
    """
    @wraps(f)                          # Сохраняем метаданные оригинальной функции
    async def decorated_function(*args, **kwargs):
        # ==================== НАЧАЛО ИЗМЕРЕНИЯ ВРЕМЕНИ ====================
        start_time = time.time()        # Записываем время начала выполнения
        
        try:
            # ==================== ВЫПОЛНЕНИЕ ОРИГИНАЛЬНОЙ ФУНКЦИИ ====================
            response = await f(*args, **kwargs)
            
            # ==================== ИЗВЛЕЧЕНИЕ СТАТУС КОДА ====================
            # Пытаемся получить статус код из ответа, по умолчанию 200
            status = getattr(response, 'status_code', 200)
            
            # ==================== ОБНОВЛЕНИЕ СЧЕТЧИКА ЗАПРОСОВ ====================
            # Увеличиваем счетчик запросов с соответствующими labels
            REQUEST_COUNT.labels(
                method=request.method,      # HTTP метод (GET, POST, etc.)
                endpoint=request.endpoint,  # Эндпоинт (например, /orders)
                status=status              # Статус код ответа
            ).inc()                        # Увеличиваем на 1
            
            return response
        finally:
            # ==================== ИЗМЕРЕНИЕ ВРЕМЕНИ ВЫПОЛНЕНИЯ ====================
            # Блок finally выполняется всегда, даже при исключениях
            REQUEST_LATENCY.labels(
                method=request.method,      # HTTP метод
                endpoint=request.endpoint   # Эндпоинт
            ).observe(time.time() - start_time)  # Записываем время выполнения
    
    return decorated_function

# ==================== ЗАПУСК PROMETHEUS HTTP SERVER ====================
# Запуск HTTP сервера для экспорта метрик на порту 8000
# Prometheus будет собирать метрики с http://service:8000/metrics
start_http_server(8000)

# ==================== ПРИМЕР ИСПОЛЬЗОВАНИЯ ====================
# Применение декоратора к функции:
# @track_requests
# async def create_order(order_data):
#     # Создание заказа
#     ORDERS_CREATED.inc()        # Увеличиваем счетчик созданных заказов
#     ACTIVE_ORDERS.inc()         # Увеличиваем счетчик активных заказов
#     return order

# Обновление активных заказов при завершении:
# def complete_order(order_id):
#     ACTIVE_ORDERS.dec()         # Уменьшаем счетчик активных заказов
```

## Grafana

### Dashboard JSON
```json
{
  "dashboard": {
    "title": "Microservices Dashboard",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{service}} - {{method}} {{endpoint}}"
          }
        ]
      },
      {
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "singlestat",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"4..|5..\"}[5m]) / rate(http_requests_total[5m])",
            "legendFormat": "Error Rate"
          }
        ]
      }
    ]
  }
}
```

## ELK Stack (Elasticsearch, Logstash, Kibana)

### Logstash конфигурация
```ruby
# logstash.conf
input {
  beats {
    port => 5044
  }
}

filter {
  if [fields][service] == "user-service" {
    grok {
      match => { 
        "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}" 
      }
    }
  }
  
  if [fields][service] == "order-service" {
    json {
      source => "message"
    }
  }
  
  date {
    match => [ "timestamp", "ISO8601" ]
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "microservices-%{+YYYY.MM.dd}"
  }
}
```

### Structured Logging
```go
// services/user-service/internal/logger/logger.go
package logger

import (
    "github.com/sirupsen/logrus"
)

type Logger struct {
    *logrus.Logger
}

func New() *Logger {
    log := logrus.New()
    log.SetFormatter(&logrus.JSONFormatter{})
    
    return &Logger{log}
}

func (l *Logger) WithUserID(userID string) *logrus.Entry {
    return l.WithField("user_id", userID)
}

func (l *Logger) WithRequestID(requestID string) *logrus.Entry {
    return l.WithField("request_id", requestID)
}

// Использование
logger := logger.New()
logger.WithFields(logrus.Fields{
    "user_id": "user-123",
    "action": "login",
    "ip": "192.168.1.1",
}).Info("User logged in")
```

```python
# services/order-service/logger.py
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_entry = {
            'timestamp': datetime.utcnow().isoformat(),
            'level': record.levelname,
            'message': record.getMessage(),
            'service': 'order-service',
        }
        
        if hasattr(record, 'user_id'):
            log_entry['user_id'] = record.user_id
        
        if hasattr(record, 'order_id'):
            log_entry['order_id'] = record.order_id
            
        return json.dumps(log_entry)

# Настройка
logger = logging.getLogger(__name__)
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)
logger.setLevel(logging.INFO)

# Использование
logger.info("Order created", extra={'user_id': 'user-123', 'order_id': 'order-456'})
```

## Health Checks

### Go Health Check
```go
// services/user-service/internal/health/health.go
package health

import (
    "context"
    "database/sql"
    "encoding/json"
    "net/http"
    "time"
)

type HealthCheck struct {
    db *sql.DB
}

type HealthStatus struct {
    Status   string            `json:"status"`
    Checks   map[string]string `json:"checks"`
    Version  string            `json:"version"`
    Uptime   string            `json:"uptime"`
}

func (h *HealthCheck) Handler(w http.ResponseWriter, r *http.Request) {
    status := HealthStatus{
        Status:  "ok",
        Checks:  make(map[string]string),
        Version: "1.0.0",
        Uptime:  getUptime(),
    }
    
    // Database check
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()
    
    if err := h.db.PingContext(ctx); err != nil {
        status.Status = "error"
        status.Checks["database"] = "unhealthy: " + err.Error()
        w.WriteHeader(http.StatusServiceUnavailable)
    } else {
        status.Checks["database"] = "healthy"
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(status)
}

func (h *HealthCheck) ReadinessHandler(w http.ResponseWriter, r *http.Request) {
    // Более строгие проверки для readiness
    if !h.isDatabaseReady() || !h.areExternalServicesReady() {
        w.WriteHeader(http.StatusServiceUnavailable)
        json.NewEncoder(w).Encode(map[string]string{
            "status": "not ready",
        })
        return
    }
    
    w.WriteHeader(http.StatusOK)
    json.NewEncoder(w).Encode(map[string]string{
        "status": "ready",
    })
}
```

### Python Health Check
```python
# services/order-service/health.py
from fastapi import FastAPI, HTTPException
from datetime import datetime
import asyncio
import asyncpg

app = FastAPI()

startup_time = datetime.utcnow()

@app.get("/health")
async def health_check():
    checks = {}
    status = "ok"
    
    # Database check
    try:
        # Простая проверка подключения к БД
        await check_database()
        checks["database"] = "healthy"
    except Exception as e:
        checks["database"] = f"unhealthy: {str(e)}"
        status = "error"
    
    # External service check
    try:
        await check_user_service()
        checks["user_service"] = "healthy"
    except Exception as e:
        checks["user_service"] = f"unhealthy: {str(e)}"
        status = "error"
    
    response = {
        "status": status,
        "checks": checks,
        "version": "1.0.0",
        "uptime": str(datetime.utcnow() - startup_time)
    }
    
    if status == "error":
        raise HTTPException(status_code=503, detail=response)
    
    return response

@app.get("/ready")
async def readiness_check():
    try:
        # Более строгие проверки
        await check_database_ready()
        await check_all_dependencies()
        return {"status": "ready"}
    except Exception:
        raise HTTPException(status_code=503, detail={"status": "not ready"})
```

## Alerting

### Prometheus Alerts
```yaml
# alerts.yml
groups:
  - name: microservices
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} for {{ $labels.service }}"

      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "95th percentile latency is {{ $value }}s"

      - alert: ServiceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service is down"
          description: "{{ $labels.instance }} is down"

      - alert: DatabaseConnectionsHigh
        expr: database_connections_active > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High database connections"
          description: "Database has {{ $value }} active connections"
```

### Alertmanager конфигурация
```yaml
# alertmanager.yml
global:
  smtp_smarthost: 'localhost:587'
  smtp_from: 'alerts@example.com'

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'default'
  routes:
    - match:
        severity: critical
      receiver: 'critical-alerts'

receivers:
  - name: 'default'
    email_configs:
      - to: 'team@example.com'
        subject: '[Alert] {{ .GroupLabels.alertname }}'
        body: |
          {{ range .Alerts }}
          Alert: {{ .Annotations.summary }}
          Description: {{ .Annotations.description }}
          {{ end }}

  - name: 'critical-alerts'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/...'
        channel: '#alerts'
        title: 'Critical Alert'
        text: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
```

## Docker Compose для мониторинга

```yaml
# docker-compose.monitoring.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alerts.yml:/etc/prometheus/alerts.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-storage:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources

  alertmanager:
    image: prom/alertmanager:latest
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.5.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:8.5.0
    ports:
      - "5044:5044"
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf

  kibana:
    image: docker.elastic.co/kibana/kibana:8.5.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200

  filebeat:
    image: docker.elastic.co/beats/filebeat:8.5.0
    user: root
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro

volumes:
  grafana-storage:
  elasticsearch-data:
```

## Kubernetes мониторинг

```yaml
# k8s/monitoring.yaml
apiVersion: v1
kind: ServiceMonitor
metadata:
  name: user-service-monitor
spec:
  selector:
    matchLabels:
      app: user-service
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics

---
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: microservices-rules
spec:
  groups:
  - name: microservices
    rules:
    - alert: PodCrashLooping
      expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
      for: 5m
      annotations:
        summary: "Pod is crash looping"
        description: "Pod {{ $labels.pod }} is crash looping"
```

## Лучшие практики

### Мониторинг
1. **Golden Signals**: Latency, Traffic, Errors, Saturation
2. **SLI/SLO**: Определите Service Level Indicators и Objectives
3. **Alerting**: Alert на симптомы, а не причины
4. **Dashboards**: Создавайте понятные и полезные дашборды

### Логирование
1. **Structured logging**: JSON формат для легкого парсинга
2. **Correlation IDs**: Для трейсинга запросов
3. **Log levels**: Правильное использование уровней
4. **Sensitive data**: Не логируйте пароли и токены

### Health Checks
1. **Liveness**: Проверяет, жив ли сервис
2. **Readiness**: Проверяет, готов ли принимать трафик
3. **Startup**: Для медленно стартующих сервисов
4. **Dependencies**: Проверяйте критичные зависимости

## Заключение

Эффективный мониторинг включает:
- **Метрики** для количественной оценки
- **Логи** для диагностики проблем
- **Трейсинг** для анализа запросов
- **Алерты** для быстрого реагирования

## См. также
- [CI/CD и DevOps](./cicd-devops.md)
- [Безопасность](./security.md)
- [Микросервисная архитектура](./microservices-architecture.md) 