# Архитектуры процессоров 🔧

> **Навигация**: [[README|← Архитектура компьютеров]] | [[memory-hierarchy|Иерархия памяти →]]

## Содержание
1. [CISC vs RISC](#🔧-cisc-vs-risc)
2. [Архитектуры набора команд (ISA)](#📋-архитектуры-набора-команд-isa)
3. [Архитектура x86-64](#💻-архитектура-x86-64)
4. [ARM архитектура](#📱-arm-архитектура)
5. [RISC-V](#🆕-risc-v)
6. [Практические применения](#🚀-практические-применения)

---

## 🔧 CISC vs RISC

**Complex Instruction Set Computer (CISC)**
- Сложные инструкции, выполняющие несколько операций
- Переменная длина инструкций
- Примеры: x86, x86-64

**Reduced Instruction Set Computer (RISC)**
- Простые инструкции фиксированной длины
- Регистровая архитектура
- Примеры: ARM, RISC-V, MIPS

```c
#include <stdio.h>
#include <string.h>

// Демонстрация различий в архитектурах
void cisc_style_example() {
    printf("=== CISC архитектура ===\n");
    
    char source[] = "Hello, World!";
    char dest[20];
    
    // Эквивалент сложной CISC инструкции
    strcpy(dest, source);  // Одна "инструкция" - много микроопераций
    
    printf("CISC: сложная операция копирования строки\n");
    printf("Источник: %s\n", source);
    printf("Назначение: %s\n", dest);
}

void risc_style_example() {
    printf("\n=== RISC архитектура ===\n");
    
    char source[] = "Hello, World!";
    char dest[20];
    
    // RISC разбивает на простые операции
    char* src_ptr = source;
    char* dst_ptr = dest;
    
    // Цикл из простых инструкций
    while (*src_ptr) {
        *dst_ptr = *src_ptr;  // LOAD + STORE
        src_ptr++;            // ADD
        dst_ptr++;            // ADD
    }
    *dst_ptr = '\0';          // STORE
    
    printf("RISC: последовательность простых операций\n");
    printf("Источник: %s\n", source);
    printf("Назначение: %s\n", dest);
}
```

**Сравнение характеристик:**

| Характеристика | CISC | RISC |
|----------------|------|------|
| **Сложность инструкций** | Высокая | Низкая |
| **Длина инструкции** | Переменная | Фиксированная |
| **Количество инструкций** | Меньше в программе | Больше в программе |
| **Скорость выполнения** | Медленнее на инструкцию | Быстрее на инструкцию |
| **Микрокод** | Используется | Минимальный |
| **Энергопотребление** | Выше | Ниже |

---

## 📋 Архитектуры набора команд (ISA)

### Стековая архитектура

```c
typedef struct {
    int stack[100];
    int top;
} StackMachine;

void stack_push(StackMachine* sm, int value) {
    if (sm->top < 99) {
        sm->stack[++sm->top] = value;
    }
}

int stack_pop(StackMachine* sm) {
    if (sm->top >= 0) {
        return sm->stack[sm->top--];
    }
    return 0;
}

void stack_architecture_demo() {
    printf("\n=== Стековая архитектура ===\n");
    
    StackMachine sm = {.top = -1};
    
    // Вычисление (3 + 4) * 2
    printf("Вычисление: (3 + 4) * 2\n");
    
    stack_push(&sm, 3);     // PUSH 3
    printf("PUSH 3\n");
    
    stack_push(&sm, 4);     // PUSH 4  
    printf("PUSH 4\n");
    
    // ADD берет два верхних элемента
    int b = stack_pop(&sm);
    int a = stack_pop(&sm);
    stack_push(&sm, a + b);
    printf("ADD -> %d\n", a + b);
    
    stack_push(&sm, 2);     // PUSH 2
    printf("PUSH 2\n");
    
    // MUL
    b = stack_pop(&sm);
    a = stack_pop(&sm);
    stack_push(&sm, a * b);
    printf("MUL -> %d\n", a * b);
    
    printf("Результат: %d\n", stack_pop(&sm));
}
```

### Регистровая архитектура

```c
typedef struct {
    int registers[8];  // R0-R7
} RegisterMachine;

void register_architecture_demo() {
    printf("\n=== Регистровая архитектура ===\n");
    
    RegisterMachine rm = {0};
    
    // Вычисление (3 + 4) * 2
    printf("Вычисление: (3 + 4) * 2\n");
    
    rm.registers[0] = 3;        // MOV R0, 3
    printf("MOV R0, 3\n");
    
    rm.registers[1] = 4;        // MOV R1, 4
    printf("MOV R1, 4\n");
    
    rm.registers[2] = rm.registers[0] + rm.registers[1];  // ADD R2, R0, R1
    printf("ADD R2, R0, R1 -> R2 = %d\n", rm.registers[2]);
    
    rm.registers[3] = 2;        // MOV R3, 2
    printf("MOV R3, 2\n");
    
    rm.registers[4] = rm.registers[2] * rm.registers[3];  // MUL R4, R2, R3
    printf("MUL R4, R2, R3 -> R4 = %d\n", rm.registers[4]);
    
    printf("Результат в R4: %d\n", rm.registers[4]);
}
```

---

## 💻 Архитектура x86-64

### Режимы адресации

```c
void x86_addressing_modes() {
    printf("\n=== Режимы адресации x86-64 ===\n");
    
    int array[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    int* base = array;
    int index = 3;
    int scale = 1;
    int displacement = 4;
    
    // 1. Прямая адресация
    int direct = array[0];
    printf("Прямая: array[0] = %d\n", direct);
    
    // 2. Косвенная через регистр
    int indirect = *base;
    printf("Косвенная: *base = %d\n", indirect);
    
    // 3. Индексная: base + index
    int indexed = base[index];
    printf("Индексная: base[%d] = %d\n", index, indexed);
    
    // 4. Масштабированная: base + index*scale
    int scaled = base[index * scale];
    printf("Масштабированная: base[%d*%d] = %d\n", index, scale, scaled);
    
    // 5. Со смещением: base + displacement
    int displaced = *((int*)((char*)base + displacement));
    printf("Со смещением: *(base + %d) = %d\n", displacement, displaced);
}
```

### Регистры x86-64

```c
#include <stdint.h>

void x86_registers_demo() {
    printf("\n=== Регистры x86-64 ===\n");
    
    // Общего назначения (64-бит)
    uint64_t rax = 0x123456789ABCDEF0;  // Аккумулятор
    uint64_t rbx = 0x1111111111111111;  // База
    uint64_t rcx = 0x2222222222222222;  // Счетчик
    uint64_t rdx = 0x3333333333333333;  // Данные
    
    printf("64-битные регистры:\n");
    printf("RAX: 0x%016llX\n", rax);
    printf("RBX: 0x%016llX\n", rbx);
    printf("RCX: 0x%016llX\n", rcx);
    printf("RDX: 0x%016llX\n", rdx);
    
    // Части регистров
    uint32_t eax = (uint32_t)rax;       // Младшие 32 бита
    uint16_t ax = (uint16_t)rax;        // Младшие 16 бит
    uint8_t al = (uint8_t)rax;          // Младшие 8 бит
    uint8_t ah = (uint8_t)(rax >> 8);   // Биты 8-15
    
    printf("\nЧасти регистра RAX:\n");
    printf("EAX (32-bit): 0x%08X\n", eax);
    printf("AX (16-bit):  0x%04X\n", ax);
    printf("AL (8-bit):   0x%02X\n", al);
    printf("AH (8-bit):   0x%02X\n", ah);
}
```

---

## 📱 ARM архитектура

```c
// Симуляция ARM инструкций
typedef struct {
    uint32_t registers[16];  // R0-R15
    uint32_t cpsr;          // Current Program Status Register
} ARMProcessor;

// ARM использует условное выполнение
typedef enum {
    COND_EQ = 0,  // Equal
    COND_NE = 1,  // Not Equal
    COND_GE = 10, // Greater or Equal
    COND_LT = 11, // Less Than
    COND_AL = 14  // Always
} ARMCondition;

void arm_conditional_execution() {
    printf("\n=== ARM условное выполнение ===\n");
    
    ARMProcessor arm = {0};
    
    // Пример: сравнение и условное выполнение
    arm.registers[0] = 5;   // R0 = 5
    arm.registers[1] = 3;   // R1 = 3
    
    printf("R0 = %d, R1 = %d\n", arm.registers[0], arm.registers[1]);
    
    // CMP R0, R1 (устанавливает флаги)
    int comparison = arm.registers[0] - arm.registers[1];
    bool zero_flag = (comparison == 0);
    bool negative_flag = (comparison < 0);
    
    printf("После CMP R0, R1:\n");
    printf("Zero flag: %d, Negative flag: %d\n", zero_flag, negative_flag);
    
    // MOVGT R2, #100  (выполняется если R0 > R1)
    if (!zero_flag && !negative_flag) {
        arm.registers[2] = 100;
        printf("MOVGT R2, #100 выполнена -> R2 = %d\n", arm.registers[2]);
    }
    
    // MOVLE R2, #200  (выполняется если R0 <= R1)
    if (zero_flag || negative_flag) {
        arm.registers[2] = 200;
        printf("MOVLE R2, #200 выполнена -> R2 = %d\n", arm.registers[2]);
    }
}

void arm_load_store_multiple() {
    printf("\n=== ARM групповые операции ===\n");
    
    ARMProcessor arm = {0};
    uint32_t memory[100] = {0};
    
    // Инициализация регистров
    for (int i = 0; i < 8; i++) {
        arm.registers[i] = i * 10;
    }
    
    printf("Исходные значения регистров R0-R7:\n");
    for (int i = 0; i < 8; i++) {
        printf("R%d = %d ", i, arm.registers[i]);
    }
    printf("\n");
    
    // STM (Store Multiple) - сохранить R0-R7 в память
    for (int i = 0; i < 8; i++) {
        memory[i] = arm.registers[i];
    }
    printf("STM: сохранили R0-R7 в память\n");
    
    // Изменяем регистры
    for (int i = 0; i < 8; i++) {
        arm.registers[i] = 0;
    }
    
    // LDM (Load Multiple) - загрузить из памяти в R0-R7
    for (int i = 0; i < 8; i++) {
        arm.registers[i] = memory[i];
    }
    
    printf("После LDM восстановили R0-R7:\n");
    for (int i = 0; i < 8; i++) {
        printf("R%d = %d ", i, arm.registers[i]);
    }
    printf("\n");
}
```

---

## 🆕 RISC-V

```c
// Симуляция базовых инструкций RISC-V
typedef struct {
    uint64_t x[32];    // x0-x31 регистры (x0 всегда 0)
    uint64_t pc;       // Program Counter
} RISCVProcessor;

void riscv_basic_operations() {
    printf("\n=== RISC-V базовые операции ===\n");
    
    RISCVProcessor rv = {0};
    
    // x0 всегда равен 0
    rv.x[0] = 0;
    
    // Базовые арифметические операции
    rv.x[1] = 10;                    // addi x1, x0, 10
    rv.x[2] = 20;                    // addi x2, x0, 20
    rv.x[3] = rv.x[1] + rv.x[2];     // add x3, x1, x2
    rv.x[4] = rv.x[2] - rv.x[1];     // sub x4, x2, x1
    rv.x[5] = rv.x[1] << 1;          // slli x5, x1, 1 (сдвиг влево)
    
    printf("RISC-V операции:\n");
    printf("x1 = %ld (addi x1, x0, 10)\n", rv.x[1]);
    printf("x2 = %ld (addi x2, x0, 20)\n", rv.x[2]);
    printf("x3 = %ld (add x3, x1, x2)\n", rv.x[3]);
    printf("x4 = %ld (sub x4, x2, x1)\n", rv.x[4]);
    printf("x5 = %ld (slli x5, x1, 1)\n", rv.x[5]);
    
    // Условные переходы
    if (rv.x[3] == 30) {
        printf("Условие выполнено: x3 == 30\n");
    }
}

void riscv_memory_operations() {
    printf("\n=== RISC-V операции с памятью ===\n");
    
    RISCVProcessor rv = {0};
    uint8_t memory[1000] = {0};
    
    // Имитация загрузки/сохранения
    uint64_t address = 100;
    
    // Сохранение слова в память
    rv.x[1] = 0x12345678;
    *((uint32_t*)(memory + address)) = (uint32_t)rv.x[1];
    printf("sw x1, %ld(x0) - сохранили 0x%08X\n", address, (uint32_t)rv.x[1]);
    
    // Загрузка слова из памяти
    rv.x[2] = *((uint32_t*)(memory + address));
    printf("lw x2, %ld(x0) - загрузили 0x%08lX\n", address, rv.x[2]);
    
    // Загрузка байта
    rv.x[3] = *((uint8_t*)(memory + address));
    printf("lb x3, %ld(x0) - загрузили байт 0x%02lX\n", address, rv.x[3]);
}
```

---

## 🚀 Практические применения

### Выбор архитектуры для задач

```c
#include <time.h>
#include <stdlib.h>

// Тест производительности для разных типов операций
void performance_comparison() {
    printf("\n=== Сравнение производительности ===\n");
    
    const int SIZE = 1000000;
    int* array = malloc(SIZE * sizeof(int));
    
    // Инициализация
    for (int i = 0; i < SIZE; i++) {
        array[i] = rand() % 1000;
    }
    
    clock_t start, end;
    
    // Тест 1: Простые арифметические операции (RISC преимущество)
    start = clock();
    int sum = 0;
    for (int i = 0; i < SIZE; i++) {
        sum += array[i];           // Простая операция
    }
    end = clock();
    printf("Простые операции: %f секунд\n", 
           ((double)(end - start)) / CLOCKS_PER_SEC);
    
    // Тест 2: Сложные операции (CISC преимущество)
    start = clock();
    for (int i = 0; i < SIZE - 1; i++) {
        // Эмуляция сложной CISC операции
        if (array[i] > array[i + 1]) {
            int temp = array[i];
            array[i] = array[i + 1];
            array[i + 1] = temp;
        }
    }
    end = clock();
    printf("Сложные операции: %f секунд\n", 
           ((double)(end - start)) / CLOCKS_PER_SEC);
    
    free(array);
}

// Демонстрация различий в энергопотреблении
void power_efficiency_demo() {
    printf("\n=== Энергоэффективность ===\n");
    
    struct {
        const char* arch;
        int mips_per_watt;    // Миллионы инструкций в секунду на ватт
        int typical_power;     // Типичное потребление в ваттах
    } processors[] = {
        {"x86-64 Desktop", 100, 65},
        {"x86-64 Mobile", 200, 15},
        {"ARM Cortex-A78", 500, 5},
        {"ARM Cortex-M4", 1000, 0.1},
        {"RISC-V RV32", 800, 0.2}
    };
    
    printf("Сравнение энергоэффективности:\n");
    printf("%-20s %15s %15s\n", "Архитектура", "MIPS/Watt", "Мощность (W)");
    printf("---------------------------------------------------------\n");
    
    for (int i = 0; i < 5; i++) {
        printf("%-20s %15d %15d\n", 
               processors[i].arch,
               processors[i].mips_per_watt,
               processors[i].typical_power);
    }
}

// Рекомендации по выбору архитектуры
void architecture_recommendations() {
    printf("\n=== Рекомендации по выбору архитектуры ===\n");
    
    printf("CISC (x86-64) лучше для:\n");
    printf("  ✓ Настольных компьютеров\n");
    printf("  ✓ Серверов с высокой производительностью\n");
    printf("  ✓ Совместимости с существующим ПО\n");
    printf("  ✓ Сложных вычислений с разнообразными операциями\n\n");
    
    printf("RISC (ARM, RISC-V) лучше для:\n");
    printf("  ✓ Мобильных устройств\n");
    printf("  ✓ Встраиваемых систем\n");
    printf("  ✓ IoT устройств\n");
    printf("  ✓ Энергоэффективных решений\n");
    printf("  ✓ Параллельных вычислений\n\n");
    
    printf("Специализированные архитектуры для:\n");
    printf("  ✓ DSP - обработка сигналов\n");
    printf("  ✓ GPU - параллельные вычисления\n");
    printf("  ✓ NPU - нейронные сети\n");
    printf("  ✓ FPGA - настраиваемая логика\n");
}

int main() {
    cisc_style_example();
    risc_style_example();
    stack_architecture_demo();
    register_architecture_demo();
    x86_addressing_modes();
    x86_registers_demo();
    arm_conditional_execution();
    arm_load_store_multiple();
    riscv_basic_operations();
    riscv_memory_operations();
    performance_comparison();
    power_efficiency_demo();
    architecture_recommendations();
    
    return 0;
}
```

---

## 🔗 Связанные темы

- [[memory-hierarchy|Иерархия памяти]] - организация памяти в архитектурах
- [[microarchitecture|Микроархитектура]] - детали реализации
- [[parallel-architectures|Параллельные архитектуры]] - многоядерные системы
- [[performance-analysis|Анализ производительности]] - оптимизация под архитектуру

## 📚 Дополнительные ресурсы

- **Книги**: "Computer Architecture: A Quantitative Approach" - Hennessy, Patterson
- **Документация**: Intel Software Developer Manual, ARM Architecture Reference Manual
- **Курсы**: MIT 6.004, Stanford CS149
- **Симуляторы**: MARS MIPS, ARMv8 Fast Models 