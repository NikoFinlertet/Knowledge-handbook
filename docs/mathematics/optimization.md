# Оптимизация: Теория и практика

## 🎯 Цель курса

Изучение методов оптимизации для решения задач в программировании, машинном обучении и операционных исследованиях.

## 📚 Содержание

1. [Линейное программирование](#линейное-программирование)
2. [Нелинейная оптимизация](#нелинейная-оптимизация)
3. [Дискретная оптимизация](#дискретная-оптимизация)
4. [Многокритериальная оптимизация](#многокритериальная-оптимизация)
5. [Стохастическая оптимизация](#стохастическая-оптимизация)
6. [Метаэвристики](#метаэвристики)

---

## Линейное программирование

### Симплекс-метод

```python
import numpy as np
from scipy.optimize import linprog

class SimplexSolver:
    """Решение задач линейного программирования симплекс-методом"""
    
    def __init__(self, c, A_ub, b_ub, A_eq=None, b_eq=None, bounds=None):
        """
        Стандартная форма: min c^T x
        subject to: A_ub x <= b_ub
                   A_eq x = b_eq
                   bounds
        """
        self.c = np.array(c)
        self.A_ub = np.array(A_ub) if A_ub is not None else None
        self.b_ub = np.array(b_ub) if b_ub is not None else None
        self.A_eq = np.array(A_eq) if A_eq is not None else None
        self.b_eq = np.array(b_eq) if b_eq is not None else None
        self.bounds = bounds
    
    def solve(self):
        """Решение задачи"""
        result = linprog(
            c=self.c,
            A_ub=self.A_ub,
            b_ub=self.b_ub,
            A_eq=self.A_eq,
            b_eq=self.b_eq,
            bounds=self.bounds,
            method='highs'
        )
        
        return result
    
    def dual_problem(self):
        """Построение двойственной задачи"""
        if self.A_ub is None:
            raise ValueError("Нужны ограничения неравенства для двойственной задачи")
        
        # Двойственная задача: max b^T y
        # subject to: A^T y <= c
        #             y >= 0
        
        c_dual = self.b_ub
        A_dual = -self.A_ub.T
        b_dual = -self.c
        
        return SimplexSolver(
            c=-c_dual,  # Меняем знак для максимизации
            A_ub=A_dual,
            b_ub=b_dual,
            bounds=[(0, None)] * len(c_dual)  # y >= 0
        )

# Пример: задача о диете
def diet_problem():
    """
    Минимизировать стоимость диеты при соблюдении питательных требований
    """
    
    # Продукты: хлеб, молоко, яйца, мясо
    # Стоимость за единицу
    costs = [2, 3, 4, 8]
    
    # Питательные вещества: белки, жиры, углеводы (на единицу продукта)
    nutrients = [
        [4, 2, 15, 20],  # белки
        [1, 4, 2, 15],   # жиры
        [15, 8, 1, 5]    # углеводы
    ]
    
    # Минимальные требования
    requirements = [50, 20, 60]
    
    # Максимальные ограничения на продукты
    max_amounts = [10, 5, 2, 1]
    
    # Формирование задачи
    solver = SimplexSolver(
        c=costs,
        A_ub=[-np.array(nutrients), np.eye(4)],  # >= constraints and <= constraints
        b_ub=[-np.array(requirements), max_amounts],
        bounds=[(0, None)] * 4  # x >= 0
    )
    
    # Исправляем формирование матрицы ограничений
    A_ub = np.vstack([
        -np.array(nutrients),  # Для >= ограничений
        np.eye(4)              # Для <= ограничений
    ])
    
    b_ub = np.hstack([
        -np.array(requirements),  # Для >= ограничений
        max_amounts               # Для <= ограничений
    ])
    
    solver = SimplexSolver(c=costs, A_ub=A_ub, b_ub=b_ub, bounds=[(0, None)] * 4)
    
    result = solver.solve()
    
    if result.success:
        print("Оптимальное решение:")
        products = ['хлеб', 'молоко', 'яйца', 'мясо']
        for i, amount in enumerate(result.x):
            print(f"{products[i]}: {amount:.2f} единиц")
        print(f"Минимальная стоимость: {result.fun:.2f}")
    else:
        print("Решение не найдено")

diet_problem()
```

### Транспортная задача

```python
def transportation_problem():
    """
    Задача о транспортировке товаров от поставщиков к потребителям
    """
    
    # Предложение (поставщики)
    supply = [20, 30, 25]
    
    # Спрос (потребители)
    demand = [15, 25, 10, 25]
    
    # Стоимость перевозки от поставщика i к потребителю j
    costs = [
        [2, 3, 4, 1],
        [1, 2, 3, 2],
        [3, 1, 2, 4]
    ]
    
    m, n = len(supply), len(demand)
    
    # Проверка сбалансированности
    total_supply = sum(supply)
    total_demand = sum(demand)
    
    if total_supply != total_demand:
        print(f"Несбалансированная задача: предложение={total_supply}, спрос={total_demand}")
        return
    
    # Формирование задачи линейного программирования
    # Переменные: x_ij - количество товара от поставщика i к потребителю j
    
    # Целевая функция: минимизировать общую стоимость
    c = []
    for i in range(m):
        for j in range(n):
            c.append(costs[i][j])
    
    # Ограничения предложения: sum_j x_ij = supply[i]
    A_eq_supply = np.zeros((m, m * n))
    b_eq_supply = supply
    
    for i in range(m):
        for j in range(n):
            A_eq_supply[i, i * n + j] = 1
    
    # Ограничения спроса: sum_i x_ij = demand[j]
    A_eq_demand = np.zeros((n, m * n))
    b_eq_demand = demand
    
    for j in range(n):
        for i in range(m):
            A_eq_demand[j, i * n + j] = 1
    
    # Объединение ограничений
    A_eq = np.vstack([A_eq_supply, A_eq_demand])
    b_eq = np.hstack([b_eq_supply, b_eq_demand])
    
    # Решение
    result = linprog(
        c=c,
        A_eq=A_eq,
        b_eq=b_eq,
        bounds=[(0, None)] * len(c),
        method='highs'
    )
    
    if result.success:
        print("Оптимальный план перевозок:")
        x = result.x.reshape(m, n)
        
        for i in range(m):
            for j in range(n):
                if x[i, j] > 1e-6:  # Избегаем очень маленьких значений
                    print(f"От поставщика {i+1} к потребителю {j+1}: {x[i, j]:.2f}")
        
        print(f"Минимальная стоимость: {result.fun:.2f}")
    else:
        print("Решение не найдено")

transportation_problem()
```

---

## Нелинейная оптимизация

### Метод множителей Лагранжа

```python
import numpy as np
from scipy.optimize import minimize, NonlinearConstraint

class LagrangeMultipliersSolver:
    """Решение задач с ограничениями методом множителей Лагранжа"""
    
    def __init__(self, objective, constraints, jacobian=None, hessian=None):
        self.objective = objective
        self.constraints = constraints
        self.jacobian = jacobian
        self.hessian = hessian
    
    def solve(self, x0, method='SLSQP'):
        """Решение задачи оптимизации"""
        
        # Преобразуем ограничения в формат scipy
        constraints_list = []
        
        for constraint in self.constraints:
            if constraint['type'] == 'eq':
                constraints_list.append({
                    'type': 'eq',
                    'fun': constraint['fun'],
                    'jac': constraint.get('jac', None)
                })
            elif constraint['type'] == 'ineq':
                constraints_list.append({
                    'type': 'ineq',
                    'fun': constraint['fun'],
                    'jac': constraint.get('jac', None)
                })
        
        # Решение
        result = minimize(
            fun=self.objective,
            x0=x0,
            method=method,
            jac=self.jacobian,
            hess=self.hessian,
            constraints=constraints_list
        )
        
        return result
    
    def analytical_solution(self, x0):
        """Аналитическое решение (для простых случаев)"""
        # Это пример для задачи с одним ограничением равенства
        # Более сложные случаи требуют численных методов
        
        def lagrangian_gradient(vars):
            """Градиент лагранжиана"""
            x = vars[:-1]  # Основные переменные
            lam = vars[-1]  # Множитель Лагранжа
            
            # Градиент целевой функции
            grad_f = np.array([2*x[0], 2*x[1]])  # Для f(x) = x1² + x2²
            
            # Градиент ограничения
            grad_g = np.array([1, 1])  # Для g(x) = x1 + x2 - 1 = 0
            
            # Условия KKT
            grad_L = grad_f + lam * grad_g
            constraint = x[0] + x[1] - 1
            
            return np.hstack([grad_L, constraint])
        
        # Решение системы уравнений
        from scipy.optimize import fsolve
        
        # Начальное приближение: x1, x2, lambda
        initial_guess = np.hstack([x0, 0])
        
        solution = fsolve(lagrangian_gradient, initial_guess)
        
        return solution[:-1], solution[-1]  # x, lambda

# Пример: задача о максимизации полезности
def utility_maximization():
    """
    Максимизировать полезность U(x1, x2) = x1^0.5 * x2^0.5
    при ограничении бюджета p1*x1 + p2*x2 = I
    """
    
    # Параметры
    p1, p2 = 2, 3  # Цены товаров
    I = 100        # Доход
    
    # Целевая функция (минимизируем отрицательную полезность)
    def utility(x):
        return -(x[0]**0.5 * x[1]**0.5)
    
    # Градиент целевой функции
    def utility_grad(x):
        return np.array([
            -0.5 * x[0]**(-0.5) * x[1]**0.5,
            -0.5 * x[0]**0.5 * x[1]**(-0.5)
        ])
    
    # Ограничение бюджета
    def budget_constraint(x):
        return p1 * x[0] + p2 * x[1] - I
    
    # Градиент ограничения
    def budget_constraint_grad(x):
        return np.array([p1, p2])
    
    # Формирование задачи
    constraints = [{
        'type': 'eq',
        'fun': budget_constraint,
        'jac': budget_constraint_grad
    }]
    
    solver = LagrangeMultipliersSolver(
        objective=utility,
        constraints=constraints,
        jacobian=utility_grad
    )
    
    # Начальное приближение
    x0 = np.array([10, 10])
    
    # Решение
    result = solver.solve(x0)
    
    if result.success:
        x1_opt, x2_opt = result.x
        max_utility = -result.fun
        
        print(f"Оптимальное потребление:")
        print(f"Товар 1: {x1_opt:.2f}")
        print(f"Товар 2: {x2_opt:.2f}")
        print(f"Максимальная полезность: {max_utility:.4f}")
        print(f"Проверка бюджета: {p1*x1_opt + p2*x2_opt:.2f} = {I}")
    else:
        print("Решение не найдено")

utility_maximization()
```

### Квадратичное программирование

```python
def quadratic_programming():
    """
    Решение задач квадратичного программирования
    min 0.5 x^T H x + c^T x
    subject to: A x <= b
               A_eq x = b_eq
    """
    
    # Пример: задача о портфеле Марковица
    # Минимизировать риск портфеля при заданной доходности
    
    # Ожидаемые доходности активов
    expected_returns = np.array([0.12, 0.10, 0.07, 0.03])
    
    # Ковариационная матрица (риск)
    cov_matrix = np.array([
        [0.005, -0.010, 0.004, 0.000],
        [-0.010, 0.040, -0.002, 0.000],
        [0.004, -0.002, 0.023, 0.000],
        [0.000, 0.000, 0.000, 0.000]
    ])
    
    # Желаемая доходность портфеля
    target_return = 0.08
    
    n = len(expected_returns)
    
    # Матрица Гессе (удваиваем для формулировки 0.5 x^T H x)
    H = 2 * cov_matrix
    
    # Линейная часть (нет в задаче минимизации дисперсии)
    c = np.zeros(n)
    
    # Ограничения
    # 1. Сумма весов = 1
    A_eq = np.array([np.ones(n)])
    b_eq = np.array([1.0])
    
    # 2. Ожидаемая доходность = target_return
    A_eq = np.vstack([A_eq, expected_returns])
    b_eq = np.hstack([b_eq, target_return])
    
    # 3. Веса >= 0 (неотрицательные позиции)
    bounds = [(0, None)] * n
    
    # Решение с помощью scipy
    from scipy.optimize import minimize
    
    def objective(x):
        return 0.5 * x.T @ H @ x
    
    def objective_grad(x):
        return H @ x
    
    def objective_hess(x):
        return H
    
    result = minimize(
        fun=objective,
        x0=np.ones(n) / n,  # Равномерное распределение как начальное приближение
        method='SLSQP',
        jac=objective_grad,
        hess=objective_hess,
        constraints={'type': 'eq', 'fun': lambda x: np.hstack([
            np.sum(x) - 1,
            np.dot(expected_returns, x) - target_return
        ])},
        bounds=bounds
    )
    
    if result.success:
        optimal_weights = result.x
        portfolio_return = np.dot(expected_returns, optimal_weights)
        portfolio_risk = np.sqrt(optimal_weights.T @ cov_matrix @ optimal_weights)
        
        print("Оптимальный портфель:")
        assets = ['Акции', 'Облигации', 'Недвижимость', 'Денежные средства']
        for i, weight in enumerate(optimal_weights):
            print(f"{assets[i]}: {weight:.3f} ({weight*100:.1f}%)")
        
        print(f"\nОжидаемая доходность: {portfolio_return:.4f} ({portfolio_return*100:.2f}%)")
        print(f"Риск (стандартное отклонение): {portfolio_risk:.4f} ({portfolio_risk*100:.2f}%)")
    else:
        print("Решение не найдено")

quadratic_programming()
```

---

## Дискретная оптимизация

### Задача о рюкзаке

```python
def knapsack_problem():
    """
    Решение задачи о рюкзаке динамическим программированием
    """
    
    # Предметы: (стоимость, вес)
    items = [
        (60, 10),  # Предмет 1
        (100, 20), # Предмет 2
        (120, 30), # Предмет 3
    ]
    
    capacity = 50  # Вместимость рюкзака
    
    n = len(items)
    values = [item[0] for item in items]
    weights = [item[1] for item in items]
    
    # DP таблица: dp[i][w] = максимальная стоимость для первых i предметов и веса w
    dp = [[0 for _ in range(capacity + 1)] for _ in range(n + 1)]
    
    # Заполнение таблицы
    for i in range(1, n + 1):
        for w in range(capacity + 1):
            # Не берем предмет i
            dp[i][w] = dp[i-1][w]
            
            # Берем предмет i, если он помещается
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i][w], dp[i-1][w-weights[i-1]] + values[i-1])
    
    # Восстановление решения
    max_value = dp[n][capacity]
    selected_items = []
    
    w = capacity
    for i in range(n, 0, -1):
        if dp[i][w] != dp[i-1][w]:
            selected_items.append(i-1)
            w -= weights[i-1]
    
    print("Решение задачи о рюкзаке:")
    print(f"Максимальная стоимость: {max_value}")
    print("Выбранные предметы:")
    
    total_weight = 0
    for item_idx in selected_items:
        value, weight = items[item_idx]
        print(f"Предмет {item_idx + 1}: стоимость={value}, вес={weight}")
        total_weight += weight
    
    print(f"Общий вес: {total_weight}/{capacity}")

knapsack_problem()
```

### Задача коммивояжера

```python
def tsp_dynamic_programming():
    """
    Решение задачи коммивояжера точным методом (алгоритм Хелда-Карпа)
    """
    
    # Матрица расстояний
    distances = [
        [0, 29, 20, 21],
        [29, 0, 15, 17],
        [20, 15, 0, 28],
        [21, 17, 28, 0]
    ]
    
    n = len(distances)
    
    # dp[mask][i] = минимальная стоимость посещения городов в mask, заканчивая в городе i
    dp = {}
    
    # Инициализация: из города 0 в каждый другой город
    for i in range(1, n):
        dp[(1 << i, i)] = distances[0][i]
    
    # Заполнение таблицы для всех подмножеств
    for mask in range(1, 1 << n):
        for i in range(n):
            if not (mask & (1 << i)):
                continue
            
            if (mask, i) in dp:
                continue
            
            dp[(mask, i)] = float('inf')
            
            for j in range(n):
                if i == j or not (mask & (1 << j)):
                    continue
                
                prev_mask = mask ^ (1 << i)
                if (prev_mask, j) in dp:
                    dp[(mask, i)] = min(dp[(mask, i)], dp[(prev_mask, j)] + distances[j][i])
    
    # Найти минимальную стоимость возвращения в город 0
    final_mask = (1 << n) - 1
    min_cost = float('inf')
    last_city = -1
    
    for i in range(1, n):
        if (final_mask, i) in dp:
            cost = dp[(final_mask, i)] + distances[i][0]
            if cost < min_cost:
                min_cost = cost
                last_city = i
    
    # Восстановление пути
    path = [0]
    mask = final_mask
    current = last_city
    
    while mask != 1:
        path.append(current)
        
        for prev in range(n):
            if prev == current or not (mask & (1 << prev)):
                continue
            
            prev_mask = mask ^ (1 << current)
            if (prev_mask, prev) in dp:
                if dp[(prev_mask, prev)] + distances[prev][current] == dp[(mask, current)]:
                    mask = prev_mask
                    current = prev
                    break
    
    path.append(0)
    path.reverse()
    
    print("Решение задачи коммивояжера:")
    print(f"Минимальная стоимость: {min_cost}")
    print(f"Оптимальный маршрут: {' -> '.join(map(str, path))}")

tsp_dynamic_programming()
```

---

## Метаэвристики

### Генетический алгоритм

```python
import random
import numpy as np

class GeneticAlgorithm:
    """Генетический алгоритм для оптимизации"""
    
    def __init__(self, fitness_function, chromosome_length, population_size=100,
                 mutation_rate=0.01, crossover_rate=0.8, elite_size=5):
        self.fitness_function = fitness_function
        self.chromosome_length = chromosome_length
        self.population_size = population_size
        self.mutation_rate = mutation_rate
        self.crossover_rate = crossover_rate
        self.elite_size = elite_size
        self.population = []
        self.generation = 0
    
    def initialize_population(self):
        """Инициализация популяции"""
        self.population = []
        for _ in range(self.population_size):
            chromosome = [random.random() for _ in range(self.chromosome_length)]
            self.population.append(chromosome)
    
    def selection(self, population, fitness_scores):
        """Турнирный отбор"""
        tournament_size = 3
        selected = []
        
        for _ in range(len(population)):
            tournament = random.sample(list(zip(population, fitness_scores)), tournament_size)
            winner = max(tournament, key=lambda x: x[1])
            selected.append(winner[0])
        
        return selected
    
    def crossover(self, parent1, parent2):
        """Одноточечное скрещивание"""
        if random.random() > self.crossover_rate:
            return parent1, parent2
        
        crossover_point = random.randint(1, self.chromosome_length - 1)
        
        child1 = parent1[:crossover_point] + parent2[crossover_point:]
        child2 = parent2[:crossover_point] + parent1[crossover_point:]
        
        return child1, child2
    
    def mutation(self, chromosome):
        """Мутация"""
        mutated = chromosome.copy()
        
        for i in range(len(mutated)):
            if random.random() < self.mutation_rate:
                mutated[i] = random.random()
        
        return mutated
    
    def evolve(self, generations=100):
        """Эволюция популяции"""
        self.initialize_population()
        
        best_fitness_history = []
        avg_fitness_history = []
        
        for generation in range(generations):
            # Оценка приспособленности
            fitness_scores = [self.fitness_function(chromo) for chromo in self.population]
            
            # Статистика
            best_fitness = max(fitness_scores)
            avg_fitness = sum(fitness_scores) / len(fitness_scores)
            best_fitness_history.append(best_fitness)
            avg_fitness_history.append(avg_fitness)
            
            # Элитизм - сохраняем лучших
            sorted_population = sorted(zip(self.population, fitness_scores), 
                                     key=lambda x: x[1], reverse=True)
            elite = [chromo for chromo, _ in sorted_population[:self.elite_size]]
            
            # Отбор
            selected = self.selection(self.population, fitness_scores)
            
            # Скрещивание и мутация
            new_population = elite.copy()
            
            while len(new_population) < self.population_size:
                parent1, parent2 = random.sample(selected, 2)
                child1, child2 = self.crossover(parent1, parent2)
                
                new_population.append(self.mutation(child1))
                if len(new_population) < self.population_size:
                    new_population.append(self.mutation(child2))
            
            self.population = new_population[:self.population_size]
            self.generation = generation
            
            if generation % 10 == 0:
                print(f"Поколение {generation}: лучшая приспособленность = {best_fitness:.4f}")
        
        # Возвращаем лучшее решение
        final_fitness = [self.fitness_function(chromo) for chromo in self.population]
        best_idx = final_fitness.index(max(final_fitness))
        
        return self.population[best_idx], max(final_fitness), best_fitness_history, avg_fitness_history

# Пример: оптимизация функции
def optimize_function():
    """Оптимизация многомерной функции"""
    
    # Функция для оптимизации: f(x) = sum(x_i^2) для x_i в [-5, 5]
    def fitness_function(chromosome):
        # Масштабируем хромосому в диапазон [-5, 5]
        scaled = [(gene - 0.5) * 10 for gene in chromosome]
        # Минимизируем сумму квадратов (отрицательная для максимизации)
        return -sum(x**2 for x in scaled)
    
    # Создание и запуск ГА
    ga = GeneticAlgorithm(
        fitness_function=fitness_function,
        chromosome_length=5,
        population_size=50,
        mutation_rate=0.1,
        crossover_rate=0.8
    )
    
    best_solution, best_fitness, history, _ = ga.evolve(generations=50)
    
    # Масштабируем результат обратно
    scaled_solution = [(gene - 0.5) * 10 for gene in best_solution]
    
    print("Результат генетического алгоритма:")
    print(f"Лучшее решение: {[f'{x:.4f}' for x in scaled_solution]}")
    print(f"Значение функции: {-best_fitness:.4f}")

optimize_function()
```

---

## 🎯 Практические задания

### Задание 1: Планирование производства
Решите задачу планирования производства с ограничениями ресурсов.

### Задание 2: Оптимизация портфеля
Создайте систему оптимизации инвестиционного портфеля.

### Задание 3: Маршрутизация
Реализуйте оптимизацию маршрутов для службы доставки.

### Задание 4: Машинное обучение
Оптимизируйте гиперпараметры модели машинного обучения.

---

## 🔗 Связанные материалы

- [Математический анализ](./mathematical-analysis.md)
- [Численные методы](./numerical-methods.md)
- [Линейная алгебра](./linear-algebra.md)
- [Машинное обучение](../algorithms-data-structures/machine-learning-fundamentals.md) 