# Статистические тесты 🧪

> [[descriptive-statistics|← Описательная статистика]] | [[README|Оглавление]] | [[time-series|Временные ряды →]]

## 🎯 Что изучим

- **A/B тестирование** - сравнение двух версий продукта
- **Проверка гипотез** - статистические выводы о данных
- **Критерии значимости** - когда различия действительно важны

---

## 1. A/B тестирование

```python
import math

class ABTest:
    def __init__(self, control_group, treatment_group):
        self.control = control_group
        self.treatment = treatment_group
    
    def two_sample_t_test(self, alpha=0.05):
        """Двухвыборочный t-тест"""
        n1, n2 = len(self.control), len(self.treatment)
        mean1 = sum(self.control) / n1
        mean2 = sum(self.treatment) / n2
        
        # Выборочные дисперсии
        var1 = sum((x - mean1) ** 2 for x in self.control) / (n1 - 1)
        var2 = sum((x - mean2) ** 2 for x in self.treatment) / (n2 - 1)
        
        # Объединенная дисперсия
        pooled_var = ((n1 - 1) * var1 + (n2 - 1) * var2) / (n1 + n2 - 2)
        
        # Стандартная ошибка разности средних
        se = math.sqrt(pooled_var * (1/n1 + 1/n2))
        
        # t-статистика
        t_stat = (mean2 - mean1) / se
        
        # Степени свободы
        df = n1 + n2 - 2
        
        # Критическое значение (приближение)
        t_critical = 1.96 if df > 30 else 2.0  # Упрощение
        
        return {
            'mean_control': mean1,
            'mean_treatment': mean2,
            'difference': mean2 - mean1,
            't_statistic': t_stat,
            'degrees_of_freedom': df,
            'significant': abs(t_stat) > t_critical,
            'effect_size': (mean2 - mean1) / math.sqrt(pooled_var)
        }
    
    def conversion_rate_test(self, alpha=0.05):
        """Тест для коэффициентов конверсии"""
        conversions_control = sum(self.control)
        conversions_treatment = sum(self.treatment)
        n1, n2 = len(self.control), len(self.treatment)
        
        p1 = conversions_control / n1
        p2 = conversions_treatment / n2
        
        # Объединенная пропорция
        p_pooled = (conversions_control + conversions_treatment) / (n1 + n2)
        
        # Стандартная ошибка
        se = math.sqrt(p_pooled * (1 - p_pooled) * (1/n1 + 1/n2))
        
        # Z-статистика
        z_stat = (p2 - p1) / se
        
        # Критическое значение
        z_critical = 1.96  # Для α = 0.05
        
        return {
            'conversion_control': p1,
            'conversion_treatment': p2,
            'lift': (p2 - p1) / p1 if p1 > 0 else float('inf'),
            'z_statistic': z_stat,
            'significant': abs(z_stat) > z_critical,
            'confidence_interval': (p2 - p1) + z_critical * se * [-1, 1]
        }

# Пример: тестирование новой функции
# Время, проведенное пользователями на сайте (в минутах)
control_time = [5.2, 3.8, 4.1, 6.3, 2.9, 5.7, 4.5, 3.2, 5.9, 4.8]
treatment_time = [6.1, 4.9, 5.3, 7.2, 4.1, 6.8, 5.7, 4.3, 6.5, 5.9]

ab_test = ABTest(control_time, treatment_time)
result = ab_test.two_sample_t_test()

print("Результаты A/B теста:")
print(f"Среднее время (контроль): {result['mean_control']:.2f} мин")
print(f"Среднее время (тест): {result['mean_treatment']:.2f} мин")
print(f"Разница: {result['difference']:.2f} мин")
print(f"T-статистика: {result['t_statistic']:.3f}")
print(f"Статистически значимо: {result['significant']}")
print(f"Размер эффекта: {result['effect_size']:.3f}")

# Тест конверсии (1 - конверсия, 0 - нет)
control_conversions = [1, 0, 1, 1, 0, 1, 0, 0, 1, 0, 1, 1, 0, 1, 0]
treatment_conversions = [1, 1, 1, 1, 0, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1]

conversion_test = ABTest(control_conversions, treatment_conversions)
conv_result = conversion_test.conversion_rate_test()

print("\nТест конверсии:")
print(f"Конверсия (контроль): {conv_result['conversion_control']:.2%}")
print(f"Конверсия (тест): {conv_result['conversion_treatment']:.2%}")
print(f"Прирост: {conv_result['lift']:.2%}")
print(f"Статистически значимо: {conv_result['significant']}")
```

### 🎯 Интерпретация результатов

