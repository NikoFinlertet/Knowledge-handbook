# Практические применения 🚀

> [[time-series|← Временные ряды]] | [[README|Оглавление]] | [[bayesian-statistics|Байесовская статистика →]]

## 🎯 Что изучим

- **A/B тестирование в разработке** - научный подход к улучшению продуктов
- **Анализ производительности** - статистические методы мониторинга систем
- **Машинное обучение** - статистические основы ML алгоритмов

---

## 1. A/B тестирование в IT

### 🧪 Практический фреймворк

```python
class ABTestFramework:
    def __init__(self):
        self.experiments = {}
        
    def design_experiment(self, metric_name, baseline_value, expected_improvement):
        """Планирование эксперимента"""
        import math
        
        # Расчёт размера выборки для пропорций
        p1 = baseline_value
        p2 = baseline_value + expected_improvement
        p_pooled = (p1 + p2) / 2
        
        effect_size = abs(p2 - p1) / math.sqrt(p_pooled * (1 - p_pooled))
        z_alpha, z_beta = 1.96, 0.84  # α=0.05, power=80%
        n_per_group = 2 * ((z_alpha + z_beta) / effect_size) ** 2
        
        return {
            'sample_size_per_group': math.ceil(n_per_group),
            'minimum_runtime_days': math.ceil(n_per_group / 1000),
            'effect_size': effect_size
        }
    
    def analyze_results(self, control_data, treatment_data):
        """Анализ результатов для пропорций"""
        import math
        
        n1, n2 = len(control_data), len(treatment_data)
        x1, x2 = sum(control_data), sum(treatment_data)
        
        p1 = x1 / n1 if n1 > 0 else 0
        p2 = x2 / n2 if n2 > 0 else 0
        
        # Z-тест для пропорций
        p_pooled = (x1 + x2) / (n1 + n2) if (n1 + n2) > 0 else 0
        se = math.sqrt(p_pooled * (1 - p_pooled) * (1/n1 + 1/n2))
        z_stat = (p2 - p1) / se if se > 0 else 0
        
        # Практические метрики
        lift = (p2 - p1) / p1 * 100 if p1 > 0 else 0
        
        return {
            'control_rate': p1,
            'treatment_rate': p2,
            'lift_percent': lift,
            'statistically_significant': abs(z_stat) > 1.96,
            'z_statistic': z_stat
        }

# Пример использования
framework = ABTestFramework()

# Планирование
design = framework.design_experiment("conversion_rate", 0.05, 0.01)
print(f"Размер выборки: {design['sample_size_per_group']} на группу")

# Анализ (симуляция)
import random
random.seed(42)
control = [1 if random.random() < 0.05 else 0 for _ in range(2000)]
treatment = [1 if random.random() < 0.058 else 0 for _ in range(2000)]

results = framework.analyze_results(control, treatment)
print(f"Конверсия контроль: {results['control_rate']:.3f}")
print(f"Конверсия тест: {results['treatment_rate']:.3f}")
print(f"Прирост: {results['lift_percent']:.1f}%")
print(f"Значимо: {results['statistically_significant']}")
```

---

## 2. Анализ производительности систем

```python
class PerformanceMonitor:
    def __init__(self):
        self.baselines = {}
    
    def set_baseline(self, metric_name, historical_data):
        """Установка базовой линии"""
        import math
        
        n = len(historical_data)
        mean = sum(historical_data) / n
        variance = sum((x - mean) ** 2 for x in historical_data) / (n - 1)
        std = math.sqrt(variance)
        
        self.baselines[metric_name] = {
            'mean': mean,
            'std': std,
            'warning_threshold': mean + 2 * std,
            'critical_threshold': mean + 3 * std
        }
        
        return self.baselines[metric_name]
    
    def analyze_current(self, metric_name, current_values):
        """Анализ текущих значений"""
        if metric_name not in self.baselines:
            return {"error": "Базовая линия не установлена"}
        
        baseline = self.baselines[metric_name]
        current_mean = sum(current_values) / len(current_values)
        
        # Z-score
        z_score = (current_mean - baseline['mean']) / baseline['std']
        
        # Классификация
        if current_mean > baseline['critical_threshold']:
            status = "CRITICAL"
        elif current_mean > baseline['warning_threshold']:
            status = "WARNING"
        else:
            status = "NORMAL"
        
        # Аномалии
        anomalies = [
            value for value in current_values 
            if abs(value - baseline['mean']) > 3 * baseline['std']
        ]
        
        return {
            'status': status,
            'current_mean': current_mean,
            'z_score': z_score,
            'anomalies_count': len(anomalies),
            'recommendations': self._get_recommendations(status, len(anomalies))
        }
    
    def _get_recommendations(self, status, anomaly_count):
        """Генерация рекомендаций"""
        recommendations = []
        
        if status == "CRITICAL":
            recommendations.append("🚨 Критический уровень - проверить систему")
        elif status == "WARNING":
            recommendations.append("⚠️ Предупреждение - мониторить ситуацию")
        
        if anomaly_count > 0:
            recommendations.append(f"🔍 Обнаружено {anomaly_count} аномалий")
        
        if not recommendations:
            recommendations.append("✅ Система в норме")
        
        return recommendations

# Пример мониторинга API
monitor = PerformanceMonitor()

# Исторические данные времени ответа (мс)
historical_times = [120, 118, 125, 122, 119, 121, 117, 123, 116, 124]
baseline = monitor.set_baseline("api_response_time", historical_times)

print(f"Базовая линия: {baseline['mean']:.1f} ± {baseline['std']:.1f} мс")

# Текущие данные (ухудшение)
current_times = [145, 138, 152, 148, 155]
analysis = monitor.analyze_current("api_response_time", current_times)

print(f"Статус: {analysis['status']}")
print(f"Текущее среднее: {analysis['current_mean']:.1f} мс")
print(f"Z-score: {analysis['z_score']:.2f}")
for rec in analysis['recommendations']:
    print(f"  {rec}")
```

