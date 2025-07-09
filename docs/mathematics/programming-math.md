# Основы математики для программирования

## 🎯 Зачем математика программисту?

### Фундаментальная необходимость
Математика - это **язык алгоритмов**. Без понимания математических основ невозможно:
- Анализировать сложность алгоритмов
- Создавать эффективные структуры данных
- Работать с графикой и играми
- Понимать криптографию и безопасность
- Разрабатывать ML и AI системы

### Практическое применение в жизни
```
Реальные задачи:
├── Оптимизация маршрутов в картах (теория графов)
├── Рекомендательные системы (линейная алгебра)
├── Компьютерная графика (геометрия, тригонометрия)
├── Сжатие данных (теория информации)
├── Поисковые системы (алгебра, теория вероятностей)
└── Игровая физика (векторы, матрицы)
```

## 📚 Дискретная математика

### 1. Логика и множества

#### Основы логики
```
Логические операции:
- И (AND): A ∧ B - истина только если оба A и B истинны
- ИЛИ (OR): A ∨ B - истина если хотя бы одно из A или B истинно
- НЕ (NOT): ¬A - противоположность A
- Импликация: A → B - "если A, то B"
- Эквиваленция: A ↔ B - "A тогда и только тогда, когда B"

Практический пример (условия доступа):
```python
def can_access_admin_panel(user):
    is_admin = user.role == 'admin'
    is_authenticated = user.is_authenticated
    has_permission = user.has_permission('admin_access')
    
    # Логическое И: все условия должны быть истинными
    return is_admin and is_authenticated and has_permission

def should_show_content(user, content):
    is_public = content.is_public
    is_owner = content.owner_id == user.id
    has_subscription = user.has_subscription
    
    # Логическое ИЛИ: достаточно одного условия
    return is_public or is_owner or has_subscription
```

#### Теория множеств
```
Основные операции:
- Объединение: A ∪ B - все элементы из A или B
- Пересечение: A ∩ B - элементы принадлежащие и A, и B
- Разность: A \ B - элементы из A, не принадлежащие B
- Дополнение: A' - элементы не принадлежащие A

Практический пример (права доступа):
```python
# Множества пользователей
admins = {'alice', 'bob', 'carol'}
editors = {'bob', 'carol', 'david'}
viewers = {'carol', 'david', 'eve', 'frank'}

# Объединение - все пользователи с правами
all_users = admins | editors | viewers
print(all_users)  # {'alice', 'bob', 'carol', 'david', 'eve', 'frank'}

# Пересечение - пользователи с несколькими ролями
admin_editors = admins & editors
print(admin_editors)  # {'bob', 'carol'}

# Разность - только админы
only_admins = admins - editors
print(only_admins)  # {'alice'}

# Проверка принадлежности
def get_user_permissions(user):
    permissions = set()
    if user in admins:
        permissions.add('admin')
    if user in editors:
        permissions.add('edit')
    if user in viewers:
        permissions.add('view')
    return permissions
```

### 2. Системы счисления

#### Двоичная система
```
Применение в программировании:
- Битовые операции
- Флаги и маски
- Оптимизация памяти
- Криптография

Практический пример (битовые флаги):
```python
# Права доступа как битовые флаги
READ = 1    # 001
WRITE = 2   # 010  
EXECUTE = 4 # 100

class FilePermissions:
    def __init__(self, permissions=0):
        self.permissions = permissions
    
    def add_permission(self, permission):
        self.permissions |= permission  # Битовое ИЛИ
    
    def remove_permission(self, permission):
        self.permissions &= ~permission  # Битовое И с отрицанием
    
    def has_permission(self, permission):
        return bool(self.permissions & permission)  # Битовое И
    
    def __str__(self):
        perms = []
        if self.has_permission(READ):
            perms.append('read')
        if self.has_permission(WRITE):
            perms.append('write')
        if self.has_permission(EXECUTE):
            perms.append('execute')
        return ', '.join(perms)

# Использование
file_perms = FilePermissions()
file_perms.add_permission(READ | WRITE)  # Добавляем чтение и запись
print(file_perms)  # "read, write"
print(file_perms.has_permission(EXECUTE))  # False
```

#### Шестнадцатеричная система
```
Применение:
- Цвета в веб-дизайне
- Адреса памяти
- Хеш-суммы
- Криптографические ключи

