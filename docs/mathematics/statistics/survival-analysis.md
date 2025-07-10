# Анализ выживаемости 📈

> [[advanced-methods|← Продвинутые методы]] | [[README|Оглавление]]

## 🎯 Что изучим

- **Анализ выживаемости** - моделирование времени до события
- **Retention анализ** - удержание пользователей и клиентов
- **Churn prediction** - прогнозирование оттока
- **A/B тесты с временными данными** - тестирование retention

---

## 1. Основы анализа выживаемости

```python
class SurvivalAnalysis:
    @staticmethod
    def kaplan_meier(times, events):
        """Оценка Каплана-Мейера для функции выживания"""
        
        # Создаем список уникальных времен событий
        event_times = sorted(set(t for t, e in zip(times, events) if e == 1))
        
        survival_function = []
        n_at_risk = len(times)
        survival_prob = 1.0
        
        for t in event_times:
            # Количество событий в момент времени t
            events_at_t = sum(1 for time, event in zip(times, events) 
                             if time == t and event == 1)
            
            # Обновляем вероятность выживания
            if n_at_risk > 0:
                survival_prob *= (n_at_risk - events_at_t) / n_at_risk
            
            survival_function.append({
                'time': t,
                'survival_probability': survival_prob,
                'events': events_at_t,
                'at_risk': n_at_risk
            })
            
            # Обновляем количество в группе риска
            n_at_risk -= sum(1 for time in times if time == t)
        
        return survival_function
    
    @staticmethod
    def median_survival_time(survival_function):
        """Медианное время выживания"""
        for point in survival_function:
            if point['survival_probability'] <= 0.5:
                return point['time']
        return None  # Медиана не достигнута
    
    @staticmethod
    def survival_at_time(survival_function, target_time):
        """Вероятность выживания в определенный момент времени"""
        last_prob = 1.0
        for point in survival_function:
            if point['time'] > target_time:
                return last_prob
            last_prob = point['survival_probability']
        return last_prob

# Пример: анализ retention пользователей
# times - дни до churn, events - 1 если churn произошел, 0 если цензурировано
user_data = [
    (7, 1), (14, 1), (21, 0), (30, 1), (45, 0),    # Дни, ушел ли пользователь
    (60, 1), (90, 0), (120, 1), (180, 0), (365, 0)
]

times = [t for t, e in user_data]
events = [e for t, e in user_data]

survival_curve = SurvivalAnalysis.kaplan_meier(times, events)

print("Анализ выживаемости пользователей:")
for point in survival_curve:
    print(f"День {point['time']:3d}: {point['survival_probability']:.3f} "
          f"(события: {point['events']}, в риске: {point['at_risk']})")

# Медианное время retention
median_time = SurvivalAnalysis.median_survival_time(survival_curve)
if median_time:
    print(f"\nМедианное время retention: {median_time} дней")
else:
    print("\nМедианное время retention не достигнуто (>50% пользователей остаются)")

# Retention на ключевых временных точках
key_timepoints = [30, 90, 180]
print("\nRetention в ключевые моменты:")
for days in key_timepoints:
    retention = SurvivalAnalysis.survival_at_time(survival_curve, days)
    print(f"  {days} дней: {retention:.1%}")
```

---

## 2. Retention анализ

