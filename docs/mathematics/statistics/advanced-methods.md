# Продвинутые методы 🚀

> [[bayesian-statistics|← Байесовская статистика]] | [[README|Оглавление]] | [[survival-analysis|Анализ выживаемости →]]

## 🎯 Что изучим

- **ANOVA** - сравнение нескольких групп
- **Кластерный анализ** - поиск скрытых паттернов  
- **Снижение размерности** - упрощение сложных данных
- **Продвинутые временные ряды** - сложные паттерны

---

## 1. Дисперсионный анализ (ANOVA)

```python
class ANOVA:
    @staticmethod
    def one_way_anova(groups):
        """Однофакторный дисперсионный анализ"""
        import math
        
        k = len(groups)
        n_total = sum(len(group) for group in groups)
        grand_mean = sum(sum(group) for group in groups) / n_total
        
        # Суммы квадратов
        ss_between = sum(len(group) * (sum(group)/len(group) - grand_mean)**2 for group in groups)
        ss_within = sum(sum((x - sum(group)/len(group))**2 for x in group) for group in groups)
        
        # F-статистика
        ms_between = ss_between / (k - 1)
        ms_within = ss_within / (n_total - k)
        f_stat = ms_between / ms_within if ms_within > 0 else float('inf')
        
        p_value = 0.001 if f_stat > 6 else 0.01 if f_stat > 4 else 0.05 if f_stat > 3 else 0.1
        
        return {
            'f_statistic': f_stat,
            'p_value': p_value,
            'significant': p_value < 0.05,
            'interpretation': f"{'Значимые' if p_value < 0.05 else 'Незначимые'} различия (F={f_stat:.2f})"
        }

# Пример: сравнение производительности серверов
server_response_times = [
    [120, 115, 125, 118, 122],  # Сервер A
    [110, 108, 112, 109, 113],  # Сервер B  
    [130, 128, 132, 129, 133]   # Сервер C
]

result = ANOVA.one_way_anova(server_response_times)
print(f"ANOVA: {result['interpretation']}")
print(f"P-value: {result['p_value']:.3f}")
```

---

## 2. Кластерный анализ

```python
class SimpleKMeans:
    @staticmethod
    def kmeans(data, k, max_iter=50):
        """Упрощенный k-means"""
        import random
        import math
        
        random.seed(42)
        n = len(data)
        
        # Случайные центроиды
        centroids = [data[random.randint(0, n-1)][:] for _ in range(k)]
        
        for _ in range(max_iter):
            # Присвоение к кластерам
            clusters = [[] for _ in range(k)]
            for point in data:
                distances = [math.sqrt(sum((a-b)**2 for a,b in zip(point, centroid))) 
                           for centroid in centroids]
                closest = distances.index(min(distances))
                clusters[closest].append(point)
            
            # Обновление центроидов
            new_centroids = []
            for cluster in clusters:
                if cluster:
                    center = [sum(dim) / len(cluster) for dim in zip(*cluster)]
                    new_centroids.append(center)
                else:
                    new_centroids.append(centroids[len(new_centroids)])
            
            # Проверка сходимости
            if all(math.sqrt(sum((a-b)**2 for a,b in zip(old, new))) < 0.001 
                   for old, new in zip(centroids, new_centroids)):
                break
            
            centroids = new_centroids
        
        return {'centroids': centroids, 'clusters': clusters}

# Пример: кластеризация пользователей
users = [
    [2.1, 1.8],   # время на сайте, страниц за сессию
    [2.3, 2.1], [1.8, 1.5],     # обычные пользователи
    [5.2, 8.1], [4.8, 7.5],     # активные пользователи  
    [0.5, 1.2], [0.3, 0.8]      # быстро ушедшие
]

clustering = SimpleKMeans.kmeans(users, k=3)
print("Кластеры пользователей:")
for i, centroid in enumerate(clustering['centroids']):
    cluster_size = len(clustering['clusters'][i])
    print(f"Группа {i+1}: {centroid[0]:.1f}ч, {centroid[1]:.1f} страниц (размер: {cluster_size})")
```

---

## 3. Снижение размерности (PCA)

```python
class SimplePCA:
    @staticmethod
    def pca_2d(data):
        """Упрощенный PCA для 2D данных"""
        import math
        
        n = len(data)
        
        # Центрирование
        mean_x = sum(point[0] for point in data) / n
        mean_y = sum(point[1] for point in data) / n
        centered = [[x - mean_x, y - mean_y] for x, y in data]
        
        # Ковариационная матрица
        cov_xx = sum(x*x for x, y in centered) / (n-1)
        cov_yy = sum(y*y for x, y in centered) / (n-1)  
        cov_xy = sum(x*y for x, y in centered) / (n-1)
        
        # Собственные значения (2x2 матрица)
        trace = cov_xx + cov_yy
        det = cov_xx * cov_yy - cov_xy**2
        lambda1 = (trace + math.sqrt(trace**2 - 4*det)) / 2
        lambda2 = (trace - math.sqrt(trace**2 - 4*det)) / 2
        
        # Объясненная дисперсия
        total_var = lambda1 + lambda2
        explained_var = [lambda1/total_var, lambda2/total_var] if total_var > 0 else [0.5, 0.5]
        
        return {
            'explained_variance_ratio': explained_var,
            'cumulative_variance': [explained_var[0], explained_var[0] + explained_var[1]]
        }

# Пример: анализ метрик производительности
performance_metrics = [
    [95, 120], [80, 110], [90, 115],    # CPU%, RAM%
    [70, 100], [85, 105], [92, 118]
]

pca_result = SimplePCA.pca_2d(performance_metrics)
print("PCA анализ производительности:")
print(f"PC1 объясняет {pca_result['explained_variance_ratio'][0]:.1%} дисперсии")
print(f"PC2 объясняет {pca_result['explained_variance_ratio'][1]:.1%} дисперсии")
print(f"Суммарно: {pca_result['cumulative_variance'][1]:.1%}")
```

