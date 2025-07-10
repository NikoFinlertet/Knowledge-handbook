# 🔧 Ansible Automation: Автоматизация развертывания

## Описание

Ansible - это инструмент автоматизации IT-инфраструктуры, позволяющий управлять конфигурациями серверов, развертывать приложения и оркестрировать сложные IT-процессы без агентов.

## Структура проекта Ansible

### Организация файлов

```
deployment/
├── ansible/
│   ├── inventory/
│   │   ├── production/
│   │   │   ├── hosts.yml           # Продакшн серверы
│   │   │   └── group_vars/         # Переменные для групп
│   │   │       ├── all.yml         # Общие переменные
│   │   │       ├── webservers.yml  # Переменные веб-серверов
│   │   │       └── databases.yml   # Переменные БД
│   │   └── staging/
│   │       ├── hosts.yml           # Тестовые серверы
│   │       └── group_vars/
│   ├── playbooks/
│   │   ├── deploy.yml              # Основной playbook развертывания
│   │   ├── setup.yml               # Первоначальная настройка
│   │   ├── rollback.yml            # Откат версии
│   │   └── update.yml              # Обновление компонентов
│   ├── roles/
│   │   ├── docker/                 # Роль установки Docker
│   │   ├── nginx/                  # Роль настройки Nginx
│   │   ├── monitoring/             # Роль мониторинга
│   │   ├── postgresql/             # Роль PostgreSQL
│   │   └── microservices/          # Роль микросервисов
│   ├── group_vars/
│   ├── host_vars/
│   └── ansible.cfg                 # Конфигурация Ansible
```

## Inventory файлы

### Production inventory

```yaml
# deployment/ansible/inventory/production/hosts.yml
# Инвентарь продакшн серверов с группировкой по ролям

all:
  children:
    # ==================== БАЛАНСИРОВЩИКИ НАГРУЗКИ ====================
    loadbalancers:
      hosts:
        lb1:
          ansible_host: 10.0.1.10          # IP-адрес первого балансировщика
          ansible_user: deploy             # Пользователь для подключения
          nginx_role: primary              # Основной балансировщик
        lb2:
          ansible_host: 10.0.1.11          # IP-адрес резервного балансировщика
          ansible_user: deploy
          nginx_role: backup               # Резервный балансировщик
    
    # ==================== ВЕБ-СЕРВЕРЫ ====================
    webservers:
      hosts:
        web1:
          ansible_host: 10.0.1.20          # Первый веб-сервер
          ansible_user: deploy
          server_id: 1                     # Уникальный идентификатор
        web2:
          ansible_host: 10.0.1.21          # Второй веб-сервер
          ansible_user: deploy
          server_id: 2
        web3:
          ansible_host: 10.0.1.22          # Третий веб-сервер
          ansible_user: deploy
          server_id: 3
    
    # ==================== БАЗЫ ДАННЫХ ====================
    databases:
      hosts:
        db1:
          ansible_host: 10.0.1.30          # Основная БД
          ansible_user: deploy
          postgresql_role: master          # Мастер-сервер
          postgresql_port: 5432
        db2:
          ansible_host: 10.0.1.31          # Реплика БД
          ansible_user: deploy
          postgresql_role: replica         # Сервер-реплика
          postgresql_port: 5432
    
    # ==================== МОНИТОРИНГ ====================
    monitoring:
      hosts:
        monitor1:
          ansible_host: 10.0.1.40          # Сервер мониторинга
          ansible_user: deploy
          monitoring_role: all             # Все компоненты мониторинга

  # ==================== ГЛОБАЛЬНЫЕ ПЕРЕМЕННЫЕ ====================
  vars:
    # SSH настройки
    ansible_ssh_private_key_file: ~/.ssh/deploy_key     # Путь к приватному ключу
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no'  # Отключение проверки host key
    
    # Общие настройки приложения
    app_environment: production         # Окружение
    app_domain: api.mycompany.com      # Доменное имя
    app_ssl_enabled: true              # Включение SSL
    
    # Docker registry
    docker_registry: registry.mycompany.com  # Адрес приватного registry
    docker_registry_user: deploy             # Пользователь registry
```

### Staging inventory

