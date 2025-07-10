# Иерархия памяти 🧠

> **Навигация**: [[README|← Архитектура компьютеров]] | [[processor-architectures|← Архитектуры процессоров]] | [[microarchitecture|Микроархитектура →]]

## Содержание
1. [Основы иерархии памяти](#🏗️-основы-иерархии-памяти)
2. [Кэш-память](#💾-кэш-память)
3. [Виртуальная память](#🌐-виртуальная-память)
4. [TLB (Translation Lookaside Buffer)](#⚡-tlb-translation-lookaside-buffer)
5. [Оптимизации доступа к памяти](#🚀-оптимизации-доступа-к-памяти)
6. [Практические применения](#💼-практические-применения)

---

## 🏗️ Основы иерархии памяти

### Принципы иерархии

**Пирамида памяти** (от быстрой к медленной):
1. **Регистры** - сотни байт, 1 цикл
2. **L1 Cache** - 32-64 KB, 1-2 цикла
3. **L2 Cache** - 256-512 KB, 10-20 циклов
4. **L3 Cache** - 8-32 MB, 50-100 циклов
5. **RAM** - GB, 200-400 циклов
6. **SSD** - TB, миллионы циклов
7. **HDD** - TB, десятки миллионов циклов

**Принципы локальности**:
- **Временная локальность** - недавно использованные данные вероятно понадобятся снова
- **Пространственная локальность** - данные рядом с использованными вероятно понадобятся

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

void locality_demonstration() {
    printf("=== Демонстрация принципов локальности ===\n");
    
    const int SIZE = 1024 * 1024;  // 1M элементов
    int* array = malloc(SIZE * sizeof(int));
    
    // Инициализация
    for (int i = 0; i < SIZE; i++) {
        array[i] = i;
    }
    
    clock_t start, end;
    
    // 1. Пространственная локальность (последовательный доступ)
    printf("\n1. Пространственная локальность:\n");
    start = clock();
    long sum1 = 0;
    for (int i = 0; i < SIZE; i++) {
        sum1 += array[i];  // Последовательный доступ
    }
    end = clock();
    double spatial_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    // 2. Временная локальность (повторное использование)
    printf("2. Временная локальность:\n");
    start = clock();
    long sum2 = 0;
    for (int pass = 0; pass < 3; pass++) {
        for (int i = 0; i < SIZE/3; i++) {
            sum2 += array[i];  // Повторный доступ к тем же данным
        }
    }
    end = clock();
    double temporal_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    // 3. Плохая локальность (случайный доступ)
    printf("3. Отсутствие локальности:\n");
    srand(42);
    start = clock();
    long sum3 = 0;
    for (int i = 0; i < SIZE; i++) {
        int index = rand() % SIZE;
        sum3 += array[index];  // Случайный доступ
    }
    end = clock();
    double random_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    printf("\nРезультаты времени выполнения:\n");
    printf("Пространственная локальность: %.4f сек\n", spatial_time);
    printf("Временная локальность: %.4f сек\n", temporal_time);
    printf("Случайный доступ: %.4f сек\n", random_time);
    printf("Замедление случайного доступа: %.2fx\n", random_time / spatial_time);
    
    free(array);
}
```

---

## 💾 Кэш-память

### Структура и организация кэша

```c
#include <string.h>

// Симуляция простого кэша
typedef struct {
    int valid;
    int tag;
    char data[64];  // Блок данных
    int lru_counter; // Для LRU замещения
} CacheBlock;

typedef struct {
    CacheBlock* blocks;
    int num_sets;
    int associativity;
    int block_size;
    int hits;
    int misses;
    int access_count;
} Cache;

Cache* create_cache(int num_sets, int associativity, int block_size) {
    Cache* cache = malloc(sizeof(Cache));
    cache->blocks = calloc(num_sets * associativity, sizeof(CacheBlock));
    cache->num_sets = num_sets;
    cache->associativity = associativity;
    cache->block_size = block_size;
    cache->hits = cache->misses = cache->access_count = 0;
    return cache;
}

int cache_access(Cache* cache, uint32_t address) {
    cache->access_count++;
    
    // Разбор адреса
    int block_offset = address % cache->block_size;
    int set_index = (address / cache->block_size) % cache->num_sets;
    int tag = address / (cache->block_size * cache->num_sets);
    
    // Поиск в наборе
    int set_start = set_index * cache->associativity;
    for (int i = 0; i < cache->associativity; i++) {
        CacheBlock* block = &cache->blocks[set_start + i];
        if (block->valid && block->tag == tag) {
            // Cache hit
            cache->hits++;
            block->lru_counter = cache->access_count;
            return 1;
        }
    }
    
    // Cache miss - находим блок для замещения
    cache->misses++;
    int replace_idx = set_start;
    int oldest_access = cache->blocks[set_start].lru_counter;
    
    for (int i = 1; i < cache->associativity; i++) {
        CacheBlock* block = &cache->blocks[set_start + i];
        if (!block->valid) {
            // Свободный блок
            replace_idx = set_start + i;
            break;
        }
        if (block->lru_counter < oldest_access) {
            oldest_access = block->lru_counter;
            replace_idx = set_start + i;
        }
    }
    
    // Загружаем новый блок
    CacheBlock* new_block = &cache->blocks[replace_idx];
    new_block->valid = 1;
    new_block->tag = tag;
    new_block->lru_counter = cache->access_count;
    
    return 0; // Cache miss
}

void cache_simulation_demo() {
    printf("\n=== Симуляция работы кэша ===\n");
    
    // Создаем кэш: 4 набора, 2-way ассоциативность, блок 64 байта
    Cache* cache = create_cache(4, 2, 64);
    
    // Последовательность адресов для тестирования
    uint32_t addresses[] = {
        0x1000, 0x1040, 0x1080,  // Разные блоки, один набор
        0x1000,                  // Повторный доступ - hit
        0x2000,                  // Другой набор
        0x3000, 0x4000,          // Заполняем набор 0
        0x5000,                  // Вытеснение из набора 0
        0x1000                   // Должен быть miss
    };
    
    int num_addresses = sizeof(addresses) / sizeof(addresses[0]);
    
    printf("Конфигурация кэша:\n");
    printf("- Наборов: %d\n", cache->num_sets);
    printf("- Ассоциативность: %d\n", cache->associativity);
    printf("- Размер блока: %d байт\n", cache->block_size);
    printf("\nПоследовательность доступов:\n");
    
    for (int i = 0; i < num_addresses; i++) {
        int hit = cache_access(cache, addresses[i]);
        printf("Адрес 0x%08X: %s\n", addresses[i], hit ? "HIT" : "MISS");
    }
    
    printf("\nСтатистика кэша:\n");
    printf("Обращений: %d\n", cache->access_count);
    printf("Попаданий: %d\n", cache->hits);
    printf("Промахов: %d\n", cache->misses);
    printf("Hit Rate: %.2f%%\n", (double)cache->hits / cache->access_count * 100);
    
    free(cache->blocks);
    free(cache);
}
```

### Cache-friendly алгоритмы

```c
// Демонстрация cache-friendly vs cache-unfriendly алгоритмов
void matrix_multiplication_cache() {
    printf("\n=== Умножение матриц: влияние кэша ===\n");
    
    const int N = 512;
    double** A = malloc(N * sizeof(double*));
    double** B = malloc(N * sizeof(double*));
    double** C1 = malloc(N * sizeof(double*));
    double** C2 = malloc(N * sizeof(double*));
    
    for (int i = 0; i < N; i++) {
        A[i] = malloc(N * sizeof(double));
        B[i] = malloc(N * sizeof(double));
        C1[i] = malloc(N * sizeof(double));
        C2[i] = malloc(N * sizeof(double));
        
        for (int j = 0; j < N; j++) {
            A[i][j] = i + j;
            B[i][j] = i * j;
            C1[i][j] = C2[i][j] = 0;
        }
    }
    
    clock_t start, end;
    
    // 1. Наивная реализация (плохая для кэша)
    printf("1. Наивная реализация (i-j-k):\n");
    start = clock();
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            for (int k = 0; k < N; k++) {
                C1[i][j] += A[i][k] * B[k][j];  // Плохой доступ к B
            }
        }
    }
    end = clock();
    double naive_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    // 2. Cache-friendly версия (блочное умножение)
    printf("2. Блочное умножение:\n");
    const int BLOCK_SIZE = 64;
    start = clock();
    for (int ii = 0; ii < N; ii += BLOCK_SIZE) {
        for (int jj = 0; jj < N; jj += BLOCK_SIZE) {
            for (int kk = 0; kk < N; kk += BLOCK_SIZE) {
                // Работаем с блоками
                for (int i = ii; i < ii + BLOCK_SIZE && i < N; i++) {
                    for (int j = jj; j < jj + BLOCK_SIZE && j < N; j++) {
                        for (int k = kk; k < kk + BLOCK_SIZE && k < N; k++) {
                            C2[i][j] += A[i][k] * B[k][j];
                        }
                    }
                }
            }
        }
    }
    end = clock();
    double blocked_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    printf("\nРезультаты производительности:\n");
    printf("Наивное умножение: %.4f сек\n", naive_time);
    printf("Блочное умножение: %.4f сек\n", blocked_time);
    printf("Ускорение: %.2fx\n", naive_time / blocked_time);
    
    // Освобождение памяти
    for (int i = 0; i < N; i++) {
        free(A[i]); free(B[i]); free(C1[i]); free(C2[i]);
    }
    free(A); free(B); free(C1); free(C2);
}
```

---

## 🌐 Виртуальная память

### Основы виртуальной памяти

```c
#include <sys/mman.h>
#include <unistd.h>
#include <signal.h>

void virtual_memory_demo() {
    printf("\n=== Виртуальная память ===\n");
    
    size_t page_size = getpagesize();
    printf("Размер страницы: %zu байт (%zu KB)\n", page_size, page_size/1024);
    
    // Выделение памяти с помощью mmap
    size_t size = page_size * 4;  // 4 страницы
    void* ptr = mmap(NULL, size, 
                     PROT_READ | PROT_WRITE,
                     MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    
    if (ptr == MAP_FAILED) {
        perror("mmap failed");
        return;
    }
    
    printf("Выделено %zu байт по виртуальному адресу %p\n", size, ptr);
    
    // Запись данных (вызывает page fault при первом обращении)
    char* buffer = (char*)ptr;
    for (size_t i = 0; i < size; i += page_size) {
        printf("Доступ к странице %zu (offset 0x%zx)...\n", 
               i / page_size, i);
        
        buffer[i] = 'A' + (i / page_size);  // Первый байт каждой страницы
        printf("  Записан символ '%c' - страница загружена в физическую память\n", 
               buffer[i]);
    }
    
    // Демонстрация изменения защиты памяти
    printf("\nИзменение защиты первой страницы на только чтение...\n");
    if (mprotect(ptr, page_size, PROT_READ) == 0) {
        printf("Первая страница теперь только для чтения\n");
        printf("Чтение: '%c' - успешно\n", buffer[0]);
        
        // Попытка записи вызовет SIGSEGV
        printf("Попытка записи в защищенную страницу вызовет ошибку\n");
        // buffer[0] = 'X';  // Раскомментировать для демонстрации SIGSEGV
    }
    
    // Освобождение памяти
    munmap(ptr, size);
    printf("Виртуальная память освобождена\n");
}

// Демонстрация copy-on-write
void cow_demo() {
    printf("\n=== Copy-on-Write демонстрация ===\n");
    
    size_t size = getpagesize() * 2;
    char* shared_memory = mmap(NULL, size,
                              PROT_READ | PROT_WRITE,
                              MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    
    if (shared_memory == MAP_FAILED) {
        perror("mmap failed");
        return;
    }
    
    // Записываем данные в родительском процессе
    strcpy(shared_memory, "Исходные данные от родителя");
    printf("Родитель записал: '%s'\n", shared_memory);
    
    pid_t pid = fork();
    
    if (pid == 0) {
        // Дочерний процесс
        printf("Дочерний процесс читает: '%s'\n", shared_memory);
        printf("Дочерний процесс изменяет данные (trigger COW)...\n");
        
        strcpy(shared_memory, "Измененные данные от ребенка");
        printf("Дочерний процесс записал: '%s'\n", shared_memory);
        
        exit(0);
    } else if (pid > 0) {
        // Родительский процесс
        wait(NULL);  // Ждем завершения дочернего процесса
        printf("Родитель после изменений ребенка читает: '%s'\n", shared_memory);
        printf("COW обеспечил независимые копии памяти\n");
    }
    
    munmap(shared_memory, size);
}
```

---

## ⚡ TLB (Translation Lookaside Buffer)

### Симуляция TLB

```c
// Симуляция TLB (Translation Lookaside Buffer)
typedef struct {
    uint64_t virtual_page;
    uint64_t physical_page;
    int valid;
    int recent;  // Для LRU
} TLBEntry;

typedef struct {
    TLBEntry entries[64];  // 64-entry TLB
    int access_count;
    int hit_count;
    int miss_count;
} TLB;

TLB* create_tlb() {
    TLB* tlb = calloc(1, sizeof(TLB));
    return tlb;
}

uint64_t translate_address(TLB* tlb, uint64_t virtual_addr) {
    uint64_t virtual_page = virtual_addr >> 12;  // Предполагаем 4KB страницы
    uint64_t offset = virtual_addr & 0xFFF;
    
    tlb->access_count++;
    
    // Поиск в TLB
    for (int i = 0; i < 64; i++) {
        if (tlb->entries[i].valid && 
            tlb->entries[i].virtual_page == virtual_page) {
            // TLB hit
            tlb->hit_count++;
            tlb->entries[i].recent = tlb->access_count;
            return (tlb->entries[i].physical_page << 12) | offset;
        }
    }
    
    // TLB miss - имитируем обращение к таблице страниц
    tlb->miss_count++;
    uint64_t physical_page = virtual_page ^ 0xDEADBEEF;  // Простая "трансляция"
    
    // Добавляем в TLB (простейшая LRU)
    int oldest_idx = 0;
    int oldest_time = tlb->entries[0].recent;
    
    for (int i = 1; i < 64; i++) {
        if (!tlb->entries[i].valid) {
            oldest_idx = i;
            break;
        }
        if (tlb->entries[i].recent < oldest_time) {
            oldest_time = tlb->entries[i].recent;
            oldest_idx = i;
        }
    }
    
    tlb->entries[oldest_idx].virtual_page = virtual_page;
    tlb->entries[oldest_idx].physical_page = physical_page;
    tlb->entries[oldest_idx].valid = 1;
    tlb->entries[oldest_idx].recent = tlb->access_count;
    
    return (physical_page << 12) | offset;
}

void tlb_simulation() {
    printf("\n=== Симуляция TLB ===\n");
    
    TLB* tlb = create_tlb();
    
    // Симулируем последовательность обращений к памяти
    uint64_t addresses[] = {
        0x1000, 0x1004, 0x1008, 0x100C,  // Одна страница
        0x2000, 0x2004,                  // Новая страница
        0x1010, 0x1014,                  // Возврат к первой странице - hits
        0x3000, 0x4000, 0x5000,          // Новые страницы
        0x6000, 0x7000, 0x8000,          // Больше страниц
        0x1020,                          // Возможно вытеснена из TLB
    };
    
    int num_addresses = sizeof(addresses) / sizeof(addresses[0]);
    
    printf("Последовательность трансляций адресов:\n");
    printf("%-12s %-16s %-16s %s\n", 
           "Виртуальный", "Физический", "Страница", "Результат");
    printf("%-12s %-16s %-16s %s\n", 
           "адрес", "адрес", "(VPN->PPN)", "");
    
    for (int i = 0; i < num_addresses; i++) {
        int prev_hits = tlb->hit_count;
        uint64_t physical = translate_address(tlb, addresses[i]);
        int hit = (tlb->hit_count > prev_hits);
        
        printf("0x%-10lX 0x%-14lX 0x%lX->0x%lX %s\n", 
               addresses[i], physical,
               addresses[i] >> 12, physical >> 12,
               hit ? "TLB HIT" : "TLB MISS");
    }
    
    printf("\nСтатистика TLB:\n");
    printf("Обращений: %d\n", tlb->access_count);
    printf("Попаданий: %d\n", tlb->hit_count);
    printf("Промахов: %d\n", tlb->miss_count);
    printf("Hit Rate: %.2f%%\n", (double)tlb->hit_count / tlb->access_count * 100);
    
    free(tlb);
}
```

---

## 🚀 Оптимизации доступа к памяти

### Memory Pool

```c
// Реализация memory pool для оптимизации аллокаций
typedef struct Block {
    struct Block* next;
} Block;

typedef struct {
    char* memory;
    Block* free_list;
    size_t block_size;
    size_t total_blocks;
    size_t allocated_blocks;
} MemoryPool;

MemoryPool* create_memory_pool(size_t block_size, size_t num_blocks) {
    MemoryPool* pool = malloc(sizeof(MemoryPool));
    pool->block_size = block_size;
    pool->total_blocks = num_blocks;
    pool->allocated_blocks = 0;
    
    // Выделяем непрерывный блок памяти
    pool->memory = malloc(block_size * num_blocks);
    
    // Инициализируем список свободных блоков
    pool->free_list = NULL;
    for (size_t i = 0; i < num_blocks; i++) {
        Block* block = (Block*)(pool->memory + i * block_size);
        block->next = pool->free_list;
        pool->free_list = block;
    }
    
    return pool;
}

void* pool_alloc(MemoryPool* pool) {
    if (pool->free_list == NULL) {
        return NULL;  // Pool исчерпан
    }
    
    Block* block = pool->free_list;
    pool->free_list = block->next;
    pool->allocated_blocks++;
    
    return (void*)block;
}

void pool_free(MemoryPool* pool, void* ptr) {
    Block* block = (Block*)ptr;
    block->next = pool->free_list;
    pool->free_list = block;
    pool->allocated_blocks--;
}

void memory_pool_demo() {
    printf("\n=== Memory Pool vs malloc ===\n");
    
    const int NUM_ALLOCATIONS = 100000;
    const int BLOCK_SIZE = 64;
    
    // Тест обычного malloc
    clock_t start = clock();
    void** ptrs = malloc(NUM_ALLOCATIONS * sizeof(void*));
    
    for (int i = 0; i < NUM_ALLOCATIONS; i++) {
        ptrs[i] = malloc(BLOCK_SIZE);
    }
    
    for (int i = 0; i < NUM_ALLOCATIONS; i++) {
        free(ptrs[i]);
    }
    
    clock_t end = clock();
    double malloc_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    // Тест memory pool
    MemoryPool* pool = create_memory_pool(BLOCK_SIZE, NUM_ALLOCATIONS);
    
    start = clock();
    
    for (int i = 0; i < NUM_ALLOCATIONS; i++) {
        ptrs[i] = pool_alloc(pool);
    }
    
    for (int i = 0; i < NUM_ALLOCATIONS; i++) {
        pool_free(pool, ptrs[i]);
    }
    
    end = clock();
    double pool_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    printf("Аллокаций: %d по %d байт\n", NUM_ALLOCATIONS, BLOCK_SIZE);
    printf("malloc/free время: %.6f сек\n", malloc_time);
    printf("Memory pool время: %.6f сек\n", pool_time);
    printf("Ускорение: %.2fx\n", malloc_time / pool_time);
    
    free(ptrs);
    free(pool->memory);
    free(pool);
}
```

### Prefetching

```c
// Демонстрация software prefetching
void prefetching_demo() {
    printf("\n=== Software Prefetching ===\n");
    
    const int SIZE = 1024 * 1024;
    int* array = malloc(SIZE * sizeof(int));
    
    // Инициализация случайных индексов
    int* indices = malloc(SIZE * sizeof(int));
    for (int i = 0; i < SIZE; i++) {
        indices[i] = (i * 12345) % SIZE;  // Псевдослучайная последовательность
        array[i] = i;
    }
    
    clock_t start, end;
    
    // Без prefetching
    start = clock();
    long sum1 = 0;
    for (int i = 0; i < SIZE; i++) {
        sum1 += array[indices[i]];
    }
    end = clock();
    double no_prefetch_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    // С software prefetching
    start = clock();
    long sum2 = 0;
    const int PREFETCH_DISTANCE = 64;  // Предзагрузка на 64 итерации вперед
    
    for (int i = 0; i < SIZE; i++) {
        // Предзагружаем данные для будущих итераций
        if (i + PREFETCH_DISTANCE < SIZE) {
            __builtin_prefetch(&array[indices[i + PREFETCH_DISTANCE]], 0, 3);
        }
        
        sum2 += array[indices[i]];
    }
    end = clock();
    double prefetch_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    printf("Обработано элементов: %d\n", SIZE);
    printf("Без prefetching: %.4f сек (sum = %ld)\n", no_prefetch_time, sum1);
    printf("С prefetching: %.4f сек (sum = %ld)\n", prefetch_time, sum2);
    printf("Ускорение: %.2fx\n", no_prefetch_time / prefetch_time);
    
    free(array);
    free(indices);
}
```

---

## 💼 Практические применения

### В базах данных

```c
// Оптимизация доступа к данным в стиле СУБД
typedef struct {
    int id;
    char name[32];
    int age;
    double salary;
} Employee;

void database_cache_optimization() {
    printf("\n=== Оптимизация доступа к данным (СУБД стиль) ===\n");
    
    const int NUM_EMPLOYEES = 100000;
    Employee* employees = malloc(NUM_EMPLOYEES * sizeof(Employee));
    
    // Инициализация данных
    for (int i = 0; i < NUM_EMPLOYEES; i++) {
        employees[i].id = i;
        sprintf(employees[i].name, "Employee_%d", i);
        employees[i].age = 25 + (i % 40);
        employees[i].salary = 50000 + (i % 100000);
    }
    
    clock_t start, end;
    
    // 1. Плохой паттерн: обращение к разным полям структуры
    printf("1. Тест: подсчет зарплат (плохая локальность)\n");
    start = clock();
    double total_salary1 = 0;
    for (int i = 0; i < NUM_EMPLOYEES; i += 100) {  // Пропускаем записи
        total_salary1 += employees[i].salary;
    }
    end = clock();
    double sparse_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    // 2. Хороший паттерн: последовательный доступ
    printf("2. Тест: подсчет зарплат (хорошая локальность)\n");
    start = clock();
    double total_salary2 = 0;
    for (int i = 0; i < NUM_EMPLOYEES / 100; i++) {  // Последовательный доступ
        total_salary2 += employees[i].salary;
    }
    end = clock();
    double sequential_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    // 3. Колоночное хранение (SOA - Structure of Arrays)
    int* ids = malloc(NUM_EMPLOYEES * sizeof(int));
    double* salaries = malloc(NUM_EMPLOYEES * sizeof(double));
    
    for (int i = 0; i < NUM_EMPLOYEES; i++) {
        ids[i] = employees[i].id;
        salaries[i] = employees[i].salary;
    }
    
    printf("3. Тест: колоночное хранение\n");
    start = clock();
    double total_salary3 = 0;
    for (int i = 0; i < NUM_EMPLOYEES; i++) {
        total_salary3 += salaries[i];
    }
    end = clock();
    double columnar_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    printf("\nРезультаты:\n");
    printf("Разреженный доступ: %.6f сек\n", sparse_time);
    printf("Последовательный доступ: %.6f сек\n", sequential_time);
    printf("Колоночное хранение: %.6f сек\n", columnar_time);
    printf("Улучшение колоночного: %.2fx\n", sparse_time / columnar_time);
    
    free(employees);
    free(ids);
    free(salaries);
}
```

### В игровых движках

```c
// Data-Oriented Design для игрового движка
typedef struct {
    float x, y, z;     // Позиция
} Position;

typedef struct {
    float vx, vy, vz;  // Скорость
} Velocity;

typedef struct {
    float health;
    int level;
} GameStats;

void game_engine_optimization() {
    printf("\n=== Оптимизация игрового движка (DOD) ===\n");
    
    const int NUM_ENTITIES = 50000;
    
    // Массивы компонентов (Data-Oriented Design)
    Position* positions = malloc(NUM_ENTITIES * sizeof(Position));
    Velocity* velocities = malloc(NUM_ENTITIES * sizeof(Velocity));
    GameStats* stats = malloc(NUM_ENTITIES * sizeof(GameStats));
    
    // Инициализация
    for (int i = 0; i < NUM_ENTITIES; i++) {
        positions[i] = (Position){i * 0.1f, i * 0.2f, i * 0.3f};
        velocities[i] = (Velocity){1.0f, 0.5f, 0.2f};
        stats[i] = (GameStats){100.0f, i % 10 + 1};
    }
    
    clock_t start, end;
    
    // Система обновления позиций (только нужные данные)
    printf("Обновление позиций %d сущностей...\n", NUM_ENTITIES);
    start = clock();
    
    for (int frame = 0; frame < 100; frame++) {  // 100 кадров
        for (int i = 0; i < NUM_ENTITIES; i++) {
            positions[i].x += velocities[i].vx * 0.016f;  // 60 FPS
            positions[i].y += velocities[i].vy * 0.016f;
            positions[i].z += velocities[i].vz * 0.016f;
        }
    }
    
    end = clock();
    double update_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    printf("Время обновления: %.4f сек\n", update_time);
    printf("Обработано кадров: 100\n");
    printf("Сущностей на кадр: %d\n", NUM_ENTITIES);
    printf("Производительность: %.2f млн обновлений/сек\n", 
           (100.0 * NUM_ENTITIES) / update_time / 1000000.0);
    
    free(positions);
    free(velocities);
    free(stats);
}
```

---

## 📚 Рекомендации по оптимизации

### Общие принципы

1. **Максимизируйте локальность данных**
   - Группируйте связанные данные вместе
   - Используйте последовательный доступ к массивам
   - Избегайте указателей на удаленные данные

2. **Оптимизируйте структуры данных**
   - Выравнивание по границам кэш-линий
   - Минимизируйте размер "горячих" структур
   - Рассмотрите SOA vs AOS

3. **Используйте блочную обработку**
   - Разбивайте большие массивы на блоки
   - Размер блока = размер кэша L1/L2
   - Temporal blocking для повторного использования

4. **Минимизируйте TLB miss'ы**
   - Используйте huge pages для больших массивов
   - Группируйте данные по страницам
   - Избегайте случайного доступа к памяти

## 🔗 Связанные темы

- [[processor-architectures|Архитектуры процессоров]] - ISA и их влияние на память
- [[microarchitecture|Микроархитектура]] - конвейер и предсказание ветвлений
- [[performance-analysis|Анализ производительности]] - профилирование памяти
- [[parallel-architectures|Параллельные архитектуры]] - когерентность кэша

## 📚 Дополнительные ресурсы

### 📖 Статьи
- "What Every Programmer Should Know About Memory" - Ulrich Drepper
- "Memory Barriers: a Hardware View for Software Hackers" - Paul McKenney
- "Cache-Conscious Concurrent Data Structures" - Maurice Herlihy

### 🛠️ Инструменты
- **cachegrind** - анализ cache miss'ов
- **perf** - hardware performance counters
- **Intel VTune** - детальный анализ памяти
- **likwid-perfctr** - NUMA и cache анализ

---

> **Следующий шаг**: Изучите [[microarchitecture|микроархитектуру]] для понимания того, как кэш интегрируется с конвейером процессора и предсказанием ветвлений. 