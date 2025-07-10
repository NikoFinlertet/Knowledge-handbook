# Вычислительная геометрия

## 🎯 Цель курса

Изучение геометрических алгоритмов и математических основ для разработки графических приложений, игр и компьютерного зрения.

## 📚 Содержание

1. [Основы геометрии](#основы-геометрии)
2. [Алгоритмы на точках](#алгоритмы-на-точках)
3. [Алгоритмы на отрезках](#алгоритмы-на-отрезках)
4. [Полигоны и многоугольники](#полигоны-и-многоугольники)
5. [3D геометрия](#3d-геометрия)
6. [Компьютерная графика](#компьютерная-графика)

---

## Основы геометрии

### Системы координат

**Декартова система:**
$$P = (x, y)$$

**Полярная система:**
$$P = (r, \theta)$$
$$x = r \cos \theta$$
$$y = r \sin \theta$$

**Переход между системами:**
$$r = \sqrt{x^2 + y^2}$$
$$\theta = \arctan\left(\frac{y}{x}\right)$$

### Векторы

**Определение вектора:**
$$\vec{v} = \langle x, y \rangle$$

**Длина вектора:**
$$|\vec{v}| = \sqrt{x^2 + y^2}$$

**Нормализация:**
$$\hat{v} = \frac{\vec{v}}{|\vec{v}|} = \left\langle \frac{x}{|\vec{v}|}, \frac{y}{|\vec{v}|} \right\rangle$$

**Скалярное произведение:**
$$\vec{a} \cdot \vec{b} = a_x b_x + a_y b_y = |\vec{a}| |\vec{b}| \cos \theta$$

**Векторное произведение (2D):**
$$\vec{a} \times \vec{b} = a_x b_y - a_y b_x$$

### Практическая реализация

```python
import math
import numpy as np

class Vector2D:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def length(self):
        return math.sqrt(self.x**2 + self.y**2)
    
    def normalize(self):
        length = self.length()
        if length > 0:
            return Vector2D(self.x / length, self.y / length)
        return Vector2D(0, 0)
    
    def dot(self, other):
        return self.x * other.x + self.y * other.y
    
    def cross(self, other):
        return self.x * other.y - self.y * other.x
    
    def angle_between(self, other):
        dot_product = self.dot(other)
        lengths = self.length() * other.length()
        return math.acos(dot_product / lengths)
```

---

## Алгоритмы на точках

### Расстояние между точками

**Евклидово расстояние:**
$$d = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$$

**Манхэттенское расстояние:**
$$d = |x_2 - x_1| + |y_2 - y_1|$$

### Выпуклая оболочка

**Алгоритм Грэхема:**
1. Найти точку с минимальной y-координатой
2. Отсортировать точки по полярному углу
3. Построить выпуклую оболочку

**Проверка поворота:**
$$\text{cross}(p_1, p_2, p_3) = (p_2.x - p_1.x)(p_3.y - p_1.y) - (p_2.y - p_1.y)(p_3.x - p_1.x)$$

```python
def convex_hull_graham(points):
    def cross_product(o, a, b):
        return (a[0] - o[0]) * (b[1] - o[1]) - (a[1] - o[1]) * (b[0] - o[0])
    
    # Найти начальную точку
    start = min(points, key=lambda p: (p[1], p[0]))
    
    # Сортировка по полярному углу
    def polar_angle(p):
        return math.atan2(p[1] - start[1], p[0] - start[0])
    
    sorted_points = sorted(points, key=polar_angle)
    
    # Построение выпуклой оболочки
    hull = []
    for p in sorted_points:
        while len(hull) > 1 and cross_product(hull[-2], hull[-1], p) <= 0:
            hull.pop()
        hull.append(p)
    
    return hull
```

### Триангуляция Делоне

**Критерий Делоне:**
Треугольник принадлежит триангуляции Делоне, если его описанная окружность не содержит других точек.

**Тест в окружности:**
$$\begin{vmatrix}
x_1 & y_1 & x_1^2 + y_1^2 & 1 \\
x_2 & y_2 & x_2^2 + y_2^2 & 1 \\
x_3 & y_3 & x_3^2 + y_3^2 & 1 \\
x_4 & y_4 & x_4^2 + y_4^2 & 1
\end{vmatrix} > 0$$

---

## Алгоритмы на отрезках

### Пересечение отрезков

**Параметрическое представление:**
$$P(t) = P_1 + t(P_2 - P_1), \quad t \in [0, 1]$$

**Пересечение двух отрезков:**
Отрезки $AB$ и $CD$ пересекаются, если:
$$\text{orientation}(A, B, C) \neq \text{orientation}(A, B, D)$$
$$\text{orientation}(C, D, A) \neq \text{orientation}(C, D, B)$$

```python
def segments_intersect(p1, q1, p2, q2):
    def orientation(p, q, r):
        val = (q[1] - p[1]) * (r[0] - q[0]) - (q[0] - p[0]) * (r[1] - q[1])
        if val == 0:
            return 0  # коллинеарные
        return 1 if val > 0 else 2  # по часовой или против часовой
    
    def on_segment(p, q, r):
        return (q[0] <= max(p[0], r[0]) and q[0] >= min(p[0], r[0]) and
                q[1] <= max(p[1], r[1]) and q[1] >= min(p[1], r[1]))
    
    o1 = orientation(p1, q1, p2)
    o2 = orientation(p1, q1, q2)
    o3 = orientation(p2, q2, p1)
    o4 = orientation(p2, q2, q1)
    
    # Общий случай
    if o1 != o2 and o3 != o4:
        return True
    
    # Специальные случаи
    if (o1 == 0 and on_segment(p1, p2, q1)) or \
       (o2 == 0 and on_segment(p1, q2, q1)) or \
       (o3 == 0 and on_segment(p2, p1, q2)) or \
       (o4 == 0 and on_segment(p2, q1, q2)):
        return True
    
    return False
```

### Расстояние от точки до отрезка

**Формула:**
$$d = \frac{|(x_2 - x_1)(y_1 - y_0) - (x_1 - x_0)(y_2 - y_1)|}{\sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}}$$

---

## Полигоны и многоугольники

### Площадь многоугольника

**Формула Гаусса:**
$$S = \frac{1}{2} \left| \sum_{i=0}^{n-1} (x_i y_{i+1} - x_{i+1} y_i) \right|$$

### Проверка принадлежности точки полигону

**Алгоритм трассировки лучей:**
```python
def point_in_polygon(point, polygon):
    x, y = point
    n = len(polygon)
    inside = False
    
    p1x, p1y = polygon[0]
    for i in range(1, n + 1):
        p2x, p2y = polygon[i % n]
        if y > min(p1y, p2y):
            if y <= max(p1y, p2y):
                if x <= max(p1x, p2x):
                    if p1y != p2y:
                        xinters = (y - p1y) * (p2x - p1x) / (p2y - p1y) + p1x
                    if p1x == p2x or x <= xinters:
                        inside = not inside
        p1x, p1y = p2x, p2y
    
    return inside
```

### Выпуклость полигона

**Проверка выпуклости:**
Полигон выпукл, если все повороты имеют одинаковый знак.

---

## 3D геометрия

### Трехмерные векторы

**Векторное произведение:**
$$\vec{a} \times \vec{b} = \begin{vmatrix}
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
a_x & a_y & a_z \\
b_x & b_y & b_z
\end{vmatrix}$$

$$= (a_y b_z - a_z b_y)\mathbf{i} - (a_x b_z - a_z b_x)\mathbf{j} + (a_x b_y - a_y b_x)\mathbf{k}$$

### Плоскости

**Уравнение плоскости:**
$$ax + by + cz + d = 0$$

**Расстояние от точки до плоскости:**
$$d = \frac{|ax_0 + by_0 + cz_0 + d|}{\sqrt{a^2 + b^2 + c^2}}$$

### Пересечение луча с плоскостью

**Параметрическое уравнение луча:**
$$\mathbf{r}(t) = \mathbf{o} + t\mathbf{d}$$

**Точка пересечения:**
$$t = -\frac{\mathbf{n} \cdot \mathbf{o} + d}{\mathbf{n} \cdot \mathbf{d}}$$

---

## Компьютерная графика

### Трансформации

**Матрица поворота (2D):**
$$R(\theta) = \begin{pmatrix}
\cos\theta & -\sin\theta \\
\sin\theta & \cos\theta
\end{pmatrix}$$

**Матрица масштабирования:**
$$S(s_x, s_y) = \begin{pmatrix}
s_x & 0 \\
0 & s_y
\end{pmatrix}$$

**Матрица переноса (гомогенные координаты):**
$$T(t_x, t_y) = \begin{pmatrix}
1 & 0 & t_x \\
0 & 1 & t_y \\
0 & 0 & 1
\end{pmatrix}$$

### Проекции

**Ортогональная проекция:**
$$P_{ortho} = \begin{pmatrix}
\frac{2}{r-l} & 0 & 0 & -\frac{r+l}{r-l} \\
0 & \frac{2}{t-b} & 0 & -\frac{t+b}{t-b} \\
0 & 0 & \frac{-2}{f-n} & -\frac{f+n}{f-n} \\
0 & 0 & 0 & 1
\end{pmatrix}$$

**Перспективная проекция:**
$$P_{persp} = \begin{pmatrix}
\frac{2n}{r-l} & 0 & \frac{r+l}{r-l} & 0 \\
0 & \frac{2n}{t-b} & \frac{t+b}{t-b} & 0 \\
0 & 0 & -\frac{f+n}{f-n} & -\frac{2fn}{f-n} \\
0 & 0 & -1 & 0
\end{pmatrix}$$

### Кривые Безье

**Кубическая кривая Безье:**
$$B(t) = (1-t)^3 P_0 + 3(1-t)^2 t P_1 + 3(1-t) t^2 P_2 + t^3 P_3$$

**Матричная форма:**
$$B(t) = \begin{pmatrix} 1 & t & t^2 & t^3 \end{pmatrix} \begin{pmatrix}
1 & 0 & 0 & 0 \\
-3 & 3 & 0 & 0 \\
3 & -6 & 3 & 0 \\
-1 & 3 & -3 & 1
\end{pmatrix} \begin{pmatrix}
P_0 \\ P_1 \\ P_2 \\ P_3
\end{pmatrix}$$

### Алгоритмы растеризации

**Алгоритм Брезенхема для отрезка:**
```python
def bresenham_line(x0, y0, x1, y1):
    points = []
    dx = abs(x1 - x0)
    dy = abs(y1 - y0)
    
    sx = 1 if x0 < x1 else -1
    sy = 1 if y0 < y1 else -1
    
    err = dx - dy
    
    while True:
        points.append((x0, y0))
        
        if x0 == x1 and y0 == y1:
            break
            
        e2 = 2 * err
        
        if e2 > -dy:
            err -= dy
            x0 += sx
            
        if e2 < dx:
            err += dx
            y0 += sy
    
    return points
```

**Алгоритм заливки:**
```python
def flood_fill(image, x, y, new_color, old_color):
    if x < 0 or x >= len(image) or y < 0 or y >= len(image[0]):
        return
    if image[x][y] != old_color:
        return
    
    image[x][y] = new_color
    
    # Рекурсивно заливаем соседние пиксели
    flood_fill(image, x + 1, y, new_color, old_color)
    flood_fill(image, x - 1, y, new_color, old_color)
    flood_fill(image, x, y + 1, new_color, old_color)
    flood_fill(image, x, y - 1, new_color, old_color)
```

---

## Применение в играх

### Детекция коллизий

**AABB (Axis-Aligned Bounding Box):**
```python
def aabb_collision(box1, box2):
    return (box1.min_x <= box2.max_x and box1.max_x >= box2.min_x and
            box1.min_y <= box2.max_y and box1.max_y >= box2.min_y)
```

**Окружности:**
```python
def circle_collision(circle1, circle2):
    distance = math.sqrt((circle1.x - circle2.x)**2 + (circle1.y - circle2.y)**2)
    return distance <= (circle1.radius + circle2.radius)
```

### Пространственные структуры данных

**Quadtree:**
```python
class QuadTree:
    def __init__(self, boundary, capacity):
        self.boundary = boundary
        self.capacity = capacity
        self.points = []
        self.divided = False
        
    def subdivide(self):
        x, y, w, h = self.boundary
        nw = QuadTree((x, y, w/2, h/2), self.capacity)
        ne = QuadTree((x + w/2, y, w/2, h/2), self.capacity)
        sw = QuadTree((x, y + h/2, w/2, h/2), self.capacity)
        se = QuadTree((x + w/2, y + h/2, w/2, h/2), self.capacity)
        
        self.northwest = nw
        self.northeast = ne
        self.southwest = sw
        self.southeast = se
        
        self.divided = True
```

---

## Связанные темы

- [[linear-algebra-computation]] - Матрицы и векторы
- [[physics-fundamentals]] - Физические симуляции
- [[optimization]] - Оптимизация геометрических алгоритмов
- [[numerical-methods]] - Численные методы в геометрии

---

*Геометрия - это основа для понимания пространственных отношений и создания визуальных приложений. Знание геометрических алгоритмов критически важно для разработки игр, графических редакторов и систем компьютерного зрения.* 