```yaml
# deployment/ansible/inventory/staging/hosts.yml
# Инвентарь тестового окружения

all:
  children:
    webservers:
      hosts:
        staging-web1:
          ansible_host: 10.0.2.20
          ansible_user: deploy
          server_id: 1
    
    databases:
      hosts:
        staging-db1:
          ansible_host: 10.0.2.30
          ansible_user: deploy
          postgresql_role: master

  vars:
    ansible_ssh_private_key_file: ~/.ssh/deploy_key
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
    app_environment: staging
    app_domain: staging-api.mycompany.com
    app_ssl_enabled: false
    docker_registry: registry.mycompany.com
```

## Основной playbook развертывания

```yaml
# deployment/ansible/playbooks/deploy.yml
# Главный playbook для развертывания микросервисной архитектуры

---
- name: Deploy Microservices Application
  hosts: all
  become: yes                          # Выполнение с правами sudo
  gather_facts: yes                    # Сбор информации о системе
  vars:
    app_version: "{{ version | default('latest') }}"     # Версия приложения
    environment: "{{ env | default('production') }}"     # Окружение
    
  # ==================== ПРЕДВАРИТЕЛЬНЫЕ ЗАДАЧИ ====================
  pre_tasks:
    - name: Update system packages (Debian/Ubuntu)
      apt:
        update_cache: yes              # Обновление кэша пакетов
        cache_valid_time: 3600         # Кэш действителен 1 час
      when: ansible_os_family == "Debian"  # Только для Debian/Ubuntu
    
    - name: Update system packages (RedHat/CentOS)
      yum:
        name: "*"
        state: latest
      when: ansible_os_family == "RedHat"   # Только для RedHat/CentOS
    
    - name: Create application directories
      file:
        path: "{{ item }}"
        state: directory
        owner: deploy
        group: deploy
        mode: '0755'
      loop:
        - /opt/microservices           # Основная директория
        - /opt/microservices/data      # Данные приложений
        - /opt/microservices/logs      # Логи
        - /opt/microservices/config    # Конфигурации

  # ==================== ПРИМЕНЕНИЕ РОЛЕЙ ====================
  roles:
    - docker                          # Установка Docker
    - nginx                           # Настройка Nginx
    - microservices                   # Развертывание микросервисов
    - monitoring                      # Настройка мониторинга
  
  # ==================== ПРОВЕРКА ПОСЛЕ РАЗВЕРТЫВАНИЯ ====================
  post_tasks:
    - name: Verify deployment
      uri:
        url: "http://{{ ansible_default_ipv4.address }}:8080/health"
        method: GET
        status_code: 200
      retries: 5                       # 5 попыток проверки
      delay: 10                        # Пауза 10 сек между попытками
      delegate_to: localhost           # Выполнение с локальной машины

# ==================== НАСТРОЙКА БАЛАНСИРОВЩИКОВ ====================
- name: Configure Load Balancers
  hosts: loadbalancers
  become: yes
  roles:
    - nginx-lb                         # Роль для балансировщика нагрузки
  
  post_tasks:
    - name: Verify load balancer
      uri:
        url: "http://{{ ansible_default_ipv4.address }}/health"
        method: GET
        status_code: 200
      retries: 3
      delay: 5
      delegate_to: localhost

# ==================== НАСТРОЙКА КЛАСТЕРА БАЗ ДАННЫХ ====================
- name: Setup Database Cluster
  hosts: databases
  become: yes
  roles:
    - postgresql                       # Роль PostgreSQL
  
  post_tasks:
    - name: Verify database connection
      postgresql_ping:
        host: "{{ ansible_default_ipv4.address }}"
        port: 5432
        database: microservices_db
        username: postgres
        password: "{{ postgresql_password }}"
      delegate_to: localhost

# ==================== РАЗВЕРТЫВАНИЕ МОНИТОРИНГА ====================
- name: Deploy Monitoring Stack
  hosts: monitoring
  become: yes
  roles:
    - prometheus                       # Система метрик
    - grafana                         # Дашборды
    - alertmanager                    # Алертинг
  
  post_tasks:
    - name: Verify monitoring endpoints
      uri:
        url: "http://{{ ansible_default_ipv4.address }}:{{ item }}"
        method: GET
        status_code: 200
      loop:
        - 9090  # Prometheus
        - 3000  # Grafana
        - 9093  # Alertmanager
      retries: 3
      delay: 10
      delegate_to: localhost
```

