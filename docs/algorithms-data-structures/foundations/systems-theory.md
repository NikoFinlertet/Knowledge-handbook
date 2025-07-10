# Теория систем и формальные методы 🔧

## Содержание
1. [Системы управления](#системы-управления)
2. [Формальные методы](#формальные-методы)
3. [Теория игр](#теория-игр)
4. [Теория очередей](#теория-очередей)
5. [Системы реального времени](#системы-реального-времени)

---

## Системы управления

### Обратная связь и стабильность
```python
class ControlSystem:
    """Система управления с обратной связью"""
    
    def __init__(self, kp=1.0, ki=0.0, kd=0.0):
        self.kp = kp  # Пропорциональный коэффициент
        self.ki = ki  # Интегральный коэффициент
        self.kd = kd  # Дифференциальный коэффициент
        
        self.previous_error = 0
        self.integral = 0
    
    def pid_control(self, setpoint, current_value, dt=0.1):
        """PID регулятор"""
        error = setpoint - current_value
        
        # Пропорциональная составляющая
        proportional = self.kp * error
        
        # Интегральная составляющая
        self.integral += error * dt
        integral_term = self.ki * self.integral
        
        # Дифференциальная составляющая
        derivative = (error - self.previous_error) / dt
        derivative_term = self.kd * derivative
        
        # Управляющий сигнал
        output = proportional + integral_term + derivative_term
        
        self.previous_error = error
        return output
    
    def simulate_system(self, setpoint, steps=100, dt=0.1):
        """Симуляция системы"""
        current_value = 0
        values = []
        
        for _ in range(steps):
            control_signal = self.pid_control(setpoint, current_value, dt)
            
            # Простая модель системы (инерция первого порядка)
            current_value += control_signal * dt * 0.1
            values.append(current_value)
        
        return values
```

---

## Формальные методы

### Логика предикатов
```python
class PredicateLogic:
    """Логика предикатов"""
    
    @staticmethod
    def evaluate_formula(formula, variables):
        """Оценка формулы"""
        # Упрощенная версия для демонстрации
        return eval(formula, {"__builtins__": {}}, variables)
    
    @staticmethod
    def model_checker(states, transitions, property_formula):
        """Проверка модели"""
        # Упрощенная версия model checking
        satisfied_states = []
        
        for state in states:
            if PredicateLogic.evaluate_formula(property_formula, state):
                satisfied_states.append(state)
        
        return satisfied_states
    
    @staticmethod
    def temporal_logic_check(trace, ltl_formula):
        """Проверка формулы линейной темпоральной логики"""
        # Упрощенная версия LTL
        if "always" in ltl_formula:
            property_part = ltl_formula.replace("always", "").strip()
            return all(PredicateLogic.evaluate_formula(property_part, state) for state in trace)
        
        if "eventually" in ltl_formula:
            property_part = ltl_formula.replace("eventually", "").strip()
            return any(PredicateLogic.evaluate_formula(property_part, state) for state in trace)
        
        return False

class FormalSpecification:
    """Формальная спецификация"""
    
    def __init__(self):
        self.preconditions = []
        self.postconditions = []
        self.invariants = []
    
    def add_precondition(self, condition):
        """Добавить предусловие"""
        self.preconditions.append(condition)
    
    def add_postcondition(self, condition):
        """Добавить постусловие"""
        self.postconditions.append(condition)
    
    def add_invariant(self, condition):
        """Добавить инвариант"""
        self.invariants.append(condition)
    
    def verify(self, initial_state, final_state, trace):
        """Верификация спецификации"""
        # Проверка предусловий
        for pre in self.preconditions:
            if not PredicateLogic.evaluate_formula(pre, initial_state):
                return False, f"Предусловие не выполнено: {pre}"
        
        # Проверка постусловий
        for post in self.postconditions:
            if not PredicateLogic.evaluate_formula(post, final_state):
                return False, f"Постусловие не выполнено: {post}"
        
        # Проверка инвариантов
        for inv in self.invariants:
            for state in trace:
                if not PredicateLogic.evaluate_formula(inv, state):
                    return False, f"Инвариант нарушен: {inv}"
        
        return True, "Спецификация выполнена"
```

---

## Теория игр

### Игры с нулевой суммой
```python
class GameTheory:
    """Теория игр"""
    
    @staticmethod
    def minimax(payoff_matrix, maximizing_player=True):
        """Алгоритм минимакс"""
        if maximizing_player:
            return max(min(row) for row in payoff_matrix)
        else:
            return min(max(row) for row in payoff_matrix)
    
    @staticmethod
    def nash_equilibrium_2x2(payoff_matrix_a, payoff_matrix_b):
        """Равновесие Нэша для игры 2x2"""
        # Поиск чистых стратегий равновесия
        equilibria = []
        
        for i in range(2):
            for j in range(2):
                # Проверяем, является ли (i,j) равновесием Нэша
                best_response_a = max(range(2), key=lambda x: payoff_matrix_a[x][j])
                best_response_b = max(range(2), key=lambda x: payoff_matrix_b[i][x])
                
                if best_response_a == i and best_response_b == j:
                    equilibria.append((i, j))
        
        return equilibria
    
    @staticmethod
    def prisoner_dilemma():
        """Дилемма заключенного"""
        # Матрица выигрышей (сотрудничать, предать)
        payoff_matrix = [
            [(3, 3), (0, 5)],  # Сотрудничать
            [(5, 0), (1, 1)]   # Предать
        ]
        
        return payoff_matrix
    
    @staticmethod
    def auction_theory(valuations, reserve_price=0):
        """Теория аукционов (аукцион второй цены)"""
        # Сортируем ставки по убыванию
        sorted_bids = sorted(valuations, reverse=True)
        
        if len(sorted_bids) < 2 or sorted_bids[1] < reserve_price:
            return None, reserve_price
        
        # Победитель платит цену второй ставки
        winner_payment = max(sorted_bids[1], reserve_price)
        
        return sorted_bids[0], winner_payment
```

---

## Теория очередей

### Модели обслуживания
```python
import random
from collections import deque
import numpy as np

class QueueingSystem:
    """Система массового обслуживания"""
    
    def __init__(self, arrival_rate, service_rate, num_servers=1):
        self.arrival_rate = arrival_rate  # λ
        self.service_rate = service_rate  # μ
        self.num_servers = num_servers
        self.queue = deque()
        self.servers = [None] * num_servers
        self.current_time = 0
        
        # Статистика
        self.total_customers = 0
        self.total_wait_time = 0
        self.total_service_time = 0
    
    def mm1_metrics(self):
        """Аналитические метрики для M/M/1"""
        if self.num_servers != 1:
            raise ValueError("Только для M/M/1")
        
        rho = self.arrival_rate / self.service_rate
        
        if rho >= 1:
            return {"stable": False, "utilization": rho}
        
        # Основные метрики
        metrics = {
            "stable": True,
            "utilization": rho,
            "avg_customers": rho / (1 - rho),
            "avg_queue_length": rho**2 / (1 - rho),
            "avg_wait_time": rho / (self.service_rate * (1 - rho)),
            "avg_service_time": 1 / self.service_rate
        }
        
        return metrics
    
    def simulate(self, simulation_time):
        """Симуляция системы"""
        events = []
        
        # Генерируем события прибытия
        time = 0
        while time < simulation_time:
            # Экспоненциальное распределение для интервалов прибытия
            inter_arrival = random.expovariate(self.arrival_rate)
            time += inter_arrival
            if time < simulation_time:
                events.append(("arrival", time))
        
        # Сортируем события по времени
        events.sort(key=lambda x: x[1])
        
        # Обрабатываем события
        for event_type, event_time in events:
            self.current_time = event_time
            
            if event_type == "arrival":
                self.handle_arrival()
        
        return self.get_statistics()
    
    def handle_arrival(self):
        """Обработка прибытия клиента"""
        self.total_customers += 1
        
        # Ищем свободный сервер
        free_server = None
        for i, server in enumerate(self.servers):
            if server is None:
                free_server = i
                break
        
        if free_server is not None:
            # Обслуживаем немедленно
            service_time = random.expovariate(self.service_rate)
            self.servers[free_server] = self.current_time + service_time
            self.total_service_time += service_time
        else:
            # Добавляем в очередь
            self.queue.append(self.current_time)
    
    def get_statistics(self):
        """Получение статистики"""
        if self.total_customers == 0:
            return {"avg_wait_time": 0, "avg_service_time": 0}
        
        return {
            "total_customers": self.total_customers,
            "avg_wait_time": self.total_wait_time / self.total_customers,
            "avg_service_time": self.total_service_time / self.total_customers,
            "avg_queue_length": len(self.queue)
        }

class NetworkQueues:
    """Сети очередей"""
    
    def __init__(self):
        self.nodes = {}
        self.routing_probabilities = {}
    
    def add_node(self, node_id, arrival_rate, service_rate):
        """Добавить узел"""
        self.nodes[node_id] = QueueingSystem(arrival_rate, service_rate)
    
    def add_routing(self, from_node, to_node, probability):
        """Добавить маршрутизацию"""
        if from_node not in self.routing_probabilities:
            self.routing_probabilities[from_node] = {}
        self.routing_probabilities[from_node][to_node] = probability
    
    def solve_jackson_network(self):
        """Решение сети Джексона"""
        # Упрощенный алгоритм для демонстрации
        # В реальности требуется решение системы линейных уравнений
        
        effective_arrival_rates = {}
        for node_id in self.nodes:
            effective_arrival_rates[node_id] = self.nodes[node_id].arrival_rate
        
        # Итеративное решение
        for _ in range(100):  # Максимум итераций
            new_rates = {}
            
            for node_id in self.nodes:
                external_rate = self.nodes[node_id].arrival_rate
                internal_rate = 0
                
                for from_node in self.routing_probabilities:
                    if node_id in self.routing_probabilities[from_node]:
                        prob = self.routing_probabilities[from_node][node_id]
                        internal_rate += effective_arrival_rates[from_node] * prob
                
                new_rates[node_id] = external_rate + internal_rate
            
            # Проверка сходимости
            if all(abs(new_rates[node] - effective_arrival_rates[node]) < 1e-6 
                   for node in new_rates):
                break
            
            effective_arrival_rates = new_rates
        
        return effective_arrival_rates
```

---

## Системы реального времени

### Планирование задач
```python
class RealTimeSystem:
    """Система реального времени"""
    
    def __init__(self):
        self.tasks = []
    
    def add_task(self, task_id, period, execution_time, deadline=None):
        """Добавить задачу"""
        if deadline is None:
            deadline = period
        
        self.tasks.append({
            'id': task_id,
            'period': period,
            'execution_time': execution_time,
            'deadline': deadline,
            'utilization': execution_time / period
        })
    
    def rate_monotonic_analysis(self):
        """Анализ Rate Monotonic"""
        # Сортируем задачи по возрастанию периодов
        sorted_tasks = sorted(self.tasks, key=lambda x: x['period'])
        
        # Проверяем необходимое условие
        total_utilization = sum(task['utilization'] for task in self.tasks)
        n = len(self.tasks)
        
        # Граница планируемости для RM
        rm_bound = n * (2**(1/n) - 1)
        
        results = {
            'total_utilization': total_utilization,
            'rm_bound': rm_bound,
            'necessary_condition': total_utilization <= 1.0,
            'sufficient_condition': total_utilization <= rm_bound
        }
        
        # Точный тест планируемости
        schedulable = True
        for i, task in enumerate(sorted_tasks):
            response_time = self.calculate_response_time(task, sorted_tasks[:i+1])
            if response_time > task['deadline']:
                schedulable = False
                break
        
        results['exact_schedulability'] = schedulable
        return results
    
    def calculate_response_time(self, task, higher_priority_tasks):
        """Расчет времени отклика"""
        # Итеративный алгоритм
        response_time = task['execution_time']
        
        for _ in range(100):  # Максимум итераций
            interference = 0
            
            for hp_task in higher_priority_tasks:
                if hp_task['id'] != task['id']:
                    interference += np.ceil(response_time / hp_task['period']) * hp_task['execution_time']
            
            new_response_time = task['execution_time'] + interference
            
            if abs(new_response_time - response_time) < 1e-6:
                break
            
            response_time = new_response_time
        
        return response_time
    
    def earliest_deadline_first(self, simulation_time):
        """Планирование EDF"""
        # Генерируем все экземпляры задач
        job_instances = []
        
        for task in self.tasks:
            for release_time in range(0, simulation_time, task['period']):
                job_instances.append({
                    'task_id': task['id'],
                    'release_time': release_time,
                    'deadline': release_time + task['deadline'],
                    'execution_time': task['execution_time'],
                    'remaining_time': task['execution_time']
                })
        
        # Сортируем по deadline
        job_instances.sort(key=lambda x: x['deadline'])
        
        # Симулируем выполнение
        current_time = 0
        schedule = []
        
        while job_instances and current_time < simulation_time:
            # Находим задачу с ближайшим deadline
            ready_jobs = [job for job in job_instances if job['release_time'] <= current_time]
            
            if not ready_jobs:
                current_time += 1
                continue
            
            current_job = min(ready_jobs, key=lambda x: x['deadline'])
            
            # Выполняем задачу
            execution_slice = min(1, current_job['remaining_time'])
            schedule.append({
                'time': current_time,
                'task_id': current_job['task_id'],
                'execution_time': execution_slice
            })
            
            current_job['remaining_time'] -= execution_slice
            current_time += execution_slice
            
            # Удаляем завершенную задачу
            if current_job['remaining_time'] == 0:
                job_instances.remove(current_job)
        
        return schedule
```

## Практические применения

### Система мониторинга
```python
class MonitoringSystem:
    """Система мониторинга с формальными гарантиями"""
    
    def __init__(self):
        self.metrics = {}
        self.alerts = []
        self.sla_spec = FormalSpecification()
    
    def add_metric(self, name, value, timestamp):
        """Добавить метрику"""
        if name not in self.metrics:
            self.metrics[name] = []
        self.metrics[name].append((timestamp, value))
    
    def define_sla(self, availability_threshold=0.99, response_time_threshold=100):
        """Определить SLA"""
        self.sla_spec.add_invariant(f"availability >= {availability_threshold}")
        self.sla_spec.add_invariant(f"response_time <= {response_time_threshold}")
    
    def check_sla_compliance(self, time_window=3600):
        """Проверить соответствие SLA"""
        current_time = max(max(timestamps) for timestamps in self.metrics.values())
        
        # Вычисляем доступность
        if 'uptime' in self.metrics:
            recent_uptime = [v for t, v in self.metrics['uptime'] 
                           if current_time - t <= time_window]
            availability = sum(recent_uptime) / len(recent_uptime) if recent_uptime else 0
        else:
            availability = 1.0
        
        # Вычисляем время отклика
        if 'response_time' in self.metrics:
            recent_response_times = [v for t, v in self.metrics['response_time'] 
                                   if current_time - t <= time_window]
            avg_response_time = sum(recent_response_times) / len(recent_response_times) if recent_response_times else 0
        else:
            avg_response_time = 0
        
        # Проверяем соответствие
        state = {'availability': availability, 'response_time': avg_response_time}
        compliance, message = self.sla_spec.verify(state, state, [state])
        
        return compliance, message, state
```

## Связь с другими разделами

- **[Алгоритмы](./algorithms.md)** - Алгоритмы планирования и оптимизации
- **[Computer Science](./computer-science-fundamentals.md)** - Системы и их свойства
- **[Математические основы](./mathematics-foundations.md)** - Теория вероятностей для моделирования
- **[AI/ML](./machine-learning-ai.md)** - Теория игр в ML

---

*Следующий раздел: [Заключение →](./README.md)* 