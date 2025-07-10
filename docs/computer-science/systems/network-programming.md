# Сетевое программирование 🌐

> **Навигация**: [[README|← Системное программирование]] | [[computer-science/README|Компьютерные науки]] | [[README|Главная]]

## 🎯 Цель изучения

Освоение сетевого программирования на системном уровне, понимание работы протоколов, создание высокопроизводительных сетевых приложений и сервисов.

## 📚 Теоретические основы

### 🔗 Модель OSI и TCP/IP стек

#### Уровни сетевой модели
```c
// Физический уровень (Physical Layer)
// - Передача битов по физической среде
// - Ethernet, Wi-Fi, оптические каналы

// Канальный уровень (Data Link Layer) 
// - Кадры, MAC-адреса
// - Ethernet протокол, ARP

// Сетевой уровень (Network Layer)
// - IP-пакеты, маршрутизация
// - IPv4/IPv6, ICMP, routing protocols

// Транспортный уровень (Transport Layer)
// - TCP (надежность), UDP (скорость)
// - Сегментация, контроль потока

// Сеансовый уровень (Session Layer)
// - Управление сессиями
// - SSL/TLS, RPC, NetBIOS

// Представления (Presentation Layer)  
// - Кодирование, шифрование
// - ASCII, JPEG, encryption

// Прикладной уровень (Application Layer)
// - HTTP, FTP, SMTP, DNS
// - Пользовательские протоколы
```

#### TCP vs UDP: выбор протокола
```c
// TCP - Transmission Control Protocol
// ✅ Надежность (acknowledgments)
// ✅ Упорядоченная доставка  
// ✅ Контроль потока и перегрузки
// ❌ Высокая задержка (latency)
// ❌ Больше overhead

// UDP - User Datagram Protocol  
// ✅ Низкая задержка
// ✅ Простота реализации
// ✅ Multicast поддержка
// ❌ Нет гарантий доставки
// ❌ Возможна потеря/дублирование
```

### 🔌 Socket Programming

#### Основы BSD Sockets API
```c
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>

// Создание TCP сервера
int create_tcp_server(int port) {
    int server_fd, client_fd;
    struct sockaddr_in address;
    int opt = 1;
    int addrlen = sizeof(address);
    
    // Создание socket descriptor
    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }
    
    // Установка socket options (reuse address)
    if (setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR | SO_REUSEPORT,
                   &opt, sizeof(opt))) {
        perror("setsockopt");
        exit(EXIT_FAILURE);
    }
    
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(port);
    
    // Привязка socket к порту
    if (bind(server_fd, (struct sockaddr *)&address, sizeof(address)) < 0) {
        perror("bind failed");
        exit(EXIT_FAILURE);
    }
    
    // Начало прослушивания (backlog = 10)
    if (listen(server_fd, 10) < 0) {
        perror("listen");
        exit(EXIT_FAILURE);
    }
    
    return server_fd;
}

// TCP клиент
int create_tcp_client(const char* server_ip, int port) {
    int sock = 0;
    struct sockaddr_in serv_addr;
    
    if ((sock = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        printf("Socket creation error\n");
        return -1;
    }
    
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(port);
    
    // Convert IPv4 and IPv6 addresses from text to binary form
    if (inet_pton(AF_INET, server_ip, &serv_addr.sin_addr) <= 0) {
        printf("Invalid address/ Address not supported\n");
        return -1;
    }
    
    if (connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) < 0) {
        printf("Connection Failed\n");
        return -1;
    }
    
    return sock;
}
```

