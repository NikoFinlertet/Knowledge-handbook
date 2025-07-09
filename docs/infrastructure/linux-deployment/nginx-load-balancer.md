# ⚖️ Nginx Load Balancer: Балансировка нагрузки

## Описание

Nginx Load Balancer обеспечивает распределение входящих запросов между несколькими экземплярами API Gateway, повышая отказоустойчивость и производительность системы.

## Конфигурация Nginx Load Balancer

### Основная конфигурация

```nginx
# /etc/nginx/sites-available/api-loadbalancer
# Конфигурация Nginx как Load Balancer для API Gateway

# ==================== ОПРЕДЕЛЕНИЕ UPSTREAM СЕРВЕРОВ ====================
# Группа серверов API Gateway для балансировки нагрузки
upstream api_gateway {
    # Алгоритм балансировки: least_conn = направляет запросы к серверу с наименьшим количеством активных соединений
    least_conn;
    
    # Сервер 1: max_fails=3 означает, что после 3 неудачных попыток сервер считается недоступным
    # fail_timeout=30s - время, в течение которого сервер считается недоступным
    server 10.0.1.20:8080 max_fails=3 fail_timeout=30s weight=1;
    
    # Сервер 2: аналогичные настройки для второго сервера
    server 10.0.1.21:8080 max_fails=3 fail_timeout=30s weight=1;
    
    # Сервер 3: третий сервер в кластере
    server 10.0.1.22:8080 max_fails=3 fail_timeout=30s weight=1;
    
    # Резервный сервер (backup) - используется только если основные недоступны
    server 10.0.1.23:8080 backup;
    
    # Keepalive соединения для улучшения производительности
    keepalive 32;
}

# Upstream для статических файлов (если есть)
upstream static_servers {
    server 10.0.1.20:8080;
    server 10.0.1.21:8080;
    server 10.0.1.22:8080;
    keepalive 16;
}

# ==================== НАСТРОЙКА ЛИМИТОВ ====================
# Зоны для ограничения скорости запросов
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;        # Общий API лимит
limit_req_zone $binary_remote_addr zone=login:10m rate=5r/s;       # Лимит для авторизации
limit_req_zone $binary_remote_addr zone=register:10m rate=2r/s;    # Лимит для регистрации

# Лимит на количество соединений
limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m;

# ==================== HTTP СЕРВЕР ====================
server {
    listen 80;                      # Прослушивание HTTP порта
    server_name api.mycompany.com;  # Доменное имя

    # ==================== ЗАГОЛОВКИ БЕЗОПАСНОСТИ ====================
    # Защита от атак через HTTP заголовки
    add_header X-Frame-Options "SAMEORIGIN" always;              # Защита от clickjacking
    add_header X-Content-Type-Options "nosniff" always;          # Защита от MIME-type sniffing
    add_header X-XSS-Protection "1; mode=block" always;          # Защита от XSS атак
    add_header Referrer-Policy "strict-origin-when-cross-origin" always; # Контроль Referer заголовка
    add_header X-Real-IP $remote_addr always;                    # Реальный IP клиента

    # ==================== СЖАТИЕ GZIP ====================
    # Настройка сжатия для улучшения производительности
    gzip on;                        # Включение сжатия
    gzip_vary on;                   # Добавление заголовка Vary: Accept-Encoding
    gzip_comp_level 6;              # Уровень сжатия (1-9, где 6 - оптимальный баланс)
    gzip_min_length 1000;           # Минимальный размер файла для сжатия
    gzip_proxied any;               # Сжатие для всех прокси-запросов
    gzip_types                      # Типы файлов для сжатия
        text/plain                  # Обычный текст
        text/css                    # CSS стили
        text/xml                    # XML документы
        text/javascript             # JavaScript код
        application/json            # JSON данные
        application/javascript      # JavaScript приложения
        application/xml+rss         # RSS фиды
        application/atom+xml        # Atom фиды
        image/svg+xml;              # SVG изображения

    # ==================== HEALTH CHECK ENDPOINT ====================
    # Эндпоинт для проверки работоспособности балансировщика
    location /health {
        access_log off;                     # Отключение логирования для health check
        return 200 "healthy\n";            # Возврат статуса 200 с текстом "healthy"
        add_header Content-Type text/plain; # Установка типа содержимого
    }

    # ==================== СПЕЦИАЛЬНЫЕ ЭНДПОИНТЫ С ЛИМИТАМИ ====================
    # Авторизация с усиленными лимитами
    location /api/auth/login {
        limit_req zone=login burst=10 nodelay;              # Лимит для логина
        limit_conn conn_limit_per_ip 5;                     # Максимум 5 соединений с IP
        
        proxy_pass http://api_gateway;
        include /etc/nginx/proxy_params;
    }
    
    # Регистрация с еще более строгими лимитами
    location /api/auth/register {
        limit_req zone=register burst=5 nodelay;            # Лимит для регистрации
        limit_conn conn_limit_per_ip 3;                     # Максимум 3 соединения с IP
        
        proxy_pass http://api_gateway;
        include /etc/nginx/proxy_params;
    }

    # ==================== ПРОКСИРОВАНИЕ API ЗАПРОСОВ ====================
    # Основной маршрут для API запросов
    location /api/ {
        # Применение лимитов
        limit_req zone=api burst=20 nodelay;                # Применение лимита с burst=20 запросов
        limit_conn conn_limit_per_ip 10;                    # Максимум 10 соединений с IP
        
        # Проксирование к upstream группе
        proxy_pass http://api_gateway;
        
        # Включение файла с общими настройками прокси
        include /etc/nginx/proxy_params;
        
        # Дополнительные заголовки для API
        proxy_set_header X-Request-ID $request_id;          # Уникальный идентификатор запроса
        proxy_set_header X-Forwarded-Proto $scheme;         # Протокол (http/https)
    }

    # ==================== СТАТИЧЕСКИЕ ФАЙЛЫ ====================
    # Обслуживание статических файлов (если есть)
    location /static/ {
        proxy_pass http://static_servers;
        include /etc/nginx/proxy_params;
        
        # Кэширование статики
        expires 1y;                                         # Кэширование на 1 год
        add_header Cache-Control "public, no-transform";    # Настройки кэширования
        
        # Отключение логирования для статики
        access_log off;
    }

    # ==================== WEBSOCKET SUPPORT ====================
    # Поддержка WebSocket соединений
    location /ws/ {
        proxy_pass http://api_gateway;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Таймауты для WebSocket
        proxy_read_timeout 86400;                          # 24 часа
        proxy_send_timeout 86400;
    }

    # ==================== РЕДИРЕКТ НА HTTPS ====================
    # Постоянный редирект (301) всех HTTP запросов на HTTPS
    return 301 https://$server_name$request_uri;
}

# ==================== HTTPS СЕРВЕР ====================
server {
    listen 443 ssl http2;          # Прослушивание HTTPS порта с поддержкой HTTP/2
    server_name api.mycompany.com;  # Доменное имя

    # ==================== SSL СЕРТИФИКАТЫ ====================
    # Пути к SSL сертификатам от Let's Encrypt
    ssl_certificate /etc/letsencrypt/live/api.mycompany.com/fullchain.pem;      # Полная цепочка сертификатов
    ssl_certificate_key /etc/letsencrypt/live/api.mycompany.com/privkey.pem;    # Приватный ключ
    ssl_trusted_certificate /etc/letsencrypt/live/api.mycompany.com/chain.pem;  # Цепочка доверенных сертификатов

    # ==================== НАСТРОЙКИ SSL СЕССИЙ ====================
    ssl_session_timeout 1d;        # Время жизни SSL сессии
    ssl_session_cache shared:SSL:50m; # Кэш SSL сессий (50MB shared между workers)
    ssl_session_tickets off;       # Отключение SSL session tickets для безопасности

    # ==================== СОВРЕМЕННАЯ SSL КОНФИГУРАЦИЯ ====================
    ssl_protocols TLSv1.2 TLSv1.3; # Поддержка только современных протоколов
    # Поддержка только безопасных cipher suites
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off; # Позволить клиенту выбирать предпочтительный cipher

    # ==================== HSTS (HTTP Strict Transport Security) ====================
    # Принудительное использование HTTPS в будущем
    add_header Strict-Transport-Security "max-age=63072000" always; # 2 года

    # ==================== ВКЛЮЧЕНИЕ ОБЩИХ НАСТРОЕК ====================
    # Подключение всех location блоков из HTTP сервера
    include /etc/nginx/sites-available/api_common.conf;
}
```

