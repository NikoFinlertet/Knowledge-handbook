# Регулярные выражения

> **Мощный инструмент для работы с текстом**  
> Паттерны, автоматы и практические применения

[[Теория вычислений]] → **Регулярные выражения**

---

## 🎯 Ключевые концепции

### 🔤 Основы регулярных выражений
- **Литералы** - точное совпадение символов
- **Метасимволы** - специальные символы с особым значением
- **Квантификаторы** - указание количества повторений
- **Группировка** - объединение частей паттерна

### 🔗 Связи с другими темами
- **[[Автоматы]]** - эквивалентная модель распознавания
- **[[Формальные языки и грамматики]]** - описание регулярных языков
- **[[Компиляторы и парсеры]]** - лексический анализ

---

## 📝 Основы синтаксиса

### Базовые конструкции

```python
import re

def regex_basics_demo():
    """Демонстрация основных конструкций регулярных выражений"""
    
    test_strings = [
        "abc", "def", "abc123", "123abc", "ABC", "a1b2c3"
    ]
    
    patterns = {
        'Литералы': {
            r'abc': 'Точное совпадение "abc"',
            r'123': 'Точное совпадение "123"'
        },
        
        'Символьные классы': {
            r'[abc]': 'Любой из символов a, b, c',
            r'[a-z]': 'Любая строчная буква',
            r'[A-Z]': 'Любая заглавная буква',
            r'[0-9]': 'Любая цифра',
            r'[^0-9]': 'Любой символ, кроме цифр'
        },
        
        'Предопределенные классы': {
            r'\d': 'Цифра (эквивалент [0-9])',
            r'\w': 'Буквенно-цифровой символ ([a-zA-Z0-9_])',
            r'\s': 'Пробельный символ',
            r'\D': 'Не цифра',
            r'\W': 'Не буквенно-цифровой',
            r'\S': 'Не пробельный'
        },
        
        'Квантификаторы': {
            r'a*': 'Ноль или более "a"',
            r'a+': 'Одно или более "a"',
            r'a?': 'Ноль или одно "a"',
            r'a{3}': 'Ровно 3 "a"',
            r'a{2,4}': 'От 2 до 4 "a"',
            r'a{3,}': '3 или более "a"'
        }
    }
    
    for category, category_patterns in patterns.items():
        print(f"\n=== {category} ===")
        for pattern, description in category_patterns.items():
            print(f"\nПаттерн: {pattern}")
            print(f"Описание: {description}")
            
            for test_string in test_strings:
                matches = re.findall(pattern, test_string)
                if matches:
                    print(f"  '{test_string}' → {matches}")

regex_basics_demo()
```

### Продвинутые конструкции

```python
def advanced_regex_demo():
    """Продвинутые возможности регулярных выражений"""
    
    # Группировка и захват
    email_pattern = r'([a-zA-Z0-9._%+-]+)@([a-zA-Z0-9.-]+\.[a-zA-Z]{2,})'
    emails = ["user@example.com", "test.email+tag@domain.co.uk", "invalid.email"]
    
    print("=== Группировка и захват ===")
    for email in emails:
        match = re.match(email_pattern, email)
        if match:
            username, domain = match.groups()
            print(f"'{email}' → пользователь: '{username}', домен: '{domain}'")
        else:
            print(f"'{email}' → не соответствует паттерну")
    
    # Опережающие и отстающие просмотры
    print("\n=== Просмотры (lookahead/lookbehind) ===")
    
    # Позитивный опережающий просмотр
    password_pattern = r'^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$'
    passwords = ["Passw0rd!", "password", "PASSWORD123", "Weak1!", "StrongP@ssw0rd"]
    
    for pwd in passwords:
        if re.match(password_pattern, pwd):
            print(f"'{pwd}' → ✓ надежный пароль")
        else:
            print(f"'{pwd}' → ✗ слабый пароль")
    
    # Именованные группы
    print("\n=== Именованные группы ===")
    
    date_pattern = r'(?P<day>\d{1,2})/(?P<month>\d{1,2})/(?P<year>\d{4})'
    dates = ["25/12/2023", "1/1/2024", "31/02/2023"]
    
    for date in dates:
        match = re.match(date_pattern, date)
        if match:
            day = match.group('day')
            month = match.group('month')
            year = match.group('year')
            print(f"'{date}' → день: {day}, месяц: {month}, год: {year}")

advanced_regex_demo()
```

