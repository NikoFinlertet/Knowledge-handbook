# Теория графов и структуры данных 🕸️

> **Навигация**: [[discrete-mathematics-algorithms|← Дискретная математика]] | [[mathematics-algorithms|Главная]] | [[linear-algebra-computation|Линейная алгебра →]]

## Содержание
1. [Представления графов](#🕸️-представления-графов)
2. [Алгоритмы обхода графов](#🔍-алгоритмы-обхода-графов)
3. [Кратчайшие пути](#📍-кратчайшие-пути)
4. [Практические применения](#🚀-практические-применения)

---

## 🕸️ Представления графов

**Математическая основа:** Граф G = (V, E), где V - множество вершин, E - множество рёбер

**Структуры данных:**
- Матрица смежности
- Список смежности  
- Список рёбер

**Когда использовать:**

| Структура | Память | Проверка рёбер | Итерация по соседям | Лучше всего для |
|-----------|---------|----------------|-------------------|-----------------|
| Матрица смежности | O(V²) | O(1) | O(V) | Плотные графы, частые запросы рёбер |
| Список смежности | O(V+E) | O(degree) | O(degree) | Разреженные графы, обходы |
| Список рёбер | O(E) | O(E) | O(E) | Алгоритмы над рёбрами (MST, сортировка) |

```python
from collections import defaultdict, deque
import heapq

class Graph:
    """Универсальное представление графа с различными реализациями"""
    
    def __init__(self, vertices, representation='adjacency_list'):
        self.V = vertices
        self.representation = representation
        
        if representation == 'adjacency_matrix':
            self.graph = [[0] * vertices for _ in range(vertices)]
        elif representation == 'adjacency_list':
            self.graph = defaultdict(list)
        elif representation == 'edge_list':
            self.edges = []
    
    def add_edge(self, u, v, weight=1, directed=False):
        """Добавить ребро в граф"""
        if self.representation == 'adjacency_matrix':
            self.graph[u][v] = weight
            if not directed:
                self.graph[v][u] = weight
        
        elif self.representation == 'adjacency_list':
            self.graph[u].append((v, weight))
            if not directed:
                self.graph[v].append((u, weight))
        
        elif self.representation == 'edge_list':
            self.edges.append((u, v, weight))
            if not directed:
                self.edges.append((v, u, weight))
    
    def _get_neighbors(self, vertex):
        """Получить соседей вершины независимо от представления"""
        if self.representation == 'adjacency_matrix':
            neighbors = []
            for i in range(self.V):
                if self.graph[vertex][i] != 0:
                    neighbors.append((i, self.graph[vertex][i]))
            return neighbors
        
        elif self.representation == 'adjacency_list':
            return self.graph[vertex]
        
        elif self.representation == 'edge_list':
            neighbors = []
            for u, v, weight in self.edges:
                if u == vertex:
                    neighbors.append((v, weight))
            return neighbors
```

---

## 🔍 Алгоритмы обхода графов

### Поиск в ширину (BFS)
**Применение:** Кратчайший путь в невзвешенном графе, проверка связности

```python
def bfs(self, start):
    """Поиск в ширину - применение теории очередей"""
    visited = [False] * self.V
    queue = deque([start])
    visited[start] = True
    result = []
    
    while queue:
        vertex = queue.popleft()
        result.append(vertex)
        
        # Получаем соседей в зависимости от представления
        neighbors = self._get_neighbors(vertex)
        
        for neighbor, _ in neighbors:
            if not visited[neighbor]:
                visited[neighbor] = True
                queue.append(neighbor)
    
    return result

def shortest_path_bfs(self, start, end):
    """Кратчайший путь в невзвешенном графе"""
    if start == end:
        return [start]
    
    visited = [False] * self.V
    queue = deque([(start, [start])])
    visited[start] = True
    
    while queue:
        vertex, path = queue.popleft()
        
        for neighbor, _ in self._get_neighbors(vertex):
            if neighbor == end:
                return path + [neighbor]
            
            if not visited[neighbor]:
                visited[neighbor] = True
                queue.append((neighbor, path + [neighbor]))
    
    return []  # Путь не найден
```

### Поиск в глубину (DFS)
**Применение:** Обнаружение циклов, топологическая сортировка, компоненты связности

```python
def dfs(self, start):
    """Поиск в глубину - применение стека"""
    visited = [False] * self.V
    stack = [start]
    result = []
    
    while stack:
        vertex = stack.pop()
        if not visited[vertex]:
            visited[vertex] = True
            result.append(vertex)
            
            neighbors = self._get_neighbors(vertex)
            # Добавляем в обратном порядке для правильного DFS
            for neighbor, _ in reversed(neighbors):
                if not visited[neighbor]:
                    stack.append(neighbor)
    
    return result

def detect_cycle_dfs(self):
    """Обнаружение циклов в ориентированном графе"""
    color = [0] * self.V  # 0: белый, 1: серый, 2: черный
    
    def dfs_visit(v):
        color[v] = 1  # Серый (в процессе обработки)
        
        for neighbor, _ in self._get_neighbors(v):
            if color[neighbor] == 1:  # Обратное ребро
                return True
            elif color[neighbor] == 0 and dfs_visit(neighbor):
                return True
        
        color[v] = 2  # Черный (обработан)
        return False
    
    for v in range(self.V):
        if color[v] == 0:
            if dfs_visit(v):
                return True
    
    return False

def topological_sort(self):
    """Топологическая сортировка (только для DAG)"""
    visited = [False] * self.V
    stack = []
    
    def dfs_topo(v):
        visited[v] = True
        
        for neighbor, _ in self._get_neighbors(v):
            if not visited[neighbor]:
                dfs_topo(neighbor)
        
        stack.append(v)
    
    for v in range(self.V):
        if not visited[v]:
            dfs_topo(v)
    
    return stack[::-1]  # Обращаем стек
```

---

## 📍 Кратчайшие пути

### Алгоритм Дейкстры
**Применение:** Кратчайшие пути из одной вершины во взвешенном графе с неотрицательными весами

```python
def dijkstra(self, start):
    """
    Алгоритм Дейкстры - применение приоритетной очереди
    Основан на принципе жадности и оптимальности подструктуры
    """
    dist = [float('inf')] * self.V
    dist[start] = 0
    pq = [(0, start)]  # (distance, vertex)
    visited = [False] * self.V
    
    while pq:
        current_dist, u = heapq.heappop(pq)
        
        if visited[u]:
            continue
        
        visited[u] = True
        
        for v, weight in self._get_neighbors(u):
            if not visited[v] and dist[u] + weight < dist[v]:
                dist[v] = dist[u] + weight
                heapq.heappush(pq, (dist[v], v))
    
    return dist

def dijkstra_with_path(self, start, end):
    """Дейкстра с восстановлением пути"""
    dist = [float('inf')] * self.V
    prev = [-1] * self.V
    dist[start] = 0
    pq = [(0, start)]
    visited = [False] * self.V
    
    while pq:
        current_dist, u = heapq.heappop(pq)
        
        if u == end:
            break
        
        if visited[u]:
            continue
        
        visited[u] = True
        
        for v, weight in self._get_neighbors(u):
            if not visited[v] and dist[u] + weight < dist[v]:
                dist[v] = dist[u] + weight
                prev[v] = u
                heapq.heappush(pq, (dist[v], v))
    
    # Восстановление пути
    path = []
    current = end
    while current != -1:
        path.append(current)
        current = prev[current]
    
    return path[::-1] if path[0] == end else [], dist[end]
```

### Алгоритм Флойда-Уоршелла
**Применение:** Кратчайшие пути между всеми парами вершин

```python
def floyd_warshall(self):
    """
    Алгоритм Флойда-Уоршелла для всех кратчайших путей
    Динамическое программирование: dist[i][j][k] = min(dist[i][j][k-1], dist[i][k][k-1] + dist[k][j][k-1])
    """
    # Инициализация матрицы расстояний
    dist = [[float('inf')] * self.V for _ in range(self.V)]
    
    # Расстояние от вершины до себя = 0
    for i in range(self.V):
        dist[i][i] = 0
    
    # Заполняем начальные расстояния из графа
    if self.representation == 'adjacency_matrix':
        for i in range(self.V):
            for j in range(self.V):
                if self.graph[i][j] != 0:
                    dist[i][j] = self.graph[i][j]
    else:
        for u in range(self.V):
            for v, weight in self._get_neighbors(u):
                dist[u][v] = weight
    
    # Основной алгоритм
    for k in range(self.V):
        for i in range(self.V):
            for j in range(self.V):
                if dist[i][k] + dist[k][j] < dist[i][j]:
                    dist[i][j] = dist[i][k] + dist[k][j]
    
    return dist
```

---

## 🚀 Практические применения

### Поиск компонент связности
```python
def connected_components(self):
    """Найти все компоненты связности в неориентированном графе"""
    visited = [False] * self.V
    components = []
    
    def dfs_component(start, component):
        visited[start] = True
        component.append(start)
        
        for neighbor, _ in self._get_neighbors(start):
            if not visited[neighbor]:
                dfs_component(neighbor, component)
    
    for v in range(self.V):
        if not visited[v]:
            component = []
            dfs_component(v, component)
            components.append(component)
    
    return components

def is_bipartite(self):
    """Проверка, является ли граф двудольным"""
    color = [-1] * self.V
    
    def dfs_bipartite(v, c):
        color[v] = c
        
        for neighbor, _ in self._get_neighbors(v):
            if color[neighbor] == -1:
                if not dfs_bipartite(neighbor, 1 - c):
                    return False
            elif color[neighbor] == color[v]:
                return False
        
        return True
    
    for v in range(self.V):
        if color[v] == -1:
            if not dfs_bipartite(v, 0):
                return False
    
    return True
```

### Минимальное остовное дерево (MST)
```python
class UnionFind:
    """Система непересекающихся множеств для алгоритма Крускала"""
    
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # Сжатие пути
        return self.parent[x]
    
    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py:
            return False
        
        # Объединение по рангу
        if self.rank[px] < self.rank[py]:
            px, py = py, px
        
        self.parent[py] = px
        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1
        
        return True

def kruskal_mst(self):
    """Алгоритм Крускала для поиска MST"""
    if self.representation != 'edge_list':
        # Преобразуем в список рёбер
        edges = []
        for u in range(self.V):
            for v, weight in self._get_neighbors(u):
                if u < v:  # Избегаем дублирования для неориентированного графа
                    edges.append((weight, u, v))
    else:
        edges = [(weight, u, v) for u, v, weight in self.edges]
    
    # Сортируем рёбра по весу
    edges.sort()
    
    uf = UnionFind(self.V)
    mst = []
    total_weight = 0
    
    for weight, u, v in edges:
        if uf.union(u, v):
            mst.append((u, v, weight))
            total_weight += weight
            
            if len(mst) == self.V - 1:
                break
    
    return mst, total_weight

def prim_mst(self, start=0):
    """Алгоритм Прима для поиска MST"""
    visited = [False] * self.V
    min_heap = [(0, start, -1)]  # (weight, vertex, parent)
    mst = []
    total_weight = 0
    
    while min_heap:
        weight, u, parent = heapq.heappop(min_heap)
        
        if visited[u]:
            continue
        
        visited[u] = True
        if parent != -1:
            mst.append((parent, u, weight))
            total_weight += weight
        
        for v, edge_weight in self._get_neighbors(u):
            if not visited[v]:
                heapq.heappush(min_heap, (edge_weight, v, u))
    
    return mst, total_weight
```

---

## 🔗 Связанные темы

- [[discrete-mathematics-algorithms|Дискретная математика]] - битовые операции и комбинаторика
- [[linear-algebra-computation|Линейная алгебра]] - матричные представления графов
- [[probability-randomized-algorithms|Вероятностные алгоритмы]] - случайные графы и сети
- [[combinatorics-generation|Комбинаторика]] - перечисление путей и циклов

## 📚 Дополнительные ресурсы

- **Книги**: "Introduction to Algorithms" - Cormen, Leiserson, Rivest, Stein
- **Курсы**: Princeton Algorithms, Stanford CS161
- **Практика**: LeetCode Graph section, Codeforces graph problems 