Практический пример (работа с цветами):
```python
def hex_to_rgb(hex_color):
    """Преобразование hex цвета в RGB"""
    hex_color = hex_color.lstrip('#')
    return tuple(int(hex_color[i:i+2], 16) for i in (0, 2, 4))

def rgb_to_hex(r, g, b):
    """Преобразование RGB в hex"""
    return f"#{r:02x}{g:02x}{b:02x}"

# Использование
color_hex = "#FF5733"
r, g, b = hex_to_rgb(color_hex)
print(f"RGB: {r}, {g}, {b}")  # RGB: 255, 87, 51

# Создание палитры
def create_gradient(start_color, end_color, steps):
    start_rgb = hex_to_rgb(start_color)
    end_rgb = hex_to_rgb(end_color)
    
    gradient = []
    for i in range(steps):
        ratio = i / (steps - 1)
        r = int(start_rgb[0] + (end_rgb[0] - start_rgb[0]) * ratio)
        g = int(start_rgb[1] + (end_rgb[1] - start_rgb[1]) * ratio)
        b = int(start_rgb[2] + (end_rgb[2] - start_rgb[2]) * ratio)
        gradient.append(rgb_to_hex(r, g, b))
    
    return gradient
```

### 3. Комбинаторика

#### Размещения, сочетания, перестановки
```
Формулы:
- Размещения: A(n,k) = n!/(n-k)!
- Сочетания: C(n,k) = n!/(k!(n-k)!)
- Перестановки: P(n) = n!

Практический пример (генерация паролей):
```python
import itertools
import math

def calculate_password_strength(length, charset_size):
    """Вычисление количества возможных паролей"""
    return charset_size ** length

def generate_combinations(items, r):
    """Генерация всех сочетаний"""
    return list(itertools.combinations(items, r))

def generate_permutations(items, r=None):
    """Генерация всех перестановок"""
    return list(itertools.permutations(items, r))

# Анализ паролей
lowercase = 26
uppercase = 26
digits = 10
special = 10

total_chars = lowercase + uppercase + digits + special  # 72
password_length = 8

total_combinations = calculate_password_strength(password_length, total_chars)
print(f"Возможных паролей: {total_combinations:,}")

# Время взлома при 1 млн попыток в секунду
seconds_to_crack = total_combinations / (2 * 1_000_000)  # В среднем половина
years_to_crack = seconds_to_crack / (365 * 24 * 3600)
print(f"Время взлома: {years_to_crack:.2f} лет")
```

#### Принцип Дирихле
```
Формулировка: Если n объектов помещено в m контейнеров, 
где n > m, то в каком-то контейнере окажется более одного объекта.

Практический пример (хеширование):
```python
def analyze_hash_collisions(hash_function, input_size, hash_size):
    """Анализ вероятности коллизий хеш-функции"""
    
    # Парадокс дня рождения
    def birthday_paradox(n, d):
        """Вероятность коллизии при n элементах и d возможных значений"""
        if n > d:
            return 1.0
        
        prob_no_collision = 1.0
        for i in range(n):
            prob_no_collision *= (d - i) / d
        
        return 1 - prob_no_collision
    
    collision_prob = birthday_paradox(input_size, 2**hash_size)
    return collision_prob

# Анализ MD5 (128 бит)
md5_collision_prob = analyze_hash_collisions(
    hash_function="MD5",
    input_size=2**64,  # 2^64 входных данных
    hash_size=128      # 128-битный хеш
)

print(f"Вероятность коллизии MD5: {md5_collision_prob:.6f}")
```

### 4. Теория графов

