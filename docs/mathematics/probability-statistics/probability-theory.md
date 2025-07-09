# Основы теории вероятностей 📊

> [[README|← Назад к оглавлению]] | [[descriptive-statistics|Описательная статистика →]]

## 🎯 Что изучим

- **Понятие вероятности** - классическое и статистическое определения
- **Условная вероятность** - формула Байеса и её применение
- **Распределения** - биномиальное, нормальное и их использование в IT

---

## 1. Понятие вероятности

### Определение и свойства

```python
# Классическое определение вероятности
def classic_probability(favorable_outcomes, total_outcomes):
    """P(A) = |A| / |Ω|"""
    return favorable_outcomes / total_outcomes

# Пример: вероятность успешного запроса к API
def api_success_probability(successful_requests, total_requests):
    """Вероятность успешного ответа API"""
    return successful_requests / total_requests

# Анализ логов
api_logs = [
    {"status": 200, "response_time": 120},
    {"status": 200, "response_time": 95},
    {"status": 500, "response_time": 0},
    {"status": 200, "response_time": 110},
    {"status": 404, "response_time": 50},
    {"status": 200, "response_time": 89}
]

successful = sum(1 for log in api_logs if log["status"] == 200)
total = len(api_logs)
success_rate = api_success_probability(successful, total)
print(f"Вероятность успешного запроса: {success_rate:.2%}")
```

### 🎯 Практическое применение

- **Мониторинг API** - расчёт uptime и SLA
- **Анализ ошибок** - частота различных типов ошибок
- **Планирование capacity** - вероятность превышения нагрузки

---

## 2. Условная вероятность

### Формула Байеса

```python
def bayes_theorem(p_a_given_b, p_b, p_a):
    """
    P(B|A) = P(A|B) * P(B) / P(A)
    """
    return (p_a_given_b * p_b) / p_a

# Пример: обнаружение аномалий в системе
class AnomalyDetector:
    def __init__(self, error_rate=0.05, false_positive_rate=0.02):
        self.error_rate = error_rate  # P(Аномалия)
        self.false_positive_rate = false_positive_rate  # P(Сигнал|Норма)
        self.true_positive_rate = 0.95  # P(Сигнал|Аномалия)
    
    def probability_of_anomaly_given_signal(self):
        """Вероятность аномалии при получении сигнала"""
        p_signal = (self.true_positive_rate * self.error_rate + 
                   self.false_positive_rate * (1 - self.error_rate))
        
        return bayes_theorem(
            self.true_positive_rate,
            self.error_rate,
            p_signal
        )

detector = AnomalyDetector()
anomaly_prob = detector.probability_of_anomaly_given_signal()
print(f"Вероятность аномалии при сигнале: {anomaly_prob:.2%}")
```

### 🎯 Практическое применение

- **Системы алертов** - снижение ложных срабатываний
- **Spam-фильтры** - классификация сообщений
- **Fraud detection** - выявление мошеннических транзакций

---

## 3. Распределения

### Биномиальное распределение

```python
import math

def binomial_coefficient(n, k):
    """Биномиальный коэффициент C(n,k)"""
    return math.factorial(n) // (math.factorial(k) * math.factorial(n - k))

def binomial_probability(n, k, p):
    """Биномиальная вероятность"""
    return binomial_coefficient(n, k) * (p ** k) * ((1 - p) ** (n - k))

# Пример: тестирование системы
def system_reliability_analysis(n_tests=10, success_rate=0.9):
    """Анализ надежности системы"""
    print(f"Анализ {n_tests} тестов с вероятностью успеха {success_rate}")
    
    for k in range(n_tests + 1):
        prob = binomial_probability(n_tests, k, success_rate)
        print(f"P(ровно {k} успехов) = {prob:.4f}")
    
    # Вероятность хотя бы 8 успехов из 10
    prob_at_least_8 = sum(binomial_probability(n_tests, k, success_rate) 
                          for k in range(8, n_tests + 1))
    print(f"P(хотя бы 8 успехов) = {prob_at_least_8:.4f}")

system_reliability_analysis()
```

### Нормальное распределение

```python
import math

def normal_pdf(x, mean, std_dev):
    """Функция плотности нормального распределения"""
    coefficient = 1 / (std_dev * math.sqrt(2 * math.pi))
    exponent = -0.5 * ((x - mean) / std_dev) ** 2
    return coefficient * math.exp(exponent)

def normal_cdf_approximation(x, mean, std_dev):
    """Приближение функции распределения (CDF)"""
    # Используем приближение для стандартного нормального распределения
    z = (x - mean) / std_dev
    return 0.5 * (1 + math.erf(z / math.sqrt(2)))

# Пример: анализ времени ответа сервера
class ResponseTimeAnalyzer:
    def __init__(self, response_times):
        self.response_times = response_times
        self.mean = sum(response_times) / len(response_times)
        self.std_dev = math.sqrt(sum((x - self.mean) ** 2 for x in response_times) / len(response_times))
    
    def probability_within_range(self, lower, upper):
        """Вероятность попадания в диапазон"""
        return (normal_cdf_approximation(upper, self.mean, self.std_dev) - 
                normal_cdf_approximation(lower, self.mean, self.std_dev))
    
    def percentile(self, p):
        """Приближенное вычисление перцентиля"""
        # Упрощенная формула для приближения
        if p == 0.5:
            return self.mean
        elif p < 0.5:
            return self.mean - self.std_dev * abs(math.log(2 * p))
        else:
            return self.mean + self.std_dev * abs(math.log(2 * (1 - p)))

# Анализ времени ответа (в миллисекундах)
response_times = [120, 95, 110, 89, 105, 130, 98, 115, 102, 125, 88, 108, 140, 92, 118]
analyzer = ResponseTimeAnalyzer(response_times)

print(f"Среднее время ответа: {analyzer.mean:.2f} мс")
print(f"Стандартное отклонение: {analyzer.std_dev:.2f} мс")
print(f"Вероятность ответа в пределах 90-120 мс: {analyzer.probability_within_range(90, 120):.2%}")
print(f"95-й перцентиль: {analyzer.percentile(0.95):.2f} мс")
```

### 🎯 Практическое применение

- **Performance monitoring** - анализ времени ответа
- **Load testing** - планирование нагрузки
- **SLA planning** - установка реалистичных метрик

---

## 📚 Связанные темы

- [[descriptive-statistics|Описательная статистика]] - анализ данных
- [[statistical-tests|Статистические тесты]] - проверка гипотез
- [[bayesian-statistics|Байесовская статистика]] - продвинутые методы

---

## 🛠️ Инструменты

### Python библиотеки
```python
import random          # Генерация случайных чисел
import statistics      # Основные статистики
import scipy.stats     # Распределения и тесты
import numpy as np     # Численные вычисления
```

### Практические задания

1. **API Monitoring** - создайте систему мониторинга success rate API
2. **Anomaly Detection** - реализуйте детектор аномалий на основе Байеса
3. **Performance Analysis** - проанализируйте распределение времени ответа

---

## 🎯 Ключевые концепции

- **Вероятность** - мера неопределённости событий
- **Условная вероятность** - вероятность при дополнительной информации
- **Распределения** - модели случайных процессов
- **Байесовский подход** - обновление знаний при получении данных 