# Математический анализ и оптимизация 📈

> **Навигация**: [[probability-randomized-algorithms|← Вероятностные алгоритмы]] | [[mathematics-algorithms|Главная]] | [[number-theory-cryptography|Теория чисел →]]

## Содержание
1. [Численные методы](#📈-численные-методы)
2. [Градиентная оптимизация](#🎯-градиентная-оптимизация)
3. [Машинное обучение](#🤖-машинное-обучение)
4. [Практические применения](#🚀-практические-применения)

---

## 📈 Численные методы

**Математическая основа:** Производные, интегралы, экстремумы функций

```python
import numpy as np
import math

class NumericalMethods:
    """Численные методы математического анализа"""
    
    @staticmethod
    def newton_raphson(f, df, x0, tolerance=1e-10, max_iterations=100):
        """
        Метод Ньютона для поиска корней
        x_{n+1} = x_n - f(x_n)/f'(x_n)
        """
        x = x0
        for i in range(max_iterations):
            fx = f(x)
            if abs(fx) < tolerance:
                return x, i
            
            dfx = df(x)
            if abs(dfx) < tolerance:
                raise ValueError("Производная близка к нулю")
            
            x_new = x - fx / dfx
            if abs(x_new - x) < tolerance:
                return x_new, i
            
            x = x_new
        
        raise ValueError("Не сошелся за максимальное количество итераций")
    
    @staticmethod
    def numerical_derivative(f, x, h=1e-8):
        """Численное вычисление производной"""
        return (f(x + h) - f(x - h)) / (2 * h)
    
    @staticmethod
    def simpson_integration(f, a, b, n=1000):
        """
        Интегрирование методом Симпсона
        ∫f(x)dx ≈ (h/3)[f(x0) + 4f(x1) + 2f(x2) + ... + f(xn)]
        """
        if n % 2 == 1:
            n += 1  # n должно быть четным
        
        h = (b - a) / n
        x = a
        result = f(a)
        
        for i in range(1, n):
            x += h
            if i % 2 == 0:
                result += 2 * f(x)
            else:
                result += 4 * f(x)
        
        result += f(b)
        return result * h / 3
```

---

## 🎯 Градиентная оптимизация

```python
class OptimizationAlgorithms:
    """Алгоритмы оптимизации"""
    
    @staticmethod
    def gradient_descent(f, gradient_f, x0, learning_rate=0.01, 
                        tolerance=1e-6, max_iterations=10000):
        """
        Градиентный спуск для минимизации функции
        x_{k+1} = x_k - α∇f(x_k)
        """
        x = np.array(x0, dtype=float)
        trajectory = [x.copy()]
        
        for i in range(max_iterations):
            grad = np.array(gradient_f(x))
            x_new = x - learning_rate * grad
            
            if np.linalg.norm(x_new - x) < tolerance:
                return x_new, trajectory, i
            
            x = x_new
            trajectory.append(x.copy())
        
        return x, trajectory, max_iterations
    
    @staticmethod
    def adam_optimizer(f, gradient_f, x0, learning_rate=0.001, 
                      beta1=0.9, beta2=0.999, epsilon=1e-8,
                      max_iterations=10000):
        """
        Adam оптимизатор - адаптивный градиентный спуск
        Использует моменты первого и второго порядка
        """
        x = np.array(x0, dtype=float)
        m = np.zeros_like(x)  # Первый момент
        v = np.zeros_like(x)  # Второй момент
        
        for t in range(1, max_iterations + 1):
            grad = np.array(gradient_f(x))
            
            # Обновление моментов
            m = beta1 * m + (1 - beta1) * grad
            v = beta2 * v + (1 - beta2) * grad**2
            
            # Коррекция смещения
            m_hat = m / (1 - beta1**t)
            v_hat = v / (1 - beta2**t)
            
            # Обновление параметров
            x = x - learning_rate * m_hat / (np.sqrt(v_hat) + epsilon)
        
        return x
```

---

## 🤖 Машинное обучение

### Линейная регрессия с градиентным спуском

```python
class LinearRegressionGD:
    """Линейная регрессия с градиентным спуском"""
    
    def __init__(self, learning_rate=0.01, max_iterations=1000):
        self.learning_rate = learning_rate
        self.max_iterations = max_iterations
        self.weights = None
        self.bias = None
        self.cost_history = []
    
    def cost_function(self, X, y):
        """Функция стоимости (MSE)"""
        m = len(y)
        predictions = X.dot(self.weights) + self.bias
        cost = (1/(2*m)) * np.sum((predictions - y)**2)
        return cost
    
    def fit(self, X, y):
        """Обучение модели"""
        m, n = X.shape
        self.weights = np.zeros(n)
        self.bias = 0
        
        for i in range(self.max_iterations):
            # Предсказания
            predictions = X.dot(self.weights) + self.bias
            
            # Вычисление градиентов
            dw = (1/m) * X.T.dot(predictions - y)
            db = (1/m) * np.sum(predictions - y)
            
            # Обновление параметров
            self.weights -= self.learning_rate * dw
            self.bias -= self.learning_rate * db
            
            # Сохранение стоимости
            cost = self.cost_function(X, y)
            self.cost_history.append(cost)
    
    def predict(self, X):
        """Предсказание"""
        return X.dot(self.weights) + self.bias

class LogisticRegressionGD:
    """Логистическая регрессия с градиентным спуском"""
    
    def __init__(self, learning_rate=0.01, max_iterations=1000):
        self.learning_rate = learning_rate
        self.max_iterations = max_iterations
        self.weights = None
        self.bias = None
    
    def sigmoid(self, z):
        """Сигмоидная функция"""
        # Ограничиваем z для предотвращения переполнения
        z = np.clip(z, -500, 500)
        return 1 / (1 + np.exp(-z))
    
    def cost_function(self, h, y):
        """Функция кросс-энтропии"""
        return (-y * np.log(h) - (1 - y) * np.log(1 - h)).mean()
    
    def fit(self, X, y):
        """Обучение модели"""
        m, n = X.shape
        self.weights = np.zeros(n)
        self.bias = 0
        
        for i in range(self.max_iterations):
            # Прямое распространение
            z = X.dot(self.weights) + self.bias
            h = self.sigmoid(z)
            
            # Вычисление градиентов
            dw = (1/m) * X.T.dot(h - y)
            db = (1/m) * np.sum(h - y)
            
            # Обновление параметров
            self.weights -= self.learning_rate * dw
            self.bias -= self.learning_rate * db
    
    def predict_proba(self, X):
        """Предсказание вероятности"""
        z = X.dot(self.weights) + self.bias
        return self.sigmoid(z)
    
    def predict(self, X):
        """Предсказание класса"""
        return (self.predict_proba(X) >= 0.5).astype(int)
```

---

## 🚀 Практические применения

### Алгоритмы графической оптимизации

```python
class ComputerGraphicsOptimization:
    """Оптимизация в компьютерной графике"""
    
    @staticmethod
    def bezier_curve(control_points, t):
        """
        Кривая Безье - применение производных в графике
        B(t) = Σ(i=0 to n) C(n,i) * (1-t)^(n-i) * t^i * P_i
        """
        n = len(control_points) - 1
        result = np.zeros_like(control_points[0])
        
        for i, point in enumerate(control_points):
            # Биномиальный коэффициент
            coeff = math.comb(n, i)
            # Базисная функция Бернштейна
            basis = coeff * ((1-t)**(n-i)) * (t**i)
            result += basis * np.array(point)
        
        return result
    
    @staticmethod
    def optimize_camera_position(target_objects, constraints):
        """
        Оптимизация позиции камеры для наилучшего обзора
        Минимизируем функцию стоимости на основе видимости объектов
        """
        def cost_function(camera_pos):
            total_cost = 0
            for obj in target_objects:
                # Расстояние до объекта
                distance = np.linalg.norm(np.array(camera_pos) - np.array(obj['position']))
                
                # Угол обзора
                view_angle = math.atan2(obj['size'], distance)
                
                # Функция стоимости (хотим минимизировать)
                cost = -view_angle + 0.1 * distance  # Баланс между близостью и углом
                total_cost += cost
            
            return total_cost
        
        def gradient_cost(camera_pos):
            """Численный градиент функции стоимости"""
            grad = np.zeros(3)
            h = 1e-8
            
            for i in range(3):
                pos_plus = camera_pos.copy()
                pos_minus = camera_pos.copy()
                pos_plus[i] += h
                pos_minus[i] -= h
                
                grad[i] = (cost_function(pos_plus) - cost_function(pos_minus)) / (2 * h)
            
            return grad
        
        # Начальная позиция камеры
        initial_pos = np.array([0.0, 0.0, 5.0])
        
        # Градиентный спуск
        optimal_pos, _, _ = OptimizationAlgorithms.gradient_descent(
            cost_function, gradient_cost, initial_pos, learning_rate=0.1
        )
        
        return optimal_pos

# Демонстрация использования
def demo_calculus_applications():
    """Демонстрация применений математического анализа"""
    
    # Поиск корня функции f(x) = x^2 - 2 (√2)
    def f(x):
        return x**2 - 2
    
    def df(x):
        return 2*x
    
    root, iterations = NumericalMethods.newton_raphson(f, df, x0=1.0)
    print(f"√2 ≈ {root:.10f} (найдено за {iterations} итераций)")
    
    # Численное интегрирование sin(x) от 0 до π
    integral = NumericalMethods.simpson_integration(math.sin, 0, math.pi)
    print(f"∫sin(x)dx от 0 до π ≈ {integral:.6f} (точное значение: 2)")
    
    # Оптимизация квадратичной функции
    def quadratic(x):
        return (x[0] - 3)**2 + (x[1] + 1)**2
    
    def grad_quadratic(x):
        return np.array([2*(x[0] - 3), 2*(x[1] + 1)])
    
    minimum, trajectory, iterations = OptimizationAlgorithms.gradient_descent(
        quadratic, grad_quadratic, [0, 0]
    )
    print(f"Минимум функции: {minimum} за {iterations} итераций")
```

**Когда использовать методы анализа:**

- ✅ **Градиентный спуск**: Обучение нейронных сетей, оптимизация параметров
- ✅ **Численное интегрирование**: Физические симуляции, вычисление площадей
- ✅ **Поиск корней**: Решение уравнений, калибровка моделей
- ✅ **Кривые Безье**: Компьютерная графика, анимация, дизайн

---

## 🔗 Связанные темы

- [[linear-algebra-computation|Линейная алгебра]] - матричные операции в оптимизации
- [[probability-randomized-algorithms|Вероятностные алгоритмы]] - стохастические методы
- [[statistics-data-analysis|Статистика]] - регрессионный анализ
- [[practical-applications|Практические применения]] - машинное обучение

## 📚 Дополнительные ресурсы

- **Книги**: "Numerical Analysis" - Burden & Faires
- **Курсы**: Stanford CS205, MIT 18.335  
- **Практика**: SciPy optimization, TensorFlow/PyTorch 