#### Основы графов
```
Определения:
- Граф G = (V, E) - множество вершин V и рёбер E
- Степень вершины - количество инцидентных рёбер
- Путь - последовательность вершин, соединённых рёбрами
- Цикл - замкнутый путь
- Связность - существование пути между любыми двумя вершинами

Практический пример (социальная сеть):
```python
class Graph:
    def __init__(self):
        self.vertices = {}
    
    def add_vertex(self, vertex):
        if vertex not in self.vertices:
            self.vertices[vertex] = []
    
    def add_edge(self, vertex1, vertex2):
        self.add_vertex(vertex1)
        self.add_vertex(vertex2)
        self.vertices[vertex1].append(vertex2)
        self.vertices[vertex2].append(vertex1)
    
    def get_degree(self, vertex):
        return len(self.vertices.get(vertex, []))
    
    def find_path(self, start, end, path=[]):
        """Поиск пути между вершинами"""
        path = path + [start]
        if start == end:
            return path
        
        for neighbor in self.vertices.get(start, []):
            if neighbor not in path:
                new_path = self.find_path(neighbor, end, path)
                if new_path:
                    return new_path
        return None
    
    def find_shortest_path(self, start, end):
        """Поиск кратчайшего пути (BFS)"""
        from collections import deque
        
        queue = deque([(start, [start])])
        visited = set([start])
        
        while queue:
            vertex, path = queue.popleft()
            
            if vertex == end:
                return path
            
            for neighbor in self.vertices.get(vertex, []):
                if neighbor not in visited:
                    visited.add(neighbor)
                    queue.append((neighbor, path + [neighbor]))
        
        return None

# Создание социальной сети
social_network = Graph()
social_network.add_edge("Alice", "Bob")
social_network.add_edge("Alice", "Carol")
social_network.add_edge("Bob", "David")
social_network.add_edge("Carol", "Eve")
social_network.add_edge("David", "Eve")

# Анализ сети
print(f"Степень Alice: {social_network.get_degree('Alice')}")  # 2
path = social_network.find_shortest_path("Alice", "Eve")
print(f"Кратчайший путь от Alice к Eve: {path}")  # ['Alice', 'Carol', 'Eve']
```

#### Алгоритмы на графах
```
Классические алгоритмы:
- DFS (Depth-First Search) - поиск в глубину
- BFS (Breadth-First Search) - поиск в ширину
- Dijkstra - кратчайшие пути
- Floyd-Warshall - все кратчайшие пути
- Kruskal/Prim - минимальное остовное дерево

Практический пример (система маршрутизации):
```python
import heapq

class WeightedGraph:
    def __init__(self):
        self.vertices = {}
    
    def add_edge(self, from_vertex, to_vertex, weight):
        if from_vertex not in self.vertices:
            self.vertices[from_vertex] = []
        self.vertices[from_vertex].append((to_vertex, weight))
    
    def dijkstra(self, start):
        """Алгоритм Дейкстры для поиска кратчайших путей"""
        distances = {vertex: float('inf') for vertex in self.vertices}
        distances[start] = 0
        pq = [(0, start)]
        visited = set()
        
        while pq:
            current_distance, current_vertex = heapq.heappop(pq)
            
            if current_vertex in visited:
                continue
            
            visited.add(current_vertex)
            
            for neighbor, weight in self.vertices.get(current_vertex, []):
                distance = current_distance + weight
                
                if distance < distances[neighbor]:
                    distances[neighbor] = distance
                    heapq.heappush(pq, (distance, neighbor))
        
        return distances

# Создание карты города
city_map = WeightedGraph()
city_map.add_edge("Home", "Work", 10)
city_map.add_edge("Home", "School", 5)
city_map.add_edge("School", "Work", 3)
city_map.add_edge("Work", "Mall", 7)
city_map.add_edge("School", "Mall", 12)

# Поиск кратчайших путей
distances = city_map.dijkstra("Home")
print("Кратчайшие расстояния от дома:")
for place, distance in distances.items():
    print(f"  {place}: {distance} км")
```

## 📐 Геометрия и тригонометрия

### 1. Векторы

#### Основы векторов
```
Операции с векторами:
- Сложение: a + b = (a₁ + b₁, a₂ + b₂)
- Скалярное произведение: a · b = |a| |b| cos θ
- Векторное произведение: a × b (для 3D)
- Длина вектора: |a| = √(a₁² + a₂²)

Практический пример (игровая физика):
```python
import math