| Показатель | Значение | Интерпретация |
|-----------|----------|---------------|
| **p-value < 0.05** | Статистически значимо | Различие не случайно |
| **Effect Size > 0.2** | Практически значимо | Различие важно для бизнеса |
| **CI не содержит 0** | Надёжное различие | Уверенность в результате |

---

## 2. Проверка гипотез

```python
class HypothesisTest:
    def __init__(self, data, null_hypothesis_value):
        self.data = data
        self.null_value = null_hypothesis_value
        self.n = len(data)
        self.mean = sum(data) / self.n
        self.std = math.sqrt(sum((x - self.mean) ** 2 for x in data) / (self.n - 1))
    
    def one_sample_t_test(self, alpha=0.05, alternative='two-sided'):
        """Одновыборочный t-тест"""
        # Стандартная ошибка среднего
        se = self.std / math.sqrt(self.n)
        
        # T-статистика
        t_stat = (self.mean - self.null_value) / se
        
        # Степени свободы
        df = self.n - 1
        
        # Критическое значение (приближение)
        if alternative == 'two-sided':
            t_critical = 2.0 if df < 30 else 1.96
            p_value_approx = 2 * (1 - 0.5 * (1 + math.erf(abs(t_stat) / math.sqrt(2))))
        else:
            t_critical = 1.65 if df < 30 else 1.64
            p_value_approx = 1 - 0.5 * (1 + math.erf(t_stat / math.sqrt(2)))
        
        return {
            'sample_mean': self.mean,
            'null_hypothesis': self.null_value,
            't_statistic': t_stat,
            'degrees_of_freedom': df,
            't_critical': t_critical,
            'p_value': p_value_approx,
            'significant': abs(t_stat) > t_critical,
            'reject_null': p_value_approx < alpha
        }
    
    def power_analysis(self, effect_size, alpha=0.05):
        """Анализ мощности теста"""
        # Упрощенный расчет мощности
        z_alpha = 1.96  # Для α = 0.05
        z_beta = effect_size * math.sqrt(self.n) - z_alpha
        power = 0.5 * (1 + math.erf(z_beta / math.sqrt(2)))
        
        return {
            'effect_size': effect_size,
            'sample_size': self.n,
            'power': power,
            'adequate_power': power >= 0.8
        }

# Пример: проверка производительности после оптимизации
# Время выполнения запросов до оптимизации: 150 мс
# Время выполнения после оптимизации:
optimized_times = [145, 138, 142, 140, 135, 148, 136, 143, 139, 141, 137, 144, 138, 142, 140]

test = HypothesisTest(optimized_times, 150)
result = test.one_sample_t_test()

print("Проверка гипотезы о производительности:")
print(f"H₀: μ = {test.null_value} мс (производительность не изменилась)")
print(f"H₁: μ ≠ {test.null_value} мс (производительность изменилась)")
print(f"Выборочное среднее: {result['sample_mean']:.2f} мс")
print(f"T-статистика: {result['t_statistic']:.3f}")
print(f"P-value: {result['p_value']:.4f}")
print(f"Результат: {'Отклоняем H₀' if result['reject_null'] else 'Не отклоняем H₀'}")

# Анализ мощности
effect_size = (test.mean - test.null_value) / test.std
power_result = test.power_analysis(effect_size)
print(f"\nАнализ мощности:")
print(f"Размер эффекта: {power_result['effect_size']:.3f}")
print(f"Мощность теста: {power_result['power']:.3f}")
print(f"Адекватная мощность: {power_result['adequate_power']}")
```

### 📊 Планирование эксперимента

