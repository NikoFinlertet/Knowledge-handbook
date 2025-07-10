# Описательная статистика 📈

> [[probability-theory|← Основы теории вероятностей]] | [[README|Оглавление]] | [[statistical-tests|Статистические тесты →]]

## 🎯 Что изучим

- **Меры центральной тенденции** - среднее, медиана, мода
- **Меры разброса** - дисперсия, стандартное отклонение, квартили
- **Корреляция и регрессия** - связи между переменными

---

## 1. Меры центральной тенденции

```python
import statistics
import math

class StatisticalAnalyzer:
    def __init__(self, data):
        self.data = data
    
    def mean(self):
        """Среднее арифметическое"""
        return sum(self.data) / len(self.data)
    
    def median(self):
        """Медиана"""
        sorted_data = sorted(self.data)
        n = len(sorted_data)
        if n % 2 == 0:
            return (sorted_data[n//2 - 1] + sorted_data[n//2]) / 2
        else:
            return sorted_data[n//2]
    
    def mode(self):
        """Мода"""
        frequency = {}
        for value in self.data:
            frequency[value] = frequency.get(value, 0) + 1
        max_freq = max(frequency.values())
        return [value for value, freq in frequency.items() if freq == max_freq]
    
    def variance(self):
        """Дисперсия"""
        mean = self.mean()
        return sum((x - mean) ** 2 for x in self.data) / len(self.data)
    
    def standard_deviation(self):
        """Стандартное отклонение"""
        return math.sqrt(self.variance())
    
    def quartiles(self):
        """Квартили"""
        sorted_data = sorted(self.data)
        n = len(sorted_data)
        
        q1_index = n // 4
        q2_index = n // 2
        q3_index = 3 * n // 4
        
        return {
            'Q1': sorted_data[q1_index],
            'Q2': sorted_data[q2_index],
            'Q3': sorted_data[q3_index]
        }
    
    def outliers(self):
        """Выбросы по правилу IQR"""
        quartiles = self.quartiles()
        iqr = quartiles['Q3'] - quartiles['Q1']
        lower_bound = quartiles['Q1'] - 1.5 * iqr
        upper_bound = quartiles['Q3'] + 1.5 * iqr
        
        return [x for x in self.data if x < lower_bound or x > upper_bound]

# Пример: анализ производительности базы данных
query_times = [15, 23, 18, 45, 12, 28, 19, 34, 16, 25, 180, 21, 17, 29, 22]
analyzer = StatisticalAnalyzer(query_times)

print("Анализ времени выполнения запросов к БД:")
print(f"Среднее время: {analyzer.mean():.2f} мс")
print(f"Медиана: {analyzer.median():.2f} мс")
print(f"Мода: {analyzer.mode()}")
print(f"Стандартное отклонение: {analyzer.standard_deviation():.2f} мс")
print(f"Квартили: {analyzer.quartiles()}")
print(f"Выбросы: {analyzer.outliers()}")
```

### 🎯 Когда использовать какую меру

| Мера | Когда использовать | Устойчивость к выбросам |
|------|-------------------|------------------------|
| **Среднее** | Нормальные данные | Низкая |
| **Медиана** | Есть выбросы | Высокая |
| **Мода** | Категориальные данные | Средняя |

---

## 2. Корреляция и регрессия

```python
class CorrelationAnalyzer:
    def __init__(self, x_data, y_data):
        self.x_data = x_data
        self.y_data = y_data
    
    def correlation_coefficient(self):
        """Коэффициент корреляции Пирсона"""
        n = len(self.x_data)
        mean_x = sum(self.x_data) / n
        mean_y = sum(self.y_data) / n
        
        numerator = sum((x - mean_x) * (y - mean_y) for x, y in zip(self.x_data, self.y_data))
        denominator_x = sum((x - mean_x) ** 2 for x in self.x_data)
        denominator_y = sum((y - mean_y) ** 2 for y in self.y_data)
        
        return numerator / math.sqrt(denominator_x * denominator_y)
    
    def linear_regression(self):
        """Линейная регрессия y = ax + b"""
        n = len(self.x_data)
        mean_x = sum(self.x_data) / n
        mean_y = sum(self.y_data) / n
        
        # Вычисление коэффициентов
        numerator = sum((x - mean_x) * (y - mean_y) for x, y in zip(self.x_data, self.y_data))
        denominator = sum((x - mean_x) ** 2 for x in self.x_data)
        
        a = numerator / denominator
        b = mean_y - a * mean_x
        
        return a, b
    
    def r_squared(self):
        """Коэффициент детерминации"""
        return self.correlation_coefficient() ** 2

# Пример: связь между размером команды и производительностью
team_sizes = [2, 3, 4, 5, 6, 7, 8, 9, 10, 12]
features_per_sprint = [8, 12, 15, 18, 20, 19, 17, 15, 12, 10]

analyzer = CorrelationAnalyzer(team_sizes, features_per_sprint)
correlation = analyzer.correlation_coefficient()
a, b = analyzer.linear_regression()
r_squared = analyzer.r_squared()

print(f"Корреляция размера команды и производительности: {correlation:.3f}")
print(f"Уравнение регрессии: y = {a:.2f}x + {b:.2f}")
print(f"R²: {r_squared:.3f}")

# Прогнозирование
def predict_performance(team_size):
    return a * team_size + b

print(f"Прогноз для команды из 11 человек: {predict_performance(11):.1f} фич за спринт")
```

