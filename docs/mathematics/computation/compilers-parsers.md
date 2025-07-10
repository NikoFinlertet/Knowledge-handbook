# Компиляторы и парсеры

> **Практическое применение теории формальных языков**  
> Лексический анализ, синтаксический разбор и генерация кода

[[Теория вычислений]] → **Компиляторы и парсеры**

---

## 🎯 Ключевые концепции

### 🏗️ Фазы компиляции
- **Лексический анализ** - разбиение на токены
- **Синтаксический анализ** - построение дерева разбора
- **Семантический анализ** - проверка типов и смысла
- **Генерация кода** - создание исполняемого кода

### 🔗 Связи с другими темами
- **[[Формальные языки и грамматики]]** - теоретическая основа
- **[[Автоматы]]** - модели для лексеров и парсеров
- **[[Регулярные выражения]]** - описание токенов

---

## 🔤 Лексический анализ

### Токенизация

```python
import re
from enum import Enum

class TokenType(Enum):
    """Типы токенов"""
    # Литералы
    NUMBER = 'NUMBER'
    STRING = 'STRING'
    IDENTIFIER = 'IDENTIFIER'
    
    # Операторы
    PLUS = 'PLUS'
    MINUS = 'MINUS'
    MULTIPLY = 'MULTIPLY'
    DIVIDE = 'DIVIDE'
    ASSIGN = 'ASSIGN'
    
    # Сравнения
    EQUAL = 'EQUAL'
    NOT_EQUAL = 'NOT_EQUAL'
    LESS_THAN = 'LESS_THAN'
    GREATER_THAN = 'GREATER_THAN'
    
    # Ключевые слова
    IF = 'IF'
    ELSE = 'ELSE'
    WHILE = 'WHILE'
    FUNCTION = 'FUNCTION'
    RETURN = 'RETURN'
    
    # Разделители
    SEMICOLON = 'SEMICOLON'
    COMMA = 'COMMA'
    LEFT_PAREN = 'LEFT_PAREN'
    RIGHT_PAREN = 'RIGHT_PAREN'
    LEFT_BRACE = 'LEFT_BRACE'
    RIGHT_BRACE = 'RIGHT_BRACE'
    
    # Служебные
    WHITESPACE = 'WHITESPACE'
    NEWLINE = 'NEWLINE'
    EOF = 'EOF'
    UNKNOWN = 'UNKNOWN'

class Token:
    """Представление токена"""
    
    def __init__(self, token_type, value, line=1, column=1):
        self.type = token_type
        self.value = value
        self.line = line
        self.column = column
    
    def __str__(self):
        return f"Token({self.type.value}, '{self.value}', {self.line}:{self.column})"
    
    def __repr__(self):
        return self.__str__()

class Lexer:
    """Лексический анализатор"""
    
    def __init__(self):
        # Определяем токены с помощью регулярных выражений
        self.token_patterns = [
            # Ключевые слова (должны быть перед IDENTIFIER)
            (r'\bif\b', TokenType.IF),
            (r'\belse\b', TokenType.ELSE),
            (r'\bwhile\b', TokenType.WHILE),
            (r'\bfunction\b', TokenType.FUNCTION),
            (r'\breturn\b', TokenType.RETURN),
            
            # Литералы
            (r'\d+\.?\d*', TokenType.NUMBER),
            (r'"[^"]*"', TokenType.STRING),
            (r'[a-zA-Z_][a-zA-Z0-9_]*', TokenType.IDENTIFIER),
            
            # Двухсимвольные операторы
            (r'==', TokenType.EQUAL),
            (r'!=', TokenType.NOT_EQUAL),
            (r'<=', TokenType.LESS_THAN),  # Упрощенно
            (r'>=', TokenType.GREATER_THAN),  # Упрощенно
            
            # Односимвольные операторы
            (r'\+', TokenType.PLUS),
            (r'-', TokenType.MINUS),
            (r'\*', TokenType.MULTIPLY),
            (r'/', TokenType.DIVIDE),
            (r'=', TokenType.ASSIGN),
            (r'<', TokenType.LESS_THAN),
            (r'>', TokenType.GREATER_THAN),
            
            # Разделители
            (r';', TokenType.SEMICOLON),
            (r',', TokenType.COMMA),
            (r'\(', TokenType.LEFT_PAREN),
            (r'\)', TokenType.RIGHT_PAREN),
            (r'\{', TokenType.LEFT_BRACE),
            (r'\}', TokenType.RIGHT_BRACE),
            
            # Пробельные символы
            (r'[ \t]+', TokenType.WHITESPACE),
            (r'\n', TokenType.NEWLINE),
        ]
        
        # Компилируем регулярные выражения
        self.compiled_patterns = [
            (re.compile(pattern), token_type)
            for pattern, token_type in self.token_patterns
        ]
    
    def tokenize(self, text, skip_whitespace=True):
        """Токенизация входного текста"""
        tokens = []
        lines = text.split('\n')
        
        for line_num, line in enumerate(lines, 1):
            column = 1
            pos = 0
            
            while pos < len(line):
                matched = False
                
                for pattern, token_type in self.compiled_patterns:
                    match = pattern.match(line, pos)
                    if match:
                        value = match.group(0)
                        
                        # Пропускаем пробелы если нужно
                        if not skip_whitespace or token_type not in [TokenType.WHITESPACE]:
                            tokens.append(Token(token_type, value, line_num, column))
                        
                        pos = match.end()
                        column += len(value)
                        matched = True
                        break
                
                if not matched:
                    # Неизвестный символ
                    tokens.append(Token(TokenType.UNKNOWN, line[pos], line_num, column))
                    pos += 1
                    column += 1
            
            # Добавляем токен новой строки
            if not skip_whitespace:
                tokens.append(Token(TokenType.NEWLINE, '\\n', line_num, len(line) + 1))
        
        # Добавляем EOF
        tokens.append(Token(TokenType.EOF, '', len(lines) + 1, 1))
        return tokens
    
    def demonstrate_lexing(self):
        """Демонстрация лексического анализа"""
        
        sample_code = '''
        function factorial(n) {
            if (n == 0) {
                return 1;
            } else {
                return n * factorial(n - 1);
            }
        }
        
        x = factorial(5);
        '''
        
        print("=== Лексический анализ ===")
        print(f"Исходный код:\n{sample_code}")
        
        tokens = self.tokenize(sample_code)
        
        print(f"\nТокены:")
        for token in tokens[:20]:  # Показываем первые 20 токенов
            print(f"  {token}")
        
        if len(tokens) > 20:
            print(f"  ... и еще {len(tokens) - 20} токенов")
        
        return tokens

# Демонстрация лексера
lexer = Lexer()
tokens = lexer.demonstrate_lexing()
```

