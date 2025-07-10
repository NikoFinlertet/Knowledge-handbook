# Дискретная математика: Продвинутый курс

## 🎯 Цель курса

Углубленное изучение дискретной математики с практическими применениями в алгоритмах, криптографии и теории вычислений.

## 📚 Содержание

1. [Теория множеств и логика](#теория-множеств-и-логика)
2. [Теория графов](#теория-графов)
3. [Комбинаторика](#комбинаторика)
4. [Теория чисел](#теория-чисел)
5. [Алгебраические структуры](#алгебраические-структуры)
6. [Автоматы и формальные языки](#автоматы-и-формальные-языки)

---

## Теория множеств и логика

### Основные операции над множествами

**Основные множественные операции**:
- **Объединение**: $A \cup B = \{x : x \in A \lor x \in B\}$
- **Пересечение**: $A \cap B = \{x : x \in A \land x \in B\}$
- **Разность**: $A \setminus B = \{x : x \in A \land x \notin B\}$
- **Симметрическая разность**: $A \triangle B = (A \setminus B) \cup (B \setminus A)$
- **Дополнение**: $\bar{A} = U \setminus A$ (где $U$ — универсальное множество)

**Законы де Моргана**:
$$\overline{A \cup B} = \bar{A} \cap \bar{B}$$
$$\overline{A \cap B} = \bar{A} \cup \bar{B}$$

**Декартово произведение**: $A \times B = \{(a, b) : a \in A, b \in B\}$

**Мощность множества**: $|A \times B| = |A| \cdot |B|$

**Булеан (множество всех подмножеств)**: $\mathcal{P}(A) = \{X : X \subseteq A\}$

Если $|A| = n$, то $|\mathcal{P}(A)| = 2^n$

### Отношения и функции

**Бинарное отношение**: $R \subseteq A \times B$

**Свойства отношений на множестве $A$**:
- **Рефлексивность**: $\forall a \in A: (a, a) \in R$
- **Симметричность**: $\forall a, b \in A: (a, b) \in R \Rightarrow (b, a) \in R$
- **Транзитивность**: $\forall a, b, c \in A: (a, b) \in R \land (b, c) \in R \Rightarrow (a, c) \in R$

**Отношение эквивалентности**: рефлексивное, симметричное и транзитивное отношение

**Частичный порядок**: рефлексивное, антисимметричное и транзитивное отношение

### Операции над множествами

```python
class Set:
    """Реализация множества с основными операциями"""
    
    def __init__(self, elements=None):
        self.elements = set(elements) if elements else set()
    
    def union(self, other):
        """Объединение множеств A ∪ B"""
        return Set(self.elements | other.elements)
    
    def intersection(self, other):
        """Пересечение множеств A ∩ B"""
        return Set(self.elements & other.elements)
    
    def difference(self, other):
        """Разность множеств A \ B"""
        return Set(self.elements - other.elements)
    
    def symmetric_difference(self, other):
        """Симметрическая разность A ⊕ B"""
        return Set(self.elements ^ other.elements)
    
    def cartesian_product(self, other):
        """Декартово произведение A × B"""
        return Set([(a, b) for a in self.elements for b in other.elements])
    
    def power_set(self):
        """Булеан множества (множество всех подмножеств)"""
        from itertools import combinations
        
        subsets = []
        for r in range(len(self.elements) + 1):
            for subset in combinations(self.elements, r):
                subsets.append(Set(subset))
        
        return subsets
    
    def is_subset(self, other):
        """Проверка включения A ⊆ B"""
        return self.elements <= other.elements
    
    def __repr__(self):
        return f"Set({sorted(list(self.elements))})"

# Пример: анализ данных с помощью множеств
def analyze_user_behavior():
    """Анализ поведения пользователей через множества"""
    
    users_page_a = Set([1, 2, 3, 4, 5, 6])
    users_page_b = Set([4, 5, 6, 7, 8, 9])
    users_purchased = Set([2, 4, 6, 8, 10])
    
    # Пользователи, посетившие обе страницы
    both_pages = users_page_a.intersection(users_page_b)
    print(f"Посетили обе страницы: {both_pages}")
    
    # Пользователи, посетившие только одну страницу
    only_a = users_page_a.difference(users_page_b)
    only_b = users_page_b.difference(users_page_a)
    print(f"Только страница A: {only_a}")
    print(f"Только страница B: {only_b}")
    
    # Конверсия пользователей
    conversion_a = users_page_a.intersection(users_purchased)
    conversion_b = users_page_b.intersection(users_purchased)
    
    print(f"Конверсия страницы A: {len(conversion_a.elements)}/{len(users_page_a.elements)}")
    print(f"Конверсия страницы B: {len(conversion_b.elements)}/{len(users_page_b.elements)}")

analyze_user_behavior()
```

### Логические операции

```python
class PropositionalLogic:
    """Класс для работы с логикой высказываний"""
    
    def __init__(self):
        self.truth_table = {}
    
    def evaluate(self, expression, variables):
        """Вычисление истинности логического выражения"""
        # Заменяем переменные на значения
        for var, value in variables.items():
            expression = expression.replace(var, str(value))
        
        # Заменяем логические операторы
        expression = expression.replace('AND', ' and ')
        expression = expression.replace('OR', ' or ')
        expression = expression.replace('NOT', ' not ')
        expression = expression.replace('XOR', ' != ')
        expression = expression.replace('IMPLIES', ' <= ')  # A -> B = not A or B
        
        return eval(expression)
    
    def generate_truth_table(self, expression, variables):
        """Генерация таблицы истинности"""
        from itertools import product
        
        table = []
        for values in product([True, False], repeat=len(variables)):
            var_assignment = dict(zip(variables, values))
            result = self.evaluate(expression, var_assignment)
            table.append((*values, result))
        
        return table
    
    def is_tautology(self, expression, variables):
        """Проверка на тавтологию"""
        table = self.generate_truth_table(expression, variables)
        return all(row[-1] for row in table)
    
    def is_contradiction(self, expression, variables):
        """Проверка на противоречие"""
        table = self.generate_truth_table(expression, variables)
        return not any(row[-1] for row in table)
    
    def logical_equivalence(self, expr1, expr2, variables):
        """Проверка логической эквивалентности"""
        table1 = self.generate_truth_table(expr1, variables)
        table2 = self.generate_truth_table(expr2, variables)
        
        return all(row1[-1] == row2[-1] for row1, row2 in zip(table1, table2))

# Пример: проверка логических законов
logic = PropositionalLogic()

# Закон де Моргана: NOT(A AND B) ≡ (NOT A) OR (NOT B)
expr1 = "NOT(A AND B)"
expr2 = "(NOT A) OR (NOT B)"
variables = ['A', 'B']

equivalent = logic.logical_equivalence(expr1, expr2, variables)
print(f"Закон де Моргана: {equivalent}")

# Таблица истинности для импликации
impl_table = logic.generate_truth_table("A IMPLIES B", ['A', 'B'])
print("Таблица истинности для A → B:")
for row in impl_table:
    print(f"A={row[0]}, B={row[1]} → {row[2]}")
```

---

## Теория графов

### Основные алгоритмы

```python
import heapq
from collections import defaultdict, deque

class Graph:
    """Класс для работы с графами"""
    
    def __init__(self, directed=False):
        self.graph = defaultdict(list)
        self.directed = directed
        self.weights = {}
    
    def add_edge(self, u, v, weight=1):
        """Добавление ребра"""
        self.graph[u].append(v)
        self.weights[(u, v)] = weight
        
        if not self.directed:
            self.graph[v].append(u)
            self.weights[(v, u)] = weight
    
    def bfs(self, start):
        """Поиск в ширину"""
        visited = set()
        queue = deque([start])
        result = []
        
        while queue:
            vertex = queue.popleft()
            if vertex not in visited:
                visited.add(vertex)
                result.append(vertex)
                
                for neighbor in self.graph[vertex]:
                    if neighbor not in visited:
                        queue.append(neighbor)
        
        return result
    
    def dfs(self, start, visited=None):
        """Поиск в глубину"""
        if visited is None:
            visited = set()
        
        visited.add(start)
        result = [start]
        
        for neighbor in self.graph[start]:
            if neighbor not in visited:
                result.extend(self.dfs(neighbor, visited))
        
        return result
    
    def dijkstra(self, start):
        """Алгоритм Дейкстры для кратчайших путей"""
        distances = defaultdict(lambda: float('inf'))
        distances[start] = 0
        pq = [(0, start)]
        visited = set()
        
        while pq:
            current_distance, current = heapq.heappop(pq)
            
            if current in visited:
                continue
                
            visited.add(current)
            
            for neighbor in self.graph[current]:
                distance = current_distance + self.weights.get((current, neighbor), 1)
                
                if distance < distances[neighbor]:
                    distances[neighbor] = distance
                    heapq.heappush(pq, (distance, neighbor))
        
        return dict(distances)
    
    def topological_sort(self):
        """Топологическая сортировка"""
        in_degree = defaultdict(int)
        
        # Подсчитываем входящие степени
        for vertex in self.graph:
            for neighbor in self.graph[vertex]:
                in_degree[neighbor] += 1
        
        # Находим вершины с нулевой входящей степенью
        queue = deque([v for v in self.graph if in_degree[v] == 0])
        result = []
        
        while queue:
            vertex = queue.popleft()
            result.append(vertex)
            
            for neighbor in self.graph[vertex]:
                in_degree[neighbor] -= 1
                if in_degree[neighbor] == 0:
                    queue.append(neighbor)
        
        return result if len(result) == len(self.graph) else None
    
    def has_cycle(self):
        """Проверка наличия цикла"""
        color = {}  # 0: белый, 1: серый, 2: черный
        
        def dfs_cycle(vertex):
            color[vertex] = 1  # Серый
            
            for neighbor in self.graph[vertex]:
                if neighbor not in color:
                    if dfs_cycle(neighbor):
                        return True
                elif color[neighbor] == 1:  # Обратное ребро
                    return True
            
            color[vertex] = 2  # Черный
            return False
        
        for vertex in self.graph:
            if vertex not in color:
                if dfs_cycle(vertex):
                    return True
        
        return False
    
    def strongly_connected_components(self):
        """Поиск сильно связных компонент (алгоритм Косарайю)"""
        # Первый DFS для получения времен завершения
        visited = set()
        finish_order = []
        
        def dfs1(v):
            visited.add(v)
            for neighbor in self.graph[v]:
                if neighbor not in visited:
                    dfs1(neighbor)
            finish_order.append(v)
        
        for vertex in self.graph:
            if vertex not in visited:
                dfs1(vertex)
        
        # Создаем транспонированный граф
        transposed = Graph(directed=True)
        for vertex in self.graph:
            for neighbor in self.graph[vertex]:
                transposed.add_edge(neighbor, vertex)
        
        # Второй DFS в обратном порядке времен завершения
        visited = set()
        components = []
        
        def dfs2(v, component):
            visited.add(v)
            component.append(v)
            for neighbor in transposed.graph[v]:
                if neighbor not in visited:
                    dfs2(neighbor, component)
        
        for vertex in reversed(finish_order):
            if vertex not in visited:
                component = []
                dfs2(vertex, component)
                components.append(component)
        
        return components

# Пример: анализ социальной сети
def social_network_analysis():
    """Анализ социальной сети"""
    
    # Создаем граф дружбы
    friends = Graph(directed=False)
    
    # Добавляем связи
    friendships = [
        ("Alice", "Bob"), ("Alice", "Charlie"), ("Bob", "David"),
        ("Charlie", "David"), ("David", "Eve"), ("Eve", "Frank"),
        ("Frank", "Alice"), ("Bob", "Eve")
    ]
    
    for friend1, friend2 in friendships:
        friends.add_edge(friend1, friend2)
    
    # Анализ связности
    print("Связи Alice:", friends.bfs("Alice"))
    
    # Поиск влиятельных пользователей (высокая степень)
    degrees = {}
    for user in friends.graph:
        degrees[user] = len(friends.graph[user])
    
    most_connected = max(degrees, key=degrees.get)
    print(f"Самый связанный пользователь: {most_connected} ({degrees[most_connected]} связей)")
    
    # Поиск кратчайшего пути между пользователями
    def shortest_path(graph, start, end):
        if start == end:
            return [start]
        
        visited = set()
        queue = deque([(start, [start])])
        
        while queue:
            vertex, path = queue.popleft()
            
            if vertex not in visited:
                visited.add(vertex)
                
                for neighbor in graph.graph[vertex]:
                    new_path = path + [neighbor]
                    if neighbor == end:
                        return new_path
                    queue.append((neighbor, new_path))
        
        return None
    
    path = shortest_path(friends, "Alice", "Frank")
    print(f"Кратчайший путь от Alice к Frank: {' -> '.join(path)}")

social_network_analysis()
```

---

## Комбинаторика

### Основные формулы и алгоритмы

```python
import math
from itertools import combinations, permutations, product

class Combinatorics:
    """Класс для комбинаторных вычислений"""
    
    @staticmethod
    def factorial(n):
        """Факториал n!"""
        if n < 0:
            raise ValueError("Factorial is not defined for negative numbers")
        return math.factorial(n)
    
    @staticmethod
    def permutations(n, r=None):
        """Размещения P(n,r) = n!/(n-r)!"""
        if r is None:
            r = n
        if r > n or r < 0:
            return 0
        return math.factorial(n) // math.factorial(n - r)
    
    @staticmethod
    def combinations(n, r):
        """Сочетания C(n,r) = n!/(r!(n-r)!)"""
        if r > n or r < 0:
            return 0
        return math.factorial(n) // (math.factorial(r) * math.factorial(n - r))
    
    @staticmethod
    def stirling_second_kind(n, k):
        """Числа Стирлинга второго рода S(n,k)"""
        if n == 0 and k == 0:
            return 1
        if n == 0 or k == 0:
            return 0
        if k > n:
            return 0
        
        # Рекуррентная формула: S(n,k) = k*S(n-1,k) + S(n-1,k-1)
        memo = {}
        
        def stirling_memo(n, k):
            if (n, k) in memo:
                return memo[(n, k)]
            
            if n == 0 and k == 0:
                result = 1
            elif n == 0 or k == 0:
                result = 0
            elif k > n:
                result = 0
            else:
                result = k * stirling_memo(n - 1, k) + stirling_memo(n - 1, k - 1)
            
            memo[(n, k)] = result
            return result
        
        return stirling_memo(n, k)
    
    @staticmethod
    def bell_number(n):
        """Числа Белла B(n) = сумма S(n,k) для всех k"""
        return sum(Combinatorics.stirling_second_kind(n, k) for k in range(n + 1))
    
    @staticmethod
    def catalan_number(n):
        """Числа Каталана C(n) = C(2n,n)/(n+1)"""
        return Combinatorics.combinations(2 * n, n) // (n + 1)
    
    @staticmethod
    def derangements(n):
        """Количество беспорядков D(n)"""
        if n == 0:
            return 1
        if n == 1:
            return 0
        
        # Рекуррентная формула: D(n) = (n-1)[D(n-1) + D(n-2)]
        d = [0] * (n + 1)
        d[0] = 1
        d[1] = 0
        
        for i in range(2, n + 1):
            d[i] = (i - 1) * (d[i - 1] + d[i - 2])
        
        return d[n]

# Пример: анализ паролей
def password_analysis():
    """Анализ сложности паролей"""
    
    comb = Combinatorics()
    
    # Количество возможных паролей
    lowercase = 26
    uppercase = 26
    digits = 10
    special = 32
    
    total_chars = lowercase + uppercase + digits + special
    
    print("Анализ сложности паролей:")
    
    for length in [6, 8, 10, 12]:
        # Все возможные пароли
        total_passwords = total_chars ** length
        
        # Только строчные буквы
        lowercase_only = lowercase ** length
        
        # Смешанные (хотя бы одна заглавная)
        mixed_case = total_passwords - (lowercase ** length)
        
        print(f"\nДлина {length} символов:")
        print(f"  Всего возможных паролей: {total_passwords:,}")
        print(f"  Только строчные: {lowercase_only:,}")
        print(f"  Со смешанным регистром: {mixed_case:,}")
        
        # Время взлома (1 млн попыток в секунду)
        attempts_per_second = 1_000_000
        worst_case_seconds = total_passwords / attempts_per_second
        worst_case_years = worst_case_seconds / (365 * 24 * 3600)
        
        print(f"  Время взлома (худший случай): {worst_case_years:.2e} лет")

password_analysis()
```

### Генерирующие функции

```python
import numpy as np
from scipy.special import comb

class GeneratingFunctions:
    """Класс для работы с производящими функциями"""
    
    def __init__(self):
        self.coefficients = []
    
    def ordinary_generating_function(self, sequence, x_values):
        """Обычная производящая функция"""
        results = []
        for x in x_values:
            result = sum(coeff * (x ** i) for i, coeff in enumerate(sequence))
            results.append(result)
        return results
    
    def exponential_generating_function(self, sequence, x_values):
        """Экспоненциальная производящая функция"""
        results = []
        for x in x_values:
            result = sum(coeff * (x ** i) / math.factorial(i) 
                        for i, coeff in enumerate(sequence))
            results.append(result)
        return results
    
    def fibonacci_generating_function(self, n_terms):
        """Производящая функция для чисел Фибоначчи"""
        # F(x) = x / (1 - x - x^2)
        fib_sequence = [0, 1]
        for i in range(2, n_terms):
            fib_sequence.append(fib_sequence[i-1] + fib_sequence[i-2])
        
        return fib_sequence
    
    def partition_function(self, n):
        """Функция разбиений - количество способов записать n как сумму натуральных чисел"""
        # Используем динамическое программирование
        dp = [0] * (n + 1)
        dp[0] = 1
        
        for i in range(1, n + 1):
            for j in range(i, n + 1):
                dp[j] += dp[j - i]
        
        return dp[n]
    
    def coin_change_ways(self, coins, amount):
        """Количество способов разменять сумму монетами"""
        dp = [0] * (amount + 1)
        dp[0] = 1
        
        for coin in coins:
            for i in range(coin, amount + 1):
                dp[i] += dp[i - coin]
        
        return dp[amount]

# Пример: анализ комбинаторных задач
def combinatorial_problems():
    """Решение комбинаторных задач"""
    
    gf = GeneratingFunctions()
    
    # Задача о разбиениях
    print("Количество разбиений чисел:")
    for n in range(1, 11):
        partitions = gf.partition_function(n)
        print(f"p({n}) = {partitions}")
    
    # Задача о размене монет
    coins = [1, 5, 10, 25]  # копейки, 5 коп, 10 коп, 25 коп
    amounts = [10, 25, 50, 100]
    
    print("\nКоличество способов разменять сумму:")
    for amount in amounts:
        ways = gf.coin_change_ways(coins, amount)
        print(f"{amount} копеек: {ways} способов")
    
    # Числа Фибоначчи
    fib_seq = gf.fibonacci_generating_function(15)
    print(f"\nПервые 15 чисел Фибоначчи: {fib_seq}")

combinatorial_problems()
```

---

## Теория чисел

### Основные алгоритмы

```python
import random
from math import gcd

class NumberTheory:
    """Класс для работы с теорией чисел"""
    
    @staticmethod
    def extended_gcd(a, b):
        """Расширенный алгоритм Евклида"""
        if a == 0:
            return b, 0, 1
        
        gcd_val, x1, y1 = NumberTheory.extended_gcd(b % a, a)
        x = y1 - (b // a) * x1
        y = x1
        
        return gcd_val, x, y
    
    @staticmethod
    def mod_inverse(a, m):
        """Модульная обратная величина"""
        gcd_val, x, _ = NumberTheory.extended_gcd(a, m)
        if gcd_val != 1:
            raise ValueError("Модульная обратная не существует")
        return (x % m + m) % m
    
    @staticmethod
    def fast_power(base, exp, mod):
        """Быстрое возведение в степень по модулю"""
        result = 1
        base = base % mod
        
        while exp > 0:
            if exp % 2 == 1:
                result = (result * base) % mod
            exp = exp >> 1
            base = (base * base) % mod
        
        return result
    
    @staticmethod
    def is_prime(n, k=10):
        """Тест Миллера-Рабина на простоту"""
        if n < 2:
            return False
        if n == 2 or n == 3:
            return True
        if n % 2 == 0:
            return False
        
        # Записываем n-1 как d * 2^r
        r = 0
        d = n - 1
        while d % 2 == 0:
            d //= 2
            r += 1
        
        # Проводим k раундов тестирования
        for _ in range(k):
            a = random.randrange(2, n - 1)
            x = NumberTheory.fast_power(a, d, n)
            
            if x == 1 or x == n - 1:
                continue
            
            for _ in range(r - 1):
                x = NumberTheory.fast_power(x, 2, n)
                if x == n - 1:
                    break
            else:
                return False
        
        return True
    
    @staticmethod
    def generate_prime(bits):
        """Генерация простого числа заданной битности"""
        while True:
            n = random.getrandbits(bits)
            n |= (1 << bits - 1) | 1  # Устанавливаем старший и младший биты
            if NumberTheory.is_prime(n):
                return n
    
    @staticmethod
    def factorize(n):
        """Факторизация числа (простая реализация)"""
        factors = []
        d = 2
        
        while d * d <= n:
            while n % d == 0:
                factors.append(d)
                n //= d
            d += 1
        
        if n > 1:
            factors.append(n)
        
        return factors
    
    @staticmethod
    def euler_phi(n):
        """Функция Эйлера φ(n)"""
        if n == 1:
            return 1
        
        factors = NumberTheory.factorize(n)
        unique_factors = set(factors)
        
        result = n
        for p in unique_factors:
            result = result * (p - 1) // p
        
        return result
    
    @staticmethod
    def chinese_remainder_theorem(remainders, moduli):
        """Китайская теорема об остатках"""
        total = 0
        prod = 1
        
        for m in moduli:
            prod *= m
        
        for r, m in zip(remainders, moduli):
            p = prod // m
            total += r * NumberTheory.mod_inverse(p, m) * p
        
        return total % prod

# Пример: RSA криптосистема
def rsa_example():
    """Пример работы RSA"""
    
    nt = NumberTheory()
    
    # Генерируем два простых числа
    p = nt.generate_prime(512)
    q = nt.generate_prime(512)
    
    print(f"p = {p}")
    print(f"q = {q}")
    
    # Вычисляем модуль
    n = p * q
    phi_n = (p - 1) * (q - 1)
    
    print(f"n = {n}")
    print(f"φ(n) = {phi_n}")
    
    # Выбираем открытую экспоненту
    e = 65537
    while gcd(e, phi_n) != 1:
        e += 2
    
    # Вычисляем секретную экспоненту
    d = nt.mod_inverse(e, phi_n)
    
    print(f"Открытый ключ: (e={e}, n={n})")
    print(f"Секретный ключ: (d={d}, n={n})")
    
    # Шифрование и дешифрование
    message = 42
    print(f"\nСообщение: {message}")
    
    # Шифрование: c = m^e mod n
    ciphertext = nt.fast_power(message, e, n)
    print(f"Зашифрованное: {ciphertext}")
    
    # Дешифрование: m = c^d mod n
    decrypted = nt.fast_power(ciphertext, d, n)
    print(f"Расшифрованное: {decrypted}")
    
    print(f"Проверка: {message == decrypted}")

# Не запускаем rsa_example() здесь, так как генерация больших простых чисел может занять время
```

---

## Алгебраические структуры

### Группы, кольца, поля

```python
class Group:
    """Базовый класс для алгебраических групп"""
    
    def __init__(self, elements, operation):
        self.elements = set(elements)
        self.operation = operation
        self.identity = self.find_identity()
    
    def find_identity(self):
        """Поиск нейтрального элемента"""
        for e in self.elements:
            if all(self.operation(e, a) == a and self.operation(a, e) == a 
                   for a in self.elements):
                return e
        return None
    
    def is_group(self):
        """Проверка аксиом группы"""
        # Замкнутость
        for a in self.elements:
            for b in self.elements:
                if self.operation(a, b) not in self.elements:
                    return False
        
        # Ассоциативность
        for a in self.elements:
            for b in self.elements:
                for c in self.elements:
                    if (self.operation(self.operation(a, b), c) != 
                        self.operation(a, self.operation(b, c))):
                        return False
        
        # Нейтральный элемент
        if self.identity is None:
            return False
        
        # Обратные элементы
        for a in self.elements:
            has_inverse = False
            for b in self.elements:
                if (self.operation(a, b) == self.identity and 
                    self.operation(b, a) == self.identity):
                    has_inverse = True
                    break
            if not has_inverse:
                return False
        
        return True
    
    def find_inverse(self, element):
        """Поиск обратного элемента"""
        for b in self.elements:
            if (self.operation(element, b) == self.identity and 
                self.operation(b, element) == self.identity):
                return b
        return None
    
    def order(self):
        """Порядок группы"""
        return len(self.elements)
    
    def element_order(self, element):
        """Порядок элемента"""
        current = element
        order = 1
        
        while current != self.identity:
            current = self.operation(current, element)
            order += 1
            if order > len(self.elements):  # Защита от бесконечного цикла
                return float('inf')
        
        return order

# Пример: циклическая группа Z_n
class CyclicGroup(Group):
    """Циклическая группа Z_n"""
    
    def __init__(self, n):
        self.n = n
        elements = list(range(n))
        operation = lambda a, b: (a + b) % n
        super().__init__(elements, operation)
    
    def generate_subgroup(self, generator):
        """Генерация подгруппы"""
        subgroup = set()
        current = 0  # Нейтральный элемент
        
        while current not in subgroup:
            subgroup.add(current)
            current = self.operation(current, generator)
        
        return subgroup

# Пример: работа с конечными полями
class FiniteField:
    """Конечное поле GF(p)"""
    
    def __init__(self, p):
        if not NumberTheory.is_prime(p):
            raise ValueError("p must be prime")
        self.p = p
        self.elements = list(range(p))
    
    def add(self, a, b):
        """Сложение в поле"""
        return (a + b) % self.p
    
    def multiply(self, a, b):
        """Умножение в поле"""
        return (a * b) % self.p
    
    def inverse(self, a):
        """Мультипликативная обратная"""
        if a == 0:
            raise ValueError("0 has no multiplicative inverse")
        return NumberTheory.mod_inverse(a, self.p)
    
    def divide(self, a, b):
        """Деление в поле"""
        return self.multiply(a, self.inverse(b))
    
    def power(self, a, n):
        """Возведение в степень"""
        return NumberTheory.fast_power(a, n, self.p)

# Пример: использование конечных полей в криптографии
def finite_field_example():
    """Пример работы с конечными полями"""
    
    # Создаем поле GF(7)
    field = FiniteField(7)
    
    print("Операции в поле GF(7):")
    print("Таблица сложения:")
    for a in range(7):
        row = []
        for b in range(7):
            row.append(field.add(a, b))
        print(f"{a}: {row}")
    
    print("\nТаблица умножения:")
    for a in range(1, 7):
        row = []
        for b in range(1, 7):
            row.append(field.multiply(a, b))
        print(f"{a}: {row}")
    
    print("\nМультипликативные обратные:")
    for a in range(1, 7):
        inv = field.inverse(a)
        print(f"{a}^(-1) = {inv}, проверка: {a} * {inv} = {field.multiply(a, inv)}")

finite_field_example()
```

---

## Автоматы и формальные языки

### Конечные автоматы

```python
class FiniteAutomaton:
    """Детерминированный конечный автомат"""
    
    def __init__(self, states, alphabet, transitions, start_state, accept_states):
        self.states = set(states)
        self.alphabet = set(alphabet)
        self.transitions = transitions
        self.start_state = start_state
        self.accept_states = set(accept_states)
    
    def process_string(self, string):
        """Обработка строки автоматом"""
        current_state = self.start_state
        
        for symbol in string:
            if symbol not in self.alphabet:
                return False
            
            key = (current_state, symbol)
            if key not in self.transitions:
                return False
            
            current_state = self.transitions[key]
        
        return current_state in self.accept_states
    
    def minimize(self):
        """Минимизация автомата"""
        # Алгоритм минимизации (упрощенная версия)
        # Разделяем состояния на принимающие и непринимающие
        accepting = self.accept_states
        non_accepting = self.states - accepting
        
        partitions = [accepting, non_accepting]
        
        changed = True
        while changed:
            changed = False
            new_partitions = []
            
            for partition in partitions:
                if len(partition) <= 1:
                    new_partitions.append(partition)
                    continue
                
                # Пытаемся разделить partition
                subpartitions = self._split_partition(partition, partitions)
                
                if len(subpartitions) > 1:
                    changed = True
                    new_partitions.extend(subpartitions)
                else:
                    new_partitions.append(partition)
            
            partitions = new_partitions
        
        return self._build_minimized_automaton(partitions)
    
    def _split_partition(self, partition, all_partitions):
        """Разделение partition на основе переходов"""
        if len(partition) <= 1:
            return [partition]
        
        # Группируем состояния по их поведению
        groups = {}
        
        for state in partition:
            signature = []
            for symbol in sorted(self.alphabet):
                key = (state, symbol)
                if key in self.transitions:
                    next_state = self.transitions[key]
                    # Находим partition, содержащий next_state
                    for i, p in enumerate(all_partitions):
                        if next_state in p:
                            signature.append(i)
                            break
                else:
                    signature.append(-1)  # Нет перехода
            
            signature = tuple(signature)
            if signature not in groups:
                groups[signature] = set()
            groups[signature].add(state)
        
        return list(groups.values())
    
    def _build_minimized_automaton(self, partitions):
        """Построение минимизированного автомата"""
        # Создаем новые состояния
        state_map = {}
        new_states = []
        
        for i, partition in enumerate(partitions):
            new_state = f"q{i}"
            new_states.append(new_state)
            for old_state in partition:
                state_map[old_state] = new_state
        
        # Создаем новые переходы
        new_transitions = {}
        for (old_state, symbol), next_old_state in self.transitions.items():
            new_state = state_map[old_state]
            new_next_state = state_map[next_old_state]
            new_transitions[(new_state, symbol)] = new_next_state
        
        # Новое начальное состояние
        new_start_state = state_map[self.start_state]
        
        # Новые принимающие состояния
        new_accept_states = set()
        for old_accept in self.accept_states:
            new_accept_states.add(state_map[old_accept])
        
        return FiniteAutomaton(
            new_states, self.alphabet, new_transitions, 
            new_start_state, new_accept_states
        )

# Пример: автомат для проверки четности количества единиц
def even_ones_automaton():
    """Автомат, принимающий строки с четным количеством единиц"""
    
    # Состояния: even (четное), odd (нечетное)
    states = ['even', 'odd']
    alphabet = ['0', '1']
    
    # Переходы
    transitions = {
        ('even', '0'): 'even',
        ('even', '1'): 'odd',
        ('odd', '0'): 'odd',
        ('odd', '1'): 'even'
    }
    
    start_state = 'even'
    accept_states = ['even']
    
    automaton = FiniteAutomaton(states, alphabet, transitions, start_state, accept_states)
    
    # Тестируем
    test_strings = ['', '0', '1', '01', '11', '101', '1001', '110011']
    
    print("Тестирование автомата для четного количества единиц:")
    for string in test_strings:
        result = automaton.process_string(string)
        ones_count = string.count('1')
        expected = (ones_count % 2 == 0)
        print(f"'{string}': {result} (единиц: {ones_count}, четное: {expected})")

even_ones_automaton()
```

## 🎯 Практические задания

### Задание 1: Граф зависимостей
Создайте граф зависимостей для системы пакетов и найдите порядок установки.

### Задание 2: Хеширование
Реализуйте криптографическую хеш-функцию на основе теории чисел.

### Задание 3: Анализ алгоритмов
Используйте производящие функции для анализа сложности рекурсивных алгоритмов.

### Задание 4: Формальная верификация
Создайте автомат для проверки корректности простого протокола.

---

## 🔗 Связанные материалы

- [Математический анализ](./mathematical-analysis.md)
- [Алгоритмы и структуры данных](../algorithms-data-structures/README.md)
- [Криптография](../technical-skills/security.md)
- [Теория вычислений](../algorithms-data-structures/computer-science-fundamentals.md) 