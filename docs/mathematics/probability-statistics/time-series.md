# Временные ряды 📊

> [[statistical-tests|← Статистические тесты]] | [[README|Оглавление]] | [[practical-applications|Практические применения →]]

## 🎯 Что изучим

- **Анализ трендов** - выявление направления изменений
- **Сезонность** - повторяющиеся паттерны в данных
- **Прогнозирование** - предсказание будущих значений

---

## 1. Анализ трендов

```python
class TimeSeriesAnalyzer:
    def __init__(self, data, time_points=None):
        self.data = data
        self.time_points = time_points or list(range(len(data)))
        self.n = len(data)
    
    def moving_average(self, window_size):
        """Скользящее среднее"""
        if window_size > self.n:
            return []
        
        moving_avg = []
        for i in range(window_size - 1, self.n):
            avg = sum(self.data[i - window_size + 1:i + 1]) / window_size
            moving_avg.append(avg)
        
        return moving_avg
    
    def exponential_smoothing(self, alpha=0.3):
        """Экспоненциальное сглаживание"""
        if self.n == 0:
            return []
        
        smoothed = [self.data[0]]
        for i in range(1, self.n):
            smoothed_value = alpha * self.data[i] + (1 - alpha) * smoothed[-1]
            smoothed.append(smoothed_value)
        
        return smoothed
    
    def linear_trend(self):
        """Линейный тренд методом наименьших квадратов"""
        sum_t = sum(self.time_points)
        sum_y = sum(self.data)
        sum_t2 = sum(t ** 2 for t in self.time_points)
        sum_ty = sum(t * y for t, y in zip(self.time_points, self.data))
        
        # Коэффициенты линейного тренда y = at + b
        a = (self.n * sum_ty - sum_t * sum_y) / (self.n * sum_t2 - sum_t ** 2)
        b = (sum_y - a * sum_t) / self.n
        
        return a, b
    
    def forecast(self, steps_ahead):
        """Простой прогноз на основе линейного тренда"""
        a, b = self.linear_trend()
        last_time = self.time_points[-1]
        
        forecasts = []
        for i in range(1, steps_ahead + 1):
            forecast_time = last_time + i
            forecast_value = a * forecast_time + b
            forecasts.append(forecast_value)
        
        return forecasts
    
    def seasonal_decomposition(self, period):
        """Упрощенная сезонная декомпозиция"""
        # Вычисление тренда с помощью скользящего среднего
        trend = self.moving_average(period)
        
        # Вычисление сезонных компонент
        seasonal = []
        for i in range(self.n):
            if i >= period // 2 and i < self.n - period // 2:
                seasonal_component = self.data[i] - trend[i - period // 2]
            else:
                seasonal_component = 0
            seasonal.append(seasonal_component)
        
        # Остатки
        residuals = []
        for i in range(self.n):
            if i >= period // 2 and i < self.n - period // 2:
                residual = self.data[i] - trend[i - period // 2] - seasonal[i]
            else:
                residual = self.data[i] - seasonal[i]
            residuals.append(residual)
        
        return trend, seasonal, residuals

# Пример: анализ трафика веб-сайта
daily_visitors = [
    1200, 1150, 1300, 1250, 1400, 1100, 900,   # Неделя 1
    1250, 1200, 1350, 1300, 1450, 1150, 950,   # Неделя 2
    1300, 1250, 1400, 1350, 1500, 1200, 1000,  # Неделя 3
    1350, 1300, 1450, 1400, 1550, 1250, 1050   # Неделя 4
]

analyzer = TimeSeriesAnalyzer(daily_visitors)

# Скользящее среднее
ma_7 = analyzer.moving_average(7)
print(f"Скользящее среднее за 7 дней: {ma_7[-5:]}")

# Экспоненциальное сглаживание
exp_smooth = analyzer.exponential_smoothing(alpha=0.2)
print(f"Экспоненциальное сглаживание: {exp_smooth[-5:]}")

# Линейный тренд
a, b = analyzer.linear_trend()
print(f"Линейный тренд: y = {a:.2f}t + {b:.2f}")

# Прогноз на 7 дней
forecast = analyzer.forecast(7)
print(f"Прогноз на следующие 7 дней: {[round(f) for f in forecast]}")

# Сезонная декомпозиция (недельная сезонность)
trend, seasonal, residuals = analyzer.seasonal_decomposition(7)
print(f"Сезонные компоненты: {seasonal[-7:]}")
```