## Роль для микросервисов

### Основные задачи роли

```yaml
# deployment/ansible/roles/microservices/tasks/main.yml
# Задачи для развертывания микросервисов

---
# ==================== ЗАГРУЗКА DOCKER ОБРАЗОВ ====================
- name: Pull latest Docker images
  docker_image:
    name: "{{ item }}"
    source: pull                       # Загрузка из registry
    force_source: yes                  # Принудительная загрузка
  loop:
    - "{{ registry }}/user-service:{{ app_version }}"
    - "{{ registry }}/order-service:{{ app_version }}"
    - "{{ registry }}/api-gateway:{{ app_version }}"
  tags: [pull]

# ==================== СОЗДАНИЕ КОНФИГУРАЦИЙ ====================
- name: Create application config
  template:
    src: "{{ item.src }}"
    dest: "/opt/microservices/config/{{ item.dest }}"
    owner: deploy
    group: deploy
    mode: '0644'
  loop:
    - { src: docker-compose.yml.j2, dest: docker-compose.yml }
    - { src: .env.j2, dest: .env }
    - { src: nginx.conf.j2, dest: nginx.conf }
  notify: restart services             # Вызов handler для перезапуска
  tags: [config]

# ==================== СОЗДАНИЕ SYSTEMD СЕРВИСА ====================
- name: Create systemd service file
  template:
    src: microservices.service.j2
    dest: /etc/systemd/system/microservices.service
    owner: root
    group: root
    mode: '0644'
  notify: 
    - reload systemd                   # Перезагрузка systemd
    - restart services                 # Перезапуск сервисов
  tags: [systemd]

# ==================== ЗАПУСК МИКРОСЕРВИСОВ ====================
- name: Start and enable microservices
  systemd:
    name: microservices
    state: started                     # Запуск сервиса
    enabled: yes                       # Автозапуск при загрузке
    daemon_reload: yes                 # Перезагрузка конфигурации
  tags: [start]

# ==================== ПРОВЕРКА РАБОТОСПОСОБНОСТИ ====================
- name: Wait for services to be ready
  wait_for:
    port: "{{ item }}"
    host: "{{ ansible_default_ipv4.address }}"
    delay: 10                          # Начальная задержка
    timeout: 300                       # Максимальное время ожидания
  loop:
    - 8080  # API Gateway
    - 8081  # User Service
    - 8082  # Order Service
  tags: [health]

# ==================== НАСТРОЙКА ЛОГРОТАЦИИ ====================
- name: Setup log rotation
  template:
    src: logrotate.j2
    dest: /etc/logrotate.d/microservices
    owner: root
    group: root
    mode: '0644'
  tags: [logs]
```

### Handlers роли

```yaml
# deployment/ansible/roles/microservices/handlers/main.yml
# Обработчики событий для роли микросервисов

---
- name: reload systemd
  systemd:
    daemon_reload: yes                 # Перезагрузка systemd daemon

- name: restart services
  systemd:
    name: microservices
    state: restarted                   # Перезапуск сервисов

- name: restart nginx
  systemd:
    name: nginx
    state: restarted                   # Перезапуск Nginx

- name: reload nginx
  systemd:
    name: nginx
    state: reloaded                    # Перезагрузка конфигурации Nginx
```

## Шаблоны конфигураций

### Docker Compose для продакшена

