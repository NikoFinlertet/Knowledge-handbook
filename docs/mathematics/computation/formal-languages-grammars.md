# Формальные языки и грамматики

> **Основы теории формальных языков**  
> Алфавиты, языки, грамматики и иерархия Хомского

[[Теория вычислений]] → **Формальные языки и грамматики**

---

## 🎯 Ключевые концепции

### 📝 Основные понятия
- **Алфавит** - конечное множество символов
- **Строка** - конечная последовательность символов из алфавита
- **Язык** - множество строк над алфавитом
- **Грамматика** - формальная система для порождения языка

### 🔗 Связи с другими темами
- **[[Автоматы]]** - модели для распознавания языков
- **[[Регулярные выражения]]** - описание регулярных языков
- **[[Компиляторы и парсеры]]** - практическое применение

---

## 📚 Основные понятия

### Алфавит и языки

```python
# Пример: алфавит двоичных чисел
alphabet = {'0', '1'}

# Язык всех двоичных строк четной длины
even_length_binary = {
    "", "01", "10", "11", "00", "0011", "0110", "1001", "1100"
    # ... и так далее
}

# Формальное определение языка
def is_even_length_binary(word):
    """Проверяет, принадлежит ли слово языку двоичных строк четной длины"""
    return len(word) % 2 == 0 and all(char in {'0', '1'} for char in word)

# Операции над языками
def language_operations_demo():
    """Демонстрация операций над языками"""
    
    # Алфавит
    sigma = {'a', 'b'}
    
    # Языки
    L1 = {'a', 'ab', 'abb'}
    L2 = {'b', 'ba'}
    
    # Объединение
    union = L1 | L2
    print(f"L1 ∪ L2 = {union}")
    
    # Пересечение
    intersection = L1 & L2
    print(f"L1 ∩ L2 = {intersection}")
    
    # Конкатенация
    concatenation = {w1 + w2 for w1 in L1 for w2 in L2}
    print(f"L1 · L2 = {concatenation}")
    
    # Замыкание Клини (упрощенная версия)
    def kleene_star(language, max_length=10):
        """Замыкание Клини до определенной длины"""
        result = {''}  # Пустая строка всегда в L*
        current_level = {''}
        
        for _ in range(max_length):
            next_level = set()
            for word in current_level:
                for lang_word in language:
                    new_word = word + lang_word
                    if len(new_word) <= max_length:
                        next_level.add(new_word)
            
            if not next_level:
                break
                
            result.update(next_level)
            current_level = next_level
        
        return result
    
    star_L1 = kleene_star(L1, 6)
    print(f"L1* (до длины 6) = {star_L1}")

# Демонстрация
language_operations_demo()
```

### Иерархия Хомского