### 🎯 Применение в IT

- **Мониторинг трафика** - выявление трендов посещаемости
- **Capacity Planning** - прогнозирование нагрузки на сервера
- **Performance Tracking** - отслеживание метрик производительности

---

## 2. Автокорреляция

```python
import math

class AutocorrelationAnalyzer:
    def __init__(self, data):
        self.data = data
        self.n = len(data)
        self.mean = sum(data) / self.n
    
    def autocorrelation(self, lag):
        """Автокорреляция с заданным лагом"""
        if lag >= self.n:
            return 0
        
        numerator = sum((self.data[i] - self.mean) * (self.data[i + lag] - self.mean) 
                       for i in range(self.n - lag))
        denominator = sum((x - self.mean) ** 2 for x in self.data)
        
        return numerator / denominator
    
    def autocorrelation_function(self, max_lag=None):
        """Функция автокорреляции"""
        if max_lag is None:
            max_lag = min(self.n // 4, 20)
        
        autocorrelations = []
        for lag in range(max_lag + 1):
            autocorr = self.autocorrelation(lag)
            autocorrelations.append(autocorr)
        
        return autocorrelations
    
    def detect_seasonality(self, max_lag=None):
        """Определение сезонности"""
        autocorrs = self.autocorrelation_function(max_lag)
        
        # Поиск пиков автокорреляции
        peaks = []
        for i in range(1, len(autocorrs) - 1):
            if autocorrs[i] > 0.3 and autocorrs[i] > autocorrs[i-1] and autocorrs[i] > autocorrs[i+1]:
                peaks.append((i, autocorrs[i]))
        
        return peaks

# Пример: анализ автокорреляции трафика
traffic_data = daily_visitors  # Используем данные из предыдущего примера
autocorr_analyzer = AutocorrelationAnalyzer(traffic_data)

# Автокорреляция с различными лагами
print("Автокорреляция:")
for lag in range(1, 8):
    autocorr = autocorr_analyzer.autocorrelation(lag)
    print(f"Лаг {lag}: {autocorr:.3f}")

# Определение сезонности
peaks = autocorr_analyzer.detect_seasonality()
print(f"\nОбнаруженные сезонные периоды: {peaks}")
```

### 📈 Интерпретация автокорреляции

| Лаг | Автокорреляция | Интерпретация |
|-----|---------------|---------------|
| **1** | > 0.7 | Сильная краткосрочная зависимость |
| **7** | > 0.3 | Недельная сезонность |
| **24** | > 0.3 | Суточная сезонность (часовые данные) |

---

## 3. Прогнозирование

