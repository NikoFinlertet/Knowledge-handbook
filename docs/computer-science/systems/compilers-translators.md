# Компиляторы и трансляторы 🔧

> **Навигация**: [[README|← Системное программирование]] | [[computer-science/README|Компьютерные науки]] | [[README|Главная]]

## 🎯 Цель изучения

Изучение принципов работы компиляторов, интерпретаторов и других языковых процессоров, их архитектуры и реализации.

## 📚 Основы теории компиляции

### 🔄 Фазы компиляции
```c
// Структура компилятора
typedef struct {
    char* source_code;
    token_list_t* tokens;
    ast_node_t* syntax_tree;
    symbol_table_t* symbols;
    ir_code_t* intermediate_code;
    char* target_code;
} compiler_state_t;

// Основные фазы:
// 1. Лексический анализ (Scanner/Lexer)
// 2. Синтаксический анализ (Parser) 
// 3. Семантический анализ
// 4. Генерация промежуточного кода
// 5. Оптимизация кода
// 6. Генерация целевого кода
```

### 📝 Лексический анализ
```c
// Токены
typedef enum {
    TOKEN_NUMBER,
    TOKEN_IDENTIFIER,
    TOKEN_PLUS,
    TOKEN_MINUS,
    TOKEN_MULTIPLY,
    TOKEN_DIVIDE,
    TOKEN_ASSIGN,
    TOKEN_SEMICOLON,
    TOKEN_LBRACE,
    TOKEN_RBRACE,
    TOKEN_IF,
    TOKEN_WHILE,
    TOKEN_EOF
} token_type_t;

typedef struct {
    token_type_t type;
    char* value;
    int line;
    int column;
} token_t;

// Лексический анализатор
typedef struct {
    char* input;
    int position;
    int line;
    int column;
    char current_char;
} lexer_t;

void lexer_init(lexer_t* lexer, char* input) {
    lexer->input = input;
    lexer->position = 0;
    lexer->line = 1;
    lexer->column = 1;
    lexer->current_char = input[0];
}

token_t* get_next_token(lexer_t* lexer) {
    while (lexer->current_char != '\0') {
        if (isspace(lexer->current_char)) {
            skip_whitespace(lexer);
            continue;
        }
        
        if (isdigit(lexer->current_char)) {
            return read_number(lexer);
        }
        
        if (isalpha(lexer->current_char)) {
            return read_identifier(lexer);
        }
        
        switch (lexer->current_char) {
            case '+': advance(lexer); return create_token(TOKEN_PLUS, "+");
            case '-': advance(lexer); return create_token(TOKEN_MINUS, "-");
            case '*': advance(lexer); return create_token(TOKEN_MULTIPLY, "*");
            case '/': advance(lexer); return create_token(TOKEN_DIVIDE, "/");
            case '=': advance(lexer); return create_token(TOKEN_ASSIGN, "=");
            case ';': advance(lexer); return create_token(TOKEN_SEMICOLON, ";");
            case '{': advance(lexer); return create_token(TOKEN_LBRACE, "{");
            case '}': advance(lexer); return create_token(TOKEN_RBRACE, "}");
        }
    }
    
    return create_token(TOKEN_EOF, "EOF");
}
```

