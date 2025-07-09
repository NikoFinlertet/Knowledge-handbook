# Ассемблер и машинный код 🔧

> **Навигация**: [[README|← Системное программирование]] | [[computer-science/README|Компьютерные науки]] | [[README|Главная]]

## 🎯 Цель изучения

Освоение программирования на ассемблере, понимание архитектуры процессора, работы с машинным кодом и низкоуровневой отладки.

## 📚 Основы архитектуры x86-64

### 🏗️ Регистры процессора
```assembly
; Основные 64-битные регистры общего назначения
; RAX - аккумулятор (возвращаемое значение функций)
; RBX - базовый регистр
; RCX - счетчик циклов
; RDX - данные (умножение/деление)
; RSI - source index (строковые операции)
; RDI - destination index (строковые операции) 
; RBP - base pointer (указатель на фрейм стека)
; RSP - stack pointer (указатель на вершину стека)
; R8-R15 - дополнительные регистры общего назначения

; Сегментные регистры
; CS - code segment
; DS - data segment  
; ES - extra segment
; FS, GS - дополнительные сегменты
; SS - stack segment

; Флаговые регистры (RFLAGS)
; CF - carry flag (перенос)
; ZF - zero flag (нуль)
; SF - sign flag (знак)
; OF - overflow flag (переполнение)
; PF - parity flag (четность)
```

### 💾 Представление данных
```assembly
section .data
    ; Байт (8 бит)
    byte_val    db 0x42        ; 1 байт
    
    ; Слово (16 бит)  
    word_val    dw 0x1234      ; 2 байта
    
    ; Двойное слово (32 бита)
    dword_val   dd 0x12345678  ; 4 байта
    
    ; Четверное слово (64 бита)
    qword_val   dq 0x123456789ABCDEF0  ; 8 байт
    
    ; Строки
    message     db 'Hello, Assembly!', 0
    buffer      times 100 db 0  ; Буфер из 100 нулевых байт
    
    ; Массивы
    numbers     dd 1, 2, 3, 4, 5
    array_size  equ ($ - numbers) / 4  ; Размер массива

section .bss
    ; Неинициализированные данные
    temp_buffer resb 256       ; 256 байт
    user_input  resq 1         ; 8 байт (1 qword)
```

### 🔄 Режимы адресации
```assembly
; Немедленная адресация
mov rax, 42                    ; rax = 42

; Регистровая адресация
mov rax, rbx                   ; rax = rbx

; Прямая адресация
mov rax, [memory_location]     ; rax = значение по адресу

; Косвенная регистровая
mov rax, [rbx]                 ; rax = значение по адресу в rbx

; Индексная адресация
mov rax, [rbx + 8]             ; rax = значение по адресу (rbx + 8)

; Масштабируемая индексная
mov rax, [rbx + rcx*4]         ; rax = значение по адресу (rbx + rcx*4)
mov rax, [rbx + rcx*4 + 8]     ; полная форма с смещением

; Относительная адресация (RIP-relative)
mov rax, [rel message]         ; rax = значение по адресу message
```

## 🛠️ Основные инструкции

### 📊 Арифметические операции
```assembly
section .text
global _start

_start:
    ; Сложение
    mov rax, 10
    add rax, 20                ; rax = 30
    
    ; Вычитание
    mov rbx, 50
    sub rbx, 15                ; rbx = 35
    
    ; Умножение
    mov rax, 6
    mov rbx, 7
    imul rax, rbx              ; rax = 42 (знаковое умножение)
    
    ; Деление
    mov rax, 84
    mov rbx, 2
    xor rdx, rdx               ; Очистка rdx для деления
    idiv rbx                   ; rax = 42, rdx = остаток
    
    ; Инкремент/декремент
    inc rax                    ; rax++
    dec rax                    ; rax--
    
    ; Сравнение
    cmp rax, 42
    je equal                   ; Переход если равно
    
equal:
    ; Логические операции
    mov rax, 0xFF00
    mov rbx, 0x00FF
    and rax, rbx               ; rax = 0x0000 (И)
    or  rax, 0xF0F0            ; rax = 0xF0F0 (ИЛИ)
    xor rax, rax               ; rax = 0 (исключающее ИЛИ)
    not rax                    ; rax = 0xFFFFFFFFFFFFFFFF (НЕ)
```