### Общие настройки прокси

```nginx
# /etc/nginx/proxy_params
# Общие параметры для проксирования

# ==================== ОСНОВНЫЕ ЗАГОЛОВКИ ====================
proxy_set_header Host $http_host;                           # Оригинальный хост
proxy_set_header X-Real-IP $remote_addr;                    # Реальный IP клиента
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # Цепочка прокси серверов
proxy_set_header X-Forwarded-Proto $scheme;                 # Протокол (http/https)

# ==================== НАСТРОЙКИ ТАЙМАУТОВ ====================
proxy_connect_timeout 5s;          # Таймаут подключения к upstream серверу
proxy_send_timeout 60s;            # Таймаут отправки данных
proxy_read_timeout 60s;            # Таймаут чтения ответа

# ==================== НАСТРОЙКИ БУФЕРИЗАЦИИ ====================
proxy_buffer_size 4k;              # Размер буфера для заголовков ответа
proxy_buffers 8 4k;                # Количество и размер буферов для тела ответа
proxy_busy_buffers_size 8k;        # Размер буферов для отправки клиенту
proxy_temp_file_write_size 8k;     # Размер блоков записи во временные файлы

# ==================== ОБРАБОТКА ОШИБОК UPSTREAM ====================
# Автоматическое переключение на другой сервер при ошибках
proxy_next_upstream error timeout invalid_header http_502 http_503 http_504;
proxy_next_upstream_tries 3;       # Максимальное количество попыток переключения
proxy_next_upstream_timeout 10s;   # Общий таймаут для попыток

# ==================== KEEPALIVE ====================
proxy_http_version 1.1;            # Использование HTTP/1.1 для keepalive
proxy_set_header Connection "";    # Очистка заголовка Connection для keepalive
```