---

## 3. Статистические основы ML

```python
class MLStatistics:
    @staticmethod
    def cross_validation_stats(cv_scores):
        """Статистический анализ CV результатов"""
        import math
        
        mean_score = sum(cv_scores) / len(cv_scores)
        variance = sum((score - mean_score) ** 2 for score in cv_scores) / (len(cv_scores) - 1)
        std_score = math.sqrt(variance)
        
        # Доверительный интервал
        margin_error = 1.96 * (std_score / math.sqrt(len(cv_scores)))
        
        # Стабильность
        stability = 1 - (std_score / mean_score) if mean_score > 0 else 0
        
        return {
            'mean_score': mean_score,
            'std_score': std_score,
            'confidence_interval': [mean_score - margin_error, mean_score + margin_error],
            'stability': stability,
            'is_stable': std_score < 0.05
        }
    
    @staticmethod
    def feature_significance_test(feature_scores):
        """Тест значимости признаков"""
        import random
        
        # Baseline из случайных значений
        random.seed(42)
        baseline_scores = [random.random() for _ in range(1000)]
        baseline_mean = sum(baseline_scores) / len(baseline_scores)
        baseline_std = (sum((x - baseline_mean) ** 2 for x in baseline_scores) / len(baseline_scores)) ** 0.5
        
        significant_features = []
        for i, score in enumerate(feature_scores):
            z_score = (score - baseline_mean) / baseline_std
            p_value = 0.01 if z_score > 2.5 else 0.05 if z_score > 1.96 else 0.1
            
            significant_features.append({
                'feature_index': i,
                'score': score,
                'z_score': z_score,
                'p_value': p_value,
                'significant': p_value <= 0.05
            })
        
        return sorted(significant_features, key=lambda x: x['score'], reverse=True)

# Пример анализа модели
ml_stats = MLStatistics()

# Cross-validation результаты
cv_scores = [0.85, 0.87, 0.83, 0.86, 0.84]
cv_analysis = ml_stats.cross_validation_stats(cv_scores)

print("Cross-Validation Analysis:")
print(f"Средняя точность: {cv_analysis['mean_score']:.3f} ± {cv_analysis['std_score']:.3f}")
print(f"Стабильность: {cv_analysis['stability']:.3f}")
print(f"Стабильная модель: {cv_analysis['is_stable']}")

# Важность признаков
feature_scores = [0.25, 0.18, 0.15, 0.12, 0.08, 0.06, 0.04, 0.03]
significance = ml_stats.feature_significance_test(feature_scores)

print("\nТоп-3 значимых признака:")
for feature in significance[:3]:
    if feature['significant']:
        print(f"  Признак {feature['feature_index']}: {feature['score']:.3f} (p={feature['p_value']:.3f})")
```

---

## 📚 Практические проекты

### 1. Система A/B тестирования
- Автоматический расчёт размера выборки
- Онлайн мониторинг результатов
- Статистическая валидация

### 2. Performance Dashboard
- Real-time аномалия детекция
- SLA мониторинг
- Predictive alerts

### 3. ML Model Validation
- Автоматизированная CV валидация
- Feature importance анализ
- Bias-variance decomposition

---

## 🛠️ Инструменты

```python
# Основные библиотеки
import scipy.stats as stats
import pandas as pd
import numpy as np
import scikit-learn as sklearn
```

---

## 🎯 Ключевые принципы

- **Научный подход** - измеряемые изменения
- **Статистическая vs практическая значимость**
- **Воспроизводимость результатов**
- **Контекстуальное понимание данных**

> 💡 **Помните**: Статистика в IT не только про числа, но и про принятие обоснованных решений в условиях неопределенности! 