class Vector2D:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __add__(self, other):
        return Vector2D(self.x + other.x, self.y + other.y)
    
    def __sub__(self, other):
        return Vector2D(self.x - other.x, self.y - other.y)
    
    def __mul__(self, scalar):
        return Vector2D(self.x * scalar, self.y * scalar)
    
    def length(self):
        return math.sqrt(self.x ** 2 + self.y ** 2)
    
    def normalize(self):
        """Нормализация вектора (единичный вектор)"""
        length = self.length()
        if length == 0:
            return Vector2D(0, 0)
        return Vector2D(self.x / length, self.y / length)
    
    def dot(self, other):
        """Скалярное произведение"""
        return self.x * other.x + self.y * other.y
    
    def angle_to(self, other):
        """Угол между векторами"""
        dot_product = self.dot(other)
        lengths = self.length() * other.length()
        if lengths == 0:
            return 0
        return math.acos(dot_product / lengths)
    
    def __repr__(self):
        return f"Vector2D({self.x}, {self.y})"

# Игровая физика - движение персонажа
class Player:
    def __init__(self, position, velocity):
        self.position = position
        self.velocity = velocity
    
    def update(self, dt):
        """Обновление позиции"""
        self.position = self.position + self.velocity * dt
    
    def move_towards(self, target, speed):
        """Движение к цели"""
        direction = (target - self.position).normalize()
        self.velocity = direction * speed

# Использование
player = Player(Vector2D(0, 0), Vector2D(1, 0))
target = Vector2D(10, 10)

player.move_towards(target, 5)
print(f"Скорость: {player.velocity}")  # Нормализованное направление * 5

# Обновление позиции
player.update(0.1)  # dt = 0.1 секунды
print(f"Новая позиция: {player.position}")
```

### 2. Матрицы

#### Основы матриц
```
Операции с матрицами:
- Сложение: A + B
- Умножение: A × B
- Транспонирование: Aᵀ
- Определитель: det(A)
- Обратная матрица: A⁻¹

Практический пример (компьютерная графика):
```python
import math

class Matrix2D:
    def __init__(self, values):
        self.values = values  # 2x2 матрица как список списков
    
    def __mul__(self, other):
        """Умножение матриц"""
        if isinstance(other, Matrix2D):
            result = [[0, 0], [0, 0]]
            for i in range(2):
                for j in range(2):
                    for k in range(2):
                        result[i][j] += self.values[i][k] * other.values[k][j]
            return Matrix2D(result)
        elif isinstance(other, Vector2D):
            # Умножение матрицы на вектор
            x = self.values[0][0] * other.x + self.values[0][1] * other.y
            y = self.values[1][0] * other.x + self.values[1][1] * other.y
            return Vector2D(x, y)
    
    @staticmethod
    def rotation(angle):
        """Матрица поворота"""
        cos_a = math.cos(angle)
        sin_a = math.sin(angle)
        return Matrix2D([
            [cos_a, -sin_a],
            [sin_a, cos_a]
        ])
    
    @staticmethod
    def scale(sx, sy):
        """Матрица масштабирования"""
        return Matrix2D([
            [sx, 0],
            [0, sy]
        ])
    
    @staticmethod
    def identity():
        """Единичная матрица"""
        return Matrix2D([
            [1, 0],
            [0, 1]
        ])

# Трансформации в 2D графике
def transform_shape(points, transformations):
    """Применение трансформаций к набору точек"""
    # Комбинирование трансформаций
    combined_transform = Matrix2D.identity()
    for transform in transformations:
        combined_transform = combined_transform * transform
    
    # Применение к точкам
    transformed_points = []
    for point in points:
        transformed_point = combined_transform * point
        transformed_points.append(transformed_point)
    
    return transformed_points

# Создание квадрата
square = [
    Vector2D(0, 0),
    Vector2D(1, 0),
    Vector2D(1, 1),
    Vector2D(0, 1)
]

# Трансформации: поворот на 45° и масштабирование в 2 раза
transformations = [
    Matrix2D.rotation(math.pi / 4),  # 45 градусов
    Matrix2D.scale(2, 2)
]

transformed_square = transform_shape(square, transformations)
print("Трансформированный квадрат:")
for i, point in enumerate(transformed_square):
    print(f"  Точка {i}: {point}")
```

### 3. Тригонометрия

#### Основы тригонометрии
```
Основные функции:
- sin(θ) = противолежащий катет / гипотенуза
- cos(θ) = прилежащий катет / гипотенуза
- tan(θ) = противолежащий катет / прилежащий катет

Практический пример (анимация и игры):
```python
import math
import time