---

## 🌳 Синтаксический анализ

### Рекурсивный спуск

```python
class ASTNode:
    """Базовый класс для узлов абстрактного синтаксического дерева"""
    pass

class Program(ASTNode):
    def __init__(self, statements):
        self.statements = statements
    
    def __str__(self):
        return f"Program({self.statements})"

class FunctionDef(ASTNode):
    def __init__(self, name, parameters, body):
        self.name = name
        self.parameters = parameters
        self.body = body
    
    def __str__(self):
        return f"FunctionDef({self.name}, {self.parameters}, {self.body})"

class IfStatement(ASTNode):
    def __init__(self, condition, then_stmt, else_stmt=None):
        self.condition = condition
        self.then_stmt = then_stmt
        self.else_stmt = else_stmt
    
    def __str__(self):
        return f"If({self.condition}, {self.then_stmt}, {self.else_stmt})"

class ReturnStatement(ASTNode):
    def __init__(self, expression):
        self.expression = expression
    
    def __str__(self):
        return f"Return({self.expression})"

class Assignment(ASTNode):
    def __init__(self, variable, expression):
        self.variable = variable
        self.expression = expression
    
    def __str__(self):
        return f"Assign({self.variable}, {self.expression})"

class BinaryOp(ASTNode):
    def __init__(self, left, operator, right):
        self.left = left
        self.operator = operator
        self.right = right
    
    def __str__(self):
        return f"BinaryOp({self.left}, {self.operator}, {self.right})"

class FunctionCall(ASTNode):
    def __init__(self, name, arguments):
        self.name = name
        self.arguments = arguments
    
    def __str__(self):
        return f"Call({self.name}, {self.arguments})"

class Identifier(ASTNode):
    def __init__(self, name):
        self.name = name
    
    def __str__(self):
        return f"Id({self.name})"

class Number(ASTNode):
    def __init__(self, value):
        self.value = float(value)
    
    def __str__(self):
        return f"Num({self.value})"

class Block(ASTNode):
    def __init__(self, statements):
        self.statements = statements
    
    def __str__(self):
        return f"Block({self.statements})"

class RecursiveDescentParser:
    """Парсер методом рекурсивного спуска"""
    
    def __init__(self, tokens):
        self.tokens = tokens
        self.current = 0
    
    def peek(self):
        """Посмотреть текущий токен без продвижения"""
        if self.current < len(self.tokens):
            return self.tokens[self.current]
        return Token(TokenType.EOF, '', 0, 0)
    
    def advance(self):
        """Перейти к следующему токену"""
        if self.current < len(self.tokens):
            token = self.tokens[self.current]
            self.current += 1
            return token
        return Token(TokenType.EOF, '', 0, 0)
    
    def match(self, *token_types):
        """Проверить, соответствует ли текущий токен одному из типов"""
        current_token = self.peek()
        return current_token.type in token_types
    
    def consume(self, token_type, message="Неожиданный токен"):
        """Потребить токен определенного типа или выдать ошибку"""
        if self.match(token_type):
            return self.advance()
        else:
            current = self.peek()
            raise SyntaxError(f"{message}. Ожидался {token_type.value}, получен {current.type.value} в строке {current.line}")
    
    def parse(self):
        """Основная функция парсинга"""
        statements = []
        
        while not self.match(TokenType.EOF):
            if self.match(TokenType.NEWLINE):
                self.advance()
                continue
            
            stmt = self.parse_statement()
            if stmt:
                statements.append(stmt)
        
        return Program(statements)
    
    def parse_statement(self):
        """Парсинг утверждения (statement)"""
        if self.match(TokenType.FUNCTION):
            return self.parse_function_def()
        elif self.match(TokenType.IF):
            return self.parse_if_statement()
        elif self.match(TokenType.RETURN):
            return self.parse_return_statement()
        elif self.match(TokenType.IDENTIFIER):
            return self.parse_assignment_or_expression()
        else:
            # Пропускаем неизвестные токены
            self.advance()
            return None
    
    def parse_function_def(self):
        """Парсинг определения функции"""
        self.consume(TokenType.FUNCTION)
        name = self.consume(TokenType.IDENTIFIER).value
        
        self.consume(TokenType.LEFT_PAREN)
        parameters = []
        
        if not self.match(TokenType.RIGHT_PAREN):
            parameters.append(self.consume(TokenType.IDENTIFIER).value)
            while self.match(TokenType.COMMA):
                self.advance()
                parameters.append(self.consume(TokenType.IDENTIFIER).value)
        
        self.consume(TokenType.RIGHT_PAREN)
        body = self.parse_block()
        
        return FunctionDef(name, parameters, body)
    
    def parse_if_statement(self):
        """Парсинг условного оператора"""
        self.consume(TokenType.IF)
        self.consume(TokenType.LEFT_PAREN)
        condition = self.parse_expression()
        self.consume(TokenType.RIGHT_PAREN)
        
        then_stmt = self.parse_block()
        else_stmt = None
        
        if self.match(TokenType.ELSE):
            self.advance()
            else_stmt = self.parse_block()
        
        return IfStatement(condition, then_stmt, else_stmt)
    
    def parse_return_statement(self):
        """Парсинг оператора возврата"""
        self.consume(TokenType.RETURN)
        expression = self.parse_expression()
        self.consume(TokenType.SEMICOLON)
        return ReturnStatement(expression)
    
    def parse_assignment_or_expression(self):
        """Парсинг присваивания или выражения"""
        if self.current + 1 < len(self.tokens) and self.tokens[self.current + 1].type == TokenType.ASSIGN:
            # Это присваивание
            variable = self.consume(TokenType.IDENTIFIER).value
            self.consume(TokenType.ASSIGN)
            expression = self.parse_expression()
            self.consume(TokenType.SEMICOLON)
            return Assignment(variable, expression)
        else:
            # Это просто выражение
            expr = self.parse_expression()
            if self.match(TokenType.SEMICOLON):
                self.advance()
            return expr
    
    def parse_block(self):
        """Парсинг блока утверждений"""
        self.consume(TokenType.LEFT_BRACE)
        statements = []
        
        while not self.match(TokenType.RIGHT_BRACE) and not self.match(TokenType.EOF):
            if self.match(TokenType.NEWLINE):
                self.advance()
                continue
            
            stmt = self.parse_statement()
            if stmt:
                statements.append(stmt)
        
        self.consume(TokenType.RIGHT_BRACE)
        return Block(statements)
    
    def parse_expression(self):
        """Парсинг выражения с учетом приоритета операций"""
        return self.parse_equality()
    
    def parse_equality(self):
        """Парсинг операций сравнения"""
        expr = self.parse_comparison()
        
        while self.match(TokenType.EQUAL, TokenType.NOT_EQUAL):
            operator = self.advance().value
            right = self.parse_comparison()
            expr = BinaryOp(expr, operator, right)
        
        return expr
    
    def parse_comparison(self):
        """Парсинг операций сравнения по величине"""
        expr = self.parse_term()
        
        while self.match(TokenType.LESS_THAN, TokenType.GREATER_THAN):
            operator = self.advance().value
            right = self.parse_term()
            expr = BinaryOp(expr, operator, right)
        
        return expr
    
    def parse_term(self):
        """Парсинг сложения и вычитания"""
        expr = self.parse_factor()
        
        while self.match(TokenType.PLUS, TokenType.MINUS):
            operator = self.advance().value
            right = self.parse_factor()
            expr = BinaryOp(expr, operator, right)
        
        return expr
    
    def parse_factor(self):
        """Парсинг умножения и деления"""
        expr = self.parse_unary()
        
        while self.match(TokenType.MULTIPLY, TokenType.DIVIDE):
            operator = self.advance().value
            right = self.parse_unary()
            expr = BinaryOp(expr, operator, right)
        
        return expr
    
    def parse_unary(self):
        """Парсинг унарных операций"""
        if self.match(TokenType.MINUS):
            operator = self.advance().value
            expr = self.parse_unary()
            return BinaryOp(Number(0), operator, expr)  # Преобразуем в бинарную операцию
        
        return self.parse_primary()
    
    def parse_primary(self):
        """Парсинг первичных выражений"""
        if self.match(TokenType.NUMBER):
            return Number(self.advance().value)
        
        if self.match(TokenType.IDENTIFIER):
            name = self.advance().value
            
            # Проверяем, это вызов функции или просто идентификатор
            if self.match(TokenType.LEFT_PAREN):
                self.advance()
                arguments = []
                
                if not self.match(TokenType.RIGHT_PAREN):
                    arguments.append(self.parse_expression())
                    while self.match(TokenType.COMMA):
                        self.advance()
                        arguments.append(self.parse_expression())
                
                self.consume(TokenType.RIGHT_PAREN)
                return FunctionCall(name, arguments)
            else:
                return Identifier(name)
        
        if self.match(TokenType.LEFT_PAREN):
            self.advance()
            expr = self.parse_expression()
            self.consume(TokenType.RIGHT_PAREN)
            return expr
        
        current = self.peek()
        raise SyntaxError(f"Неожиданный токен {current.type.value} в строке {current.line}")
    
    def demonstrate_parsing(self):
        """Демонстрация синтаксического анализа"""
        
        print("\n=== Синтаксический анализ ===")
        
        try:
            ast = self.parse()
            print("Абстрактное синтаксическое дерево:")
            self.print_ast(ast, 0)
            return ast
        except SyntaxError as e:
            print(f"Ошибка синтаксиса: {e}")
            return None
    
    def print_ast(self, node, depth):
        """Печать AST с отступами"""
        indent = "  " * depth
        
        if isinstance(node, Program):
            print(f"{indent}Program:")
            for stmt in node.statements:
                self.print_ast(stmt, depth + 1)
        
        elif isinstance(node, FunctionDef):
            print(f"{indent}Function: {node.name}")
            print(f"{indent}  Parameters: {node.parameters}")
            print(f"{indent}  Body:")
            self.print_ast(node.body, depth + 2)
        
        elif isinstance(node, IfStatement):
            print(f"{indent}If:")
            print(f"{indent}  Condition:")
            self.print_ast(node.condition, depth + 2)
            print(f"{indent}  Then:")
            self.print_ast(node.then_stmt, depth + 2)
            if node.else_stmt:
                print(f"{indent}  Else:")
                self.print_ast(node.else_stmt, depth + 2)
        
        elif isinstance(node, ReturnStatement):
            print(f"{indent}Return:")
            self.print_ast(node.expression, depth + 1)
        
        elif isinstance(node, Assignment):
            print(f"{indent}Assignment: {node.variable} =")
            self.print_ast(node.expression, depth + 1)
        
        elif isinstance(node, BinaryOp):
            print(f"{indent}BinaryOp: {node.operator}")
            print(f"{indent}  Left:")
            self.print_ast(node.left, depth + 2)
            print(f"{indent}  Right:")
            self.print_ast(node.right, depth + 2)
        
        elif isinstance(node, FunctionCall):
            print(f"{indent}Call: {node.name}")
            for arg in node.arguments:
                self.print_ast(arg, depth + 1)
        
        elif isinstance(node, Identifier):
            print(f"{indent}Identifier: {node.name}")
        
        elif isinstance(node, Number):
            print(f"{indent}Number: {node.value}")
        
        elif isinstance(node, Block):
            print(f"{indent}Block:")
            for stmt in node.statements:
                self.print_ast(stmt, depth + 1)
        
        else:
            print(f"{indent}{type(node).__name__}: {node}")

# Демонстрация парсера
parser = RecursiveDescentParser(tokens)
ast = parser.demonstrate_parsing()
```