## SSL сертификаты с Let's Encrypt

### Скрипт установки SSL

```bash
#!/bin/bash
# scripts/setup-ssl.sh
# Автоматическая настройка SSL сертификатов с Let's Encrypt

# ==================== ПАРАМЕТРЫ ====================
DOMAIN="$1"
EMAIL="$2"
WEBROOT_PATH="/var/www/html"

# Проверка параметров
if [ -z "$DOMAIN" ] || [ -z "$EMAIL" ]; then
    echo "❌ Использование: $0 <domain> <email>"
    echo "   Пример: $0 api.mycompany.com admin@mycompany.com"
    exit 1
fi

echo "🔒 Настройка SSL сертификата для $DOMAIN..."

# ==================== УСТАНОВКА CERTBOT ====================
echo "📦 Установка Certbot..."

# Обновление пакетов
sudo apt update

# Установка Certbot и плагина для Nginx
sudo apt install -y certbot python3-certbot-nginx

# ==================== ПРОВЕРКА NGINX ====================
echo "🔍 Проверка конфигурации Nginx..."

# Проверка синтаксиса конфигурации
if ! sudo nginx -t; then
    echo "❌ Ошибка в конфигурации Nginx. Исправьте ошибки перед продолжением."
    exit 1
fi

# ==================== СОЗДАНИЕ WEBROOT ====================
echo "📁 Создание webroot директории..."

# Создание директории для webroot метода верификации
sudo mkdir -p "$WEBROOT_PATH"
sudo chown -R www-data:www-data "$WEBROOT_PATH"

# ==================== ВРЕМЕННАЯ КОНФИГУРАЦИЯ ДЛЯ ВЕРИФИКАЦИИ ====================
echo "⚙️ Создание временной конфигурации для верификации..."

sudo tee /etc/nginx/sites-available/temp-ssl-$DOMAIN << EOF
server {
    listen 80;
    server_name $DOMAIN;
    
    location /.well-known/acme-challenge/ {
        root $WEBROOT_PATH;
    }
    
    location / {
        return 301 https://\$server_name\$request_uri;
    }
}
EOF

# Активация временной конфигурации
sudo ln -sf /etc/nginx/sites-available/temp-ssl-$DOMAIN /etc/nginx/sites-enabled/
sudo nginx -s reload

# ==================== ПОЛУЧЕНИЕ СЕРТИФИКАТА ====================
echo "🔐 Получение SSL сертификата..."

# Получение сертификата с помощью webroot метода
sudo certbot certonly \
    --webroot \
    --webroot-path="$WEBROOT_PATH" \
    --email "$EMAIL" \
    --agree-tos \
    --no-eff-email \
    --non-interactive \
    --domains "$DOMAIN"

# Проверка успешности получения сертификата
if [ $? -eq 0 ]; then
    echo "✅ SSL сертификат успешно получен!"
else
    echo "❌ Ошибка при получении SSL сертификата"
    exit 1
fi

# ==================== НАСТРОЙКА АВТОМАТИЧЕСКОГО ОБНОВЛЕНИЯ ====================
echo "⏰ Настройка автоматического обновления..."

# Создание скрипта для обновления
sudo tee /etc/cron.daily/certbot-renew << 'EOF'
#!/bin/bash
# Автоматическое обновление SSL сертификатов

# Попытка обновления всех сертификатов
/usr/bin/certbot renew --quiet --deploy-hook "systemctl reload nginx"

# Очистка старых сертификатов
/usr/bin/certbot delete --cert-name $(certbot certificates | grep "INVALID" | cut -d' ' -f5) 2>/dev/null
EOF

# Установка прав на выполнение
sudo chmod +x /etc/cron.daily/certbot-renew

# Альтернативный способ через crontab (если нет cron.daily)
(sudo crontab -l 2>/dev/null | grep -v certbot; echo "0 2 * * * /usr/bin/certbot renew --quiet --deploy-hook 'systemctl reload nginx'") | sudo crontab -

# ==================== ФИНАЛЬНАЯ КОНФИГУРАЦИЯ ====================
echo "🔧 Применение финальной конфигурации..."

# Удаление временной конфигурации
sudo rm -f /etc/nginx/sites-enabled/temp-ssl-$DOMAIN
sudo rm -f /etc/nginx/sites-available/temp-ssl-$DOMAIN

# Активация основной конфигурации с SSL
sudo ln -sf /etc/nginx/sites-available/api-loadbalancer /etc/nginx/sites-enabled/

# Проверка конфигурации
if sudo nginx -t; then
    sudo systemctl reload nginx
    echo "✅ Nginx перезагружен с новой SSL конфигурацией"
else
    echo "❌ Ошибка в SSL конфигурации Nginx"
    exit 1
fi

# ==================== ПРОВЕРКА SSL ====================
echo "🧪 Проверка SSL сертификата..."

# Проверка сертификата
if curl -I "https://$DOMAIN" 2>/dev/null | grep -q "HTTP/"; then
    echo "✅ SSL сертификат работает корректно!"
    
    # Показать информацию о сертификате
    echo "📄 Информация о сертификате:"
    echo "   Домен: $DOMAIN"
    echo "   Выдан: $(sudo certbot certificates | grep -A 2 "$DOMAIN" | grep "Expiry Date" | cut -d: -f2-)"
    echo "   Автообновление: Настроено"
else
    echo "⚠️ SSL сертификат получен, но может потребоваться время для распространения DNS"
fi

echo ""
echo "🎉 Настройка SSL завершена!"
echo "📝 Следующие шаги:"
echo "   1. Проверьте доступность https://$DOMAIN"
echo "   2. Обновите DNS записи если необходимо"
echo "   3. Протестируйте автообновление: sudo certbot renew --dry-run"
```