---

## 🔄 Связь с автоматами

### Конструкция Томпсона (Regex → NFA)

```python
class NFAState:
    """Состояние NFA для конструкции Томпсона"""
    def __init__(self, state_id):
        self.id = state_id
        self.transitions = {}  # symbol -> [states]
        self.epsilon_transitions = []  # epsilon переходы

class ThompsonConstruction:
    """Алгоритм Томпсона для построения NFA из регулярного выражения"""
    
    def __init__(self):
        self.state_counter = 0
    
    def new_state(self):
        """Создание нового состояния"""
        state = NFAState(self.state_counter)
        self.state_counter += 1
        return state
    
    def char_nfa(self, char):
        """NFA для одного символа"""
        start = self.new_state()
        end = self.new_state()
        
        start.transitions[char] = [end]
        
        return start, end
    
    def concat_nfa(self, nfa1, nfa2):
        """Конкатенация двух NFA"""
        start1, end1 = nfa1
        start2, end2 = nfa2
        
        # Соединяем epsilon-переходом
        end1.epsilon_transitions.append(start2)
        
        return start1, end2
    
    def union_nfa(self, nfa1, nfa2):
        """Объединение двух NFA (операция |)"""
        start1, end1 = nfa1
        start2, end2 = nfa2
        
        new_start = self.new_state()
        new_end = self.new_state()
        
        # Epsilon переходы к началам обеих альтернатив
        new_start.epsilon_transitions.extend([start1, start2])
        
        # Epsilon переходы от концов к общему финальному состоянию
        end1.epsilon_transitions.append(new_end)
        end2.epsilon_transitions.append(new_end)
        
        return new_start, new_end
    
    def star_nfa(self, nfa):
        """Замыкание Клини (операция *)"""
        start, end = nfa
        
        new_start = self.new_state()
        new_end = self.new_state()
        
        # Epsilon переходы для пустой строки и повторений
        new_start.epsilon_transitions.extend([start, new_end])
        end.epsilon_transitions.extend([start, new_end])
        
        return new_start, new_end
    
    def build_nfa(self, regex):
        """Построение NFA из упрощенного регулярного выражения"""
        # Упрощенный парсер для демонстрации
        if len(regex) == 1:
            return self.char_nfa(regex)
        
        # Обработка базовых случаев
        if regex.endswith('*'):
            base_nfa = self.build_nfa(regex[:-1])
            return self.star_nfa(base_nfa)
        
        # Для более сложных случаев нужен полноценный парсер
        return self.char_nfa(regex[0])

# Демонстрация
thompson = ThompsonConstruction()

simple_regexes = ['a', 'a*', 'ab']
for regex in simple_regexes:
    start, end = thompson.build_nfa(regex)
    print(f"NFA для '{regex}': начало {start.id}, конец {end.id}")
```

### Алгоритм подмножеств (NFA → DFA)

```python
def simulate_nfa_on_string(start_state, string):
    """Симуляция NFA на строке"""
    
    def epsilon_closure(states):
        """Вычисление epsilon-замыкания"""
        closure = set(states)
        stack = list(states)
        
        while stack:
            state = stack.pop()
            for next_state in state.epsilon_transitions:
                if next_state not in closure:
                    closure.add(next_state)
                    stack.append(next_state)
        
        return closure
    
    current_states = epsilon_closure([start_state])
    
    for symbol in string:
        next_states = set()
        
        for state in current_states:
            if symbol in state.transitions:
                next_states.update(state.transitions[symbol])
        
        current_states = epsilon_closure(next_states)
        
        if not current_states:
            return False
    
    return len(current_states) > 0

# Пример использования будет работать с полноценной реализацией NFA
```

---

## 🛠️ Практические применения

### 1. Валидация данных

