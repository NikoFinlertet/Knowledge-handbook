# 🛡️ Этика и безопасность ИИ

## ⚖️ Принципы этичного ИИ

### Основные принципы
```
Этические принципы ИИ:
├── Справедливость (Fairness)
├── Подотчетность (Accountability) 
├── Прозрачность (Transparency)
├── Объяснимость (Explainability)
├── Человеческий контроль
├── Конфиденциальность
└── Недискриминация
```

### Обнаружение предвзятости (Bias)
```python
import pandas as pd
import numpy as np
from sklearn.metrics import confusion_matrix, classification_report

def evaluate_bias(y_true, y_pred, sensitive_attributes):
    """Оценка предвзятости модели по защищенным атрибутам"""
    results = {}
    
    for attr_name, attr_values in sensitive_attributes.items():
        results[attr_name] = {}
        
        for value in np.unique(attr_values):
            mask = attr_values == value
            group_y_true = y_true[mask]
            group_y_pred = y_pred[mask]
            
            # Метрики для группы
            group_accuracy = np.mean(group_y_true == group_y_pred)
            
            # Положительная предсказанная ценность
            if np.sum(group_y_pred == 1) > 0:
                ppv = np.sum((group_y_true == 1) & (group_y_pred == 1)) / np.sum(group_y_pred == 1)
            else:
                ppv = 0
            
            results[attr_name][value] = {
                'accuracy': group_accuracy,
                'positive_predictive_value': ppv,
                'sample_size': len(group_y_true)
            }
    
    return results

# Пример использования
bias_results = evaluate_bias(y_test, predictions, {'gender': gender_data, 'age_group': age_groups})
```

### Метрики справедливости
```python
def fairness_metrics(y_true, y_pred, sensitive_attr):
    """Вычисление метрик справедливости"""
    
    # Demographic Parity
    positive_rate_group_0 = np.mean(y_pred[sensitive_attr == 0])
    positive_rate_group_1 = np.mean(y_pred[sensitive_attr == 1])
    demographic_parity = abs(positive_rate_group_0 - positive_rate_group_1)
    
    # Equal Opportunity
    tpr_group_0 = np.mean(y_pred[(sensitive_attr == 0) & (y_true == 1)])
    tpr_group_1 = np.mean(y_pred[(sensitive_attr == 1) & (y_true == 1)])
    equal_opportunity = abs(tpr_group_0 - tpr_group_1)
    
    # Equalized Odds
    fpr_group_0 = np.mean(y_pred[(sensitive_attr == 0) & (y_true == 0)])
    fpr_group_1 = np.mean(y_pred[(sensitive_attr == 1) & (y_true == 0)])
    equalized_odds = abs(equal_opportunity) + abs(fpr_group_0 - fpr_group_1)
    
    return {
        'demographic_parity': demographic_parity,
        'equal_opportunity': equal_opportunity,
        'equalized_odds': equalized_odds
    }
```

---

## 🔍 Объяснимость моделей (XAI)

### SHAP (SHapley Additive exPlanations)
```python
import shap
from sklearn.ensemble import RandomForestClassifier

# Обучение модели
model = RandomForestClassifier()
model.fit(X_train, y_train)

# SHAP объяснения
explainer = shap.TreeExplainer(model)
shap_values = explainer.shap_values(X_test)

# Визуализация
shap.summary_plot(shap_values[1], X_test, feature_names=feature_names)
shap.waterfall_plot(explainer.expected_value[1], shap_values[1][0], X_test.iloc[0])
```

### LIME (Local Interpretable Model-agnostic Explanations)
```python
import lime
import lime.lime_tabular

# Создание объяснителя
explainer = lime.lime_tabular.LimeTabularExplainer(
    X_train.values,
    feature_names=feature_names,
    class_names=['Class 0', 'Class 1'],
    mode='classification'
)

# Объяснение конкретного предсказания
explanation = explainer.explain_instance(
    X_test.iloc[0].values, 
    model.predict_proba,
    num_features=10
)

explanation.show_in_notebook(show_table=True)
```