```python
class RetentionAnalysis:
    @staticmethod
    def cohort_retention(user_cohorts):
        """Когортный анализ retention"""
        cohort_data = {}
        
        for cohort_month, users in user_cohorts.items():
            cohort_data[cohort_month] = {}
            initial_size = len(users)
            
            # Подсчет активных пользователей по месяцам
            for month_offset in range(12):  # Анализируем 12 месяцев
                active_users = sum(1 for user in users 
                                 if user.get('active_months', 0) > month_offset)
                
                retention_rate = active_users / initial_size if initial_size > 0 else 0
                cohort_data[cohort_month][f'month_{month_offset}'] = {
                    'active_users': active_users,
                    'retention_rate': retention_rate,
                    'initial_size': initial_size
                }
        
        return cohort_data
    
    @staticmethod
    def calculate_ltv(monthly_revenue, churn_rate):
        """Расчет Lifetime Value"""
        if churn_rate <= 0 or churn_rate >= 1:
            return float('inf') if churn_rate <= 0 else 0
        
        # LTV = Average Revenue Per User / Churn Rate
        ltv = monthly_revenue / churn_rate
        return ltv
    
    @staticmethod
    def retention_metrics(active_users_timeline):
        """Основные метрики retention"""
        if not active_users_timeline:
            return {}
        
        # Day 1, Day 7, Day 30 retention
        retention_points = [1, 7, 30]
        metrics = {}
        
        initial_users = active_users_timeline[0] if active_users_timeline else 0
        
        for day in retention_points:
            if day < len(active_users_timeline):
                active_at_day = active_users_timeline[day]
                retention_rate = active_at_day / initial_users if initial_users > 0 else 0
                metrics[f'day_{day}_retention'] = retention_rate
            else:
                metrics[f'day_{day}_retention'] = 0
        
        # Churn rate (обратный показатель retention)
        if 'day_7_retention' in metrics:
            metrics['weekly_churn_rate'] = 1 - metrics['day_7_retention']
        
        if 'day_30_retention' in metrics:
            metrics['monthly_churn_rate'] = 1 - metrics['day_30_retention']
        
        return metrics

# Пример: анализ retention мобильного приложения
app_cohorts = {
    '2024-01': [
        {'user_id': 1, 'active_months': 6},
        {'user_id': 2, 'active_months': 3},
        {'user_id': 3, 'active_months': 8},
        {'user_id': 4, 'active_months': 1},
        {'user_id': 5, 'active_months': 12}
    ],
    '2024-02': [
        {'user_id': 6, 'active_months': 5},
        {'user_id': 7, 'active_months': 2},
        {'user_id': 8, 'active_months': 7},
        {'user_id': 9, 'active_months': 4}
    ]
}

retention_data = RetentionAnalysis.cohort_retention(app_cohorts)

print("Когортный анализ retention:")
for cohort, data in retention_data.items():
    print(f"\nКогорта {cohort}:")
    print(f"  Начальный размер: {data['month_0']['initial_size']}")
    for month in [0, 1, 3, 6]:
        if f'month_{month}' in data:
            rate = data[f'month_{month}']['retention_rate']
            print(f"  Месяц {month}: {rate:.1%}")

# Расчет LTV
average_monthly_revenue = 15  # $15 в месяц на пользователя
monthly_churn = 0.05  # 5% в месяц

ltv = RetentionAnalysis.calculate_ltv(average_monthly_revenue, monthly_churn)
print(f"\nLifetime Value: ${ltv:.0f}")

# Дневной retention
daily_active_users = [1000, 850, 750, 700, 650, 600, 580, 560]  # День 0-7
retention_metrics = RetentionAnalysis.retention_metrics(daily_active_users)

print("\nMetrics retention:")
for metric, value in retention_metrics.items():
    if 'retention' in metric:
        print(f"  {metric}: {value:.1%}")
    elif 'churn' in metric:
        print(f"  {metric}: {value:.1%}")
```

---

## 3. Churn Prediction

