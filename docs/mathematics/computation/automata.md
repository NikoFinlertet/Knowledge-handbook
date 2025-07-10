# Автоматы

> **Математические модели вычислений**  
> Конечные автоматы, магазинные автоматы и линейно-ограниченные автоматы

[[Теория вычислений]] → **Автоматы**

---

## 🎯 Ключевые концепции

### 🏗️ Иерархия автоматов
- **[[Конечные автоматы]]** → [[Регулярные выражения|регулярные языки]]
- **[[Магазинные автоматы]]** → [[Формальные языки и грамматики|контекстно-свободные языки]]
- **[[Машины Тьюринга]]** → рекурсивно перечислимые языки

### 🔗 Связи с другими темами
- **[[Формальные языки и грамматики]]** - языки, которые распознают
- **[[Регулярные выражения]]** - эквивалентное представление
- **[[Сложность вычислений]]** - временная и пространственная сложность

---

## 🤖 Конечные автоматы

### Детерминированный конечный автомат (DFA)

```python
class DeterministicFiniteAutomaton:
    """Детерминированный конечный автомат"""
    
    def __init__(self, states, alphabet, transitions, start_state, accept_states):
        self.states = states
        self.alphabet = alphabet
        self.transitions = transitions  # dict: (state, symbol) -> state
        self.start_state = start_state
        self.accept_states = accept_states
    
    def process_string(self, input_string):
        """Обработка строки автоматом"""
        current_state = self.start_state
        
        for symbol in input_string:
            if symbol not in self.alphabet:
                return False, f"Symbol '{symbol}' not in alphabet"
            
            if (current_state, symbol) not in self.transitions:
                return False, f"No transition from {current_state} with {symbol}"
            
            current_state = self.transitions[(current_state, symbol)]
        
        accepted = current_state in self.accept_states
        return accepted, current_state

# Пример: автомат для строк с четным количеством нулей
even_zeros_dfa = DeterministicFiniteAutomaton(
    states={'q0', 'q1'},
    alphabet={'0', '1'},
    transitions={
        ('q0', '0'): 'q1',  # четное → нечетное
        ('q0', '1'): 'q0',  # четное остается четным
        ('q1', '0'): 'q0',  # нечетное → четное
        ('q1', '1'): 'q1'   # нечетное остается нечетным
    },
    start_state='q0',      # начинаем с четного количества (0)
    accept_states={'q0'}   # принимаем четное количество
)

# Тестирование
test_cases = ['', '0', '00', '101', '1010', '1100']
for string in test_cases:
    accepted, final_state = even_zeros_dfa.process_string(string)
    zeros = string.count('0')
    print(f"'{string}' (нулей: {zeros}): {'✓' if accepted else '✗'}")
```

### Недетерминированный конечный автомат (NFA)

```python
class NondeterministicFiniteAutomaton:
    """Недетерминированный конечный автомат"""
    
    def __init__(self, states, alphabet, transitions, start_state, accept_states):
        self.states = states
        self.alphabet = alphabet
        self.transitions = transitions  # dict: (state, symbol) -> set of states
        self.start_state = start_state
        self.accept_states = accept_states
    
    def process_string(self, input_string):
        """Обработка строки через симуляцию множества состояний"""
        current_states = {self.start_state}
        
        for symbol in input_string:
            next_states = set()
            for state in current_states:
                if (state, symbol) in self.transitions:
                    next_states.update(self.transitions[(state, symbol)])
            current_states = next_states
            
            if not current_states:
                return False, "No valid transitions"
        
        accepted = bool(current_states & self.accept_states)
        return accepted, current_states

# Пример: NFA для строк, заканчивающихся на "01"
ends_with_01_nfa = NondeterministicFiniteAutomaton(
    states={'q0', 'q1', 'q2'},
    alphabet={'0', '1'},
    transitions={
        ('q0', '0'): {'q0', 'q1'},  # Недетерминизм: "угадываем" начало "01"
        ('q0', '1'): {'q0'},        # Остаемся в начальном состоянии
        ('q1', '1'): {'q2'}         # Завершаем последовательность "01"
    },
    start_state='q0',
    accept_states={'q2'}
)

# Демонстрация
examples = ['01', '101', '001', '10', '11', '0101']
for string in examples:
    accepted, _ = ends_with_01_nfa.process_string(string)
    print(f"'{string}': {'✓' if accepted else '✗'}")
```