```python
class ChomskiHierarchy:
    """Иерархия формальных языков по Хомскому"""
    
    def __init__(self):
        self.types = {
            0: "Неограниченные грамматики (рекурсивно перечислимые языки)",
            1: "Контекстно-зависимые грамматики", 
            2: "Контекстно-свободные грамматики",
            3: "Регулярные грамматики"
        }
    
    def explain_type(self, type_num):
        """Объяснение типа грамматики с примерами"""
        explanations = {
            3: """
            Регулярные языки (Тип 3):
            - Распознаются конечными автоматами
            - Примеры: email валидация, лексический анализ
            - Правила: A → aB, A → a (только правые регулярные)
            - Применение: [[Регулярные выражения]], лексеры
            """,
            2: """
            Контекстно-свободные языки (Тип 2):
            - Распознаются магазинными автоматами
            - Примеры: арифметические выражения, языки программирования
            - Правила: A → α (левая часть - одна переменная)
            - Применение: [[Компиляторы и парсеры]]
            """,
            1: """
            Контекстно-зависимые языки (Тип 1):
            - Распознаются линейно-ограниченными автоматами
            - Примеры: {aⁿbⁿcⁿ | n ≥ 1}
            - Правила: αAβ → αγβ (контекст α и β)
            - Применение: семантический анализ
            """,
            0: """
            Неограниченные языки (Тип 0):
            - Распознаются [[Машины Тьюринга|машинами Тьюринга]]
            - Примеры: любые алгоритмически разрешимые задачи
            - Правила: любые продукции
            - Применение: общие алгоритмы
            """
        }
        return explanations.get(type_num, "Неизвестный тип")
    
    def get_example_grammar(self, type_num):
        """Примеры грамматик каждого типа"""
        examples = {
            3: {
                'description': 'Регулярная грамматика для языка (ab)*',
                'productions': ['S → abS', 'S → ε'],
                'language': 'Язык всех строк вида (ab)*'
            },
            2: {
                'description': 'КС-грамматика для сбалансированных скобок',
                'productions': ['S → (S)', 'S → SS', 'S → ε'],
                'language': 'Язык правильно вложенных скобок'
            },
            1: {
                'description': 'КЗ-грамматика для языка {aⁿbⁿcⁿ | n ≥ 1}',
                'productions': ['S → aSBC', 'CB → BC', 'aB → ab', 'bB → bb', 'bC → bc', 'cC → cc'],
                'language': 'Язык с равным количеством a, b, c'
            },
            0: {
                'description': 'Неограниченная грамматика для вычисления функций',
                'productions': ['Любые продукции α → β'],
                'language': 'Любой рекурсивно перечислимый язык'
            }
        }
        return examples.get(type_num, {})

# Создание и демонстрация иерархии
hierarchy = ChomskiHierarchy()
for i in range(4):
    print(f"\n=== Тип {i} ===")
    print(hierarchy.explain_type(i))
    example = hierarchy.get_example_grammar(i)
    if example:
        print(f"Пример: {example['description']}")
        print(f"Язык: {example['language']}")
```

---

## 🔧 Грамматики

### Контекстно-свободная грамматика

```python
class ContextFreeGrammar:
    """Контекстно-свободная грамматика для арифметических выражений"""
    
    def __init__(self):
        # Грамматика для простых арифметических выражений
        self.productions = {
            'E': ['E+T', 'E-T', 'T'],           # Expression
            'T': ['T*F', 'T/F', 'F'],           # Term  
            'F': ['(E)', 'id', 'num']           # Factor
        }
        
        self.start_symbol = 'E'
        self.terminals = {'+', '-', '*', '/', '(', ')', 'id', 'num'}
        self.non_terminals = {'E', 'T', 'F'}
    
    def parse_expression(self, expression):
        """Парсинг выражения с помощью рекурсивного спуска"""
        tokens = self.tokenize(expression)
        return self.parse_E(tokens, 0)
    
    def tokenize(self, expression):
        """Простой токенизатор для демонстрации"""
        import re
        pattern = r'(\d+|\w+|[+\-*/()])'
        return re.findall(pattern, expression)
    
    def parse_E(self, tokens, pos):
        """Парсинг выражения E"""
        left, pos = self.parse_T(tokens, pos)
        
        while pos < len(tokens) and tokens[pos] in ['+', '-']:
            op = tokens[pos]
            pos += 1
            right, pos = self.parse_T(tokens, pos)
            left = {'type': 'binary_op', 'op': op, 'left': left, 'right': right}
        
        return left, pos
    
    def parse_T(self, tokens, pos):
        """Парсинг терма T"""
        left, pos = self.parse_F(tokens, pos)
        
        while pos < len(tokens) and tokens[pos] in ['*', '/']:
            op = tokens[pos] 
            pos += 1
            right, pos = self.parse_F(tokens, pos)
            left = {'type': 'binary_op', 'op': op, 'left': left, 'right': right}
        
        return left, pos
    
    def parse_F(self, tokens, pos):
        """Парсинг фактора F"""
        if pos >= len(tokens):
            raise ValueError("Unexpected end of expression")
        
        token = tokens[pos]
        
        if token == '(':
            pos += 1  # пропускаем '('
            result, pos = self.parse_E(tokens, pos)
            if pos >= len(tokens) or tokens[pos] != ')':
                raise ValueError("Missing closing parenthesis")
            pos += 1  # пропускаем ')'
            return result, pos
        
        elif token.isdigit():
            return {'type': 'number', 'value': int(token)}, pos + 1
        
        elif token.isalpha():
            return {'type': 'variable', 'name': token}, pos + 1
        
        else:
            raise ValueError(f"Unexpected token: {token}")
    
    def generate_strings(self, max_depth=3):
        """Генерация строк языка до определенной глубины"""
        def expand_symbol(symbol, depth):
            if depth <= 0:
                return []
            
            if symbol in self.terminals:
                return [symbol]
            
            if symbol not in self.productions:
                return []
            
            results = []
            for production in self.productions[symbol]:
                if len(production) == 1 and production in self.terminals:
                    results.append(production)
                else:
                    # Рекурсивно разворачиваем продукцию
                    sub_results = [[]]
                    for char in production:
                        new_sub_results = []
                        for sub_result in sub_results:
                            expansions = expand_symbol(char, depth - 1)
                            for expansion in expansions:
                                new_sub_results.append(sub_result + [expansion])
                        sub_results = new_sub_results
                    
                    for sub_result in sub_results:
                        results.append(''.join(sub_result))
            
            return results
        
        return expand_symbol(self.start_symbol, max_depth)

# Пример использования
grammar = ContextFreeGrammar()
try:
    ast, _ = grammar.parse_expression("2 + 3 * 4")
    print("Дерево разбора для '2 + 3 * 4':")
    print(ast)
    
    # Генерация некоторых строк языка
    generated = grammar.generate_strings(2)
    print(f"\nНекоторые строки языка: {generated[:10]}")
    
except Exception as e:
    print(f"Ошибка: {e}")
```

