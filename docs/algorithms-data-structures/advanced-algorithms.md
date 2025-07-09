# Продвинутые алгоритмы

## 🎯 Цель курса

Изучение сложных алгоритмических техник для решения нетривиальных задач в Computer Science.

## 📚 Содержание

1. [Продвинутые структуры данных](#продвинутые-структуры-данных)
2. [Алгоритмы на графах](#алгоритмы-на-графах)
3. [Строковые алгоритмы](#строковые-алгоритмы)
4. [Геометрические алгоритмы](#геометрические-алгоритмы)
5. [Параллельные алгоритмы](#параллельные-алгоритмы)
6. [Аппроксимационные алгоритмы](#аппроксимационные-алгоритмы)

---

## Продвинутые структуры данных

### Segment Tree (Дерево отрезков)

```python
class SegmentTree:
    """Дерево отрезков для операций на отрезках"""
    
    def __init__(self, arr, operation='sum'):
        self.n = len(arr)
        self.tree = [0] * (4 * self.n)
        self.lazy = [0] * (4 * self.n)  # Для ленивых обновлений
        self.operation = operation
        self.arr = arr[:]
        self.build(1, 0, self.n - 1)
    
    def build(self, node, start, end):
        """Построение дерева"""
        if start == end:
            self.tree[node] = self.arr[start]
        else:
            mid = (start + end) // 2
            self.build(2 * node, start, mid)
            self.build(2 * node + 1, mid + 1, end)
            self.tree[node] = self._combine(
                self.tree[2 * node], 
                self.tree[2 * node + 1]
            )
    
    def _combine(self, left, right):
        """Комбинирование значений в зависимости от операции"""
        if self.operation == 'sum':
            return left + right
        elif self.operation == 'min':
            return min(left, right)
        elif self.operation == 'max':
            return max(left, right)
        elif self.operation == 'gcd':
            from math import gcd
            return gcd(left, right)
    
    def update_lazy(self, node, start, end):
        """Применение ленивых обновлений"""
        if self.lazy[node] != 0:
            self.tree[node] += self.lazy[node] * (end - start + 1)
            if start != end:  # Не лист
                self.lazy[2 * node] += self.lazy[node]
                self.lazy[2 * node + 1] += self.lazy[node]
            self.lazy[node] = 0
    
    def update_range(self, node, start, end, l, r, val):
        """Обновление отрезка [l, r] на значение val"""
        self.update_lazy(node, start, end)
        
        if start > r or end < l:
            return
        
        if start >= l and end <= r:
            self.tree[node] += val * (end - start + 1)
            if start != end:
                self.lazy[2 * node] += val
                self.lazy[2 * node + 1] += val
            return
        
        mid = (start + end) // 2
        self.update_range(2 * node, start, mid, l, r, val)
        self.update_range(2 * node + 1, mid + 1, end, l, r, val)
        
        self.update_lazy(2 * node, start, mid)
        self.update_lazy(2 * node + 1, mid + 1, end)
        
        self.tree[node] = self._combine(
            self.tree[2 * node], 
            self.tree[2 * node + 1]
        )
    
    def query_range(self, node, start, end, l, r):
        """Запрос на отрезке [l, r]"""
        if start > r or end < l:
            return 0 if self.operation == 'sum' else float('inf')
        
        self.update_lazy(node, start, end)
        
        if start >= l and end <= r:
            return self.tree[node]
        
        mid = (start + end) // 2
        left_result = self.query_range(2 * node, start, mid, l, r)
        right_result = self.query_range(2 * node + 1, mid + 1, end, l, r)
        
        return self._combine(left_result, right_result)
    
    def update(self, l, r, val):
        """Публичный метод для обновления отрезка"""
        self.update_range(1, 0, self.n - 1, l, r, val)
    
    def query(self, l, r):
        """Публичный метод для запроса на отрезке"""
        return self.query_range(1, 0, self.n - 1, l, r)

# Пример использования
arr = [1, 3, 5, 7, 9, 11]
st = SegmentTree(arr, 'sum')

print(f"Сумма на отрезке [1, 3]: {st.query(1, 3)}")  # 3 + 5 + 7 = 15
st.update(1, 3, 2)  # Добавляем 2 к элементам [1, 3]
print(f"Сумма после обновления: {st.query(1, 3)}")  # 5 + 7 + 9 = 21
```

### Trie (Префиксное дерево)

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end_word = False
        self.frequency = 0

class Trie:
    """Префиксное дерево для работы со строками"""
    
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        """Вставка слова"""
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
            node.frequency += 1
        node.is_end_word = True
    
    def search(self, word):
        """Поиск слова"""
        node = self._find_node(word)
        return node is not None and node.is_end_word
    
    def starts_with(self, prefix):
        """Проверка наличия слов с префиксом"""
        return self._find_node(prefix) is not None
    
    def _find_node(self, prefix):
        """Поиск узла по префиксу"""
        node = self.root
        for char in prefix:
            if char not in node.children:
                return None
            node = node.children[char]
        return node
    
    def get_words_with_prefix(self, prefix):
        """Получение всех слов с заданным префиксом"""
        node = self._find_node(prefix)
        if not node:
            return []
        
        words = []
        self._dfs(node, prefix, words)
        return words
    
    def _dfs(self, node, current_word, words):
        """DFS для сбора слов"""
        if node.is_end_word:
            words.append(current_word)
        
        for char, child_node in node.children.items():
            self._dfs(child_node, current_word + char, words)
    
    def delete(self, word):
        """Удаление слова"""
        def _delete_helper(node, word, index):
            if index == len(word):
                if not node.is_end_word:
                    return False
                node.is_end_word = False
                return len(node.children) == 0
            
            char = word[index]
            if char not in node.children:
                return False
            
            should_delete_child = _delete_helper(node.children[char], word, index + 1)
            
            if should_delete_child:
                del node.children[char]
                return not node.is_end_word and len(node.children) == 0
            
            return False
        
        _delete_helper(self.root, word, 0)

# Пример: автокомплит
def autocomplete_example():
    trie = Trie()
    
    # Добавляем слова
    words = ["apple", "app", "application", "apply", "banana", "band", "bandana"]
    for word in words:
        trie.insert(word)
    
    # Тестируем автокомплит
    prefix = "app"
    suggestions = trie.get_words_with_prefix(prefix)
    print(f"Автокомплит для '{prefix}': {suggestions}")

autocomplete_example()
```

---

## Алгоритмы на графах

### Алгоритм Форда-Фалкерсона (максимальный поток)

```python
from collections import defaultdict, deque

class MaxFlow:
    """Алгоритм максимального потока"""
    
    def __init__(self, vertices):
        self.V = vertices
        self.graph = [[0] * vertices for _ in range(vertices)]
    
    def add_edge(self, u, v, capacity):
        """Добавление ребра с пропускной способностью"""
        self.graph[u][v] = capacity
    
    def bfs(self, source, sink, parent):
        """BFS для поиска пути в остаточном графе"""
        visited = [False] * self.V
        queue = deque([source])
        visited[source] = True
        
        while queue:
            u = queue.popleft()
            
            for v in range(self.V):
                if not visited[v] and self.graph[u][v] > 0:
                    queue.append(v)
                    visited[v] = True
                    parent[v] = u
                    if v == sink:
                        return True
        return False
    
    def ford_fulkerson(self, source, sink):
        """Алгоритм Форда-Фалкерсона"""
        parent = [-1] * self.V
        max_flow = 0
        
        # Создаем копию графа для остаточной сети
        residual_graph = [row[:] for row in self.graph]
        self.graph = residual_graph
        
        while self.bfs(source, sink, parent):
            # Находим минимальную пропускную способность на пути
            path_flow = float('inf')
            s = sink
            
            while s != source:
                path_flow = min(path_flow, self.graph[parent[s]][s])
                s = parent[s]
            
            # Обновляем остаточную сеть
            v = sink
            while v != source:
                u = parent[v]
                self.graph[u][v] -= path_flow
                self.graph[v][u] += path_flow
                v = parent[v]
            
            max_flow += path_flow
        
        return max_flow

# Пример: сетевой поток
def network_flow_example():
    # Создаем граф с 6 вершинами
    g = MaxFlow(6)
    
    # Добавляем ребра (источник, приемник, пропускная способность)
    edges = [
        (0, 1, 16), (0, 2, 13),
        (1, 2, 10), (1, 3, 12),
        (2, 1, 4), (2, 4, 14),
        (3, 2, 9), (3, 5, 20),
        (4, 3, 7), (4, 5, 4)
    ]
    
    for u, v, capacity in edges:
        g.add_edge(u, v, capacity)
    
    source, sink = 0, 5
    max_flow = g.ford_fulkerson(source, sink)
    print(f"Максимальный поток от {source} к {sink}: {max_flow}")

network_flow_example()
```

### Алгоритм Тарьяна (сильно связные компоненты)

```python
class TarjanSCC:
    """Алгоритм Тарьяна для поиска сильно связных компонент"""
    
    def __init__(self, vertices):
        self.V = vertices
        self.graph = defaultdict(list)
        self.time = 0
    
    def add_edge(self, u, v):
        self.graph[u].append(v)
    
    def scc(self):
        """Поиск всех сильно связных компонент"""
        # Инициализация
        disc = [-1] * self.V
        low = [-1] * self.V
        on_stack = [False] * self.V
        stack = []
        sccs = []
        
        def dfs(u):
            disc[u] = low[u] = self.time
            self.time += 1
            stack.append(u)
            on_stack[u] = True
            
            # Рекурсивно обходим соседей
            for v in self.graph[u]:
                if disc[v] == -1:  # Если v не посещен
                    dfs(v)
                    low[u] = min(low[u], low[v])
                elif on_stack[v]:  # Если v в стеке
                    low[u] = min(low[u], disc[v])
            
            # Если u - корень SCC
            if low[u] == disc[u]:
                scc = []
                while True:
                    w = stack.pop()
                    on_stack[w] = False
                    scc.append(w)
                    if w == u:
                        break
                sccs.append(scc)
        
        # Запускаем DFS для всех непосещенных вершин
        for i in range(self.V):
            if disc[i] == -1:
                dfs(i)
        
        return sccs

# Пример использования
tarjan = TarjanSCC(5)
edges = [(1, 0), (0, 2), (2, 1), (0, 3), (3, 4)]
for u, v in edges:
    tarjan.add_edge(u, v)

sccs = tarjan.scc()
print(f"Сильно связные компоненты: {sccs}")
```

---

## Строковые алгоритмы

### Алгоритм Кнута-Морриса-Пратта (KMP)

```python
def kmp_search(text, pattern):
    """Алгоритм Кнута-Морриса-Пратта для поиска подстроки"""
    
    def compute_lps(pattern):
        """Вычисление массива LPS (Longest Proper Prefix which is also Suffix)"""
        m = len(pattern)
        lps = [0] * m
        length = 0
        i = 1
        
        while i < m:
            if pattern[i] == pattern[length]:
                length += 1
                lps[i] = length
                i += 1
            else:
                if length != 0:
                    length = lps[length - 1]
                else:
                    lps[i] = 0
                    i += 1
        
        return lps
    
    n = len(text)
    m = len(pattern)
    
    if m == 0:
        return []
    
    lps = compute_lps(pattern)
    matches = []
    
    i = 0  # Индекс для text
    j = 0  # Индекс для pattern
    
    while i < n:
        if pattern[j] == text[i]:
            i += 1
            j += 1
        
        if j == m:
            matches.append(i - j)
            j = lps[j - 1]
        elif i < n and pattern[j] != text[i]:
            if j != 0:
                j = lps[j - 1]
            else:
                i += 1
    
    return matches

# Пример использования
text = "ABABDABACDABABCABCABCABCABC"
pattern = "ABABCABCABCABC"
matches = kmp_search(text, pattern)
print(f"Паттерн найден в позициях: {matches}")
```

### Z-алгоритм

```python
def z_algorithm(s):
    """Z-алгоритм для поиска всех вхождений"""
    n = len(s)
    z = [0] * n
    left = right = 0
    
    for i in range(1, n):
        if i <= right:
            z[i] = min(right - i + 1, z[i - left])
        
        while i + z[i] < n and s[z[i]] == s[i + z[i]]:
            z[i] += 1
        
        if i + z[i] - 1 > right:
            left, right = i, i + z[i] - 1
    
    return z

def find_all_occurrences(text, pattern):
    """Поиск всех вхождений с помощью Z-алгоритма"""
    combined = pattern + "$" + text
    z_values = z_algorithm(combined)
    
    pattern_length = len(pattern)
    occurrences = []
    
    for i in range(pattern_length + 1, len(combined)):
        if z_values[i] == pattern_length:
            occurrences.append(i - pattern_length - 1)
    
    return occurrences

# Пример
text = "AABAACAADAABAABA"
pattern = "AABA"
occurrences = find_all_occurrences(text, pattern)
print(f"Паттерн '{pattern}' найден в позициях: {occurrences}")
```

---

## Геометрические алгоритмы

### Выпуклая оболочка (Graham Scan)

```python
import math

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __repr__(self):
        return f"({self.x}, {self.y})"

def cross_product(o, a, b):
    """Векторное произведение"""
    return (a.x - o.x) * (b.y - o.y) - (a.y - o.y) * (b.x - o.x)

def distance_squared(p1, p2):
    """Квадрат расстояния между точками"""
    return (p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2

def convex_hull_graham(points):
    """Алгоритм Грэхема для построения выпуклой оболочки"""
    n = len(points)
    if n < 3:
        return points
    
    # Находим нижнюю точку (или левую при равенстве y)
    start = min(points, key=lambda p: (p.y, p.x))
    
    # Сортируем точки по полярному углу относительно start
    def polar_angle_key(point):
        if point == start:
            return -math.pi, 0
        
        dx = point.x - start.x
        dy = point.y - start.y
        
        angle = math.atan2(dy, dx)
        dist = distance_squared(start, point)
        
        return angle, dist
    
    sorted_points = sorted(points, key=polar_angle_key)
    
    # Удаляем точки с одинаковым углом, оставляя самую дальнюю
    unique_points = [sorted_points[0]]
    for i in range(1, len(sorted_points)):
        while (i < len(sorted_points) - 1 and 
               cross_product(start, sorted_points[i], sorted_points[i + 1]) == 0):
            i += 1
        unique_points.append(sorted_points[i])
    
    if len(unique_points) < 3:
        return unique_points
    
    # Строим выпуклую оболочку
    hull = []
    
    for point in unique_points:
        while (len(hull) > 1 and 
               cross_product(hull[-2], hull[-1], point) <= 0):
            hull.pop()
        hull.append(point)
    
    return hull

# Пример использования
points = [
    Point(0, 3), Point(1, 1), Point(2, 2), Point(4, 4),
    Point(0, 0), Point(1, 2), Point(3, 1), Point(3, 3)
]

hull = convex_hull_graham(points)
print(f"Выпуклая оболочка: {hull}")
```

---

## Аппроксимационные алгоритмы

### Задача коммивояжера (2-аппроксимация)

```python
import math
from collections import defaultdict

def tsp_approximation(graph):
    """2-аппроксимационный алгоритм для TSP"""
    
    def prim_mst(graph):
        """Алгоритм Прима для минимального остовного дерева"""
        vertices = list(graph.keys())
        if not vertices:
            return []
        
        mst = []
        visited = {vertices[0]}
        edges = []
        
        # Добавляем все ребра из начальной вершины
        for neighbor, weight in graph[vertices[0]].items():
            edges.append((weight, vertices[0], neighbor))
        
        edges.sort()
        
        while len(visited) < len(vertices):
            weight, u, v = edges.pop(0)
            
            if v not in visited:
                visited.add(v)
                mst.append((u, v, weight))
                
                # Добавляем новые ребра
                for neighbor, w in graph[v].items():
                    if neighbor not in visited:
                        edges.append((w, v, neighbor))
                        edges.sort()
        
        return mst
    
    def dfs_preorder(mst, start):
        """DFS для обхода MST в порядке preorder"""
        # Строим список смежности из MST
        adj = defaultdict(list)
        for u, v, weight in mst:
            adj[u].append(v)
            adj[v].append(u)
        
        visited = set()
        tour = []
        
        def dfs(node):
            visited.add(node)
            tour.append(node)
            for neighbor in adj[node]:
                if neighbor not in visited:
                    dfs(neighbor)
        
        dfs(start)
        return tour
    
    # Шаг 1: Находим MST
    mst = prim_mst(graph)
    
    # Шаг 2: Делаем DFS обход MST
    vertices = list(graph.keys())
    tour = dfs_preorder(mst, vertices[0])
    
    # Шаг 3: Замыкаем цикл
    if tour and tour[0] != tour[-1]:
        tour.append(tour[0])
    
    return tour

def calculate_tour_length(graph, tour):
    """Вычисление длины маршрута"""
    total_length = 0
    for i in range(len(tour) - 1):
        total_length += graph[tour[i]][tour[i + 1]]
    return total_length

# Пример использования
graph = {
    'A': {'B': 10, 'C': 15, 'D': 20},
    'B': {'A': 10, 'C': 35, 'D': 25},
    'C': {'A': 15, 'B': 35, 'D': 30},
    'D': {'A': 20, 'B': 25, 'C': 30}
}

tour = tsp_approximation(graph)
length = calculate_tour_length(graph, tour)
print(f"Приближенный маршрут: {' -> '.join(tour)}")
print(f"Длина маршрута: {length}")
```

---

## 🎯 Практические задания

### Задание 1: Система контроля версий
Реализуйте систему merge для Git используя алгоритмы на графах.

### Задание 2: Поисковая система
Создайте поисковый движок используя Trie и строковые алгоритмы.

### Задание 3: Маршрутизация
Реализуйте алгоритм поиска кратчайшего пути в дорожной сети.

### Задание 4: Планировщик задач
Создайте планировщик используя алгоритмы на графах и аппроксимацию.

---

## 🔗 Связанные материалы

- [Базовые алгоритмы](./algorithms.md)
- [Структуры данных](./data-structures.md)
- [Дискретная математика](../mathematics/discrete-mathematics.md)
- [Параллельное программирование](../technical-skills/concurrency-async.md) 