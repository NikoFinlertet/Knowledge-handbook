# Микроархитектура 🔬

> **Навигация**: [[README|← Архитектура компьютеров]] | [[memory-hierarchy|← Иерархия памяти]] | [[parallel-architectures|Параллельные архитектуры →]]

## Содержание
1. [Конвейер выполнения команд](#⚙️-конвейер-выполнения-команд)
2. [Предсказание ветвлений](#🎯-предсказание-ветвлений)
3. [Суперскалярные процессоры](#🚀-суперскалярные-процессоры)
4. [Out-of-Order выполнение](#🔀-out-of-order-выполнение)
5. [Спекулятивное выполнение](#🔮-спекулятивное-выполнение)

---

## ⚙️ Конвейер выполнения команд

### 5-ступенчатый конвейер RISC

```c
// Симуляция простого 5-ступенчатого конвейера
typedef enum {
    STAGE_FETCH = 0,    // IF - Instruction Fetch
    STAGE_DECODE,       // ID - Instruction Decode  
    STAGE_EXECUTE,      // EX - Execute
    STAGE_MEMORY,       // MEM - Memory Access
    STAGE_WRITEBACK,    // WB - Write Back
    STAGE_COUNT
} PipelineStage;

typedef struct {
    int instruction_id;
    int opcode;
    int src1, src2, dst;
    int result;
    int cycles_in_stage;
} Instruction;

typedef struct {
    Instruction pipeline[STAGE_COUNT];
    int cycle;
    int instructions_completed;
    int hazard_stalls;
    int branch_mispredicts;
} Pipeline;

void simulate_pipeline() {
    printf("=== Симуляция конвейера процессора ===\n");
    
    Pipeline cpu = {0};
    
    // Простая программа с зависимостями
    struct {
        int id;
        int opcode;
        int src1, src2, dst;
        char* name;
    } program[] = {
        {1, 0x100, 2, 3, 1, "ADD R1, R2, R3"},
        {2, 0x200, 1, 5, 4, "SUB R4, R1, R5"},  // RAW зависимость от ADD
        {3, 0x300, 7, 8, 6, "MUL R6, R7, R8"},
        {4, 0x400, 6, 10, 9, "DIV R9, R6, R10"}, // RAW зависимость от MUL
        {5, 0x000, 0, 0, 0, "NOP"}
    };
    int program_size = 5;
    int pc = 0;
    
    printf("\nПрограмма:\n");
    for (int i = 0; i < program_size; i++) {
        printf("%d: %s", i, program[i].name);
        if (i == 1) printf("  // Зависимость от инструкции 0");
        if (i == 3) printf("  // Зависимость от инструкции 2");
        printf("\n");
    }
    printf("\nВыполнение по циклам:\n");
    
    while (cpu.instructions_completed < program_size) {
        cpu.cycle++;
        printf("Цикл %2d: ", cpu.cycle);
        
        // Обратный порядок: WB -> MEM -> EX -> ID -> IF
        
        // Write Back
        if (cpu.pipeline[STAGE_WRITEBACK].instruction_id != 0) {
            printf("WB[%d] ", cpu.pipeline[STAGE_WRITEBACK].instruction_id);
            cpu.instructions_completed++;
            cpu.pipeline[STAGE_WRITEBACK] = (Instruction){0};
        }
        
        // Memory Access
        if (cpu.pipeline[STAGE_MEMORY].instruction_id != 0) {
            printf("MEM[%d] ", cpu.pipeline[STAGE_MEMORY].instruction_id);
            cpu.pipeline[STAGE_WRITEBACK] = cpu.pipeline[STAGE_MEMORY];
            cpu.pipeline[STAGE_MEMORY] = (Instruction){0};
        }
        
        // Execute с проверкой зависимостей
        if (cpu.pipeline[STAGE_EXECUTE].instruction_id != 0) {
            int current_id = cpu.pipeline[STAGE_EXECUTE].instruction_id;
            printf("EX[%d] ", current_id);
            
            // Проверка RAW зависимостей
            int stall = 0;
            if (current_id == 2) { // SUB зависит от ADD (инструкция 1)
                // Проверяем, завершилась ли инструкция 1
                if (cpu.instructions_completed < 1) {
                    stall = 1;
                }
            } else if (current_id == 4) { // DIV зависит от MUL (инструкция 3)
                if (cpu.instructions_completed < 3) {
                    stall = 1;
                }
            }
            
            if (stall) {
                printf("STALL(RAW) ");
                cpu.hazard_stalls++;
            } else {
                cpu.pipeline[STAGE_MEMORY] = cpu.pipeline[STAGE_EXECUTE];
                cpu.pipeline[STAGE_EXECUTE] = (Instruction){0};
            }
        }
        
        // Instruction Decode
        if (cpu.pipeline[STAGE_DECODE].instruction_id != 0) {
            printf("ID[%d] ", cpu.pipeline[STAGE_DECODE].instruction_id);
            if (cpu.pipeline[STAGE_EXECUTE].instruction_id == 0) {
                cpu.pipeline[STAGE_EXECUTE] = cpu.pipeline[STAGE_DECODE];
                cpu.pipeline[STAGE_DECODE] = (Instruction){0};
            }
        }
        
        // Instruction Fetch
        if (cpu.pipeline[STAGE_FETCH].instruction_id != 0) {
            printf("IF[%d] ", cpu.pipeline[STAGE_FETCH].instruction_id);
            if (cpu.pipeline[STAGE_DECODE].instruction_id == 0) {
                cpu.pipeline[STAGE_DECODE] = cpu.pipeline[STAGE_FETCH];
                cpu.pipeline[STAGE_FETCH] = (Instruction){0};
            }
        }
        
        // Загружаем новую инструкцию
        if (pc < program_size && cpu.pipeline[STAGE_FETCH].instruction_id == 0) {
            cpu.pipeline[STAGE_FETCH].instruction_id = pc + 1;
            cpu.pipeline[STAGE_FETCH].opcode = program[pc].opcode;
            printf("IF[%d] ", pc + 1);
            pc++;
        }
        
        printf("\n");
    }
    
    printf("\nСтатистика конвейера:\n");
    printf("Инструкций: %d\n", program_size);
    printf("Циклов: %d\n", cpu.cycle);
    printf("Stall'ов: %d\n", cpu.hazard_stalls);
    printf("CPI: %.2f\n", (double)cpu.cycle / program_size);
    printf("Эффективность: %.1f%%\n", (double)program_size / cpu.cycle * 100);
}
```

---

## 🎯 Предсказание ветвлений

### Алгоритмы предсказания

```c
// Симуляция различных предсказателей ветвлений
typedef struct {
    int predictions;
    int correct_predictions;
    char name[50];
} BranchPredictor;

// 1. Статический предсказатель
int static_predictor(int pc, int* state) {
    return 1;  // Всегда "взять"
}

// 2. 1-битный предсказатель  
int one_bit_predictor(int pc, int* state) {
    return state[pc % 256];
}

void update_one_bit(int pc, int actual, int* state) {
    state[pc % 256] = actual;
}

// 3. 2-битный насыщающий счетчик
int two_bit_predictor(int pc, int* state) {
    return state[pc % 256] >= 2;
}

void update_two_bit(int pc, int actual, int* state) {
    int* counter = &state[pc % 256];
    if (actual) {
        if (*counter < 3) (*counter)++;
    } else {
        if (*counter > 0) (*counter)--;
    }
}

void branch_prediction_demo() {
    printf("\n=== Предсказание ветвлений ===\n");
    
    // Тестовая последовательность: 3 раза взять, 1 раз не взять
    int branches[] = {1,1,1,0, 1,1,1,0, 1,1,1,0, 1,1,1,0, 1,1,1,0};
    int num_branches = sizeof(branches) / sizeof(branches[0]);
    
    BranchPredictor predictors[3] = {
        {0, 0, "Статический"},
        {0, 0, "1-битный"},  
        {0, 0, "2-битный"}
    };
    
    int one_bit_state[256] = {0};
    int two_bit_state[256];
    for (int i = 0; i < 256; i++) two_bit_state[i] = 1;
    
    printf("Паттерн ветвлений: ");
    for (int i = 0; i < num_branches; i++) printf("%d", branches[i]);
    printf("\n\n");
    
    for (int pc = 0; pc < num_branches; pc++) {
        int actual = branches[pc];
        
        // Тестируем предсказатели
        int preds[3] = {
            static_predictor(pc, NULL),
            one_bit_predictor(pc, one_bit_state),
            two_bit_predictor(pc, two_bit_state)
        };
        
        for (int i = 0; i < 3; i++) {
            predictors[i].predictions++;
            if (preds[i] == actual) predictors[i].correct_predictions++;
        }
        
        // Обновляем состояния
        update_one_bit(pc, actual, one_bit_state);
        update_two_bit(pc, actual, two_bit_state);
        
        printf("PC %2d: actual=%d, предсказания=%d/%d/%d\n",
               pc, actual, preds[0], preds[1], preds[2]);
    }
    
    printf("\nТочность предсказания:\n");
    for (int i = 0; i < 3; i++) {
        double accuracy = (double)predictors[i].correct_predictions / 
                         predictors[i].predictions * 100;
        printf("%s: %.1f%% (%d/%d)\n", 
               predictors[i].name, accuracy,
               predictors[i].correct_predictions, predictors[i].predictions);
    }
}
```

---

## 🚀 Суперскалярные процессоры

### Модель выполнения нескольких инструкций

```c
// Симуляция 2-way суперскалярного процессора
typedef struct {
    int issue_width;        // Сколько инструкций можем выдать за цикл
    int execution_units[4]; // ALU, MUL, LOAD/STORE, BRANCH
    int issued_this_cycle;
    int completed_this_cycle;
} SuperscalarCPU;

void superscalar_simulation() {
    printf("\n=== Суперскалярный процессор ===\n");
    
    SuperscalarCPU cpu = {
        .issue_width = 2,
        .execution_units = {2, 1, 1, 1}, // 2 ALU, 1 MUL, 1 LS, 1 BR
        .issued_this_cycle = 0,
        .completed_this_cycle = 0
    };
    
    // Программа с различными типами инструкций
    struct {
        char* name;
        int type; // 0=ALU, 1=MUL, 2=LOAD, 3=BRANCH
        int latency;
    } program[] = {
        {"ADD R1, R2, R3", 0, 1},
        {"ADD R4, R5, R6", 0, 1},   // Параллельно с предыдущей
        {"MUL R7, R8, R9", 1, 3},   // Длинная операция
        {"LOAD R10, [R11]", 2, 2},  // Загрузка
        {"ADD R12, R1, R4", 0, 1},  // Зависит от первых двух ADD
        {"BEQ R10, R12, LABEL", 3, 1}
    };
    int program_size = 6;
    
    printf("Программа:\n");
    for (int i = 0; i < program_size; i++) {
        printf("%d: %s (тип=%d, латентность=%d)\n", 
               i, program[i].name, program[i].type, program[i].latency);
    }
    
    printf("\nВыполнение (до %d инструкций за цикл):\n", cpu.issue_width);
    
    int cycle = 0;
    int issued = 0;
    int completed = 0;
    
    while (completed < program_size) {
        cycle++;
        printf("Цикл %d: ", cycle);
        
        int can_issue = cpu.issue_width;
        
        // Пытаемся выдать инструкции
        while (can_issue > 0 && issued < program_size) {
            int type = program[issued].type;
            
            if (cpu.execution_units[type] > 0) {
                printf("ISSUE[%d:%s] ", issued, program[issued].name);
                cpu.execution_units[type]--;
                issued++;
                can_issue--;
            } else {
                printf("STALL[%d:no_unit_%d] ", issued, type);
                break;
            }
        }
        
        // Имитируем завершение инструкций
        if (cycle >= 1 && completed < issued) {
            completed++;
            printf("COMPLETE[%d] ", completed - 1);
        }
        
        // Восстанавливаем исполнительные устройства
        cpu.execution_units[0] = 2; // ALU
        cpu.execution_units[1] = 1; // MUL
        cpu.execution_units[2] = 1; // LOAD/STORE
        cpu.execution_units[3] = 1; // BRANCH
        
        printf("\n");
    }
    
    printf("\nРезультаты:\n");
    printf("Инструкций: %d\n", program_size);
    printf("Циклов: %d\n", cycle);
    printf("IPC (Instructions Per Cycle): %.2f\n", (double)program_size / cycle);
    printf("Теоретический максимум IPC: %d\n", cpu.issue_width);
    printf("Эффективность: %.1f%%\n", 
           (double)program_size / cycle / cpu.issue_width * 100);
}
```

---

## 🔀 Out-of-Order выполнение

### Переименование регистров

```c
// Симуляция переименования регистров (Register Renaming)
typedef struct {
    int physical_reg;
    int in_use;
    int ready;
} PhysicalRegister;

typedef struct {
    int arch_to_phys[32];     // Архитектурные -> физические регистры
    PhysicalRegister phys_regs[64]; // Пул физических регистров
    int free_list[64];        // Список свободных регистров
    int free_count;
} RegisterFile;

void register_renaming_demo() {
    printf("\n=== Переименование регистров ===\n");
    
    RegisterFile rf = {0};
    
    // Инициализация
    for (int i = 0; i < 64; i++) {
        rf.phys_regs[i].physical_reg = i;
        rf.phys_regs[i].in_use = 0;
        rf.phys_regs[i].ready = 1;
        rf.free_list[i] = i;
    }
    rf.free_count = 64;
    
    // Начальное отображение архитектурных регистров
    for (int i = 0; i < 32; i++) {
        rf.arch_to_phys[i] = i;
        rf.phys_regs[i].in_use = 1;
    }
    rf.free_count = 32;
    
    // Последовательность инструкций с WAW зависимостями
    printf("Исходный код с зависимостями:\n");
    printf("R1 = R2 + R3  // Инструкция 1\n");
    printf("R1 = R4 + R5  // Инструкция 2 - WAW с инструкцией 1\n");
    printf("R6 = R1 + R7  // Инструкция 3 - RAW с инструкцией 2\n");
    
    printf("\nПереименование:\n");
    
    // Инструкция 1: R1 = R2 + R3
    int new_r1_1 = rf.free_list[--rf.free_count];
    printf("R1 = R2 + R3 -> P%d = P%d + P%d\n", 
           new_r1_1, rf.arch_to_phys[2], rf.arch_to_phys[3]);
    rf.arch_to_phys[1] = new_r1_1;
    rf.phys_regs[new_r1_1].in_use = 1;
    
    // Инструкция 2: R1 = R4 + R5  
    int new_r1_2 = rf.free_list[--rf.free_count];
    printf("R1 = R4 + R5 -> P%d = P%d + P%d  // Нет WAW!\n",
           new_r1_2, rf.arch_to_phys[4], rf.arch_to_phys[5]);
    rf.arch_to_phys[1] = new_r1_2;
    rf.phys_regs[new_r1_2].in_use = 1;
    
    // Инструкция 3: R6 = R1 + R7
    int new_r6 = rf.free_list[--rf.free_count];
    printf("R6 = R1 + R7 -> P%d = P%d + P%d  // Ссылается на новый R1\n",
           new_r6, rf.arch_to_phys[1], rf.arch_to_phys[7]);
    rf.arch_to_phys[6] = new_r6;
    rf.phys_regs[new_r6].in_use = 1;
    
    printf("\nРезультат: инструкции 1 и 2 могут выполняться параллельно!\n");
    printf("Использованных физических регистров: %d из 64\n", 64 - rf.free_count);
}
```

---

## 🔮 Спекулятивное выполнение

### Реализация спекуляции

```c
// Простая модель спекулятивного выполнения
typedef struct {
    int instruction_id;
    int is_speculative;
    int branch_target;
    int prediction;
    int actual_outcome;
    int completed;
} SpeculativeInstruction;

void speculative_execution_demo() {
    printf("\n=== Спекулятивное выполнение ===\n");
    
    printf("Код с условным переходом:\n");
    printf("1: CMP R1, R2\n");
    printf("2: BEQ LABEL     // Переход на инструкцию 6\n");
    printf("3: ADD R3, R4, R5\n");
    printf("4: MUL R6, R7, R8\n");
    printf("5: JMP END\n");
    printf("6: LABEL: SUB R3, R4, R5\n");
    printf("7: END: ...\n");
    
    SpeculativeInstruction instructions[7] = {
        {1, 0, 0, 0, 0, 0},      // CMP
        {2, 0, 6, 0, 1, 0},      // BEQ - предсказываем "не взять", фактически "взять"
        {3, 1, 0, 0, 0, 0},      // ADD - спекулятивно
        {4, 1, 0, 0, 0, 0},      // MUL - спекулятивно  
        {5, 1, 0, 0, 0, 0},      // JMP - спекулятивно
        {6, 0, 0, 0, 0, 0},      // SUB - не выполнялась
        {7, 0, 0, 0, 0, 0}       // END
    };
    
    printf("\nСценарий 1: Правильное предсказание\n");
    printf("Предсказание: не брать переход\n");
    printf("Фактический результат: не брать переход\n");
    
    for (int cycle = 1; cycle <= 5; cycle++) {
        printf("Цикл %d: ", cycle);
        
        if (cycle <= 2) {
            printf("Выполняется инструкция %d ", cycle);
        } else if (cycle <= 5) {
            printf("Спекулятивно выполняется инструкция %d ", cycle);
        }
        
        if (cycle == 2) {
            printf("(предсказание подтверждено)");
        }
        
        printf("\n");
    }
    
    printf("Результат: Все спекулятивные инструкции валидны\n");
    
    printf("\nСценарий 2: Неправильное предсказание\n");
    printf("Предсказание: не брать переход\n");
    printf("Фактический результат: взять переход\n");
    
    for (int cycle = 1; cycle <= 8; cycle++) {
        printf("Цикл %d: ", cycle);
        
        if (cycle <= 2) {
            printf("Выполняется инструкция %d ", cycle);
        } else if (cycle <= 5) {
            printf("Спекулятивно выполняется инструкция %d ", cycle);
        } else if (cycle == 6) {
            printf("MISPREDICTION! Сброс спекулятивных инструкций");
        } else if (cycle >= 7) {
            printf("Перезапуск с инструкции %d ", 6 + cycle - 7);
        }
        
        printf("\n");
    }
    
    printf("Результат: Потеря 3 циклов на неправильное предсказание\n");
    printf("Штраф за misprediction: 50%% производительности\n");
}
```

---

## 📊 Влияние на производительность

### Анализ bottleneck'ов

```c
void performance_analysis() {
    printf("\n=== Анализ узких мест микроархитектуры ===\n");
    
    struct {
        char* component;
        double impact_percent;
        char* optimization;
    } bottlenecks[] = {
        {"Branch misprediction", 15.0, "Лучшие предсказатели"},
        {"Cache misses", 25.0, "Оптимизация локальности"},
        {"Pipeline stalls", 10.0, "Out-of-order execution"},
        {"Instruction dependencies", 20.0, "Register renaming"},
        {"Limited issue width", 30.0, "Суперскалярность"}
    };
    
    printf("Типичные потери производительности:\n");
    for (int i = 0; i < 5; i++) {
        printf("%-25s: %5.1f%% | %s\n", 
               bottlenecks[i].component, 
               bottlenecks[i].impact_percent,
               bottlenecks[i].optimization);
    }
    
    printf("\nРекомендации по оптимизации:\n");
    printf("1. Минимизируйте ветвления в критических циклах\n");
    printf("2. Максимизируйте instruction-level parallelism\n");
    printf("3. Используйте profile-guided optimization\n");
    printf("4. Оптимизируйте под конкретную микроархитектуру\n");
}

int main() {
    simulate_pipeline();
    branch_prediction_demo();
    superscalar_simulation();
    register_renaming_demo();
    speculative_execution_demo();
    performance_analysis();
    return 0;
}
```

## 🔗 Связанные темы

- [[processor-architectures|Архитектуры процессоров]] - ISA и их реализация
- [[memory-hierarchy|Иерархия памяти]] - взаимодействие с кэшем
- [[parallel-architectures|Параллельные архитектуры]] - многоядерные расширения
- [[performance-analysis|Анализ производительности]] - профилирование микроархитектуры

## 📚 Дополнительные ресурсы

### 📖 Книги
- "Modern Processor Design" - Shen & Lipasti
- "Superscalar Microprocessor Design" - Johnson
- "Computer Architecture: A Quantitative Approach" - Hennessy & Patterson

### 🛠️ Инструменты
- **Intel VTune** - микроархитектурное профилирование
- **uops.info** - характеристики инструкций
- **LLVM-MCA** - анализ производительности кода
- **gem5** - детальная симуляция микроархитектуры

---

> **Следующий шаг**: Изучите [[parallel-architectures|параллельные архитектуры]] для понимания того, как микроархитектура масштабируется на многоядерные системы. 