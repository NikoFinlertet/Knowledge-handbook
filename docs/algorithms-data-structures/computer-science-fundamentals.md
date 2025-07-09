# Computer Science Fundamentals 🖥️

## Содержание
1. [Операционные системы](#операционные-системы)
2. [Сети и протоколы](#сети-и-протоколы)
3. [Компиляторы и интерпретаторы](#компиляторы-и-интерпретаторы)
4. [Теория вычислений](#теория-вычислений)
5. [Криптография](#криптография)
6. [Параллельные вычисления](#параллельные-вычисления)
7. [Электротехника и электроника](./electrical-engineering.md) 🔌

---

## Операционные системы

### Процессы и потоки
```python
# Пример создания процесса в Python
import multiprocessing
import threading
import time

def cpu_bound_task(n):
    """CPU-интенсивная задача - вычисление суммы квадратов"""
    total = 0
    for i in range(n):
        total += i * i
    return total

def io_bound_task(duration):
    """I/O-интенсивная задача - симуляция сетевого запроса"""
    time.sleep(duration)
    return f"Completed after {duration} seconds"

# Многопроцессность для CPU задач
def run_with_processes():
    with multiprocessing.Pool(processes=4) as pool:
        # Распределяем задачи между процессами
        results = pool.map(cpu_bound_task, [1000000, 1000000, 1000000, 1000000])
    return results

# Многопоточность для I/O задач
def run_with_threads():
    threads = []
    results = []
    
    for i in range(4):
        thread = threading.Thread(target=lambda: results.append(io_bound_task(1)))
        threads.append(thread)
        thread.start()
    
    # Ждем завершения всех потоков
    for thread in threads:
        thread.join()
    
    return results
```

### Синхронизация
```go
// Пример работы с мьютексами и каналами в Go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Безопасный счетчик с мьютексом
type SafeCounter struct {
    mu    sync.Mutex
    count int
}

func (c *SafeCounter) Increment() {
    c.mu.Lock()         // Блокируем доступ к счетчику
    defer c.mu.Unlock() // Разблокируем при выходе из функции
    c.count++
}

func (c *SafeCounter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}

// Паттерн Worker Pool с каналами
func workerPool() {
    const numWorkers = 3
    const numJobs = 10
    
    jobs := make(chan int, numJobs)    // Канал для задач
    results := make(chan int, numJobs) // Канал для результатов
    
    // Запускаем воркеров
    for w := 1; w <= numWorkers; w++ {
        go worker(w, jobs, results)
    }
    
    // Отправляем задачи
    for j := 1; j <= numJobs; j++ {
        jobs <- j
    }
    close(jobs)
    
    // Собираем результаты
    for r := 1; r <= numJobs; r++ {
        <-results
    }
}

func worker(id int, jobs <-chan int, results chan<- int) {
    for job := range jobs {
        fmt.Printf("Worker %d processing job %d\n", id, job)
        time.Sleep(time.Second) // Симуляция работы
        results <- job * 2
    }
}
```

### Управление памятью
```python
# Пример различных стратегий управления памятью
class MemoryManager:
    """Демонстрация концепций управления памятью"""
    
    def __init__(self):
        self.heap = [0] * 1000  # Симуляция кучи
        self.free_blocks = [(0, 1000)]  # Список свободных блоков
    
    def allocate(self, size):
        """First Fit алгоритм аллокации"""
        for i, (start, length) in enumerate(self.free_blocks):
            if length >= size:
                # Выделяем блок
                allocated_block = start
                
                # Обновляем список свободных блоков
                if length == size:
                    self.free_blocks.pop(i)
                else:
                    self.free_blocks[i] = (start + size, length - size)
                
                return allocated_block
        return None  # Нет свободного места
    
    def deallocate(self, start, size):
        """Освобождение блока памяти"""
        # Добавляем освобожденный блок в список
        self.free_blocks.append((start, size))
        
        # Сортируем и объединяем соседние блоки
        self.free_blocks.sort()
        merged = []
        current_start, current_size = self.free_blocks[0]
        
        for start, size in self.free_blocks[1:]:
            if current_start + current_size == start:
                # Объединяем блоки
                current_size += size
            else:
                merged.append((current_start, current_size))
                current_start, current_size = start, size
        
        merged.append((current_start, current_size))
        self.free_blocks = merged
```

---

## Сети и протоколы

### TCP/IP Stack
```python
# Пример работы с сокетами - TCP сервер
import socket
import threading

class TCPServer:
    """Простой TCP сервер с обработкой множественных соединений"""
    
    def __init__(self, host='localhost', port=8080):
        self.host = host
        self.port = port
        self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    
    def start(self):
        """Запуск сервера"""
        self.socket.bind((self.host, self.port))
        self.socket.listen(5)  # Очередь из 5 соединений
        print(f"Server listening on {self.host}:{self.port}")
        
        while True:
            client_socket, address = self.socket.accept()
            print(f"Connection from {address}")
            
            # Обрабатываем каждое соединение в отдельном потоке
            client_thread = threading.Thread(
                target=self.handle_client, 
                args=(client_socket, address)
            )
            client_thread.start()
    
    def handle_client(self, client_socket, address):
        """Обработка клиентского соединения"""
        try:
            while True:
                data = client_socket.recv(1024)  # Читаем данные
                if not data:
                    break
                
                # Эхо-сервер - отправляем данные обратно
                response = f"Echo: {data.decode('utf-8')}"
                client_socket.send(response.encode('utf-8'))
                
        except Exception as e:
            print(f"Error handling client {address}: {e}")
        finally:
            client_socket.close()
            print(f"Connection with {address} closed")
```

### HTTP протокол
```python
# Простой HTTP сервер
import http.server
import socketserver
import json
from urllib.parse import urlparse, parse_qs

class HTTPHandler(http.server.SimpleHTTPRequestHandler):
    """Кастомный HTTP обработчик"""
    
    def do_GET(self):
        """Обработка GET запросов"""
        parsed_path = urlparse(self.path)
        
        if parsed_path.path == '/api/health':
            self.send_json_response({'status': 'healthy'})
        elif parsed_path.path == '/api/data':
            # Обработка query параметров
            params = parse_qs(parsed_path.query)
            response = {
                'data': 'sample data',
                'params': params
            }
            self.send_json_response(response)
        else:
            # Стандартная обработка статических файлов
            super().do_GET()
    
    def do_POST(self):
        """Обработка POST запросов"""
        content_length = int(self.headers['Content-Length'])
        post_data = self.rfile.read(content_length)
        
        try:
            json_data = json.loads(post_data.decode('utf-8'))
            response = {
                'message': 'Data received',
                'received_data': json_data
            }
            self.send_json_response(response)
        except json.JSONDecodeError:
            self.send_error(400, 'Invalid JSON')
    
    def send_json_response(self, data):
        """Отправка JSON ответа"""
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(data).encode('utf-8'))
```

---

## Компиляторы и интерпретаторы

### Лексический анализ
```python
# Простой лексический анализатор
import re
from enum import Enum

class TokenType(Enum):
    """Типы токенов"""
    NUMBER = 'NUMBER'
    IDENTIFIER = 'IDENTIFIER'
    PLUS = 'PLUS'
    MINUS = 'MINUS'
    MULTIPLY = 'MULTIPLY'
    DIVIDE = 'DIVIDE'
    LPAREN = 'LPAREN'
    RPAREN = 'RPAREN'
    ASSIGN = 'ASSIGN'
    EOF = 'EOF'

class Token:
    """Токен - минимальная единица языка"""
    def __init__(self, type_, value, line, column):
        self.type = type_
        self.value = value
        self.line = line
        self.column = column
    
    def __repr__(self):
        return f'Token({self.type}, {self.value}, {self.line}:{self.column})'

class Lexer:
    """Лексический анализатор"""
    
    def __init__(self, text):
        self.text = text
        self.pos = 0
        self.line = 1
        self.column = 1
        
        # Регулярные выражения для токенов
        self.token_patterns = [
            ('NUMBER', r'\d+(\.\d+)?'),
            ('IDENTIFIER', r'[a-zA-Z_][a-zA-Z0-9_]*'),
            ('ASSIGN', r'='),
            ('PLUS', r'\+'),
            ('MINUS', r'-'),
            ('MULTIPLY', r'\*'),
            ('DIVIDE', r'/'),
            ('LPAREN', r'\('),
            ('RPAREN', r'\)'),
            ('WHITESPACE', r'\s+'),
            ('NEWLINE', r'\n'),
        ]
    
    def tokenize(self):
        """Преобразование текста в последовательность токенов"""
        tokens = []
        
        while self.pos < len(self.text):
            match_found = False
            
            for token_type, pattern in self.token_patterns:
                regex = re.compile(pattern)
                match = regex.match(self.text, self.pos)
                
                if match:
                    value = match.group(0)
                    
                    if token_type == 'WHITESPACE':
                        # Пропускаем пробелы
                        self.column += len(value)
                    elif token_type == 'NEWLINE':
                        # Обновляем позицию строки
                        self.line += 1
                        self.column = 1
                    else:
                        # Создаем токен
                        token = Token(
                            TokenType(token_type), 
                            value, 
                            self.line, 
                            self.column
                        )
                        tokens.append(token)
                        self.column += len(value)
                    
                    self.pos = match.end()
                    match_found = True
                    break
            
            if not match_found:
                raise SyntaxError(f"Unexpected character '{self.text[self.pos]}' at {self.line}:{self.column}")
        
        tokens.append(Token(TokenType.EOF, None, self.line, self.column))
        return tokens
```

### Синтаксический анализ
```python
# Простой парсер для арифметических выражений
class ASTNode:
    """Базовый узел абстрактного синтаксического дерева"""
    pass

class NumberNode(ASTNode):
    """Узел для чисел"""
    def __init__(self, value):
        self.value = float(value)

class BinaryOpNode(ASTNode):
    """Узел для бинарных операций"""
    def __init__(self, left, operator, right):
        self.left = left
        self.operator = operator
        self.right = right

class UnaryOpNode(ASTNode):
    """Узел для унарных операций"""
    def __init__(self, operator, operand):
        self.operator = operator
        self.operand = operand

class Parser:
    """Синтаксический анализатор"""
    
    def __init__(self, tokens):
        self.tokens = tokens
        self.pos = 0
        self.current_token = self.tokens[0] if tokens else None
    
    def consume(self, expected_type):
        """Потребляем токен ожидаемого типа"""
        if self.current_token.type == expected_type:
            self.current_token = self.tokens[self.pos + 1] if self.pos + 1 < len(self.tokens) else None
            self.pos += 1
        else:
            raise SyntaxError(f"Expected {expected_type}, got {self.current_token.type}")
    
    def parse(self):
        """Парсим выражение"""
        return self.expression()
    
    def expression(self):
        """Выражение: term (('+' | '-') term)*"""
        node = self.term()
        
        while self.current_token and self.current_token.type in [TokenType.PLUS, TokenType.MINUS]:
            operator = self.current_token
            self.consume(operator.type)
            right = self.term()
            node = BinaryOpNode(node, operator, right)
        
        return node
    
    def term(self):
        """Терм: factor (('*' | '/') factor)*"""
        node = self.factor()
        
        while self.current_token and self.current_token.type in [TokenType.MULTIPLY, TokenType.DIVIDE]:
            operator = self.current_token
            self.consume(operator.type)
            right = self.factor()
            node = BinaryOpNode(node, operator, right)
        
        return node
    
    def factor(self):
        """Фактор: ('+' | '-') factor | NUMBER | '(' expression ')'"""
        token = self.current_token
        
        if token.type in [TokenType.PLUS, TokenType.MINUS]:
            self.consume(token.type)
            return UnaryOpNode(token, self.factor())
        
        elif token.type == TokenType.NUMBER:
            self.consume(TokenType.NUMBER)
            return NumberNode(token.value)
        
        elif token.type == TokenType.LPAREN:
            self.consume(TokenType.LPAREN)
            node = self.expression()
            self.consume(TokenType.RPAREN)
            return node
        
        else:
            raise SyntaxError(f"Unexpected token: {token.type}")
```

---

## Теория вычислений

### Конечные автоматы
```python
# Пример конечного автомата для распознавания строк
class FiniteAutomaton:
    """Детерминированный конечный автомат"""
    
    def __init__(self, states, alphabet, transitions, initial_state, accepting_states):
        self.states = states
        self.alphabet = alphabet
        self.transitions = transitions  # dict: (state, symbol) -> new_state
        self.initial_state = initial_state
        self.accepting_states = accepting_states
    
    def run(self, input_string):
        """Запуск автомата на входной строке"""
        current_state = self.initial_state
        
        for symbol in input_string:
            if symbol not in self.alphabet:
                return False
            
            transition_key = (current_state, symbol)
            if transition_key not in self.transitions:
                return False
            
            current_state = self.transitions[transition_key]
        
        return current_state in self.accepting_states

# Пример: автомат для строк, оканчивающихся на "01"
def create_binary_string_automaton():
    """Создает автомат для строк, заканчивающихся на '01'"""
    states = {'q0', 'q1', 'q2'}
    alphabet = {'0', '1'}
    transitions = {
        ('q0', '0'): 'q1',  # Начальное состояние, видим '0'
        ('q0', '1'): 'q0',  # Остаемся в начальном состоянии
        ('q1', '0'): 'q1',  # Видим еще один '0'
        ('q1', '1'): 'q2',  # Видим '1' после '0' - переходим в принимающее состояние
        ('q2', '0'): 'q1',  # Видим '0' - возможное начало новой последовательности
        ('q2', '1'): 'q0',  # Видим '1' - сбрасываемся
    }
    initial_state = 'q0'
    accepting_states = {'q2'}
    
    return FiniteAutomaton(states, alphabet, transitions, initial_state, accepting_states)
```

### Машина Тьюринга
```python
# Упрощенная модель машины Тьюринга
class TuringMachine:
    """Упрощенная реализация машины Тьюринга"""
    
    def __init__(self, states, input_alphabet, tape_alphabet, transitions, initial_state, accepting_states):
        self.states = states
        self.input_alphabet = input_alphabet
        self.tape_alphabet = tape_alphabet
        self.transitions = transitions  # dict: (state, symbol) -> (new_state, new_symbol, direction)
        self.initial_state = initial_state
        self.accepting_states = accepting_states
        
        self.tape = []
        self.head_position = 0
        self.current_state = initial_state
    
    def initialize_tape(self, input_string):
        """Инициализация ленты входной строкой"""
        self.tape = list(input_string) + ['_'] * 100  # Добавляем пустые символы
        self.head_position = 0
        self.current_state = self.initial_state
    
    def step(self):
        """Один шаг выполнения машины"""
        if self.current_state in self.accepting_states:
            return False  # Машина остановилась
        
        current_symbol = self.tape[self.head_position]
        transition_key = (self.current_state, current_symbol)
        
        if transition_key not in self.transitions:
            return False  # Нет перехода - машина зависла
        
        new_state, new_symbol, direction = self.transitions[transition_key]
        
        # Обновляем состояние машины
        self.current_state = new_state
        self.tape[self.head_position] = new_symbol
        
        # Двигаем головку
        if direction == 'L':
            self.head_position = max(0, self.head_position - 1)
        elif direction == 'R':
            self.head_position = min(len(self.tape) - 1, self.head_position + 1)
        
        return True
    
    def run(self, input_string, max_steps=1000):
        """Запуск машины на входной строке"""
        self.initialize_tape(input_string)
        
        for step in range(max_steps):
            if not self.step():
                break
        
        return self.current_state in self.accepting_states
```

---

## Криптография

### Симметричное шифрование
```python
# Простая реализация шифра Цезаря и более сложных алгоритмов
class CaesarCipher:
    """Шифр Цезаря - простейший пример симметричного шифрования"""
    
    def __init__(self, shift):
        self.shift = shift
    
    def encrypt(self, plaintext):
        """Шифрование текста"""
        result = []
        for char in plaintext:
            if char.isalpha():
                # Определяем базу (A или a)
                base = ord('A') if char.isupper() else ord('a')
                # Применяем сдвиг с учетом цикличности
                shifted = (ord(char) - base + self.shift) % 26
                result.append(chr(shifted + base))
            else:
                result.append(char)
        return ''.join(result)
    
    def decrypt(self, ciphertext):
        """Расшифрование текста"""
        # Расшифрование = шифрование с обратным сдвигом
        decrypt_cipher = CaesarCipher(-self.shift)
        return decrypt_cipher.encrypt(ciphertext)

# Более сложный пример - AES подобный алгоритм
class SimpleAES:
    """Упрощенная версия AES для демонстрации"""
    
    def __init__(self, key):
        self.key = key
        self.rounds = 10
    
    def _xor_bytes(self, a, b):
        """XOR двух байт-массивов"""
        return bytes(x ^ y for x, y in zip(a, b))
    
    def _substitute_bytes(self, data, s_box):
        """Подстановка байтов через S-box"""
        return bytes(s_box[b] for b in data)
    
    def _shift_rows(self, state):
        """Сдвиг строк в матрице состояния"""
        # Упрощенная версия для демонстрации
        return state[1:] + state[:1]
    
    def encrypt_block(self, plaintext_block):
        """Шифрование одного блока"""
        state = bytearray(plaintext_block)
        
        # Начальный раунд
        state = self._xor_bytes(state, self.key)
        
        # Основные раунды
        for round_num in range(self.rounds):
            # Упрощенные операции AES
            state = self._shift_rows(state)
            # В реальном AES здесь были бы SubBytes, ShiftRows, MixColumns, AddRoundKey
            state = self._xor_bytes(state, self.key)
        
        return bytes(state)
```

### Асимметричное шифрование
```python
# Простая реализация RSA
import random

class SimpleRSA:
    """Упрощенная реализация RSA для демонстрации принципов"""
    
    def __init__(self, key_size=8):
        self.key_size = key_size
        self.public_key = None
        self.private_key = None
    
    def _is_prime(self, n):
        """Проверка на простоту числа"""
        if n < 2:
            return False
        for i in range(2, int(n**0.5) + 1):
            if n % i == 0:
                return False
        return True
    
    def _generate_prime(self, bits):
        """Генерация простого числа"""
        while True:
            candidate = random.getrandbits(bits)
            if self._is_prime(candidate):
                return candidate
    
    def _gcd(self, a, b):
        """Наибольший общий делитель"""
        while b:
            a, b = b, a % b
        return a
    
    def _mod_inverse(self, a, m):
        """Модульное обращение"""
        def extended_gcd(a, b):
            if a == 0:
                return b, 0, 1
            gcd, x1, y1 = extended_gcd(b % a, a)
            x = y1 - (b // a) * x1
            y = x1
            return gcd, x, y
        
        gcd, x, _ = extended_gcd(a, m)
        if gcd != 1:
            raise ValueError("Modular inverse does not exist")
        return x % m
    
    def generate_keypair(self):
        """Генерация пары ключей"""
        # Генерируем два простых числа
        p = self._generate_prime(self.key_size // 2)
        q = self._generate_prime(self.key_size // 2)
        
        # Вычисляем n и phi(n)
        n = p * q
        phi_n = (p - 1) * (q - 1)
        
        # Выбираем e (обычно 65537)
        e = 65537
        while self._gcd(e, phi_n) != 1:
            e += 2
        
        # Вычисляем d
        d = self._mod_inverse(e, phi_n)
        
        self.public_key = (n, e)
        self.private_key = (n, d)
        
        return self.public_key, self.private_key
    
    def encrypt(self, message, public_key):
        """Шифрование сообщения"""
        n, e = public_key
        # Преобразуем сообщение в число
        m = int.from_bytes(message.encode(), 'big')
        if m >= n:
            raise ValueError("Message too long for key size")
        
        # Шифруем: c = m^e mod n
        ciphertext = pow(m, e, n)
        return ciphertext
    
    def decrypt(self, ciphertext, private_key):
        """Расшифрование сообщения"""
        n, d = private_key
        # Расшифровываем: m = c^d mod n
        m = pow(ciphertext, d, n)
        
        # Преобразуем число обратно в строку
        message_bytes = m.to_bytes((m.bit_length() + 7) // 8, 'big')
        return message_bytes.decode()
```

---

## Параллельные вычисления

### MapReduce Framework
```python
# Простая реализация MapReduce
from multiprocessing import Pool
from functools import reduce
from collections import defaultdict

class MapReduceFramework:
    """Простая реализация MapReduce паттерна"""
    
    def __init__(self, num_workers=4):
        self.num_workers = num_workers
    
    def map_reduce(self, data, mapper, reducer):
        """Основная функция MapReduce"""
        # Map фаза
        with Pool(processes=self.num_workers) as pool:
            mapped_data = pool.map(mapper, data)
        
        # Shuffle фаза - группировка по ключам
        shuffled = defaultdict(list)
        for key_value_pairs in mapped_data:
            for key, value in key_value_pairs:
                shuffled[key].append(value)
        
        # Reduce фаза
        with Pool(processes=self.num_workers) as pool:
            reduced_data = pool.starmap(reducer, shuffled.items())
        
        return dict(reduced_data)

# Пример использования: подсчет слов в тексте
def word_count_mapper(text):
    """Mapper для подсчета слов"""
    words = text.lower().split()
    return [(word, 1) for word in words if word.isalpha()]

def word_count_reducer(word, counts):
    """Reducer для подсчета слов"""
    return (word, sum(counts))

# Пример использования
def demonstrate_mapreduce():
    texts = [
        "hello world hello",
        "world of programming",
        "hello programming world",
        "python programming language"
    ]
    
    framework = MapReduceFramework(num_workers=2)
    word_counts = framework.map_reduce(
        texts, 
        word_count_mapper, 
        word_count_reducer
    )
    
    print("Word counts:", word_counts)
    # Результат: {'hello': 3, 'world': 3, 'programming': 2, 'of': 1, 'python': 1, 'language': 1}
```

### GPU Computing (концептуальный пример)
```python
# Концептуальный пример параллельных вычислений на GPU
import numpy as np
from typing import List, Callable

class ParallelComputing:
    """Демонстрация концепций параллельных вычислений"""
    
    @staticmethod
    def parallel_matrix_multiply(A, B, num_threads=4):
        """Параллельное умножение матриц"""
        def multiply_row(args):
            i, A_row, B = args
            return np.dot(A_row, B)
        
        # Разделяем работу между потоками
        with Pool(processes=num_threads) as pool:
            args = [(i, A[i], B) for i in range(len(A))]
            result_rows = pool.map(multiply_row, args)
        
        return np.array(result_rows)
    
    @staticmethod
    def parallel_reduce(data: List, operation: Callable, num_threads=4):
        """Параллельная редукция"""
        if len(data) <= 1:
            return data[0] if data else None
        
        # Разделяем данные на части
        chunk_size = len(data) // num_threads
        chunks = [data[i:i + chunk_size] for i in range(0, len(data), chunk_size)]
        
        # Параллельно обрабатываем каждую часть
        with Pool(processes=num_threads) as pool:
            partial_results = pool.map(
                lambda chunk: reduce(operation, chunk), 
                chunks
            )
        
        # Объединяем результаты
        return reduce(operation, partial_results)
    
    @staticmethod
    def parallel_scan(data: List, operation: Callable):
        """Параллельное сканирование (prefix sum)"""
        n = len(data)
        if n <= 1:
            return data
        
        # Фаза "вверх" - вычисляем частичные суммы
        levels = []
        current_level = data[:]
        
        while len(current_level) > 1:
            levels.append(current_level)
            next_level = []
            
            for i in range(0, len(current_level), 2):
                if i + 1 < len(current_level):
                    next_level.append(operation(current_level[i], current_level[i + 1]))
                else:
                    next_level.append(current_level[i])
            
            current_level = next_level
        
        # Фаза "вниз" - распространяем результаты
        result = [0] * n
        result[0] = data[0]
        
        for i in range(1, n):
            result[i] = operation(result[i - 1], data[i])
        
        return result
```

## Практическое применение

### Реальные примеры использования
```python
# Пример системы с применением изученных концепций
class DistributedSystem:
    """Пример распределенной системы"""
    
    def __init__(self):
        self.nodes = {}
        self.load_balancer = LoadBalancer()
        self.message_queue = MessageQueue()
    
    def process_big_data(self, data):
        """Обработка больших данных с использованием MapReduce"""
        # Разделяем данные между узлами
        chunks = self.partition_data(data)
        
        # Обрабатываем параллельно
        results = []
        for chunk in chunks:
            node = self.load_balancer.get_next_node()
            result = node.process(chunk)
            results.append(result)
        
        # Объединяем результаты
        return self.merge_results(results)
    
    def secure_communication(self, message, recipient):
        """Безопасная передача сообщений"""
        # Шифруем сообщение
        encrypted = self.encrypt_message(message, recipient.public_key)
        
        # Отправляем через защищенный канал
        self.message_queue.send(encrypted, recipient)
        
        return True
```

## Связь с другими разделами

- **[Алгоритмы](./algorithms.md)** - Реализация алгоритмов для системного программирования
- **[Структуры данных](./data-structures.md)** - Структуры данных в операционных системах
- **[AI/ML](./machine-learning-ai.md)** - Параллельные вычисления для ML
- **[Практика](./practice-problems.md)** - Задачи по системному программированию

---

*Следующий раздел: [Machine Learning и AI →](./machine-learning-ai.md)* 