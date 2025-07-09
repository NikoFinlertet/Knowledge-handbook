# 📊 Monitoring & Logging: Мониторинг и логирование

## Описание

Комплексная система мониторинга и логирования для микросервисной архитектуры с использованием Prometheus, Grafana, ELK Stack и централизованного сбора метрик.

## Prometheus мониторинг

### Конфигурация Prometheus

```yaml
# monitoring/prometheus.yml
# Конфигурация Prometheus для мониторинга микросервисов

global:
  scrape_interval: 15s              # Интервал сбора метрик
  evaluation_interval: 15s          # Интервал оценки правил
  external_labels:
    cluster: 'production'           # Внешние метки
    datacenter: 'dc1'

# ==================== ПРАВИЛА АЛЕРТОВ ====================
rule_files:
  - "rules/*.yml"

# ==================== ALERTMANAGER ====================
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

# ==================== КОНФИГУРАЦИЯ СБОРА МЕТРИК ====================
scrape_configs:
  # Самомониторинг Prometheus
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  # Мониторинг Node Exporter (системные метрики)
  - job_name: 'node-exporter'
    static_configs:
      - targets: 
        - '10.0.1.20:9100'
        - '10.0.1.21:9100'
        - '10.0.1.22:9100'
    scrape_interval: 10s
    metrics_path: /metrics
  
  # User Service микросервис
  - job_name: 'user-service'
    static_configs:
      - targets:
        - '10.0.1.20:8081'
        - '10.0.1.21:8081'
        - '10.0.1.22:8081'
    metrics_path: /metrics
    scrape_interval: 10s
    scrape_timeout: 5s
  
  # Order Service микросервис
  - job_name: 'order-service'
    static_configs:
      - targets:
        - '10.0.1.20:8082'
        - '10.0.1.21:8082'
        - '10.0.1.22:8082'
    metrics_path: /metrics
    scrape_interval: 10s
  
  # API Gateway
  - job_name: 'api-gateway'
    static_configs:
      - targets:
        - '10.0.1.20:8080'
        - '10.0.1.21:8080'
        - '10.0.1.22:8080'
    metrics_path: /metrics
    scrape_interval: 5s             # Более частый сбор для gateway
  
  # PostgreSQL мониторинг
  - job_name: 'postgres-exporter'
    static_configs:
      - targets: ['10.0.1.30:9187']
    scrape_interval: 15s
  
  # Redis мониторинг
  - job_name: 'redis-exporter'
    static_configs:
      - targets: ['10.0.1.30:9121']
    scrape_interval: 15s
  
  # Nginx мониторинг
  - job_name: 'nginx-exporter'
    static_configs:
      - targets: ['10.0.1.10:9113']
    scrape_interval: 10s
```

### Правила алертов

```yaml
# monitoring/rules/microservices.yml
# Правила алертов для микросервисной архитектуры

groups:
  # ==================== ВЫСОКИЙ ПРИОРИТЕТ ====================
  - name: microservices.critical
    rules:
    # Сервис недоступен
    - alert: ServiceDown
      expr: up == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "Service {{ $labels.job }} is down"
        description: "Service {{ $labels.job }} on {{ $labels.instance }} has been down for more than 1 minute"
    
    # Высокая нагрузка на CPU
    - alert: HighCPUUsage
      expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "High CPU usage detected"
        description: "CPU usage is above 85% on {{ $labels.instance }} for more than 5 minutes"
    
    # Высокое использование памяти
    - alert: HighMemoryUsage
      expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
      for: 3m
      labels:
        severity: critical
      annotations:
        summary: "High memory usage detected"
        description: "Memory usage is above 90% on {{ $labels.instance }}"
  
  # ==================== СРЕДНИЙ ПРИОРИТЕТ ====================
  - name: microservices.warning
    rules:
    # Высокая частота ошибок HTTP
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.1
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: "High error rate detected"
        description: "Error rate is above 10% for service {{ $labels.job }}"
    
    # Медленные запросы
    - alert: HighResponseTime
      expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
      for: 3m
      labels:
        severity: warning
      annotations:
        summary: "High response time detected"
        description: "95th percentile response time is above 1s for {{ $labels.job }}"
    
    # Заполнение диска
    - alert: DiskSpaceRunningOut
      expr: (1 - (node_filesystem_free_bytes / node_filesystem_size_bytes)) * 100 > 80
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Disk space running out"
        description: "Disk usage is above 80% on {{ $labels.instance }} {{ $labels.mountpoint }}"
```

## Grafana дашборды

### Дашборд для микросервисов

```json
{
  "dashboard": {
    "title": "Microservices Monitoring",
    "panels": [
      {
        "title": "Service Health",
        "type": "stat",
        "targets": [
          {
            "expr": "up{job=~\".*-service|api-gateway\"}"
          }
        ]
      },
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])"
          }
        ]
      },
      {
        "title": "Response Time (95th percentile)",
        "type": "graph", 
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])"
          }
        ]
      }
    ]
  }
}
```

## ELK Stack для логирования

### Filebeat конфигурация