#### UDP Programming
```c
#include <sys/socket.h>
#include <netinet/in.h>

// UDP сервер
int create_udp_server(int port) {
    int sockfd;
    struct sockaddr_in servaddr;
    
    // Creating socket file descriptor
    if ((sockfd = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
        perror("socket creation failed");
        exit(EXIT_FAILURE);
    }
    
    memset(&servaddr, 0, sizeof(servaddr));
    
    // Filling server information
    servaddr.sin_family = AF_INET; // IPv4
    servaddr.sin_addr.s_addr = INADDR_ANY;
    servaddr.sin_port = htons(port);
    
    // Bind the socket with the server address
    if (bind(sockfd, (const struct sockaddr *)&servaddr, sizeof(servaddr)) < 0) {
        perror("bind failed");
        exit(EXIT_FAILURE);
    }
    
    return sockfd;
}

// UDP message handling
void handle_udp_messages(int sockfd) {
    char buffer[1024];
    struct sockaddr_in cliaddr;
    socklen_t len = sizeof(cliaddr);
    
    while (1) {
        int n = recvfrom(sockfd, buffer, 1024, MSG_WAITALL,
                        (struct sockaddr*)&cliaddr, &len);
        buffer[n] = '\0';
        
        printf("Received: %s\n", buffer);
        
        // Echo back to client
        sendto(sockfd, buffer, strlen(buffer), MSG_CONFIRM,
               (const struct sockaddr*)&cliaddr, len);
    }
}
```

## 🚀 Высокопроизводительное сетевое программирование

### ⚡ Асинхронный I/O

#### select() и poll()
```c
#include <sys/select.h>
#include <poll.h>

// Использование select()
void handle_multiple_clients_select(int server_fd) {
    fd_set readfds;
    int max_sd, activity, new_socket;
    int client_sockets[30] = {0}; // максимум 30 клиентов
    
    while (TRUE) {
        // Clear the socket set
        FD_ZERO(&readfds);
        
        // Add master socket to set
        FD_SET(server_fd, &readfds);
        max_sd = server_fd;
        
        // Add child sockets to set
        for (int i = 0; i < 30; i++) {
            int sd = client_sockets[i];
            
            if (sd > 0)
                FD_SET(sd, &readfds);
            
            if (sd > max_sd)
                max_sd = sd;
        }
        
        // Wait for activity on one of the sockets
        activity = select(max_sd + 1, &readfds, NULL, NULL, NULL);
        
        if ((activity < 0) && (errno != EINTR)) {
            printf("select error");
        }
        
        // If something happened on the master socket, it's an incoming connection
        if (FD_ISSET(server_fd, &readfds)) {
            struct sockaddr_in address;
            int addrlen = sizeof(address);
            
            if ((new_socket = accept(server_fd, (struct sockaddr *)&address,
                                   (socklen_t*)&addrlen)) < 0) {
                perror("accept");
                exit(EXIT_FAILURE);
            }
            
            // Add new socket to array of sockets
            for (int i = 0; i < 30; i++) {
                if (client_sockets[i] == 0) {
                    client_sockets[i] = new_socket;
                    printf("Adding to list of sockets as %d\n", i);
                    break;
                }
            }
        }
        
        // Handle I/O operation on client sockets
        for (int i = 0; i < 30; i++) {
            int sd = client_sockets[i];
            
            if (FD_ISSET(sd, &readfds)) {
                // Handle client communication
                handle_client_message(sd, client_sockets, i);
            }
        }
    }
}

// Использование poll()
void handle_multiple_clients_poll(int server_fd) {
    struct pollfd fds[100];
    int nfds = 1;
    
    // Initialize pollfd structure
    fds[0].fd = server_fd;
    fds[0].events = POLLIN;
    
    while (1) {
        int ret = poll(fds, nfds, -1);
        
        if (ret < 0) {
            perror("poll");
            break;
        }
        
        for (int i = 0; i < nfds; i++) {
            if (fds[i].revents & POLLIN) {
                if (fds[i].fd == server_fd) {
                    // New connection
                    int new_fd = accept(server_fd, NULL, NULL);
                    if (new_fd >= 0) {
                        fds[nfds].fd = new_fd;
                        fds[nfds].events = POLLIN;
                        nfds++;
                    }
                } else {
                    // Handle existing connection
                    handle_client_data(fds[i].fd);
                }
            }
        }
    }
}
```

