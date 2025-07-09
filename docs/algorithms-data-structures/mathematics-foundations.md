# Математические основы 🧮

## Содержание
1. [Линейная алгебра](#линейная-алгебра)
2. [Статистика и вероятность](#статистика-и-вероятность)
3. [Дискретная математика](#дискретная-математика)
4. [Теория графов](#теория-графов)
5. [Численные методы](#численные-методы)
6. [Теория информации](#теория-информации)

---

## Линейная алгебра

### Векторы и матрицы
```python
import numpy as np

class LinearAlgebra:
    """Основные операции линейной алгебры"""
    
    @staticmethod
    def dot_product(a, b):
        """Скалярное произведение векторов"""
        return np.sum(a * b)
    
    @staticmethod
    def vector_norm(v, p=2):
        """Норма вектора (L1, L2, L∞)"""
        if p == 1:
            return np.sum(np.abs(v))  # L1 норма
        elif p == 2:
            return np.sqrt(np.sum(v**2))  # L2 норма (Евклидова)
        elif p == float('inf'):
            return np.max(np.abs(v))  # L∞ норма
    
    @staticmethod
    def matrix_multiply(A, B):
        """Умножение матриц"""
        m, n = A.shape
        n_b, p = B.shape
        
        if n != n_b:
            raise ValueError("Размерности матриц не совпадают")
        
        C = np.zeros((m, p))
        for i in range(m):
            for j in range(p):
                for k in range(n):
                    C[i, j] += A[i, k] * B[k, j]
        return C
    
    @staticmethod
    def gaussian_elimination(A, b):
        """Решение системы линейных уравнений методом Гаусса"""
        n = len(A)
        
        # Прямой ход
        for i in range(n):
            # Поиск главного элемента
            max_row = i
            for k in range(i + 1, n):
                if abs(A[k][i]) > abs(A[max_row][i]):
                    max_row = k
            
            # Перестановка строк
            A[i], A[max_row] = A[max_row], A[i]
            b[i], b[max_row] = b[max_row], b[i]
            
            # Приведение к треугольному виду
            for k in range(i + 1, n):
                factor = A[k][i] / A[i][i]
                for j in range(i, n):
                    A[k][j] -= factor * A[i][j]
                b[k] -= factor * b[i]
        
        # Обратный ход
        x = [0] * n
        for i in range(n - 1, -1, -1):
            x[i] = b[i]
            for j in range(i + 1, n):
                x[i] -= A[i][j] * x[j]
            x[i] /= A[i][i]
        
        return x
    
    @staticmethod
    def eigenvalues_power_method(A, max_iter=1000, tol=1e-6):
        """Нахождение наибольшего собственного значения методом степенной итерации"""
        n = A.shape[0]
        v = np.random.rand(n)
        v = v / np.linalg.norm(v)
        
        for _ in range(max_iter):
            v_new = np.dot(A, v)
            eigenvalue = np.dot(v, v_new)
            v_new = v_new / np.linalg.norm(v_new)
            
            if np.linalg.norm(v_new - v) < tol:
                break
            
            v = v_new
        
        return eigenvalue, v
    
    @staticmethod
    def svd_decomposition(A):
        """Сингулярное разложение (упрощенная версия)"""
        # A = U * Σ * V^T
        # Используем встроенную функцию для демонстрации
        U, s, Vt = np.linalg.svd(A)
        
        # Создаем диагональную матрицу
        Sigma = np.zeros_like(A)
        min_dim = min(A.shape)
        Sigma[:min_dim, :min_dim] = np.diag(s)
        
        return U, Sigma, Vt
    
    @staticmethod
    def pca_analysis(X, n_components=2):
        """Анализ главных компонент"""
        # Центрирование данных
        X_centered = X - np.mean(X, axis=0)
        
        # Ковариационная матрица
        cov_matrix = np.cov(X_centered.T)
        
        # Собственные векторы и значения
        eigenvalues, eigenvectors = np.linalg.eig(cov_matrix)
        
        # Сортировка по убыванию собственных значений
        indices = np.argsort(eigenvalues)[::-1]
        eigenvalues = eigenvalues[indices]
        eigenvectors = eigenvectors[:, indices]
        
        # Выбираем первые n_components главных компонент
        principal_components = eigenvectors[:, :n_components]
        
        # Проекция данных
        X_pca = np.dot(X_centered, principal_components)
        
        # Объясненная дисперсия
        explained_variance_ratio = eigenvalues[:n_components] / np.sum(eigenvalues)
        
        return X_pca, principal_components, explained_variance_ratio

# Пример использования PCA
def demonstrate_pca():
    # Генерируем данные
    np.random.seed(42)
    X = np.random.randn(100, 5)
    
    # Применяем PCA
    la = LinearAlgebra()
    X_pca, components, variance_ratio = la.pca_analysis(X, n_components=2)
    
    print(f"Главные компоненты: {components.shape}")
    print(f"Объясненная дисперсия: {variance_ratio}")
    print(f"Сумма объясненной дисперсии: {np.sum(variance_ratio):.3f}")
```

---

## Статистика и вероятность

### Дескриптивная статистика
```python
class Statistics:
    """Основные статистические функции"""
    
    @staticmethod
    def mean(data):
        """Среднее арифметическое"""
        return sum(data) / len(data)
    
    @staticmethod
    def median(data):
        """Медиана"""
        sorted_data = sorted(data)
        n = len(sorted_data)
        if n % 2 == 0:
            return (sorted_data[n//2 - 1] + sorted_data[n//2]) / 2
        else:
            return sorted_data[n//2]
    
    @staticmethod
    def mode(data):
        """Мода"""
        from collections import Counter
        counts = Counter(data)
        max_count = max(counts.values())
        return [value for value, count in counts.items() if count == max_count]
    
    @staticmethod
    def variance(data, ddof=0):
        """Дисперсия"""
        mean_val = Statistics.mean(data)
        return sum((x - mean_val) ** 2 for x in data) / (len(data) - ddof)
    
    @staticmethod
    def standard_deviation(data, ddof=0):
        """Стандартное отклонение"""
        return np.sqrt(Statistics.variance(data, ddof))
    
    @staticmethod
    def correlation(x, y):
        """Коэффициент корреляции Пирсона"""
        n = len(x)
        sum_x = sum(x)
        sum_y = sum(y)
        sum_xy = sum(x[i] * y[i] for i in range(n))
        sum_x2 = sum(x[i] ** 2 for i in range(n))
        sum_y2 = sum(y[i] ** 2 for i in range(n))
        
        numerator = n * sum_xy - sum_x * sum_y
        denominator = np.sqrt((n * sum_x2 - sum_x ** 2) * (n * sum_y2 - sum_y ** 2))
        
        if denominator == 0:
            return 0
        
        return numerator / denominator
    
    @staticmethod
    def percentile(data, p):
        """Процентиль"""
        sorted_data = sorted(data)
        n = len(sorted_data)
        
        if p < 0 or p > 100:
            raise ValueError("Процентиль должен быть между 0 и 100")
        
        # Линейная интерполяция
        k = (n - 1) * p / 100
        floor_k = int(k)
        ceil_k = min(floor_k + 1, n - 1)
        
        if floor_k == ceil_k:
            return sorted_data[floor_k]
        
        d = k - floor_k
        return sorted_data[floor_k] * (1 - d) + sorted_data[ceil_k] * d
    
    @staticmethod
    def quartiles(data):
        """Квартили"""
        return {
            'Q1': Statistics.percentile(data, 25),
            'Q2': Statistics.percentile(data, 50),  # Медиана
            'Q3': Statistics.percentile(data, 75)
        }
    
    @staticmethod
    def outliers_iqr(data):
        """Выбросы по методу IQR"""
        q = Statistics.quartiles(data)
        iqr = q['Q3'] - q['Q1']
        lower_bound = q['Q1'] - 1.5 * iqr
        upper_bound = q['Q3'] + 1.5 * iqr
        
        outliers = [x for x in data if x < lower_bound or x > upper_bound]
        return outliers, lower_bound, upper_bound

class ProbabilityDistributions:
    """Распределения вероятностей"""
    
    @staticmethod
    def binomial_probability(n, k, p):
        """Биномиальное распределение"""
        from math import comb
        return comb(n, k) * (p ** k) * ((1 - p) ** (n - k))
    
    @staticmethod
    def poisson_probability(k, lambda_param):
        """Распределение Пуассона"""
        from math import factorial, exp
        return (lambda_param ** k) * exp(-lambda_param) / factorial(k)
    
    @staticmethod
    def normal_pdf(x, mu=0, sigma=1):
        """Плотность нормального распределения"""
        return (1 / (sigma * np.sqrt(2 * np.pi))) * np.exp(-0.5 * ((x - mu) / sigma) ** 2)
    
    @staticmethod
    def normal_cdf(x, mu=0, sigma=1):
        """Функция распределения нормального распределения"""
        return 0.5 * (1 + np.tanh((x - mu) / (sigma * np.sqrt(2))))
    
    @staticmethod
    def confidence_interval(data, confidence_level=0.95):
        """Доверительный интервал для среднего"""
        n = len(data)
        mean = Statistics.mean(data)
        std = Statistics.standard_deviation(data, ddof=1)
        
        # t-распределение для малых выборок
        from scipy.stats import t
        alpha = 1 - confidence_level
        t_value = t.ppf(1 - alpha/2, n - 1)
        
        margin_error = t_value * std / np.sqrt(n)
        
        return mean - margin_error, mean + margin_error
    
    @staticmethod
    def hypothesis_test_mean(sample, null_hypothesis_mean, alpha=0.05):
        """t-тест для среднего"""
        n = len(sample)
        sample_mean = Statistics.mean(sample)
        sample_std = Statistics.standard_deviation(sample, ddof=1)
        
        # t-статистика
        t_stat = (sample_mean - null_hypothesis_mean) / (sample_std / np.sqrt(n))
        
        # Критическое значение
        from scipy.stats import t
        t_critical = t.ppf(1 - alpha/2, n - 1)
        
        # p-значение
        p_value = 2 * (1 - t.cdf(abs(t_stat), n - 1))
        
        reject_null = abs(t_stat) > t_critical
        
        return {
            't_statistic': t_stat,
            't_critical': t_critical,
            'p_value': p_value,
            'reject_null': reject_null,
            'conclusion': 'Отвергаем H0' if reject_null else 'Не отвергаем H0'
        }
```

---

## Дискретная математика

### Комбинаторика
```python
class Combinatorics:
    """Комбинаторные функции"""
    
    @staticmethod
    def factorial(n):
        """Факториал"""
        if n < 0:
            raise ValueError("Факториал не определен для отрицательных чисел")
        if n == 0 or n == 1:
            return 1
        return n * Combinatorics.factorial(n - 1)
    
    @staticmethod
    def permutations(n, r):
        """Размещения P(n,r) = n! / (n-r)!"""
        if r > n or r < 0:
            return 0
        return Combinatorics.factorial(n) // Combinatorics.factorial(n - r)
    
    @staticmethod
    def combinations(n, r):
        """Сочетания C(n,r) = n! / (r!(n-r)!)"""
        if r > n or r < 0:
            return 0
        if r == 0 or r == n:
            return 1
        
        # Оптимизация: используем меньшее из r и n-r
        r = min(r, n - r)
        
        result = 1
        for i in range(r):
            result = result * (n - i) // (i + 1)
        
        return result
    
    @staticmethod
    def stirling_second_kind(n, k):
        """Числа Стирлинга второго рода S(n,k)"""
        if n == 0 and k == 0:
            return 1
        if n == 0 or k == 0:
            return 0
        if k > n:
            return 0
        if k == 1 or k == n:
            return 1
        
        # Рекуррентное соотношение: S(n,k) = k*S(n-1,k) + S(n-1,k-1)
        return k * Combinatorics.stirling_second_kind(n - 1, k) + \
               Combinatorics.stirling_second_kind(n - 1, k - 1)
    
    @staticmethod
    def catalan_number(n):
        """n-е число Каталана"""
        if n <= 1:
            return 1
        
        # C(n) = (2n)! / ((n+1)! * n!)
        return Combinatorics.combinations(2 * n, n) // (n + 1)
    
    @staticmethod
    def fibonacci(n):
        """n-е число Фибоначчи"""
        if n <= 1:
            return n
        
        a, b = 0, 1
        for _ in range(2, n + 1):
            a, b = b, a + b
        
        return b
    
    @staticmethod
    def generate_permutations(items):
        """Генерация всех перестановок"""
        if len(items) <= 1:
            yield items
        else:
            for i in range(len(items)):
                for perm in Combinatorics.generate_permutations(items[:i] + items[i+1:]):
                    yield [items[i]] + perm
    
    @staticmethod
    def generate_combinations(items, r):
        """Генерация всех сочетаний размера r"""
        if r == 0:
            yield []
        elif r == len(items):
            yield items
        elif r > len(items):
            return
        else:
            # Включаем первый элемент
            for combo in Combinatorics.generate_combinations(items[1:], r - 1):
                yield [items[0]] + combo
            
            # Не включаем первый элемент
            for combo in Combinatorics.generate_combinations(items[1:], r):
                yield combo
    
    @staticmethod
    def generate_subsets(items):
        """Генерация всех подмножеств"""
        n = len(items)
        for i in range(2**n):
            subset = []
            for j in range(n):
                if i & (1 << j):
                    subset.append(items[j])
            yield subset

class SetTheory:
    """Теория множеств"""
    
    @staticmethod
    def union(set1, set2):
        """Объединение множеств"""
        return set1 | set2
    
    @staticmethod
    def intersection(set1, set2):
        """Пересечение множеств"""
        return set1 & set2
    
    @staticmethod
    def difference(set1, set2):
        """Разность множеств"""
        return set1 - set2
    
    @staticmethod
    def symmetric_difference(set1, set2):
        """Симметрическая разность"""
        return set1 ^ set2
    
    @staticmethod
    def cartesian_product(set1, set2):
        """Декартово произведение"""
        return {(a, b) for a in set1 for b in set2}
    
    @staticmethod
    def power_set(s):
        """Множество всех подмножеств"""
        items = list(s)
        power_set = set()
        
        for subset in Combinatorics.generate_subsets(items):
            power_set.add(frozenset(subset))
        
        return power_set
    
    @staticmethod
    def is_subset(set1, set2):
        """Проверка на подмножество"""
        return set1 <= set2
    
    @staticmethod
    def is_superset(set1, set2):
        """Проверка на надмножество"""
        return set1 >= set2
```

---

## Теория графов

### Основные алгоритмы на графах
```python
from collections import defaultdict, deque

class Graph:
    """Реализация графа"""
    
    def __init__(self, directed=False):
        self.directed = directed
        self.vertices = set()
        self.edges = defaultdict(list)
        self.edge_weights = {}
    
    def add_vertex(self, vertex):
        """Добавление вершины"""
        self.vertices.add(vertex)
    
    def add_edge(self, u, v, weight=1):
        """Добавление ребра"""
        self.vertices.add(u)
        self.vertices.add(v)
        self.edges[u].append(v)
        self.edge_weights[(u, v)] = weight
        
        if not self.directed:
            self.edges[v].append(u)
            self.edge_weights[(v, u)] = weight
    
    def bfs(self, start):
        """Обход в ширину"""
        visited = set()
        queue = deque([start])
        result = []
        
        while queue:
            vertex = queue.popleft()
            if vertex not in visited:
                visited.add(vertex)
                result.append(vertex)
                
                for neighbor in self.edges[vertex]:
                    if neighbor not in visited:
                        queue.append(neighbor)
        
        return result
    
    def dfs(self, start):
        """Обход в глубину"""
        visited = set()
        result = []
        
        def dfs_recursive(vertex):
            visited.add(vertex)
            result.append(vertex)
            
            for neighbor in self.edges[vertex]:
                if neighbor not in visited:
                    dfs_recursive(neighbor)
        
        dfs_recursive(start)
        return result
    
    def shortest_path_dijkstra(self, start, end):
        """Алгоритм Дейкстры для кратчайшего пути"""
        import heapq
        
        distances = {vertex: float('inf') for vertex in self.vertices}
        distances[start] = 0
        previous = {}
        
        pq = [(0, start)]
        
        while pq:
            current_distance, current_vertex = heapq.heappop(pq)
            
            if current_vertex == end:
                break
            
            if current_distance > distances[current_vertex]:
                continue
            
            for neighbor in self.edges[current_vertex]:
                weight = self.edge_weights.get((current_vertex, neighbor), 1)
                distance = current_distance + weight
                
                if distance < distances[neighbor]:
                    distances[neighbor] = distance
                    previous[neighbor] = current_vertex
                    heapq.heappush(pq, (distance, neighbor))
        
        # Восстановление пути
        path = []
        current = end
        while current is not None:
            path.append(current)
            current = previous.get(current)
        
        path.reverse()
        return path if path[0] == start else None
    
    def minimum_spanning_tree_kruskal(self):
        """Алгоритм Краскала для минимального остовного дерева"""
        # Объединяем все ребра
        edges = []
        for u in self.vertices:
            for v in self.edges[u]:
                if not self.directed or u < v:  # Избегаем дублирования в неориентированном графе
                    weight = self.edge_weights.get((u, v), 1)
                    edges.append((weight, u, v))
        
        # Сортируем ребра по весу
        edges.sort()
        
        # Система непересекающихся множеств
        parent = {}
        rank = {}
        
        def find(x):
            if x not in parent:
                parent[x] = x
                rank[x] = 0
            if parent[x] != x:
                parent[x] = find(parent[x])
            return parent[x]
        
        def union(x, y):
            px, py = find(x), find(y)
            if px == py:
                return False
            if rank[px] < rank[py]:
                px, py = py, px
            parent[py] = px
            if rank[px] == rank[py]:
                rank[px] += 1
            return True
        
        mst = []
        for weight, u, v in edges:
            if union(u, v):
                mst.append((u, v, weight))
        
        return mst
    
    def topological_sort(self):
        """Топологическая сортировка"""
        if not self.directed:
            raise ValueError("Топологическая сортировка только для ориентированных графов")
        
        in_degree = {vertex: 0 for vertex in self.vertices}
        
        # Подсчет входящих степеней
        for u in self.vertices:
            for v in self.edges[u]:
                in_degree[v] += 1
        
        # Очередь вершин с нулевой входящей степенью
        queue = deque([v for v in self.vertices if in_degree[v] == 0])
        result = []
        
        while queue:
            vertex = queue.popleft()
            result.append(vertex)
            
            for neighbor in self.edges[vertex]:
                in_degree[neighbor] -= 1
                if in_degree[neighbor] == 0:
                    queue.append(neighbor)
        
        if len(result) != len(self.vertices):
            raise ValueError("Граф содержит циклы")
        
        return result
    
    def has_cycle(self):
        """Проверка на наличие циклов"""
        if not self.directed:
            # Для неориентированного графа
            visited = set()
            
            def dfs_undirected(vertex, parent):
                visited.add(vertex)
                for neighbor in self.edges[vertex]:
                    if neighbor not in visited:
                        if dfs_undirected(neighbor, vertex):
                            return True
                    elif neighbor != parent:
                        return True
                return False
            
            for vertex in self.vertices:
                if vertex not in visited:
                    if dfs_undirected(vertex, None):
                        return True
            return False
        else:
            # Для ориентированного графа (поиск обратных ребер)
            WHITE, GRAY, BLACK = 0, 1, 2
            color = {vertex: WHITE for vertex in self.vertices}
            
            def dfs_directed(vertex):
                color[vertex] = GRAY
                for neighbor in self.edges[vertex]:
                    if color[neighbor] == GRAY:
                        return True
                    elif color[neighbor] == WHITE and dfs_directed(neighbor):
                        return True
                color[vertex] = BLACK
                return False
            
            for vertex in self.vertices:
                if color[vertex] == WHITE:
                    if dfs_directed(vertex):
                        return True
            return False
    
    def strongly_connected_components(self):
        """Сильно связные компоненты (алгоритм Косараю)"""
        if not self.directed:
            raise ValueError("SCC только для ориентированных графов")
        
        # Первый DFS для получения порядка завершения
        visited = set()
        finish_order = []
        
        def dfs1(vertex):
            visited.add(vertex)
            for neighbor in self.edges[vertex]:
                if neighbor not in visited:
                    dfs1(neighbor)
            finish_order.append(vertex)
        
        for vertex in self.vertices:
            if vertex not in visited:
                dfs1(vertex)
        
        # Создаем транспонированный граф
        transposed = Graph(directed=True)
        for vertex in self.vertices:
            transposed.add_vertex(vertex)
        
        for u in self.vertices:
            for v in self.edges[u]:
                transposed.add_edge(v, u)
        
        # Второй DFS в обратном порядке
        visited = set()
        scc_list = []
        
        def dfs2(vertex, component):
            visited.add(vertex)
            component.append(vertex)
            for neighbor in transposed.edges[vertex]:
                if neighbor not in visited:
                    dfs2(neighbor, component)
        
        for vertex in reversed(finish_order):
            if vertex not in visited:
                component = []
                dfs2(vertex, component)
                scc_list.append(component)
        
        return scc_list
```

---

## Численные методы

### Методы решения уравнений
```python
class NumericalMethods:
    """Численные методы"""
    
    @staticmethod
    def bisection_method(f, a, b, tol=1e-6, max_iter=1000):
        """Метод бисекции для поиска корня"""
        if f(a) * f(b) >= 0:
            raise ValueError("f(a) и f(b) должны иметь разные знаки")
        
        for _ in range(max_iter):
            c = (a + b) / 2
            
            if abs(f(c)) < tol or abs(b - a) < tol:
                return c
            
            if f(a) * f(c) < 0:
                b = c
            else:
                a = c
        
        raise ValueError("Не удалось найти корень за заданное количество итераций")
    
    @staticmethod
    def newton_method(f, df, x0, tol=1e-6, max_iter=1000):
        """Метод Ньютона для поиска корня"""
        x = x0
        
        for _ in range(max_iter):
            fx = f(x)
            
            if abs(fx) < tol:
                return x
            
            dfx = df(x)
            if abs(dfx) < 1e-12:
                raise ValueError("Производная близка к нулю")
            
            x = x - fx / dfx
        
        raise ValueError("Не удалось найти корень за заданное количество итераций")
    
    @staticmethod
    def trapezoidal_rule(f, a, b, n=1000):
        """Метод трапеций для численного интегрирования"""
        h = (b - a) / n
        result = 0.5 * (f(a) + f(b))
        
        for i in range(1, n):
            x = a + i * h
            result += f(x)
        
        return result * h
    
    @staticmethod
    def simpson_rule(f, a, b, n=1000):
        """Метод Симпсона для численного интегрирования"""
        if n % 2 == 1:
            n += 1  # n должно быть четным
        
        h = (b - a) / n
        result = f(a) + f(b)
        
        for i in range(1, n):
            x = a + i * h
            if i % 2 == 0:
                result += 2 * f(x)
            else:
                result += 4 * f(x)
        
        return result * h / 3
    
    @staticmethod
    def runge_kutta_4(f, y0, x0, x_end, h=0.1):
        """Метод Рунге-Кутта 4-го порядка для решения ОДУ"""
        x = x0
        y = y0
        x_values = [x]
        y_values = [y]
        
        while x < x_end:
            k1 = h * f(x, y)
            k2 = h * f(x + h/2, y + k1/2)
            k3 = h * f(x + h/2, y + k2/2)
            k4 = h * f(x + h, y + k3)
            
            y += (k1 + 2*k2 + 2*k3 + k4) / 6
            x += h
            
            x_values.append(x)
            y_values.append(y)
        
        return x_values, y_values
    
    @staticmethod
    def gradient_descent(f, grad_f, x0, learning_rate=0.01, tol=1e-6, max_iter=10000):
        """Градиентный спуск для минимизации функции"""
        x = np.array(x0)
        
        for i in range(max_iter):
            grad = grad_f(x)
            
            if np.linalg.norm(grad) < tol:
                return x, i
            
            x = x - learning_rate * grad
        
        return x, max_iter
    
    @staticmethod
    def monte_carlo_integration(f, a, b, n=1000000):
        """Метод Монте-Карло для численного интегрирования"""
        # Генерируем случайные точки
        x_random = np.random.uniform(a, b, n)
        
        # Вычисляем значения функции
        f_values = np.array([f(x) for x in x_random])
        
        # Оценка интеграла
        integral = (b - a) * np.mean(f_values)
        
        return integral

# Пример использования численных методов
def demonstrate_numerical_methods():
    # Поиск корня x^2 - 2 = 0
    f = lambda x: x**2 - 2
    df = lambda x: 2*x
    
    # Метод бисекции
    root_bisection = NumericalMethods.bisection_method(f, 0, 2)
    print(f"Корень (бисекция): {root_bisection}")
    
    # Метод Ньютона
    root_newton = NumericalMethods.newton_method(f, df, 1.0)
    print(f"Корень (Ньютон): {root_newton}")
    
    # Интегрирование x^2 от 0 до 1
    g = lambda x: x**2
    
    integral_trap = NumericalMethods.trapezoidal_rule(g, 0, 1)
    integral_simpson = NumericalMethods.simpson_rule(g, 0, 1)
    integral_mc = NumericalMethods.monte_carlo_integration(g, 0, 1)
    
    print(f"Интеграл (трапеции): {integral_trap}")
    print(f"Интеграл (Симпсон): {integral_simpson}")
    print(f"Интеграл (Монте-Карло): {integral_mc}")
    print(f"Точное значение: {1/3}")
```

---

## Теория информации

### Основные концепции
```python
class InformationTheory:
    """Теория информации"""
    
    @staticmethod
    def entropy(probabilities):
        """Энтропия Шеннона"""
        return -sum(p * np.log2(p) for p in probabilities if p > 0)
    
    @staticmethod
    def cross_entropy(p, q):
        """Перекрестная энтропия"""
        return -sum(p[i] * np.log2(q[i]) for i in range(len(p)) if p[i] > 0 and q[i] > 0)
    
    @staticmethod
    def kl_divergence(p, q):
        """Дивергенция Кульбака-Лейблера"""
        return sum(p[i] * np.log2(p[i] / q[i]) for i in range(len(p)) if p[i] > 0 and q[i] > 0)
    
    @staticmethod
    def mutual_information(joint_prob, marginal_x, marginal_y):
        """Взаимная информация"""
        mi = 0
        for i in range(len(marginal_x)):
            for j in range(len(marginal_y)):
                if joint_prob[i][j] > 0:
                    mi += joint_prob[i][j] * np.log2(joint_prob[i][j] / (marginal_x[i] * marginal_y[j]))
        return mi
    
    @staticmethod
    def huffman_coding(frequencies):
        """Кодирование Хаффмана"""
        import heapq
        
        # Создаем узлы дерева
        heap = [[freq, i, char] for i, (char, freq) in enumerate(frequencies.items())]
        heapq.heapify(heap)
        
        # Строим дерево
        while len(heap) > 1:
            lo = heapq.heappop(heap)
            hi = heapq.heappop(heap)
            
            for pair in lo[2:]:
                pair[1] = '0' + pair[1]
            for pair in hi[2:]:
                pair[1] = '1' + pair[1]
            
            heapq.heappush(heap, [lo[0] + hi[0]] + lo[2:] + hi[2:])
        
        # Извлекаем коды
        codes = {}
        for pair in heap[0][2:]:
            codes[pair[0]] = pair[1]
        
        return codes
    
    @staticmethod
    def compression_ratio(original_size, compressed_size):
        """Коэффициент сжатия"""
        return original_size / compressed_size
    
    @staticmethod
    def channel_capacity(snr_db):
        """Пропускная способность канала (формула Шеннона)"""
        snr_linear = 10 ** (snr_db / 10)
        return np.log2(1 + snr_linear)

# Пример использования теории информации
def demonstrate_information_theory():
    # Энтропия
    prob_dist = [0.5, 0.25, 0.125, 0.125]
    entropy = InformationTheory.entropy(prob_dist)
    print(f"Энтропия: {entropy:.3f} бит")
    
    # Кодирование Хаффмана
    frequencies = {'A': 0.5, 'B': 0.25, 'C': 0.125, 'D': 0.125}
    codes = InformationTheory.huffman_coding(frequencies)
    print(f"Коды Хаффмана: {codes}")
    
    # Средняя длина кода
    avg_length = sum(len(codes[char]) * freq for char, freq in frequencies.items())
    print(f"Средняя длина кода: {avg_length:.3f}")
    print(f"Эффективность: {entropy/avg_length:.3f}")
```

## Связь с другими разделами

- **[Алгоритмы](./algorithms.md)** - Математические алгоритмы
- **[Machine Learning](./machine-learning-ai.md)** - Математические основы ML
- **[Статистика](./statistics.md)** - Статистические методы
- **[Оптимизация](./optimization.md)** - Методы оптимизации

---

*Следующий раздел: [Практические применения →](./real-world-applications.md)* 