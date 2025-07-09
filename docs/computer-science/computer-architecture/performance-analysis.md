# Анализ производительности 📊

> **Навигация**: [[README|← Архитектура компьютеров]] | [[gpu-architectures|← GPU архитектуры]]

## Содержание
1. [Методы измерения производительности](#⏱️-методы-измерения-производительности)
2. [Профилирование кода](#🔍-профилирование-кода)
3. [Бенчмаркинг](#🏁-бенчмаркинг)
4. [Метрики производительности](#📏-метрики-производительности)
5. [Оптимизация bottleneck'ов](#🚧-оптимизация-bottleneckов)

---

## ⏱️ Методы измерения производительности

### Точное измерение времени

```c
#include <sys/time.h>
#include <time.h>

double get_time() {
    struct timeval tv;
    gettimeofday(&tv, NULL);
    return tv.tv_sec + tv.tv_usec * 1e-6;
}

// Высокоточное измерение на x86
static inline uint64_t rdtsc() {
    uint32_t lo, hi;
    __asm__ __volatile__ ("rdtsc" : "=a" (lo), "=d" (hi));
    return ((uint64_t)hi << 32) | lo;
}

void timing_demo() {
    printf("=== Методы измерения времени ===\n\n");
    
    const int ITERATIONS = 1000000;
    
    // 1. gettimeofday (микросекундная точность)
    double start_time = get_time();
    volatile long sum1 = 0;
    for (int i = 0; i < ITERATIONS; i++) {
        sum1 += i * i;
    }
    double end_time = get_time();
    printf("gettimeofday: %.6f сек (сумма: %ld)\n", 
           end_time - start_time, sum1);
    
    // 2. clock() (процессорное время)
    clock_t start_clock = clock();
    volatile long sum2 = 0;
    for (int i = 0; i < ITERATIONS; i++) {
        sum2 += i * i;
    }
    clock_t end_clock = clock();
    printf("clock(): %.6f сек (сумма: %ld)\n", 
           (double)(end_clock - start_clock) / CLOCKS_PER_SEC, sum2);
    
    // 3. RDTSC (циклы процессора)
    uint64_t start_cycles = rdtsc();
    volatile long sum3 = 0;
    for (int i = 0; i < ITERATIONS; i++) {
        sum3 += i * i;
    }
    uint64_t end_cycles = rdtsc();
    printf("RDTSC: %lu циклов (сумма: %ld)\n", 
           end_cycles - start_cycles, sum3);
    
    printf("\nРекомендации:\n");
    printf("• gettimeofday(): для wall clock времени\n");
    printf("• clock(): для CPU времени\n");
    printf("• RDTSC: для микро-бенчмарков\n");
}
```

### Измерение пропускной способности памяти

```c
void memory_bandwidth_test() {
    printf("\n=== Тест пропускной способности памяти ===\n\n");
    
    const size_t sizes[] = {
        1024,           // 1KB - L1 кэш
        32 * 1024,      // 32KB - L1 кэш  
        256 * 1024,     // 256KB - L2 кэш
        8 * 1024 * 1024, // 8MB - L3 кэш
        128 * 1024 * 1024 // 128MB - RAM
    };
    const char* size_names[] = {"1KB", "32KB", "256KB", "8MB", "128MB"};
    const int num_sizes = sizeof(sizes) / sizeof(sizes[0]);
    
    printf("Размер      | Bandwidth | Латентность\n");
    printf("─────────────────────────────────────\n");
    
    for (int s = 0; s < num_sizes; s++) {
        size_t size = sizes[s];
        int* data = malloc(size);
        
        // Инициализация для избежания page faults
        for (size_t i = 0; i < size / sizeof(int); i++) {
            data[i] = i;
        }
        
        // Тест bandwidth (последовательное чтение)
        const int iterations = 1000;
        double start_time = get_time();
        
        volatile long sum = 0;
        for (int iter = 0; iter < iterations; iter++) {
            for (size_t i = 0; i < size / sizeof(int); i++) {
                sum += data[i];
            }
        }
        
        double end_time = get_time();
        double elapsed = end_time - start_time;
        double bytes_accessed = (double)size * iterations;
        double bandwidth = bytes_accessed / elapsed / (1024 * 1024 * 1024); // GB/s
        
        // Тест latency (случайный доступ)
        // ... код латентности ...
        
        printf("%-10s | %6.2f GB/s | %s\n", 
               size_names[s], bandwidth, 
               s < 3 ? "~1-10ns" : s == 3 ? "~50ns" : "~100ns");
        
        free(data);
    }
}
```

---

## 🔍 Профилирование кода

### CPU профилирование

```c
void cpu_profiling_demo() {
    printf("\n=== CPU профилирование ===\n\n");
    
    printf("Основные инструменты:\n\n");
    
    printf("1. gprof (GNU Profiler):\n");
    printf("   Компиляция: gcc -pg program.c\n");
    printf("   Запуск: ./a.out\n");
    printf("   Анализ: gprof a.out gmon.out\n");
    printf("   + Простота использования\n");
    printf("   - Overhead ~30%%, только user-space\n\n");
    
    printf("2. perf (Linux Performance Tools):\n");
    printf("   perf record ./program\n");
    printf("   perf report\n");
    printf("   perf stat -e cache-misses,branch-misses ./program\n");
    printf("   + Низкий overhead, kernel events\n");
    printf("   - Только Linux\n\n");
    
    printf("3. Intel VTune:\n");
    printf("   vtune -collect hotspots -result-dir r001hs ./program\n");
    printf("   vtune -report summary -result-dir r001hs\n");
    printf("   + Микроархитектурный анализ\n");
    printf("   - Только Intel CPU\n\n");
    
    printf("4. Valgrind (Callgrind):\n");
    printf("   valgrind --tool=callgrind ./program\n");
    printf("   kcachegrind callgrind.out.XXX\n");
    printf("   + Детальная информация\n");
    printf("   - Очень большой overhead (10-50x)\n");
}
```

### Hardware Performance Counters

```c
void performance_counters_demo() {
    printf("\n=== Hardware Performance Counters ===\n\n");
    
    printf("Основные события:\n\n");
    
    printf("Cache события:\n");
    printf("• cache-references: обращения к кэшу\n");
    printf("• cache-misses: промахи кэша\n");
    printf("• L1-dcache-loads: загрузки L1 кэша\n");
    printf("• L1-dcache-load-misses: промахи L1\n");
    printf("• LLC-loads: загрузки Last Level Cache\n");
    printf("• LLC-load-misses: промахи LLC\n\n");
    
    printf("Branch события:\n");
    printf("• branches: выполненные ветвления\n");
    printf("• branch-misses: неправильные предсказания\n\n");
    
    printf("Memory события:\n");
    printf("• dTLB-loads: обращения к TLB\n");
    printf("• dTLB-load-misses: промахи TLB\n");
    printf("• page-faults: ошибки страниц\n\n");
    
    printf("Пример использования perf:\n");
    printf("```bash\n");
    printf("# Базовая статистика\n");
    printf("perf stat ./program\n\n");
    
    printf("# Конкретные события\n");
    printf("perf stat -e cache-misses,branch-misses,page-faults ./program\n\n");
    
    printf("# Профилирование с sampling\n");
    printf("perf record -e cache-misses ./program\n");
    printf("perf report\n\n");
    
    printf("# Системный профайлинг\n");
    printf("perf top\n");
    printf("```\n");
}
```

---

## 🏁 Бенчмаркинг

### Микро-бенчмарки

```c
void microbenchmark_example() {
    printf("\n=== Микро-бенчмарки ===\n\n");
    
    const int ITERATIONS = 10000000;
    
    printf("Сравнение операций (млн операций/сек):\n\n");
    
    // 1. Целочисленное сложение
    uint64_t start = rdtsc();
    volatile int sum = 0;
    for (int i = 0; i < ITERATIONS; i++) {
        sum += i;
    }
    uint64_t end = rdtsc();
    double int_add_cycles = (double)(end - start) / ITERATIONS;
    
    // 2. Целочисленное умножение
    start = rdtsc();
    volatile int prod = 1;
    for (int i = 1; i < ITERATIONS; i++) {
        prod *= (i % 1000 + 1);  // Избегаем overflow
    }
    end = rdtsc();
    double int_mul_cycles = (double)(end - start) / ITERATIONS;
    
    // 3. Плавающая точка сложение
    start = rdtsc();
    volatile double fsum = 0.0;
    for (int i = 0; i < ITERATIONS; i++) {
        fsum += i * 0.1;
    }
    end = rdtsc();
    double float_add_cycles = (double)(end - start) / ITERATIONS;
    
    // 4. Плавающая точка деление
    start = rdtsc();
    volatile double fdiv = 1.0;
    for (int i = 1; i < ITERATIONS; i++) {
        fdiv = fdiv / (1.0 + 1e-10);
    }
    end = rdtsc();
    double float_div_cycles = (double)(end - start) / ITERATIONS;
    
    printf("Операция           | Циклов на операцию\n");
    printf("─────────────────────────────────────\n");
    printf("INT ADD            | %.2f\n", int_add_cycles);
    printf("INT MUL            | %.2f\n", int_mul_cycles);
    printf("FLOAT ADD          | %.2f\n", float_add_cycles);
    printf("FLOAT DIV          | %.2f\n", float_div_cycles);
    
    printf("\nОтносительная стоимость:\n");
    printf("INT MUL vs ADD: %.1fx\n", int_mul_cycles / int_add_cycles);
    printf("FLOAT DIV vs ADD: %.1fx\n", float_div_cycles / float_add_cycles);
}
```

### Системные бенчмарки

```c
void system_benchmarks() {
    printf("\n=== Системные бенчмарки ===\n\n");
    
    printf("Популярные benchmark suites:\n\n");
    
    printf("SPEC CPU2017:\n");
    printf("• Индустриальный стандарт\n");
    printf("• Integer и Floating Point workloads\n");
    printf("• Rate (throughput) и Speed (latency)\n");
    printf("• Стоимость: коммерческий\n\n");
    
    printf("STREAM:\n");
    printf("• Пропускная способность памяти\n");
    printf("• Copy, Scale, Add, Triad операции\n");
    printf("• Бесплатный, простой\n");
    printf("• Команда: gcc -O3 -fopenmp stream.c && ./a.out\n\n");
    
    printf("Linpack (HPL):\n");
    printf("• Решение системы линейных уравнений\n");
    printf("• Используется в TOP500\n");
    printf("• FLOPS метрика\n");
    printf("• Хорошо оптимизируется\n\n");
    
    printf("PARSEC:\n");
    printf("• Параллельные workloads\n");
    printf("• Реальные приложения\n");
    printf("• Различные типы параллелизма\n");
    printf("• Исследовательская направленность\n\n");
    
    printf("Собственные бенчмарки:\n");
    printf("✅ Представляют реальную нагрузку\n");
    printf("✅ Фокус на критичных операциях\n");
    printf("❌ Требуют разработки и поддержки\n");
    printf("❌ Сложно сравнивать с другими системами\n");
}
```

---

## 📏 Метрики производительности

### Основные метрики

```c
void performance_metrics() {
    printf("\n=== Ключевые метрики производительности ===\n\n");
    
    printf("Throughput метрики:\n");
    printf("• IPC (Instructions Per Cycle)\n");
    printf("• FLOPS (Floating-Point Operations Per Second)\n");
    printf("• Bandwidth (GB/s)\n");
    printf("• QPS (Queries Per Second)\n");
    printf("• TPS (Transactions Per Second)\n\n");
    
    printf("Latency метрики:\n");
    printf("• Время отклика (response time)\n");
    printf("• Время выполнения (execution time)\n");
    printf("• Персентили (P50, P95, P99)\n\n");
    
    printf("Efficiency метрики:\n");
    printf("• CPU utilization (%)\n");
    printf("• Memory utilization (%)\n");
    printf("• Cache hit rate (%)\n");
    printf("• Branch prediction accuracy (%)\n\n");
    
    printf("Пример расчета IPC:\n");
    printf("```bash\n");
    printf("perf stat -e instructions,cycles ./program\n");
    printf("# Результат:\n");
    printf("# 1,234,567,890 instructions\n");
    printf("#   987,654,321 cycles\n");
    printf("# IPC = 1,234,567,890 / 987,654,321 = 1.25\n");
    printf("```\n\n");
    
    printf("Интерпретация IPC:\n");
    printf("• IPC < 1: CPU-bound, много stalls\n");
    printf("• IPC ≈ 1: Сбалансированная нагрузка\n");
    printf("• IPC > 1: Суперскалярное выполнение\n");
    printf("• IPC > 4: Очень эффективный код\n");
}
```

---

## 🚧 Оптимизация bottleneck'ов

### Типичные узкие места

```c
void common_bottlenecks() {
    printf("\n=== Типичные bottleneck'и ===\n\n");
    
    printf("1. Memory-bound:\n");
    printf("   Признаки:\n");
    printf("   • Высокий cache miss rate\n");
    printf("   • Низкий IPC (~0.5-1.0)\n");
    printf("   • Много stalls на memory\n");
    printf("   \n");
    printf("   Решения:\n");
    printf("   ✅ Улучшить локальность данных\n");
    printf("   ✅ Использовать prefetching\n");
    printf("   ✅ Оптимизировать структуры данных\n");
    printf("   ✅ Уменьшить memory footprint\n\n");
    
    printf("2. CPU-bound:\n");
    printf("   Признаки:\n");
    printf("   • Высокий CPU utilization (>90%%)\n");
    printf("   • Высокий IPC (>2.0)\n");
    printf("   • Низкий cache miss rate\n");
    printf("   \n");
    printf("   Решения:\n");
    printf("   ✅ Алгоритмическая оптимизация\n");
    printf("   ✅ Векторизация (SIMD)\n");
    printf("   ✅ Параллелизация\n");
    printf("   ✅ Компилерные оптимизации\n\n");
    
    printf("3. Branch-bound:\n");
    printf("   Признаки:\n");
    printf("   • Высокий branch miss rate (>5%%)\n");
    printf("   • Много branch misprediction stalls\n");
    printf("   \n");
    printf("   Решения:\n");
    printf("   ✅ Уменьшить количество ветвлений\n");
    printf("   ✅ Использовать branchless код\n");
    printf("   ✅ Profile-guided optimization\n\n");
    
    printf("4. I/O-bound:\n");
    printf("   Признаки:\n");
    printf("   • Низкий CPU utilization\n");
    printf("   • Много времени в wait состоянии\n");
    printf("   \n");
    printf("   Решения:\n");
    printf("   ✅ Асинхронный I/O\n");
    printf("   ✅ Буферизация\n");
    printf("   ✅ Батчинг операций\n");
    printf("   ✅ SSD вместо HDD\n");
}
```

### Стратегии оптимизации

```c
void optimization_strategies() {
    printf("\n=== Стратегии оптимизации ===\n\n");
    
    printf("Поэтапный подход:\n\n");
    
    printf("1. Измерить (Measure):\n");
    printf("   • Профилирование real workload\n");
    printf("   • Определение hotspots (80/20 rule)\n");
    printf("   • Baseline метрики\n\n");
    
    printf("2. Анализировать (Analyze):\n");
    printf("   • Тип bottleneck'а\n");
    printf("   • Root cause analysis\n");
    printf("   • Теоретические пределы\n\n");
    
    printf("3. Оптимизировать (Optimize):\n");
    printf("   • Начинать с самых горячих функций\n");
    printf("   • Одно изменение за раз\n");
    printf("   • A/B тестирование\n\n");
    
    printf("4. Проверить (Verify):\n");
    printf("   • Измерить снова\n");
    printf("   • Проверить корректность\n");
    printf("   • Regression тесты\n\n");
    
    printf("Порядок оптимизаций:\n");
    printf("1. Алгоритм и структуры данных\n");
    printf("2. Компилерные флаги (-O3, -march=native)\n");
    printf("3. Локальность памяти\n");
    printf("4. Векторизация\n");
    printf("5. Параллелизация\n");
    printf("6. Микро-оптимизации\n\n");
    
    printf("❗ Правило: профилируйте до и после каждой оптимизации!\n");
}
```

---

## 🔗 Связанные темы

- [[memory-hierarchy|Иерархия памяти]] - оптимизация доступа к памяти
- [[microarchitecture|Микроархитектура]] - понимание pipeline и предсказания
- [[parallel-architectures|Параллельные архитектуры]] - многопоточная оптимизация
- [[gpu-architectures|GPU архитектуры]] - GPU профилирование

## 📚 Дополнительные ресурсы

### 🛠️ Инструменты профилирования

**Linux:**
- **perf** - системный профайлер
- **gprof** - GNU профайлер  
- **Valgrind** - память и производительность
- **Intel VTune** - микроархитектурный анализ

**Cross-platform:**
- **Intel Inspector** - thread safety
- **AMD CodeXL** - CPU/GPU профилирование
- **Arm MAP** - HPC профайлер

### 📊 Бенчмарки

**CPU:**
- **SPEC CPU2017** - индустриальный стандарт
- **STREAM** - memory bandwidth
- **Linpack** - научные вычисления

**Системные:**
- **PARSEC** - параллельные workloads
- **SPLASH** - shared memory
- **NPB** - параллельные бенчмарки

---

> **Заключение**: Эффективная оптимизация требует понимания архитектуры, правильного измерения и систематического подхода. Профилируйте реальные workloads и фокусируйтесь на самых критичных bottleneck'ах. 