---

## 🎯 Семантический анализ

### Проверка типов и область видимости

```python
class SymbolTable:
    """Таблица символов для отслеживания переменных и функций"""
    
    def __init__(self, parent=None):
        self.parent = parent
        self.symbols = {}
        self.functions = {}
    
    def define_variable(self, name, var_type):
        """Определить переменную в текущей области видимости"""
        self.symbols[name] = var_type
    
    def define_function(self, name, return_type, param_types):
        """Определить функцию"""
        self.functions[name] = {
            'return_type': return_type,
            'param_types': param_types
        }
    
    def lookup_variable(self, name):
        """Найти переменную в таблице символов"""
        if name in self.symbols:
            return self.symbols[name]
        elif self.parent:
            return self.parent.lookup_variable(name)
        else:
            return None
    
    def lookup_function(self, name):
        """Найти функцию в таблице символов"""
        if name in self.functions:
            return self.functions[name]
        elif self.parent:
            return self.parent.lookup_function(name)
        else:
            return None

class SemanticAnalyzer:
    """Семантический анализатор"""
    
    def __init__(self):
        self.symbol_table = SymbolTable()
        self.errors = []
        self.current_function = None
    
    def analyze(self, ast):
        """Выполнить семантический анализ"""
        self.errors = []
        self.visit(ast)
        return self.errors
    
    def error(self, message):
        """Зарегистрировать ошибку"""
        self.errors.append(message)
    
    def visit(self, node):
        """Обход узла AST"""
        method_name = f'visit_{type(node).__name__}'
        method = getattr(self, method_name, self.generic_visit)
        return method(node)
    
    def generic_visit(self, node):
        """Обобщенное посещение узла"""
        pass
    
    def visit_Program(self, node):
        """Анализ программы"""
        for stmt in node.statements:
            self.visit(stmt)
    
    def visit_FunctionDef(self, node):
        """Анализ определения функции"""
        # Проверяем, не определена ли функция уже
        if self.symbol_table.lookup_function(node.name):
            self.error(f"Функция '{node.name}' уже определена")
        
        # Определяем функцию в таблице символов
        param_types = ['number'] * len(node.parameters)  # Упрощенно: все параметры числа
        self.symbol_table.define_function(node.name, 'number', param_types)
        
        # Создаем новую область видимости для функции
        old_table = self.symbol_table
        old_function = self.current_function
        self.symbol_table = SymbolTable(old_table)
        self.current_function = node.name
        
        # Определяем параметры как локальные переменные
        for param in node.parameters:
            self.symbol_table.define_variable(param, 'number')
        
        # Анализируем тело функции
        self.visit(node.body)
        
        # Восстанавливаем предыдущую область видимости
        self.symbol_table = old_table
        self.current_function = old_function
    
    def visit_IfStatement(self, node):
        """Анализ условного оператора"""
        condition_type = self.visit(node.condition)
        
        # Условие должно быть булевым (упрощенно: любое выражение)
        self.visit(node.then_stmt)
        if node.else_stmt:
            self.visit(node.else_stmt)
    
    def visit_ReturnStatement(self, node):
        """Анализ оператора возврата"""
        if not self.current_function:
            self.error("return вне функции")
        
        expr_type = self.visit(node.expression)
        # В реальном анализаторе проверили бы соответствие типа возвращаемого значения
    
    def visit_Assignment(self, node):
        """Анализ присваивания"""
        expr_type = self.visit(node.expression)
        self.symbol_table.define_variable(node.variable, expr_type)
    
    def visit_BinaryOp(self, node):
        """Анализ бинарной операции"""
        left_type = self.visit(node.left)
        right_type = self.visit(node.right)
        
        # Проверяем совместимость типов для операции
        if node.operator in ['+', '-', '*', '/']:
            if left_type != 'number' or right_type != 'number':
                self.error(f"Арифметическая операция '{node.operator}' требует числовые операнды")
            return 'number'
        elif node.operator in ['==', '!=', '<', '>']:
            if left_type != right_type:
                self.error(f"Сравнение '{node.operator}' требует операнды одного типа")
            return 'boolean'
        else:
            self.error(f"Неизвестная операция '{node.operator}'")
            return 'unknown'
    
    def visit_FunctionCall(self, node):
        """Анализ вызова функции"""
        func_info = self.symbol_table.lookup_function(node.name)
        
        if not func_info:
            self.error(f"Неизвестная функция '{node.name}'")
            return 'unknown'
        
        # Проверяем количество аргументов
        expected_args = len(func_info['param_types'])
        actual_args = len(node.arguments)
        
        if expected_args != actual_args:
            self.error(f"Функция '{node.name}' ожидает {expected_args} аргументов, получено {actual_args}")
        
        # Проверяем типы аргументов
        for i, arg in enumerate(node.arguments):
            arg_type = self.visit(arg)
            if i < len(func_info['param_types']):
                expected_type = func_info['param_types'][i]
                if arg_type != expected_type and arg_type != 'unknown':
                    self.error(f"Аргумент {i+1} функции '{node.name}': ожидался {expected_type}, получен {arg_type}")
        
        return func_info['return_type']
    
    def visit_Identifier(self, node):
        """Анализ идентификатора"""
        var_type = self.symbol_table.lookup_variable(node.name)
        
        if var_type is None:
            self.error(f"Неопределенная переменная '{node.name}'")
            return 'unknown'
        
        return var_type
    
    def visit_Number(self, node):
        """Анализ числового литерала"""
        return 'number'
    
    def visit_Block(self, node):
        """Анализ блока"""
        for stmt in node.statements:
            self.visit(stmt)
    
    def demonstrate_semantic_analysis(self, ast):
        """Демонстрация семантического анализа"""
        if not ast:
            print("\n=== Семантический анализ пропущен (нет AST) ===")
            return
        
        print("\n=== Семантический анализ ===")
        
        errors = self.analyze(ast)
        
        if errors:
            print("Найдены ошибки:")
            for error in errors:
                print(f"  • {error}")
        else:
            print("Семантических ошибок не найдено")
        
        return errors

# Демонстрация семантического анализа
semantic_analyzer = SemanticAnalyzer()
semantic_errors = semantic_analyzer.demonstrate_semantic_analysis(ast)
```