---

## 🔒 Безопасность ИИ

### Adversarial Attacks
```python
import torch
import torch.nn.functional as F

def fgsm_attack(model, data, target, epsilon=0.1):
    """Fast Gradient Sign Method атака"""
    data.requires_grad_(True)
    
    output = model(data)
    loss = F.cross_entropy(output, target)
    
    model.zero_grad()
    loss.backward()
    
    # Создание adversarial примера
    perturbation = epsilon * data.grad.data.sign()
    adversarial_data = data + perturbation
    
    return adversarial_data

def pgd_attack(model, data, target, epsilon=0.1, alpha=0.01, num_iter=10):
    """Projected Gradient Descent атака"""
    adversarial_data = data.clone().detach()
    
    for _ in range(num_iter):
        adversarial_data.requires_grad_(True)
        output = model(adversarial_data)
        loss = F.cross_entropy(output, target)
        
        model.zero_grad()
        loss.backward()
        
        # Обновление
        adversarial_data = adversarial_data + alpha * adversarial_data.grad.sign()
        
        # Проекция в допустимую область
        perturbation = torch.clamp(adversarial_data - data, -epsilon, epsilon)
        adversarial_data = data + perturbation
        adversarial_data = torch.clamp(adversarial_data, 0, 1)
        adversarial_data = adversarial_data.detach()
    
    return adversarial_data
```

### Защита от атак
```python
def adversarial_training(model, train_loader, optimizer, epochs=10):
    """Adversarial обучение для повышения устойчивости"""
    model.train()
    
    for epoch in range(epochs):
        for batch_idx, (data, target) in enumerate(train_loader):
            # Обычные примеры
            optimizer.zero_grad()
            output = model(data)
            clean_loss = F.cross_entropy(output, target)
            
            # Adversarial примеры
            adversarial_data = fgsm_attack(model, data, target, epsilon=0.1)
            adv_output = model(adversarial_data)
            adv_loss = F.cross_entropy(adv_output, target)
            
            # Комбинированная функция потерь
            total_loss = 0.5 * clean_loss + 0.5 * adv_loss
            total_loss.backward()
            optimizer.step()
```

---

## 🔐 Конфиденциальность данных

### Differential Privacy
```python
import numpy as np

class DifferentialPrivacy:
    def __init__(self, epsilon=1.0):
        self.epsilon = epsilon
    
    def add_laplace_noise(self, value, sensitivity):
        """Добавление шума Лапласа"""
        scale = sensitivity / self.epsilon
        noise = np.random.laplace(0, scale)
        return value + noise
    
    def private_mean(self, data, sensitivity=1.0):
        """Приватное вычисление среднего"""
        true_mean = np.mean(data)
        return self.add_laplace_noise(true_mean, sensitivity / len(data))
    
    def private_histogram(self, data, bins):
        """Приватная гистограмма"""
        hist, _ = np.histogram(data, bins)
        private_hist = [self.add_laplace_noise(count, 1.0) for count in hist]
        return np.maximum(private_hist, 0)  # Убираем отрицательные значения

# Пример использования
dp = DifferentialPrivacy(epsilon=1.0)
private_avg = dp.private_mean(sensitive_data)
```

### Federated Learning
```python
class FederatedLearning:
    def __init__(self, global_model):
        self.global_model = global_model
        self.client_weights = []
    
    def client_update(self, client_data, client_labels, epochs=5):
        """Обновление модели на клиенте"""
        local_model = copy.deepcopy(self.global_model)
        optimizer = torch.optim.SGD(local_model.parameters(), lr=0.01)
        
        for epoch in range(epochs):
            optimizer.zero_grad()
            output = local_model(client_data)
            loss = F.cross_entropy(output, client_labels)
            loss.backward()
            optimizer.step()
        
        return local_model.state_dict()
    
    def federated_averaging(self, client_weights):
        """Федеративное усреднение весов"""
        global_weights = {}
        
        # Инициализация
        for key in client_weights[0].keys():
            global_weights[key] = torch.zeros_like(client_weights[0][key])
        
        # Усреднение
        for client_weight in client_weights:
            for key in global_weights.keys():
                global_weights[key] += client_weight[key]
        
        for key in global_weights.keys():
            global_weights[key] = global_weights[key] / len(client_weights)
        
        self.global_model.load_state_dict(global_weights)
```