```yaml
# deployment/ansible/roles/microservices/templates/docker-compose.yml.j2
# Шаблон Docker Compose для продакшн развертывания

version: '3.8'

# ==================== СЕТИ ====================
networks:
  microservices:
    driver: bridge                     # Мостовая сеть
    ipam:
      config:
        - subnet: 172.20.0.0/16       # Подсеть для контейнеров

# ==================== ТОМА ====================
volumes:
  postgres_data:
    driver: local                      # Локальное хранилище для PostgreSQL
  redis_data:
    driver: local                      # Локальное хранилище для Redis
  logs_data:
    driver: local                      # Общие логи приложений

# ==================== СЕРВИСЫ ====================
services:
  # ==================== POSTGRESQL ====================
  postgres:
    image: postgres:15-alpine
    container_name: postgres
    environment:
      POSTGRES_DB: {{ postgresql_database }}
      POSTGRES_USER: {{ postgresql_user }}
      POSTGRES_PASSWORD: {{ postgresql_password }}
      POSTGRES_INITDB_ARGS: "--encoding=UTF8 --locale=C"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - microservices
    restart: unless-stopped            # Автоматический перезапуск
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U {{ postgresql_user }}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s               # Время на первоначальную загрузку
    deploy:
      resources:
        limits:
          memory: 512M                # Лимит памяти
          cpus: '0.5'                 # Лимит CPU
        reservations:
          memory: 256M                # Резерв памяти
          cpus: '0.25'                # Резерв CPU

  # ==================== REDIS ====================
  redis:
    image: redis:7-alpine
    container_name: redis
    command: redis-server --appendonly yes --requirepass {{ redis_password }}
    volumes:
      - redis_data:/data
    networks:
      - microservices
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "--no-auth-warning", "-a", "{{ redis_password }}", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          memory: 256M
          cpus: '0.3'

  # ==================== USER SERVICE ====================
  user-service:
    image: {{ registry }}/user-service:{{ app_version }}
    container_name: user-service
    environment:
      - DATABASE_URL=postgres://{{ postgresql_user }}:{{ postgresql_password }}@postgres:5432/{{ postgresql_database }}
      - REDIS_URL=redis://:{{ redis_password }}@redis:6379
      - JWT_SECRET={{ jwt_secret }}
      - LOG_LEVEL={{ log_level }}
      - ENVIRONMENT={{ environment }}
      - SERVER_ID={{ server_id }}
    depends_on:
      postgres:
        condition: service_healthy     # Ждать готовности PostgreSQL
      redis:
        condition: service_healthy     # Ждать готовности Redis
    networks:
      - microservices
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8081/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s
    logging:
      driver: json-file
      options:
        max-size: "10m"               # Максимальный размер лог-файла
        max-file: "3"                 # Количество ротируемых файлов
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'

  # ==================== ORDER SERVICE ====================
  order-service:
    image: {{ registry }}/order-service:{{ app_version }}
    container_name: order-service
    environment:
      - DATABASE_URL=postgres://{{ postgresql_user }}:{{ postgresql_password }}@postgres:5432/{{ postgresql_database }}
      - USER_SERVICE_URL=http://user-service:8081
      - REDIS_URL=redis://:{{ redis_password }}@redis:6379
      - LOG_LEVEL={{ log_level }}
      - ENVIRONMENT={{ environment }}
      - SERVER_ID={{ server_id }}
    depends_on:
      postgres:
        condition: service_healthy
      user-service:
        condition: service_healthy     # Зависимость от User Service
    networks:
      - microservices
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8082/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'

  # ==================== API GATEWAY ====================
  api-gateway:
    image: {{ registry }}/api-gateway:{{ app_version }}
    container_name: api-gateway
    environment:
      - USER_SERVICE_URL=http://user-service:8081
      - ORDER_SERVICE_URL=http://order-service:8082
      - REDIS_URL=redis://:{{ redis_password }}@redis:6379
      - LOG_LEVEL={{ log_level }}
      - ENVIRONMENT={{ environment }}
      - SERVER_ID={{ server_id }}
    ports:
      - "8080:8080"                   # Внешний порт для доступа
    depends_on:
      user-service:
        condition: service_healthy
      order-service:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - microservices
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
```

## Переменные окружений

### Group variables

```yaml
# deployment/ansible/inventory/production/group_vars/all.yml
# Общие переменные для всех серверов продакшн окружения

# ==================== ПРИЛОЖЕНИЕ ====================
app_name: microservices-app
app_version: "{{ version | default('latest') }}"
app_environment: production
app_user: deploy
app_group: deploy

# ==================== ДОМЕН И SSL ====================
domain_name: api.mycompany.com
ssl_enabled: true
ssl_cert_email: admin@mycompany.com

# ==================== DOCKER REGISTRY ====================
registry: registry.mycompany.com
registry_user: deploy
registry_password: "{{ vault_registry_password }}"

# ==================== БАЗА ДАННЫХ ====================
postgresql_version: "15"
postgresql_database: microservices_db
postgresql_user: microservices_user
postgresql_password: "{{ vault_postgresql_password }}"
postgresql_max_connections: 200

# ==================== REDIS ====================
redis_version: "7"
redis_password: "{{ vault_redis_password }}"
redis_maxmemory: 256mb

# ==================== ЛОГИРОВАНИЕ ====================
log_level: info
log_retention_days: 30

# ==================== БЕЗОПАСНОСТЬ ====================
jwt_secret: "{{ vault_jwt_secret }}"
encryption_key: "{{ vault_encryption_key }}"

# ==================== МОНИТОРИНГ ====================
monitoring_enabled: true
metrics_retention: 30d
alerting_enabled: true
```