## Мониторинг Load Balancer

### Nginx статус и метрики

```nginx
# /etc/nginx/sites-available/nginx-status
# Конфигурация для мониторинга Nginx

server {
    listen 127.0.0.1:8088;          # Только локальный доступ
    server_name localhost;
    
    # ==================== БАЗОВЫЙ СТАТУС ====================
    location /nginx_status {
        stub_status on;              # Включение базовой статистики
        access_log off;              # Отключение логирования
        allow 127.0.0.1;            # Разрешить только localhost
        allow 10.0.0.0/8;           # Разрешить внутреннюю сеть
        deny all;                   # Запретить все остальные
    }
    
    # ==================== ПОДРОБНЫЕ МЕТРИКИ ====================
    location /nginx_metrics {
        # Возврат метрик в формате Prometheus
        access_log off;
        allow 127.0.0.1;
        allow 10.0.0.0/8;
        deny all;
        
        return 200 '# HELP nginx_active_connections Active connections
# TYPE nginx_active_connections gauge
nginx_active_connections $connections_active
# HELP nginx_reading_connections Reading connections  
# TYPE nginx_reading_connections gauge
nginx_reading_connections $connections_reading
# HELP nginx_writing_connections Writing connections
# TYPE nginx_writing_connections gauge  
nginx_writing_connections $connections_writing
# HELP nginx_waiting_connections Waiting connections
# TYPE nginx_waiting_connections gauge
nginx_waiting_connections $connections_waiting
';
        add_header Content-Type text/plain;
    }
    
    # ==================== ПРОВЕРКА UPSTREAM ====================
    location /upstream_health {
        access_log off;
        allow 127.0.0.1;
        allow 10.0.0.0/8;
        deny all;
        
        # Простая проверка upstream серверов
        content_by_lua_block {
            local http = require "resty.http"
            local httpc = http.new()
            
            local upstreams = {
                "10.0.1.20:8080",
                "10.0.1.21:8080", 
                "10.0.1.22:8080"
            }
            
            ngx.say("Upstream Health Check:")
            for _, upstream in ipairs(upstreams) do
                local res, err = httpc:request_uri("http://" .. upstream .. "/health", {
                    method = "GET",
                    timeout = 5000
                })
                
                if res and res.status == 200 then
                    ngx.say(upstream .. " - OK")
                else
                    ngx.say(upstream .. " - FAIL (" .. (err or res.status) .. ")")
                end
            end
        }
    }
}
```