---

## ⚠️ Оценка рисков

### Матрица рисков ИИ
```python
class AIRiskAssessment:
    def __init__(self):
        self.risk_categories = {
            'bias_discrimination': 'Предвзятость и дискриминация',
            'privacy_breach': 'Нарушение конфиденциальности',
            'security_vulnerability': 'Уязвимости безопасности',
            'algorithmic_harm': 'Алгоритмический вред',
            'human_autonomy': 'Угроза человеческой автономии'
        }
    
    def assess_risk(self, model_type, use_case, data_sensitivity):
        """Оценка рисков ИИ системы"""
        risk_scores = {}
        
        # Факторы риска
        high_risk_models = ['deep_learning', 'ensemble', 'neural_network']
        sensitive_use_cases = ['healthcare', 'finance', 'criminal_justice', 'hiring']
        
        for risk_type in self.risk_categories:
            base_risk = 0.3  # Базовый уровень риска
            
            # Увеличение риска в зависимости от факторов
            if model_type in high_risk_models:
                base_risk += 0.2
            
            if use_case in sensitive_use_cases:
                base_risk += 0.3
            
            if data_sensitivity == 'high':
                base_risk += 0.2
            
            risk_scores[risk_type] = min(base_risk, 1.0)
        
        return risk_scores
    
    def generate_mitigation_plan(self, risk_scores):
        """Генерация плана снижения рисков"""
        mitigation_strategies = []
        
        for risk_type, score in risk_scores.items():
            if score > 0.7:
                strategies = self.get_mitigation_strategies(risk_type)
                mitigation_strategies.extend(strategies)
        
        return mitigation_strategies
    
    def get_mitigation_strategies(self, risk_type):
        """Стратегии снижения конкретных рисков"""
        strategies = {
            'bias_discrimination': [
                'Аудит данных на предвзятость',
                'Использование метрик справедливости',
                'Разнообразие в команде разработки'
            ],
            'privacy_breach': [
                'Дифференциальная приватность',
                'Федеративное обучение',
                'Анонимизация данных'
            ],
            'security_vulnerability': [
                'Adversarial training',
                'Регулярные security аудиты',
                'Валидация входных данных'
            ]
        }
        
        return strategies.get(risk_type, [])
```

---

## 📋 Чек-лист этичного ИИ

```
✅ Этический чек-лист:

📊 Данные:
├── Проверка на предвзятость в данных
├── Согласие на использование данных
├── Минимизация сбора данных
├── Обеспечение качества данных
└── Документирование источников данных

🤖 Модель:
├── Тестирование справедливости
├── Обеспечение объяснимости
├── Валидация на различных группах
├── Мониторинг дрифта
└── Версионирование и аудит

🚀 Развертывание:
├── Человеческий надзор
├── Возможность отмены решений
├── Прозрачность для пользователей
├── Обратная связь и жалобы
└── Регулярные проверки

🔒 Безопасность:
├── Защита от adversarial атак
├── Конфиденциальность данных
├── Контроль доступа
├── Мониторинг безопасности
└── План реагирования на инциденты
```

---

## 📚 Связанные темы
- [[ai-in-production|AI в продакшене]] - безопасное развертывание
- [[large-language-models|LLM]] - этические аспекты языковых моделей
- [[ai-fundamentals|Основы AI]] - принципы ответственного ИИ

---

💡 **Совет:** Этика и безопасность должны закладываться с самого начала разработки ИИ системы, а не добавляться в конце! 