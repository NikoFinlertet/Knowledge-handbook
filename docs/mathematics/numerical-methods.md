# Численные методы

## 🎯 Цель курса

Изучение численных методов для решения математических задач в программировании и научных вычислениях.

## 📚 Содержание

1. [Решение уравнений](#решение-уравнений)
2. [Интерполяция и аппроксимация](#интерполяция-и-аппроксимация)
3. [Численное интегрирование](#численное-интегрирование)
4. [Решение систем линейных уравнений](#решение-систем-линейных-уравнений)
5. [Обыкновенные дифференциальные уравнения](#обыкновенные-дифференциальные-уравнения)
6. [Оптимизация](#оптимизация)

---

## Решение уравнений

### Метод Ньютона

```python
import numpy as np
import matplotlib.pyplot as plt

def newton_method(f, df, x0, tol=1e-10, max_iter=100):
    """
    Метод Ньютона для решения уравнения f(x) = 0
    
    Args:
        f: функция
        df: производная функции
        x0: начальное приближение
        tol: точность
        max_iter: максимальное число итераций
    """
    x = x0
    history = [x]
    
    for i in range(max_iter):
        fx = f(x)
        dfx = df(x)
        
        if abs(fx) < tol:
            return x, history, i + 1
        
        if abs(dfx) < 1e-15:
            raise ValueError("Производная близка к нулю")
        
        x_new = x - fx / dfx
        history.append(x_new)
        
        if abs(x_new - x) < tol:
            return x_new, history, i + 1
        
        x = x_new
    
    raise ValueError("Не сходится за заданное число итераций")

def secant_method(f, x0, x1, tol=1e-10, max_iter=100):
    """
    Метод секущих (не требует производной)
    """
    history = [x0, x1]
    
    for i in range(max_iter):
        f0, f1 = f(x0), f(x1)
        
        if abs(f1) < tol:
            return x1, history, i + 1
        
        if abs(f1 - f0) < 1e-15:
            raise ValueError("Деление на ноль в методе секущих")
        
        x2 = x1 - f1 * (x1 - x0) / (f1 - f0)
        history.append(x2)
        
        if abs(x2 - x1) < tol:
            return x2, history, i + 1
        
        x0, x1 = x1, x2
    
    raise ValueError("Не сходится за заданное число итераций")

# Пример: решение уравнения x^3 - 2x - 5 = 0
def example_equation():
    def f(x):
        return x**3 - 2*x - 5
    
    def df(x):
        return 3*x**2 - 2
    
    # Метод Ньютона
    try:
        root_newton, history_newton, iterations = newton_method(f, df, 2.0)
        print(f"Метод Ньютона: корень = {root_newton:.8f}, итераций = {iterations}")
    except ValueError as e:
        print(f"Ошибка в методе Ньютона: {e}")
    
    # Метод секущих
    try:
        root_secant, history_secant, iterations = secant_method(f, 2.0, 2.1)
        print(f"Метод секущих: корень = {root_secant:.8f}, итераций = {iterations}")
    except ValueError as e:
        print(f"Ошибка в методе секущих: {e}")

example_equation()
```

### Метод бисекции

```python
def bisection_method(f, a, b, tol=1e-10, max_iter=100):
    """
    Метод бисекции для решения уравнения f(x) = 0
    Требует, чтобы f(a) и f(b) имели разные знаки
    """
    if f(a) * f(b) > 0:
        raise ValueError("f(a) и f(b) должны иметь разные знаки")
    
    history = []
    
    for i in range(max_iter):
        c = (a + b) / 2
        fc = f(c)
        history.append(c)
        
        if abs(fc) < tol or abs(b - a) < tol:
            return c, history, i + 1
        
        if f(a) * fc < 0:
            b = c
        else:
            a = c
    
    raise ValueError("Не сходится за заданное число итераций")

# Пример использования
def f(x):
    return x**3 - 2*x - 5

root, history, iterations = bisection_method(f, 2, 3)
print(f"Метод бисекции: корень = {root:.8f}, итераций = {iterations}")
```

---

## Интерполяция и аппроксимация

### Интерполяция Лагранжа

```python
def lagrange_interpolation(x_points, y_points, x):
    """
    Интерполяция Лагранжа
    """
    n = len(x_points)
    result = 0
    
    for i in range(n):
        # Вычисляем базисный полином Лагранжа
        basis = 1
        for j in range(n):
            if i != j:
                basis *= (x - x_points[j]) / (x_points[i] - x_points[j])
        
        result += y_points[i] * basis
    
    return result

def lagrange_polynomial(x_points, y_points):
    """
    Возвращает функцию интерполяционного полинома
    """
    def polynomial(x):
        if isinstance(x, (list, np.ndarray)):
            return [lagrange_interpolation(x_points, y_points, xi) for xi in x]
        else:
            return lagrange_interpolation(x_points, y_points, x)
    
    return polynomial

# Пример: интерполяция функции
x_data = [0, 1, 2, 3, 4]
y_data = [1, 2, 5, 10, 17]  # Примерно x^2 + 1

# Создаем интерполяционный полином
poly = lagrange_polynomial(x_data, y_data)

# Тестируем
x_test = np.linspace(0, 4, 100)
y_interp = poly(x_test)

print(f"Интерполяция в точке x=2.5: {poly(2.5)}")
```

### Кубические сплайны

```python
import numpy as np
from scipy.linalg import solve

def cubic_spline(x_points, y_points, x):
    """
    Кубический сплайн интерполяции
    """
    n = len(x_points) - 1
    
    # Создаем систему уравнений для вторых производных
    A = np.zeros((n + 1, n + 1))
    B = np.zeros(n + 1)
    
    # Граничные условия (естественный сплайн)
    A[0, 0] = 1
    A[n, n] = 1
    
    # Внутренние условия
    for i in range(1, n):
        h_i = x_points[i] - x_points[i - 1]
        h_i1 = x_points[i + 1] - x_points[i]
        
        A[i, i - 1] = h_i
        A[i, i] = 2 * (h_i + h_i1)
        A[i, i + 1] = h_i1
        
        B[i] = 6 * ((y_points[i + 1] - y_points[i]) / h_i1 - 
                    (y_points[i] - y_points[i - 1]) / h_i)
    
    # Решаем систему
    second_derivatives = solve(A, B)
    
    # Находим интервал для x
    for i in range(n):
        if x_points[i] <= x <= x_points[i + 1]:
            h = x_points[i + 1] - x_points[i]
            
            # Вычисляем значение сплайна
            a = (x_points[i + 1] - x) / h
            b = (x - x_points[i]) / h
            
            result = (a * y_points[i] + b * y_points[i + 1] + 
                     ((a**3 - a) * second_derivatives[i] + 
                      (b**3 - b) * second_derivatives[i + 1]) * h**2 / 6)
            
            return result
    
    raise ValueError("x вне диапазона интерполяции")

# Пример использования
x_data = np.array([0, 1, 2, 3, 4, 5])
y_data = np.array([0, 1, 4, 9, 16, 25])  # y = x^2

x_test = 2.5
y_spline = cubic_spline(x_data, y_data, x_test)
print(f"Кубический сплайн в точке x={x_test}: {y_spline}")
```

---

## Численное интегрирование

### Квадратурные формулы

```python
def trapezoidal_rule(f, a, b, n):
    """Правило трапеций"""
    h = (b - a) / n
    x = np.linspace(a, b, n + 1)
    y = f(x)
    
    return h * (y[0]/2 + np.sum(y[1:-1]) + y[-1]/2)

def simpson_rule(f, a, b, n):
    """Правило Симпсона (n должно быть четным)"""
    if n % 2 != 0:
        n += 1
    
    h = (b - a) / n
    x = np.linspace(a, b, n + 1)
    y = f(x)
    
    return h/3 * (y[0] + 4*np.sum(y[1:-1:2]) + 2*np.sum(y[2:-1:2]) + y[-1])

def adaptive_simpson(f, a, b, tol=1e-10, max_depth=10):
    """Адаптивное правило Симпсона"""
    
    def simpson_basic(f, a, b):
        c = (a + b) / 2
        return (b - a) / 6 * (f(a) + 4*f(c) + f(b))
    
    def adaptive_helper(f, a, b, tol, whole, depth):
        if depth > max_depth:
            return whole
        
        c = (a + b) / 2
        left = simpson_basic(f, a, c)
        right = simpson_basic(f, c, b)
        
        if abs(left + right - whole) <= 15 * tol:
            return left + right + (left + right - whole) / 15
        
        return (adaptive_helper(f, a, c, tol/2, left, depth+1) + 
                adaptive_helper(f, c, b, tol/2, right, depth+1))
    
    whole = simpson_basic(f, a, b)
    return adaptive_helper(f, a, b, tol, whole, 0)

# Пример: интегрирование различных функций
def integration_examples():
    def f1(x):
        return np.exp(-x**2)  # Гауссова функция
    
    def f2(x):
        return np.sin(x) / x if x != 0 else 1  # sinc функция
    
    a, b = 0, 2
    n = 100
    
    print("Интегрирование exp(-x²) от 0 до 2:")
    print(f"Трапеции: {trapezoidal_rule(f1, a, b, n):.8f}")
    print(f"Симпсон: {simpson_rule(f1, a, b, n):.8f}")
    print(f"Адаптивный Симпсон: {adaptive_simpson(f1, a, b):.8f}")
    
    print("\nИнтегрирование sin(x)/x от 0 до 2:")
    print(f"Трапеции: {trapezoidal_rule(f2, a, b, n):.8f}")
    print(f"Симпсон: {simpson_rule(f2, a, b, n):.8f}")
    print(f"Адаптивный Симпсон: {adaptive_simpson(f2, a, b):.8f}")

integration_examples()
```

### Гауссовы квадратуры

```python
def gauss_legendre_quadrature(f, a, b, n):
    """
    Квадратура Гаусса-Лежандра
    """
    # Узлы и веса для интервала [-1, 1]
    nodes_weights = {
        2: ([-0.5773502692, 0.5773502692], [1.0, 1.0]),
        3: ([-0.7745966692, 0.0, 0.7745966692], [0.5555555556, 0.8888888889, 0.5555555556]),
        4: ([-0.8611363116, -0.3399810436, 0.3399810436, 0.8611363116], 
            [0.3478548451, 0.6521451549, 0.6521451549, 0.3478548451]),
        5: ([-0.9061798459, -0.5384693101, 0.0, 0.5384693101, 0.9061798459],
            [0.2369268851, 0.4786286705, 0.5688888889, 0.4786286705, 0.2369268851])
    }
    
    if n not in nodes_weights:
        raise ValueError(f"Узлы для n={n} не определены")
    
    nodes, weights = nodes_weights[n]
    
    # Преобразование к интервалу [a, b]
    def transform_x(t):
        return (b - a) * t / 2 + (b + a) / 2
    
    jacobian = (b - a) / 2
    
    result = 0
    for i in range(n):
        x_i = transform_x(nodes[i])
        result += weights[i] * f(x_i)
    
    return jacobian * result

# Пример использования
def f(x):
    return np.exp(-x**2)

a, b = 0, 1
for n in [2, 3, 4, 5]:
    result = gauss_legendre_quadrature(f, a, b, n)
    print(f"Гаусс-Лежандр (n={n}): {result:.8f}")
```

---

## Решение систем линейных уравнений

### Метод Гаусса

```python
import numpy as np

def gaussian_elimination(A, b):
    """
    Метод Гаусса для решения системы Ax = b
    """
    n = len(A)
    
    # Создаем расширенную матрицу
    augmented = np.column_stack([A.astype(float), b.astype(float)])
    
    # Прямой ход
    for i in range(n):
        # Поиск главного элемента
        max_row = i
        for k in range(i + 1, n):
            if abs(augmented[k, i]) > abs(augmented[max_row, i]):
                max_row = k
        
        # Перестановка строк
        if max_row != i:
            augmented[[i, max_row]] = augmented[[max_row, i]]
        
        # Проверка на вырожденность
        if abs(augmented[i, i]) < 1e-10:
            raise ValueError("Матрица вырожденная")
        
        # Обнуление элементов под главным
        for k in range(i + 1, n):
            factor = augmented[k, i] / augmented[i, i]
            for j in range(i, n + 1):
                augmented[k, j] -= factor * augmented[i, j]
    
    # Обратный ход
    x = np.zeros(n)
    for i in range(n - 1, -1, -1):
        x[i] = augmented[i, n]
        for j in range(i + 1, n):
            x[i] -= augmented[i, j] * x[j]
        x[i] /= augmented[i, i]
    
    return x

# Пример использования
A = np.array([[2, 1, -1],
              [-3, -1, 2],
              [-2, 1, 2]], dtype=float)
b = np.array([8, -11, -3], dtype=float)

x = gaussian_elimination(A, b)
print(f"Решение системы: {x}")
print(f"Проверка Ax = {np.dot(A, x)}")
```

### Итерационные методы

```python
def jacobi_method(A, b, x0=None, tol=1e-10, max_iter=100):
    """
    Метод Якоби для решения системы Ax = b
    """
    n = len(A)
    if x0 is None:
        x0 = np.zeros(n)
    
    x = x0.copy()
    x_new = np.zeros(n)
    
    for iteration in range(max_iter):
        for i in range(n):
            sum_ax = sum(A[i, j] * x[j] for j in range(n) if j != i)
            x_new[i] = (b[i] - sum_ax) / A[i, i]
        
        # Проверка сходимости
        if np.linalg.norm(x_new - x) < tol:
            return x_new, iteration + 1
        
        x = x_new.copy()
    
    raise ValueError("Не сходится за заданное число итераций")

def gauss_seidel_method(A, b, x0=None, tol=1e-10, max_iter=100):
    """
    Метод Гаусса-Зейделя для решения системы Ax = b
    """
    n = len(A)
    if x0 is None:
        x0 = np.zeros(n)
    
    x = x0.copy()
    
    for iteration in range(max_iter):
        x_old = x.copy()
        
        for i in range(n):
            sum_ax = sum(A[i, j] * x[j] for j in range(n) if j != i)
            x[i] = (b[i] - sum_ax) / A[i, i]
        
        # Проверка сходимости
        if np.linalg.norm(x - x_old) < tol:
            return x, iteration + 1
    
    raise ValueError("Не сходится за заданное число итераций")

# Пример: сравнение методов
A = np.array([[4, -1, 0],
              [-1, 4, -1],
              [0, -1, 4]], dtype=float)
b = np.array([1, 2, 3], dtype=float)

print("Сравнение итерационных методов:")

try:
    x_jacobi, iter_jacobi = jacobi_method(A, b)
    print(f"Якоби: {x_jacobi}, итераций: {iter_jacobi}")
except ValueError as e:
    print(f"Якоби: {e}")

try:
    x_gauss_seidel, iter_gs = gauss_seidel_method(A, b)
    print(f"Гаусс-Зейдель: {x_gauss_seidel}, итераций: {iter_gs}")
except ValueError as e:
    print(f"Гаусс-Зейдель: {e}")
```

---

## Оптимизация

### Градиентный спуск

```python
def gradient_descent(f, grad_f, x0, lr=0.01, tol=1e-8, max_iter=1000):
    """
    Градиентный спуск для минимизации функции
    """
    x = np.array(x0)
    history = [x.copy()]
    
    for i in range(max_iter):
        grad = grad_f(x)
        
        if np.linalg.norm(grad) < tol:
            return x, history, i + 1
        
        x = x - lr * grad
        history.append(x.copy())
    
    return x, history, max_iter

def armijo_line_search(f, grad_f, x, direction, alpha=1.0, c1=1e-4, rho=0.5):
    """
    Поиск шага по правилу Армихо
    """
    f_x = f(x)
    grad_x = grad_f(x)
    
    while f(x + alpha * direction) > f_x + c1 * alpha * np.dot(grad_x, direction):
        alpha *= rho
        if alpha < 1e-10:
            break
    
    return alpha

def bfgs_method(f, grad_f, x0, tol=1e-8, max_iter=1000):
    """
    Метод BFGS (квази-Ньютоновский)
    """
    n = len(x0)
    x = np.array(x0)
    H = np.eye(n)  # Начальная аппроксимация обратной матрицы Гессе
    
    history = [x.copy()]
    
    for i in range(max_iter):
        grad = grad_f(x)
        
        if np.linalg.norm(grad) < tol:
            return x, history, i + 1
        
        # Направление поиска
        direction = -H @ grad
        
        # Поиск шага
        alpha = armijo_line_search(f, grad_f, x, direction)
        
        # Обновление x
        x_new = x + alpha * direction
        
        # Обновление аппроксимации Гессе
        s = x_new - x
        y = grad_f(x_new) - grad
        
        if np.dot(s, y) > 1e-10:
            rho = 1 / np.dot(s, y)
            I = np.eye(n)
            H = (I - rho * np.outer(s, y)) @ H @ (I - rho * np.outer(y, s)) + rho * np.outer(s, s)
        
        x = x_new
        history.append(x.copy())
    
    return x, history, max_iter

# Пример: минимизация функции Розенброка
def rosenbrock_example():
    def f(x):
        return 100 * (x[1] - x[0]**2)**2 + (1 - x[0])**2
    
    def grad_f(x):
        return np.array([
            -400 * x[0] * (x[1] - x[0]**2) - 2 * (1 - x[0]),
            200 * (x[1] - x[0]**2)
        ])
    
    x0 = np.array([-1.2, 1.0])
    
    # Градиентный спуск
    x_gd, history_gd, iter_gd = gradient_descent(f, grad_f, x0, lr=0.001)
    print(f"Градиентный спуск: x = {x_gd}, f(x) = {f(x_gd):.8f}, итераций = {iter_gd}")
    
    # BFGS
    x_bfgs, history_bfgs, iter_bfgs = bfgs_method(f, grad_f, x0)
    print(f"BFGS: x = {x_bfgs}, f(x) = {f(x_bfgs):.8f}, итераций = {iter_bfgs}")

rosenbrock_example()
```

---

## 🎯 Практические задания

### Задание 1: Финансовое моделирование
Реализуйте численное решение уравнения Блэка-Шоулза для оценки опционов.

### Задание 2: Обработка сигналов
Используйте сплайн-интерполяцию для восстановления сигнала.

### Задание 3: Машинное обучение
Реализуйте оптимизацию параметров нейронной сети.

### Задание 4: Научные вычисления
Решите систему дифференциальных уравнений методом Рунге-Кутты.

---

## 🔗 Связанные материалы

- [Математический анализ](./mathematical-analysis.md)
- [Линейная алгебра](./linear-algebra.md)
- [Машинное обучение](../algorithms-data-structures/machine-learning-fundamentals.md)
- [Оптимизация](./optimization.md) 