### 📊 Интерпретация корреляции

| Значение r | Сила связи | Интерпретация |
|------------|------------|---------------|
| **0.0 - 0.3** | Слабая | Почти нет связи |
| **0.3 - 0.7** | Средняя | Умеренная связь |
| **0.7 - 1.0** | Сильная | Сильная связь |

---

## 3. Практические применения

### Анализ производительности

```python
class PerformanceDashboard:
    def __init__(self):
        self.metrics = {
            'response_times': [],
            'memory_usage': [],
            'cpu_usage': [],
            'error_rates': []
        }
    
    def add_measurements(self, response_time, memory, cpu, errors):
        """Добавление новых измерений"""
        self.metrics['response_times'].append(response_time)
        self.metrics['memory_usage'].append(memory)
        self.metrics['cpu_usage'].append(cpu)
        self.metrics['error_rates'].append(errors)
    
    def generate_report(self):
        """Генерация отчёта"""
        report = {}
        
        for metric_name, values in self.metrics.items():
            if values:
                analyzer = StatisticalAnalyzer(values)
                report[metric_name] = {
                    'mean': analyzer.mean(),
                    'median': analyzer.median(),
                    'std_dev': analyzer.standard_deviation(),
                    'outliers_count': len(analyzer.outliers()),
                    'percentile_95': analyzer.quartiles()['Q3']  # Приближение
                }
        
        return report
    
    def detect_correlations(self):
        """Поиск корреляций между метриками"""
        correlations = {}
        metric_names = list(self.metrics.keys())
        
        for i in range(len(metric_names)):
            for j in range(i + 1, len(metric_names)):
                metric1 = metric_names[i]
                metric2 = metric_names[j]
                
                if self.metrics[metric1] and self.metrics[metric2]:
                    analyzer = CorrelationAnalyzer(
                        self.metrics[metric1], 
                        self.metrics[metric2]
                    )
                    corr = analyzer.correlation_coefficient()
                    correlations[f"{metric1}_vs_{metric2}"] = corr
        
        return correlations

# Пример использования
dashboard = PerformanceDashboard()

# Добавление данных за несколько дней
sample_data = [
    (120, 85, 45, 0.02),
    (135, 88, 52, 0.03),
    (110, 82, 38, 0.01),
    (180, 95, 65, 0.05),
    (125, 87, 48, 0.02)
]

for data in sample_data:
    dashboard.add_measurements(*data)

# Генерация отчёта
report = dashboard.generate_report()
print("Отчёт о производительности:")
for metric, stats in report.items():
    print(f"\n{metric}:")
    print(f"  Среднее: {stats['mean']:.2f}")
    print(f"  Медиана: {stats['median']:.2f}")
    print(f"  Выбросов: {stats['outliers_count']}")

# Анализ корреляций
correlations = dashboard.detect_correlations()
print("\nКорреляции между метриками:")
for pair, corr in correlations.items():
    print(f"{pair}: {corr:.3f}")
```

---

## 📚 Связанные темы

- [[probability-theory|Основы теории вероятностей]] - фундамент статистики
- [[statistical-tests|Статистические тесты]] - проверка гипотез
- [[time-series|Временные ряды]] - анализ динамики

---

## 🛠️ Инструменты

### Python библиотеки
```python
import statistics       # Встроенная статистика
import pandas as pd    # Анализ данных
import numpy as np     # Численные вычисления
import matplotlib.pyplot as plt  # Визуализация
import seaborn as sns  # Статистическая визуализация
```

### Практические задания

1. **Анализ логов** - проанализируйте статистики времени ответа API
2. **Correlation Analysis** - найдите связи между метриками системы
3. **Outlier Detection** - создайте систему обнаружения аномальных значений

---

## 🎯 Ключевые концепции

- **Центральная тенденция** - типичные значения в данных
- **Разброс данных** - насколько данные отличаются от среднего
- **Корреляция** - сила и направление связи между переменными
- **Выбросы** - нетипичные значения, требующие внимания 