### Преобразование NFA в DFA

```python
def nfa_to_dfa(nfa):
    """Алгоритм конструкции подмножеств для преобразования NFA в DFA"""
    
    dfa_states = set()
    dfa_transitions = {}
    dfa_start = frozenset([nfa.start_state])
    dfa_accept = set()
    
    # Очередь для обработки новых состояний DFA
    queue = [dfa_start]
    processed = {dfa_start}
    
    while queue:
        current_subset = queue.pop(0)
        dfa_states.add(current_subset)
        
        # Если подмножество содержит принимающее состояние NFA
        if current_subset & nfa.accept_states:
            dfa_accept.add(current_subset)
        
        # Для каждого символа алфавита
        for symbol in nfa.alphabet:
            next_states = set()
            
            # Собираем все состояния, достижимые по символу
            for state in current_subset:
                if (state, symbol) in nfa.transitions:
                    next_states.update(nfa.transitions[(state, symbol)])
            
            if next_states:
                next_subset = frozenset(next_states)
                dfa_transitions[(current_subset, symbol)] = next_subset
                
                if next_subset not in processed:
                    queue.append(next_subset)
                    processed.add(next_subset)
    
    return DeterministicFiniteAutomaton(
        states=dfa_states,
        alphabet=nfa.alphabet,
        transitions=dfa_transitions,
        start_state=dfa_start,
        accept_states=dfa_accept
    )

# Преобразование примера
equivalent_dfa = nfa_to_dfa(ends_with_01_nfa)
print(f"NFA состояний: {len(ends_with_01_nfa.states)}")
print(f"DFA состояний: {len(equivalent_dfa.states)}")
```

---

## 📚 Магазинные автоматы

### Автомат с магазинной памятью (PDA)

```python
class PushdownAutomaton:
    """Автомат с магазинной памятью для КС-языков"""
    
    def __init__(self, states, alphabet, stack_alphabet, transitions, 
                 start_state, start_stack, accept_states):
        self.states = states
        self.alphabet = alphabet
        self.stack_alphabet = stack_alphabet
        self.transitions = transitions  # (state, input, stack_top) -> [(new_state, stack_action)]
        self.start_state = start_state
        self.start_stack = start_stack
        self.accept_states = accept_states
    
    def process_string(self, input_string):
        """Недетерминированная обработка строки"""
        # Конфигурация: (состояние, позиция, стек)
        configs = [(self.start_state, 0, [self.start_stack])]
        
        while configs:
            state, pos, stack = configs.pop()
            
            # Конец строки
            if pos == len(input_string):
                if state in self.accept_states:
                    return True, stack
                continue
            
            symbol = input_string[pos]
            stack_top = stack[-1] if stack else None
            
            # Поиск возможных переходов
            for (s, inp, st), actions in self.transitions.items():
                if s == state and (inp == symbol or inp == 'ε') and st == stack_top:
                    for new_state, stack_action in actions:
                        new_stack = stack[:]
                        
                        # Выполняем действие со стеком
                        if stack_action.startswith('pop'):
                            if new_stack:
                                new_stack.pop()
                        elif stack_action.startswith('push_'):
                            symbol_to_push = stack_action[5:]
                            new_stack.append(symbol_to_push)
                        
                        new_pos = pos + (1 if inp != 'ε' else 0)
                        configs.append((new_state, new_pos, new_stack))
        
        return False, []

# Пример: PDA для языка {aⁿbⁿ | n ≥ 0}
balanced_ab_pda = PushdownAutomaton(
    states={'q0', 'q1', 'q2'},
    alphabet={'a', 'b'},
    stack_alphabet={'Z', 'A'},  # Z - дно стека, A - счетчик для 'a'
    transitions={
        ('q0', 'a', 'Z'): [('q0', 'push_A')],     # Первое 'a'
        ('q0', 'a', 'A'): [('q0', 'push_A')],     # Последующие 'a'
        ('q0', 'b', 'A'): [('q1', 'pop')],       # Переход к чтению 'b'
        ('q1', 'b', 'A'): [('q1', 'pop')],       # Последующие 'b'
        ('q1', 'ε', 'Z'): [('q2', 'keep')],      # Конец при пустом стеке
    },
    start_state='q0',
    start_stack='Z',
    accept_states={'q2'}
)

# Тестирование
test_strings = ['', 'ab', 'aabb', 'aaabbb', 'aab', 'abb']
for string in test_strings:
    accepted, final_stack = balanced_ab_pda.process_string(string)
    print(f"'{string}': {'✓' if accepted else '✗'}")
```