### 🔄 Управление потоком
```assembly
; Безусловные переходы
jmp label                      ; Безусловный переход

; Условные переходы (после cmp)
je  label                      ; Jump if Equal (ZF=1)
jne label                      ; Jump if Not Equal (ZF=0)
jl  label                      ; Jump if Less (SF≠OF)
jle label                      ; Jump if Less or Equal
jg  label                      ; Jump if Greater
jge label                      ; Jump if Greater or Equal

; Беззнаковые сравнения
jb  label                      ; Jump if Below (CF=1)
jbe label                      ; Jump if Below or Equal
ja  label                      ; Jump if Above
jae label                      ; Jump if Above or Equal

; Циклы
loop_start:
    ; Тело цикла
    dec rcx                    ; Уменьшить счетчик
    jnz loop_start             ; Повторить если не ноль

; Цикл с инструкцией loop
mov rcx, 10                    ; Счетчик цикла
loop_with_instruction:
    ; Тело цикла
    loop loop_with_instruction ; rcx--, если rcx!=0 то переход
```

### 📚 Работа со стеком
```assembly
; Сохранение и восстановление регистров
push rax                       ; Сохранить rax в стек
push rbx                       ; Сохранить rbx в стек
; ... некоторые операции ...
pop rbx                        ; Восстановить rbx из стека
pop rax                        ; Восстановить rax из стека

; Работа с кадром стека
push rbp                       ; Сохранить старый base pointer
mov rbp, rsp                   ; Установить новый base pointer

; Выделение места для локальных переменных
sub rsp, 32                    ; Выделить 32 байта

; Доступ к локальным переменным
mov dword [rbp-4], 42          ; Локальная переменная 1
mov dword [rbp-8], 84          ; Локальная переменная 2

; Очистка стека и возврат
mov rsp, rbp                   ; Восстановить указатель стека
pop rbp                        ; Восстановить base pointer
ret                            ; Возврат из функции
```

## 🔧 Системные вызовы

### 📞 Linux System Calls
```assembly
section .data
    message db 'Hello, World!', 10, 0
    msg_len equ $ - message - 1

section .text
global _start

_start:
    ; write(1, message, msg_len)
    mov rax, 1                 ; sys_write
    mov rdi, 1                 ; stdout
    mov rsi, message           ; буфер
    mov rdx, msg_len           ; количество байт
    syscall                    ; вызов системы
    
    ; exit(0)
    mov rax, 60                ; sys_exit
    mov rdi, 0                 ; код возврата
    syscall

; Чтение из файла
read_file:
    ; open("filename", O_RDONLY)
    mov rax, 2                 ; sys_open
    mov rdi, filename          ; имя файла
    mov rsi, 0                 ; O_RDONLY
    mov rdx, 0644              ; права доступа
    syscall
    
    mov r8, rax                ; сохранить file descriptor
    
    ; read(fd, buffer, size)
    mov rax, 0                 ; sys_read
    mov rdi, r8                ; file descriptor
    mov rsi, buffer            ; буфер для чтения
    mov rdx, 1024              ; размер буфера
    syscall
    
    ; close(fd)
    mov rax, 3                 ; sys_close
    mov rdi, r8                ; file descriptor
    syscall
    
    ret
```

### 🧵 Создание потоков
```assembly
; fork() system call
create_process:
    mov rax, 57                ; sys_fork
    syscall
    
    cmp rax, 0
    je child_process           ; В дочернем процессе rax = 0
    jl fork_error              ; Ошибка если rax < 0
    
    ; Родительский процесс (rax = PID дочернего)
    mov r9, rax                ; Сохранить PID дочернего
    jmp parent_code
    
child_process:
    ; Код дочернего процесса
    call child_function
    
    ; exit дочернего процесса
    mov rax, 60                ; sys_exit
    mov rdi, 0
    syscall
    
fork_error:
    ; Обработка ошибки
    mov rax, 60
    mov rdi, 1                 ; Код ошибки
    syscall
```