```python
class ChurnPredictor:
    def __init__(self):
        self.features = []
        self.weights = []
        self.threshold = 0.5
    
    def extract_features(self, user_data):
        """Извлечение признаков для прогнозирования churn"""
        features = []
        
        for user in user_data:
            feature_vector = [
                user.get('days_since_last_login', 0),
                user.get('session_frequency', 0),
                user.get('total_sessions', 0),
                user.get('average_session_duration', 0),
                user.get('feature_usage_score', 0),
                user.get('support_tickets', 0),
                1 if user.get('premium_user', False) else 0
            ]
            features.append(feature_vector)
        
        return features
    
    def simple_logistic_regression(self, features, labels, learning_rate=0.01, iterations=1000):
        """Упрощенная логистическая регрессия"""
        import math
        
        n_features = len(features[0])
        self.weights = [0.0] * (n_features + 1)  # +1 для bias
        
        for _ in range(iterations):
            for i, (feature_vector, label) in enumerate(zip(features, labels)):
                # Добавляем bias term
                x = [1.0] + feature_vector
                
                # Предсказание
                z = sum(w * xi for w, xi in zip(self.weights, x))
                prediction = 1 / (1 + math.exp(-min(max(z, -250), 250)))  # Sigmoid с ограничениями
                
                # Обновление весов
                error = label - prediction
                for j in range(len(self.weights)):
                    self.weights[j] += learning_rate * error * x[j]
    
    def predict_churn(self, user_features):
        """Предсказание вероятности churn"""
        import math
        
        if not self.weights:
            return 0.5  # Нет обученной модели
        
        x = [1.0] + user_features  # Добавляем bias
        z = sum(w * xi for w, xi in zip(self.weights, x))
        
        # Sigmoid функция с защитой от переполнения
        try:
            probability = 1 / (1 + math.exp(-z))
        except OverflowError:
            probability = 0.0 if z < 0 else 1.0
        
        return probability
    
    def risk_segmentation(self, users_data):
        """Сегментация пользователей по риску churn"""
        risk_segments = {'low': [], 'medium': [], 'high': []}
        
        for user in users_data:
            features = self.extract_features([user])[0]
            churn_prob = self.predict_churn(features)
            
            user_risk = {
                'user_id': user.get('user_id'),
                'churn_probability': churn_prob,
                'features': features
            }
            
            if churn_prob < 0.3:
                risk_segments['low'].append(user_risk)
            elif churn_prob < 0.7:
                risk_segments['medium'].append(user_risk)
            else:
                risk_segments['high'].append(user_risk)
        
        return risk_segments

# Пример данных пользователей
training_users = [
    {'days_since_last_login': 1, 'session_frequency': 5, 'total_sessions': 100, 
     'average_session_duration': 15, 'feature_usage_score': 8, 'support_tickets': 0, 
     'premium_user': True, 'churned': 0},
    
    {'days_since_last_login': 15, 'session_frequency': 1, 'total_sessions': 20, 
     'average_session_duration': 5, 'feature_usage_score': 3, 'support_tickets': 2, 
     'premium_user': False, 'churned': 1},
     
    {'days_since_last_login': 3, 'session_frequency': 4, 'total_sessions': 80, 
     'average_session_duration': 12, 'feature_usage_score': 7, 'support_tickets': 0, 
     'premium_user': True, 'churned': 0},
     
    {'days_since_last_login': 30, 'session_frequency': 0.5, 'total_sessions': 10, 
     'average_session_duration': 3, 'feature_usage_score': 2, 'support_tickets': 3, 
     'premium_user': False, 'churned': 1}
]

# Обучение модели
predictor = ChurnPredictor()
features = predictor.extract_features(training_users)
labels = [user['churned'] for user in training_users]

predictor.simple_logistic_regression(features, labels)

# Предсказание для новых пользователей
new_users = [
    {'user_id': 'user_001', 'days_since_last_login': 7, 'session_frequency': 3, 
     'total_sessions': 50, 'average_session_duration': 10, 'feature_usage_score': 6, 
     'support_tickets': 1, 'premium_user': False},
     
    {'user_id': 'user_002', 'days_since_last_login': 25, 'session_frequency': 0.8, 
     'total_sessions': 15, 'average_session_duration': 4, 'feature_usage_score': 2, 
     'support_tickets': 4, 'premium_user': False}
]

# Сегментация по риску
risk_segments = predictor.risk_segmentation(new_users)

print("Сегментация пользователей по риску churn:")
for risk_level, users in risk_segments.items():
    print(f"\n{risk_level.upper()} риск ({len(users)} пользователей):")
    for user in users:
        print(f"  {user['user_id']}: {user['churn_probability']:.3f}")

# Рекомендации по retention
print("\nРекомендации по retention:")
if risk_segments['high']:
    print("🚨 Высокий риск: Персональные предложения, поддержка клиентов")
if risk_segments['medium']:
    print("⚠️ Средний риск: Email-кампании, feature highlighting")
if risk_segments['low']:
    print("✅ Низкий риск: Программы лояльности, upselling")
```

---

## 4. A/B тесты с временными данными