#### epoll (Linux) - высокая производительность
```c
#include <sys/epoll.h>

#define MAX_EVENTS 1000

// Создание epoll instance
int setup_epoll_server(int server_fd) {
    int epoll_fd;
    struct epoll_event ev, events[MAX_EVENTS];
    
    epoll_fd = epoll_create1(0);
    if (epoll_fd == -1) {
        perror("epoll_create1");
        exit(EXIT_FAILURE);
    }
    
    ev.events = EPOLLIN;
    ev.data.fd = server_fd;
    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, server_fd, &ev) == -1) {
        perror("epoll_ctl: server_fd");
        exit(EXIT_FAILURE);
    }
    
    return epoll_fd;
}

// Event loop с epoll
void epoll_event_loop(int epoll_fd, int server_fd) {
    struct epoll_event events[MAX_EVENTS];
    
    while (1) {
        int nfds = epoll_wait(epoll_fd, events, MAX_EVENTS, -1);
        if (nfds == -1) {
            perror("epoll_wait");
            exit(EXIT_FAILURE);
        }
        
        for (int n = 0; n < nfds; ++n) {
            if (events[n].data.fd == server_fd) {
                // Accept new connection
                struct sockaddr_in client_addr;
                socklen_t client_len = sizeof(client_addr);
                int client_fd = accept(server_fd, 
                                     (struct sockaddr*)&client_addr, 
                                     &client_len);
                
                if (client_fd == -1) {
                    perror("accept");
                    continue;
                }
                
                // Make non-blocking
                int flags = fcntl(client_fd, F_GETFL, 0);
                fcntl(client_fd, F_SETFL, flags | O_NONBLOCK);
                
                // Add to epoll
                struct epoll_event ev;
                ev.events = EPOLLIN | EPOLLET; // Edge-triggered
                ev.data.fd = client_fd;
                if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, client_fd, &ev) == -1) {
                    perror("epoll_ctl: client_fd");
                    exit(EXIT_FAILURE);
                }
            } else {
                // Handle client data
                handle_client_epoll(events[n].data.fd, epoll_fd);
            }
        }
    }
}

// Обработка клиентских данных
void handle_client_epoll(int client_fd, int epoll_fd) {
    char buffer[1024];
    ssize_t bytes_read;
    
    while ((bytes_read = read(client_fd, buffer, sizeof(buffer))) > 0) {
        // Process data
        printf("Received %zd bytes: %.*s\n", bytes_read, (int)bytes_read, buffer);
        
        // Echo back
        write(client_fd, buffer, bytes_read);
    }
    
    if (bytes_read == 0) {
        // Client disconnected
        printf("Client disconnected\n");
        epoll_ctl(epoll_fd, EPOLL_CTL_DEL, client_fd, NULL);
        close(client_fd);
    } else if (bytes_read < 0 && errno != EAGAIN) {
        perror("read");
        epoll_ctl(epoll_fd, EPOLL_CTL_DEL, client_fd, NULL);
        close(client_fd);
    }
}
```

### 🌊 io_uring (современный Linux)
```c
#include <liburing.h>

// Современный асинхронный I/O
int setup_io_uring_server(int server_fd) {
    struct io_uring ring;
    int ret;
    
    ret = io_uring_queue_init(256, &ring, 0);
    if (ret < 0) {
        fprintf(stderr, "queue_init: %s\n", strerror(-ret));
        return -1;
    }
    
    // Добавление accept операции
    struct io_uring_sqe *sqe = io_uring_get_sqe(&ring);
    io_uring_prep_accept(sqe, server_fd, NULL, NULL, 0);
    
    // Submit and wait
    io_uring_submit(&ring);
    
    struct io_uring_cqe *cqe;
    ret = io_uring_wait_cqe(&ring, &cqe);
    if (ret < 0) {
        fprintf(stderr, "wait_cqe: %s\n", strerror(-ret));
        return -1;
    }
    
    int client_fd = cqe->res;
    io_uring_cqe_seen(&ring, cqe);
    
    return client_fd;
}
```