## 🔍 Отладка и анализ

### 🐛 GDB отладка ассемблера
```bash
# Компиляция с отладочной информацией
nasm -f elf64 -g program.asm -o program.o
ld program.o -o program

# Запуск в отладчике
gdb ./program

# Основные команды GDB
(gdb) break _start          # Установить breakpoint
(gdb) run                   # Запустить программу
(gdb) stepi                 # Выполнить одну инструкцию
(gdb) info registers        # Показать регистры
(gdb) x/10i $rip           # Показать 10 инструкций от RIP
(gdb) x/4xg $rsp           # Показать 4 qword из стека
(gdb) disassemble          # Дизассемблировать текущую функцию
```

### 📊 Анализ производительности
```assembly
; Измерение времени выполнения
section .data
    start_time    dq 0
    end_time      dq 0
    
section .text

; Получение текущего времени (RDTSC)
get_timestamp:
    rdtsc                      ; Результат в EDX:EAX
    shl rdx, 32               ; Сдвинуть старшие биты
    or  rax, rdx              ; Объединить в RAX
    ret

benchmark_function:
    ; Получить время начала
    call get_timestamp
    mov [start_time], rax
    
    ; Выполнить тестируемый код
    call tested_function
    
    ; Получить время окончания
    call get_timestamp
    mov [end_time], rax
    
    ; Вычислить разность
    mov rax, [end_time]
    sub rax, [start_time]
    ; RAX содержит количество тактов процессора
    
    ret

; Оптимизированная функция копирования памяти
optimized_memcpy:
    ; Входные параметры: RDI = dest, RSI = src, RDX = size
    mov rcx, rdx
    shr rcx, 3                 ; Количество 8-байтных блоков
    rep movsq                  ; Копировать по 8 байт
    
    mov rcx, rdx
    and rcx, 7                 ; Оставшиеся байты
    rep movsb                  ; Копировать по байту
    
    ret
```

## 🔧 Встроенный ассемблер

### 📝 Inline Assembly в C
```c
#include <stdio.h>
#include <stdint.h>

// Простой inline assembly
int add_numbers(int a, int b) {
    int result;
    asm ("addl %1, %2; movl %2, %0"
         : "=r" (result)           // output
         : "r" (a), "r" (b)        // input  
         :                         // clobbered registers
    );
    return result;
}

// Работа с флагами процессора
int check_cpu_features() {
    uint32_t eax, ebx, ecx, edx;
    
    // CPUID instruction
    asm volatile ("cpuid"
                  : "=a" (eax), "=b" (ebx), "=c" (ecx), "=d" (edx)
                  : "a" (1)  // EAX = 1 для получения feature flags
    );
    
    // Проверка SSE2 поддержки
    if (edx & (1 << 26)) {
        printf("SSE2 supported\n");
        return 1;
    }
    return 0;
}

// Атомарные операции
int atomic_increment(volatile int* value) {
    int result;
    asm volatile ("lock incl %1; movl %1, %0"
                  : "=r" (result), "+m" (*value)
                  :
                  : "memory"
    );
    return result;
}

// Быстрая функция вычисления sqrt
double fast_sqrt(double x) {
    double result;
    asm ("sqrtsd %1, %0"
         : "=x" (result)
         : "x" (x)
    );
    return result;
}

// Получение указателя стека
void* get_stack_pointer() {
    void* sp;
    asm ("movq %%rsp, %0" : "=r" (sp));
    return sp;
}
```

