# Линейная алгебра в вычислениях 🔢

> **Навигация**: [[graph-theory-structures|← Теория графов]] | [[mathematics-algorithms|Главная]] | [[probability-randomized-algorithms|Вероятностные алгоритмы →]]

## Содержание
1. [Матричные операции и сложность](#🔢-матричные-операции-и-их-алгоритмическая-сложность)
2. [Быстрое возведение в степень](#⚡-быстрое-возведение-в-степень)
3. [Собственные векторы и PageRank](#🏆-собственные-векторы-и-pagerank)
4. [SVD и сжатие данных](#📊-svd-и-сжатие-данных)
5. [Практические применения](#🚀-практические-применения)

---

## 🔢 Матричные операции и их алгоритмическая сложность

**Математическая основа:** Операции с матрицами, собственные векторы, разложения

**Алгоритмические применения:**
- Быстрое возведение в степень
- Решение систем уравнений
- Анализ главных компонент (PCA)
- PageRank алгоритм

```python
import numpy as np
from typing import List, Tuple

class MatrixAlgorithms:
    """Алгоритмы линейной алгебры и их применения"""
    
    @staticmethod
    def matrix_power_fast(matrix, n):
        """
        Быстрое возведение матрицы в степень за O(log n)
        Основано на бинарном разложении степени
        """
        if n == 0:
            return np.eye(matrix.shape[0])
        if n == 1:
            return matrix
        
        if n % 2 == 0:
            half_power = MatrixAlgorithms.matrix_power_fast(matrix, n // 2)
            return np.dot(half_power, half_power)
        else:
            return np.dot(matrix, MatrixAlgorithms.matrix_power_fast(matrix, n - 1))
    
    @staticmethod
    def gaussian_elimination(A, b):
        """
        Решение системы линейных уравнений методом Гаусса
        Ax = b
        """
        n = len(A)
        # Создаем расширенную матрицу
        augmented = np.column_stack([A.astype(float), b.astype(float)])
        
        # Прямой ход
        for i in range(n):
            # Поиск максимального элемента (частичное выбор ведущего элемента)
            max_row = i
            for k in range(i + 1, n):
                if abs(augmented[k][i]) > abs(augmented[max_row][i]):
                    max_row = k
            
            # Обмен строк
            augmented[[i, max_row]] = augmented[[max_row, i]]
            
            # Проверка на вырожденность
            if abs(augmented[i][i]) < 1e-10:
                raise ValueError("Матрица вырождена")
            
            # Обнуление столбца под ведущим элементом
            for k in range(i + 1, n):
                factor = augmented[k][i] / augmented[i][i]
                augmented[k] = augmented[k] - factor * augmented[i]
        
        # Обратный ход
        x = np.zeros(n)
        for i in range(n - 1, -1, -1):
            x[i] = augmented[i][n]
            for j in range(i + 1, n):
                x[i] -= augmented[i][j] * x[j]
            x[i] /= augmented[i][i]
        
        return x
    
    @staticmethod
    def lu_decomposition(A):
        """
        LU разложение матрицы A = LU
        """
        n = A.shape[0]
        L = np.eye(n)
        U = A.copy().astype(float)
        
        for i in range(n):
            for j in range(i + 1, n):
                if abs(U[i][i]) < 1e-10:
                    raise ValueError("Матрица не допускает LU разложение без перестановок")
                
                factor = U[j][i] / U[i][i]
                L[j][i] = factor
                U[j] = U[j] - factor * U[i]
        
        return L, U
```

---

## ⚡ Быстрое возведение в степень

### Применение к последовательности Фибоначчи

```python
@staticmethod
def fibonacci_matrix(n):
    """
    Вычисление n-го числа Фибоначчи через матричное возведение в степень
    [[1, 1], [1, 0]]^n = [[F(n+1), F(n)], [F(n), F(n-1)]]
    """
    if n == 0:
        return 0
    
    base_matrix = np.array([[1, 1], [1, 0]], dtype=object)
    result_matrix = MatrixAlgorithms.matrix_power_fast(base_matrix, n)
    return result_matrix[0][1]

@staticmethod
def recurrence_relation_solver(coefficients, initial_values, n):
    """
    Решение линейных рекуррентных соотношений через матрицы
    a_n = c_1*a_{n-1} + c_2*a_{n-2} + ... + c_k*a_{n-k}
    """
    k = len(coefficients)
    if n < k:
        return initial_values[n] if n < len(initial_values) else 0
    
    # Создаем матрицу перехода
    transition_matrix = np.zeros((k, k))
    transition_matrix[0] = coefficients
    for i in range(1, k):
        transition_matrix[i][i-1] = 1
    
    # Начальное состояние
    state = np.array(initial_values[:k][::-1])  # Обращаем порядок
    
    # Возводим матрицу в степень (n - k + 1)
    if n >= k:
        result_matrix = MatrixAlgorithms.matrix_power_fast(transition_matrix, n - k + 1)
        final_state = np.dot(result_matrix, state)
        return final_state[0]
    
    return initial_values[n]

# Пример: числа Трибоначчи T(n) = T(n-1) + T(n-2) + T(n-3)
def tribonacci(n):
    """Числа Трибоначчи через матричный метод"""
    if n < 3:
        return [0, 1, 1][n] if n >= 0 else 0
    
    return MatrixAlgorithms.recurrence_relation_solver(
        coefficients=[1, 1, 1],
        initial_values=[0, 1, 1],
        n=n
    )
```

---

## 🏆 Собственные векторы и PageRank

```python
@staticmethod
def pagerank(adjacency_matrix, damping_factor=0.85, max_iterations=100, tolerance=1e-6):
    """
    PageRank алгоритм - применение собственных векторов
    Основан на марковских цепях и стационарном распределении
    """
    n = adjacency_matrix.shape[0]
    
    # Нормализация матрицы (стохастическая матрица)
    column_sums = adjacency_matrix.sum(axis=0)
    # Избегаем деления на ноль
    column_sums[column_sums == 0] = 1
    transition_matrix = adjacency_matrix / column_sums
    
    # Матрица PageRank
    M = damping_factor * transition_matrix + (1 - damping_factor) / n * np.ones((n, n))
    
    # Начальное распределение
    pagerank_vector = np.ones(n) / n
    
    # Итеративное вычисление
    for iteration in range(max_iterations):
        new_pagerank = np.dot(M, pagerank_vector)
        
        # Проверка сходимости
        if np.linalg.norm(new_pagerank - pagerank_vector, 1) < tolerance:
            print(f"Сходимость достигнута за {iteration + 1} итераций")
            break
        
        pagerank_vector = new_pagerank
    
    return pagerank_vector

@staticmethod
def power_iteration(matrix, max_iterations=1000, tolerance=1e-10):
    """
    Степенной метод для поиска наибольшего собственного значения и вектора
    """
    n = matrix.shape[0]
    # Случайная начальная точка
    v = np.random.rand(n)
    v = v / np.linalg.norm(v)
    
    eigenvalue = 0
    
    for i in range(max_iterations):
        # Умножение матрицы на вектор
        Av = np.dot(matrix, v)
        
        # Новое собственное значение (отношение Рэлея)
        new_eigenvalue = np.dot(v, Av)
        
        # Нормализация
        v = Av / np.linalg.norm(Av)
        
        # Проверка сходимости
        if abs(new_eigenvalue - eigenvalue) < tolerance:
            break
        
        eigenvalue = new_eigenvalue
    
    return eigenvalue, v
```

---

## 📊 SVD и сжатие данных

```python
@staticmethod
def svd_compression(image_matrix, k):
    """
    Сжатие изображений через SVD разложение
    A ≈ U_k Σ_k V_k^T, где k << min(m,n)
    """
    U, s, Vt = np.linalg.svd(image_matrix, full_matrices=False)
    
    # Оставляем только k главных компонент
    U_k = U[:, :k]
    s_k = s[:k]
    Vt_k = Vt[:k, :]
    
    # Восстанавливаем приближенную матрицу
    compressed = np.dot(U_k, np.dot(np.diag(s_k), Vt_k))
    
    # Степень сжатия
    original_size = image_matrix.shape[0] * image_matrix.shape[1]
    compressed_size = k * (image_matrix.shape[0] + image_matrix.shape[1] + 1)
    compression_ratio = compressed_size / original_size
    
    return compressed, compression_ratio

@staticmethod
def pca_analysis(data_matrix, n_components):
    """
    Анализ главных компонент (PCA) через SVD
    """
    # Центрирование данных
    mean = np.mean(data_matrix, axis=0)
    centered_data = data_matrix - mean
    
    # SVD разложение
    U, s, Vt = np.linalg.svd(centered_data, full_matrices=False)
    
    # Главные компоненты
    principal_components = Vt[:n_components]
    
    # Объясненная дисперсия
    explained_variance = (s**2) / (len(data_matrix) - 1)
    explained_variance_ratio = explained_variance / np.sum(explained_variance)
    
    # Проекция данных на главные компоненты
    transformed_data = np.dot(centered_data, principal_components.T)
    
    return {
        'principal_components': principal_components,
        'explained_variance_ratio': explained_variance_ratio[:n_components],
        'transformed_data': transformed_data,
        'mean': mean
    }
```

---

## 🚀 Практические применения

### Линейная регрессия

```python
@staticmethod
def least_squares_regression(X, y):
    """
    Метод наименьших квадратов: β = (X^T X)^(-1) X^T y
    Применение псевдообращения матрицы
    """
    # Добавляем столбец единиц для свободного члена
    X_with_intercept = np.column_stack([np.ones(X.shape[0]), X])
    
    # Нормальное уравнение
    XTX = np.dot(X_with_intercept.T, X_with_intercept)
    XTy = np.dot(X_with_intercept.T, y)
    
    # Решение системы
    beta = np.linalg.solve(XTX, XTy)
    
    return beta

@staticmethod
def ridge_regression(X, y, alpha=1.0):
    """
    Регуляризованная регрессия (Ridge)
    β = (X^T X + αI)^(-1) X^T y
    """
    X_with_intercept = np.column_stack([np.ones(X.shape[0]), X])
    
    XTX = np.dot(X_with_intercept.T, X_with_intercept)
    XTy = np.dot(X_with_intercept.T, y)
    
    # Добавляем регуляризацию
    regularization = alpha * np.eye(XTX.shape[0])
    regularization[0, 0] = 0  # Не регуляризуем свободный член
    
    beta = np.linalg.solve(XTX + regularization, XTy)
    
    return beta
```

### Рекомендательные системы

```python
class MatrixFactorization:
    """Факторизация матриц для рекомендательных систем"""
    
    def __init__(self, n_factors=50, learning_rate=0.01, regularization=0.1):
        self.n_factors = n_factors
        self.learning_rate = learning_rate
        self.regularization = regularization
    
    def fit(self, rating_matrix, max_iterations=100):
        """
        Обучение модели через градиентный спуск
        R ≈ U V^T, где U - факторы пользователей, V - факторы товаров
        """
        n_users, n_items = rating_matrix.shape
        
        # Инициализация матриц факторов
        self.user_factors = np.random.normal(0, 0.1, (n_users, self.n_factors))
        self.item_factors = np.random.normal(0, 0.1, (n_items, self.n_factors))
        
        # Находим ненулевые элементы
        user_indices, item_indices = np.nonzero(rating_matrix)
        
        for iteration in range(max_iterations):
            for idx in range(len(user_indices)):
                u = user_indices[idx]
                i = item_indices[idx]
                rating = rating_matrix[u, i]
                
                # Предсказание
                prediction = np.dot(self.user_factors[u], self.item_factors[i])
                error = rating - prediction
                
                # Градиентный спуск
                user_factor_old = self.user_factors[u].copy()
                
                self.user_factors[u] += self.learning_rate * (
                    error * self.item_factors[i] - self.regularization * self.user_factors[u]
                )
                
                self.item_factors[i] += self.learning_rate * (
                    error * user_factor_old - self.regularization * self.item_factors[i]
                )
    
    def predict(self, user_id, item_id):
        """Предсказание рейтинга"""
        return np.dot(self.user_factors[user_id], self.item_factors[item_id])
    
    def get_recommendations(self, user_id, n_recommendations=10):
        """Получить рекомендации для пользователя"""
        scores = np.dot(self.user_factors[user_id], self.item_factors.T)
        top_items = np.argsort(scores)[::-1][:n_recommendations]
        return [(item, scores[item]) for item in top_items]
```

**Когда использовать линейную алгебру:**

- ✅ **Быстрое возведение в степень**: Последовательности типа Фибоначчи, вычисление путей в графах
- ✅ **SVD**: Сжатие данных, рекомендательные системы, обработка изображений  
- ✅ **Собственные векторы**: PageRank, анализ социальных сетей, стабильность систем
- ✅ **Псевдообращение**: Регрессионный анализ, решение переопределенных систем

---

## 🔗 Связанные темы

- [[graph-theory-structures|Теория графов]] - матричные представления графов
- [[probability-randomized-algorithms|Вероятностные алгоритмы]] - случайные матрицы
- [[calculus-optimization|Математический анализ]] - оптимизация матричных функций
- [[statistics-data-analysis|Статистика]] - PCA, факторный анализ

## 📚 Дополнительные ресурсы

- **Книги**: "Matrix Computations" - Golub & Van Loan
- **Курсы**: MIT 18.06 Linear Algebra, Stanford CS229
- **Практика**: Numpy tutorials, SciPy linear algebra 