### Скрипт мониторинга балансировщика

```bash
#!/bin/bash
# scripts/monitor-loadbalancer.sh
# Мониторинг состояния Nginx Load Balancer

# Цвета для вывода
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

echo "🔍 Мониторинг Nginx Load Balancer"
echo "=================================="

# ==================== СТАТУС NGINX ====================
echo -e "\n📊 ${YELLOW}Статус Nginx:${NC}"
if systemctl is-active --quiet nginx; then
    echo -e "${GREEN}✅ Nginx запущен${NC}"
    
    # Получение базовой статистики
    if curl -s http://127.0.0.1:8088/nginx_status > /dev/null 2>&1; then
        stats=$(curl -s http://127.0.0.1:8088/nginx_status)
        echo "📈 Статистика подключений:"
        echo "$stats" | while read line; do
            echo "   $line"
        done
    else
        echo -e "${YELLOW}⚠️ Статистика недоступна (модуль stub_status не настроен)${NC}"
    fi
else
    echo -e "${RED}❌ Nginx не запущен${NC}"
    exit 1
fi

# ==================== ПРОВЕРКА КОНФИГУРАЦИИ ====================
echo -e "\n🔧 ${YELLOW}Проверка конфигурации:${NC}"
if sudo nginx -t 2>/dev/null; then
    echo -e "${GREEN}✅ Конфигурация валидна${NC}"
else
    echo -e "${RED}❌ Ошибка в конфигурации${NC}"
    sudo nginx -t
fi

# ==================== ПРОВЕРКА UPSTREAM СЕРВЕРОВ ====================
echo -e "\n🖥️ ${YELLOW}Проверка upstream серверов:${NC}"

# Список серверов из конфигурации
UPSTREAM_SERVERS=("10.0.1.20:8080" "10.0.1.21:8080" "10.0.1.22:8080")

for server in "${UPSTREAM_SERVERS[@]}"; do
    if curl -f -s "http://$server/health" > /dev/null; then
        echo -e "   ${GREEN}✅ $server - Доступен${NC}"
    else
        echo -e "   ${RED}❌ $server - Недоступен${NC}"
    fi
done

# ==================== ПРОВЕРКА SSL СЕРТИФИКАТОВ ====================
echo -e "\n🔒 ${YELLOW}Проверка SSL сертификатов:${NC}"

# Домены для проверки
DOMAINS=("api.mycompany.com")

for domain in "${DOMAINS[@]}"; do
    if command -v openssl &> /dev/null; then
        expiry=$(echo | openssl s_client -servername "$domain" -connect "$domain:443" 2>/dev/null | openssl x509 -noout -dates | grep notAfter | cut -d= -f2)
        
        if [ -n "$expiry" ]; then
            expiry_date=$(date -d "$expiry" +%s)
            current_date=$(date +%s)
            days_left=$(( (expiry_date - current_date) / 86400 ))
            
            if [ $days_left -gt 30 ]; then
                echo -e "   ${GREEN}✅ $domain - SSL действителен ($days_left дней)${NC}"
            elif [ $days_left -gt 7 ]; then
                echo -e "   ${YELLOW}⚠️ $domain - SSL истекает через $days_left дней${NC}"
            else
                echo -e "   ${RED}❌ $domain - SSL истекает через $days_left дней!${NC}"
            fi
        else
            echo -e "   ${RED}❌ $domain - Не удалось получить информацию о SSL${NC}"
        fi
    else
        echo -e "   ${YELLOW}⚠️ OpenSSL не установлен для проверки SSL${NC}"
        break
    fi
done

# ==================== ПРОВЕРКА ЛОГОВ НА ОШИБКИ ====================
echo -e "\n📝 ${YELLOW}Проверка логов (последние 10 ошибок):${NC}"

if [ -f /var/log/nginx/error.log ]; then
    recent_errors=$(sudo tail -50 /var/log/nginx/error.log | grep -i error | tail -10)
    
    if [ -n "$recent_errors" ]; then
        echo -e "${RED}❌ Найдены ошибки:${NC}"
        echo "$recent_errors" | while read line; do
            echo "   $line"
        done
    else
        echo -e "${GREEN}✅ Критические ошибки не найдены${NC}"
    fi
else
    echo -e "${YELLOW}⚠️ Лог-файл ошибок не найден${NC}"
fi

# ==================== ПРОВЕРКА НАГРУЗКИ ====================
echo -e "\n⚡ ${YELLOW}Текущая нагрузка:${NC}"

# Проверка активных соединений через ss
active_connections=$(ss -tuln | grep ':80\|:443' | wc -l)
echo "   Активных соединений: $active_connections"

# Проверка нагрузки на систему
load_avg=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | sed 's/,//')
echo "   Средняя нагрузка: $load_avg"

# ==================== ПРОВЕРКА ДИСКОВОГО ПРОСТРАНСТВА ====================
echo -e "\n💾 ${YELLOW}Использование диска для логов:${NC}"

if [ -d /var/log/nginx ]; then
    log_size=$(du -sh /var/log/nginx 2>/dev/null | cut -f1)
    echo "   Размер логов Nginx: $log_size"
    
    # Проверка места на диске
    disk_usage=$(df /var/log | tail -1 | awk '{print $5}' | sed 's/%//')
    if [ $disk_usage -gt 85 ]; then
        echo -e "   ${RED}❌ Диск заполнен на $disk_usage%${NC}"
    elif [ $disk_usage -gt 70 ]; then
        echo -e "   ${YELLOW}⚠️ Диск заполнен на $disk_usage%${NC}"
    else
        echo -e "   ${GREEN}✅ Диск заполнен на $disk_usage%${NC}"
    fi
fi

echo -e "\n✅ Мониторинг завершен"
```

