# Параллельные архитектуры 🔄

> **Навигация**: [[README|← Архитектура компьютеров]] | [[microarchitecture|← Микроархитектура]] | [[gpu-architectures|GPU архитектуры →]]

## Содержание
1. [Многоядерные процессоры](#🔧-многоядерные-процессоры)
2. [NUMA архитектура](#🌐-numa-архитектура)
3. [SIMD инструкции](#⚡-simd-инструкции)
4. [Масштабируемость системы](#📈-масштабируемость-системы)
5. [Синхронизация и когерентность](#🔒-синхронизация-и-когерентность)

---

## 🔧 Многоядерные процессоры

### Демонстрация многопоточности

```c
#include <pthread.h>
#include <sched.h>

typedef struct {
    int thread_id;
    int cpu_core;
    int* data;
    int start_index;
    int end_index;
    long result;
} ThreadData;

void* cpu_intensive_task(void* arg) {
    ThreadData* td = (ThreadData*)arg;
    
    // Привязываем поток к конкретному ядру
    cpu_set_t cpuset;
    CPU_ZERO(&cpuset);
    CPU_SET(td->cpu_core, &cpuset);
    pthread_setaffinity_np(pthread_self(), sizeof(cpu_set_t), &cpuset);
    
    printf("Thread %d привязан к ядру %d\n", td->thread_id, td->cpu_core);
    
    // CPU-интенсивная задача
    td->result = 0;
    for (int i = td->start_index; i < td->end_index; i++) {
        td->result += td->data[i] * td->data[i];  // Квадраты чисел
    }
    
    return NULL;
}

void multicore_demo() {
    printf("\n=== Многоядерная обработка ===\n");
    
    const int DATA_SIZE = 10000000;
    const int NUM_THREADS = 4;
    
    // Создаем данные
    int* data = malloc(DATA_SIZE * sizeof(int));
    for (int i = 0; i < DATA_SIZE; i++) {
        data[i] = i % 1000;
    }
    
    // Проверяем количество ядер
    int num_cores = sysconf(_SC_NPROCESSORS_ONLN);
    printf("Доступно ядер: %d\n", num_cores);
    
    clock_t start, end;
    
    // 1. Однопоточная версия
    start = clock();
    long sequential_result = 0;
    for (int i = 0; i < DATA_SIZE; i++) {
        sequential_result += data[i] * data[i];
    }
    end = clock();
    double sequential_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    // 2. Многопоточная версия
    pthread_t threads[NUM_THREADS];
    ThreadData thread_data[NUM_THREADS];
    int chunk_size = DATA_SIZE / NUM_THREADS;
    
    start = clock();
    
    // Создаем потоки
    for (int i = 0; i < NUM_THREADS; i++) {
        thread_data[i].thread_id = i;
        thread_data[i].cpu_core = i % num_cores;
        thread_data[i].data = data;
        thread_data[i].start_index = i * chunk_size;
        thread_data[i].end_index = (i == NUM_THREADS - 1) ? 
                                  DATA_SIZE : (i + 1) * chunk_size;
        
        pthread_create(&threads[i], NULL, cpu_intensive_task, &thread_data[i]);
    }
    
    // Ждем завершения и собираем результаты
    long parallel_result = 0;
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
        parallel_result += thread_data[i].result;
    }
    
    end = clock();
    double parallel_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    printf("\nРезультаты:\n");
    printf("Последовательно: %.4f сек (результат: %ld)\n", 
           sequential_time, sequential_result);
    printf("Параллельно (%d потоков): %.4f сек (результат: %ld)\n", 
           NUM_THREADS, parallel_time, parallel_result);
    printf("Ускорение: %.2fx\n", sequential_time / parallel_time);
    printf("Эффективность: %.1f%%\n", 
           (sequential_time / parallel_time) / NUM_THREADS * 100);
    
    free(data);
}
```

### Закон Амдала и масштабирование

```c
void amdahl_law_demo() {
    printf("\n=== Закон Амдала ===\n");
    
    // Различные пропорции параллельного кода
    double parallel_fractions[] = {0.5, 0.75, 0.9, 0.95, 0.99};
    int num_fractions = sizeof(parallel_fractions) / sizeof(parallel_fractions[0]);
    
    printf("Максимальное ускорение по закону Амдала:\n");
    printf("Параллельная часть | 2 ядра | 4 ядра | 8 ядер | 16 ядер | ∞ ядер\n");
    printf("─────────────────────────────────────────────────────────────────\n");
    
    for (int i = 0; i < num_fractions; i++) {
        double p = parallel_fractions[i];
        double sequential = 1.0 - p;
        
        printf("    %.0f%%          ", p * 100);
        
        int cores[] = {2, 4, 8, 16};
        for (int j = 0; j < 4; j++) {
            double speedup = 1.0 / (sequential + p / cores[j]);
            printf(" %.2fx ", speedup);
        }
        
        // Теоретический максимум (бесконечное количество ядер)
        double max_speedup = 1.0 / sequential;
        printf(" %.2fx\n", max_speedup);
    }
    
    printf("\nВывод: Даже 5%% последовательного кода ограничивает ускорение до 20x!\n");
}
```

---

## 🌐 NUMA архитектура

### Основы NUMA

```c
#ifdef __linux__
#include <numa.h>
#include <numaif.h>

void numa_demo() {
    printf("\n=== NUMA архитектура ===\n");
    
    if (numa_available() < 0) {
        printf("NUMA не поддерживается\n");
        return;
    }
    
    int num_nodes = numa_max_node() + 1;
    printf("NUMA узлов: %d\n", num_nodes);
    
    // Показываем топологию NUMA
    for (int i = 0; i <= numa_max_node(); i++) {
        if (numa_node_size(i, NULL) > 0) {
            printf("Узел %d: %ld MB памяти\n", i, numa_node_size(i, NULL) >> 20);
        }
    }
    
    const int SIZE = 1000000;
    
    // Тест 1: Локальная память
    clock_t start = clock();
    
    // Привязываемся к узлу 0
    numa_run_on_node(0);
    int* local_data = numa_alloc_onnode(SIZE * sizeof(int), 0);
    
    // Инициализация и обработка на том же узле
    for (int i = 0; i < SIZE; i++) {
        local_data[i] = i;
    }
    
    long local_sum = 0;
    for (int i = 0; i < SIZE; i++) {
        local_sum += local_data[i];
    }
    
    clock_t end = clock();
    double local_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    // Тест 2: Удаленная память
    if (num_nodes > 1) {
        start = clock();
        
        // Выделяем память на узле 1, но работаем с узла 0
        int* remote_data = numa_alloc_onnode(SIZE * sizeof(int), 1);
        numa_run_on_node(0);  // Работаем с узла 0
        
        for (int i = 0; i < SIZE; i++) {
            remote_data[i] = i;
        }
        
        long remote_sum = 0;
        for (int i = 0; i < SIZE; i++) {
            remote_sum += remote_data[i];
        }
        
        end = clock();
        double remote_time = ((double)(end - start)) / CLOCKS_PER_SEC;
        
        printf("\nРезультаты NUMA:\n");
        printf("Локальная память: %.4f сек (sum = %ld)\n", local_time, local_sum);
        printf("Удаленная память: %.4f сек (sum = %ld)\n", remote_time, remote_sum);
        printf("Штраф за удаленный доступ: %.2fx\n", remote_time / local_time);
        
        numa_free(remote_data, SIZE * sizeof(int));
    }
    
    numa_free(local_data, SIZE * sizeof(int));
}
#else
void numa_demo() {
    printf("\n=== NUMA демонстрация недоступна на данной платформе ===\n");
}
#endif
```

### NUMA-aware программирование

```c
void numa_optimization_tips() {
    printf("\n=== Оптимизация для NUMA ===\n");
    
    printf("🎯 Принципы NUMA-aware программирования:\n\n");
    
    printf("1. Локальность данных:\n");
    printf("   ✅ Размещайте данные рядом с вычислениями\n");
    printf("   ✅ Используйте numa_alloc_onnode()\n");
    printf("   ❌ Не используйте общие массивы для всех потоков\n\n");
    
    printf("2. Привязка потоков:\n");
    printf("   ✅ pthread_setaffinity_np() для привязки к ядрам\n");
    printf("   ✅ Группируйте related потоки на одном узле\n");
    printf("   ❌ Не позволяйте потокам мигрировать между узлами\n\n");
    
    printf("3. Паттерны доступа:\n");
    printf("   ✅ First-touch policy - первый доступ определяет размещение\n");
    printf("   ✅ Инициализируйте данные тем же потоком, что будет их использовать\n");
    printf("   ❌ Избегайте false sharing между узлами\n\n");
    
    printf("Пример NUMA-оптимизированного кода:\n");
    printf("```c\n");
    printf("// Плохо: один поток инициализирует, другие используют\n");
    printf("for (int i = 0; i < SIZE; i++) data[i] = 0;  // main thread\n");
    printf("#pragma omp parallel for\n");
    printf("for (int i = 0; i < SIZE; i++) data[i] += compute(i);  // worker threads\n\n");
    
    printf("// Хорошо: каждый поток инициализирует свою часть\n");
    printf("#pragma omp parallel for\n");
    printf("for (int i = 0; i < SIZE; i++) {\n");
    printf("    data[i] = 0;  // first touch\n");
    printf("    data[i] += compute(i);  // same thread\n");
    printf("}\n");
    printf("```\n");
}
```

---

## ⚡ SIMD инструкции

### AVX демонстрация

```c
#ifdef __AVX2__
#include <immintrin.h>

void avx_vectorization_demo() {
    printf("\n=== AVX2 векторизация ===\n");
    
    const int SIZE = 8192;  // Кратно 8 для AVX2
    
    // Выравненные массивы для AVX
    float* a = _mm_malloc(SIZE * sizeof(float), 32);
    float* b = _mm_malloc(SIZE * sizeof(float), 32);
    float* result_scalar = _mm_malloc(SIZE * sizeof(float), 32);
    float* result_vector = _mm_malloc(SIZE * sizeof(float), 32);
    
    // Инициализация
    for (int i = 0; i < SIZE; i++) {
        a[i] = i * 0.1f;
        b[i] = i * 0.2f;
    }
    
    clock_t start, end;
    
    // 1. Скалярная версия
    start = clock();
    for (int iter = 0; iter < 10000; iter++) {
        for (int i = 0; i < SIZE; i++) {
            result_scalar[i] = a[i] * b[i] + a[i];  // FMA операция
        }
    }
    end = clock();
    double scalar_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    // 2. AVX2 векторная версия
    start = clock();
    for (int iter = 0; iter < 10000; iter++) {
        for (int i = 0; i < SIZE; i += 8) {  // AVX2 обрабатывает 8 float за раз
            __m256 va = _mm256_load_ps(&a[i]);
            __m256 vb = _mm256_load_ps(&b[i]);
            
            // FMA: a * b + a
            __m256 vresult = _mm256_fmadd_ps(va, vb, va);
            
            _mm256_store_ps(&result_vector[i], vresult);
        }
    }
    end = clock();
    double vector_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    // Проверяем корректность
    int correct = 1;
    for (int i = 0; i < SIZE; i++) {
        if (fabsf(result_scalar[i] - result_vector[i]) > 1e-6) {
            correct = 0;
            break;
        }
    }
    
    printf("SIMD результаты:\n");
    printf("Скалярная версия: %.4f сек\n", scalar_time);
    printf("AVX2 версия: %.4f сек\n", vector_time);
    printf("Ускорение: %.2fx\n", scalar_time / vector_time);
    printf("Теоретический максимум: 8x (8 float в AVX2)\n");
    printf("Эффективность: %.1f%%\n", (scalar_time / vector_time) / 8 * 100);
    printf("Корректность: %s\n", correct ? "✅" : "❌");
    
    _mm_free(a); _mm_free(b); _mm_free(result_scalar); _mm_free(result_vector);
}
#else
void avx_vectorization_demo() {
    printf("\n=== AVX2 недоступно на данной платформе ===\n");
}
#endif
```

### Автовекторизация компилятора

```c
void compiler_vectorization_demo() {
    printf("\n=== Автовекторизация компилятора ===\n");
    
    printf("Советы по автовекторизации:\n\n");
    
    printf("1. Простые циклы:\n");
    printf("   ✅ for (int i = 0; i < n; i++) c[i] = a[i] + b[i];\n");
    printf("   ❌ for (int i = 0; i < n; i++) c[i] = a[random[i]] + b[i];\n\n");
    
    printf("2. Выравнивание данных:\n");
    printf("   ✅ __attribute__((aligned(32))) float data[SIZE];\n");
    printf("   ✅ Используйте размеры кратные векторным регистрам\n\n");
    
    printf("3. Зависимости данных:\n");
    printf("   ✅ for (i = 0; i < n; i++) a[i] = b[i] + c[i];  // Независимые\n");
    printf("   ❌ for (i = 1; i < n; i++) a[i] = a[i-1] + b[i]; // Зависимость\n\n");
    
    printf("4. Флаги компиляции:\n");
    printf("   GCC: -O3 -march=native -ftree-vectorize\n");
    printf("   Clang: -O3 -march=native\n");
    printf("   MSVC: /O2 /arch:AVX2\n\n");
    
    printf("5. Анализ векторизации:\n");
    printf("   GCC: -fopt-info-vec\n");
    printf("   Clang: -Rpass=loop-vectorize\n");
    printf("   Intel: -qopt-report=5\n");
}
```

---

## 📈 Масштабируемость системы

### Производительность vs количество ядер

```c
void scalability_analysis() {
    printf("\n=== Анализ масштабируемости ===\n");
    
    // Симуляция различных алгоритмов
    typedef struct {
        char* name;
        double serial_fraction;    // Доля последовательного кода
        double communication_overhead; // Накладные расходы на синхронизацию
    } Algorithm;
    
    Algorithm algorithms[] = {
        {"Embarrassingly Parallel", 0.01, 0.01},
        {"Matrix Multiplication", 0.05, 0.02},  
        {"Quicksort", 0.15, 0.05},
        {"Graph Traversal", 0.30, 0.10},
        {"Sequential Algorithm", 0.95, 0.05}
    };
    int num_algorithms = sizeof(algorithms) / sizeof(algorithms[0]);
    
    int core_counts[] = {1, 2, 4, 8, 16, 32, 64};
    int num_core_counts = sizeof(core_counts) / sizeof(core_counts[0]);
    
    printf("Ускорение для различных алгоритмов:\n");
    printf("%-20s", "Алгоритм");
    for (int i = 0; i < num_core_counts; i++) {
        printf(" %2d ядер", core_counts[i]);
    }
    printf("\n");
    printf("────────────────────────────────────────────────────────\n");
    
    for (int a = 0; a < num_algorithms; a++) {
        Algorithm* alg = &algorithms[a];
        printf("%-20s", alg->name);
        
        for (int c = 0; c < num_core_counts; c++) {
            int cores = core_counts[c];
            
            if (cores == 1) {
                printf("  1.0x");
                continue;
            }
            
            // Модифицированный закон Амдала с накладными расходами
            double parallel_fraction = 1.0 - alg->serial_fraction;
            double comm_overhead = alg->communication_overhead * (cores - 1);
            
            double speedup = 1.0 / (alg->serial_fraction + 
                                   parallel_fraction / cores + 
                                   comm_overhead);
            
            printf(" %4.1fx", speedup);
        }
        printf("\n");
    }
    
    printf("\nВыводы:\n");
    printf("• Embarrassingly parallel задачи масштабируются почти линейно\n");
    printf("• Алгоритмы с зависимостями имеют ограниченную масштабируемость\n");
    printf("• Накладные расходы на коммуникацию растут с количеством ядер\n");
}
```

---

## 🔒 Синхронизация и когерентность

### Cache coherence протоколы

```c
void cache_coherence_demo() {
    printf("\n=== Когерентность кэша ===\n");
    
    printf("Протокол MESI (Modified, Exclusive, Shared, Invalid):\n\n");
    
    printf("Состояния кэш-линий:\n");
    printf("• Modified (M): Изменено, эксклюзивно в одном кэше\n");
    printf("• Exclusive (E): Чистое, эксклюзивно в одном кэше\n");
    printf("• Shared (S): Чистое, может быть в нескольких кэшах\n");
    printf("• Invalid (I): Недействительно\n\n");
    
    printf("Пример сценария:\n");
    printf("1. CPU0 читает адрес X → E (эксклюзивно в CPU0)\n");
    printf("2. CPU1 читает адрес X → S (теперь разделяется)\n");
    printf("3. CPU0 пишет в X → M (CPU1 получает invalidate)\n");
    printf("4. CPU1 кэш помечает X как I (невалидно)\n");
    printf("5. CPU1 читает X → miss, запрос к CPU0\n");
    printf("6. CPU0 отправляет данные → оба в состоянии S\n\n");
    
    printf("False Sharing проблема:\n");
    printf("```c\n");
    printf("struct {\n");
    printf("    int counter1;  // Используется CPU0\n");
    printf("    int counter2;  // Используется CPU1\n");
    printf("} shared_data;  // В одной кэш-линии!\n\n");
    
    printf("// CPU0 и CPU1 непрерывно invalidate друг друга\n");
    printf("// Решение: выравнивание или padding\n");
    printf("struct {\n");
    printf("    int counter1;\n");
    printf("    char padding[64 - sizeof(int)];\n");
    printf("    int counter2;\n");
    printf("} optimized_data;\n");
    printf("```\n");
}
```

### Atomic операции и lock-free структуры

```c
#include <stdatomic.h>

// Простая lock-free очередь
typedef struct Node {
    atomic_intptr_t data;
    atomic(struct Node*) next;
} Node;

typedef struct {
    atomic(Node*) head;
    atomic(Node*) tail;
} LockFreeQueue;

void lockfree_demo() {
    printf("\n=== Lock-free структуры данных ===\n");
    
    printf("Преимущества lock-free:\n");
    printf("• Нет блокировок → нет deadlock'ов\n");
    printf("• Прогресс гарантирован хотя бы для одного потока\n");
    printf("• Лучшая масштабируемость на многих ядрах\n\n");
    
    printf("Недостатки:\n");
    printf("• Сложность реализации и отладки\n");
    printf("• ABA проблема\n");
    printf("• Memory ordering сложности\n\n");
    
    printf("Atomic операции:\n");
    printf("```c\n");
    printf("atomic_int counter = 0;\n");
    printf("int old_val = atomic_fetch_add(&counter, 1);  // Атомарно\n");
    printf("bool success = atomic_compare_exchange_strong(&ptr, &expected, new_val);\n");
    printf("```\n\n");
    
    printf("Memory ordering:\n");
    printf("• memory_order_relaxed: Только атомарность\n");
    printf("• memory_order_acquire: Барьер для loads\n");
    printf("• memory_order_release: Барьер для stores\n");
    printf("• memory_order_seq_cst: Последовательная консистентность\n");
}
```

---

## 🎯 Практические применения

### Высокопроизводительные сервера

```c
void hpc_architecture_patterns() {
    printf("\n=== Паттерны HPC архитектур ===\n");
    
    printf("1. Shared Memory (SMP):\n");
    printf("   • До 4-8 сокетов\n");
    printf("   • Общая память через crossbar/mesh\n");
    printf("   • Применение: Базы данных, in-memory аналитика\n\n");
    
    printf("2. Distributed Memory (Cluster):\n");
    printf("   • Тысячи узлов\n");
    printf("   • Коммуникация через сеть (InfiniBand/Ethernet)\n");
    printf("   • Применение: Научные симуляции, ML обучение\n\n");
    
    printf("3. Hybrid (NUMA + Network):\n");
    printf("   • NUMA узлы внутри сервера\n");
    printf("   • Кластер из NUMA серверов\n");
    printf("   • Применение: Современные суперкомпьютеры\n\n");
    
    printf("Топологии соединений:\n");
    printf("• Mesh: Каждый узел соединен с соседями\n");
    printf("• Torus: Mesh с wrapped connections\n");
    printf("• Fat tree: Иерархическая структура\n");
    printf("• Dragonfly: Группы узлов с all-to-all внутри\n");
}
```

---

## 🔗 Связанные темы

- [[microarchitecture|Микроархитектура]] - детали реализации параллелизма
- [[gpu-architectures|GPU архитектуры]] - массивный параллелизм
- [[memory-hierarchy|Иерархия памяти]] - NUMA и cache coherence
- [[performance-analysis|Анализ производительности]] - профилирование параллельного кода

## 📚 Дополнительные ресурсы

### 📖 Книги
- "The Art of Multiprocessor Programming" - Herlihy & Shavit
- "Parallel Computer Architecture" - Culler, Singh, Gupta
- "Programming Massively Parallel Processors" - Kirk & Hwu

### 🛠️ Инструменты
- **Intel VTune** - профилирование многопоточных приложений
- **ThreadSanitizer** - обнаружение race conditions
- **Intel Inspector** - анализ параллельного кода
- **TAU** - профилирование HPC приложений

### 🔬 Бенчмарки
- **STREAM** - пропускная способность памяти
- **HPL (Linpack)** - производительность HPC
- **PARSEC** - параллельные бенчмарки
- **SPLASH** - shared memory workloads

---

> **Следующий шаг**: Изучите [[gpu-architectures|GPU архитектуры]] для понимания массивно-параллельных вычислений. 