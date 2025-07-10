# AI Algorithms & Modern Approaches

> **Современные алгоритмы искусственного интеллекта**
>
> От классических подходов до современных архитектур

## 🔬 Алгоритмы поиска

### Поиск в пространстве состояний

```python
import heapq
import numpy as np
from typing import List, Dict, Tuple, Optional, Set
from collections import deque
from dataclasses import dataclass
from abc import ABC, abstractmethod

# ==================== БАЗОВЫЕ АЛГОРИТМЫ ПОИСКА ====================
@dataclass
class SearchNode:
    """
    Узел для алгоритмов поиска
    """
    state: tuple
    parent: Optional['SearchNode'] = None
    action: Optional[str] = None
    path_cost: float = 0
    heuristic_cost: float = 0
    
    def __lt__(self, other):
        return (self.path_cost + self.heuristic_cost) < (other.path_cost + other.heuristic_cost)

class SearchProblem(ABC):
    """
    Абстрактный класс для задач поиска
    """
    
    @abstractmethod
    def get_initial_state(self) -> tuple:
        """Начальное состояние"""
        pass
    
    @abstractmethod
    def is_goal(self, state: tuple) -> bool:
        """Проверка целевого состояния"""
        pass
    
    @abstractmethod
    def get_successors(self, state: tuple) -> List[Tuple[tuple, str, float]]:
        """Возвращает список (следующее_состояние, действие, стоимость)"""
        pass
    
    @abstractmethod
    def heuristic(self, state: tuple) -> float:
        """Эвристическая функция"""
        pass

class AISearch:
    """
    Классические алгоритмы поиска ИИ
    """
    
    def breadth_first_search(self, problem: SearchProblem) -> Optional[List[str]]:
        """
        Поиск в ширину (BFS)
        
        Гарантирует оптимальность для невзвешенных графов
        Временная сложность: O(b^d)
        Пространственная сложность: O(b^d)
        """
        frontier = deque([SearchNode(problem.get_initial_state())])
        explored = set()
        
        while frontier:
            node = frontier.popleft()
            
            if problem.is_goal(node.state):
                return self._get_path(node)
            
            if node.state in explored:
                continue
                
            explored.add(node.state)
            
            for next_state, action, cost in problem.get_successors(node.state):
                if next_state not in explored:
                    child = SearchNode(
                        state=next_state,
                        parent=node,
                        action=action,
                        path_cost=node.path_cost + cost
                    )
                    frontier.append(child)
        
        return None
    
    def depth_first_search(self, problem: SearchProblem) -> Optional[List[str]]:
        """
        Поиск в глубину (DFS)
        
        Может не найти оптимальное решение
        Временная сложность: O(b^m)
        Пространственная сложность: O(bm)
        """
        frontier = [SearchNode(problem.get_initial_state())]
        explored = set()
        
        while frontier:
            node = frontier.pop()
            
            if problem.is_goal(node.state):
                return self._get_path(node)
            
            if node.state in explored:
                continue
                
            explored.add(node.state)
            
            for next_state, action, cost in problem.get_successors(node.state):
                if next_state not in explored:
                    child = SearchNode(
                        state=next_state,
                        parent=node,
                        action=action,
                        path_cost=node.path_cost + cost
                    )
                    frontier.append(child)
        
        return None
    
    def uniform_cost_search(self, problem: SearchProblem) -> Optional[List[str]]:
        """
        Поиск с равномерной стоимостью (UCS)
        
        Гарантирует оптимальность для взвешенных графов
        Эквивалентен алгоритму Дейкстры
        """
        frontier = [SearchNode(problem.get_initial_state())]
        explored = set()
        
        while frontier:
            node = heapq.heappop(frontier)
            
            if problem.is_goal(node.state):
                return self._get_path(node)
            
            if node.state in explored:
                continue
                
            explored.add(node.state)
            
            for next_state, action, cost in problem.get_successors(node.state):
                if next_state not in explored:
                    child = SearchNode(
                        state=next_state,
                        parent=node,
                        action=action,
                        path_cost=node.path_cost + cost
                    )
                    heapq.heappush(frontier, child)
        
        return None
    
    def a_star_search(self, problem: SearchProblem) -> Optional[List[str]]:
        """
        A* поиск
        
        Оптимален если эвристика допустимая (не переоценивает)
        f(n) = g(n) + h(n)
        """
        frontier = [SearchNode(
            state=problem.get_initial_state(),
            heuristic_cost=problem.heuristic(problem.get_initial_state())
        )]
        explored = set()
        
        while frontier:
            node = heapq.heappop(frontier)
            
            if problem.is_goal(node.state):
                return self._get_path(node)
            
            if node.state in explored:
                continue
                
            explored.add(node.state)
            
            for next_state, action, cost in problem.get_successors(node.state):
                if next_state not in explored:
                    child = SearchNode(
                        state=next_state,
                        parent=node,
                        action=action,
                        path_cost=node.path_cost + cost,
                        heuristic_cost=problem.heuristic(next_state)
                    )
                    heapq.heappush(frontier, child)
        
        return None
    
    def _get_path(self, node: SearchNode) -> List[str]:
        """Восстановление пути от узла до корня"""
        path = []
        current = node
        
        while current.parent is not None:
            path.append(current.action)
            current = current.parent
        
        return path[::-1]

# ==================== ЗАДАЧА N-ГОЛОВОЛОМКИ ====================
class NPuzzleProblem(SearchProblem):
    """
    Задача N-головоломки (например, 8-головоломка)
    """
    
    def __init__(self, initial_state: tuple, goal_state: tuple, size: int = 3):
        self.initial_state = initial_state
        self.goal_state = goal_state
        self.size = size
    
    def get_initial_state(self) -> tuple:
        return self.initial_state
    
    def is_goal(self, state: tuple) -> bool:
        return state == self.goal_state
    
    def get_successors(self, state: tuple) -> List[Tuple[tuple, str, float]]:
        """
        Генерация следующих состояний
        """
        successors = []
        state_list = list(state)
        
        # Находим позицию пустой клетки (0)
        empty_pos = state_list.index(0)
        row, col = empty_pos // self.size, empty_pos % self.size
        
        # Возможные движения
        moves = [
            (-1, 0, 'UP'),
            (1, 0, 'DOWN'),
            (0, -1, 'LEFT'),
            (0, 1, 'RIGHT')
        ]
        
        for dr, dc, action in moves:
            new_row, new_col = row + dr, col + dc
            
            # Проверка границ
            if 0 <= new_row < self.size and 0 <= new_col < self.size:
                new_empty_pos = new_row * self.size + new_col
                
                # Обмен местами
                new_state = state_list.copy()
                new_state[empty_pos], new_state[new_empty_pos] = new_state[new_empty_pos], new_state[empty_pos]
                
                successors.append((tuple(new_state), action, 1))
        
        return successors
    
    def heuristic(self, state: tuple) -> float:
        """
        Манхэттенское расстояние как эвристика
        
        Сумма расстояний каждой клетки до её целевой позиции
        """
        distance = 0
        
        for i, value in enumerate(state):
            if value != 0:  # Пропускаем пустую клетку
                # Текущая позиция
                current_row, current_col = i // self.size, i % self.size
                
                # Целевая позиция
                target_pos = self.goal_state.index(value)
                target_row, target_col = target_pos // self.size, target_pos % self.size
                
                # Манхэттенское расстояние
                distance += abs(current_row - target_row) + abs(current_col - target_col)
        
        return distance

# ==================== ГЕНЕТИЧЕСКИЕ АЛГОРИТМЫ ====================
class GeneticAlgorithm:
    """
    Генетический алгоритм для оптимизации
    """
    
    def __init__(self, population_size: int = 100, mutation_rate: float = 0.1,
                 crossover_rate: float = 0.7, elitism: int = 2):
        self.population_size = population_size
        self.mutation_rate = mutation_rate
        self.crossover_rate = crossover_rate
        self.elitism = elitism
    
    def create_individual(self, gene_length: int) -> List[int]:
        """
        Создание случайной особи
        """
        return [np.random.randint(0, 2) for _ in range(gene_length)]
    
    def create_population(self, gene_length: int) -> List[List[int]]:
        """
        Создание начальной популяции
        """
        return [self.create_individual(gene_length) for _ in range(self.population_size)]
    
    def fitness_function(self, individual: List[int]) -> float:
        """
        Функция приспособленности (переопределяется для конкретной задачи)
        
        Пример: OneMax problem - максимизация количества единиц
        """
        return sum(individual)
    
    def selection(self, population: List[List[int]], fitness_scores: List[float]) -> List[List[int]]:
        """
        Турнирная селекция
        
        Выбирает лучших особей для размножения
        """
        selected = []
        
        for _ in range(self.population_size):
            # Турнир между двумя случайными особями
            i, j = np.random.choice(len(population), 2, replace=False)
            
            if fitness_scores[i] > fitness_scores[j]:
                selected.append(population[i].copy())
            else:
                selected.append(population[j].copy())
        
        return selected
    
    def crossover(self, parent1: List[int], parent2: List[int]) -> Tuple[List[int], List[int]]:
        """
        Одноточечное скрещивание
        """
        if np.random.random() < self.crossover_rate:
            crossover_point = np.random.randint(1, len(parent1))
            
            offspring1 = parent1[:crossover_point] + parent2[crossover_point:]
            offspring2 = parent2[:crossover_point] + parent1[crossover_point:]
            
            return offspring1, offspring2
        else:
            return parent1.copy(), parent2.copy()
    
    def mutate(self, individual: List[int]) -> List[int]:
        """
        Мутация с заданной вероятностью
        """
        mutated = individual.copy()
        
        for i in range(len(mutated)):
            if np.random.random() < self.mutation_rate:
                mutated[i] = 1 - mutated[i]  # Битовая инверсия
        
        return mutated
    
    def evolve(self, gene_length: int, max_generations: int = 1000) -> Tuple[List[int], List[float]]:
        """
        Основной цикл эволюции
        """
        # Создание начальной популяции
        population = self.create_population(gene_length)
        fitness_history = []
        
        for generation in range(max_generations):
            # Оценка приспособленности
            fitness_scores = [self.fitness_function(ind) for ind in population]
            
            # Сохранение лучшего результата
            best_fitness = max(fitness_scores)
            fitness_history.append(best_fitness)
            
            # Элитизм - сохранение лучших особей
            elite_indices = np.argsort(fitness_scores)[-self.elitism:]
            elite = [population[i].copy() for i in elite_indices]
            
            # Селекция
            selected = self.selection(population, fitness_scores)
            
            # Создание нового поколения
            new_population = elite.copy()
            
            while len(new_population) < self.population_size:
                # Выбор родителей
                parent1, parent2 = np.random.choice(len(selected), 2, replace=False)
                
                # Скрещивание
                offspring1, offspring2 = self.crossover(selected[parent1], selected[parent2])
                
                # Мутация
                offspring1 = self.mutate(offspring1)
                offspring2 = self.mutate(offspring2)
                
                new_population.extend([offspring1, offspring2])
            
            # Обрезка до нужного размера
            population = new_population[:self.population_size]
            
            # Проверка на сходимость
            if best_fitness == gene_length:  # Для OneMax problem
                break
        
        # Возвращение лучшей особи
        final_fitness = [self.fitness_function(ind) for ind in population]
        best_individual = population[np.argmax(final_fitness)]
        
        return best_individual, fitness_history

# ==================== PARTICLE SWARM OPTIMIZATION ====================
class ParticleSwarmOptimization:
    """
    Алгоритм роя частиц для непрерывной оптимизации
    """
    
    def __init__(self, num_particles: int = 30, max_iterations: int = 1000,
                 w: float = 0.5, c1: float = 1.5, c2: float = 1.5):
        self.num_particles = num_particles
        self.max_iterations = max_iterations
        self.w = w    # Инерция
        self.c1 = c1  # Когнитивный параметр
        self.c2 = c2  # Социальный параметр
    
    def objective_function(self, x: np.ndarray) -> float:
        """
        Целевая функция (переопределяется)
        
        Пример: функция Сферы f(x) = Σ(x_i²)
        """
        return np.sum(x**2)
    
    def optimize(self, dimensions: int, bounds: Tuple[float, float]) -> Tuple[np.ndarray, float]:
        """
        Оптимизация с использованием PSO
        """
        lower_bound, upper_bound = bounds
        
        # Инициализация частиц
        positions = np.random.uniform(lower_bound, upper_bound, (self.num_particles, dimensions))
        velocities = np.random.uniform(-1, 1, (self.num_particles, dimensions))
        
        # Лучшие позиции частиц
        personal_best_positions = positions.copy()
        personal_best_scores = np.array([self.objective_function(pos) for pos in positions])
        
        # Глобальный лучший результат
        global_best_index = np.argmin(personal_best_scores)
        global_best_position = personal_best_positions[global_best_index].copy()
        global_best_score = personal_best_scores[global_best_index]
        
        # История оптимизации
        history = []
        
        for iteration in range(self.max_iterations):
            for i in range(self.num_particles):
                # Обновление скорости
                r1, r2 = np.random.random(), np.random.random()
                
                velocity = (self.w * velocities[i] + 
                           self.c1 * r1 * (personal_best_positions[i] - positions[i]) +
                           self.c2 * r2 * (global_best_position - positions[i]))
                
                # Обновление позиции
                position = positions[i] + velocity
                
                # Ограничение по границам
                position = np.clip(position, lower_bound, upper_bound)
                
                # Оценка новой позиции
                score = self.objective_function(position)
                
                # Обновление лучшей личной позиции
                if score < personal_best_scores[i]:
                    personal_best_positions[i] = position.copy()
                    personal_best_scores[i] = score
                    
                    # Обновление глобального лучшего
                    if score < global_best_score:
                        global_best_position = position.copy()
                        global_best_score = score
                
                # Сохранение обновлений
                velocities[i] = velocity
                positions[i] = position
            
            history.append(global_best_score)
            
            # Логирование прогресса
            if iteration % 100 == 0:
                print(f"Iteration {iteration}: Best score = {global_best_score:.6f}")
        
        return global_best_position, global_best_score

# ==================== СИМУЛЯЦИЯ ОТЖИГА ====================
class SimulatedAnnealing:
    """
    Алгоритм симуляции отжига
    """
    
    def __init__(self, initial_temp: float = 1000, cooling_rate: float = 0.95,
                 min_temp: float = 1e-8, max_iterations: int = 10000):
        self.initial_temp = initial_temp
        self.cooling_rate = cooling_rate
        self.min_temp = min_temp
        self.max_iterations = max_iterations
    
    def objective_function(self, x: np.ndarray) -> float:
        """
        Целевая функция для минимизации
        """
        return np.sum(x**2)
    
    def get_neighbor(self, current: np.ndarray, step_size: float = 0.1) -> np.ndarray:
        """
        Генерация соседнего решения
        """
        neighbor = current + np.random.normal(0, step_size, len(current))
        return neighbor
    
    def acceptance_probability(self, current_energy: float, new_energy: float, 
                             temperature: float) -> float:
        """
        Вероятность принятия худшего решения
        
        P = exp(-(E_new - E_current) / T)
        """
        if new_energy < current_energy:
            return 1.0
        else:
            return np.exp(-(new_energy - current_energy) / temperature)
    
    def optimize(self, dimensions: int, bounds: Tuple[float, float]) -> Tuple[np.ndarray, float]:
        """
        Оптимизация методом симуляции отжига
        """
        lower_bound, upper_bound = bounds
        
        # Начальное решение
        current_solution = np.random.uniform(lower_bound, upper_bound, dimensions)
        current_energy = self.objective_function(current_solution)
        
        # Лучшее решение
        best_solution = current_solution.copy()
        best_energy = current_energy
        
        # История оптимизации
        temperature = self.initial_temp
        history = []
        
        for iteration in range(self.max_iterations):
            # Генерация соседнего решения
            neighbor = self.get_neighbor(current_solution)
            
            # Ограничение по границам
            neighbor = np.clip(neighbor, lower_bound, upper_bound)
            
            # Оценка нового решения
            neighbor_energy = self.objective_function(neighbor)
            
            # Решение о принятии
            if (neighbor_energy < current_energy or 
                np.random.random() < self.acceptance_probability(current_energy, neighbor_energy, temperature)):
                
                current_solution = neighbor
                current_energy = neighbor_energy
                
                # Обновление лучшего решения
                if current_energy < best_energy:
                    best_solution = current_solution.copy()
                    best_energy = current_energy
            
            # Охлаждение
            temperature *= self.cooling_rate
            
            # Остановка при низкой температуре
            if temperature < self.min_temp:
                break
            
            history.append(best_energy)
        
        return best_solution, best_energy

# ==================== REINFORCEMENT LEARNING ====================
class QLearningAgent:
    """
    Q-Learning агент для обучения с подкреплением
    """
    
    def __init__(self, num_states: int, num_actions: int, learning_rate: float = 0.1,
                 discount_factor: float = 0.95, epsilon: float = 0.1):
        self.num_states = num_states
        self.num_actions = num_actions
        self.learning_rate = learning_rate
        self.discount_factor = discount_factor
        self.epsilon = epsilon
        
        # Q-таблица
        self.q_table = np.zeros((num_states, num_actions))
    
    def choose_action(self, state: int) -> int:
        """
        Выбор действия с epsilon-greedy стратегией
        """
        if np.random.random() < self.epsilon:
            # Исследование: случайное действие
            return np.random.randint(self.num_actions)
        else:
            # Эксплуатация: лучшее действие
            return np.argmax(self.q_table[state])
    
    def update_q_value(self, state: int, action: int, reward: float, next_state: int):
        """
        Обновление Q-значения
        
        Q(s,a) = Q(s,a) + α * (r + γ * max(Q(s',a')) - Q(s,a))
        """
        best_next_action = np.argmax(self.q_table[next_state])
        td_target = reward + self.discount_factor * self.q_table[next_state][best_next_action]
        td_error = td_target - self.q_table[state][action]
        
        self.q_table[state][action] += self.learning_rate * td_error
    
    def train(self, environment, num_episodes: int = 1000):
        """
        Обучение агента в среде
        """
        episode_rewards = []
        
        for episode in range(num_episodes):
            state = environment.reset()
            total_reward = 0
            done = False
            
            while not done:
                action = self.choose_action(state)
                next_state, reward, done = environment.step(action)
                
                self.update_q_value(state, action, reward, next_state)
                
                state = next_state
                total_reward += reward
            
            episode_rewards.append(total_reward)
            
            # Уменьшение epsilon (исследование -> эксплуатация)
            self.epsilon = max(0.01, self.epsilon * 0.995)
        
        return episode_rewards

# ==================== ПРОСТАЯ СРЕДА ДЛЯ RL ====================
class GridWorld:
    """
    Простая сетка для демонстрации RL
    """
    
    def __init__(self, size: int = 5, goal_position: Tuple[int, int] = (4, 4)):
        self.size = size
        self.goal_position = goal_position
        self.current_position = (0, 0)
        self.actions = [(0, 1), (0, -1), (1, 0), (-1, 0)]  # right, left, down, up
    
    def reset(self) -> int:
        """Сброс среды"""
        self.current_position = (0, 0)
        return self.position_to_state(self.current_position)
    
    def position_to_state(self, position: Tuple[int, int]) -> int:
        """Преобразование позиции в номер состояния"""
        return position[0] * self.size + position[1]
    
    def state_to_position(self, state: int) -> Tuple[int, int]:
        """Преобразование состояния в позицию"""
        return (state // self.size, state % self.size)
    
    def step(self, action: int) -> Tuple[int, float, bool]:
        """
        Выполнение действия
        
        Возвращает: (next_state, reward, done)
        """
        dx, dy = self.actions[action]
        x, y = self.current_position
        
        new_x = max(0, min(self.size - 1, x + dx))
        new_y = max(0, min(self.size - 1, y + dy))
        
        self.current_position = (new_x, new_y)
        
        # Награда
        if self.current_position == self.goal_position:
            reward = 10
            done = True
        else:
            reward = -0.1  # Небольшой штраф за каждый шаг
            done = False
        
        next_state = self.position_to_state(self.current_position)
        return next_state, reward, done

# ==================== ПРАКТИЧЕСКИЕ ПРИМЕРЫ ====================
def ai_algorithms_examples():
    """
    Примеры использования AI алгоритмов
    """
    print("=== Примеры AI алгоритмов ===")
    
    # Пример 1: Решение 8-головоломки
    print("\n1. Решение 8-головоломки с A*")
    
    initial_state = (1, 2, 3, 4, 5, 6, 7, 8, 0)
    goal_state = (1, 2, 3, 4, 5, 6, 0, 7, 8)
    
    puzzle = NPuzzleProblem(initial_state, goal_state)
    search = AISearch()
    
    solution = search.a_star_search(puzzle)
    print(f"Решение найдено: {solution}")
    print(f"Количество шагов: {len(solution) if solution else 0}")
    
    # Пример 2: Генетический алгоритм
    print("\n2. Генетический алгоритм (OneMax)")
    
    ga = GeneticAlgorithm(population_size=50, mutation_rate=0.1)
    best_individual, fitness_history = ga.evolve(gene_length=20, max_generations=100)
    
    print(f"Лучшая особь: {best_individual}")
    print(f"Приспособленность: {ga.fitness_function(best_individual)}")
    print(f"Поколений: {len(fitness_history)}")
    
    # Пример 3: Particle Swarm Optimization
    print("\n3. Particle Swarm Optimization")
    
    pso = ParticleSwarmOptimization(num_particles=20, max_iterations=200)
    best_position, best_score = pso.optimize(dimensions=2, bounds=(-10, 10))
    
    print(f"Лучшая позиция: {best_position}")
    print(f"Лучший результат: {best_score}")
    
    # Пример 4: Q-Learning
    print("\n4. Q-Learning в GridWorld")
    
    env = GridWorld(size=5, goal_position=(4, 4))
    agent = QLearningAgent(num_states=25, num_actions=4)
    
    rewards = agent.train(env, num_episodes=500)
    
    print(f"Средняя награда за последние 100 эпизодов: {np.mean(rewards[-100:]):.2f}")
    print(f"Q-таблица (первые 5 состояний):")
    for i in range(5):
        print(f"Состояние {i}: {agent.q_table[i]}")
    
    return {
        'puzzle_solution': solution,
        'ga_fitness': fitness_history,
        'pso_score': best_score,
        'rl_rewards': rewards
    }

# ==================== СОВРЕМЕННЫЕ ПОДХОДЫ ====================
class ModernAIApproaches:
    """
    Современные подходы в AI
    """
    
    @staticmethod
    def few_shot_learning_concepts():
        """
        Концепции обучения с малым количеством примеров
        """
        return {
            "Meta-Learning": {
                "Идея": "Обучение тому, как учиться",
                "Методы": ["MAML", "Prototypical Networks", "Matching Networks"],
                "Применение": "Быстрая адаптация к новым задачам"
            },
            
            "Transfer Learning": {
                "Идея": "Перенос знаний между задачами",
                "Методы": ["Fine-tuning", "Feature extraction", "Domain adaptation"],
                "Применение": "Использование предобученных моделей"
            },
            
            "Data Augmentation": {
                "Идея": "Увеличение данных искусственно",
                "Методы": ["Rotation", "Scaling", "Synthetic data generation"],
                "Применение": "Улучшение обобщения при малых данных"
            }
        }
    
    @staticmethod
    def explainable_ai_methods():
        """
        Методы объяснимого ИИ
        """
        return {
            "Local Explanations": {
                "LIME": "Local Interpretable Model-agnostic Explanations",
                "SHAP": "SHapley Additive exPlanations",
                "Применение": "Объяснение отдельных предсказаний"
            },
            
            "Global Explanations": {
                "Feature Importance": "Важность признаков",
                "Partial Dependence": "Зависимость от признаков",
                "Применение": "Понимание модели в целом"
            },
            
            "Attention Visualization": {
                "Attention Maps": "Карты внимания",
                "Activation Maximization": "Максимизация активации",
                "Применение": "Понимание нейронных сетей"
            }
        }
    
    @staticmethod
    def ai_safety_considerations():
        """
        Соображения безопасности ИИ
        """
        return {
            "Alignment Problem": {
                "Описание": "Согласование целей ИИ с человеческими",
                "Методы": ["RLHF", "Constitutional AI", "Value Learning"],
                "Важность": "Критично для AGI"
            },
            
            "Robustness": {
                "Описание": "Устойчивость к атакам и ошибкам",
                "Методы": ["Adversarial Training", "Certified Defense", "Randomized Smoothing"],
                "Важность": "Безопасность в production"
            },
            
            "Fairness": {
                "Описание": "Справедливость и отсутствие дискриминации",
                "Методы": ["Bias Detection", "Fairness Constraints", "Demographic Parity"],
                "Важность": "Этичность применения"
            }
        }

# ==================== БУДУЩИЕ НАПРАВЛЕНИЯ ====================
future_ai_directions = {
    "Neuromorphic Computing": {
        "Описание": "Вычисления, имитирующие мозг",
        "Преимущества": "Энергоэффективность, параллелизм",
        "Примеры": "Intel Loihi, IBM TrueNorth"
    },
    
    "Quantum Machine Learning": {
        "Описание": "ML на квантовых компьютерах",
        "Преимущества": "Экспоненциальное ускорение",
        "Примеры": "Quantum SVM, Variational Quantum Eigensolver"
    },
    
    "Continual Learning": {
        "Описание": "Обучение без забывания",
        "Проблемы": "Catastrophic forgetting",
        "Методы": "Elastic Weight Consolidation, Progressive Networks"
    },
    
    "Multimodal AI": {
        "Описание": "Интеграция разных модальностей",
        "Примеры": "Vision-Language models, Audio-Visual learning",
        "Применения": "Робототехника, AR/VR"
    }
}

# ==================== ПРАКТИЧЕСКИЕ СОВЕТЫ ====================
ai_development_best_practices = {
    "Выбор алгоритма": [
        "Понимание природы задачи",
        "Размер и качество данных",
        "Вычислительные ограничения",
        "Требования к интерпретируемости"
    ],
    
    "Оптимизация": [
        "Правильная метрика оценки",
        "Регуляризация и валидация",
        "Гиперпараметры и их настройка",
        "Мониторинг обучения"
    ],
    
    "Продакшн": [
        "Мониторинг drift данных",
        "A/B тестирование",
        "Откат к предыдущим версиям",
        "Логирование и отладка"
    ],
    
    "Этика": [
        "Bias в данных и моделях",
        "Прозрачность решений",
        "Конфиденциальность данных",
        "Социальное воздействие"
    ]
}
```

---

*Искусственный интеллект - это не замена человеческого разума, а его усиление* 🤖🧠 