### ⚡ SIMD инструкции
```assembly
section .data
    vector1     dd 1.0, 2.0, 3.0, 4.0     ; 4 float числа
    vector2     dd 5.0, 6.0, 7.0, 8.0     ; 4 float числа
    result      dd 0.0, 0.0, 0.0, 0.0     ; результат

section .text

; Векторное сложение с SSE
vector_add_sse:
    movups xmm0, [vector1]     ; Загрузить 4 float в XMM0
    movups xmm1, [vector2]     ; Загрузить 4 float в XMM1
    addps  xmm0, xmm1          ; Параллельное сложение
    movups [result], xmm0      ; Сохранить результат
    ret

; Векторное умножение с AVX
vector_mul_avx:
    vmovups ymm0, [vector1]    ; Загрузить 8 float в YMM0
    vmovups ymm1, [vector2]    ; Загрузить 8 float в YMM1
    vmulps  ymm0, ymm0, ymm1   ; Параллельное умножение
    vmovups [result], ymm0     ; Сохранить результат
    ret

; Скалярное произведение векторов
dot_product:
    movups xmm0, [vector1]
    movups xmm1, [vector2]
    mulps  xmm0, xmm1          ; Поэлементное умножение
    
    ; Горизонтальное сложение
    haddps xmm0, xmm0          ; x0+x1, x2+x3, x0+x1, x2+x3
    haddps xmm0, xmm0          ; sum, sum, sum, sum
    
    movss [result], xmm0       ; Сохранить результат
    ret
```

## 🛡️ Безопасность и защита

### 🔒 Stack Canaries и Buffer Overflow
```assembly
section .data
    canary_value dq 0xDEADBEEFCAFEBABE

section .text

; Функция с защитой от переполнения буфера
secure_function:
    push rbp
    mov rbp, rsp
    
    ; Установка canary на стек
    mov rax, [canary_value]
    push rax
    
    ; Выделение локального буфера
    sub rsp, 256
    
    ; ... код функции ...
    
    ; Проверка canary перед возвратом
    add rsp, 256
    pop rax
    cmp rax, [canary_value]
    jne stack_overflow_detected
    
    mov rsp, rbp
    pop rbp
    ret

stack_overflow_detected:
    ; Обработка переполнения буфера
    mov rax, 60                ; sys_exit
    mov rdi, 1                 ; код ошибки
    syscall

; ASLR bypass example (только для образовательных целей)
get_libc_base:
    ; Чтение /proc/self/maps для получения адресов
    mov rax, 2                 ; sys_open
    lea rdi, [maps_file]
    mov rsi, 0                 ; O_RDONLY
    syscall
    
    ; ... парсинг выхода для поиска libc ...
    ret

maps_file: db '/proc/self/maps', 0
```

### 🛡️ Control Flow Integrity
```assembly
; Пример использования Intel CET (Control-flow Enforcement Technology)
section .text

; Функция с защитой возвратного адреса
protected_function:
    endbr64                    ; End Branch 64-bit (Intel CET)
    push rbp
    mov rbp, rsp
    
    ; ... код функции ...
    
    mov rsp, rbp
    pop rbp
    ret                        ; Защищенный возврат

; Indirect call с проверкой
safe_indirect_call:
    ; Проверка целевого адреса
    test rax, 0xF              ; Проверка выравнивания
    jnz invalid_target
    
    cmp rax, code_start        ; Проверка диапазона
    jb invalid_target
    cmp rax, code_end
    ja invalid_target
    
    call rax                   ; Безопасный вызов
    ret

invalid_target:
    ; Обработка некорректного адреса
    int 3                      ; Breakpoint для отладки
    ret
```

## 🎯 Практические проекты

### 🎯 Проект 1: Простая операционная система (6-8 недель)
```assembly
; Bootloader
section .text
org 0x7c00

boot_start:
    ; Установка режима VGA
    mov ax, 0x03
    int 0x10
    
    ; Загрузка kernel
    mov ah, 0x02               ; Функция чтения секторов
    mov al, 5                  ; Количество секторов
    mov ch, 0                  ; Цилиндр
    mov cl, 2                  ; Сектор
    mov dh, 0                  ; Головка
    mov bx, 0x1000            ; Адрес загрузки
    int 0x13                  ; BIOS прерывание
    
    jmp 0x1000                ; Переход к kernel
    
    times 510-($-$$) db 0     ; Заполнение нулями
    dw 0xAA55                 ; Boot signature
```