```python
def validation_patterns():
    """Практические паттерны для валидации"""
    
    validators = {
        'Email': r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$',
        'Телефон (RU)': r'^(\+7|8)[\s\-]?\(?\d{3}\)?[\s\-]?\d{3}[\s\-]?\d{2}[\s\-]?\d{2}$',
        'IP адрес': r'^((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$',
        'URL': r'^https?://(?:[-\w.])+(?:\:[0-9]+)?(?:/(?:[\w/_.])*(?:\?(?:[\w&=%.]*))?(?:\#(?:[\w.]*))?)?$',
        'Кредитная карта': r'^(?:4[0-9]{12}(?:[0-9]{3})?|5[1-5][0-9]{14}|3[47][0-9]{13}|3[0-9]{13}|6(?:011|5[0-9]{2})[0-9]{12})$',
        'Пароль': r'^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$'
    }
    
    test_data = {
        'Email': ['user@example.com', 'invalid.email', 'test@domain.co.uk'],
        'Телефон (RU)': ['+7 (123) 456-78-90', '8-123-456-78-90', '123456789'],
        'IP адрес': ['192.168.1.1', '256.1.1.1', '127.0.0.1'],
        'URL': ['https://example.com', 'http://test.site.com/path?param=value', 'not-a-url'],
        'Кредитная карта': ['4111111111111111', '5555555555554444', '123456789'],
        'Пароль': ['StrongP@ssw0rd', 'weak', 'NoSpecial123']
    }
    
    for category, pattern in validators.items():
        print(f"\n=== {category} ===")
        print(f"Паттерн: {pattern}")
        
        for test_value in test_data[category]:
            is_valid = bool(re.match(pattern, test_value))
            status = "✓" if is_valid else "✗"
            print(f"  {status} '{test_value}'")

validation_patterns()
```

### 2. Извлечение данных

```python
def data_extraction_demo():
    """Извлечение структурированных данных из текста"""
    
    # Логи веб-сервера
    log_pattern = r'(\d+\.\d+\.\d+\.\d+) - - \[([^\]]+)\] "(\w+) ([^"]+)" (\d+) (\d+)'
    
    log_lines = [
        '192.168.1.1 - - [10/Oct/2023:13:55:36 +0000] "GET /index.html" 200 2326',
        '10.0.0.1 - - [10/Oct/2023:13:55:37 +0000] "POST /api/data" 404 512',
        '172.16.0.1 - - [10/Oct/2023:13:55:38 +0000] "GET /images/logo.png" 200 15432'
    ]
    
    print("=== Анализ логов веб-сервера ===")
    for line in log_lines:
        match = re.match(log_pattern, line)
        if match:
            ip, timestamp, method, path, status, size = match.groups()
            print(f"IP: {ip}, Метод: {method}, Путь: {path}, Статус: {status}, Размер: {size}")
    
    # Извлечение ссылок из HTML
    html_content = '''
    <html>
        <body>
            <a href="https://example.com">Example</a>
            <a href="mailto:contact@site.com">Contact</a>
            <a href="/internal/page">Internal</a>
        </body>
    </html>
    '''
    
    link_pattern = r'<a\s+href="([^"]+)"[^>]*>([^<]+)</a>'
    
    print("\n=== Извлечение ссылок из HTML ===")
    links = re.findall(link_pattern, html_content)
    for url, text in links:
        print(f"URL: {url}, Текст: {text}")
    
    # Парсинг CSV с кавычками
    csv_line = '"Иванов, Иван","Москва","ivan@email.com","28"'
    csv_pattern = r'"([^"]*)"|([^,]+)'
    
    print("\n=== Парсинг CSV ===")
    fields = [match[0] if match[0] else match[1] for match in re.findall(csv_pattern, csv_line)]
    print(f"Поля: {fields}")

data_extraction_demo()
```

### 3. Замены и трансформации