---

## ⚙️ Генерация кода

### Простой генератор байт-кода

```python
class Instruction:
    """Инструкция виртуальной машины"""
    
    def __init__(self, opcode, operand=None):
        self.opcode = opcode
        self.operand = operand
    
    def __str__(self):
        if self.operand is not None:
            return f"{self.opcode} {self.operand}"
        return self.opcode

class CodeGenerator:
    """Генератор байт-кода"""
    
    def __init__(self):
        self.instructions = []
        self.label_counter = 0
        self.local_vars = {}
        self.var_counter = 0
    
    def emit(self, opcode, operand=None):
        """Выдать инструкцию"""
        instruction = Instruction(opcode, operand)
        self.instructions.append(instruction)
        return len(self.instructions) - 1  # Возвращаем адрес инструкции
    
    def new_label(self):
        """Создать новую метку"""
        self.label_counter += 1
        return f"L{self.label_counter}"
    
    def get_var_address(self, name):
        """Получить адрес переменной"""
        if name not in self.local_vars:
            self.local_vars[name] = self.var_counter
            self.var_counter += 1
        return self.local_vars[name]
    
    def generate(self, ast):
        """Генерировать код для AST"""
        self.instructions = []
        self.local_vars = {}
        self.var_counter = 0
        
        self.visit(ast)
        return self.instructions
    
    def visit(self, node):
        """Обход узла AST для генерации кода"""
        method_name = f'generate_{type(node).__name__}'
        method = getattr(self, method_name, self.generic_generate)
        return method(node)
    
    def generic_generate(self, node):
        """Обобщенная генерация"""
        pass
    
    def generate_Program(self, node):
        """Генерация кода для программы"""
        for stmt in node.statements:
            self.visit(stmt)
        self.emit('HALT')  # Остановка программы
    
    def generate_FunctionDef(self, node):
        """Генерация кода для определения функции"""
        func_label = f"func_{node.name}"
        self.emit('LABEL', func_label)
        
        # Сохраняем контекст переменных
        old_vars = self.local_vars.copy()
        old_counter = self.var_counter
        
        # Параметры функции
        for i, param in enumerate(node.parameters):
            self.local_vars[param] = i
        self.var_counter = len(node.parameters)
        
        # Генерируем тело функции
        self.visit(node.body)
        
        # Если нет явного return, возвращаем 0
        self.emit('LOAD_CONST', 0)
        self.emit('RETURN')
        
        # Восстанавливаем контекст
        self.local_vars = old_vars
        self.var_counter = old_counter
    
    def generate_IfStatement(self, node):
        """Генерация кода для условного оператора"""
        else_label = self.new_label()
        end_label = self.new_label()
        
        # Вычисляем условие
        self.visit(node.condition)
        
        # Переход, если условие ложно
        self.emit('JUMP_IF_FALSE', else_label)
        
        # Код для then-ветки
        self.visit(node.then_stmt)
        self.emit('JUMP', end_label)
        
        # Код для else-ветки
        self.emit('LABEL', else_label)
        if node.else_stmt:
            self.visit(node.else_stmt)
        
        self.emit('LABEL', end_label)
    
    def generate_ReturnStatement(self, node):
        """Генерация кода для return"""
        self.visit(node.expression)
        self.emit('RETURN')
    
    def generate_Assignment(self, node):
        """Генерация кода для присваивания"""
        # Вычисляем выражение
        self.visit(node.expression)
        
        # Сохраняем в переменную
        var_addr = self.get_var_address(node.variable)
        self.emit('STORE_VAR', var_addr)
    
    def generate_BinaryOp(self, node):
        """Генерация кода для бинарной операции"""
        # Вычисляем левый операнд
        self.visit(node.left)
        
        # Вычисляем правый операнд
        self.visit(node.right)
        
        # Выполняем операцию
        op_map = {
            '+': 'ADD',
            '-': 'SUB',
            '*': 'MUL',
            '/': 'DIV',
            '==': 'EQ',
            '!=': 'NE',
            '<': 'LT',
            '>': 'GT'
        }
        
        if node.operator in op_map:
            self.emit(op_map[node.operator])
        else:
            raise ValueError(f"Неизвестная операция: {node.operator}")
    
    def generate_FunctionCall(self, node):
        """Генерация кода для вызова функции"""
        # Передаем аргументы на стек
        for arg in node.arguments:
            self.visit(arg)
        
        # Вызываем функцию
        self.emit('CALL', f"func_{node.name}")
    
    def generate_Identifier(self, node):
        """Генерация кода для идентификатора"""
        var_addr = self.get_var_address(node.name)
        self.emit('LOAD_VAR', var_addr)
    
    def generate_Number(self, node):
        """Генерация кода для числа"""
        self.emit('LOAD_CONST', node.value)
    
    def generate_Block(self, node):
        """Генерация кода для блока"""
        for stmt in node.statements:
            self.visit(stmt)
    
    def demonstrate_code_generation(self, ast):
        """Демонстрация генерации кода"""
        if not ast:
            print("\n=== Генерация кода пропущена (нет AST) ===")
            return
        
        print("\n=== Генерация кода ===")
        
        try:
            instructions = self.generate(ast)
            
            print("Сгенерированный байт-код:")
            for i, instr in enumerate(instructions):
                print(f"  {i:3d}: {instr}")
            
            return instructions
        except Exception as e:
            print(f"Ошибка генерации кода: {e}")
            return None

# Демонстрация генерации кода
code_generator = CodeGenerator()
bytecode = code_generator.demonstrate_code_generation(ast)
```