### Регулярная грамматика

```python
class RegularGrammar:
    """Регулярная грамматика и ее связь с конечными автоматами"""
    
    def __init__(self, productions, start_symbol):
        self.productions = productions  # dict: non_terminal -> [list of productions]
        self.start_symbol = start_symbol
        self.terminals = self._extract_terminals()
        self.non_terminals = set(productions.keys())
    
    def _extract_terminals(self):
        """Извлечение терминальных символов из продукций"""
        terminals = set()
        for productions_list in self.productions.values():
            for production in productions_list:
                for char in production:
                    if char.islower() or char in '()+-*/':
                        terminals.add(char)
        return terminals
    
    def to_finite_automaton(self):
        """Преобразование регулярной грамматики в конечный автомат"""
        states = set(self.non_terminals)
        states.add('ACCEPT')  # Принимающее состояние
        
        transitions = {}
        accept_states = {'ACCEPT'}
        
        for non_terminal, productions_list in self.productions.items():
            for production in productions_list:
                if production == 'ε':  # Пустая продукция
                    accept_states.add(non_terminal)
                elif len(production) == 1 and production in self.terminals:
                    # A → a
                    transitions[(non_terminal, production)] = 'ACCEPT'
                elif len(production) == 2 and production[0] in self.terminals:
                    # A → aB
                    terminal, next_state = production[0], production[1]
                    transitions[(non_terminal, terminal)] = next_state
        
        return {
            'states': states,
            'alphabet': self.terminals,
            'transitions': transitions,
            'start_state': self.start_symbol,
            'accept_states': accept_states
        }
    
    def generate_language_sample(self, max_length=5):
        """Генерация примеров строк языка"""
        def generate_from_symbol(symbol, current_string="", max_len=5):
            if len(current_string) > max_len:
                return []
            
            if symbol not in self.productions:
                return [current_string] if symbol == 'ACCEPT' else []
            
            results = []
            for production in self.productions[symbol]:
                if production == 'ε':
                    results.append(current_string)
                elif len(production) == 1 and production in self.terminals:
                    results.append(current_string + production)
                elif len(production) == 2:
                    terminal, next_symbol = production[0], production[1]
                    results.extend(generate_from_symbol(
                        next_symbol, 
                        current_string + terminal, 
                        max_len
                    ))
            
            return results
        
        return generate_from_symbol(self.start_symbol, "", max_length)

# Пример: регулярная грамматика для языка (ab)*
ab_star_grammar = RegularGrammar(
    productions={
        'S': ['abS', 'ε']  # S → abS | ε
    },
    start_symbol='S'
)

# Преобразование в автомат
automaton = ab_star_grammar.to_finite_automaton()
print("Автомат для языка (ab)*:")
print(f"Состояния: {automaton['states']}")
print(f"Переходы: {automaton['transitions']}")

# Генерация примеров
examples = ab_star_grammar.generate_language_sample(8)
print(f"Примеры строк языка: {examples}")
```