### 🎯 Проект 2: Криптографическая библиотека (4-5 недель)
```assembly
; Оптимизированный AES с AES-NI инструкциями
aes_encrypt_block:
    ; Загрузка данных
    movdqu xmm0, [rdi]        ; Plaintext
    movdqu xmm1, [rsi]        ; First round key
    
    ; Initial round
    pxor xmm0, xmm1
    
    ; 9 rounds для AES-128
    mov rcx, 9
encrypt_loop:
    add rsi, 16
    movdqu xmm1, [rsi]
    aesenc xmm0, xmm1
    loop encrypt_loop
    
    ; Final round
    add rsi, 16
    movdqu xmm1, [rsi]
    aesenclast xmm0, xmm1
    
    ; Сохранение результата
    movdqu [rdx], xmm0
    ret
```

### 🎯 Проект 3: JIT компилятор (5-6 недель)
```c
// Генерация машинного кода в runtime
typedef struct {
    uint8_t* code_buffer;
    size_t code_size;
    size_t code_capacity;
} jit_compiler_t;

// Эмиссия инструкций
void emit_mov_reg_imm(jit_compiler_t* jit, int reg, int64_t value) {
    // MOV reg, imm64 (REX.W + B8 + rd, imm64)
    uint8_t rex = 0x48 | ((reg & 8) >> 3);  // REX.W + REX.B
    uint8_t opcode = 0xB8 + (reg & 7);
    
    emit_byte(jit, rex);
    emit_byte(jit, opcode);
    emit_qword(jit, value);
}

void emit_add_reg_reg(jit_compiler_t* jit, int dst, int src) {
    // ADD dst, src (REX.W + 01 /r)
    uint8_t rex = 0x48 | ((src & 8) >> 1) | ((dst & 8) >> 3);
    uint8_t modrm = 0xC0 | ((src & 7) << 3) | (dst & 7);
    
    emit_byte(jit, rex);
    emit_byte(jit, 0x01);
    emit_byte(jit, modrm);
}

// Выполнение сгенерированного кода
int64_t execute_jit_code(jit_compiler_t* jit) {
    // Установка прав выполнения
    mprotect(jit->code_buffer, jit->code_size, 
             PROT_READ | PROT_EXEC);
    
    // Вызов как функции
    int64_t (*func)() = (int64_t(*)())jit->code_buffer;
    return func();
}
```

## 📚 Дополнительные ресурсы

### 📖 Рекомендуемая литература
- **"Programming from the Ground Up" - Jonathan Bartlett** `🚀 Начинающим`
- **"PC Assembly Language" - Paul Carter** `💻 x86 специфика`
- **"The Art of Assembly Language" - Randall Hyde** `🎨 Углубленное изучение`

### 🔗 Связанные темы
- [[compilers-translators|Компиляторы и трансляторы]] - для генерации ассемблера
- [[memory-management|Управление памятью]] - для низкоуровневой работы с памятью
- [[../computer-architecture/README|Компьютерная архитектура]] - архитектура процессоров
- [[../technical-skills/security|Безопасность]] - защита на уровне железа
- [[../technical-skills/operating-systems|Операционные системы]] - системное программирование

### 🏗️ Roadmap развития
1. **Неделя 1**: Основы x86-64, регистры, простые инструкции
2. **Неделя 2**: Системные вызовы, работа с файлами
3. **Неделя 3**: Продвинутые инструкции, SIMD
4. **Неделя 4**: Отладка, анализ производительности
5. **Неделя 5**: Встроенный ассемблер, интеграция с C
6. **Неделя 6**: Безопасность, защитные механизмы

---

> 💡 **Совет**: Изучайте ассемблер параллельно с изучением архитектуры процессора. Понимание "железа" критично для эффективного программирования! 