---

## 🔗 Связанные темы

### 📚 Теоретические основы
- **[[Формальные языки и грамматики]]** - синтаксическая структура языков
- **[[Автоматы]]** - модели для лексеров и парсеров
- **[[Регулярные выражения]]** - описание токенов

### 🛠️ Практические техники
- **[[Анализ алгоритмов]]** - оптимизация компиляторов
- **[[Теория типов]]** - системы типов в языках программирования
- **[[Архитектура процессоров]]** - генерация машинного кода

### 🔧 Инструменты
- **ANTLR/Yacc** - генераторы парсеров
- **Flex/Lex** - генераторы лексеров
- **LLVM** - инфраструктура компиляторов

---

## 🎓 Практические задания

### 📝 Начальный уровень
1. **Лексический анализ**
   - [ ] Расширить лексер новыми токенами
   - [ ] Добавить поддержку комментариев
   - [ ] Обработать строковые литералы с экранированием

2. **Простой парсер**
   - [ ] Парсить арифметические выражения
   - [ ] Добавить поддержку унарных операций
   - [ ] Реализовать ассоциативность операций

### 🔧 Средний уровень
3. **Семантический анализ**
   - [ ] Проверка типов в выражениях
   - [ ] Отслеживание области видимости переменных
   - [ ] Проверка возвращаемых значений функций

4. **Генерация кода**
   - [ ] Генерация кода для циклов
   - [ ] Оптимизация выражений
   - [ ] Поддержка локальных переменных

### 🚀 Продвинутый уровень
5. **Оптимизация**
   - [ ] Устранение мертвого кода
   - [ ] Сворачивание констант
   - [ ] Анализ потока данных

---

*"Компиляторы превращают красивый код в эффективные программы - мост между человеческим мышлением и машинными вычислениями"* ⚙️🔧 