## 🌐 Протоколы и сервисы

### 📄 HTTP Server implementation
```c
#include <string.h>

// Простой HTTP сервер
typedef struct {
    char method[16];
    char path[256];
    char version[16];
    char headers[2048];
    char body[4096];
} http_request_t;

// Парсинг HTTP запроса
void parse_http_request(const char* raw_request, http_request_t* request) {
    char* line = strtok((char*)raw_request, "\r\n");
    
    // Парсинг первой строки: METHOD PATH HTTP/VERSION
    sscanf(line, "%s %s %s", request->method, request->path, request->version);
    
    // Парсинг заголовков
    line = strtok(NULL, "\r\n");
    while (line && strlen(line) > 0) {
        strcat(request->headers, line);
        strcat(request->headers, "\n");
        line = strtok(NULL, "\r\n");
    }
    
    // Тело запроса (если есть)
    line = strtok(NULL, "\0");
    if (line) {
        strcpy(request->body, line);
    }
}

// Создание HTTP ответа
void create_http_response(char* response, const char* status, 
                         const char* content_type, const char* body) {
    sprintf(response,
        "HTTP/1.1 %s\r\n"
        "Content-Type: %s\r\n"
        "Content-Length: %zu\r\n"
        "Connection: close\r\n"
        "\r\n"
        "%s",
        status, content_type, strlen(body), body);
}

// HTTP сервер с routing
void handle_http_request(int client_fd) {
    char buffer[4096] = {0};
    char response[4096];
    http_request_t request = {0};
    
    // Чтение запроса
    read(client_fd, buffer, 4096);
    parse_http_request(buffer, &request);
    
    printf("Method: %s, Path: %s\n", request.method, request.path);
    
    // Простой routing
    if (strcmp(request.path, "/") == 0) {
        create_http_response(response, "200 OK", "text/html",
                           "<h1>Welcome to our server!</h1>");
    } else if (strcmp(request.path, "/api/status") == 0) {
        create_http_response(response, "200 OK", "application/json",
                           "{\"status\": \"OK\", \"timestamp\": 1234567890}");
    } else {
        create_http_response(response, "404 Not Found", "text/html",
                           "<h1>404 - Page Not Found</h1>");
    }
    
    // Отправка ответа
    write(client_fd, response, strlen(response));
    close(client_fd);
}
```

### 🔒 SSL/TLS с OpenSSL
```c
#include <openssl/ssl.h>
#include <openssl/err.h>

// Инициализация SSL контекста
SSL_CTX* init_ssl_context() {
    const SSL_METHOD *method;
    SSL_CTX *ctx;
    
    SSL_library_init();
    OpenSSL_add_all_algorithms();
    SSL_load_error_strings();
    
    method = TLS_server_method();
    ctx = SSL_CTX_new(method);
    
    if (!ctx) {
        perror("Unable to create SSL context");
        ERR_print_errors_fp(stderr);
        exit(EXIT_FAILURE);
    }
    
    // Загрузка сертификата и ключа
    if (SSL_CTX_use_certificate_file(ctx, "server.crt", SSL_FILETYPE_PEM) <= 0) {
        ERR_print_errors_fp(stderr);
        exit(EXIT_FAILURE);
    }
    
    if (SSL_CTX_use_PrivateKey_file(ctx, "server.key", SSL_FILETYPE_PEM) <= 0) {
        ERR_print_errors_fp(stderr);
        exit(EXIT_FAILURE);
    }
    
    return ctx;
}

// SSL соединение
void handle_ssl_client(int client_fd, SSL_CTX* ctx) {
    SSL *ssl;
    
    ssl = SSL_new(ctx);
    SSL_set_fd(ssl, client_fd);
    
    if (SSL_accept(ssl) <= 0) {
        ERR_print_errors_fp(stderr);
    } else {
        // Обработка HTTPS запроса
        char buffer[1024];
        int bytes = SSL_read(ssl, buffer, sizeof(buffer));
        
        if (bytes > 0) {
            buffer[bytes] = '\0';
            printf("Received: %s\n", buffer);
            
            // Отправка ответа
            const char* response = "HTTP/1.1 200 OK\r\n"
                                 "Content-Length: 13\r\n\r\n"
                                 "Hello, SSL!";
            SSL_write(ssl, response, strlen(response));
        }
    }
    
    SSL_shutdown(ssl);
    SSL_free(ssl);
    close(client_fd);
}
```

