# Процессы и потоки 🔄

> **Навигация**: [[README|← Системное программирование]] | [[memory-management|← Управление памятью]] | [[network-programming|Сетевое программирование →]]

## Содержание
1. [Process API в Unix](#🖥️-process-api-в-unix)
2. [POSIX Threads](#🧵-posix-threads)
3. [Синхронизация потоков](#🔒-синхронизация-потоков)
4. [Inter-Process Communication](#📞-inter-process-communication)
5. [Планировщик процессов](#⚡-планировщик-процессов)
6. [Производительность многопоточности](#📊-производительность-многопоточности)

---

## 🖥️ Process API в Unix

### Жизненный цикл процесса

```c
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <signal.h>
#include <errno.h>

void process_lifecycle_demo() {
    printf("=== Жизненный цикл процесса ===\n\n");
    
    printf("Состояния процесса:\n");
    printf("RUNNING  → выполняется на CPU\n");
    printf("READY    → готов к выполнению, ждет CPU\n");
    printf("BLOCKED  → заблокирован на I/O или синхронизации\n");
    printf("ZOMBIE   → завершился, но родитель не вызвал wait()\n\n");
    
    pid_t pid = fork();
    
    if (pid == 0) {
        // Дочерний процесс
        printf("[Child %d] Дочерний процесс запущен\n", getpid());
        printf("[Child %d] PPID = %d\n", getpid(), getppid());
        
        // Демонстрация exec()
        printf("[Child %d] Выполняю execl()...\n", getpid());
        execl("/bin/echo", "echo", "Hello from exec!", NULL);
        
        // Эта строка не выполнится при успешном exec
        perror("execl failed");
        exit(1);
        
    } else if (pid > 0) {
        // Родительский процесс
        printf("[Parent %d] Создал дочерний процесс %d\n", getpid(), pid);
        
        int status;
        pid_t waited_pid = wait(&status);
        
        printf("[Parent %d] Дочерний процесс %d завершился\n", 
               getpid(), waited_pid);
        
        if (WIFEXITED(status)) {
            printf("[Parent %d] Код завершения: %d\n", 
                   getpid(), WEXITSTATUS(status));
        }
        if (WIFSIGNALED(status)) {
            printf("[Parent %d] Завершен сигналом: %d\n", 
                   getpid(), WTERMSIG(status));
        }
    } else {
        perror("fork failed");
    }
}

// Демонстрация создания daemon процесса
pid_t create_daemon() {
    pid_t pid = fork();
    
    if (pid < 0) {
        return -1;  // Ошибка fork
    }
    
    if (pid > 0) {
        return pid;  // Родительский процесс возвращает PID daemon'а
    }
    
    // Дочерний процесс становится daemon'ом
    
    // 1. Создание новой сессии
    if (setsid() < 0) {
        exit(1);
    }
    
    // 2. Второй fork для отсоединения от terminal
    pid = fork();
    if (pid > 0) {
        exit(0);  // Первый дочерний процесс завершается
    }
    
    // 3. Изменение рабочей директории
    chdir("/");
    
    // 4. Закрытие файловых дескрипторов
    for (int fd = 0; fd < 1024; fd++) {
        close(fd);
    }
    
    // 5. Переопределение стандартных потоков
    open("/dev/null", O_RDONLY);  // stdin
    open("/dev/null", O_WRONLY);  // stdout
    open("/dev/null", O_WRONLY);  // stderr
    
    return 0;  // Daemon готов к работе
}
```

### Сигналы и их обработка

```c
#include <signal.h>
#include <setjmp.h>

static jmp_buf jump_buffer;
static volatile sig_atomic_t signal_received = 0;

// Продвинутая обработка сигналов
void advanced_signal_handler(int sig, siginfo_t* info, void* context) {
    signal_received = sig;
    
    switch (sig) {
        case SIGINT:
            printf("\n[Signal] SIGINT получен от PID %d\n", info->si_pid);
            printf("[Signal] Graceful shutdown...\n");
            break;
            
        case SIGTERM:
            printf("[Signal] SIGTERM получен\n");
            longjmp(jump_buffer, 1);
            break;
            
        case SIGSEGV:
            printf("[Signal] Segmentation fault в адресе %p\n", info->si_addr);
            printf("[Signal] Контекст: PC = %p\n", 
                   ((ucontext_t*)context)->uc_mcontext.gregs[REG_RIP]);
            abort();
            break;
            
        case SIGUSR1:
            printf("[Signal] Пользовательский сигнал 1\n");
            break;
    }
}

void setup_advanced_signals() {
    struct sigaction sa;
    
    // Настройка продвинутого обработчика
    sa.sa_sigaction = advanced_signal_handler;
    sa.sa_flags = SA_SIGINFO | SA_RESTART;
    sigemptyset(&sa.sa_mask);
    
    sigaction(SIGINT, &sa, NULL);
    sigaction(SIGTERM, &sa, NULL);
    sigaction(SIGSEGV, &sa, NULL);
    sigaction(SIGUSR1, &sa, NULL);
    
    // Блокировка сигналов при необходимости
    sigset_t blocked_signals;
    sigemptyset(&blocked_signals);
    sigaddset(&blocked_signals, SIGUSR1);
    
    // Временная блокировка
    pthread_sigmask(SIG_BLOCK, &blocked_signals, NULL);
    
    printf("Продвинутые обработчики сигналов установлены\n");
}

// Self-pipe trick для async-safe signal handling
int signal_pipe[2];

void pipe_signal_handler(int sig) {
    char byte = sig;
    write(signal_pipe[1], &byte, 1);
}

void setup_signal_pipe() {
    if (pipe(signal_pipe) == -1) {
        perror("pipe failed");
        return;
    }
    
    // Неблокирующий режим для write end
    int flags = fcntl(signal_pipe[1], F_GETFL);
    fcntl(signal_pipe[1], F_SETFL, flags | O_NONBLOCK);
    
    signal(SIGINT, pipe_signal_handler);
    signal(SIGTERM, pipe_signal_handler);
}

void handle_signal_events() {
    fd_set readfds;
    char sig_byte;
    
    while (1) {
        FD_ZERO(&readfds);
        FD_SET(signal_pipe[0], &readfds);
        
        if (select(signal_pipe[0] + 1, &readfds, NULL, NULL, NULL) > 0) {
            if (FD_ISSET(signal_pipe[0], &readfds)) {
                if (read(signal_pipe[0], &sig_byte, 1) > 0) {
                    printf("Обработан сигнал %d через pipe\n", sig_byte);
                    
                    if (sig_byte == SIGTERM || sig_byte == SIGINT) {
                        break;
                    }
                }
            }
        }
    }
}
```

---

## 🧵 POSIX Threads

### Основы работы с потоками

```c
#include <pthread.h>
#include <stdatomic.h>

// Thread-local storage
__thread int thread_local_var = 0;

// Данные потока
typedef struct thread_data {
    int thread_id;
    int iterations;
    void* shared_data;
    pthread_barrier_t* barrier;
} thread_data_t;

// Функция потока с продвинутыми возможностями
void* advanced_thread_function(void* arg) {
    thread_data_t* data = (thread_data_t*)arg;
    
    // Установка имени потока (Linux)
    char thread_name[16];
    snprintf(thread_name, sizeof(thread_name), "worker_%d", data->thread_id);
    pthread_setname_np(pthread_self(), thread_name);
    
    // Thread-local storage
    thread_local_var = data->thread_id * 100;
    
    printf("[Thread %d] Запущен, TLS = %d\n", 
           data->thread_id, thread_local_var);
    
    // Установка атрибутов потока
    int policy;
    struct sched_param param;
    pthread_getschedparam(pthread_self(), &policy, &param);
    
    printf("[Thread %d] Политика планирования: %s, приоритет: %d\n",
           data->thread_id, 
           (policy == SCHED_OTHER) ? "NORMAL" : "RT",
           param.sched_priority);
    
    // Синхронизация с другими потоками
    printf("[Thread %d] Ожидание на барьере...\n", data->thread_id);
    pthread_barrier_wait(data->barrier);
    
    // Основная работа потока
    for (int i = 0; i < data->iterations; i++) {
        // Имитация работы
        usleep(10000);  // 10ms
        
        if (i % 100 == 0) {
            printf("[Thread %d] Итерация %d/%d\n", 
                   data->thread_id, i, data->iterations);
        }
        
        // Проверка на отмену
        pthread_testcancel();
    }
    
    // Возврат результата
    int* result = malloc(sizeof(int));
    *result = data->thread_id * data->iterations;
    
    printf("[Thread %d] Завершен, результат: %d\n", 
           data->thread_id, *result);
    
    return result;
}

// Продвинутое управление потоками
void advanced_threading_demo() {
    const int num_threads = 4;
    pthread_t threads[num_threads];
    thread_data_t thread_data[num_threads];
    pthread_barrier_t barrier;
    
    // Инициализация барьера
    pthread_barrier_init(&barrier, NULL, num_threads);
    
    // Создание потоков с атрибутами
    for (int i = 0; i < num_threads; i++) {
        pthread_attr_t attr;
        pthread_attr_init(&attr);
        
        // Установка размера стека
        size_t stack_size = 1024 * 1024;  // 1MB
        pthread_attr_setstacksize(&attr, stack_size);
        
        // Detached или joinable состояние
        pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);
        
        // Подготовка данных
        thread_data[i].thread_id = i;
        thread_data[i].iterations = 1000;
        thread_data[i].shared_data = NULL;
        thread_data[i].barrier = &barrier;
        
        if (pthread_create(&threads[i], &attr, 
                          advanced_thread_function, &thread_data[i]) != 0) {
            perror("pthread_create failed");
            exit(1);
        }
        
        pthread_attr_destroy(&attr);
    }
    
    // Ожидание завершения и получение результатов
    for (int i = 0; i < num_threads; i++) {
        void* result;
        pthread_join(threads[i], &result);
        
        if (result) {
            printf("Поток %d вернул результат: %d\n", i, *(int*)result);
            free(result);
        }
    }
    
    pthread_barrier_destroy(&barrier);
}
```

---

## 🔒 Синхронизация потоков

### Мьютексы и их виды

```c
// Различные типы мьютексов
void mutex_types_demo() {
    printf("=== Типы мьютексов ===\n\n");
    
    // 1. Обычный мьютекс
    pthread_mutex_t normal_mutex = PTHREAD_MUTEX_INITIALIZER;
    
    // 2. Рекурсивный мьютекс
    pthread_mutex_t recursive_mutex;
    pthread_mutexattr_t attr;
    
    pthread_mutexattr_init(&attr);
    pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_RECURSIVE);
    pthread_mutex_init(&recursive_mutex, &attr);
    
    // 3. Приоритетный мьютекс (priority inheritance)
    pthread_mutex_t priority_mutex;
    pthread_mutexattr_t prio_attr;
    
    pthread_mutexattr_init(&prio_attr);
    pthread_mutexattr_setprotocol(&prio_attr, PTHREAD_PRIO_INHERIT);
    pthread_mutex_init(&priority_mutex, &prio_attr);
    
    printf("Мьютексы инициализированы:\n");
    printf("• Normal mutex: базовый\n");
    printf("• Recursive mutex: можно захватывать несколько раз\n");
    printf("• Priority mutex: наследование приоритета\n");
    
    pthread_mutex_destroy(&recursive_mutex);
    pthread_mutex_destroy(&priority_mutex);
    pthread_mutexattr_destroy(&attr);
    pthread_mutexattr_destroy(&prio_attr);
}

// Trylock и timeout operations
void mutex_timeouts_demo() {
    pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
    struct timespec timeout;
    
    // Неблокирующий захват
    if (pthread_mutex_trylock(&mutex) == 0) {
        printf("Мьютекс захвачен немедленно\n");
        
        // Timeout для второго захвата
        clock_gettime(CLOCK_REALTIME, &timeout);
        timeout.tv_sec += 1;  // 1 секунда timeout
        
        int result = pthread_mutex_timedlock(&mutex, &timeout);
        if (result == ETIMEDOUT) {
            printf("Timeout при попытке захвата мьютекса\n");
        }
        
        pthread_mutex_unlock(&mutex);
    }
}
```

### Read-Write locks

```c
// Демонстрация читатель-писатель
typedef struct shared_database {
    pthread_rwlock_t rwlock;
    int data[1000];
    int version;
} shared_database_t;

shared_database_t* database;

void* reader_thread(void* arg) {
    int reader_id = *(int*)arg;
    
    for (int i = 0; i < 10; i++) {
        // Захват для чтения
        pthread_rwlock_rdlock(&database->rwlock);
        
        // Множественные читатели могут работать одновременно
        printf("[Reader %d] Версия: %d, данные[0] = %d\n", 
               reader_id, database->version, database->data[0]);
        
        // Имитация чтения
        usleep(100000);  // 100ms
        
        pthread_rwlock_unlock(&database->rwlock);
        
        usleep(50000);  // Пауза между чтениями
    }
    
    return NULL;
}

void* writer_thread(void* arg) {
    int writer_id = *(int*)arg;
    
    for (int i = 0; i < 5; i++) {
        // Захват для записи (эксклюзивный)
        pthread_rwlock_wrlock(&database->rwlock);
        
        printf("[Writer %d] Обновление базы данных...\n", writer_id);
        
        // Модификация данных
        database->version++;
        for (int j = 0; j < 1000; j++) {
            database->data[j] = writer_id * 1000 + i * 100 + j;
        }
        
        printf("[Writer %d] База обновлена до версии %d\n", 
               writer_id, database->version);
        
        pthread_rwlock_unlock(&database->rwlock);
        
        usleep(200000);  // Пауза между записями
    }
    
    return NULL;
}

void rwlock_demo() {
    database = malloc(sizeof(shared_database_t));
    pthread_rwlock_init(&database->rwlock, NULL);
    database->version = 0;
    
    const int num_readers = 3;
    const int num_writers = 2;
    
    pthread_t readers[num_readers];
    pthread_t writers[num_writers];
    int reader_ids[num_readers];
    int writer_ids[num_writers];
    
    // Создание читателей
    for (int i = 0; i < num_readers; i++) {
        reader_ids[i] = i;
        pthread_create(&readers[i], NULL, reader_thread, &reader_ids[i]);
    }
    
    // Создание писателей
    for (int i = 0; i < num_writers; i++) {
        writer_ids[i] = i;
        pthread_create(&writers[i], NULL, writer_thread, &writer_ids[i]);
    }
    
    // Ожидание завершения
    for (int i = 0; i < num_readers; i++) {
        pthread_join(readers[i], NULL);
    }
    for (int i = 0; i < num_writers; i++) {
        pthread_join(writers[i], NULL);
    }
    
    pthread_rwlock_destroy(&database->rwlock);
    free(database);
}
```

### Condition variables

```c
// Producer-Consumer с condition variables
typedef struct buffer {
    int* data;
    int size;
    int count;
    int in;
    int out;
    pthread_mutex_t mutex;
    pthread_cond_t not_full;
    pthread_cond_t not_empty;
} buffer_t;

buffer_t* create_buffer(int size) {
    buffer_t* buf = malloc(sizeof(buffer_t));
    buf->data = malloc(sizeof(int) * size);
    buf->size = size;
    buf->count = 0;
    buf->in = 0;
    buf->out = 0;
    
    pthread_mutex_init(&buf->mutex, NULL);
    pthread_cond_init(&buf->not_full, NULL);
    pthread_cond_init(&buf->not_empty, NULL);
    
    return buf;
}

void buffer_put(buffer_t* buf, int item) {
    pthread_mutex_lock(&buf->mutex);
    
    // Ждем, пока буфер не освободится
    while (buf->count == buf->size) {
        printf("Буфер полон, производитель ждет...\n");
        pthread_cond_wait(&buf->not_full, &buf->mutex);
    }
    
    // Добавляем элемент
    buf->data[buf->in] = item;
    buf->in = (buf->in + 1) % buf->size;
    buf->count++;
    
    printf("Произведен элемент %d (буфер: %d/%d)\n", 
           item, buf->count, buf->size);
    
    // Уведомляем потребителей
    pthread_cond_signal(&buf->not_empty);
    
    pthread_mutex_unlock(&buf->mutex);
}

int buffer_get(buffer_t* buf) {
    pthread_mutex_lock(&buf->mutex);
    
    // Ждем, пока в буфере появятся данные
    while (buf->count == 0) {
        printf("Буфер пуст, потребитель ждет...\n");
        pthread_cond_wait(&buf->not_empty, &buf->mutex);
    }
    
    // Извлекаем элемент
    int item = buf->data[buf->out];
    buf->out = (buf->out + 1) % buf->size;
    buf->count--;
    
    printf("Потреблен элемент %d (буфер: %d/%d)\n", 
           item, buf->count, buf->size);
    
    // Уведомляем производителей
    pthread_cond_signal(&buf->not_full);
    
    pthread_mutex_unlock(&buf->mutex);
    
    return item;
}
```

---

## 📞 Inter-Process Communication

### Shared Memory (POSIX)

```c
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>

// Структура в разделяемой памяти
typedef struct shared_data {
    pthread_mutex_t mutex;
    pthread_cond_t condition;
    int counter;
    int ready;
    char message[256];
} shared_data_t;

void shared_memory_demo() {
    const char* shm_name = "/my_shared_memory";
    const size_t shm_size = sizeof(shared_data_t);
    
    // Создание/открытие разделяемой памяти
    int shm_fd = shm_open(shm_name, O_CREAT | O_RDWR, 0666);
    if (shm_fd == -1) {
        perror("shm_open failed");
        return;
    }
    
    // Установка размера
    if (ftruncate(shm_fd, shm_size) == -1) {
        perror("ftruncate failed");
        return;
    }
    
    // Mapping в адресное пространство
    shared_data_t* shared = mmap(NULL, shm_size,
                                PROT_READ | PROT_WRITE,
                                MAP_SHARED, shm_fd, 0);
    
    if (shared == MAP_FAILED) {
        perror("mmap failed");
        return;
    }
    
    // Инициализация синхронизации (только один раз)
    static int initialized = 0;
    if (!initialized) {
        // Атрибуты для процессов
        pthread_mutexattr_t mutex_attr;
        pthread_mutexattr_init(&mutex_attr);
        pthread_mutexattr_setpshared(&mutex_attr, PTHREAD_PROCESS_SHARED);
        
        pthread_condattr_t cond_attr;
        pthread_condattr_init(&cond_attr);
        pthread_condattr_setpshared(&cond_attr, PTHREAD_PROCESS_SHARED);
        
        pthread_mutex_init(&shared->mutex, &mutex_attr);
        pthread_cond_init(&shared->condition, &cond_attr);
        
        shared->counter = 0;
        shared->ready = 0;
        
        pthread_mutexattr_destroy(&mutex_attr);
        pthread_condattr_destroy(&cond_attr);
        
        initialized = 1;
    }
    
    pid_t pid = fork();
    if (pid == 0) {
        // Дочерний процесс - производитель
        for (int i = 0; i < 10; i++) {
            pthread_mutex_lock(&shared->mutex);
            
            shared->counter = i;
            snprintf(shared->message, sizeof(shared->message), 
                    "Message %d from child process", i);
            shared->ready = 1;
            
            printf("[Child] Отправлено: %s\n", shared->message);
            
            pthread_cond_signal(&shared->condition);
            pthread_mutex_unlock(&shared->mutex);
            
            sleep(1);
        }
        
        exit(0);
    } else {
        // Родительский процесс - потребитель
        for (int i = 0; i < 10; i++) {
            pthread_mutex_lock(&shared->mutex);
            
            while (!shared->ready) {
                pthread_cond_wait(&shared->condition, &shared->mutex);
            }
            
            printf("[Parent] Получено: %s (counter: %d)\n", 
                   shared->message, shared->counter);
            
            shared->ready = 0;
            pthread_mutex_unlock(&shared->mutex);
        }
        
        wait(NULL);  // Ждем завершения дочернего процесса
    }
    
    // Cleanup
    munmap(shared, shm_size);
    close(shm_fd);
    shm_unlink(shm_name);
}
```

### Message Queues (POSIX)

```c
#include <mqueue.h>

void message_queue_demo() {
    const char* queue_name = "/my_message_queue";
    struct mq_attr attr;
    
    // Настройка атрибутов очереди
    attr.mq_flags = 0;
    attr.mq_maxmsg = 10;      // Максимум сообщений
    attr.mq_msgsize = 256;    // Размер сообщения
    attr.mq_curmsgs = 0;
    
    // Создание/открытие очереди
    mqd_t mq = mq_open(queue_name, O_CREAT | O_RDWR, 0666, &attr);
    if (mq == (mqd_t)-1) {
        perror("mq_open failed");
        return;
    }
    
    pid_t pid = fork();
    if (pid == 0) {
        // Дочерний процесс - отправитель
        for (int i = 0; i < 5; i++) {
            char message[256];
            snprintf(message, sizeof(message), "Message %d from child", i);
            
            if (mq_send(mq, message, strlen(message) + 1, i) == -1) {
                perror("mq_send failed");
                break;
            }
            
            printf("[Child] Отправлено: %s (приоритет: %d)\n", message, i);
            sleep(1);
        }
        
        exit(0);
    } else {
        // Родительский процесс - получатель
        char buffer[256];
        unsigned int priority;
        
        for (int i = 0; i < 5; i++) {
            ssize_t bytes_read = mq_receive(mq, buffer, sizeof(buffer), &priority);
            if (bytes_read >= 0) {
                buffer[bytes_read] = '\0';
                printf("[Parent] Получено: %s (приоритет: %u)\n", 
                       buffer, priority);
            }
        }
        
        wait(NULL);
    }
    
    // Cleanup
    mq_close(mq);
    mq_unlink(queue_name);
}
```

---

## ⚡ Планировщик процессов

### CPU affinity и приоритеты

```c
#include <sched.h>

void cpu_affinity_demo() {
    printf("=== CPU Affinity и планирование ===\n\n");
    
    // Получение количества CPU
    int num_cpus = sysconf(_SC_NPROCESSORS_ONLN);
    printf("Доступно CPU: %d\n", num_cpus);
    
    // Получение текущей CPU affinity
    cpu_set_t current_mask;
    CPU_ZERO(&current_mask);
    
    if (sched_getaffinity(0, sizeof(current_mask), &current_mask) == 0) {
        printf("Текущая CPU affinity: ");
        for (int i = 0; i < num_cpus; i++) {
            if (CPU_ISSET(i, &current_mask)) {
                printf("%d ", i);
            }
        }
        printf("\n");
    }
    
    // Установка affinity на конкретный CPU
    cpu_set_t new_mask;
    CPU_ZERO(&new_mask);
    CPU_SET(0, &new_mask);  // Привязка к CPU 0
    
    if (sched_setaffinity(0, sizeof(new_mask), &new_mask) == 0) {
        printf("Процесс привязан к CPU 0\n");
    }
    
    // Информация о планировщике
    int policy = sched_getscheduler(0);
    struct sched_param param;
    sched_getparam(0, &param);
    
    printf("Политика планирования: ");
    switch (policy) {
        case SCHED_OTHER: printf("SCHED_OTHER (CFS)\n"); break;
        case SCHED_FIFO:  printf("SCHED_FIFO (RT)\n"); break;
        case SCHED_RR:    printf("SCHED_RR (RT)\n"); break;
        default:          printf("Unknown (%d)\n", policy); break;
    }
    
    printf("Приоритет: %d\n", param.sched_priority);
    printf("Nice value: %d\n", getpriority(PRIO_PROCESS, 0));
}

// Демонстрация RT планирования
void realtime_scheduling_demo() {
    printf("\n=== Real-time планирование ===\n");
    
    // Проверка прав для RT
    if (geteuid() != 0) {
        printf("Нужны root права для RT планирования\n");
        return;
    }
    
    struct sched_param param;
    
    // Установка FIFO планирования с высоким приоритетом
    param.sched_priority = 50;
    if (sched_setscheduler(0, SCHED_FIFO, &param) == 0) {
        printf("Установлен SCHED_FIFO с приоритетом %d\n", param.sched_priority);
        
        // Критическая секция реального времени
        struct timespec start, end;
        clock_gettime(CLOCK_MONOTONIC, &start);
        
        // Интенсивная работа
        volatile int counter = 0;
        for (int i = 0; i < 1000000; i++) {
            counter += i;
        }
        
        clock_gettime(CLOCK_MONOTONIC, &end);
        
        long elapsed_ns = (end.tv_sec - start.tv_sec) * 1000000000L + 
                         (end.tv_nsec - start.tv_nsec);
        
        printf("RT задача выполнена за %ld ns\n", elapsed_ns);
        
        // Возврат к обычному планированию
        param.sched_priority = 0;
        sched_setscheduler(0, SCHED_OTHER, &param);
    }
}
```

---

## 📊 Производительность многопоточности

### Измерение overhead'а потоков

```c
#include <time.h>

void threading_overhead_benchmark() {
    printf("=== Benchmark производительности потоков ===\n\n");
    
    const int num_operations = 1000000;
    struct timespec start, end;
    
    // 1. Последовательное выполнение
    clock_gettime(CLOCK_MONOTONIC, &start);
    
    volatile int sequential_result = 0;
    for (int i = 0; i < num_operations; i++) {
        sequential_result += i * i;
    }
    
    clock_gettime(CLOCK_MONOTONIC, &end);
    long sequential_time = (end.tv_sec - start.tv_sec) * 1000000000L + 
                          (end.tv_nsec - start.tv_nsec);
    
    printf("Последовательно: %ld ns (%ld ops/sec)\n", 
           sequential_time, num_operations * 1000000000L / sequential_time);
    
    // 2. Многопоточное выполнение
    const int num_threads = 4;
    pthread_t threads[num_threads];
    volatile int thread_results[num_threads];
    
    typedef struct {
        int thread_id;
        int start_idx;
        int end_idx;
        volatile int* result;
    } thread_work_t;
    
    thread_work_t work_data[num_threads];
    
    void* thread_work(void* arg) {
        thread_work_t* work = (thread_work_t*)arg;
        *work->result = 0;
        
        for (int i = work->start_idx; i < work->end_idx; i++) {
            *work->result += i * i;
        }
        
        return NULL;
    }
    
    clock_gettime(CLOCK_MONOTONIC, &start);
    
    // Создание потоков
    int chunk_size = num_operations / num_threads;
    for (int i = 0; i < num_threads; i++) {
        work_data[i].thread_id = i;
        work_data[i].start_idx = i * chunk_size;
        work_data[i].end_idx = (i == num_threads - 1) ? 
                               num_operations : (i + 1) * chunk_size;
        work_data[i].result = &thread_results[i];
        
        pthread_create(&threads[i], NULL, thread_work, &work_data[i]);
    }
    
    // Ожидание завершения
    for (int i = 0; i < num_threads; i++) {
        pthread_join(threads[i], NULL);
    }
    
    clock_gettime(CLOCK_MONOTONIC, &end);
    long parallel_time = (end.tv_sec - start.tv_sec) * 1000000000L + 
                        (end.tv_nsec - start.tv_nsec);
    
    printf("Параллельно (%d потоков): %ld ns (%ld ops/sec)\n", 
           num_threads, parallel_time, 
           num_operations * 1000000000L / parallel_time);
    
    printf("Ускорение: %.2fx\n", (double)sequential_time / parallel_time);
    printf("Эффективность: %.1f%%\n", 
           (double)sequential_time / parallel_time / num_threads * 100);
}

// Lock contention benchmark
atomic_int atomic_counter = 0;
pthread_mutex_t mutex_counter_lock = PTHREAD_MUTEX_INITIALIZER;
int mutex_counter = 0;

void* atomic_increment_worker(void* arg) {
    int iterations = *(int*)arg;
    
    for (int i = 0; i < iterations; i++) {
        atomic_fetch_add(&atomic_counter, 1);
    }
    
    return NULL;
}

void* mutex_increment_worker(void* arg) {
    int iterations = *(int*)arg;
    
    for (int i = 0; i < iterations; i++) {
        pthread_mutex_lock(&mutex_counter_lock);
        mutex_counter++;
        pthread_mutex_unlock(&mutex_counter_lock);
    }
    
    return NULL;
}

void lock_contention_benchmark() {
    printf("\n=== Lock Contention Benchmark ===\n");
    
    const int num_threads = 8;
    const int iterations_per_thread = 100000;
    
    pthread_t threads[num_threads];
    struct timespec start, end;
    
    // 1. Atomic operations
    atomic_counter = 0;
    clock_gettime(CLOCK_MONOTONIC, &start);
    
    for (int i = 0; i < num_threads; i++) {
        int* arg = malloc(sizeof(int));
        *arg = iterations_per_thread;
        pthread_create(&threads[i], NULL, atomic_increment_worker, arg);
    }
    
    for (int i = 0; i < num_threads; i++) {
        void* result;
        pthread_join(threads[i], &result);
        free(result);
    }
    
    clock_gettime(CLOCK_MONOTONIC, &end);
    long atomic_time = (end.tv_sec - start.tv_sec) * 1000000000L + 
                      (end.tv_nsec - start.tv_nsec);
    
    printf("Atomic operations: %ld ns (результат: %d)\n", 
           atomic_time, atomic_counter);
    
    // 2. Mutex operations
    mutex_counter = 0;
    clock_gettime(CLOCK_MONOTONIC, &start);
    
    for (int i = 0; i < num_threads; i++) {
        int* arg = malloc(sizeof(int));
        *arg = iterations_per_thread;
        pthread_create(&threads[i], NULL, mutex_increment_worker, arg);
    }
    
    for (int i = 0; i < num_threads; i++) {
        void* result;
        pthread_join(threads[i], &result);
        free(result);
    }
    
    clock_gettime(CLOCK_MONOTONIC, &end);
    long mutex_time = (end.tv_sec - start.tv_sec) * 1000000000L + 
                     (end.tv_nsec - start.tv_nsec);
    
    printf("Mutex operations: %ld ns (результат: %d)\n", 
           mutex_time, mutex_counter);
    
    printf("Atomic vs Mutex: %.2fx быстрее\n", 
           (double)mutex_time / atomic_time);
}
```

---

## 🔗 Связанные темы

- [[memory-management|Управление памятью]] - процессы как контейнеры памяти
- [[network-programming|Сетевое программирование]] - многопоточные серверы
- [[../computer-architecture/parallel-architectures|Параллельные архитектуры]] - аппаратная поддержка

## 📚 Дополнительные ресурсы

### 📖 Книги
- "Operating System Concepts" - Silberschatz, Galvin & Gagne
- "Modern Operating Systems" - Andrew S. Tanenbaum
- "Programming with POSIX Threads" - David R. Butenhof

### 🛠️ Инструменты
- **perf** - профилирование производительности
- **htop** - мониторинг процессов и потоков
- **strace** - трассировка системных вызовов
- **gdb** - отладка многопоточных программ

---

> **Следующий шаг**: Изучите [[network-programming|сетевое программирование]] для создания распределенных систем. 