## Оптимизация производительности

### Настройки для высоких нагрузок

```nginx
# /etc/nginx/nginx.conf
# Оптимизированная конфигурация Nginx для высоких нагрузок

# ==================== ОСНОВНЫЕ ПАРАМЕТРЫ ====================
user www-data;
worker_processes auto;              # Автоматическое определение количества процессов
worker_rlimit_nofile 65535;        # Лимит открытых файлов для worker процессов

# ==================== СОБЫТИЯ ====================
events {
    worker_connections 4096;        # Максимум соединений на worker
    use epoll;                      # Эффективная модель событий для Linux
    multi_accept on;                # Принимать несколько соединений за раз
    accept_mutex off;               # Отключение mutex для лучшей производительности
}

# ==================== HTTP БЛОК ====================
http {
    # ==================== ОСНОВНЫЕ НАСТРОЙКИ ====================
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # Отключение показа версии Nginx
    server_tokens off;
    
    # ==================== ОПТИМИЗАЦИЯ СЕТЕВОГО ВВОДА-ВЫВОДА ====================
    sendfile on;                    # Эффективная передача файлов
    tcp_nopush on;                  # Оптимизация TCP пакетов
    tcp_nodelay on;                 # Отключение алгоритма Nagle
    
    # ==================== ТАЙМАУТЫ ====================
    client_header_timeout 60s;     # Таймаут чтения заголовков клиента
    client_body_timeout 60s;       # Таймаут чтения тела запроса
    send_timeout 60s;              # Таймаут отправки ответа
    keepalive_timeout 65s;         # Таймаут keep-alive соединений
    keepalive_requests 1000;       # Количество запросов в keep-alive соединении
    
    # ==================== РАЗМЕРЫ БУФЕРОВ ====================
    client_header_buffer_size 4k;  # Буфер для заголовков клиента
    large_client_header_buffers 4 8k; # Большие буферы для заголовков
    client_body_buffer_size 128k;  # Буфер для тела запроса
    client_max_body_size 10m;      # Максимальный размер тела запроса
    
    # ==================== ХЭШИРОВАНИЕ ====================
    server_names_hash_bucket_size 128;  # Размер bucket для хэш-таблицы имен серверов
    types_hash_max_size 2048;           # Максимальный размер хэш-таблицы типов
    
    # ==================== СЖАТИЕ ====================
    gzip on;
    gzip_vary on;
    gzip_comp_level 6;
    gzip_min_length 1000;
    gzip_proxied any;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;
    
    # ==================== КЭШИРОВАНИЕ ====================
    open_file_cache max=10000 inactive=60s;    # Кэш открытых файлов
    open_file_cache_valid 60s;                 # Время валидности кэша
    open_file_cache_min_uses 2;                # Минимальное использование для кэша
    open_file_cache_errors on;                 # Кэширование ошибок
    
    # ==================== ЛОГИРОВАНИЕ ====================
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for" '
                    'rt=$request_time uct="$upstream_connect_time" '
                    'uht="$upstream_header_time" urt="$upstream_response_time"';
    
    # Буферизация логов для производительности
    access_log /var/log/nginx/access.log main buffer=64k flush=5s;
    error_log /var/log/nginx/error.log warn;
    
    # ==================== ПОДКЛЮЧЕНИЕ КОНФИГУРАЦИЙ ====================
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

## Проверочные вопросы

1. **Какие алгоритмы балансировки поддерживает Nginx?**
2. **Как настроить health check для upstream серверов?**
3. **Что такое SSL session resumption и зачем он нужен?**
4. **Как ограничить количество запросов от одного IP?**
5. **Какие методы оптимизации производительности Nginx существуют?**

## Практические задания

### Задание 1: Базовая настройка
- Настройте Nginx как load balancer для 3 серверов
- Добавьте health check и failover
- Протестируйте балансировку нагрузки

### Задание 2: SSL и безопасность
- Установите SSL сертификат с Let's Encrypt
- Настройте HSTS и другие заголовки безопасности
- Добавьте rate limiting

### Задание 3: Мониторинг и оптимизация
- Настройте мониторинг upstream серверов
- Оптимизируйте конфигурацию для высоких нагрузок
- Создайте алерты для критических ситуаций

## Связанные темы

- [[server-setup]] - Подготовка серверов
- [[ansible-automation]] - Автоматизация настройки
- [[kubernetes-deployment]] - Альтернативы в K8s
- [[monitoring-logging]] - Мониторинг балансировщика 