---

## 4. Продвинутый анализ временных рядов

```python
class TimeSeriesAdvanced:
    @staticmethod  
    def seasonal_decomposition(data, period=12):
        """Сезонная декомпозиция"""
        n = len(data)
        
        # Тренд (скользящее среднее)
        trend = []
        half_period = period // 2
        
        for i in range(n):
            if i < half_period or i >= n - half_period:
                trend.append(sum(data) / n)  # Общее среднее для краев
            else:
                window = data[i-half_period:i+half_period+1]
                trend.append(sum(window) / len(window))
        
        # Сезонная компонента
        seasonal_sums = [0] * period
        seasonal_counts = [0] * period
        
        for i in range(n):
            detrended = data[i] - trend[i]
            season_idx = i % period
            seasonal_sums[season_idx] += detrended
            seasonal_counts[season_idx] += 1
        
        seasonal_pattern = [s/c if c > 0 else 0 for s, c in zip(seasonal_sums, seasonal_counts)]
        seasonal = [seasonal_pattern[i % period] for i in range(n)]
        
        # Остатки
        residual = [data[i] - trend[i] - seasonal[i] for i in range(n)]
        
        return {
            'trend': trend,
            'seasonal': seasonal, 
            'residual': residual,
            'seasonal_pattern': seasonal_pattern
        }
    
    @staticmethod
    def detect_anomalies(data, threshold=2):
        """Обнаружение аномалий на основе Z-score"""
        import math
        
        mean_val = sum(data) / len(data)
        variance = sum((x - mean_val)**2 for x in data) / (len(data) - 1)
        std_val = math.sqrt(variance)
        
        anomalies = []
        for i, value in enumerate(data):
            z_score = (value - mean_val) / std_val if std_val > 0 else 0
            if abs(z_score) > threshold:
                anomalies.append({
                    'index': i,
                    'value': value,
                    'z_score': z_score,
                    'type': 'высокая' if z_score > 0 else 'низкая'
                })
        
        return anomalies

# Пример: анализ месячных продаж
monthly_sales = [
    100, 95, 110, 120, 140, 160,      # H1
    180, 170, 150, 130, 120, 190,     # H2 (пик в декабре)
    105, 100, 115, 125, 145, 165,     # H1 следующего года
    185, 175, 155, 135, 125, 195      # H2
]

# Сезонная декомпозиция
decomp = TimeSeriesAdvanced.seasonal_decomposition(monthly_sales, period=12)
print("Сезонный анализ продаж:")

max_seasonal = max(decomp['seasonal_pattern'])
min_seasonal = min(decomp['seasonal_pattern'])
peak_month = decomp['seasonal_pattern'].index(max_seasonal) + 1
low_month = decomp['seasonal_pattern'].index(min_seasonal) + 1

print(f"Пиковый месяц: {peak_month} (+{max_seasonal:.1f})")
print(f"Слабый месяц: {low_month} ({min_seasonal:.1f})")

# Обнаружение аномалий
anomalies = TimeSeriesAdvanced.detect_anomalies(monthly_sales)
if anomalies:
    print(f"\nОбнаружено {len(anomalies)} аномалий:")
    for anomaly in anomalies:
        print(f"  Месяц {anomaly['index']+1}: {anomaly['value']} ({anomaly['type']} аномалия)")
```

---

## 📚 Практические применения

### 🔧 В разработке
- **ANOVA**: Сравнение производительности алгоритмов
- **Кластеризация**: Группировка пользователей, поиск аномалий
- **PCA**: Сжатие данных, визуализация высокоразмерных данных

### 📊 В мониторинге
- **Временные ряды**: Прогнозирование нагрузки
- **Аномалии**: Автоматическое обнаружение проблем
- **Сезонность**: Планирование ресурсов

---

## 🔗 Связанные темы

- [[statistical-tests|Статистические тесты]] - базовые методы
- [[time-series|Временные ряды]] - основы анализа динамики
- [[practical-applications|Практические применения]] - реальные кейсы

---

## 💡 Ключевые принципы

- **Выбор метода**: Понимание когда использовать каждый подход
- **Интерпретация**: Перевод статистических результатов в бизнес-решения
- **Валидация**: Проверка предположений и ограничений методов

> 🚀 **Продвинутые методы**: Мощные инструменты для сложных аналитических задач! 