class Animation:
    def __init__(self, amplitude=1, frequency=1, phase=0):
        self.amplitude = amplitude
        self.frequency = frequency
        self.phase = phase
    
    def sine_wave(self, t):
        """Синусоидальная анимация"""
        return self.amplitude * math.sin(2 * math.pi * self.frequency * t + self.phase)
    
    def cosine_wave(self, t):
        """Косинусоидальная анимация"""
        return self.amplitude * math.cos(2 * math.pi * self.frequency * t + self.phase)
    
    def circular_motion(self, t, radius=1):
        """Круговое движение"""
        angle = 2 * math.pi * self.frequency * t + self.phase
        x = radius * math.cos(angle)
        y = radius * math.sin(angle)
        return Vector2D(x, y)

# Система частиц с тригонометрической анимацией
class Particle:
    def __init__(self, position, animation):
        self.initial_position = position
        self.animation = animation
    
    def get_position(self, t):
        """Получение позиции в момент времени t"""
        offset = self.animation.circular_motion(t, radius=2)
        return self.initial_position + offset

# Создание системы частиц
particles = []
for i in range(8):
    angle = i * math.pi / 4  # Равномерно распределенные по кругу
    pos = Vector2D(math.cos(angle) * 5, math.sin(angle) * 5)
    anim = Animation(frequency=0.5, phase=i * math.pi / 4)
    particles.append(Particle(pos, anim))

# Симуляция анимации
def simulate_particles(duration=10, fps=30):
    dt = 1.0 / fps
    for frame in range(int(duration * fps)):
        t = frame * dt
        print(f"Кадр {frame} (t={t:.2f}s):")
        for i, particle in enumerate(particles):
            pos = particle.get_position(t)
            print(f"  Частица {i}: ({pos.x:.2f}, {pos.y:.2f})")
        time.sleep(dt)

# Запуск симуляции (закомментировано для статического примера)
# simulate_particles(duration=2, fps=10)
```

## 📊 Линейная алгебра

### 1. Системы линейных уравнений

#### Методы решения
```
Методы:
- Метод Гаусса
- Правило Крамера
- Матричный метод
- Итерационные методы

Практический пример (балансировка нагрузки):
```python
import numpy as np

def solve_load_balancing(servers, requests):
    """
    Решение задачи балансировки нагрузки
    servers: [(capacity, cost_per_request), ...]
    requests: [request_count_per_hour, ...]
    """
    n_servers = len(servers)
    n_hours = len(requests)
    
    # Создаем матрицу ограничений
    # x[i][j] = количество запросов на сервер i в час j
    
    # Ограничения по мощности серверов
    A = []
    b = []
    
    for i, (capacity, _) in enumerate(servers):
        for j in range(n_hours):
            constraint = [0] * (n_servers * n_hours)
            constraint[i * n_hours + j] = 1
            A.append(constraint)
            b.append(capacity)
    
    # Ограничения по обслуживанию всех запросов
    for j in range(n_hours):
        constraint = [0] * (n_servers * n_hours)
        for i in range(n_servers):
            constraint[i * n_hours + j] = 1
        A.append(constraint)
        b.append(requests[j])
    
    # Решение системы (упрощенный пример)
    A = np.array(A)
    b = np.array(b)
    
    # Используем метод наименьших квадратов для переопределенной системы
    x = np.linalg.lstsq(A, b, rcond=None)[0]
    
    return x.reshape(n_servers, n_hours)

# Пример использования
servers = [(100, 0.1), (150, 0.08), (200, 0.12)]  # (мощность, стоимость)
requests = [80, 120, 90, 110, 95]  # запросы по часам

allocation = solve_load_balancing(servers, requests)
print("Распределение нагрузки:")
for i, (capacity, cost) in enumerate(servers):
    print(f"Сервер {i} (мощность {capacity}):")
    for j, allocation_value in enumerate(allocation[i]):
        print(f"  Час {j}: {allocation_value:.1f} запросов")
```

### 2. Eigenvalues и Eigenvectors