### 🌳 Синтаксический анализ
```c
// AST узлы
typedef enum {
    AST_NUMBER,
    AST_IDENTIFIER,
    AST_BINARY_OP,
    AST_ASSIGN,
    AST_BLOCK,
    AST_IF,
    AST_WHILE
} ast_node_type_t;

typedef struct ast_node {
    ast_node_type_t type;
    union {
        int number_value;
        char* identifier;
        struct {
            token_type_t operator;
            struct ast_node* left;
            struct ast_node* right;
        } binary_op;
        struct {
            char* variable;
            struct ast_node* expression;
        } assignment;
        struct {
            struct ast_node* condition;
            struct ast_node* then_stmt;
            struct ast_node* else_stmt;
        } if_stmt;
    };
} ast_node_t;

// Рекурсивный спуск парсер
typedef struct {
    lexer_t* lexer;
    token_t* current_token;
} parser_t;

ast_node_t* parse_expression(parser_t* parser) {
    ast_node_t* node = parse_term(parser);
    
    while (parser->current_token->type == TOKEN_PLUS || 
           parser->current_token->type == TOKEN_MINUS) {
        
        token_type_t op = parser->current_token->type;
        eat(parser, op);
        
        ast_node_t* binary_node = malloc(sizeof(ast_node_t));
        binary_node->type = AST_BINARY_OP;
        binary_node->binary_op.operator = op;
        binary_node->binary_op.left = node;
        binary_node->binary_op.right = parse_term(parser);
        
        node = binary_node;
    }
    
    return node;
}

ast_node_t* parse_statement(parser_t* parser) {
    if (parser->current_token->type == TOKEN_IF) {
        return parse_if_statement(parser);
    } else if (parser->current_token->type == TOKEN_WHILE) {
        return parse_while_statement(parser);
    } else if (parser->current_token->type == TOKEN_IDENTIFIER) {
        return parse_assignment(parser);
    }
    
    error("Unexpected token in statement");
    return NULL;
}
```

### 🔍 Семантический анализ
```c
// Таблица символов
typedef struct symbol {
    char* name;
    data_type_t type;
    int scope_level;
    bool is_initialized;
    struct symbol* next;
} symbol_t;

typedef struct {
    symbol_t* symbols[HASH_SIZE];
    int current_scope;
} symbol_table_t;

// Проверка типов
data_type_t check_binary_op(ast_node_t* node, symbol_table_t* table) {
    data_type_t left_type = analyze_expression(node->binary_op.left, table);
    data_type_t right_type = analyze_expression(node->binary_op.right, table);
    
    if (left_type != right_type) {
        semantic_error("Type mismatch in binary operation");
    }
    
    return left_type;
}

bool analyze_assignment(ast_node_t* node, symbol_table_t* table) {
    symbol_t* symbol = lookup_symbol(table, node->assignment.variable);
    if (symbol == NULL) {
        semantic_error("Undefined variable: %s", node->assignment.variable);
        return false;
    }
    
    data_type_t expr_type = analyze_expression(node->assignment.expression, table);
    if (symbol->type != expr_type) {
        semantic_error("Type mismatch in assignment");
        return false;
    }
    
    symbol->is_initialized = true;
    return true;
}
```

## ⚙️ Генерация кода

### 📊 Промежуточное представление
```c
// Three-address code (TAC)
typedef enum {
    OP_ADD, OP_SUB, OP_MUL, OP_DIV,
    OP_ASSIGN, OP_GOTO, OP_IF_GOTO,
    OP_LABEL, OP_PARAM, OP_CALL, OP_RETURN
} opcode_t;

typedef struct {
    opcode_t op;
    char* arg1;
    char* arg2;
    char* result;
} tac_instruction_t;

// Генератор промежуточного кода
typedef struct {
    tac_instruction_t* instructions;
    int instruction_count;
    int temp_counter;
    int label_counter;
} ir_generator_t;

char* new_temp(ir_generator_t* gen) {
    char* temp = malloc(16);
    snprintf(temp, 16, "t%d", gen->temp_counter++);
    return temp;
}

char* new_label(ir_generator_t* gen) {
    char* label = malloc(16);
    snprintf(label, 16, "L%d", gen->label_counter++);
    return label;
}

char* generate_expression(ast_node_t* node, ir_generator_t* gen) {
    switch (node->type) {
        case AST_NUMBER: {
            char* result = malloc(16);
            snprintf(result, 16, "%d", node->number_value);
            return result;
        }
        
        case AST_IDENTIFIER:
            return strdup(node->identifier);
            
        case AST_BINARY_OP: {
            char* left = generate_expression(node->binary_op.left, gen);
            char* right = generate_expression(node->binary_op.right, gen);
            char* result = new_temp(gen);
            
            emit_instruction(gen, token_to_opcode(node->binary_op.operator),
                           left, right, result);
            return result;
        }
    }
    
    return NULL;
}
```