## 🔧 Практические инструменты

### 📊 Мониторинг сетевой активности
```c
#include <sys/time.h>

// Структура для статистики
typedef struct {
    unsigned long bytes_sent;
    unsigned long bytes_received;
    unsigned long connections_total;
    unsigned long connections_active;
    double avg_response_time;
    struct timeval start_time;
} network_stats_t;

// Инициализация статистики
void init_network_stats(network_stats_t* stats) {
    memset(stats, 0, sizeof(network_stats_t));
    gettimeofday(&stats->start_time, NULL);
}

// Обновление статистики
void update_stats(network_stats_t* stats, size_t bytes_sent, 
                  size_t bytes_received, double response_time) {
    stats->bytes_sent += bytes_sent;
    stats->bytes_received += bytes_received;
    
    // Обновление среднего времени ответа
    static double total_response_time = 0;
    static int response_count = 0;
    
    total_response_time += response_time;
    response_count++;
    stats->avg_response_time = total_response_time / response_count;
}

// Вывод статистики
void print_network_stats(const network_stats_t* stats) {
    struct timeval current_time;
    gettimeofday(&current_time, NULL);
    
    double uptime = (current_time.tv_sec - stats->start_time.tv_sec) +
                   (current_time.tv_usec - stats->start_time.tv_usec) / 1000000.0;
    
    printf("=== Network Statistics ===\n");
    printf("Uptime: %.2f seconds\n", uptime);
    printf("Bytes sent: %lu\n", stats->bytes_sent);
    printf("Bytes received: %lu\n", stats->bytes_received);
    printf("Total connections: %lu\n", stats->connections_total);
    printf("Active connections: %lu\n", stats->connections_active);
    printf("Average response time: %.2f ms\n", stats->avg_response_time);
    printf("Throughput: %.2f KB/s\n", 
           (stats->bytes_sent + stats->bytes_received) / 1024.0 / uptime);
}
```

### 🧪 Сетевые утилиты и тестирование
```c
// Ping implementation
#include <netinet/ip_icmp.h>

// ICMP ping
int send_ping(const char* target_ip) {
    int sockfd;
    struct sockaddr_in target_addr;
    struct icmp icmp_hdr;
    char packet[64];
    
    sockfd = socket(AF_INET, SOCK_RAW, IPPROTO_ICMP);
    if (sockfd < 0) {
        perror("socket");
        return -1;
    }
    
    // Настройка адреса
    target_addr.sin_family = AF_INET;
    inet_pton(AF_INET, target_ip, &target_addr.sin_addr);
    
    // Создание ICMP пакета
    icmp_hdr.icmp_type = ICMP_ECHO;
    icmp_hdr.icmp_code = 0;
    icmp_hdr.icmp_id = getpid();
    icmp_hdr.icmp_seq = 1;
    icmp_hdr.icmp_cksum = 0;
    
    memcpy(packet, &icmp_hdr, sizeof(icmp_hdr));
    
    // Вычисление checksum
    icmp_hdr.icmp_cksum = calculate_checksum((unsigned short*)packet, sizeof(icmp_hdr));
    memcpy(packet, &icmp_hdr, sizeof(icmp_hdr));
    
    // Отправка пакета
    if (sendto(sockfd, packet, sizeof(icmp_hdr), 0,
               (struct sockaddr*)&target_addr, sizeof(target_addr)) < 0) {
        perror("sendto");
        close(sockfd);
        return -1;
    }
    
    close(sockfd);
    return 0;
}

// Port scanner
void scan_port(const char* host, int port) {
    int sock;
    struct sockaddr_in target;
    struct timeval timeout;
    
    sock = socket(AF_INET, SOCK_STREAM, 0);
    
    // Установка таймаута
    timeout.tv_sec = 1;
    timeout.tv_usec = 0;
    setsockopt(sock, SOL_SOCKET, SO_RCVTIMEO, &timeout, sizeof(timeout));
    setsockopt(sock, SOL_SOCKET, SO_SNDTIMEO, &timeout, sizeof(timeout));
    
    target.sin_family = AF_INET;
    target.sin_port = htons(port);
    inet_pton(AF_INET, host, &target.sin_addr);
    
    if (connect(sock, (struct sockaddr*)&target, sizeof(target)) == 0) {
        printf("Port %d: OPEN\n", port);
    } else {
        printf("Port %d: CLOSED\n", port);
    }
    
    close(sock);
}
```