---

## 🎯 Практические применения

### 1. Лексический анализ

```python
def lexical_analyzer_demo():
    """Демонстрация лексического анализатора на основе регулярных грамматик"""
    
    # Определение токенов с помощью регулярных выражений
    import re
    
    token_patterns = [
        ('NUMBER', r'\d+'),
        ('IDENTIFIER', r'[a-zA-Z_][a-zA-Z0-9_]*'),
        ('PLUS', r'\+'),
        ('MINUS', r'-'),
        ('MULTIPLY', r'\*'),
        ('DIVIDE', r'/'),
        ('LPAREN', r'\('),
        ('RPAREN', r'\)'),
        ('WHITESPACE', r'\s+'),
    ]
    
    def tokenize(text):
        """Токенизация текста"""
        tokens = []
        position = 0
        
        while position < len(text):
            matched = False
            
            for token_type, pattern in token_patterns:
                regex = re.compile(pattern)
                match = regex.match(text, position)
                
                if match:
                    token_value = match.group(0)
                    if token_type != 'WHITESPACE':  # Игнорируем пробелы
                        tokens.append((token_type, token_value))
                    position = match.end()
                    matched = True
                    break
            
            if not matched:
                raise ValueError(f"Неизвестный символ на позиции {position}: '{text[position]}'")
        
        return tokens
    
    # Тестирование
    test_expressions = [
        "x + 42",
        "result = (a + b) * 2",
        "func_name_123 - 456 / (x + y)"
    ]
    
    for expr in test_expressions:
        print(f"\nВыражение: {expr}")
        try:
            tokens = tokenize(expr)
            print("Токены:", tokens)
        except ValueError as e:
            print(f"Ошибка: {e}")

lexical_analyzer_demo()
```

### 2. Проверка синтаксиса