```python
def text_transformations():
    """Продвинутые замены с использованием регулярных выражений"""
    
    # Форматирование телефонных номеров
    def format_phone(phone):
        # Убираем все кроме цифр
        digits = re.sub(r'[^\d]', '', phone)
        
        # Форматируем российский номер
        if len(digits) == 11 and digits.startswith('8'):
            digits = '7' + digits[1:]
        
        if len(digits) == 11 and digits.startswith('7'):
            return f"+7 ({digits[1:4]}) {digits[4:7]}-{digits[7:9]}-{digits[9:11]}"
        
        return phone  # Возвращаем исходный, если не удалось отформатировать
    
    phones = ['89123456789', '+7(912)345-67-89', '8 912 345 67 89']
    print("=== Форматирование телефонов ===")
    for phone in phones:
        formatted = format_phone(phone)
        print(f"'{phone}' → '{formatted}'")
    
    # Markdown → HTML (упрощенная версия)
    def markdown_to_html(text):
        """Базовая конвертация Markdown в HTML"""
        
        # Заголовки
        text = re.sub(r'^# (.+)$', r'<h1>\1</h1>', text, flags=re.MULTILINE)
        text = re.sub(r'^## (.+)$', r'<h2>\1</h2>', text, flags=re.MULTILINE)
        
        # Жирный текст
        text = re.sub(r'\*\*(.+?)\*\*', r'<strong>\1</strong>', text)
        
        # Курсив
        text = re.sub(r'\*(.+?)\*', r'<em>\1</em>', text)
        
        # Ссылки
        text = re.sub(r'\[([^\]]+)\]\(([^)]+)\)', r'<a href="\2">\1</a>', text)
        
        # Код (инлайн)
        text = re.sub(r'`([^`]+)`', r'<code>\1</code>', text)
        
        return text
    
    markdown_text = """# Заголовок
    ## Подзаголовок
    
    Это **жирный текст** и *курсивный текст*.
    
    Ссылка: [Google](https://google.com)
    
    Код: `print("Hello")`
    """
    
    print("\n=== Markdown → HTML ===")
    html_result = markdown_to_html(markdown_text)
    print("Исходный Markdown:")
    print(markdown_text)
    print("\nРезультат HTML:")
    print(html_result)
    
    # Обфускация email адресов
    def obfuscate_emails(text):
        """Замена email адресов на обфусцированные версии"""
        email_pattern = r'\b([a-zA-Z0-9._%+-]+)@([a-zA-Z0-9.-]+\.[a-zA-Z]{2,})\b'
        
        def replace_email(match):
            username, domain = match.groups()
            return f"{username[0]}***@{domain}"
        
        return re.sub(email_pattern, replace_email, text)
    
    text_with_emails = "Свяжитесь с нами: support@company.com или admin@example.org"
    print("\n=== Обфускация email ===")
    print(f"Исходный текст: {text_with_emails}")
    print(f"Обфусцированный: {obfuscate_emails(text_with_emails)}")

text_transformations()
```

---

## ⚡ Оптимизация производительности

### Компиляция и кэширование

```python
import re
import time

def performance_optimization():
    """Демонстрация оптимизации производительности регулярных выражений"""
    
    # Тестовые данные
    test_strings = [f"user{i}@example{i % 10}.com" for i in range(10000)]
    email_pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    
    # Без компиляции
    start_time = time.time()
    valid_emails_1 = [s for s in test_strings if re.match(email_pattern, s)]
    time_1 = time.time() - start_time
    
    # С компиляцией
    compiled_pattern = re.compile(email_pattern)
    start_time = time.time()
    valid_emails_2 = [s for s in test_strings if compiled_pattern.match(s)]
    time_2 = time.time() - start_time
    
    print("=== Оптимизация производительности ===")
    print(f"Без компиляции: {time_1:.4f} сек")
    print(f"С компиляцией: {time_2:.4f} сек")
    print(f"Ускорение: {time_1/time_2:.2f}x")
    print(f"Найдено email: {len(valid_emails_1)}")
    
    # Избегание катастрофического отката
    print("\n=== Избегание катастрофического отката ===")
    
    # Плохой паттерн (может вызвать экспоненциальное время)
    bad_pattern = r'^(a+)+b$'
    good_pattern = r'^a+b$'
    
    test_string = 'a' * 20 + 'c'  # Строка, которая не соответствует паттерну
    
    # Демонстрация только для понимания (не выполняем плохой паттерн)
    print(f"Плохой паттерн: {bad_pattern}")
    print(f"Хороший паттерн: {good_pattern}")
    print(f"Тестовая строка: {test_string}")
    print("Плохой паттерн может работать экспоненциально долго!")
    
    # Используем хороший паттерн
    start_time = time.time()
    result = re.match(good_pattern, test_string)
    time_good = time.time() - start_time
    print(f"Хороший паттерн: {time_good:.6f} сек, результат: {result}")

performance_optimization()
```

### Альтернативы для сложных случаев

```python
def complex_parsing_alternatives():
    """Альтернативы регулярным выражениям для сложных случаев"""
    
    # Парсинг JSON (лучше использовать специализированный парсер)
    json_like = '{"name": "John", "age": 30, "city": "New York"}'
    
    # Regex подход (плохо для сложного JSON)
    simple_json_pattern = r'"(\w+)": "?([^",}]+)"?'
    regex_pairs = re.findall(simple_json_pattern, json_like)
    
    # Правильный подход
    import json
    parsed_json = json.loads(json_like)
    
    print("=== Парсинг JSON ===")
    print(f"Regex подход: {regex_pairs}")
    print(f"JSON парсер: {parsed_json}")
    
    # Парсинг HTML (лучше использовать HTML парсер)
    html_content = '<div class="content"><p>Hello <strong>world</strong>!</p></div>'
    
    # Regex подход (проблематично для сложного HTML)
    text_pattern = r'>([^<]+)<'
    regex_text = ' '.join(re.findall(text_pattern, html_content))
    
    print("\n=== Парсинг HTML ===")
    print(f"Regex подход: '{regex_text}'")
    print("Рекомендация: используйте BeautifulSoup или lxml для HTML")
    
    # Конечный автомат для сложных состояний
    class SimpleStateMachine:
        """Простой конечный автомат для парсинга"""
        
        def __init__(self):
            self.state = 'start'
            self.tokens = []
        
        def process_char(self, char):
            if self.state == 'start':
                if char.isalpha():
                    self.state = 'identifier'
                    self.current_token = char
                elif char.isdigit():
                    self.state = 'number'
                    self.current_token = char
                elif char in ' \t\n':
                    pass  # Игнорируем пробелы
                else:
                    self.tokens.append(char)
            
            elif self.state == 'identifier':
                if char.isalnum():
                    self.current_token += char
                else:
                    self.tokens.append(('ID', self.current_token))
                    self.state = 'start'
                    self.process_char(char)  # Обрабатываем текущий символ
            
            elif self.state == 'number':
                if char.isdigit():
                    self.current_token += char
                else:
                    self.tokens.append(('NUM', self.current_token))
                    self.state = 'start'
                    self.process_char(char)
        
        def parse(self, text):
            for char in text:
                self.process_char(char)
            
            # Завершаем последний токен
            if hasattr(self, 'current_token'):
                if self.state == 'identifier':
                    self.tokens.append(('ID', self.current_token))
                elif self.state == 'number':
                    self.tokens.append(('NUM', self.current_token))
            
            return self.tokens
    
    # Демонстрация автомата
    sm = SimpleStateMachine()
    code = "var x = 42 + y"
    tokens = sm.parse(code)
    
    print("\n=== Конечный автомат для токенизации ===")
    print(f"Исходный код: {code}")
    print(f"Токены: {tokens}")

complex_parsing_alternatives()
```

---

## 🔗 Связанные темы

### 📚 Теоретические основы
- **[[Автоматы]]** - эквивалентная модель для регулярных языков
- **[[Формальные языки и грамматики]]** - регулярные грамматики
- **[[Сложность вычислений]]** - сложность сопоставления регулярных выражений

### 🛠️ Практические применения
- **[[Компиляторы и парсеры]]** - лексический анализ
- **[[Анализ алгоритмов]]** - алгоритмы поиска в тексте
- **[[Формальная верификация]]** - верификация текстовых форматов

### 🌐 Смежные технологии
- **Grep и sed** - утилиты командной строки
- **Парсеры** - альтернативы для структурированных данных
- **Лексические анализаторы** - генераторы на основе регулярных выражений

---

## 🎓 Практические задания

### 📝 Начальный уровень
1. **Базовые паттерны**
   - [ ] Валидатор номера телефона
   - [ ] Поиск всех чисел в тексте
   - [ ] Замена множественных пробелов на одинарные

2. **Извлечение данных**
   - [ ] Извлечение email адресов из текста
   - [ ] Парсинг простого CSV
   - [ ] Поиск URL в HTML

### 🔧 Средний уровень
3. **Сложные паттерны**
   - [ ] Валидация паролей с требованиями
   - [ ] Парсинг логов веб-сервера
   - [ ] Обфускация персональных данных

4. **Трансформации**
   - [ ] Markdown в HTML конвертер
   - [ ] Форматирование номеров телефонов
   - [ ] Очистка и нормализация текста

### 🚀 Продвинутый уровень
5. **Оптимизация и альтернативы**
   - [ ] Бенчмарк производительности регулярных выражений
   - [ ] Построение автомата из регулярного выражения
   - [ ] Конечный автомат для сложного парсинга

---

*Регулярные выражения - это мощный инструмент, но помните: "Some people, when confronted with a problem, think 'I know, I'll use regular expressions.' Now they have two problems."* 😄🔧 