## 🎯 Оптимизация производительности

### ⚡ Zero-copy techniques
```c
#include <sys/sendfile.h>

// Отправка файла с zero-copy
ssize_t send_file_zerocopy(int client_fd, const char* filename) {
    int file_fd = open(filename, O_RDONLY);
    if (file_fd < 0) {
        perror("open");
        return -1;
    }
    
    struct stat file_stat;
    if (fstat(file_fd, &file_stat) < 0) {
        perror("fstat");
        close(file_fd);
        return -1;
    }
    
    // sendfile() - zero-copy transfer
    ssize_t bytes_sent = sendfile(client_fd, file_fd, NULL, file_stat.st_size);
    
    close(file_fd);
    return bytes_sent;
}

// Memory mapping для больших файлов
void* map_file_to_memory(const char* filename, size_t* size) {
    int fd = open(filename, O_RDONLY);
    if (fd < 0) {
        perror("open");
        return NULL;
    }
    
    struct stat sb;
    if (fstat(fd, &sb) < 0) {
        perror("fstat");
        close(fd);
        return NULL;
    }
    
    *size = sb.st_size;
    void* mapped = mmap(NULL, *size, PROT_READ, MAP_PRIVATE, fd, 0);
    
    close(fd);
    
    if (mapped == MAP_FAILED) {
        perror("mmap");
        return NULL;
    }
    
    return mapped;
}
```

### 🔥 Connection pooling
```c
#define MAX_POOL_SIZE 100

typedef struct {
    int fd;
    int in_use;
    time_t last_used;
} connection_t;

typedef struct {
    connection_t connections[MAX_POOL_SIZE];
    int size;
    pthread_mutex_t mutex;
} connection_pool_t;

// Инициализация пула соединений
connection_pool_t* create_connection_pool() {
    connection_pool_t* pool = malloc(sizeof(connection_pool_t));
    pool->size = 0;
    pthread_mutex_init(&pool->mutex, NULL);
    
    for (int i = 0; i < MAX_POOL_SIZE; i++) {
        pool->connections[i].fd = -1;
        pool->connections[i].in_use = 0;
        pool->connections[i].last_used = 0;
    }
    
    return pool;
}

// Получение соединения из пула
int get_connection(connection_pool_t* pool) {
    pthread_mutex_lock(&pool->mutex);
    
    for (int i = 0; i < MAX_POOL_SIZE; i++) {
        if (pool->connections[i].fd >= 0 && !pool->connections[i].in_use) {
            pool->connections[i].in_use = 1;
            pool->connections[i].last_used = time(NULL);
            pthread_mutex_unlock(&pool->mutex);
            return pool->connections[i].fd;
        }
    }
    
    pthread_mutex_unlock(&pool->mutex);
    return -1; // No available connections
}

// Возврат соединения в пул
void return_connection(connection_pool_t* pool, int fd) {
    pthread_mutex_lock(&pool->mutex);
    
    for (int i = 0; i < MAX_POOL_SIZE; i++) {
        if (pool->connections[i].fd == fd) {
            pool->connections[i].in_use = 0;
            pool->connections[i].last_used = time(NULL);
            break;
        }
    }
    
    pthread_mutex_unlock(&pool->mutex);
}
```

