# Управление памятью 🧠

> **Навигация**: [[README|← Системное программирование]] | [[processes-threads|Процессы и потоки →]]

## Содержание
1. [Основы управления памятью](#🏗️-основы-управления-памятью)
2. [Пользовательские аллокаторы](#🔧-пользовательские-аллокаторы)
3. [Пулы памяти](#🏊-пулы-памяти)
4. [Виртуальная память](#🌐-виртуальная-память)
5. [Garbage Collection](#♻️-garbage-collection)
6. [Отладка памяти](#🐛-отладка-памяти)

---

## 🏗️ Основы управления памятью

### Иерархия памяти в системе

```c
void memory_hierarchy_demo() {
    printf("=== Иерархия памяти ===\n\n");
    
    printf("Сегменты памяти процесса:\n");
    printf("┌─────────────────┐ ← Высокие адреса\n");
    printf("│     Stack       │ ← Локальные переменные\n");
    printf("│        ↓        │\n");
    printf("├─────────────────┤\n");
    printf("│                 │ ← Свободная память\n");
    printf("├─────────────────┤\n");
    printf("│        ↑        │\n");
    printf("│      Heap       │ ← malloc/new\n");
    printf("├─────────────────┤\n");
    printf("│      BSS        │ ← Неинициализированные\n");
    printf("├─────────────────┤\n");
    printf("│      Data       │ ← Инициализированные\n");
    printf("├─────────────────┤\n");
    printf("│      Text       │ ← Код программы\n");
    printf("└─────────────────┘ ← Низкие адреса\n\n");
    
    // Демонстрация размещения переменных
    int stack_var = 42;                    // Stack
    static int bss_var;                    // BSS
    static int data_var = 100;             // Data  
    int* heap_var = malloc(sizeof(int));   // Heap
    
    printf("Адреса переменных:\n");
    printf("Stack:  %p\n", (void*)&stack_var);
    printf("Heap:   %p\n", (void*)heap_var);
    printf("Data:   %p\n", (void*)&data_var);
    printf("BSS:    %p\n", (void*)&bss_var);
    printf("Text:   %p\n", (void*)memory_hierarchy_demo);
    
    free(heap_var);
}
```

---

## 🔧 Пользовательские аллокаторы

### Простой heap allocator

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <string.h>
#include <sys/mman.h>

// Структура блока памяти
typedef struct block {
    size_t size;        // Размер блока данных
    struct block* next; // Следующий блок в списке
    int is_free;        // Флаг: свободен ли блок
    uint32_t magic;     // Magic number для проверки целостности
} block_t;

#define BLOCK_MAGIC 0xDEADBEEF
#define MIN_BLOCK_SIZE 32

static block_t* heap_start = NULL;
static block_t* heap_end = NULL;
static size_t total_allocated = 0;

// Поиск подходящего свободного блока
block_t* find_free_block(size_t size) {
    block_t* current = heap_start;
    block_t* best_fit = NULL;
    
    while (current) {
        if (current->is_free && current->size >= size) {
            // Best fit стратегия
            if (!best_fit || current->size < best_fit->size) {
                best_fit = current;
                if (current->size == size) break; // Exact fit
            }
        }
        current = current->next;
    }
    
    return best_fit;
}

// Разделение блока на две части
void split_block(block_t* block, size_t size) {
    if (block->size > size + sizeof(block_t) + MIN_BLOCK_SIZE) {
        block_t* new_block = (block_t*)((char*)block + sizeof(block_t) + size);
        
        new_block->size = block->size - size - sizeof(block_t);
        new_block->next = block->next;
        new_block->is_free = 1;
        new_block->magic = BLOCK_MAGIC;
        
        block->size = size;
        block->next = new_block;
    }
}

// Запрос памяти у системы
block_t* request_memory(size_t size) {
    // Выравниваем размер до страницы
    size_t page_size = getpagesize();
    size_t total_size = sizeof(block_t) + size;
    size_t aligned_size = ((total_size + page_size - 1) / page_size) * page_size;
    
    void* ptr = mmap(NULL, aligned_size, 
                     PROT_READ | PROT_WRITE, 
                     MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    
    if (ptr == MAP_FAILED) return NULL;
    
    block_t* block = (block_t*)ptr;
    block->size = aligned_size - sizeof(block_t);
    block->next = NULL;
    block->is_free = 0;
    block->magic = BLOCK_MAGIC;
    
    return block;
}

// Пользовательский malloc
void* my_malloc(size_t size) {
    if (size <= 0) return NULL;
    
    // Выравниваем размер
    size = (size + 7) & ~7;  // Выравнивание на 8 байт
    
    block_t* block = find_free_block(size);
    
    if (block) {
        block->is_free = 0;
        split_block(block, size);
        return (void*)((char*)block + sizeof(block_t));
    }
    
    // Запрашиваем новую память
    block = request_memory(size);
    if (!block) return NULL;
    
    // Добавляем в список
    if (!heap_start) {
        heap_start = block;
        heap_end = block;
    } else {
        heap_end->next = block;
        heap_end = block;
    }
    
    total_allocated += block->size;
    return (void*)((char*)block + sizeof(block_t));
}

// Проверка валидности блока
int validate_block(block_t* block) {
    return block && block->magic == BLOCK_MAGIC;
}

// Слияние соседних свободных блоков
void coalesce_free_blocks() {
    block_t* current = heap_start;
    
    while (current && current->next) {
        if (current->is_free && current->next->is_free) {
            // Проверяем, что блоки соседние
            char* end_of_current = (char*)current + sizeof(block_t) + current->size;
            if (end_of_current == (char*)current->next) {
                current->size += current->next->size + sizeof(block_t);
                current->next = current->next->next;
                continue;
            }
        }
        current = current->next;
    }
}

// Пользовательский free
void my_free(void* ptr) {
    if (!ptr) return;
    
    block_t* block = (block_t*)((char*)ptr - sizeof(block_t));
    
    if (!validate_block(block)) {
        printf("ERROR: Invalid block or double free detected!\n");
        return;
    }
    
    block->is_free = 1;
    coalesce_free_blocks();
}

// Статистика heap'а
void heap_stats() {
    printf("\n=== Heap Statistics ===\n");
    
    size_t total_size = 0;
    size_t used_size = 0;
    size_t free_size = 0;
    int total_blocks = 0;
    int free_blocks = 0;
    
    block_t* current = heap_start;
    while (current) {
        total_size += current->size + sizeof(block_t);
        total_blocks++;
        
        if (current->is_free) {
            free_size += current->size;
            free_blocks++;
        } else {
            used_size += current->size;
        }
        
        current = current->next;
    }
    
    printf("Total heap size: %zu bytes\n", total_size);
    printf("Used memory:     %zu bytes (%.1f%%)\n", 
           used_size, (double)used_size / total_size * 100);
    printf("Free memory:     %zu bytes (%.1f%%)\n", 
           free_size, (double)free_size / total_size * 100);
    printf("Total blocks:    %d\n", total_blocks);
    printf("Free blocks:     %d\n", free_blocks);
    printf("Fragmentation:   %.1f%%\n", 
           (double)free_blocks / total_blocks * 100);
}
```

### Buddy allocator

```c
// Buddy system allocator
#define MAX_ORDER 10
#define MIN_BLOCK_SIZE (1 << 4)  // 16 bytes

typedef struct buddy_block {
    int order;
    int is_free;
    struct buddy_block* next;
} buddy_block_t;

static buddy_block_t* free_lists[MAX_ORDER + 1];
static void* buddy_memory = NULL;
static size_t buddy_size = 0;

// Инициализация buddy allocator
int buddy_init(size_t size) {
    // Округляем размер до степени двойки
    int order = 0;
    size_t aligned_size = MIN_BLOCK_SIZE;
    while (aligned_size < size && order < MAX_ORDER) {
        aligned_size <<= 1;
        order++;
    }
    
    buddy_memory = mmap(NULL, aligned_size,
                       PROT_READ | PROT_WRITE,
                       MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    
    if (buddy_memory == MAP_FAILED) return -1;
    
    buddy_size = aligned_size;
    
    // Инициализируем free list для максимального блока
    buddy_block_t* block = (buddy_block_t*)buddy_memory;
    block->order = order;
    block->is_free = 1;
    block->next = NULL;
    
    free_lists[order] = block;
    
    return 0;
}

// Получение адреса buddy блока
void* get_buddy_address(void* block, int order) {
    size_t offset = (char*)block - (char*)buddy_memory;
    size_t buddy_offset = offset ^ (1 << (order + 4));
    return (char*)buddy_memory + buddy_offset;
}

// Разделение блока на два buddy блока
void split_buddy_block(int order) {
    if (order > MAX_ORDER || !free_lists[order]) return;
    
    buddy_block_t* block = free_lists[order];
    free_lists[order] = block->next;
    
    // Создаем два блока меньшего размера
    buddy_block_t* left = block;
    buddy_block_t* right = (buddy_block_t*)((char*)block + (1 << (order - 1 + 4)));
    
    left->order = order - 1;
    left->is_free = 1;
    left->next = free_lists[order - 1];
    
    right->order = order - 1;
    right->is_free = 1;
    right->next = left;
    
    free_lists[order - 1] = right;
}

// Buddy malloc
void* buddy_malloc(size_t size) {
    if (!buddy_memory) return NULL;
    
    // Находим нужный order
    int order = 0;
    size_t block_size = MIN_BLOCK_SIZE;
    while (block_size < size + sizeof(buddy_block_t) && order < MAX_ORDER) {
        block_size <<= 1;
        order++;
    }
    
    // Ищем свободный блок нужного или большего размера
    int current_order = order;
    while (current_order <= MAX_ORDER && !free_lists[current_order]) {
        current_order++;
    }
    
    if (current_order > MAX_ORDER) return NULL;
    
    // Разделяем блоки до нужного размера
    while (current_order > order) {
        split_buddy_block(current_order);
        current_order--;
    }
    
    // Выделяем блок
    buddy_block_t* block = free_lists[order];
    free_lists[order] = block->next;
    block->is_free = 0;
    
    return (void*)((char*)block + sizeof(buddy_block_t));
}

// Buddy free с объединением
void buddy_free(void* ptr) {
    if (!ptr || !buddy_memory) return;
    
    buddy_block_t* block = (buddy_block_t*)((char*)ptr - sizeof(buddy_block_t));
    block->is_free = 1;
    
    // Пытаемся объединить с buddy блоком
    int order = block->order;
    while (order < MAX_ORDER) {
        buddy_block_t* buddy = (buddy_block_t*)get_buddy_address(block, order);
        
        // Проверяем, можно ли объединить
        if ((char*)buddy >= (char*)buddy_memory + buddy_size ||
            buddy->order != order || !buddy->is_free) {
            break;
        }
        
        // Удаляем buddy из free list
        buddy_block_t** current = &free_lists[order];
        while (*current && *current != buddy) {
            current = &(*current)->next;
        }
        if (*current) *current = buddy->next;
        
        // Объединяем блоки
        if (buddy < block) block = buddy;
        block->order = order + 1;
        order++;
    }
    
    // Добавляем в free list
    block->next = free_lists[order];
    free_lists[order] = block;
}
```

---

## 🏊 Пулы памяти

### Object pool для фиксированного размера

```c
// Пул памяти для объектов фиксированного размера
typedef struct memory_pool {
    void* pool_start;       // Начало пула
    void* free_list;        // Список свободных блоков
    size_t block_size;      // Размер одного блока
    size_t total_blocks;    // Общее количество блоков
    size_t free_blocks;     // Количество свободных блоков
    char* bitmap;           // Битовая карта занятости
} memory_pool_t;

memory_pool_t* create_memory_pool(size_t block_size, size_t num_blocks) {
    memory_pool_t* pool = malloc(sizeof(memory_pool_t));
    if (!pool) return NULL;
    
    // Выравниваем размер блока
    block_size = (block_size + 7) & ~7;
    
    pool->block_size = block_size;
    pool->total_blocks = num_blocks;
    pool->free_blocks = num_blocks;
    
    // Выделяем память для пула
    pool->pool_start = aligned_alloc(64, block_size * num_blocks);
    if (!pool->pool_start) {
        free(pool);
        return NULL;
    }
    
    // Создаем битовую карту
    size_t bitmap_size = (num_blocks + 7) / 8;
    pool->bitmap = calloc(bitmap_size, 1);
    if (!pool->bitmap) {
        free(pool->pool_start);
        free(pool);
        return NULL;
    }
    
    // Инициализация free list
    pool->free_list = pool->pool_start;
    char* current = (char*)pool->pool_start;
    
    for (size_t i = 0; i < num_blocks - 1; i++) {
        *(void**)current = current + block_size;
        current += block_size;
    }
    *(void**)current = NULL;  // Последний блок
    
    return pool;
}

// Быстрое выделение из пула
void* pool_alloc(memory_pool_t* pool) {
    if (!pool || !pool->free_list) return NULL;
    
    void* block = pool->free_list;
    pool->free_list = *(void**)pool->free_list;
    pool->free_blocks--;
    
    // Обновляем битовую карту
    size_t index = ((char*)block - (char*)pool->pool_start) / pool->block_size;
    pool->bitmap[index / 8] |= (1 << (index % 8));
    
    return block;
}

// Быстрое освобождение
void pool_free(memory_pool_t* pool, void* ptr) {
    if (!pool || !ptr) return;
    
    // Проверяем, что указатель принадлежит пулу
    if (ptr < pool->pool_start ||
        ptr >= (char*)pool->pool_start + pool->total_blocks * pool->block_size) {
        printf("ERROR: Pointer doesn't belong to pool!\n");
        return;
    }
    
    // Проверяем выравнивание
    size_t offset = (char*)ptr - (char*)pool->pool_start;
    if (offset % pool->block_size != 0) {
        printf("ERROR: Invalid pointer alignment!\n");
        return;
    }
    
    size_t index = offset / pool->block_size;
    
    // Проверяем double free
    if (!(pool->bitmap[index / 8] & (1 << (index % 8)))) {
        printf("ERROR: Double free detected!\n");
        return;
    }
    
    // Обновляем битовую карту
    pool->bitmap[index / 8] &= ~(1 << (index % 8));
    
    // Добавляем в free list
    *(void**)ptr = pool->free_list;
    pool->free_list = ptr;
    pool->free_blocks++;
}

// Статистика пула
void pool_stats(memory_pool_t* pool) {
    if (!pool) return;
    
    printf("\n=== Pool Statistics ===\n");
    printf("Block size:      %zu bytes\n", pool->block_size);
    printf("Total blocks:    %zu\n", pool->total_blocks);
    printf("Free blocks:     %zu\n", pool->free_blocks);
    printf("Used blocks:     %zu\n", pool->total_blocks - pool->free_blocks);
    printf("Utilization:     %.1f%%\n", 
           (double)(pool->total_blocks - pool->free_blocks) / pool->total_blocks * 100);
    printf("Memory used:     %zu KB\n", 
           (pool->total_blocks - pool->free_blocks) * pool->block_size / 1024);
}

void destroy_memory_pool(memory_pool_t* pool) {
    if (pool) {
        free(pool->bitmap);
        free(pool->pool_start);
        free(pool);
    }
}
```

### Stack allocator для временных объектов

```c
// Stack allocator для быстрого выделения временных объектов
typedef struct stack_allocator {
    void* memory;
    size_t size;
    size_t offset;
    size_t* markers;
    int marker_count;
    int marker_capacity;
} stack_allocator_t;

stack_allocator_t* create_stack_allocator(size_t size) {
    stack_allocator_t* allocator = malloc(sizeof(stack_allocator_t));
    if (!allocator) return NULL;
    
    allocator->memory = malloc(size);
    if (!allocator->memory) {
        free(allocator);
        return NULL;
    }
    
    allocator->size = size;
    allocator->offset = 0;
    allocator->marker_count = 0;
    allocator->marker_capacity = 32;
    allocator->markers = malloc(sizeof(size_t) * allocator->marker_capacity);
    
    return allocator;
}

// Очень быстрое выделение (просто сдвиг указателя)
void* stack_alloc(stack_allocator_t* allocator, size_t size) {
    if (!allocator) return NULL;
    
    // Выравниваем размер
    size = (size + 7) & ~7;
    
    if (allocator->offset + size > allocator->size) {
        return NULL;  // Не хватает места
    }
    
    void* ptr = (char*)allocator->memory + allocator->offset;
    allocator->offset += size;
    
    return ptr;
}

// Установка маркера для массового освобождения
void stack_push_marker(stack_allocator_t* allocator) {
    if (!allocator) return;
    
    if (allocator->marker_count >= allocator->marker_capacity) {
        allocator->marker_capacity *= 2;
        allocator->markers = realloc(allocator->markers, 
                                   sizeof(size_t) * allocator->marker_capacity);
    }
    
    allocator->markers[allocator->marker_count++] = allocator->offset;
}

// Освобождение до маркера (массовое освобождение)
void stack_pop_marker(stack_allocator_t* allocator) {
    if (!allocator || allocator->marker_count == 0) return;
    
    allocator->offset = allocator->markers[--allocator->marker_count];
}

// Полная очистка
void stack_clear(stack_allocator_t* allocator) {
    if (allocator) {
        allocator->offset = 0;
        allocator->marker_count = 0;
    }
}
```

---

## 🌐 Виртуальная память

### Работа с virtual memory

```c
#include <sys/mman.h>

void virtual_memory_demo() {
    printf("=== Виртуальная память ===\n\n");
    
    size_t page_size = getpagesize();
    printf("Размер страницы: %zu bytes\n\n", page_size);
    
    // 1. Выделение большого блока виртуальной памяти
    size_t virtual_size = 1024 * 1024 * 1024;  // 1GB
    void* virtual_mem = mmap(NULL, virtual_size,
                            PROT_NONE,  // Нет доступа
                            MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    
    if (virtual_mem == MAP_FAILED) {
        perror("mmap failed");
        return;
    }
    
    printf("Выделено виртуальной памяти: 1GB по адресу %p\n", virtual_mem);
    printf("Физическая память пока не выделена!\n\n");
    
    // 2. Постепенное выделение физических страниц
    for (int i = 0; i < 10; i++) {
        void* page_addr = (char*)virtual_mem + i * page_size;
        
        // Даем доступ к странице
        if (mprotect(page_addr, page_size, PROT_READ | PROT_WRITE) != 0) {
            perror("mprotect failed");
            break;
        }
        
        // Первое обращение вызывает page fault и выделение физической страницы
        *(int*)page_addr = i;
        
        printf("Страница %d: виртуальный адрес %p, значение %d\n", 
               i, page_addr, *(int*)page_addr);
    }
    
    // 3. Copy-on-Write демонстрация
    pid_t pid = fork();
    if (pid == 0) {
        // Дочерний процесс - модифицируем память
        printf("\n[Child] Модифицирую shared память...\n");
        *(int*)virtual_mem = 999;
        printf("[Child] Новое значение: %d\n", *(int*)virtual_mem);
        exit(0);
    } else {
        wait(NULL);
        printf("[Parent] Значение после fork: %d\n", *(int*)virtual_mem);
        printf("Copy-on-Write сработал - родитель видит старое значение\n");
    }
    
    munmap(virtual_mem, virtual_size);
}

// Демонстрация memory mapping файлов
void file_mapping_demo() {
    printf("\n=== Memory Mapping файлов ===\n\n");
    
    // Создаем файл
    const char* filename = "/tmp/mmap_demo.txt";
    FILE* file = fopen(filename, "w");
    
    const char* content = "Hello, Memory Mapped Files!\n";
    fwrite(content, 1, strlen(content), file);
    fclose(file);
    
    // Открываем файл для mapping
    int fd = open(filename, O_RDWR);
    if (fd == -1) {
        perror("open failed");
        return;
    }
    
    struct stat sb;
    fstat(fd, &sb);
    size_t file_size = sb.st_size;
    
    // Mapping файла в память
    char* mapped = mmap(NULL, file_size,
                       PROT_READ | PROT_WRITE,
                       MAP_SHARED, fd, 0);
    
    if (mapped == MAP_FAILED) {
        perror("mmap failed");
        close(fd);
        return;
    }
    
    printf("Файл замапен в память по адресу %p\n", mapped);
    printf("Содержимое: %.*s", (int)file_size, mapped);
    
    // Модификация через память автоматически записывается в файл
    mapped[0] = 'h';  // Hello -> hello
    
    // Синхронизация с диском
    msync(mapped, file_size, MS_SYNC);
    
    printf("После модификации: %.*s", (int)file_size, mapped);
    
    munmap(mapped, file_size);
    close(fd);
    unlink(filename);
}
```

---

## ♻️ Garbage Collection

### Простой mark-and-sweep GC

```c
// Простая реализация mark-and-sweep garbage collector
typedef struct gc_object {
    size_t size;
    int marked;
    struct gc_object* next;
    char data[];
} gc_object_t;

typedef struct gc_heap {
    gc_object_t* objects;
    size_t total_size;
    size_t used_size;
    void** roots;
    int root_count;
    int root_capacity;
} gc_heap_t;

static gc_heap_t* gc = NULL;

// Инициализация GC
void gc_init() {
    gc = malloc(sizeof(gc_heap_t));
    gc->objects = NULL;
    gc->total_size = 0;
    gc->used_size = 0;
    gc->roots = malloc(sizeof(void*) * 16);
    gc->root_count = 0;
    gc->root_capacity = 16;
}

// Выделение объекта
void* gc_alloc(size_t size) {
    if (!gc) gc_init();
    
    gc_object_t* obj = malloc(sizeof(gc_object_t) + size);
    if (!obj) return NULL;
    
    obj->size = size;
    obj->marked = 0;
    obj->next = gc->objects;
    gc->objects = obj;
    
    gc->used_size += size;
    gc->total_size += sizeof(gc_object_t) + size;
    
    return obj->data;
}

// Добавление root объекта
void gc_add_root(void* ptr) {
    if (!gc || !ptr) return;
    
    if (gc->root_count >= gc->root_capacity) {
        gc->root_capacity *= 2;
        gc->roots = realloc(gc->roots, sizeof(void*) * gc->root_capacity);
    }
    
    gc->roots[gc->root_count++] = ptr;
}

// Mark фаза - пометка достижимых объектов
void gc_mark_object(void* ptr) {
    if (!ptr) return;
    
    // Находим объект по указателю на данные
    gc_object_t* obj = gc->objects;
    while (obj) {
        if (obj->data == ptr) {
            if (!obj->marked) {
                obj->marked = 1;
                
                // Рекурсивно помечаем ссылки в объекте
                // (это упрощенная версия, в реальности нужен тип-специфичный код)
                void** ptrs = (void**)obj->data;
                for (size_t i = 0; i < obj->size / sizeof(void*); i++) {
                    gc_mark_object(ptrs[i]);
                }
            }
            break;
        }
        obj = obj->next;
    }
}

// Sweep фаза - освобождение непомеченных объектов
void gc_sweep() {
    gc_object_t** current = &gc->objects;
    size_t freed_objects = 0;
    size_t freed_memory = 0;
    
    while (*current) {
        gc_object_t* obj = *current;
        
        if (obj->marked) {
            obj->marked = 0;  // Сбрасываем метку для следующего цикла
            current = &obj->next;
        } else {
            // Удаляем непомеченный объект
            *current = obj->next;
            freed_memory += obj->size;
            freed_objects++;
            free(obj);
        }
    }
    
    gc->used_size -= freed_memory;
    printf("GC: освобождено %zu объектов (%zu bytes)\n", 
           freed_objects, freed_memory);
}

// Запуск сборки мусора
void gc_collect() {
    if (!gc) return;
    
    printf("Запуск GC: используется %zu KB\n", gc->used_size / 1024);
    
    // Mark фаза - помечаем все достижимые объекты
    for (int i = 0; i < gc->root_count; i++) {
        gc_mark_object(gc->roots[i]);
    }
    
    // Sweep фаза - освобождаем непомеченные
    gc_sweep();
    
    printf("После GC: используется %zu KB\n", gc->used_size / 1024);
}
```

---

## 🐛 Отладка памяти

### Обнаружение утечек памяти

```c
#ifdef DEBUG_MEMORY

// Структура для отслеживания выделений
typedef struct alloc_info {
    void* ptr;
    size_t size;
    const char* file;
    int line;
    struct alloc_info* next;
} alloc_info_t;

static alloc_info_t* allocations = NULL;
static size_t total_allocated = 0;
static int allocation_count = 0;

// Отслеживание malloc
void* debug_malloc(size_t size, const char* file, int line) {
    void* ptr = malloc(size);
    if (!ptr) return NULL;
    
    alloc_info_t* info = malloc(sizeof(alloc_info_t));
    info->ptr = ptr;
    info->size = size;
    info->file = file;
    info->line = line;
    info->next = allocations;
    allocations = info;
    
    total_allocated += size;
    allocation_count++;
    
    return ptr;
}

// Отслеживание free
void debug_free(void* ptr, const char* file, int line) {
    if (!ptr) return;
    
    alloc_info_t** current = &allocations;
    while (*current) {
        if ((*current)->ptr == ptr) {
            alloc_info_t* info = *current;
            *current = info->next;
            
            total_allocated -= info->size;
            allocation_count--;
            
            free(info);
            free(ptr);
            return;
        }
        current = &(*current)->next;
    }
    
    printf("ERROR: free() вызван для неизвестного указателя %p в %s:%d\n",
           ptr, file, line);
}

// Проверка утечек
void check_memory_leaks() {
    if (allocation_count == 0) {
        printf("✅ Утечек памяти не обнаружено\n");
        return;
    }
    
    printf("❌ Обнаружено %d утечек памяти (%zu bytes):\n", 
           allocation_count, total_allocated);
    
    alloc_info_t* current = allocations;
    while (current) {
        printf("  %p: %zu bytes в %s:%d\n", 
               current->ptr, current->size, current->file, current->line);
        current = current->next;
    }
}

#define malloc(size) debug_malloc(size, __FILE__, __LINE__)
#define free(ptr) debug_free(ptr, __FILE__, __LINE__)

#endif // DEBUG_MEMORY
```

### AddressSanitizer интеграция

```c
// Использование AddressSanitizer для отладки памяти
void memory_debugging_demo() {
    printf("=== Отладка памяти ===\n\n");
    
    printf("Инструменты для отладки памяти:\n\n");
    
    printf("1. AddressSanitizer (ASan):\n");
    printf("   Компиляция: gcc -fsanitize=address -g program.c\n");
    printf("   Обнаруживает:\n");
    printf("   • Buffer overflows\n");
    printf("   • Use after free\n");
    printf("   • Memory leaks\n");
    printf("   • Double free\n\n");
    
    printf("2. Valgrind:\n");
    printf("   Запуск: valgrind --tool=memcheck --leak-check=full ./program\n");
    printf("   Обнаруживает:\n");
    printf("   • Утечки памяти\n");
    printf("   • Неинициализированные переменные\n");
    printf("   • Buffer overruns\n\n");
    
    printf("3. Static Analysis:\n");
    printf("   • Clang Static Analyzer\n");
    printf("   • Cppcheck\n");
    printf("   • PVS-Studio\n\n");
    
    // Пример использования memory sanitizer
    #ifdef __has_feature
    #if __has_feature(address_sanitizer)
        printf("✅ AddressSanitizer активирован\n");
    #endif
    #endif
}
```

---

## 🔗 Связанные темы

- [[processes-threads|Процессы и потоки]] - процессы как контейнеры памяти
- [[file-systems|Файловые системы]] - memory mapping файлов
- [[../computer-architecture/memory-hierarchy|Иерархия памяти]] - аппаратная основа

## 📚 Дополнительные ресурсы

### 📖 Книги
- "Understanding the Linux Kernel" - Bovet & Cesati
- "Computer Systems: A Programmer's Perspective" - Bryant & O'Hallaron
- "The Garbage Collection Handbook" - Jones, Hosking & Moss

### 🛠️ Инструменты
- **AddressSanitizer** - обнаружение ошибок памяти
- **Valgrind** - анализ памяти и производительности
- **jemalloc** - производительный allocator
- **tcmalloc** - Google's thread-caching malloc

### 🔬 Эксперименты
- Сравнение производительности allocators
- Анализ фрагментации памяти
- Benchmark различных GC алгоритмов

---

> **Следующий шаг**: Изучите [[processes-threads|процессы и потоки]] для понимания многозадачности и синхронизации. 