### Vault переменные (зашифрованные)

```yaml
# deployment/ansible/inventory/production/group_vars/vault.yml
# Зашифрованные переменные (создать с помощью ansible-vault)

$ANSIBLE_VAULT;1.1;AES256
66623034323264623537653936653961643334613834623436616234646137313664663765396364
3136616462383966393964623764336235623538343539390a346466643833323235373433363764
36356136623138643437396164646564343333323636326635653865616634313166643030633831
6664623566343662320a626535616237376234313434353233336361313137623737393962646464
...
# Для создания vault файла:
# ansible-vault create group_vars/vault.yml
# ansible-vault edit group_vars/vault.yml
```

## Запуск развертывания

### Команды для развертывания

```bash
#!/bin/bash
# scripts/ansible-deploy.sh
# Скрипт запуска Ansible развертывания

# ==================== ПРОВЕРКА ОКРУЖЕНИЯ ====================
ENVIRONMENT=${1:-production}
VERSION=${2:-latest}
VAULT_PASSWORD_FILE=${3:-~/.ansible_vault_pass}

if [ ! -f "$VAULT_PASSWORD_FILE" ]; then
    echo "❌ Файл с паролем vault не найден: $VAULT_PASSWORD_FILE"
    exit 1
fi

# ==================== ПРОВЕРКА ПОДКЛЮЧЕНИЯ ====================
echo "🔍 Проверка подключения к серверам..."
ansible -i inventory/$ENVIRONMENT/hosts.yml all -m ping \
    --vault-password-file="$VAULT_PASSWORD_FILE"

if [ $? -ne 0 ]; then
    echo "❌ Ошибка подключения к серверам"
    exit 1
fi

# ==================== РАЗВЕРТЫВАНИЕ ====================
echo "🚀 Запуск развертывания на $ENVIRONMENT..."
ansible-playbook -i inventory/$ENVIRONMENT/hosts.yml \
    playbooks/deploy.yml \
    --vault-password-file="$VAULT_PASSWORD_FILE" \
    --extra-vars "version=$VERSION env=$ENVIRONMENT" \
    --diff \
    --check

# Подтверждение развертывания
read -p "Продолжить развертывание? (y/N): " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
    ansible-playbook -i inventory/$ENVIRONMENT/hosts.yml \
        playbooks/deploy.yml \
        --vault-password-file="$VAULT_PASSWORD_FILE" \
        --extra-vars "version=$VERSION env=$ENVIRONMENT"
    
    echo "✅ Развертывание завершено!"
else
    echo "❌ Развертывание отменено"
fi
```

## Проверочные вопросы

1. **Что такое Ansible inventory и как он организован?**
2. **В чем разница между roles и playbooks?**
3. **Как работают handlers в Ansible?**
4. **Зачем нужен ansible-vault?**
5. **Как проверить изменения перед применением?**

## Практические задания

### Задание 1: Настройка inventory
- Создайте inventory для тестового окружения
- Добавьте группы серверов и переменные
- Протестируйте подключение с помощью ping

### Задание 2: Создание роли
- Создайте роль для установки и настройки Nginx
- Добавьте templates для конфигурации
- Напишите handlers для перезапуска

### Задание 3: Автоматизация развертывания
- Создайте playbook для полного развертывания
- Добавьте проверки работоспособности
- Настройте rollback механизм

## Связанные темы

- [[server-setup]] - Подготовка серверов
- [[nginx-load-balancer]] - Настройка Nginx
- [[kubernetes-deployment]] - Развертывание в K8s
- [[monitoring-logging]] - Мониторинг инфраструктуры 