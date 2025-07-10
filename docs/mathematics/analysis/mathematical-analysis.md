	# Математический анализ для программистов

## 🎯 Цель курса

Глубокое понимание математического анализа для решения задач программирования, алгоритмов и машинного обучения.

## 📚 Содержание

1. [Пределы и непрерывность](#пределы-и-непрерывность)
2. [Дифференциальное исчисление](#дифференциальное-исчисление)
3. [Интегральное исчисление](#интегральное-исчисление)
4. [Ряды и последовательности](#ряды-и-последовательности)
5. [Многомерный анализ](#многомерный-анализ)
6. [Применения в программировании](#применения-в-программировании)

---

## Пределы и непрерывность

### Определения и основные понятия

**Предел функции**: Число $L$ называется пределом функции $f(x)$ при $x \to a$, если для любого $\varepsilon > 0$ существует $\delta > 0$ такое, что:

$$\forall x: 0 < |x - a| < \delta \Rightarrow |f(x) - L| < \varepsilon$$

Обозначение: $\lim_{x \to a} f(x) = L$

**Важные пределы**:
- $\lim_{x \to 0} \frac{\sin x}{x} = 1$
- $\lim_{x \to \infty} \left(1 + \frac{1}{x}\right)^x = e$
- $\lim_{x \to 0} \frac{e^x - 1}{x} = 1$
- $\lim_{x \to 0} \frac{\ln(1 + x)}{x} = 1$

**Теоремы о пределах**:
$$\lim_{x \to a} [f(x) \pm g(x)] = \lim_{x \to a} f(x) \pm \lim_{x \to a} g(x)$$
$$\lim_{x \to a} [f(x) \cdot g(x)] = \lim_{x \to a} f(x) \cdot \lim_{x \to a} g(x)$$
$$\lim_{x \to a} \frac{f(x)}{g(x)} = \frac{\lim_{x \to a} f(x)}{\lim_{x \to a} g(x)}, \quad \text{если } \lim_{x \to a} g(x) \neq 0$$

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import minimize_scalar

def visualize_limit(f, x_approach, epsilon=0.01):
    """Визуализация предела функции"""
    x_values = np.linspace(x_approach - 1, x_approach + 1, 1000)
    y_values = [f(x) for x in x_values if abs(x - x_approach) > epsilon]
    
    plt.figure(figsize=(10, 6))
    plt.plot(x_values[:-1], y_values, 'b-', linewidth=2)
    plt.axvline(x=x_approach, color='r', linestyle='--', alpha=0.7)
    plt.title(f'Предел функции при x → {x_approach}')
    plt.grid(True, alpha=0.3)
    plt.show()

# Пример: предел sin(x)/x при x → 0
# Математически: lim(x→0) sin(x)/x = 1
def sinc_limit(x):
    if abs(x) < 1e-10:
        return 1  # Предел равен 1
    return np.sin(x) / x

# Численная оценка предела
def numerical_limit(f, x_approach, h=1e-6):
    """Численная оценка предела"""
    left_limit = f(x_approach - h)
    right_limit = f(x_approach + h)
    return (left_limit + right_limit) / 2

# Пример использования
print(f"Предел sin(x)/x при x→0: {numerical_limit(sinc_limit, 0)}")
```

### Непрерывность в алгоритмах

**Определение непрерывности**: Функция $f(x)$ непрерывна в точке $a$, если:
$$\lim_{x \to a} f(x) = f(a)$$

**Виды разрывов**:
1. **Устранимый разрыв**: $\lim_{x \to a} f(x)$ существует, но $\lim_{x \to a} f(x) \neq f(a)$
2. **Разрыв первого рода**: $\lim_{x \to a^-} f(x) \neq \lim_{x \to a^+} f(x)$ (скачок)
3. **Разрыв второго рода**: один из односторонних пределов не существует

```python
class ContinuousFunction:
    """Класс для работы с непрерывными функциями"""
    
    def __init__(self, func, domain):
        self.func = func
        self.domain = domain
    
    def is_continuous_at(self, x, tolerance=1e-6):
        """Проверка непрерывности в точке"""
        if x not in self.domain:
            return False
        
        # Проверяем пределы слева и справа
        h = tolerance / 10
        left_limit = self.func(x - h)
        right_limit = self.func(x + h)
        function_value = self.func(x)
        
        return (abs(left_limit - function_value) < tolerance and 
                abs(right_limit - function_value) < tolerance)
    
    def find_discontinuities(self, num_points=1000):
        """Поиск разрывов функции"""
        discontinuities = []
        
        for x in np.linspace(self.domain[0], self.domain[1], num_points):
            if not self.is_continuous_at(x):
                discontinuities.append(x)
        
        return discontinuities

# Пример: функция с разрывом
def step_function(x):
    return 1 if x >= 0 else -1

step_func = ContinuousFunction(step_function, [-2, 2])
print(f"Разрывы: {step_func.find_discontinuities()}")
```

---

## Дифференциальное исчисление

### Производные и их применение

**Определение производной**: Производной функции $f(x)$ в точке $x$ называется предел:
$$f'(x) = \lim_{h \to 0} \frac{f(x + h) - f(x)}{h}$$

**Основные правила дифференцирования**:
- $(c)' = 0$ (константа)
- $(x^n)' = nx^{n-1}$ (степенная функция)
- $(e^x)' = e^x$ (экспонента)
- $(\ln x)' = \frac{1}{x}$ (натуральный логарифм)
- $(\sin x)' = \cos x$ (синус)
- $(\cos x)' = -\sin x$ (косинус)

**Правила дифференцирования**:
- $(f \pm g)' = f' \pm g'$ (сумма/разность)
- $(fg)' = f'g + fg'$ (произведение)
- $\left(\frac{f}{g}\right)' = \frac{f'g - fg'}{g^2}$ (частное)
- $(f(g(x)))' = f'(g(x)) \cdot g'(x)$ (цепное правило)

```python
import sympy as sp
from scipy.misc import derivative

def automatic_differentiation(f, x, h=1e-8):
    """Автоматическое дифференцирование"""
    return (f(x + h) - f(x - h)) / (2 * h)

def gradient_descent(f, df, x_start, learning_rate=0.01, max_iter=1000):
    """Градиентный спуск для оптимизации"""
    x = x_start
    history = [x]
    
    for i in range(max_iter):
        grad = df(x)
        x_new = x - learning_rate * grad
        
        if abs(x_new - x) < 1e-6:
            break
        
        x = x_new
        history.append(x)
    
    return x, history

# Пример: минимизация функции f(x) = x² + 2x + 1
# f'(x) = 2x + 2
def f(x):
    return x**2 + 2*x + 1

def df(x):
    return 2*x + 2

minimum, path = gradient_descent(f, df, x_start=5)
print(f"Минимум функции: x = {minimum}, f(x) = {f(minimum)}")
```

### Многомерные производные

**Частная производная**: Для функции $f(x, y)$ частная производная по $x$:
$$\frac{\partial f}{\partial x} = \lim_{h \to 0} \frac{f(x + h, y) - f(x, y)}{h}$$

**Градиент**: Вектор частных производных функции $f(x_1, x_2, \ldots, x_n)$:
$$\nabla f = \left(\frac{\partial f}{\partial x_1}, \frac{\partial f}{\partial x_2}, \ldots, \frac{\partial f}{\partial x_n}\right)$$

**Матрица Гессе**: Матрица вторых частных производных:
$$H_f = \begin{pmatrix}
\frac{\partial^2 f}{\partial x_1^2} & \frac{\partial^2 f}{\partial x_1 \partial x_2} & \cdots \\
\frac{\partial^2 f}{\partial x_2 \partial x_1} & \frac{\partial^2 f}{\partial x_2^2} & \cdots \\
\vdots & \vdots & \ddots
\end{pmatrix}$$

**Условия экстремума**:
- **Необходимое условие**: $\nabla f = 0$
- **Достаточное условие**: $H_f$ положительно определена (минимум) или отрицательно определена (максимум)

```python
import numpy as np
from scipy.optimize import minimize

def partial_derivative(f, x, var_index, h=1e-8):
    """Вычисление частной производной"""
    x_plus = x.copy()
    x_minus = x.copy()
    
    x_plus[var_index] += h
    x_minus[var_index] -= h
    
    return (f(x_plus) - f(x_minus)) / (2 * h)

def gradient(f, x, h=1e-8):
    """Вычисление градиента"""
    return np.array([partial_derivative(f, x, i, h) for i in range(len(x))])

def hessian(f, x, h=1e-6):
    """Вычисление матрицы Гессе"""
    n = len(x)
    H = np.zeros((n, n))
    
    for i in range(n):
        for j in range(n):
            if i == j:
                # Вторая производная по одной переменной
                H[i, j] = (f(x + h*np.eye(n)[i]) - 2*f(x) + f(x - h*np.eye(n)[i])) / h**2
            else:
                # Смешанная производная
                x_pp = x + h*np.eye(n)[i] + h*np.eye(n)[j]
                x_pm = x + h*np.eye(n)[i] - h*np.eye(n)[j]
                x_mp = x - h*np.eye(n)[i] + h*np.eye(n)[j]
                x_mm = x - h*np.eye(n)[i] - h*np.eye(n)[j]
                
                H[i, j] = (f(x_pp) - f(x_pm) - f(x_mp) + f(x_mm)) / (4 * h**2)
    
    return H

# Пример: функция двух переменных
def f_2d(x):
    return x[0]**2 + x[1]**2 + 2*x[0]*x[1] + 3*x[0] + 4*x[1]

x_point = np.array([1.0, 2.0])
grad = gradient(f_2d, x_point)
H = hessian(f_2d, x_point)

print(f"Градиент в точке {x_point}: {grad}")
print(f"Матрица Гессе:\n{H}")
```

---

## Интегральное исчисление

### Основные понятия

**Неопределенный интеграл**: Множество всех первообразных функции $f(x)$:
$$\int f(x) dx = F(x) + C, \quad \text{где } F'(x) = f(x)$$

**Определенный интеграл**: Предел интегральных сумм Римана:
$$\int_a^b f(x) dx = \lim_{n \to \infty} \sum_{i=1}^n f(x_i) \Delta x$$

**Основная теорема исчисления**:
$$\int_a^b f(x) dx = F(b) - F(a), \quad \text{где } F'(x) = f(x)$$

**Основные интегралы**:
- $\int x^n dx = \frac{x^{n+1}}{n+1} + C$ (для $n \neq -1$)
- $\int \frac{1}{x} dx = \ln|x| + C$
- $\int e^x dx = e^x + C$
- $\int \sin x dx = -\cos x + C$
- $\int \cos x dx = \sin x + C$

**Методы интегрирования**:
- **По частям**: $\int u dv = uv - \int v du$
- **Замена переменной**: $\int f(g(x))g'(x) dx = \int f(u) du$, где $u = g(x)$

### Численное интегрирование

```python
import numpy as np
from scipy.integrate import quad, dblquad

def trapezoidal_rule(f, a, b, n=100):
    """Правило трапеций"""
    h = (b - a) / n
    x = np.linspace(a, b, n + 1)
    y = f(x)
    
    return h * (y[0]/2 + np.sum(y[1:-1]) + y[-1]/2)

def simpson_rule(f, a, b, n=100):
    """Правило Симпсона"""
    if n % 2 == 1:
        n += 1  # Должно быть четным
    
    h = (b - a) / n
    x = np.linspace(a, b, n + 1)
    y = f(x)
    
    return h/3 * (y[0] + 4*np.sum(y[1:-1:2]) + 2*np.sum(y[2:-1:2]) + y[-1])

def monte_carlo_integration(f, a, b, n=100000):
    """Интегрирование методом Монте-Карло"""
    x_random = np.random.uniform(a, b, n)
    y_values = f(x_random)
    
    return (b - a) * np.mean(y_values)

# Пример: интегрирование функции e^(-x^2)
def gaussian(x):
    return np.exp(-x**2)

# Сравнение методов
a, b = -2, 2
true_value, _ = quad(gaussian, a, b)
trap_value = trapezoidal_rule(gaussian, a, b)
simp_value = simpson_rule(gaussian, a, b)
mc_value = monte_carlo_integration(gaussian, a, b)

print(f"Точное значение: {true_value:.6f}")
print(f"Трапеции: {trap_value:.6f}")
print(f"Симпсон: {simp_value:.6f}")
print(f"Монте-Карло: {mc_value:.6f}")
```

### Применение в машинном обучении

```python
def gaussian_kernel(x, mu, sigma):
    """Гауссово ядро для SVM"""
    return np.exp(-(x - mu)**2 / (2 * sigma**2))

def integrate_gaussian_mixture(components, x_range):
    """Интегрирование смеси гауссовых распределений"""
    def mixture(x):
        return sum(w * gaussian_kernel(x, mu, sigma) 
                  for w, mu, sigma in components)
    
    integral, _ = quad(mixture, x_range[0], x_range[1])
    return integral

# Пример: смесь двух гауссовых распределений
components = [
    (0.3, 0, 1),    # вес=0.3, μ=0, σ=1
    (0.7, 2, 0.5)   # вес=0.7, μ=2, σ=0.5
]

total_prob = integrate_gaussian_mixture(components, [-5, 5])
print(f"Полная вероятность: {total_prob}")
```

---

## Ряды и последовательности

### Степенные ряды

```python
import numpy as np
from scipy.special import factorial

def taylor_series(f, x0, n_terms=10):
    """Разложение в ряд Тейлора"""
    def get_derivative(f, x, n, h=1e-8):
        """Численная производная n-го порядка"""
        if n == 0:
            return f(x)
        elif n == 1:
            return (f(x + h) - f(x - h)) / (2 * h)
        else:
            # Рекурсивное вычисление высших производных
            return (get_derivative(f, x + h, n - 1, h) - 
                   get_derivative(f, x - h, n - 1, h)) / (2 * h)
    
    def taylor_approximation(x):
        result = 0
        for n in range(n_terms):
            coeff = get_derivative(f, x0, n) / factorial(n)
            result += coeff * (x - x0)**n
        return result
    
    return taylor_approximation

# Пример: разложение exp(x) в ряд Тейлора
def exp_func(x):
    return np.exp(x)

exp_taylor = taylor_series(exp_func, x0=0, n_terms=15)

# Проверка точности
x_test = 1.0
exact = np.exp(x_test)
approx = exp_taylor(x_test)

print(f"Точное значение exp({x_test}): {exact}")
print(f"Приближение Тейлора: {approx}")
print(f"Ошибка: {abs(exact - approx)}")
```

### Ряды Фурье

```python
def fourier_series(f, period, n_terms=10):
    """Разложение в ряд Фурье"""
    T = period
    
    # Вычисляем коэффициенты
    def a0():
        return 2/T * quad(f, 0, T)[0]
    
    def an(n):
        return 2/T * quad(lambda x: f(x) * np.cos(2*np.pi*n*x/T), 0, T)[0]
    
    def bn(n):
        return 2/T * quad(lambda x: f(x) * np.sin(2*np.pi*n*x/T), 0, T)[0]
    
    # Аппроксимация
    def fourier_approx(x):
        result = a0() / 2
        for n in range(1, n_terms + 1):
            result += an(n) * np.cos(2*np.pi*n*x/T)
            result += bn(n) * np.sin(2*np.pi*n*x/T)
        return result
    
    return fourier_approx

# Пример: разложение прямоугольной функции
def square_wave(x):
    return 1 if x % 2 < 1 else -1

square_fourier = fourier_series(square_wave, period=2, n_terms=20)

# Визуализация
x_values = np.linspace(0, 4, 1000)
original = [square_wave(x) for x in x_values]
approximation = [square_fourier(x) for x in x_values]

plt.figure(figsize=(12, 6))
plt.plot(x_values, original, 'b-', label='Оригинальная функция', linewidth=2)
plt.plot(x_values, approximation, 'r--', label='Ряд Фурье', linewidth=2)
plt.legend()
plt.grid(True)
plt.title('Разложение прямоугольной функции в ряд Фурье')
plt.show()
```

---

## Многомерный анализ

### Векторный анализ

```python
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

class VectorField:
    """Класс для работы с векторными полями"""
    
    def __init__(self, field_func):
        self.field_func = field_func
    
    def divergence(self, point, h=1e-6):
        """Вычисление дивергенции"""
        x, y, z = point
        
        # Частные производные компонент
        fx_x = (self.field_func([x + h, y, z])[0] - 
               self.field_func([x - h, y, z])[0]) / (2 * h)
        fy_y = (self.field_func([x, y + h, z])[1] - 
               self.field_func([x, y - h, z])[1]) / (2 * h)
        fz_z = (self.field_func([x, y, z + h])[2] - 
               self.field_func([x, y, z - h])[2]) / (2 * h)
        
        return fx_x + fy_y + fz_z
    
    def curl(self, point, h=1e-6):
        """Вычисление ротора"""
        x, y, z = point
        
        # Компоненты ротора
        curl_x = ((self.field_func([x, y, z + h])[1] - 
                  self.field_func([x, y, z - h])[1]) / (2 * h) -
                 (self.field_func([x, y + h, z])[2] - 
                  self.field_func([x, y - h, z])[2]) / (2 * h))
        
        curl_y = ((self.field_func([x + h, y, z])[2] - 
                  self.field_func([x - h, y, z])[2]) / (2 * h) -
                 (self.field_func([x, y, z + h])[0] - 
                  self.field_func([x, y, z - h])[0]) / (2 * h))
        
        curl_z = ((self.field_func([x, y + h, z])[0] - 
                  self.field_func([x, y - h, z])[0]) / (2 * h) -
                 (self.field_func([x + h, y, z])[1] - 
                  self.field_func([x - h, y, z])[1]) / (2 * h))
        
        return np.array([curl_x, curl_y, curl_z])

# Пример: электростатическое поле
def electric_field(r):
    x, y, z = r
    r_mag = np.sqrt(x**2 + y**2 + z**2)
    if r_mag == 0:
        return np.array([0, 0, 0])
    
    # Поле точечного заряда
    k = 1  # константа
    return k * np.array([x, y, z]) / r_mag**3

efield = VectorField(electric_field)

# Проверяем дивергенцию в разных точках
test_points = [[1, 0, 0], [0, 1, 0], [1, 1, 1]]
for point in test_points:
    div = efield.divergence(point)
    curl = efield.curl(point)
    print(f"Точка {point}: div = {div:.6f}, curl = {curl}")
```

---

## Применения в программировании

### Оптимизация алгоритмов

```python
from scipy.optimize import minimize
import numpy as np

def gradient_based_optimization():
    """Оптимизация на основе градиента"""
    
    # Функция Розенброка (сложная для оптимизации)
    def rosenbrock(x):
        return sum(100 * (x[i+1] - x[i]**2)**2 + (1 - x[i])**2 
                  for i in range(len(x) - 1))
    
    def rosenbrock_gradient(x):
        grad = np.zeros_like(x)
        grad[0] = -400 * x[0] * (x[1] - x[0]**2) - 2 * (1 - x[0])
        
        for i in range(1, len(x) - 1):
            grad[i] = (200 * (x[i] - x[i-1]**2) - 
                      400 * x[i] * (x[i+1] - x[i]**2) - 
                      2 * (1 - x[i]))
        
        grad[-1] = 200 * (x[-1] - x[-2]**2)
        return grad
    
    # Различные методы оптимизации
    methods = ['BFGS', 'L-BFGS-B', 'CG', 'Newton-CG']
    x0 = np.array([0, 0])
    
    for method in methods:
        if method == 'Newton-CG':
            result = minimize(rosenbrock, x0, method=method, 
                            jac=rosenbrock_gradient)
        else:
            result = minimize(rosenbrock, x0, method=method)
        
        print(f"{method}: x = {result.x}, f(x) = {result.fun}, "
              f"iterations = {result.nit}")

gradient_based_optimization()
```

### Анализ сложности алгоритмов

```python
import time
import numpy as np
import matplotlib.pyplot as plt

def complexity_analysis():
    """Эмпирический анализ сложности алгоритмов"""
    
    def bubble_sort(arr):
        n = len(arr)
        for i in range(n):
            for j in range(0, n - i - 1):
                if arr[j] > arr[j + 1]:
                    arr[j], arr[j + 1] = arr[j + 1], arr[j]
        return arr
    
    def merge_sort(arr):
        if len(arr) <= 1:
            return arr
        
        mid = len(arr) // 2
        left = merge_sort(arr[:mid])
        right = merge_sort(arr[mid:])
        
        result = []
        i = j = 0
        
        while i < len(left) and j < len(right):
            if left[i] <= right[j]:
                result.append(left[i])
                i += 1
            else:
                result.append(right[j])
                j += 1
        
        result.extend(left[i:])
        result.extend(right[j:])
        return result
    
    # Тестирование на различных размерах
    sizes = [100, 200, 500, 1000, 2000]
    bubble_times = []
    merge_times = []
    
    for size in sizes:
        # Генерируем случайные данные
        data = np.random.randint(0, 1000, size).tolist()
        
        # Тестируем bubble sort
        start = time.time()
        bubble_sort(data.copy())
        bubble_times.append(time.time() - start)
        
        # Тестируем merge sort
        start = time.time()
        merge_sort(data.copy())
        merge_times.append(time.time() - start)
    
    # Анализ роста времени выполнения
    plt.figure(figsize=(10, 6))
    plt.plot(sizes, bubble_times, 'ro-', label='Bubble Sort O(n²)')
    plt.plot(sizes, merge_times, 'bo-', label='Merge Sort O(n log n)')
    plt.xlabel('Размер массива')
    plt.ylabel('Время выполнения (сек)')
    plt.legend()
    plt.grid(True)
    plt.title('Сравнение сложности алгоритмов сортировки')
    plt.show()
    
    # Теоретические vs эмпирические оценки
    theoretical_bubble = [n**2 for n in sizes]
    theoretical_merge = [n * np.log(n) for n in sizes]
    
    # Нормализация для сравнения
    bubble_normalized = np.array(bubble_times) / max(bubble_times)
    merge_normalized = np.array(merge_times) / max(merge_times)
    theoretical_bubble_norm = np.array(theoretical_bubble) / max(theoretical_bubble)
    theoretical_merge_norm = np.array(theoretical_merge) / max(theoretical_merge)
    
    print("Корреляция эмпирических и теоретических оценок:")
    print(f"Bubble Sort: {np.corrcoef(bubble_normalized, theoretical_bubble_norm)[0,1]:.3f}")
    print(f"Merge Sort: {np.corrcoef(merge_normalized, theoretical_merge_norm)[0,1]:.3f}")

complexity_analysis()
```

## 🎯 Практические задания

### Задание 1: Оптимизация нейронной сети
Реализуйте backpropagation используя производные и цепное правило.

### Задание 2: Численное интегрирование
Создайте систему для вычисления площади под кривой ROC.

### Задание 3: Ряды в анализе данных
Используйте ряды Фурье для анализа периодических данных.

### Задание 4: Многомерная оптимизация
Реализуйте алгоритм градиентного спуска для линейной регрессии.

---

## 🔗 Связанные материалы

- [Линейная алгебра](./linear-algebra.md)
- [Численные методы](./numerical-methods.md)
- [Машинное обучение](../algorithms-data-structures/machine-learning-fundamentals.md)
- [Оптимизация](./optimization.md) 