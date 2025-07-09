# Статистика и анализ данных 📊

> **Навигация**: [[number-theory-cryptography|← Теория чисел]] | [[mathematics-algorithms|Главная]] | [[combinatorics-generation|Комбинаторика →]]

## Содержание
1. [Дескриптивная статистика](#📊-дескриптивная-статистика)
2. [Статистические тесты](#🧪-статистические-тесты)
3. [Корреляция и регрессия](#📈-корреляция-и-регрессия)
4. [Практические применения](#🚀-практические-применения)

---

## 📊 Дескриптивная статистика

**Математическая основа:** Меры центральной тенденции, меры разброса, распределения

```python
import math
import numpy as np
from collections import Counter

class DescriptiveStatistics:
    """Дескриптивная статистика и анализ данных"""
    
    @staticmethod
    def mean(data):
        """Среднее арифметическое"""
        return sum(data) / len(data)
    
    @staticmethod
    def median(data):
        """Медиана - значение, разделяющее выборку пополам"""
        sorted_data = sorted(data)
        n = len(sorted_data)
        
        if n % 2 == 0:
            return (sorted_data[n//2 - 1] + sorted_data[n//2]) / 2
        else:
            return sorted_data[n//2]
    
    @staticmethod
    def mode(data):
        """Мода - наиболее часто встречающееся значение"""
        counter = Counter(data)
        max_count = max(counter.values())
        return [value for value, count in counter.items() if count == max_count]
    
    @staticmethod
    def variance(data, ddof=0):
        """
        Дисперсия
        ddof=0 для генеральной совокупности
        ddof=1 для выборки
        """
        mean_val = DescriptiveStatistics.mean(data)
        squared_deviations = [(x - mean_val)**2 for x in data]
        return sum(squared_deviations) / (len(data) - ddof)
    
    @staticmethod
    def standard_deviation(data, ddof=0):
        """Стандартное отклонение"""
        return math.sqrt(DescriptiveStatistics.variance(data, ddof))
    
    @staticmethod
    def quantile(data, q):
        """Квантиль уровня q (0 ≤ q ≤ 1)"""
        sorted_data = sorted(data)
        n = len(sorted_data)
        index = q * (n - 1)
        
        if index.is_integer():
            return sorted_data[int(index)]
        else:
            lower = sorted_data[int(index)]
            upper = sorted_data[int(index) + 1]
            return lower + (index - int(index)) * (upper - lower)
    
    @staticmethod
    def five_number_summary(data):
        """Пятичисловое резюме: мин, Q1, медиана, Q3, макс"""
        return {
            'min': min(data),
            'Q1': DescriptiveStatistics.quantile(data, 0.25),
            'median': DescriptiveStatistics.median(data),
            'Q3': DescriptiveStatistics.quantile(data, 0.75),
            'max': max(data)
        }
    
    @staticmethod
    def correlation_coefficient(x, y):
        """Коэффициент корреляции Пирсона"""
        n = len(x)
        if n != len(y):
            raise ValueError("Векторы должны быть одинаковой длины")
        
        sum_x = sum(x)
        sum_y = sum(y)
        sum_xy = sum(x[i] * y[i] for i in range(n))
        sum_x2 = sum(x[i]**2 for i in range(n))
        sum_y2 = sum(y[i]**2 for i in range(n))
        
        numerator = n * sum_xy - sum_x * sum_y
        denominator = math.sqrt((n * sum_x2 - sum_x**2) * (n * sum_y2 - sum_y**2))
        
        if denominator == 0:
            return 0
        
        return numerator / denominator

class OnlineStatistics:
    """Онлайн-алгоритмы для потоковых данных"""
    
    def __init__(self):
        self.count = 0
        self.mean = 0
        self.m2 = 0  # Для вычисления дисперсии
        self.min_val = float('inf')
        self.max_val = float('-inf')
    
    def update(self, value):
        """
        Обновление статистики новым значением
        Алгоритм Уэлфорда для онлайн-вычисления среднего и дисперсии
        """
        self.count += 1
        
        # Обновление минимума и максимума
        self.min_val = min(self.min_val, value)
        self.max_val = max(self.max_val, value)
        
        # Обновление среднего и дисперсии
        delta = value - self.mean
        self.mean += delta / self.count
        delta2 = value - self.mean
        self.m2 += delta * delta2
    
    def get_mean(self):
        """Текущее среднее"""
        return self.mean
    
    def get_variance(self, ddof=0):
        """Текущая дисперсия"""
        if self.count < 2:
            return 0
        return self.m2 / (self.count - ddof)
    
    def get_std(self, ddof=0):
        """Текущее стандартное отклонение"""
        return math.sqrt(self.get_variance(ddof))
    
    def get_range(self):
        """Размах данных"""
        return self.max_val - self.min_val
```

---

## 🧪 Статистические тесты

```python
class StatisticalTests:
    """Статистические тесты и проверка гипотез"""
    
    @staticmethod
    def t_test_one_sample(data, population_mean, alpha=0.05):
        """
        Одновыборочный t-тест
        Проверяет H0: μ = population_mean против H1: μ ≠ population_mean
        """
        n = len(data)
        sample_mean = DescriptiveStatistics.mean(data)
        sample_std = DescriptiveStatistics.standard_deviation(data, ddof=1)
        
        # t-статистика
        t_statistic = (sample_mean - population_mean) / (sample_std / math.sqrt(n))
        
        # Степени свободы
        df = n - 1
        
        # Критическое значение (приближенное для больших выборок)
        t_critical = 1.96 if n > 30 else 2.0  # Упрощенное приближение
        
        p_value = 2 * (1 - StatisticalTests._t_cdf(abs(t_statistic), df))
        
        return {
            't_statistic': t_statistic,
            'p_value': p_value,
            'critical_value': t_critical,
            'reject_null': abs(t_statistic) > t_critical or p_value < alpha,
            'confidence_interval': StatisticalTests._confidence_interval(
                sample_mean, sample_std, n, alpha
            )
        }
    
    @staticmethod
    def _t_cdf(t, df):
        """Приближенная CDF для t-распределения"""
        # Упрощенная аппроксимация для демонстрации
        if df > 30:
            # Для больших df t-распределение близко к нормальному
            return StatisticalTests._normal_cdf(t)
        else:
            # Очень грубая аппроксимация
            return 0.5 + 0.5 * math.tanh(t / 2)
    
    @staticmethod
    def _normal_cdf(x):
        """Приближенная CDF для стандартного нормального распределения"""
        return 0.5 * (1 + math.erf(x / math.sqrt(2)))
    
    @staticmethod
    def _confidence_interval(mean, std, n, alpha):
        """Доверительный интервал для среднего"""
        margin_error = 1.96 * std / math.sqrt(n)  # Приближение для больших выборок
        return (mean - margin_error, mean + margin_error)
    
    @staticmethod
    def chi_square_test(observed, expected):
        """
        Тест хи-квадрат для проверки согласия
        H0: наблюдаемые частоты соответствуют ожидаемым
        """
        if len(observed) != len(expected):
            raise ValueError("Векторы должны быть одинаковой длины")
        
        chi_square = sum((obs - exp)**2 / exp for obs, exp in zip(observed, expected))
        df = len(observed) - 1
        
        # Критическое значение (приближенное)
        critical_value = 3.84 if df == 1 else 5.99 if df == 2 else 7.81
        
        return {
            'chi_square': chi_square,
            'degrees_of_freedom': df,
            'critical_value': critical_value,
            'reject_null': chi_square > critical_value
        }
    
    @staticmethod
    def ab_test_analysis(control_group, treatment_group, alpha=0.05):
        """
        A/B тест для сравнения двух групп
        Используется двухвыборочный t-тест
        """
        n1, n2 = len(control_group), len(treatment_group)
        mean1 = DescriptiveStatistics.mean(control_group)
        mean2 = DescriptiveStatistics.mean(treatment_group)
        var1 = DescriptiveStatistics.variance(control_group, ddof=1)
        var2 = DescriptiveStatistics.variance(treatment_group, ddof=1)
        
        # Объединенная дисперсия
        pooled_var = ((n1 - 1) * var1 + (n2 - 1) * var2) / (n1 + n2 - 2)
        
        # Стандартная ошибка разности средних
        se = math.sqrt(pooled_var * (1/n1 + 1/n2))
        
        # t-статистика
        t_statistic = (mean2 - mean1) / se
        
        # Степени свободы
        df = n1 + n2 - 2
        
        # Размер эффекта (Cohen's d)
        cohens_d = (mean2 - mean1) / math.sqrt(pooled_var)
        
        # Критическое значение
        t_critical = 1.96 if df > 30 else 2.0
        
        return {
            'control_mean': mean1,
            'treatment_mean': mean2,
            'difference': mean2 - mean1,
            'relative_improvement': (mean2 - mean1) / mean1 * 100,
            't_statistic': t_statistic,
            'cohens_d': cohens_d,
            'critical_value': t_critical,
            'statistically_significant': abs(t_statistic) > t_critical,
            'effect_size': 'small' if abs(cohens_d) < 0.5 else 'medium' if abs(cohens_d) < 0.8 else 'large'
        }

class ABTestFramework:
    """Фреймворк для проведения A/B тестов"""
    
    def __init__(self, name, hypothesis):
        self.name = name
        self.hypothesis = hypothesis
        self.control_data = []
        self.treatment_data = []
        self.start_time = None
        self.end_time = None
    
    def add_observation(self, group, value):
        """Добавить наблюдение в группу"""
        if group == 'control':
            self.control_data.append(value)
        elif group == 'treatment':
            self.treatment_data.append(value)
        else:
            raise ValueError("Группа должна быть 'control' или 'treatment'")
    
    def calculate_sample_size(self, effect_size, power=0.8, alpha=0.05):
        """
        Расчет размера выборки для A/B теста
        Упрощенная формула для равных групп
        """
        # Коэффициенты для стандартных значений
        z_alpha = 1.96  # для alpha = 0.05
        z_beta = 0.84   # для power = 0.8
        
        n = 2 * ((z_alpha + z_beta) / effect_size) ** 2
        return math.ceil(n)
    
    def analyze(self):
        """Анализ результатов A/B теста"""
        if len(self.control_data) == 0 or len(self.treatment_data) == 0:
            raise ValueError("Недостаточно данных для анализа")
        
        return StatisticalTests.ab_test_analysis(self.control_data, self.treatment_data)
    
    def generate_report(self):
        """Генерация отчета по A/B тесту"""
        results = self.analyze()
        
        report = f"""
        A/B Test Report: {self.name}
        ================================
        
        Hypothesis: {self.hypothesis}
        
        Sample Sizes:
        - Control: {len(self.control_data)}
        - Treatment: {len(self.treatment_data)}
        
        Results:
        - Control Mean: {results['control_mean']:.4f}
        - Treatment Mean: {results['treatment_mean']:.4f}
        - Difference: {results['difference']:.4f}
        - Relative Improvement: {results['relative_improvement']:.2f}%
        
        Statistical Significance:
        - t-statistic: {results['t_statistic']:.4f}
        - Statistically Significant: {results['statistically_significant']}
        
        Effect Size:
        - Cohen's d: {results['cohens_d']:.4f}
        - Effect Size: {results['effect_size']}
        
        Recommendation:
        {'Implement treatment' if results['statistically_significant'] and results['difference'] > 0 
         else 'Continue with control' if results['statistically_significant'] 
         else 'Inconclusive - collect more data'}
        """
        
        return report
```

---

## 📈 Корреляция и регрессия

```python
class RegressionAnalysis:
    """Регрессионный анализ"""
    
    @staticmethod
    def simple_linear_regression(x, y):
        """
        Простая линейная регрессия: y = a + bx
        Метод наименьших квадратов
        """
        n = len(x)
        if n != len(y):
            raise ValueError("Векторы должны быть одинаковой длины")
        
        sum_x = sum(x)
        sum_y = sum(y)
        sum_xy = sum(x[i] * y[i] for i in range(n))
        sum_x2 = sum(x[i]**2 for i in range(n))
        
        # Коэффициенты регрессии
        b = (n * sum_xy - sum_x * sum_y) / (n * sum_x2 - sum_x**2)
        a = (sum_y - b * sum_x) / n
        
        # Предсказанные значения
        y_pred = [a + b * xi for xi in x]
        
        # Коэффициент детерминации R²
        y_mean = sum(y) / n
        ss_tot = sum((yi - y_mean)**2 for yi in y)
        ss_res = sum((y[i] - y_pred[i])**2 for i in range(n))
        r_squared = 1 - (ss_res / ss_tot) if ss_tot != 0 else 1
        
        return {
            'intercept': a,
            'slope': b,
            'r_squared': r_squared,
            'predictions': y_pred,
            'equation': f'y = {a:.4f} + {b:.4f}x'
        }
    
    @staticmethod
    def polynomial_regression(x, y, degree=2):
        """Полиномиальная регрессия"""
        n = len(x)
        
        # Создаем матрицу Вандермонда
        X = [[xi**j for j in range(degree + 1)] for xi in x]
        
        # Решаем систему нормальных уравнений X^T * X * beta = X^T * y
        # Упрощенная версия для квадратичной регрессии (degree=2)
        if degree == 2:
            sum_x = sum(x)
            sum_x2 = sum(xi**2 for xi in x)
            sum_x3 = sum(xi**3 for xi in x)
            sum_x4 = sum(xi**4 for xi in x)
            sum_y = sum(y)
            sum_xy = sum(x[i] * y[i] for i in range(n))
            sum_x2y = sum(x[i]**2 * y[i] for i in range(n))
            
            # Решение системы 3x3 (упрощенное)
            # Здесь должна быть полная реализация решения СЛАУ
            # Для демонстрации возвращаем простую линейную регрессию
            return RegressionAnalysis.simple_linear_regression(x, y)
        else:
            return RegressionAnalysis.simple_linear_regression(x, y)

class TimeSeriesAnalysis:
    """Анализ временных рядов"""
    
    @staticmethod
    def moving_average(data, window_size):
        """Скользящее среднее"""
        if window_size > len(data):
            raise ValueError("Размер окна больше размера данных")
        
        result = []
        for i in range(len(data) - window_size + 1):
            window = data[i:i + window_size]
            result.append(sum(window) / window_size)
        
        return result
    
    @staticmethod
    def exponential_smoothing(data, alpha=0.3):
        """Экспоненциальное сглаживание"""
        if not (0 < alpha <= 1):
            raise ValueError("Альфа должна быть в диапазоне (0, 1]")
        
        smoothed = [data[0]]  # Первое значение остается неизменным
        
        for i in range(1, len(data)):
            smoothed_value = alpha * data[i] + (1 - alpha) * smoothed[i - 1]
            smoothed.append(smoothed_value)
        
        return smoothed
    
    @staticmethod
    def detect_trend(data):
        """Обнаружение тренда методом линейной регрессии"""
        x = list(range(len(data)))
        regression_result = RegressionAnalysis.simple_linear_regression(x, data)
        
        slope = regression_result['slope']
        
        if slope > 0.1:
            return 'increasing'
        elif slope < -0.1:
            return 'decreasing'
        else:
            return 'stable'
    
    @staticmethod
    def seasonal_decomposition(data, period):
        """Простое сезонное разложение"""
        if len(data) < period * 2:
            raise ValueError("Недостаточно данных для сезонного разложения")
        
        # Тренд (скользящее среднее)
        trend = TimeSeriesAnalysis.moving_average(data, period)
        
        # Дополняем тренд до полной длины
        trend_full = [None] * (period // 2) + trend + [None] * (period // 2)
        
        # Сезонная компонента
        seasonal = []
        for i in range(len(data)):
            if trend_full[i] is not None:
                seasonal.append(data[i] - trend_full[i])
            else:
                seasonal.append(0)
        
        # Остатки
        residual = []
        for i in range(len(data)):
            if trend_full[i] is not None:
                residual.append(data[i] - trend_full[i] - seasonal[i])
            else:
                residual.append(0)
        
        return {
            'original': data,
            'trend': trend_full,
            'seasonal': seasonal,
            'residual': residual
        }
```

---

## 🚀 Практические применения

```python
class WebAnalyticsStatistics:
    """Статистика для веб-аналитики"""
    
    def __init__(self):
        self.page_views = []
        self.conversion_rates = []
        self.session_durations = []
    
    def add_session_data(self, page_views, converted, duration):
        """Добавить данные сессии"""
        self.page_views.append(page_views)
        self.conversion_rates.append(1 if converted else 0)
        self.session_durations.append(duration)
    
    def calculate_metrics(self):
        """Расчет основных метрик"""
        total_sessions = len(self.page_views)
        
        if total_sessions == 0:
            return {}
        
        metrics = {
            'total_sessions': total_sessions,
            'avg_page_views': DescriptiveStatistics.mean(self.page_views),
            'conversion_rate': DescriptiveStatistics.mean(self.conversion_rates),
            'avg_session_duration': DescriptiveStatistics.mean(self.session_durations),
            'bounce_rate': sum(1 for pv in self.page_views if pv == 1) / total_sessions,
            'page_views_stats': DescriptiveStatistics.five_number_summary(self.page_views)
        }
        
        return metrics
    
    def compare_periods(self, other_period):
        """Сравнение двух периодов"""
        current_metrics = self.calculate_metrics()
        other_metrics = other_period.calculate_metrics()
        
        if not current_metrics or not other_metrics:
            return None
        
        comparison = {}
        for metric in ['conversion_rate', 'avg_page_views', 'avg_session_duration']:
            if metric in current_metrics and metric in other_metrics:
                current_val = current_metrics[metric]
                other_val = other_metrics[metric]
                
                if other_val != 0:
                    change_percent = (current_val - other_val) / other_val * 100
                    comparison[metric] = {
                        'current': current_val,
                        'previous': other_val,
                        'change_percent': change_percent,
                        'improvement': change_percent > 0
                    }
        
        return comparison

# Демонстрация использования
def demo_statistics():
    """Демонстрация статистических методов"""
    
    # Генерация тестовых данных
    import random
    random.seed(42)
    
    # Данные для A/B теста конверсии
    control_conversions = [random.random() < 0.12 for _ in range(1000)]  # 12% базовая конверсия
    treatment_conversions = [random.random() < 0.15 for _ in range(1000)]  # 15% после изменений
    
    # Настройка A/B теста
    ab_test = ABTestFramework(
        "Landing Page Optimization",
        "Новый дизайн главной страницы увеличивает конверсию"
    )
    
    for conversion in control_conversions:
        ab_test.add_observation('control', conversion)
    
    for conversion in treatment_conversions:
        ab_test.add_observation('treatment', conversion)
    
    # Анализ результатов
    print(ab_test.generate_report())
    
    # Веб-аналитика
    analytics = WebAnalyticsStatistics()
    
    # Добавляем тестовые данные сессий
    for _ in range(1000):
        page_views = random.randint(1, 10)
        converted = random.random() < 0.1
        duration = random.randint(30, 600)  # секунды
        analytics.add_session_data(page_views, converted, duration)
    
    metrics = analytics.calculate_metrics()
    print(f"\nВеб-аналитика:")
    print(f"Общее количество сессий: {metrics['total_sessions']}")
    print(f"Средние просмотры страниц: {metrics['avg_page_views']:.2f}")
    print(f"Коэффициент конверсии: {metrics['conversion_rate']:.2%}")
    print(f"Средняя продолжительность сессии: {metrics['avg_session_duration']:.0f} сек")
    print(f"Показатель отказов: {metrics['bounce_rate']:.2%}")
```

**Когда использовать статистические методы:**

- ✅ **A/B тесты**: Сравнение вариантов интерфейса, алгоритмов
- ✅ **Описательная статистика**: Анализ производительности, метрик
- ✅ **Корреляционный анализ**: Выявление связей между переменными
- ✅ **Временные ряды**: Мониторинг системы, прогнозирование

---

## 🔗 Связанные темы

- [[probability-randomized-algorithms|Вероятностные алгоритмы]] - статистические тесты
- [[calculus-optimization|Математический анализ]] - оптимизация в ML
- [[linear-algebra-computation|Линейная алгебра]] - многомерная статистика
- [[../probability-statistics/practical-applications|Практические применения статистики]]

## 📚 Дополнительные ресурсы

- **Книги**: "The Elements of Statistical Learning" - Hastie, Tibshirani, Friedman
- **Курсы**: Stanford CS109, MIT 18.05
- **Практика**: Kaggle competitions, A/B testing platforms 