```python
class TimeSeriesForecaster:
    def __init__(self, data):
        self.data = data
        self.n = len(data)
        self.trend_params = None
        self.seasonal_params = None
        
    def simple_exponential_smoothing(self, alpha=0.3):
        """Простое экспоненциальное сглаживание"""
        if self.n == 0:
            return []
        
        smoothed = [self.data[0]]
        for i in range(1, self.n):
            smoothed_value = alpha * self.data[i] + (1 - alpha) * smoothed[-1]
            smoothed.append(smoothed_value)
        
        return smoothed
    
    def double_exponential_smoothing(self, alpha=0.3, beta=0.3):
        """Двойное экспоненциальное сглаживание (Holt)"""
        if self.n < 2:
            return self.data
        
        # Инициализация
        level = [self.data[0]]
        trend = [self.data[1] - self.data[0]]
        forecast = [self.data[0] + trend[0]]
        
        for i in range(1, self.n):
            level_new = alpha * self.data[i] + (1 - alpha) * (level[i-1] + trend[i-1])
            trend_new = beta * (level_new - level[i-1]) + (1 - beta) * trend[i-1]
            
            level.append(level_new)
            trend.append(trend_new)
            
            if i < self.n - 1:
                forecast.append(level_new + trend_new)
        
        self.trend_params = {'level': level[-1], 'trend': trend[-1]}
        return forecast
    
    def forecast_steps(self, steps, method='double_exponential'):
        """Прогнозирование на несколько шагов вперед"""
        if method == 'double_exponential':
            if not self.trend_params:
                self.double_exponential_smoothing()
            
            forecasts = []
            for h in range(1, steps + 1):
                forecast_value = self.trend_params['level'] + h * self.trend_params['trend']
                forecasts.append(forecast_value)
            
            return forecasts
        
        elif method == 'linear_trend':
            analyzer = TimeSeriesAnalyzer(self.data)
            return analyzer.forecast(steps)
        
        else:
            raise ValueError("Неподдерживаемый метод прогнозирования")
    
    def evaluate_forecast_accuracy(self, actual, predicted):
        """Оценка точности прогноза"""
        if len(actual) != len(predicted):
            min_len = min(len(actual), len(predicted))
            actual = actual[:min_len]
            predicted = predicted[:min_len]
        
        n = len(actual)
        if n == 0:
            return {}
        
        # Средняя абсолютная ошибка
        mae = sum(abs(actual[i] - predicted[i]) for i in range(n)) / n
        
        # Средняя квадратичная ошибка
        mse = sum((actual[i] - predicted[i]) ** 2 for i in range(n)) / n
        rmse = math.sqrt(mse)
        
        # Средняя абсолютная процентная ошибка
        mape = sum(abs((actual[i] - predicted[i]) / actual[i]) for i in range(n) 
                  if actual[i] != 0) / sum(1 for x in actual if x != 0) * 100
        
        return {
            'MAE': mae,
            'MSE': mse,
            'RMSE': rmse,
            'MAPE': mape
        }

# Пример: прогнозирование производительности сервера
server_response_times = [
    120, 118, 125, 122, 119, 121, 117, 123, 116, 124,
    121, 119, 126, 123, 120, 122, 118, 125, 119, 127
]

forecaster = TimeSeriesForecaster(server_response_times)

# Двойное экспоненциальное сглаживание
holt_forecast = forecaster.double_exponential_smoothing(alpha=0.4, beta=0.2)
print("Сглаженные значения (последние 5):", [round(x, 1) for x in holt_forecast[-5:]])

# Прогноз на 5 шагов вперёд
future_forecast = forecaster.forecast_steps(5, method='double_exponential')
print("Прогноз на 5 периодов:", [round(x, 1) for x in future_forecast])

# Оценка точности (используем последние данные как тестовые)
train_data = server_response_times[:-5]
test_data = server_response_times[-5:]

train_forecaster = TimeSeriesForecaster(train_data)
test_forecast = train_forecaster.forecast_steps(5, method='double_exponential')

accuracy = train_forecaster.evaluate_forecast_accuracy(test_data, test_forecast)
print(f"\nТочность прогноза:")
print(f"MAE: {accuracy['MAE']:.2f}")
print(f"RMSE: {accuracy['RMSE']:.2f}")
print(f"MAPE: {accuracy['MAPE']:.2f}%")
```

### 🎯 Выбор метода прогнозирования

| Характеристика данных | Рекомендуемый метод |
|----------------------|-------------------|
| **Стабильный уровень** | Простое экспоненциальное сглаживание |
| **Есть тренд** | Двойное экспоненциальное сглаживание |
| **Есть сезонность** | Тройное экспоненциальное сглаживание |
| **Сложные паттерны** | ARIMA, ML методы |

---

## 4. Аномалии во временных рядах