---

## ⚙️ Минимизация автоматов

### Алгоритм минимизации DFA

```python
def minimize_dfa(dfa):
    """Минимизация DFA методом таблицы различимости"""
    
    states_list = list(dfa.states)
    n = len(states_list)
    
    # Инициализация таблицы различимости
    distinguishable = {}
    for i in range(n):
        for j in range(i+1, n):
            state1, state2 = states_list[i], states_list[j]
            # Различимы, если один принимающий, другой нет
            distinguishable[(state1, state2)] = (
                (state1 in dfa.accept_states) != (state2 in dfa.accept_states)
            )
    
    # Итеративное обновление таблицы
    changed = True
    while changed:
        changed = False
        for i in range(n):
            for j in range(i+1, n):
                state1, state2 = states_list[i], states_list[j]
                
                if distinguishable.get((state1, state2), False):
                    continue
                
                # Проверяем переходы по всем символам
                for symbol in dfa.alphabet:
                    next1 = dfa.transitions.get((state1, symbol))
                    next2 = dfa.transitions.get((state2, symbol))
                    
                    if next1 != next2:
                        # Упорядочиваем для поиска в таблице
                        pair = tuple(sorted([next1, next2]))
                        if distinguishable.get(pair, False):
                            distinguishable[(state1, state2)] = True
                            changed = True
                            break
    
    # Построение классов эквивалентности
    equivalence_classes = []
    processed = set()
    
    for state in states_list:
        if state in processed:
            continue
            
        equivalent_class = {state}
        for other_state in states_list:
            if other_state != state and other_state not in processed:
                pair = tuple(sorted([state, other_state]))
                if not distinguishable.get(pair, False):
                    equivalent_class.add(other_state)
        
        equivalence_classes.append(equivalent_class)
        processed.update(equivalent_class)
    
    # Построение минимизированного автомата
    class_representatives = {frozenset(cls): min(cls) for cls in equivalence_classes}
    
    new_states = set(class_representatives.values())
    new_start = None
    new_accept = set()
    new_transitions = {}
    
    # Определяем новые состояния и переходы
    for cls in equivalence_classes:
        representative = class_representatives[frozenset(cls)]
        
        if dfa.start_state in cls:
            new_start = representative
        
        if any(state in dfa.accept_states for state in cls):
            new_accept.add(representative)
        
        # Переходы (одинаковые для всех состояний в классе)
        state_from_class = next(iter(cls))
        for symbol in dfa.alphabet:
            if (state_from_class, symbol) in dfa.transitions:
                target = dfa.transitions[(state_from_class, symbol)]
                
                # Находим класс целевого состояния
                target_representative = None
                for other_cls in equivalence_classes:
                    if target in other_cls:
                        target_representative = class_representatives[frozenset(other_cls)]
                        break
                
                if target_representative:
                    new_transitions[(representative, symbol)] = target_representative
    
    return DeterministicFiniteAutomaton(
        states=new_states,
        alphabet=dfa.alphabet,
        transitions=new_transitions,
        start_state=new_start,
        accept_states=new_accept
    )

# Демонстрация минимизации
print(f"Исходный DFA: {len(equivalent_dfa.states)} состояний")
minimized = minimize_dfa(equivalent_dfa)
print(f"Минимизированный DFA: {len(minimized.states)} состояний")
```

---

## 🎯 Практические применения

### 1. Лексический анализ