```yaml
# logging/filebeat.yml
# Конфигурация Filebeat для сбора логов

filebeat.inputs:
  # ==================== ЛОГИ ПРИЛОЖЕНИЙ ====================
  - type: log
    enabled: true
    paths:
      - /var/log/microservices/*.log
      - /var/log/microservices/*/*.log
    fields:
      log_type: application
      environment: production
    fields_under_root: true
    multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
    multiline.negate: true
    multiline.match: after
  
  # ==================== ЛОГИ NGINX ====================
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/access.log
      - /var/log/nginx/error.log
    fields:
      log_type: nginx
      service: load-balancer
  
  # ==================== СИСТЕМНЫЕ ЛОГИ ====================
  - type: log
    enabled: true
    paths:
      - /var/log/syslog
      - /var/log/auth.log
    fields:
      log_type: system

# ==================== ПРОЦЕССОРЫ ====================
processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_docker_metadata:
      host: "unix:///var/run/docker.sock"

# ==================== ВЫВОД В LOGSTASH ====================
output.logstash:
  hosts: ["logstash:5044"]

# ==================== НАСТРОЙКИ ЛОГИРОВАНИЯ ====================
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644
```

### Logstash конфигурация

```ruby
# logging/logstash.conf
# Конфигурация Logstash для обработки логов

input {
  beats {
    port => 5044
  }
}

filter {
  # ==================== ОБРАБОТКА ЛОГОВ ПРИЛОЖЕНИЙ ====================
  if [log_type] == "application" {
    grok {
      match => { 
        "message" => "%{TIMESTAMP_ISO8601:timestamp} \[%{DATA:thread}\] %{LOGLEVEL:level} %{DATA:logger} - %{GREEDYDATA:log_message}" 
      }
    }
    
    date {
      match => [ "timestamp", "yyyy-MM-dd HH:mm:ss.SSS" ]
    }
    
    # Извлечение дополнительной информации
    if [log_message] =~ /RequestId/ {
      grok {
        match => { 
          "log_message" => "RequestId:%{DATA:request_id}" 
        }
      }
    }
  }
  
  # ==================== ОБРАБОТКА ЛОГОВ NGINX ====================
  if [log_type] == "nginx" {
    grok {
      match => { 
        "message" => "%{NGINXACCESS}" 
      }
    }
    
    mutate {
      convert => { 
        "response" => "integer"
        "bytes" => "integer"
        "responsetime" => "float"
      }
    }
  }
  
  # ==================== ОБЩИЕ ФИЛЬТРЫ ====================
  mutate {
    add_field => { "[@metadata][index]" => "microservices-%{+YYYY.MM.dd}" }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "%{[@metadata][index]}"
    template_name => "microservices"
    template_pattern => "microservices-*"
  }
  
  # Дополнительный вывод для отладки
  if [level] == "ERROR" {
    stdout { codec => rubydebug }
  }
}
```

## Docker Compose для мониторинга

```yaml
# monitoring/docker-compose.yml
# Полный стек мониторинга и логирования

version: '3.8'

services:
  # ==================== PROMETHEUS ====================
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./rules:/etc/prometheus/rules
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
      - '--web.enable-lifecycle'
    restart: unless-stopped
  
  # ==================== GRAFANA ====================
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
      - GF_USERS_ALLOW_SIGN_UP=false
    restart: unless-stopped
  
  # ==================== ALERTMANAGER ====================
  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    restart: unless-stopped
  
  # ==================== ELASTICSEARCH ====================
  elasticsearch:
    image: elasticsearch:7.17.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    restart: unless-stopped
  
  # ==================== LOGSTASH ====================
  logstash:
    image: logstash:7.17.0
    container_name: logstash
    ports:
      - "5044:5044"
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch
    restart: unless-stopped
  
  # ==================== KIBANA ====================
  kibana:
    image: kibana:7.17.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch
    restart: unless-stopped

volumes:
  prometheus_data:
  grafana_data:
  elasticsearch_data:
```

## Скрипт мониторинга

```bash
#!/bin/bash
# scripts/monitoring-setup.sh
# Настройка полного стека мониторинга

echo "🔧 Настройка мониторинга и логирования..."

# ==================== СОЗДАНИЕ ДИРЕКТОРИЙ ====================
mkdir -p monitoring/{grafana/{dashboards,datasources},rules}
mkdir -p logging

# ==================== УСТАНОВКА NODE EXPORTER ====================
echo "📊 Установка Node Exporter..."
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-*-linux-amd64.tar.gz
tar xvfz node_exporter-*-linux-amd64.tar.gz
sudo mv node_exporter-*/node_exporter /usr/local/bin/
rm -rf node_exporter-*

# Создание systemd сервиса
sudo tee /etc/systemd/system/node_exporter.service << 'EOF'
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

# ==================== ЗАПУСК МОНИТОРИНГА ====================
echo "🚀 Запуск стека мониторинга..."
cd monitoring
docker-compose up -d

echo "✅ Мониторинг настроен!"
echo "📋 Доступные интерфейсы:"
echo "   Prometheus: http://localhost:9090"
echo "   Grafana: http://localhost:3000 (admin/admin123)"
echo "   Kibana: http://localhost:5601"
```

## Проверочные вопросы

1. **Какие типы метрик собирает Prometheus?**
2. **В чем разница между logs и metrics?**
3. **Как работает алертинг в Prometheus?**
4. **Зачем нужен Logstash в ELK стеке?**

## Практические задания

### Задание 1: Настройка базового мониторинга
- Установите Prometheus и Node Exporter
- Создайте простые алерты
- Настройте Grafana дашборд

### Задание 2: Логирование приложений
- Настройте ELK стек
- Добавьте парсинг логов микросервисов
- Создайте дашборд в Kibana

### Задание 3: Комплексный мониторинг
- Интегрируйте все компоненты
- Настройте алертинг по email/Slack
- Создайте SLA дашборды

## Связанные темы

- [[server-setup]] - Подготовка серверов
- [[kubernetes-deployment]] - Мониторинг в K8s
- [[automation-scripts]] - Автоматизация мониторинга 