## 📈 Продвинутые темы

### 🌐 Load Balancing
```c
// Round-robin load balancer
typedef struct {
    char ip[16];
    int port;
    int active;
    int current_connections;
} backend_server_t;

typedef struct {
    backend_server_t servers[10];
    int server_count;
    int current_index;
    pthread_mutex_t mutex;
} load_balancer_t;

// Получение следующего сервера (round-robin)
backend_server_t* get_next_server(load_balancer_t* lb) {
    pthread_mutex_lock(&lb->mutex);
    
    int start_index = lb->current_index;
    do {
        if (lb->servers[lb->current_index].active) {
            backend_server_t* server = &lb->servers[lb->current_index];
            lb->current_index = (lb->current_index + 1) % lb->server_count;
            pthread_mutex_unlock(&lb->mutex);
            return server;
        }
        lb->current_index = (lb->current_index + 1) % lb->server_count;
    } while (lb->current_index != start_index);
    
    pthread_mutex_unlock(&lb->mutex);
    return NULL; // No active servers
}

// Health check для серверов
void* health_check_thread(void* arg) {
    load_balancer_t* lb = (load_balancer_t*)arg;
    
    while (1) {
        for (int i = 0; i < lb->server_count; i++) {
            int sock = create_tcp_client(lb->servers[i].ip, lb->servers[i].port);
            if (sock >= 0) {
                lb->servers[i].active = 1;
                close(sock);
            } else {
                lb->servers[i].active = 0;
            }
        }
        sleep(30); // Check every 30 seconds
    }
    
    return NULL;
}
```

### 🔄 Протокол WebSocket
```c
#include <openssl/sha.h>
#include <openssl/bio.h>
#include <openssl/evp.h>
#include <openssl/buffer.h>

// WebSocket handshake
int websocket_handshake(int client_fd, const char* websocket_key) {
    char accept_key[64];
    char response[512];
    
    // Создание WebSocket Accept ключа
    char combined[256];
    snprintf(combined, sizeof(combined), "%s258EAFA5-E914-47DA-95CA-C5AB0DC85B11", 
             websocket_key);
    
    unsigned char hash[SHA_DIGEST_LENGTH];
    SHA1((unsigned char*)combined, strlen(combined), hash);
    
    // Base64 encoding
    BIO *bmem, *b64;
    BUF_MEM *bptr;
    
    b64 = BIO_new(BIO_f_base64());
    bmem = BIO_new(BIO_s_mem());
    b64 = BIO_push(b64, bmem);
    BIO_write(b64, hash, SHA_DIGEST_LENGTH);
    BIO_flush(b64);
    BIO_get_mem_ptr(b64, &bptr);
    
    strncpy(accept_key, bptr->data, bptr->length - 1);
    accept_key[bptr->length - 1] = '\0';
    
    BIO_free_all(b64);
    
    // Формирование ответа
    snprintf(response, sizeof(response),
        "HTTP/1.1 101 Switching Protocols\r\n"
        "Upgrade: websocket\r\n"
        "Connection: Upgrade\r\n"
        "Sec-WebSocket-Accept: %s\r\n"
        "\r\n", accept_key);
    
    return write(client_fd, response, strlen(response));
}

// Обработка WebSocket фрейма
typedef struct {
    int fin;
    int opcode;
    int masked;
    uint64_t payload_length;
    char mask[4];
    char* payload;
} websocket_frame_t;

int parse_websocket_frame(const char* data, websocket_frame_t* frame) {
    int offset = 0;
    
    // Первый байт
    frame->fin = (data[0] & 0x80) >> 7;
    frame->opcode = data[0] & 0x0F;
    offset++;
    
    // Второй байт
    frame->masked = (data[1] & 0x80) >> 7;
    frame->payload_length = data[1] & 0x7F;
    offset++;
    
    // Расширенная длина
    if (frame->payload_length == 126) {
        frame->payload_length = (data[2] << 8) | data[3];
        offset += 2;
    } else if (frame->payload_length == 127) {
        // 64-bit length (упрощенная версия)
        frame->payload_length = 0;
        for (int i = 0; i < 8; i++) {
            frame->payload_length = (frame->payload_length << 8) | data[offset + i];
        }
        offset += 8;
    }
    
    // Маска
    if (frame->masked) {
        memcpy(frame->mask, &data[offset], 4);
        offset += 4;
    }
    
    // Полезная нагрузка
    frame->payload = malloc(frame->payload_length + 1);
    memcpy(frame->payload, &data[offset], frame->payload_length);
    
    // Демаскирование
    if (frame->masked) {
        for (uint64_t i = 0; i < frame->payload_length; i++) {
            frame->payload[i] ^= frame->mask[i % 4];
        }
    }
    
    frame->payload[frame->payload_length] = '\0';
    
    return offset + frame->payload_length;
}
```