```python
class TimeBasedABTest:
    @staticmethod
    def retention_ab_test(control_group, treatment_group, time_periods):
        """A/B тест для retention метрик"""
        
        def calculate_retention_curve(group, periods):
            curve = []
            initial_size = len(group)
            
            for period in periods:
                retained = sum(1 for user in group 
                             if user.get('retention_days', 0) >= period)
                retention_rate = retained / initial_size if initial_size > 0 else 0
                curve.append(retention_rate)
            
            return curve
        
        control_curve = calculate_retention_curve(control_group, time_periods)
        treatment_curve = calculate_retention_curve(treatment_group, time_periods)
        
        # Сравнение кривых retention
        improvements = []
        for i, period in enumerate(time_periods):
            control_rate = control_curve[i]
            treatment_rate = treatment_curve[i]
            
            lift = ((treatment_rate - control_rate) / control_rate * 100 
                   if control_rate > 0 else 0)
            
            improvements.append({
                'period': period,
                'control_retention': control_rate,
                'treatment_retention': treatment_rate,
                'lift_percent': lift
            })
        
        return improvements
    
    @staticmethod
    def ltv_comparison(control_ltv_data, treatment_ltv_data):
        """Сравнение LTV между группами"""
        control_avg_ltv = sum(control_ltv_data) / len(control_ltv_data)
        treatment_avg_ltv = sum(treatment_ltv_data) / len(treatment_ltv_data)
        
        ltv_lift = ((treatment_avg_ltv - control_avg_ltv) / control_avg_ltv * 100 
                   if control_avg_ltv > 0 else 0)
        
        return {
            'control_avg_ltv': control_avg_ltv,
            'treatment_avg_ltv': treatment_avg_ltv,
            'ltv_lift_percent': ltv_lift,
            'significant': abs(ltv_lift) > 5  # Простая эвристика
        }

# Пример: A/B тест новой onboarding последовательности
control_users = [
    {'user_id': f'c_{i}', 'retention_days': days} 
    for i, days in enumerate([7, 14, 30, 45, 60, 90, 120])
]

treatment_users = [
    {'user_id': f't_{i}', 'retention_days': days} 
    for i, days in enumerate([14, 21, 35, 50, 75, 105, 135])  # Улучшенный retention
]

test_periods = [7, 14, 30, 60, 90]
retention_results = TimeBasedABTest.retention_ab_test(
    control_users, treatment_users, test_periods
)

print("A/B тест retention:")
for result in retention_results:
    print(f"День {result['period']:2d}: "
          f"Контроль {result['control_retention']:.1%}, "
          f"Тест {result['treatment_retention']:.1%}, "
          f"Прирост {result['lift_percent']:+.1f}%")

# Сравнение LTV
control_ltv = [150, 120, 180, 200, 160, 140, 190]
treatment_ltv = [180, 150, 210, 240, 200, 170, 220]

ltv_comparison = TimeBasedABTest.ltv_comparison(control_ltv, treatment_ltv)
print(f"\nСравнение LTV:")
print(f"Контроль: ${ltv_comparison['control_avg_ltv']:.0f}")
print(f"Тест: ${ltv_comparison['treatment_avg_ltv']:.0f}")
print(f"Прирост: {ltv_comparison['ltv_lift_percent']:+.1f}%")
print(f"Значимо: {ltv_comparison['significant']}")
```

---

## 📚 Практические применения

### 🎯 В продуктовой аналитике
- **User retention**: Анализ удержания пользователей
- **Feature adoption**: Время до принятия новых функций
- **Customer journey**: Анализ пути клиента

### 💼 В бизнес-аналитике
- **Churn prediction**: Прогнозирование оттока клиентов
- **LTV optimization**: Оптимизация жизненной ценности
- **Cohort analysis**: Сравнение поколений пользователей

---

## 🔗 Связанные темы

- [[time-series|Временные ряды]] - анализ динамики данных
- [[statistical-tests|Статистические тесты]] - A/B тестирование
- [[practical-applications|Практические применения]] - реальные кейсы

---

## 💡 Ключевые метрики

- **Retention Rate**: Процент пользователей, остающихся активными
- **Churn Rate**: Процент пользователей, прекращающих использование
- **LTV (Lifetime Value)**: Общая ценность клиента за время жизни
- **Cohort Analysis**: Сравнение групп пользователей по времени регистрации

> 📈 **Анализ выживаемости**: Понимание жизненного цикла пользователей для принятия стратегических решений! 