#### Собственные значения и векторы
```
Применение:
- Анализ главных компонент (PCA)
- Поисковые алгоритмы (PageRank)
- Рекомендательные системы
- Обработка изображений

Практический пример (анализ социальной сети):
```python
import numpy as np
from scipy.linalg import eig

def analyze_social_network(adjacency_matrix):
    """
    Анализ социальной сети через собственные векторы
    adjacency_matrix: матрица смежности графа
    """
    # Нормализация матрицы (аналог Google PageRank)
    n = adjacency_matrix.shape[0]
    
    # Создание матрицы переходов
    row_sums = adjacency_matrix.sum(axis=1)
    transition_matrix = np.zeros_like(adjacency_matrix, dtype=float)
    
    for i in range(n):
        if row_sums[i] > 0:
            transition_matrix[i] = adjacency_matrix[i] / row_sums[i]
    
    # Поиск собственных значений и векторов
    eigenvalues, eigenvectors = eig(transition_matrix.T)
    
    # Поиск главного собственного вектора (максимальное собственное значение)
    max_eigenvalue_idx = np.argmax(eigenvalues.real)
    principal_eigenvector = eigenvectors[:, max_eigenvalue_idx].real
    
    # Нормализация для получения вероятностей
    principal_eigenvector = np.abs(principal_eigenvector)
    principal_eigenvector = principal_eigenvector / principal_eigenvector.sum()
    
    return principal_eigenvector, eigenvalues

# Пример: анализ влиятельности пользователей
# Матрица смежности (кто на кого подписан)
social_network = np.array([
    [0, 1, 1, 0, 0],  # User 0 подписан на 1, 2
    [1, 0, 1, 1, 0],  # User 1 подписан на 0, 2, 3
    [1, 1, 0, 1, 1],  # User 2 подписан на всех остальных
    [0, 1, 1, 0, 1],  # User 3 подписан на 1, 2, 4
    [0, 0, 1, 1, 0]   # User 4 подписан на 2, 3
])

influence_scores, eigenvalues = analyze_social_network(social_network)

print("Оценки влиятельности пользователей:")
for i, score in enumerate(influence_scores):
    print(f"Пользователь {i}: {score:.3f}")

# Ранжирование по влиятельности
ranked_users = sorted(enumerate(influence_scores), key=lambda x: x[1], reverse=True)
print("\nРанжирование по влиятельности:")
for rank, (user, score) in enumerate(ranked_users, 1):
    print(f"{rank}. Пользователь {user}: {score:.3f}")
```

## 🔢 Численные методы

### 1. Приближенные вычисления

#### Метод Ньютона
```python
def newton_method(f, df, x0, tolerance=1e-10, max_iterations=100):
    """
    Метод Ньютона для поиска корней функции
    f: функция
    df: производная функции
    x0: начальное приближение
    """
    x = x0
    for i in range(max_iterations):
        fx = f(x)
        if abs(fx) < tolerance:
            return x, i
        
        dfx = df(x)
        if abs(dfx) < 1e-15:
            raise ValueError("Производная близка к нулю")
        
        x = x - fx / dfx
    
    raise ValueError("Не удалось найти корень за максимальное количество итераций")

# Пример: поиск квадратного корня
def sqrt_newton(a, x0=None):
    """Вычисление квадратного корня методом Ньютона"""
    if x0 is None:
        x0 = a / 2
    
    f = lambda x: x**2 - a
    df = lambda x: 2*x
    
    root, iterations = newton_method(f, df, x0)
    return root, iterations

# Использование
number = 25
sqrt_result, iterations = sqrt_newton(number)
print(f"√{number} = {sqrt_result:.10f} (найден за {iterations} итераций)")
print(f"Проверка: {sqrt_result**2:.10f}")
```

### 2. Интегрирование

#### Метод трапеций
```python
def trapezoidal_rule(f, a, b, n):
    """
    Численное интегрирование методом трапеций
    f: функция
    a, b: пределы интегрирования
    n: количество разбиений
    """
    h = (b - a) / n
    result = 0.5 * (f(a) + f(b))
    
    for i in range(1, n):
        x = a + i * h
        result += f(x)
    
    return result * h