```python
def create_lexer_automaton():
    """Создание автомата для лексического анализа"""
    
    # Комбинированный автомат для различных токенов
    class LexicalAnalyzer:
        def __init__(self):
            self.patterns = [
                ('NUMBER', r'\d+'),
                ('IDENTIFIER', r'[a-zA-Z_][a-zA-Z0-9_]*'),
                ('PLUS', r'\+'),
                ('MINUS', r'-'),
                ('MULTIPLY', r'\*'),
                ('DIVIDE', r'/'),
                ('LPAREN', r'\('),
                ('RPAREN', r'\)')
            ]
        
        def tokenize(self, text):
            """Токенизация с использованием автоматов"""
            import re
            
            tokens = []
            position = 0
            
            while position < len(text):
                if text[position].isspace():
                    position += 1
                    continue
                
                matched = False
                for token_type, pattern in self.patterns:
                    regex = re.compile(pattern)
                    match = regex.match(text, position)
                    
                    if match:
                        tokens.append((token_type, match.group(0)))
                        position = match.end()
                        matched = True
                        break
                
                if not matched:
                    raise ValueError(f"Неизвестный символ: {text[position]}")
            
            return tokens
    
    lexer = LexicalAnalyzer()
    
    test_code = "x + 42 * (y - 3)"
    tokens = lexer.tokenize(test_code)
    
    print("Исходный код:", test_code)
    print("Токены:", tokens)
    
    return lexer

lexer = create_lexer_automaton()
```

### 2. Валидация форматов

```python
def create_validators():
    """Автоматы для валидации различных форматов"""
    
    # Email валидатор (упрощенный)
    email_dfa = DeterministicFiniteAutomaton(
        states={'start', 'local', 'at', 'domain', 'dot', 'accept'},
        alphabet=set('abcdefghijklmnopqrstuvwxyz0123456789@.'),
        transitions={
            # Упрощенная логика для демонстрации
        },
        start_state='start',
        accept_states={'accept'}
    )
    
    # IP адрес валидатор  
    def validate_ip(ip_string):
        """Валидация IPv4 адреса"""
        parts = ip_string.split('.')
        
        if len(parts) != 4:
            return False
        
        for part in parts:
            try:
                num = int(part)
                if not (0 <= num <= 255):
                    return False
                # Проверка ведущих нулей
                if part != str(num):
                    return False
            except ValueError:
                return False
        
        return True
    
    # Тестирование
    test_ips = ['192.168.1.1', '256.1.1.1', '192.168.01.1', '192.168.1']
    for ip in test_ips:
        print(f"{ip}: {'✓' if validate_ip(ip) else '✗'}")

create_validators()
```

---

## 🔗 Связанные темы

### 📚 Теоретические основы
- **[[Формальные языки и грамматики]]** - языки, распознаваемые автоматами
- **[[Регулярные выражения]]** - альтернативная запись регулярных языков
- **[[Машины Тьюринга]]** - наиболее мощная модель автомата

### 🛠️ Практические применения
- **[[Компиляторы и парсеры]]** - лексический и синтаксический анализ
- **[[Формальная верификация]]** - проверка корректности систем
- **[[Анализ алгоритмов]]** - оценка сложности автоматов

### 🧮 Смежные области
- **[[Сложность вычислений]]** - классы сложности для автоматов
- **[[Теория информации]]** - энтропия и сжатие для автоматных языков

---

## 🎓 Практические задания

### 📝 Начальный уровень
1. **Построение автоматов**
   - [ ] DFA для чисел, делящихся на 3
   - [ ] NFA для строк, содержащих подстроку "abc"
   - [ ] PDA для сбалансированных скобок

2. **Преобразования**
   - [ ] NFA → DFA для простого примера
   - [ ] Минимизация DFA
   - [ ] Регулярная грамматика → автомат

### 🔧 Средний уровень
3. **Сложные автоматы**
   - [ ] Автомат для валидации email
   - [ ] PDA для языка {aⁱbʲcⁱ | i,j ≥ 0}
   - [ ] Композиция автоматов

4. **Оптимизация**
   - [ ] Алгоритм минимизации Хопкрофта
   - [ ] Детерминизация с минимальным числом состояний
   - [ ] Анализ сложности автоматов

### 🚀 Продвинутый уровень
5. **Исследования**
   - [ ] Доказательство эквивалентности автоматов
   - [ ] Построение автомата пересечения
   - [ ] Исследование недетерминизма

---

*Автоматы - это элегантные математические модели, лежащие в основе многих практических алгоритмов* 🤖⚙️ 