### 🎯 Целевой код (x86-64)
```c
// Генерация ассемблера
typedef struct {
    FILE* output;
    int stack_offset;
    hash_table_t* register_allocation;
} codegen_t;

void generate_x86_prologue(codegen_t* gen) {
    fprintf(gen->output, "    push %%rbp\n");
    fprintf(gen->output, "    mov %%rsp, %%rbp\n");
}

void generate_x86_epilogue(codegen_t* gen) {
    fprintf(gen->output, "    mov %%rbp, %%rsp\n");
    fprintf(gen->output, "    pop %%rbp\n");
    fprintf(gen->output, "    ret\n");
}

void generate_binary_op(codegen_t* gen, tac_instruction_t* instr) {
    // Загрузка операндов в регистры
    fprintf(gen->output, "    mov %s, %%rax\n", instr->arg1);
    fprintf(gen->output, "    mov %s, %%rbx\n", instr->arg2);
    
    switch (instr->op) {
        case OP_ADD:
            fprintf(gen->output, "    add %%rbx, %%rax\n");
            break;
        case OP_SUB:
            fprintf(gen->output, "    sub %%rbx, %%rax\n");
            break;
        case OP_MUL:
            fprintf(gen->output, "    imul %%rbx, %%rax\n");
            break;
        case OP_DIV:
            fprintf(gen->output, "    cqo\n");
            fprintf(gen->output, "    idiv %%rbx\n");
            break;
    }
    
    // Сохранение результата
    fprintf(gen->output, "    mov %%rax, %s\n", instr->result);
}
```

## 🔧 Оптимизация кода

### ⚡ Локальные оптимизации
```c
// Constant folding
ast_node_t* optimize_constant_folding(ast_node_t* node) {
    if (node->type == AST_BINARY_OP) {
        node->binary_op.left = optimize_constant_folding(node->binary_op.left);
        node->binary_op.right = optimize_constant_folding(node->binary_op.right);
        
        if (node->binary_op.left->type == AST_NUMBER && 
            node->binary_op.right->type == AST_NUMBER) {
            
            int left_val = node->binary_op.left->number_value;
            int right_val = node->binary_op.right->number_value;
            int result;
            
            switch (node->binary_op.operator) {
                case TOKEN_PLUS: result = left_val + right_val; break;
                case TOKEN_MINUS: result = left_val - right_val; break;
                case TOKEN_MULTIPLY: result = left_val * right_val; break;
                case TOKEN_DIVIDE: 
                    if (right_val != 0) result = left_val / right_val;
                    else return node;
                    break;
                default: return node;
            }
            
            // Замена на константу
            ast_node_t* const_node = malloc(sizeof(ast_node_t));
            const_node->type = AST_NUMBER;
            const_node->number_value = result;
            
            free(node->binary_op.left);
            free(node->binary_op.right);
            free(node);
            
            return const_node;
        }
    }
    
    return node;
}

// Dead code elimination
void eliminate_dead_code(tac_instruction_t* instructions, int count) {
    bool* used = calloc(count, sizeof(bool));
    
    // Отмечаем используемые инструкции
    for (int i = 0; i < count; i++) {
        if (is_side_effect_instruction(&instructions[i])) {
            used[i] = true;
            mark_dependencies(&instructions[i], instructions, count, used);
        }
    }
    
    // Удаляем неиспользуемые инструкции
    int write_pos = 0;
    for (int i = 0; i < count; i++) {
        if (used[i]) {
            if (write_pos != i) {
                instructions[write_pos] = instructions[i];
            }
            write_pos++;
        }
    }
    
    free(used);
}
```