# Пример: вычисление площади под кривой
def calculate_area_under_curve(data_points):
    """Вычисление площади под кривой по точкам данных"""
    def interpolate(x):
        # Простая линейная интерполяция
        for i in range(len(data_points) - 1):
            x1, y1 = data_points[i]
            x2, y2 = data_points[i + 1]
            if x1 <= x <= x2:
                return y1 + (y2 - y1) * (x - x1) / (x2 - x1)
        return 0
    
    if len(data_points) < 2:
        return 0
    
    a = data_points[0][0]
    b = data_points[-1][0]
    
    return trapezoidal_rule(interpolate, a, b, 1000)

# Пример: анализ трафика веб-сайта
traffic_data = [
    (0, 100),   # 0:00 - 100 пользователей
    (4, 50),    # 4:00 - 50 пользователей
    (8, 200),   # 8:00 - 200 пользователей
    (12, 300),  # 12:00 - 300 пользователей
    (16, 250),  # 16:00 - 250 пользователей
    (20, 150),  # 20:00 - 150 пользователей
    (24, 100)   # 24:00 - 100 пользователей
]

total_user_hours = calculate_area_under_curve(traffic_data)
print(f"Общее количество пользователь-часов: {total_user_hours:.2f}")
```

## 🎯 Практические применения

### 1. Криптография

#### Шифры и хеширование
```python
def simple_cipher(text, key):
    """Простой шифр Цезаря"""
    result = ""
    for char in text:
        if char.isalpha():
            ascii_offset = ord('A') if char.isupper() else ord('a')
            shifted = (ord(char) - ascii_offset + key) % 26
            result += chr(shifted + ascii_offset)
        else:
            result += char
    return result

def gcd(a, b):
    """Наибольший общий делитель (алгоритм Евклида)"""
    while b:
        a, b = b, a % b
    return a