```python
class AnomalyDetector:
    def __init__(self, data, window_size=10):
        self.data = data
        self.window_size = window_size
        self.n = len(data)
    
    def statistical_anomalies(self, threshold=2.0):
        """Обнаружение аномалий на основе Z-score"""
        anomalies = []
        
        for i in range(self.window_size, self.n):
            # Локальное окно
            window = self.data[i - self.window_size:i]
            mean = sum(window) / len(window)
            std = math.sqrt(sum((x - mean) ** 2 for x in window) / len(window))
            
            if std > 0:
                z_score = abs(self.data[i] - mean) / std
                if z_score > threshold:
                    anomalies.append({
                        'index': i,
                        'value': self.data[i],
                        'z_score': z_score,
                        'expected': mean
                    })
        
        return anomalies
    
    def seasonal_anomalies(self, period=7, threshold=2.0):
        """Обнаружение аномалий с учётом сезонности"""
        anomalies = []
        
        for i in range(period, self.n):
            # Исторические значения в тот же день недели/час
            seasonal_values = []
            for j in range(i - period, -1, -period):
                if j >= 0:
                    seasonal_values.append(self.data[j])
                if len(seasonal_values) >= 4:  # Минимум 4 исторических значения
                    break
            
            if len(seasonal_values) >= 2:
                mean = sum(seasonal_values) / len(seasonal_values)
                std = math.sqrt(sum((x - mean) ** 2 for x in seasonal_values) / len(seasonal_values))
                
                if std > 0:
                    z_score = abs(self.data[i] - mean) / std
                    if z_score > threshold:
                        anomalies.append({
                            'index': i,
                            'value': self.data[i],
                            'z_score': z_score,
                            'seasonal_expected': mean,
                            'historical_values': seasonal_values
                        })
        
        return anomalies

# Пример: обнаружение аномалий в метриках API
api_response_times = [
    120, 118, 125, 122, 119, 121, 117, 123, 116, 124,
    121, 119, 126, 123, 120, 122, 118, 125, 119, 127,
    180, 119, 121, 118, 125, 122, 119, 121, 117, 200  # Две аномалии
]

detector = AnomalyDetector(api_response_times, window_size=10)

# Обнаружение статистических аномалий
stat_anomalies = detector.statistical_anomalies(threshold=2.5)
print("Обнаруженные аномалии:")
for anomaly in stat_anomalies:
    print(f"Позиция {anomaly['index']}: значение {anomaly['value']}, "
          f"ожидалось ~{anomaly['expected']:.1f}, Z-score: {anomaly['z_score']:.2f}")

# Сезонные аномалии (для демонстрации используем период 7)
seasonal_anomalies = detector.seasonal_anomalies(period=7, threshold=2.0)
print(f"\nСезонные аномалии: {len(seasonal_anomalies)} обнаружено")
```

---

## 📚 Связанные темы

- [[descriptive-statistics|Описательная статистика]] - основы анализа данных
- [[statistical-tests|Статистические тесты]] - проверка изменений
- [[practical-applications|Практические применения]] - реальные кейсы

---

## 🛠️ Инструменты

### Python библиотеки
```python
import pandas as pd           # Работа с временными рядами
import numpy as np           # Численные вычисления
import matplotlib.pyplot as plt  # Визуализация
import statsmodels.api as sm     # Статистические модели
from statsmodels.tsa import holtwinters, arima
```

### Практические задания

1. **Traffic Analysis** - проанализируйте сезонность веб-трафика
2. **Performance Forecasting** - спрогнозируйте нагрузку на сервер
3. **Anomaly Detection** - создайте систему обнаружения аномалий в метриках

---

## 🎯 Ключевые концепции

- **Тренд** - долгосрочное направление изменений
- **Сезонность** - регулярные повторяющиеся паттерны
- **Автокорреляция** - связь значений с прошлыми значениями
- **Сглаживание** - устранение шума для выявления паттернов
- **Прогнозирование** - предсказание будущих значений 