```python
class ExperimentPlanner:
    @staticmethod
    def sample_size_calculator(effect_size, power=0.8, alpha=0.05):
        """Расчёт размера выборки"""
        # Упрощенная формула для двухвыборочного t-теста
        z_alpha = 1.96  # Для α = 0.05 (двусторонний тест)
        z_beta = 0.84   # Для мощности 80%
        
        # Приближенная формула
        n_per_group = 2 * ((z_alpha + z_beta) / effect_size) ** 2
        
        return {
            'n_per_group': math.ceil(n_per_group),
            'total_sample_size': math.ceil(n_per_group * 2),
            'effect_size': effect_size,
            'power': power,
            'alpha': alpha
        }
    
    @staticmethod
    def minimum_detectable_effect(n_per_group, power=0.8, alpha=0.05):
        """Минимальный обнаруживаемый эффект"""
        z_alpha = 1.96
        z_beta = 0.84
        
        mde = (z_alpha + z_beta) * math.sqrt(2 / n_per_group)
        
        return {
            'minimum_detectable_effect': mde,
            'n_per_group': n_per_group,
            'power': power,
            'alpha': alpha
        }

# Планирование A/B теста для конверсии
planner = ExperimentPlanner()

# Хотим обнаружить улучшение конверсии на 2%
baseline_conversion = 0.15  # 15%
target_conversion = 0.17    # 17%
effect_size = (target_conversion - baseline_conversion) / math.sqrt(baseline_conversion * (1 - baseline_conversion))

sample_size = planner.sample_size_calculator(effect_size)
print("Планирование A/B теста:")
print(f"Базовая конверсия: {baseline_conversion:.1%}")
print(f"Целевая конверсия: {target_conversion:.1%}")
print(f"Размер эффекта: {effect_size:.3f}")
print(f"Необходимый размер выборки: {sample_size['n_per_group']} на группу")
print(f"Общий размер: {sample_size['total_sample_size']} пользователей")

# Что можем обнаружить с текущим трафиком
current_traffic = 1000  # пользователей в день на группу
mde = planner.minimum_detectable_effect(current_traffic)
print(f"\nС текущим трафиком ({current_traffic} на группу):")
print(f"Минимальный обнаруживаемый эффект: {mde['minimum_detectable_effect']:.3f}")
```

---

## 3. Множественные сравнения

```python
class MultipleComparisons:
    @staticmethod
    def bonferroni_correction(p_values, alpha=0.05):
        """Поправка Бонферрони"""
        n_tests = len(p_values)
        corrected_alpha = alpha / n_tests
        
        significant = [p < corrected_alpha for p in p_values]
        
        return {
            'original_alpha': alpha,
            'corrected_alpha': corrected_alpha,
            'significant': significant,
            'n_significant': sum(significant),
            'family_wise_error_rate': alpha
        }
    
    @staticmethod
    def false_discovery_rate(p_values, alpha=0.05):
        """Контроль ложных открытий (FDR)"""
        n_tests = len(p_values)
        
        # Сортируем p-values с индексами
        indexed_p_values = [(p, i) for i, p in enumerate(p_values)]
        indexed_p_values.sort()
        
        significant = [False] * n_tests
        
        # Процедура Бенджамини-Хохберга
        for k, (p_value, original_index) in enumerate(indexed_p_values):
            threshold = (k + 1) / n_tests * alpha
            if p_value <= threshold:
                significant[original_index] = True
            else:
                break
        
        return {
            'alpha': alpha,
            'significant': significant,
            'n_significant': sum(significant),
            'expected_fdr': alpha
        }

# Пример: тестирование нескольких версий страницы
page_versions = ['A', 'B', 'C', 'D', 'E']
p_values = [0.03, 0.01, 0.07, 0.002, 0.08]

bonferroni = MultipleComparisons.bonferroni_correction(p_values)
fdr = MultipleComparisons.false_discovery_rate(p_values)

print("Множественные сравнения:")
print("Версия | P-value | Bonferroni | FDR")
print("-------|---------|------------|----")
for i, version in enumerate(page_versions):
    print(f"   {version}   | {p_values[i]:.3f}   |    {'Да' if bonferroni['significant'][i] else 'Нет'}     | {'Да' if fdr['significant'][i] else 'Нет'}")

print(f"\nПоправка Бонферрони: {bonferroni['n_significant']}/{len(page_versions)} значимых")
print(f"FDR контроль: {fdr['n_significant']}/{len(page_versions)} значимых")
```

---

## 📚 Связанные темы

- [[probability-theory|Основы теории вероятностей]] - фундамент тестирования
- [[descriptive-statistics|Описательная статистика]] - анализ данных
- [[bayesian-statistics|Байесовская статистика]] - альтернативный подход

---

## 🛠️ Инструменты

### Python библиотеки
```python
import scipy.stats as stats    # Статистические тесты
import statsmodels.api as sm   # Продвинутая статистика
import pingouin as pg         # Удобные статистические функции
from scipy.stats import ttest_ind, chi2_contingency
```

### Практические задания

1. **API Performance Test** - проверьте, улучшила ли оптимизация время ответа
2. **Feature Launch Analysis** - оцените влияние новой функции на метрики
3. **Multi-variant Testing** - сравните несколько версий интерфейса

---

## 🎯 Ключевые концепции

- **Нулевая гипотеза** - предположение об отсутствии эффекта
- **Статистическая значимость** - низкая вероятность случайного результата
- **Практическая значимость** - важность результата для бизнеса
- **Мощность теста** - способность обнаружить реальный эффект
- **Множественные сравнения** - контроль ошибок при многих тестах 