```python
def syntax_validator_demo():
    """Проверка синтаксиса с помощью КС-грамматик"""
    
    class SimpleExpressionValidator:
        """Валидатор простых арифметических выражений"""
        
        def __init__(self):
            # Определяем грамматику: E → T (('+' | '-') T)*
            #                      T → F (('*' | '/') F)*  
            #                      F → '(' E ')' | number | identifier
            pass
        
        def validate(self, tokens):
            """Проверка синтаксической корректности"""
            self.tokens = tokens
            self.position = 0
            
            try:
                self.parse_expression()
                return self.position == len(tokens), "Выражение корректно"
            except Exception as e:
                return False, f"Синтаксическая ошибка: {e}"
        
        def current_token(self):
            """Получение текущего токена"""
            if self.position < len(self.tokens):
                return self.tokens[self.position]
            return None, None
        
        def consume_token(self, expected_type=None):
            """Потребление токена"""
            if self.position >= len(self.tokens):
                raise ValueError("Неожиданный конец выражения")
            
            token_type, token_value = self.tokens[self.position]
            
            if expected_type and token_type != expected_type:
                raise ValueError(f"Ожидался {expected_type}, получен {token_type}")
            
            self.position += 1
            return token_type, token_value
        
        def parse_expression(self):
            """E → T (('+' | '-') T)*"""
            self.parse_term()
            
            while True:
                token_type, _ = self.current_token() or (None, None)
                if token_type in ['PLUS', 'MINUS']:
                    self.consume_token()
                    self.parse_term()
                else:
                    break
        
        def parse_term(self):
            """T → F (('*' | '/') F)*"""
            self.parse_factor()
            
            while True:
                token_type, _ = self.current_token() or (None, None)
                if token_type in ['MULTIPLY', 'DIVIDE']:
                    self.consume_token()
                    self.parse_factor()
                else:
                    break
        
        def parse_factor(self):
            """F → '(' E ')' | NUMBER | IDENTIFIER"""
            token_type, token_value = self.current_token() or (None, None)
            
            if token_type == 'LPAREN':
                self.consume_token('LPAREN')
                self.parse_expression()
                self.consume_token('RPAREN')
            elif token_type in ['NUMBER', 'IDENTIFIER']:
                self.consume_token()
            else:
                raise ValueError(f"Ожидался NUMBER, IDENTIFIER или '(', получен {token_type}")
    
    # Тестирование валидатора
    validator = SimpleExpressionValidator()
    
    # Используем токенизатор из предыдущего примера
    def simple_tokenize(text):
        import re
        pattern = r'(\d+|\w+|[+\-*/()])'
        tokens = []
        for match in re.finditer(pattern, text):
            value = match.group(0)
            if value.isdigit():
                tokens.append(('NUMBER', value))
            elif value.isalpha():
                tokens.append(('IDENTIFIER', value))
            elif value == '+':
                tokens.append(('PLUS', value))
            elif value == '-':
                tokens.append(('MINUS', value))
            elif value == '*':
                tokens.append(('MULTIPLY', value))
            elif value == '/':
                tokens.append(('DIVIDE', value))
            elif value == '(':
                tokens.append(('LPAREN', value))
            elif value == ')':
                tokens.append(('RPAREN', value))
        return tokens
    
    test_cases = [
        ("2 + 3", True),
        ("(a + b) * c", True),
        ("x + y + z", True),
        ("2 + + 3", False),
        ("(a + b", False),
        ("* 2 + 3", False),
    ]
    
    for expression, expected in test_cases:
        tokens = simple_tokenize(expression)
        is_valid, message = validator.validate(tokens)
        status = "✓" if is_valid == expected else "✗"
        print(f"{status} '{expression}': {message}")

syntax_validator_demo()
```

---

## 🔗 Связанные темы

### 📚 Внутренние связи
- **[[Автоматы]]** - модели распознавания формальных языков
- **[[Регулярные выражения]]** - описание регулярных языков  
- **[[Машины Тьюринга]]** - распознавание языков типа 0
- **[[Сложность вычислений]]** - сложность распознавания языков

### 🔧 Практические применения
- **[[Компиляторы и парсеры]]** - синтаксический анализ
- **[[Формальная верификация]]** - спецификация систем
- **[[Криптография]]** - описание протоколов

### 📖 Смежные дисциплины
- **[[Дискретная математика]]** - множества, отношения
- **[[Логика]]** - формальные системы
- **[[Алгоритмы]]** - алгоритмы обработки строк

---

## 🎓 Практические задания

### 📝 Начальный уровень
1. **Определение языков**
   - [ ] Определить язык для email адресов
   - [ ] Построить регулярную грамматику для телефонных номеров
   - [ ] Создать КС-грамматику для JSON объектов

2. **Операции над языками**
   - [ ] Реализовать объединение и пересечение языков
   - [ ] Вычислить замыкание Клини для простого языка
   - [ ] Найти дополнение регулярного языка

### 🔧 Средний уровень
3. **Преобразования грамматик**
   - [ ] Преобразовать регулярную грамматику в ДКА
   - [ ] Устранить левую рекурсию в КС-грамматике
   - [ ] Построить LL(1) парсер для арифметических выражений

4. **Анализ языков**
   - [ ] Доказать, что язык {aⁿbⁿ} не является регулярным
   - [ ] Построить КС-грамматику для сбалансированных скобок
   - [ ] Исследовать неоднозначность грамматики

### 🚀 Продвинутый уровень
5. **Сложные конструкции**
   - [ ] Реализовать LR парсер
   - [ ] Построить грамматику для фрагмента языка программирования
   - [ ] Исследовать пересечение КС языков

---

*Формальные языки и грамматики - это математический фундамент для понимания синтаксиса и семантики языков программирования* 📚🔧 