### 🌐 Глобальные оптимизации
```c
// Анализ потока данных
typedef struct {
    bitset_t* gen;   // Генерируемые определения
    bitset_t* kill;  // Уничтожаемые определения  
    bitset_t* in;    // Входящие определения
    bitset_t* out;   // Исходящие определения
} dataflow_info_t;

// Reaching definitions analysis
void compute_reaching_definitions(basic_block_t* blocks, int block_count) {
    bool changed = true;
    
    while (changed) {
        changed = false;
        
        for (int i = 0; i < block_count; i++) {
            basic_block_t* block = &blocks[i];
            
            // IN[B] = ∪ OUT[P] для всех предшественников P блока B
            bitset_clear(block->dataflow.in);
            for (int j = 0; j < block->predecessor_count; j++) {
                basic_block_t* pred = block->predecessors[j];
                bitset_union(block->dataflow.in, pred->dataflow.out);
            }
            
            // OUT[B] = GEN[B] ∪ (IN[B] - KILL[B])
            bitset_t* old_out = bitset_copy(block->dataflow.out);
            bitset_copy_from(block->dataflow.out, block->dataflow.in);
            bitset_subtract(block->dataflow.out, block->dataflow.kill);
            bitset_union(block->dataflow.out, block->dataflow.gen);
            
            if (!bitset_equal(block->dataflow.out, old_out)) {
                changed = true;
            }
            
            bitset_free(old_out);
        }
    }
}

// Register allocation (graph coloring)
typedef struct {
    char* variable;
    bitset_t* conflicts;  // Конфликтующие переменные
    int color;           // Назначенный регистр
} interference_node_t;

void allocate_registers(interference_node_t* nodes, int node_count, int num_registers) {
    // Упрощение графа (удаление узлов со степенью < num_registers)
    stack_t* simplify_stack = stack_create();
    bool* removed = calloc(node_count, sizeof(bool));
    
    while (true) {
        bool found = false;
        
        for (int i = 0; i < node_count; i++) {
            if (!removed[i] && count_active_conflicts(&nodes[i], removed) < num_registers) {
                stack_push(simplify_stack, &nodes[i]);
                removed[i] = true;
                found = true;
                break;
            }
        }
        
        if (!found) break;
    }
    
    // Раскраска (назначение регистров)
    while (!stack_empty(simplify_stack)) {
        interference_node_t* node = stack_pop(simplify_stack);
        
        bool used_colors[num_registers] = {false};
        
        // Отмечаем цвета конфликтующих узлов
        for (int i = 0; i < node_count; i++) {
            if (bitset_test(node->conflicts, i) && nodes[i].color >= 0) {
                used_colors[nodes[i].color] = true;
            }
        }
        
        // Назначаем первый доступный цвет
        node->color = -1;
        for (int c = 0; c < num_registers; c++) {
            if (!used_colors[c]) {
                node->color = c;
                break;
            }
        }
        
        // Если нет доступных регистров, переменная остается в памяти
        if (node->color == -1) {
            printf("Spilling variable: %s\n", node->variable);
        }
    }
    
    free(removed);
    stack_free(simplify_stack);
}
```

## 🏗️ LLVM интеграция

