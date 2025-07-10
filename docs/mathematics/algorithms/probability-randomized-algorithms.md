# Теория вероятностей и рандомизированные алгоритмы 🎲

> **Навигация**: [[linear-algebra-computation|← Линейная алгебра]] | [[mathematics-algorithms|Главная]] | [[calculus-optimization|Математический анализ →]]

## Содержание
1. [Основы теории вероятностей](#🎲-основы-теории-вероятностей)
2. [Рандомизированные алгоритмы](#🎯-рандомизированные-алгоритмы)
3. [Хеширование](#🔐-хеширование)
4. [Практические применения](#🚀-практические-применения)

---

## 🎲 Основы теории вероятностей

**Математическая основа:** Вероятностное пространство, независимость событий, условная вероятность

**Алгоритмические применения:**
- Рандомизированные алгоритмы
- Вероятностные структуры данных
- Анализ сложности в среднем
- Моделирование и симуляции

```python
import random
import hashlib
from collections import defaultdict
import math

class ProbabilityAlgorithms:
    """Алгоритмы на основе теории вероятностей"""
    
    @staticmethod
    def monte_carlo_pi(n_samples=1000000):
        """
        Вычисление π методом Монте-Карло
        Используем случайные точки в квадрате [-1,1] x [-1,1]
        P(точка в круге) = π/4
        """
        inside_circle = 0
        
        for _ in range(n_samples):
            x = random.uniform(-1, 1)
            y = random.uniform(-1, 1)
            
            if x*x + y*y <= 1:
                inside_circle += 1
        
        pi_estimate = 4 * inside_circle / n_samples
        return pi_estimate
    
    @staticmethod
    def reservoir_sampling(stream, k):
        """
        Алгоритм резервуарной выборки
        Выбирает k элементов из потока данных неизвестной длины
        Каждый элемент имеет равную вероятность быть выбранным
        """
        reservoir = []
        
        for i, item in enumerate(stream):
            if i < k:
                reservoir.append(item)
            else:
                # Генерируем случайный индекс от 0 до i
                j = random.randint(0, i)
                if j < k:
                    reservoir[j] = item
        
        return reservoir
    
    @staticmethod
    def randomized_quicksort(arr):
        """
        Рандомизированная быстрая сортировка
        Ожидаемая сложность O(n log n) даже в худшем случае входных данных
        """
        if len(arr) <= 1:
            return arr
        
        # Случайный выбор опорного элемента
        pivot_index = random.randint(0, len(arr) - 1)
        pivot = arr[pivot_index]
        
        less = [x for x in arr if x < pivot]
        equal = [x for x in arr if x == pivot]
        greater = [x for x in arr if x > pivot]
        
        return (ProbabilityAlgorithms.randomized_quicksort(less) + 
                equal + 
                ProbabilityAlgorithms.randomized_quicksort(greater))
    
    @staticmethod
    def randomized_select(arr, k):
        """
        Рандомизированный поиск k-го порядкового элемента
        Ожидаемая сложность O(n)
        """
        if len(arr) == 1:
            return arr[0]
        
        # Случайный выбор опорного элемента
        pivot_index = random.randint(0, len(arr) - 1)
        pivot = arr[pivot_index]
        
        less = [x for x in arr if x < pivot]
        equal = [x for x in arr if x == pivot]
        greater = [x for x in arr if x > pivot]
        
        if k <= len(less):
            return ProbabilityAlgorithms.randomized_select(less, k)
        elif k <= len(less) + len(equal):
            return pivot
        else:
            return ProbabilityAlgorithms.randomized_select(greater, k - len(less) - len(equal))
```

---

## 🎯 Рандомизированные алгоритмы

### Алгоритмы Las Vegas vs Monte Carlo

```python
class RandomizedAlgorithms:
    """Реализация различных типов рандомизированных алгоритмов"""
    
    def __init__(self, seed=None):
        if seed:
            random.seed(seed)
    
    def primality_test_miller_rabin(self, n, k=10):
        """
        Тест простоты Миллера-Рабина (Monte Carlo алгоритм)
        Вероятность ошибки ≤ (1/4)^k
        """
        if n < 2:
            return False
        if n == 2 or n == 3:
            return True
        if n % 2 == 0:
            return False
        
        # Представляем n-1 в виде d * 2^r
        r = 0
        d = n - 1
        while d % 2 == 0:
            r += 1
            d //= 2
        
        # Повторяем тест k раз
        for _ in range(k):
            a = random.randint(2, n - 2)
            x = pow(a, d, n)  # a^d mod n
            
            if x == 1 or x == n - 1:
                continue
            
            for _ in range(r - 1):
                x = pow(x, 2, n)
                if x == n - 1:
                    break
            else:
                return False  # Составное число
        
        return True  # Вероятно простое
    
    def randomized_graph_coloring(self, adjacency_list, max_colors):
        """
        Рандомизированная раскраска графа
        Las Vegas алгоритм - всегда корректный результат
        """
        n = len(adjacency_list)
        coloring = {}
        
        # Случайный порядок вершин
        vertices = list(range(n))
        random.shuffle(vertices)
        
        for vertex in vertices:
            # Находим доступные цвета
            used_colors = set()
            for neighbor in adjacency_list[vertex]:
                if neighbor in coloring:
                    used_colors.add(coloring[neighbor])
            
            available_colors = [c for c in range(max_colors) if c not in used_colors]
            
            if not available_colors:
                return None  # Невозможно раскрасить
            
            # Случайно выбираем доступный цвет
            coloring[vertex] = random.choice(available_colors)
        
        return coloring
    
    def skip_list_search(self, skip_list, target):
        """
        Поиск в Skip List - рандомизированная структура данных
        Ожидаемая сложность O(log n)
        """
        current = skip_list.header
        
        # Идем с верхнего уровня вниз
        for level in range(skip_list.max_level - 1, -1, -1):
            while (current.forward[level] and 
                   current.forward[level].value < target):
                current = current.forward[level]
        
        # Переходим на уровень 0
        current = current.forward[0]
        
        if current and current.value == target:
            return current
        else:
            return None

class SkipListNode:
    """Узел Skip List"""
    def __init__(self, value, level):
        self.value = value
        self.forward = [None] * (level + 1)

class SkipList:
    """Рандомизированная структура данных Skip List"""
    
    def __init__(self, max_level=16, p=0.5):
        self.max_level = max_level
        self.p = p
        self.header = SkipListNode(None, max_level)
        self.level = 0
    
    def random_level(self):
        """Генерирует случайный уровень для нового узла"""
        level = 0
        while random.random() < self.p and level < self.max_level:
            level += 1
        return level
    
    def insert(self, value):
        """Вставка элемента в Skip List"""
        update = [None] * self.max_level
        current = self.header
        
        # Поиск позиции для вставки
        for i in range(self.level, -1, -1):
            while (current.forward[i] and 
                   current.forward[i].value < value):
                current = current.forward[i]
            update[i] = current
        
        current = current.forward[0]
        
        if current is None or current.value != value:
            # Создаем новый узел
            new_level = self.random_level()
            
            if new_level > self.level:
                for i in range(self.level + 1, new_level + 1):
                    update[i] = self.header
                self.level = new_level
            
            new_node = SkipListNode(value, new_level)
            
            for i in range(new_level + 1):
                new_node.forward[i] = update[i].forward[i]
                update[i].forward[i] = new_node
```

---

## 🔐 Хеширование

### Универсальное хеширование

```python
class UniversalHashFamily:
    """Семейство универсальных хеш-функций"""
    
    def __init__(self, table_size):
        self.table_size = table_size
        self.p = self._next_prime(table_size * 2)  # Большое простое число
        self.a = random.randint(1, self.p - 1)
        self.b = random.randint(0, self.p - 1)
    
    def _next_prime(self, n):
        """Найти ближайшее простое число >= n"""
        while not self._is_prime(n):
            n += 1
        return n
    
    def _is_prime(self, n):
        """Простая проверка на простоту"""
        if n < 2:
            return False
        for i in range(2, int(n ** 0.5) + 1):
            if n % i == 0:
                return False
        return True
    
    def hash_function(self, key):
        """
        Универсальная хеш-функция: h(k) = ((ak + b) mod p) mod m
        Для любых двух различных ключей k1, k2:
        P(h(k1) = h(k2)) ≤ 1/m
        """
        return ((self.a * key + self.b) % self.p) % self.table_size

class BloomFilter:
    """
    Фильтр Блума - вероятностная структура данных
    Может давать ложноположительные результаты, но не ложноотрицательные
    """
    
    def __init__(self, size, num_hash_functions):
        self.size = size
        self.num_hash_functions = num_hash_functions
        self.bit_array = [False] * size
        self.hash_functions = [
            UniversalHashFamily(size) for _ in range(num_hash_functions)
        ]
    
    def add(self, item):
        """Добавить элемент в фильтр"""
        item_hash = hash(item)
        for hash_func in self.hash_functions:
            index = hash_func.hash_function(item_hash)
            self.bit_array[index] = True
    
    def contains(self, item):
        """
        Проверить, может ли элемент быть в множестве
        True: возможно есть (может быть ложноположительный)
        False: точно нет
        """
        item_hash = hash(item)
        for hash_func in self.hash_functions:
            index = hash_func.hash_function(item_hash)
            if not self.bit_array[index]:
                return False
        return True
    
    def false_positive_probability(self, num_elements):
        """
        Вычисляет теоретическую вероятность ложноположительного результата
        P ≈ (1 - e^(-kn/m))^k
        где k = количество хеш-функций, n = количество элементов, m = размер фильтра
        """
        k = self.num_hash_functions
        n = num_elements
        m = self.size
        
        return (1 - math.exp(-k * n / m)) ** k

class CountMinSketch:
    """
    Count-Min Sketch - вероятностная структура для подсчета частот
    Гарантирует верхнюю оценку с высокой вероятностью
    """
    
    def __init__(self, width, depth):
        self.width = width
        self.depth = depth
        self.table = [[0] * width for _ in range(depth)]
        self.hash_functions = [
            UniversalHashFamily(width) for _ in range(depth)
        ]
    
    def update(self, item, count=1):
        """Увеличить счетчик для элемента"""
        item_hash = hash(item)
        for i in range(self.depth):
            j = self.hash_functions[i].hash_function(item_hash)
            self.table[i][j] += count
    
    def estimate(self, item):
        """
        Оценить частоту элемента
        Возвращает верхнюю оценку с высокой вероятностью
        """
        item_hash = hash(item)
        estimates = []
        
        for i in range(self.depth):
            j = self.hash_functions[i].hash_function(item_hash)
            estimates.append(self.table[i][j])
        
        return min(estimates)  # Возвращаем минимальную оценку
```

---

## 🚀 Практические применения

### Кэширование и балансировка нагрузки

```python
class ConsistentHashing:
    """
    Консистентное хеширование для распределенных систем
    Минимизирует количество перемещений при изменении количества серверов
    """
    
    def __init__(self, replicas=3):
        self.replicas = replicas
        self.ring = {}
        self.sorted_keys = []
    
    def _hash(self, key):
        """Хеш-функция для кольца"""
        return int(hashlib.md5(key.encode()).hexdigest(), 16)
    
    def add_server(self, server):
        """Добавить сервер в кольцо"""
        for i in range(self.replicas):
            virtual_key = f"{server}:{i}"
            key = self._hash(virtual_key)
            self.ring[key] = server
            self.sorted_keys.append(key)
        
        self.sorted_keys.sort()
    
    def remove_server(self, server):
        """Удалить сервер из кольца"""
        for i in range(self.replicas):
            virtual_key = f"{server}:{i}"
            key = self._hash(virtual_key)
            del self.ring[key]
            self.sorted_keys.remove(key)
    
    def get_server(self, data_key):
        """Найти сервер для данного ключа"""
        if not self.ring:
            return None
        
        key = self._hash(data_key)
        
        # Найти первый сервер по часовой стрелке
        for server_key in self.sorted_keys:
            if key <= server_key:
                return self.ring[server_key]
        
        # Если не найден, берем первый сервер (кольцевая структура)
        return self.ring[self.sorted_keys[0]]

class RandomizedLoadBalancer:
    """Рандомизированный балансировщик нагрузки"""
    
    def __init__(self, servers):
        self.servers = servers
        self.server_weights = {server: 1.0 for server in servers}
        self.server_loads = {server: 0 for server in servers}
    
    def power_of_two_choices(self):
        """
        Алгоритм "сила двух выборов"
        Выбираем два случайных сервера и направляем запрос на менее загруженный
        """
        server1, server2 = random.sample(self.servers, 2)
        
        if self.server_loads[server1] <= self.server_loads[server2]:
            chosen_server = server1
        else:
            chosen_server = server2
        
        self.server_loads[chosen_server] += 1
        return chosen_server
    
    def weighted_random_selection(self):
        """Взвешенный случайный выбор сервера"""
        total_weight = sum(self.server_weights.values())
        r = random.uniform(0, total_weight)
        
        cumulative_weight = 0
        for server in self.servers:
            cumulative_weight += self.server_weights[server]
            if r <= cumulative_weight:
                self.server_loads[server] += 1
                return server
        
        # На случай погрешности вычислений
        return random.choice(self.servers)
    
    def update_server_weight(self, server, new_weight):
        """Обновить вес сервера"""
        self.server_weights[server] = new_weight
    
    def decrease_load(self, server):
        """Уменьшить нагрузку на сервер (после завершения запроса)"""
        if self.server_loads[server] > 0:
            self.server_loads[server] -= 1

# Демонстрация использования
def demo_randomized_algorithms():
    """Демонстрация рандомизированных алгоритмов"""
    
    # Тест простоты числа
    prob_algo = RandomizedAlgorithms()
    large_number = 1000000007
    is_prime = prob_algo.primality_test_miller_rabin(large_number)
    print(f"Число {large_number} простое: {is_prime}")
    
    # Фильтр Блума
    bloom = BloomFilter(size=1000, num_hash_functions=3)
    
    # Добавляем элементы
    words = ["apple", "banana", "orange", "grape"]
    for word in words:
        bloom.add(word)
    
    # Проверяем наличие
    test_words = ["apple", "cherry", "banana", "kiwi"]
    for word in test_words:
        result = bloom.contains(word)
        print(f"'{word}' в фильтре: {result}")
    
    # Консистентное хеширование
    ch = ConsistentHashing()
    servers = ["server1", "server2", "server3"]
    
    for server in servers:
        ch.add_server(server)
    
    # Распределяем данные
    data_keys = ["user:1", "user:2", "user:3", "user:4", "user:5"]
    for key in data_keys:
        server = ch.get_server(key)
        print(f"Ключ '{key}' -> {server}")
```

**Когда использовать рандомизированные алгоритмы:**

- ✅ **Monte Carlo**: Когда нужна приближенная оценка (вычисление π, интегралов)
- ✅ **Las Vegas**: Когда нужен точный результат за случайное время (QuickSort, поиск)
- ✅ **Фильтр Блума**: Проверка наличия в больших наборах данных
- ✅ **Консистентное хеширование**: Распределенные системы и кэширование

---

## 🔗 Связанные темы

- [[linear-algebra-computation|Линейная алгебра]] - случайные матрицы и проекции
- [[statistics-data-analysis|Статистика]] - оценка вероятностей и анализ
- [[number-theory-cryptography|Теория чисел]] - тесты простоты и криптография
- [[combinatorics-generation|Комбинаторика]] - случайная генерация и выборка

## 📚 Дополнительные ресурсы

- **Книги**: "Randomized Algorithms" - Motwani & Raghavan  
- **Курсы**: Stanford CS265, MIT 6.856
- **Практика**: Codeforces probabilistic problems 