def mod_inverse(a, m):
    """Модульное обращение"""
    if gcd(a, m) != 1:
        return None
    
    # Расширенный алгоритм Евклида
    def extended_gcd(a, b):
        if a == 0:
            return b, 0, 1
        gcd, x1, y1 = extended_gcd(b % a, a)
        x = y1 - (b // a) * x1
        y = x1
        return gcd, x, y
    
    _, x, _ = extended_gcd(a, m)
    return (x % m + m) % m

# Простая реализация RSA (для демонстрации)
def generate_rsa_keys(p, q):
    """Генерация ключей RSA"""
    n = p * q
    phi = (p - 1) * (q - 1)
    
    # Выбираем e (обычно 65537)
    e = 65537
    if gcd(e, phi) != 1:
        e = 3
    
    # Вычисляем d
    d = mod_inverse(e, phi)
    
    return (e, n), (d, n)  # (открытый ключ, закрытый ключ)

# Пример использования
p, q = 61, 53  # Простые числа
public_key, private_key = generate_rsa_keys(p, q)
print(f"Открытый ключ: {public_key}")
print(f"Закрытый ключ: {private_key}")
```

### 2. Компьютерная графика

#### 3D трансформации
```python
import math

class Vector3D:
    def __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z
    
    def __add__(self, other):
        return Vector3D(self.x + other.x, self.y + other.y, self.z + other.z)
    
    def __sub__(self, other):
        return Vector3D(self.x - other.x, self.y - other.y, self.z - other.z)
    
    def __mul__(self, scalar):
        return Vector3D(self.x * scalar, self.y * scalar, self.z * scalar)
    
    def dot(self, other):
        return self.x * other.x + self.y * other.y + self.z * other.z
    
    def cross(self, other):
        return Vector3D(
            self.y * other.z - self.z * other.y,
            self.z * other.x - self.x * other.z,
            self.x * other.y - self.y * other.x
        )
    
    def length(self):
        return math.sqrt(self.x**2 + self.y**2 + self.z**2)
    
    def normalize(self):
        length = self.length()
        if length == 0:
            return Vector3D(0, 0, 0)
        return Vector3D(self.x / length, self.y / length, self.z / length)
    
    def __repr__(self):
        return f"Vector3D({self.x:.2f}, {self.y:.2f}, {self.z:.2f})"

class Matrix4x4:
    def __init__(self, values):
        self.values = values  # 4x4 матрица
    
    def __mul__(self, other):
        if isinstance(other, Matrix4x4):
            result = [[0 for _ in range(4)] for _ in range(4)]
            for i in range(4):
                for j in range(4):
                    for k in range(4):
                        result[i][j] += self.values[i][k] * other.values[k][j]
            return Matrix4x4(result)
        elif isinstance(other, Vector3D):
            # Умножение на вектор (с добавлением w=1)
            x = self.values[0][0] * other.x + self.values[0][1] * other.y + self.values[0][2] * other.z + self.values[0][3]
            y = self.values[1][0] * other.x + self.values[1][1] * other.y + self.values[1][2] * other.z + self.values[1][3]
            z = self.values[2][0] * other.x + self.values[2][1] * other.y + self.values[2][2] * other.z + self.values[2][3]
            return Vector3D(x, y, z)
    
    @staticmethod
    def translation(x, y, z):
        return Matrix4x4([
            [1, 0, 0, x],
            [0, 1, 0, y],
            [0, 0, 1, z],
            [0, 0, 0, 1]
        ])
    
    @staticmethod
    def rotation_x(angle):
        c = math.cos(angle)
        s = math.sin(angle)
        return Matrix4x4([
            [1, 0, 0, 0],
            [0, c, -s, 0],
            [0, s, c, 0],
            [0, 0, 0, 1]
        ])
    
    @staticmethod
    def rotation_y(angle):
        c = math.cos(angle)
        s = math.sin(angle)
        return Matrix4x4([
            [c, 0, s, 0],
            [0, 1, 0, 0],
            [-s, 0, c, 0],
            [0, 0, 0, 1]
        ])
    
    @staticmethod
    def scale(x, y, z):
        return Matrix4x4([
            [x, 0, 0, 0],
            [0, y, 0, 0],
            [0, 0, z, 0],
            [0, 0, 0, 1]
        ])

# Пример: создание и трансформация 3D объекта
def create_cube():
    """Создание вершин куба"""
    return [
        Vector3D(-1, -1, -1), Vector3D(1, -1, -1),
        Vector3D(1, 1, -1), Vector3D(-1, 1, -1),
        Vector3D(-1, -1, 1), Vector3D(1, -1, 1),
        Vector3D(1, 1, 1), Vector3D(-1, 1, 1)
    ]

def transform_object(vertices, transformations):
    """Применение трансформаций к объекту"""
    combined_transform = Matrix4x4([[1, 0, 0, 0], [0, 1, 0, 0], [0, 0, 1, 0], [0, 0, 0, 1]])
    
    for transform in transformations:
        combined_transform = combined_transform * transform
    
    return [combined_transform * vertex for vertex in vertices]

# Создание куба
cube = create_cube()
print("Исходный куб:")
for i, vertex in enumerate(cube):
    print(f"  Вершина {i}: {vertex}")

# Трансформации: масштабирование, поворот, перемещение
transformations = [
    Matrix4x4.scale(2, 2, 2),
    Matrix4x4.rotation_y(math.pi / 4),
    Matrix4x4.translation(5, 0, 0)
]

transformed_cube = transform_object(cube, transformations)
print("\nТрансформированный куб:")
for i, vertex in enumerate(transformed_cube):
    print(f"  Вершина {i}: {vertex}")
```

## 🎯 Заключение

Математика для программистов - это не просто **теоретические знания**, а **практический инструмент** для решения реальных задач:

### Ключевые принципы
1. **Понимание основ** - логика, множества, системы счисления
2. **Алгоритмическое мышление** - графы, комбинаторика, оптимизация
3. **Геометрическое представление** - векторы, матрицы, трансформации
4. **Численные методы** - приближенные вычисления, интегрирование

### Практическое применение
- **Веб-разработка**: оптимизация алгоритмов, анализ данных
- **Игровая разработка**: физика, графика, ИИ
- **Мобильная разработка**: интерфейсы, производительность
- **Data Science**: статистика, машинное обучение

### Следующие шаги
1. Изучить **теорию вероятностей** для анализа данных
2. Освоить **линейную алгебру** для машинного обучения
3. Применить знания в **реальных проектах**
4. Изучить **специализированные области** (криптография, компьютерная графика)

Математика - это **фундамент** для понимания того, как работают современные технологии и как создавать эффективные решения. 