## 🛠️ Практические проекты

### 🎯 Проект 1: HTTP Load Balancer (4-5 недель)
```c
// Основные компоненты:
// 1. HTTP proxy с round-robin балансировкой
// 2. Health checks для backend серверов
// 3. Connection pooling
// 4. Request/response логирование
// 5. SSL termination
// 6. Rate limiting по IP
```

### 🎯 Проект 2: Real-time Chat Server (3-4 недели)
```c
// Функциональность:
// 1. WebSocket поддержка
// 2. Комнаты и пользователи
// 3. Broadcast сообщений
// 4. Сохранение истории
// 5. Административные команды
// 6. Reconnection handling
```

### 🎯 Проект 3: High-performance Web Server (5-6 недель)
```c
// Возможности:
// 1. HTTP/1.1 и HTTP/2 поддержка
// 2. Static file serving с zero-copy
// 3. CGI/FastCGI support
// 4. Virtual hosts
// 5. Access logging и metrics
// 6. Configuration файлы
```

## 📚 Дополнительные ресурсы

### 📖 Рекомендуемая литература
- **"Unix Network Programming" - W. Richard Stevens** `📘 Классика`
- **"High Performance Browser Networking" - Ilya Grigorik** `🌐 Современные протоколы`
- **"TCP/IP Illustrated" - W. Richard Stevens** `🔍 Детали протоколов`

### 🔗 Связанные темы
- [[processes-threads|Процессы и потоки]] - для многопользовательских серверов
- [[memory-management|Управление памятью]] - для буферизации и кэширования
- [[../infrastructure/computer-networks|Компьютерные сети]] - теория сетей
- [[../technical-skills/databases/README|Базы данных]] - для хранения данных
- [[../technical-skills/security|Безопасность]] - защита сетевых приложений

### 🏗️ Roadmap развития
1. **Неделя 1-2**: Socket programming основы
2. **Неделя 3-4**: Асинхронный I/O (select, poll, epoll)
3. **Неделя 5-6**: HTTP протокол и веб-серверы
4. **Неделя 7-8**: SSL/TLS и безопасность
5. **Неделя 9-10**: Высокопроизводительные паттерны
6. **Неделя 11-12**: Продвинутые протоколы (WebSocket, HTTP/2)

---

> 💡 **Совет**: Начните с простого echo-сервера, затем постепенно добавляйте функциональность. Всегда тестируйте производительность под нагрузкой! 