### 🔨 LLVM IR генерация
```c
#include <llvm-c/Core.h>
#include <llvm-c/ExecutionEngine.h>

// LLVM контекст
typedef struct {
    LLVMContextRef context;
    LLVMModuleRef module;
    LLVMBuilderRef builder;
    LLVMValueRef function;
    hash_table_t* variables;
} llvm_codegen_t;

llvm_codegen_t* llvm_init(const char* module_name) {
    llvm_codegen_t* gen = malloc(sizeof(llvm_codegen_t));
    
    gen->context = LLVMContextCreate();
    gen->module = LLVMModuleCreateWithNameInContext(module_name, gen->context);
    gen->builder = LLVMCreateBuilderInContext(gen->context);
    gen->variables = hash_table_create();
    
    return gen;
}

LLVMValueRef generate_llvm_expression(ast_node_t* node, llvm_codegen_t* gen) {
    switch (node->type) {
        case AST_NUMBER:
            return LLVMConstInt(LLVMInt32TypeInContext(gen->context), 
                               node->number_value, 0);
                               
        case AST_IDENTIFIER: {
            LLVMValueRef* var = hash_table_get(gen->variables, node->identifier);
            if (var == NULL) {
                fprintf(stderr, "Undefined variable: %s\n", node->identifier);
                return NULL;
            }
            return LLVMBuildLoad(gen->builder, *var, node->identifier);
        }
        
        case AST_BINARY_OP: {
            LLVMValueRef left = generate_llvm_expression(node->binary_op.left, gen);
            LLVMValueRef right = generate_llvm_expression(node->binary_op.right, gen);
            
            switch (node->binary_op.operator) {
                case TOKEN_PLUS:
                    return LLVMBuildAdd(gen->builder, left, right, "addtmp");
                case TOKEN_MINUS:
                    return LLVMBuildSub(gen->builder, left, right, "subtmp");
                case TOKEN_MULTIPLY:
                    return LLVMBuildMul(gen->builder, left, right, "multmp");
                case TOKEN_DIVIDE:
                    return LLVMBuildSDiv(gen->builder, left, right, "divtmp");
            }
        }
    }
    
    return NULL;
}

void generate_llvm_function(ast_node_t* function_ast, llvm_codegen_t* gen) {
    // Создание типа функции
    LLVMTypeRef param_types[] = {};
    LLVMTypeRef ret_type = LLVMInt32TypeInContext(gen->context);
    LLVMTypeRef function_type = LLVMFunctionType(ret_type, param_types, 0, 0);
    
    // Создание функции
    gen->function = LLVMAddFunction(gen->module, "main", function_type);
    
    // Создание basic block
    LLVMBasicBlockRef entry = LLVMAppendBasicBlockInContext(gen->context, 
                                                           gen->function, "entry");
    LLVMPositionBuilderAtEnd(gen->builder, entry);
    
    // Генерация тела функции
    LLVMValueRef result = generate_llvm_expression(function_ast, gen);
    LLVMBuildRet(gen->builder, result);
}
```

## 🛠️ Практические проекты

### 🎯 Проект 1: Простой компилятор языка C (4-5 недель)
```c
// Основные компоненты:
// 1. Лексический анализатор для C-подобного синтаксиса
// 2. Рекурсивный спуск парсер
// 3. Семантический анализ с проверкой типов
// 4. Генерация промежуточного кода
// 5. Простые оптимизации
// 6. Генерация x86-64 ассемблера
```

### 🎯 Проект 2: Интерпретатор скриптового языка (3-4 недели)
```c
// Функциональность:
// 1. Динамическая типизация
// 2. Встроенные типы данных (string, array, hash)
// 3. Функции первого класса
// 4. Garbage collection
// 5. Встроенная стандартная библиотека
// 6. REPL интерфейс
```

### 🎯 Проект 3: JIT компилятор (5-6 недель)
```c
// Архитектура:
// 1. Bytecode интерпретатор
// 2. Профилирование hot spots
// 3. Компиляция в native код
// 4. Runtime оптимизации
// 5. Deoptimization механизм
// 6. Memory management
```

## 📚 Дополнительные ресурсы

### 📖 Рекомендуемая литература
- **"Compilers: Principles, Techniques, and Tools" - Aho, Sethi, Ullman** `📘 Классика`
- **"Modern Compiler Implementation in C" - Andrew Appel** `🔧 Практический подход`
- **"Engineering a Compiler" - Cooper & Torczon** `⚙️ Современные техники`

### 🔗 Связанные темы
- [[assembly-machine-code|Ассемблер и машинный код]] - для генерации кода
- [[memory-management|Управление памятью]] - для garbage collection
- [[../mathematics/theory-of-computation/README|Теория вычислений]] - формальные языки
- [[../algorithms-data-structures/README|Алгоритмы]] - для оптимизаций
- [[../technical-skills/databases/README|Базы данных]] - для символьных таблиц

### 🏗️ Roadmap развития
1. **Неделя 1-2**: Лексический и синтаксический анализ
2. **Неделя 3**: Семантический анализ, таблицы символов
3. **Неделя 4**: Генерация промежуточного кода
4. **Неделя 5**: Базовые оптимизации
5. **Неделя 6-7**: Генерация целевого кода
6. **Неделя 8**: LLVM интеграция и JIT

---

> 💡 **Совет**: Начните с простого калькулятора, затем добавляйте переменные, функции и сложные конструкции языка! 