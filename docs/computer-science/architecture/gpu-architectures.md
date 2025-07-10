# GPU архитектуры 🚀

> **Навигация**: [[README|← Архитектура компьютеров]] | [[parallel-architectures|← Параллельные архитектуры]] | [[performance-analysis|Анализ производительности →]]

## Содержание
1. [Основы GPU архитектуры](#🎮-основы-gpu-архитектуры)
2. [CUDA программирование](#⚡-cuda-программирование)
3. [Иерархия памяти GPU](#🧠-иерархия-памяти-gpu)
4. [Оптимизация производительности](#🚀-оптимизация-производительности)
5. [Современные GPU архитектуры](#🔬-современные-gpu-архитектуры)

---

## 🎮 Основы GPU архитектуры

### GPU vs CPU парадигма

```c
void gpu_vs_cpu_concepts() {
    printf("=== GPU vs CPU архитектура ===\n\n");
    
    printf("CPU архитектура:\n");
    printf("• Мало ядер (4-32), сложные\n");
    printf("• Высокая частота (3-5 ГГц)\n");
    printf("• Большие кэши (MB)\n");
    printf("• Оптимизирован для latency\n");
    printf("• Сложное предсказание ветвлений\n");
    printf("• Out-of-order execution\n\n");
    
    printf("GPU архитектура:\n");
    printf("• Много ядер (1000+), простые\n");
    printf("• Низкая частота (1-2 ГГц)\n");
    printf("• Малые кэши (KB)\n");
    printf("• Оптимизирован для throughput\n");
    printf("• Простое/отсутствующее предсказание\n");
    printf("• In-order execution\n\n");
    
    printf("Применимость:\n");
    printf("CPU: Последовательные алгоритмы, сложная логика\n");
    printf("GPU: Параллельные алгоритмы, простая арифметика\n");
}
```

---

## ⚡ CUDA программирование

### Иерархия threads

```c
void cuda_hierarchy_concepts() {
    printf("=== CUDA иерархия выполнения ===\n\n");
    
    printf("Трехуровневая иерархия:\n");
    printf("Grid → Blocks → Threads\n\n");
    
    printf("Thread:\n");
    printf("• Минимальная единица выполнения\n");
    printf("• Собственные регистры\n");
    printf("• Выполняет один CUDA kernel\n\n");
    
    printf("Block (Thread Block):\n");
    printf("• Группа до 1024 threads\n");
    printf("• Общая shared memory (48KB)\n");
    printf("• Синхронизация через __syncthreads()\n");
    printf("• Выполняется на одном SM\n\n");
    
    printf("Grid:\n");
    printf("• Группа blocks\n");
    printf("• Один kernel = один grid\n");
    printf("• Blocks независимы\n\n");
    
    printf("Пример kernel:\n");
    printf("```c\n");
    printf("__global__ void vector_add(float* a, float* b, float* c, int n) {\n");
    printf("    int idx = blockIdx.x * blockDim.x + threadIdx.x;\n");
    printf("    if (idx < n) {\n");
    printf("        c[idx] = a[idx] + b[idx];\n");
    printf("    }\n");
    printf("}\n\n");
    
    printf("// Запуск:\n");
    printf("dim3 blockSize(256);\n");
    printf("dim3 gridSize((n + blockSize.x - 1) / blockSize.x);\n");
    printf("vector_add<<<gridSize, blockSize>>>(d_a, d_b, d_c, n);\n");
    printf("```\n");
}
```

### Warp execution model

```c
void warp_execution_demo() {
    printf("\n=== Warp модель выполнения ===\n\n");
    
    printf("Warp (варп):\n");
    printf("• 32 threads выполняются синхронно\n");
    printf("• SIMT (Single Instruction, Multiple Thread)\n");
    printf("• Все threads в warp выполняют одну инструкцию\n\n");
    
    printf("Branch divergence:\n");
    printf("```c\n");
    printf("// ПЛОХО - divergent branches:\n");
    printf("if (threadIdx.x %% 2 == 0) {\n");
    printf("    result = a[idx] * 2;      // 16 threads\n");
    printf("} else {\n");
    printf("    result = a[idx] + 1;      // 16 threads\n");
    printf("}\n");
    printf("// Эффективность: 50%% (сериализация)\n\n");
    
    printf("// ХОРОШО - no divergence:\n");
    printf("if (blockIdx.x %% 2 == 0) {\n");
    printf("    result = a[idx] * 2;      // Весь warp\n");
    printf("} else {\n");
    printf("    result = a[idx] + 1;      // Весь warp\n");
    printf("}\n");
    printf("// Эффективность: 100%%\n");
    printf("```\n\n");
    
    printf("Occupancy:\n");
    printf("• Отношение active warps / maximum warps\n");
    printf("• Цель: высокий occupancy для скрытия latency\n");
    printf("• Ограничения: регистры, shared memory, block size\n");
}
```

---

## 🧠 Иерархия памяти GPU

### Типы памяти

```c
void gpu_memory_hierarchy() {
    printf("\n=== Иерархия памяти GPU ===\n\n");
    
    printf("1. Registers (на thread):\n");
    printf("   • Размер: ~64KB на SM\n");
    printf("   • Латентность: 1 цикл\n");
    printf("   • Пропускная способность: >40 TB/s\n");
    printf("   • Видимость: thread-local\n\n");
    
    printf("2. Shared Memory (на block):\n");
    printf("   • Размер: 48-164KB на SM\n");
    printf("   • Латентность: 1-32 цикла\n");
    printf("   • Пропускная способность: ~19 TB/s\n");
    printf("   • Видимость: block-local\n\n");
    
    printf("3. Global Memory:\n");
    printf("   • Размер: 8-80GB (весь GPU)\n");
    printf("   • Латентность: 200-800 циклов\n");
    printf("   • Пропускная способность: 0.5-1.5 TB/s\n");
    printf("   • Видимость: все threads\n\n");
    
    printf("4. Constant Memory:\n");
    printf("   • Размер: 64KB\n");
    printf("   • Кэшируется\n");
    printf("   • Read-only\n\n");
    
    printf("5. Texture Memory:\n");
    printf("   • Оптимизирована для 2D локальности\n");
    printf("   • Фильтрация\n");
    printf("   • Read-only\n");
}
```

### Coalesced memory access

```c
void memory_coalescing_demo() {
    printf("\n=== Memory Coalescing ===\n\n");
    
    printf("Coalesced (эффективный) доступ:\n");
    printf("```c\n");
    printf("__global__ void good_access(float* data) {\n");
    printf("    int idx = blockIdx.x * blockDim.x + threadIdx.x;\n");
    printf("    float value = data[idx];  // Последовательные адреса\n");
    printf("    // Thread 0: data[0], Thread 1: data[1], ...\n");
    printf("}\n");
    printf("```\n");
    printf("Результат: 1 транзакция памяти для 32 threads\n\n");
    
    printf("Non-coalesced (неэффективный) доступ:\n");
    printf("```c\n");
    printf("__global__ void bad_access(float* data) {\n");
    printf("    int idx = blockIdx.x * blockDim.x + threadIdx.x;\n");
    printf("    float value = data[idx * 37];  // Stride 37\n");
    printf("    // Thread 0: data[0], Thread 1: data[37], ...\n");
    printf("}\n");
    printf("```\n");
    printf("Результат: 32 транзакции памяти для 32 threads\n\n");
    
    printf("Влияние на производительность:\n");
    printf("• Coalesced: ~900 GB/s\n");
    printf("• Non-coalesced: ~30 GB/s\n");
    printf("• Замедление: до 30x!\n");
}
```

---

## 🚀 Оптимизация производительности

### Shared memory оптимизация

```c
void shared_memory_optimization() {
    printf("\n=== Shared Memory оптимизация ===\n\n");
    
    printf("Пример: Matrix transpose\n");
    printf("```c\n");
    printf("// Версия 1: Наивная (медленная)\n");
    printf("__global__ void naive_transpose(float* input, float* output, int n) {\n");
    printf("    int row = blockIdx.y * blockDim.y + threadIdx.y;\n");
    printf("    int col = blockIdx.x * blockDim.x + threadIdx.x;\n");
    printf("    \n");
    printf("    if (row < n && col < n) {\n");
    printf("        output[col * n + row] = input[row * n + col];\n");
    printf("    }\n");
    printf("}\n\n");
    
    printf("// Версия 2: С shared memory (быстрая)\n");
    printf("__global__ void optimized_transpose(float* input, float* output, int n) {\n");
    printf("    __shared__ float tile[32][33];  // +1 для избежания bank conflicts\n");
    printf("    \n");
    printf("    int row = blockIdx.y * 32 + threadIdx.y;\n");
    printf("    int col = blockIdx.x * 32 + threadIdx.x;\n");
    printf("    \n");
    printf("    // Coalesced load в shared memory\n");
    printf("    if (row < n && col < n) {\n");
    printf("        tile[threadIdx.y][threadIdx.x] = input[row * n + col];\n");
    printf("    }\n");
    printf("    __syncthreads();\n");
    printf("    \n");
    printf("    // Coalesced store из shared memory\n");
    printf("    row = blockIdx.x * 32 + threadIdx.y;\n");
    printf("    col = blockIdx.y * 32 + threadIdx.x;\n");
    printf("    if (row < n && col < n) {\n");
    printf("        output[row * n + col] = tile[threadIdx.x][threadIdx.y];\n");
    printf("    }\n");
    printf("}\n");
    printf("```\n\n");
    
    printf("Ускорение: до 10x благодаря:\n");
    printf("• Coalesced memory access\n");
    printf("• Использование fast shared memory\n");
    printf("• Избежание bank conflicts\n");
}
```

### Occupancy оптимизация

```c
void occupancy_optimization() {
    printf("\n=== Occupancy оптимизация ===\n\n");
    
    printf("Факторы, влияющие на occupancy:\n\n");
    
    printf("1. Регистры на thread:\n");
    printf("   • SM имеет 65536 регистров\n");
    printf("   • 256 threads/block × 64 reg/thread = 16384 reg\n");
    printf("   • Максимум 4 block на SM\n");
    printf("   • Если 128 reg/thread → только 2 block\n\n");
    
    printf("2. Shared memory на block:\n");
    printf("   • SM имеет 48KB shared memory\n");
    printf("   • 16KB/block → максимум 3 block\n");
    printf("   • 24KB/block → максимум 2 block\n\n");
    
    printf("3. Threads на block:\n");
    printf("   • Должно быть кратно 32 (warp size)\n");
    printf("   • Рекомендуется: 128, 256, 512\n");
    printf("   • Больше blocks = лучше load balancing\n\n");
    
    printf("Инструменты анализа:\n");
    printf("• nvprof --metrics achieved_occupancy\n");
    printf("• CUDA Occupancy Calculator\n");
    printf("• Nsight Compute\n\n");
    
    printf("Целевая occupancy: 50-100%%\n");
    printf("Низкий occupancy может быть OK для memory-bound kernels\n");
}
```

---

## 🔬 Современные GPU архитектуры

### Архитектуры NVIDIA

```c
void nvidia_architectures() {
    printf("\n=== Эволюция NVIDIA архитектур ===\n\n");
    
    printf("Maxwell (GTX 900):\n");
    printf("• 28nm процесс\n");
    printf("• Первая архитектура с unified memory\n");
    printf("• Улучшенный scheduler\n\n");
    
    printf("Pascal (GTX 10xx, P100):\n");
    printf("• 16nm FinFET\n");
    printf("• HBM2 память\n");
    printf("• Half-precision (FP16) support\n");
    printf("• NVLink interconnect\n\n");
    
    printf("Volta (V100):\n");
    printf("• 12nm FFN\n");
    printf("• Tensor Cores (mixed precision)\n");
    printf("• Independent thread scheduling\n");
    printf("• Cooperative groups\n\n");
    
    printf("Turing (RTX 20xx):\n");
    printf("• 12nm FFN\n");
    printf("• RT Cores (ray tracing)\n");
    printf("• Improved Tensor Cores\n");
    printf("• Variable rate shading\n\n");
    
    printf("Ampere (RTX 30xx, A100):\n");
    printf("• 7nm Samsung / 7nm TSMC\n");
    printf("• 3rd gen Tensor Cores\n");
    printf("• Multi-instance GPU (MIG)\n");
    printf("• Structural sparsity support\n\n");
    
    printf("Ada Lovelace (RTX 40xx):\n");
    printf("• 4nm TSMC\n");
    printf("• 4th gen RT Cores\n");
    printf("• DLSS 3.0\n");
    printf("• AV1 encoding\n\n");
    
    printf("Hopper (H100):\n");
    printf("• 4nm TSMC\n");
    printf("• 4th gen Tensor Cores\n");
    printf("• Transformer Engine\n");
    printf("• DPX instructions\n");
}
```

### Сравнение с AMD и Intel

```c
void gpu_vendor_comparison() {
    printf("\n=== Сравнение вендоров GPU ===\n\n");
    
    printf("NVIDIA:\n");
    printf("✅ Лидер в GPGPU\n");
    printf("✅ Зрелая CUDA экосистема\n");
    printf("✅ Отличные ML библиотеки\n");
    printf("✅ Tensor Cores для AI\n");
    printf("❌ Высокая стоимость\n");
    printf("❌ Проприетарные технологии\n\n");
    
    printf("AMD (RDNA/CDNA):\n");
    printf("✅ Открытый ROCm stack\n");
    printf("✅ Хорошее соотношение цена/производительность\n");
    printf("✅ Интеграция с CPU\n");
    printf("❌ Меньшая экосистема\n");
    printf("❌ Отставание в ML accelerators\n\n");
    
    printf("Intel (Arc/Ponte Vecchio):\n");
    printf("✅ oneAPI унифицированная модель\n");
    printf("✅ Интеграция CPU+GPU\n");
    printf("✅ Открытые стандарты\n");
    printf("❌ Новичок на рынке\n");
    printf("❌ Ограниченная поддержка legacy\n");
}
```

---

## 📊 Практические применения

### Machine Learning workloads

```c
void ml_gpu_optimization() {
    printf("\n=== ML оптимизация на GPU ===\n\n");
    
    printf("Типичные операции:\n");
    printf("• Matrix multiplication (GEMM): 80-90%% времени\n");
    printf("• Convolution: можно свести к GEMM\n");
    printf("• Element-wise operations: bandwidth-bound\n");
    printf("• Reductions: sync-intensive\n\n");
    
    printf("Tensor Cores оптимизация:\n");
    printf("```c\n");
    printf("// FP16 input/output, FP32 accumulate\n");
    printf("// C = A × B + C\n");
    printf("// A: [M×K] FP16, B: [K×N] FP16, C: [M×N] FP32\n");
    printf("cublasGemmEx(handle, CUBLAS_OP_N, CUBLAS_OP_N,\n");
    printf("             m, n, k,\n");
    printf("             &alpha, A, CUDA_R_16F, lda,\n");
    printf("                     B, CUDA_R_16F, ldb,\n");
    printf("             &beta,  C, CUDA_R_32F, ldc,\n");
    printf("             CUDA_R_32F, CUBLAS_GEMM_DEFAULT_TENSOR_OP);\n");
    printf("```\n\n");
    
    printf("Batch processing:\n");
    printf("• Увеличивайте batch size для лучшего GPU utilization\n");
    printf("• Используйте cudaStreamCreate() для overlap\n");
    printf("• Mixed precision для 2x speedup\n\n");
    
    printf("Memory optimization:\n");
    printf("• Gradient checkpointing для экономии памяти\n");
    printf("• Model parallelism для больших моделей\n");
    printf("• Unified memory для простоты разработки\n");
}
```

---

## 🔗 Связанные темы

- [[parallel-architectures|Параллельные архитектуры]] - общие принципы параллелизма
- [[performance-analysis|Анализ производительности]] - профилирование GPU кода
- [[memory-hierarchy|Иерархия памяти]] - принципы работы с памятью

## 📚 Дополнительные ресурсы

### 📖 Документация
- **CUDA Programming Guide** - официальная документация NVIDIA
- **CUDA Best Practices Guide** - рекомендации по оптимизации
- **PTX ISA** - низкоуровневый ассемблер GPU

### 🛠️ Инструменты профилирования
- **Nsight Compute** - детальный анализ kernel'ов
- **Nsight Systems** - системный профайлер
- **nvprof** - командная строка профайлер
- **NVIDIA Visual Profiler** - GUI профайлер

### 📊 Бенчмарки
- **CUDA Samples** - примеры кода
- **SHOC** - GPU benchmark suite
- **Rodinia** - параллельные алгоритмы

---

> **Следующий шаг**: Изучите [[performance-analysis|анализ производительности]] для практического измерения и оптимизации GPU кода. 